# Zapier fallback — email CSV → Raw Leads (or Make webhook)

Use when **Make** cannot access a source directly (e.g. CSV arrives **only** by email). **Zapier** ingests the file or attachment and either **creates Airtable rows** or **calls a Make webhook** with the same shape as [`sample-data/manual-upload-template.csv`](../sample-data/manual-upload-template.csv).

**Scope:** optional path; primary ingest remains **Make** ([`make/DAILY_PULL.md`](../make/DAILY_PULL.md)).

## Option A — Zapier → Airtable (no Make)

1. **Trigger:** **Email by Zapier** (inbound email) *or* **Gmail** — **New attachment** (filter `*.csv`) *or* **New email in folder** with label `marketplace-csv`.  
2. **Action:** **Formatter — Text** or **CSV** — parse rows (Zapier’s line-item support varies by plan).  
3. **Action:** **Airtable — Create record** (loop / line items) — map columns to **Raw Leads**: **Source**, **Title**, **Description**, **Price**, **Location**, **Category**, **Timestamp**, **Raw_Data** (JSON of row), **Status** = `New`.  

**Note:** Large CSVs may hit Zap **task** limits; split files or use **Make** webhook for bulk JSON.

## Option B — Zapier → Make webhook (recommended for parity)

1. **Trigger:** Same as Option A (email + CSV attachment).  
2. **Action:** **Webhooks by Zapier — POST** to Make **Custom webhook** URL (same as CSV branch in Daily Pull).  
3. **Body:** JSON array of row objects matching your Make **Parse JSON** → **Iterator** → **Airtable** path, or **multipart** file upload if Make accepts file webhook.

This keeps **one** implementation of geocoding + Raw Leads mapping in Make.

## Option C — Zapier → Google Drive → Make

1. **Gmail** saves attachment to **Google Drive** folder.  
2. **Make** **Google Drive — Watch files** (already documented in [`DAILY_PULL.md`](../make/DAILY_PULL.md)) picks up the file.

## Secrets

- Store **Airtable PAT**, **Make webhook URL**, and email filters in Zapier **Connections**; never commit.

## After configuration

- Document the inbound address or label in your runbook; add a row to [`make/IMPORT_INSTRUCTIONS.md`](../make/IMPORT_INSTRUCTIONS.md) table if you maintain a Zap export JSON (optional `zapier/zap-export.json` placeholder not required).
