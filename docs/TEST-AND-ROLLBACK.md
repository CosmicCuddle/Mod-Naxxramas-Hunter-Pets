# Test, backup, acceptance and rollback plan

**Planning artifact only. Nothing here has been executed.**

## Before any future implementation or trial

1. Record exact revision/commit and installation paths for worldserver, AzerothCore, IP, Playerbots and Naxx Core. Do not assume main-branch GitHub equals production.
2. Take timestamped, **verified restore-capable** backups of:
   - `acore_characters` or corresponding characters database (including native pet owner/pet/skill, inventory and character settings tables).
   - Any affected `acore_world` rows and any module-owned SQL tables (before migration).
   - Active module configs, `worldserver.conf`, module source, worldserver binary/known-working build and any changed server/client DBC/MPQ assets.
3. Preserve hashes, migration IDs and existing pet counts; test restore on a **non-production copy** before feature rollout.
4. Use a **disposable Hunter and test beast** initially, on a staging realm/test clone, not valued players' pets.
5. Set all custom toggles **off by default**. No SQL write or DBC change is part of the documentation phase.

## Functional matrix

| Area | Expected outcome |
|---|---|
| Taming | Only legal tameable creatures; original pet source entry and appearance retained; no free blanket rarity stat bonuses. |
| Original attack speed | Exact approved speed for tested beast after tame, level-up, resummon, relog, restart; control pet unchanged. |
| Damage normalization | Faster pet hits faster but each hit has appropriate damage; no unexpected general DPS multiplier. |
| Discover ability | Hunter only learns rank after eligible tamed pet actually uses it; cannot gain unknown or future-expansion rank. |
| Teach ability | Correct family/level/TP; incompatible family or insufficient TP causes no change or charge. |
| Ranks | Higher-rank replacement accounts for refund; no duplicated effects or outdated rank left on bar. |
| Active cap | Fifth active ability refused; passive abilities allowed within TP and family rules. |
| Growl | Zero TP cost; no negative TP debit; autocast persists. |
| Training points | Level 60 + Loyalty 6 = 300 total TP, reduced only by net trained costs; no overflow/underflow. |
| Loyalty | Six ranks progress correctly and persist; unsafe abandon/runaway disabled in pilot. |
| Happiness | Existing feed/food acceptance and modifiers work; no double damage multipliers. |
| Spell grant bypass | WotLK automatic spells **do not reappear** after pet level, stable, load or revive under Vanilla rules. |
| Trainer | Purchases, untraining/respec, insufficient currency and rollback atomicity. |
| Pet XP | Classic tame-level, pet XP and owner-level cap; IP HunterPetLevelFix not double-applied. |
| Family roles | Verify HP/armor/damage, focus, food and special skills by all family categories and sample named mobs. |
| Client | Action bar, tooltips, autocast, known skills, available TP, pet talent UI and addon/gossip remain intelligible on 3.3.5a. |
| Disabled config | Unchanged AzerothCore behaviour; no residual ability stripping/data mutation. |

## Persistence / destructive edge cases

- Pet stable A→B→A; relog; pet dismissed; pet dead/revived; worldserver restarted; server crash simulation in staging.
- Tamable beast source entry lost? Verify source identity after pet load, not merely for just-tamed pets.
- Existing Hunter with 3.3.5 talents, many stable pets and future-era spells; migrates **without loss**, or migration safely refuses and leaves player untouched.
- Abandoning a *disposable* pet and deleting a *disposable* test Hunter cleans module-owned ledger correctly; no orphan rows.
- Failed database transaction, malformed skill records, unknown creature, low TP, wrong family, duplicate rank, concurrent teach attempts; no desynchronization.
- Version upgrade/reinstall/disabled config and migration backward compatibility.
- Only enable irreversible pet abandonment once historical policy, warnings, opt-out and recovery have owner approval.

## Individual Progression tests

- New Vanilla Hunter; Vanilla raid-tier Hunter at level 60; alternate character at same level but TBC stage; TBC/WotLK Hunter as control.
- Two Hunters of different IP stages in same party/raid and instance; verify no cross-character leakage.
- Individual Progression on/off (if supported by deployment) and unavailable API fallback: module must fail safely rather than guess by level.
- Test Zul'Gurub creature access consistent with IP settings; pet ability source cannot bypass locked instance.
- Verify bot's own IP era and account-level exceptions independently; do not infer bot era from the player issuing a command.

## Playerbot and Naxx Core regression tests

- Human, Random Playerbot, and persistent Alt Bot separately.
- Hunter bot has correct legal rank access and correct combat decision/target, without wrong-rank auto-teach.
- Bot pet persists after group disband, logout, server restart and role/talent changes.
- **No bot talent respec, gear refresh or equipment swaps caused by this module.**
- Existing `.bot consumables mc` Hunter pet interactions remain safe; current Naxx Core Playerbot talent and NT1 integration unaffected.
- Non-Hunter controlled pets: Warlock demons, DK ghoul, Mage elementals are **identical** to baseline.

## Acceptance and proof

For every test: record realm environment, SHA, config, tester, character/pet, initial state, actions, expected result, actual result, screenshot/log/SQL read-only evidence, pass/fail, issue reference. Automated unit/integration checks supplement—but never replace—3.3.5 in-game pet-stable and packet/UI testing.

Sign-off gates:
- Read-only research complete.
- Feasibility spikes pass with **no upstream changes**.
- All Vanilla features accepted (or gaps explicitly approved as partial scope).
- No data loss, verified restore, no regression, production feature flags documented.
- Owner explicitly approves staged deployment and later integration.

## Rollback sequence — to be refined against implemented schema

1. Disable experimental module config **before** writing further pet-state changes; stop/restart worldserver in a controlled window.
2. Restore previous tested module build/config. Do **not** blindly delete module tables if native pets depend on their data.
3. If migrations touched native pet data, restore from pre-test characters DB backup on a **staging clone**, inspect affected characters, then decide production recovery. Generic database restore would overwrite unrelated player progress; only execute with owner-approved downtime and impact plan.
4. Roll back owned SQL with verified reversible migration (or snapshot restore) and restore client/server DBC together if modified.
5. Inspect pet inventory, stable, spell list, levels and Hunter progression after recovery. Log findings and block redeployment until resolved.

**Never use a Git revert as the only rollback for pet systems; persistent character state and client/server data also matter.**
