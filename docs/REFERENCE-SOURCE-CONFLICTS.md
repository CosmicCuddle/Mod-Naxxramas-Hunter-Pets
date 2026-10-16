# Source disagreements, availability and data-use restrictions

**Research only — not runtime rules.** Different evidence sources can disagree because of original patch changes, Classic Era reimplementations, datamining omissions, accidental NPC bugs, out-of-era trainer availability or simply an incomplete community list.

## Conflict C-001: Why 111 rank records but perhaps only 106 original Vanilla ranks?

- The [Petopia-based 111-row catalogue](../data/reference/vanilla-ability-ranks.csv) currently contains **59 wild ranks and 52 trainer ranks** across 21 ability names.
- The five excess rows appear to be **Rank 5 of Arcane, Fire, Frost, Nature and Shadow Resistance**. [Petopia's own 2019 maintainer audit](https://forums.wow-petopia.com/viewtopic.php?t=26093) says these could not be found on Classic trainers and might have been introduced during early Burning Crusade, despite being shown on its reference page.
- [Warcraft Wiki Arcane](https://warcraft.wiki.gg/wiki/Arcane_Resistance_(pet_ability)) and [Frost](https://warcraft.wiki.gg/wiki/Frost_Resistance_(pet_ability)) explicitly distinguish **four original Vanilla ranks** from **five Burning Crusade ranks**, corroborated by [archived Classic Hunter pet ability summaries](https://classic-wow-archive.fandom.com/wiki/Hunter_pet_abilities).
- A prior [PetTips GitHub README](https://github.com/entrail/PetTips) listed **106 ranks** (111−5). Its current [CurseForge description](https://www.curseforge.com/wow/addons/pettips) has reported 111. **Third-party count metadata is version-dependent**; neither can serve as original 1.12 proof, and the PetTips license restricts copying its dataset.
- The [archived CMaNGOS spell chains](https://github.com/cmangos/mangos-classic/blob/master/sql/archive/0.7/4096_pet.sql) contain all five disputed 2705x IDs, but **also contain confirmed later-expansion ranks**. A database entry is not proof of a trainer offering or original retail client compatibility.

**Proposed strict late-Vanilla interpretation, NOT owner-approved:** 59 wild-rank records + **47 trainer rank records** = **106 possible in-era rank records**. Two separate wild ranks have no known natural teaching source (Lightning Breath 1 and Charge 4), so **106 does not mean 106 naturally obtainable ranks**.

**Resolution still required:** a genuine **1.12.1.5875** Spell/SkillLineAbility DBC inspection and original-patch trainer offerings, including trainer wrapper IDs. Retain all 111 rows with historical status flags until conclusively resolved. See [complete trainer report](COMPLETE-TRAINER-RANK-RESEARCH.md).

## Conflict C-002: Rank exists, no known source

[Classic Petopia's ability catalogue](https://www.wow-petopia.com/classic/abilities.php) says **Lightning Breath 1** and **Charge 4** lack known wild sources. A rank may exist client-side without any animal that teaches it. Do not grant it automatically and do not mistake an incomplete local source list for an unobtainable rank. The 141-row sample now includes at least one candidate source for each of the other **57** wild-ranked entries.

## Conflict C-003: Original caster-pet stat problem versus Classic Era

[Petopia caster pets](https://www.wow-petopia.com/classic/casterpets.php) documents older original-Vanilla inferior pet stats (especially on some Wind Serpents). Classic Era generally does not reproduce this historical defect. Implementation policy remains **unapproved**: deliberately recreating underpowered old pets may undermine the owner's desire for meaningful raid choices while recreating truly original behaviour.

## Conflict C-004: Era-specific ability mechanics and spell IDs

- Same spell name can be different in Vanilla/TBC/WotLK. Client Spell.dbc must match server to avoid tooltip/effect mismatch.
- Original per-creature attack cadence is **not** an unlimited DPS buff; WotLK normalises hunter-pet autoattacks.
- 1.9's `base pet speed standardized` wording needs scrutiny: movement speed and melee swing interval should be evaluated separately, not conflated.
- Classic Petopia source info may contain patch-later tameability while modern AzerothCore world DB retains other NPC flags.

## Conflict C-005: Wild beast family vs ability whitelist

The [Petopia Claw Rank 5 list](https://www.wow-petopia.com/classic/abilities.php) includes a **Drywallow Snapper** labelled a **Crocolisk** as a tame-learning source, while Petopia's Claw ability-family whitelist excludes Crocolisks. This is a **research discrepancy**, not evidence that Crocolisks should universally gain Claw. Resolve actual source creature ID and its original `CreatureFamily`/skill data before adding this anomalous source. The current atlas does **not** include it as an approved mapping.

## Conflict C-006: Source creature level vs minimum training level

The [Bite Rank 6 list](https://www.wow-petopia.com/classic/abilities.php) includes **Barnabus (wild level 38)** although the listed **minimum pet training level is 40**. These fields represent different concepts. A developer must not automatically reject or alter historic tame-source evidence because the wild NPC's level is below the rank's training eligibility level. Validate what a just-tamed pet may initially know and when a second pet may be taught the rank.

## Conflict C-007: A 'Classic Playerbot' spell mapping is not original 1.12 DBC

The first [59-row candidate spell table](../data/reference/vanilla-wild-spell-id-candidates.csv) compares [CMaNGOS Classic bot Hunter spell presets](https://github.com/cmangos/playerbots/blob/master/playerbot/PlayerbotFactory.cpp) against [Petopia Classic ranks](https://www.wow-petopia.com/classic/abilities.php). **Eight** configured minimum pet levels disagree: Claw ranks 3–6, Furious Howl ranks 2–4, and Screech rank 3. [Claw 3](https://wowclassicdb.com/spell/16829) and [Screech 3](https://www.wowhead.com/classic/spell=24578) independently support the Petopia level candidates, not the bot map. This demonstrates the difference between **bot skill provisioning heuristics** and historically accurate player training requirements.

**Never** bulk-import these levels. Original 1.12 client/SkillLineAbility and independent rank guide must drive the authoritative spec.

## Conflict C-008: Screech 27051 is a dangerous cross-era spell-ID collision

The same CMaNGOS Classic bot mapping assigns **24579** to some Screech Rank 4 families and **27051** to another. Independent spell records identify [24579](https://www.wowhead.com/classic/spell=24579) with the **Vanilla level-56 Rank 4** effect, but [27051](https://www.wowhead.com/tbc/de/spell=27051) with a **level-64 TBC variant**, and [WotLK 27051](https://www.wowhead.com/wotlk/spell=27051) with **Demoralizing Screech**. **27051 must not be treated as Vanilla Screech Rank 4**. Client and server 3.3.5 DBC still need verification for safe handling of 24579.

## Conflict C-009: Old pet passives versus WotLK same-name effects

[Classic Great Stamina 4187](https://www.wowhead.com/classic/spell=4187) is a **flat +3 Stamina** training spell, while [WotLK Great Stamina 61687](https://www.wowhead.com/wotlk/spell=61687) raises Stamina by **8%**. Name matching would silently inflate or alter stat maths. Native 3.3.5 pet passive/talent interactions need source and in-game checks. See [trainer comparison](TRAINER-SPELL-IDENTITY-REFERENCE.md).

## Data-rights policy

**PetTips [LICENSE.md](https://github.com/entrail/PetTips/blob/main/LICENSE.md)** specifically states its source and data may not be used in other projects without prior written permission and does not permit redistribution of modified derivatives. Therefore:
- Link to public repository and README for research only.
- Do **not** copy PetTips Lua data, generate our files from PetTips Lua records or fork/copy its data into Naxx Hunter Pets.
- Build our own factual reference tables independently from original patch sources, Classic Petopia and the owner's *own* AzerothCore/DBC, retaining source attribution.
- If direct code/data reuse ever becomes desirable, seek explicit licence permission and retain the evidence; absent permission treat it as out of scope.

[Petopia](https://www.wow-petopia.com/classic/abilities.php) also owns its website design and editorial work: link, cite and independently tabulate concise underlying facts; **do not duplicate large portions of guide prose, page markup, illustrations or site code**.

## Conflict review workflow

For any disputed row, save `conflict_id`, `claim`, `patch`, `source_A`, `source_B`, `current evidence`, `original-client verification`, `deployed 3.3.5 DBC verdict`, `owner decision ID`, `resolution status`. Keep disputed fields disabled for runtime until independently resolved. No "guess nearest spell" fallback.
