# Read-only 3.3.5 Hunter pet DBC input checklist (Linux/MobaXterm + HeidiSQL)

**Status: FUTURE INPUT COLLECTION ONLY, NOT RUN.** Owner has not approved installation, database changes or code compilation. This page gives a short safe path to the real data needed by the [three-path AzerothCore auto-grant comparison](WOTLK-AUTO-GRANT-DBC-COMPARISON.md).

## Why an actual server input is needed

The public `creaturefamily_dbc.sql`, `skilllineability_dbc.sql` and `creaturespelldata_dbc.sql` tables contain **no INSERT records in the repository**. Actual runtime spell ranks are built from **binary `DataDir/dbc/*.dbc`** combined with optional local SQL overrides, and filtered by the current core. Even a source-supported 3.3.5 spell ID must not be accepted until the actual server/client row is checked.

Do not post game client MPQ/DBC binaries or private character databases into the public GitHub repository. **Version, SHA-256 hashes and narrowly scoped research values** are sufficient for the next public report. Work only on copies if exporting DBC.

## 1. MobaXterm — locate the server's working DBC files

First locate the actual configured AzerothCore `DataDir` under the active **worldserver.conf**, without sharing its entire contents. Do not modify the config. The following is an **example after you have confirmed the actual directory**:

```bash
# REPLACE ONLY after confirming the real active DataDir.
DBC_DIR=/root/cosmic_core/data/dbc

# Read only: show whether four source files exist.
ls -l "$DBC_DIR/CreatureFamily.dbc" \
      "$DBC_DIR/SkillLineAbility.dbc" \
      "$DBC_DIR/CreatureSpellData.dbc" \
      "$DBC_DIR/Spell.dbc"

# Read only: record hashes without opening or modifying DBC.
sha256sum "$DBC_DIR/CreatureFamily.dbc" \
          "$DBC_DIR/SkillLineAbility.dbc" \
          "$DBC_DIR/CreatureSpellData.dbc" \
          "$DBC_DIR/Spell.dbc"
```

**If any file path fails**, don't copy/download random replacement DBC files or change `DataDir`. First identify which worldserver configuration file and DBC directory the **running binary** actually uses. Do not assume the example directory is correct.

## 2. HeidiSQL — check optional WorldDB DBC overrides

Open your existing connection, select the **world database** (historically `acore_world`, verify actual name first). These queries are read-only and **do not modify tables**. Execute **one at a time** after the schema is confirmed:

```sql
-- First: do these optional tables even exist?
SHOW TABLES LIKE 'creaturefamily_dbc';
SHOW TABLES LIKE 'skilllineability_dbc';
SHOW TABLES LIKE 'creaturespelldata_dbc';

-- Only for confirmed existing tables:
SELECT COUNT(*) AS creature_family_override_rows
FROM creaturefamily_dbc;

SELECT COUNT(*) AS skill_line_override_rows
FROM skilllineability_dbc;

SELECT COUNT(*) AS creature_spelldata_override_rows
FROM creaturespelldata_dbc;
```

A zero count means **no records in that WorldDB override table**; it does **not** mean corresponding DBC data or auto-granted spells are absent.

If an override table has rows, use the following **read-only and bounded** samples after confirming columns; no exports of full private DBs needed:

```sql
SELECT ID, SkillLine_1, SkillLine_2
FROM creaturefamily_dbc
ORDER BY ID
LIMIT 50;

SELECT ID, SkillLine, Spell, AcquireMethod
FROM skilllineability_dbc
ORDER BY ID
LIMIT 50;

SELECT ID, Spells_1, Spells_2, Spells_3, Spells_4
FROM creaturespelldata_dbc
ORDER BY ID
LIMIT 50;
```

These are **only partial samples**, not the complete pet ability list. Any nonempty table may require a *controlled full read-only extraction of precisely the relevant family/skill-line IDs* once those IDs are established from server DBC.

## 3. Distinguish installed build from Git HEAD

The `git` revision of the source directory may be newer than the actual **compiled/running worldserver**. Before an implementation trial, record the running build/date using the existing safe diagnostics and [installed IP checklist](READ-ONLY-INSTALLED-IP-INVENTORY.md), then cross-check actual header symbols. No `git pull`, rebuild or restart is required to run the read-only checks above.

## 4. Actual DBC row extraction is still separately required

The `SELECT` counts and hashes **alone** cannot enumerate 3.3.5 spells. Later research must parse **read-only copies** of all four versioned server files with a trustworthy 3.3.5-compatible reader and record:

- Family ID and **both SkillLine IDs**.
- For each matching SkillLineAbility: `SkillLine`, `Spell`, `AcquireMethod`, and any rank ordering.
- For each matching 3.3.5 Spell: `SpellLevel`, passive flag, rank, focus cost, school, cooldown, damage/effects and relevant script overlays.
- For relevant source beasts: actual `creature_template` entry / `PetSpellDataId`, matching `CreatureSpellData` spells and summon fallback eligibility.
- The **actual client's** corresponding Spell.dbc data/version so no tooltip/effect mismatch is silently assumed.

Write only narrow, versioned derived findings into [21-ability audit](../data/reference/wotlk-versus-vanilla-ability-audit.csv) and [17-family audit](../data/reference/wotlk-family-auto-grant-audit.csv). Never copy raw copyrighted binaries to public repository or turn a candidate Classic ID into a live spell grant.

## 5. Safety / stop rules

- **No SQL writes:** no `INSERT`, `UPDATE`, `DELETE`, `REPLACE`, `ALTER` or database restore for this research.
- **No gameplay mutation:** no pet spells granted/removed, no stabling/abandoning valuable pets, no `.ip set`, no Playerbot gear or talent changes.
- **No new client patches / DBC edits / external data uploads.** Do not use source tables to replace global family skill lines.
- Native `pet_spell` invalid ID cleanup can DELETE by spell ID **across all pets** (see [pet DB safety audit](PET-SPELL-GRANT-PACKET-DB-SAFETY-AUDIT.md)), so **never inject unverified original spell candidates**, even into test characters on production.
- If actual running server revision/DBC sources cannot be pinned, status is `UNVERIFIED` and no implementation follows.

**No output in this document represents a command performed on the owner's server.**
