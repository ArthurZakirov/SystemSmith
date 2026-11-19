---
name: artifact-information-ownership
description: Use before translating a change request into code, tests, documentation, configuration, or agent instructions, and when reviewing their diffs. Decide what to implement, what to verify, and which artifact owns any text about the work.
---

# 🧭 Artifact Information Ownership

Use this skill when turning a request into Git changes, verification work, or communication. Account for each requirement, including context that guides a decision without needing another written record.

## 🧭 Workflow for handling a request

Follow these steps from intake through review.

1. **Identify the obligations.** For each part of the request, determine who needs to do, know, or verify what. One sentence can describe a new capability, impose a deadline, and require evidence about existing behavior.
2. **Choose an action and an owner.** Use the routing table below. For unfamiliar cases, ask what decision or action would be lost if the information were omitted, who can maintain its accuracy, and when it stops being relevant. An existing authoritative record can already meet the need.
3. **Implement and collect evidence.** Group commits so reviewers can inspect each meaningful change and reproduce any comparison the request requires. Use the authorized destination and audience for coordination. Choosing a communication channel does not authorize sending a message.
4. **Review the actual diff and results.** Read names, tests, comments, examples, configuration, and generated docs from a single checkout without the ticket. Verify that each changed document helps its reader act and that each claim is supported by the observations recorded. Remove incidental narration before committing. For time-sensitive information, check placement before applying [Living Artifacts](../living-artifacts/SKILL.md).

## 🗂️ Routing table by information type

Use each applicable row independently. The same request can require several destinations.

| Information type | Owning surface | What belongs there |
| --- | --- | --- |
| Project coordination | Existing Jira story, Slack discussion, or experiment plan | Deadline, decision owner, trial duration, success criteria, and rollback arrangements. If expiry must occur, schedule it or assign the action. |
| Stakeholder or customer context | Decision record for attribution or handoff. Domain model, configuration, or legal notice for an ongoing obligation. | Capture who requested a decision only where that matters. Describe a recurring upload by role and destination. Use a customer identifier where it selects a tenant, permission, or route. Preserve required attribution. |
| Migration or reference implementation | Target repository for its operating contract. Existing story for the agreed reference. | Describe actual invocation, settings, and dependencies. Reuse the agreed pattern without restating its provenance. Record a transition in a migration guide or ADR when readers need its history or steps. |
| Changed behavior | Production code, behavior tests, and any affected public contract. Commit message and PR description for the delta. | Express the current inputs, outputs, and failure conditions in the repository. Update operating docs only where a reader needs different instructions. Explain what changed to reviewers in the change record. |
| Preserved behavior | Characterization tests for the contract. Test runs on both revisions for comparative evidence. | Name tests for the condition and observable result. Establish the baseline before changing production code, then repeat the same checks. Scope the claim to the cases exercised. |
| Remaining implementation work | Private, uncommitted work record available to the user and agent | Track outstanding components, tests, deployments, and release pins. The user chooses what to share at standup, planning, or handover. Put a reusable deployment procedure in the runbook. |
| Cross-repository dependency | Consumer's contract and a link to the authoritative provider | State required arguments, formats, guarantees, and error behavior. Keep independently maintained file paths and implementation details with their owner. A shared company architecture page can describe the system-wide flow. |

## 🧩 Examples of decomposing a request

Each example follows one request across the surfaces needed to fulfill it.

### Extend a model and preserve a default

“Add two optional fields by Friday. Omitting the timeout must still use the default.” The deadline belongs in the existing story. The compatibility claim determines the order of work:

1. On the original implementation, run the omitted-timeout test. If coverage is missing, add a test such as `test_omitted_timeout_uses_default` in a test-only commit and verify that it passes.
2. In a separate commit, implement the fields and test their supported inputs. Update the API reference if callers need the field names, meanings, or omission rules.
3. Rerun the same baseline assertions under equivalent conditions. Record revisions, command, and observed outcomes in the PR. Report a failed or unavailable baseline as a gap in the comparison. A test intended to reproduce a bug instead has a different expected baseline result: it should expose the defect.

### Trial a configuration supplied by another repository

“Use the provider's retry settings for one day, then decide whether to keep them.” The consumer validates the agreed settings and links to their authoritative definition. The experiment plan owns the cutoff, decision owner, and rollback action. If the policy must expire automatically, configure that expiry and verify behavior at the boundary. Track unfinished deployment steps in the private work record. Each destination gives its reader enough information to act independently.
