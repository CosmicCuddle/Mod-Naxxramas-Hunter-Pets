# Architecture and design decision log

**Status legend**
- **AGREED:** expressly selected by repository owner in the planning discussion.
- **PROPOSED:** recommendation, not yet decided.
- **OPEN:** needs owner/technical finding.
- **BLOCKED:** cannot be assumed to work.

| ID | Status | Decision / question | Rationale |
|---|---|---|---|
| D-001 | AGREED | Create separate experimental repository/module `Mod-Naxxramas-Hunter-Pets`. | Isolate risk; merge into Naxx Core only when tested. |
| D-002 | AGREED | Restore **the full old Vanilla Hunter pet experience**, including original abilities, taming meaning, pet families, rare-beast traits, training, loyalty, feeding and pet progression. | The specific creature and effort to train it should matter. |
| D-003 | AGREED | Vanilla behaviour persists throughout **all Vanilla Individual Progression stages**. | No arbitrary mid-Vanilla rule switch. |
| D-004 | AGREED | Defer TBC behaviour/design until the server's progression reaches TBC planning. | Avoid premature conversion choices. |
| D-005 | AGREED | Source changes must stay **inside standalone module** while experimental; no core, IP, Playerbots or Naxx Core source edits. | Strict module isolation. |
| D-006 | AGREED | Repository currently serves as brainstorming and handover; implementation needs a later explicit work assignment. | Avoid unapproved code and live changes. |
| D-007 | PROPOSED | Research against **late Vanilla 1.12-era** mechanics, with explicit exceptions if desired. | Consistent historical baseline. |
| D-008 | PROPOSED | Use 3.3.5-safe NPC gossip UI first; optional addon/client patch requires separate assessment and approval. | Original Beast Training UI was removed from 3.3.5. |
| D-009 | PROPOSED | Default experimental flags OFF; no live data changes, automatic abandonment or irreversible migrations during early pilots. | Player and pet safety. |
| D-010 | OPEN | Post-Vanilla Hunter behaviour when mixed with Vanilla players: default WotLK, dormant module, or state migration? | Need careful future-era handling; never delete trained skills implicitly. |
| D-011 | OPEN | Should historical pet runaway/abandonment be enabled once tested, and what warning/recovery protections are acceptable? | Historical fidelity vs irreversible loss. |
| D-012 | OPEN | Which early Vanilla exceptional pet behaviours, if any (e.g. Lupos Shadow damage), are desirable? | Not part of the provisional 1.12 baseline. |
| D-013 | OPEN | Native client/addon look, Beast Training window, classic stable limits and pet talent pane behaviour? | 3.3.5 UI limitations. |
| D-014 | OPEN | Existing pre-project pets: opt-in conversion, grandfathering, or migration? | Never break valued Hunter pets. |
| D-015 | OPEN | Allowed Hunter Playerbot skill setup policy, distinguished for Random vs Alt Bot. | Respect original gameplay and raid readiness. |
| D-016 | BLOCKED / INVESTIGATE | Can all WotLK automatic pet spell learn events be safely prevented within module-only hooks? | Current PetScript shows unlearn guards, no generic pet-spell learn permission hook. |
| D-017 | OPEN | Initial proof of concept: Broken Tooth speed first, then Son of Hakkar learn-by-use pilot. | Stage-gate the riskiest mechanics early. |
| D-018 | OPEN | Exact effort/delivery after source audit. | 42–80 engineering days is an initial rough estimate, not a commitment. |
| D-019 | AGREED | Treat repository as **both a detailed historical/technical blueprint and actionable developer roadmap**. | Every requirement links evidence, implementation package, risk, acceptance and approval; detail and provenance matter more than vague progress descriptions. |
| D-020 | PROPOSED / SOURCE-CONFLICT | Charge Rank 4 appears in Classic Petopia but **no known tame-teaching source** is recorded. | Do not invent a source or mark obtainable without new evidence. |
| D-021 | AGREED | Aim to restore **as much of the entire original Vanilla Hunter pet system as technically possible**. | Include gameplay, taming, ability families/ranks, unique beast properties, loyalty, TP, happiness, XP, stables, UI and compatible bots; don't shrink scope to a few headline pets. |
| D-022 | PROPOSED | Use one coherent late-original-Vanilla gameplay ruleset as the first reference implementation; preserve early original variants as documented optional candidates. | Research all Vanilla patch changes rather than silently excluding or combining them; specific exception choices require owner approval. |
| D-023 | AGREED | Prioritise **historical references and source-data quality before implementing mechanics**. | A comprehensive blueprint should distinguish original patch evidence, modern Classic reconstructions, core behaviour and actual 3.3.5 runtime proof. |
| D-024 | PROPOSED / SOURCE-CONFLICT | Treat only **47 of 52** candidate trainer ranks as likely available in strict late-Vanilla; **five school resistance Rank 5s** remain blocked pending 1.12 DBC and actual trainer-offering proof. | Petopia's maintainer and multiple independent historical summaries distinguish four original ranks vs five BC; a CMaNGOS spell-chain row does not prove original trainability. Do not remove source records, claim full proof or auto-train without an explicit decision. |
| D-025 | BLOCKED / RESEARCH | **Strict Vanilla Beast Training requires a per-owner, module-only way to prevent WotLK automatic pet rank grants or equivalent before-client native state.** | Public SpellMgr builds global 3.3.5 DBC-derived family/default spell maps; Pet::learnSpell directly grants and sends packets with no generic PetScript learn veto; exact deployed build must be investigated. See WP-03 research report and Issue #4. |
| D-026 | PROPOSED / NOT APPROVED | If true prevention proves unavailable, consider an explicitly reduced-fidelity **active-cast permission guard + after-load spell reconciliation** only after controlled staging evidence and owner review. | Global OnSpellCheckCast could deny some active spells, but cannot stop native learning, passive bonuses or already-sent client spellbook; may not meet the original Vanilla goal. No automatic scope reduction. |
| D-027 | PROPOSED / PUBLIC-SOURCE RESEARCH | Reuse Individual Progression's **pet-owner-and-era-aware hook pattern** within the standalone Hunter Pets module, while leaving IP responsible for existing pet XP and damage scaling. | Public IP UnitScript affects owned pet damage/healing; its OnPlayerGiveXP awards capped pet XP. No matching current-public Pet learn veto found. Old HunterPetLevelFix has a reported failure in upstream issue #423; deployed IP fork must be checked before implementing. |
| D-028 | PROPOSED / NOT APPROVED | Build a **standalone Hunter Pets IP adapter** referencing Naxx Core's modern/legacy source pattern, without calling its internal symbols or changing Naxx Core. | Modern current IP uses TBC entry stage 8, legacy may use stage 7; `hasPassedProgression` observes progression cap; missing legacy reader must fail safely. |
| D-029 | OPEN / OWNER POLICY REQUIRED | Determine whether ungrouped Random Bot pets, grouped Random Bot pets and Alt Bot pets follow bot own current IP stage, synchronised party leader or human command issuer. | IP group joins can sync bot stage; NT1 uses human issuer for a **different** permission. Until decided, log/research only for ambiguous bot era; **never freely train or wipe pets**. |



| D-030 | OWNER-REPORTED / IMPLEMENTATION NOT APPROVED | Owner reports newest Individual Progression installed. Modern hidden-quest era rules (Vanilla 0–7, TBC 8–12, WotLK 13+) are the first planned compatibility target; legacy support is deferred unless needed. | This does not establish exact compiled source revision and is not permission for C++ code, server tests or deployment. Native pet auto-learning remains unsolved. |

## Approval discipline

Update status only after an explicit owner decision or a documented reproducible technical finding. Record date, affected docs/files, user-visible impact, any backup requirements, and release state. A developer's suggestion is not an owner approval.

## Future editor handoff prompt

> Continue the planning and (only if separately authorized) staged experimental development of `CosmicCuddle/Mod-Naxxramas-Hunter-Pets`. Read the root README and all `docs/*.md` first. Maintain strict isolation from AzerothCore, mod-playerbots, Individual Progression and Naxxramas Core. Confirm actual deployed versions and prove module-only feasibility before broad implementation. Do not assume proposed or open decisions are approved, merge, install or release. Preserve original Hunter gameplay and all Vanilla IP stages. Report evidence and blockers explicitly.
