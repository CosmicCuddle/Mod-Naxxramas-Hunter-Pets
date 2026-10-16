# Source disagreements, availability and data-use restrictions

**Research only — not runtime rules.** Different evidence sources can disagree because of original patch changes, Classic Era reimplementations, datamining omissions, accidental NPC bugs, out-of-era trainer availability or simply an incomplete community list.

## Conflict C-001: How many Vanilla Hunter pet ranks?

- Our initial [Petopia-based reference table](../data/reference/vanilla-ability-ranks.csv) enumerates **111 rank records** for **21 abilities**, including ranks with no documented tame source.
- The independent [PetTips Classic addon](https://github.com/entrail/PetTips) README reports **21 abilities, 106 rank records and 297 teaching beasts**. Its author describes reconciling Petopia and client ability/trainer datasets, but it targets **Classic Era 1.15**, **not original 2006 patch 1.12 or our AzerothCore 3.3.5a client**.
- Difference: **five rank records**. This is a **documented research discrepancy**, not proof that five particular ranks should be removed, and not permission to copy a third-party dataset to see what fits.

**Required reconciliation method:** independently enumerate every Vanilla rank from original client/skill-line evidence and authoritative patch references. Match by canonical ability, rank number, source availability, training-point cost, pet family and spell ID, comparing historical version. Produce a row-by-row delta with source citations and a decision for inclusion/exclusion, *not* a forced 106 or 111 target. Preserve ranks that exist but are unobtainable where historically justified.

## Conflict C-002: Rank exists, no known source

[Classic Petopia's ability catalogue](https://www.wow-petopia.com/classic/abilities.php) says **Lightning Breath 1** and **Charge 4** lack known wild sources. A rank may exist client-side without any animal that teaches it. Do not grant it automatically and do not mislabel thirteen currently unresearched rank-source cases as impossible.

## Conflict C-003: Original caster-pet stat problem versus Classic Era

[Petopia caster pets](https://www.wow-petopia.com/classic/casterpets.php) documents older original-Vanilla inferior pet stats (especially on some Wind Serpents). Classic Era generally does not reproduce this historical defect. Implementation policy remains **unapproved**: deliberately recreating underpowered old pets may undermine the owner's desire for meaningful raid choices while recreating truly original behaviour.

## Conflict C-004: Era-specific ability mechanics and spell IDs

- Same spell name can be different in Vanilla/TBC/WotLK. Client Spell.dbc must match server to avoid tooltip/effect mismatch.
- Original per-creature attack cadence is **not** an unlimited DPS buff; WotLK normalises hunter-pet autoattacks.
- 1.9's `base pet speed standardized` wording needs scrutiny: movement speed and melee swing interval should be evaluated separately, not conflated.
- Classic Petopia source info may contain patch-later tameability while modern AzerothCore world DB retains other NPC flags.

## Data-rights policy

**PetTips [LICENSE.md](https://github.com/entrail/PetTips/blob/main/LICENSE.md)** specifically states its source and data may not be used in other projects without prior written permission and does not permit redistribution of modified derivatives. Therefore:
- Link to public repository and README for research only.
- Do **not** copy PetTips Lua data, generate our files from PetTips Lua records or fork/copy its data into Naxx Hunter Pets.
- Build our own factual reference tables independently from original patch sources, Classic Petopia and the owner's *own* AzerothCore/DBC, retaining source attribution.
- If direct code/data reuse ever becomes desirable, seek explicit licence permission and retain the evidence; absent permission treat it as out of scope.

[Petopia](https://www.wow-petopia.com/classic/abilities.php) also owns its website design and editorial work: link, cite and independently tabulate concise underlying facts; **do not duplicate large portions of guide prose, page markup, illustrations or site code**.

## Conflict review workflow

For any disputed row, save `conflict_id`, `claim`, `patch`, `source_A`, `source_B`, `current evidence`, `original-client verification`, `deployed 3.3.5 DBC verdict`, `owner decision ID`, `resolution status`. Keep disputed fields disabled for runtime until independently resolved. No "guess nearest spell" fallback.
