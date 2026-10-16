# WP-03 — disposable staging proof protocol for manual pet training

**Status: proposed experiment only, NOT executed and NOT approved for coding/deployment.**  
**Strict boundary:** all new implementation, if separately approved, inside the standalone Hunter Pets module. No AzerothCore/IP/Playerbots/Naxx Core source changes. No global world/character SQL or client DBC patches.

This experiment is intended to answer one narrow engineering question: **Can original Vanilla training avoid automatic WotLK Hunter pet skills, preserve truthful client-visible state, and survive pet lifecycle events using only public AzerothCore module interfaces?** A partly successful test must not be called 'Vanilla Beast Training restored.'

## Preconditions and protective measures

1. Pin installed **exact compiled** AzerothCore, IP, Playerbots and Naxx Core commits, current custom modules and client 3.3.5 build. Compare their actual hook signatures with [public-source analysis](AUTO-SPELL-LEARNING-FEASIBILITY-DECISION.md).
2. Work on separate **staging** instance with **disposable Hunter/pet/Alt Bot** data. Before any modification, preserve tested restore points for **characters DB**, relevant **world DB** and working server configuration; keep copies offline or outside the modified tree. No reason to change a live realm during research.
3. Confirm the intended Vanilla IP stage with **authoritative IP API**. Do not guess from level. Confirm disabling the module returns complete native baseline.
4. Collect spell IDs and family rows from actual server/client DBC; no guessed IDs from classic emulator bots and no copyrighted DBC files in public GitHub.
5. Start with **logging-only** hooks. Grant/remove/deny no spells until timing and default state are reproduced. Production characters and valuable stable pets are excluded.
6. The owner's explicit approval is required for even an additive C++ proof-of-concept on staging; documentation does not grant approval.

## Experiment 0 — read-only source and build inventory

Report:
```text
AzerothCore commit + build time:
IP module SHA and era accessor:
Playerbots SHA + enabled mode:
Naxx Core SHA and related pet hooks:
Hunter Pets module state (absent / docs / experimental):
3.3.5 server DBC hashes:
3.3.5 client DBC hashes:
characters/world schema revision:
PetScript/AllSpellScript/PlayerScript available signatures:
backup/staging restore check:
```

**Exit:** all exact compiled headers match the planned module interfaces, or update the blueprint before attempting a code spike.

## Experiment 1 — pet spell arrival timeline, no mutations

Disposable cases:
- Level-20 Hunter tames level-20 beast; owner in first Vanilla IP stage.
- Level-60 Hunter tames level-20 beast; owner in last Vanilla IP stage.
- Newly tamed Hunter pet is stabled, reloaded, dismissed, resummoned, logged out and server-restarted.
- The same pet gains a level, is revived and changes area/instance.
- Level-70+ post-Vanilla Hunter (control) and Warlock pet (negative control).
- Persistent Alt Bot Hunter and Random Bot Hunter (only if staging controls can prevent high-impact bot provisioning).

Record timestamped **event ID**, owner persistent character ID (anonymized), native PetNumber, source creature entry, pet level, module era verdict, `isBeingLoaded()`, `HasSpell()` snapshots, Pet action bar state, native known ranks and the **client's received learned-spell/spellbook/talent packets**. No raw player account IDs or passwords in the public GitHub report.

**Critical order assertions:**
- New tame: `InitPetCreateSpells` and WotLK grant before `OnPetAddToWorld`, then saved after `SetMinion`.
- Reload: `OnPetAddToWorld` before async `_LoadSpells` and auto-rank grants; `PetSpellInitialize` before `m_loading=false`.
- Level-up: direct `learnSpell` can send client packets immediately if loading false.
- Original low wild level is lost by owner−5 catchup before AddToWorld unless proven alternative exists.

**Stop** if timing differs on installed revision, custom bot code bypasses source path, or any mutation occurs in 'logging-only' test.

## Experiment 2 — active-skill permission candidate

Only if the owner approves a constrained module spike *after* Experiment 1:
- Use global `AllSpellScript::OnSpellCheckCast`, scoped to an actual **owned Hunter pet** under a **Vanilla IP owner**. Do not intercept wild beasts, Warlock/DK/minions, post-Vanilla pets or unrelated spells.
- For an ability legally possessed **naturally by this source pet** or trained through owner knowledge/TP, permit normal check.
- For an automatically granted **but unearned** rank, return a controlled failed-cast result **before** effect/cooldown (spell error messaging still needs client verification).
- Try manual action bar cast, autocast, triggered casts, Playerbot AI use, combat restart and race during async load. Measure any cases where an aura or effect is applied despite failure.
- Examine the exact callsite order in `Spell::CheckCast`, and prove the same handler is invoked from all applicable spell paths; do **not** infer from a normal cast succeeding once.

**Fail criteria:** unauthorized ability reaches target, pet gains passive benefit, legal unrelated cast denied, performance spikes or invalid spell packet. A passing active-cast test is **not** a full spellbook/auto-learning GO.

## Experiment 3 — after-load reconciliation candidate

Only after Experiment 2 is shown safe and owner permits further staging:
- Use `Pet::isBeingLoaded()` **read-only** and an owner `PlayerScript::OnPlayerAfterUpdate` observer to measure first moment eligible for reconciliation, without edits.
- Compare **client-visible packets and action bar BEFORE this moment**. The public core sends the spellbook before clearing the flag, so a temporary illegal spell display is expected unless installed branch provides another supported interception.
- If exploring an owner-approved reduced-fidelity cleanup, operate **once per pet load generation** (not every tick), preserve all originally legitimate natural/learned ranks and non-Vanilla pets, clean both action bar and passive states and confirm stable/relog persistence.
- Stop if cleanup reintroduces old ranks via `learn_prev`, changes native family scaling, loses existing active/autocast spells or reveals abilities through client UI for a duration the owner has not accepted.

**Fail criteria:** any durable wrong rank, passive aura benefit, client event inconsistency, repeated cleanup storm, or loss of existing pet skill. **Do not** hide any failure by mass resetting the pet.

## Experiment 4 — native persistence/failure injection

- Use **disposable** pet and staging database, snapshot before every test and rehearse restore.
- Induce controlled termination (under test harness only) **between native spell transaction, main pet transaction and module ledger commit**. Verify no loss of PetNumber, source entry, stable slot, pet name, active bars, learned rank or TP calculation.
- Test interrupted logout, server restart, duplicate event callbacks, rapid stable swaps and repeated packet retransmission.
- An apparent fix that requires nonmodule SQL changes or a background cleanup service touching `character_pet` directly is not an approved solution.

## Experiment 5 — per-era policy and rollout simulation

| Actor | Owner stage | Expected default if module is OFF | Expected under experimental module |
|---|---|---|---|
| Human Hunter | earliest Vanilla | Native WotLK behaviour | Restore approved Vanilla rules or fail closed; no unearned rank use |
| Human Hunter | final Vanilla | Native WotLK behaviour | **Same Vanilla rules**, irrespective of raid/content tier |
| Human Hunter | post-Vanilla | Native WotLK behaviour | **Untouched** unless separate later-era policy approved |
| Alt Bot Hunter | Vanilla | Native Playerbots/IP/Naxx behavior | Same skill legality with no gear or talent refresh and correct persistent pet |
| Random Bot Hunter | Vanilla | Native Playerbots/IP/Naxx behavior | Do not invent Hunter skill knowledge, test provisioning separately |
| Non-Hunter | any | Native behaviour | No changes |

## GO/NO-GO decision report (mandatory)

```text
WP-03 Status:
Exact tested commit/build/DBC:
Staging backup and restore proven:
Logging-only event order:
Native WotLK auto-learn paths found:
Per-owner pre-learn veto exists in deployed build?:
Cast denial prevents *every* illegal active effect?:
Client ever displays automatic unearned spells?:
Passive/stat effects ever leak?:
OnLoad timing and no stale action-bar entries?:
Native/module transaction crash recovery:
IP human/Alt/Random and post-Vanilla controls:
Pass/fail/blocked test IDs:
Outcome: STRICT_GO | REDUCED_FIDELITY_CANDIDATE | NO_GO | INSUFFICIENT_EVIDENCE
Remaining compromises requiring owner approval:
Files / SQL / client assets proposed to modify:
```

**Gate meaning:**
- `STRICT_GO`: original ability eligibility, rank *existence in pet/client*, passives, action bar, save and cross-era behavior all proven true, with no upstream patch.
- `REDUCED_FIDELITY_CANDIDATE`: active skill permission might be enforced, but temporary learned ranks/client packets or other nonoriginal behavior remain; cannot call it 'full vanilla' or install without a *new explicit owner decision*.
- `NO_GO`: no module-only path meets acceptable state/permission/persistence requirements.
- `INSUFFICIENT_EVIDENCE`: installed revisions or staging tests lacking; do not declare GO.

## Research source index

- [Auto-learning decision and 16-way alternative matrix](AUTO-SPELL-LEARNING-FEASIBILITY-DECISION.md)
- [16 interception pathway CSV](../data/reference/auto-spell-interception-candidates.csv)
- [Tame/loading native lifecycle](SOURCE-PINNED-PET-LIFECYCLE-AUDIT.md)
- [Native staging test and rollback principles](TEST-AND-ROLLBACK.md)
- [IP/bot coexistence contract](IP-PLAYERBOTS-INTEROPERABILITY.md)

**Zero staging experiments have been executed or signed off by the owner.**
