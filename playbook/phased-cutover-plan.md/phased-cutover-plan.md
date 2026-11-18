# Phased Cutover Plan — Engagement → Next, One Wave at a Time

> Fictional brand, design study. *Northstar Credit Union* is invented. No production Marketing Cloud Next experience is claimed. Not employer work.

## How to read this plan

Each wave has: **scope** (what moves), **entry criteria** (all must be true before the wave starts), **parallel run** (pointer to `parallel-run-validation.md`), **cutover steps**, and **exit criteria** (all must be true before the next wave starts). If an entry criterion is not met, the wave does not start — the plan slips, the criterion does not.

Asset IDs are Day 1's: J-01 onboarding, J-02 card activation, J-03 application nudge (Stays), J-04 savings / CD maturity, J-05 dormant re-engagement, J-06 duplicate win-back (Retires), J-07 service notices (Stays), CP-01 preference centre (Stays as capture surface), A-04 duplicate welcome path (Retires), DE-05 / SEG-00 global suppression.

---

## Wave 0 — Foundation. No journey moves.

**Scope**

- Consent sync live per `data-360/consent-preference-sync-design.md`: Data 360 is the consent system of record, CP-01 writes into it, Engagement consumes the DE-05 successor feed.
- Suppression gate built: SEG-00 on the unified profile, enforced at segment build, activation / Data Action, and send.
- Shared sending domains confirmed per Day 2 §6 — same authenticated domains on both platforms, classifications kept distinct, no authentication changes.
- MCI reporting baseline running per `reporting-continuity-mci.md`.
- Retires executed: J-06 switched off (replaced by J-05), A-04 switched off (J-01 keeps its single DE entry path). Both are paused first, observed for one cycle, then deactivated — never deleted on day one, so the decision stays reversible inside Wave 0.

**Entry criteria**

- Day 2 artefacts signed off by the asset owners named in the Day 1 inventory.
- Invented test-member set created in both surfaces for consent-sync testing (Day 2 §7).

**Exit criteria — the gate for everything after**

- SEG-00 membership reconciles with DE-05 under the parity method in `data-360/segment-definitions.md` §4, with every difference explained by a named cause.
- Consent-sync test pass: a deliberate opt-out in each surface (CP-01, Engagement unsubscribe, Data 360-side change) is honoured at all three gates, in both platforms.
- MCI baseline shows Engagement sends only, with the harmonised dimensions in the continuity spec — so a later change in the numbers can be attributed to the wave, not to the reporting.

## Wave 1 — J-05 Dormant Re-engagement (SEG-04)

**Why first:** Day 2 named SEG-04 the parallel-run proof. Dormancy has the clearest written definition in the org (Q-03, carried verbatim), the cadence is forgiving, and a mistake here does not touch onboarding or card servicing.

**Entry criteria**

- Wave 0 exit criteria met.
- Full activity lookback ingested into Data 360 (the SEG-04 condition in Day 2 §2) — a partial history silently redefines "90 days of no activity," so this is checked, not assumed.
- DA-04 (dormant entry, with pre-send activity re-check) and DA-05 / DA-06 patterns tested with test members.

**Cutover steps**

1. Shadow parallel run (see validation file) — at least two refresh cycles.
2. Split-audience live parallel run — Engagement and Next each send to a distinct half of the eligible audience; no member is in both.
3. On a passed comparison: Next Flow becomes the sole entry for J-05; the Engagement journey is paused, not deleted, and kept intact as the rollback target for one full cycle.
4. After one clean cycle on Next alone, the Engagement journey is deactivated and marked superseded in the inventory.

**Exit criteria**

- Parallel-run comparison passed per `parallel-run-validation.md` (counts explained, suppression parity clean, no duplicate sends).
- No rollback trigger from `rollback-criteria.md` fired during the split run or the first solo cycle.

## Wave 2 — J-02 Card Activation & First Use (SEG-02 + DA-01)

**Why second:** the cleanest event shape in the inventory. The INT-02 card-issued payload is reused as-is (Day 2 legacy flag: Yes), so this wave proves the event / Data Action path, not just the segment path.

**Entry criteria**

- Wave 1 exit criteria met.
- DA-01 tested end-to-end with test card events, including the exit-on-activation behaviour (DA-05): a member who activates leaves the Flow, and the exit is logged with its reason.

**Cutover steps**

1. Shadow run on the batch / reissue path (SEG-02) and event shadowing for DA-01 (events mirrored, Flows not sending).
2. Split live run by entry path cohort — batch path first, then event path — so a problem can be attributed to one path.
3. Next becomes sole entry for both paths; Engagement journey paused as rollback target for one full 14-day activation window.
4. Engagement journey deactivated after one clean window.

**Exit criteria**

- Entry, exit, and dedupe behaviour matches the Data Action specs (one entry path per Flow, no double entry from batch + event).
- Activation-window comparison passed; no member received the reminder after activating.

## Wave 3 — J-01 New Member Onboarding (SEG-01)

**Why third, not first:** J-01 is high-volume and member-facing from day one of membership, and it carried the duplicate-entry risk (A-04). It moves only after A-04 is retired, the suppression gate has held through two waves, and the team has two cutovers of practice.

**Entry criteria**

- Waves 1–2 exit criteria met; A-04 confirmed off since Wave 0.
- SEG-01 staff-account exclusion verified explicitly — it is Q-01's rule, carried unchanged, and it is a suppression-class rule, not a targeting preference.

**Cutover steps**

1. Shadow run across at least two daily refresh cycles (SEG-01 is a 1-day window segment, so a single cycle proves very little).
2. Split live run by join-date cohort (alternate days), not by splitting a single day's new members — a new member's first 21 days stay on one platform end-to-end.
3. Next becomes sole entry; Engagement journey paused as rollback target for one full 21-day onboarding cycle.
4. Engagement journey deactivated after one clean cycle.

**Exit criteria**

- No duplicate welcome sends (the A-04 failure mode) in any cycle.
- Day-cohort counts explained under the parity order; consent gate held at entry and at every send step in the 21-day cadence.

## Wave 4 — J-04 Savings Goal & CD Maturity (SEG-03a / SEG-03b)

**Why last of the Moves:** J-04 depends on the hardest data change in Day 2 — maturity and last-contribution dates as real date fields in Data 360 (DE-02 verdict: transform). Date-anchored Flows fail quietly when the dates are wrong, so this wave waits until the date fields have been observed across earlier waves' refreshes.

**Entry criteria**

- Waves 1–3 exit criteria met.
- SEG-03a / SEG-03b definitions checked against Q-04's 60-day window, carried unchanged; the savings-goal stall definition checked against the definition recorded next to it in Day 2, so the two time-based definitions have not drifted.

**Cutover steps**

1. Shadow run across a full weekly refresh cycle, plus a targeted check of members entering the 60-day maturity window during the run.
2. Split live run: SEG-03a (CD maturity) first, SEG-03b (savings goal stalled) second — they anchor differently and are cut over as two sub-waves.
3. Next becomes sole entry per sub-wave; Engagement journey paused per sub-wave as rollback target until each audience's anchor event has passed.
4. Engagement journey deactivated after both sub-waves complete cleanly.

**Exit criteria**

- No missed or early maturity reminders attributable to date-field errors in the shadow or split runs.
- All four Moves journeys now run in Next; the inventory is updated to show final state.

## What does not move in this playbook

| Asset | Final state in this playbook |
|---|---|
| J-03 Loan Application Nudge | Stays in Engagement. Re-judged only when its send-time lookup is simplified and its data lands on the application record (Day 1 exit condition) |
| J-07 Service / Transactional Notices | Stays in Engagement, on its own basis. Sharing a sending domain never merges service and marketing consent |
| CP-01 Preference Centre | Stays on CloudPages as the capture surface for the whole coexistence period, writing into the Data 360 consent record. A Next-side preference experience is a later-phase judgement, after a full wave has run cleanly on this sync |
| J-06, A-04 | Retired in Wave 0. Not rebuilt anywhere |

## After Wave 4

- Engagement is not "switched off." It still runs J-03, J-07, and CP-01. Decommissioning Engagement is a separate decision, outside this playbook, taken only when the Stays assets each have their own plan.
- The inventory, mapping, and this playbook are updated to as-built state. A playbook that describes the plan nobody followed is worse than no playbook.
