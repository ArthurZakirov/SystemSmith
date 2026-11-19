---
name: artifact-information-ownership
description: Use when turning a ticket or conversation into code, tests, docs, commits, or a PR. Decide what to change, what to test, and which details can stay in the ticket.
---

# 🧭 Artifact Information Ownership

Imagine opening a repository six months after a ticket closed. A test named `test_missing_timeout_uses_30_seconds` tells you what to expect from the code. The ticket and the PR can tell you who asked for the change and when it shipped.

**Contents**

- [🛠️ How to work from a ticket](#workflow)
- [🗂️ Where each part goes](#routing)
- [🧩 Example: one Jira story, two commits](#example)

The workflow shows how to split a ticket into instructions, the table shows where each piece goes, and the CSV example follows one ticket through two commits and a PR.

<a id="workflow"></a>

## 🛠️ How to work from a ticket

1. **Read the request one piece at a time.** In “Copy the CSV reader from the reports repo by Friday,” the first part tells you where to find the code. The last two words tell you when the work is due.
2. **Decide what to do with each piece.** Use the table in [🗂️ Where each part goes](#routing). Copy the reader, run it on a sample CSV, and use Friday to plan the work. If Friday is already in Jira, that part needs no new text.
3. **Read the diff before committing.** Look at the actual test names, comments, examples, and generated docs. Ask what someone opening each file needs to know. “A missing header raises an error” helps someone prepare a CSV. “Copied for Friday’s release” can come out. Check any claim in the PR against the test output before posting it.

<a id="routing"></a>

## 🗂️ Where each part goes

Use the same question for requests you haven't seen before: who will use this detail, and what will they do with it (e.g., a caller needs the timeout default to decide whether to send a value)?

| Words you might hear | What to do with them |
| --- | --- |
| “Finish by Friday.” | Keep Friday in the Jira story. Use the team's Slack thread if someone needs a scheduling update and the user asks you to send it. |
| “Try three retries for one day. Tomorrow we'll decide whether to keep them.” | Set the retry count to three. Put tomorrow's review in the experiment plan. If the user asks for automatic expiry, schedule the switch back and check the retry count after it runs. |
| “Max asked for uploads. Peter puts the spreadsheet in Drive.” | Name the feature for its job (e.g., spreadsheet upload). If operators need upload instructions, tell them which folder to use. Include Peter's name in a handover if he is the person to contact for access. |
| “Customer ABC needs a separate upload folder.” | Use ABC in configuration if the application looks up ABC's folder by customer ID. If ABC simply suggested a feature for everyone, use the feature's name in code. Keep names where readers are required to see them (e.g., an author's copyright notice in a license). |
| “Copy the reader from the reports repo.” | Use that reader as the starting point. Describe how to call the reader in its new home. If the source repo is already linked in the story, that link is enough. If operators must move files during a migration, give them the steps in the migration guide. |
| “The endpoint now accepts a timeout.” | Implement and test the field. If callers need instructions, write “Send `timeout_seconds` to set the timeout.” A commit message or PR can say “Add an optional timeout to the endpoint.” |
| “Requests without a timeout must still work.” | Pick an input and expected result (e.g., omit the timeout and expect 30 seconds). Run that test before and after editing production code. See [🧩 Example: one Jira story, two commits](#example) for the two runs. |
| “We still need to deploy and pin the release.” | Keep those unchecked tasks in the private list shared by the user and agent, outside Git. The user chooses what to share at standup or handover. If every release uses the same pinning command, the release runbook can document that command. |
| “The config repo supplies the settings this service reads.” | Document the settings this service expects (e.g., `bucket`, `region`, and `timeout_seconds`) and link to the config repo. The provider can rename its Python files without changing those settings. If several teams need the whole flow, draw it once on their shared architecture page. |

<a id="example"></a>

## 🧩 Example: one Jira story, two commits

> For DATA-123, copy the CSV reader from the reports repo by the end of today. Let callers supply a delimiter. Files with no delimiter setting must still read as comma-separated.

Here is how to carry out that example:

1. **Before changing the reader**, run a test that reads `name,city` with no delimiter setting and expects two columns, `name` and `city`. If that test is missing, add it in a test-only commit named “Test comma-separated headers with no delimiter setting.” Verify that it passes. If a suitable test already exists, run it and record the commit you ran it on.
2. **In the implementation commit**, accept the optional delimiter. Add a test that reads `name|city` with `|` as the delimiter and expects the same two columns. Rerun the original comma test with the same input and assertions.
3. **In the PR**, give the two commit IDs, the test command, and the actual results. Two passing runs show that the comma case worked on both commits. If you could only run it after the edit, say so.
4. **For callers**, document how to pass a delimiter if the existing instructions leave that question open. The Jira story already records today's deadline and where to copy the reader from.

For documentation that can go stale (e.g., a copied list of supported settings), see [Living Artifacts](../living-artifacts/SKILL.md).
