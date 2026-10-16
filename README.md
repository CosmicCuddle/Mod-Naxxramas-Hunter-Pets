# Mod-Naxxramas-Hunter-Pets

> **FULL DEVELOPMENT BLUEPRINT — RESEARCH AND DESIGN ONLY**
>
> A standalone experimental AzerothCore 3.3.5a module project to recreate the **original Vanilla Hunter pet experience**. **No module C++, SQL migrations, client patch, compiled build, installation or in-game test has been performed through this planning work.**

## Vision

**Scope clarification (16 October 2026):** Aim for the **maximum faithful, safe restoration of the original Vanilla Hunter pet system**, not just a 1.12 snapshot of famous pet features. Research original Vanilla patch history as well as Classic gameplay; clearly separate agreed full-scope goals, historical variations, technical limits, and optional compromises. The planned restoration stays enabled through **all Vanilla IP stages**, while TBC decisions remain future work.

Bring back the time when a Hunter's choice of **specific beast** meant something: original per-creature attack speeds, taming to discover and use new ability ranks, teaching different pets, pet trainer skills, loyalty and happiness, feeding, training points and four active skills, unique family strengths and meaningful rare/dungeon/raid beast hunts.

Illustrative pet goals: **Son of Hakkar** for Lightning Breath rank 6; **Broken Tooth** for 1.0-second attacks; **Bloodaxe Worg** for Bite 8, Dash 3 and Furious Howl 4; **Winterspring Screecher** for Claw 8 and Screech 4.

**Locked project direction:** Vanilla Hunter pet mechanics run across **every Vanilla Individual Progression stage**, up to the actual TBC-entry milestone. Design of later TBC/WotLK pet systems is deferred. Source implementation must remain inside **this standalone module**, with **no edits** to AzerothCore, Individual Progression, Playerbots or Mod-Naxxramas-Core. A later integration into Naxx Core requires separate owner approval and migration tests.

## Three essential starting documents

1. **[UNIFIED BLUEPRINT + ROADMAP](docs/UNIFIED-BLUEPRINT-AND-ROADMAP.md)** — the complete, traceable project connecting historical requirements, engineering work, tests and approval gates.
2. **[DEVELOPER START HERE](docs/DEVELOPER-START-HERE.md)** — reading order and first read-only technical investigation.
3. **[MASTER VANILLA BLUEPRINT](docs/BLUEPRINT.md)** — detailed historical gameplay principles and engineering invariants.

## Complete blueprint documentation

### Product and historic gameplay
- [Official project handover](docs/PROJECT-HANDOVER.md) — owner requirements, limitations, future implementation process.
- [Vanilla gameplay specification](docs/VANILLA-FEATURE-SPEC.md) — tame, ability acquisition, loyalty/happiness, pet training, family balance, XP, stables and bots.
- [All **17 Vanilla pet families**](docs/VANILLA-FAMILY-REFERENCE.md) — family roles, health/armor/damage ratios, diets, unique skills and 17 source links.
- [All **21 ability names** and rank levels](docs/ABILITY-CATALOG.md) — rank requirements, TP cost examples, family restrictions, teach-through-use.
- [Rare and special pet research](docs/RARE-AND-SPECIAL-BEASTS.md) — Broken Tooth, Son of Hakkar, Bloodaxe Worg, rare speeds, special beast source skills.
- [Player experience and interface](docs/PLAYER-EXPERIENCE-AND-UI.md) — level-10 journey, accessible 3.3.5 trainer UI proposal, possible addon, stable capacity decisions.

### Engineering, compatibility and safety
- [Technical feasibility](docs/TECHNICAL-FEASIBILITY.md) — what source proves, what remains unproven module-only.
- [Engine source audit](docs/ENGINE-SOURCE-AUDIT.md) — AzerothCore hooks and classic CMaNGOS comparison references.
- [Source-pinned pet lifecycle and hook ordering](docs/SOURCE-PINNED-PET-LIFECYCLE-AUDIT.md) — exact public source file blobs/lines for tame, async load, auto-learn, level sync, spell-hit and native save.
- [IP owner-stage adapter and bot-sync design](docs/IP-STAGE-ADAPTER-DESIGN.md) — reuse Naxx Core's modern/legacy stage compatibility, distinguish pet owner vs NT1 issuer, handle IP's group bot progression synchronisation.
- [Installed IP read-only MobaXterm inventory](docs/READ-ONLY-INSTALLED-IP-INVENTORY.md) — no git pull, no SQL, no config changes; identify exact running/source versions and cross-check IP stage.
- [22-scenario IP Hunter-pet acceptance matrix](data/reference/ip-pet-owner-era-acceptance-matrix.csv) — all Vanilla stages, TBC/WotLK, Alt/Random Bots, group sync and capped XP.
- [IP Hunter-pet precedent and compatibility audit](docs/IP-HUNTER-PET-PRECEDENT-AUDIT.md) — **IP's owner-stage-aware pet damage/XP hooks are reusable as a design pattern**, but its Player spell cancellation is not a pet auto-learn veto; historical HunterPetLevelFix complaint documented.
- [Auto spell-learning module-only feasibility decision](docs/AUTO-SPELL-LEARNING-FEASIBILITY-DECISION.md) — exact global DBC-derived grant path, scope-aware cast-denial candidate, and why neither late reconciliation nor a global override proves full Vanilla fidelity.
- [WP-03 disposable staging protocol](docs/WP03-AUTO-SPELL-LEARNING-STAGING-PROTOCOL.md) — permission-gated, backup-first, logging-only sequence then approval-specific experiments, packet-state/crash controls and GO/NO-GO criteria.
- [16-event lifecycle trace (CSV)](data/reference/public-core-pet-lifecycle-events.csv) — source, stage ordering, event risks, API hook and expected test.
- [Behavioural state machines](docs/STATE-MACHINES.md) — tame, cast-to-learn, train, loyalty, stable, progression and owner isolation.
- [Persistence, native pet API and migrations](docs/PERSISTENCE-AND-API.md) — proposed module-owned tables, concurrency, idempotence, saved spells, rollback.
- [IP, Playerbots and Naxx Core interoperability](docs/IP-PLAYERBOTS-INTEROPERABILITY.md) — actual Hunter-era gating, Alt/Random bots, raid AI, strict upstream isolation.
- [Risk register and hard feasibility gates](docs/RISK-AND-FEASIBILITY-GATES.md) — what must be proven before full development.
- [QA and acceptance matrix](docs/QA-ACCEPTANCE-MATRIX.md) — reproducible test IDs across the full lifecycle.
- [Backup and rollback plan](docs/TEST-AND-ROLLBACK.md) — preexisting pets, character DB, installed DBC and safe recovery.
- [Implementation roadmap and estimate](docs/IMPLEMENTATION-ROADMAP.md) — Stage 0 audit, small prototypes, full Vanilla data, integration and initial 42–80 approximate developer-days.
- [Actionable work packages](docs/WORK-PACKAGES.md) — WP-00 through WP-10 with dependencies, deliverables, acceptance and no-go conditions.

### Research and decision provenance
- [Annotated historical/reference bibliography](docs/REFERENCE-BIBLIOGRAPHY.md) — Classic Petopia, Warcraft Wiki, Wowhead, CMaNGOS Classic, AzerothCore and comparison material.
- [Research register and missing verification tasks](docs/RESEARCH-REGISTER.md).
- [Research evidence standards](docs/RESEARCH-EVIDENCE-STANDARDS.md) — distinguish original Vanilla, modern Classic, CMaNGOS and deployed AzerothCore; document uncertain ranks such as Charge 4.
- [Agreed / proposed / open decisions](docs/DECISIONS.md).

## Machine-readable, **research-only** reference tables

| File | Contents | Readiness |
|---|---|---|
| [`data/reference/vanilla-families.csv`](data/reference/vanilla-families.csv) | **17** named families; original role, stat modifiers, diets, allowed active abilities and citations | Candidate design data, not deployed family IDs |
| [`data/reference/vanilla-ability-ranks.csv`](data/reference/vanilla-ability-ranks.csv) | **111** candidate Classic ranks with level and Petopia training cost | All costs now sourced from Petopia; **original 1.12 verification and installed server/client spell IDs still missing** |
| [`data/reference/vanilla-special-beasts.csv`](data/reference/vanilla-special-beasts.csv) | **13** high-value beast examples and documented historic traits | Names/speeds/skills are references; deployed creature IDs still unverified |
| [`data/reference/requirement-traceability.csv`](data/reference/requirement-traceability.csv) | **54** requirement links joining historical sources → developer WP → acceptance QA → decision | All implementation statuses NOT_STARTED, feasibility NOT_PROVEN |
| [`data/reference/vanilla-wild-spell-id-candidates.csv`](data/reference/vanilla-wild-spell-id-candidates.csv) | **59** wild ability-rank IDs from independent Classic emulator code | **Research candidates only:** no original 1.12 or deployed 3.3.5 DBC proof; explicit conflict flags |
| [`data/reference/vanilla-pilot-spell-effects.csv`](data/reference/vanilla-pilot-spell-effects.csv) | **13** priority ability effect test fixtures (Bite, Claw, Screech, LB6, Furious Howl etc.) | Effect values sourced from Classic references; **zero original/client/server DBC runtime mapping** |
| [`data/reference/vanilla-trainer-spell-id-candidates.csv`](data/reference/vanilla-trainer-spell-id-candidates.csv) | **52** trainer-rank ID/effect candidates, including **5 disputed resistances** | 47 **likely** Vanilla trainer ranks, five not offered according to multiple historic reconstructions; no DBC proof yet |
| [`data/reference/vanilla-all-rank-id-candidate-crosswalk.csv`](data/reference/vanilla-all-rank-id-candidate-crosswalk.csv) | **111** joined wild + trainer candidate IDs, risk tags and strict evidence fields | **Do not import into runtime**; every record still requires original/deployed DBC verification |
| [`data/reference/public-core-pet-lifecycle-events.csv`](data/reference/public-core-pet-lifecycle-events.csv) | **16** critical tame/load/level/spell/save events with public source lines and call-order | Public upstream **file-blob** proof only, **no live module feasibility or installed revision confirmation** |
| [`data/reference/ip-pet-owner-era-acceptance-matrix.csv`](data/reference/ip-pet-owner-era-acceptance-matrix.csv) | **22** IP/owner/bot/era scenarios for per-pet eligibility | Planned tests only: no installed IP build or staged pet tests verified |
| [`data/reference/auto-spell-interception-candidates.csv`](data/reference/auto-spell-interception-candidates.csv) | **16** candidate auto-rank interception paths, actual APIs and strict-vs-compromise status | Global map overrides rejected, per-pet veto absent in reviewed public sources; no runtime proof |

**Important:** 111 is the number of rank/level rows currently enumerated from the Petopia Classic guide, *not* independently certified total for original Vanilla 1.12. **Five resistance Rank 5 rows may not have been available in original Vanilla**, leaving 106 possible in-era rank records, of which two wild ranks lack a known tame source. Exact trainer availability and original DBC remain unverified. Reference catalogues can differ. CSV records and historic guides must not be inserted into an active world database without ID and patch verification.

## Historical references — research first

- [Historical Reference Foundation](docs/HISTORICAL-REFERENCE-FOUNDATION.md) — complete research scope, original patch evidence and sources ranked by confidence.
- [Original Vanilla Patch Timeline](docs/VANILLA-PATCH-REFERENCE-TIMELINE.md) — highlights pet mechanics introduced, corrected or removed between early Vanilla and the final Vanilla era.
- [90-source provenance index](data/reference/historical-source-index.csv) — source type, original patch, purpose and required validation. All entries are references; none are production-ready 3.3.5 IDs.
- [Tame & ability-source atlas](docs/TAME-ABILITY-SOURCE-ATLAS.md) — initial **141 sourced beast-to-skill-rank relationships** across 13 wild-learned abilities, with six independently referenced candidate NPC IDs.
- [Rank source coverage](data/reference/vanilla-rank-source-coverage.csv) — **57 candidate-source rank combinations**, two explicitly without known tame source, plus 52 *listed* trainer ranks (**five resistance Rank 5s are historically disputed**); all awaiting runtime validation.
- [Reference conflicts and licensing](docs/REFERENCE-SOURCE-CONFLICTS.md) — rank-count and spell-ID mismatches, out-of-era spell variants, original vs Classic bugs, and data reuse restrictions.
- [Wild spell-ID and effect audit](docs/SPELL-ID-AND-EFFECT-RESEARCH.md) — all **59** wild ranks have research-only candidate IDs; **eight level mismatches** and the out-of-era Screech 27051 found.
- [Trainer spell reference pilot](docs/TRAINER-SPELL-IDENTITY-REFERENCE.md) — highlights old flat +Stamina versus modern percent-aura and nonfree Focus cost for 0-TP Growl.
- [Full 52-rank trainer research](docs/COMPLETE-TRAINER-RANK-RESEARCH.md) — all 8 trainer ability lines, 52 candidate spell IDs and the **5 disputed resistance Rank 5** entries.
- [Original 1.12 vs deployed 3.3.5 DBC comparison protocol](docs/ORIGINAL-DBC-AND-DEPLOYED-COMPARISON.md) — reproducible, read-only original data and server/client verification steps.
- [13 priority spell-effect fixtures](data/reference/vanilla-pilot-spell-effects.csv) — rank, candidate IDs, spell-effect numbers and the full original+3.3.5 DBC proof gates.
- [Read-only world/character DB audit](docs/READ-ONLY-SERVER-DATA-AUDIT.md) — safe template/spawn/schema verification procedure before any future implementation.

## Current module-only feasibility frontier

The [source-pinned pet lifecycle audit](docs/SOURCE-PINNED-PET-LIFECYCLE-AUDIT.md) confirms on **public upstream** (not the owner's installed build) that:
- A **new tame** first gets WotLK Hunter−5 level adjustment and automatic pet ranks **before** the pet is added to the world.
- An **existing pet** triggers `OnPetAddToWorld` **before** asynchronously loading saved native spells, granting family/passive/level ranks and updating the client. Thus the AddToWorld hook cannot be trusted as a post-load reconcile point.
- `PetScript` provides no generic CanLearnPetSpell veto in the inspected public interface.
- Native pet saving uses multiple transactions; loyalty/TP module persistence needs crash-safe reconciliation.
- Public Individual Progression includes capped-level pet-XP handoff; original Vanilla tame-level/XP restoration must not duplicate or suppress it.

**New (16 October 2026):** [WP-03 research](docs/AUTO-SPELL-LEARNING-FEASIBILITY-DECISION.md) shows global skill map is built from 3.3.5 DBC and offers no public per-owner filter. `AllSpellScript::OnSpellCheckCast` may **block active casts** but not passive effects or temporary client spell packets; `Pet::isBeingLoaded()` becomes false **after** `PetSpellInitialize` on existing-pet load. Full authentic manual training remains **NOT PROVEN**; don't present a cast-denial workaround as strict Vanilla restoration.

**All remain unproven on the deployed realm.** Do not claim a working module-only solution or install any data based on these findings. See [Stage 0 research issue #1](https://github.com/CosmicCuddle/Mod-Naxxramas-Hunter-Pets/issues/1).

## Research coordination

- [Stage 0 read-only feasibility issue #1](https://github.com/CosmicCuddle/Mod-Naxxramas-Hunter-Pets/issues/1) — module-only hooks and actual core audit.
- [Historical source catalogue issue #2](https://github.com/CosmicCuddle/Mod-Naxxramas-Hunter-Pets/issues/2) — fully index original tame sources, verify client/creature IDs and resolve historical conflicts.
- [Pet spell-ID and effects research issue #3](https://github.com/CosmicCuddle/Mod-Naxxramas-Hunter-Pets/issues/3) — verify all 111 ranks, original/3.3.5 client compatibility and cross-era spell conflicts.
- [WP-03 auto-spell module-only research issue #4](https://github.com/CosmicCuddle/Mod-Naxxramas-Hunter-Pets/issues/4) — strict Vanilla training vs core auto ranks, candidate spell-cast guard, client packet/passive test and staged GO/NO-GO.
- **None of these research issues grants approval to implement code or change the server.**
- The repository deliberately combines a **historical+technical blueprint** with a **dependency-ordered work programme**. Every research fact is tied to a work package, QA test and open/approved decision where possible.

## Known architecture obstacles — don't overpromise

- **Top blocker:** 3.3.5 `Pet::InitLevelupSpellsForLevel` automatically teaches pet spells; no identified general module hook vetoes these grants. **Full authentic manual pet training may be impossible under strict module-only rules**. Prove a reliable, non-destructive approach before broad implementation.
- AzerothCore keeps happiness, but removed Vanilla loyalty/training points. Existing code must not be duplicated or subverted.
- Wild pet spell use does not necessarily have a verified successful-cast module callback; a generic cast hook is not enough.
- 3.3.5 pet talent UI differs from the missing original Beast Training UI.
- WotLK supports **four stable slots plus one active pet**, compared to Vanilla's **two plus one**. Do not delete or strand existing pets to enforce old limits.
- Client/server DBC/MPQ may need approved custom assets to recover exact original spell effect values; do not assume names imply matching effects.
- Mixed IP-era Hunter characters and Playerbot spell/AI refresh require real tests, not merely source inspection.

## Status

| Work | Status |
|---|---|
| Vision, requirements and initial research | **Documented; continuously expandable** |
| 17 families and 111 reference rank rows | **Catalogued from sources, not runtime verified** |
| Module-only feasibility | **Preliminary, major unresolved blockers** |
| Source/DB/DBC audit of actual production revision | **Not started** |
| C++ module / config / migrations / addon | **Not started** |
| Compiled build, client in-game QA, release | **Not started** |

No source editor should treat this repository as proof that the restoration is buildable in full. **Research Stage 0 comes first**. The future developer should begin with the [handover assignment](docs/DEVELOPER-START-HERE.md), not with an unreviewed game-system rewrite.

## Backups and future integration

When experimental implementation is eventually approved: back up the active module source/config, worldserver build, affected world DB rows, full character DB including native pet/stable/skill tables, and any server+client DBC/MPQ assets; **verify restore in staging** before touching valuable pets. Keep all changes in the separate module. Merge with [Naxxramas Core](https://github.com/CosmicCuddle/Mod-Naxxramas-Core) only after all acceptance gates and a separately approved persistence migration.
