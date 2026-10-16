# Read-only installed Individual Progression inventory (MobaXterm / Linux)

**Purpose:** identify **which revision of IP and AzerothCore this server actually builds**, without copying private data, changing configuration or touching Hunter pets. This is a *future owner-guided diagnostic*, NOT an executed audit. No request to run it immediately.

**Safety:** commands below are **read-only**. They do not require an AzerothCore recompilation, `git pull`, database access, Worldserver restart or pet login. Do **not** post passwords, environment variables, account tokens, full config files, character data or world database dumps. Back up everything before **future mutating tests**, but backups are not required merely for these queries.

## Step A — locate the module first

Open the server's normal MobaXterm SSH terminal. The project's previously mentioned root is `/root/cosmic_core`, but paths can differ; if it doesn't exist, stop and confirm the correct path rather than guessing.

```bash
# Display discovered IP-related module directories; no modifications.
find /root/cosmic_core/modules -mindepth 1 -maxdepth 3 -type d -iname '*individual*progress*' -print
```

**Do not** assume a similarly named duplicate/backup directory is the actual compiled module. Compare its root against the current server build configuration before acting on it.

## Step B — record revisions without updating anything

After identifying the exact IP source directory, replace the example `/root/cosmic_core/modules/mod-individual-progression` below **only if it matches Step A**:

```bash
# Example only: adjust path to exact module from Step A.
CORE=/root/cosmic_core
IP=/root/cosmic_core/modules/mod-individual-progression

# Read-only Git revision checks (works with submodules/worktrees too).
# If either fails, STOP here and check the directory; do not 'git pull'.
git -C "$CORE" rev-parse --verify HEAD
git -C "$IP" rev-parse --verify HEAD

# Continue only after the commands above show valid commit hashes:
git -C "$CORE" branch --show-current
git -C "$IP" branch --show-current
git -C "$IP" status --short
```

The IP directory might be worktree/submodule without a `.git` **directory** (it may have a `.git` file). `git -C ... rev-parse` is the authoritative check; don't interpret a missing `.git` directory as automatically missing a valid worktree. Also, the source worktree's current HEAD might be **newer than the compiled binary**—capture the build/restart history separately. Do **not** share raw `git remote -v` if it contains embedded credentials.

## Step C — confirm public header and pet hooks by exact symbol

```bash
# Read-only C++ source discovery. These may return no matches.
grep -R -n -E \
'GetPlayerProgressionFromQuests|hasPassedProgression|PROGRESSION_PRE_TBC|HunterPetLevelFix|OnPlayerGiveXP|SyncBotsProgressionToLeader|OnPetAddToWorld|CanLearnPetSpell|InitLevelupSpellsForLevel' \
"$IP/src" --include='*.cpp' --include='*.h'
```

Read the results **before** interpreting:
- `hasPassedProgression` and `GetPlayerProgressionFromQuests` indicate a modern IP-style API; actual semantics depend on the installed implementation.
- `HunterPetLevelFix` being present in **a comment or config** is not proof that any live callback applies the fix. Original issue [#423](https://github.com/ZhengPeiRu21/mod-individual-progression/issues/423) reports the feature did not work in an older environment.
- `OnPlayerGiveXP` identifies existing pet XP handoff, not a general override of WotLK pet spell learning.
- Finding no source match is not proof a third-party patch or compiled binary lacks equivalent behaviour; record it as unknown.

## Step D — check stage settings narrowly (no full config dump)

After identifying the **actual active** IP configuration file, use selective key lookup (never post the entire file). The possible keys are:

```text
IndividualProgression.Enable
IndividualProgression.ProgressionLimit
IndividualProgression.StartingProgression
IndividualProgression.BotAccountsMaxLevel
IndividualProgression.EnableAllSpellRanks
IndividualProgression.BotOnlyAdjustments
IndividualProgression.HunterPetLevelFix   (legacy only; not assumed supported)
```

For modern IP, stage **0–7** maps to Vanilla, **8–12** TBC, **13–18** WotLK. For older IP, TBC can begin at **7**. A configured global progression cap can change effective behaviour even if the player has a rewarded TBC milestone. Bot-account matching and group sync need separate handling. Don't share `BotAccountsRegex`, account names or other account metadata unless specifically required and redacted.

## Step E — compare with the existing Naxx Core stage check

Our related Naxx Core currently has:

- [`src/Systems/MeetingStoneProgression.cpp`](https://github.com/CosmicCuddle/Mod-Naxxramas-Core/blob/main/src/Systems/MeetingStoneProgression.cpp) — modern/legacy public-header check; safe fallback caveat.
- [`src/Systems/BotTalentImport.cpp`](https://github.com/CosmicCuddle/Mod-Naxxramas-Core/blob/main/src/Systems/BotTalentImport.cpp) — modern human-issuer stage check for NT1, **not bot-pet owner stage**.
- [`src/Systems/BattlemasterQueueProgression.cpp`](https://github.com/CosmicCuddle/Mod-Naxxramas-Core/blob/main/src/Systems/BattlemasterQueueProgression.cpp) — BG bot exemptions, **not pet training exemptions**.

These are **source-based design references** and must not be silently recompiled or modified while diagnosing Hunter Pets.

## Step F — optional in-game progression cross-check (read-only)

On a character with sufficient command permissions, `.ip get` can display the current **raw recorded progression** without mutating quest data. The modern IP [command implementation](https://github.com/ZhengPeiRu21/mod-individual-progression/blob/master/src/cs_individualProgression.cpp) (public blob `04c15a3dae8b49ebc314e09522d7941ac8258225`, lines 18 and 68–90) currently registers it at **GM security level** and reports `GetPlayerProgressionFromQuests`. The exact command permissions of the compiled realm may differ.

1. With a safe test character, use `.ip get` **only if authorised**; this is a nonmutating query.
2. If you have permission and the target is an approved **disposable** bot, use the documented `.ip get [player]` syntax to inspect that actor's recorded progression.
3. **Do not** use `.ip set`, `.ip setbot` or GM account/character modifications as a substitute for this read-only audit.
4. Record the stage number and whether a global `ProgressionLimit` is active. `.ip get` calls the **raw quest reader**, whereas IP's `hasPassedProgression` additionally respects the configured cap. Those may intentionally disagree; the pet adapter must apply the effective rules consistently.
5. Compare human and bot stages **before and after legitimate group activity**, *only on an approved test instance or with test-only characters*. IP may resynchronise the bot's quests when joining a group; changing group membership can itself affect progression, so do not do it on valued characters just for diagnostics.

The IP [public README](https://github.com/ZhengPeiRu21/mod-individual-progression) still contains some Player Settings wording even though current `GetPlayerProgressionFromQuests` reads hidden rewarded quests. This is another reason the **actual installed revision and exact source** must prevail over generic documentation.

## Step F — exact audit report fields

```text
Worldserver compiled build identifier, if known:
AzerothCore worktree HEAD:
IP source worktree HEAD:
IP working tree clean? (yes/no, do not paste secret diffs):
IP header export supported?:
Modern GetPlayerProgressionFromQuests available?:
Modern hasPassedProgression available?:
Progression limit configured? (can share capped stage number only):
IP enabled?:
Legacy HunterPetLevelFix present AND executable?:
OnPlayerGiveXP pet path present?:
Bot group progression sync present?:
Naxx Core stage adapter version/current build?:
Client server Spell.dbc versions/SHAs, if already known:
Outstanding unknowns:
```

These results tell us how to write an **adapter for the actual build**, rather than guessing at APIs. The worktree HEAD alone does not verify the running binary. No source modification, rebuild or client/DB patch is justified by merely discovering a mismatch.

## What we do after the inventory (not automatically)

1. Compare installed symbol signatures and stage semantics with [IP adapter design](IP-STAGE-ADAPTER-DESIGN.md).
2. Draft a narrow module-owned stage resolver **only after owner-approved implementation**.
3. On *disposable backed-up staging*, test the [22-case owner-stage matrix](../data/reference/ip-pet-owner-era-acceptance-matrix.csv).
4. Keep the independent native **pet auto-learn** gate blocked until the [WP-03 protocol](WP03-AUTO-SPELL-LEARNING-STAGING-PROTOCOL.md) proves a safe strict path or the owner separately approves a compromise.

**Nothing above has been run on the user's server in this documentation session.**
