# Day 1 method — inventory, then mapping

This folder is Day 1 of the Marketing Cloud Next Migration Playbook for the fictional *Northstar Credit Union*. It contains a method, not just Northstar's answers — the same two steps work on any Marketing Cloud Engagement org.

## The order of work (do not swap it)

1. **Inventory first.** List every asset before judging any of it. An asset nobody listed still sends, still suppresses, or still feeds a journey — the inventory is where those surprises surface cheaply.
2. **Map second.** For each journey, write down today's entry, decision logic, and sends in plain language *before* naming a Next equivalent. If today's logic cannot be written plainly, the asset is not understood well enough to rebuild.
3. **Verdict last, per asset.** Moves / Stays / Retires, with reasoning a second person can check.

## The verdict rules used for Northstar

**Moves** when all of these hold:
- Entry is segment-driven or a clean lifecycle event (account opened, card activated, maturity date reached) that Data 360 can hold or receive
- Decision logic is mostly audience / attribute splits, waits, and sends — not heavy per-send scripting
- The data the journey needs either already lands in Data 360 or has a named source that can
- Consent and suppression for its audience can be expressed on the unified profile (Day 2 designs this)

**Stays** when any of these hold:
- The pattern is transactional / service messaging tightly coupled to Engagement send infrastructure
- It depends on a CloudPage, AMPscript, or SSJS behaviour with no honest Next equivalent yet — the preference centre is the textbook case
- Its logic is so tangled that rebuilding it now would copy the tangle; simplify in place first, then re-judge
- Every Stays row must name the condition that would change the verdict later. "Stays forever, no reason" is not allowed.

**Retires** when any of these hold:
- Another asset already does the same job (duplicate entry path, overlapping journey)
- It was built for a campaign or product that no longer exists
- Nobody can name its owner or its last meaningful change, and pausing it harms nothing
- Every Retires row names what replaces it, or states that nothing needs to.

## Traps this method is designed to catch

| Trap | How the inventory catches it |
|---|---|
| Duplicate entry into the same journey (automation + API + manual import) | The inventory has one row per *entry path*, not one per journey name |
| Suppression that lives in a single SQL query nobody documented | Dependencies column: every journey row must name the queries / DEs its audience depends on |
| A journey that looks simple but personalises with deep AMPscript lookups | The mapping table records decision logic *and* send logic separately — script-heavy sends are visible before a Moves verdict |
| Consent captured on a CloudPage that nothing else reads | The preference centre is inventoried as an asset with downstream dependencies, not as "just a page" |
| Zombie automations still refreshing DEs for retired campaigns | Last-change and volume-signal columns; an automation feeding nothing live is a Retires candidate |

## Files

- `engagement-asset-inventory.md` — the blank template + field guide, then Northstar's filled inventory
- `journey-to-next-flow-mapping.md` — the mapping table and per-journey reasoning
