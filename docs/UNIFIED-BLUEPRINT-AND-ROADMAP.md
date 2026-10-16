# Unified blueprint and development programme

**Status:** Detailed research and implementation-planning document. **No feature is implemented or approved for development.**
**Last drafted:** 2026-10-16
**Scope:** All original Vanilla Hunter-pet systems throughout *every* Vanilla Individual Progression stage.
**Integration:** Standalone experimental module first; future Mod-Naxxramas-Core integration only by separate approval.

## Why this document exists

This project is **not solely a roadmap** and **not solely a historic specification**. It is a traceable engineering programme:

```text
HISTORIC CLAIM + SOURCE
          ↓
USER-APPROVED GAMEPLAY REQUIREMENT
          ↓
CORE / CLIENT FEASIBILITY EVIDENCE
          ↓
IMPLEMENTATION WORK PACKAGE + DEPENDENCIES
          ↓
TEST CASES + REPRODUCTION EVIDENCE
          ↓
OWNER SIGN-OFF
          ↓
SAFE STANDALONE RELEASE
          ↓
SEPARATELY APPROVED INTEGRATION INTO NAXX CORE
```

No step can be inferred from the following one. A sourced Classic-era rule does not establish that 3.3.5 supports it, and a working proof of concept does not establish production safety.

## A. Blueprint fidelity model

Three independent dimensions must be tracked for **every** feature:

1. **Historical fidelity:** exact late-Vanilla behaviour, known patch ambiguity, ranked sources. Example: original attack speed, ranks, family restrictions, Trainer vs tame-acquired source.
2. **Technical fidelity:** can the 3.3.5 client, AzerothCore module system, existing DBC and native pet persistence correctly reproduce the gameplay **without modifying upstream source**?
3. **Operational fidelity:** does it work through login/logout, stabling, XP, combat, concurrent access, IP stages and bot use **without data loss**, with monitoring and rollback?

Each requirement is `NOT_RESEARCHED`, `SOURCE_CANDIDATE`, `SOURCE_VERIFIED`, `TECHNICALLY_PROVEN`, `IMPLEMENTED`, `TESTED_STAGING`, `TESTED_IN_GAME`, or `ACCEPTED`. An independent `BLOCKED` flag includes a reproducible technical explanation. Never use `SOURCE_VERIFIED` to imply `TECHNICALLY_PROVEN`.

## B. Product scope: what must eventually feel authentically Vanilla

| ID group | Gameplay pillar | Blueprint | Must be demonstrated |
|---|---|---|---|
| HUNT | Hunter class onboarding | Original tame-related quests; Feed Pet, Beast Lore, pet trainer and Beast Training workflow | A fresh Hunter can progress organically and learn how to find/train a useful pet |
| TAME | Beast hunting and pet identity | Original tameability, source NPC identity, family, colour/model, starting level, available natural skills, rare spawn/raid sources | Taming **Son of Hakkar** is different from taming a random Wind Serpent for obtaining Lightning Breath 6 |
| SKILL | Learn by using a tamed pet | Hunter-level skill catalogue distinct from pet-owned spell; trainer catalogue; deliberate abilities and ranks | Skill acquisition cannot be fabricated by pet level-up or another owner's spell cast |
| TP | Training points and active-skill limit | `pet_level × (loyalty - 1)`, costs, ranks, upgrades/refunds, four active skills, zero-TP Growl | 60/6 = 300 TP, wrong-family/rank/repeat request is rejected atomically |
| LOY | Loyalty, happiness and food | Six loyalty ranks; food/diet effectiveness, time/XP progression, happy/content/unhappy state, pet-loss policy | Pet progress survives stabling/relog; native happiness not double-applied |
| FAM | Family differences | Seventeen Vanilla families, permitted skill sets, health/armour/damage and diet, no exotic/TBC family leakage | Different pets have **historically grounded** strengths without blanket 'rare = stronger' |
| BEAST | Individual creature traits | Original per-creature attack speed and availability, special tame spells and beautiful unique appearances | Broken Tooth original speed persists, without unexplained DPS inflation |
| BATTLE | Combat effects | Vanilla spell ranks/effects, focus/autocast, procs, threat, party buffs and raid roles | Spell test fixtures show Vanilla mechanics rather than WotLK effects with an old tooltip |
| LEVEL | Pet growth, stable and ownership | Pet XP/level, source identity, native per-pet state, stabling without loss, 3.3.5 client constraints | Existing pet collection remains safe, source-level skills survive changeover |
| IP | Individual Progression eligibility | Determine *Hunter's* authoritative Vanilla stage; no mid-Vanilla mode changes; TBC transition undecided | Vanilla tier-0 and final Vanilla stage behave alike despite content progression |
| BOTS | Playerbot / Alt Bot support | Separate legality, AI use and persistence; no upstream edits, no automatic Alt Bot gear/talent changes | Human, persistent Alt Bot, Random Bot and non-Hunter controls all pass |
| UI | 3.3.5-facing controls | Honest Beast Training substitute; pet trainer; possible approved addon; never pretend old client API exists | Hunter can inspect/learn/train with simple steps and correct server validation |
| SAFE | Maintenance | Versioned module-owned data, backup/restore, disable parity, gated migration, native pet save reconciliation | No irreversible loss and safe rollback rehearsed on staging |

Full original rule wording: [VANILLA-FEATURE-SPEC](VANILLA-FEATURE-SPEC.md), [ABILITY-CATALOG](ABILITY-CATALOG.md), [VANILLA-FAMILY-REFERENCE](VANILLA-FAMILY-REFERENCE.md), [RARE-AND-SPECIAL-BEASTS](RARE-AND-SPECIAL-BEASTS.md).

## C. Dependency-ordered engineering programme

These are **work packages**, not committed deliverables. No coding assignment should silently override a failed gate.

| Package | Estimate | Depends on | Deliverables | Exit evidence |
|---|---:|---|---|---|
| WP-00: read-only baseline | 3–5 engineering days | Access to deployed revision and reference data | Pin AC/IP/Playerbots/Naxx Core revisions, client+DBC and DB schema; audit pet lifecycle | Exact SHA, call graph, hooks and immutable backup plan |
| WP-01: source-data provenance | research parallel to WP-00 onwards | Classic reference sources; owner's acore_world | All 17 families; each of 111 candidate rank rows; validated tame-source entries; current 3.3.5 spell mapping | Per-row source, patch, confidence, installed mapping, no invented IDs |
| WP-02: beast identity + speed spike | 4–8 days | WP-00; staging permission | Broken Tooth and a control-pet test using module only | Source identity and correct normalized speed survive full pet lifecycle |
| WP-03: auto-learn veto feasibility | before broad skills work | WP-00; disposable test pets | Trace every automatic WotLK pet spell learning path; prove non-destructive module-only interception or explain blocker | **GO/NO-GO** on faithful training under strict boundary |
| WP-04: learned skill acquisition spike | part of 8–15 days for skills | WP-01, WP-03 | Son of Hakkar use→Hunter knows Lightning Breath 6→another eligible serpent trained | Successful cast proof, rank provenance, persistence, family/TP validation |
| WP-05: loyalty + happy/food | part of 12–22 days | WP-00 and lifecycle model | Six loyalty ranks, earned gains/decay, feeding, safe persistence | 1→6 progression and unhappy tests; no irreversible runaway |
| WP-06: TP and rank training | part of 12–22 days | WP-03, WP-04, WP-05 | Exact TP ledger, training/refund, active cap, passives and native pet spell consistency | Transaction/crash/stable/reload tests; negative balance handled |
| WP-07: all pet data + combat | effort already partly embedded WP-01/04/05/06 | Safe engine bridge | All original families, attack speeds, ability effects, tame creatures, source phase gates | Full family, rank and named-beast acceptance matrices |
| WP-08: player interface | 5–10 days | WP-06 + client UI evidence | Appropriate NPC menu and perhaps separately approved addon | Can operate without typing IDs, client controls remain functional |
| WP-09: IP/Alt/Random bots + acceptance | 10–20 days | All affected WP | Mixed IP era, human/Alt/Random bot, raid, stability and rollback | Full regression evidence, owner sign-off |
| WP-10: optional integration | not estimated/approved | Successful standalone release | Naxx Core merge proposal, persistence transfer, disable standalone | Approved integration and double-hook/rollback tests |

The old rough total **42–80 engineering days** remains a placeholder for WP-00/02/04/05/06/08/09, **not** a sum of all work-package rows. Research and testing overlap with those efforts; WP-01, WP-03 and WP-07 could materially increase the estimate. Publish a revised bottom-up effort range **after** WP-00/WP-03; no delivery date should be promised beforehand.

### Critical path

```text
WP-00 source audit
    ├─ WP-01 evidence and mapping ──────────────────────────┐
    ├─ WP-02 tame/speed proof                             │
    └─ WP-03 WotLK auto-learn GO/NO-GO ─────────────────────┤
                                                          ↓
          WP-04 learned ranks from real pets → WP-05/06 → WP-07
                                                       ↓
                                             WP-08 usable UI
                                                       ↓
                                      WP-09 IP/bots/full acceptance
                                                       ↓
                           STANDALONE RELEASE (owner approves)
                                                       ↓
                            WP-10 optional Naxx Core integration
```

**Blocker policy:** If WP-03 fails, *do not* proceed with a deceptive training system that strips auto-granted spells later without safety guarantees. Document the exact missing interception and present an explicitly smaller scope for owner decision.

## D. Detailed blueprint artifact contract

A work package can be called *designed* only if it has all of:
- **Mechanic:** original expected rule, patch identity, exception, source URLs, independent cross-check where possible.
- **Affected entities:** human owner, persistent pet, source creature template, learned catalogue, skill rank, IP stage, bot role and client.
- **Read events:** taming, pet cast, XP, loyalty tick, stable, pet trainer, login/load/summon and world reload.
- **Write events:** what changes in native vs module-owned persistence; atomicity and duplicate-event policy.
- **Engine boundary:** exact extension hook, introduced order, compiled version, no upstream patch.
- **User flow:** client-facing controls, error messages, accessibility and source of truth.
- **Acceptance:** reproduction steps, expected outputs, negative/boundary tests, logs and passing evidence.
- **Safety:** backup targets, reversible migration, disable/uninstall behaviour and conflict handling.
- **Status:** evidence state and owner-approved scope. No presumption that a planned feature works.

## E. System test scenarios every milestone must cover

**Baseline controls:** Hunter normal 3.3.5 pet with project disabled; non-Hunter pet; current pet with high-value learned abilities; full modern stable. **Creature pilot:** Broken Tooth vs 2.0 speed cat plus another fast family. **Skill pilot:** Son of Hakkar Rank 6, wrong family, underlevel, no TP, Hunter not previously knowing rank, interrupted cast. **Persistence:** stable A/B/A, level-up, death/revive, logout, restart and simulated failure. **Progression:** IP initial Vanilla, final Vanilla, equal-level post-Vanilla control. **Bots:** human, Alt Bot, Random Bot, Warlock negative control. **Raid:** Molten Core and Zul'Gurub pet buff/spell compatibility, existing `.bot consumables mc`.

Full ID-numbered tests: [QA-ACCEPTANCE-MATRIX](QA-ACCEPTANCE-MATRIX.md). No report of 'fully tested' may omit the failing/skipped list.

## F. Decision ownership and version control

The module owner approves: historical baseline, old client UI compromise, existing pet migrations, legacy stable-slot policy, pet abandonment, bot provisioning, cross-era transition and any eventual release/merge. The editor may investigate and propose choices, **not silently select them**. See [DECISIONS](DECISIONS.md).

Record each change to blueprint, roadmap, historical source and runtime data using a reviewed GitHub commit/PR with:
- `design_version`, `research_revision`, `reference_checked`, `core_revision_checked`, `source_confidence`, `decision_id`, `qa_id`.
- Distinguish proposal from approved spec; never rewrite history to make missing work seem completed.
- Exact source SHAs and date of reference access if possible; future dead links need archived reference metadata.
- If a source contradicts the blueprint, flag it as a conflict, reopen requirement and request review.

## G. Practical navigation

- **Start here:** [Developer instructions](DEVELOPER-START-HERE.md), [Official handover](PROJECT-HANDOVER.md).
- **Historical catalogue:** [Family matrix](VANILLA-FAMILY-REFERENCE.md), [Ability catalogue](ABILITY-CATALOG.md), [Rare beasts](RARE-AND-SPECIAL-BEASTS.md), [bibliography](REFERENCE-BIBLIOGRAPHY.md).
- **Engine truth:** [Source audit](ENGINE-SOURCE-AUDIT.md), [feasibility](TECHNICAL-FEASIBILITY.md), [risks](RISK-AND-FEASIBILITY-GATES.md).
- **Implementation design:** [State machines](STATE-MACHINES.md), [persistence](PERSISTENCE-AND-API.md), [client UI](PLAYER-EXPERIENCE-AND-UI.md), [IP/bots](IP-PLAYERBOTS-INTEROPERABILITY.md).
- **Development control:** [Work package cards](WORK-PACKAGES.md), [Roadmap](IMPLEMENTATION-ROADMAP.md), [QA](QA-ACCEPTANCE-MATRIX.md), [Testing and rollback](TEST-AND-ROLLBACK.md), [decision log](DECISIONS.md).
- **Dataset:** `data/reference/*.csv`, including [traceability](../data/reference/requirement-traceability.csv). All research-only until validated.
