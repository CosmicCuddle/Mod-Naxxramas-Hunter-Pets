# Vanilla Hunter pet restoration — feature specification

**Status:** Target requirements; **not implemented**. Historical mechanics should be verified against reference data before coding.

## Era and eligibility

- Every **human Hunter character still in Vanilla IP** receives the Vanilla pet gameplay package throughout all Vanilla content and raids, until IP's actual TBC-entry milestone.
- Resolve the **owner Hunter's individual IP state**, not global server patch level, location, pet level or raid tier. A level 60 Hunter is not automatically proof of Vanilla IP.
- Audit the actual installed IP branch/API; Naxxramas Core documents a modern stage 8 TBC entry, but hardcoding an unverified stage ID is not acceptable.
- For Hunters past Vanilla: retain existing WotLK behaviour **pending owner decision**, with a clear migration/compatibility design; avoid inconsistent pet state in mixed-IP groups. No automatic conversion may delete pet abilities.
- Only hunter-controlled hunter pets, not Warlock demons, Death Knight ghouls or summoned guardians. Bot inclusion is a separate controlled rollout.

## A. Taming and beast identity

- Preserve real **Tame Beast** gameplay and normal tame restrictions, hunter quest progression, stable interactions and dismissal/revival.
- Capture and persist the **specific original creature template/entry** (and relevant appearance) at tame time so authentic creature-specific properties are recoverable after logout, death, stabling, level changes and restart.
- Pet family determines eligible ability list, diet, family stat profile, role strengths/limitations and permitted rank training.
- Rare status by itself is **not** a damage multiplier. Only verified creature-specific traits are used.
- Assess original Vanilla availability: beast families, tame eligibility, special/unique skins, elite/rare beasts, unusual attack speed, resistances, innate skills and historical patch exceptions.
- Review pet level on tame: Vanilla should not silently grant WotLK catch-up levels. IP already has/had a `HunterPetLevelFix` option in some revisions; check behaviour first, avoid a second competing implementation.

## B. Abilities and ranks — full old system

- Wild-beast-taught skills: **Bite, Charge, Claw, Cower, Dash, Dive, Furious Howl, Lightning Breath, Prowl, Scorpid Poison, Screech, Shell Shield, Thunderstomp** (exact ranks/eligibility to be verified).
- Pet trainer skills: **Arcane Resistance, Fire Resistance, Frost Resistance, Great Stamina, Growl, Natural Armor, Nature Resistance, Shadow Resistance** (verify full rank tables and source).
- A newly tamed beast may already know historically appropriate skills. The Hunter learns the skill/rank by having that beast **use it**, then can teach the learned rank to compatible pets. Taming alone should not indiscriminately unlock every rank.
- Maintain a **Hunter-level learned-skill catalogue** distinct from each individual pet's learned ranks. Persist it across logouts, pet swaps and realm restarts.
- Track learned ranks per pet, action-bar state, autocast preferences, level requirements, focus costs/cooldowns and ability-chain replacements. Do not inject incompatible ranks.
- No WotLK **automatic pet ability/rank learning** should bypass meaningful Vanilla training. Prevent re-grant after level-up, summon, loading or stable swap. This suppression is a major feasibility gate.
- Verify ranks against a documented **creature entry → natural spell/rank** dataset; do not assume current WotLK `creature_template` or DBC data accurately represents 1.12.

## C. Loyalty, happiness and feeding

- Six loyalty ranks (Rebellious → Best Friend), with progression based on historical mechanics.
- Happiness states and feeding/diets, food effectiveness, happiness loss and any associated pet performance changes must follow a chosen, documented Vanilla ruleset; **reuse working AzerothCore happiness where possible**.
- Assess pet abandonment/runaway mechanic as a historically authentic feature, but **never enable irreversible abandonment during experiments**. Owner must approve its exact warning, eligibility and recovery policy.
- Happiness and loyalty must persist and survive stable, logout, death, revive and restart.
- Audit pet damage modifiers at happy/content/unhappy and avoid applying a multiplier twice.

## D. Training points and beast training

- Target Classic formula: **total training-point budget = pet level × (loyalty rank − 1)**. At level 60 and loyalty 6 this is **300 TP**.
- Skills have **explicit training-point costs, ranks, pet level requirements and pet-family restrictions**.
- **Growl costs zero TP**, including when the budget is exhausted. Upgrading a rank replaces it and refunds the old rank's cost; a pet can skip intermediate ranks where historically legal.
- Original training rule: up to **four active** pet abilities simultaneously; passive skills use TP but do not count toward active-skill slots.
- Trainer respec/untraining must recalculate cost and TP safely. Repeated resets and refund policy need researched data.
- Preserve existing abilities and points on failed requests; do not charge or corrupt state if a train action is rejected.
- Pet-training UI: explore existing 3.3.5-compatible interfaces; **NPC gossip or a dedicated addon** can provide an accessible replacement. Original pre-3.0 Beast Training client window cannot be assumed available in an unmodified 3.3.5 client.

## E. Combat roles and historically significant beasts

- Restore verified creature attack speeds and proper damage normalization: **faster attacks ≠ automatic extra DPS**. Assess PvP spell pushback, threat and raid DPS separately.
- Family differences in damage, health, armor, diet, abilities and tactical niche should matter, with **historical data and balance evidence**.
- **Son of Hakkar:** taming for Lightning Breath Rank 6 should be meaningful; Hunter can teach it to other eligible Wind Serpents. Ensure Zul'Gurub unlock timing follows IP.
- **Broken Tooth:** study its 1.0-second speed and actual effect in PvP; ensure speed survives pet-stat reinitialization.
- **Lupos:** its earlier Shadow-damage phase predates final Vanilla; **not approved** as a custom early-patch exception.
- Consider wolves, cats, owls/bats, boars, wind serpents, scorpids, turtles, bears and other actual Classic families using verified family/ability data. Do not simply label one species universally 'best'.
- Audit Hunter's Beast Mastery talents, Mend Pet, Revive Pet, Bestial Wrath, Kill Command (not a Vanilla spell), Growl autocast and focus mechanics for unintended WotLK leakage. Only change Hunter pet mechanics in approved scope.

## F. Pet slots, levelling and ecosystem

- Audit authentic Vanilla active/stabled pet capacity and current 3.3.5 client/server constraints. Do not reduce current capacity or delete existing pets before explicit migration approval.
- Pet XP gains, tame level, level caps, loyalty growth and stable/rehire behaviour should not bypass progression.
- Pet skill availability must match Vanilla-era rank and level rules even if 3.3.5 spell data contains stronger future ranks.
- Trainer NPCs, Beast Lore identification, hunter taming quest, pet food vendors, stables and spellbooks all require end-to-end investigation.

## G. Playerbot / Alt Bot support

- **Do not bypass** real Hunters' learning loop for convenience; human experience is acceptance priority.
- For Hunter Playerbots: research an explicit, **legitimate pet-knowledge preparation** path so bots can obtain legal Vanilla pet skills without pretending they tamed beasts they never tamed. Policy on provisioning must be approved; default may be a documented deterministic bot-only initialization.
- Random Playerbots and persistent Alt Bots must be tested separately. Never reset Alt Bot gear or unrelated NT1 talents.
- Check pet death/revive, despawn, combat decisions, Growl autocast, focus, raid pet buffs, consumables and existing AI ability usage.
- Respect the **bot's own eligibility/progression** where known; never infer bot era solely from the human issuer's progression.

## Initial acceptance examples

1. Vanilla Hunter tames a Wind Serpent with Lightning Breath Rank 6 and **does not automatically gain all other pet ranks**; it uses the skill and the Hunter learns it.
2. The Hunter stables this pet, summons another Wind Serpent and can teach Rank 6 only if its family/level/TP permit.
3. The same rank cannot be trained on an incompatible pet. An invalid training attempt changes no state.
4. Broken Tooth retains documented original speed across level-up, resurrection, stable and relog; measured DPS remains balanced.
5. Loyalty, happiness, training budget and learned ranks persist; untrained ranks do not reappear automatically.
6. Different human Hunters at different IP eras can coexist without leaking Vanilla modifications to others.
7. No unrelated Warlock/guardian pet behaviour changes; no bot gear changes.
