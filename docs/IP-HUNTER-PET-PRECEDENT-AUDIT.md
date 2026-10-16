# Individual Progression Hunter-pet precedent: module-only reuse audit

**Research-only, 16 October 2026.** Public IP and AzerothCore sources inspected. Installed realm/IP branch unverified; no SQL, C++, DBC, build or server changes. No upstream edits are permissible.

## Verdict

**USE THE IP PATTERN:** resolve a pet's owning Player, determine their effective Individual Progression stage through the *installed* API, and apply a rule only to that owner's legitimate Hunter pet. Preserve distinct human, Alt Bot, and Random Bot behaviour, and leave all post-Vanilla pet handling alone until the owner selects a policy.

**DO NOT CLAIM IP ALREADY SOLVES AUTO-LEARNING:** the inspected IP public code does not contain a generic pet spell-learn veto. Pet abilities granted by AzerothCore Pet::InitLevelupSpellsForLevel are an operation distinct from IP's combat, capped-XP and Player spell casting hooks.

## Pinned public IP evidence

Source: [IndividualProgressionPlayer.cpp](https://github.com/ZhengPeiRu21/mod-individual-progression/blob/master/src/IndividualProgressionPlayer.cpp), blob SHA e3c5f1781885a4f2ded5b135d2cf4c96467cd9da.

| Lines | Hook | What IP does | What we can safely take from it |
|---|---|---|---|
| 116–159 | OnPlayerGiveXP | At Vanilla human level-60 cap and relevant bot-account cap, send kill XP to existing pet before owner XP becomes zero | **Coordinate with IP; avoid paying pet XP twice** |
| 161–175, 227–274 | OnPlayerSpellCast | Cancel certain old PLAYER lower-rank spells. The code skips bots and is gated to post-TBC tier-5 / WotLK-level conditions | Scoped spell policy is a precedent, **but hook is for Player casts, NOT pet known spells** |
| 1377–1501 | UnitScript heal/spell/melee/periodic hooks | Identify pet-attacker/healer with GetOwner(), then resolve owner stage and BotOnlyAdjustments; scale output by Vanilla/TBC policy | Use *owner-aware scope*, and **do not double-apply IP pet damage/healing multipliers** |
| Public IP source search | HunterPetLevelFix, CanLearnPetSpell, OnPetAddToWorld, Pet::learnSpell | No corresponding current-public branch implementation found for pre-learning | Exact installed fork/commit may differ: **do not infer absence in the user's copy** |

The old IP configuration called IndividualProgression.HunterPetLevelFix was advertised to keep original low tame levels. But [upstream issue #423](https://github.com/ZhengPeiRu21/mod-individual-progression/issues/423), opened **17 November 2024**, reported a Hunter level 59 taming a level-42 wolf and receiving a level-54 pet even with the setting enabled. Issue closed with 'wontfix'. That complaint is evidence of a **reported failure**, not a proof that every historical/forked revision fails. It concerns **level catch-up**, NOT spell learning.

[IP main file](https://github.com/ZhengPeiRu21/mod-individual-progression/blob/master/src/IndividualProgression.cpp) public blob SHA 865560587f0ebb202c4d73852f68358a3f398aeb; reviewed with Player source. Read-only searches of this public repo showed no implementation for the old HunterPetLevelFix name.

## Practical adaptation for Hunter Pets module

One future module-owned eligibility function should:
1. Confirm feature enabled and a real owned Hunter pet, not another class, creature, guardian or charmed unit.
2. Obtain the Player owner. Query **authoritative current IP effective stage** for this exact human/bot; neither pet level nor owner level suffices.
3. If owner is in **any Vanilla stage**, compute legal source-natural spell and Hunter-earned/trained pet spells from our own persistence, source identity, and TP ledger.
4. Otherwise **do not change native pet behaviour** without a later, explicit TBC/WotLK transition decision.
5. Keep IP as owner of its existing XP and scaled combat power; add only old features proved not to double-apply.

**Candidate cast guard:** global AzerothCore AllSpellScript::OnSpellCheckCast can inspect actual pet caster and owner's effective stage, denying *active* unearned ability casts if full spell load interception is unavailable. This is analogous in principle to IP's scoped Player spell policy, but **it is NOT a true pet pre-learn veto** and must be registered/tested in its own legitimate module callback.

**Critical strict-fidelity blocker unchanged:** Pet.cpp calls Pet::learnSpell during new tame, level-up and async pet load. The client may receive unauthorized ranks before any after-load reconciliation; passives and pet action bar may also change. IP's UnitScript/output scaling and OnPlayerSpellCast do not prevent this. No modification of global SpellMgr maps, core Pet.cpp or IP source permitted. See [auto-learning feasibility decision](AUTO-SPELL-LEARNING-FEASIBILITY-DECISION.md).

## Read-only and later approval-gated verification

- **IP-H01:** Pin deployed IP SHA, config and stage API, find old HunterPetLevelFix (if any) in actual *installed* files; verify no assumptions based on historical config.
- **IP-H02:** Human Vanilla earliest/final stage and post-Vanilla stage correctly resolved by actual IP API; no guessed level test.
- **IP-H03:** Real Alt Bot and Random Bot owner/account branches correct; no gear/talent refresh or invasive bot changes.
- **IP-H04:** Confirm IP-only pet melee, spell, heal, periodic damage baseline; no double modifiers with Hunter Pets active.
- **IP-H05:** Vanilla Hunter at XP cap awards pet XP **once**, not zero/twice.
- **IP-H06:** If permitted on disposable staging later, test Pet caster/manual/autocast/Playerbot/triggered cast paths via AllSpellScript, **not** just OnPlayerSpellCast.
- **IP-H07:** Verify native Pet::learnSpell, passives, spellbook/client packet, action bar and persistence; strict training fails if unearned spells are visible or have effects.
- **IP-H08:** Disable Hunter Pets module and test that IP-alone behaviour returns intact.

**Decision:** Borrow IP's owner-aware design and XP coordination now in the blueprint. Do **not** mark automatic pet-spell interception solved or accept a reduced-fidelity gameplay workaround without owner approval. All testing and implementation remain future gated work.
