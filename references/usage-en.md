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

### Edit mode examples

- "Continue filling FAE for the remaining tickets."
- "Open each ticket, go to FAE, fill label/pattern/comment, then update."
- "If the FAE section already has content, skip rewriting and just update."

## Key operator details

### Audit mode

- Edit only to inspect, then cancel.
- Do not write or update content.
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
