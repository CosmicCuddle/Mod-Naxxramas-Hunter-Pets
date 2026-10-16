# Annotated reference library — Vanilla Hunter pets

**Prepared:** 16 October 2026. Webpages and upstream source may change; freeze SHA/data snapshots as necessary before implementation. Links are citations/reading material, **not permission to bulk-copy copyrighted prose, screenshots or source code**.

## Primary gameplay research — Vanilla / Classic

| Resource | Use | Caveat |
|---|---|---|
| [Classic Petopia: full abilities](https://www.wow-petopia.com/classic/abilities.php) | 21 ability names, rank levels, TP costs, tame teaching beasts | Classic reconstruction; exact original 1.12 IDs require cross-check |
| [Classic Petopia: training](https://www.wow-petopia.com/classic/training.php) | Learn-by-use, trainer, TP formula, rank refund, four-active cap, Growl | UI screenshots are from different client; do not assume available 3.3.5 |
| [Classic Petopia: differences](https://www.wow-petopia.com/classic/differences.php) | Three owned pets (1 active + 2 stable), loyalty, pet XP, 17 families | Modern Classic recreation not always identical to historical 2006 |
| [Classic Petopia: attack speeds](https://www.wow-petopia.com/classic/attackspeed.php) | Listed rare/normal original speeds 1.0–2.5 sec and damage normalization | Need original NPC entry mapping + actual in-game timing |
| [Classic Petopia: pet search](https://www.wow-petopia.com/classic/search.php) | Search by zone, beast family, rare, original teach rank, tameable | Data may include later patch interpretations |
| [Classic Petopia: 17 family pages](VANILLA-FAMILY-REFERENCE.md) | Family stats, role, ability whitelist, diets, looks | Source ratios, not ready-to-apply 3.3.5 formula |
| [WoW Classic Pet Guide — Wowhead](https://www.wowhead.com/classic/guide/classic-wow-hunter-pets) | Independent summary of TP, happiness damage and ability training | Useful cross-check not independent retail patch data |
| [Warcraft Wiki: loyalty](https://warcraft.wiki.gg/wiki/Loyalty) | Historical six ranks, 5%-XP and time requirements, patch removal | Historic reported timings require replication |
| [Warcraft Wiki: training points](https://warcraft.wiki.gg/wiki/Training_point) | Removed in patch 3.0.2; old TP and respec history | Contains later-era context, label patch |
| [Warcraft Wiki: Classic pet abilities](https://warcraft.wiki.gg/wiki/Pet_abilities_(Classic)) | Independent rank/family and cost data | May differ in patch-specific details |

## Reference emulator/source code (for logic and differences only)

| Source | Why | Rule |
|---|---|---|
| [CMaNGOS Classic pet engine](https://github.com/cmangos/mangos-classic/blob/master/src/game/Entities/Pet.cpp) | **Critical:** loyalty, happiness, training, teach-to-Hunter, tame/save handling | Read-only study; no direct transplant |
| [CMaNGOS Classic pet class header](https://github.com/cmangos/mangos-classic/blob/master/src/game/Entities/Pet.h) | Loyalty/TP APIs and class state | Revision-pin and inspect license |
| [CMaNGOS Classic pet handler](https://github.com/cmangos/mangos-classic/blob/master/src/game/Entities/PetHandler.cpp) | Training spell validation, reset and packet expectations | Different protocol / core design |
| [CMaNGOS Classic character DB](https://github.com/cmangos/mangos-classic/blob/master/sql/base/characters.sql) | Historical persistent loyalty/TP/learnable spells | Schema is reference, not migration template for AC |
| [AzerothCore Pet.cpp](https://github.com/azerothcore/azerothcore-wotlk/blob/master/src/server/game/Entities/Pet/Pet.cpp) | Target engine level/spell/happiness/attack stat behaviour | Pin actual deployed source |
| [AzerothCore PetScript.h](https://github.com/azerothcore/azerothcore-wotlk/blob/master/src/server/game/Scripting/ScriptDefines/PetScript.h) | Available hooks and missing general learn veto | Hook definitions may change |
| [AzerothCore PlayerScript.h](https://github.com/azerothcore/azerothcore-wotlk/blob/master/src/server/game/Scripting/ScriptDefines/PlayerScript.h) | Login, logout, level, save, gossip | Check correct event order |
| [AzerothCore AllSpellScript.h](https://github.com/azerothcore/azerothcore-wotlk/blob/master/src/server/game/Scripting/ScriptDefines/AllSpellScript.h) | Cast hooks relevant to learn-by-use | OnSpellCast not proof of success |
| [AzerothCore hook reference](https://www.azerothcore.org/wiki/hooks-script-reference) | Hook discovery | Source for exact revision beats documentation |
| [Individual Progression](https://github.com/ZhengPeiRu21/mod-individual-progression) | Per-character expansion policy and pet level adjustments | Deployed fork/revision may differ |
| [Playerbots](https://github.com/mod-playerbots/mod-playerbots) | Bot pet creation/ability/autocast AI | Never edit upstream |
| [Mod-Naxxramas-Core](https://github.com/CosmicCuddle/Mod-Naxxramas-Core) | Target eventual integration, existing MC consumables, NT1 | Separate repository during experiments |

## Explicit expansion comparison sources — to detect contaminating information

- [TBC Petopia ability catalogue](https://www.wow-petopia.com/classic_bc/abilities.php): six introduced abilities (Avoidance, Cobra Reflexes, Fire Breath, Gore, Poison Spit, Warp) and new families; **not** allowed in initial Vanilla reference data.
- [WotLK Petopia differences](https://www.wow-petopia.com/classic_lk/differences.php): increased stable capacity, simplified skill progression, happiness and diet changes.
- [WotLK Lightning Breath](https://www.wow-petopia.com/classic_lk/ability.php?id=lightningbreath) vs [Vanilla Lightning Breath](https://www.wow-petopia.com/classic/abilities.php): clear case of same-named ability with different ranks/levels/effects.

## Version conflicts requiring explicit reconciliation

1. Late-original Vanilla 1.12 vs Classic Era release features (e.g. pet stat bugfixes and NPC skill datasets).
2. Some third-party catalogues report slightly different aggregate **rank counts** for the same 21 names; this blueprint's **111 rank records** are a count from the Petopia rank-level index, not independently verified source count. Never treat count match as proof of valid spell IDs.
3. Native 3.3.5 spell/family IDs may not match classic spell IDs even if names do.
4. Pre-1.9 historical Lupos Shadow damage must not be silently mixed into 1.12 baseline.
5. Classic 2-stable capacity vs WotLK 4-stable capacity is a destructive migration risk.
6. Guide descriptions for Furious Howl differ significantly between Classic, TBC, Wrath and some modern 'Forever' formats; use explicitly **Classic** guides only for Vanilla.

## Provenance template (per research assertion)

```text
fact_id:
topic:
claim:
era:
patch:
source_url:
source_type: (original data | emulator source | reconstructed guide | in-game observation)
source_revision_or_checked_at:
independent_corroboration:
confidence: (high | medium | low | conflicting)
actual_deployed_core_checked: (yes/no)
actual_client_dbc_checked: (yes/no)
implementation_status: (research_only | verified | blocked | approved)
notes:
```

No unverified number goes into production SQL/DBC. Apparent conflicts should be recorded with both sources and addressed in a specific decision entry.
