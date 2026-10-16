# Individual Progression compatibility adapter — architecture and evidence

**Created:** 16 October 2026  
**Status:** DESIGN / RESEARCH ONLY. **Not implemented, compiled or tested.**  
**Owner boundary:** preserve AzerothCore, Individual Progression, Playerbots and Naxx Core upstream source. Any eventual code is within the standalone Hunter Pets module, not the related Naxx Core module. No changes to live DB/client/server.

## Owner-reported version and current target priority (16 October 2026)

The owner confirms their server uses **the newest Individual Progression version**. This is useful project context, but **a version description is not the exact compiled IP/core commit**. Accordingly, the **first development target should be modern hidden-quest IP**: Vanilla stages **0–7**, TBC **8–12**, WotLK **13–18**, with effective eligibility through the installed cap-aware IP API where possible.

This **does not authorise** implementation, install or server changes. Legacy player-settings/IP-stage-7 support is now a **lower-priority compatibility backlog** for the first version, rather than a reason to block continuing modern-first source research. Keep a safe no-mutation fallback if the expected API is absent or the actual build differs. Before compilation, still record exact IP source and compiled worldserver versions.

Bot stage policy remains open because IP may synchronise a bot's hidden milestones on joining a human-led group. WotLK pet auto-learn control also remains **independently unproven**.

## Key discovery: this project's existing Naxx Core integration is a useful proven *source pattern*

Naxx Core already contains **a build-aware adapter for IP's Vanilla/TBC boundary**:

- [MeetingStoneProgression.cpp](https://github.com/CosmicCuddle/Mod-Naxxramas-Core/blob/main/src/Systems/MeetingStoneProgression.cpp) (public blob `b46d0f2b5c16e490878e68d259092ebb4b2fdc7a`, lines **23–97**) uses `__has_include("IndividualProgression.h")` and a compile-time check for modern `GetPlayerProgressionFromQuests(Player*)`. When the header is available, it calls IP's own `hasPassedProgression`. It accommodates **modern TBC entry 8** vs **legacy TBC entry 7**.
- If the public IP header is **not** exported, it checks `IndividualProgression.Enable`, `ProgressionLimit` and modern hidden rewarded quests **66008–66018**. The source itself warns this fallback **cannot read old player-settings-based progression**. Its README marks latest build/in-game testing incomplete. **Do not call this deployed or proven safe in all IP variants.**
- [BotTalentImport.cpp](https://github.com/CosmicCuddle/Mod-Naxxramas-Core/blob/main/src/Systems/BotTalentImport.cpp) (blob `1c56ca15b8925b8b7e38cd059912986c1da5c4fb`, lines **199–260**) scans quest milestones **66001–66018**, applies `ProgressionLimit`, and labels **0–7 Vanilla**, **8–12 TBC**, **13–18 WotLK**. However it expressly uses the **issuing HUMAN** as the authoritative progression, NOT the target bot. That is correct for authorising human-controlled NT1 talent imports, but **not automatically correct** for a persistent Hunter pet owner.
- [BattlemasterQueueProgression.cpp](https://github.com/CosmicCuddle/Mod-Naxxramas-Core/blob/main/src/Systems/BattlemasterQueueProgression.cpp) (blob `ddaa20746ffd6edc55e4548c4ec876c586749140`, lines **27–110**) uses quest milestones and bot account exemptions. Exempting a random bot from BG restriction **does not mean** exempting that bot's pet from legitimate Vanilla rules.

**Reusable lesson:** centralise reliable modern/legacy IP stage interpretation, config disable/cap behaviour and account classification instead of scattering independent `GetLevel()` checks through every pet hook. **Do not copy** source implementations or inherit an unrelated Naxx Core exemption as pet policy. Use these as technical references and implement only after installed headers/ABI are verified.

## Public IP source confirms the authoritative API for the modern branch

- [IP header](https://github.com/ZhengPeiRu21/mod-individual-progression/blob/master/src/IndividualProgression.h), blob `510896357ac883590c6876ac2ec605878c6dfca1`: `ProgressionState` enum at lines **223–244**, `PROGRESSION_PRE_TBC = 8`; `GetPlayerProgressionFromQuests` and `hasPassedProgression` public declarations around **418–422**.
- [IP source](https://github.com/ZhengPeiRu21/mod-individual-progression/blob/master/src/IndividualProgression.cpp), blob `865560587f0ebb202c4d73852f68358a3f398aeb`: lines **15–38** read rewarded milestone quests and `hasPassedProgression` **honours both IP enabled and `progressionLimit`**. A raw quest read may differ from this effective eligibility when a global cap is active.
- [IP bot sync](https://github.com/ZhengPeiRu21/mod-individual-progression/blob/master/src/IndividualProgression.cpp), lines **464–488**: `SyncBotsProgressionToLeader(Group*)` copies the normal group leader's hidden-quest progression to non-normal accounts. [`IndividualProgressionPlayer.cpp`](https://github.com/ZhengPeiRu21/mod-individual-progression/blob/master/src/IndividualProgressionPlayer.cpp), blob `e3c5f1781885a4f2ded5b135d2cf4c96467cd9da`, lines **1360–1374**, triggers this from `GroupScript::OnAddMember`.
- IP detects `isBotAccount` and `isExcludedAccount` using account matching. **The bot's personal current hidden quest stage, party leader's current stage, and the controlling human's NT1 command stage are not always interchangeable.** Actual installed bot policy must be determined before treating any as authoritative.
- [IP player pet XP](https://github.com/ZhengPeiRu21/mod-individual-progression/blob/master/src/IndividualProgressionPlayer.cpp), lines **116–159**, and owner-aware output scaling, lines **1377–1501**, already handle some pet behaviours. Do not duplicate their effects.

## Proposed standalone module API, to validate on deployed build

```text
EvaluateHunterPetContext(pet):
    if config disabled             -> NativeBehaviour
    if pet is absent / not active  -> NoAction
    if not a real Hunter pet       -> NativeBehaviour
    owner = actual Pet::GetOwner()
    if not a valid Player/Hunter   -> NoAction (fail safely)
    if IP is disabled              -> NativeBehaviour
    effectiveStage = ResolveUsingActualInstalledIPApi(owner)
    if cannot verify stage        -> NoMutationAndNoNewClassicTraining
    if effectiveStage < TBC entry -> VanillaPolicy
    otherwise                     -> NativeBehaviour pending later-era decision
```

**Return two pieces of information, not just a boolean:** `ruleSet` and `evidence` (source: live IP API / supported modern quest fallback / unavailable), plus `actorKind` (human, persistent Alt Bot, Random Bot, unknown). Persisting the *current* era as a permanent pet property could strand valid pets when IP group sync or stage changes occur. Pet identity and learned history should persist separately from **effective active policy**.

**Owner classification requires installed API inspection:** Naxx Core NT1 intentionally uses the *command issuer*; our pet engine must use the *pet owner*, and use master/leader only if an explicit bot policy is approved. An unknown bot era means **no dangerous pet mutations or free training**, not silent assumptions or forced level-60 Vanilla based on pet name.

## Five tricky stage scenarios

1. **Vanilla human, IP stage 0–7:** pet uses Vanilla mode across every Vanilla tier, even if its owner is level 60 or the pet's innate level does not match owner.
2. **Modern TBC/WotLK human, IP stage ≥8:** no automatic Vanilla-only pet restriction unless a separate later-era transition is expressly approved. A pet's **Vanilla history** should not be destroyed when policy stops applying.
3. **Global `ProgressionLimit = 7` but player had rewarded quest 66008 before cap:** `GetPlayerProgressionFromQuests` might still return 8, while `hasPassedProgression(...PRE_TBC)` returns false under the cap. Need effective-stage/cap interpretation, not raw quest alone.
4. **Alt Bot versus Random Bot in party:** IP may synchronise a non-normal bot's quests from the normal leader on group join; a bot's stage may change **without a level or zone change**. Assess legal pet abilities on owner-stage change with no pet reset, XP duplication or lost training.
5. **IP public header unavailable on deployed legacy build:** Naxx Core's modern quest fallback explicitly cannot read old player-settings stage. **Do not forcibly impose a Vanilla restriction based on no milestone quests**; require a validated adapter or disable Hunter Pets changes for that actor.

## Module architecture preference

- `IPAdapter` (read-only stage and owner classification) in Hunter Pets: one vetted implementation, no global SQL/DBC edits, no direct dependency on Naxx Core binary symbols.
- `PetIdentity`: native pet number, owning character persistent ID and true source creature entry, independent of current IP stage.
- `PetRules`: strict Vanilla eligibility and family/level/rank reference, remains disabled for all actors until actual core spell interception proven.
- `ProgressionTransitionObserver`: detect a change in actor's effective rule set after group/quest/cap events *without mutating native pet state* in research mode. Must not assume any universal post-load callback.
- `PetXPBridge`: observe only in initial design; let IP own the existing XP cap compensation.
- `SpellPolicy`: don't conflate cast denial with native spell learning, passive aura and client spellbook fidelity. [WP-03 auto-learning feasibility](AUTO-SPELL-LEARNING-FEASIBILITY-DECISION.md) remains a hard blocker.

There is **no approved C++ source tree yet**; these are conceptual components, not files to create automatically or guaranteed hooks.

## Special safety instructions

- **No automatic gear/talent resets or bot maintenance changes.** Naxx Core NT1 and Playerbots remain as currently designed.
- No copying individual progression query/damage code into unrelated per-tick loops that duplicate scaling or break IP's group logic.
- No manual `UPDATE quest_rewarded` or character quest progression edits to fake a test. Future tests should use disposable players and approved commands/legitimate progression.
- Backup **characters database, server configuration, current modules and DBC** before a future experimental deployment. Read-only source exploration and documentation do not require modifying production.
- Any new module hook must be measurable and reversible, and must leave an IP-disabled or non-Hunter actor completely unaffected.

## Open decisions—not granted by creating this document

- Which progression policy applies to a **standalone Random Bot** that IP exempts, a **grouped Random Bot** that inherits a leader stage, and an **Alt Bot** owned by a human but represented by its own character/account?
- Whether to retain Vanilla-known skill/TP/loyalty history after TBC transition without deleting/rewriting native WotLK pet spells.
- Whether the strict no-client-visible-automatic-spell constraint allows a reduced-fidelity workaround. **No approval has been given.**
- Whether and how the Hunter Pets module is integrated into Naxx Core in the future, after separate proven release; **not now**.

## Next read-only evidence

Use [Installed IP inventory checklist](READ-ONLY-INSTALLED-IP-INVENTORY.md). Pin the *actual* deployed IP and Naxx Core SHAs and verify whether IP header is available and matches the public modern API. This document alone **does not confirm** the user's live worldserver version or runtime configuration.
