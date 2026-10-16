# Implementation roadmap — phased, gated, module-only

**Status: proposal, not authorization to code or deploy.**
**Estimated total:** **42–80 developer-days** as a broad initial *single-developer engineering* estimate. This excludes long waiting periods for the owner's in-game testing and may change substantially after the source audit. **No promised delivery date.**

## Stage 0 — Baseline and safety inventory (research only)
**Approx. 3–5 developer-days.**

Deliver:
- Exact deployed revisions / compile options of AzerothCore, IP, Playerbots, Naxxramas Core and matching 3.3.5a client build.
- Existing pet system behaviour matrix: tame, auto-learn, family roles, food/happiness, XP, stable, talents, commands, bot AI.
- Read-only database schema inventory; inspect all references to affected tables and relevant hooks; identify existing IP pet tweaks.
- Historical 1.12 reference dataset and primary/secondary source quality notes.
- Module-only feasibility table and risk list, updated from [technical review](TECHNICAL-FEASIBILITY.md).

**Gate:** no production changes; proceed only once safe extension points and rollback principles are documented.

## Stage 1 — Tiny, isolated proof of concept
**Approx. 4–8 developer-days.**

Aim: restore a verified original creature property to a *single test Hunter* without changing normal pets.

- Prototype per-entry pet attack-speed override in experimental module, starting with Broken Tooth and a control pet.
- Ensure damage normalization, no double scaling; compare autoattacks, focus and threat.
- Exercise tame → level-up → stable → resummon → logout → restart → death/revive.
- Feature flags off by default; hook registration and override order checked.

**Gate:** if it cannot survive the full lifecycle entirely module-only, stop and document blocker.

## Stage 2 — Beast-specific ability discovery
**Approx. 8–15 developer-days.**

- Build audited creature→ability/rank data and Hunter-known catalogue.
- Implement narrow learn-by-use prototype for one ability (suggest Lightning Breath Rank 6, using a safe test environment and appropriate ZG unlock).
- Test Hunter learns only what the pet actually uses; stable/swap and teach another eligible pet.
- Explore genuine suppression of WotLK automatic skill progression and validate persistence.
- Reject ranks outside Vanilla policy without deleting pre-existing data.

**Gate:** **true module-only ability acquisition and automatic-skill suppression must both be proven**. If not, re-scope with the owner; do not invent a hidden upstream core patch.

## Stage 3 — Loyalty, feeding and Beast Training
**Approx. 12–22 developer-days.**

- Implement/persist six loyalty ranks, loyalty gains/losses and safe happiness interactions using existing AzerothCore logic where correct.
- Formula/ledger: pet level × (loyalty−1) training points. Validate affordability, four-active-skill cap, ranks, rank replacement/refund, zero-cost Growl.
- Family skill lists, ability learning, pet trainer interactions, untrain/respec, UI (NPC gossip first unless a working native client-safe UI is proven).
- Anti-loss migration for existing pets; initially **disable irreversible runaway**.
- Build authoritative audit and reconciliation for native `pet_spell`/character saves.

**Gate:** no partial state updates, no duplicate skills, and save/load transitions pass.

## Stage 4 — All Vanilla pet family data and progression fidelity
**Estimate: included across Stages 2–3, refine after audit.**

- Fill and verify **every** original Vanilla pet family and each skill's ranks, training costs, compatible families and tame source.
- Audit appearance, diets, HP/armor/damage family multipliers, special attack speeds, pet XP and tame catch-up.
- Check Classic hunter quest, Beast Lore, hunter/pet trainer interactions, stable mechanics and class talent overlaps.
- Identify any data requiring client/server DBC, world SQL or extra client addon; owner approval needed before such assets.
- Compare PvP, raid and solo roles; ensure pets are different without arbitrary rare-only damage buffs.

## Stage 5 — IP + Playerbots integration and acceptance
**Approx. 10–20 developer-days.**

- Vanilla **all-stage** IP state gate; explicit non-Vanilla strategy agreed before live mixed-IP testing.
- Human Hunters, persistent Alt Bots and Random Playerbots treated as separate users with distinct persistence/ownership.
- Bot-only provisioning of historically legitimate legal abilities requires a signed-off policy; avoid granting every possible skill by default.
- PvP/BGs, Molten Core, Zul'Gurub, instance save/restart; pre-existing NT1 imports and bot consumable/raid behaviour unaffected.
- Include upgrade/downgrade/disable test, actual in-game test report and owner sign-off.

**Gate:** module-only compliance, no irreversible data loss, full reproducible test log, disabled-config parity.

## Stage 6 — Integration review (future, separately approved)

1. Tag and preserve the last proven standalone module state.
2. Conduct a **separate review** on merging code into `Mod-Naxxramas-Core`; verify no conflict with current Naxx Core development.
3. Plan SQL/config ownership transfer and backward-compatible persistence; test disablement/removal of standalone without pet corruption.
4. Merge only through reviewed development PRs and owner approval. Standalone and merged implementations must not run simultaneously.
5. Retain this repo as historical specification/test evidence unless explicitly decided otherwise.

## Risk and contingency

- **Critical:** no hook to stop automatic 3.3.5 pet spell teaching; may block strict Vanilla training.
- **High:** 3.3.5 pet-talent UI and spells conflict with Vanilla training points.
- **High:** irreversible runaway/stable/skill migration of existing characters.
- **High:** Playerbot refresh/AI overwrites learned ranks or pet behaviours.
- **Medium:** rare original creature properties not preserved in modern template data.
- **Medium:** exact source/patch variation (1.12 vs early Vanilla) and DBC/client patch requirements.

**Contingency principle:** prefer an explicit honest partial restoration (e.g. per-beast attack speed) over an unreliable full restoration. If feature cannot fit module-only boundaries, flag it as blocked and ask owner to revise scope.

## Developer reporting format

Each stage must produce: scope; exact repo base/commit; source of historical data; files changed; build/test status; manual tests with evidence; current limitations; config/SQL/DBC changes; backup and rollback; impact on IP/Playerbots/Naxx Core; approval needed. *Source committed* never means *installed or validated*.
