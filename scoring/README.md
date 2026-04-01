# Scoring formula

## Weighted components (reference)

| Component | Weight (points) | Typical implementation |
|-----------|-----------------|-------------------------|
| Loc_Match (concept: location) | 30 | 1 if same city or within radius; else 0 |
| Cat_Match (concept: category) | 25 | 1 on exact/parent match; partial weights optional |
| Price_Fit | 20 | Scale 0–1: how well supply price fits demand budget |
| Keyword_Overlap | 15 | Scale 0–1 from overlap count / max keywords |
| Freshness_Mult | 10 | 0–1 then ×10 in formula; 1 = full freshness &lt; 24h (see [`airtable/formulas.md`](../airtable/formulas.md)) |

**Total:** 100 when all components are normalized to 0–1 except where noted.

## Formula (conceptual)

```
Score = (Loc_Match × 30) + (Cat_Match × 25) + (Price_Fit × 20)
        + (Keyword_Overlap × 15) + (Freshness_Mult × 10)
```

Field names match the **Matches** table in [`airtable/schema.md`](../airtable/schema.md).

## Airtable vs Make

- **Option A:** Make computes each component and writes numeric fields to **Matches**; Airtable **formula** field sums the weighted score.
- **Option B:** Components live in **Classified** or helper tables; **Rollups** and **Lookups** feed a **Matches** formula.

## Thresholds

- **> 70:** strong match for prioritization.
- **> 80:** typical cutoff for **Top Opportunities** view and Daily Top 10.

## Spreadsheet

See [`scoring-components.csv`](scoring-components.csv) for a Google Sheets–friendly layout (weights, example values, and calculated score).
