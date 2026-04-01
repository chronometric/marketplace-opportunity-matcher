# Match & Score — Classified pairs → Matches (Make + Airtable)

Pair **Supply** and **Demand** rows that pass **location**, **category**, **price**, and **keyword** rules; write **Matches** with component fields so **Match_Score** is computed in Airtable (see [`airtable/formulas.md`](../airtable/formulas.md)).

**Thresholds:** **>70** = strong match; **>80** + last 24h = **Top Opportunities** view (used by Daily Top 10 scenario).

## Preconditions

- **Classified** rows have **Label**, **Clean_Location** (or lat/lng in **Raw_Data**), **Category_Standard**, **Standardized_Price**, **Keywords**.  
- **Matches** table has **Loc_Match**, **Cat_Match**, **Price_Fit**, **Keyword_Overlap**, **Freshness_Mult**, formula **Match_Score**, formula **Is_Recent** ([`airtable/SETUP.md`](../airtable/SETUP.md)).

## Matching rules (all must pass to create a candidate match)

| Rule | Logic |
|------|--------|
| **Location** | Same city **OR** driving distance ≤ **50 mi** via **Google Distance Matrix API** (origins/destinations from geocoded addresses or `place_id`). |
| **Category** | **Exact** match on **Category_Standard**, **OR** allowed parent→child map (e.g. Electronics ⊃ Phones) via Router or lookup table. |
| **Price** | **P_s** = supply **Standardized_Price**; **P_d** = demand max budget (same field or parsed “under $X”). **Pass** if demand can afford supply within **±20%** (e.g. `P_s <= P_d * 1.2`). Map result to **Price_Fit** on 0–1 (1 = best fit). |
| **Keywords** | Tokenize both sides; count overlapping stems or exact matches; **pass if count ≥ 3** (or fuzzy match score from **Text parser** above threshold). |

**Performance:** Avoid N×N across all rows—pre-filter by **metro** (first ZIP/city token) or **Category_Standard** in Airtable **Search** before Distance Matrix calls.

## Flow (recommended)

1. **Airtable — Search records**: **Classified** where `Label` = `Supply` and not yet fully matched (optional flag) — batch.  
2. **Iterator** (outer): each **Supply** row.  
3. **Airtable — Search records**: **Classified** where `Label` = `Demand` and **Category_Standard** = `{Supply.Category_Standard}` (or compatible via map).  
4. **Iterator** (inner): each candidate **Demand** row.  
5. **HTTP — Distance Matrix**  
   - **GET** `https://maps.googleapis.com/maps/api/distancematrix/json`  
   - `origins` / `destinations` = URL-encoded addresses (or `lat,lng` if stored).  
   - Parse `rows[0].elements[0].distance.value` (meters); **≤ 80467 m** (~50 mi) → **Loc_Match** = 1, else skip pair.  
6. **Numeric — Price fit**  
   - Compute **Price_Fit** 0–1 (e.g. 1 when `P_s <= P_d`, decay if over budget).  
7. **Text parser**  
   - Overlap count between Supply **Keywords** and Demand **Keywords**; if **≥ 3**, set **Keyword_Overlap** scalar 0–1; else **skip** or set 0 and do not create match.  
8. **Freshness_Mult**  
   - From Raw Lead **Timestamp** or classified age: 1 if &lt; 24h, else linear decay (same as [`formulas.md`](../airtable/formulas.md)).  
9. **Airtable — Create a record** — **Matches**:  
   - **Matched_Supply** → supply Classified id  
   - **Matched_Demand** → demand Classified id  
   - **Loc_Match**, **Cat_Match** (1 if category rule passed), **Price_Fit**, **Keyword_Overlap**, **Freshness_Mult**  
   - **Notes** optional  
   - **Match_Score** is a **formula field** in Airtable—do not overwrite; ensure numeric inputs are set.

**Alternative:** Compute **Match_Score** entirely in Make and write to a **Number** field if you avoid Airtable formulas—then sync strategy with [`airtable/schema.md`](../airtable/schema.md).

## Deduping

- Before create, **Search** Matches for existing pair `(Matched_Supply, Matched_Demand)`; **update** if re-run with better score.

## Quotas

- **Distance Matrix** costs per element; minimize calls via geographic bucketing.

## Views (Airtable)

- **Strong matches:** `Match_Score` > **70**.  
- **Top Opportunities:** `Match_Score` > **80** AND last 24h (`Is_Recent`).

## After build

Export blueprint → [`scenarios/match-and-score.json`](scenarios/match-and-score.json). See [`IMPORT_INSTRUCTIONS.md`](IMPORT_INSTRUCTIONS.md).
