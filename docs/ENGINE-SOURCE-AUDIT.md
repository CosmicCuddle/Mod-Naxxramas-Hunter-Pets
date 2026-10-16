# Emulator engineering reference — **not** a ready-made port

**Inspected:** source in upstream public repositories during planning on 16 October 2026. The live realm's compiled commits may differ. This document records where developers should inspect; no changes to upstream repositories are allowed.

## Source-pinned event order — new findings

The [exact file-blob audited lifecycle](SOURCE-PINNED-PET-LIFECYCLE-AUDIT.md) and [16-event CSV](../data/reference/public-core-pet-lifecycle-events.csv) now supply the concrete tame, level, summon/stable, spell cast and save call order. Major observations:

1. **New tame:** Tame Beast → CreateTamedPetFrom → native WotLK pet-level catchup (`max(beast level, Hunter−5)`) → `InitPetCreateSpells` → `InitLevelupSpellsForLevel` → **then** `OnPetAddToWorld` → native save. OnAdd is **after** auto-granted skills on this path.
2. **Existing pet:** `LoadPetFromDB` → `SynchronizeLevelWithOwner` (same −5 catchup) → **`OnPetAddToWorld`** → **asynchronous** `_LoadSpells` and `InitLevelupSpellsForLevel` → pet spellbook/client packet. OnAdd is **before** saved spells and auto-granted skills on this path.
3. **Level-up:** `GivePetLevel` → `InitStatsForLevel` → `InitLevelupSpellsForLevel` → `InitTalentForLevel`. No generic `CanLearnPetSpell` hook found in inspected `PetScript`.
4. **Native save:** `SavePetToDB` saves aura/spells/cooldowns in one committed transaction and pet main identity/slot in another. A module-owned ledger would add another independently committed unit unless an explicit integration strategy is proven.
5. **Successful use:** global `AllSpellScript::OnSpellCast` cannot by itself prove target impact for every projectile/periodic/immune spell; `SpellScript` exposes `AfterHit`/effect target hooks, requiring ability-specific proof.
6. **Rare swing:** `Guardian::InitStatsForLevel` overwrites Hunter attack time to `BASE_ATTACK_TIME` and invokes `OnInitStatsForLevel` **before** `UpdateAllStats`; override is plausible but must survive subsequent recalculation.

These findings are **public source facts only**. No compiled live worldserver SHA, installed IP module, or deployed client DBC has been checked.

## AzerothCore 3.3.5a — primary target

| Location / symbol | Observed behaviour | Design implication |
|---|---|---|
| [`src/server/game/Entities/Pet/Pet.cpp`](https://github.com/azerothcore/azerothcore-wotlk/blob/master/src/server/game/Entities/Pet/Pet.cpp) — `Pet::InitLevelupSpellsForLevel` | Calls `learnSpell` for level-appropriate family/default pet spells | Classic *manual* rank training conflicts with core auto-learn; **critical blocker** until module-only suppression is proven |
| Same — `Pet::GivePetLevel` | Calls stat init, auto pet levelup spells and talent init | Must test rank regrant and pet talents on XP |
| Same — `Pet::InitPetCreateSpells` | Clears/reinitialises pet spells with passives and levelup spells | Danger to persisted custom trained ranks |
| Same — `Pet::LearnPetPassives` | Learns family passive spells | Distinguish passive auras required for engine from Vanilla trainable passives |
| Same — `Guardian::InitStatsForLevel` | Hunter pets assigned `BASE_ATTACK_TIME` rather than source `BaseAttackTime` | Named-creature attack speed potentially restored by `OnInitStatsForLevel`, test hook timing |
| Same — `Pet::LoseHappiness` and `GetHappinessState` | Existing Hunter happiness decay/states | Do not duplicate existing effects, compare decay with Vanilla |
| Same — `Pet::SynchronizeLevelWithOwner` | Hunter pet lower than owner level by >5 may be raised to owner−5 | Classic tame-level/XP rules conflict; check Individual Progression's installed HunterPetLevelFix |
| [`PetScript.h`](https://github.com/azerothcore/azerothcore-wotlk/blob/master/src/server/game/Scripting/ScriptDefines/PetScript.h) | `OnInitStatsForLevel`, `OnCalculateMaxTalentPointsForLevel`, `CanUnlearnSpellSet`, `CanUnlearnSpellDefault`, `CanResetTalents`, `OnPetAddToWorld` | Useful interception points, **not** a generic `CanLearnPetSpell` veto |
| [`AllSpellScript.h`](https://github.com/azerothcore/azerothcore-wotlk/blob/master/src/server/game/Scripting/ScriptDefines/AllSpellScript.h) | `OnSpellCast`, `OnSpellCastCancel`, `OnSpellPrepare`, `OnSpellCheckCast` | A casting event may not imply a **successful** effect. Need actual lifecycle proof for learn-by-use |
| [`PlayerScript.h`](https://github.com/azerothcore/azerothcore-wotlk/blob/master/src/server/game/Scripting/ScriptDefines/PlayerScript.h) | Login/logout/save/level change/gossip hooks | Possible persistence and training UI integration; examine timing relative to pet save |
| [`PetHandler.cpp`](https://github.com/azerothcore/azerothcore-wotlk/blob/master/src/server/game/Handlers/PetHandler.cpp) | Current pet-talent and native pet actions handled under WotLK logic | Cannot assume client will display old TP or Beast Training; wire-protocol ownership risk |

**Primary task:** build an event graph covering wild beast→Tame Beast→new pet ID→save; pet casting→effect resolution; level/stat init→spell init→talent update; stable→load→save; and pet trainer/gossip/DB transactions.

**Hook limitation:** OnPetAddToWorld runs after a pet has entered the world, but whether it runs before or after automatic spell packets, or whether it can safely reconcile without transient WotLK ranks, requires an instrumented staging test. Do not treat a post-hook removal loop as an approved fix.

## CMaNGOS Classic — historical implementation reference

[CMaNGOS Classic repository](https://github.com/cmangos/mangos-classic)
[Classic `Pet.cpp`](https://github.com/cmangos/mangos-classic/blob/master/src/game/Entities/Pet.cpp)
[Classic `Pet.h`](https://github.com/cmangos/mangos-classic/blob/master/src/game/Entities/Pet.h)
[Classic `PetHandler.cpp`](https://github.com/cmangos/mangos-classic/blob/master/src/game/Entities/PetHandler.cpp)
[Classic `characters.sql`](https://github.com/cmangos/mangos-classic/blob/master/sql/base/characters.sql)

Specific conceptual references observed:
- Persistent `loyaltypoints`, `loyalty`, `trainpoint`, `xpForNextLoyalty`, `TeachSpelldata` and reset metadata in its classic `character_pet` schema.
- `Pet::TickLoyaltyChange` adjusts loyalty according to happy/content/unhappy status; `ModifyLoyalty` handles rank thresholds, pet training changes and potential desertion.
- `Pet::SetRequiredXpForNextLoyaltyLevel` refers to 5% of Hunter level XP to next level (with cap accommodation).
- `Pet::CheckLearning` consults teachable spells on the pet, with a probabilistic gain after ability use (**10%** condition in inspected code). This is emulator implementation, not independently certified retail RNG.
- `Pet::CanTakeMoreActiveSpells` and handler paths enforce the classic four-active limit and training semantics.
- `Pet::resetTalentsCost` handles old pet untraining cost escalation and a time-based reset.

**Use it as a behavioural cross-check, never transplant CMaNGOS `Pet.cpp` into AzerothCore.** Differences in licensing obligations, structs, client packet layout, update fields, threading and transaction rules must be checked before any reuse.

## Source references for UI and limits

- [Classic Petopia differences](https://www.wow-petopia.com/classic/differences.php): **one active pet + two stable slots** in Classic (three owned maximum); newly tamed pet stays its level.
- [WotLK Petopia differences](https://www.wow-petopia.com/classic_lk/differences.php): **one active pet + four stable slots** in WotLK. Before considering reducing slots, inspect existing pets; no deletion is ever acceptable during first deployment.
- [Original training UI](https://www.wow-petopia.com/classic/training.php) vs 3.3.5 client: historically General spellbook Beast Training, absent in post-3.0.2 mechanics.

**Recommendation (not approved):** retain safe existing WotLK stable capacity while restoring Vanilla meaningful learning until the owner explicitly chooses whether exact two-slot constraints justify data-migration and client work. Do not force a third-pet loss as a 'Vanilla fix'.

## IP + Playerbots contracts

- IP reference: https://github.com/ZhengPeiRu21/mod-individual-progression and the installed branch, not unverified assumptions from latest upstream. Inspect pet-level override, individual progression stage getter, player/bot distinctions and NPC availability.
- Playerbots reference: https://github.com/mod-playerbots/mod-playerbots; examine Hunter pet/tame/pet talent/learn-spell pathways **read-only**, plus database restarts, rerandomization and raid AI.
- Naxx Core: https://github.com/CosmicCuddle/Mod-Naxxramas-Core with `src/Systems/BotRaidConsumables.cpp` and NT1 talent importer. No companion module may touch Alt Bot gear.
- Keep test coverage of combat in MC and ZG, pet buff interaction, player-controlled stable and spell bars.

## Proof before implementation

For each desired extension write: exact installed commit; source function path/line; event order diagram; eligible pet type; cross-module callback ownership; what it can **guarantee**; data side effects; counterexample; proof screenshot/log/test; pass/fail and fallback. If mandatory auto-learn suppression or success-cast observation cannot be achieved, record **BLOCKED: module-only** instead of editing core.

## Licence / provenance note

Ideas and factual mechanics can be used in the blueprint. Developers must respect upstream license headers and distribution conditions for any code reuse; this repository planning work contains **original specifications**, not copied CMaNGOS source code.
