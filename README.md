# Titan FAE Tickets Field Checking Skill

🌐 English | 🇨🇳 [中文说明](README.zh-CN.md)

An OpenClaw skill for auditing and updating Telechips TITAN Jira tickets on `tcs.telechips.com`, covering **FAE** tab fields, **Field** tab fields, and Reporter/Assignee correction status.

## System & scope

- TCS (`tcs.telechips.com`) is the Jira deployment domain; TITAN is the Jira instance/context. Treat them as the same system for this skill.
- Audit customer-scope tickets: all `TANCS*` prefixes plus `TMRCR-*`.
- Skip non-scope projects such as `TMCF-*`, `TPCP-*`, `IM*`, `IS*`, and `IG*`; they are outside this skill's FAE/Field audit scope.
- For large check-only audits, use Jira REST API paging instead of opening tickets one by one.

## What this skill does

This skill always runs two required audit parts for check-only reports, plus one edit/update scenario:

1. **Field completeness audit**
   - Check required fields in the FAE Tab and Field Tab
   - Group by reporter / FAE person
2. **Reporter/Assignee TITAN linked-assignee audit**
   - After the field audit, scan TCS tickets whose Reporter and Assignee are both TITAN
   - Keep only tickets with `Issue Links -> links to -> TITAN Issue`
   - Open the linked TITAN Issue and report only when its Assignee is `Unassigned` or one of the 9 China FAE members
3. **Edit/update scenario**
   - Only update FAE tab fields when the user explicitly asks and permission is clear

## Telechips Jira workflow context

Customers create tickets in TITAN Jira at `https://tcs.telechips.com`. The system automatically creates matching tickets in the `TITAN_Customer` project, usually as `TANCS-xxxx`.

Auto-created tickets default both Reporter and Assignee to the TITAN system account:

- Username: `titan`
- Email: `titan@telechips.com`

This is only a placeholder account. China FAE team members must manually correct Reporter and Assignee:

- Reporter should be one of the 9 China FAE members in about 99% of cases.
- Assignee is usually a headquarters AE/R&D engineer.
- Assignee may also be a China FAE member when the FAE team can handle the issue directly.
- Assignee must not remain TITAN and must not be empty.

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
- `FAE Person`
- `git/repo command`

Do not audit Field Tab `Labels`, `SDK Version (TITAN)`, `Ref. H/W version`, or other fields outside the lists above.

## Fixed field ID mapping

### FAE tab

| Field | customfield_id | Type | Empty condition |
|---|---|---|---|
| `FAE_Label` | `customfield_15300` | array | null or `[]` |
| `FAE Pattern` | `customfield_15200` | option | null |
| `Comment` | `customfield_15201` | textarea/string | null or `''` |

### Field tab

| Field | customfield_id | Type | Empty condition |
|---|---|---|---|
| `O/S` | `customfield_10684` | option | null or `value === 'None'` |
| `Self Resolution` | `customfield_15009` | option | null or `value === 'None'` |
| `Cause (Customer)` | `customfield_15044` | option | null or `value === 'None'` |
| `Hardware Issue Pattern` | `customfield_15045` | option | null or `value === 'None'` |
| `Software Issue Pattern` | `customfield_15046` | option, first dropdown only | null or `value === 'None'` |
| `Labels` | `labels` | array | Ignored; do not audit |
| `FAE Person` | `customfield_15100` | user/string | null or `''` |
| `git/repo command` | `customfield_15101` | string | null or `''` |

`Labels` is shown for mapping completeness only. Do not fetch it or report it as missing in Field Tab completeness audits.

Note: Jira metadata contains two fields named `git/repo command` (`customfield_15008` and `customfield_15101`). The Field tab uses `customfield_15101`.

## Reporter/Assignee TITAN linked-assignee audit

This required audit runs after the field completeness audit every time. It replaces the older broad correction rule.

Candidate TCS ticket JQL:

```jql
reporter = "system.titan@telechips.com"
AND assignee = "system.titan@telechips.com"
ORDER BY created DESC
```

For each candidate, inspect the TCS issue page and keep only links shown as:

```text
Issue Links -> links to -> TITAN Issue: <linked-key>
```

Then open the linked TITAN Issue, for example:

```text
https://telechips-itan.atlassian.net/browse/<linked-key>
```

Report the TCS ticket only when the linked TITAN Issue Assignee is `Unassigned` or one of the 9 China FAE members. Skip linked TITAN Issues assigned to headquarters AE/R&D or any other non-China-FAE user.

Use REST/search APIs with pagination when available. If REST/XML/export is blocked, use the logged-in browser page/DOM method. Do not read or export browser cookies; let the browser session carry authentication naturally.

### China FAE team

| Username | Display Name | Email |
|---|---|---|
| `williamTang` | William Tang | williamTang@telechips.com |
| `hmyang` | HongMing Yang | hmyang@telechips.com |
| `shzhzeng` | ShengZhou Zeng | shzhzeng@telechips.com |
| `zyzhong` | ZhiYong Zhong | zyzhong@telechips.com |
| `jingoust` | 박정진 (JJ PARK) | jingoust@telechips.com |
| `Chris.Hsieh` | Chris Hsieh | Chris.Hsieh@telechips.com |
| `richard.li` | Richard Li | richard.li@telechips.com |
| `simon.sun` | Simon Sun | simon.sun@telechips.com |
| `junkai.he` | Junkai He | junkai.he@telechips.com |

## REST API audit

Use the Jira search API for bulk read-only audits:

```text
GET https://tcs.telechips.com/rest/api/2/search?jql=<JQL>&fields=<field_ids>&maxResults=50&startAt=<offset>
```

For faster missing-field exports, use JQL pre-filters. Option fields must include both `IS EMPTY` and `= None`; for example `cf[15044] IS EMPTY OR cf[15044] = None` for `Cause (Customer)`. FAE Tab `Comment` is `customfield_15201`, so use `cf[15201] IS EMPTY`, not Jira system comments.

Always paginate REST results with `startAt=0,50,100...` until collected issues reach `total`. Do not trust a visible Jira page or XML/RSS export copy as the full filter result. For leader-friendly reports, provide a compact table with assignee/FAE, missing ticket count, missing ticket list, and in-scope ticket count. Even when requested as "Only Titan Issue", keep the standard customer-scope rule and include both `TANCS*` and `TMRCR-*`; exclude `TMRCR` only if the user explicitly asks for a one-off TANCS-only report.

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
- Restrict audit results to `TANCS*` and `TMRCR` ticket prefixes listed above
- Do not report Field Tab `Labels` as missing
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
- Skipped tickets outside customer scope or with no FAE tab
- Tickets with missing required fields in either tab
- Reporter/Assignee TITAN linked-assignee findings, even when the finding count is zero

## Recent updates

- Replaced the older broad correction audit with the required Reporter/Assignee TITAN linked-assignee audit
- Kept FAE tab scope at 3 fields: `FAE_Label`, `FAE Pattern`, `Comment`
- Restricted Field tab scope to 7 checked fields and explicitly excluded `Labels`, `SDK Version (TITAN)`, and `Ref. H/W version`
- Clarified that TCS and TITAN refer to the same Jira system for this skill
- Added fixed customfield ID mapping and REST API guidance for fast audits
- Added required Reporter/Assignee TITAN linked-assignee audit for tickets still using the TITAN system account
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
