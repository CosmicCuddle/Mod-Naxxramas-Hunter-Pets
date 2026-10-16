# Historical reference foundation — maximum feasible Vanilla Hunter pet restoration

**Research edition:** 16 October 2026. **Status:** Evidence and design inputs only. No code, DBC or SQL authorisation.

## Owner's actual objective

Bring back **as much of the original Vanilla Hunter pet system as can be faithfully and safely restored** on AzerothCore 3.3.5a, without upstream source edits. Research comprehensively; do not shrink the target to only famous pets or a few mechanics. If a feature cannot be restored exactly through an isolated module, prove and document the obstacle, propose a faithful alternative and defer the trade-off for owner approval.

**Era boundary:** the resulting Vanilla rules apply throughout every Vanilla Individual Progression stage. Whether later era mechanics should replace them is deferred to TBC planning.

**Historical baseline recommendation, not owner approval:** start with a coherent *late original Vanilla* ruleset; research and preserve earlier original Vanilla variants separately so the owner can eventually decide whether some historical exceptions improve the experience. Modern Classic Era references are useful but are **not interchangeable** with 2004–2006 Vanilla.

## Evidence ranking and how to use it

| Tier | Type | Use | Limitation |
|---|---|---|---|
| 1 | Original Blizzard-era patch notes and authentic versioned client/database evidence | Establish what changed and when; spell/stat/tame data for the selected patch | Archived copies should be checked for transcription and patch context |
| 2 | Historical pet research maintained by Petopia and independent Classic guides | Rank lists, tame-source animals, families, feeding, original individual pet speeds | Modern Classic may omit original retail bugs and patch-specific quirks |
| 3 | Classic emulator source (CMaNGOS Classic or comparable) | Understand data storage, loyalty ticks, teach-spell logic and original protocol behaviour | Working emulator logic isn't automatically retail-perfect; never transplant upstream code |
| 4 | Current AzerothCore/IP/Playerbots source and **deployed** world/client DB/DBC | Determine which APIs, creature IDs, spell IDs and in-game behaviours exist | 3.3.5 is not historical Vanilla; code in upstream default branch might not match installed revision |

Per assertion record source URL, relevant passage or symbol, original patch, checked date, secondary corroboration, conflicts, confidence, mapping to actual server, and acceptance-test ID. The research data remains **non-executable**.

## Core gameplay source set

| Reference | Purpose | Observations relevant to scope |
|---|---|---|
| [Petopia: Differences in Classic](https://www.wow-petopia.com/classic/differences.php) | Complete system overview | 17 families, no pet talent trees, 1 active + 2 stable, retain tame level, loyalty/happiness, rank teaching, individual family profiles |
| [Petopia: How to Train](https://www.wow-petopia.com/classic/training.php) | Learning and training rules | Level-10 quest chain, teach-by-pet-use, trainer versus wild skills, 4 active maximum, `level × (loyalty-1)` TP, 0-TP Growl, rank refund and negative TP |
| [Petopia: Every pet ability](https://www.wow-petopia.com/classic/abilities.php) | 21 named skills and 111 indexed ranks | Ability rank requirements, TP cost and **known wild creature sources**; existence of rank does **not** always imply a tame source |
| [Petopia: Native attack speeds](https://www.wow-petopia.com/classic/attackspeed.php) | Rare-pet distinctions | Broken Tooth, ZG bats, Wolves, Cats etc. with individual native attack rates; same-family fast attacks are not free overall DPS |
| [Petopia: historical caster pets](https://www.wow-petopia.com/classic/casterpets.php) | Original Vanilla vs modern Classic discrepancy | Original caster-category creatures sometimes had weaker health/armor/DPS, often Wind Serpents; modern Classic did **not** retain the issue |
| [Petopia: 17 individual family pages](VANILLA-FAMILY-REFERENCE.md) | Family roles and diets | Family multipliers, allowed active skills, food, appearance diversity; verify against original version and actual 3.3.5 rules |
| [Wowhead Classic Hunter Pets](https://www.wowhead.com/classic/guide/classic-wow-hunter-pets) | Secondary cross-check | Classic pet training, 300 TP at max level/loyalty and rank/tame sources |
| [Warcraft Wiki: Pet Happiness](https://warcraft.wiki.gg/wiki/Happiness) | Damage and happiness | Reported happiness damage multipliers 75/100/125%; compare with AzerothCore, verify patch |
| [Warcraft Wiki: Loyalty](https://warcraft.wiki.gg/wiki/Loyalty) | Historical rank/XP/time details | Six ranks and progression; do not import timing formula without independent check |
| [Warcraft Wiki: Training points](https://warcraft.wiki.gg/wiki/Training_point) | Original TP and WotLK removal | TP system replaced with talents at patch 3.0.2; source interleaves Vanilla and TBC examples |
| [Warcraft Wiki: 3.3.5 client API GetPetHappiness](https://warcraft.wiki.gg/wiki/API:GetPetHappiness) | Interface feasibility | Historical API returned happiness, damage percent and loyalty rate, but 3.3.5 availability/meaning must be tested |
| [Blizzard: Restoring History for WoW Classic](https://worldofwarcraft.blizzard.com/en-gb/news/22646759) | Primary retrospective design context | Blizzard explicitly discussed the difficulty of reinstating removed Hunter systems; not source for raw pet spell IDs |

## Original patch-change evidence: why researching all Vanilla matters

### Patch 1.7.0 — customisation and skill ecosystem

Archived [patch 1.7 notes](https://warcraft.wiki.gg/wiki/Patch_1.7.0), independently reproduced in [2005 historical press coverage](https://worthplaying.com/article/2005/9/14/news/27376-world-of-warcraft-v170-euro-patches-available-now/), document:
- Major Hunter pet customization: trainable armor, stamina and school resistance passives.
- Ability untraining/reset, escalating pet trainer charges, **four-active ability limit**, Growl free of TP.
- Hunter knowledge retained when previously trained pet ranks were reset.
- Screech, Prowl, Scorpid Poison, Furious Howl and particular family abilities.
- Pet XP determined by target level relative to pet, with qualification based on Hunter's own kill XP.
This original patch is important evidence for the *interrelationship* between acquired Hunter knowledge, individual pet spell lists, respec cost, TP and pet XP. It is more authoritative for the patch than a later emulator implementation.

### Patch 1.9.0 — new taming options and named ranks

The original [1.9 patch notes archive](https://warcraft.wiki.gg/wiki/Patch_1.9.0) and [contemporary patch copy](https://www.wowinterface.com/forums/showthread.php?t=2763) record:
- **Sons of Hakkar became tameable**, providing **Lightning Breath Rank 6**.
- **Boar Charge**, **Gorilla Thunderstomp**, **Turtle Shell Shield** added/changed.
- Hunter pet base melee attacks became **Physical damage**; earlier Lupos Shadow-damage exception stopped.
- 'Base pet speed standardized' wording is present. **Important unresolved distinction:** this is often interpreted as *movement speed*, not individual melee attack intervals; Classic Petopia and subsequent Hunter discussions still document Broken Tooth's 1.0-second attack speed. Do not mark original melee speed normalized in 1.9 without a version-specific source/call-graph check.

### Additional original Vanilla patch references

- **1.5:** improved following movement speed and Growl Rank 1, battleground pet resurrection and dead-stable pet fixes. [Original 1.5 notes](https://warcraft.wiki.gg/wiki/Patch_1.5.0).
- **1.8:** introduced Wind Serpent Lightning Breath. [Original 1.8 notes](https://warcraft.wiki.gg/wiki/Patch_1.8.0).
- **1.10:** revised Bite/Claw threat, introduced high-level Charge 6 source boars, revised Shell Shield melee slow/cooldown, altered Frenzy proc conditions. [Original 1.10 notes](https://warcraft.wiki.gg/wiki/Patch_1.10.0).
- **1.11:** altered Scorpid Poison duration and corrected Natural Armor stacking with family armor modifiers. [Original 1.11 notes](https://warcraft.wiki.gg/wiki/Patch_1.11.0).
- **1.12:** corrected Growl interaction, Ferocity for magic pet abilities, and original model/color preservation for named rare/dungeon creatures on resummon. [Original 1.12 notes](https://warcraft.wiki.gg/wiki/Patch_1.12.0).

### Later engine changes

- The [WotLK Classic differences](https://www.wow-petopia.com/classic_lk/differences.php) document removal of learned pet training and loyalty, introduction of talent trees and automatic family spells, additional stable slots and 5-level tame catch-up.
- Need to separate original patch **2.0 family additions** from late pre-2.0 Vanilla. Serpents were added in patch 2.0; do not merge family lists even though the 2.0 prepatch was geographically 'before TBC release'.
- Historical pet changes **within Vanilla** do not imply the project must dynamically simulate each old patch. That is a separate owner decision. The complete record matters so optional differences remain available.

## Original exceptional pet properties — preserve references even if optional

- **Son of Hakkar:** verified by original 1.9 notes and Petopia as tame source for LB 6. Proven historically meaningful acquisition example.
- **Broken Tooth:** Petopia lists 1.0-second attack speed; apply proper same-family normalized damage. Compare current 3.3.5 Pet::InitStatsForLevel before any code.
- **Bloodseeker Bat / Frenzied Bloodseeker Bat:** also 1.0-second speed in Zul'Gurub, proof that fast attacks weren't a unique 'cat-only' property.
- **Lupos:** Shadow physical-base exception in early Vanilla; **no longer active after 1.9**. Track as explicitly optional historical variant, not an automatic 1.12 feature.
- **Historic inferior caster pets:** some original creatures (e.g. old wind serpent variants) had worse armor/health/DPS. Modern Classic deliberately doesn't reproduce this problem. Document as a historical defect and let owner decide whether exact bug reproduction is worthwhile, avoiding accidental nerfs.
- **The Snarler / special resistances:** community anecdotes exist, but source and precise patch/shape are **unverified**; do not promise or implement on anecdote alone.

## What a complete reference package eventually contains

1. **Every Vanilla creature worth taming**, with original NPC ID, location, family, level, spawn/tame availability, rare/elite properties, innate abilities and cast rank, original attack speed, source URL/patch and current deployed NPC/DBC mapping.
2. **Every source and rank for all 21 named skills**, including family restrictions, focus/cooldown/effect, training-point cost, money cost if trained, natural learn-by-use source, unavailable ranks and 3.3.5 client spell mapping.
3. **Loyalty/happiness/feeding/XPs**, including tick/threshold math, initial tame, death/dismiss/stable, correct accepted food and effectiveness.
4. **Pet identity and family stats**, original attack and movement speed, health, armor, damage, school/range, cross-era interaction, skill-learning exceptions.
5. **Talent/ability interaction**, WotLK pet talent replacement, Hunter Beast Mastery talent, fixed passive inheritance, IP power scaling, original raid party effects and Playerbot AI choices.
6. **Implementation feasibility & tests**, exact deployed AzerothCore extension points, staged read-only schema audit, module-only GO/NO-GO, rollback safety and support for every Vanilla IP progression stage.

See the [unified roadmap](UNIFIED-BLUEPRINT-AND-ROADMAP.md), [evidence standards](RESEARCH-EVIDENCE-STANDARDS.md) and [traceability dataset](../data/reference/requirement-traceability.csv).

**No gameplay code has been implemented or tested.** The next objective is completeness and reliability of evidence, rather than assuming a large reference library automatically equals a working Hunter module.
