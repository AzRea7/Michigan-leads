# Metro Detroit Lead Gen System

Working lead-gen tracker for a solo low-voltage / structured cabling business in metro Detroit, with an adaptive lead-scoring algorithm that learns from real outcomes.

- `Metro_Detroit_Lead_Gen_System.xlsx` — the pipeline CRM. Tabs: Read Me (full algorithm writeup + sources, incl. how larger companies source/advertise for this work), Channel Performance (the adaptive scoring engine, 10 channels), Dashboard (live formulas), Pipeline (master prospect tracker, 32 seeded leads), Weekly Call Rotation, Events (in-person networking calendar), Competitor Intel (6 named metro Detroit incumbents, sourced).
- `Outreach_Toolkit.docx` — capabilities one-pager, cold-call scripts, follow-up email templates by channel, and an In-Person Event Playbook (what to bring, pitch, follow-up).
- `AsBuilt_Documentation_Template.docx` — TIA-606-structured as-built documentation to fill in and deliver with every completed job (cable schedule, labeling legend, test results, photos, sign-off).
- `WEEKLY_LOG.md` — dated log the weekly routine appends to (created on first run).
- `QUICKLOG_ISSUES.md` — created by the quick-log processor (below) when a dictated log email can't be confidently matched to a Pipeline row.

## The scoring algorithm

`Lead Score = Base Fit × Channel Multiplier`, clipped to 1-10. Base Fit, Channel Multiplier, Channel Confidence, and Lead Score all live in the `Pipeline` sheet; `Channel Multiplier`, `Channel Confidence`, and `Lead Score` are **live Excel formulas**, not values the routine sets.

- **Base Fit (1-10, column D)** — set once per lead, company-level only: `0.35×Recurring Revenue Potential + 0.25×Deal Size + 0.20×Urgency Signal + 0.20×Access Ease` (each sub-score 1-10, judgment call at lead-creation time).
- **Channel Multiplier (column E)** — `INDEX/MATCH` pull from the `Channel Performance` tab's `Multiplier` column for that row's Channel. Range 0.6x-1.4x. Recalculates instantly, no waiting for a routine run.
- **Channel Confidence (column F)** — `INDEX/MATCH` pull from `Channel Performance`'s `Confidence` column: a plain-language label (`No data`, `Low`, `Building`, `High`) based on Evidence Share = Attempts / (Attempts + K) — how much of the Smoothed Win Rate is real data vs. the starting Prior. A high Lead Score on `Low` confidence is still mostly a guess.
- **Channel Performance tab** computes, per channel, from the Pipeline's own `Status` column: `Attempts`, `Won`, `Lost`, `Raw Win Rate`, and a `Smoothed Win Rate` via empirical Bayesian smoothing: `(Prior×K + Won) / (K + Attempts)`, K=5. `Multiplier = 0.6 + 0.8×Smoothed Win Rate`. `Evidence Share = Attempts/(Attempts+K)` drives the Confidence label. Starting `Prior Win Rate` per channel is a documented, sourced assumption (see the tab's Source/Rationale column and the Read Me tab) — not measured data, and it's designed to be overridden by real results within a handful of attempts per channel. 10 channels: Property Mgmt, CRE Broker, MSP, TI Electrical/GC, Platform, Bid Board, Vendor Program, DC Integrator/Staffing, Association/Networking, **Direct Corporate Prospect** (new — a company with a specific, dated, sourced relocation/expansion signal you call directly; prior 0.16, above generic cold-outbound since the need is documented, not guessed).
- **Service Match (column H)** — tags each lead against Austin's actual service list: Fiber, Copper/Structured Cabling, Decommissions, Network Setup, Security Camera & Access Control, Cloud/IT Support. Judgment call, set once per lead (blue), based on what that specific company's environment plausibly needs — distinct from `Likely Service Need` (column I), the free-text one-line reasoning.
- **Evidence (column J)** — the specific, cited reason a lead was added: a job posting, a named project, a prequalification portal, a confirmed event, a news article about a relocation/expansion. This is the answer to "how do you know this company needs my services." Most of the original 22 seed leads are honestly labeled `Category match only — no specific signal found; worth a discovery call to verify current need`, because that's genuinely how they were sourced. A growing set have real citations — see "Concrete-evidence leads" below. Every new lead added from here forward requires a real Evidence citation, not just category match.

This is the actual "learning" mechanism: it's a live spreadsheet formula chain driven by Austin logging real Won/Lost outcomes, not something that only updates when the weekly routine runs.

## Concrete-evidence leads (Direct Corporate Prospect channel)

Found via Crain's Detroit Business "Companies on the Move" coverage — the highest-signal source pattern found so far, and the model for future sourcing:

- **F.H. Paschen Contractors** — nearly tripling its Detroit office size (2026). Strong evidence: dated, specific, physical-office-expansion signal.
- **Gardner White** — relocating HQ from Warren to Bloomfield Hills (old Taubman building). Strong evidence: named household retailer, full HQ move.
- **Eccalon** — $71M / 800-job new Detroit HQ for a Maryland cybersecurity firm. Strong evidence, but likely too large-scale for a solo op to land directly — worth tracking to find out who's running the buildout rather than expecting to win it solo.

All three: the source articles are paywalled on crainsdetroit.com, so only the search-result snippet was verified, not the full article — read the full piece before calling to get exact addresses/timelines.

## Events (in-person networking)

New `Events` tab + live Google Calendar sync. Two found so far:

- **IREM Michigan — 9th Annual Sporting Clays Classic**, Aug 20, 2026, Bald Mountain, Lake Orion, MI. Added to Google Calendar as an all-day placeholder (exact time not published — call 734-655-0268 to confirm before attending).
- **BOMA Metro Detroit — Trade Fair**: confirmed date was May 1, 2026 — **already passed** by the time this was researched (Aug 12, 2026). Not added to calendar as upcoming. Logged as a reminder to call BOMA now for vendor membership + next year's date so it isn't missed again.

The weekly routine now also searches for new in-person events (BOMA, IREM, and similar trade/networking events relevant to property/facility managers) and pushes confirmed dates to Google Calendar directly via the Google_Calendar MCP connector, logging its search method in the Events tab's "How Found" column and in `WEEKLY_LOG.md`.

## Competitor Intel

New `Competitor Intel` tab: 6 real, sourced metro Detroit incumbents (T-Tech Solutions, Wolverine Low Voltage, CTC Technologies, Detroit Field Techs, Metro Detroit Network Installers, US Cabling Pros), each with size/positioning where verifiable, their claimed strength, and Austin's specific wedge against them. Two honest notes from this pass: (1) Wolverine Low Voltage's own site says they "partner with local businesses and individuals looking for work" — worth a call as a potential subcontract channel, not just competition; (2) Metro Detroit Network Installers and US Cabling Pros could not be independently verified as distinct operating companies (no reviews/presence beyond a listing) — treat as unconfirmed until checked further, exactly the same evidence discipline applied to leads.

## Personalized draft emails (Gmail)

For leads with a real Evidence citation AND a real named contact + email, a Gmail **draft** (never auto-sent) gets created referencing that lead's specific situation instead of a generic template. First one: **F.H. Paschen** — confirmed via their own press release (primary source) that they moved into a new riverfront HQ (Talon Center) March 2, 2026, tripling their Detroit office, and are actively hiring PM/Superintendent roles. Named contact: Ken Swartz (VP, Detroit office), kswartz@fhpaschen.com — draft is in Gmail Drafts, needs [Your Name]/[Your Business Name]/[Your Phone] filled in before sending. The weekly routine now does this automatically going forward for `Direct Corporate Prospect` and `TI Electrical/GC` leads with real evidence + a real contact — never for "category match only" leads, since there's nothing specific to personalize. Drafts never change a lead's Status to Contacted — only Austin actually sending it (and logging that) does.

## Quick-log processor (new routine, every 2 hours)

A second scheduled routine (separate from the weekly one) turns a quick voice-dictated note into a Pipeline update, so Austin never has to open the spreadsheet after a call:

1. Right after a call, dictate a note using your phone's native voice-to-text (built into the iOS/Android Mail app) into a new email: subject `LOG: <company name>`, body whatever you said, unedited (e.g. "talked to ken sounds interested wants a quote by friday mark contacted follow up friday").
2. Every 2 hours, the routine searches Gmail for unread `LOG:` emails, fuzzy-matches the company name to a Pipeline row, and updates Status / Last Contact Date / Next Action / Next Action Date / Notes (appending, never overwriting) based on the dictated content — conservatively, defaulting to `Contacted` if the implied status is ambiguous rather than guessing wrong.
3. If it can't confidently match a company name, it leaves the email unread and logs the ambiguity in `QUICKLOG_ISSUES.md` instead of guessing or creating a duplicate row.
4. Marks processed emails read so they aren't reprocessed. Commits and pushes only when it actually found and processed something — no empty commits.

Routine ID: `trig_01XYxurFfX1PXgcKJddTjb6V`, cron every 2 hours, Gmail MCP connector attached.

## Contact data

`Phone` and `Website` (columns L, N) are populated by research when publicly available. `Contact Name` and direct `Email` (columns K, M, yellow) are usually gatekept by property managers/MSPs specifically and rarely public — they're expected to come from Austin after the first call, not from research. The weekly routine still checks each new lead's public "Team"/"Contact" page for a named contact or general email and fills it in when it genuinely finds one, but won't fabricate or guess one.

## How larger companies source these leads (researched, see Read Me tab for full writeup + sources)

- **Trade association membership (BOMA, IREM)** — puts you directly in a room with property/facility managers, your exact Tier 1 buyer, via vendor membership and events.
- **GC prequalification portals** — large GCs (Barton Malow via BuildingConnected) proactively vet and list subs before they need them, rather than waiting for calls.
- **Local SEO / Google Business Profile, case studies/portfolio content, manufacturer installer directories** — documented in the Read Me tab as worth doing but not tracked as Pipeline rows (one-time setup tasks or require completed jobs first).
- **Paid lead-gen agencies** — exist and are used by larger firms, explicitly not recommended at this stage.

## Weekly refresh routine

A scheduled cloud agent runs weekly (Mondays, 6am ET) against this repo, with `Google_Calendar` and `Gmail` MCP connectors attached (routine ID `trig_01R1sVb99nWWSTVnrU86qaRV`). Each run:

1. Reads `README.md` (this file) and the `Read Me` tab for full context.
2. Reads the `Pipeline` sheet: header row 5, data from row 6. Columns: A Company, B Tier, C Channel, D Base Fit, E Channel Multiplier (live — don't touch), F Channel Confidence (live — don't touch), G Lead Score (live — don't touch), H Service Match, I Likely Service Need, J Evidence, K Contact Name, L Phone, M Email, N Website, O Last Contact Date, P Status, Q Next Action, R Next Action Date, S Notes. Also reads the `Events` sheet: header row 5, data from row 6 (Event, Organization, Date, Location, Status, How Found, On Calendar?, Action Needed, Notes).
3. Researches metro Detroit for new, real signals — bid postings, GC prequalification openings, MSP hiring activity, DC vendor windows, and **"companies on the move" relocation/expansion news** (Crain's Detroit Business is the proven pattern — search it specifically) for the Direct Corporate Prospect channel. Also searches for new in-person trade/networking events (BOMA Metro Detroit, IREM Michigan, similar). Never invents a company or event, and **requires a real Evidence citation for every new lead** — a category match alone is not enough to add a row.
4. Appends new Pipeline rows only (never edits existing ones) starting at the first blank row in column A, following the Base-Fit-only rule (never sets Channel Multiplier, Channel Confidence, or Lead Score — copy the formula pattern from an existing row).
5. For any new event found with a real, confirmed date: appends a row to the `Events` tab (including the specific search method used, in "How Found") AND creates a Google Calendar event via the `Google_Calendar` MCP connector on Austin's primary calendar, with the source URL and a "verify before attending" note in the description if any detail (time, cost) wasn't confirmed. Never creates a calendar event for a date that's already in the past.
6. Logs a dated entry to `WEEKLY_LOG.md`: new leads by channel with their Evidence in one line each; new events found and whether they were added to the calendar; Channel Performance's current Attempts/Won/Lost/Multiplier/Confidence per channel.
7. Saves, commits, pushes to `main`. Never touches Dashboard formulas, Channel Performance formulas, or any user-filled Pipeline/Events column.

Uses `openpyxl` to edit the xlsx in place. Only appends real, sourced, evidenced leads and events — if nothing credible turns up in a given week, it logs that instead of padding the list.
