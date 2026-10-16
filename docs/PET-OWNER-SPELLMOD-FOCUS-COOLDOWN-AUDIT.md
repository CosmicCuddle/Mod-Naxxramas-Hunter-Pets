# Pet Focus and cooldown — newly identified owner spell-modifier path

**16 October 2026. RESEARCH ONLY.** Source-pinned public AzerothCore 3.3.5a analysis. No code, SQL, DBC, addon, build, staging tests, character records or live worldserver changes. Owner reports newest IP; actual compiled source/data and module-only binding viability remain unverified. Do not edit AzerothCore/IP/Playerbots/Naxx Core source.

## Updated verdict (corrects the earlier 'no documented path' phrasing)

**A promising NATIVE calculation path DOES exist**, though no usable **module-only, stage/ability-specific modifier** has been demonstrated or approved.

- Pet **Focus** cost ultimately uses `SpellInfo::CalcPowerCost` which consults **the pet's controlling Player's `SPELLMOD_COST` modifiers** through `unitCaster->GetSpellModOwner()`.
- Pet **spell and category cooldowns** use `Creature::AddSpellCooldown`, which consults the controlling Player's **`SPELLMOD_COOLDOWN` modifiers** on both recovery values. Both manual pet command handling and PetAI autocasts call that function.
- **Critical limitation:** a `SpellModifier` is keyed by a spell class family/mask, and applied through the **owner Player's spell-modifier container**. It does **not** automatically distinguish a pet casting from the Hunter casting and it is not directly keyed to an arbitrary exact spell ID.
- Public `GlobalScript::OnIsAffectedBySpellModCheck(affectSpell, checkSpell, mod)` offers a **global match override**. The default returns `true` (normal family/mask check). If any enabled handler returns `false`, the `SpellInfo::IsAffectedBySpellMod` path **returns true**, treating the spell as affected; this is **not a veto**, and not an owner/caster-specific callback. Incorrect use could make **unrelated spells eligible for a modifier**. This path is an **EXPERIMENTAL SCOPE FILTER**, not a demonstrated fix.

**Result:** Update from `NO_DOCUMENTED_PER_OWNER_COST_COOLDOWN_MECHANISM` to `EXISTING_OWNER_SPELLMOD_PATH__ERA_AND_SPELL_SCOPE_UNPROVEN`. Do **not** equate existence with compatibility or successful restored Vanilla costs.

## Exact pinned source proof

| Public source and lines | Blob SHA | What it demonstrates |
|---|---|---|
| [`SpellInfo.cpp` 2829–2920](https://github.com/azerothcore/azerothcore-wotlk/blob/master/src/server/game/Spells/SpellInfo.cpp) | `84e3900796fc02931bc65c65e86b205880d1e1b4` | Native `CalcPowerCost`: base ManaCost (used as Focus), per-school and % modifiers; `unitCaster->GetSpellModOwner()->ApplySpellMod(Id, SPELLMOD_COST, powerCost, spell)` before returning the actual value |
| [`Spell.cpp` 3626–3636, 7071–7105](https://github.com/azerothcore/azerothcore-wotlk/blob/master/src/server/game/Spells/Spell.cpp) | `5778a88832e862cc7a139c42b6cc9cdc5354ce57` | `Spell::prepare` and `CheckPetCast` both calculate power cost through `CalcPowerCost`; `CheckPetCast` tests existing creature cooldown before cast |
| [`Unit.cpp` 12583–12608](https://github.com/azerothcore/azerothcore-wotlk/blob/master/src/server/game/Entities/Unit/Unit.cpp) | `050bdfce8655b89e36a4f5767416c2ec653391c3` | `Unit::GetSpellModOwner()` resolves a direct Player owner of a pet, not necessarily the pet itself |
| [`Creature.cpp` 3013–3076](https://github.com/azerothcore/azerothcore-wotlk/blob/master/src/server/game/Entities/Creature/Creature.cpp) | `2182bd8d0255b9be0b57ba641c30e3484d1a73d9` | Native `AddSpellCooldown` uses the owner `ApplySpellMod(..., SPELLMOD_COOLDOWN, ...)` for both spell and category recovery, then stores cooldowns and sends owner packets when appropriate |
| [`PetHandler.cpp` ~380–420](https://github.com/azerothcore/azerothcore-wotlk/blob/master/src/server/game/Handlers/PetHandler.cpp) | Source read in this audit | Manual pet command path checks `CheckPetCast` then calls `pet->ToCreature()->AddSpellCooldown(spellId,0,0)` before `spell->prepare` |
| [`PetAI.cpp` ~350–362](https://github.com/azerothcore/azerothcore-wotlk/blob/master/src/server/game/AI/CoreAI/PetAI.cpp) | `c646f94def0c0ee98ba8ae9ed115d6d4999175ff` | AI autocast also calls `me->AddSpellCooldown(...)` before `spell->prepare`. Focus + cooldown approach must cover both routes |
| [`Player.cpp` ~10004–10110](https://github.com/azerothcore/azerothcore-wotlk/blob/master/src/server/game/Entities/Player/Player.cpp) | `138e7eab4905ec366338a54cc18e6cfe749a3670` | `ApplySpellMod` looks up all active owner modifiers for operation, filters with `SpellInfo::IsAffectedBySpellMod`, combines flat and percent values; `AddSpellMod` produces client modifier packets |
| [`Player.h` ~181 and 1810–1819](https://github.com/azerothcore/azerothcore-wotlk/blob/master/src/server/game/Entities/Player/Player.h) | `5a29ac7b40ade7a11f1fcf36623dc7530ea39595` | `SpellModifier` fields: `op,type,value,mask,spellId,ownerAura`; no intrinsic exact-target spell ID or caster field |
| [`SpellAuraEffects.cpp` ~691–710 and 813–830](https://github.com/azerothcore/azerothcore-wotlk/blob/master/src/server/game/Spells/Auras/SpellAuraEffects.cpp) | `f6ad1b90b5399d6f161daf58721348603ce79490` | Native aura with `SPELL_AURA_ADD_FLAT_MODIFIER`/`_PCT_MODIFIER` constructs a `SpellModifier`; `ApplySpellMod` registers it **only on Player aura targets** |
| [`SpellScript.h` ~791–799](https://github.com/azerothcore/azerothcore-wotlk/blob/master/src/server/game/Spells/SpellScript.h) | `473c2d36a5845fb0239bda1cf7b2f2f70a8e9eb2` | `AuraScript::DoEffectCalcSpellMod` permits a calculated `SpellModifier*&` for a matching aura; **does not establish an available valid aura SpellID** |
| [`SpellInfo.cpp` ~1338–1369](https://github.com/azerothcore/azerothcore-wotlk/blob/master/src/server/game/Spells/SpellInfo.cpp) | same `84e390...` | `IsAffectedBySpellMod` checks modifier origin aura SpellInfo, invokes global script, then default `IsAffected(family,mask)`. A global-script `false` response **forces affected=true** |
| [`GlobalScript.h` ~89 and `GlobalScript.cpp` ~92–97](https://github.com/azerothcore/azerothcore-wotlk/blob/master/src/server/game/Scripting/ScriptDefines/GlobalScript.h) | `36b46814c087e087c31a689316b367dd3538bb55`; dispatcher `9bcfd4634a410262f2f16522ec7bf8132f7e3c6c` | `OnIsAffectedBySpellModCheck` defaults to `true`; uses boolean-hook dispatcher |
| [`ScriptMgrMacros.h` ~76–81](https://github.com/azerothcore/azerothcore-wotlk/blob/master/src/server/game/Scripting/ScriptMgrMacros.h) | `80895976796c2cc0219256b5b653d3e05c7c908c` | One handler returning `false` makes `OnIsAffectedBySpellModCheck` false. Since `IsAffectedBySpellMod` negates this result to return `true`, **misuse forces a match** |

**Source reading is not deployed proof.** These APIs may be absent/changed in the owner's actual compiled binaries; current owner IP API only gives an initial version target.

## Implications for Lightning Breath rank 5 (25011)

Our [public-reference effect comparison](PUBLIC-335-SPELL-EFFECT-CONFLICT-PILOT.md) says original-style **50 Focus** and no listed cooldown versus public WotLK **20 Focus / 10 seconds**. This is a candidate test specification, **not a verified original 1.12 retail or deployed 3.3.5 spell record**.

For a **hypothetical 20 → 50 Focus increase**, a properly scoped **flat +30 `SPELLMOD_COST`** is conceptually possible on an owner aura, because `CalcPowerCost` includes the owner mod **before** native power checks and deductions. **It must not be created or enabled** until an existing source-valid aura can safely carry the modifier, all relevant rank SpellIDs/family flags are confirmed, and it is shown NOT to affect any hunter/randombot/non-Vanilla spells. Extra Focus deducted after successful cast is still **rejected** because it bypasses pre-cast permission/client prediction.

For a **hypothetical 10s → 0 cooldown**, a properly scoped **`SPELLMOD_COOLDOWN`** could change native pet and category recovery before `_AddCreatureSpellCooldown` (unlike a reactive delete). But:
- the **spell cooldown and category cooldown are modified separately**; category sharing may still block another rank/ability,
- actual 3.3.5 `GetCategory()`, `RecoveryTime`, `CategoryRecoveryTime` and `StartRecoveryCategory` are unknown,
- `Player::ApplySpellMod` templates run with `uint32` for Creature cooldown, so **negative flat values require demonstrated underflow/zero-clamp correctness**, not guesswork,
- the pet/client action bar and packet may not show original cooldown accurately,
- existing other owner spellmod auras and random bot scripts may compound adjustments.

**No claim that cooldown can be safely reduced to zero** is warranted until these facts are proven on test builds.

## Implications for Furious Howl (24604)

Owner `SPELLMOD_COST`/`SPELLMOD_COOLDOWN` could potentially address its **Focus 60 vs 20 and 10s vs 40s** public-reference differences, **independently** of its much harder **WotLK pet+master target filter and AP buff vs Vanilla party next-hit proc**. Solving just resource timing **does not restore the spell**. See [existing `-24604` script binding](VANILLA-PET-SPELL-EFFECT-HOOK-FEASIBILITY.md).

## Strict compatibility and owner-isolation gates

1. Confirm actual 3.3.5 spell IDs, class-family flags, effect types, rank chain, spell/category/GCD recovery and client spell tooltip. Do **not** assume same ID implies original effect.
2. **Aura delivery:** find a **safe, valid existing 3.3.5 aura spell ID** usable on the Hunter owner that can host AuraScript `DoEffectCalcSpellMod`; no guessed IDs or DBC patch may be inserted into `pet_spell` or global `spell_dbc`. If this requires a WorldDB script binding, it remains **owner-approval-gated** and cannot touch live world data in a research pass.
3. **Owner gating:** module must only maintain modifier aura on a real Hunter **in modern effective IP Vanilla stage 0–7**, not WotLK/TBC, and only while its correctly identified pet needs original rules. Alt/Random Bot owner identity follows the separately open [IP policy](IP-STAGE-ADAPTER-DESIGN.md). On IP group sync, owner change, logout, pet dismissal, stable, death and module toggle, mods must not linger or bleed into unrelated spells.
4. **Exact spell targeting:** spell-family class mask alone might cover unrelated player/pet spells. The global `OnIsAffectedBySpellModCheck` hook can only **add applicability by returning false**, not reject a modifier that already matches family flags. It receives spell infos and mod, but **no current caster or IP stage argument**. Any override would need rigorously verified modifier **provenance**, spell ID, aura ownership and no interference with other scripts; default `true` for everything else.
5. **Other mods:** confirm stacking with normal hunter pet talents, legacy trainer passives, IP damage modifiers and existing SPELLMOD_COST/COOLDOWN. A new owner modifier can also alter the Hunter's client-visible modifier packets and potentially Hunter spells with matching family flags.
6. **Manual and autocast:** verify both `PetHandler.cpp` and `PetAI.cpp` use expected native costs/cooldowns (and unearned spells remain blocked via independent WP-03). Triggered pet casts may explicitly **ignore Focus/cooldowns** and need separate tests.
7. **Source-only boundary:** no global `SpellInfo` mutation, native `spell_script_names` replacement, DBC modification, core/Pet.cpp alteration or cross-era skill rewriting is approved.
8. **No user-visible claim of success** from this source proof. At most an owner-approved, logging-only probe could verify cost/cooldown/owner aura dispatch, without applying a change until staging backups and strict negative controls are ready.

## Related important target-selection discovery

`Spell::CallScriptObjectAreaTargetSelectHandlers` in [Spell.cpp lines 9051–9063](https://github.com/azerothcore/azerothcore-wotlk/blob/master/src/server/game/Spells/Spell.cpp) loops through **each loaded SpellScript sequentially** and passes the **same mutable target list** to each. `Spell::SelectImplicitAreaTargets` calls those handlers **after** its area search, then commits surviving targets. This confirms:
- The existing `spell_hun_target_self_and_pet` rank-wide Furious Howl filter can **remove party units before a later script sees them**.
- A later script cannot **just decline to filter** and recover members already removed. It would need source-proof that it runs in a safe order or explicitly reconstructs valid targets; **no reliable solution demonstrated**. Adding party members after previous filtering could bypass line-of-sight, range, grouping or other checks and is not approved.
- `Spell::LoadScripts` builds the list from registered scripts. The source code reviewed here confirms traversal order, not the actual ordering/configuration of live DB `spell_script_names` or the safety of manipulating it.

## Next research milestone

**Prioritise one read-only pilot**: map **Lightning Breath Rank 5 / Furious Howl Rank 1** installed spell family flags, cost, three cooldown fields, rank relation, WotLK target effects and all active script bindings, and determine whether an existing valid spell-modifier aura can scope just these pet SpellIDs. Until then `FOCUS_COOLDOWN_POSSIBLE_BY_OWNER_SPELLMOD`, **NOT `RESTORED`**.

Still independent: [WP-03](AUTO-SPELL-LEARNING-FEASIBILITY-DECISION.md) has **no demonstrated pet pre-learn veto**, so authentic Beast Training remains blocked.
