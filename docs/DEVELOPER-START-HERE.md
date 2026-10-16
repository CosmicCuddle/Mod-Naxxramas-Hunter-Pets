# Developer start here — implementation team briefing

**STOP: This repository is a planning blueprint, not an approved build assignment.** Do not assume permission to install, merge or modify game code because this documentation exists.

## Mission in one paragraph

Rebuild the complete late-Vanilla Hunter pet experience for every character still within any Vanilla Individual Progression stage. Specific beast taming, wild learn-by-use ranks, pet trainer skills, loyalty, feeding/happiness, training points, pet families, distinct creature attack speed and pet progression all matter. This must run on AzerothCore WotLK 3.3.5a in an **isolated module only**, leave the AzerothCore/IP/Playerbots/Naxx Core sources unchanged and be safe for existing valuable pets. Merge into Naxx Core only after proven acceptance and a separate approval. TBC/WotLK design is deferred.

## Read in this exact order

1. [Unified blueprint and roadmap](UNIFIED-BLUEPRINT-AND-ROADMAP.md) — integrated vision, engineering work packages, evidence and approval gates; then [Master Vanilla blueprint](BLUEPRINT.md) — detailed gameplay invariants.
2. [Project handover](PROJECT-HANDOVER.md) — agreed user decisions and architectural boundaries.
3. [Decision log](DECISIONS.md) — open questions must stay open.
4. [Engine audit](ENGINE-SOURCE-AUDIT.md) and [risk gates](RISK-AND-FEASIBILITY-GATES.md) — first check if it's actually possible module-only.
5. [Families](VANILLA-FAMILY-REFERENCE.md), [Ability catalogue](ABILITY-CATALOG.md) and [special beasts](RARE-AND-SPECIAL-BEASTS.md) — candidate historical data.
6. [State machines](STATE-MACHINES.md), [persistence](PERSISTENCE-AND-API.md), [IP/Bot interoperability](IP-PLAYERBOTS-INTEROPERABILITY.md), [player UI](PLAYER-EXPERIENCE-AND-UI.md).
7. [Work packages](WORK-PACKAGES.md), [QA matrix](QA-ACCEPTANCE-MATRIX.md), [backup/rollback](TEST-AND-ROLLBACK.md), [roadmap](IMPLEMENTATION-ROADMAP.md), [bibliography](REFERENCE-BIBLIOGRAPHY.md), [research register](RESEARCH-REGISTER.md) and [historical evidence standards](RESEARCH-EVIDENCE-STANDARDS.md).
8. Machine-readable reference files under `data/reference/` include a 54-requirement traceability matrix and **research-only** family, special-beast and ability-rank datasets. Costs are candidates sourced from modern Classic guides; deployed spell IDs/NPC entries and original 1.12 confirmation are missing. Never feed research files directly to runtime.

## First assignment — Stage 0 RESEARCH ONLY

Return a source audit report with:
- Exact installed AzerothCore, IP, Playerbots, Naxx Core commits and configuration values; don't assume default `main`.
- Pet save/level/tame/cast/trainer pipeline annotated with actual callback signatures and timing.
- `character_pet`, `pet_spell` and related table DDL read-only inventory (deployed branch).
- Exact installed DBC spells, pet family flags, tameable NPC entries and source world SQL maps.
- **Proof/disproof** of suppressing WotLK auto-learning, capturing a successful pet skill use and persisting custom pet ranks without touching upstream files.
- Separate human, Alt Bot and Random Bot ownership map and IP-era resolver.
- Incompatibilities with WotLK pet talent UI, default pet spells, currently owned stable capacity and pet level catch-up.
- Ranked blockers and an updated effort/estimate that acknowledges uncertainty.

**No C++, SQL, addons, DBC edits, experimental feature branch or test on the live realm until approval**, although a future explicitly authorised disposable staging spike may be proposed.

## First prototype proposal (once Stage 0 approved)

**P1 — Broken Tooth speed**: stage an isolated module prototype on disposable Hunter; prove source identity and corrected 1.0-sec swing while normalising damage through relog/stable/XP.

**P2 — Son of Hakkar skill acquisition**: prove learn-by-use Lightning Breath 6, character-wide catalogue, teach compatible Wind Serpent, and **prevent auto-grant** through all core reload paths.

Only after P1 and P2 pass may loyalty/TP and full 17-family data be considered.

## Output format for development correspondence

```text
STAGE / BRANCH / EXACT SHA:
GOAL:
FILES REVIEWED (with line numbers):
HOOKS FOUND:
FEASIBLE FULL / PARTIAL / BLOCKED:
PROOF (tests/fixtures and output):
RISKS (pet loss / IP / bots / client):
PROPOSED NEXT CHANGE:
DB/CONFIG/DBC IMPACT:
BACKUP AND ROLLBACK:
APPROVAL REQUIRED:
```

**Do not say 'complete' without recorded evidence.** Document TODOs are not implemented features, and upstream project code is read-only under this assignment.

## Questions for owner, when evidence warrants

- Final 1.12 baseline or earlier patch-specific exceptions?
- What to do with already-owned modern pets / extra stable slots?
- Can an accessible NPC gossip window replace original Beast Training, or is an addon desired?
- Should pet desertion ever be enabled after fully testing?
- Bot-only legal training policy that protects human authentic learning.
- Appropriate post-Vanilla conversion once planning reaches TBC.

Ask about these as actual decisions at stage gates, not all at once in advance. None should delay purely read-only source auditing.
