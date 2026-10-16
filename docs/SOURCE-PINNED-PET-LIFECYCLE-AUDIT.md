# Source-pinned Hunter pet lifecycle and hook-order audit

**Read-only source review:** 16 October 2026. **No executable implementation, configuration, SQL or game asset was changed.**
**Evidence class:** Public upstream AzerothCore **file blob SHAs** checked in connected GitHub. These are **file contents**, **not** the user's installed worldserver commit or installed database. Before code, pin installed *commit*, build flags, module versions, pet DB and DBC.

## 1. Scope, code pins and evidence strength

| Source file | Public upstream blob SHA | Relevant lines | Confidence |
|---|---|---|---|
| [Pet.cpp](https://github.com/azerothcore/azerothcore-wotlk/blob/master/src/server/game/Entities/Pet/Pet.cpp) | `c58f4cc10364cafe68e994bf4f13946630fee611` | 72–118, 290–450, 502–623, 948–960, 1380–1436, 1920–1965, 2389–2411 | High **for these publicly inspected source paths only** |
| [Unit.cpp](https://github.com/azerothcore/azerothcore-wotlk/blob/master/src/server/game/Entities/Unit/Unit.cpp) | `050bdfce8655b89e36a4f5767416c2ec653391c3` | 13384–13454 | High for current source |
| [SpellEffects.cpp](https://github.com/azerothcore/azerothcore-wotlk/blob/master/src/server/game/Spells/SpellEffects.cpp) | `d684660c9959422202546e944f039beb392a117f` | 3311–3368 | High for current source |
| [Spell.cpp](https://github.com/azerothcore/azerothcore-wotlk/blob/master/src/server/game/Spells/Spell.cpp) | `5778a88832e862cc7a139c42b6cc9cdc5354ce57` | 3895–4216; other target-hit paths | High for order; not proof of staged behaviour |
| [PetScript.h](https://github.com/azerothcore/azerothcore-wotlk/blob/master/src/server/game/Scripting/ScriptDefines/PetScript.h) | `f1455f20ca3a83e3fc38f0ba9c7dbff9d2faef46` | 40–60 | High for exposed hook interface |
| [SpellScript.h](https://github.com/azerothcore/azerothcore-wotlk/blob/master/src/server/game/Spells/SpellScript.h) | `473c2d36a5845fb0239bda1cf7b2f2f70a8e9eb2` | 323–347 | High for declared hooks; attach/load feasibility untested |
| [PlayerScript.h](https://github.com/azerothcore/azerothcore-wotlk/blob/master/src/server/game/Scripting/ScriptDefines/PlayerScript.h) | `0d2314edf7715f383bbab996c6c5fc010b282308` | 377, 525 | High for hook declarations |
| [spell_hunter.cpp](https://github.com/azerothcore/azerothcore-wotlk/blob/master/src/server/scripts/Spells/spell_hunter.cpp) | `ece192038bc4865309bc0586b0ac217f52af627e` | 1003–1077 | High for Tame Beast preconditions |

All line numbers refer to these specific file blobs, **not stable latest file line numbers**. When the installed code differs, redo the audit.

## 2. What really happens on a successful new tame

### Source order

```text
Hunter casts Tame Beast (spell 1515)
    |
    ├─ spell_hun_tame_beast::CheckCast
    |   ├─ player, target level, exotic flag, tameable flag
    |   └─ current/unslotted pet and charm/owner checks
    |
    └─ Spell::EffectTameCreature [hit target]
          ├─ validate caster/target/no existing pet
          ├─ Unit::CreateTamedPetFrom(creatureTarget, 1515)
          │    ├─ Pet::CreateBaseAtCreature → source creature template/display
          │    ├─ CURRENT WOTLK-STYLE pet level:
          │    │   max(wild creature level, Hunter level − 5)
          │    └─ Unit::InitTamedPet
          │          ├─ InitStatsForLevel (includes potential stats script)
          │          ├─ assign unique PetNumber
          │          ├─ InitPetCreateSpells
          │          │     ├─ family passives
          │          │     └─ InitLevelupSpellsForLevel (WotLK auto rank teaching)
          │          └─ FillPetInfo(current)
          ├─ despawn original creature
          ├─ AddToMap → Pet::AddToWorld → OnPetAddToWorld
          ├─ owner SetMinion / InitTalentForLevel
          └─ SavePetToDB / PetSpellInitialize
```

**Confirmed implications:**
1. Source creature **template entry** survives the pet creation process (`Pet::CreateBaseAtCreature` → `GetEntry()`/native `PetInfo.CreatureId`), so source identity is promising. Creature entry and individual spawn GUID are **different**; the latter is not implied to survive. The source visual/display is copied. Verify live persistence, cases with transforms, phasing, and pet resurrection.
2. **Pet level is changed before the new pet appears in the world**. Vanilla's keep-original-wild-level requirement cannot be assumed implementable by a mere pet-level config or OnAdd hook. The source formula occurs twice in `Unit::CreateTamedPetFrom` and `EffectTameCreature` (visual). Need module-only **pre-effect** interception or an owner-approved safe post-creation state transition *before release*.
3. The new pet's natural and family spells are initialized through the core WotLK `InitPetCreateSpells` BEFORE `OnPetAddToWorld`. Neither a source rank dataset nor preserved `GetEntry()` automatically recreates original learning.
4. `OnPetAddToWorld` on **new tame** is after first spell creation and before the new pet's save. That differs from **loading an existing pet** (section 3), so one handler cannot simply assume this callback always means 'all spells loaded'.
5. In `EffectTameCreature`, `AddToMap` occurs **before** `unitCaster->SetMinion(pet, true)`. Therefore a handler must **not** assume `owner->GetPet()` already returns the new pet at `OnPetAddToWorld`. In `InitTamedPet`, `InitStatsForLevel` is **before** `SetPetNumber`—the early stat hook cannot assume a native unique pet number exists either. Both are important for identity/ledger design.

## 3. What really happens when loading a stabled/existing pet

```text
Pet::LoadPetFromDB
  ├─ OnPlayerBeforeLoadPetFromDB
  ├─ checks native PetStable / entry / Tameable
  ├─ Pet::Create(saved PetInfo.CreatureId, saved PetNumber)
  ├─ set saved display, level, name
  ├─ InitStatsForLevel
  ├─ SynchronizeLevelWithOwner
  │     └─ raises hunter pet to Hunter−5 if lower; may GivePetLevel
  ├─ owner SetMinion; load action bar
  ├─ AddToMap → OnPetAddToWorld   <--- TOO EARLY TO ASSUME SPELLS LOADED
  └─ async CharacterDatabase query holder completion callback
        ├─ InitTalentForLevel
        ├─ _LoadAuras
        ├─ _LoadSpells (native spells from character DB)
        ├─ InitTalentForLevel (again)
        ├─ _LoadSpellCooldowns
        ├─ LearnPetPassives
        ├─ InitLevelupSpellsForLevel  <--- AUTOMATIC WOTLK RANK GRANTS
        ├─ CastPetAuras; cleanup action bar
        ├─ owner PetSpellInitialize; SendTalentsInfoData(true)
        └─ m_loading = false (later in callback)
```

**Critical finding:** `OnPetAddToWorld` occurs on the path **before the async spell/auras/cooldown load**. Any module handler that removes WotLK ranks, restores custom ranks or sends a reconciled spellbook **in that hook** can be overwritten by `_LoadSpells` or `InitLevelupSpellsForLevel` afterward. Merely attaching a pet reconcile callback there is **not a proven implementation**.

`OnPlayerBeforeLoadPetFromDB` is a pre-load callback with parameters and `forceLoadFromDB`; it is **not** an automatic 'post native spells loaded' event. We found **no matching `OnPetAfterDBSpellsLoaded` hook** in inspected `PetScript.h`.

**Investigate, do not assume:** whether a safe module-owned deferred coordination mechanism exists, how it detects actual m_loading state via public APIs, and whether its reconciliation can avoid already-sent WotLK spell packets, transient illegal ranks and save races. A repeating post-hoc remove/add patch is not accepted without a complete proof and owner review.

## 4. Auto spell-learning call paths

From `Pet::InitLevelupSpellsForLevel`:
- Family levelup map: for spells above level the **unlearn branch** consults `CanUnlearnSpellSet`; otherwise the function directly invokes **`learnSpell(spellId)`**.
- Pet-default map: for too-high spells the **unlearn branch** consults `CanUnlearnSpellDefault`; otherwise directly invokes **`learnSpell(spellId)`**.
- Therefore **these hooks veto unlearning, not learning**. Neither one is an auto-learn permission hook.
- `Unit::InitTamedPet` calls `Pet::InitPetCreateSpells` → `InitLevelupSpellsForLevel` on tame.
- `Pet::GivePetLevel` calls `InitStatsForLevel`, **then** `InitLevelupSpellsForLevel`, **then** `InitTalentForLevel` on level change.
- The async existing-pet DB callback calls it again **after loading native saved spells**.
- `Pet::LearnPetPassives` separately adds native family passive spells regardless of the Classic trained-rank ledger.

**Gate verdict:** Original-style pet training with WotLK auto-grant suppression is **SOURCE-BLOCKED / NOT YET SHOWN MODULE-ONLY POSSIBLE**, not safe to claim accomplished.

## 5. How source-specific attack speed could be restored

`Guardian::InitStatsForLevel` obtains pet's original creature template and sets Hunter-pet attack time to `BASE_ATTACK_TIME` rather than `cinfo->BaseAttackTime` because the owner is a Hunter. The method subsequently calls:

```text
OnInitStatsForLevel(this, petlevel)
    ↓
UpdateAllStats()
    ↓
SetFullHealth / mana
    ↓
OnPlayerAfterGuardianInitStatsForLevel(owner, this)
```

`PetScript::OnInitStatsForLevel` is a promising hook for a per-entry attack interval because the Pet has a creature entry and level. **But do not claim success**: verify if `UpdateAllStats` re-normalizes timing, whether melee swing damage is normalized for the faster interval, and every level/stable/load/revive/happiness/haste transition. Original *wild* attack speed isn't the same as active *pet* attack speed on 3.3.5.

The historical [Broken Tooth / Zul'Gurub bat speed references](https://www.wow-petopia.com/classic/attackspeed.php) are **research input only**, not proof that a live module can safely override the core path.

## 6. Learn-by-use event semantics

- [`AllSpellScript.h`](https://github.com/azerothcore/azerothcore-wotlk/blob/master/src/server/game/Scripting/ScriptDefines/AllSpellScript.h) declares global `OnSpellCast`, `OnSpellCastCancel`, `OnSpellPrepare`, etc.
- [`Spell.cpp`](https://github.com/azerothcore/azerothcore-wotlk/blob/master/src/server/game/Spells/Spell.cpp) calls `sScriptMgr->OnSpellCast` toward the end of `Spell::cast`. **Immediate** spell effects may already have been applied; **delayed/projectile** impacts may still be pending. Thus it is **not a universal 'target successfully affected' callback**.
- [`SpellScript.h`](https://github.com/azerothcore/azerothcore-wotlk/blob/master/src/server/game/Spells/SpellScript.h) declares `AfterCast` at around line 323, `OnEffectHitTarget` at line 336, and `AfterHit` at line 347. Target-scoped hit hooks are **promising candidates** for a precise learn-by-use trigger; they are not a guarantee for non-target/passive/periodic abilities. A failed/immune/dodged/triggered spell and ownership changes must be tested.
- CMaNGOS Classic `Pet::CheckLearning` maintains a `m_teachspells` map and checks skill use; its code includes a 10% chance. This is an **implementation**, not proven original retail chance. [CMaNGOS Pet.cpp](https://github.com/cmangos/mangos-classic/blob/master/src/game/Entities/Pet.cpp).
- **Important**: do not grant the Hunter *wild source skill knowledge* because a non-eligible pet auto-learned a WotLK family spell. The source identity and natural-rank provenance must be verified separately.

## 7. Native save boundaries and rollback risk

`Pet::SavePetToDB` visibly uses **two separate CharacterDatabase transactions**:
1. save pet aura, spells and cooldowns, then `CommitTransaction`;
2. save/replace main pet details and action bar in a **second** `CommitTransaction` (or delete path).

A separate module ledger transaction could be a **third** independent commit. The design must **not** promise an all-or-nothing transaction spanning native pet operations unless actual integration into the same transaction is available in public APIs.

**Required safe model to explore:** native pet IDs and owner identity; idempotent module journal with generation/version; mutation replay/reconcile after restart; database crash simulation between native and module commits; rollback without losing native spells, action bars, pet skins or stabled slots. Treat this as source-informed feasibility study, not an authorized data migration.

## 8. Risks sharpened by this audit

| ID | Finding | Consequence | Status |
|---|---|---|---|
| L-01 | OnPetAddToWorld is pre-async-load on existing pets | Can't treat as after-native-rank reconciliation | **Verified upstream / module solution not proven** |
| L-02 | WotLK auto-grant in tame, level-up, DB-load paths | Need true suppression or complete safe substitute | **Hard blocker pending proof** |
| L-03 | Original wild tame level overwritten to Hunter−5 before AddToWorld | Vanilla pet XP journey distorted | **Hard blocker pending module-only interception** |
| L-04 | Native pet save split across multiple transactions | Ledger can diverge/crash mid-save | **Hard safety gate pending solution** |
| L-05 | OnSpellCast not universal successful target hit | False permanent Hunter learning possible | **Need per-spell hook pilot** |
| L-06 | Creature entry and pet number both present | Source ID retention is promising | **Pending deployed/stable/visual proof** |
| L-07 | Stats hook after attack speed normalization | Rare swing restoration plausible | **Pending speed and DPS proof** |
| L-08 | Original 1.12/client spell effect compatibility unknown | Same name/ID may have changed effect | **Blocked until DBC verified** |

## 9. Related Individual Progression source cross-check

A read-only inspection of [`src/IndividualProgressionPlayer.cpp`](https://github.com/ZhengPeiRu21/mod-individual-progression/blob/master/src/IndividualProgressionPlayer.cpp), **public blob SHA `e3c5f1781885a4f2ded5b135d2cf4c96467cd9da`**, lines **116–159**, shows an additional interaction with pet levelling:

- `OnPlayerGiveXP` handles capped bot accounts and normal accounts separately.
- For a **normal human account at level 60 that has not passed `PROGRESSION_PRE_TBC`**, it calls `pet->GivePetXP(...)` **for kill-sourced XP** and then sets the player's XP award to zero.
- Capped bot accounts have their own `BotAccountsMaxLevel` check and pet XP side path.
- For normal post-Vanilla / pre-next-cap characters at level 70 it has another pet XP handoff.
- These are **current public Individual Progression source observations**, not proof the user's deployed IP revision is identical or that all Playerbot accounts follow the same rules.

**Implication:** restoring a level-20 original tame to a level-60 Vanilla Hunter must preserve the expected XP journey even while player XP is capped. Any module-own XP adjustment must account for the installed IP `OnPlayerGiveXP` handoff so the pet is not deprived of XP or awarded it twice. A read-only audit of IP stage API and exact deployed modules is mandatory before a change; **no upstream IP modification is allowed**.

## 10. Precise next developer actions — read-only, then only approved disposable staging

### Read-only Stage A (can be planned immediately)
- Pin exact deployed core, IP, Playerbots, Naxx Core SHAs and pet DB schema. Compare this public blob audit to **installed build**. Note if branch-specific modules already intercept these operations.
- Trace actual public module hooks and their invocation order after every **tame**, level-up, stable, logout, resurrection, reload and owner IP stage transition.
- Identify any documented **module-only** pre-tame level override, post-async native-spells-ready event, rank-teaching veto or supported persistence transaction hook. If none, mark the affected subsystem **NO-GO** rather than editing the core.
- Inspect original 1.12 DBC, installed 3.3.5 DBC and exact world `creature_template` via owner-approved read-only copies. Do not publish private character rows.
- Establish what a 3.3.5 client can represent without an approved DBC patch or addon.

### Approval-gated disposable staging Stage B
- **Tame level test**: level 60 owner tames level 20 beast; compare tame packet, assigned native PetNumber, native DB level, summoned level and XP. Vanilla expectation is 20 (subject to selected original patch spec), AzerothCore default source shows 55.
- **Add hook order test**: log `OnPetAddToWorld` pet spells, post-async native spells, client packets, saved state; new tame vs stabled pet. Confirm no illegitimate rank appears.
- **Level-up test**: audit `InitLevelupSpellsForLevel` grants; veto must be real and module-owned if offered as solution.
- **Spell-use test**: natural skill cast hits, misses, immune, interrupted, delayed/projectile, passive and periodic case. A false Hunter-learn is release blocker.
- **Crash/restart test**: force failure between first native save commit, second commit and module ledger save on disposable characters; no missing pet/ability and successful idempotent recovery.
- **Rare speed test**: Broken Tooth vs control Cat over level+stable+restart with speed and DPS logs.

**Do not run staging, compile, alter DB/DBC, patch upstream or deploy production until owner explicitly approves.** This document is the practical source-level handoff for module-only GO/NO-GO decisions.
