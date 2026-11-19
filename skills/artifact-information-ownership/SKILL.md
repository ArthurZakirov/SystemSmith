---
name: artifact-information-ownership
description: Use when reviewing a diff or turning a ticket or conversation into code, tests, docs, commits, or a PR. Decide what to change, what to test, and which details can stay in the ticket.
---

# 🧭 Artifact Information Ownership

**Contents**

- [🛠️ How to work from a ticket](#workflow)
- [🔎 How to review a diff](#review)
- [🗂️ Where each part goes](#routing)
- [🧩 Example: one Jira story, two commits](#example)

Start with the ticket when implementing a request. Start with the changed lines when reviewing a diff. Both workflows use the same routing table. The CSV example follows one ticket through two commits and a PR.

<a id="workflow"></a>

## 🛠️ How to work from a ticket

1. **Read the request one piece at a time.**

   > *“Copy the CSV reader from the reports repo by Friday.”*

   - **Source.** The reports repo contains the reader.
   - **Deadline.** The work is due on Friday.

2. **Decide what to do with each piece.** Use the table in [🗂️ Where each part goes](#routing).

   - *“Copy the CSV reader”* — Reference Code Context.
   - *“By Friday”* — Project Management Context. If Friday is already in Jira, that part needs no new text.

3. **Decide who does it.**

   - **Agent.** Copy the reader and run its tests. If the team needs a deadline update, give the user a draft privately.
   - **User.** Decide whether to post the draft in Slack or Jira. The agent sends it only on explicit request.

4. **Carry out the agreed work.** Complete the agent’s tasks and hand the user any draft or decision assigned to them in step 3.

<a id="review"></a>

## 🔎 How to review a diff

1. **Open the diff.** Inspect the changed lines, including test names, comments, examples, and generated docs.
2. **Check each piece against the table.** Find the matching row in [🗂️ Where each part goes](#routing). Compare the file where the text appears with the place that row identifies.
3. **Report each mismatch.** Point to the line and suggest the edit. If the user asked for fixes, apply the edit and check the resulting diff.

<a id="routing"></a>

## 🗂️ Where each part goes

Use the same question for requests you haven't seen before: who will use this detail, and what will they do with it (e.g., a caller needs the timeout default to decide whether to send a value)?

| Category | Words you might hear | What to do with them |
| --- | --- | --- |
| Project Management Context | *“Finish by Friday.”* | • Keep Friday in the Jira story.<br>• Use the team's Slack thread if someone needs a scheduling update and the user asks you to send it. |
| Project Management Context | *“Try three retries for one day. Tomorrow we'll decide whether to keep them.”* | • Set the retry count to three.<br>• Put tomorrow's review in the experiment plan.<br>• If the user asks for automatic expiry, schedule the switch back and check the retry count after it runs. |
| Stakeholder / Customer Context | *“Max asked for uploads. Peter puts the spreadsheet in Drive.”* | • Name the feature for its job (e.g., spreadsheet upload).<br>• If operators need upload instructions, tell them which folder to use.<br>• Include Peter's name in a handover if he is the person to contact for access. |
| Stakeholder / Customer Context | *“Customer ABC needs a separate upload folder.”* | • Use ABC in configuration if the application looks up ABC's folder by customer ID.<br>• If ABC simply suggested a feature for everyone, use the feature's name in code.<br>• Keep names where readers are required to see them (e.g., an author's copyright notice in a license). |
| Reference Code Context · Migration | *“Copy the reader from the reports repo.”* | • Use that reader as the starting point.<br>• Describe how to call the reader in its new home.<br>• If the source repo is already linked in the story, that link is enough.<br>• If operators must move files during a migration, give them the steps in the migration guide. |
| Reference Code Context · Reference | *“Use the same validation pattern as the reports repo.”* | • Read its validator and apply the pattern.<br>• Document the rule a caller needs (e.g., a missing header raises an error).<br>• The story already explains why you looked at the reports repo. |
| Code Behavior Change | *“The endpoint now accepts a timeout.”* | • Implement and test the field.<br>• If callers need instructions, write *“Send `timeout_seconds` to set the timeout.”*<br>• A commit message or PR can say *“Add an optional timeout to the endpoint.”* |
| Code Behavior Preservation | *“Requests without a timeout must still work.”* | • Pick an input and expected result (e.g., omit the timeout and expect 30 seconds).<br>• Run that test before and after editing production code.<br>• See [🧩 Example: one Jira story, two commits](#example) for the two runs. |
| Remaining Tasks / TODOs | *“We still need to deploy and pin the release.”* | • Keep those unchecked tasks in the private list shared by the user and agent, outside Git.<br>• The user chooses what to share at standup or handover.<br>• If every release uses the same pinning command, the release runbook can document that command. |
| Cross-Repo Context | *“The config repo supplies the settings this service reads.”* | • Document the settings this service expects (e.g., `bucket`, `region`, and `timeout_seconds`) and link to the config repo.<br>• The provider can rename its Python files without changing those settings.<br>• If several teams need the whole flow, draw it once on their shared architecture page. |

<a id="example"></a>

## 🧩 Example: one Jira story, two commits

> *“For DATA-123, copy the CSV reader from the reports repo by the end of today. Let callers supply a delimiter. Files with no delimiter setting must still read as comma-separated.”*

The story contains five pieces. Read the two code requirements below in commit order: first test the comma case, then add the delimiter option. In this example the target repo already has a comma-only reader.

### Jira ticket · Project Management Context

> *“For DATA-123”*

The agent opens DATA-123 to read the request. When preparing the PR, link it to DATA-123 so the reviewer can open the story. The reader function needs no ticket ID.

### Source reader · Reference Code Context

> *“copy the CSV reader from the reports repo”*

The agent opens the reader in the reports repo and checks how it passes a delimiter to Python’s CSV parser. The code to edit is the target repo’s `reader.py`. Before either commit, it reads:

```python
import csv


def read_header(line):
    return next(csv.reader([line]))
```

### Deadline · Project Management Context

> *“by the end of today.”*

Today is the due date on DATA-123. Leave it there. If the tests are still failing at the end of the day, tell the user privately which test failed. The user decides whether to update the team.

### Comma-separated files · Code Behavior Preservation

> *“Files with no delimiter setting must still read as comma-separated.”*

**Commit 1.** Leave `reader.py` alone. In this example the comma case has no test yet. Create `test_reader.py` with a call that omits the delimiter:

```python
from unittest import TestCase

from reader import read_header


class HeaderTests(TestCase):
    def test_missing_delimiter_reads_comma_separated_columns(self):
        self.assertEqual(read_header("name,city"), ["name", "city"])
```

Run the test and save it in its own commit:

```sh
python3 -m unittest -v test_reader
git add test_reader.py
git commit -m "Test CSV header with no delimiter setting"
```

The test passes if the reader returns two strings. Returning one string containing the comma would fail the assertion.

### Caller-supplied delimiter · Code Behavior Change

> *“Let callers supply a delimiter.”*

**Commit 2.** In `reader.py`, pass the caller’s delimiter to the parser. Give the parameter a comma as its default:

```python
import csv


def read_header(line, delimiter=","):
    return next(csv.reader([line], delimiter=delimiter))
```

Add this method to `HeaderTests` in `test_reader.py`:

```python
    def test_pipe_delimiter_reads_pipe_separated_columns(self):
        self.assertEqual(read_header("name|city", delimiter="|"), ["name", "city"])
```

Run both tests and commit the change:

```sh
python3 -m unittest -v test_reader
git add reader.py test_reader.py
git commit -m "Accept a delimiter when reading CSV headers"
```

**PR description.** Record the command and the results from the two commits. In this example, the first run passed the comma test. The second run passed that same test and the pipe test. If callers need a usage example, show the call directly:

```python
read_header("name|city", delimiter="|")
```

For documentation that can go stale (e.g., a copied list of supported settings), see [Living Artifacts](../living-artifacts/SKILL.md).
