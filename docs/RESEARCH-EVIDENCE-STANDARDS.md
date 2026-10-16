# Historical research standards and source-verification protocol

**Status:** research process specification. The aim is maximum **accuracy and reproducibility**, not filling in unknown fields for appearance.

## 1. Separate four truths

| Kind | Example | What it supports |
|---|---|---|
| Original retail Vanilla (patch-specific) | Archived 1.12 client/world data, contemporary original records | Historical authenticity for a defined patch |
| Modern Classic Era reconstruction | [Petopia Classic](https://www.wow-petopia.com/classic/) | Strong gameplay catalogue; must still check discrepancies and retail version |
| Classic emulator implementation | [CMaNGOS Classic](https://github.com/cmangos/mangos-classic) | Design insight into loyalty, trainpoint, teachspells, spell/cast/DB logic; **not retail proof** |
| Actual deployed AzerothCore 3.3.5a | Pin precise compiled core revision, DBC, world and characters DB | What hooks and IDs work on this server, **not historical authority** |

An unmodified WotLK spell called 'Lightning Breath' is **not necessarily** the same spell mechanically as its 1.12 counterpart. An old CMaNGOS probability in source code is **not** proof Blizzard originally used those exact odds.

## 2. Confidence labels (apply per claim and CSV row)

- `OWNER_REQUIREMENT`: specifically desired user experience; doesn't imply historical accuracy.
- `HISTORIC_CANDIDATE`: supported by one known source, not verified against exact target patch.
- `CORROBORATED`: two independent gameplay references with compatible patch meaning.
- `ORIGINAL_DATA_VERIFIED`: directly cross-checked against authentic versioned original data, source/record SHA documented.
- `RUNTIME_MAPPED`: mapped to exact deployed creature/spell/client IDs.
- `MODULE_SPIKE_PASSED`: actual effect proven on staged server using only module code and reproducible evidence.
- `ACCEPTED`: complete QA and explicit owner decision.

`NO_KNOWN_SOURCE`, `SOURCE_CONFLICT`, `DBC_MISSING`, `CLIENT_LIMITATION`, `MODULE_HOOK_MISSING`, `OUT_OF_ERA` are separate **blocker flags**, not invitation to guess.

## 3. Data-entry contract

Required, where relevant: `canonical_ability`, `rank`, `min_pet_level`, `trainpoint_cost`, `trainer_or_wild`, `eligible_family`, `pet_source_creature_entry`, `source_zone`, `original_patch`, `classic_spell_id`, `deployed_spell_id`, `source_url`, `source_revision_or_accessed_date`, `confidence`, `blocked_reason`, `human_checked_by`, `runtime_verified`.

Original creature entries must come from an independently grounded creature database or verified exact NPC pages; their **names are not unique IDs**. The 3.3.5 `creature_template` entry of a beast and old version’s entry should be checked rather than presumed equal.

Do **not** put guessed IDs in live SQL. Missing fields must remain empty/unknown with an explicit reason, and row validation should fail closed.

## 4. Verified research from Classic Petopia (October 2026)

### Ability training

- **21 named abilities**: 13 obtained through wild-pet taming and 8 from trainers. The [ability page](https://www.wow-petopia.com/classic/abilities.php) lists 111 rank/level entries across the 21 names. This is a **current Classic Petopia candidate catalogue**, not proof that the original 1.12 client has an identical set of binary spell IDs.
- Per-rank training costs were initially incomplete in `data/reference/vanilla-ability-ranks.csv`. On 2026-10-16 the 13 omitted costs were extracted and filled from the Petopia ability page: **Charge 2–6: 9, 13, 17, 21, 25; Claw 1–3: 1, 4, 7; Cower 4–6: 14, 16, 18; Screech 2: 15; Thunderstomp 3: 25 TP**. Full 111-row comparison against original data and DBC remains pending.
- **Special exception:** [Charge 4](https://www.wow-petopia.com/classic/abilities.php) is listed at **pet level 36 / 17 TP**, but Petopia explicitly says **'No known training source.'** Until evidence resolves that, rank existence and rank teachability must be recorded as *different fields*. Do not invent a source NPC or offer the rank automatically.
- **TP formula:** pet level × (loyalty − 1); loyalty 6, level 60 = 300. [Training reference](https://www.wow-petopia.com/classic/training.php).
- Pet may start with **negative TP** from naturally possessed skills, and old ranks refund costs on upgrade; max **four active** skills; Growl costs zero. [Training reference](https://www.wow-petopia.com/classic/training.php).
- Beast-knowledge progression requires tame→*skill used by pet*→Hunter knows→other eligible pet trained. A source beast initially knowing an ability does not automatically teach the Hunter.
- Original Vanilla includes **17 family categories** and **three pets owned max: one active, two stabled**; enforcing three on existing 3.3.5 realm is **not approved**. [Classic differences](https://www.wow-petopia.com/classic/differences.php).
- Petopia notes original-era inferior **'caster pet'** stats were a bug *not recreated in Classic Era*. Therefore current Classic Era family data is **not** fully faithful to every old 2004–2006 bug. Decide exact 1.12 baseline and whether to replicate any historical bugs **before** coding.
- Faster attack cadence does not mean higher same-family baseline DPS: swings do proportionately less damage. [Attack speed reference](https://www.wow-petopia.com/classic/attackspeed.php).

### Comparing CMaNGOS to current AzerothCore

- CMaNGOS Classic pet implementation includes loyalty, a TP ledger, stored teaching spells and a use-triggered learning mechanism. It is an **implementation candidate**, never a direct port or proof of retail probabilistic odds. [CMaNGOS Pet.cpp](https://github.com/cmangos/mangos-classic/blob/master/src/game/Entities/Pet.cpp).
- AzerothCore `Pet::InitLevelupSpellsForLevel` directly auto-teaches rank/default skills, no generic veto identified in `PetScript.h`; module-only **feasibility must be proven first**. [AzerothCore Pet.cpp](https://github.com/azerothcore/azerothcore-wotlk/blob/master/src/server/game/Entities/Pet/Pet.cpp), [PetScript.h](https://github.com/azerothcore/azerothcore-wotlk/blob/master/src/server/game/Scripting/ScriptDefines/PetScript.h).

## 5. Do not conflate ability *existence* with ability *availability*

Every rank/source pair needs:
- `rank_exists_in_ruleset`: is rank defined?
- `known_wild_sources`: original animals recorded as teaching it, possibly **empty**.
- `source_accessible_in_ip_stage`: Hunter can actually reach tame target under Individual Progression.
- `tameable_in_current_world`: deployed NPC flags and rules allow tame.
- `effect_supported_in_335`: specific Spell.dbc/server script behaviour can be reproduced.
- `learnable_by_hunter`: teach event reliably occurs through module.
- `trainable_on_pet`: level/TP/family and native persistence all satisfy requirements.

A value can be historically *listed* but unavailable, as Charge 4 demonstrates.

## 6. Structured validation, not broad assertions

For each ability rank:
1. Compare source page rank level, cost, spell effect, eligible families, source creature names and zone.
2. Cross-check against a separate historical source or original versioned client data; record any differences.
3. Confirm original pet source's spawn/creature ID against actual world DB.
4. Confirm required ability rank spell ID and effect from exact deployed 3.3.5 server DBC and client patch.
5. Confirm every permitted family/level/TP combination in mock and game-level tests.
6. Check spell does not auto-appear when pet levels, summons, loads or stables.
7. Save two evidence artifacts: research justification and actual runtime test outcome.
8. Update requirement traceability and a specific QA identifier.

For each family: diet, multipliers, tame eligibility, original skills, available appearances, IP stages, special beast attack times and PvP/raid effect. Do not claim that a **17-family name list** is a finished source-creature database.

## 7. Research backlog order

1. **Critical**: hook feasibility, automatic WotLK pet spells, native pet save semantics, source tame identity.
2. **High**: historical rank provenance (especially no-source Charge 4), 3.3.5 DBC rank mapping and client UI constraints.
3. **High**: initial 13 special beasts (Broken Tooth, Son of Hakkar, Bloodaxe Worg etc.) and their exact world IDs/IP access.
4. **High**: fill comprehensive **creature→ability source** DB from Classic pet teaching lists and compare to world DB.
5. **Medium**: all 17 family damage/armour/health scaling, food and focus, present in IP and Naxx Core.
6. **Medium**: loyalty timing, XP cap behaviour, desertion risk and trainer respec costs.
7. **Future**: TBC transition and client-addon options require separate decisions.

**Completion condition for research:** comprehensive evidence and open gaps are explicit and actionable; a broad documentation quantity target is not a substitute for quality. Prioritise the core/module hard blockers before many low-impact spell descriptions.
