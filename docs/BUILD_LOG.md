# Build Log — Marketing Cloud Next Migration Playbook

Personal portfolio design study for the fictional *Northstar Credit Union*. Honest framing throughout: no Marketing Cloud Next production experience is claimed; this is a migration design study built from real Marketing Cloud Engagement + Data Cloud experience. Not employer work.

## Day 1 — Inventory & mapping (Nov 16, 2026)

- Created the repo and the Day 1 structure: `README.md`, `mapping/`, `docs/BUILD_LOG.md`.
- Wrote the README with the honest framing up front and Northstar's scenario: a phased Engagement → Next move on Data 360, running both platforms side-by-side.
- Built `mapping/engagement-asset-inventory.md`: a reusable inventory template (field guide + blank table), then Northstar's filled inventory — 7 journeys, 5 automations, 5 Data Extensions, the content sets, the CloudPages preference centre, and 2 integrations, each with owner, entry, dependencies, logic weight, and risk notes.
- Built `mapping/journey-to-next-flow-mapping.md`: entry / decision logic / sends for every journey, the Next Flow / segment / Data Action equivalent, and a Moves / Stays / Retires verdict with checkable reasoning per asset.
- Day 1 result: **4 journeys move** (J-01 onboarding, J-02 card activation, J-04 savings/CD maturity, J-05 dormant re-engagement), **3 stay** (J-03 application nudge until its send-time lookup is simplified, J-07 service/transactional notices, CP-01 preference centre until the Day 2 consent sync design exists), **2 retire** (J-06 duplicate win-back on an expired offer, A-04 duplicate welcome entry path). Global suppression (A-05 / DE-05) is flagged as the gate: its logic must exist on the Data 360 unified profile before any Moves journey cuts over.
- Wrote `mapping/README.md`: the Day 1 method — inventory first, mapping second, verdict last — plus the verdict rules and the traps the inventory is designed to catch.
- Next: Day 2 — Data & consent on Data 360 (segments rebuilt from the Engagement definitions, preference/consent sync design, Data Action trigger specs).
