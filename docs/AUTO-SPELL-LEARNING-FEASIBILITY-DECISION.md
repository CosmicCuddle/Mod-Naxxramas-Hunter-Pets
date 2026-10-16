# WotLK automatic Hunter-pet abilities — module-only feasibility decision (research)

**Reviewed:** 16 October 2026  
**Status:** SOURCE-AUDITED, **NOT IMPLEMENTED**, **NO GO FOR FULL-FIDELITY TRAINING UNTIL A SAFE MODULE-ONLY PATH IS PROVEN**.  
**Owner boundary:** No changes to AzerothCore, Individual Progression, Playerbots or Naxx Core source. No deployment or world/characters DB/DBC edits. **All Vanilla IP stages** must use any approved Vanilla ruleset; later stages must remain unaffected.

## Executive finding

The core knows **two** types of automatically taught pet spells: (1) family-level rank maps generated from 3.3.5 `CreatureFamily.dbc` + `SkillLineAbility.dbc`, and (2) default pet spells loaded from `CreatureSpellData.dbc` / creature templates. It calls **`Pet::learnSpell` directly** for both. The inspected public `PetScript` does **not** provide a generic 'may this pet learn this spell?' veto.

There **is** a global `AllSpellScript::OnSpellCheckCast` (exposed through `SpellSC`) that can influence *cast legality*, and a public `Pet::isBeingLoaded()` read-only state plus `PlayerScript::OnPlayerAfterUpdate`. These are promising for **a constrained compromise**, **not** equivalent to stopping the core from teaching the rank before it reaches the pet or client.

**Two different goals must be treated separately:**

- **Strict authentic Beast Training:** an unlearned rank is never taught, displayed or cast before a valid tame→Hunter-knowledge→TP purchase event. Native spell list, action bar, passives, client packets, persistence and bots all match.
- **Functional eligibility compatibility compromise:** core might internally teach extra ranks; module blocks attempts to use disallowed active spells and later removes/reconciles ranks. This must be labelled **compromise** and needs proof that client state, passive effects, owner/era gating and crash safety are acceptable. Owner has *not* approved such a compromise.

## Pinned inspected upstream source

| Asset | Public blob SHA | Line(s) / precise behaviour |
|---|---|---|
| [SpellMgr.cpp](https://github.com/azerothcore/azerothcore-wotlk/blob/master/src/server/game/Spells/SpellMgr.cpp) | `ea99b0e0a9e2fad716d21d2edbba0006e1eadd91` | 2637–2675: `LoadPetLevelupSpellMap` reads all CreatureFamily skill lines and `SkillLineAbility` rows with learned-on-skill-learn acquisition method; maps family→spell level→spell ID. 2731–2800: default spells from `CreatureSpellData`, plus summon creature template spell IDs. |
| [SpellMgr.h](https://github.com/azerothcore/azerothcore-wotlk/blob/master/src/server/game/Spells/SpellMgr.h) | `b5cc6a338e73b12ccf019a64115bbd80cd772b92` | 731–732: **const** getters for family/default pet spell lists. Family/default maps are private SpellMgr members around 840–841. No inspected public per-pet-owner filter or safe mutable registry for overriding only Vanilla-stage Hunter ranks. |
| [Pet.cpp](https://github.com/azerothcore/azerothcore-wotlk/blob/master/src/server/game/Entities/Pet/Pet.cpp) | `c58f4cc10364cafe68e994bf4f13946630fee611` | 1903–1917: `learnSpell` sends `PetLearnedSpell` and `PetSpellInitialize` if not loading. 1920–1957: automatic family/default `learnSpell` calls without per-owner veto. 435–451: saved spell load + auto-grants + **client spell packet BEFORE load flag cleared at 496**. |
| [PetScript.h](https://github.com/azerothcore/azerothcore-wotlk/blob/master/src/server/game/Scripting/ScriptDefines/PetScript.h) | `f1455f20ca3a83e3fc38f0ba9c7dbff9d2faef46` | 43–58: stat, talent/unlearn, AddToWorld hooks. `CanUnlearnSpellSet` and `CanUnlearnSpellDefault` guard **unlearning**, not learning. |
| [Pet.h](https://github.com/azerothcore/azerothcore-wotlk/blob/master/src/server/game/Entities/Pet/Pet.h) | `3c70ef377ed46a70056b048fadb87c5dcec68c55` | 65: public `isBeingLoaded()`; 113+ public `learnSpell`, `unlearnSpell`, `removeSpell`. Existence of these APIs does not guarantee correct packet/TP/native transaction semantics. |
| [AllSpellScript.h](https://github.com/azerothcore/azerothcore-wotlk/blob/master/src/server/game/Scripting/ScriptDefines/AllSpellScript.h) | `b9f2fa1f6bb2a49377617a193bef1580223fd247` | 58: `OnSpellCheckCast(Spell*, bool strict, SpellCastResult&)`, can set cast result. |
| [AllSpellScript.cpp](https://github.com/azerothcore/azerothcore-wotlk/blob/master/src/server/game/Scripting/ScriptDefines/AllSpellScript.cpp) | `2c310e5ec9479121d0cba9ec0fe7fc13f5269d0f` | 27: ScriptMgr dispatches global cast-check hook. |
| [Spell.cpp](https://github.com/azerothcore/azerothcore-wotlk/blob/master/src/server/game/Spells/Spell.cpp) | `5778a88832e862cc7a139c42b6cc9cdc5354ce57` | ~5865: calls cast-check script, returns non-OK result, before core cooldown/other spell checks. Must prove which pet autocasts/triggered effects hit this path. |
| [PlayerScript.h](https://github.com/azerothcore/azerothcore-wotlk/blob/master/src/server/game/Scripting/ScriptDefines/PlayerScript.h) | `0d2314edf7715f383bbab996c6c5fc010b282308` | 300–302: before/after/player update hooks; 929: `OnPlayerCanLearnSpell` affects **Player** learn, not a Pet::learnSpell veto. |
| [GlobalScript.h](https://github.com/azerothcore/azerothcore-wotlk/blob/master/src/server/game/Scripting/ScriptDefines/GlobalScript.h) | `36b46814c087e087c31a689316b367dd3538bb55` | 95: `OnLoadSpellCustomAttr(SpellInfo*)` is global spell metadata, **not a per-owner era-aware pet teach veto**. |
| [AzerothCore hooks reference](https://www.azerothcore.org/wiki/hooks-script) | live documentation | Confirms PetScript, SpellSC and related script surfaces; documentation does **not** promise automatic pet spell-veto or post-async-load hooks. |
| [AzerothCore Core Scripts](https://www.azerothcore.org/wiki/core-scripts) | live documentation | Spell-specific scripts normally registered with macros and optionally `spell_script_names` DB bindings. A valid registration is not proof every spell's execution or timing can be controlled safely. |

The **installed** worldserver revision and owner IP/Naxx/Playerbots source must be pinned separately before extrapolating any line/symbol.

## Why a global data shortcut fails our owner/era contract

```text
3.3.5 CreatureFamily.dbc + SkillLineAbility.dbc
            ↓ SpellMgr::LoadPetLevelupSpellMap (GLOBAL)
family X → all default level-up spells
            ↓ Pet::InitLevelupSpellsForLevel
         Pet::learnSpell (no owner-stage veto)
            ↓
hunter pet and other consumers of same global map
```

- Replacing or clearing `mPetLevelupSpellMap`, changing 3.3.5 DBC family skills globally, or altering default spell data could suppress ranks for post-Vanilla Hunters, other pet users, or all families; violates **era-local, module-only, no global data damage** conditions.
- The exposed getters are `const`; bypassing via `const_cast`, pointer manipulation, global shared-state tricks, or non-reentrant map edits is **not a supported module API** and is **REJECTED**.
- Adding/removing rows in native world `spell_ranks` SQL is *not* a full solution; the auto-grant list is assembled from DBC family/skill-line data. `spell_ranks` has a different function and is not a proxy for client Spell.dbc.
- Altering the live `CreatureFamily` or `PetSpellData` client files as a workaround is not a module-only per-Hunter switch and would require user-approved patch packaging/version and broad cross-era tests.
- A `GlobalScript::OnLoadSpellCustomAttr` callback is global, cannot evaluate **the owning Hunter's current Individual Progression stage** at the time SpellMgr composes every pet's spell list. Do not mutate shared spell objects based on last player seen.

## Candidate module-only route (COMPROMISE — not approval)

```text
AllSpellScript::OnSpellCheckCast
       └── confirm caster is an owned HUNTER_PET
           ├── module disabled / owner beyond Vanilla IP? → preserve normal core
           ├── owner and pet stable identity unresolved? → fail closed for selected skills, not for all pets
           ├── legitimate wild natural skill or Hunter-trained rank? → allow
           └── automatically learned but unearned rank? → deny cast with appropriate error
                              +
PlayerScript::OnPlayerAfterUpdate (potentially)
       └── pet->isBeingLoaded() must be FALSE
           └── reconcile illegal rank/native pet state ONCE per generation
```

**This is NOT strict prevention of auto-learning**, for several reasons:

1. **Packet timing:** existing pet async load sends `owner->PetSpellInitialize()` around Pet.cpp line 450 while `m_loading` is still true, clearing it only around line **496**. A post-load `PlayerScript::OnPlayerAfterUpdate` observer sees readiness **too late** to prevent the original client packet. On *pet level-up*, `learnSpell` sends `PetLearnedSpell` and refreshes pet spellbook **immediately** when loading is false.
2. **Passives:** cast validation cannot automatically stop already-applied *passive auras* (e.g. family stats/talents), nor ensure no bonus exists between auto-grant and reconciliation.
3. **Action bar:** auto-learn and auto-unlearn manipulate rank chains, autocast state and client action-bar bindings; `unlearnSpell` can itself learn previous rank when invoked with `learn_prev=true`. Reconciliation must not accidentally restore banned ranks.
4. **Native persistence:** a pet's spell list and core pet main row save in separate commits; repeated reconciliation may race pet logout/stable or a crash. Must be idempotent and no player loss.
5. **Cross-owner legality:** some natural source pets legitimately start with skills unlearned by the Hunter; ordinary taming/old trainer purchase does not give permission to every pet. Spell-cast decision requires trusted **per-pet** provenance, not just Hunter's knowledge.
6. **All cast paths:** regular player-selected, pet AI autocast, playerbot/Alt Bot casts, triggered spells, off-GCD actions, and after-death events must be tested. A global check must not deny unrelated wild creatures that use the same spell ID.
7. **Cross-era:** if the user reaches TBC, the module must not strip/deny valid WotLK abilities or force old-data pet state onto them. Transition policy remains unapproved.

**Consequence:** Candidate *ability-use gate + reconciliation* may be a **reduced-fidelity opt-in mode** if safe under strict negative tests. Do not present it as having brought back authentic Vanilla Beast Training, or silently ship it without explicit owner trade-off approval.

## Alternatives compared

| Candidate | Strict fidelity | Era-local | Module-only | Finding |
|---|---|---|---|---|
| Native **pre-learn pet veto** hook | **Potentially yes** | Yes | Yes **if exposed in deployed build** | **NOT FOUND in inspected public upstream** — GO only with actual compiled hook/valid extension point |
| Cast denial + after-load reconciliation | **No, until disproven** (temporary rank packet and passives) | Can be | Probably possible for *part* via documented hooks | **EXPERIMENTAL COMPROMISE**, no C++ or staging test yet |
| Clear pet level-up spell maps globally | No | **No** | Not supported safely | **REJECT** |
| Change 3.3.5 client/DBC family skill lines globally | No | **No** | Assets outside safe per-owner design | **REJECT** |
| Direct world DB edits to default NPC spells | Incomplete | Generally no | Violates module-owned isolation | **REJECT** |
| Per-tick blind remove/learn loop | No | Can leak/race | Can appear to work | **REJECT** as unbounded workaround |
| Modify `Pet.cpp` or IP/Playerbot source | Could enable fidelity | Yes | **NO** | **FORBIDDEN by owner boundary** |
| Recreate unrelated safe Vanilla mechanics while training blocked | Partial | Yes | Depends by feature | **Possible scope fallback** only after owner's explicit decision |

## Hard release gates for any proposed active-skill compromise

**G-AL1** A validation hook demonstrably prevents **all** unearned active spell effects, including AI/autocast, triggered and Playerbot uses, with no other caster/class affected.

**G-AL2** On *new tame* as well as *existing load*, no illegitimate passive effects or action-bar state persist or cause an observable advantage. If client receives an inappropriate temporary spell, label reduced fidelity and request specific owner approval.

**G-AL3** After stable, relog, server restart, XP change, trainer rank upgrade, refund, pet death and IP transition, no unwanted rank reappears as usable or corrupts native pet state.

**G-AL4** Module status disabled reproduces the baseline native 3.3.5 behaviour without leftover world/character DB modifications or missing spells; post-Vanilla owner completely unaffected.

**G-AL5** Module ledger and pet spell sync are crash-safe across two native CharacterDB commits and any own writes; valued existing Hunter/Alt Bot pets protected by rehearsed backup/restore on staging.

**G-AL6** No prior legal pet talent/consumables/Naxx Core features are modified; Alt Bots do not get gear or talent changes through this pet system.

**G-AL7** Client action bar/spell list does not allow a race during load, delayed spell use, duplicate autocast or uncontrolled mismatch after reconciliation.

**Stop rule:** one reproduced illegal passive benefit, cross-era alteration, native pet spell loss, repeated illicit rank, bad client packet with no owner-approved compromise, or missing real hook means NO-GO for strict manual training.

## Minimum safe research/test assignments

1. **Read-only deployed revision**: obtain `git rev-parse HEAD` and actual module SHA list; verify all cited public symbols are present. No need to recompile yet.
2. **Read-only original/deployed DBC**: map 59 wild + 47 likely trainer ranks; identify the exact automatic family/default spell IDs that would be blocked.
3. **API compile proof** (only if owner later approves staging C++): in a **separate module**, log `AllSpellScript::OnSpellCheckCast`, `PlayerScript::OnPlayerAfterUpdate`, `Pet::isBeingLoaded`, `Pet::learnSpell`/source state—do **not mutate** pet spells yet. Check exact hook dispatch and ordering, no performance regression.
4. **Staging proof**: choose disposable Hunter + one disposable Alt Bot, one pet with named source, second control Hunter **beyond Vanilla IP**, one Warlock. Evaluate native spells before tame, during spawn, while loading, after async callback and after client packet.
5. **Proposal gate**: compare **strict authentic** vs **approved compromised** UX with failing/skipped tests. Show owner actual in-game evidence before requesting acceptance.
6. If no strict viable path exists, continue optional source-driven features that do not depend on auto-spell suppression (identity, rare attack cadence if proven, feeding/loyalty UI) **only after separate approval**.

**Zero compiled/running behaviour verified. No code, worldserver, or native data was changed during this audit.**
