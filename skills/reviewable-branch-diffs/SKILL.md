---
name: reviewable-branch-diffs
description: Apply coding standards, style guidance, linting, formatting, or refactoring instructions to changes on a Git feature branch without adding unrelated cleanup. Use when a user, another skill, AGENTS.md, project policy, or a tool asks an agent to improve code quality while implementing or reviewing branch changes. Determine eligibility from the Git diff against the intended integration branch, not from all violations found in touched files; preserve a focused, easy-to-review branch diff.
---

# Reviewable Branch Diffs

## Objective

Make the **aggregate Git branch diff** easy for a human to review. Coding standards describe good code; this skill decides **which changes on the current branch** should be made to satisfy those standards. Leave unrelated pre-existing violations in place.

The standards themselves come from user instructions, other skills, repository instructions, or tools—not from this skill.

## 1. Establish the Git baseline

Identify the intended integration branch using **Git only**:

1. If the user or repository workflow explicitly identifies the integration branch, use it.
2. Otherwise, determine the relevant remote with `git remote`. If more than one exists, use available Git tracking/configuration information to select the right one; do not silently pick an arbitrary remote.
3. Resolve that remote's default branch via `git symbolic-ref --quiet --short refs/remotes/<remote>/HEAD`. If the symbolic ref is missing, consult `git ls-remote --symref <remote> HEAD` when network access is appropriate. This queries the remote's `HEAD`, **not** the current checkout's `HEAD`.
4. Use the resulting remote-tracking branch as the *candidate* integration branch. A remote default is not proof of the branch's actual merge destination. If evidence conflicts or is ambiguous, establish the target before applying scope-sensitive rules.
5. Ensure the candidate ref is present and sufficiently current (fetch when appropriate).

Inspect the committed changes and the prospective complete working-tree diff:

```bash
git diff --find-renames <base-ref>...HEAD
git diff --find-renames "$(git merge-base <base-ref> HEAD)"
git ls-files --others --exclude-standard
```

The three-dot form compares committed branch changes with their merge base. The second command compares that same baseline with the current working tree, including **staged and unstaged tracked changes**. The last command discovers untracked files that neither diff includes.

## 2. Apply rules according to what the branch introduced

| Code relative to the baseline | Action |
| --- | --- |
| Newly introduced function, class, argument, or other code | Apply relevant coding rules, including placement, typing, and formatting. |
| Existing code changed for the feature | Apply rules only to the genuinely new or necessary changes; preserve the surrounding structure and minimize adjacent churn. |
| Existing code unchanged by the feature—even in a touched file | Do not clean it up solely because a rule identifies a violation. |

A change to one line does not make an entire function or file eligible for reorganization.

## 3. Preserve reviewable changes

- **Existing function edited in place:** If a feature modifies line five of an existing function, leave the function at its original location even if a separate style rule would place it elsewhere. Moving it hides the localized logic change inside distant deletion/addition blocks.
- **New function:** Insert it in the position required by the applicable external coding rules; there is no previous position to preserve.
- **Argument typing:** Type new functions and newly introduced parameters according to the relevant rule. Do not add annotations across unrelated existing functions; avoid reflowing an existing signature just for incidental cleanup.
- **Formatting:** Format new or necessary changed code without reformatting unrelated lines. If a formatter rewrites large existing regions, isolate the required changes and revert unrelated formatting churn where feasible.
- **Commits:** A separate cleanup commit **on the same feature branch** still adds noise to its aggregate diff. Unrelated changes require separate scope, not just separate commits.

If a mandatory code formatter or check requires wider changes, identify the constraint and make the smallest technically valid change; do not silently create a large formatting-only diff.

## 4. Verify the result

1. Reinspect the full diff against the resolved baseline, including outstanding tracked edits and untracked files.
2. Check each moved block, reflowed signature, and formatting-only hunk for a **necessary feature-related reason**. Remove incidental cleanup.
3. Ensure the actual behavior change remains visible as a localized diff where feasible, and run relevant tests.
4. Report any unavoidable scope expansion or unresolved baseline ambiguity.

This skill is **agent guidance**, not a guaranteed Git policy gate. Where enforcement must be deterministic, use an independently validated checker or hook appropriate to the agent runtime.
