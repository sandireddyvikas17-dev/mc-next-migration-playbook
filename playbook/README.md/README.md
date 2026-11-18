# Cutover Playbook — Day 3

> Fictional brand, design study. *Northstar Credit Union* is invented. Honest framing from the repo README applies throughout: no Marketing Cloud Next production experience is claimed; this is a migration design built from real Marketing Cloud Engagement + Data Cloud experience. Not employer work.

Day 1 inventoried what Northstar has and gave every asset a Moves / Stays / Retires verdict. Day 2 designed the Data 360 segments, the consent sync, and the Data Action triggers every Moves verdict depends on. Day 3 is how the move actually happens — without a big-bang switch-off.

**The Day 3 lesson, stated plainly:** run both, compare everything, then cut over. A migration you cannot compare is a migration you cannot prove, and a cutover you cannot roll back is not a plan — it is a hope.

## What is in this folder

| File | Contents |
|---|---|
| `phased-cutover-plan.md` | The waves: Wave 0 foundation (suppression gate + consent sync + reporting baseline), then one Moves journey at a time — J-05, J-02, J-01, J-04 — with entry criteria, exit criteria, and what Stays and Retires do in each wave |
| `parallel-run-validation.md` | How the parallel run works: shadow comparison, then split-audience live sends; audience-count checks, the parity investigation order from Day 2, and suppression-parity SQL for the Engagement side |
| `rollback-criteria.md` | The stop and rollback rules: what triggers a pause, what triggers a rollback to Engagement, who decides, and how a rollback is executed without creating a second consent or suppression problem |
| `reporting-continuity-mci.md` | Marketing Cloud Intelligence (MCI) reporting-continuity spec: one reporting view across Engagement and Next for the whole coexistence period, so performance history does not break at cutover |

## The three Day 3 decisions, up front

1. **No wave starts until its gate passes.** Wave 0 is not a journey — it is the suppression gate (SEG-00 reconciled with DE-05), the consent sync running cleanly, and the MCI baseline. Every later wave inherits that gate. A wave that starts on an unreconciled suppression set is already broken, whatever its sends look like.
2. **Parallel run before cutover, every time.** Each Moves journey runs in Next alongside Engagement first — shadow mode (build the audience, send nothing), then a split live audience — and the counts, suppression, and send results are compared on the same refresh cycle. Cutover is the last step of a wave, not the first.
3. **Rollback is designed, not improvised.** The criteria, the decision owner, and the steps are written here before any wave starts. Rolling back means re-pointing entry to the Engagement journey that was kept intact for exactly this purpose — it never means rebuilding anything under pressure.

## Principles

1. **One journey, one wave, one change at a time.** A wave changes the sending platform for one journey. It does not also change authentication, domains, content strategy, or the audience definition. If two things change, no comparison means anything.
2. **Consent and suppression outrank schedule.** Any suppression or consent mismatch is a stop, not a variance to note and carry. See `rollback-criteria.md`.
3. **Stays assets are the safety net.** J-03, J-07, and CP-01 keep running in Engagement through every wave. That is what makes rollback possible and what keeps service messaging untouched by a marketing migration.
4. **Retires happen on purpose, early.** J-06 and A-04 are switched off in Wave 0, with their replacement named (J-05 and J-01 respectively). Nothing retired is rebuilt, and nothing retired is left running "just in case."
5. **Fictional data only.** No real member, account, or employer data appears anywhere in this playbook. Validation examples use invented test members only.
