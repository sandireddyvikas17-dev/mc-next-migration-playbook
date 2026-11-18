# Rollback Criteria — When to Pause, When to Go Back

> Fictional brand, design study. *Northstar Credit Union* is invented. Not employer work.

Rollback is part of the plan, not an admission the plan failed. Each Moves journey's Engagement version is kept intact and paused — not deleted — for a full cycle after cutover (see the cutover plan), precisely so rollback is a re-pointing exercise, not a rebuild.

## Pause vs rollback — two different decisions

- **Pause** stops new entries on the Next side and holds sends for the affected journey while the cause is investigated. Engagement does not automatically resume; a pause is a hold, not a switch.
- **Rollback** ends the wave's cutover: entry is re-pointed to the Engagement journey, the Next Flow is disabled for that journey, and the wave returns to its entry criteria. A rolled-back wave is re-attempted only after the cause is fixed and the parallel run is repeated — never by simply trying the cutover again.

## Immediate rollback triggers (any one is enough)

| # | Trigger | Why it is immediate |
|---|---|---|
| R-1 | Any member suppressed in Engagement (DE-05) is sent to, or entered into a Flow, from Next — Q-P2 in the validation file returns rows and a send or entry occurred | Emailing someone who asked not to be emailed is the failure this whole playbook is built to prevent. No threshold, no "small number" exception |
| R-2 | A global opt-out or channel opt-out recorded in either platform is not honoured at any of the three gates (segment build, activation, send) | Same class as R-1, at the consent layer rather than the suppression layer |
| R-3 | Duplicate sends: the same member receives the same journey's message from both platforms in one cycle | Proves the split / entry control failed; members experience it directly |
| R-4 | Consent sync is degraded or down and marketing sends continued on stale consent state | Day 2's rule is explicit: sends that depend on the sync pause when it degrades. Continuing anyway converts an infrastructure problem into a consent problem |
| R-5 | A Data Action fires entry for a Stays or Retires asset (J-03, J-07, J-06, A-04), or a retired asset is found running | Means entry control, not just one journey, is misconfigured |

## Pause triggers (stop, investigate, then decide)

| # | Trigger | Investigation |
|---|---|---|
| P-1 | Net sendable counts differ between platforms with no named cause after the parity order (definition → data completeness → identity → consent) has been worked through | Work the parity order again with the data owners; an unexplained difference is not waived |
| P-2 | Suppression counts match in total but differ by member or by reason | By-member comparison (validation Check 2). Totals matching is not parity |
| P-3 | Exit behaviour diverges: goal events are not exiting members, or exits are not logged with reasons | Check DA-05 handling and event ordering before any further sends in that journey |
| P-4 | Deliverability signals for Next sends diverge sharply from the Engagement cohort in the same split run, with no content or audience explanation | Confirm shared-domain configuration (Day 2 §6) was not changed inside the wave; a wave changes the platform and nothing else |
| P-5 | MCI reporting for the wave cannot be reconciled (see the continuity spec) | A wave whose results cannot be read cannot be judged; pause until reporting is trustworthy again |

Pause triggers that remain unexplained after investigation escalate to rollback for that wave. The default direction is always toward the platform whose behaviour is understood.

## Who decides

- Any team member who observes an R-trigger pauses the journey first and reports second. Nobody needs permission to stop a send that is emailing a suppressed member.
- The rollback decision for a wave belongs to the journey's named owner from the Day 1 inventory, with the consent / data owner concurring on R-1, R-2, and R-4. Schedule pressure is not a decision input recorded anywhere in this playbook.

## How a rollback is executed

1. Disable entry to the Next Flow for the affected journey (segment entry and Data Action triggers for that journey only — DA-06 suppression holds stay active; rollback never weakens suppression).
2. Hold any Next sends already queued for that journey and review them against current suppression state before releasing or cancelling them.
3. Re-activate the paused Engagement journey with its original entry DE and schedule, unchanged. It was kept intact for this purpose; do not "improve" it during a rollback.
4. Confirm Engagement entry, suppression (DE-05), and consent checks are operating as they did before the wave — rollback returns to a known state and then verifies it, it does not assume it.
5. Members mid-journey on the Next side are not migrated mid-flight. They either complete the short remaining Next path (if the trigger was not consent- or suppression-related) or are exited with a logged reason. For R-1 / R-2 rollbacks, they are exited — no further Next sends.
6. Record the rollback: trigger, evidence, decision, and the fix required before re-attempt. The wave's parallel run is repeated in full before a second cutover attempt.

## What rollback is not

- Rollback is never a reason to loosen SEG-00, widen a segment definition, or override a consent state to "get the numbers back." A smaller audience that is correct beats a larger one that is not.
- Rollback of one wave does not roll back earlier, completed waves. Each journey stands on its own comparison.
- Rollback does not touch Stays assets. J-03, J-07, and CP-01 were never part of the wave and are not part of its recovery.
