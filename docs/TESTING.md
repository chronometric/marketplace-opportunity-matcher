# Testing, pilot, and launch

Use this after scenarios are wired end-to-end. Replace **reference** numbers in the main [`README.md`](../README.md) with **measured** results before portfolio or stakeholder sign-off.

## 1. Labeled evaluation set

1. Build a **CSV or Airtable view** of at least **~500–2,500** rows (synthetic, anonymized, or public-sample listings).  
2. **Human labels:**  
   - **Classification:** expected `Supply` / `Demand` / `Pending`.  
   - **Matching (optional):** expected pairs or “no match” for subsamples.  
3. Run **Clean & Classify** on the set; compare **Label** to human label.

**Metrics:**

- **Auto-classify accuracy** = correct Supply or Demand / (Supply + Demand + Pending) rows, or strict accuracy including Pending as a class.  
- Target reference: **~95%** auto-route to Supply/Demand vs Pending (tune Router in [`make/CLEAN_CLASSIFY.md`](../make/CLEAN_CLASSIFY.md)).

4. Run **Match & Score** on labeled pairs where applicable; compare **Match_Score** ranking or binary pass/fail to labels.

**Metrics:**

- **Match accuracy** = correct or acceptable pairs vs ground truth (define “acceptable” in your rubric).  
- Reference used in docs: **~98%** on test runs — **only claim this if your labeled set supports it.**

Document: date, sample size, version of scenarios, and brief methodology in this file or an appendix row below.

## 2. Fourteen-day stability pilot

- Run **production schedules** (ingest, classify, match, Daily Top 10) for **14 consecutive days** with real or staging feeds.  
- **Log:** Make scenario failures, Slack alerts, Airtable errors, Google API quota errors.  
- **Success criteria (example):** no **unplanned** full-day outage; all critical alerts acknowledged within SLA.

Update README “Impact & delivery” table only with **observed** uptime narrative.

## 3. Cost check

- **Make** — operations per month vs plan limit.  
- **Airtable** — record count vs plan; API usage.  
- **Google Maps Platform** — Geocoding + Distance Matrix spend (billing account).  
- **Zapier** — tasks if used.

Reference **~$15/mo** (Make + Airtable Pro–class) is indicative — **replace** with your actual monthly bill before quoting externally.

## 4. README alignment

After measurements:

1. Edit [`README.md`](../README.md) **Impact & delivery** table (test volume, accuracy, uptime, cost, timeline).  
2. Ensure **At a glance** and **Testing & reliability** do not contradict `docs/TESTING.md`.

## Sign-off checklist

- [ ] Labeled set evaluated; classification and match metrics recorded  
- [ ] 14-day pilot completed or consciously skipped (document reason)  
- [ ] Cost documented  
- [ ] README numbers updated to measured values  
