# Vanilla ability and training-rank catalogue

**Status:** Human-readable blueprint reference. Source: [Classic Petopia abilities](https://www.wow-petopia.com/classic/abilities.php) and [Classic training](https://www.wow-petopia.com/classic/training.php). A **provisional** 111-row rank/level/cost dataset now exists at [vanilla-ability-ranks.csv](../data/reference/vanilla-ability-ranks.csv), with all costs populated from this Classic guide. An independently verified **original 1.12 dataset**, exact deployed spell IDs, source NPC entries and per-family mappings are still mandatory.

## Scope: exactly 21 catalogue ability names

**13 taming-acquired active skills:** Bite, Charge, Claw, Cower, Dash, Dive, Furious Howl, Lightning Breath, Prowl, Scorpid Poison, Screech, Shell Shield and Thunderstomp.

**Eight pet-trainer skills:** Arcane Resistance, Fire Resistance, Frost Resistance, Great Stamina, Growl, Natural Armor, Nature Resistance and Shadow Resistance. Growl is an active ability; the other seven are passives.

**Not Vanilla:** Avoidance, Cobra Reflexes, Fire Breath, Gore, Poison Spit, Warp. These appeared with TBC-era pet systems; do not backport inadvertently. Source: [Petopia BC](https://www.wow-petopia.com/classic_bc/abilities.php).

## Rank → required pet level (source indexed)

The table lists all known Vanilla ranks by **minimum PET level**. It is **not** a table of creature level or Hunter level, and not proof of installed spell IDs.

| Ability | Rank: minimum pet level |
|---|---|
| Bite | 1:1, 2:8, 3:16, 4:24, 5:32, 6:40, 7:48, 8:56 |
| Charge | 1:1, 2:12, 3:24, 4:36, 5:48, 6:60 |
| Claw | 1:1, 2:8, 3:16, 4:24, 5:32, 6:40, 7:48, 8:56 |
| Cower | 1:5, 2:15, 3:25, 4:35, 5:45, 6:55 |
| Dash | 1:30, 2:40, 3:50 |
| Dive | 1:30, 2:40, 3:50 |
| Furious Howl | 1:10, 2:24, 3:40, 4:56 |
| Lightning Breath | 1:1, 2:12, 3:24, 4:36, 5:48, 6:60 |
| Prowl | 1:30, 2:40, 3:50 |
| Scorpid Poison | 1:8, 2:24, 3:40, 4:56 |
| Screech | 1:8, 2:24, 3:48, 4:56 |
| Shell Shield | 1:20 |
| Thunderstomp | 1:30, 2:40, 3:50 |
| Growl | 1:1, 2:10, 3:20, 4:30, 5:40, 6:50, 7:60 |
| Great Stamina | 1:10, 2:12, 3:18, 4:24, 5:30, 6:36, 7:42, 8:48, 9:54, 10:60 |
| Natural Armor | 1:10, 2:12, 3:18, 4:24, 5:30, 6:36, 7:42, 8:48, 9:54, 10:60 |
| Arcane Resistance | 1:20, 2:30, 3:40, 4:50, 5:60 |
| Fire Resistance | 1:20, 2:30, 3:40, 4:50, 5:60 |
| Frost Resistance | 1:20, 2:30, 3:40, 4:50, 5:60 |
| Nature Resistance | 1:20, 2:30, 3:40, 4:50, 5:60 |
| Shadow Resistance | 1:20, 2:30, 3:40, 4:50, 5:60 |

**Validation note:** Compare these level/rank entries to 1.12 client spells and each project's exact loaded `Spell.dbc` before converting them to code or SQL; some of the original rank IDs may be absent or repurposed in 3.3.5a.

## Training-point cost semantics

- Use TP cost **of the current trained rank**, not cumulative rank 1+2+3 costs.
- Upgrade rank N→N+1: release old rank cost and charge new rank cost in one atomic operation.
- An initial tame may inherit an ability and start with zero or **negative unspent** TP until loyalty/level catches up; validate historically.
- Training an active ability already at 4 slots as a rank upgrade counts as the **same slot**.
- Growl has **zero TP at every rank**, even if the balance is negative.
- Pet untraining refunds pet learned-rank cost but may charge a historically increasing **money** fee at pet trainers. Never rewrite the Hunter-known catalogue on pet respec.
- Passives have different, often substantial costs; do not count passives as active.
- A canonical table must include `tp_cost`, `level_min`, `ability_category`, `spell_family`, `known_source_kind` and per-rank spell ID provenance.

### Research note: Charge Rank 4

[Classic Petopia](https://www.wow-petopia.com/classic/abilities.php) lists Charge 4 at pet level 36 for **17 TP**, but explicitly reports **no known training source**. The rank can exist in a ruleset without having a wild source that makes it learnable. This distinction must be preserved in the final creature→rank catalogue. **Do not invent a boar that teaches it.** See [evidence standards](RESEARCH-EVIDENCE-STANDARDS.md).

### Selected reference costs, good test fixtures

| Skill | Rank | Pet level | TP cost | Source |
|---|---:|---:|---:|---|
| Bite | 8 | 56 | 25 | Bloodaxe Worg |
| Claw | 8 | 56 | 25 | Winterspring Screecher / Elder Shardtooth |
| Furious Howl | 4 | 56 | 25 | Bloodaxe Worg |
| Lightning Breath | 6 | 60 | 25 | Son of Hakkar |
| Growl | 7 | 60 | 0 | Pet trainer |
| Great Stamina | 10 | 60 | 185 | Pet trainer |
| Natural Armor | 10 | 60 | 150 | Pet trainer |
| Fire Resistance | 5 | 60 | 105 | Pet trainer |
| Dash | 3 | 50 | 25 | Multiple wild sources |
| Dive | 3 | 50 | 25 | Multiple wild sources |
| Screech | 4 | 56 | 25 | Monstrous Plaguebat / Winterspring Screecher |
| Shell Shield | 1 | 20 | 15 | Kresh and other turtles |
| Thunderstomp | 1 | 30 | 15 | Mistvale Gorilla / Jungle Thunderer |

**Critical:** A level-60 loyalty-6 pet has only 300 TP. Great Stamina 10 (185) + Natural Armor 10 (150) = **335 TP**, so it cannot simultaneously train both top ranks with no special rules. This is a useful *negative acceptance test*, not a recommendation to change cost.

## Ability-specific concerns for code

- **Lightning Breath**: spell school Nature and rank 6 source; inspect line-of-sight, target/attack range, focus, immunities and spell rank chain.
- **Furious Howl**: next-physical-attack party buff in Vanilla (not a modern minute-long AP buff); target selection, party range, stacking/refresh and multiple wolves must be inspected. [Original reference](https://www.wow-petopia.com/classic/abilities.php).
- **Scorpid Poison**: stacking DoT up to five stacks; owner spell bonuses/resist checks and refresh rules; avoid WotLK redesign.
- **Screech**: deals damage and weakens melee attack power in melee range; debuff refresh and raid stacking tests.
- **Shell Shield**: turtle defensive cooldown, attack cadence penalty; validate exact effect and duration.
- **Prowl**: pet stealth, movement reduction and first-hit damage; distinguish pet stealth from player invisibility.
- **Charge**: boar acceleration, temporary stun/immobilization and next-attack bonus; ensure not translated to WotLK knockback or unrelated spell.
- **Growl/Cower**: contrasting threat functions, correct autocast. Threat should follow Vanilla implementation, not today's unrelated spell rank text.
- **Dash/Dive**: land/flying-family movement speed abilities, not flight permission.
- **Passives**: stat/resistance ranks must not stack with obsolete lower ranks or modern baseline bonuses.

## Authoritative input data to build next

`data/reference/vanilla-ability-ranks.csv`: existing candidate rank catalogue; extend and verify to canonical `ability_key,rank,min_pet_level,tp_cost,is_active,trainer_or_wild,allowed_family_codes,classic_spell_id,server_spell_id,source_url,patch_id,evidence_confidence,status` per rank.

`data/reference/vanilla-teaching-beasts.csv`: `creature_entry,creature_name,family_code,map_or_zone,level_min,level_max,ability_key,rank,availability_patch,source_url,provenance,confirmed_on_server`.

**Do not fill any `creature_entry` or deployed `server_spell_id` from a guess or name-only lookup.** Cross-check exact creature entry against the owner's acore_world and skill rank against current server and client DBC.

## Major test: Son of Hakkar

1. Character has Vanilla IP and legitimate access to Zul'Gurub.
2. Pet is legitimately tamed from Son of Hakkar (or disposable staging equivalent explicitly marked synthetic).
3. Wild-known `Lightning Breath 6` is present on this pet, not on all wind serpents.
4. Hunter learns it only after observed valid pet skill use.
5. Stable the Son of Hakkar; choose second eligible Wind Serpent at pet level ≥ 60.
6. Teach rank 6, spending 25 TP net (after any existing LB rank refund), maintaining four-active limit.
7. Log out / swap / restart: still known by Hunter and trained on second pet; no WotLK auto-replacement.
8. Attempt to teach a cat and a level-59 wind serpent: reject without modifying database or consuming currency.

**Caveat:** 3.3.5 WotLK Lightning Breath damage differs significantly from 1.12; never reuse WotLK rank ID without inspecting actual spell data.
