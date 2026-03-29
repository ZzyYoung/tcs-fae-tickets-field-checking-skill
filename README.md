# Titan FAE Tickets Field Checking Skill

🌐 English | 🇨🇳 [中文说明](README.zh-CN.md)

A skill for handling Telechips TITAN Jira FAE workflows.

## What this skill does

This skill supports two practical workflows:

1. **FAE field update mode**
   - Open tickets from a Jira filter/JQL result
   - Enter the **FAE** tab
   - Fill or update:
     - `FAE_Label`
     - `FAE Pattern`
     - `Comment`
   - Update the ticket safely

2. **FAE audit mode**
   - Check another reporter's tickets
   - Verify whether these fields are empty:
     - `FAE_Label`
     - `FAE Pattern`
     - `Comment`
   - Skip tickets with no FAE tab
   - Return only the missing ticket keys and missing fields
   - For large multi-page audits, prefer running the work in a separate sub-task/agent while the main session only reports progress and final results
   - For read-only audits, prefer a high-efficiency inspection method instead of slow visible ticket-by-ticket clicking when possible
   - After the sub-task starts, the main session should tell the user that the query is in progress and that the result will be returned here when finished

## Preconditions

Before using this skill:

- You must already be logged in to Telechips TITAN Jira
- The login session must be active in the **same Chrome browser that OpenClaw can control**
- The Jira list/filter page must be open or reachable in that browser
- For bulk actions, keep the browser window open and stable

## Important startup rule for editing your own tickets

Before bulk editing tickets reported by the current user, first apply this JQL in Jira:

`created >= 2025-01-01 AND created <= 2026-03-27 AND reporter in (currentUser()) order by created DESC`

If the filter field is not editable in the current mode, click **Advanced** first and then enter the JQL directly.

## Important FAE rules

- Always click **FAE** before checking or editing FAE-related fields
- `FAE_Label` is a label picker, not plain text
- After typing a label, you must:
  1. wait for the suggestion list
  2. click the focused suggestion or create-item
  3. wait briefly
  4. then click **Update**
- Do not use labels with spaces when creating a new label
  - Good: `notification`
  - Bad: `Technical Document`

## Label examples

- `safellink` → only for **TCC5110** and **SDM** related tickets
- `CarPlay` → for CarPlay related tickets
- `notification` → for notice/announcement type tickets when a new label is needed
- Other labels should follow the actual ticket context

## Reporting style

Recommended audit report includes:

- Check time
- Filter time range
- JQL/filter used
- Total pages / total tickets
- Skipped tickets with no FAE tab
- Tickets with missing FAE fields

## Included files

- `SKILL.md` — main skill instructions
- `references/usage-en.md` — English usage notes
- `references/usage-zh.md` — 中文使用说明
- `jira-fae-tickets.skill` — packaged skill file

## Chinese version

See: [README.zh-CN.md](README.zh-CN.md)
