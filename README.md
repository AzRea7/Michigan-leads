# Metro Detroit Lead Gen System

Working lead-gen tracker for a solo low-voltage / structured cabling business in metro Detroit, with an adaptive lead-scoring algorithm that learns from real outcomes.

- `Metro_Detroit_Lead_Gen_System.xlsx` — the pipeline CRM. Tabs: Read Me (full algorithm writeup + sources), Channel Performance (the adaptive scoring engine), Dashboard (live formulas), Pipeline (master prospect tracker), Weekly Call Rotation.
- `Outreach_Toolkit.docx` — capabilities one-pager, cold-call scripts, and follow-up email templates by channel (property manager, MSP, CRE broker).
- `WEEKLY_LOG.md` — dated log the weekly routine appends to (created on first run).

## The scoring algorithm

`Lead Score = Base Fit × Channel Multiplier`, clipped to 1-10. Both factors live in the `Pipeline` sheet; `Channel Multiplier` and `Lead Score` are **live Excel formulas**, not values the routine sets.

- **Base Fit (1-10, column D)** — set once per lead, company-level only: `0.35×Recurring Revenue Potential + 0.25×Deal Size + 0.20×Urgency Signal + 0.20×Access Ease` (each sub-score 1-10, judgment call at lead-creation time).
- **Channel Multiplier (column E)** — `INDEX/MATCH` pull from the `Channel Performance` tab's `Multiplier` column for that row's Channel. Range 0.6x-1.4x. Recalculates instantly, no waiting for a routine run.
- **Channel Performance tab** computes, per channel, from the Pipeline's own `Status` column: `Attempts`, `Won`, `Lost`, `Raw Win Rate`, and a `Smoothed Win Rate` via empirical Bayesian smoothing: `(Prior×K + Won) / (K + Attempts)`, K=5. `Multiplier = 0.6 + 0.8×Smoothed Win Rate`. Starting `Prior Win Rate` per channel is a documented, sourced assumption (see the tab's Source/Rationale column and the Read Me tab) — not measured data, and it's designed to be overridden by real results within a handful of attempts per channel.

This is the actual "learning" mechanism: it's a live spreadsheet formula chain driven by Austin logging real Won/Lost outcomes, not something that only updates when the weekly routine runs.

## Weekly refresh routine

A scheduled cloud agent runs weekly (Mondays, 6am ET) against this repo. Each run:

1. Reads `README.md` (this file) and the `Read Me` tab for full context.
2. Reads the `Pipeline` sheet: header row 5, data from row 6. Columns: A Company, B Tier, C Channel, D Base Fit, E Channel Multiplier (live — don't touch), F Lead Score (live — don't touch), G Likely Service Need, H Contact Name, I Phone, J Email, K Website, L Last Contact Date, M Status, N Next Action, O Next Action Date, P Notes.
3. Researches metro Detroit for new, real signals (new bid postings, leasing/repositioning news, MSP hiring activity, DC vendor windows) — never invents a company.
4. Appends new rows only (never edits existing ones) starting at the first blank row in column A: Company, Tier, Channel (must exactly match one of the 8 values in `Channel Performance` column A / the sheet's dropdown), **Base Fit only** (using the weighted rubric above — never sets Channel Multiplier or Lead Score, those are formulas), Likely Service Need, Website/Phone if publicly findable, Status = `Not Contacted`, Next Action. Leaves Contact Name/Email/Last Contact Date/Next Action Date blank for Austin.
5. Logs a dated entry to `WEEKLY_LOG.md`: how many new leads added by channel, and — by reading the `Channel Performance` tab's current Attempts/Won/Lost/Multiplier per channel — a one-line note on which channels are pulling ahead or falling behind, with the numbers.
6. Saves, commits, pushes to `main`. Never touches Dashboard formulas, Channel Performance formulas, or any user-filled Pipeline column.

Uses `openpyxl` to edit the xlsx in place. Only appends real, sourced leads — if nothing credible turns up in a given week, it logs that instead of padding the list.
