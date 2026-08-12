# Metro Detroit Lead Gen System

Working lead-gen tracker for a solo low-voltage / structured cabling business in metro Detroit, with an adaptive lead-scoring algorithm that learns from real outcomes.

- `Metro_Detroit_Lead_Gen_System.xlsx` — the pipeline CRM. Tabs: Read Me (full algorithm writeup + sources), Channel Performance (the adaptive scoring engine), Dashboard (live formulas), Pipeline (master prospect tracker), Weekly Call Rotation.
- `Outreach_Toolkit.docx` — capabilities one-pager, cold-call scripts, and follow-up email templates by channel (property manager, MSP, CRE broker).
- `WEEKLY_LOG.md` — dated log the weekly routine appends to (created on first run).

## The scoring algorithm

`Lead Score = Base Fit × Channel Multiplier`, clipped to 1-10. Base Fit, Channel Multiplier, Channel Confidence, and Lead Score all live in the `Pipeline` sheet; `Channel Multiplier`, `Channel Confidence`, and `Lead Score` are **live Excel formulas**, not values the routine sets.

- **Base Fit (1-10, column D)** — set once per lead, company-level only: `0.35×Recurring Revenue Potential + 0.25×Deal Size + 0.20×Urgency Signal + 0.20×Access Ease` (each sub-score 1-10, judgment call at lead-creation time).
- **Channel Multiplier (column E)** — `INDEX/MATCH` pull from the `Channel Performance` tab's `Multiplier` column for that row's Channel. Range 0.6x-1.4x. Recalculates instantly, no waiting for a routine run.
- **Channel Confidence (column F)** — `INDEX/MATCH` pull from `Channel Performance`'s `Confidence` column: a plain-language label (`No data`, `Low`, `Building`, `High`) based on Evidence Share = Attempts / (Attempts + K) — how much of the Smoothed Win Rate is real data vs. the starting Prior. A high Lead Score on `Low` confidence is still mostly a guess.
- **Channel Performance tab** computes, per channel, from the Pipeline's own `Status` column: `Attempts`, `Won`, `Lost`, `Raw Win Rate`, and a `Smoothed Win Rate` via empirical Bayesian smoothing: `(Prior×K + Won) / (K + Attempts)`, K=5. `Multiplier = 0.6 + 0.8×Smoothed Win Rate`. `Evidence Share = Attempts/(Attempts+K)` drives the Confidence label. Starting `Prior Win Rate` per channel is a documented, sourced assumption (see the tab's Source/Rationale column and the Read Me tab) — not measured data, and it's designed to be overridden by real results within a handful of attempts per channel.
- **Service Match (column H)** — tags each lead against Austin's actual service list: Fiber, Copper/Structured Cabling, Decommissions, Network Setup, Security Camera & Access Control, Cloud/IT Support. Judgment call, set once per lead (blue), based on what that specific company's environment plausibly needs — distinct from `Likely Service Need` (column I), which is the free-text one-line reasoning.

This is the actual "learning" mechanism: it's a live spreadsheet formula chain driven by Austin logging real Won/Lost outcomes, not something that only updates when the weekly routine runs.

## Contact data

`Phone` and `Website` (columns K, M) are populated by research when publicly available. `Contact Name` and direct `Email` (columns J, L, yellow) are usually gatekept by property managers/MSPs specifically and rarely public — they're expected to come from Austin after the first call, not from research. The weekly routine still checks each new lead's public "Team"/"Contact" page for a named contact or general email and fills it in when it genuinely finds one, but won't fabricate or guess one.

## Weekly refresh routine

A scheduled cloud agent runs weekly (Mondays, 6am ET) against this repo. Each run:

1. Reads `README.md` (this file) and the `Read Me` tab for full context.
2. Reads the `Pipeline` sheet: header row 5, data from row 6. Columns: A Company, B Tier, C Channel, D Base Fit, E Channel Multiplier (live — don't touch), F Channel Confidence (live — don't touch), G Lead Score (live — don't touch), H Service Match, I Likely Service Need, J Contact Name, K Phone, L Email, M Website, N Last Contact Date, O Status, P Next Action, Q Next Action Date, R Notes.
3. Researches metro Detroit for new, real signals (new bid postings, leasing/repositioning news, MSP hiring activity, DC vendor windows) — never invents a company.
4. Appends new rows only (never edits existing ones) starting at the first blank row in column A: Company, Tier, Channel (must exactly match one of the 8 values in `Channel Performance` column A / the sheet's dropdown), **Base Fit only** (using the weighted rubric above — never sets Channel Multiplier, Channel Confidence, or Lead Score, those are formulas — copy the same formula pattern from an existing row so new rows stay live), Service Match (from the six service types above), Likely Service Need, Website/Phone/named-Contact/general-Email if publicly findable, Status = `Not Contacted`, Next Action. Leaves Contact Name/Email blank if not genuinely found, and always leaves Last Contact Date/Next Action Date blank for Austin.
5. Logs a dated entry to `WEEKLY_LOG.md`: how many new leads added by channel, and — by reading the `Channel Performance` tab's current Attempts/Won/Lost/Multiplier/Confidence per channel — a one-line note on which channels are pulling ahead or falling behind, with the numbers.
6. Saves, commits, pushes to `main`. Never touches Dashboard formulas, Channel Performance formulas, or any user-filled Pipeline column.

Uses `openpyxl` to edit the xlsx in place. Only appends real, sourced leads — if nothing credible turns up in a given week, it logs that instead of padding the list.
