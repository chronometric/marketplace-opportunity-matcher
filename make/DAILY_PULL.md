# Daily Pull — RSS & CSV → Raw Leads (Make)

Ingest listings **without scraping**: **RSS feeds**, optional **manual CSV** (or webhook), then **geocode** and write **Airtable → Raw Leads**. On failure: **retry up to 3×** and **notify Slack**.

## Goals

| Requirement | Implementation |
|-------------|------------------|
| Schedule | **Schedule** module — e.g. **06:00** daily, timezone = your market |
| RSS | **RSS > Read feed** (or **HTTP** GET + XML parse if feed is non-standard) |
| CSV / manual | Separate **Webhook** (instant) and/or **Google Drive** “Watch files” → **CSV** parse |
| Geocode | **HTTP** — Google **Geocoding API** (Maps Platform; enable billing & restrict API key) |
| Raw Leads | **Airtable** — Create record |
| Reliability | **Break** / **Error handler** → **Repeater** or **retry** pattern; cap **3** attempts |
| Alerts | **Slack** — post channel message with error text + bundle excerpt |

**Zapier fallback (optional):** If email delivers CSVs, use Zapier to **parse attachment** → **Make webhook** with JSON payload matching the same shape as the CSV branch below.

---

## Airtable mapping — `Raw Leads`

| Airtable field | Source | Notes |
|----------------|--------|--------|
| **Title** | RSS `title` or CSV column | Primary field |
| **Source** | Constant per feed (e.g. `Craigslist`) or CSV column | |
| **Description** | RSS `description` / `content` or CSV | Strip HTML if needed |
| **Price** | Parsed from title/description (regex) or CSV | May be empty |
| **Location** | Parsed string or CSV; **before** geocode | |
| **Category** | RSS category or CSV | |
| **Raw_Data** | `JSON.stringify` of full RSS item + geocode snippet | Audit trail |
| **Timestamp** | `now` or RSS `pubDate` ISO | |
| **Status** | `New` | Always on create |

---

## Flow A — RSS (scheduled)

1. **Schedule** — Cron `0 6 * * *` equivalent (06:00 daily); adjust per timezone.
2. **RSS** — URL per marketplace (one feed per module, or **Iterator** over a list of feed URLs stored in a small Airtable config table / Data store).
3. **Iterator** — One bundle per item.
4. **Text parser / Replace / Regex** — Extract **price** (e.g. `$1,200`, ranges), **location** (city, region), optional **category** from title/description.
5. **HTTP — Geocoding**  
   - **GET** `https://maps.googleapis.com/maps/api/geocode/json`  
   - Query: `address` = URL-encoded location string; `key` = API key (Make **HTTP** “Query string” or connection).  
   - **Map** `formatted_address` or `address_components` (locality) into a **normalized** string for `Location` field if you want a single clean line; or keep raw + add `formatted_address` into `Raw_Data` JSON.
6. **Airtable — Create a record** — Base **Opportunity Hub**, table **Raw Leads**; map fields above.
7. **Error routing** — **Break** on HTTP 4xx/5xx or empty geocode; **retry** module or **Router** with attempt counter ≤ **3**; on final failure → **Slack**.

---

## Flow B — CSV (webhook or file)

1. **Trigger** — **Webhook** (Custom webhook) *or* **Google Drive** → new file in folder *or* **HTTP** from Zapier.
2. **CSV** — **Parse CSV** (Make) or **Text parser** with line breaks; **Iterator** rows.
3. **Map columns** to same shape as Flow A (use [`sample-data/manual-upload-template.csv`](../sample-data/manual-upload-template.csv)).
4. **HTTP** geocode + **Airtable** create — same as steps 5–6 in Flow A.
5. **Errors** — same Slack + retry policy.

---

## Slack (failure)

Post a short message:

- **Channel** — `#ops` or `#marketplace-alerts` (replace with yours).
- **Text** — Scenario name, module that failed, `error.message`, **first 500 chars** of input bundle (avoid PII if policy requires).

Use a **dedicated Slack webhook** or **Slack app** OAuth in Make.

---

## Retries (3×)

Typical patterns:

- **Break** handler on the **HTTP** or **Airtable** module → **Resume** with delay **30–60 s**; store **attempt** in a **Data store** key `scenario_id + item_id` and stop after **3**.
- Or use Make’s **Auto commit** / **error handler** “**Run another scenario**” only for permanent failures.

Keep **idempotency** in mind: if the same RSS item retries after a partial Airtable write, use **dedupe** (search Raw Leads by `title + timestamp` or link) before create.

---

## Quotas & limits

- **Google Geocoding:** daily quota, billing; cache by normalized address string if you re-run feeds.
- **Airtable:** **5 requests/sec** per base (rate limit); batch or throttle if many items per run.
- **Make:** operations per scenario run — monitor usage.

---

## After build

1. **Export blueprint** from Make → replace [`scenarios/daily-pull.json`](scenarios/daily-pull.json) with the real JSON (or save alongside as `daily-pull.export.json`).
2. Run a dry run with **one** RSS URL and **one** CSV row.
3. Confirm **Raw Leads** rows and `Status = New`.

See [`IMPORT_INSTRUCTIONS.md`](IMPORT_INSTRUCTIONS.md).
