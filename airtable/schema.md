# Airtable schema — Opportunity Hub (reference)

Single source of truth for table and field names. Align Make modules and [`SETUP.md`](SETUP.md) with this document.

## Base

- **Name:** Opportunity Hub  
- **Tables:** Raw Leads → Classified → Matches  

---

## Table: Raw Leads

Primary field: **Title**

| Field name | Type | Required | Notes |
|------------|------|----------|--------|
| Title | Single line text | Yes | Primary; listing title |
| Source | Single line text | | Marketplace name (Craigslist, etc.) |
| Description | Long text | | Full body |
| Price | Number or Single line text | | Normalize in Make for downstream math |
| Location | Single line text | | Geocode in Make |
| Category | Single line text | | From feed or inferred |
| Raw_Data | Long text | | Original JSON / bundle |
| Timestamp | Date (time) | | Ingest time if not using Created only |
| Status | Single select | | `New`, `Processed` |
| Classified | Link to Classified | | Usually one classified row per raw lead |

**Volume:** Designed for **500+ rows/day**; index/filter by `Status` and `Timestamp` in views.

---

## Table: Classified

Primary field: **Display Title** (or equivalent short text)

| Field name | Type | Required | Notes |
|------------|------|----------|--------|
| Display Title | Single line text | Yes | Primary; mirror of listing title for scanning |
| Raw Lead | Link to Raw Leads | Yes | Source row |
| Label | Single select | | `Supply`, `Demand`, `Pending` |
| Keywords | Long text | | Auto-extracted in Make |
| Standardized_Price | Number | | e.g. USD |
| Clean_Location | Single line text | | After geocode / cleanup |
| Category_Standard | Single line text | | Mapped category (Electronics, …) |

**Auto-classify target:** ~**95%** into Supply/Demand; remainder **Pending** for manual review.

---

## Table: Matches

Links two **Classified** rows (one supply, one demand). Scoring components live here; **Match_Score** is a **Formula** (see [`formulas.md`](formulas.md)).

Primary field: **Match ID** (Autonumber recommended)

| Field name | Type | Notes |
|------------|------|--------|
| Match ID | Autonumber | Primary |
| Matched_Supply | Link to Classified | Supply-side classified row |
| Matched_Demand | Link to Classified | Demand-side classified row |
| Loc_Match | Number | 0 or 1 |
| Cat_Match | Number | 0 or 1 |
| Price_Fit | Number | 0–1 |
| Keyword_Overlap | Number | 0–1 |
| Freshness_Mult | Number | 0–1 |
| Match_Score | Formula | Weighted 0–100 |
| Is_Recent | Formula | Last 24h (for views) |
| Notes | Long text | Optional |
| Created | Created time | For freshness filters |

**Thresholds:** **>70** = strong match (optional view); **>80** + last 24h = **Top Opportunities** (see views).

---

## Views

### Matches → Top Opportunities

- **Filter:** `Match_Score` **>** `80` **AND** `Is_Recent` (last 24h).  
- **Sort:** `Match_Score` descending.

### Matches → Strong matches (optional)

- **Filter:** `Match_Score` **>** `70`.

### Raw Leads → New queue (optional)

- **Filter:** `Status` = `New`.

### Classified → Pending (optional)

- **Filter:** `Label` = `Pending`.

---

## Formula summary

```
Match_Score =
  (Loc_Match × 30) + (Cat_Match × 25) + (Price_Fit × 20)
  + (Keyword_Overlap × 15) + (Freshness_Mult × 10)
```

Copy-paste versions: [`formulas.md`](formulas.md).

---

## Naming: spec vs base

| Spec wording | Suggested field name in base |
|--------------|------------------------------|
| Matched_Supply_ID | `Matched_Supply` (link) |
| Matched_Demand_ID | `Matched_Demand` (link) |
| Supply/Demand label | `Label` on Classified |

Record IDs for API/Make: use Airtable record `id` from linked rows.
