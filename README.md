# Marketing Cloud Next Migration Playbook
## Engagement → Next, on Data 360 — a design study

**Builder:** Vikas Sandireddy — Salesforce Marketing Cloud practitioner (7+ years overall, 5+ years hands-on SFMC; Data Cloud / Data 360, Marketing Cloud Intelligence, and Personalization hands-on since 2023)

**What this is — read this first:** A personal portfolio **design study**. Not employer work. No employer data, no employer systems, no real customer data.

> **Honest framing:** I have **not** run Marketing Cloud Next in production. This playbook is a migration design study built from my real Marketing Cloud Engagement and Data Cloud experience. It shows how I would plan the move — inventory first, mapping second, cutover last — not a claim of hands-on Marketing Cloud Next production experience. No certifications are claimed in this repo, and no metrics are invented.

**Demo brand:** *Northstar Credit Union* — a fictional financial-services brand. All members, accounts, products, and addresses in this repo are invented for this study. Northstar is not a real client and this is not work done for any employer.

## The scenario

Northstar runs its member lifecycle marketing today in **Marketing Cloud Engagement**:

- Journey Builder journeys for onboarding, card activation, application nudges, and re-engagement
- Automation Studio SQL and scheduled automations refreshing sendable Data Extensions
- Content Builder emails with AMPscript personalisation
- A CloudPages preference centre
- Data coming in batches from core banking extracts, plus API entry for a few real-time events

Leadership wants a phased move to **Marketing Cloud Next on Data 360**, running both platforms side-by-side during the transition — the pattern most real migrations actually follow. Nobody sensible migrates by switching everything off on a Friday.

The mistake this playbook is built to avoid: starting with licences, org setup, or a target-state diagram before anyone can answer a simpler question —

**What do we actually have, what does each thing do, and what should happen to it?**

## Day 1 answer: inventory, then mapping

Day 1 produces two artefacts in `mapping/`:

1. **Engagement asset inventory** — a reusable template, filled in for Northstar's lifecycle programme. Every journey, automation, Data Extension, content asset, CloudPage, script, and integration gets one row: owner, entry, audience, dependencies, volume signal, last-change signal, and risk notes. You cannot map what you have not listed.
2. **Journey → Next Flow mapping table** — for each Northstar journey: how it enters today, how its decision logic works today, what it sends today — and the Next-side equivalent (segment- or event-triggered Flow, Data Cloud segment, Data Action), with one verdict per asset:

| Verdict | Meaning |
|---|---|
| **Moves** | Rebuild it in Next / on Data 360. The use case belongs on the new platform and the data and consent model can support it there. |
| **Stays** | Keep it in Engagement for now. It works, it is deeply Engagement-specific (for example heavy AMPscript / SSJS, transactional patterns, or a CloudPage), or moving it now adds risk without member benefit. Stays is a decision, not a delay tactic — each Stays row names what would have to change for it to move later. |
| **Retires** | Do not rebuild it anywhere. It is duplicated, superseded, low-value, or kept alive only because nobody owned switching it off. Retiring is part of the migration saving. |

The headline Day 1 finding for Northstar, from its own inventory:

- **4 journeys move** — onboarding, card activation & first use, savings / CD maturity, and dormant re-engagement. All four are segment- or lifecycle-event driven, which is exactly the shape Next on Data 360 is built for.
- **3 things stay** — the transactional / service messaging pattern, the CloudPages preference centre (until the Data 360 consent design in Day 2 replaces it properly), and one heavily scripted application-nudge journey whose logic needs simplifying before it deserves a rebuild.
- **2 things retire** — a duplicate welcome automation that feeds the onboarding journey a second way, and a dormant win-back journey that has overlapped the main re-engagement journey for two cycles.

That split — not a lift-and-shift of everything — is the point of Day 1.

## Build plan (3 days)

| Day | Focus | Artefacts |
|---|---|---|
| **Day 1 — Inventory & mapping** | Engagement asset inventory template + filled Northstar inventory; Journey → Next Flow mapping with moves / stays / retires verdicts | This README, `mapping/` |
| **Day 2 — Data & consent on Data 360** | Segments rebuilt as Data Cloud segments, consent / preference sync design between the Engagement preference centre and Data 360, Data Action trigger specs | `data-360/` |
| **Day 3 — Cutover playbook** | Phased cutover plan, parallel-run validation checks (audience counts, suppression parity), rollback criteria, MCI reporting-continuity spec | `playbook/` |

## Repo layout

| Folder / file | Contents |
|---|---|
| `mapping/engagement-asset-inventory.md` | The reusable inventory template (blank structure + field guide) and Northstar's filled inventory: journeys, automations, Data Extensions, content, CloudPages, scripts, and integrations |
| `mapping/journey-to-next-flow-mapping.md` | Journey-by-journey mapping: today's entry / decision logic / sends → Next Flow / segment / Data Action equivalent, verdict, and reasoning per asset |
| `mapping/README.md` | How to run the Day 1 method on any Engagement org: the order of work, the verdict rules, and the traps |
| `data-360/README.md` | Day 2 overview: the three data & consent decisions, and where the five items carried forward from the Day 1 gap check land |
| `data-360/segment-definitions.md` | Engagement audiences rebuilt as Data Cloud segments on the unified profile, per-DE sync verdicts, legacy-compatibility flags, and the parity rule |
| `data-360/consent-preference-sync-design.md` | Consent / preference sync between CP-01 and Data 360: system of record, communication subscriptions (email + SMS mapped separately), conflict rules, the DE-05 suppression gate rebuilt, shared sending domains, test environments |
| `data-360/data-action-triggers.md` | Data Action trigger specs DA-01–DA-06: payloads, guardrails, dedupe keys, failure handling |
| `playbook/README.md` | Day 3 overview: run both, compare everything, then cut over — the three cutover decisions and where each spec lives |
| `playbook/phased-cutover-plan.md` | Waves 0–4: foundation gate first, then J-05, J-02, J-01, J-04 one at a time, with entry / exit criteria per wave and the final state of Stays and Retires |
| `playbook/parallel-run-validation.md` | Shadow then split-live parallel runs: audience-count comparison table, the Day 2 parity investigation order, suppression-parity SQL (Q-P1–Q-P4), consent spot checks, and send-behaviour checks |
| `playbook/rollback-criteria.md` | Immediate rollback triggers (suppression / consent breach, duplicates, degraded consent sync), pause triggers, who decides, and how a rollback is executed |
| `playbook/reporting-continuity-mci.md` | MCI reporting-continuity spec: one programme view across both platforms, harmonised dimensions frozen during waves, baseline and per-wave comparison windows |
| `docs/BUILD_LOG.md` | Build log — one entry per day |

## Day 3 answer: cutover, proven in parallel

Day 3 produces the playbook in `playbook/`. Wave 0 moves no journey at all — it reconciles the suppression gate (SEG-00 with DE-05), proves the consent sync at all three gates, retires J-06 and A-04, and baselines MCI reporting. Only then do the Moves journeys cut over, one wave each, in this order: **J-05 dormant re-engagement** (the parallel-run proof), **J-02 card activation** (the event / Data Action proof), **J-01 onboarding** (high volume, after its duplicate entry path is gone), and **J-04 savings / CD maturity** (last, because it depends on real date fields in Data 360). Every wave runs shadow first, then split-live, compares audience counts and suppression by member and by reason, and keeps its Engagement journey paused and intact as the rollback target for a full cycle. J-03, J-07, and CP-01 stay in Engagement at the end of this playbook — that is the design, not an unfinished remainder.

## Principles used throughout

1. **Inventory before licence, mapping before build.** If an asset has no owner, no entry source, or no one who can explain it, that is a finding — record it, do not quietly carry it into the new platform.
2. **A verdict needs a reason a second person can check.** "Moves because it is segment-driven and its data already lands in Data 360" can be challenged. "Moves because it is important" cannot.
3. **Stays is a legitimate outcome.** Running Engagement and Next side-by-side is the normal phased pattern. Forcing every asset across on day one is how consent and suppression break.
4. **Retire on purpose.** Every retired asset is volume, maintenance, and confusion the new platform never inherits. Name what replaces it — or state plainly that nothing needs to.
5. **Consent and suppression are mapping inputs, not Day 3 cleanup.** An asset whose audience depends on a suppression rule that only exists in one Engagement automation is not ready to move, whatever its journey diagram looks like. Day 2 designs the Data 360 side of this properly.
6. **Fictional data only.** Sample members use invented names and `example.com` / credit-union-style fictional addresses. No real member, account, or employer data appears anywhere in this repo.

## Status

**Complete — 3 of 3 days built** — inventory & mapping (Day 1), the Data 360 data & consent design (Day 2), and the cutover playbook (Day 3). See `docs/BUILD_LOG.md` for the per-day record.

Other portfolio projects by the same builder live under the same GitHub account: Project #1 (SFMC campaign portfolio), Project #2 (Data 360 Unified Profile & Activation Lab), and Project #3 (Transactional & Journey API Integration Kit).
