# Individual Progression, Naxx Core and Playerbots compatibility contract

**Status:** Proposed integration design. Exact deployed APIs and bot branch are unknown until Stage 0 audit.

## I. Per-character IP eligibility

Rule: **Every Vanilla IP stage is eligible for Classic pet mechanics**, from starting stage through final Vanilla content. Do not turn on WotLK behaviour during AQ, Naxx, ZG, etc. merely because it's late Vanilla. Exit only when **that Hunter** meets the authoritative TBC entry milestone, not when their level reaches 61.

- Use the installed IP module's public per-player progression API, or approved read-only milestone check when no public API exists.
- Do not hardcode stage numbers from a different IP revision.
- Validate effective expansion settings if IP is disabled, or if global progression limit forces Vanilla.
- For a Hunter at IP TBC/WotLK, avoid any destructive interpretation; post-Vanilla pet-policy decision still OPEN.
- NPC tameability, zone unlocks and raids like Zul'Gurub must follow existing IP rules. **Do not edit IP** to unlock a pet skill source.
- An owned skill learned legally in an earlier Vanilla stage must remain known throughout later Vanilla stages.

## II. Human and bot ownership is separate

**Human:** all authentic taming/learning/training/loyalty systems, no free catalogue injection.

**Alt Bot:** persistent character/pet, may be controlled by same-account character. No gear modification, no NT1 talent changes and no implicit automatic full skill unlock. Needs explicit source/legal ability provisioning choice and no risk to valued alt pets.

**Random Bot:** generated/recycled by Playerbots. Track actual bot GUID/pet ID and lifecycle; no global sweep or unbounded module rows. Bot-specific setup may be session scoped or deterministic but needs an owner-approved legal rank policy and cleanup.

**Uncontrolled companions:** Warlock demons, Mage elementals, DK ghouls, Shaman guardians and NPC pets must never be modified.

## III. Eligibility and era mismatch risk

Naxx Core NT1 issuer-era checks may validate a *human player* only; that is **not proof of a Playerbot's own IP era or pet-legality**. Any bot pet operation must independently validate actual bot owner eligibility or refuse it safely. No accidental whitelist from the human master.

## IV. Playerbots learning strategy — choices for later decision

| Proposal | Advantage | Cost/risk |
|---|---|---|
| A. True AI tames/learns ranks via gameplay | Highest historical fidelity | Expensive AI changes; forbidden upstream edits; may be blocked module-only |
| B. Deterministic *bot-only* class/level/family/era training catalogue with provenance | Raid-ready and auditable; keeps human authentic learning | Bot shortcut must be tightly isolated; may dilute realism for bots |
| C. Preconfigured curated bot pets per raid tier, with training points and legal skills | Predictable AI, lower complexity | Needs pet/source legality audit and per-bot persistence; may feel artificial |

**No option approved.** Default code must not secretly grant the full rank list to every bot. A previous raid-planning discussion identified the need for Hunters to learn skills they have never tamed; this is an operational requirement, not permission to bypass all player rules.

## V. Naxx Core regressions

Source: [Mod-Naxxramas-Core](https://github.com/CosmicCuddle/Mod-Naxxramas-Core), especially its Playerbot consumables and NT1 systems.

- Hunter `.bot consumables mc` currently includes pet-related raid consumable handling. New module must not remove pet auras, mishandle stacking, or assume native PetSkill data format.
- NT1 bot talent imports must remain separate from pet TP training. Talent reset is **not** equivalent to pet untrain.
- Alt Bots' gear and bags remain untouched. Random bots may rerandomize talents/pets; new ability ledger must fail safely.
- Raid wolf Furious Howl, wind serpent Lightning Breath, cat Bite/Claw and owl Screech must be understood by existing AI or exposed via documented public interface. Training a skill does not mean AI will actually cast it.
- MultiBot and other addons remain optional convenience surfaces only; they cannot become server-authoritative.

## VI. Concrete test roster

- Human fresh level 10 Hunter, active level-60 Vanilla Hunter and level-60 TBC-stage Hunter (same level, different era).
- Disposable persistent Alt Bot Hunter with pet; unique second Alt Bot owner as isolation control.
- Disposable grouped Random Hunter Bot with pet, then group disband + randomization/regeneration.
- Warlock and Shaman in same raid as negative pet class controls.
- MC and ZG instance environments (no production pet or live loot change while testing).

## VII. Collaboration boundary for module editor

All compatibility changes must be inside **this new module** during experimentation. The editor may *inspect* source and call documented interfaces, not rewrite Playerbots, IP, Naxx Core or AzerothCore. A feature requiring upstream AI edits must be labeled **blocked under current constraints** and offered back as a decision, not silently implemented.

## VIII. Future TBC expansion boundary

At TBC entry: owner decides whether to restore TBC's distinct pet training/abilities or remain on WotLK base; no auto-conversion policy can be assumed now. Preserve ability knowledge/loyalty metadata for reversibility and re-entry tests until explicitly decided. Data model must support a future `ruleset_version` without discarding learned skills.
