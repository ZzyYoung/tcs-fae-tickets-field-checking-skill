# Usage Guide — English

## Quick start

1. Ensure Chrome is open and TITAN is logged in.
2. Provide the JQL for the audit (reporter, date range).
3. Tell Claude whether this is **check-only** or **edit/update** mode.
4. For check-only: Claude will audit both the Field tab AND the FAE tab.
5. For edit/update: Claude will only update FAE tab fields (as instructed).

## Audit scope

### Field Tab (8 fields)
- O/S
- Self Resolution
- Cause(Customer)
- Hardware Issue Pattern
- Software Issue Pattern (first field only)
- Labels
- FAE Person
- git/repo command

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

## Audit modes

### Check-only (recommended for auditing another person's tickets)
- Claude inspects tickets without modifying them.
- Both Field tab and FAE tab fields are checked.
- Results are returned as a structured report.

### Edit/update (only for your own tickets or with explicit permission)
- Claude modifies FAE tab fields only.
- Field tab fields are checked but NOT modified automatically (user decides).
- Always clicks FAE tab first before editing.

## Output format

The audit report includes:
- Summary counts
- Per-ticket breakdown of missing fields (Field tab vs FAE tab)
- List of skipped tickets (no FAE tab)

## Tips

- For large audits (50+ tickets), the REST API method is preferred over browser clicking.
- Always verify custom field IDs using `GET /rest/api/2/field` before first use.
- Software Issue Pattern: only the first field is audited if multiple exist.
