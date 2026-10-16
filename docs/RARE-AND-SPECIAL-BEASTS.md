# Rare beasts, unusual attack speeds and meaningful ability hunting

**Purpose:** A list of historically grounded *reasons to tame*, not an arbitrary rare-pet power tier. Sources from [Classic Petopia attack speeds](https://www.wow-petopia.com/classic/attackspeed.php), [ability catalogue](https://www.wow-petopia.com/classic/abilities.php) and [Classic looks](https://www.wow-petopia.com/classic/).

## Strong first test candidates

| Beast (historical) | Family | Area | Classic property | Intended test |
|---|---|---|---|---|
| **Broken Tooth** | Cat | Badlands | **1.0 second** swing speed | Fast hits and PvP spell pushback; normalized damage |
| **Bloodseeker Bat** | Bat | Zul'Gurub | **1.0 second** swing speed | Distinct family, equally fast attack; not one 'best' cat |
| **Frenzied Bloodseeker Bat** | Bat | Zul'Gurub | **1.0 second** swing speed | Same as above, world availability and taming checks |
| **Deathmaw** | Wolf | Burning Steppes | **1.2 second** swing speed | Faster wolf without blanket rare buff |
| **The Rake** | Cat | Mulgore | **1.2 second** swing speed | Low-level rare speed preserved across levels |
| **Spiteflayer** | Carrion Bird | Blasted Lands | **1.2 second** swing speed | Non-cat fast attacker; family differences preserved |
| **Slavering Worg** | Wolf | Shadowfang Keep | **1.2 second** swing speed | Dungeon tame viability |
| **Humar the Pridelord** | Cat | The Barrens | **1.3 second** swing speed | Unique skin + speed, no undocumented extra DPS |
| **Son of Hakkar** | Wind Serpent | Zul'Gurub | **Lightning Breath Rank 6** teaching source | Ability acquisition and transfer pilot |
| **Bloodaxe Worg** | Wolf | Blackrock Spire | **Bite 8, Dash 3, Furious Howl 4** teaching source; **1.5 sec** speed | Multi-skill acquisition from one tame |
| **Winterspring Screecher** | Owl | Winterspring | **Claw 8 / Screech 4** teaching source | Two learnable ranks, ability learn-by-use |
| **Elder Shardtooth** | Bear | Winterspring | **Claw 8** teaching source | Alternate source for the same rank |
| **Kresh** | Turtle | Wailing Caverns | **Shell Shield 1** teaching source | Defensive family ability |

All entries require exact creature entry verification against the deployed **world DB**, historical spawn/tame eligibility and current IP availability. Do not promote human-readable names to database keys.

## Attack speed principles

- Classic Petopia records possible speeds: **1.0, 1.2, 1.3, 1.4, 1.5, 1.6, 1.7, 2.0, 2.4 and 2.5 seconds**.
- Pets of the same family with different swing speed should have **roughly equal baseline autoattack DPS** before contextual effects; faster swings do proportionally smaller damage.
- High swing frequency was most notable for PvP spellcast pushback and certain timing/proc contexts; do not promise flat PvE DPS superiority.
- Reassess any special haste/proc interactions, Focus and relative impact of ability casts under WotLK player scaling.
- Preserve **per-creature** source identity, even when multiple creatures share a family or appearance.
- Rare spawn chance and elite creature encounter stats do not need to persist as tamed-pet combat bonuses unless specifically documented.

## Historical edge case: Lupos

Lupos's old Shadow-damage special behaviour is associated with earlier Vanilla, **before the final late-Vanilla baseline**. It is **not in approved scope** and must remain excluded unless the owner chooses an earlier patch exception. Classic Petopia's broader historical notes should be cross-checked to establish exact patch change and rules.

## Ability hunting is often more important than retaining the source pet

Classic pet progression can involve temporarily taming a wolf or serpent to acquire a rank, teaching that rank to a treasured different pet later, and managing historically limited stable slots. The **Hunter's learned-catalogue** is therefore critical and must not be tied to the continued existence of the temporary source pet.

## Unlock/phase rules

- Zul'Gurub can be a source only while accessible to that character under IP; no out-of-tier free ability vendor.
- Players legitimately acquiring a higher-rank skill via another Hunter's pet must not have an illicit trade path—ability catalogue is **character-owned**, not shared across group or account.
- An ability from a previously unlocked stage can persist when its owner continues through later Vanilla stages, unless a separately approved feature says otherwise.
- No pet loot / quest reward / rare-spawn system should be rewritten incidentally.

## Validation checklist for each named creature

Name; verified creature entry; actual `creature_template` family; `type_flags` and tame eligibility; original attack time in Classic reference; deployed wild attack time; associated visual and scale; tame result entry after restart; pre-tame skills; 1.12 rank ID; available stage/spawn; trainer availability; intended role; references and confidence; screenshots/test logs; conflict flags.
