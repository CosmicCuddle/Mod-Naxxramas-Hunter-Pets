# Project handover — Naxxramas Hunter Pets

**Status:** Concept/research only. No pet-system implementation has been approved, written, built, installed, or tested as part of this project.
**Owner:** CosmicCuddle
**Repository:** https://github.com/CosmicCuddle/Mod-Naxxramas-Hunter-Pets
**Target server:** AzerothCore WotLK 3.3.5a, with Individual Progression (IP), mod-playerbots, and Mod-Naxxramas-Core.
**Prepared:** 16 October 2026

## Vision

Recreate the *complete, meaningful Vanilla Hunter pet journey* on a WotLK 3.3.5a server: finding/taming beasts, learning their abilities through use, training abilities and ranks, loyalty, happiness, feeding, pet families and strengths, individual creature characteristics, pet levelling, stabling, and distinct PvE/PvP niches. Rare creatures should matter when they historically have a distinctive property or offer a sought-after skill—not receive invented blanket rarity bonuses.

**Specific examples:** Son of Hakkar should be sought for Lightning Breath Rank 6; Broken Tooth should be investigated for its original 1.0-second attack speed and PvP value. These are requirements to research/validate, not claims that implementation is done.

## Decisions already made

1. Build and validate this as a **separate experimental module**. Consider *migration* into Mod-Naxxramas-Core only after full acceptance, explicit owner approval, and a rollback plan.
2. **All Vanilla IP progression stages** should retain the Vanilla Hunter pet system, up to the actual transition into TBC. Do **not** change to TBC rules halfway through Vanilla.
3. **TBC/WotLK treatment is deferred**. Do not silently force Vanilla mechanics upon post-Vanilla characters or choose a later-era conversion policy without approval.
4. **All custom implementation belongs to this module**, including future SQL migrations and optional supporting assets. **Do not modify AzerothCore, mod-playerbots, Individual Progression or Mod-Naxxramas-Core source.**
5. First discover and measure what can truly be done with existing AzerothCore hooks and public interfaces. A missing required hook is a **hard architectural blocker**, to report—not permission to edit upstream.
6. **Documentation/research phase only** at present. No code, SQL, DBC, client patch, or live-server change until separately commissioned.
7. Use historical **late Vanilla / 1.12-era** behaviour as the research baseline *provisionally*. Earlier patch oddities (such as old Lupos damage) need separate owner decisions; do not implement them by assumption.
8. Human Hunter experience comes first; Playerbot/Alt Bot support is necessary before release, but cannot dictate a shortcut that skips meaningful player taming and progression.

## Related systems and boundaries

- Naxxramas Core: https://github.com/CosmicCuddle/Mod-Naxxramas-Core
- Its architecture and release rules: https://github.com/CosmicCuddle/Mod-Naxxramas-Core/blob/main/ROADMAP.md
- AzerothCore: https://github.com/azerothcore/azerothcore-wotlk
- Individual Progression: https://github.com/ZhengPeiRu21/mod-individual-progression
- Playerbots: https://github.com/mod-playerbots/mod-playerbots

Naxxramas Core already handles bot talents and consumables, including Hunter pet raid support. The experimental pet project **must not** overwrite those systems, bot equipment, unrelated talent rules, or learned abilities. Investigate the exact IP branch installed by the owner—IP may already implement some Hunter pet catch-up rules.

## Reading order for the implementation team

**For a future editor, start with [DEVELOPER-START-HERE.md](DEVELOPER-START-HERE.md) and [BLUEPRINT.md](BLUEPRINT.md).** The master blueprint is now the design index and the root README provides full navigation.

1. This handover + [decisions](DECISIONS.md): agreements and forbidden changes.
2. [Master blueprint](BLUEPRINT.md), [Vanilla specification](VANILLA-FEATURE-SPEC.md), [all families](VANILLA-FAMILY-REFERENCE.md), [ability/rank catalogue](ABILITY-CATALOG.md), [special beasts](RARE-AND-SPECIAL-BEASTS.md): the *desired gameplay*.
3. [Source audit](ENGINE-SOURCE-AUDIT.md), [technical feasibility](TECHNICAL-FEASIBILITY.md), [risk gates](RISK-AND-FEASIBILITY-GATES.md): the *hard module-only blockers*.
4. [State machines](STATE-MACHINES.md), [persistence and API](PERSISTENCE-AND-API.md), [IP/bot interoperability](IP-PLAYERBOTS-INTEROPERABILITY.md), [player-facing UI](PLAYER-EXPERIENCE-AND-UI.md): the *proposed architecture*.
5. [QA matrix](QA-ACCEPTANCE-MATRIX.md), [testing/rollback](TEST-AND-ROLLBACK.md), [roadmap](IMPLEMENTATION-ROADMAP.md): the *evidence, staging and release gates*.
6. [Reference bibliography](REFERENCE-BIBLIOGRAPHY.md), [research register](RESEARCH-REGISTER.md), machine-readable `../data/reference/*.csv`: provenance and gaps. **All datasets are research-only, not verified native 3.3.5 mappings**.

## Development handoff contract

Before writing feature code, the module editor should report:
- Exact deployed AzerothCore, IP, Playerbots and Naxxramas Core revisions and available hooks.
- An inventory of existing Hunter pet spells, database tables, pet-loading/level-up paths, taming flow and conflicting auto-learning.
- A **module-only feasibility verdict by feature**: possible, possible with compromises, or blocked.
- A small proof-of-concept proposal, not a complete implementation.
- Data migration and reversibility assessment for **existing** pets and bots.
- An updated estimate and a list of decisions requiring owner approval.

**No assertion of build, CI, live testing, or release without evidence.** Always distinguish planned / drafted / compiled / installed / tested / released. If a strict, accurate implementation requires editing upstream, stop and discuss the limitation with CosmicCuddle.

## Explicit non-goals for this phase

No premature TBC/WotLK rework; no AzerothCore edits; no Playerbots edits; no IP edits; no unreviewed client/DBC patch; no altering live creature templates globally; no automatic irreversible pet abandonment; no direct merge into Naxxramas Core; no deployment to the production realm.
