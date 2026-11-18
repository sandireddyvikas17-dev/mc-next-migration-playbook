# Reporting Continuity — Marketing Cloud Intelligence (MCI)

> Fictional brand, design study. *Northstar Credit Union* is invented. No reporting results or metrics are claimed in this file — it is a continuity spec: how reporting is set up so the numbers stay comparable. Not employer work.
>
> Builder background note: Marketing Cloud Intelligence (formerly Datorama) is part of the builder's hands-on experience since 2023, at the employers named in the repo README framing. This spec applies that reporting experience to the migration design; it does not claim an MCI implementation was run for Northstar, which is fictional.

## The problem this spec solves

During coexistence, the same lifecycle programme sends from two platforms. If reporting treats "Engagement sends" and "Next sends" as two unrelated programmes, then at every cutover the trend line breaks: last month's onboarding performance and this month's cannot be compared, a wave cannot be judged, and `rollback-criteria.md` trigger P-5 (a wave whose results cannot be read) fires by design rather than by accident.

Continuity means: **one programme view, two platform sources, for the whole migration — and the same view after it.**

## Spec

### 1. One workspace view per programme, not per platform

- MCI continues to report Northstar's lifecycle programme as one programme per journey (J-01, J-02, J-04, J-05), with platform as a dimension — never as the report itself.
- Every dashboard and recurring report that exists for Engagement today is extended to include the Next source before Wave 1, during the Wave 0 baseline. No new "Next dashboard" is created that only the migration team reads.

### 2. Harmonised dimensions (agreed in Wave 0, frozen during waves)

| Dimension | Rule |
|---|---|
| Journey / Flow | One value per Day 1 asset ID (J-01…J-05), used by both platforms' data. A Next Flow inherits its Engagement journey's ID — reporting follows the member programme, not the tool |
| Campaign / message | Each message keeps one stable name across platforms (the Engagement content name, E-01…E-10 where applicable), with platform as a separate field. Renaming a message at cutover is a reporting break disguised as tidiness |
| Platform | `Engagement` / `Next` — a filter and a comparison dimension, never a reason to split a trend |
| Channel | Email / SMS kept separate, matching the per-channel consent model in Day 2. Service (J-07) traffic is classified separately and excluded from marketing programme views, on both platforms |
| Audience cohort | In split runs, the cohort key from the cutover plan (member-ID hash, join-date cohort, or sub-wave) is carried into reporting, so Stage B comparisons in MCI match the validation file's cohorts exactly |
| Date | Send date, in one timezone convention, for both sources. A wave never changes the reporting timezone |

Dimension definitions are frozen once Wave 1 starts. A dimension change mid-migration is treated like an authentication change inside a wave (Day 2 §6): out of scope, because it makes every comparison in the wave uninterpretable.

### 3. Data sources and classification

- Engagement send, open, click, bounce, and unsubscribe data continues to flow into MCI exactly as it does today — no existing source is reconfigured inside a wave.
- Next / Data 360 send and engagement data is added as a second source mapped to the same harmonised dimensions above, with send classifications kept distinct from service classifications even though the sending domains are shared (Day 2 §6).
- Bounce and complaint events from Next sends must also be visible as suppression inputs (validation Check 4): reporting that shows a bounce but never feeds suppression state is a gap, and it is checked in Wave 0, not discovered in Wave 2.

### 4. Baseline and comparison windows

- **Baseline (Wave 0):** at least one full cycle of each Moves journey reported from Engagement alone, through the harmonised view, before any Next send exists. This is the comparison every wave is judged against.
- **Per wave:** reporting compares (a) the shadow-stage audience counts (validation Check 1), (b) the split-run cohorts side by side on the same dates, and (c) the first solo Next cycle against the baseline, with platform as the only intended difference.
- **No invented targets:** this spec deliberately sets no performance thresholds or expected rates. Northstar is fictional and no results exist. In a real programme the owners would set wave tolerances from their own baseline; inventing numbers here would be inventing metrics.

### 5. What continuity must survive

- A rollback. If a wave rolls back, reporting still shows one continuous journey view — the rollback appears as platform dimension returning to Engagement, not as a missing period.
- The end of the playbook. After Wave 4, the same views continue with Next as the source for the four Moves journeys and Engagement as the source for Stays. There is no "final migration report" that replaces the programme reporting; the programme reporting was the migration reporting all along.
- Asset retirement. J-06 and A-04 remain visible in historical reporting as retired, with their end date noted. History is not rewritten to make the new structure look like it was always the structure.

### 6. Ownership and checks

- The reporting owner confirms, at each wave's exit review, that the wave's sends from both platforms appear in the harmonised view with correct journey, message, platform, and cohort values. That confirmation is part of the wave exit criteria in the cutover plan.
- Any wave whose sends cannot be reconciled in MCI triggers pause trigger P-5 in `rollback-criteria.md` until reporting is trustworthy again.
