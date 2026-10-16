# Vanilla families — complete initial 17-family reference

**Status:** Research reference as of 16 October 2026; derived from each [Classic Petopia](https://www.wow-petopia.com/classic/) family page. **Not verified as directly usable** in the deployed 3.3.5a pet-stat implementation.

## Families, roles, multiplicative modifiers and diet

| Family | Role | Health | Armor | Damage | Diet | Distinctive active spell(s) |
|---|---|---:|---:|---:|---|---|
| Bat | Offense | 1.00 | 1.00 | 1.07 | Fruit, fungus | Screech, Dive |
| Bear | Defense | 1.08 | 1.05 | 0.91 | Bread, cheese, fish, fruit, fungus, meat | Bite, Claw |
| Boar | Defense | 1.04 | 1.09 | 0.90 | Bread, cheese, fish, fruit, fungus, meat | Charge, Dash |
| Carrion Bird | General | 1.00 | 1.05 | 1.00 | Fish, meat | Screech, Dive |
| Cat | Offense | 0.98 | 1.00 | 1.10 | Fish, meat | Claw, Bite, Dash, Prowl |
| Crab | Defense | 0.96 | 1.13 | 0.95 | Bread, fish, fruit, fungus | Claw |
| Crocolisk | Defense | 0.95 | 1.10 | 1.00 | Fish, meat | Bite |
| Gorilla | Defense | 1.04 | 1.00 | 1.02 | Fruit, fungus | Thunderstomp |
| Hyena | General | 1.00 | 1.05 | 1.00 | Fruit, meat | Bite, Dash |
| Owl | Offense | 1.00 | 1.00 | 1.07 | Meat | Screech, Dive, Claw |
| Raptor | Offense | 0.95 | 1.03 | 1.10 | Meat | Claw, Bite |
| Scorpid | Defense | 1.00 | 1.10 | 0.94 | Meat | Scorpid Poison, Claw |
| Spider | Offense | 1.00 | 1.00 | 1.07 | Meat | Bite |
| Tallstrider | Defense | 1.05 | 1.00 | 1.00 | Cheese, fruit, fungus | Dash, Bite |
| Turtle | Defense | 1.00 | 1.13 | 0.90 | Fish, fruit, fungus | Shell Shield |
| Wind Serpent | Offense | 1.00 | 1.00 | 1.07 | Bread, cheese, fish | Lightning Breath, Dive |
| Wolf | General | 1.00 | 1.05 | 1.00 | Meat | Furious Howl, Dash, Bite |

All families can historically train **Cower and Growl** and the relevant universal passives (Arcane, Fire, Frost, Nature, Shadow Resistance, Great Stamina, Natural Armor). Above 'distinctive' column is *not* the exhaustive whitelist. The canonical per-family allowed active list must use [the CSV](../data/reference/vanilla-families.csv) and be reviewed against the full Petopia family page before coding.

**Critical interpretation:** These are ratios against a reference family-stat base, not a mandate to multiply already-family-modified WotLK totals. Exact multiplication stage and interaction with IP level/stat scaling must be discovered. Do not blindly reapply a multiplier to a value that already includes it.

## One reference per family

- [Bats](https://www.wow-petopia.com/classic/family.php?id=bat)
- [Bears](https://www.wow-petopia.com/classic/family.php?id=bear)
- [Boars](https://www.wow-petopia.com/classic/family.php?id=boar)
- [Carrion Birds](https://www.wow-petopia.com/classic/family.php?id=carrionbird)
- [Cats](https://www.wow-petopia.com/classic/family.php?id=cat)
- [Crabs](https://www.wow-petopia.com/classic/family.php?id=crab)
- [Crocolisks](https://www.wow-petopia.com/classic/family.php?id=crocolisk)
- [Gorillas](https://www.wow-petopia.com/classic/family.php?id=gorilla)
- [Hyenas](https://www.wow-petopia.com/classic/family.php?id=hyena)
- [Owls](https://www.wow-petopia.com/classic/family.php?id=owl)
- [Raptors](https://www.wow-petopia.com/classic/family.php?id=raptor)
- [Scorpids](https://www.wow-petopia.com/classic/family.php?id=scorpid)
- [Spiders](https://www.wow-petopia.com/classic/family.php?id=spider)
- [Tallstriders](https://www.wow-petopia.com/classic/family.php?id=tallstrider)
- [Turtles](https://www.wow-petopia.com/classic/family.php?id=turtle)
- [Wind Serpents](https://www.wow-petopia.com/classic/family.php?id=windserpent)
- [Wolves](https://www.wow-petopia.com/classic/family.php?id=wolf)

## Additional family work still needed

- Match **actual numeric CreatureFamily.dbc family IDs** against each human-readable family in the **deployed** 3.3.5 server; do not invent IDs in research.
- Verify family whitelist to original Classic ability ranks and current server spell IDs.
- Verify NPC entries that are tamable and available under IP; modern creature templates may mark formerly tameable or untameable beasts differently.
- Verify food categories versus real `item_template` FoodType and effective pet level; do not infer food acceptance from display name.
- Confirm spell-school and family resistances, HP/armor/damage formulas, Beast Mastery talent effects, happy/content/unhappy modifiers, tier-limited scaling.
- Confirm all 17 families are selectable without adding pre-TBC families (Serpents, Dragonhawks, Ravagers, etc.) and no 3.3.5 Exotic-only logic leaks into Vanilla.
- Distinguish pets with the same family but different appearance, innate ability rank and original attack speed.
