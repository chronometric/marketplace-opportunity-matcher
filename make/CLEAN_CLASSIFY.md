# Clean & Classify — Raw Leads → Classified (Make)

Read **Raw Leads** where **Status = New**, normalize text and numbers, map categories, extract **keywords**, route **Supply** / **Demand** / **Pending**, write **Classified**, then set Raw Lead **Status = Processed**.

**Target:** ~**95%** auto-classified to Supply or Demand; the rest **Pending** for manual review.

## Trigger

- **Airtable — Watch records** (when `Status` = `New`), or  
- **Schedule** (poll every N minutes) + **Search records** with filter `Status = New` and **limit** batch size (e.g. 50) to stay under rate limits.

## Step 1 — Load

- **Airtable — Search records** (or **Get a record** in a loop): table **Raw Leads**, formula or filter `{Status} = 'New'`.

## Step 2 — Clean text

| Task | Approach |
|------|----------|
| Strip emojis | **Replace** with regex or remove non-ASCII ranges (test on sample titles). |
| Collapse whitespace | **Trim**, replace multiple spaces/newlines. |
| Title + description | Concatenate for keyword extraction if needed. |

## Step 3 — Price normalization

| Input | Output |
|-------|--------|
| `$1,200` | `1200` (number) |
| `$100-200` or `$100–200` | Average **150** (split on `-` / en-dash, parse both, `(a+b)/2`) |
| `Free` / `0` | `0` |
| No price | Empty or `0` (Router may still infer Supply from “for sale” text) |

Store in **Classified → Standardized_Price** (and optionally keep raw on Raw Lead unchanged until next ingest revision).

## Step 4 — Category mapping

Use a **Router** or **Switch** on keywords in **Title** + **Description** (case-insensitive):

| If contains (examples) | Set **Category_Standard** |
|------------------------|----------------------------|
| iphone, ipad, macbook, galaxy | Electronics |
| bike, bicycle | Sports |
| couch, sofa, table | Furniture |
| *(default)* | Use Raw **Category** or `General` |

Maintain a **Data store** or small **Airtable table** “Category Map” if you want non-developers to edit rules without opening the scenario.

## Step 5 — Location

- Copy **Location** from Raw Lead into **Clean_Location** after light cleanup (strip extra commas), or use geocoded `formatted_address` from **Raw_Data** JSON if you stored it at ingest.

## Step 6 — Keywords (for matching)

- **Split** title/description on spaces/punctuation; **lowercase**; remove stopwords (`the`, `a`, `for`, …).  
- **Join** unique tokens (max N, e.g. 20) into **Keywords** (comma or newline separated).

Optional: **Text parser** module for fuzzy tokens (stemming not required for v1).

## Step 7 — Router (Supply / Demand / Pending)

**Order matters:** evaluate **Demand** signals first if “wanted” can coexist with a price.

| Route | Condition (examples) | **Label** |
|-------|------------------------|-----------|
| 1 | Title/description matches `(?i)(wanted\|iso\|buying\|need\|looking\s+for\|wtb)` | **Demand** |
| 2 | Matches `(?i)(selling\|for sale\|obo\|firm)` **OR** `Standardized_Price` > 0 with no strong demand phrase | **Supply** |
| 3 | Default | **Pending** |

Tune regex per marketplace jargon. **Free** listings (`price = 0`) often go **Pending** unless “curb alert” + Supply heuristics are explicit.

## Step 8 — Write Classified

**Airtable — Create a record** — table **Classified**:

| Field | Value |
|-------|--------|
| **Display Title** | From Raw **Title** (truncated if needed) |
| **Raw Lead** | Link to current Raw Lead record id |
| **Label** | `Supply` \| `Demand` \| `Pending` |
| **Keywords** | From step 6 |
| **Standardized_Price** | From step 3 |
| **Clean_Location** | From step 5 |
| **Category_Standard** | From step 4 |

## Step 9 — Mark Raw Lead processed

**Airtable — Update a record** — **Raw Leads**: `Status` = `Processed`.

## Errors

- On Airtable failure: retry with backoff; optional **Slack** (same pattern as [DAILY_PULL.md](DAILY_PULL.md)).
- **Pending** rows: optional **Slack** digest once daily (separate scenario) for human review.

## Measuring ~95% auto-classify

- Export a labeled set of Raw Leads; compare **Label** to human label.  
- Adjust Router order and regex until **Pending** ≈ **5%** (or your SLA).

## After build

Export blueprint → [`scenarios/clean-classify.json`](scenarios/clean-classify.json). See [`IMPORT_INSTRUCTIONS.md`](IMPORT_INSTRUCTIONS.md).
