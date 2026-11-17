# Consent & Preference Sync Design — Engagement Preference Centre ↔ Data 360

> Fictional brand, design study. This is the Day 2 artefact CP-01's Stays verdict was gated on in Day 1: the preference centre stays on CloudPages *until this design exists*. It now exists — CP-01's verdict is revisited in §8.

**The Day 2 LinkedIn lesson, stated plainly:** consent is the hardest part of any platform move. Journeys are visible; consent is a web of member choices captured in one system, enforced in another, and — during a phased migration — expected to be honoured in both at once. Get the sync wrong and the failure mode is not a broken dashboard. It is emailing someone who asked you not to.

## 1. The problem this design solves

During Northstar's phased move, all of the following are true at the same time:

- The preference centre members actually use is CP-01, a CloudPages page in Engagement, writing to `DE_Preferences` with AMPscript + SSJS handlers.
- Journeys still running in Engagement enforce consent from Engagement subscriber state and DE-05.
- Flows running in Next enforce consent from Data 360 — the unified profile.
- A member does not know or care which platform a send came from. An opt-out given anywhere must be honoured everywhere, immediately in effect and verifiably in audit.

So there cannot be two consent truths. This design makes **Data 360 the consent system of record** — consent and suppression state live on the unified profile — while **CP-01 remains a capture surface** that writes into that record. Engagement becomes an enforcement consumer of the same record, not a second owner of it.

## 2. What is stored, and where

On the unified profile in Data 360, per member:

| State | Values | Notes |
|---|---|---|
| Channel consent — email | `opted_in` / `opted_out` / `no_record` | `no_record` is not consent. Marketing sends require `opted_in` (or the applicable subscription below) |
| Channel consent — SMS | `opted_in` / `opted_out` / `no_record` | Separate from email, always. An email opt-in says nothing about SMS |
| Communication subscriptions | Per-subscription `subscribed` / `unsubscribed`, per channel | See §3 — this is where Engagement lists land |
| Suppression state | One flag per reason: complaint, hard bounce, staff account, deceased member, legal hold, global opt-out | Each with source system + timestamp. Any one flag suppresses all marketing sends |
| Preference attributes | Product-interest and frequency choices from CP-01 | Targeting inputs, not consent. Losing one is a quality defect; losing an opt-out is a compliance defect — the design treats them differently (§4) |

Engagement keeps a synced copy for the journeys still running there: subscriber status + a suppression DE refreshed from Data 360 state (the successor of DE-05 — same role, opposite direction of flow from today).

## 3. Communication subscriptions (carried-forward item 1)

In Engagement, Northstar's consent granularity lives in lists and publication lists. In the Next / Data 360 model, the equivalent unit is the **communication subscription**: a named subscription a member can be in or out of, per channel. Every Engagement list gets an explicit mapping row — no list is carried over "by similarity."

| Engagement source | Communication subscription (Data 360) | Email | SMS | Notes |
|---|---|---|---|---|
| All Subscribers (master list) | — (not a subscription) | Channel consent — email | — | Master-list status maps to channel consent, not to a subscription. Conflating the two is how global opt-outs get lost |
| Member Offers publication list | `member-offers` | ✔ mapped | — | Email-only today; no SMS subscription is created from an email list |
| Product News publication list | `product-news` | ✔ mapped | — | — |
| Mobile Alerts list (SMS programme) | `mobile-alerts` | — | ✔ mapped | SMS-only. Its opt-ins do **not** create any email state |
| Statement & Service list | — (not a marketing subscription) | — | — | Service messaging (J-07, Stays) runs on its own basis in Engagement; it is deliberately **not** mapped into marketing subscriptions |
| CP-01 global opt-out | — (not a subscription) | Channel consent → `opted_out` + suppression flag | Channel consent → `opted_out` + suppression flag | A global opt-out is stronger than any subscription state and overrides all of them (§4) |

Rules that come with the table:

1. **Email and SMS are mapped in separate columns, in separate decisions.** A row with a ✔ in one channel and — in the other means exactly that.
2. **Unmapped = not migrated.** Any list found later that is not in this table does not silently become a subscription; it gets a row and a decision, or it retires.
3. **Subscription state migrates with its timestamp.** A member who unsubscribed from `member-offers` in Engagement is unsubscribed in Data 360 from the moment of migration — there is no "fresh start" that resets preferences.

## 4. Sync directions and conflict rules

| Data | Direction | Cadence (design) | Conflict rule |
|---|---|---|---|
| Channel consent (opt-in / opt-out) | Bidirectional: CP-01 (Engagement) ↔ Data 360; also written by Engagement unsubscribe handling and by Next-side preference capture when it exists | Near-real-time for opt-outs; batch is acceptable for opt-ins | **Most recent explicit member action wins** — except that an opt-out is never overwritten by an older opt-in arriving late. Every write carries its action timestamp, and sync applies writes in action-time order, not arrival order |
| Communication subscriptions | Bidirectional, same pattern as channel consent | Batch acceptable; near-real-time for unsubscribes | Same as channel consent, applied per subscription per channel |
| Suppression state (complaint, hard bounce, legal hold, deceased, staff) | **Into Data 360 from wherever it originates; out to Engagement as the DE-05 successor feed. Never deleted by sync.** | Near-real-time | **Most restrictive wins.** A suppression flag is cleared only by its owning process (e.g. Operations clears a legal hold), never by a conflicting sync write and never by a member preference change |
| Preference attributes (interests, frequency) | Bidirectional | Batch | Most recent write wins. A lost preference write degrades personalisation; it can never create permission to send |

The asymmetry is the design: **preferences are eventually consistent; opt-outs and suppressions are not allowed to be.** If the sync is down, sends that depend on it pause for the affected audiences rather than proceeding on stale consent (see §7 and the failure handling in `data-action-triggers.md`).

## 5. The suppression gate, rebuilt (DE-05 → unified-profile state)

Day 1: A-05 / DE-05 / Q-05 carry every hard exclusion, and no Moves verdict is safe until that logic exists on the Data 360 side. Day 2 build:

1. Each Q-05 rule becomes a named suppression reason on the unified profile (complaint, hard bounce, staff account, deceased member, legal hold — plus global opt-out from §3). The rule logic is carried, not summarised: the same source lists, the same inclusion criteria.
2. Enforcement happens at **three gates**, because consent can change between any two of them: **segment build** (segments in `segment-definitions.md` are built net of SEG-00), **activation / Data Action** (the trigger re-checks state — `data-action-triggers.md`), and **send** (the Flow's send step checks channel consent + suppression at send time).
3. Engagement's remaining journeys consume the same state via the DE-05 successor feed (§2), so a member suppressed on the unified profile is suppressed in both platforms from one decision.
4. **Gate criterion for cutover:** SEG-00 and DE-05 reconcile under the parity method in `segment-definitions.md` §4, with every difference explained by a named cause (definition / data / identity / consent) — before any Moves journey's Flow is enabled.

## 6. Shared sending domains (carried-forward item 3)

Engagement and Next send side-by-side during the migration, and they send **from the same authenticated sending domains**. Northstar does not stand up a new "Next" domain and split its sending reputation in two:

- The existing authenticated domain(s) and their SPF / DKIM / DMARC alignment are shared by both platforms during coexistence, so a member's mail client sees one sender identity throughout the move, and deliverability history is not forked mid-migration.
- Send classifications stay distinct even though domains are shared: service notices (J-07) keep their classification and their separate basis; sharing a domain never merges marketing and service consent.
- Subdomain strategy, IP warming, and authentication changes are explicitly **out of scope for a cutover wave**: a wave changes the sending platform for a journey, and nothing else about how its mail is authenticated. Changing both at once makes every deliverability signal from Day 3's parallel run uninterpretable.

## 7. Test environments (carried-forward item 5)

Engagement and Next do not offer the same kind of test environment, and the migration plan has to say so instead of discovering it:

- Engagement testing happens in the live org's structures (test DEs, test sends, preview audiences) — there is no full Engagement sandbox copy of Northstar's programme in this design.
- Next / Data 360 testing works against Data 360 data spaces / test configurations rather than a cloned Engagement org, so a Flow test exercises the *Data 360 side* (segments, consent state, Data Actions) with test profiles — it does not replay Engagement data.
- Practical consequence written into Day 3: consent-sync testing uses a small set of invented test members whose consent state is changed deliberately in each surface (CP-01 opt-out, Engagement unsubscribe, Data 360-side change) and verified at all three gates in §5, in both platforms. Fictional test members only — the same fictional-data rule as the rest of this repo.

## 8. What this design changes for CP-01

CP-01's Day 1 verdict was **Stays — gated by this design**. With the design in place:

- CP-01 **stays on CloudPages as the capture surface** for the whole coexistence period. Members keep one familiar preference page; nothing about the member experience changes on migration day.
- What changes is behind the page: CP-01's writes go to the Data 360 consent record (with Engagement receiving its synced copy), instead of `DE_Preferences` being the end of the line.
- A Next-side preference experience can be judged only after this sync has run cleanly through a full cutover wave — that judgement belongs to a later phase, not to this playbook's Day 3.
