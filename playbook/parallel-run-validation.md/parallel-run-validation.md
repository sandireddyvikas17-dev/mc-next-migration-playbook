# Parallel-Run Validation — Run Both, Compare Everything

> Fictional brand, design study. *Northstar Credit Union* is invented. No production metrics are claimed anywhere in this file: the checks below are a comparison method, not results. Not employer work.

## The two stages of every parallel run

**Stage A — Shadow.** The Data 360 segment builds and the Next Flow's entry logic runs, but nothing sends from Next. Engagement sends as normal. The comparison is audience-only: who would Next have sent to, versus who Engagement did send to?

**Stage B — Split live.** The eligible audience is split into two distinct cohorts (by member-ID hash, by join-date cohort for J-01, or by sub-wave for J-04 — the split key is named per wave in the cutover plan). Engagement sends to one cohort, Next to the other. No member is in both cohorts, ever — a duplicate send is a failed run, not a rounding error. The comparison is now audience + send behaviour + results, per cohort.

A wave does not advance from Stage A to Stage B, or from Stage B to cutover, on a single clean cycle. The cutover plan names the minimum cycles per wave.

## Check 1 — Audience counts, compared honestly

For each refresh cycle in the run, record side by side:

| Measure | Engagement | Data 360 / Next | Notes |
|---|---|---|---|
| Gross audience (before consent / suppression) | DE count for the journey's entry DE | Segment count before SEG-00 exclusion | A difference here is definition, data, or identity — see the parity order below |
| Suppressed / excluded | Count excluded by DE-05 / Q-05, by reason | Count excluded by SEG-00, by reason | Must match by reason, not just in total — see Check 2 |
| Consent-excluded | Count with no marketing consent in Engagement state | Count failing the channel-consent gate on the unified profile | A Next-side count that is smaller *because consent is finally being honoured* is a finding to report, not a defect to loosen |
| Net sendable | Gross − suppressed − consent-excluded | Same, Next side | This is the number the send actually uses |
| Entered | Journey entry count | Flow entry count (Stage A: would-enter count) | Stage B: per cohort |
| Exited / goal-met | Journey exit count, by reason where available | Flow exit count, by DA-05 reason | Unlogged exits are a failed check — a Flow that cannot show why members left cannot be evaluated |

**The parity order (from Day 2, `segment-definitions.md` §4 — use it in this order, only this order):**

1. **Definition difference** — did the rebuild change the rule? Checked first because it is the cheapest cause to prove. Day 2's verbatim rule exists to make this check pass.
2. **Data completeness** — is the full lookback / source set ingested? (The SEG-04 condition in §2.)
3. **Identity difference** — did identity resolution merge or split profiles differently from Engagement's member-ID keying?
4. **Consent / suppression difference** — is SEG-00 applying exclusions the Engagement send did not, or missing ones it did?

Every count difference gets a named cause from this list, in writing, before the run advances. "Close enough" is not a cause.

## Check 2 — Suppression parity

Suppression parity is checked separately from total counts, because two audiences can match in size and still differ in *who* is suppressed — and the second failure is the dangerous one.

Rules:

- Parity is **by member and by reason** (complaint, hard bounce, staff account, deceased member, legal hold, global opt-out), not by total.
- **Any** member suppressed in one platform and sendable in the other is investigated before the run advances. A member suppressed in Engagement and sendable in Next is a stop (see `rollback-criteria.md`). A member suppressed in Next and sendable in Engagement is a finding: it usually means suppression is finally being honoured, and it is reported — never "fixed" by loosening SEG-00.
- The check runs on the same refresh cycle on both sides. Comparing Monday's DE-05 with Tuesday's SEG-00 proves nothing.

### Suppression parity SQL — Engagement side (Automation Studio)

These queries run in Engagement against Northstar's existing structures. They are design-study examples written for Northstar's named DEs; table and field names follow Day 1's inventory.

**Q-P1 — Suppression counts by reason, Engagement (DE-05) vs the DE-05 successor feed from Data 360:**

```sql
-- Run against DE_Suppression_Global (DE-05) and the successor feed DE
-- refreshed from Data 360 suppression state. Counts must match by reason.
SELECT
    s.Reason,
    COUNT(DISTINCT s.MemberID) AS EngagementCount,
    COUNT(DISTINCT f.MemberID) AS Data360FeedCount
FROM DE_Suppression_Global s
FULL OUTER JOIN DE_Suppression_FromData360 f
    ON f.MemberID = s.MemberID
   AND f.Reason    = s.Reason
GROUP BY s.Reason
-- Any row where the two counts differ is investigated by member (Q-P2),
-- not explained away in aggregate.
```

**Q-P2 — Members suppressed in Engagement but missing from the Data 360 feed (the stop condition):**

```sql
-- Members in DE-05 with no matching suppression state in the feed.
-- Expected result before any Moves cutover: zero rows.
-- Any row here stops the wave until the member's suppression state
-- exists on the unified profile.
SELECT
    s.MemberID,
    s.Reason,
    s.SuppressedDate
FROM DE_Suppression_Global s
LEFT JOIN DE_Suppression_FromData360 f
    ON f.MemberID = s.MemberID
   AND f.Reason    = s.Reason
WHERE f.MemberID IS NULL
```

**Q-P3 — Net sendable audience for a Moves journey, Engagement definition:**

```sql
-- Example: J-05 dormant audience (DE_Dormant_90 / DE-04), net of DE-05.
-- The Data 360 side of this comparison is SEG-04 net of SEG-00,
-- built on the same refresh cycle. Definitions: Q-03 verbatim.
SELECT COUNT(DISTINCT d.MemberID) AS NetSendable_Engagement
FROM DE_Dormant_90 d
WHERE NOT EXISTS (
    SELECT 1
    FROM DE_Suppression_Global s
    WHERE s.MemberID = d.MemberID
)
-- Consent is checked as a separate count (Check 1), not folded in here:
-- folding it in hides whether a difference is suppression or consent.
```

**Q-P4 — Duplicate-entry guard (the A-04 / J-01 failure mode):**

```sql
-- Members appearing in a journey's entry DE from more than one entry path
-- in the same cycle. Expected: zero rows after A-04 is retired.
SELECT
    MemberID,
    COUNT(DISTINCT EntrySource) AS EntryPaths
FROM DE_NewMembers_Daily
GROUP BY MemberID
HAVING COUNT(DISTINCT EntrySource) > 1
```

## Check 3 — Consent-sync spot checks (test members)

Using the invented test-member set from Wave 0, in each parallel run:

1. Change consent in one surface (CP-01 opt-out, Engagement unsubscribe, Data 360-side change — rotate the starting surface per wave).
2. Verify the new state at all three gates (segment build, activation / Data Action, send) in both platforms.
3. Verify an opt-out is never overwritten by an older opt-in arriving late (Day 2 §4: writes apply in action-time order).
4. Verify DA-06 behaviour: a suppression change to a more restrictive value holds the member from marketing Flows; verify there is no automatic re-admission when a flag clears.

## Check 4 — Send behaviour (Stage B only)

- **No duplicates:** join the two cohorts' send records on member ID for the cycle; expected overlap is zero.
- **Exit behaviour:** a member who meets the goal (activates a card, logs in, contributes to a goal) exits on both platforms' logic — and, in the split run, a goal event in one cohort never exits a member in the other.
- **Personalisation spot check:** a small sample of Next sends rendered for test members, checked against the Engagement equivalent for the same member profile. Salutation-level AMPscript rebuilds are expected to match; anything in the CB-03 class is out of scope here because J-03 Stays.
- **Bounce / complaint handling:** hard bounces and complaints from Next sends write suppression state to the unified profile and appear in the DE-05 successor feed for Engagement — verified with a test address, not waited for from real sends.

## Sign-off record (per wave, per stage)

For each stage, record: wave and journey ID, refresh cycles compared, count table (Check 1), suppression result by reason (Check 2), every difference with its named parity cause, consent spot-check result (Check 3), Stage B send checks where applicable (Check 4), and the named person signing off. No sign-off record, no cutover.
