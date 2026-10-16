# Authoritative game-state machines — behavioural blueprint

**Status:** Proposed transitions. These are testable specifications, not literal AzerothCore function names or accepted implementation pseudocode. A developer must prove every event is observable using **module-only** hooks before coding.

## 1. Vocabulary: three distinct forms of "knowing" a skill

- `WILD_SOURCE_SKILL`: ability/rank possessed natively by an eligible **tamed source pet** on acquisition. Does not imply the Hunter or another pet learned it.
- `HUNTER_KNOWN_SKILL`: persistent player-character knowledge of a learnable ability/rank acquired from a pet's **successful skill use** or purchased from a pet trainer. The source pet need not remain owned.
- `PET_TRAINED_SKILL`: rank actually enabled/purchased on **one individual pet**, consuming available TP and an active slot where applicable.

Never conflate native initial pet skills, transient spell auras, action bar content, WotLK learned-level spells or Hunter knowledge.

## 2. Acquisition state machine

```text
UNOWNED_WILD_BEAST
  ├── tame invalid → remain UNOWNED
  └── valid normal Tame Beast → TAME_PENDING
        ├── interrupted/failed → discard pending source metadata
        └── confirmed new owned Hunter pet → SOURCE_PET_OWNED
              ├── record source NPC entry, family, level, visual, natural ranks
              ├── assign loyalty 1 and authentic tame happiness (verify)
              ├── persist to unique native pet number; no duplicate source row
              └── allow normal stable/dismiss/death lifecycle
```

**Hard technical gate:** if no existing module hook makes source beast + new persistent pet ID reliably observable and stable, do not synthesize identity from nearest creature of same name or location. Read-only audit must locate an exact event or approved idempotent association.

## 3. Discovery state machine

```text
Hunter owns eligible pet with unlearned natural skill/rank
   ↓ pet performs skill
PET_CAST_OBSERVED
   ├── cast failed / cancelled / wrong owner / wrong source → no change
   ├── excluded spell or WotLK synthetic auto-grant → no change
   ├── Hunter already knows this rank → no change
   └── valid successful skill use
         ↓ apply chosen faithful acquisition policy
HUNTER_SKILL_LEARNED
         ↓ write skill catalogue, notify Hunter ONCE
         ↓ retriable event must not duplicate or charge
```

**Vanilla reference:** CMaNGOS Classic `Pet::CheckLearning` uses the pet's teach-spell map and probabilistic learning. Its code presently shows a **10% chance** for an eligible use; this is an **implementation reference** to compare, **not** independent proof of exact 1.12 retail learning odds. See [CMaNGOS pet source](https://github.com/cmangos/mangos-classic/blob/master/src/game/Entities/Pet.cpp). Do not hardcode odds before the owner-approved historical baseline and source validation.

**Required correctness:** distinguish successful effect/cast from `AllSpellScript::OnSpellCast` timing. That general hook alone may not prove a successful execution. Passive spell / triggered spell / rank chain / non-pet source must be excluded.

## 4. Training state machine

```text
REQUEST_TRAIN(owner, nativePetId, ability, rank, requestNonce)
  ├── module off/not eligible/unknown IP era → reject safely
  ├── wrong class/owner/pet not active → reject
  ├── rank not known by Hunter / invalid rank / future-era rank → reject
  ├── pet family disallowed or pet level too low → reject
  ├── new active skill and 4 actives already trained → reject
  ├── not enough TP (unless Growl, which is 0) → reject
  ├── native Pet::learnSpell not safe or can't persist → reject/feature block
  └── atomic mutation:
        old trained rank cost refunded + new rank cost charged
        old rank removed; new rank granted; action bar validated
        pet TP projection and native spell list reconciled
        persistence version increments; respond success only when committed
```

When an old rank is already present, the **net difference** is charged; never count both ranks in the active count. A failed native spell operation must roll back the ledger and not claim success. Account for same-pet repeated requests and asynchronous pet switching.

## 5. Loyalty / happiness state machine

**Pet rank names and advancement:** 1 Rebellious → 2 Unruly → 3 Submissive → 4 Dependable → 5 Faithful → 6 Best Friend.

Historic sources indicate time **and** XP requirements for gain, including pets at owner-level cap; see [Warcraft Wiki loyalty](https://warcraft.wiki.gg/wiki/Loyalty) and [CMaNGOS loyalty implementation](https://github.com/cmangos/mangos-classic/blob/master/src/game/Entities/Pet.cpp).

```text
LOYALTY[n], happiness=HAPPY/CONTENT/UNHAPPY
   ├── feed accepted food → restore happiness via native system
   ├── positive care + elapsed time + required experience → progress loyalty points
   ├── threshold & prerequisites satisfied → LOYALTY[n+1], TP budget increases
   ├── unhappy/neglect → loyalty declines; TP total may fall below spent cost
   └── extreme abandonment threshold → DISABLED in pilot; owner policy required
```

**Research threshold table:** Unruly 5% XP + 0 min; Submissive 5% XP + 30 min; Dependable 5% XP + 45 min; Faithful 5% XP + 60 min; Best Friend 5% XP + 90 min. These are reference values reported by the Wiki and must be tested against selected Vanilla baseline. Clarify whether these are minimum cumulative elapsed durations **per transition**, how XP counts while pet is level-capped, logged out, or stabled, and whether IP changes the owner's XP rates.

**CMaNGOS behavioural comparison, not coding instruction:** CMaNGOS records loyalty points, required XP per rank and a loyalty timer, with loyalty ticks depending on happy/content/unhappy state. Modern AzerothCore records happiness but lacks the old loyalty engine. Different implementations might produce different timing; reproduce observed effects, not blindly transplant code.

**Training-point shortage after loyalty loss:** preserve trained ranks safely and consider historically negative TP; do not silently strip abilities. Actual retail behaviour under loyalty downgrade needs validation.

## 6. Stable / relog / restart lifecycle

```text
ACTIVE ↔ STABLED ↔ ACTIVE
ACTIVE → DISMISSED → ACTIVE
ACTIVE → DEAD → REVIVED
ACTIVE → LOGOUT → LOGIN
ACTIVE → WORLD_SERVER_STOP → WORLD_SERVER_START
```

At each boundary, verify: **native pet number**, original tame entry, loyalty/points, happiness, training rank set, Hunter knowledge, cooldowns, pet level, ability autocast state and actual attack speed. If the ledger/native server pet data disagree, fail safe, log precisely and stop destructive reconciliation until a documented recovery policy is available.

## 7. IP era state machine

```text
VANILLA_STAGE_0 ... VANILLA_FINAL_STAGE
          ↕ normal IP stage progression within Vanilla
    VANILLA_PET_RULES (unchanged)
          ↓ authoritative IP TBC entry milestone
       POST_VANILLA_POLICY_PENDING
```

Must not automatically choose WotLK pet talents, automatically erase Vanilla loyalty, or keep force-removing spells without an approved TBC migration. Until decided, mixed-era deployment should remain disabled for affected characters or safely retain legacy state under a documented coexistence contract.

## 8. Human/Alt Bot/Random Bot

For a pet event, resolve **actual owner class, identity, current bot type and progression**. If owner is not eligible or bot rollout isn't enabled, ignore without mutation. A Random Bot that is subsequently deleted must not become the identity for a human-owned pet. Alt Bot learned skills must persist and may not alter their equipment/talents via this module.

## 9. Concurrency / idempotence cases

- Two simultaneous train clicks; stale UI after swap; logout during database write; server restart between ledger and native pet save; pet rank already known; repeat cast event.
- Stable move while XP/happiness tick fires; owner dies while tame pending; two beasts with identical names; pet level changes mid-train.
- Unknown or removed pet IDs, owner deleted/recreated, incorrect IP era cache, dual-specialization switches, WotLK auto-teach after training.
- All need deterministic single outcome and no net irreversible loss. Reject stale requests rather than silently reassign to a different pet.
