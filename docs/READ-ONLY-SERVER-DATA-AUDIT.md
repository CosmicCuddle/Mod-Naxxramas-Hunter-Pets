# Read-only world/character database verification plan

**Status:** Future research instructions ONLY. **No SQL has been executed**, and no files or servers have been changed. These are example **read-only SQL queries**, to run only after the owner chooses a read-only snapshot or approves read access in HeidiSQL/Kiera3. Never edit live templates as part of research.

## Why we need this

Classic Petopia and other archival guides establish the **historic design**, but this server uses **AzerothCore 3.3.5a** with an individual progression module, custom bosses/quests and Playerbots. NPC IDs and names are starting clues, not proof that an NPC in `acore_world` can be tamed, spawns in an available raid or possesses the proper 1.12 ability spell.

## Safe preparation, before any future inspection

1. Confirm the **actual deployed database name** (`acore_world` may have been renamed), DB version, AzerothCore commit and installed IP module version. If choosing a live server connection, use a login with **SELECT-only permissions** where practical.
2. Read-only work should be performed on an **existing backup or a dedicated read replica** when one is available; do not perform an unapproved database restore on production. Preserve a timestamped **backup** of critical world/characters data before any eventual change or migration.
3. The first SQL query should inspect the current table structure rather than assuming `creature_template` matches a particular AzerothCore branch.
4. Do **not** run `UPDATE`, `DELETE`, `INSERT`, `ALTER`, `TRUNCATE`, `REPLACE` or mass repair helpers. All listed examples below are `SELECT` or `SHOW`.
5. Results containing player names or private information should remain in local protected reports rather than being committed to this **public** GitHub repository.

## Query A — inspect actual creature_template columns

Run only after setting the **correct read-only schema**.

```sql
SHOW COLUMNS FROM acore_world.creature_template;
```

Typical relevant fields may include `entry`, `name`, `family`, `type`, `type_flags`, `minlevel`, `maxlevel`, `BaseAttackTime`, `RangeAttackTime` and potentially pet-related spell data. **Do not assume every field exists or is meaningful on this build.** Record actual types.

## Query B — six confirmed historic NPC ID candidates

Use the columns already confirmed by Query A. The following minimal query depends only on `entry` and `name`:

```sql
SELECT entry, name
FROM acore_world.creature_template
WHERE entry IN (2850, 7456, 9696, 11357, 14476, 16117)
ORDER BY entry;
```

Expected *historic* names, to verify rather than force:
- `2850`: Broken Tooth
- `7456`: Winterspring Screecher
- `9696`: Bloodaxe Worg
- `11357`: Son of Hakkar
- `14476`: Krellack
- `16117`: Plagued Swine

If entries are absent or names don't match, report discrepancy. Do **not** create new templates just so the research table matches.

## Query C — inspect all candidate template fields

**Only after Query A confirms the columns**, an extended *example* for AzerothCore-style schemas is:

```sql
SELECT
    entry, name, minlevel, maxlevel,
    type, type_flags, family, BaseAttackTime, RangeAttackTime
FROM acore_world.creature_template
WHERE entry IN (2850, 7456, 9696, 11357, 14476, 16117)
ORDER BY entry;
```

This returns *wild template* values, **not guaranteed post-tame attack speed, abilities or pet statistics**; AzerothCore pet calculations can ignore wild NPC attack timing. Also `type_flags` by itself isn't a complete proof of taming eligibility.

## Query D — inspect spawn storage layout before checking availability

```sql
SHOW COLUMNS FROM acore_world.creature;
```

Some branches use `id`, others `id1`/multiple template fields and other mapping conventions. Adapt the subsequent read-only source query to **what Query D actually shows**. Record spawn map IDs, position, phase/conditions, explicit instance spawns and IP locks. A beast could be in a database without being accessible to a particular Hunter at that IP stage.

## Query E — discover native pet storage shape, not players' details

```sql
SHOW TABLES FROM acore_characters LIKE '%pet%';
```

Then run `SHOW COLUMNS` only for tables that actually exist (e.g. pet main/stable/spell/cooldown tables), and save schema/metadata only. Avoid retrieving or publishing character names, accounts, raw pet identifiers or owners before permission and privacy review.

## Query F — reference checks beyond world SQL

- Inspect actual `CreatureFamily.dbc` and compare family masks to [17-family reference](VANILLA-FAMILY-REFERENCE.md).
- Inspect deployed **server AND client** `Spell.dbc` (plus `SkillLineAbility.dbc`, `PetSpellData.dbc` if relevant), map all [111 catalogue rows](../data/reference/vanilla-ability-ranks.csv) by **canonical ability+rank**, not by name alone.
- For every creature, compare 1.12 availability and original tame source with **actual installed IP** gating, core/world spawn and tame eligibility; do not infer IP stage from level alone.
- Capture exact attached world database snapshot version, and note if Naxx custom NPCs or modules override the template or spell list after worldserver startup.
- Perform spell→natural source mapping only with server-side code/DBC facts. The wild NPC's `spellX` columns (where present) may not represent hunter-pet learned spells, and some NPC AI casts are not automatically inherited on tame.

## Required compact evidence output (not raw DB dump)

```text
AUDIT-ID:
world schema / date:
AzerothCore/IP/Playerbots commits:
source-beast name:
historic candidate NPC entry:
actual world template entry:
actual family/tameability:
actual wild level and BaseAttackTime:
present in spawn records:
IP phase accessibility:
expected original ability+rank:
deployed server spell mapping:
deployed client spell mapping:
historical source URL:
technical feasibility status:
discrepancies / follow-up:
```

Run Query B before expanding to all [141 source rows](../data/reference/vanilla-tame-ability-sources.csv). Save only the **non-sensitive evidence needed to prove template identity and rank mapping** to GitHub; keep any protected realm character data private.

## Safety and deployment boundary

These `SELECT`/schema examples form a **future research checklist**, not an instruction to connect to the server now. **No server or database has been accessed during this documentation work.** Gameplay implementation remains entirely in this standalone module, with no edits to AzerothCore, IP, Playerbots or Naxx Core.
