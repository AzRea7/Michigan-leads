# Metro Detroit Lead Gen System

Working lead-gen tracker for a solo low-voltage / structured cabling business in metro Detroit, with an adaptive lead-scoring algorithm that learns from real outcomes.

- `Metro_Detroit_Lead_Gen_System.xlsx` — the pipeline CRM. Tabs: Read Me (full algorithm writeup + sources, incl. how larger companies source/advertise for this work), Channel Performance (the adaptive scoring engine, 9 channels), Dashboard (live formulas), Pipeline (master prospect tracker, 29 seeded leads), Weekly Call Rotation.
- `Outreach_Toolkit.docx` — capabilities one-pager, cold-call scripts, and follow-up email templates by channel (property manager, MSP, CRE broker).
- `WEEKLY_LOG.md` — dated log the weekly routine appends to (created on first run).

## The scoring algorithm

`Lead Score = Base Fit × Channel Multiplier`, clipped to 1-10. Base Fit, Channel Multiplier, Channel Confidence, and Lead Score all live in the `Pipeline` sheet; `Channel Multiplier`, `Channel Confidence`, and `Lead Score` are **live Excel formulas**, not values the routine sets.

- **Base Fit (1-10, column D)** — set once per lead, company-level only: `0.35×Recurring Revenue Potential + 0.25×Deal Size + 0.20×Urgency Signal + 0.20×Access Ease` (each sub-score 1-10, judgment call at lead-creation time).
- **Channel Multiplier (column E)** — `INDEX/MATCH` pull from the `Channel Performance` tab's `Multiplier` column for that row's Channel. Range 0.6x-1.4x. Recalculates instantly, no waiting for a routine run.
- **Channel Confidence (column F)** — `INDEX/MATCH` pull from `Channel Performance`'s `Confidence` column: a plain-language label (`No data`, `Low`, `Building`, `High`) based on Evidence Share = Attempts / (Attempts + K) — how much of the Smoothed Win Rate is real data vs. the starting Prior. A high Lead Score on `Low` confidence is still mostly a guess.
- **Channel Performance tab** computes, per channel, from the Pipeline's own `Status` column: `Attempts`, `Won`, `Lost`, `Raw Win Rate`, and a `Smoothed Win Rate` via empirical Bayesian smoothing: `(Prior×K + Won) / (K + Attempts)`, K=5. `Multiplier = 0.6 + 0.8×Smoothed Win Rate`. `Evidence Share = Attempts/(Attempts+K)` drives the Confidence label. Starting `Prior Win Rate` per channel is a documented, sourced assumption (see the tab's Source/Rationale column and the Read Me tab) — not measured data, and it's designed to be overridden by real results within a handful of attempts per channel. 9 channels: Property Mgmt, CRE Broker, MSP, TI Electrical/GC, Platform, Bid Board, Vendor Program, DC Integrator/Staffing, **Association/Networking** (new — BOMA/IREM trade-association membership, prior 0.22, closer to warm-intro than cold outbound since you're in a room with people who chose to be there).
- **Service Match (column H)** — tags each lead against Austin's actual service list: Fiber, Copper/Structured Cabling, Decommissions, Network Setup, Security Camera & Access Control, Cloud/IT Support. Judgment call, set once per lead (blue), based on what that specific company's environment plausibly needs — distinct from `Likely Service Need` (column I), the free-text one-line reasoning.
- **Evidence (column J, new)** — the specific, cited reason a lead was added: a job posting, a named project, a prequalification portal, a confirmed event. This is the answer to "how do you know this company needs my services" — most of the original 22 seed leads are honestly labeled `Category match only — no specific signal found; worth a discovery call to verify current need`, because that's genuinely how they were sourced (right company type, right region, not a verified need). A handful have real citations (Pacer Staffing's site lists low-voltage roles; Walbridge is the confirmed GC on Saline Barn; Barton Malow publishes a prequalification process; BOMA/IREM are real trade associations with confirmed events). Going forward, every new lead the weekly routine adds must have a real Evidence citation, not just category match.

This is the actual "learning" mechanism: it's a live spreadsheet formula chain driven by Austin logging real Won/Lost outcomes, not something that only updates when the weekly routine runs.

## Contact data

`Phone` and `Website` (columns L, N) are populated by research when publicly available. `Contact Name` and direct `Email` (columns K, M, yellow) are usually gatekept by property managers/MSPs specifically and rarely public — they're expected to come from Austin after the first call, not from research. The weekly routine still checks each new lead's public "Team"/"Contact" page for a named contact or general email and fills it in when it genuinely finds one, but won't fabricate or guess one.

## How larger companies source these leads (researched, see Read Me tab for full writeup + sources)

- **Trade association membership (BOMA, IREM)** — the highest-leverage discovery of this pass. BOMA Metro Detroit and IREM Michigan put you directly in a room with property/facility managers — your exact Tier 1 buyer — via vendor membership and events (BOMA's Trade Fair at Eastern Market; IREM's Sporting Clays Classic, confirmed Aug 20, 2026). Both added as real Pipeline leads.
- **GC prequalification portals** — large GCs (Barton Malow via BuildingConnected) proactively vet and list subs before they need them, rather than waiting for calls. Added Barton Malow and DeMaria as real TI Electrical/GC leads — that channel had zero named companies before this pass; also added J. Simon & Sons, Sheker Construction, and Metro General Contractors, all confirmed to market commercial tenant-improvement work specifically.
- **Local SEO / Google Business Profile, case studies/portfolio content, manufacturer installer directories** — documented in the Read Me tab as worth doing but not tracked as Pipeline rows (they're one-time setup tasks or require completed jobs first, not individual leads).
- **Paid lead-gen agencies** — exist and are used by larger firms, explicitly not recommended at this stage (real ongoing cost, assumes spare capacity Austin doesn't have yet).

## Weekly refresh routine

A scheduled cloud agent runs weekly (Mondays, 6am ET) against this repo. Each run:

1. Reads `README.md` (this file) and the `Read Me` tab for full context.
2. Reads the `Pipeline` sheet: header row 5, data from row 6. Columns: A Company, B Tier, C Channel, D Base Fit, E Channel Multiplier (live — don't touch), F Channel Confidence (live — don't touch), G Lead Score (live — don't touch), H Service Match, I Likely Service Need, J Evidence, K Contact Name, L Phone, M Email, N Website, O Last Contact Date, P Status, Q Next Action, R Next Action Date, S Notes.
3. Researches metro Detroit for new, real signals (new bid postings, leasing/repositioning news, MSP hiring activity, DC vendor windows, GC prequalification openings, association events) — never invents a company, and **requires a real Evidence citation for every new lead** — a category match alone is not enough to add a row anymore.
4. Appends new rows only (never edits existing ones) starting at the first blank row in column A: Company, Tier, Channel (must exactly match one of the 9 values in `Channel Performance` column A / the sheet's dropdown), **Base Fit only** (using the weighted rubric above — never sets Channel Multiplier, Channel Confidence, or Lead Score, those are formulas — copy the same formula pattern from an existing row so new rows stay live), Service Match, Likely Service Need, **Evidence (required, cited)**, Website/Phone/named-Contact/general-Email if publicly findable, Status = `Not Contacted`, Next Action. Leaves Contact Name/Email blank if not genuinely found, and always leaves Last Contact Date/Next Action Date blank for Austin.
5. Logs a dated entry to `WEEKLY_LOG.md`: how many new leads added by channel (with their Evidence in one line each), and — by reading the `Channel Performance` tab's current Attempts/Won/Lost/Multiplier/Confidence per channel — a one-line note on which channels are pulling ahead or falling behind, with the numbers.
6. Saves, commits, pushes to `main`. Never touches Dashboard formulas, Channel Performance formulas, or any user-filled Pipeline column.

Uses `openpyxl` to edit the xlsx in place. Only appends real, sourced, evidenced leads — if nothing credible turns up in a given week, it logs that instead of padding the list.
