# Spell-ID and original pet-ability effect research — first audited cross-check

**Research edition:** 16 October 2026. **Status:** Candidate IDs and historical effect specifications, **not an approved or runtime-verified Spell.dbc mapping**. No client/DBC/server files were changed.

## Why spell identity comes before implementation

Original Vanilla Hunter pet abilities sometimes reuse familiar names while their **focus cost, cooldown, damage school, rank, target area, pet family or proc rules** changed during TBC and WotLK. A 3.3.5 server can store the same-looking spell ID but execute a significantly different mechanic. A module-only attempt is **not faithful** until the relevant *server* spell entry **and** client representation are proven.

To build an initial independent cross-check, we compared:
- [Classic Petopia original-rank catalogue](https://www.wow-petopia.com/classic/abilities.php), supported by [Wowhead Classic pet guide](https://www.wowhead.com/classic/guide/classic-wow-hunter-pets), as the **gameplay requirement candidate**.
- The `MANGOSBOT_ZERO` / Classic Hunter family mappings in [CMaNGOS PlayerbotFactory.cpp](https://github.com/cmangos/playerbots/blob/master/playerbot/PlayerbotFactory.cpp), as a **technical spell-ID candidate**. This is a **Playerbot fallback training map**, *not* authoritative 2006 Blizzard DBC.
- Named spell pages such as [Claw Rank 3](https://wowclassicdb.com/spell/16829), [Screech Rank 3](https://www.wowhead.com/classic/spell=24578), [Screech Rank 4](https://www.wowhead.com/classic/spell=24579), and [Screech/TBC later rank](https://www.wowhead.com/tbc/de/spell=27051).
- Target client consistency guidance: [WoW Spell Editor](https://github.com/stoneharry/WoW-Spell-Editor) documents **matching server and client Spell.dbc**; [Vanilla DBC CLI](https://github.com/suprsokr/vanilladbc-cli) supports version-specific read-only original DBC extraction (e.g. 1.12.1.5875).

**Never import/copy CMaNGOS Playerbot code**, use it only as a factual lead for which individual spell IDs to test. IDs still require exact historical spell properties and the user's deployed 3.3.5 assets.

## Complete *wild-ability* candidate coverage

The [59-row spell-ID candidate CSV](../data/reference/vanilla-wild-spell-id-candidates.csv) covers **all 59 entries across 13 wild-learned ability names** in our 111-rank reference table.

- The CMaNGOS Classic Playerbot section provides **at least one candidate spell ID for all 59** rows.
- **Eight** rank-level discrepancies arise when comparing the bot's configured min levels with Petopia's Classic rank requirements.
- **One** rank has two different spell IDs in separate CMaNGOS bot family mappings.
- All 59 rows are marked `original_112_dbc_confirmed = no`, `deployed_335_dbc_confirmed = no`, and `production_mapping_status = not_assigned`.
- A populated candidate ID is **not** equivalent to a verified production mapping, nor a historically available tame source.

The **52 trainer-rank IDs** (five school resistances, Great Stamina, Natural Armor and Growl) remain to be sourced from authentic original DBC and trainer data; no guessing from modern WotLK rank trees.

## High-value reference spell identities (research candidates)

| Vanilla ability | Rank | Reference pet level | Candidate Classic spell ID | Candidate historical effect / test |
|---|---:|---:|---:|---|
| Bite | 8 | 56 | **17261** | 81–99 bonus physical damage; ensure classic focus/attack and cooldown mechanics |
| Claw | 8 | 56 | **3009** | 43–59 additional physical damage |
| Charge | 6 | 60 | **27685** | Boar charge, short immobilization, **+550 melee attack power** on next attack |
| Cower | 6 | 55 | **16697** | Decrease pet threat; no damage; threat math needs original rank verification |
| Dash | 3 | 50 | **23110** | 80% pet movement speed increase for 15 sec, not free flight |
| Dive | 3 | 50 | **23147** | Flying-associated family movement ability; numeric movement change still to confirm |
| Furious Howl | 4 | 56 | **24597** | Party members within 15 yards: **45–57 bonus damage** on next physical attack within 10 sec |
| Lightning Breath | 6 | 60 | **25012** | 99–113 Nature damage, one target; Son of Hakkar source |
| Prowl | 3 | 50 | **24453** | Pet stealth, 40% movement-speed reduction, +50% first hit damage |
| Scorpid Poison | 4 | 56 | **24587** | 40 Nature damage over 10 sec, **up to five stacks** |
| Screech | 4 | 56 | **24579** / **27051** in two bot family rows | **24579** has the Vanilla candidate effect: 26–46 damage and -100 melee AP, 4 sec; **27051 is out-of-era/unsafe for Vanilla** (see below) |
| Shell Shield | 1 | 20 | **26064** | Damage taken reduced 50% and time between attacks increased 43%, 12 sec |
| Thunderstomp | 3 | 50 | **26188** | 115–133 Nature damage to enemies within 8 yards; test AoE and threat |

**Meaning of figures:** concise **Petopia Classic** rank descriptions, not values measured in AzerothCore. Numeric effect and rank IDs require an actual 1.12 DBC check and a precise 3.3.5 equivalent or approved custom spell/client patch. All thirteen require validation of focus cost, cooldown, school, threat, resist checks, auras, effect magnitudes and family masks. For Screech, the 24579 ID has additional independent [Wowhead Classic support](https://www.wowhead.com/classic/spell=24579).

## Concrete compatibility bug exposed: Screech 27051

The inspected CMaNGOS Classic Playerbot family mapping includes **27051** as a fourth Screech entry for one family, while another family uses **24579**. The distinction is **not safe to erase by selecting the first integer**.

Independent [TBC spell 27051](https://www.wowhead.com/tbc/de/spell=27051) is a **level-64 later-expansion Screech** with 33–61 damage and -210 melee AP; the [WotLK spell 27051](https://www.wowhead.com/wotlk/spell=27051) is **Demoralizing Screech** with yet different damage/scaling/duration. It is **not** the level-56 Vanilla Screech Rank 4 effect. The Classic [24579](https://www.wowhead.com/classic/spell=24579) reference is Rank 4.

**Disposition:** `CANDIDATE_24579`, `REJECT_27051_AS_VANILLA`. This is evidence that a **'Classic' code branch can nevertheless reference a spell with later-expansion behaviour**, especially when used for bot preset spell learning. The 3.3.5 server still must be checked to ensure even 24579 is preserved appropriately in both DBC and server scripts.

## Eight bot-level discrepancies — don't allow automatic contamination

| Ability | Rank | Petopia Classic min level | CMaNGOS bot-configured level | Evidence-based interpretation |
|---|---:|---:|---:|---|
| Claw | 3 | 16 | 15 | [Classic spell 16829](https://wowclassicdb.com/spell/16829) independently lists 16; bot appears inconsistent |
| Claw | 4 | 24 | 22 | Retain 24 as requirement candidate until authentic client verified |
| Claw | 5 | 32 | 29 | Retain 32 as requirement candidate |
| Claw | 6 | 40 | 36 | Retain 40 as requirement candidate |
| Furious Howl | 2 | 24 | 20 | [Petopia rank catalogue](https://www.wow-petopia.com/classic/abilities.php) specifies 24 |
| Furious Howl | 3 | 40 | 30 | Petopia specifies 40 |
| Furious Howl | 4 | 56 | 40 | [Petopia Howl 4](https://www.wow-petopia.com/classic/abilities.php) specifies 56, Bloodaxe Worg |
| Screech | 3 | 48 | 40 | [Wowhead Classic Screech 3](https://www.wowhead.com/classic/spell=24578) independently lists level 48 |

**Disposition:** these are **source-code reference disagreements**, not proof that Petopia has incorrect levels. The bot may simply use wrong skill thresholds. Do not import those thresholds into the module's eligibility rules. Source/client-DBC verification and independent spell links decide.

## Research scope beyond rank identity

### Spell family masks and tamed source

A spell ID and rank can be valid but illegal for that *pet family*. Our 17-family matrix must be cross-checked against `CreatureFamily.dbc`, `SkillLineAbility.dbc` and 1.12 permitted skill line masks. Also separate a tamed wild source naturally possessing an ability from the **Hunter having learnt the rank through use**.

### Spell effects, school, client display

For all 13 active wild abilities and 8 trainer lines, record:
- `canonical_ability`, `rank`, `min_pet_level`, `training_points`, `original_spell_id` with patch/build and skill-line provenance.
- `cast_time`, `focus_cost`, `cooldown`, `duration`, `range`, `damage_school`, `effect_type_1..3`, `base_points`, `radius`, `proc_flags`, `auras`, `target_rules`, `threat` and `talent_scaling`.
- `deployed_335_server_spell_id`, `deployed_335_client_spell_id`, actual client tooltip/icons and any spell rank-chain mapping.
- `original creature entry`, `tame outcome`, `actual pet spell after stable/load`, `Hunter-known catalogue event`, `pet trained rank` and IP era rule.
- Confidence, two sources where available, reference revision/date, actual in-game test IDs.

A spell with correct ID but wrong 3.3.5 effect is **not complete**. A desired custom `Spell.dbc` change needs **separate owner approval** and safe client distribution and would remain a **module-only/asset feasibility issue**, never a silent upstream edit.

## Concrete test cases for Stage 0 (research before any experiments)

1. For each CMaNGOS candidate ID, validate original `Spell.dbc`, `SkillLineAbility.dbc` and rank chain at **actual original 1.12.1.5875 build** or another explicitly documented patch source.
2. Compare 1.12 result against **exact installed** AzerothCore 3.3.5a Spell.dbc and 3.3.5 client matching patch. If spell ID points to different rank/ability/behaviour, block mapping instead of sending wrong packets.
3. Compute and record **all discrepant ranks** with source A and B per-row. Confirm CMaNGOS bot is not an authoritative pet-training list.
4. Verify legal family and natural tame-source acquisition separately from native spell existence.
5. Pilot only later with disposable pets under an owner-approved staging build: Bite/Claw (basic damage), Broken Tooth (attack speed), Screech 4 (AOE AP), Furious Howl 4 (raid group), Lightning Breath 6 (ability transfer), Shell Shield (survivability), Scorpid Poison 4 (stacks).
6. Before introducing any new client spell/DBC, determine whether the original rank can be implemented via existing client-visible spell or whether a controlled matching server+client asset patch is unavoidable. A missing client feature remains **not proven module-only**.

## Exact output required from future implementer

```text
ABILITY / RANK:
ORIGINAL PATCH / BUILD:
ORIGINAL CLIENT DBC SPELL ID + ROW HASH:
CMANGOS BOT CANDIDATE ID:
DEFINITIVE SOURCE / CONFLICTS:
DEPLOYED 3.3.5 SERVER SPELL ID / EFFECTS:
DEPLOYED 3.3.5 CLIENT SPELL ID / TOOLTIP:
FAMILY MASK / FOCUS / COOLDOWN / THREAT:
TAME SOURCE / HUNTER KNOWN / PET TRAINED:
CURRENT IP RULE / PLAYERBOT IMPACT:
MODULE-ONLY FEASIBILITY:
TEST IDS + EVIDENCE:
OWNER DECISION REQUIRED:
```

**No feature has been implemented, no original or deployed DBC was accessed, and no in-game testing has been performed during this research update.**
