# Build “Opportunity Hub” in Airtable

Follow this order so links and formulas resolve. After the base matches this spec, use **Share → duplicate link / copy base** and paste the URL into [`AIRTABLE_TEMPLATE.md`](AIRTABLE_TEMPLATE.md).

## 1. Create the base

1. [Airtable](https://airtable.com) → **Add a base** → **Start from scratch**.
2. Name: **Opportunity Hub**.
3. Rename **Table 1** to **Raw Leads** (exact name helps Make mappings).

## 2. Table: Raw Leads

Set the **primary field** (first column) to **Title** (short text). Add fields:

| Field name | Type | Config |
|------------|------|--------|
| Title | Single line text | Primary |
| Source | Single line text | |
| Description | Long text | |
| Price | Number | Format: decimal or integer (or Single line text if you ingest raw strings first) |
| Location | Single line text | |
| Category | Single line text | |
| Raw_Data | Long text | Full JSON / payload |
| Timestamp | Date | Include time; or use **Created time** system field via API only—in UI add **Ingested At** Date if needed |
| Status | Single select | Options: `New`, `Processed` (default `New`) |
| Classified | Link to another record | Link to **Classified** (create table in next step; you can add link after Classified exists) |

**Order tip:** Add **Classified** link *after* creating the **Classified** table (step 3), then allow **multiple records** only if one raw lead can spawn multiple classified rows (usually **no**—single link).

## 3. Table: Classified

1. Add a new table **Classified**.
2. Primary field: **Display Title** (Single line text)—Make can set this from Raw Lead title for readability.

| Field name | Type | Config |
|------------|------|--------|
| Display Title | Single line text | Primary |
| Raw Lead | Link to another record | Links to **Raw Leads** (allow one record) |
| Label | Single select | `Supply`, `Demand`, `Pending` |
| Keywords | Long text | |
| Standardized_Price | Number | |
| Clean_Location | Single line text | |
| Category_Standard | Single line text | Normalized category (e.g. Electronics) |

Back on **Raw Leads**, add or confirm **Classified** ↔ link so each processed raw row points to one classified row.

## 4. Table: Matches

1. New table **Matches**.
2. Primary field: **Match ID** — use **Autonumber** or a short text key; optional second column for a human-readable label later.

| Field name | Type | Config |
|------------|------|--------|
| Match ID | Autonumber | Primary (recommended) |
| Matched_Supply | Link to another record | Links to **Classified**; description: supply-side row |
| Matched_Demand | Link to another record | Links to **Classified**; description: demand-side row |
| Loc_Match | Number | Integer `0` or `1` (filled by Make) |
| Cat_Match | Number | Integer `0` or `1` |
| Price_Fit | Number | Decimal **0–1** |
| Keyword_Overlap | Number | Decimal **0–1** |
| Freshness_Mult | Number | Decimal **0–1** (1 = full freshness &lt; 24h) |
| Match_Score | Formula | See [`formulas.md`](formulas.md) |
| Is_Recent | Formula | See [`formulas.md`](formulas.md) |
| Notes | Long text | Optional |
| Created | Created time | System: add **Created time** field type if not present |

**Validation (optional):** Use Airtable **Interface** or automation later to ensure Supply link has `Label = Supply` and Demand has `Label = Demand`.

## 5. Views on Matches

### Top Opportunities

- **Type:** Grid (or Gallery).
- **Filter:**  
  - `Match_Score` **>** `80` **AND** `Is_Recent` **is** checked / equals formula true (see formulas—use `Is_Recent` numeric 1 or checkbox-style formula).
- **Sort:** `Match_Score` descending.

If your `Is_Recent` is a number (1/0), filter `Is_Recent = 1` and `Match_Score > 80`.

### Strong matches (optional)

- **Filter:** `Match_Score` **>** `70`.
- Use for triage before the daily Top 10.

## 6. Views on Raw Leads / Classified (optional)

- **Raw Leads — New:** `Status` = `New`.
- **Classified — Pending review:** `Label` = `Pending`.

## 7. Publish template link

1. Base menu → **Share** → create a link that allows **Copy base** (or duplicate into workspace).
2. Copy URL into [`AIRTABLE_TEMPLATE.md`](AIRTABLE_TEMPLATE.md).
3. Record **Base ID** from **Help → API documentation** for Make.

## 8. API token

Create a **Personal Access Token** with access to this base ([API docs](https://airtable.com/developers/web/api/introduction)). Store in Make connections only—never commit secrets to git.

---

**Setup complete when:** All three tables exist, links work, **Match_Score** and **Is_Recent** formulas calculate, **Top Opportunities** view shows only high scores from the last 24h, and the share link is in the repo.
