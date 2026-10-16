# Original Vanilla Hunter pet spell effects — module-only script feasibility audit

**16 October 2026 — research-only.** This is a **public-source API review**, not a working C++ module, verified original 1.12.1 client DBC, installed 3.3.5 build, test, migration or gameplay release. All work remains in the separate Hunter Pets repository; never modify AzerothCore, Individual Progression (IP), Playerbots or Naxx Core source. The owner reports **modern/latest IP**, with all Vanilla stages 0–7 as the initial intended ruleset, but compiled build, spell data and exact era API remain unverified.

## Executive answer

**Some effects have credible module scripting hooks**: per-target hit damage, aura amount, aura duration, and target selection. **No full-fidelity solution is yet proven** for original Focus costs, cooldowns, native client tooltips, pre-learning, or the existing WotLK Furious Howl target and aura behaviours. A spell ID that exists in both eras may produce a *different effect*. Do not label a whole ability RESTORED after proving one damage number.

The evidence is especially strong for a **previously overlooked, already-installed upstream binding**: public WorldDB base `spell_script_names` contains `(-24604, 'spell_hun_target_self_and_pet')`. That script registers `OnObjectAreaTargetSelect` for `TARGET_UNIT_CASTER_AREA_PARTY` but **removes all targets except the casting pet and its owner**. It is shared with WotLK **Call of the Wild 53434**. Simply attaching an extra party-target script to 24604 would **not guarantee** that party members survive this existing filter or preserve WotLK behaviour. Negative first-rank bindings apply across the chain per [AzerothCore spell_script_names docs](https://www.azerothcore.org/wiki/spell_script_names), contingent on actual `spell_ranks` metadata. A separate module-owned test and era-conditional **complete target/effect strategy** is required.

## Source-pinned extension points

| Public source | Reviewed file blob SHA | Verified interface / limitation |
|---|---|---|
| [SpellScript.h](https://github.com/azerothcore/azerothcore-wotlk/blob/master/src/server/game/Spells/SpellScript.h) | `473c2d36a5845fb0239bda1cf7b2f2f70a8e9eb2` | `OnHit`/`AfterHit`, `OnEffectHitTarget`, `SetHitDamage`, `SetEffectValue` (only in effect hook), `OnObjectAreaTargetSelect`; `PreventHitDefaultEffect` expressly **does not work for aura/damage/heal** or already-handled effects. AuraScript `DoEffectCalcAmount`, `OnEffectProc`, `SetDuration`/`SetMaxDuration` |
| [SpellScript.cpp](https://github.com/azerothcore/azerothcore-wotlk/blob/master/src/server/game/Spells/SpellScript.cpp) | `d1ebc7ee4e7152d042730e1cb7c43f3e81b85a20` | `SetHitDamage` requires a **target hook** and writes `Spell::m_damage`; `SetEffectValue` requires an **effect hook** and writes `Spell::damage`. These are **not interchangeable** |
| [AllSpellScript.h](https://github.com/azerothcore/azerothcore-wotlk/blob/master/src/server/game/Scripting/ScriptDefines/AllSpellScript.h) | `b9f2fa1f6bb2a49377617a193bef1580223fd247` | `OnSpellCheckCast` can reject casts; `OnCalcMaxDuration(Aura const*, int32&)` can adjust calculated aura duration. These are not global grant-veto or client DBC APIs |
| [SpellAuras.cpp](https://github.com/azerothcore/azerothcore-wotlk/blob/master/src/server/game/Spells/Auras/SpellAuras.cpp) | `c309ee31544cd3345812329794b3af12dc6d881d` | `Aura::CalcMaxDuration` calls `OnCalcMaxDuration` on each aura, after native duration lookup. Owner-aware duration override is **source-plausible**, not tested |
| [SpellEffects.cpp](https://github.com/azerothcore/azerothcore-wotlk/blob/master/src/server/game/Spells/SpellEffects.cpp) | `d684660c9959422202546e944f039beb392a117f` | `Spell::EffectThreat` consumes `Spell::damage` in `SPELL_EFFECT_HANDLE_HIT_TARGET` and calls `ThreatMgr::AddThreat`. A targeted `SetEffectValue` pilot may be worth testing **only if spell effect identity matches**. No generic UnitScript ModifyThreat hook found |
| [UnitScript.h](https://github.com/azerothcore/azerothcore-wotlk/blob/master/src/server/game/Scripting/ScriptDefines/UnitScript.h) | `8df6eea403a62d189d51ce34257fd527f7ee57b8` | `ModifySpellDamageTaken`, `ModifyMeleeDamage`, `ModifyPeriodicDamageAurasTick` modify damage; **not** a general passive/cooldown/cost rewrite. IP already uses these, so test hook ordering to prevent double scaling |
| [spell_hunter.cpp](https://github.com/azerothcore/azerothcore-wotlk/blob/master/src/server/scripts/Spells/spell_hunter.cpp) | `ece192038bc4865309bc0586b0ac217f52af627e` | Lines ~1339–1364, existing `spell_hun_target_self_and_pet` **filters Furious Howl and Call of the Wild to pet+master** |
| [base spell_script_names.sql](https://github.com/azerothcore/azerothcore-wotlk/blob/master/data/sql/base/db_world/spell_script_names.sql) | `e93f9e13d5ab2c98b134c5c632fa1f6f688afa15` | Binding `-24604` at line ~162 and `53434` at ~1573 to **same target script** |
| [base spell_ranks.sql](https://github.com/azerothcore/azerothcore-wotlk/blob/master/data/sql/base/db_world/spell_ranks.sql) | `ca2ca5916cca570d219ff11088688eb4c80d24a6` | Includes Furious Howl base/rank metadata around line ~1794; **installed** table may differ |
| [Official AzerothCore core script docs](https://www.azerothcore.org/wiki/core-scripts) | online documentation | SpellScripts/AuraScripts need script registry and **WorldDB `spell_script_names` binding**; a spell can have multiple scripts, but that is not proof their combined target-filter results will preserve Vanilla semantics |

The installed worldserver's source and **active** `spell_script_names` rows are still unknown. Public base SQL is not proof of the live DB state. **Never remove/overwrite the existing global Furious Howl/Call of the Wild binding** to experiment on production. Any future module-owned SQL installation requires separate owner approval, a reversible script-specific change and an identified rollback; no current permission exists.

## Five primary candidate abilities — verdicts

### A. Screech Rank 4 — 24579

**Reference conflict:** modern Classic page: level 56, 26–46 Physical, −100 melee AP for 4 seconds. Public WotLK 3.3.5 spell reference: level 48, 27–37 Physical, −140 melee AP for 10 seconds and a 10-second cooldown.

- `SpellScript::OnHit` + `SetHitDamage`: **candidate** to set per-target Physical damage, after native calculations. Need verify actual 3.3.5 spell damage effect and order relative to IP pet damage modifications, resistances and glancing/crit calculations.
- `AuraScript::DoEffectCalcAmount`: **candidate** to adjust `SPELL_AURA_MOD_ATTACK_POWER` amount *if the actual DBC effect uses this aura type*; can't convert a different aura effect type.
- `AllSpellScript::OnCalcMaxDuration`: **candidate** to produce 4 seconds on the aura for a truly Vanilla-owned pet. Verify same aura may be cast by nonpet NPCs, target has correct caster/owner and a reliable stage.
- Native **10-second cooldown**, client tooltip and cast UI: **not solved** by the above adjustments.
- **Verdict:** `SOME_FIELDS_SCRIPTABLE_WITH_PROOF`; **FULL_FIDELITY_NOT_PROVEN**.

### B. Furious Howl Rank 1 — 24604

**Reference conflict:** Vanilla shared party next physical hit +9–11 within 15 yards; WotLK 3.3.5 pet/master AP aura +18.

- Existing upstream `spell_hun_target_self_and_pet` script **already removes party members except pet+owner**. It is registered on **`-24604` for Furious Howl rank chain**, and **53434 Call of the Wild**. Replacing that script globally breaks later-era pet behaviour; attaching another filter is not proof it overrides existing filters.
- **Target issue:** prove whether an era-aware module script could replace the targeting function safely **on Vanilla owners only**, without changing the WotLK 53434 script. Caller script order, DBC `TARGET_UNIT_CASTER_AREA_PARTY` and rank propagation are not guaranteed by source alone.
- **Effect issue:** WotLK passive AP buff is **not** a Vanilla 'next hit' proc. `OnEffectProc` could be a potential building block **if an appropriate 3.3.5 aura/trigger exists**; just modifying `DoEffectCalcAmount` on AP buff will not convert it into next-hit damage mechanics. Avoid accidentally granting both AP and one-hit damage.
- **Verdict:** `EXISTING_CORE_SCRIPT_CONFLICT`; **not safe to implement until 3.3.5 DBC/proc/targeting proof**.

### C. Lightning Breath Rank 5 — 25011

**Reference conflict:** Vanilla 78–90 Nature/50 Focus/no listed cooldown; public 3.3.5 40–52 base Nature, 20 Focus, 10-second cooldown and different scaling.

- `SpellScript::SetHitDamage` is **promising** for Nature damage **if the installed DBC effect and source caster match**.
- **Resource/cooldown:** The inspected `SpellScript.h` hooks include cast checks and hit/effect callbacks but **do not expose a documented per-owner `SetFocusCost` or `SetSpellRecoveryTime`**. AzerothCore `Spell::TakePower`, `CheckPower` and native cooldown logic work from loaded spell data and native `m_powerCost`. A second postcast Focus charge is **not equivalent**: missing Focus/cooldown checks and client predictions could allow unauthorized casts or desync.
- Historical Son of Hakkar **Rank 6 (25012)** must be audited **separately**. Never infer its availability, damage or native learned state from 25011.
- **Verdict:** `DAMAGE_PILOT_POSSIBLE`, **full Focus/cooldown/client fidelity blocked pending further engine API evidence**.

### D. Growl Rank 2 — 14916

**Reference conflict:** Classic reference base 65 threat vs WotLK database 65 + per-level term; both use Focus per cast, while Vanilla Growl's **Training Point** cost is zero.

- `Spell::EffectThreat` in public `SpellEffects.cpp` calls `ThreatMgr::AddThreat(unitCaster, float(damage), spellInfo, true)` on hit. `SetEffectValue` can adjust an in-flight effect value **only from a matching effect hook**, but this must be proven for the actual 14916 spell effects; **`SetHitDamage` is not a generic threat amount setter**.
- No general `ModifyThreat` was found in inspected public `UnitScript.h`. Threat from other pet actions/taunt/proc must remain unaffected.
- Native threat already stacks with IP pet damage/XP-era rules; test other enemies, taunt immune and tank target changes.
- **Verdict:** `THREAT_EFFECT_HOOK_PILOT_UNVERIFIED`; no gameplay-proof claim.

### E. Great Stamina Rank 1 — Vanilla 4187 vs WotLK pet talent 61686

**Reference conflict:** Original-style trainer passive +3 **flat Stamina**; WotLK pet talent +4% **Stamina** with a **different spell ID**.

- An `AuraScript::DoEffectCalcAmount` callback may alter the amount **within an aura of the same type**, not convert a **percentage Stamina aura** to **flat Stamina** by simply assigning the number 3.
- Need verify whether original passive **4187** still exists in **installed** 3.3.5 SpellStore/client as a *correct flat stat aura*. If absent or altered, full support may require a verified different compatible spell/aura or owner-approved server+client data; **no new DBC asset allowed in this research phase**.
- Native 3.3.5 pet talent auras and family passives must not double stack. IP already applies owner-related pet modifiers. Recompute **pet Stamina and actual MaxHealth**, stable/relog and talent refresh.
- **Verdict:** `EXISTING_FLAT_AURA_ID_UNVERIFIED`, **NOT equivalent to WotLK talent**.

## What a standalone script can and cannot promise (current public evidence)

| Mechanic | Public hook candidate | Current research confidence |
|---|---|---|
| Target-specific spell damage | `OnHit + SetHitDamage`, or `UnitScript::ModifySpellDamageTaken` | **SUPPORTED API, real ability/era order UNTESTED** |
| Aura bonus amount of same aura type | `AuraScript::DoEffectCalcAmount` | **SUPPORTED API, actual spell effect DBC UNTESTED** |
| Aura duration per cast | `AllSpellScript::OnCalcMaxDuration`, `Aura::SetDuration` | **SUPPORTED API, owner/target filtering UNTESTED** |
| Target selection | `OnObjectAreaTargetSelect` | **SUPPORTED API, existing Furious Howl filter conflict** |
| Proc on next physical hit | `AuraScript::OnEffectProc` candidate | **Hook exists, compatible aura/proc and 3.3.5 spell not established** |
| Threat component of Growl | Effect-specific `SetEffectValue` candidate | **Effect type and handler ordering not yet proven** |
| Per-owner Focus cost | No straightforward documented SpellScript hook in inspected interface | **NOT PROVEN** |
| Per-owner native cooldown | No straightforward documented SpellScript hook in inspected interface | **NOT PROVEN** |
| Client spellbook, tooltip, spell description and visual | Server spell scripts do not edit existing client Spell.dbc | **NOT SOLVED without separate compatible client presentation path** |
| Suppress native auto-learnt rank before client/passive change | No generic PetScript pre-learn veto found | **WP-03 HARD BLOCKER** |

## A safer alternative to an all-or-nothing rewrite

Proceed **mechanic by mechanic**, but *do not enable* any part that creates misleading client behaviour or valuable-pet data risk. The first best research-only, future **staging-approved** spike is **Screech R4 numeric damage + AP debuff amount/duration**, because all three changes use documented candidate hooks; it does **not** become an approved gameplay fix until cooldown, client description, spell-grant eligibility and stable/relog are also tested. `Lightning Breath` damage-only would be the second pilot, but Focus/cooldown mismatch is a core limitation. **Furious Howl is explicitly deferred** until target-script collision and aura proc semantics are resolved.

### Planned owner-approved disposable staging proof (not performed)

1. Pin exact compiled worldserver/IP/Playerbots/Naxx Core version, spell script registration and installed server/client DBC SHA. Back up characters and world DB, module source/config and working DBC copies; verify restore on isolated staging **before** any spell script binding edits.
2. Record baseline **six** spell entries and rank chains + 3.3.5 effect indexes/types, resource, cooldown, actual tooltip. Query active `spell_script_names` for `-24604`, `53434`, the five pilot spell IDs and any additional ranks **read-only**.
3. Start with **logging-only** module hook probe (after explicit approval) and measure cast/hit/aura/client effect/timing on one *disposable* stage-0 human Hunter and a stage-7 Vanilla Hunter, stage-8 TBC control, Random Bot/Alt Bot Hunter controls, a Warlock and wild creature sharing an ability ID.
4. For Screech, independently verify **damage, AP debuff amount, aura duration, cooldown and displayed tooltip**. A three-field success with an erroneous client tooltip/cooldown is a **partial result**, not full Vanilla restoration.
5. For Furious Howl, log **target list before/after the native filter** and active SQL bindings, group party next-hit/buff target state, post-Vanilla WotLK outcome. Stop if compatibility requires unsupported script order or replacing a global binding.
6. For Lightning Breath, separate damage from **Focus checks, cooldown and client prediction**. Do not add a post-cast penalty as a workaround.
7. For Growl, validate threat calculations **against actual generated enemy threat**, including classic rank and WotLK owner stage controls; no double IP damage modifier.
8. For Stamina, verify stat *and* HP after gain/remove/relog/stable, WotLK talent controls and passive regrant paths.
9. Disable module on staging, verify native WotLK pet spellbook, AP auras, talents, autocast and normal target selection return unchanged. Confirm **all source changes exclusively module-owned**.

**No source change, SQL migration, DBC edit, build, runtime experiment or owner approval has occurred.** Full mechanic restoration and strict native auto-learn interception remain unproven.
