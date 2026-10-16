# Mod-Naxxramas-Hunter-Pets

> **Planning and feasibility stage — NOT an installable module.**
>
> Experimental AzerothCore 3.3.5a project to restore the original **Vanilla Hunter pet experience**: meaningful beast taming, learn-by-use abilities, loyalty, happiness, feeding, training points, ranks, pet families and unique pet characteristics.

## Our goal

Bring back the feeling that finding, taming and training the **right creature** actually matters. Different pets should have different historically grounded purposes—for raiding, PvP, farming and solo play—without arbitrary bonuses just because a beast is rare.

Illustrations of intended research:
- **Son of Hakkar:** acquire Lightning Breath Rank 6 by taming the right Wind Serpent and learning the skill through its use, then train another eligible pet.
- **Broken Tooth:** restore and test its original 1.0-second attack cadence, with proper damage normalization.
- **Every Vanilla pet:** examine original tame rules, source ability ranks, diet, happiness, six loyalty ranks, training-point budget, four-active-skill limit, pet family roles, stable interactions and pet XP.

**Era rule:** The Vanilla system stays active **through all Vanilla Individual Progression stages** until the character reaches TBC. **TBC and WotLK behaviour are intentionally undecided** and will be separately planned later.

## Documentation — start here

| Document | Purpose |
|---|---|
| **[Official project handover](docs/PROJECT-HANDOVER.md)** | Decisions, project boundaries and developer handover instructions |
| **[Vanilla feature specification](docs/VANILLA-FEATURE-SPEC.md)** | Comprehensive list of proposed original Hunter pet systems and acceptance cases |
| **[Technical feasibility](docs/TECHNICAL-FEASIBILITY.md)** | AzerothCore hooks, confirmed source observations, uncertain/blocking mechanisms |
| **[Implementation roadmap](docs/IMPLEMENTATION-ROADMAP.md)** | Feasibility gates, staged approach and preliminary **42–80 developer-day** estimate |
| **[Test and rollback plan](docs/TEST-AND-ROLLBACK.md)** | Human/bot/IP regression matrix, data safety, backups and acceptance |
| **[Research register](docs/RESEARCH-REGISTER.md)** | Historical references, data datasets to compile and unanswered research questions |
| **[Decision log](docs/DECISIONS.md)** | Agreed vs suggested vs open decisions; do not mistake proposals for approval |

## Architecture and safety

- **Independent development:** Keep experimental C++, configuration, SQL and optional assets in **this module**. Nothing is being merged into [Mod-Naxxramas-Core](https://github.com/CosmicCuddle/Mod-Naxxramas-Core) during research.
- **Read-only upstream:** Do **not** edit AzerothCore, Individual Progression, Playerbots or Naxxramas Core source to make a prototype work. If an upstream change appears mandatory, **stop and report** why.
- **Do not assume 3.3.5 is Vanilla:** Its pet UI, pet talents and automated rank learning differ. AzerothCore still has happiness, but **WotLK auto-learning is a major unresolved module-only feasibility blocker**.
- **No unapproved live changes:** No experimental code, SQL, configuration switches, DBC changes, released artifacts or deployments have been created by this repository planning work.
- **Backups before future tests:** Snapshot and verify restoration of characters database (including native pet data), module configs, working server build and relevant world/client DBC assets. Test with disposable characters on staging first.
- **No irreversible pet loss:** Do not enable pet runaway, abandon existing pets, reset pet talents/skills or force stable-capacity changes without an explicit migration policy and owner approval.
- **Playerbots:** Future compatibility required, but no modification to mod-playerbots; Alt Bots' gear and existing Naxx Core NT1 and consumable systems must remain untouched.

## What the future module editor should do first

1. Read **all** documentation linked above, especially the handover and decision log.
2. Perform the read-only **Stage 0** audit of the *deployed* AzerothCore, Individual Progression, mod-playerbots, and Naxx Core revisions and actual pet database state.
3. Report which requirements are fully possible using documented module hooks, which require a safe compromise, and which are blocked.
4. Propose the **Broken Tooth attack speed** and **Son of Hakkar ability-discovery** proof-of-concepts with exact tests and rollback.
5. **Request separate implementation approval** before writing code or applying SQL. Source files, compilation, installation and in-game verification are distinct states.

## Current state

| Area | Status |
|---|---|
| Design concept and owner constraints | **Documented** |
| Historical research and data completeness | **In progress / not validated** |
| Hook and architectural feasibility | **Preliminary only** |
| C++ module implementation | **Not started** |
| SQL migrations, DBC files, client addon | **Not started** |
| Compile and automated tests | **Not started** |
| In-game tests / release | **Not started** |

**This repository is currently a coherent specification and development handover—not proof that the restoration is already technically complete or feasible within every stated restriction.**
