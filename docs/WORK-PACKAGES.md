# Developer work packages — actionable milestone cards

**PLANNING ONLY:** Not a request to implement code. All packages are blocked from starting beyond read-only analysis until explicitly approved.

This file converts the [unified blueprint and roadmap](UNIFIED-BLUEPRINT-AND-ROADMAP.md) into auditable developer work units. The authoritative product scope remains [Vanilla specification](VANILLA-FEATURE-SPEC.md). Stage gate definitions in [risk register](RISK-AND-FEASIBILITY-GATES.md).

## Status vocabulary and rules

`PLANNED` → `RESEARCHING` → `DESIGNED` → `APPROVED_FOR_IMPLEMENTATION` → `IN_DEVELOPMENT` → `BUILD_VERIFIED` → `STAGING_VERIFIED` → `IN_GAME_VERIFIED` → `OWNER_ACCEPTED`.

`BLOCKED`/ `DEFERRED` are parallel statuses. An engineer cannot use `STAGING_VERIFIED` without exact test environment and logs; cannot use `OWNER_ACCEPTED` without explicit user approval. Current statuses below are `PLANNED` only.

## WP-00: Baseline and reproducible source audit — top priority

**Current:** PLANNED. **Type:** read-only research, safe to assign before implementation approval. **Dependencies:** access to deployed revisions and schema.

**New source-pinned priority (16 October 2026):** see [Pet lifecycle source trace](SOURCE-PINNED-PET-LIFECYCLE-AUDIT.md). A newly tamed pet gets WotLK Hunter−5 catch-up **before** OnPetAddToWorld; existing pet OnPetAddToWorld fires **before asynchronous saved-spell load and auto-grant**. Native pet data and spells save in separate transactions. All three must receive explicit feasibility verdicts, not merely be flagged as vague risks.

**Questions:** Which pet operations are native? Who saves what? Where do Core, IP, Playerbots and Naxx Core mutate pet levels/spells/talents? Which scripts can observe these transitions? Are existing DBC skills fully compatible?

**Deliverables:**
- Version-pinned C++ call graph for tame/owner/pet number, cast/skill use, learnSpell/InitLevelupSpellsForLevel, GivePetLevel, stable/save/load, hunter-known spells and pet talent init.
- Available `PetScript`, `PlayerScript`, `AllSpellScript` and other relevant callbacks, with exact signatures/timing and potential collisions.
- Read-only `acore_characters` tables inventory: pet identity, owner, stable slots, native spells, spell cooldowns/action bar; expected foreign-key/transaction behaviour.
- Inventory of live hunter/pet rows, sampled without revealing identifiable player data.
- IP pet level override and actual expansion-tier resolver; Playerbots and Naxx Core interactions.
- Source/SQL/DBC assets requiring backup, proposed staging environment, first-pass feasibility for each product pillar.

**Exit:** [engine audit](ENGINE-SOURCE-AUDIT.md) revised with deployed code SHA, file/line citations, explicit possible/partial/blocked table and owner-readable conclusion. No code changes.

## WP-01: Historical data authority and creature-to-skill database

**Current:** PLANNED. **Type:** historical research and data preparation. **Dependencies:** Petopia data and current world/DBC access.

**Deliverables:**
- Original creature IDs, NPC name, family IDs, zone/map, levels, rare/elite, tame flag, IP availability, natural skills/ranks and attack speed. Start with known named pilots; expand to all Vanilla pet sources.
- All 17 family profiles, diet, stat modifiers, active-ability compatibility and unusual traits with historically correct patch references.
- All rank costs/level caps, trainer price/rank sources, Classic skill ID, deployed Spell.dbc ID, required missing/custom DBC changes and per-rank focus/cooldown/effects.
- Field-by-field verification of **111 candidate rank rows** in `vanilla-ability-ranks.csv`; blanks remain blanks until proven. Verify aggregate against independent Classic references.
- Reconcile contradictory information between original Vanilla, Classic Era, TBC, WotLK, current DB and emulator implementations.

**Exit:** machine-readable tables are consistent and every production-candidate row has a source, patch, confidence, runtime verified flag, and functional test coverage; no invented IDs.

## WP-02: Per-creature tame identity and attack-speed pilot

**Current:** PLANNED. **Type:** experimental code proposal, approval required. **Dependencies:** WP-00.

**Pilot:** disposable Vanilla Hunter tames Broken Tooth; record its exact creature entry, native pet ID and correct speed; compare with standard-speed cat and known fast non-cat. Normalize autoattack damage so faster ≠ free flat DPS increase.

**Lifecycle:** raw tame, XP increase, stable A→B→A, logout, restart, resurrection, movement/auras. **Cross-control:** 3.3.5 Hunter after IP TBC and Warlock pet unchanged.

**Exit:** stable module-only hook architecture, retained identity, measured speed and DPS, logs, full rollback and no upstream changes. If identity or reinit hook cannot preserve state, stop.

## WP-03: WotLK auto-spell and client-feasibility GO/NO-GO

**Current:** SOURCE RESEARCH IN PROGRESS; **no C++ implementation or staging test approved or run**. **Type:** research then permission-gated experimental spike. **Dependencies:** WP-00.

**Core question:** `InitLevelupSpellsForLevel` unconditionally teaches family/default ranks on several paths. **Current public PetScript has no generic learn-veto and no proven after-native-async-spells-loaded hook**; OnPetAddToWorld is early on an existing pet. Explore and prove a proper module-only event bridge, not a patch that silently strips spells whenever convenient. Can all automatic WotLK skill grants be prevented **only inside module APIs**? Are pet talent trees and old abilities accessible in the 3.3.5 client?

**Evidence from current public core:** [Auto-spell decision](AUTO-SPELL-LEARNING-FEASIBILITY-DECISION.md), [16 option matrix](../data/reference/auto-spell-interception-candidates.csv). `SpellMgr::LoadPetLevelupSpellMap` uses **global** 3.3.5 DBC family skill lines; `Pet::learnSpell` directly grants ranks. `OnSpellCheckCast` offers cast denial, **not** a veto of learning/passives/packets. `isBeingLoaded` becomes false after the original spellbook packet on reload. No strict module-only solution demonstrated.

**Protocol:** [Full WP-03 staging protocol](WP03-AUTO-SPELL-LEARNING-STAGING-PROTOCOL.md); instrument controlled test (only after approval), reproduce level-up, initial tame, logon, summon, stable, pet respec, client spell packet/action bar update, server restart. Document every automatic grant path and safe interception order. Compare exact PetScript hooks, not only their names.

**Required exit:** detailed GO/NO-GO with reproducible evidence for *all* paths. Merely deleting a free spell once is **not** a pass. If impossible, document exact architectural blocker and offer truthful reduced project scope; **do not alter AzerothCore**.

## WP-04: Earn a rank by taming and using a beast

**Current:** PLANNED. **Dependencies:** WP-01 and WP-03 GO.

**Pilot:** a legally available Son of Hakkar casts Lightning Breath Rank 6; Hunter learns it from a successful eligible use; second compatible Wind Serpent can be trained under level/TP/family rules. Record catalogue separately from per-pet spells. Broken cast, no matching source, nonowner, wrong family, underlevel and replay are denied.

**Exit:** complete tame→learn→stable→teach→relog→restart demonstration with exact rank/DBC evidence, no WotLK regrant.

## WP-05: Loyalty, happiness and feeding

**Current:** PLANNED. **Dependencies:** WP-00, WP-02 identity proof.

Restore six loyalty levels with documented time/XP, positive/negative happiness impacts and pet TP budget interaction. Reuse native 3.3.5 hunger/happiness only if exact game mechanics fit; verify diet, food levels, damage modifiers, desynchronization and pet XP caps. Do not enable pet desertion before separate approval.

**Exit:** deterministic rank transitions, persistence and rollback, zero double bonuses; distinguish historical mechanics from CMaNGOS implementation details.

## WP-06: Classic Beast Training and authoritative persistence

**Current:** PLANNED. **Dependencies:** WP-03 GO, WP-04, WP-05.

Server checks Hunter learned rank, pet family, level, TP, active count and correct target owner. Rank replace/refund and respec must be atomic with native pet spell list. Mutating client commands are verified and idempotent; stale requests rejected. Test negative TP and zero-TP Growl.

**Exit:** [QA TP/SKL/ENG](QA-ACCEPTANCE-MATRIX.md) passes, including failure/crash, character save, stable, logon and 300-point checks.

## WP-07: Full Vanilla families, special creatures, spell effects

**Current:** PLANNED. **Dependencies:** WP-01, WP-02, WP-04, WP-06.

Implement validated family multipliers only once; complete approved beast-specific attack speeds and source abilities; restore Classic-ranked damage, resistances, focus, threat, buffs, Poison/Howl/Screech, Shell Shield, Charge and other effects where 3.3.5 differs. Confirm talent and IP stat scaling. Test all families and ranks, plus raid-specific special beasts.

**Exit:** full 17-family and 21-name coverage, every rank from validated table testable, no unexplained Vanilla power inflation.

## WP-08: Human training UI

**Current:** PLANNED. **Dependencies:** WP-06.

Design an accessible NPC gossip prototype in unchanged 3.3.5 client; server owns training decisions. Optional standalone N-themed addon requires a separate scope/approval and real 3.3.5 API verification. Do not replace the old client executable by stealth. Present ability-known vs pet-trained, TP cost, rank details, unavailable reasons, confirmation and cancel. Preserve normal pet action bar.

**Exit:** complete level-10 player journey, independent repeated control tests, no inaccessible buttons/client taint.

## WP-09: IP, bots and full acceptance

**Current:** PLANNED. **Dependencies:** previous implementation packages.

Ensure all Vanilla IP stages use same rules, with level-60 TBC owner as counterexample; no unfinished cross-era destructive transition. Separate human/Alt Bot/Random bot. Random bot provisioning is not equivalent to a character knowing every skill; bot-only legal policy requires approval. Naxx Core consumable and NT1 behaviour remain intact.

**Exit:** read-only migration preview, staging restored from backup, integration/raid test matrix, honest pass/fail, owner sign-off.

## WP-10: Production rollout and optional Naxx Core migration

**Current:** DEFERRED, not authorised. **Depends on:** accepted standalone build.

Document staged deployment, maintenance window, exact DB migration/up/down scripts, versioned client assets, player announcements, support, telemetry and fallback. Later propose separate PR to integrate Naxx Core, reconcile SQL/config ownership, and prove only one copy of hooks executes. No migration/merge assumed.

## Per-package review template

```text
PACKAGE:
STATUS / LAST REVIEWED:
OWNER DECISION IDs:
SOURCE REVISIONS (AC/IP/Playerbots/NaxxCore/client):
HISTORIC REFERENCES / PATCH:
REQUIREMENT IDS:
EXACT MODIFIED FILES:
BUILD STATUS:
TEST IDS PASS / FAIL / BLOCKED:
KNOWN LIMITATIONS:
DATA/CONFIG/DBC CONSEQUENCES:
BACKUP & ROLLBACK VERIFIED:
NEXT DEPENDENCY:
REQUESTED APPROVAL:
```

The roadmap's old **42–80 developer-day** figure is an initial unproven planning interval. Do not simply sum overlapping work packages. Publish more defensible estimate with uncertainty and evidence once WP-00/03 have resolved critical unknowns.
