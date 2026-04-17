# Titan FAE Tickets Field Checking Skill

🌐 English | 🇨🇳 [中文说明](README.zh-CN.md)

An OpenClaw skill for auditing and updating Telechips TITAN Jira tickets, covering both the **FAE** tab and the **Field** tab.

## What this skill does

This skill supports two practical workflows:

1. **Check-only audit mode**
   - Audit another reporter's Jira tickets
   - Check both **FAE** tab fields and **Field** tab fields
   - Return only ticket keys and the missing fields
   - Prefer high-efficiency read-only inspection for large audits

2. **Edit/update mode**
   - Open your own Jira tickets from a filtered JQL result
   - Enter the **FAE** tab before editing FAE-related content
   - Fill or update relevant fields safely
   - Keep browser session intact after updates

## Fields covered

### FAE tab
- `FAE_Label`
- `FAE Pattern`
- `Comment`

### Field tab
- `O/S`
- `Self Resolution`
- `Cause(Customer)`
- `Hardware Issue Pattern`
- `Software Issue Pattern` (first one only)
- `Labels`
- `FAE Person`
- `git/repo command`

## Preconditions

Before using this skill:

- You must already be logged in to Telechips TITAN Jira
- The login session must be active in the same Chrome browser that OpenClaw can control
- The Jira list/filter page must be open or reachable in that browser
- For bulk work, keep the browser window stable

## Fixed startup URL

Always start from:

`https://tcs.telechips.com/secure/Dashboard.jspa`

## Key operating rules

- Always click **FAE** before checking or editing FAE-related fields
- In audit mode, inspect **both** the Field tab and the FAE tab
- `FAE_Label` is a label picker, not plain text
- When creating/selecting labels, wait for suggestions and select the intended item before updating
- For large check-only audits, prefer a faster authenticated inspection method when available

## Included files

- `SKILL.md` — main skill instructions
- `references/usage-en.md` — English usage notes
- `references/usage-zh.md` — 中文使用说明
- `references/field-tab-fields.md` — Field tab guidance
- `jira-fae-tickets.skill` — packaged skill archive

## Chinese version

See: [README.zh-CN.md](README.zh-CN.md)
