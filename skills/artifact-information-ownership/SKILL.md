---
name: artifact-information-ownership
description: Use before translating a change request into code, tests, documentation, configuration, or agent instructions, and when reviewing their diffs. Decide what to implement, what to verify, and which artifact owns any text about the work.
---

# 🧭 Artifact Information Ownership

A request describes work from the requester's perspective. Before choosing files or drafting names, ask what a reader, operator, or reviewer must be able to do once the work is complete. A single sentence can imply several actions and produce no durable prose.

## 🎯 Choose the action before writing

For each part of the request, identify its purpose and the observable result that would satisfy it. Use these questions for unfamiliar requests as well.

| What does the requester need? | What fulfills that need? |
| --- | --- |
| The system must do something. | Implement the behavior in the component that controls it. A deadline or expiry requires a mechanism or an assigned operational action. A sentence describing it cannot enforce it. |
| Someone must establish that a claim holds. | Choose an observation that could disprove the claim. Run the check where its result matters and retain evidence at the necessary revisions or environments. |
| A reader must understand, use, or maintain something. | Identify the reader's decision and what they would lose by omitting the text. Describe the current contract, including any continuing constraint or rationale needed for correctness. Leave an adequate document alone. |
| Someone must coordinate, review, or reconstruct a change. | Put the delta, attribution, status, or remaining action in the authorized work record that serves that reader, such as a ticket, commit, PR, experiment plan, or ADR. |

For each proposed statement, ask who owns its accuracy and what event could retire it. Completing a release can retire a rollout checklist. It cannot retire a compatibility promise to callers. Even permanently true provenance earns space only when it serves the artifact's purpose. Link independently maintained definitions and state the dependency contract locally.

## 🧪 Establish preserved behavior

When a request combines a change with a requirement to preserve behavior, identify the specific existing inputs and observable outcomes that must remain valid. Name tests for those conditions and outcomes. A reader should understand each assertion from a single checkout.

When the reviewer needs evidence across revisions, use this sequence:

1. On the production baseline, run the relevant tests. Where coverage is missing, add characterization tests in a **test-only commit**. Verify that they pass against the original implementation. Existing adequate tests can supply this baseline without a new commit.
2. Make the production change in a **separate commit**. Add checks for newly supported behavior on the revision where that behavior is expected.
3. Run the same preservation tests with the same assertions and relevant conditions. Give the reviewer the revisions, command, and observed results in the PR or work record. Limit the claim to the cases exercised. If the baseline was unavailable or failed, report that gap.

For example, a request to extend a model while preserving omitted-field handling might warrant a test called `test_omitted_timeout_uses_default` if that is what it asserts. The assertion defines the contract. The two runs establish whether both revisions satisfy it. A failure on the baseline can be appropriate for a bug-reproduction test when the requested outcome is a fix.

## 📋 Place information by purpose

Use these examples to check the intended reader and lifetime of a statement.

| Situation | Placement decision |
| --- | --- |
| A model gains two optional fields. | If callers need the fields to use the API, document their names, meanings, and omission behavior in its authoritative reference. If existing docs already serve their readers, leave them untouched. Record the extension in the change history. |
| A script moves between repositories or adopts a shared settings class. | Describe current invocation, dependencies, environment prefix, and loading behavior where needed. Keep incidental origin or resemblance in the work record. |
| A reviewer requests validation or a customer commissions storage. | Name the validation rule or storage capability. Include identity where it determines routing, authorization, legal attribution, or another continuing obligation. |
| Generation runs locally and CI consumes the result. | Explain that division if it guides an operator. The workflow stands on its own without a comparison to another project. |
| A behavior is being tried for one day. | Put behavior on the instruction surface and duration, review, and rollback in the experiment plan. Arrange the expiry action when automatic or scheduled removal is required. |
| A release has deployments or dependent PRs outstanding. | Track remaining work in the release record. Put repeatable deployment instructions in the runbook. A dated migration guide can legitimately describe a transition. |
| An example needs a client identifier. | Use an obvious placeholder. Add clarification only if it prevents a plausible operational mistake. |

## 🔎 Review the actual diff

Read names, assertions, docstrings, comments, examples, configuration, and generated documentation as someone who has never seen the ticket. Can that reader understand what the system does and why a continuing constraint matters? For every changed document, identify the decision its reader can now make. Remove edits whose only purpose is to narrate the work.

Then inspect the evidence for each verification claim. A test title, comment, or freshness caveat cannot establish a comparison between revisions. Preserve required attribution and intentionally historical records. Once placement is settled, apply [Living Artifacts](../living-artifacts/SKILL.md) to information whose accuracy depends on time.
