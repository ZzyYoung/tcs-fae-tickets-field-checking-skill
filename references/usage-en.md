# Jira FAE Tickets Skill - English Usage

## Purpose

This skill is for two related Jira workflows in Telechips TITAN:

1. **Audit mode**: inspect someone else's filtered tickets and find which tickets have missing FAE data.
2. **Edit mode**: update your own filtered tickets by filling FAE_Label, FAE Pattern, and Comment.

## Requirements

- The user must already be logged in to TITAN Jira.
- The login must exist in the same Chrome browser that OpenClaw can control.
- The ticket list must be reachable through a browser-controlled tab.
- For large audits, allow enough time for multi-page checking.

## Example user requests

### Audit mode examples

- "Check this reporter's tickets and tell me which ones are missing FAE fields."
- "Use this JQL and inspect all pages. Only report tickets with empty FAE_Label, FAE Pattern, or Comment."
- "Do not modify the tickets, just audit them."
- "Create a separate task/agent to run the full audit and just report progress back to me."

### Edit mode examples

- "Continue filling FAE for the remaining tickets."
- "Open each ticket, go to FAE, fill label/pattern/comment, then update."
- "If the FAE section already has content, skip rewriting and just update."

Before bulk editing the current user's own tickets, first apply this JQL:

`created >= 2025-01-01 AND created <= 2026-03-27 AND reporter in (currentUser()) order by created DESC`

If the filter box is not editable in basic mode, click **Advanced** and enter the JQL directly.

## Key operator details

### Audit mode

- For large multi-page audits, prefer running the inspection in a dedicated sub-task/sub-agent.
- For read-only audits, prefer a high-efficiency authenticated inspection method when available instead of visible human-like clicking through every ticket.
- The main session should supervise, share progress, and present final results.
- Immediately after launching the worker, the main session should tell the user that the query is in progress and that the result will be sent back here when finished.
- The worker task should inspect tickets without modifying data.
- Skip tickets with no FAE tab.
- Record ticket key + which fields are missing.

### Edit mode

- Always click the FAE tab first.
- For FAE_Label, select a real label suggestion after typing.
- Wait briefly before updating.
- Use ticket-specific English comments, not one generic comment for all tickets.

## Label guidance

- `safellink`: only for TCC5110 and SDM related tickets.
- `CarPlay`: for CarPlay related issues.
- `notification`: for notice/announcement tickets when creating a new label.
- Other labels must follow actual ticket context.

## Reporting template

Recommended result structure:

- Check time
- Filter range: 2025-01-01 to 2026-03-27
- JQL used
- Pages checked / total tickets
- Skipped tickets (no FAE tab)
- Tickets with missing FAE fields
- For each missing ticket: specify missing `FAE_Label`, `FAE Pattern`, `Comment`
