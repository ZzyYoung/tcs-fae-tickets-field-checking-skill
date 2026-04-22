# Field Tab — Custom Field ID Lookup Guide

This file explains how to find the correct Jira custom field IDs for the 10 Field Tab fields
that must be audited in TITAN. Custom field IDs differ per Jira instance and must be verified
before running API-based audits.

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
| `Cause(Customer)` | May include parentheses |
| `Hardware Issue Pattern` | May be abbreviated |
| `Software Issue Pattern` | If multiple, take the one with the **lowest** customId (first defined) |
| `Labels` | This is a standard built-in field — use `labels` directly, no customfield ID needed |
| `FAE Person` | May appear as "FAE_Person" or similar |
| `git/repo command` | May include slash or be hyphenated |
| `SDK Version (TITAN)` | TITAN SDK version field |
| `Ref. H/W version` | Reference hardware version field |

## Step 3: Note the IDs

Record the `id` value (e.g. `customfield_10400`) for each field. Use these IDs in the
`fields` parameter of search API calls.

## Step 4: Example search API call

Once you have the IDs, use them like this (replace the customfield IDs with actual ones):

```
GET /rest/api/2/search
  ?jql=created >= 2025-01-01 AND reporter in ("zyzhong@telechips.com") ORDER BY created DESC
  &fields=customfield_10400,customfield_10401,customfield_10402,customfield_10403,
          customfield_10404,labels,customfield_10405,customfield_10406
  &maxResults=50
  &startAt=0
```

## Step 5: Empty value detection per field type

Different field types serialize differently in the API response:

| Field type | Empty looks like |
|---|---|
| Single select / option | `null` or `{ "value": "None" }` or `{ "value": "" }` |
| Multi select | `null` or `[]` |
| Text / string | `null` or `""` |
| Labels (built-in) | `[]` |
| User picker | `null` |

Use the `isBlank()` function defined in SKILL.md to normalize all of these to a simple true/false.

## Known TITAN field IDs (update as discovered)

> Fill in as you confirm IDs by running the /field API call.

| Field | Confirmed customfield ID | Type |
|---|---|---|
| O/S | TBD | single select |
| Self Resolution | TBD | single select |
| Cause(Customer) | TBD | single select |
| Hardware Issue Pattern | TBD | single select |
| Software Issue Pattern | TBD | single select |
| Labels | `labels` (built-in) | label array |
| FAE Person | TBD | user picker |
| git/repo command | TBD | text |
| SDK Version (TITAN) | TBD | text or select |
| Ref. H/W version | TBD | text or select |
