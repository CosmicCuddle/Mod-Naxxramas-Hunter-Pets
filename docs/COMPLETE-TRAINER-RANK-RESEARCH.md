# Complete pet-trainer rank reference — 52 research candidates, 47 likely Vanilla-trainable

**Research edition:** 16 October 2026. **STATUS: REFERENCE DATA ONLY.** This report does **not** verify original 1.12 client rows, actual Pet Trainer offerings, AzerothCore deployed Spell.dbc, the user's acore_world, or a working Hunter training interface.

## Summary of current evidence

The [52-row complete trainer-spell candidate table](../data/reference/vanilla-trainer-spell-id-candidates.csv) now links every Petopia Classic rank/level/TP cost in **all eight trainer ability lines** to a candidate pet spell ID in the CMaNGOS [archived spell-chain table](https://github.com/cmangos/mangos-classic/blob/master/sql/archive/0.7/4096_pet.sql).

| Trainer ability | Rank count in Petopia table | Historical Vanilla candidate ranks | Highest candidate spell ID in that core range | Notes |
|---|---:|---:|---:|---|
| Growl | 7 | 7 | 14921 | Active, **zero Training Points** at every rank, still consumes **Focus** when cast |
| Great Stamina | 10 | 10 | 5042 | Passive flat Stamina **+3 → +40** |
| Natural Armor | 10 | 10 | 24630 | Passive flat Armor **+50 → +1000** |
| Arcane Resistance | 5 | **4 likely** | 24501 for Rank 4 | Rank 5 **27052** uncertain/out-of-Vanilla source availability |
| Fire Resistance | 5 | **4 likely** | 24445 for Rank 4 | Rank 5 **27053** uncertain/out-of-Vanilla source availability |
| Frost Resistance | 5 | **4 likely** | 24449 for Rank 4 | Rank 5 **27054** uncertain/out-of-Vanilla source availability |
| Nature Resistance | 5 | **4 likely** | 24504 for Rank 4 | Rank 5 **27055** uncertain/out-of-Vanilla source availability |
| Shadow Resistance | 5 | **4 likely** | 24507 for Rank 4 | Rank 5 **27056** uncertain/out-of-Vanilla source availability |
| **Total** | **52** | **47 likely** | | **Five Rank 5 resistance rows must not be enabled without proof** |

**Critical discovery — five-rank reconciliation:** The [Petopia Classic abilities](https://www.wow-petopia.com/classic/abilities.php) page displays five school resistance ranks, including +140 Resistance at level 60. However:
- [Warcraft Wiki: Arcane Resistance](https://warcraft.wiki.gg/wiki/Arcane_Resistance_(pet_ability)) and [Frost Resistance](https://warcraft.wiki.gg/wiki/Frost_Resistance_(pet_ability)) explicitly distinguish **four ranks in Vanilla** from **five ranks in Burning Crusade**.
- [Petopia maintainer Wain's original 2019 research discussion](https://forums.wow-petopia.com/viewtopic.php?t=26093) says rank 5s were included in early gathered data but could not be found on Classic pet trainers and **might have been introduced in BC**. This is unusually valuable direct evidence of how a later reconstruction acquired the conflicting rows.
- [Blizzard Classic forum players in 2019](https://us.forums.blizzard.com/en/wow/t/pet-abilities-classic/394549) independently report **Rank 5 resistance not offered** despite the guide listing it. Availability on original 1.12 retail still needs an original client/trainer cross-check.
- An independent [PetTips GitHub README](https://github.com/entrail/PetTips) snapshot reported **21 abilities / 106 ranks**, which differs from the **111 Petopia rows by exactly five**. **This is consistent with, but not proof of**, removing five unavailable resistance Rank 5 records; current marketing/release snapshots may differ. PetTips is proprietary and its data is **not** copied or used to build this table.
- The five Rank 5 IDs all appear in a later-compatible **2705x** archived CMaNGOS spell chain; the chain also includes other later ranks (e.g. Growl Rank 8). **An archived emulator rank chain is not an original 1.12 trainer list**.

**Disposition:** Keep all 111 reference ranks in the research index for historical tracking. Mark **five resistance Rank 5** entries as `RANK_5_TRAINER_AVAILABILITY_DISPUTED` / `BLOCK_BY_DEFAULT` for a strict Vanilla baseline. Thus 106 candidate **in-era rank rows** (59 wild + 47 trainer) remain, not 106 proven trainable ranks. Two wild ranks (Lightning Breath 1 and Charge 4) separately have no known tame sources, so total actual obtainable rank count may be **less than 106**.

## Spell chains and training-point/effect semantics

### Growl

| Rank | Minimum pet level | TP | CMaNGOS candidate spell ID |
|---|---:|---:|---:|
| 1 | 1 | 0 | 2649 |
| 2 | 10 | 0 | 14916 |
| 3 | 20 | 0 | 14917 |
| 4 | 30 | 0 | 14918 |
| 5 | 40 | 0 | 14919 |
| 6 | 50 | 0 | 14920 |
| 7 | 60 | 0 | 14921 |

The level-1 pet rank is not proof that a Hunter receives Beast Training before their level-10 quests. Growl **0 TP cost** is distinct from spell-casting **15 Focus**. A 3.3.5 threat spell with this name must be audited for owner AP scaling and native levelup auto-grant.

### Great Stamina

| Rank | Minimum pet level | TP | Stamina bonus | Candidate spell ID |
|---|---:|---:|---:|---:|
| 1 | 10 | 5 | +3 | 4187 |
| 2 | 12 | 10 | +5 | 4188 |
| 3 | 18 | 15 | +7 | 4189 |
| 4 | 24 | 25 | +10 | 4190 |
| 5 | 30 | 50 | +13 | 4191 |
| 6 | 36 | 75 | +17 | 4192 |
| 7 | 42 | 100 | +21 | 4193 |
| 8 | 48 | 125 | +26 | 4194 |
| 9 | 54 | 150 | +32 | 5041 |
| 10 | 60 | 185 | +40 | 5042 |

**Source:** [Petopia Classic](https://www.wow-petopia.com/classic/abilities.php), [Classic Wowhead Rank 1](https://www.wowhead.com/classic/spell=4187) and [CMaNGOS archived spell chain](https://github.com/cmangos/mangos-classic/blob/master/sql/archive/0.7/4096_pet.sql).

**Reliability issue:** A [2024 CMaNGOS 1.12 bug report](https://github.com/cmangos/issues/issues/3817) described Great Stamina 1–5 raising the pet's displayed Stamina without increasing its total Health on that emulator revision. This is **an unverified bug report**, not proof of original retail behaviour or of your server's condition. QA must inspect both **stat and actual maximum HP**.

### Natural Armor

| Rank | Minimum pet level | TP | Flat Armor bonus | Candidate spell ID |
|---|---:|---:|---:|---:|
| 1 | 10 | 1 | +50 | 24545 |
| 2 | 12 | 5 | +100 | 24549 |
| 3 | 18 | 10 | +160 | 24550 |
| 4 | 24 | 15 | +240 | 24551 |
| 5 | 30 | 25 | +330 | 24552 |
| 6 | 36 | 50 | +430 | 24553 |
| 7 | 42 | 75 | +550 | 24554 |
| 8 | 48 | 100 | +675 | 24555 |
| 9 | 54 | 125 | +810 | 24629 |
| 10 | 60 | 150 | +1000 | 24630 |

Family Armor modifiers, client inspection, spell aura type and Native AzerothCore guardian stat initialization determine whether the bonus can be applied safely. **Do not multiply the modifier twice** or silently add a WotLK talent passive.

### The five resistance schools — common requirements, distinct spell chains

| Rank | Minimum pet level | TP | Flat school resistance |
|---|---:|---:|---:|
| 1 | 20 | 5 | +30 |
| 2 | 30 | 15 | +60 |
| 3 | 40 | 45 | +90 |
| 4 | 50 | 90 | +120 |
| 5 — **disputed Vanilla availability** | 60 | 105 | +140 |

| School | Rank 1 | Rank 2 | Rank 3 | Rank 4 | Rank 5 **disputed** |
|---|---:|---:|---:|---:|---:|
| Arcane | 24493 | 24497 | 24500 | 24501 | **27052** |
| Fire | 23992 | 24439 | 24444 | 24445 | **27053** |
| Frost | 24446 | 24447 | 24448 | 24449 | **27054** |
| Nature | 24492 | 24502 | 24503 | 24504 | **27055** |
| Shadow | 24488 | 24505 | 24506 | 24507 | **27056** |

**Sources:** [CMaNGOS 0.7 archived chain](https://github.com/cmangos/mangos-classic/blob/master/sql/archive/0.7/4096_pet.sql), [Wowhead Classic Arcane Resistance Rank 1](https://www.wowhead.com/classic/spell=24493), [Frost Rank 2](https://www.wowhead.com/classic/spell=24447), [Petopia Classic](https://www.wow-petopia.com/classic/abilities.php). These are **pet effect spell candidates**, **not** confirmed trainer wrapper spell IDs.

## Distinguish three independent spell identities

1. **Trainer's learn-spell wrapper**: NPC purchase teaches Hunter the ability/rank to train on pets. Example [Wowhead Nature Resistance trainer wrapper](https://www.wowhead.com/classic/spell=24512) shows **Learn Spell** effect rather than the pet's passive resistance aura. Pet-trainer money cost, minimum Hunter level and access must be verified separately from pet TP.
2. **Hunter's Beast Training catalogue**: a rank permanently known by the Hunter; must persist even if the current pet is stabled, renamed, respec'd or abandoned.
3. **Pet's learned/cast spell**: the native pet ability with its own *spell ID*, aura/focus/cooldown, trained rank and TP debit. This is the chain catalogued above; some passive entries have different cast/learn wrappers.

There is **no evidence** that inserting the **pet spell ID** into an NPC trainer row on AzerothCore would correctly recreate the original training workflow. Doing so would risk teaching the wrong actor/packet.

## Preliminary comparison against PUBLIC upstream AzerothCore — limited scope

We searched the publicly connected [AzerothCore `data/sql/base/db_world/spell_ranks.sql`](https://github.com/azerothcore/azerothcore-wotlk/blob/master/data/sql/base/db_world/spell_ranks.sql) for exact spell IDs from the 52 rows. **Eleven** occur as rank-chain rows: all seven Growl candidates, and Nature Resistance 2–5. The other **41** do not appear as exact IDs in that **single SQL table**.

**This does NOT prove that 41 spells are missing from Spell.dbc**. AzerothCore's `spell_ranks` table is a selective rank-chain reference and Spell.dbc is loaded separately. We did **not** inspect any actual 3.3.5 Spell.dbc, client MPQ, installed world DB or currently running server. Therefore **zero original-to-deployed mappings are verified**.

A selective test is more valuable than assuming a single public SQL file tells the whole story. The deployed AzerothCore branch's exact `SpellMgr::LoadSpellRanks` and loaded DBC must be checked, and a spell may exist while its associated trainer/pet skill line is no longer used.

## Gating and what remains unknown

- **Trainer availability:** confirm rank from genuine 1.12 pet trainer NPC/vendor lists and versioned client Spell/SkillLineAbility.
- **Pet aura:** compare old flat ranks with current spells, ensure stat recalculation occurs after training/unlearning.
- **Charge and growl ranks:** original rank existence does not imply availability from a source.
- **Trainer prices:** record historical copper, eligibility and Hunter vs pet level separately; do not invent copper prices.
- **Spell mapping:** verify exact server+client 3.3.5 spell rows and native auto-learn, pet-talent interaction, pet save/reload.
- **Original data:** the source CMaNGOS archived chain is **not a frozen original 1.12 dataset**. See [DBC verification protocol](ORIGINAL-DBC-AND-DEPLOYED-COMPARISON.md).

**Release baseline recommendation (not owner-approved):** 47 trainer ranks (7 Growl + 10 Great Stamina + 10 Natural Armor + 4 each school); **five resistance Rank 5s held out unless genuinely proven Vanilla-available**. This is a stronger historical starting point than automatically enabling all 52.
