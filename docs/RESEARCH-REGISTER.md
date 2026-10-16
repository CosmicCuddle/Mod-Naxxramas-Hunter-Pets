# Research register and historical data sources

**Status:** Open evidence log. Do not treat this list as an implemented database, or as proof that a specific Vanilla patch's data exists in AzerothCore 3.3.5a.

## Source hierarchy

1. Historical Vanilla 1.12 patch data or credible original research, verified against independently documented examples.
2. Classic Era archival/recreation sources, with version differences explicitly identified.
3. Current AzerothCore C++ and SQL, for **implementation behaviour**, not presumed Vanilla authenticity.
4. Observations from the owner's actual server and client test build (record SHA/config), with repeatable steps.

Do not combine original 2004–2006 Vanilla, Classic Era, TBC Classic and WotLK abilities into one unsourced table. Each row must specify *era / patch* and *confidence*.

## Researched starting references

- Classic Petopia — abilities and where to tame ranks: https://www.wow-petopia.com/classic/abilities.php
- Classic Petopia — learning from tamed beasts, trainer skills, four-active cap and training point formula: https://www.wow-petopia.com/classic/training.php
- Classic Petopia — version differences and families: https://www.wow-petopia.com/classic/differences.php
- Wowhead — Classic Hunter pets / specific pet use cases: https://www.wowhead.com/classic/guide/classic-wow-hunter-pets
- Warcraft Wiki — training points removed in patch 3.0.2: https://warcraft.wiki.gg/wiki/Training_point
- AzerothCore `Pet.cpp` for normalised 3.3.5a attack speed, happiness and auto-learning: https://github.com/azerothcore/azerothcore-wotlk/blob/master/src/server/game/Entities/Pet/Pet.cpp
- AzerothCore PetScript hook definitions: https://github.com/azerothcore/azerothcore-wotlk/blob/master/src/server/game/Scripting/ScriptDefines/PetScript.h
- Individual Progression upstream: https://github.com/ZhengPeiRu21/mod-individual-progression
- IP reported `HunterPetLevelFix` issue (not proof about installed version): https://github.com/ZhengPeiRu21/mod-individual-progression/issues/423
- Naxxramas Core roadmap: https://github.com/CosmicCuddle/Mod-Naxxramas-Core/blob/main/ROADMAP.md

## Research datasets required before feature completeness can be claimed

| Dataset | Minimum columns / validation |
|---|---|
| Creature taming source | creature entry, name, map/location, Vanilla phase availability, family, tamable? level, source and confidence |
| Wild-known skills | creature entry, spell ID, canonical ability, rank, learns-by-use trigger, source and patch |
| Full skill ranks | canonical ability, rank, exact spell ID in **deployed** DBC, level requirement, TP cost, family masks, source |
| Trainer skills | trainer NPC(s), skill/rank, level requirement, money cost, prerequisite/source |
| Pet family | family ID, diet, armour/HP/damage profile, base abilities, unique behaviours, source |
| Exceptional pets | creature entry, attack interval, damage normalization, special school/behaviour, patch boundaries |
| Loyalty/happiness | rank thresholds, time/XP logic, food gains, decay, damage modifiers, abandonment rules |
| Pet XP/stables | tame level handling, XP thresholds, level restrictions, available stable capacity and UI |
| Interface/protocol | what 3.3.5 client can display without DBC or executable patch; addon feasibility |
| Save/reload | native table names/keys and script hook lifecycle on deployed branch |

Every row must carry a reliable citation and be reviewed before merging into production configuration or SQL.

## WotLK automatic rank suppression feasibility — no strict module-only solution proven

- [Auto-spell learning feasibility report](AUTO-SPELL-LEARNING-FEASIBILITY-DECISION.md) shows current public **SpellMgr** loads Hunter family rank maps from 3.3.5 `CreatureFamily` + `SkillLineAbility` DBC into **global** read-only-exposed lists; Pet::learnSpell grants direct without per-owner learning veto. Editing global data/shared maps is not an approved or era-safe module mechanism.
- [16 interception candidates](../data/reference/auto-spell-interception-candidates.csv) compare true pre-learn veto (absent), `AllSpellScript::OnSpellCheckCast` cast-denial (candidate only), public `Pet::isBeingLoaded` and PlayerUpdate after-load observation (too late for pet spellbook), passive effects and rejected unsafe hacks.
- [WP-03 staging protocol](WP03-AUTO-SPELL-LEARNING-STAGING-PROTOCOL.md) specifies backup-first, read-only source+client/DBC verification, logging-only source timings and later explicit approval of any disposable staging mutation, plus client/passive/Playerbot/IP edge cases.
- [Dedicated issue #4](https://github.com/CosmicCuddle/Mod-Naxxramas-Hunter-Pets/issues/4) tracks this hard research gate. **No owner approval, implementation or staged test so far.**

## Engine event-order and module-only hard gates — public source audit

- [Source-pinned pet lifecycle audit](SOURCE-PINNED-PET-LIFECYCLE-AUDIT.md) and [16-event call-order index](../data/reference/public-core-pet-lifecycle-events.csv): new tame spell init **before** OnPetAddToWorld, existing pet async saved-spell load **after** OnPetAddToWorld, WotLK auto pet level catchup in tame/load, and native pet save split into independently committed transactions.
- [Public Individual Progression source](https://github.com/ZhengPeiRu21/mod-individual-progression/blob/master/src/IndividualProgressionPlayer.cpp) still awards capped level-60 Hunters' pets kill-XP in `OnPlayerGiveXP`, then sets the player's XP to zero. This creates a possible **double/missed pet XP** integration risk if a second module tries to restore authentic Vanilla pet growth. Installed IP revision and exact bots remain unverified.
- **No safe generic CanLearnPetSpell veto** was found in the inspected public `PetScript.h`; `OnPetAddToWorld` cannot be assumed a complete 'pet ready after DB spells' hook. **No module-only solution has been demonstrated**.
- 3.3.5 `SpellScript` exposes per-target `AfterHit` and `OnEffectHitTarget` hooks worth evaluating; these are only **promising discovery-logic candidates**, not proof of successful learning.

## Trainer-rank research — all 52 candidates indexed

- [Complete pet trainer rank reference](COMPLETE-TRAINER-RANK-RESEARCH.md) now covers all **52** initially enumerated trainer ranks and expected Growl, Great Stamina, Natural Armor and school-resistance values, linked to [52-row candidate spell-ID CSV](../data/reference/vanilla-trainer-spell-id-candidates.csv).
- **New important historical conflict:** five resistance **Rank 5s** are listed by Petopia but appear **not offered during late Vanilla**. Original Vanilla Wiki records four ranks versus five BC ranks; [Petopia maintainer investigation](https://forums.wow-petopia.com/viewtopic.php?t=26093) independently reports missing trainer offerings. **47** trainer ranks are the provisional strict Vanilla subset; **not yet verified with original 1.12 files**.
- [111-rank combined research crosswalk](../data/reference/vanilla-all-rank-id-candidate-crosswalk.csv) now joins all **59 wild + 52 trainer** entries with provisional ID and risk. Five disputed Rank 5s remain clearly marked, **not deleted**; Charge 4 and Lightning Breath 1 separately have no known taming source.
- [Original DBC ↔ actual 3.3.5 verification protocol](ORIGINAL-DBC-AND-DEPLOYED-COMPARISON.md) specifies lawful original 1.12.1.5875 DBC analysis, trainer wrapper spell checks, actual server+client DBC hashes and IP/Playerbot audits.
- A read-only check of **public AzerothCore `spell_ranks.sql`** found **11 of 52** candidate trainer IDs in that SQL file. This **does not** test whether the other 41 IDs exist in real Spell.dbc. We have **not accessed the deployed core, acore_world, owner character database or game DBC**.
- [CMaNGOS bug report #3817](https://github.com/cmangos/issues/issues/3817) reports stamina bonuses not increasing total HP in one emulator revision; this is a **test hypothesis**, not a finding on this server.

## Spell-ID research progress

- [59 rank-by-rank wild spell candidates](../data/reference/vanilla-wild-spell-id-candidates.csv) were assembled as an **independent technical cross-check**, drawing from the CMaNGOS Classic Playerbot family mapping with Petopia as rank-level reference. All remain **unverified against original 1.12 and deployed 3.3.5 DBC**.
- [Spell-ID and effect audit](SPELL-ID-AND-EFFECT-RESEARCH.md): **eight** ability rank-level mismatches in the bot code; Screech 4 has two candidate IDs, one of which (**27051**) is a confirmed out-of-era Screech variant. Do not use this ID as Vanilla Rank 4.
- [Trainer spell identity comparison](TRAINER-SPELL-IDENTITY-REFERENCE.md): three independently sourced examples from **52** trainer rank records, including the flat Classic Great Stamina versus percentage WotLK effect. Trainer wrapper and pet-cast spell may use distinct IDs.
- [13 pilot spell-effect reference fixtures](../data/reference/vanilla-pilot-spell-effects.csv) cover all wild ability names, with cited rank levels/costs and selected effect values. None of the effect rows is original 1.12 or deployed 3.3.5 DBC verified.
- The [source index](../data/reference/historical-source-index.csv) now has **50** annotated research references, including targeted spell database records and tools for lawful original DBC extraction.
- No 3.3.5 runtime/client spell assets have been inspected; **ID candidates are not implementation mappings**.

## Creature-to-rank research progress

- [Tame-source atlas](TAME-ABILITY-SOURCE-ATLAS.md): **141** curated Classic Petopia beast/ability/rank reference rows. Coverage: **57** of 59 wild-rank combinations with examples; **two** with no known wild source. Candidate NPC IDs only for six named beasts; no deployed AC/IP/client validation.
- [Rank-by-rank coverage status](../data/reference/vanilla-rank-source-coverage.csv): distinguishes trainer skills, sampled wild skill sources, historically unknown wild sources and remaining runtime mapping tasks.
- [Reference conflicts and data licensing](REFERENCE-SOURCE-CONFLICTS.md): independent 111 vs 106 rank count, caster-pet stat discrepancy, Claw/Crocolisk source anomaly, and constraints on reuse of PetTips data.

## Known examples for verification, not yet authoritative implementation data

- **Son of Hakkar:** Lightning Breath Rank 6 as a tame-discovered rank, transferable to eligible Wind Serpents after Hunter learning.
- **Broken Tooth:** 1.0-second historical melee attack cadence; verify damage-per-swing scaling, PvP pushback and template ID.
- **Lupos:** early Vanilla Shadow-damage behaviour removed before late Vanilla; only implement if owner approves an early-patch exception.
- **Furious Howl / wolves:** investigate whether and why specific tames matter for top-rank acquisition.
- **Rare vs elite:** do not conflate spawn rarity, elite creature combat stats and tamed-pet performance.

## Specific source gaps and authenticity conflicts

- **Charge Rank 4** is listed in [Classic Petopia](https://www.wow-petopia.com/classic/abilities.php) as a level-36, 17-TP spell with **no known training source**. Preserve this distinction: the rank's existence does not prove that a Hunter can earn it from a specific beast. Do not invent a source entry.
- **Research table costs:** the 111 candidate rank rows now all have training-point costs transcribed from the Petopia Classic catalogue. This does **not** verify original 1.12 values or any 3.3.5 spell ID; a complete independent audit remains required.
- **Classic vs original Vanilla:** Petopia notes that some inferior original 'caster' pets were bugged and that Classic Era does not reproduce that bug. Decide whether authentic end-state or bug-for-bug fidelity is intended; the late-Vanilla 1.12 baseline remains **provisional**.
- **Traceability:** [54 initial requirement-to-work-package-to-test links](../data/reference/requirement-traceability.csv) are planning references, not confirmation of gameplay support.

## Open research questions — must answer before broad coding

1. Which IP branch is actually installed, and does it contain/enable Hunter-specific pet changes today?
2. Can module-only hooks reliably suppress automatic pet skill granting **at all stages** (taming, level, load, talent reset, pet save)?
3. Does 3.3.5 retain all original skills and rank spell data, or are **DBC/server+client** additions required?
4. Can the original pet's attack speed be restored with correct DPS normalization under all damage/haste buffs?
5. Will classic *four-active-skill* rules conflict with mandatory built-in WotLK pet actions?
6. How is the owner-wide known ability catalogue persisted separately from per-pet spells?
7. What transitions will affect existing pets if the owner later unlocks TBC?
8. How will Random bots and Alt Bots legally acquire usable Vanilla pet skills without bypassing the Hunter's intended gameplay?
9. Will classic stable limits or a classic-looking Beast Training window be achievable without client work? What compromises are acceptable?
10. How do happiness, pet talent points and source-IP levelling already interact with Naxx Core's bot consumables?

## Provenance convention

Use a CSV or Markdown table, with `patch_reference`, `source_url`, `checked_on`, `evidence_type`, `confidence`, `deployed_spell_id`, `verified_in_game` columns. Never silently resolve conflicting historical sources by choosing the most convenient result.
