---
name: review-ready-code
description: "Turn a finished, tested implementation into a human-reviewable Git commit stack without changing the final code. Use when the user says a completed change is ready for review, asks to make a branch or diff review-ready, wants a large agent-generated diff split into small conceptual changes, or needs a review sequence that follows Google's Small CL guidance."
---

# Review-Ready Code

Transform a completed implementation into a review representation that is easier for a human to understand. The implementation branch remains the source of truth; the review-ready branch is a disposable projection of the same final tree with a deliberately structured commit history.

## Source Method

At every invocation, fetch and read Google's current **Small CLs** guidance before choosing commit boundaries:

- [Google Engineering Practices — Small CLs](https://google.github.io/eng-practices/review/developer/small-cls.html)

Do not rely only on remembered summaries of the article. If the source cannot be fetched, report that limitation before reorganizing the history.

Use the article as the method for deciding what constitutes a reviewable change. In particular, prefer one self-contained conceptual change, keep related tests with the behavior they verify, separate meaningful refactors from behavior changes, and choose horizontal or vertical splits according to which makes each change independently understandable.

## Invariant

This workflow changes **history and presentation only**, never the completed implementation.

Capture the finished implementation by immutable commit ID before doing anything else:

```bash
SOURCE_HEAD="$(git rev-parse HEAD)"
SOURCE_TREE="$(git rev-parse "${SOURCE_HEAD}^{tree}")"
```

After rebuilding the review stack, both checks must succeed:

```bash
test "$(git rev-parse HEAD^{tree})" = "$SOURCE_TREE"
git diff --exit-code "$SOURCE_HEAD" HEAD --
```

Tree identity is the acceptance criterion. Passing tests is not a substitute for tree identity.

## Preconditions

Before reorganizing the change:

1. Identify the intended PR/review base branch from repository context; do not assume `main` or `master`.
2. Fetch current remote refs according to the repository's Git workflow.
3. Require the finished implementation to be committed and the source worktree to be clean. Do not hide uncommitted work inside the review transformation.
4. Run the implementation's required validation before capturing `SOURCE_HEAD`.
5. Record:
   - source branch name;
   - `SOURCE_HEAD`;
   - `SOURCE_TREE`;
   - review base commit;
   - the cumulative diff from review base to `SOURCE_HEAD`.

The source branch must remain intact while the review representation is prepared.

## Review Branch And Worktree

Use one linear review branch rather than a chain of dependent review branches.

Naming:

- review branch: `<source-branch>--review-ready`
- review worktree: a sibling directory ending in `--review-ready`

If the source branch contains `/`, preserve the source branch name in the Git branch and replace path separators with `-` only where required for the filesystem worktree path.

Create the review branch from the intended review base, not from the finished implementation HEAD. Use a separate worktree so the implementation branch remains untouched.

If the review branch already exists, do not overwrite it blindly. Capture its tip for comparison or create a fresh regeneration branch, then replace the old projection only after validation.

## Decompose The Finished Diff

Analyze the whole finished diff before making the first review commit. Build a coverage map so every changed hunk belongs to exactly one planned review unit.

A review unit should answer one coherent question such as:

- What externally visible behavior is being introduced?
- What domain rule or data-model capability is required?
- What adapter/integration change enables that behavior?
- What refactor is necessary independently of the behavior change?

Prefer conceptual boundaries over file boundaries or arbitrary line counts.

### Keep Together

Keep code together when separating it would make the reviewer hold missing context in their head. Typical examples:

- a behavior change and the focused tests that demonstrate it;
- an API and at least one usage that makes its purpose understandable;
- a BDD scenario, the relevant step definition, and the production path needed for that scenario when they form one coherent vertical behavior;
- a schema/model addition and the minimal consumer needed to make its role clear.

### Split Apart

Prefer separate review units for:

- unrelated behaviors;
- meaningful refactors versus feature/bug-fix behavior;
- independent test-framework or test-infrastructure work;
- generated/mechanical changes whose review task differs from handwritten logic;
- distinct vertical sub-features;
- architectural layers only when each horizontal slice remains independently understandable and valid.

Do not create one commit per file merely because files are easy to stage.

## Order For Human Comprehension

Order review units so each commit can be understood using only:

1. the existing codebase;
2. the commit description;
3. commits already reviewed earlier in the stack.

Minimize forward references to concepts that only appear in later commits.

When several valid orders exist, prefer the order that minimizes how much unresolved context the reviewer must retain. Tests may come before, with, or after supporting code only when that ordering improves comprehension and the commit remains self-contained; related tests normally belong in the same conceptual change.

Each commit message should state the conceptual change, not the staging mechanism.

## Construct The Stack

Rebuild the review branch from the review base using only content already present in `SOURCE_HEAD`.

For every planned review unit:

1. Materialize/stage only the hunks belonging to that unit.
2. Inspect the staged diff before committing.
3. Confirm no hunk assigned to another unit has leaked into the staged change.
4. Commit the unit with a conceptual message.
5. When practical, validate the repository at that intermediate commit. Adjust boundaries if an intermediate commit would leave the project invalid.
6. Continue until the coverage map has no unassigned hunks.

Do not opportunistically fix, clean up, rename, reformat, or refactor code while building the review stack. Any desired code change belongs on the source implementation branch first.

## Completeness And Identity Checks

Before handing the stack to the user:

1. Verify every original changed path and hunk is represented exactly once across the stack.
2. Verify the review HEAD tree equals `SOURCE_TREE`.
3. Verify `git diff --exit-code "$SOURCE_HEAD" HEAD --` succeeds.
4. Compare the cumulative review-base-to-HEAD diff with the original review-base-to-`SOURCE_HEAD` diff when useful for diagnosis.
5. Run applicable repository validation on review HEAD.
6. Stop and repair the stack if any identity check fails.

Do not claim the stack is review-ready when the final tree differs, even if tests pass.

## Human Handoff

Present the commits in review order. For each commit, give only:

- commit SHA and subject;
- the conceptual question the commit answers;
- the main entry point or changed behavior to inspect first.

The human should be able to review commit 1, mark that concept understood, then move to commit 2 without scanning the entire finished diff at once.

## Handling Review Feedback

Treat the implementation branch as canonical and the review-ready history as regenerable.

When feedback requires a code change:

1. Apply the requested change to the source implementation branch, not only to the review projection.
2. Re-run implementation validation and capture the new source HEAD/tree.
3. Regenerate the review-ready stack from the same review base.
4. Compare the previous and regenerated stacks with `git range-diff` when useful so the reviewer can see which conceptual units actually changed.
5. Re-run the final tree-identity checks.

Do not maintain correctness by manually rebasing a chain of dependent review branches after every feedback change. Regeneration from the canonical finished state is the default because it keeps the end-state invariant mechanically checkable.
