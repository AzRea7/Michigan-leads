# Metro Detroit Lead Gen System

Working lead-gen tracker for a solo low-voltage / structured cabling business in metro Detroit.

- `Metro_Detroit_Lead_Gen_System.xlsx` — the pipeline CRM. Tabs: Read Me, Dashboard (live formulas), Pipeline (master prospect tracker with Lead Score + Likely Service Need columns), Weekly Call Rotation.
- `Outreach_Toolkit.docx` — capabilities one-pager, cold-call scripts, and follow-up email templates by channel (property manager, MSP, CRE broker).

## Weekly refresh routine

A scheduled cloud agent runs weekly against this repo. Each run:

1. Reads the `Pipeline` sheet in the xlsx — specifically the `Status` column (Not Contacted / Contacted / Follow-up Scheduled / Quoted / Won / Lost) and existing companies already listed.
2. Researches the metro Detroit market for new signals: new PlanHub/Blue Book postings, office leasing/repositioning news, MSP hiring activity, data-center vendor/staffing registration windows.
3. Appends newly-found, real (not fabricated) companies as new rows, each scored 1-10 (`Lead Score`) with a `Likely Service Need` note, following the same rubric already used for the seed rows: conversion likelihood x deal size x access ease.
4. Adjusts scoring weight for existing rows/channels based on logged outcomes — channels with `Won` status trend upward, channels attempted multiple times and stuck at `Contacted`/`Lost` trend downward. Never touches user-filled columns (Contact Name, Phone, Email, Last Contact Date, Status, Next Action, Next Action Date, Notes) — only Lead Score and Likely Service Need, and only appends new rows rather than editing existing company rows' identity.
5. Commits and pushes with a clear commit message summarizing what changed (how many new leads added, which channels re-weighted and why).

Uses `openpyxl` to edit the xlsx in place — never hardcode over the Dashboard tab's formulas.
