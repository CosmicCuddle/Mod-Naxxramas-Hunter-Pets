# WP-03 — pet spell granting, packets and database integrity audit

**16 October 2026. RESEARCH ONLY.** Public AzerothCore source reviewed through connected GitHub. No staging experiments, compiled module, SQL execution, character inspection, client packet capture, or server/client modification. Owner reports **newest Individual Progression** (modern hidden-quest stage targeting), but exact running compiled AzerothCore/IP revisions are unverified.

## Technical conclusion — strict original Beast Training remains unproven module-only

In public AzerothCore, automatic family/default pet spells reach **`Pet::learnSpell` → `Pet::addSpell` directly**. The inspected `PetScript` does not expose an owner-aware **pre-learn veto**. A separately exposed `ServerScript::CanPacketSend` can veto outgoing **entire packets**, not the actual underlying pet spell grant. A global `AllSpellScript::OnSpellCheckCast` can reject active casts but doesn't stop spell ownership, passive aura application or action-bar insertion.

**Do not claim that packet suppression, cast denial, or delayed rank removal equals the strict old Vanilla pet training system.** The exact deployed IP/source, original DBC and staging results remain required for implementation approval.

## Public source table and stable file identity

| File | Reviewed public file blob SHA | Critical source behaviour |
|---|---|---|
| [Pet.cpp](https://github.com/azerothcore/azerothcore-wotlk/blob/master/src/server/game/Entities/Pet/Pet.cpp) | `c58f4cc10364cafe68e994bf4f13946630fee611` | `addSpell` lines 1766–1900, `learnSpell` 1903–1917, `InitLevelupSpellsForLevel` 1920–1957, `LearnPetPassives` 2339–2358, `_LoadSpells` 1559–1570, `_SaveSpells` 1572 onward |
| [CharacterDatabase.cpp](https://github.com/azerothcore/azerothcore-wotlk/blob/master/src/server/database/Database/Implementation/CharacterDatabase.cpp) | `e4dffef486e273308599723275640c73514e851f` | Prepared statement around line 383: `DELETE FROM pet_spell WHERE spell = ?`; later `CHAR_DEL_PET_SPELLS` is scoped by `guid = ?` |
| [ServerScript.h](https://github.com/azerothcore/azerothcore-wotlk/blob/master/src/server/game/Scripting/ScriptDefines/ServerScript.h) | `525ab38a8067ee824260d41372d098600683247c` | `CanPacketSend(WorldSession*, WorldPacket const&)` returns bool, line 65; `OnPacketSent` is observer-only, line 85 |
| [ServerScript.cpp](https://github.com/azerothcore/azerothcore-wotlk/blob/master/src/server/game/Scripting/ScriptDefines/ServerScript.cpp) | `30b7156039da386fe0bfc20031ce096c27346f78` | `ScriptMgr::CanPacketSend` dispatch, line 47; `OnPacketSent` observer dispatch, line 65 |
| [WorldSession.cpp](https://github.com/azerothcore/azerothcore-wotlk/blob/master/src/server/game/Server/WorldSession.cpp) | `fc96398079b50984ed0b82183d713c666af6939b` | `SendPacket`: `OnPacketSent` at 310, socket check, then `CanPacketSend` at 351, then socket sends at 356 |
| [AllSpellScript.h](https://github.com/azerothcore/azerothcore-wotlk/blob/master/src/server/game/Scripting/ScriptDefines/AllSpellScript.h) | `b9f2fa1f6bb2a49377617a193bef1580223fd247` | Global `OnSpellCheckCast` may change cast result, not `Pet::learnSpell` permission |

**Line numbers reference this public-source snapshot**, not a pinned compiled worldserver. A changed running commit requires the audit to be repeated.

## Finding S-01 — auto-granted pet spells mutate more than a list

`Pet::addSpell`:

1. Finds spell in core SpellStore; in a *saved-spell loading* path, an invalid/missing ID triggers cleanup (see next finding).
2. Replaces existing lower ranks (or rejects lower candidate ranks). When upgrading, it transfers prior autocast status and calls `unlearnSpell` for the previous rank.
3. Inserts the new spell in `m_spells`.
4. **Immediately casts passive spells on the pet** through `CastSpell(this, spellId, true)`; inserts non-passive spells into pet **action bar**.
5. Alters autocast state and, for pet talent spells, used/free talent points.
6. `Pet::learnSpell` sends `PetLearnedSpell` plus **a refreshed pet spellbook packet** if not loading.

Therefore a later "remove unwanted rank" patch must prove it removes all corresponding auras, actionbar/autocast state, talent bookkeeping and packets without harming the existing pet. An active cast veto does **not** undo a passive spell that already applied.

## Finding S-02 — global database cleanup when loading an invalid saved pet spell

`Pet::_LoadSpells` loops saved `pet_spell` rows and calls `addSpell(..., PETSPELL_UNCHANGED)`.

If that spell ID isn't found in SpellStore, `Pet::addSpell` executes the prepared statement named `CHAR_DEL_INVALID_PET_SPELL`. Current public [CharacterDatabase.cpp](https://github.com/azerothcore/azerothcore-wotlk/blob/master/src/server/database/Database/Implementation/CharacterDatabase.cpp) defines it as:

```sql
DELETE FROM pet_spell WHERE spell = ?
```

**That statement has no per-pet `guid` constraint.** One invalid saved pet spell ID may therefore remove that spell record from **every** pet using that ID. The separate normal `CHAR_DEL_PET_SPELLS` uses `WHERE guid = ?`.

This is **a confirmed source-code behaviour, not an observed incident on the user's realm**. It means:
- Never directly insert guessed historic spell IDs into *any* native `pet_spell` row or publish a migration that populates them from unverified 1.12 references.
- Check actual 3.3.5 server/client DBC and native SpellStore ID **before** introducing any new pet spell.
- Keep original ability source/history/TP knowledge in **module-owned, versioned data**, not by injecting nonexistent native spells to `pet_spell`.
- Use a backup of **characters** DB and a staging replica before ever testing a new spell mapping; test rollback integrity and verify no cross-pet deletion.
- A harmless read-only SQL query is safe, but logging into a **staging realm that loads an invalid saved spell** can itself exercise the native cleanup path. Do not copy risky invalid test data to production.

## Finding S-03 — outbound server packet interception exists, but is not spell-grant interception

Public `ServerScript` exposes:

- `OnPacketSent(WorldSession*, WorldPacket const&)`: **observer**, invoked in `WorldSession::SendPacket` **before** the socket check and before packet send veto. Its name must not be interpreted as proof that the client received the packet.
- `CanPacketSend(WorldSession*, WorldPacket const&)`: **boolean veto** of the whole packet; cannot surgically remove one spell using the documented `const WorldPacket&` interface.
- `WorldSession::SendPacket` currently invokes `OnPacketSent`, checks for a socket, invokes `CanPacketSend`, then sends to the socket.

**Potential application on approved disposable staging:** logging-only `OnPacketSent` can record opcode, approximate packet sequence and number of `PetLearnedSpell`/full spellbar refresh packets, with no mutation and without publishing private payloads.

**Rejected workaround:** simply returning `false` for full pet spellbook updates. This may suppress **legitimate learned spells, bar bindings and client refresh**, not remove the native server spell, and cannot guarantee consistency across new tame, async load, level-up, pet talents or bots. Creating/replacing raw client packets is *not* a supported simple solution, is not tested, and could violate the module-only client requirement. Keep `CanPacketSend` strictly **allow-all** for initial logging pilots.

## Finding S-04 — family passives regenerate independently and aren't saved as native pet_spell rows

`Pet::LearnPetPassives` takes the creature template family, looks up `sPetFamilySpellsStore` and calls:

```text
addSpell(spellId, ACT_DECIDE, PETSPELL_NEW, PETSPELL_FAMILY)
```

`Pet::_SaveSpells` explicitly **skips `PETSPELL_FAMILY`** entries. Consequently removing one family passive from the native spellbook and saving the pet is **not enough**; it may be regenerated on loading by the core. The old Vanilla Great Stamina/Natural Armor/resistance passive effects also need their own independent data and effect verification, and must not be confused with 3.3.5 native pet family passives.

## Finding S-05 — removal also has a rank-chain trap

A closer public-source check of `Pet::removeSpell(spell_id, learn_prev, clear_ab)` shows:
- It removes auras, may adjust native WotLK talent points, and optionally removes the action-bar spell.
- If **`learn_prev=true`**, it can call `sSpellMgr->GetPrevSpellInChain(spell_id)` and **learn the previous rank automatically**.
- `Pet::InitLevelupSpellsForLevel` uses `unlearnSpell(spellId, true)` when level would be below a spell's level, which can **recreate a lower, unearned rank**.
- For a custom Vanilla rules reconciliation experiment, setting an unlearn option indiscriminately could create exactly the lower rank the owner did not teach. Every chosen `learn_prev`/`clear_ab` combination needs a real disposable test and explicit family/source legality check.

This does not mean every pet rank removal necessarily resets talents; the native talent-cost branch only applies to actual talent spells.

## Finding S-06 — native pet-spell persistence is destructive by design

At save, `Pet::_SaveSpells` clears all rows `WHERE guid = ?` (pet number) and rewrites current non-family, non-removed `m_spells` records. It explicitly comments that **pet numbers are recycled**, requiring full rewrite to prevent one new pet inheriting old spells.

**Ledger consequence:** merely using a native PetNumber as a permanent independent unique identity is insufficient without owner identity, a well-defined generation/source identity and recycle handling. Otherwise a custom historical rank ledger might attach old training points/loyalty to a different pet. No separate module ledger table/schema has been approved yet.

## Check matrix for later owner-approved staging

| Test ID | Action | Must prove |
|---|---|---|
| WP03-P01 | New tame with old Vanilla source, log native grants | Event order, passive aura immediately applied, actionbar state and per-era owner |
| WP03-P02 | Existing pet async load, log outgoing packets | `OnPacketSent` log is clearly distinguished from actual socket receipt; expected client spellbar emitted before `m_loading=false` |
| WP03-P03 | Pet XP level-up, log `Pet::learnSpell` packets | A potential wrong rank causes packets; no assumption that subsequent reconciliation prevents initial UI state |
| WP03-P04 | Nonmatching default/family passive | Detect recreated family passive and whether effect persists across stable/logon; cannot be prevented by active cast guard |
| WP03-P05 | Source ID missing from real server SpellStore | **Do not create invalid native DB rows**; instead use read-only DBC and schema comparison, confirm presence/absence without triggering cleanup |
| WP03-P06 | Native PetNumber recycling / pet replacement | Module-owned historical ledger never belongs to a different pet or owner; preserves stable pets and actionbar |
| WP03-P07 | Passive rank vs active cast gate | No claim of strict restored Vanilla if a forbidden passive aura already applied |
| WP03-P08 | Packet veto thought experiment, no actual dropping | Document why whole-packet denial causes inconsistent client state; explicitly prohibit during initial logging proof |

## Safety decision

**Status: strict authentic Beast Training still blocked pending supported pre-learn / before-native-client-state extension point in exact compiled core.** Modern Individual Progression solves era identification and provides owner-aware effects but **does not create that missing Pet::learnSpell interception**.

Recommended near-term research: compare exact installed source headers and DBC, identify all native WotLK auto ranks and passive families relevant to the 111-row catalogue, then design a **logging-only** module proposal for the owner to evaluate later. No current permission to implement it.
