# Project handover — Naxxramas Hunter Pets

**Status:** Concept/research only. No pet-system implementation has been approved, written, built, installed, or tested as part of this project.
**Owner:** CosmicCuddle
**Repository:** https://github.com/CosmicCuddle/Mod-Naxxramas-Hunter-Pets
**Target server:** AzerothCore WotLK 3.3.5a, with Individual Progression (IP), mod-playerbots, and Mod-Naxxramas-Core.
**Prepared:** 16 October 2026

**Read-only DBC data collection:** [MobaXterm and HeidiSQL checklist](READ-ONLY-WOTLK-PET-DBC-INPUT-CHECKLIST.md) explains precisely which loaded 3.3.5 file hashes and world override row counts are needed before claiming a spell ID is present; nothing executed on server.

**Latest source-level effect-hook audit:** [module-only effect feasibility](VANILLA-PET-SPELL-EFFECT-HOOK-FEASIBILITY.md), [18-case data](../data/reference/vanilla-spell-effect-hook-feasibility.csv). **Furious Howl 24604 already has a WotLK `-24604` target-filter binding**, shared with 53434, that strips party targets to pet+owner. SpellScript supports hit damage, effect value and AuraScript amount/duration hooks, but **strict Focus cost/cooldown/client tooltip and WP-03 pre-learn veto remain unproven**. No code or staged testing approved. Source index **120**.

**Public 3.3.5 spell-effect conflicts (new):** [five-ability case study](PUBLIC-335-SPELL-EFFECT-CONFLICT-PILOT.md), [six-case CSV](../data/reference/public-335-spell-effect-conflicts.csv), and annotated [21-ability matrix](../data/reference/wotlk-versus-vanilla-ability-audit.csv) now document **same-ID changed mechanics** for Screech 24579, Furious Howl 24604, Lightning Breath 25011, Growl 14916, and a separate WotLK % Stamina talent 61686 versus Classic trainer spell 4187. **These are public sources; no installed DBC or native auto-grant has been verified.** Never mark identical spell ID as equivalent effect or trainable rank.

**New 3.3.5 source-data research:** [WotLK DBC auto-grant comparison](WOTLK-AUTO-GRANT-DBC-COMPARISON.md) identifies three spell generation paths, with [21-ability comparison](../data/reference/wotlk-versus-vanilla-ability-audit.csv) and [17-family audit](../data/reference/wotlk-family-auto-grant-audit.csv). Public AzerothCore base `creaturefamily_dbc`, `skilllineability_dbc` and `creaturespelldata_dbc` SQL have **zero inserted records**; actual 3.3.5 server DBC and local overrides remain UNVERIFIED, so do **not invent** the automatic spell IDs. No implementation approved.  

**Modern IP planning priority (owner report, 16 October 2026):** The owner reports using the newest Individual Progression version. Prioritise modern hidden-quest stages (Vanilla 0–7; TBC 8–12; WotLK 13+) for the first module-only IP adapter. Exact compiled revision remains unverified and legacy handling is deferred rather than presumed necessary. No implementation is approved. See [IP adapter design](IP-STAGE-ADAPTER-DESIGN.md).

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

**For a future editor, start with [UNIFIED-BLUEPRINT-AND-ROADMAP.md](UNIFIED-BLUEPRINT-AND-ROADMAP.md), [DEVELOPER-START-HERE.md](DEVELOPER-START-HERE.md) and [BLUEPRINT.md](BLUEPRINT.md).** The unified programme connects requirements to work packages, acceptance cases and approval gates; the root README provides full navigation.

1. This handover + [decisions](DECISIONS.md): agreements and forbidden changes.
2. [Master blueprint](BLUEPRINT.md), [Vanilla specification](VANILLA-FEATURE-SPEC.md), [all families](VANILLA-FAMILY-REFERENCE.md), [ability/rank catalogue](ABILITY-CATALOG.md), [tame-source atlas](TAME-ABILITY-SOURCE-ATLAS.md), [spell-ID/effect cross-check](SPELL-ID-AND-EFFECT-RESEARCH.md), [trainer spell research](TRAINER-SPELL-IDENTITY-REFERENCE.md), [special beasts](RARE-AND-SPECIAL-BEASTS.md): the *desired gameplay and evidence*.
3. [Source audit](ENGINE-SOURCE-AUDIT.md), [IP pet-handling precedent](IP-HUNTER-PET-PRECEDENT-AUDIT.md), [IP stage adapter blueprint](IP-STAGE-ADAPTER-DESIGN.md), [read-only installed IP inventory](READ-ONLY-INSTALLED-IP-INVENTORY.md), [22 owner/era QA scenarios](../data/reference/ip-pet-owner-era-acceptance-matrix.csv), [source-pinned hook/lifecycle trace](SOURCE-PINNED-PET-LIFECYCLE-AUDIT.md), [auto spell learning feasibility](AUTO-SPELL-LEARNING-FEASIBILITY-DECISION.md), [WP-03 staging test protocol](WP03-AUTO-SPELL-LEARNING-STAGING-PROTOCOL.md), [16-event source index](../data/reference/public-core-pet-lifecycle-events.csv), [16 auto-spell interception candidates](../data/reference/auto-spell-interception-candidates.csv), [technical feasibility](TECHNICAL-FEASIBILITY.md), [risk gates](RISK-AND-FEASIBILITY-GATES.md): the *hard module-only blockers*. **Additional validated public-source risk:** loading an invalid pet spell triggers native **all-pets-by-spell-ID cleanup** (`DELETE FROM pet_spell WHERE spell = ?`), so never seed unverified original rank IDs into native character DB. See [packet and DB integrity audit](PET-SPELL-GRANT-PACKET-DB-SAFETY-AUDIT.md) and [10 WP-03 tests](../data/reference/wp03-pet-spell-safety-tests.csv). **Key blockers:** no per-owner pre-learn veto found in inspected public PetScript; per-owner cast denial is a possible **reduced-fidelity** fallback and is not owner approved. WotLK native ranks are re-granted after an async saved-spell load; new tames and loads force Hunter−5 pet level catchup; the native pet save spans multiple commits.
4. [State machines](STATE-MACHINES.md), [persistence and API](PERSISTENCE-AND-API.md), [IP/bot interoperability](IP-PLAYERBOTS-INTEROPERABILITY.md), [player-facing UI](PLAYER-EXPERIENCE-AND-UI.md): the *proposed architecture*.
5. [Work packages](WORK-PACKAGES.md), [QA matrix](QA-ACCEPTANCE-MATRIX.md), [testing/rollback](TEST-AND-ROLLBACK.md), [roadmap](IMPLEMENTATION-ROADMAP.md): the *dependency chain, evidence, staging and release gates*.
6. [Evidence standards](RESEARCH-EVIDENCE-STANDARDS.md), [source disagreements/licensing](REFERENCE-SOURCE-CONFLICTS.md), [read-only world audit](READ-ONLY-SERVER-DATA-AUDIT.md), [reference bibliography](REFERENCE-BIBLIOGRAPHY.md), [research register](RESEARCH-REGISTER.md), machine-readable `../data/reference/*.csv` including [requirement traceability](../data/reference/requirement-traceability.csv): provenance and gaps. **All datasets are research-only, not verified native 3.3.5 mappings**.

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
