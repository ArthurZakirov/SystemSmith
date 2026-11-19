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
3. **Report each mismatch.** Name the file and line, quote the wording, and say what to remove, move, or rewrite.

4. **Fix the affected commit.** For a review-only request, stop after reporting the findings in step 3. When fixing the branch:

   - **Edit the file.** For example, delete `# Must ship by Friday` from `reader.py`. If Friday is already recorded in Jira, the deletion completes the fix.
   - **Fold the edit into the commit that added the line.** On a feature branch you can rewrite, amend that commit if it is the latest one. For an earlier commit, create a fixup commit and autosquash it into that specific commit. Keep unrelated commits separate.
   - **Verify the history.** Inspect the rewritten commit’s patch and the branch diff. The deadline comment should appear in neither. If the commit is already merged or the branch is shared and rewriting it has not been agreed, make a separate correction commit.

<a id="routing"></a>

## 🗂️ Where each part goes

Use the same question for requests you haven't seen before: who will use this detail, and what will they do with it (e.g., a caller needs the timeout default to decide whether to send a value)?

Name the actual file, page, message, or tool when choosing where something belongs. The paths and page titles below are examples. Use the file or page the team already maintains.

| Category | Words you might hear | What to do with them | What not to do with them |
| --- | --- | --- | --- |
| Project Management Context | *“Finish by Friday.”* | <ul><li>**Jira story:** Keep the due date there.</li><li>**Private chat with the user:** Draft a scheduling update if needed. Send it to the team’s Slack thread only on explicit request.</li></ul> | Write `# Must ship by Friday` above `def read_header` in `reader.py`. |
| Project Management Context | *“Try three retries for one day. Tomorrow we'll decide whether to keep them.”* | <ul><li>**Committed Python settings file (e.g., `settings.py`):** Set the value the service reads (e.g., `max_retries = 3`).</li><li>**Private `todo.md` outside Git:** Add *“Tomorrow: review retry failures and decide whether to keep three retries.”*</li><li>**One-time job in the deployment tool (e.g., an AWS EventBridge Scheduler schedule):** If automatic expiry is requested, schedule a job to restore the previous retry count, then check the value of `max_retries` on the running service.</li></ul> | Write *“Try three retries tomorrow”* in the README while the service still uses its old retry count. Or put tomorrow’s review reminder in a source comment. |
| Stakeholder / Customer Context | *“Max asked for uploads. Peter puts the spreadsheet in Drive.”* | <ul><li>**Committed code:** Name the function `upload_spreadsheet`.</li><li>**Confluence page (e.g., *Spreadsheet uploads*):** Give the Drive folder operators should use.</li><li>**Slack handover message drafted in private chat for the user:** Name Peter if the next operator needs to ask him for access.</li></ul> | Name the function `upload_for_max` because Max requested it, or copy Peter’s daily routine into the docstring for `upload_spreadsheet`. |
| Stakeholder / Customer Context | *“Customer ABC needs a separate upload folder.”* | <ul><li>**Committed YAML file the service reads (e.g., `config/customers.yaml`):** Map ABC’s customer ID to its folder if the service selects folders by customer.</li><li>**Committed code:** Use `upload_spreadsheet` if all customers upload the same way.</li><li>**Committed `LICENSE` file:** Preserve any copyright notice the license requires.</li></ul> | Rename the shared function `upload_for_abc` just because ABC first asked for it. |
| Reference Code Context · Migration | *“Copy `read_header` from the reports repo’s `reader.py`.”* | <ul><li>**Committed code:** Adapt `read_header` in the destination repo’s `reader.py`.</li><li>**Committed `README.md`:** Show a call such as `read_header("name,city")` if users need an example.</li><li>**Jira story:** Keep the link to the reports repo’s `reader.py` in the ticket.</li><li>**Confluence page for the move (e.g., *Move report files*):** List any file moves operators must perform.</li></ul> | Start the docstring for `read_header` with *“Copied from the reports repo for this story.”* |
| Reference Code Context · Reference | *“Use the same validation pattern as the reports repo.”* | <ul><li>**Committed request-validation code:** Check for the required header (e.g., `X-Request-ID`) and reject requests that omit it.</li><li>**Committed API documentation (e.g., `docs/api.md`):** Name the required header and show the error returned when it is missing.</li><li>**Jira story:** Link to the function that checks the header in the reports repo.</li></ul> | Write *“Validate like the reports repo”* in `docs/api.md` and leave callers guessing which headers they must send. |
| Code Behavior Change | *“The endpoint now accepts a timeout.”* | <ul><li>**Committed request handler and unit test:** Accept `timeout_seconds`. Send `timeout_seconds=5` in the test and assert that the operation uses five seconds.</li><li>**Committed API documentation (e.g., `docs/api.md`):** Show how to send `timeout_seconds`.</li><li>**Git commit message or PR description:** Describe the change as *“Add an optional timeout to the endpoint.”*</li></ul> | Add *“We extended the model in this sprint”* to the request model’s docstring. |
| Code Behavior Preservation | *“Requests without a timeout must still work.”* | <ul><li>**Committed test:** Omit the timeout and assert 30 seconds.</li><li>**Terminal in the repo:** Run that test on both commits.</li><li>**PR description:** Record the command, revisions, and results as in [🧩 Example: one Jira story, two commits](#example).</li></ul> | Name a test `test_still_works_after_adding_timeout` and claim the old behavior was verified after running it only on the new code. |
| Remaining Tasks / TODOs | *“We still need to deploy and pin the release.”* | <ul><li>**Private `todo.md` outside Git:** List the pending deployment and release pin.</li><li>**Spoken standup update or Slack handover message:** The user chooses which tasks to share.</li><li>**Confluence page (e.g., *Deploy a release*):** Document the pinning command if it applies to every release.</li></ul> | Commit this week’s unchecked deployment tasks to the README or post the private checklist to Slack unasked. |
| Cross-Repo Context | *“The config repo supplies the settings this service reads.”* | <ul><li>**Committed Markdown file in the service repo (e.g., `docs/configuration.md`):** List the keys the service reads (e.g., `bucket`, `region`, and `timeout_seconds`) and link to the repo that supplies them.</li><li>**Shared Confluence page (e.g., *Which services use these settings?*):** If several teams use these settings, show which repo publishes them and which services read them.</li></ul> | Copy an internal path such as `deploy/write_settings.py` from the config repo into the service README. Renaming that script would make the path in the README wrong. |

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

**PR description.** Record the command and the results from the two commits. In this example, the first run passed the comma test. The second run passed that same test and the pipe test. If callers need a usage example, put this call in the repo’s `README.md`:

```python
read_header("name|city", delimiter="|")
```

For documentation that can go stale (e.g., a copied list of supported settings), see [Living Artifacts](../living-artifacts/SKILL.md).
