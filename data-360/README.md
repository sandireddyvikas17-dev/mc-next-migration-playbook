# Data & Consent on Data 360 — Day 2

> Fictional brand, design study. *Northstar Credit Union* is invented. Honest framing from the repo README applies throughout: no Marketing Cloud Next production experience is claimed; this is a migration design built from real Marketing Cloud Engagement + Data Cloud experience. Not employer work.

Day 1 answered *what Northstar has* and gave each asset a Moves / Stays / Retires verdict. Day 2 designs the layer every Moves verdict depends on: the data and consent model on **Data 360 (Data Cloud)**.

## What is in this folder

| File | Contents |
|---|---|
| `segment-definitions.md` | Every Engagement audience that feeds a Moves journey, rebuilt as a Data Cloud segment on the unified profile — with entry/exit rules, the source Engagement definition it was carried from, and the parity rule that keeps Day 3 counts honest |
| `consent-preference-sync-design.md` | How consent and preferences stay in step between the Engagement preference centre (CP-01) and Data 360 during the side-by-side period — sources of truth, sync directions, conflict rules, communication subscriptions, and the suppression gate (DE-05 rebuilt on the unified profile) |
| `data-action-triggers.md` | Data Action trigger specs: which Data 360 event or segment change fires which Next Flow, with payload, guardrails, and failure handling |

## The three Day 2 decisions, up front

1. **Definitions travel verbatim.** A segment rebuilt in Data 360 uses the Engagement definition it replaces — word for word where a written definition exists. Northstar's dormancy definition lives only in Automation Studio SQL Q-03, so `segment-definitions.md` carries it unchanged. A "close enough" redefinition is a silent audience change, and it makes every parallel-run comparison in Day 3 meaningless.
2. **Consent is one model, two surfaces.** During the phased move, members can change preferences in Engagement (CP-01) and be messaged from Next. There is therefore one consent model on the unified profile in Data 360, with CP-01 as a capture surface writing into it — not two preference stores that are reconciled later. The sync design in `consent-preference-sync-design.md` is bidirectional for preferences and one-directional (most-restrictive-wins) for hard suppressions.
3. **Suppression is the gate, again.** Day 1 named A-05 / DE-05 the gate for every Moves verdict. Day 2 builds that gate: the global suppression logic is re-expressed as consent and suppression state on the unified profile, checked at segment build, again at activation / Data Action, and again at send. No Moves journey cuts over until this exists and its counts reconcile with DE-05.

## The five items carried forward from the Day 1 gap check

Day 1's inventory was checked against Salesforce's Engagement + Next coexistence guidance and five items were deferred to this build. They are designed here, by name:

| # | Item | Where it lands |
|---|---|---|
| 1 | Communication subscriptions — each Engagement list / publication list mapped to a communication subscription, separately for email and for SMS | `consent-preference-sync-design.md` §3 |
| 2 | Data Extension sync verdict — per DE, whether it syncs to Data 360, is rebuilt as a segment input, or does not sync, and how it relates to the unified-profile attribute model | `segment-definitions.md` §2 |
| 3 | Shared sending domains — Engagement and Next send from the same authenticated domains during coexistence, so deliverability and member recognition do not fork | `consent-preference-sync-design.md` §6 |
| 4 | Legacy-compatibility flag — per asset, whether legacy Engagement data is usable by Next features as-is or needs transformation / cannot be used | `segment-definitions.md` §2 (per-DE flag) and §5 (per-segment flag) |
| 5 | Test-environment note — Engagement and Next handle test / sandbox environments differently; what that means for testing this migration | `consent-preference-sync-design.md` §7 |

## Principles

1. **A segment is not permission to send.** Membership, consent, and suppression are three separate checks. A member can be correctly in a segment and correctly un-sendable.
2. **Most restrictive wins, always, for suppression.** An opt-out, complaint, hard bounce, legal hold, or deceased-member flag from either platform suppresses in both, until a newer explicit member action — never a sync job — changes it.
3. **Every exclusion carries a reason.** Segment build, activation, and send-time checks record *why* a member was held out, in a form a second person can look up. "Audience was smaller" is not an explanation.
4. **Fictional data only.** No real member, account, or employer data appears anywhere in this design.
