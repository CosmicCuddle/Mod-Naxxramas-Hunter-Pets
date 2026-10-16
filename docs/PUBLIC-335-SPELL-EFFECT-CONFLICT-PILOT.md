# First public 3.3.5 spell-effect conflict pilot — Screech, Howl, Lightning Breath, Growl and Stamina

**Research edition: 16 October 2026. No code, server, database, DBC or client changes.** This is a **public spell-database cross-check**, **not** evidence from the owner's installed server and **not** a verified reconstruction of an actual Blizzard original 1.12.1 DBC.

## What has now been established

The new [six-case source-linked effect comparison table](../data/reference/public-335-spell-effect-conflicts.csv) covers five important ability names with distinct **Classic Era reconstructed references** and public **WotLK 3.3.5a** database records. It corrects a tempting but unsafe assumption: **even where the numerical spell ID survives into 3.3.5, its level, cost, target, school/effect, duration, cooldown or scaling may have changed**.

| Case | Spell ID(s) | Classic reference | Public WotLK reference | Why it matters |
|---|---|---|---|---|
| **Screech R4** | **24579** in both | level **56**; **26–46** damage; melee AP **−100** for **4 sec**; no cooldown listed | 3.3.5 site: level **48**; **27–37** damage; melee AP **−140** for **10 sec**; **10-sec** cooldown | Same spell ID, **changed damage/debuff duration/cooldown/level**; requires effect restoration, not just spellbook filtering |
| **Furious Howl R1** | **24604** in both | Wolf's party members within **15 yards** get **+9–11 damage on their next physical attack** for up to **10 sec**; **60 Focus**, **10-sec** cooldown | 3.3.5 site: **+18 melee/ranged AP** to **wolf and master** for **20 sec**; **20 Focus**, **40-sec** cooldown | Different **buff mechanic, targets and resources**; directly affects the original Bloodaxe Worg identity and party utility |
| **Lightning Breath R5** | **25011** in both | Wind Serpent deals **78–90 Nature** damage; **50 Focus**; no cooldown listed; **level 48** | 3.3.5 site: **40–52 Nature** damage, **20 Focus**, **10-sec** cooldown, database reports spellpower scaling | Son of Hakkar Rank 6 and all lower wild-learn ranks need **effect-level** validation, not a raw spell-ID reuse |
| **Growl R2** | **14916** in both | Classic Rank 2: **65 threat**, **15 Focus**, **5-sec** cooldown | 3.3.5 site reports **65 threat plus 4.1 per level**, **15 Focus**, **5-sec** cooldown | A shared ID can have **threat-scaling differences**; original and WotLK server threat formulas need actual code/DBC comparison |
| **Great Stamina R1** | **4187** vs **61686**, **different IDs** | Passive **+3 flat Stamina**, pet-trainer/Training Points | 3.3.5 pet-talent spell **61686** grants **+4% Stamina** | **Different spell entries and training systems**; matching the *name* alone must not map 4187 to WotLK pet talent |
| **Post-Vanilla Screech R5** | **27051**, not Classic R4 | No original Vanilla fifth rank (Screech R4 uses 24579) | WotLK Classic database page labels 27051 as Rank 5, **level 64** | Never mistakenly import 27051 as Vanilla Screech R4 just because bot code lists it |

### Exact public references

- Screech: [Classic spell 24579](https://www.wowhead.com/classic/spell=24579) and [3.3.5 database spell 24579](https://wotlkdb.com/?spell=24579).
- Furious Howl: [Classic spell 24604](https://www.wowhead.com/classic/spell=24604) and [3.3.5 database spell 24604](https://wotlkdb.com/?spell=24604).
- Lightning Breath: [Classic spell 25011](https://www.wowhead.com/classic/spell=25011) and [3.3.5 database spell 25011](https://wotlkdb.com/?spell=25011).
- Growl: [Classic spell 14916](https://www.wowhead.com/classic/spell=14916) and [3.3.5 database spell 14916](https://wotlkdb.com/?spell=14916).
- Great Stamina: [Classic trained spell 4187](https://www.wowhead.com/classic/spell=4187) and [3.3.5 pet talent 61686](https://wotlkdb.com/?spell=61686). These **do not** imply spell 4187 is absent in 3.3.5.
- Extra Screech rank: [WotLK Classic 27051](https://www.wowhead.com/wotlk/spell=27051); **separate reconstructed expansion source**, not exact old retail 3.3.5 DBC or owner's binary.

**Source caveats:** Wowhead's current **Classic** and WotLK Classic entries are *modern reconstructions*, not direct Blizzard 2006 client file proof. `wotlkdb.com` says **WotLK 3.3.5a**, but its individual rows are **independent third-party reference pages** with no demonstrated correspondence to the owner's modified server. Do not attribute precise damage to the owner's runtime, since AC SpellInfo and SQL hotfixes, owner AP/SP scaling, IP modifiers or client patch assets can alter actual results. All source-derived numbers remain **public-reference candidates**.

## What this changes in the engineering plan

1. Treat **ability identity**, **rank availability**, **spell ID existence**, **automatic learning source**, **actual spell effect**, **trainer/wild source**, **native pet spell save/client packet** as **seven separate proof fields**.
2. Do not promote a Classic-to-WotLK **same ID** match to `IMPLEMENTABLE`, even if the WotLK client can display the spell. The different effect/cooldown/focus may require a separately approved module-owned spell script or genuine compatible client/server assets. **Do not assume hooks can replace passive, threat, tooltip or rank level consistently.**
3. For a Wind Serpent, test **Rank 5** and **Son of Hakkar Rank 6** separately: old Rank 6 is taught from a raid beast, while native WotLK auto-rank logic may override it; its actual 3.3.5 effect remains unverified.
4. For wolves, confirm Furious Howl's **party next-hit buff** can exist alongside modern 3.3.5 Ferocity/talent systems *only* on Vanilla-stage owner. It must not become a persistent WotLK-style AP multiplier for an entire group.
5. For Growl, the **0 Training Point** and **15 Focus cast** differences must be preserved while actual threat calculations follow the era policy; a zero-TP skill does *not* imply a free cast.
6. For Stamina/Armor/resistance trained passives, check if the original spell entry still exists and whether it increases stat and HP **exactly once**, versus auto-applied WotLK percent talent.
7. **Crucial:** these public spell pages say NOTHING about which spells are actually included in loaded `CreatureFamily`, `SkillLineAbility`, or `PetSpellData`, so [21-ability auto-grant table](../data/reference/wotlk-versus-vanilla-ability-audit.csv) must keep its three auto-grant flags **UNVERIFIED**.

## Next evidence gate

Read-only copy of actual **3.3.5 12340** server DBC plus applied `*_dbc` SQL overrides (and client DBC if custom), pinned SHA and 1.12 DBC if available. Analyse effects and source skill-line entries with a version-correct reader, **never** import unverified historical spell IDs into `pet_spell` (the core has broad invalid-ID cleanup). Then, only on owner-approved, backed-up, **disposable staging**, test **Cat, Wolf, Bat/Owl, Wind Serpent**, a TBC control Hunter, and a non-Hunter pet.

**Final result:** five spell-system names now have concrete **public-reference conflicts** documented. **Zero deployed/runtime spell mappings are proven**, and strict module-only auto-learn suppression remains a separate unsolved GO/NO-GO.
