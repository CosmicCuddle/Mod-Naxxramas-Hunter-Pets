# Risk register, hard feasibility gates and contingency decisions

**Status:** Pre-coding architecture risk assessment. Estimates below are qualitative, not actuarial probabilities.

| Risk ID | Priority | Threat | Evidence / trigger | Required treatment |
|---|---|---|---|---|
| R-01 | **Critical** | AzerothCore automatically teaches Hunter pet ranks | `Pet::InitLevelupSpellsForLevel` calls learnSpell; no documented generic `CanLearnPetSpell` hook | Instrument exact deployed call order; prove module-only prevention; if impossible mark strict training **blocked** |
| R-02 | **Critical** | Vanilla ledger and native pet spell save diverge | Pet save/load is core-owned; module tables independently transactional | Design native-state reconciliation/rollback; prove crash/stable restart cases |
| R-03 | **Critical** | Incorrectly altered existing stabled/valuable pets | 3.3.5 has four stable slots whereas Classic had two | No mass conversion without audited preview, snapshot, owner approval |
| R-04 | High | No event captures successful tame source identity | PetScript's AddToWorld is after pet creation | Prove source NPC → persistent pet number association or stop |
| R-05 | High | Spellcast hook fires before successful execution | `AllSpellScript::OnSpellCast` semantics uncertain for specific spells | Test cancelled/immune/failed skill use before granting Hunter knowledge |
| R-06 | High | WotLK pet-talent UI conflicts with TP | 3.3.5 client removed original Beast Training | UI strategy; prevent free talent effects; do not claim exact native UI |
| R-07 | High | Pet abilities and ranks absent/changed in 3.3.5 DBC | Comparing [Vanilla](https://www.wow-petopia.com/classic/abilities.php) to [Wrath Lightning Breath](https://www.wow-petopia.com/classic_lk/ability.php?id=lightningbreath) reveals different numeric values | Validate actual server/client Spell.dbc; optional client patch only upon approval |
| R-08 | High | Mod-IP alters pet leveling or owner stage | Existing IP pet-level feature and individual tier state | Audit exact IP revision, avoid duplicate level sync/era inference |
| R-09 | High | Bot AI retrains or casts nonexistent spells | Playerbots refresh and spell-rank preference not mapped to Classic | Human-first pilot; separate bot policy and AI regression |
| R-10 | High | Early Vanilla historical differences misinterpreted as 1.12 | Lupos old Shadow damage, TBC Cobra Reflexes | Dataset requires patch/provenance, avoid invented bonus |
| R-11 | Medium | Attack speed restores but increases net DPS | Faster swings without appropriate per-hit normalization | Profile same-family DPS and proc timings |
| R-12 | Medium | Food/happiness has duplicate modifiers | AzerothCore still maintains POWER_HAPPINESS | Native effect audit and exact damage sample |
| R-13 | Medium | Pet desertion causes irreversible valued pet loss | Loyalty loss in Vanilla could cause runaway | Disable until opt-in and recovery design explicitly approved |
| R-14 | Medium | Module hook order conflicts with Naxx Core | Naxx Core has pet-consumables and Playerbot integrations | Pin callback order, integration tests, no direct edits |
| R-15 | Medium | Rank guide catalogs disagree on count/data | Different Classic/TBC/WotLK guide versions | Treat reference datasets as candidates; verify in exact game patch |
| R-16 | Medium | Client display is misleading or unusable | No original 1.12 Beast Training in 3.3.5 | Accessible gossip/addon proposal, explicit UI testing |

## Source-pinned additional critical risks (October 2026)

| Risk ID | Priority | Verified public-source fact | Mandatory action |
|---|---|---|---|
| R-17 | **Critical** | `OnPetAddToWorld` fires before asynchronous saved-spell restoration on existing pets, but after WotLK rank initialization on new tame | Never use one OnAdd reconciliation assumption for both lifecycle paths. Prove safe after-native-load synchronization without client-visible illegal ranks |
| R-18 | **Critical** | `Unit::CreateTamedPetFrom` and `Pet::SynchronizeLevelWithOwner` each enforce owner-level-minus-five Hunter pet catch-up | Prove original wild-level and pet-XP rules can persist entirely module-only; do not edit AzerothCore or IP |
| R-19 | **Critical** | Native pet `SavePetToDB` commits spell/cooldown data separately from main pet identity/slot | Module ledger must survive crash between commits with idempotent recoverability and owner-specific pet identity |
| R-20 | High | `OnSpellCast` does not guarantee target hit for all spell types; `AfterHit` is per-effect/per-target | Test learn-by-use for hit, resisted/immune, missed, interrupted, projectile and non-target spells before granting Hunter knowledge |
| R-21 | High | Hunter pet attack times overwritten before `OnInitStatsForLevel`, `UpdateAllStats` follows afterward | Prove correct source-specific swing, normalized damage and no reset on life cycle |
| R-22 | High | Original creature template entry and unique native pet number are available in tame/save paths | Prove stable/relog persistence and source-creature identity rather than infer from model/family/name |
| R-23 | **Critical** | Family auto spell mapping is loaded from 3.3.5 CreatureFamily/SkillLineAbility DBC into shared SpellMgr maps; exposed accessors are const and no per-owner veto observed | **No shared DBC/map hack**; strict Vanilla rank suppression remains blocked until supported per-owner prelearn interception proven |
| R-24 | **Critical** | Core emits native pet spellbook packet on async load before `isBeingLoaded=false`; levelup `learnSpell` sends packets immediately outside load | A later player update reconcile cannot promise clean Vanilla spell UI; stop/review if client sees unearned rank |
| R-25 | High | `AllSpellScript::OnSpellCheckCast` can reject active cast, not native passive aura or skill ownership | Classify active-cast denial as **reduced-fidelity experiment only**; test passives, bot autocasts and triggered spells |
| R-26 | High | Global metadata manipulation applies beyond a Vanilla-stage owner; post-Vanilla Hunters must remain untouched | Reject cross-era data mutation, unsupported const-cast and global default spell SQL workarounds |

Evidence: [source-pinned lifecycle audit](SOURCE-PINNED-PET-LIFECYCLE-AUDIT.md), [16-event data](../data/reference/public-core-pet-lifecycle-events.csv), [auto-grant feasibility verdict](AUTO-SPELL-LEARNING-FEASIBILITY-DECISION.md) and [16 candidate interceptions](../data/reference/auto-spell-interception-candidates.csv). All implementation verdicts remain **NOT PROVEN** until exact deployed source revision and staging tests.

## Mandatory hard gates (stop on failed gate)

**G0 – Read-only source truth.** Deployed commit hashes, current DB/DBC, IP/API, source hooks and owner-visible pet inventory recorded; backup/restore plan passed on staging.

**G1 – Module-only identity.** Tame source identity and native pet ID correlation is lossless over relog/stable/XP.

**G2 – Auto-learning suppression.** All WotLK automatic rank grants are prevented *without* corrupting pet spell persistence or packet state. One demo isn't enough; all entry paths must pass.

**G3 – Valid learn-by-use.** Pet-cast outcome can reliably trigger *Hunter-known* rank acquisition without allowing failed casts, other owners, ranks not native to source pet or bot-injected spells.

**G4 – Atomic training.** Native pet spell list and independent ledger stay consistent through concurrency and crash tests.

**G5 – Era/client compatibility.** Correct IP owner state, adequate stable/UI behaviour, no false WotLK pet talent/rank effects, no pet loss.

**G6 – Bot validation.** Human, Alt Bot and Random Bot test separately, Naxx Core NT1 and consumables preserved.

**G7 – Deployment and rollback.** Restore rehearsed, migration preview signed off, human release acceptance and owner approval.

## Failure response

If G1/G2/G3/G4 cannot be satisfied under **no-upstream-edits**, clearly list exact functions, observed event order, demonstrated failure and possible **compromise with tradeoffs**. Ask owner whether to accept a partial feature, e.g. only family stat/attack-speed restoration. **Do not stealth-edit AzerothCore or present a workaround that breaks pet saves as complete.**

## Milestone estimate breakdown

Research 3–5 developer-days; source-property spike 4–8; ability pilot 8–15; loyalty/training 12–22; UI 5–10; IP/bot/acceptance 10–20. Total **42–80 developer-days** roughly, substantially uncertain until hard gate G2 is proven. These are engineering-effort estimates, not wall-clock timing guarantees or a committed schedule.
