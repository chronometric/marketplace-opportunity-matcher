# Scoring formula

## Weighted components (reference)

| Component | Weight (points) | Typical implementation |
|-----------|-----------------|-------------------------|
| Location_Match | 30 | 1 if same city or within radius; else 0 |
| Category_Match | 25 | 1 on exact/parent match; partial weights optional |
| Price_Fit | 20 | Scale 0–1: how well supply price fits demand budget |
| Keyword_Overlap | 15 | Scale 0–1 from overlap count / max keywords |
| Freshness_Bonus | 10 | 10 if &lt; 24h; linear decay to 0 |

**Total:** 100 when all components are normalized to 0–1 except where noted.

## Formula (conceptual)

```
Score = (Location_Match × 30) + (Category_Match × 25) + (Price_Fit × 20)
        + (Keyword_Overlap × 15) + (Freshness_Bonus × 10)
```

## Airtable vs Make

- **Option A:** Make computes each component and writes numeric fields to **Matches**; Airtable **formula** field sums the weighted score.
- **Option B:** Components live in **Classified** or helper tables; **Rollups** and **Lookups** feed a **Matches** formula.

## Thresholds

- **> 70:** strong match for prioritization.
- **> 80:** typical cutoff for **Top Opportunities** view and Daily Top 10.

## Spreadsheet

See [`scoring-components.csv`](scoring-components.csv) for a Google Sheets–friendly layout (weights, example values, and calculated score).
