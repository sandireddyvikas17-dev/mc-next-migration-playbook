# Engagement Asset Inventory — template + Northstar Credit Union (filled)

> Fictional brand, fictional assets. This is a design-study inventory for *Northstar Credit Union*, built to show the method. Counts and volumes are illustrative planning signals, not real production metrics.

## Part 1 — The reusable template

Copy this structure for any Engagement org. One row per asset — and one row per *entry path* where a journey can be entered more than one way.

### Field guide

| Field | What to write | Why it matters for migration |
|---|---|---|
| Asset ID | Short stable code (`J-01`, `A-03`, `DE-02`…) | Mapping tables and cutover plans reference IDs, not long names that drift |
| Asset / type | Name + type (Journey, Automation, DE, Email/Content, CloudPage, Script, Integration) | Different types move differently — or don't move at all |
| Purpose | One plain sentence: what member outcome it exists for | If nobody can write this, the asset is a Retires candidate |
| Owner | Team or named owner | Unowned assets are findings, not passengers |
| Entry / trigger | How it starts: schedule, segment refresh, API event, file drop, manual | Entry shape decides whether a Next Flow can replace it cleanly |
| Audience / data in | Source DEs, lists, queries, upstream systems | Shows the Data 360 data work a Moves verdict actually requires |
| Dependencies | Other assets it needs (SQL, suppression DEs, content, pages) | Hidden dependencies are the #1 cause of broken migrations |
| Personalisation / logic | AMPscript, SSJS, SQL complexity — light / moderate / heavy, with a note | Heavy scripting is the most common honest reason to Stay |
| Volume signal | High / medium / low sends or runs (planning signal only) | Sequence the migration by member impact, not by what is easiest |
| Last-change signal | Recent / ageing / untouched | Untouched + unowned = investigate before migrating anything |
| Risk notes | Consent, suppression, compliance, duplicate-entry — anything a rebuild must not lose | These become Day 2 / Day 3 requirements |

### Blank template

```markdown
| ID | Asset / type | Purpose | Owner | Entry / trigger | Audience / data in | Dependencies | Logic | Volume | Last change | Risk notes |
|---|---|---|---|---|---|---|---|---|---|---|
| J-01 |  |  |  |  |  |  |  |  |  |  |
| A-01 |  |  |  |  |  |  |  |  |  |  |
| DE-01 |  |  |  |  |  |  |  |  |  |  |
```

---

## Part 2 — Northstar's filled inventory

### Journeys

| ID | Asset | Purpose | Owner | Entry / trigger | Audience / data in | Dependencies | Logic | Volume | Last change | Risk notes |
|---|---|---|---|---|---|---|---|---|---|---|
| J-01 | New Member Onboarding — Journey Builder | Welcome a new member, set up online banking, introduce direct deposit over 21 days | Member Marketing | Data Extension entry, daily after core extract | `DE_NewMembers_Daily` from Automation A-01 | A-01, E-01–E-04, suppression DE-05 | Moderate: attribute splits (product held, age band), waits; light AMPscript salutation | High | Recent | Duplicate entry risk: A-04 also feeds this journey (see Retires) |
| J-02 | Card Activation & First Use — Journey Builder | Get a newly issued debit card activated and used once within 14 days | Cards Team | API event on card issued + DE entry for reissue batch | Card platform event; `DE_Cards_Issued` | E-05, E-06; exit on activation event | Light: event exit, one decision split (activated / not) | High | Recent | Entry is event-shaped already — cleanest Moves candidate |
| J-03 | Loan Application Nudge — Journey Builder | Bring an abandoned loan application back to completion | Lending Marketing | DE entry, hourly automation A-02 | `DE_AbandonedApps` from A-02 SQL | A-02, E-07, heavy AMPscript lookup block CB-03 (rate/product lookup across 3 DEs) | Heavy: per-send AMPscript lookups + SSJS status check | Medium | Recent | Logic must be simplified before any rebuild is honest |
| J-04 | Savings Goal & CD Maturity — Journey Builder | Remind members before a CD matures and when a savings goal stalls | Member Marketing | DE entry, weekly | `DE_Maturity_60d`, `DE_GoalStalled` from A-03 | A-03, E-08, E-09 | Light: date-based waits, product split | Medium | Recent | Dates live in core extract — Data 360 must hold maturity date (Day 2) |
| J-05 | Dormant Member Re-engagement — Journey Builder | Win back members with no login / transaction in 90+ days | Member Marketing | DE entry, weekly | `DE_Dormant_90` from A-03 | A-03, E-10, suppression DE-05 | Light: engagement split, 2 sends, goal = login | Medium | Recent | Overlaps J-06 — one of them retires |
| J-06 | Old Win-Back (2024 campaign) — Journey Builder | 2024 win-back offer | Unclear | DE entry, weekly | `DE_Dormant_90` (same DE as J-05) | E-11 (expired offer content) | Light | Low | Untouched | Same audience as J-05, expired offer, no clear owner — Retires |
| J-07 | Service & Transactional Notices — sends via Transactional / triggered patterns | Address change, statement-ready, payment-due service notices | Operations | API / triggered sends | Core banking events | E-12 templates, send classifications | Light content, strict delivery expectations | High | Recent | Service messaging, not marketing — consent model differs; Stays |

### Automations & SQL

| ID | Asset | Purpose | Owner | Entry / trigger | Data in → out | Dependencies | Logic | Volume | Last change | Risk notes |
|---|---|---|---|---|---|---|---|---|---|---|
| A-01 | New Member Extract & Build — Automation Studio | Land core extract, build `DE_NewMembers_Daily` | Marketing Ops | Daily schedule + file drop | Core banking extract → DE-01 | SQL Q-01, import activity | Moderate SQL: dedupe on member ID, exclude staff accounts | High | Recent | Staff-account exclusion must survive any move |
| A-02 | Abandoned Application Refresh — Automation Studio | Hourly refresh of `DE_AbandonedApps` | Marketing Ops | Hourly schedule | Application system extract → DE-03 | SQL Q-02 | Moderate SQL + status mapping | Medium | Recent | Hourly batch pretending to be real-time — flag in mapping |
| A-03 | Lifecycle Audiences Weekly — Automation Studio | Build maturity, goal-stalled, and dormant audiences | Marketing Ops | Weekly schedule | Core + digital activity → DE-02, DE-04 | SQL Q-03 (dormancy), Q-04 (maturity window) | Moderate SQL | Medium | Recent | Q-03's dormancy definition is the *only* written definition — preserve it verbatim for Day 2 segment rebuild |
| A-04 | Welcome Re-send Automation — Automation Studio | Re-feeds members who missed J-01 entry "just in case" | Unclear | Daily schedule | DE-01 → J-01 re-entry | None documented | Light | Low | Ageing | Duplicate entry path into J-01 — Retires; fix entry, don't carry a second feed |
| A-05 | Global Suppression Refresh — Automation Studio | Rebuild `DE_Suppression_Global` (complaints, hard bounces, staff, deceased-member flags, legal holds) | Marketing Ops | Daily schedule | All subscribers + ops lists → DE-05 | SQL Q-05 | Moderate SQL, high consequence | High | Recent | **Highest-risk asset in the org.** Every Moves verdict depends on this logic existing on the Data 360 side (Day 2) |

### Data Extensions, content, pages, and scripts

| ID | Asset / type | Purpose | Owner | Notes that matter for migration |
|---|---|---|---|---|
| DE-01 | `DE_NewMembers_Daily` — sendable DE | J-01 entry audience | Marketing Ops | Becomes a Data Cloud segment input in Day 2; member ID is the join key |
| DE-02 | `DE_Maturity_60d` — sendable DE | J-04 entry audience | Marketing Ops | Needs maturity date as a real date field in Data 360, not a weekly snapshot |
| DE-03 | `DE_AbandonedApps` — sendable DE | J-03 entry audience | Marketing Ops | Carries application status the heavy AMPscript reads at send time |
| DE-04 | `DE_Dormant_90` — sendable DE | J-05 + J-06 entry audience | Marketing Ops | One audience, two journeys — the overlap Day 1 retires |
| DE-05 | `DE_Suppression_Global` — non-sendable DE | Exclusion for every marketing send | Marketing Ops | Must be rebuilt as unified-profile consent / suppression in Day 2 before any journey moves |
| E-01–E-04 | Onboarding email set — Content Builder | J-01 sends (welcome, online banking, direct deposit, product intro) | Member Marketing | Light AMPscript (name, product) — content rebuild in Next is straightforward |
| E-05–E-06 | Card activation set — Content Builder | J-02 sends | Cards Team | Light personalisation; last-4-digits only, never full card data |
| E-07 + CB-03 | Application nudge email + lookup content block | J-03 send | Lending Marketing | CB-03's multi-DE AMPscript lookup is why J-03 Stays until simplified |
| E-08–E-11 | Maturity / goal / re-engagement sets | J-04, J-05, J-06 sends | Member Marketing | E-11 carries an expired offer — retires with J-06 |
| E-12 | Service notice templates | J-07 sends | Operations | Separate send classification; stays with J-07 |
| CP-01 | Preference Centre — CloudPages | Member email preferences + global opt-out | Member Marketing | AMPscript + SSJS handlers writing to DE_Preferences; **no honest Next equivalent until Day 2's consent sync design exists — Stays** |
| INT-01 | Core banking extract — SFTP file drop | Feeds A-01, A-03 | IT / Marketing Ops | The Data 360 ingestion design (Day 2) starts from this same source |
| INT-02 | Card platform events — API | J-02 entry + exit events | Cards / IT | Event pattern maps naturally to a Next trigger — reuse the payload shape |

## What the inventory surfaced (before any mapping)

1. **One journey, two doors.** J-01 is fed by both A-01 (by design) and A-04 (by habit). Migration would faithfully rebuild the duplicate if the inventory hadn't separated entry paths.
2. **Suppression is a single point of truth — and a single point of failure.** DE-05 / A-05 / Q-05 carry every hard exclusion Northstar relies on. No journey verdict is safe until Day 2 rebuilds that logic on the unified profile.
3. **The only written dormancy definition lives in SQL Q-03.** If Day 2's Data Cloud segment defines "dormant" differently, parallel-run counts will never match — and the team will blame the platform for a definition change nobody declared.
4. **One journey is script-heavy, not journey-heavy.** J-03's real logic sits in CB-03's AMPscript, not in Journey Builder. Mapping the journey diagram alone would have missed the actual work.
5. **An unowned journey is still sending.** J-06 runs weekly on an expired offer to the same audience as J-05. The cheapest migration win is switching it off.
