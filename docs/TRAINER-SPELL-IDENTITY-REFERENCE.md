# Trainer-sourced spell identity — first focused comparison

**Status:** Early research. **3 representative entries from 52 trainer-rank rows** are examined here; not a complete source-to-spell mapping and not a runtime patch. All original 1.12 IDs must be verified from proper versioned DBC.

Why this matters: **eight** Vanilla pet trainer ability names have been retained in the Hunter ecosystem in various ways, but the **underlying spells changed** by 3.3.5. A textual match such as `Great Stamina` or `Growl` must never be interpreted as sufficient evidence of the correct spell effect.

## Three research examples

| Vanilla ability and rank | Pet level / TP | Classic-era spell ID candidate | Classic behaviour observed in reference | Dangerous WotLK comparison |
|---|---|---:|---|---|
| **Great Stamina 1** | 10 / 5 | **4187** | A **flat +3 Stamina passive**, from a pet trainer; it is not a percent-of-Stat aura | [WotLK Great Stamina 61687](https://www.wowhead.com/wotlk/spell=61687) describes **8% Stamina**, a different effect/ID |
| **Natural Armor 2** | 12 / 5 | **24549** | **+100 physical armor**, passive | Other 3.3.5 family passive and scaling spells may already add armor; don't double-apply |
| **Growl 7** | 60 / **0** | **14921** | Active pet threat ability with [Classic spell record](https://www.wowhead.com/classic/spell=14921) documenting 15 Focus, 5 sec cooldown, base 415 threat | WotLK pet threat/taunt scaling can differ; 0 **training points** is independent of the 15 **Focus** cast cost |

**Sources:** [Classic Great Stamina 4187](https://www.wowhead.com/classic/spell=4187), [Classic Natural Armor 24549](https://www.wowhead.com/classic/spell=24549) or [TBC page retaining old data](https://www.wowhead.com/tbc/spell=24549), [Classic Growl 14921](https://www.wowhead.com/classic/spell=14921), [Petopia trainer ranks and TP costs](https://www.wow-petopia.com/classic/abilities.php), [WotLK Great Stamina](https://www.wowhead.com/wotlk/spell=61687). Record source-version issues: modern Wowhead 'classic' is not direct original 2006 binary proof.

**Key technical lesson:** Old Great Stamina **+3 flat** and WotLK **+8%** may produce dramatically different results. It is unsafe to use the WotLK spell because the UI label matches.

## Mechanics to clarify for trainer passives

- The old TP cost is incurred **per individual pet** when training, and only the highest current rank should provide its full non-stacking effect.
- A pet trainer can teach the **Hunter** a rank, but the ability is not necessarily granted to the active pet until the owner trains it and has enough points.
- The native core may grant family passive spells automatically. Determine which spells are engine-required and which are distinct optional Vanilla trainer passives before suppressing them.
- Resistances are five separate school-specific skill lines; verify their *actual spell IDs, rank and values* independently. Never map all resistances to a single modern family aura.
- Growl is an **active** skill but costs **0 TP**; its **Focus** cast cost still applies. That exception must be reflected in validation, display and tests.
- The 3.3.5 *pet talent* system is a separate WotLK feature and does not count as the old classic TP progression. Both may need safe coexistence/disable policy inside the permitted module boundaries.

## Read-only evidence required for all 52 trainer ranks

```text
ABILITY / RANK / MIN PET LEVEL / TP:
ORIGINAL CLASSIC/1.12 SPELL ID:
TRAINER KNOWN-SPELL SPELL ID / TRAINER WRAPPER ID:
TRAINER COIN PRICE / REQUIRED OWNER LEVEL:
AURA OR ACTIVE / FOCUS:
ORIGINAL EFFECT / SCHOOL / MODIFIER:
INDEPENDENT SOURCE URL:
DEPLOYED SERVER + CLIENT 3.3.5 DBC MATCH:
PRESENT IN NATIVE PET SPELL LIST / PASSIVE:
ACTUAL VALIDATED TRAINER NPC:
MODULE-ONLY FEASIBILITY:
RESEARCH STATUS:
```

The trainer **wrapper** (Hunter learning a training recipe/rank) and pet's **actual passive/combat spell** may be separate IDs. Don't conflate these. A pet's actual spell ID cannot be derived from an NPC trainer's spell-list entry without examining its effects.

## Future scope

There are **52** trainer-rank rows in our current research dataset: **7 Growl**, **10 Great Stamina**, **10 Natural Armor**, and **5 each for Arcane, Fire, Frost, Nature and Shadow Resistance**. Only three candidate IDs have been sampled here, and none has been checked against installed `Spell.dbc` on either client or server.

Recommended next research: find version-pinned original 1.12 trainer wrapper and pet spell IDs from original-compatible DBC/trainer lists, then audit all 52 against deployed 3.3.5. If missing, do **not** synthesise ID mappings from similar names.

**No SQL executed, no client files modified, and no AzerothCore source changed.**
