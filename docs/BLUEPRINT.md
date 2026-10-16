# Master blueprint — Vanilla Hunter pet restoration

**Document status:** Detailed Vanilla gameplay blueprint, preliminary research edition, 16 October 2026. Read alongside the [unified blueprint and roadmap](UNIFIED-BLUEPRINT-AND-ROADMAP.md) and the [work packages](WORK-PACKAGES.md) for implementation sequence and gated evidence. No implementation is authorised or claimed. Changes to this design require review and the project owner's approval.

## 0. Mission

Restore the complete distinctive Hunter-pet ecosystem of **late Vanilla WoW** on AzerothCore 3.3.5a with Individual Progression, preserving the meaning of **taming a specific beast**. All eligible Hunters in any Vanilla IP stage use the Vanilla rules until actual TBC entry. Later expansion policy remains unresolved.

This system must be a **standalone module** while experimental. It may move into Mod-Naxxramas-Core only after owner-approved test/release and a separate integration plan. No AzerothCore, IP, Playerbots, or Naxx Core source files may be changed.

## 1. Non-negotiable gameplay invariants

1. Tame originates in the actual world from a legal beast. Do not replace the tame journey with a gift menu.
2. Each successfully tamed pet remembers its source beast **creature entry** and pet identity, including stabled pets.
3. A wild beast's existing ability does not automatically become a permanent **Hunter-known** ability. Hunter learns it only through an accepted observed cast/use while that pet is possessed.
4. Pet trainer abilities are learned using the historical pet trainer, not as auto-granted WotLK pet ranks.
5. Hunter-known ability catalogue persists separately from the current pet; any compatible future pet can be trained from it.
6. Each pet has its own skill/rank, loyalty, happiness/food state and training-point budget.
7. Training is permitted only when **correct family + minimum level + Hunter knows rank + sufficient TP + no 5th active skill**.
8. `total TP = pet level × (loyalty rank − 1)`; net remaining TP = total TP minus current trained costs, including refund/replacement logic. Level 60, loyalty 6 = 300 TP. Negative balances on newly tamed creatures may occur historically and cannot be normalised away without design review.
9. Growl costs zero TP; upgrades need not purchase lower ranks first.
10. Exactly four **active** skills maximum; trainer passives do not count as active slots. Action-bar presence is not necessarily the same as learned-skill count.
11. Pet family, diet, physical stats and any validated creature-specific attack speed should be meaningful. **Rarity/elite status is not itself a pet damage multiplier**.
12. Faster attacks do not automatically give higher overall same-family autoattack DPS; normalize individual swing damage.
13. Existing WotLK auto-teaching and pet-talent behaviour may not silently invalidate Vanilla rank progression.
14. Happiness logic already present in AzerothCore should be reused if correct. Avoid double happiness damage bonuses.
15. A level 60 Hunter in a later IP era and a level 60 Hunter in Vanilla **must not** be conflated.
16. Preserve every pre-existing pet or refuse migration safely; never silently delete/overwrite stabled pets or spells.
17. No experimental pet desertion/runaway in early rollout. Permanent pet loss needs specific owner consent.
18. Warlock, Mage, Shaman and Death Knight controlled summons remain unaffected.
19. Human behaviour is first; Random Playerbots and Alt Bots require separate policies, persistence validation and careful compatibility.
20. Disabling module must have a defined **reversible transition**, not merely skip all handlers and abandon half-migrated characters.

## 2. Intended features by subsystem

| Subsystem | Historical behaviour | Module design direction | Release gate |
|---|---|---|---|
| Hunter quest | Complete level-10 tame/feed/training onboarding | Verify quest/API/client restrictions | Quest chain test |
| Tame | Source-specific family, appearance, abilities and stats | Capture source identity in a reliable event or documented before/after association | Prove lifecycle |
| Skill discovery | Hunter learns from *pet using the skill* | Record successful eligible casts; distinguish from passive effects; grant through persistent journal | Skill learned only once |
| Trainer | Buy passives/Growl rank catalogue | Pet trainer menu and Hunter-owned catalogue | Currency/level and retry |
| Skill teaching | Family/level/TP/active-limit | Authoritative server validation; idempotent commit | Atomic correctness |
| Loyalty | Six levels, XP and elapsed-time thresholds | Character+pet persistent state | Rank transition tests |
| Happiness | Feed correct diet; happy/content/unhappy damage | Reuse AzerothCore power if proven accurate | Damage and food tests |
| TP | Level × loyalty; respec; rank refund | Pet-specific ledger based on trained rank set | 300-TP fixture |
| Family profiles | Seventeen Vanilla families with distinct roles and food | Reference data + runtime family/stat integration | Matrix fixtures |
| Creature speeds | Preserved from specific tames | Original entry→attack time with damage normalization | Broken Tooth spike |
| Pet XP | Pet starts at tame level, levels through gameplay | Investigate IP catch-up and core synchronisation | No free WotLK catch-up |
| Stables | Native slot lifecycle, unique per-pet history | Never change slot count without migration approval | All slots remain intact |
| UI | Original Beast Training concept | NPC gossip proof-of-concept; optional 3.3.5 addon later | No client confusion |
| Bot AI | Pet remains functional for raids/solo | Compatibility layer, not upstream source edit | Human/Alt/Random tests |
| Era | Vanilla rules through all Vanilla IP | Owner-character era API; possible future migration | Mixed-era party tests |

## 3. Gameplay journey / end-to-end user story

1. Hunter unlocks normal taming through the level-10 quest. Hunter uses Beast Lore at higher levels to inspect intended tame skills.
2. Hunter sees a specific beast with an ability/rank worth obtaining. The beast's world availability and level follow IP.
3. Hunter tames it with normal spell; new pet starts with only historically appropriate native skills and its source identity.
4. Hunter fights with the pet and witnesses the desired skill actually cast; server records rank learned in Hunter's permanent catalogue.
5. Hunter may stable that pet or choose another (original limited stable capacity and owner migration policy must be decided).
6. Hunter visits a pet trainer to purchase universal ranks or opens the Beast Training substitute interface.
7. Hunter selects a skill/rank. Server validates exact pet identity, family, level, learned catalogue, active cap and available points.
8. Skill training and ledger change commit once, then pet spell list/action bar reflect it. Retry and relog must not duplicate charges.
9. Loyalty increases when historical elapsed-time, XP and happiness conditions are met; budget expands. Feeding correct foods keeps it happy.
10. For special pets, inherited attack speed (e.g., Broken Tooth) is retained after every lifecycle transition without inflating autoattack DPS.
11. At any Vanilla IP patch/raid stage this persists. TBC transition remains an explicitly gated future design.

## 4. Explicit version boundaries

- **Vanilla late-patch baseline** = provisional reference for all original progression tiers, *not* a promise of time-phased microchanges.
- **BC-only pet abilities excluded for now**: Avoidance, Cobra Reflexes, Fire Breath, Gore, Poison Spit and Warp. New BC families and Serpent family likewise excluded until TBC planning.
- **WotLK pet talent trees** are not a substitute for Classic training points; built-in pet talent UI cannot be assumed editable without client work.
- Early Vanilla anomalies such as Lupos Shadow damage are **not approved** just because they existed briefly.
- Do not treat private-server WowDB and current 3.3.5 Spell.dbc IDs as interchangeable with Vanilla Classic ability entries.

## 5. Source and authority model

- Gameplay rule sources: [Classic Petopia training](https://www.wow-petopia.com/classic/training.php), [ability/rank index](https://www.wow-petopia.com/classic/abilities.php), [differences](https://www.wow-petopia.com/classic/differences.php), [attack speeds](https://www.wow-petopia.com/classic/attackspeed.php) and [Loyalty history](https://warcraft.wiki.gg/wiki/Loyalty).
- Concrete runtime behaviour: **deployed AzerothCore** C++/SQL/DBC, not assumptions from Classic Era.
- Every future spell ID, creature entry, rank cost, family flag, creature acquisition and stat multiplier must have a dataset provenance and a server/client verification state before shipping.
- Machine-readable reference: [family CSV](../data/reference/vanilla-families.csv); gameplay and spell model: [Ability catalogue](ABILITY-CATALOG.md); architecture: [State machines](STATE-MACHINES.md); [Persistence and API](PERSISTENCE-AND-API.md).

## 6. What can NOT yet be called feasible

The existing `PetScript` does **not** expose a known veto to `Pet::InitLevelupSpellsForLevel` learning ranks. A post-factum workaround risks sending wrong spell packets, unlearning valuable skills, or race conditions. This alone can prevent a faithful module-only recreation. The developer must prove an acceptable solution in isolation. **No assumed core patch is an escape route.**

Likewise, exact pre-3.0 client Beast Training UI, 3.3.5 stable limits, saved native skill rank interactions, successful pet spell-use observation and WotLK talent suppression need explicit tests. Prove each before promising completion.

## 7. Delivery package required of development team

- Historical reference datasets with attribution, confidence, patch and exact 3.3.5 ID mappings.
- Version-pinned architecture diagram and exact callback signatures.
- Feature gates and roll-forward/back migrations.
- C++ implementation and reproducible build instructions, only once authorised.
- Test fixtures across all 17 families/21 abilities, named rarity outliers, IP stages and bots.
- Safety report on existing pets and rollback from enabled→disabled and module removal.
- Per-phase evidence, blockers, costs, owner sign-offs and clean integration handover.

This document is the high-level blueprint; linked specifications are normative for detail when approved.
