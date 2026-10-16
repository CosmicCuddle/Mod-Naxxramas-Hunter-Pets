# Vanilla Hunter pet patch history — research timeline

**Status: historical research**, not a request to simulate every patch as a separate ruleset. **All Vanilla IP stages** currently intended to use a single coherent Vanilla pet ruleset. Original variation is catalogued to maximise possible future restorations. Exact patch references should be frozen to an archive/source revision before implementation.

| Era/patch | Historical observation | Classification | Implementation significance | Evidence |
|---|---|---|---|---|
| Original early Vanilla, before 1.7 | Hunter pets already had tame/feed, loyalty, training concepts; earlier spell costs/family abilities differed | HISTORICAL_PHASE — DETAILS INCOMPLETE | Do **not** mix early costs with late-Vanilla TP rules | [1.7 change notes](https://warcraft.wiki.gg/wiki/Patch_1.7.0) |
| 1.3.0 | Removed accidental inherited wild creature resistances from tamed pets; immediate TP after pet level-up | ORIGINAL_PATCH_CHANGE — verify full context | Do not automatically grant creature's wild resistances as a tamed-pet bonus | [Hunter pet change history](https://warcraft.wiki.gg/wiki/Hunter_pet) |
| 1.5.0 | Fixed unstabling dead pets and preventing pets beyond stable capacity | ORIGINAL_PATCH_CHANGE | Persistence and slot handling need staging tests | [Hunter pet change history](https://warcraft.wiki.gg/wiki/Hunter_pet) |
| **1.7.0** | Trainer passives, TP recalculation, zero TP Growl, 4 active, new/changed skill lines, untrain costs and pet XP logic | STRONG ORIGINAL-PATCH SOURCE | Base reference for complete meaningful training ecosystem | [Original patch archive](https://warcraft.wiki.gg/wiki/Patch_1.7.0), [2005 press copy](https://worthplaying.com/article/2005/9/14/news/27376-world-of-warcraft-v170-euro-patches-available-now/) |
| **1.9.0** | Son of Hakkar tameable with Lightning Breath 6; boar Charge, gorilla Thunderstomp, turtle Shell Shield; physical base attacks | STRONG ORIGINAL-PATCH SOURCE | Ensures raid ability hunting and named unique examples | [Original patch notes](https://warcraft.wiki.gg/wiki/Patch_1.9.0), [archived copy](https://www.wowinterface.com/forums/showthread.php?t=2763) |
| **1.9.0** | 'Base pet speed standardized' wording; movement-versus-attack distinction ambiguous without supporting data | NEEDS ADJUDICATION | Do not remove pet-specific Broken Tooth swing speed based only on this sentence | [Petopia attack intervals](https://www.wow-petopia.com/classic/attackspeed.php), [Hunter discussion](https://us.forums.blizzard.com/en/wow/t/any-confirmation-on-hunter-pets/157836) |
| 1.9.0 | Lupos former Shadow physical-base attack removed by general physical-attack rule | HISTORICAL_EXCEPTION | Earlier Shadow phase is optional and off by default, not late-Vanilla | [Lupos history](https://warcraft.wiki.gg/wiki/Lupos), [1.9 notes](https://warcraft.wiki.gg/wiki/Patch_1.9.0) |
| Late Vanilla, pre-2.0 | 17 families, trainer/wild rank training and loyalty; rare melee attack-speed variants documented | PROVISIONAL BASELINE | Consistent player-experience target across Vanilla IP | [Classic Petopia differences](https://www.wow-petopia.com/classic/differences.php), [training](https://www.wow-petopia.com/classic/training.php) |
| 2.0 prepatch | New family eligibility including Serpents changes Vanilla late patch boundary | OUTSIDE INITIAL PRE-2.0 BASELINE | Do not silently include in original 17 | [Petopia differences](https://www.wow-petopia.com/classic/differences.php) |
| 3.0.2 | Training points/learned skills replaced by pet talent trees and automatic skill learning; loyalty removed | CONTRAST — CURRENT 3.3.5 ENGINE | Primary technical restoration hurdle | [Training point patch history](https://warcraft.wiki.gg/wiki/Training_point), [WotLK differences](https://www.wow-petopia.com/classic_lk/differences.php) |

## Research decisions to resolve later

- The project wants **maximum original Vanilla restoration**, not arbitrary early-patch **bugs** only for their novelty. Decide which original rules are deliberately preferred over modern Classic reconstruction; choices require exact source proof.
- Full patch-by-patch dynamic simulation is **not part of agreed requirements**. Research all patches, ship one coherent Vanilla ruleset unless asked otherwise.
- Treat **melee attack speed**, **movement speed**, and **base damage school** as independent fields. Source confusion around patch 1.9 must be resolved before writing per-creature modifiers.
- Pet families, source-level abilities and spawn/tame eligibility may vary with original patches and current IP phasing. The actual installed IP stage needs explicit testing, not assumptions based on character level.

## Next evidence collection

Original patch notes beyond 1.7/1.9 where pets/taming/XP were changed, an archived version-specific creature/ability dataset (e.g., properly versioned classic emulator DB), exact creature and spell identifiers for the initial named pet pilots, and a server-side comparison against deployed AzerothCore and IP.
