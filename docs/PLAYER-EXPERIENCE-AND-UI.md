# Player experience, accessibility and interface blueprint

**Status: exploration, not a decision to create an addon or replace the 3.3.5 executable.**

## Classic user journey and client constraints

Original Vanilla taught Hunter pet management in the **two-part level-10 taming quest**. The Hunter gained Tame Beast, Feed Pet and Beast Training after completing the appropriate chain. Beast Training lived in the General spellbook tab and was backed by pre-3.0 mechanics. WotLK removed that system and substituted pet talents/automatic skills.

[Classic Petopia training](https://www.wow-petopia.com/classic/training.php) documents original workflow; [WotLK differences](https://www.wow-petopia.com/classic_lk/differences.php) documents changed stable/pet systems. A 3.3.5 client **cannot be assumed to render old UI** just because server-side state exists.

## Minimal safe rollout UI (proposed)

Start with a **pet trainer NPC gossip menu**, keeping the normal 3.3.5 client binary:

```text
Beast Training
  — Current pet: [name, family, level]
  — Loyalty: 4 / 6 (Dependable)
  — Happiness: Happy
  — Training Points: 84 available / 180 total
  — Known abilities
      [Claw rank 7] — Already trained
      [Bite rank 8] — Requires level 56
      [Great Stamina rank 7] — Train (100 TP)
      [Lightning Breath rank 6] — Wrong pet family
  — View how to learn new ranks
  — Untrain this pet (confirm & cost)
```

These figures are **mock interface placeholders**, not a suggested balance/test dataset; the menu must always query server-side authority. Status label must distinguish **Hunter knows rank** from **current pet has rank**. Disabled choices should state why they cannot be trained and must not trigger mutations.

## Accessibility and input-effort guidelines

- One clear interaction at a time, concise labels and an obvious Back/Cancel action.
- Paginate the catalogue and group by Active, Passive, Unavailable and Already Trained.
- Do not force repeated typing of spell IDs/ranks. Select from a menu; optionally supply a command for power users.
- Display available points, cost and net difference **before** committing. Confirm irreversible untraining/abandonment.
- Clear success/error messages: wrong family, level too low, unknown rank, too few points, wrong IP, stale pet.
- No overlay obscuring normal client action bar or pet commands; no bogus spell buttons that fail when clicked.
- GM read-only diagnostic mode should print a short summary plus detailed optional output, not a giant chat flood.

## Optional dedicated Naxx-themed addon — later

If the gameplay prototype succeeds, assess a companion **N Hunter Pets** addon for 3.3.5a. It may provide a Vanilla-styled Beast Training catalogue, rare-beast ability reference, pet TP tracker and source hints, using **read-only** lookup except for validated server-owned training requests.

Addon design cannot substitute for server-side checks or create real pet skills. Verify available 3.3.5 WoW API functions, protected actions and addon↔server communication before approval. It should integrate with existing Naxx addons without taint, duplicate hotkeys or frame overlap.

## Original stable capacity vs current 3.3.5

Vanilla: **one active + two stable**; WotLK: **one active + four stable** according to Petopia. Restoring exact slot count is **not yet approved** because it could make existing pets inaccessible. Consider an optional *Vanilla-style soft limit* that doesn't delete data, but only after proving client interaction, migration/rollback and owner choice. Initial safe default: **do not alter existing stable capacity**.

## Beast Lore and research assistance

Original Beast Lore could identify learnable wild skills. Investigate if 3.3.5 still exposes a suitable spell/tooltip; if not, a future addon can display vetted ability-rank source information without changing taming mechanics. It must respect IP stage access; showing where to tame is fine, granting locked skills is not.

## Human and bot UI separation

Human players use normal tame, trainer and optional addon menus. Bots can require deterministic legal initialization/training, but no UI-only shortcut should become a public player bypass. Future admin "prepare bot for raid" tooling must be behind authorization and clearly record what is granted.

## Acceptance

- From fresh Hunter level 10, no external documentation is needed to discover how to learn/feed/train.
- Player can see available and missing ability ranks for both Hunter and current pet.
- A new eligible rank uses server-side rules; UI never suggests a valid training action that server cannot safely perform.
- UI remains understandable at 1024×768 and common 3.3.5 font/scaling conditions.
- Can train/decline/inspect via accessible clicks; no high-frequency macro or chat typing required.
- No destructive action is available without explicit confirmation and backup/recovery policy.
