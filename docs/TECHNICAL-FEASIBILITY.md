# Technical feasibility and architecture review

**Preliminary, source-informed feasibility review (16 October 2026).** Not a build result or guarantee. The *exact installed revision* must be audited before implementation.

## Non-negotiable architecture

- Independent module in this repository during experiments.
- Use existing AzerothCore module interfaces, `PetScript`, `PlayerScript`, `CreatureScript`, `SpellScript`, `CommandScript` or other public interfaces **only after checking their actual signatures**.
- Module-owned additive SQL tables/data; migration has explicit down/rollback instructions. Never silently overwrite shared core DB rows.
- No edits to AzerothCore, IP, mod-playerbots or Mod-Naxxramas-Core.
- No assumed client DBC patch, custom executable, or rewritten packet handler.
- Strict configuration off by default, exact IP-era gating, optional bot gating, verbose diagnostic mode with rate limiting.

## Confirmed source observations

AzerothCore upstream paths (not necessarily identical to deployed revision):

1. [`PetScript.h`](https://github.com/azerothcore/azerothcore-wotlk/blob/master/src/server/game/Scripting/ScriptDefines/PetScript.h) exposes `OnInitStatsForLevel`, `OnCalculateMaxTalentPointsForLevel`, `CanUnlearnSpellSet`, `CanUnlearnSpellDefault`, `CanResetTalents`, `OnPetAddToWorld`.
2. [`Pet.cpp`](https://github.com/azerothcore/azerothcore-wotlk/blob/master/src/server/game/Entities/Pet/Pet.cpp) still uses `POWER_HAPPINESS`, `LoseHappiness` and `GetHappinessState`. **Don't reimplement existing happiness blindly.**
3. In `Guardian::InitStatsForLevel`, Hunter pets are assigned `BASE_ATTACK_TIME` rather than the tamed creature's original `BaseAttackTime`; the `OnInitStatsForLevel` script call provides a potential module-only interception point for restoring specific speeds.
4. `Pet::InitLevelupSpellsForLevel` **actively calls `learnSpell`** for family level-up/default pet spells. The listed `CanUnlearnSpellSet` and `CanUnlearnSpellDefault` hooks guard an **unlearn** path, **not** a generic `CanLearnPetSpell` check. Therefore *stopping WotLK auto-grants at source is not proven module-only feasible*. A safe, deterministic alternative must be investigated and tested for timing, action bar, persistence and replication. A periodic remove/regrant hack is **not accepted as an architecture** without review.
5. `Pet::GivePetLevel` calls `InitStatsForLevel`, `InitLevelupSpellsForLevel`, and `InitTalentForLevel`. Level-up, taming, load, stable and resummon transitions need explicit event-path mapping.
6. Hunter pet normalization includes physical damage school and family-based stats. Creature template fields alone will not necessarily be inherited when tamed.
7. `PetScript` has a hook affecting **pet talent point totals**, but that is not the same as restoring Vanilla training points. Pet talent UI/spell data in the 3.3.5 client remain a separate problem.
8. Individual Progression has historically advertised `IndividualProgression.HunterPetLevelFix` for pre-WotLK pet catch-up. Upstream [issue #423](https://github.com/ZhengPeiRu21/mod-individual-progression/issues/423) reports that it did not work in one build. **Verify installed IP source and runtime; do not assume the setting is present or working.**

## Feasibility matrix

| Feature | Initial verdict | Reason / investigation |
|---|---|---|
| Creature-specific attack speed | **Promising** | Pet stat initialization hook exists; prove reapplication after level/stable/relog; cap and normalize correctly. |
| Creature source identity | **Promising** | Pet entry is kept at creation; verify stable persistence and field availability. |
| Existing happiness/feeding | **Partially present** | Existing POWER_HAPPINESS and decay; compare rules/diets/damage factors and avoid double effects. |
| Loyalty, training-point accounting | **Likely module-feasible** | Own character/pet tables and events; must solve proper hooks, persistence, UX and safe abandonment. |
| Wild-tame ability discovery | **Unproven** | Need reliable tame provenance and skill-use event mapping; exact spell families and source creature data. |
| Suppress automatic WotLK skill grants | **Highest-risk / unproven** | Core's automatic teach routine has no confirmed allow/deny hook. Must prove lawful module-only handling or report blocked. |
| Remove/disable pet talent system | **Unproven** | Pet talent points hook exists, client talent UI may remain. Assess acceptable UI compromise. |
| Exact 1.12 Beast Training interface | **Not assumed** | 3.3.5 client removed original. Likely gossip/addon solution; client patch only with separate approval. |
| Per-character Vanilla IP gating | **Likely feasible, branch-dependent** | Use actual installed IP API; avoid level/zone assumptions. |
| Playerbot automatic ability management | **Unproven** | Bot module AI could retrain/replace spells; no changes to upstream Playerbots allowed. |
| Unmodified stable slot limits | **Needs investigation** | Risk of existing-pet loss/3.3.5 UI mismatch. |

## Mandatory spike order

**SPIKE 0 — Source audit, read-only:** pin live AzerothCore/IP/Playerbots revisions, build flags, relevant hooks and `character_pet`/`pet_spell`/trainer DB schemas. Establish whether IP has HunterPetLevelFix and any Hunter/Warlock pet retrofits.

**SPIKE 1 — Creature identity + attack speed:** With disposable test Hunter, tame a designated beast (e.g., Broken Tooth), verify inherited original speed after summon, level-up, stable swap, relog, death/revive, restart; test other pets unchanged and measured DPS correct.

**SPIKE 2 — Classic ability pilot:** From an approved source beast (e.g., Son of Hakkar, after ZG access), distinguish beast knows spell vs Hunter learned rank vs other pet trained. Test learn-by-use, stable, logout/restart and correct family restriction. **Prove non-automatic learning** rather than just showing a custom command grants spells.

**SPIKE 3 — Training UI/state:** prototype TP ledger and family restrictions, validate atomically, reject invalid actions without partial writes; resolve WotLK talent and client UI overlap.

**SPIKE 4 — IP + bot isolation:** mixed-IP and human/Alt/Random Bot behaviour, no downstream regressions.

If any spike fails under module-only restrictions, stop that track and record the exact blocker rather than editing AzerothCore.

## Proposed persistence design (unapproved)

Module-owned tables, versioned by migration:
- Hunter-known ability/rank catalogue (keyed by *character GUID + canonical ability/rank*).
- Pet-specific identity/mapping to native persistent pet number, pet learned ability/ranks, loyalty and TP accounting.
- Optional event journal for reconciliation and audit; no periodic forced respec without authorization.

**Critical:** native AzerothCore pet save/load remains authoritative for the live pet spell list. A separate ledger is not enough unless state is reflected reliably and safely; design idempotent reconciliation with collision protection and transactional updates. Test pet deletion, owner deletion, account transfers where supported, stable swaps, rollback and corrupted rows.

## Configuration concepts — not yet implemented

```ini
# PROPOSALS ONLY, not available in worldserver today
NaxxHunterPets.Enabled = 0
NaxxHunterPets.VanillaOnly = 1
NaxxHunterPets.IncludePlayerbots = 0
NaxxHunterPets.AllowIrreversibleRunaway = 0
NaxxHunterPets.Debug = 0
```

## Migration/integration later

Do not run this prototype in parallel with an eventual Naxxramas Core integration of the same feature. Before merging, migrate ownership of SQL/config, resolve duplicate hook registrations and name collisions, stage backup/restoration for persisted skill/loyalty data and verify that removing the standalone module does **not** make existing pets unusable. Require explicit sign-off.

## Source links

- AzerothCore PetScript API: https://github.com/azerothcore/azerothcore-wotlk/blob/master/src/server/game/Scripting/ScriptDefines/PetScript.h
- AzerothCore pet implementation: https://github.com/azerothcore/azerothcore-wotlk/blob/master/src/server/game/Entities/Pet/Pet.cpp
- Classic Petopia training: https://www.wow-petopia.com/classic/training.php
- Classic Petopia abilities: https://www.wow-petopia.com/classic/abilities.php
- IP pet level issue: https://github.com/ZhengPeiRu21/mod-individual-progression/issues/423
