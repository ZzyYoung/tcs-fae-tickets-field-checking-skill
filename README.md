# Titan FAE Tickets Field Checking Skill

🌐 English | 🇨🇳 [中文说明](README.zh-CN.md)

An OpenClaw skill for auditing and updating Telechips TITAN Jira tickets on `tcs.telechips.com`, covering **FAE** tab fields, **Field** tab fields, and Reporter/Assignee correction status.

## System & scope

- TCS (`tcs.telechips.com`) is the Jira deployment domain; TITAN is the Jira instance/context. Treat them as the same system for this skill.
- Audit customer-scope tickets: all `TANCS*` prefixes plus `TMRCR-*`.
- Skip non-scope projects such as `TMCF-*`, `TPCP-*`, `IM*`, `IS*`, and `IG*`; they are outside this skill's FAE/Field audit scope.
- For large check-only audits, use Jira REST API paging instead of opening tickets one by one.

## What this skill does

This skill supports two audit modes plus one edit workflow:

1. **Field completeness audit**
   - Audit another reporter's Jira tickets
   - Check both **FAE** tab fields and **Field** tab fields
   - Return only ticket keys and the missing fields
   - Ignore Field Tab `Labels`
   - Prefer high-efficiency read-only inspection for large audits

2. **Reporter/Assignee correction audit**
   - Scan `TITAN_Customer` TANCS candidates whose Reporter/Assignee still uses the TITAN system account or whose Assignee is empty
   - Use Issue Links to check the linked issue Assignee before reporting
   - Report only candidates whose linked issue Assignee is one of the 9 China FAE members
   - Do not group by reporter because problem tickets may still have `reporter = titan`

3. **Edit/update mode**
   - Open your own Jira tickets from a filtered JQL result
   - Enter the **FAE** tab before editing FAE-related content
   - Fill or update relevant fields safely
   - Keep browser session intact after updates

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

## Reporter/Assignee correction audit

Use this JQL to scan candidate TANCS tickets:

```jql
project = TITAN_Customer
AND (reporter in ("titan") OR assignee in ("titan") OR assignee is EMPTY)
AND created >= 2025-01-01
ORDER BY created DESC
```

Fetch fields:

```text
summary,reporter,assignee,created,issuelinks
```

Then extract linked issue keys from both `outwardIssue` and `inwardIssue`, without any linked-key prefix filter. Fetch each linked issue with:

```text
GET /rest/api/2/issue/<linked-issue-key>?fields=assignee,summary
```

Only report the TANCS candidate when at least one linked issue Assignee username, lowercased, is in the China FAE list. If no Issue Links exist, put the ticket in an `UNCERTAIN - no issue links` section for manual review. If linked issue Assignees are not China FAE, skip the ticket as out of China FAE scope.

Judgment rules:

| Field | Expected value | Non-compliant case | Severity |
|---|---|---|---|
| Reporter | China FAE identified via linked issue Assignee | Still TITAN system account on in-scope TANCS ticket | HIGH |
| Assignee | China FAE identified via linked issue Assignee, or valid owner after correction | Still TITAN system account on in-scope TANCS ticket | HIGH |
| Assignee | Valid owner after correction | Unassigned / null on in-scope TANCS ticket | MEDIUM |
| Scope | Linked issue Assignee is China FAE | No Issue Links | UNCERTAIN |

China FAE assignee is valid; headquarters AE/R&D assignee is also valid. For TITAN-placeholder TANCS candidates, the China FAE ownership decision must come from linked issue Assignee only. Match usernames case-insensitively.

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

Always paginate REST results with `startAt=0,50,100...` until collected issues reach `total`. Do not trust a visible Jira page or XML/RSS export copy as the full filter result. For leader-friendly reports, provide a compact table with assignee/FAE, missing ticket count, missing ticket list, and in-scope ticket count; when requested as "Only Titan Issue", include only `TANCS*` and exclude `TMRCR`.

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
- For Mode B: China FAE tickets to fix, uncertain tickets with no Issue Links, and skipped out-of-scope summary/examples

## Recent updates

- Refined Mode B to confirm China FAE scope through linked issue Assignee lookup before reporting TITAN-placeholder TANCS tickets
- Kept FAE tab scope at 3 fields: `FAE_Label`, `FAE Pattern`, `Comment`
- Restricted Field tab scope to 7 checked fields and explicitly excluded `Labels`, `SDK Version (TITAN)`, and `Ref. H/W version`
- Clarified that TCS and TITAN refer to the same Jira system for this skill
- Added fixed customfield ID mapping and REST API guidance for fast audits
- Added Reporter/Assignee correction audit for tickets still using the TITAN system account
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
