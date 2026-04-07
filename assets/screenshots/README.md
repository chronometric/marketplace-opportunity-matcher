# Screenshots and portfolio assets

Add images here for **documentation**, **README** embeds, and **portfolio** (Upwork, case studies). Prefer **PNG** or **WebP**; keep files **under ~500 KB** each when possible.

## Required redactions

Before **git commit**:

- API keys, tokens, webhook URLs  
- Personal emails, phone numbers, real street addresses  
- Client or marketplace account names if under NDA  

Use fake data or crop panels that show structure only.

## Suggested captures

| # | Asset | What to show |
|---|--------|----------------|
| 1 | `airtable-raw-leads.png` | **Raw Leads** grid; Status filter **New** / **Processed** |
| 2 | `airtable-classified.png` | **Classified** with **Label** Supply / Demand / Pending |
| 3 | `airtable-matches.png` | **Matches** sorted by **Match_Score** |
| 4 | `make-daily-pull.png` | **Make** scenario canvas — **Daily Pull** (blur connection names if needed) |
| 5 | `make-flow-overview.png` | Optional second scenario — **Clean & Classify** or **Match & Score** |
| 6 | `sheets-daily-top-10.png` | **Google Sheets** tab **Daily_Top_10** with headers |

## README embed

After the PNG files are committed on `main`, paste the full grid from **[`EMBED.md`](EMBED.md)** into the root [`README.md`](../../README.md) **Screenshots** section (or use the one-line example below).

```markdown
![Airtable Raw Leads](assets/screenshots/airtable-raw-leads.png)
```

Use **relative paths** so GitHub renders them on the default branch. If images look broken, the files are missing from the branch or the path does not match exactly (case-sensitive on some systems).

## Portfolio checklist

- [ ] At least **three** images: Airtable + Make + Sheets  
- [ ] Short **caption** in README or case-study doc  
- [ ] Same visual style (light theme, similar zoom)  

See [`docs/TESTING.md`](../../docs/TESTING.md) for launch validation.
