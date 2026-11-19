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

| Category | Words you might hear | What to do with them | What not to do with them |
| --- | --- | --- | --- |
| Project Management Context | *“Finish by Friday.”* | **Jira story:** Keep the due date there. <br>**Private chat with the user:** Draft a scheduling update if needed. Send it to the team’s Slack thread only on explicit request. | Add a comment to the reader: *“Must ship by Friday.”* |
| Project Management Context | *“Try three retries for one day. Tomorrow we'll decide whether to keep them.”* | **Committed retry code or config:** Set the value the service reads (e.g., `max_retries = 3`). <br>**Private, uncommitted Markdown TODO:** Add *“Tomorrow: review retry failures and decide whether to keep three retries.”* <br>**Deployment scheduler:** If automatic expiry is requested, schedule the reset and verify the live setting afterward. | Write *“Try three retries tomorrow”* in the README while the service still uses its old retry count. Or put tomorrow’s review reminder in a source comment. |
| Stakeholder / Customer Context | *“Max asked for uploads. Peter puts the spreadsheet in Drive.”* | **Committed code:** Name the function `upload_spreadsheet`. <br>**Operator guide:** Give the Drive folder operators should use. <br>**Handover note:** Name Peter if the next operator needs to ask him for access. | Name the function `upload_for_max` because Max requested it, or copy Peter’s daily routine into its docstring. |
| Stakeholder / Customer Context | *“Customer ABC needs a separate upload folder.”* | **Tenant config:** Map ABC’s customer ID to its folder if the service selects folders by customer. <br>**Committed code:** Use a shared upload function if all customers use the same behavior. Keep required attribution in its designated file (e.g., an author’s notice in the license). | Rename the shared function `upload_for_abc` just because ABC first asked for it. |
| Reference Code Context · Migration | *“Copy the reader from the reports repo.”* | **Target repo:** Adapt the reader in source code and show its invocation in the usage docs if needed. <br>**Jira story:** Keep the source link there. <br>**Migration guide:** List any file moves operators must perform. | Start the function’s docstring with *“Copied from the reports repo for this story.”* |
| Reference Code Context · Reference | *“Use the same validation pattern as the reports repo.”* | **Target repo’s validator:** Implement the check. <br>**API reference:** Explain the caller-visible result (e.g., a missing header raises an error). <br>**Jira story:** Keep the reference to the original validator there. | Write *“Validate like the reports repo”* in the API reference and leave callers guessing which headers they must send. |
| Code Behavior Change | *“The endpoint now accepts a timeout.”* | **Committed endpoint and unit test:** Accept the field and test a supplied value. <br>**API reference:** Show how to send `timeout_seconds`. <br>**Commit message or PR:** Describe the change as *“Add an optional timeout to the endpoint.”* | Add *“We extended the model in this sprint”* to its docstring. |
| Code Behavior Preservation | *“Requests without a timeout must still work.”* | **Committed test:** Omit the timeout and assert 30 seconds. <br>**Test runner:** Run that test on both commits. <br>**PR description:** Record the command, revisions, and results as in [🧩 Example: one Jira story, two commits](#example). | Name a test `test_still_works_after_adding_timeout` and claim the old behavior was verified after running it only on the new code. |
| Remaining Tasks / TODOs | *“We still need to deploy and pin the release.”* | **Private, uncommitted TODO:** List the pending deployment and release pin. <br>**Standup or handover:** The user chooses what to share. <br>**Release runbook:** Document the pinning command if it applies to every release. | Commit this week’s unchecked deployment tasks to the README or post the private checklist to Slack unasked. |
| Cross-Repo Context | *“The config repo supplies the settings this service reads.”* | **Consumer repo’s config reference:** List the keys it reads (e.g., `bucket`, `region`, and `timeout_seconds`) and link to the provider repo. <br>**Shared architecture page:** Show the flow if several teams need it. | Copy the provider’s internal Python file paths into the consumer README. A provider-side rename would make those instructions stale. |

<a id="example"></a>

## 🧩 Example: one Jira story, two commits

> *“For DATA-123, copy the CSV reader from the reports repo by the end of today. Let callers supply a delimiter. Files with no delimiter setting must still read as comma-separated.”*

Read the rows in ticket order. Run the comma test in commit 1 before adding the delimiter option in commit 2.

| Words from the story | Category | Steps |
| --- | --- | --- |
| *“For DATA-123”* | Project Management Context | Open DATA-123. Link the story in the PR so the reviewer can read the request. |
| *“copy the CSV reader from the reports repo”* | Reference Code Context · Migration | Open the reports reader and check how it passes a delimiter to Python’s parser. Compare it with the [target reader before either commit](#csv-baseline). |
| *“by the end of today.”* | Project Management Context | Keep the due date in DATA-123. If tests are still failing at the deadline, tell the user privately which test failed. The user decides whether to update the team. |
| *“Let callers supply a delimiter.”* | Code Behavior Change | In [commit 2](#csv-change), add the optional parameter in `reader.py`, test a caller-supplied delimiter, and rerun the comma test. |
| *“Files with no delimiter setting must still read as comma-separated.”* | Code Behavior Preservation | In [commit 1](#csv-test), add a test in `test_reader.py` that omits the delimiter. Run it against the existing reader and commit the passing test. |

<a id="csv-baseline"></a>

### Target reader before either commit

In this example, the target repo’s `reader.py` already reads comma-separated headers:

```python
import csv


def read_header(line):
    return next(csv.reader([line]))
```

<a id="csv-test"></a>

### Commit 1 · Test the comma default

Leave `reader.py` alone. In this example the comma case has no test yet. Create `test_reader.py` with a call that omits the delimiter:

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

<a id="csv-change"></a>

### Commit 2 · Accept the caller’s delimiter

In `reader.py`, pass the caller’s delimiter to the parser. Give the parameter a comma as its default:

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
