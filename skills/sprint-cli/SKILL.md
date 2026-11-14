---
name: sprint-cli
description: "Manage local stories and work in Sprint using the sprint CLI. Use when beginning work on a Jira ticket, fetching a Jira ticket to start working on it, or when viewing or updating Sprint stories, tasks, tracked effort, speaking notes, references, or unplanned work."
---

# Sprint CLI

Use Sprint as the local work record. Use the available Jira integration to fetch the source issue.

## Start a Jira story

When the user begins work on a Jira ticket, fetch its current details, then look for its exact key in `sprint ticket list --json`. Import verified title, Why, Remarks, acceptance criteria, and story points when present. Preserve existing local tasks, logged time, and any fields the source does not provide. Each `--criterion` argument contributes to a full replacement list, so supply that flag only when the complete current list is known.

- New story: `sprint ticket create KEY --title TITLE --story-status in-progress`.
- Existing story: `sprint --ticket KEY ticket edit --story-status in-progress`. Reuse the existing record and update only intended fields. If it is already done, reopen it only when resuming that completed work is explicit.
- Verify with `sprint --ticket KEY ticket view --json` and report the key and local status.

For read-only Jira lookups, leave Sprint unchanged. If the issue cannot be fetched or Sprint is unavailable, report the blocker and preserve the existing record.

## Continue work in Sprint

Use the explicit story key for ticket-scoped commands. Consult `sprint --help` and the relevant subcommand help before modifying tasks, recording actual work, adding references, or saving meeting notes. Record work or change status when supported by the user's request and observed progress. A local Sprint edit does not change Jira.
