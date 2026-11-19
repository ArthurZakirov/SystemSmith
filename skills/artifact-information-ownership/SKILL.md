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

   > *“Copy the `read_header` function from the reports repo’s `reader.py` by Friday.”*

   - **Source.** Open `reader.py` in the reports repo and find `def read_header`.
   - **Deadline.** The work is due on Friday.

2. **Decide what to do with each piece.** Use the table in [🗂️ Where each part goes](#routing).

   - *“Copy the `read_header` function”* — Reference Code Context.
   - *“By Friday”* — Project Management Context. If Friday is already in Jira, that part needs no new text.

3. **Decide who does it.**

   - **Agent.** Copy `read_header` into the destination repo’s `reader.py` and run `python3 -m unittest -v test_reader`. If the team needs a deadline update, give the user a draft privately.
   - **User.** Decide whether to post the draft in Slack or Jira. The agent sends it only on explicit request.

4. **Carry out the agreed work.** Make the code and test changes assigned to the agent. If the user needs to update the team, give them the proposed Slack message in private chat.

<a id="review"></a>

## 🔎 How to review a diff

1. **Open the diff.** Inspect the changed lines, including test names, comments, examples, and generated docs.
2. **Check each piece against the table.** Find the matching row in [🗂️ Where each part goes](#routing). Check where the text was added (e.g., a comment in `reader.py`) against the location named in that row (e.g., the due-date field in Jira).
3. **Report each mismatch.** Name the file and line, quote the wording, and say what to remove, move, or rewrite. If the user asked for fixes, apply the edit and check the resulting diff.

<a id="routing"></a>

## 🗂️ Where each part goes

Use the same question for requests you haven't seen before: who will use this detail, and what will they do with it (e.g., a caller needs the timeout default to decide whether to send a value)?

| Category | Words you might hear | What to do with them | What not to do with them |
| --- | --- | --- | --- |
| Project Management Context | *“Finish by Friday.”* | **Jira story:** Keep the due date there. <br>**Private chat with the user:** Draft a scheduling update if needed. Send it to the team’s Slack thread only on explicit request. | Write `# Must ship by Friday` above `def read_header` in `reader.py`. |
| Project Management Context | *“Try three retries for one day. Tomorrow we'll decide whether to keep them.”* | **Committed retry code or config:** Set the value the service reads (e.g., `max_retries = 3`). <br>**Private, uncommitted Markdown TODO:** Add *“Tomorrow: review retry failures and decide whether to keep three retries.”* <br>**Deployment scheduler:** If automatic expiry is requested, schedule a job to restore the previous retry count, then check the value of `max_retries` on the running service. | Write *“Try three retries tomorrow”* in the README while the service still uses its old retry count. Or put tomorrow’s review reminder in a source comment. |
| Stakeholder / Customer Context | *“Max asked for uploads. Peter puts the spreadsheet in Drive.”* | **Committed code:** Name the function `upload_spreadsheet`. <br>**Operator guide:** Give the Drive folder operators should use. <br>**Handover note:** Name Peter if the next operator needs to ask him for access. | Name the function `upload_for_max` because Max requested it, or copy Peter’s daily routine into the docstring for `upload_spreadsheet`. |
| Stakeholder / Customer Context | *“Customer ABC needs a separate upload folder.”* | **Tenant config:** Map ABC’s customer ID to its folder if the service selects folders by customer. <br>**Committed code:** Use `upload_spreadsheet` if all customers upload the same way. <br>**License file:** Preserve any copyright notice the license requires. | Rename the shared function `upload_for_abc` just because ABC first asked for it. |
| Reference Code Context · Migration | *“Copy `read_header` from the reports repo’s `reader.py`.”* | **Committed code:** Adapt `read_header` in the destination repo’s `reader.py`. <br>**Usage docs:** Show a call such as `read_header("name,city")` if users need an example. <br>**Jira story:** Keep the link to the reports repo’s `reader.py` in the ticket. <br>**Migration guide:** List any file moves operators must perform. | Start the docstring for `read_header` with *“Copied from the reports repo for this story.”* |
| Reference Code Context · Reference | *“Use the same validation pattern as the reports repo.”* | **Committed request-validation code:** Check for the required header (e.g., `X-Request-ID`) and reject requests that omit it. <br>**API reference:** Name the required header and show the error returned when it is missing. <br>**Jira story:** Link to the function that checks the header in the reports repo. | Write *“Validate like the reports repo”* in the API reference and leave callers guessing which headers they must send. |
| Code Behavior Change | *“The endpoint now accepts a timeout.”* | **Committed request handler and unit test:** Accept `timeout_seconds`. Send `timeout_seconds=5` in the test and assert that the operation uses five seconds. <br>**API reference:** Show how to send `timeout_seconds`. <br>**Commit message or PR:** Describe the change as *“Add an optional timeout to the endpoint.”* | Add *“We extended the model in this sprint”* to the request model’s docstring. |
| Code Behavior Preservation | *“Requests without a timeout must still work.”* | **Committed test:** Omit the timeout and assert 30 seconds. <br>**Test runner:** Run that test on both commits. <br>**PR description:** Record the command, revisions, and results as in [🧩 Example: one Jira story, two commits](#example). | Name a test `test_still_works_after_adding_timeout` and claim the old behavior was verified after running it only on the new code. |
| Remaining Tasks / TODOs | *“We still need to deploy and pin the release.”* | **Private, uncommitted TODO:** List the pending deployment and release pin. <br>**Standup or handover:** The user chooses what to share. <br>**Release runbook:** Document the pinning command if it applies to every release. | Commit this week’s unchecked deployment tasks to the README or post the private checklist to Slack unasked. |
| Cross-Repo Context | *“The config repo supplies the settings this service reads.”* | **Service repo’s configuration docs:** List the keys the service reads (e.g., `bucket`, `region`, and `timeout_seconds`) and link to the repo that supplies them. <br>**Shared architecture page:** If several teams use these settings, show which repo publishes them and which services read them. | Copy an internal path such as `deploy/write_settings.py` from the config repo into the service README. Renaming that script would make the path in the README wrong. |

<a id="example"></a>

## 🧩 Example: one Jira story, two commits

> *“For DATA-123, copy the `read_header` function from the reports repo’s `reader.py` by the end of today. Let callers supply a delimiter. Files with no delimiter setting must still read as comma-separated.”*

Read the rows in ticket order. Run the comma test in commit 1 before adding the delimiter option in commit 2.

| Words from the story | Category | Steps |
| --- | --- | --- |
| *“For DATA-123”* | Project Management Context | Open DATA-123. Link the story in the PR so the reviewer can read the request. |
| *“copy the `read_header` function from the reports repo’s `reader.py`”* | Reference Code Context · Migration | Open `read_header` in the reports repo’s `reader.py` and check how it calls `csv.reader`. Compare the function with [read_header in the destination repo before either commit](#csv-baseline). |
| *“by the end of today.”* | Project Management Context | Keep the due date in DATA-123. If tests are still failing at the deadline, tell the user privately which test failed. The user decides whether to update the team. |
| *“Let callers supply a delimiter.”* | Code Behavior Change | In [commit 2](#csv-change), add the optional parameter in `reader.py`, test a caller-supplied delimiter, and rerun the comma test. |
| *“Files with no delimiter setting must still read as comma-separated.”* | Code Behavior Preservation | In [commit 1](#csv-test), add a test in `test_reader.py` that omits the delimiter. Run it against the current `read_header` in `reader.py` and commit the passing test. |

<a id="csv-baseline"></a>

### read_header in the destination repo before either commit

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

The test expects `read_header("name,city")` to return `["name", "city"]`. Returning `["name,city"]` would fail the assertion.

<a id="csv-change"></a>

### Commit 2 · Accept the caller’s delimiter

In `reader.py`, pass the `delimiter` argument from `read_header` to `csv.reader`. Give the parameter a comma as its default:

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
