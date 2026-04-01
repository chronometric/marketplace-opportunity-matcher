# Airtable formulas — Opportunity Hub

Paste into **Formula** fields on the **Matches** table. Field names must match your base exactly (rename fields in Airtable to match or adjust references).

## Prerequisites

On **Matches**, these **Number** fields are filled by Make (or manual test):

| Field | Allowed range | Meaning |
|-------|----------------|--------|
| `Loc_Match` | 0 or 1 | Same city / within distance rule |
| `Cat_Match` | 0 or 1 | Category match |
| `Price_Fit` | 0–1 | How well supply price fits demand budget |
| `Keyword_Overlap` | 0–1 | Normalized overlap (e.g. count / max) |
| `Freshness_Mult` | 0–1 | 1 = full freshness (&lt; 24h), 0 = stale |

## Match_Score (Formula field)

Weighted total **0–100**:

```
(
  {Loc_Match} * 30
) + (
  {Cat_Match} * 25
) + (
  {Price_Fit} * 20
) + (
  {Keyword_Overlap} * 15
) + (
  {Freshness_Mult} * 10
)
```

**Note:** If any component is empty, the formula may error—default blanks to `0` in Make or use `IF` wrappers:

```
(
  IF({Loc_Match}, {Loc_Match}, 0) * 30
) + (
  IF({Cat_Match}, {Cat_Match}, 0) * 25
) + (
  IF({Price_Fit}, {Price_Fit}, 0) * 20
) + (
  IF({Keyword_Overlap}, {Keyword_Overlap}, 0) * 15
) + (
  IF({Freshness_Mult}, {Freshness_Mult}, 0) * 10
)
```

## Is_Recent (Formula field)

True when the match record was created in the last **24 hours** (uses **Created** created-time field—add a **Created time** field named `Created` if you use a custom name, and reference it below).

```
DATETIME_DIFF(NOW(), CREATED_TIME(), 'hours') < 24
```

If your workspace uses a custom **Created time** field called `Created` instead of `CREATED_TIME()`:

```
DATETIME_DIFF(NOW(), {Created}, 'hours') < 24
```

Airtable returns **1** or **0** for comparisons in some contexts; for filters, use this field in the view as “is” checked or `= 1`.

## Freshness_Mult (optional: compute in Airtable instead of Make)

If you prefer freshness from a **listing** timestamp, add lookups from Classified → Raw Leads → `Timestamp`, then a formula on **Matches**—more complex. By default, **Make** sets `Freshness_Mult` from listing age.

Linear decay example **on a helper field** (if you store `Hours_Old` as a number from Make):

```
MAX(0, 1 - ({Hours_Old} / 24))
```

(Use only if you add `Hours_Old`; optional.)
