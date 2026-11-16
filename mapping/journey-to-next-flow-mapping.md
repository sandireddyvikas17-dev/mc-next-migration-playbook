# Journey → Next Flow Mapping — Northstar Credit Union

> Fictional brand, design study. "Next equivalent" means the Marketing Cloud Next / Data 360 pattern this Engagement asset would be rebuilt as — described from the Engagement + Data Cloud experience behind this study, with the honest limit stated in the README: this is a design, not a production migration record.

## Summary table

| ID | Engagement asset | Entry today | Decision logic today | Sends today | Next equivalent | Verdict |
|---|---|---|---|---|---|---|
| J-01 | New Member Onboarding | Daily DE entry after core extract (A-01) | Product-held + age-band splits; waits over 21 days; re-entry off | 4 emails (E-01–E-04) | Segment-triggered Flow: Data Cloud segment "New Members — last 1 day" on the unified profile; Flow branches on product / attributes held in Data 360; same wait cadence | **Moves** |
| J-02 | Card Activation & First Use | API event (card issued) + batch DE for reissues | Activated / not split; exit on activation event | 2 emails (E-05, E-06) | Event-triggered Flow: card-issued event received as a Data Action / event trigger on Data 360; activation event exits the Flow | **Moves** |
| J-03 | Loan Application Nudge | Hourly DE refresh (A-02) | Status checks at send time; heavy AMPscript rate/product lookup (CB-03) | 1–2 emails (E-07) | No rebuild yet. Simplify first: move the lookup data onto the audience record so no send-time multi-DE lookup is needed; then re-judge as an event-triggered Flow on application status | **Stays** |
| J-04 | Savings Goal & CD Maturity | Weekly DE entry (A-03) | Date-based waits; product split (CD vs savings goal) | 2 emails (E-08, E-09) | Segment-triggered Flow: Data Cloud segments on maturity-date window + goal-stalled calculated insight; Flow waits anchored to the maturity date field | **Moves** |
| J-05 | Dormant Member Re-engagement | Weekly DE entry (A-03) | Engagement split (any login / none); goal = login | 2 emails (E-10 + reminder) | Segment-triggered Flow: Data Cloud segment "Dormant 90+" using Q-03's dormancy definition verbatim; engagement outcome read back from Data 360 activity | **Moves** |
| J-06 | Old Win-Back (2024) | Weekly DE entry, same DE as J-05 | None beyond entry | 1 email (E-11, expired offer) | Nothing. J-05 is the surviving re-engagement pattern | **Retires** |
| J-07 | Service & Transactional Notices | API / triggered sends from core events | Send classification rules; no marketing consent logic | Service templates (E-12) | No Next equivalent sought in this phase; Engagement transactional patterns and classifications stay as they are | **Stays** |
| A-04 | Welcome Re-send Automation | Daily schedule | None — re-feeds J-01 | — (entry path, no send of its own) | Nothing. Fix J-01's entry reliability instead of carrying a second feed | **Retires** |
| A-05 / DE-05 | Global Suppression Refresh / `DE_Suppression_Global` | Daily schedule | SQL Q-05 builds every hard exclusion | — (suppression, no send) | Rebuilt as consent / suppression on the Data 360 unified profile — designed in Day 2, and a **gate**: no Moves journey cuts over before this exists | **Moves (as a design, Day 2 — gate for all other Moves)** |
| CP-01 | Preference Centre (CloudPages) | Member click from any email footer | Preference writes to `DE_Preferences`; global opt-out | — (page, no send) | Stays on CloudPages until Day 2's consent sync design defines how Engagement preferences and Data 360 consent stay in step both ways; only then can a Next-side preference experience be judged | **Stays** |

## Per-journey reasoning

### J-01 — New Member Onboarding → Moves

- **Why it can move:** Entry is a clean daily segment ("became a member yesterday"), its splits are attributes Data 360 can hold (products held, age band), and its AMPscript is salutation-level. Nothing in the send logic needs Engagement-only machinery.
- **What must be true first:** A-05's suppression rebuilt on the unified profile (Day 2); the staff-account exclusion from Q-01 expressed in the segment; A-04 retired so the new Flow has exactly one entry path.
- **What changes for the member:** Nothing visible — same four messages, same cadence. That is the correct ambition for a first migration wave.

### J-02 — Card Activation & First Use → Moves

- **Why it can move:** It is already event-shaped. The card platform emits issued / activated events today (INT-02); a Next Flow triggered by the issued event, exited by the activated event, is the same design with a new trigger surface.
- **Watch-out:** Reissue batch entry (the DE path) must be designed as a segment entry into the same Flow, or reissues silently lose their nudge. The mapping keeps both entry shapes explicit for that reason.

### J-03 — Loan Application Nudge → Stays (with a named exit condition)

- **Why it stays:** The journey canvas is simple; the complexity is CB-03 — AMPscript lookups across three DEs at send time to resolve rate and product. Rebuilding the canvas in a Flow while the data still needs send-time lookup theatre would copy the problem onto a new platform.
- **Exit condition:** When application status + the two lookup values land on the member / application record itself (a Data 360 data-model task), re-judge. The likely verdict then is Moves, as an event-triggered Flow on application-status change — which also fixes the "hourly batch pretending to be real-time" smell in A-02.

### J-04 — Savings Goal & CD Maturity → Moves

- **Why it can move:** Pure date-and-segment logic. Its one real requirement is that maturity date arrives in Data 360 as a usable date field rather than a weekly snapshot DE — a Day 2 ingestion item, named here so it cannot be discovered at cutover.
- **Watch-out:** Goal-stalled needs a calculated-insight-style definition (no goal contribution in N days). Day 2 defines it next to the dormancy definition so both come from one place.

### J-05 — Dormant Member Re-engagement → Moves

- **Why it can move:** Segment + two sends + a measurable goal (login). It is also the natural parallel-run candidate for Day 3: run the Engagement journey and the Next Flow on split audiences and compare counts and outcomes.
- **Non-negotiable:** The segment uses SQL Q-03's dormancy definition word-for-word. A "close enough" redefinition makes every Day 3 parity check meaningless.

### J-06 — Old Win-Back → Retires

- Same audience as J-05, expired offer content (E-11), no clear owner, untouched. Nothing replaces it because J-05 already does its job. Retiring it also halves the confusion in every dormancy count from now on.

### J-07 — Service & Transactional Notices → Stays

- Service notices run on a different consent and delivery basis from marketing. Forcing them into a marketing-platform migration in phase one adds compliance risk and no member benefit. Stays means: keep the patterns, classifications, and templates in Engagement, and revisit only if a later phase has a specific reason.

### CP-01 — Preference Centre → Stays (gated by Day 2)

- The preference centre is where member consent is *captured*; everything else is where it is *enforced*. Moving the capture surface before the Day 2 sync design exists would risk preferences written in one platform and honoured in neither. It stays on CloudPages, wired into the Day 2 design as a consent source.

## Wave order implied by this mapping

1. **Gate first:** A-05 / DE-05 suppression + consent on the unified profile (Day 2 design).
2. **Wave 1 (Moves, lowest risk):** J-02 (event-shaped already), then J-01 (with A-04 retired).
3. **Wave 2 (Moves, data-dependent):** J-04 and J-05, once Day 2 segments and definitions are in place — J-05 doubles as the parallel-run proof for Day 3.
4. **Stays, revisited only on their exit conditions:** J-03, J-07, CP-01.
5. **Retires, immediately:** J-06 and A-04 — no build required, benefit starts now.
