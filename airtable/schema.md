# Airtable schema — Opportunity Hub

Use these tables and fields to align Make modules and formulas. Adjust types if your base uses slightly different Airtable field types (e.g., Currency vs Number).

## Table: Raw Leads

| Field name | Suggested type | Description |
|------------|----------------|-------------|
| Source | Single line text | Marketplace identifier |
| Title | Single line text | Listing title |
| Description | Long text | Body text |
| Price | Number or Single line text | Raw or cleaned in Make first |
| Location | Single line text | City, region, or free text |
| Category | Single line text | From feed or inferred |
| Raw_Data | Long text | JSON of original item |
| Timestamp | Created time or Date | Ingest time |
| Status | Single select | `New`, `Processed` |
| Classified | Link to Classified | One-to-one or one-to-many per your design |

## Table: Classified

| Field name | Suggested type | Description |
|------------|----------------|-------------|
| Raw Lead | Link to Raw Leads | Source row |
| Label | Single select | `Supply`, `Demand`, `Pending` |
| Keywords | Long text | Comma or newline separated |
| Standardized_Price | Number | Normalized USD (or your currency) |
| Clean_Location | Single line text | Geocoded label or city |
| Category_Standard | Single line text | Mapped category e.g. Electronics |

Helper fields (optional) for scoring in formulas:

| Field name | Suggested type | Description |
|------------|----------------|-------------|
| Location_Match | Number | 0 or 1 (set by automation) |
| Category_Match | Number | 0 or 1 |
| Price_Fit | Number | 0–1 |
| Keyword_Overlap | Number | 0–1 |
| Freshness_Bonus | Number | 0–10 |

## Table: Matches

| Field name | Suggested type | Description |
|------------|----------------|-------------|
| Matched_Supply_ID | Link to Classified (record type Supply) | |
| Matched_Demand_ID | Link to Classified (record type Demand) | |
| Match_Score | Number | 0–100 |
| Notes | Long text | |
| Created | Created time | |

## Formula field example (Match_Score)

Implement component fields first (either filled by Make or rollup/lookup), then a formula on **Matches** similar to:

```
(Location_Match * 30) + (Category_Match * 25) + (Price_Fit * 20)
+ (Keyword_Overlap * 15) + (Freshness_Bonus * 10)
```

In Airtable, reference numeric fields with `{Field_Name}` and ensure each component is stored as a number. Normalize **Price_Fit** and **Keyword_Overlap** to 0–1 **before** multiplying by 20 and 15 if your formula expects that scale.

## View: Top Opportunities

- **Filter:** `{Match_Score} > 80` AND created within last 24 hours (use a formula field `Is_Recent` if needed).
- **Sort:** `{Match_Score}` descending.
