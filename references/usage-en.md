# Usage Guide — English

## Quick start

1. Ensure Chrome is open and TITAN is logged in.
2. Provide the JQL for the audit (reporter, date range).
3. Tell Claude whether this is field completeness audit, Reporter/Assignee correction audit, or edit/update mode.
4. For field completeness: Claude will audit both the Field tab AND the FAE tab, excluding Field Tab Labels.
5. For Reporter/Assignee correction: Claude will scan TANCS candidates, then use Issue Links to keep only tickets whose linked issue Assignee is China FAE.
6. For edit/update: Claude will only update FAE tab fields (as instructed).

## System and project scope

- TCS (`tcs.telechips.com`) is the Jira deployment domain; TITAN is the Jira instance/context. They are the same system for this skill.
- Audit only customer-scope issue keys: any prefix starting with `TANCS` plus `TMRCR`.
- Skip `TMCF-*`, `TPCP-*`, `IM*`, `IS*`, `IG*`, and other projects outside customer scope.
- For large audits, use Jira REST API search pages (`maxResults=50`) instead of ticket-by-ticket browser clicking.

## Audit scope

### Field Tab (7 checked fields)
- O/S
- Self Resolution
- Cause (Customer)
- Hardware Issue Pattern
- Software Issue Pattern (first field only)
- FAE Person
- git/repo command

Do not audit Field Tab Labels, SDK Version (TITAN), Ref. H/W version, or other out-of-scope fields.

### FAE Tab (3 fields)
- FAE_Label
- FAE Pattern
- Comment

## Common JQL patterns

**By reporter, date range:**
```
created >= 2025-01-01 AND created <= 2026-04-17 AND reporter in ("user@telechips.com") order by created DESC
```

**Current user:**
```
created >= 2025-01-01 AND reporter in (currentUser()) order by created DESC
```

**By project:**
```
project = TANCS5 AND created >= 2025-01-01 AND reporter in ("user@telechips.com") order by created DESC
```

If the JQL is broad, still filter fetched results to `TANCS*` and `TMRCR` before reporting.

**Reporter/Assignee correction audit:**
```jql
project = TITAN_Customer
AND (reporter in ("titan") OR assignee in ("titan") OR assignee is EMPTY)
AND created >= 2025-01-01
ORDER BY created DESC
```

## REST API quick audit

Use:

```text
GET https://tcs.telechips.com/rest/api/2/search?jql=<JQL>&fields=<field_ids>&maxResults=50&startAt=<offset>
```

Required fields:

```text
summary,customfield_10684,customfield_15009,customfield_15044,customfield_15045,customfield_15046,customfield_15100,customfield_15101,customfield_15200,customfield_15300,comment
```

Authentication options: browser Console on `tcs.telechips.com`, PAT if supported, or a copied `JSESSIONID` Cookie header.

For Reporter/Assignee correction audits, fetch:

```text
summary,reporter,assignee,created,issuelinks
```

## Audit modes

### Field completeness audit (recommended for auditing another person's tickets)
- Claude inspects tickets without modifying them.
- Both Field tab and FAE tab fields are checked.
- Field Tab Labels is ignored.
- Results are returned as a structured report.
- For large multi-page audits, prefer the REST API.
- Skip tickets outside customer scope or with no FAE tab.
- Record ticket key plus all missing fields.

### Reporter/Assignee correction audit
- Claude inspects tickets without modifying them.
- Use the reverse-filter JQL above to scan TANCS candidates.
- Extract linked issue keys from both `outwardIssue` and `inwardIssue`; do not filter linked issue keys by prefix.
- Fetch each linked issue with `GET /rest/api/2/issue/<linked-key>?fields=assignee,summary`.
- Report only TANCS candidates whose linked issue Assignee username is one of the 9 China FAE members.
- Put candidates with no Issue Links in an uncertain/manual-review section.
- Skip candidates linked only to non-China-FAE assignees as out of China FAE scope.
- Match usernames case-insensitively.

### Edit/update (only for your own tickets or with explicit permission)
- Claude modifies FAE tab fields only.
- Field tab fields are checked but NOT modified automatically, unless the user explicitly asks.
- Always click FAE tab first before editing.
- For FAE_Label, select a real label suggestion after typing.
- Wait briefly before updating.
- Use ticket-specific English comments, not one generic comment for all tickets.

## Output format

The audit report includes:
- Summary counts
- Per-ticket breakdown of missing fields (Field tab vs FAE tab)
- Reporter/Assignee severity counts when that audit mode is used
- For Mode B: in-scope China FAE tickets to fix, uncertain tickets with no Issue Links, and skipped out-of-scope summary/examples

## Tips

- For large audits (50+ tickets), the REST API method is preferred over browser clicking.
- Use the fixed customfield IDs in `SKILL.md`; call `GET /rest/api/2/field` only to verify metadata drift.
- Software Issue Pattern: only the first field is audited if multiple exist.
- Labels is a built-in Jira field, but it is ignored for Field Tab completeness audits.
- `git/repo command`: use `customfield_15101`, not the duplicate metadata field `customfield_15008`.
- China FAE as Assignee is valid; headquarters AE/R&D as Assignee is valid.
- Assignee empty/null is non-compliant.
- `safellink`: only for TCC5110 and SDM related tickets.
- `CarPlay`: for CarPlay related issues.
- `notification`: for notice/announcement tickets when creating a new label.
