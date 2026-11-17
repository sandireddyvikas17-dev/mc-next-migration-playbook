# Data Action Trigger Specs — Data 360 → Next Flows

> Fictional brand, design study. A Data Action here means the Data 360-side trigger that starts or exits a Marketing Cloud Next Flow when profile data, segment membership, or an ingested event changes. Payloads are design specs with fictional values only — no real member data, no real endpoints, no credentials anywhere in this file.

## How to read a spec

Every spec states: **trigger** (what fires it), **Flow** (which Day 1 Moves journey it serves), **payload** (the minimum data the Flow receives — minimum on purpose), **guardrails** (the checks that run before the Flow may send anything), and **failure handling** (what happens when the trigger, the consent check, or the Flow fails). Guardrails are part of the spec, not an appendix: a trigger that can fire without its consent check is a defect in this design.

**Universal guardrails — apply to every spec below:**

1. **Consent + suppression re-check at trigger time.** Segment membership was true when the segment refreshed; consent is checked again when the Data Action fires, against current unified-profile state. Suppressed or opted-out → the Flow is not entered, and the hold-out is recorded with its reason.
2. **Idempotency.** Every trigger carries a dedupe key (named per spec). The same event delivered twice enters a member once.
3. **One entry path.** Day 1 retired A-04 for feeding J-01 a second way. No Data Action in this file creates a second door into a Flow that already has one; batch and event entries into the same Flow are specified as one entry with two named sources, as in DA-01 / SEG-02.
4. **Sync-health dependency.** If the consent sync (see `consent-preference-sync-design.md` §4) is degraded, marketing Data Actions pause rather than fire on stale consent. Service messaging (J-07) is unaffected — it never used this path.

## DA-01 — Card Issued → Card Activation Flow (serves J-02, Moves)

- **Trigger:** Card-issued event ingested into Data 360 from the card platform — the existing INT-02 payload shape, reused as-is (the one legacy artefact Day 2 marks fully compatible).
- **Payload (minimum):** member ID (unified-profile key), card-issued timestamp, card type, last-4 reference token. Never a full card number — Day 1's content rule (last-4 only) applies to data, not just emails.
- **Batch companion:** Reissues that arrive as a batch enter via SEG-02 membership change (DA-04 pattern) into the *same* Flow, with the source recorded on entry so reporting can separate event entries from batch entries.
- **Exit trigger:** Card-activated event for the same member exits the Flow immediately (DA-05 pattern).
- **Dedupe key:** member ID + card-issued timestamp.
- **Failure handling:** Event ingestion failure → the batch segment path still catches the member on next refresh (the two sources are each other's safety net, by design). Duplicate event → dedupe key blocks re-entry.

## DA-02 — New Member Segment Entry → Onboarding Flow (serves J-01, Moves)

- **Trigger:** Unified profile enters SEG-01 (New Members, Last 1 Day) on the daily segment refresh.
- **Payload:** member ID, join date, products held, age band — the attributes the Flow's decision splits read. No extract file, no DE row: the Flow reads profile attributes, which is the point of moving.
- **Guardrails:** Universal guardrails, plus the Q-01 staff-account exclusion is already inside SEG-01's definition — a staff profile never reaches this trigger.
- **Dedupe key:** member ID + join date. A member joins once; re-entry is off, matching the Engagement journey.
- **Failure handling:** A missed daily refresh delays entry by one cycle and is visible as a segment-count anomaly in Day 3's parity check — it does not silently double-enter the next day, because the dedupe key is the join date, not the refresh date.

## DA-03 — Maturity / Goal-Stalled Segment Entry → Savings & CD Flow (serves J-04, Moves)

- **Trigger:** Unified profile enters SEG-03a (CD Maturity Window) or SEG-03b (Savings Goal Stalled). Two triggers, one Flow, entry source recorded — the Flow's waits anchor to the maturity date field (03a) or the stall date (03b), which is why the dates had to become real date fields (`segment-definitions.md` §2, DE-02).
- **Payload:** member ID, entry source (`maturity` / `goal-stalled`), maturity date or last-contribution date, product type.
- **Dedupe key:** member ID + product instance + window entry date. A renewed CD is a new product instance and may legitimately re-enter; the same CD in the same window may not.
- **Failure handling:** Missing or malformed date field → no trigger, and the profile is flagged as a data-quality hold-out with reason `missing-date-field`. A Flow that guesses a maturity date is worse than a Flow that does not fire.

## DA-04 — Dormant Segment Entry → Re-engagement Flow (serves J-05, Moves)

- **Trigger:** Unified profile enters SEG-04 (Dormant 90+, Q-03's definition verbatim).
- **Payload:** member ID, days since last activity, last activity type (login / transaction), last activity date.
- **Guardrails:** Universal guardrails, plus a pre-send activity re-check inside the Flow: a member who became active between segment refresh and send exits before the send, not after it. Dormancy is the segment most likely to be overtaken by events, so it gets the extra check.
- **Exit trigger:** Any login or transaction (the Flow's goal event) — DA-05 pattern.
- **Dedupe key:** member ID + dormancy episode start date. A member who reactivates and later goes dormant again starts a new episode and may re-enter; a member still in the same episode may not.
- **Failure handling:** This is Day 3's parallel-run Flow, so its trigger additionally logs entry counts per refresh in the form Day 3's parity check compares against the Engagement journey.

## DA-05 — Exit Events (pattern, used by DA-01 and DA-04)

- **Trigger:** A goal event lands on the unified profile (card activated; any login / transaction).
- **Action:** Exit the member from the named Flow, record exit reason `goal-achieved`. Exits are triggers too and get the same logging as entries — a Flow that cannot show why members left it cannot be evaluated.
- **Dedupe / ordering:** Exit events are applied in event-time order. A late-arriving exit for an already-exited member is a no-op, logged as such.

## DA-06 — Suppression State Change → Immediate Hold (pattern, all Flows)

- **Trigger:** Suppression state or channel consent on the unified profile changes to a more restrictive value (opt-out, complaint, hard bounce, legal hold, deceased flag) — from any source: CP-01 sync, Engagement unsubscribe handling, or an operations feed.
- **Action:** The member is held from all marketing Flows at the next send step at the latest, and removed from activatable segment audiences. This trigger is the enforcement half of "most restrictive wins" in the consent design.
- **The reverse deliberately does not exist:** there is no Data Action that re-admits a member when a flag clears. Clearing a suppression is an owning-process decision (`consent-preference-sync-design.md` §4), after which normal segment membership applies on the next refresh. Automation may restrict instantly; it may not un-restrict on its own.

## What is intentionally not a Data Action

- **J-03 (Loan Application Nudge)** has no spec here. It Stays in Engagement until its send-time lookup is simplified and its data lands on the application record; writing a trigger spec for it now would design around the exact problem Day 1 named.
- **J-07 (Service & Transactional Notices)** has no spec here. Service messaging does not enter through marketing triggers and does not depend on marketing consent state.
- **J-06 / A-04** have no specs. Retired means retired.
