---
name: change-consistency
description: "Use when implementing or revising behavior, terminology, interfaces, or usage patterns, or reviewing a branch for changes that were not carried through consistently."
---

# 🔄 Complete the consequences of a change

Complete the consequences of the accepted change across affected code and
supporting artifacts, including places outside the initially edited files.
Preserve unrelated pre-existing issues. Apply this procedure again to follow-up
changes, regardless of where the requirement originated.

1. **Establish before and after.** Identify the branch's starting point: the most
   recent commit shared with the branch it will merge into, unless a comparison
   commit was supplied. Compare it with the current state, including uncommitted
   work. State what behavior, meaning, or contract changes and identify the
   intended-use entry points. These may be application entry points, end-to-end
   scenarios, or library consumers. Distinguish consequences of this branch from
   pre-existing issues, even within edited files.

2. **Follow relationships in both directions.** Inspect consumers needing
   adaptation and dependencies that may become unnecessary, including functions
   inside still-used modules. Follow representations of the same concept:
   names, assertions, fixtures, configuration, documentation, and examples.
   These are search directions, not an exhaustive checklist. Trace assumptions
   and equivalent meanings even when their names differ; inspect occurrences
   rather than relying on text matches or reference counts.

3. **Decide from purpose and evidence.** Ask whether each affected element still
   serves a supported purpose and accurately expresses the intended behavior.
   Update inaccurate representations, including test names after assertion
   changes. Remove newly unnecessary implementation together with obsolete
   tests and dependencies. Unit-test callers alone do not establish a continuing
   intended use. Check supported external interfaces, framework discovery, test
   infrastructure, intentional historical references, and required compatibility
   before changing or removing them.

4. **Repeat after consequential edits.** Removing one element can make its
   callers, tests, imports, fixtures, or configuration obsolete. Continue until
   another pass identifies no unresolved consequences. Report uncertainty about
   dynamic or external usage rather than inventing a reason to retain or delete
   something. Keep the review within the change's causal scope.

5. **Verify behavior and completeness separately.** Run applicable checks, then
   review the complete branch diff and affected relationships for superseded
   assumptions. Passing tests, remaining references, or a clean search alone do
   not establish completeness. Briefly report consequential updates beyond the
   initial edit and any remaining uncertainty; limit coverage claims to what was
   actually inspected.
