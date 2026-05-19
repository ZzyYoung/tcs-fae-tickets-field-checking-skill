# Field Tab — Custom Field ID Lookup Guide

This file explains the correct Jira custom field IDs for the 7 checked Field Tab fields
that must be audited in customer-scope tickets. TCS (`tcs.telechips.com`) is the Jira
deployment domain and TITAN is the Jira instance/context; treat them as the same
system for this skill.

Only audit customer-scope tickets: all `TANCS*` prefixes (for example `TANCS-*`,
`TANCS1-*`, `TANCS2-*`, `TANCS3-*`, etc.) plus `TMRCR-*`. Skip non-scope
projects such as `TMCF-*`, `TPCP-*`, `IM*`, `IS*`, and `IG*`.

## Step 1: Fetch all fields from TITAN

```
GET https://tcs.telechips.com/rest/api/2/field
```

This returns a JSON array. Each entry looks like:

```json
{
  "id": "customfield_10400",
  "name": "O/S",
  "custom": true,
  "orderable": true,
  "navigable": true,
  "searchable": true,
  "clauseNames": ["cf[10400]", "OS"],
  "schema": { "type": "string", "custom": "...", "customId": 10400 }
}
```

## Step 2: Search for each target field by name

Search the returned array for entries where `name` matches (case-insensitive):

| Target Name | Notes |
|---|---|
| `O/S` | May also appear as "OS" |
| `Self Resolution` | Exact match |
| `Cause (Customer)` | Customer cause classification |
| `Hardware Issue Pattern` | May be abbreviated |
| `Software Issue Pattern` | Audit only the first dropdown: `customfield_15046` |
| `Labels` | Built-in Jira field `labels`, kept for mapping only; do not audit |
| `FAE Person` | May appear as "FAE_Person" or similar |
| `git/repo command` | Use `customfield_15101`; metadata also contains `customfield_15008` with the same name |

Do not audit Field Tab `Labels`, `SDK Version (TITAN)`, `Ref. H/W version`, or any other fields outside the scope above. `Labels` is still a built-in Jira field named `labels`, but it is intentionally ignored for Field Tab completeness audits.

## Step 3: Note the IDs

Record the `id` value (e.g. `customfield_10400`) for each field. Use these IDs in the
`fields` parameter of search API calls.

## Step 4: Example search API call

Use the known TITAN_Customer IDs like this:

```
GET /rest/api/2/search
  ?jql=created >= 2025-01-01 AND reporter in ("zyzhong@telechips.com") ORDER BY created DESC
  &fields=summary,customfield_10684,customfield_15009,customfield_15044,
          customfield_15045,customfield_15046,customfield_15100,
          customfield_15101,customfield_15200,customfield_15201,customfield_15300
  &maxResults=50
  &startAt=0
```

## Step 4a: JQL pre-filter for missing values

Use Jira JQL to export only likely-missing tickets when possible. Option fields must check both empty and `None`:

```jql
(
  cf[10684] IS EMPTY OR cf[10684] = None OR
  cf[15009] IS EMPTY OR cf[15009] = None OR
  cf[15044] IS EMPTY OR cf[15044] = None OR
  cf[15045] IS EMPTY OR cf[15045] = None OR
  cf[15046] IS EMPTY OR cf[15046] = None OR
  cf[15100] IS EMPTY OR
  cf[15101] IS EMPTY OR
  FAE_Label = empty OR
  cf[15200] = empty OR
  cf[15201] IS EMPTY
)
AND created >= 2025-01-01
AND reporter in ("user@telechips.com")
ORDER BY created DESC
```

Do not miss `cf[15044] = None`; `Cause (Customer): None` is a missing value.

## Step 5: Empty value detection per field type

Different field types serialize differently in the API response:

| Field type | Empty looks like |
|---|---|
| Single select / option | `null` or `{ "value": "None" }` or `{ "value": "" }` |
| Multi select | `null` or `[]` |
| Text / string | `null` or `""` |
| User picker | `null` |

Use the empty-value rules in `SKILL.md` to normalize all of these to a simple true/false.

## Known TITAN_Customer field IDs

### FAE Tab

| Field | customfield ID | Type | Empty condition |
|---|---|---|---|
| FAE_Label | `customfield_15300` | array | null or `[]` |
| FAE Pattern | `customfield_15200` | option | null |
| Comment | `customfield_15201` | textarea/string | null or `""` |

### Field Tab

| Field | customfield ID | Type | Empty condition |
|---|---|---|---|
| O/S | `customfield_10684` | single select | null or `value === 'None'` |
| Self Resolution | `customfield_15009` | single select | null or `value === 'None'` |
| Cause (Customer) | `customfield_15044` | single select | null or `value === 'None'` |
| Hardware Issue Pattern | `customfield_15045` | single select | null or `value === 'None'` |
| Software Issue Pattern | `customfield_15046` | single select, first dropdown only | null or `value === 'None'` |
| Labels | `labels` (built-in) | label array | ignored; do not audit |
| FAE Person | `customfield_15100` | user picker or string | null or empty string |
| git/repo command | `customfield_15101` | text | null or empty string |

`Labels` maps to Jira built-in field `labels`, but it is kept only for mapping awareness and is not part of the current Field Tab completeness audit.
