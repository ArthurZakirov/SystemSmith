---
name: artifact-information-ownership
description: Use when turning a ticket or conversation into code, tests, docs, commits, or a PR. Decide what to change, what to test, and which details can stay in the ticket.
---

# 🧭 Artifact Information Ownership

**Contents**

- [🛠️ How to work from a ticket](#workflow)
- [🗂️ Where each part goes](#routing)
- [🧩 Example: one Jira story, two commits](#example)

The workflow shows how to split a ticket into instructions, the table shows where each piece goes, and the CSV example follows one ticket through two commits and a PR.

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

4. **Read the diff before committing.** Check the test names, comments, examples, and generated docs:

   - **Keep useful instructions.** *“A missing header raises an error”* tells someone how to prepare a CSV.
   - **Remove project narration.** *“Copied for Friday’s release”* only repeats the release plan.
   - **Verify PR claims.** Compare each reported result with the actual test output before posting.

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
| Code Behavior Change | *“The endpoint now accepts a timeout.”* | • Implement and test the field.<br>• If callers need instructions, write *“Send `timeout_seconds` to set the timeout.”* A commit message or PR can say *“Add an optional timeout to the endpoint.”* |
| Code Behavior Preservation | *“Requests without a timeout must still work.”* | • Pick an input and expected result (e.g., omit the timeout and expect 30 seconds).<br>• Run that test before and after editing production code.<br>• See [🧩 Example: one Jira story, two commits](#example) for the two runs. |
| Remaining Tasks / TODOs | *“We still need to deploy and pin the release.”* | • Keep those unchecked tasks in the private list shared by the user and agent, outside Git.<br>• The user chooses what to share at standup or handover.<br>• If every release uses the same pinning command, the release runbook can document that command. |
| Cross-Repo Context | *“The config repo supplies the settings this service reads.”* | • Document the settings this service expects (e.g., `bucket`, `region`, and `timeout_seconds`) and link to the config repo.<br>• The provider can rename its Python files without changing those settings.<br>• If several teams need the whole flow, draw it once on their shared architecture page. |

<a id="example"></a>

## 🧩 Example: one Jira story, two commits

> *“For DATA-123, copy the CSV reader from the reports repo by the end of today. Let callers supply a delimiter. Files with no delimiter setting must still read as comma-separated.”*

Here is how to carry out that example:

1. **Before changing the reader**, establish how it handles a missing delimiter setting.

   - **Input and result.** Read `name,city` with no delimiter setting. Expect two columns, `name` and `city`.
   - **If the test exists**, run it and record the commit ID and result.
   - **If coverage is missing**, add the test in a test-only commit named *“Test comma-separated headers with no delimiter setting.”* Verify that it passes.

2. **In the implementation commit**, add the optional delimiter and check both cases.

   - **Explicit delimiter.** Read `name|city` with `|` as the delimiter. Expect the same two columns.
   - **Omitted delimiter.** Rerun the original comma test with the same input and assertions.

3. **In the PR**, show what you ran.

   - **Both runs available.** Give the two commit IDs, command, and results. If both passed, say that the comma case passed on both commits.
   - **Only the later run available.** State which commit you tested and that the earlier result is unknown.

4. **Place the remaining details.**

   - **Caller instructions.** Document how to pass a delimiter if the existing instructions leave that question open.
   - **Jira story.** The deadline and source repo are already recorded there.

For documentation that can go stale (e.g., a copied list of supported settings), see [Living Artifacts](../living-artifacts/SKILL.md).
