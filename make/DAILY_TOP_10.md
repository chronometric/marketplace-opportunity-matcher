# Daily Top 10 — Top Opportunities → Google Sheets (Make)

After **Match & Score** has run, publish a fixed **Top 10** list for stakeholders: query Airtable **Top Opportunities**, **dedupe** pairs, sort by **Match_Score**, write **`Daily_Top_10`** tab, optionally **email** the link and **Slack** a short summary.

**Schedule:** typically **07:00** daily (after ingest + matching windows; adjust timezone).

## Preconditions

- **Matches** table + **Top Opportunities** view: Score **>80**, last **24h** ([`airtable/SETUP.md`](../airtable/SETUP.md)).  
- Google Sheet created; tab **`Daily_Top_10`** (or name you map in Make).  
- OAuth: **Google Sheets** + optional **Gmail** in Make.

## Flow

1. **Schedule** — Cron e.g. **07:00** daily; timezone = team.  
2. **Airtable — Search records** — Table **Matches**, use **view** **Top Opportunities** *or* same filters as that view (Score, `Is_Recent`).  
3. **Sort** — **Match_Score** descending (if not already in view).  
4. **Deduplicate** — Unique by pair `(Matched_Supply record id, Matched_Demand record id)`. If duplicates exist from reruns, keep highest score: **Array** / **Aggregate** by key or **Iterator** + **Data store** last-seen.  
5. **Limit** — Take first **10** bundles (`limit` or **Iterator** with counter).  
6. **Google Sheets — Add a row** (batch) or **Update a range**  
   - **Option A:** **Clear** rows 2–11 (keep header row 1), then **Add rows** for each of 10 records.  
   - **Option B:** Single **Update range** with a 10-row × 4-column matrix built in **Array aggregator**.

## Sheet columns

| Column | Source / formula |
|--------|------------------|
| **Opportunity_Pair** | e.g. `{Supply Display Title} ↔ {Demand Display Title}` from lookups to **Classified** via **Matched_Supply** / **Matched_Demand** |
| **Score** | **Match_Score** (formula field) |
| **Links** | Airtable record URLs: `https://airtable.com/appXXX/tblYYY/recZZZ` for supply and demand Classified rows (build with **Base ID** + table IDs + record ids from links), or listing URLs from **Raw_Data** if stored |
| **Action_Next** | Static text or template: e.g. `Review in Airtable` + Slack channel `@here` |

Header row (row 1): `Opportunity_Pair | Score | Links | Action_Next`

## Lookups in Make

- From each **Match**, expand **Matched_Supply** → **Classified** → **Display Title**, **Raw Lead** → URL if needed.  
- Same for **Matched_Demand**.

## Optional notifications

| Channel | Purpose |
|---------|---------|
| **Gmail** | Send team distribution list a **link** to the Sheet (read-only) each morning. |
| **Slack** | Post **#channel**: date, count of rows, top Score, link to Sheet. |

## Airtable dashboard

- Embed read-only **Interface** or **shared view** in Notion/Confluence; link from **Action_Next** or team wiki (manual step outside Make).

## Errors

- If Airtable returns 0 rows: write a single “No opportunities today” row or skip Sheet update; optional Slack info message.  
- **Slack** on module failure (same pattern as [`DAILY_PULL.md`](DAILY_PULL.md)).

## After build

Export blueprint → [`scenarios/daily-top-10.json`](scenarios/daily-top-10.json). See [`IMPORT_INSTRUCTIONS.md`](IMPORT_INSTRUCTIONS.md).
