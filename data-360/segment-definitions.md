# Segment Definitions — Engagement Audiences Rebuilt as Data Cloud Segments

> Fictional brand, design study. Audience sizes are not stated anywhere in this file: Northstar is invented, and inventing counts would be inventing metrics. Parity in Day 3 is defined as a *comparison method*, not a number.

## 1. How to read this file

Each segment below replaces one Engagement audience named in Day 1's inventory (`mapping/engagement-asset-inventory.md`). For each one:

- **Replaces** — the Engagement DE / automation / SQL it supersedes
- **Definition** — the rule, carried from the Engagement source. Where the source is SQL, the business definition is carried verbatim in words; the Data Cloud segment expresses the same rule on the unified profile, it does not reinterpret it
- **Entry / exit** — when a unified profile joins and leaves
- **Consent gate** — the consent / suppression state that must also be true before the segment can be activated (see `consent-preference-sync-design.md`)
- **Legacy flag** — whether the underlying Engagement data is usable by Next as-is (see §5)

Segments are built on the **unified profile** (the Data 360 harmonised Individual), keyed on member ID — the same join key DE-01 uses in Engagement. Email address is a contact point, never the identity key: members share inboxes and change addresses, and Day 1's inventory already keys every audience on member ID.

## 2. Data Extension sync verdicts (carried-forward item 2 + legacy flag, item 4)

Before any segment can be rebuilt, each Engagement DE gets a sync verdict: what happens to its data in Data 360, and how it relates to the unified-profile attribute model.

| DE | Engagement role | Sync verdict | Data 360 relation | Legacy-compatible with Next as-is? |
|---|---|---|---|---|
| DE-01 `DE_NewMembers_Daily` | J-01 entry audience | **Do not sync the DE. Rebuild from source.** Membership is derived from the core banking extract (INT-01) ingested into Data 360; the daily DE is a snapshot of that source, and syncing a snapshot would freeze it | Member attributes (join date, products held, age band, staff-account flag) land as attributes on the unified profile | **No — transform.** The DE carries join date as a snapshot row, not a profile attribute; it must land as a dated attribute before a segment can use it |
| DE-02 `DE_Maturity_60d` | J-04 entry audience | **Do not sync the DE. Ingest the underlying dates.** | CD maturity date and savings-goal last-contribution date land as real date fields on the unified profile / related account objects | **No — transform.** Day 1 flagged this directly: maturity must be a real date field in Data 360, not a weekly snapshot, or date-anchored Flows cannot work |
| DE-03 `DE_AbandonedApps` | J-03 entry audience (J-03 Stays) | **Sync for reference only, no segment yet.** J-03 stays in Engagement, so no Next segment is built from this DE in this phase | Application status is modelled on an application object related to the unified profile, ready for J-03's named exit condition | **No — transform.** The send-time lookup values CB-03 reads across three DEs must land on the application record itself before any rebuild is honest (Day 1 exit condition) |
| DE-04 `DE_Dormant_90` | J-05 (+ retired J-06) entry audience | **Do not sync the DE. Rebuild as a segment** using Q-03's definition verbatim (§3, SEG-04) | Login and transaction activity land as engagement events on the unified profile, from core + digital sources | **Yes, with one condition.** Activity history is usable as-is *if* the full lookback window is ingested; a partial history load would silently re-define "90 days of no activity" — see the parity rule in §4 |
| DE-05 `DE_Suppression_Global` | Exclusion for every marketing send | **Never sync as an audience. Rebuild as consent / suppression state** on the unified profile — designed in `consent-preference-sync-design.md` §5 | Complaint, hard-bounce, staff, deceased-member, and legal-hold flags become suppression state on the unified profile, each with source and timestamp | **No — transform, and it is the gate.** Suppression expressed as "member ID in a DE" does not transfer to Next; it must exist as profile state before any Moves journey cuts over |

`DE_Preferences` (written by CP-01) is not in Day 1's DE table because it is a consent store, not an audience. Its sync design is the whole subject of `consent-preference-sync-design.md`.

## 3. The segments

### SEG-01 — New Members, Last 1 Day (replaces DE-01 / A-01 / Q-01, feeds J-01 → Moves)

- **Definition:** Unified profiles whose member join date is the previous calendar day, excluding staff accounts. The staff-account exclusion is Q-01's rule and is carried unchanged — it is a suppression-class rule, not a targeting preference.
- **Entry:** Daily, on segment refresh after the core extract lands in Data 360. **Exit:** Automatic after 1 day (membership is a window, not a state); the J-01 successor Flow owns the 21-day cadence after entry, exactly as the Engagement journey does.
- **Consent gate:** Marketing consent present, no suppression state (any reason).
- **Legacy flag:** Transform (DE-01, §2).

### SEG-02 — Card Issued, Not Yet Activated (replaces DE entry path of J-02 → Moves)

- **Definition:** Unified profiles with a card-issued event and no card-activated event within 14 days of issue. This segment exists for the reissue / batch path only; the primary J-02 entry is event-triggered (see `data-action-triggers.md`, DA-01).
- **Entry:** On segment refresh when the issued-not-activated condition becomes true. **Exit:** Immediately on a card-activated event, or at day 14.
- **Consent gate:** Marketing consent present, no suppression state. (Activation reminders are marketing sends in this design; service notices about the card itself stay with J-07 in Engagement and follow its own basis.)
- **Legacy flag:** Yes — the card platform event payload (INT-02) is reused in its existing shape.

### SEG-03 — CD Maturing in 60 Days / Savings Goal Stalled (replaces DE-02 / A-03 / Q-04, feeds J-04 → Moves)

Two related segments, kept separate because their Flows anchor differently:

- **SEG-03a — CD Maturity Window:** Unified profiles holding a CD whose maturity date falls within the next 60 days. Q-04's window is carried unchanged. Entry on entering the window; exit on maturity date passing or the CD closing/renewing.
- **SEG-03b — Savings Goal Stalled:** Unified profiles with an active savings goal and no goal contribution in the stall window defined alongside Q-04 in A-03. Day 1 required this definition to be written next to the dormancy definition, from one place — it is recorded here, in the same file and the same terms (days since last contribution), so the two time-based definitions cannot drift apart. Entry when the stall window is crossed; exit on any new contribution.
- **Consent gate (both):** Marketing consent present, no suppression state.
- **Legacy flag:** Transform (DE-02, §2) — both segments are impossible until maturity and contribution dates exist as real date fields.

### SEG-04 — Dormant 90+ (replaces DE-04 / A-03 / Q-03, feeds J-05 → Moves; J-06 Retires)

- **Definition — carried verbatim from Q-03, the only written dormancy definition in Northstar's Engagement org:** a member is dormant when they have had **no login and no transaction in 90 or more days**. Day 2 does not reinterpret this: no engagement-score substitute, no "or no email click" addition, no window change. If the business ever wants a different definition, that is a declared definition change with its own sign-off — not a migration side-effect.
- **Entry:** On segment refresh when the 90-day threshold is crossed, computed across *all* ingested activity sources (core transactions + digital logins). A member active in either source is not dormant — that cross-source rule is what the unified profile exists for, and it is already implicit in Q-03 running over core + digital activity in A-03.
- **Exit:** On any login or transaction (the J-05 successor Flow's goal event), or on suppression state appearing.
- **Consent gate:** Marketing consent present, no suppression state. This segment is also Day 3's parallel-run proof: the Engagement journey and the Next Flow run it on split audiences and compare.
- **Legacy flag:** Yes, with the §2 condition — full activity lookback must be ingested before this segment is trusted.

### SEG-00 — Global Suppression (replaces DE-05 / A-05 / Q-05 — the gate)

Not a targetable segment. SEG-00 is the exclusion set every segment above is built *net of*: unified profiles carrying any suppression state (complaint, hard bounce, staff account, deceased-member flag, legal hold, global opt-out). It is defined here so segment builders reference one named exclusion, and specified fully in `consent-preference-sync-design.md` §5. **No Moves journey cuts over until SEG-00's membership reconciles with DE-05 under the §4 parity method.**

## 4. The parity rule (why Day 3 comparisons will mean anything)

For every rebuilt segment, Day 3's parallel run compares the Engagement audience and the Data Cloud segment on the same refresh cycle. A count difference is investigated in this order, and only in this order:

1. **Definition difference** — did the rebuild change the rule? (Prevented by §3's verbatim rule; checked first because it is the cheapest cause to prove.)
2. **Data completeness** — is the full lookback / source set ingested? (The SEG-04 condition in §2.)
3. **Identity difference** — did identity resolution merge or split profiles differently from Engagement's member-ID keying?
4. **Consent / suppression difference** — is SEG-00 applying exclusions the Engagement send did not, or missing ones it did? A segment that is smaller *because suppression is finally being honoured* is a finding to report, not a defect to "fix" by loosening the gate.

## 5. Legacy-compatibility summary (carried-forward item 4)

| Asset | Usable by Next as-is? | What has to happen first |
|---|---|---|
| INT-02 card event payloads | Yes | Nothing — payload shape is reused (DA-01) |
| DE-04 activity history | Yes, conditionally | Full lookback ingested before SEG-04 is trusted |
| DE-01 / DE-02 snapshots | No — transform | Snapshot rows become dated profile attributes / real date fields |
| DE-03 + CB-03 lookup data | No — transform | Lookup values land on the application record (also J-03's exit condition) |
| DE-05 suppression DE | No — transform | Rebuilt as profile consent / suppression state (§5 of the consent design) |
| AMPscript personalisation in E-01–E-10 | Partially | Salutation-level logic rebuilds trivially; CB-03-class multi-DE lookups do not transfer at all and are the reason J-03 Stays |
