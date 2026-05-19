# Titan FAE Tickets Field Checking Skill

🌐 English | 🇨🇳 [中文说明](README.zh-CN.md)

An OpenClaw skill for auditing and updating Telechips TITAN Jira tickets on `tcs.telechips.com`, covering both the **FAE** tab and the **Field** tab.

## System & scope

- TCS (`tcs.telechips.com`) is the Jira deployment domain; TITAN is the Jira instance/context. Treat them as the same system for this skill.
- Audit only TITAN_Customer tickets: `TANCS-*`, `TANCS4-*`, `TANCS5-*`, `TANCS6-*`, and `TANCS7-*`.
- Skip other projects such as `TMRCR-*`, `TMCF-*`, `TPCP-*`, `IM*`, `IS*`, and `IG*`; they are outside this skill's FAE/Field audit scope.
- For large check-only audits, use Jira REST API paging instead of opening tickets one by one.

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
- `Cause (Customer)`
- `Hardware Issue Pattern`
- `Software Issue Pattern` (first one only)
- `Labels`
- `FAE Person`
- `git/repo command`

Do not audit `SDK Version (TITAN)`, `Ref. H/W version`, or other fields outside the lists above.

## Fixed field ID mapping

### FAE tab

| Field | customfield_id | Type | Empty condition |
|---|---|---|---|
| `FAE_Label` | `customfield_15300` | array | null or `[]` |
| `FAE Pattern` | `customfield_15200` | option | null |
| `Comment` | `comment` | array | `comment.comments.length === 0` |

### Field tab

| Field | customfield_id | Type | Empty condition |
|---|---|---|---|
| `O/S` | `customfield_10684` | option | null or `value === 'None'` |
| `Self Resolution` | `customfield_15009` | option | null or `value === 'None'` |
| `Cause (Customer)` | `customfield_15044` | option | null or `value === 'None'` |
| `Hardware Issue Pattern` | `customfield_15045` | option | null or `value === 'None'` |
| `Software Issue Pattern` | `customfield_15046` | option, first dropdown only | null or `value === 'None'` |
| `Labels` | `labels` | array | `[]` |
| `FAE Person` | `customfield_15100` | user/string | null or `''` |
| `git/repo command` | `customfield_15101` | string | null or `''` |

Note: Jira metadata contains two fields named `git/repo command` (`customfield_15008` and `customfield_15101`). The Field tab uses `customfield_15101`.

## REST API audit

Use the Jira search API for bulk read-only audits:

```text
GET https://tcs.telechips.com/rest/api/2/search?jql=<JQL>&fields=<field_ids>&maxResults=50&startAt=<offset>
```

Authentication options:

1. Browser Console: open DevTools on `https://tcs.telechips.com` and paste the audit script from `SKILL.md`.
2. Personal Access Token (PAT), if TITAN supports it.
3. `JSESSIONID` cookie copied from DevTools and sent as a Cookie header, for example `export JIRA_COOKIE='JSESSIONID=...'`.

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
- Restrict audit results to the TITAN_Customer ticket prefixes listed above
- `FAE_Label` is a label picker, not plain text
- When creating/selecting labels, wait for suggestions and select the intended item before updating
- For large check-only audits, prefer Jira REST API paging over browser clicking

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
- Skipped tickets outside TITAN_Customer scope or with no FAE tab
- Tickets with missing required fields in either tab

## Recent updates

- Kept FAE tab scope at 3 fields: `FAE_Label`, `FAE Pattern`, `Comment`
- Restricted Field tab scope to 8 fields and explicitly excluded `SDK Version (TITAN)` and `Ref. H/W version`
- Clarified that TCS and TITAN refer to the same Jira system for this skill
- Added fixed customfield ID mapping and REST API guidance for fast audits
- Kept the newer dual-tab audit model, which checks both **Field** and **FAE** tabs
- Clarified that reports should explicitly show whether missing items come from the Field tab or the FAE tab

## Included files

- `SKILL.md` — main skill instructions
- `references/usage-en.md` — English usage notes
- `references/usage-zh.md` — 中文使用说明
- `references/field-tab-fields.md` — Field tab guidance
- `jira-fae-tickets.skill` — packaged skill archive

## Chinese version

See: [README.zh-CN.md](README.zh-CN.md)
