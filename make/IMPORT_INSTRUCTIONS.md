# Make scenario import / export

## Exporting from Make (source of truth)

1. Open your scenario in [Make](https://www.make.com).
2. Use the scenario menu → **Export Blueprint** (wording may vary slightly by UI version).
3. Save the JSON into `make/scenarios/` with a clear name, e.g. `daily-pull.json`.
4. Commit to this repo so the team shares the same version.

The JSON files committed here are **templates / documentation aids**. After you build live scenarios in Make, **replace or supplement** them with real exports so connection IDs and module settings match your organization.

## Importing into Make

1. Make → **Create a new scenario** → **Import Blueprint** (or equivalent).
2. Select the JSON file from `make/scenarios/`.
3. Re-authenticate every module (Airtable, Google Sheets, HTTP, Slack, etc.).
4. Map Airtable base ID, table names, and field names to match [`airtable/schema.md`](../airtable/schema.md).

## Scenario files in this repo

| File | Purpose |
|------|---------|
| `daily-pull.json` | RSS + CSV path → Raw Leads, geocoding hooks |
| `clean-classify.json` | Cleaning Router → Classified records |
| `match-and-score.json` | Pair supply/demand → Matches + score inputs |
| `daily-top-10.json` | Top view → Google Sheet |

If a file is a **flow description** (`_type: "documentation"`) rather than a raw Make export, recreate the scenario in the Make UI and export JSON for production use.
