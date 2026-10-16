# Tame and ability-source atlas — research edition 01

**Prepared:** 16 October 2026. **Status:** sourced candidates only, **not installed, tested or validated** against the user's actual `acore_world` or client. The goal is eventually to document **all** Vanilla creature→learnable-rank relationships, rather than only a handful of famous rare pets.

## Machine-readable atlas and scope

[Vanilla tame/ability sources (CSV)](../data/reference/vanilla-tame-ability-sources.csv)

This first curated, independently assembled *subset* contains:
- **141** individual **beast→ability-rank relationships**, based on the [Classic Petopia abilities catalogue](https://www.wow-petopia.com/classic/abilities.php).
- **13 / 13** distinct wild-learned ability names represented.
- **57 / 59** wild-learned rank combinations with at least one indexed **candidate** tame source and **two** documented as having **no known wild training source**. All listed wild rank combinations have now been researched at least to *one example or explicit unavailable reference*.
- Six NPC names with independent **Classic-era ID candidates** confirmed against separately linked Wowhead/ClassicDB pages. The same NPC may appear across multiple ability records; **10 rows** currently have this candidate ID.
- **No row** has been verified against the deployed world DB, Individual Progression rules, the installed AzerothCore build or client Spell.dbc. `ip_era_access_checked = no` and `native_335_entry_checked = no` are intentional.

The companion [111-rank coverage report](../data/reference/vanilla-rank-source-coverage.csv) explicitly separates **not yet researched** from **historically no known source**. An *empty* local source list must not automatically make a rank unavailable. All 115 rows are **source examples**, not an exhaustive catalogue.

## Verified Classic source examples — not yet confirmed in deployed server

| Ability | Rank | Candidate beast | Family | Historical zone | Source significance |
|---|---:|---|---|---|---|
| Bite | 8 | **Bloodaxe Worg** | Wolf | Blackrock Spire | Highest Bite rank, alongside Dash 3 and Furious Howl 4 on the same source |
| Claw | 8 | **Winterspring Screecher** or Elder Shardtooth | Owl / Bear | Winterspring | A higher-rank Claw source without needing a cat tame |
| Charge | 6 | **Plagued Swine** | Boar | Eastern Plaguelands | Original high-level boar Charge; added in later Vanilla patches |
| Cower | 6 | Frostsaber Cub or **Monstrous Plaguebat** | Cat / Bat | Winterspring / Eastern Plaguelands | Highest Cower rank; may be obtained from non-cat family |
| Dash | 1 | **Broken Tooth** | Cat | Badlands | Dash 1 naturally, plus unusual original 1.0-sec base attack cadence |
| Dash | 3 | **Bloodaxe Worg** or Rak'Shiri | Wolf / Cat | Blackrock Spire / Winterspring | Shows a pet can teach more than one skill |
| Dive | 3 | **Spawn of Hakkar** or Dark Screecher | Wind Serpent / Bat | Sunken Temple / Blackrock Depths | Travel-speed skill on several families |
| Furious Howl | 4 | **Bloodaxe Worg** | Wolf | Blackrock Spire | Original 10-second next-physical-attack party bonus, not modern AP buff |
| Lightning Breath | 2 | Deviate Coiler | Wind Serpent | Wailing Caverns | Early, dungeon-taught breath |
| Lightning Breath | 3 | Washte Pawne | Wind Serpent | The Barrens | Regional unique tame skill source |
| Lightning Breath | 4 | Vale Screecher | Wind Serpent | Feralas | Higher-rank source |
| Lightning Breath | 5 | Hakkari Sapper / Spawn of Hakkar | Wind Serpent | Sunken Temple | Distinguish from Son of Hakkar in a different raid |
| Lightning Breath | 6 | **Son of Hakkar** | Wind Serpent | Zul'Gurub | Highest Nature-damage breath rank, raid-taming progression milestone |
| Prowl | 3 | Frostsaber Stalker | Cat | Winterspring | Original cat stealth movement/first attack bonus |
| Scorpid Poison | 4 | **Krellack** or Firetail Scorpid | Scorpid | Silithus / Burning Steppes | Highest original stacking Nature poison |
| Screech | 4 | **Winterspring Screecher** or Monstrous Plaguebat | Owl / Bat | Winterspring / Eastern Plaguelands | Original melee attack-power reduction |
| Shell Shield | 1 | **Kresh** or Ghamoo-ra | Turtle | Wailing Caverns / Blackfathom Deeps | Original turtle defensive cooldown |
| Thunderstomp | 3 | **U'cha** or Un'Goro Thunderer | Gorilla | Un'Goro Crater | Highest original gorilla AoE thunder skill |

**Source:** [Classic Petopia](https://www.wow-petopia.com/classic/abilities.php), especially each rank's enumerated tame sources. Cross-check examples against [Wowhead Classic Hunter pets guide](https://www.wowhead.com/classic/guide/classic-wow-hunter-pets) and exact source creature records before runtime use.

## Independently referenced Classic NPC ID candidates

| Beast | Candidate creature entry | External corroboration | Not yet validated |
|---|---:|---|---|
| Broken Tooth | 2850 | [Wowhead Classic NPC](https://www.wowhead.com/classic/npc=2850) | Actual deployed world entry / tame / source speed |
| Bloodaxe Worg | 9696 | [ClassicDB](https://classicdb.ch/?npc=9696), [Wowhead Classic](https://www.wowhead.com/classic/npc=9696) | Actual summoned pet original entry / multiple innate skills |
| Winterspring Screecher | 7456 | [ClassicDB](https://www.classicdb.ch/?npc=7456), [Wowhead Classic](https://www.wowhead.com/classic/npc=7456) | Original skills and client pet appearance after stable/relog |
| Son of Hakkar | 11357 | [Wowhead Classic](https://www.wowhead.com/classic/npc=11357) | IP raid availability / Lightning Breath 6 / taming checks |
| Krellack | 14476 | [Wowhead Classic](https://www.wowhead.com/classic/npc=14476) | Actual world template / poison rank |
| Plagued Swine | 16117 | [Wowhead Classic/Forever](https://www.wowhead.com/forever/npc=16117) | Actual world and spell-rank mapping / late Vanilla eligibility |

These **candidate IDs are a starting point for read-only SQL**, not permission to change `creature_template` or spawn tables. All other unverified NPC entries remain blank in the CSV.

## Two importantly different "missing source" cases

- **Lightning Breath Rank 1:** Petopia has a rank entry (pet level 1, 1 TP), but explicitly lists **no known training source**. It must **not** be auto-unlocked because the ability exists in a data file.
- **Charge Rank 4:** Petopia lists rank 4 (pet level 36, 17 TP), **no known training source**. A boar family containing this rank in its client data does not establish any legal source beast.

For other ranks, **the sample coverage now has at least one source**, but is *not exhaustive*: more alternative beasts, verified creature IDs, exact client spell IDs and original-patch provenance are still needed. A row with no direct NPC ID is still a source candidate, not a complete runtime mapping. See [coverage table](../data/reference/vanilla-rank-source-coverage.csv).

## Another historical variant: weaker original "caster" tames

Petopia's [caster pets reference](https://www.wow-petopia.com/classic/casterpets.php) describes inferior stat distributions for certain original 2004–2006 tames (e.g. Azzere the Skyblade, Elder Cloud Serpent, Hakkari Sapper/Frostwing, Fire Roc), **not reproduced in Classic Era**. This could impact a goal of strict *original* Vanilla fidelity versus recreating an inadvertent disadvantage. Mark as **historical variant / owner decision**, not an automatic stat nerf.

## Required next-step validation against deployed assets

1. Pin `acore_world` snapshot and document AzerothCore/IP/Playerbots exact revision and 3.3.5 client patch/DBC hashes. Work on read-only copies or `SELECT` queries; no mass UPDATE.
2. For every candidate NPC, identify true `creature_template.entry`, `family`, tame eligibility, template abilities, spawn map/zone, original attack speed and installed IP availability gates. Petopia and Classic NPC entry aren't proof of AzerothCore's current template identity.
3. For every ability rank, map **canonical ability name+rank** to original/Classic spell ID, **deployed** 3.3.5 server spell ID and matching client Spell.dbc entry. Check effects/damage/focus/cooldown/target/known chain and pet allowed family; no name-only matching.
4. Expand indexed sources from sample→complete using original patch data, Petopia's detailed rank lists and independent historical sources. Never assume all listed beasts have their correct ranks in a WotLK server.
5. Confirm skill learning on actual owner Hunter under their own IP era, exact pet ownership, native pet ability persistence and Playerbot cast usage only after source/architecture gates pass.
6. Flag any original creature no longer tameable in 3.3.5 as a **module-only feasibility question**, not a silent `creature_template` edit.

**Implementation note:** A no-data-loss restoration should preserve a pet's original tame identity; a generic pet family/skin alone cannot determine where the Hunter learned the rank.

## Separate rights/provenance consideration

The independent Classic addon [PetTips](https://github.com/entrail/PetTips) reports **21 ability names / 106 Classic-era ranks / 297 teaching beasts**, whereas this project's provisional Petopia-based table lists **111 rank entries**. **Do not claim either count is definitive** until independently reconciled. PetTips's [LICENSE.md](https://github.com/entrail/PetTips/blob/main/LICENSE.md) restricts redistribution and use of its source/data in other projects without author permission. It may be **read as a reference for comparison**; **do not copy its files or data** to this project. Research independently from accessible historical/reference sources.

**Next deliverable:** Expand **every rank** to its full original tame-source list; verify exact creature and spell IDs, investigate family eligibility anomalies and independently reconcile the **111-vs-106** count discrepancy. The 141-row sample is an initial evidence set, not a complete tame atlas.
