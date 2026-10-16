# WotLK pet auto-grant versus original Vanilla — three-path source comparison

**Research date:** 16 October 2026  
**Status:** Source-derived rules and crosswalk prepared; **actual 3.3.5 spell-ID enumeration NOT YET COMPLETE** because this research session does not have the user's deployed `dbc/` files, local WorldDB DBC overrides or a verified external, version-pinned **complete** 12340 DBC extraction. No source/game/SQL/client modification. Newest IP is owner-reported, not compile-SHA verified.

## Executive result

We now know **exactly what data must be compared** before modifying any Hunter spell.

**Three distinct AzerothCore paths:**

1. **Family level-up list:** `SpellMgr::LoadPetLevelupSpellMap` matches each `CreatureFamily.dbc`'s two skill lines to `SkillLineAbility.dbc`, requires **`AcquireMethod = SKILL_LINE_ABILITY_LEARNED_ON_SKILL_LEARN`**, requires valid server `SpellInfo` and **nonzero `SpellLevel`**, then records `familyID → SpellLevel → SpellID`. `Pet::InitLevelupSpellsForLevel` grants entries at the pet's current level.
2. **Creature-specific defaults:** `SpellMgr::LoadPetDefaultSpells` first checks `creature_template.PetSpellDataId` and the corresponding `CreatureSpellData.dbc` row for template-specific defaults (map key **negative** PetSpellDataId). Separately, for **summonable creature templates without PetSpellDataId**, it can use template spell slots (map key **positive** creature entry). A deduplication helper clears defaults whose IDs also occur in the family's level-up set. `Pet::InitLevelupSpellsForLevel` uses the negative ID when present, otherwise the creature entry to look up defaults. **Do not assume every tamed beast obtains template defaults simply because it has creature_template spell slots.**
3. **Family passive list:** `DBCStores.cpp` builds `sPetFamilySpellsStore` from `SkillLineAbility.dbc` for a family's skill line when the corresponding `Spell.dbc` entry is **passive**, its **SpellLevel is zero**, and the acquisition method is **LEARNED_ON_SKILL_LEARN**. `Pet::LearnPetPassives` inserts the resulting spells as `PETSPELL_FAMILY` and `Pet::addSpell` directly casts each passive. `Pet::_SaveSpells` excludes family-typed entries, so they are regenerated, not treated as ordinary trained ranks.

Sources: [public AzerothCore SpellMgr.cpp](https://github.com/azerothcore/azerothcore-wotlk/blob/master/src/server/game/Spells/SpellMgr.cpp), blob `ea99b0e0a9e2fad716d21d2edbba0006e1eadd91`; [DBCStores.cpp](https://github.com/azerothcore/azerothcore-wotlk/blob/master/src/server/game/DataStores/DBCStores.cpp), blob `2fa6d8ae1d24d65dfd8c5d6b631c055a375404a2`; [Pet.cpp](https://github.com/azerothcore/azerothcore-wotlk/blob/master/src/server/game/Entities/Pet/Pet.cpp), blob `c58f4cc10364cafe68e994bf4f13946630fee611`.

**Important detail:** both **family level-up** and **family passive** selection use the same family skill lines/acquisition method, separated largely by **SpellLevel = nonzero vs zero and passive flag**. Thus a script that simply flags 'any spell in pet skill line is an auto-grant' would yield false matches. The final eligibility depends on loaded `SpellInfo` and actual loaded DBC data.

## Public AzerothCore SQL placeholders do not enumerate the spells

The public base WorldDB SQL for `creaturefamily_dbc`, `skilllineability_dbc`, and `creaturespelldata_dbc` creates empty tables: **zero INSERT statements in each**. These are NOT evidence that the realm's DBC stores are empty, nor that the user's deployed WorldDB override tables are empty.

[DBCStores.cpp](https://github.com/azerothcore/azerothcore-wotlk/blob/master/src/server/game/DataStores/DBCStores.cpp) uses `LoadDBC` to load `DataDir/dbc/<name>.dbc` and then calls `storage.LoadFromDB(dbTable, ...)`. Thus server file records and optional DB-supplied replacement/additional rows can BOTH matter.

Relevant public base schemas:
- [creaturefamily_dbc.sql](https://github.com/azerothcore/azerothcore-wotlk/blob/master/data/sql/base/db_world/creaturefamily_dbc.sql): `ID`, `SkillLine_1`, `SkillLine_2`, `PetTalentType`, etc.; **no source creature-template mapping**.
- [skilllineability_dbc.sql](https://github.com/azerothcore/azerothcore-wotlk/blob/master/data/sql/base/db_world/skilllineability_dbc.sql): `ID`, `SkillLine`, `Spell`, `AcquireMethod`, etc.; no Spell effect/damage columns.
- [creaturespelldata_dbc.sql](https://github.com/azerothcore/azerothcore-wotlk/blob/master/data/sql/base/db_world/creaturespelldata_dbc.sql): `ID`, `Spells_1..4`, `Availability_1..4`; doesn't prove that a particular Hunter tame has that PetSpellDataId.
- [Spell.dbc 3.3.5 field reference](https://www.azerothcore.org/wiki/spell): spells also need server effects, flags, levels and rank identities.

**No full 3.3.5 spell list was derived from these empty SQL files.** Do not falsely mark candidate IDs `NOT_PRESENT` or `NOT_AUTO_GRANTED` based on a missing `INSERT`.

## New machine-readable audit assets

- [21-ability audit](../data/reference/wotlk-versus-vanilla-ability-audit.csv) groups **all 111 existing historical research rank rows** into the 21 original ability names; preserves wild/trainer source, candidate Classic emulator spell IDs and historical exceptions. It contains *separate unknown fields* for native 3.3.5 family level-up, template default, family passive, server SpellStore and client Spell.dbc. **No native auto-grant status has been asserted without data**.
- [17-family audit](../data/reference/wotlk-family-auto-grant-audit.csv) retains every original Vanilla pet family and its Petopia permitted active abilities, with explicit unknown fields for 3.3.5 family IDs, both skill-line IDs, all three automatic grant paths, client/server DBC and actual summoned pet spell sets.
- Historical 1.12 spell candidates remain in [111-rank source crosswalk](../data/reference/vanilla-all-rank-id-candidate-crosswalk.csv), and **none** have a confirmed installed server/client spell mapping.

## Exact source-derived extraction recipe (READ-ONLY, for future data audit)

| Step | Inputs | Join/rule | Evidence/what to record |
|---|---|---|---|
| A | Actual deployed **CreatureFamily.dbc**, plus `creaturefamily_dbc` optional override | Family ID, SkillLine_1 and SkillLine_2 | One row per **deployed** family; join back to 17 Classic family candidates by independently verified family identity, never guess numeric family ID |
| B | Actual deployed **SkillLineAbility.dbc**, plus local `skilllineability_dbc` override | `SkillLine` matches either family skill line and `AcquireMethod` equals core's learned-on-skill-learn enum | Source SkillLineAbility row ID, SpellID and acquire method; distinguish from mere historical trainer ability |
| C | Actual deployed **Spell.dbc**/SpellStore and any overrides | Valid spell, **nonzero SpellLevel** | `LEVELUP_AUTO_GRANT`, which level, family, spell ID, rank chain, school/effects/client known |
| D | Same sources | Valid spell, `SpellLevel = 0`, **passive flag present** | `FAMILY_PASSIVE_AUTO_ADD`; immediate aura, family ID, spell ID, actual server effect and client state |
| E | Actual deployed **creature_template** and **CreatureSpellData.dbc** | For PetSpellDataId > 0: map **-PetSpellDataId** to up to four spell IDs; else templates only used by core when discovered in summon effects | `CREATURE_DEFAULT_AUTO_GRANT`, creature entry, PetSpellDataId, source row, spell slot and duplicate filtering versus family level-up |
| F | Actual runtime **Pet::InitLevelupSpellsForLevel** at each test level | Grant any eligible family-level/default spell; reject/retain too-high based on code's specific branches | Actual current spell list before/after tame, level-up and async pet DB load |
| G | **Original Vanilla 1.12** identified DBC and source list | Compare each 3.3.5 candidate's name, rank, school, effect, school resistance, TP cost and acquisition type | `MATCH`, `DIFFERENT_EFFECT`, `OUT_OF_ERA`, `SOURCE_UNAVAILABLE`, `MISSING` or `UNVERIFIED`; never use an ambiguous ID |
| H | Owner and IP modern API (effective stage) | Vanilla stages 0–7; TBC 8+ (owner-reported newest IP) | Human, Alt Bot and Random Bot actual owner stage, effective cap and group progression sync |

**Order matters:** a static table of DBC rows does not prove that a pet actually got the spell. The native runtime list, pet type, source entry, pet level, owner IP era and saved spell load are a separate verification tier.

## Confidence labels for every row

- `SOURCE_RULE_VERIFIED`: core's filter/call path is confirmed in pinned public source.
- `DBC_ROW_335_VERIFIED`: actual deployed DBC + override composite row has been inspected.
- `CLIENT_ROW_335_VERIFIED`: matching actual 3.3.5 client Spell.dbc exists with correct behaviour/tooltip.
- `RUNTIME_GRANT_VERIFIED`: an approved disposable pet actually received this spell from the recorded branch.
- `HISTORICAL_112_VERIFIED`: authenticated original versioned client/spell source verified.
- `MODULE_ONLY_GO`: after all previous safety checks, an owner-approved module-only method blocks unsupported ranks **before** unearned spell client/passive state; none established yet.
- `UNVERIFIED`: default for every 3.3.5 spell row in the new research CSVs.

Never confuse `SOURCE_RULE_VERIFIED` with `DBC_ROW_335_VERIFIED`; that distinction prevents 'automatic spell IDs' from being invented based on probable names.

## Module-only implementation and integrity implications

- **Modern IP is not the hard part now:** owner-reported latest IP makes 0–7 Vanilla stage policy a reasonable *planning baseline*. Still pin compiled SHA once testing is approved.
- **Auto-grant suppression remains independently blocked:** no generic `CanLearnPetSpell` veto found in public PetScript; cast denial and whole-packet veto do not solve passive rank/client spells.
- **WorldDB schema rows are not harmless imports:** do not populate native `skilllineability_dbc` or `creaturefamily_dbc` with original data, since they feed global pet spell maps across IP eras.
- **Native `pet_spell` invalid-ID handling has global side effects:** [documented](PET-SPELL-GRANT-PACKET-DB-SAFETY-AUDIT.md) that unknown native spell IDs can trigger `DELETE FROM pet_spell WHERE spell=?` for all pets. Do not seed raw historical candidates; backup character DB before staged mutation.
- **Save/reload passives regenerate:** even if native `pet_spell` rows appear correct, the core adds `PETSPELL_FAMILY` separately, so verify aura stats and action bar after stable/relog/restart.
- **No edits to AzerothCore, IP, Playerbots or Naxx Core** are proposed.

## Safe read-only input capture requested for a later stage

For a verified **read-only** server snapshot, collect the file hashes/build numbers and the **specific extracted row fields**, not copyrighted MPQ/DBC files uploaded to public GitHub. Any DBC extraction must use a compatible offline reader on **copies**, not overwrite the running server's assets. Check the active world database's three optional DBC override tables with **SELECT/COUNT only** after schema and backup context are known. Do not trust public empty base tables as the realm's live contents.

A future permission-gated pilot on disposable Hunters should start with only **one Cat and one Wind Serpent**, plus a control non-Vanilla Hunter, and observe passive flags and actual family/template spell grants before attempting any spell intervention.

**No such snapshot, staging pilot or module code exists yet in this project.**
