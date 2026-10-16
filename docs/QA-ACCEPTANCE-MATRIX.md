# Reproducible acceptance and regression test matrix

**Status: specification — zero tests run as part of this planning work.** To call any stage complete, supply evidence per case: test ID, commit, deployed revision, SQL migration hashes, config, actual IP stage, player/bot identity, pet ID/source creature, setup, actions, expected and actual result, logs/screenshots, PASS/FAIL/BLOCKED.

## A. Module isolation and controls

| ID | Test | Expected outcome |
|---|---|---|
| ISO-01 | Build module against pinned live AzerothCore without modifying core source | Build reproducible; core Git working tree unchanged |
| ISO-02 | Inspect IP, Playerbots, Naxx Core before/after experiment | No source changes to those repos |
| ISO-03 | Start with config OFF | Existing 3.3.5 Hunter/pet behaviour untouched |
| ISO-04 | Disable after experimental enable | Defined, reversible pet state; never strips learned abilities behind the user's back |
| ISO-05 | Unknown IP API or stage mapping | Fail closed safely; no guess from character level or zone |
| ISO-06 | Non-Hunter owner (Warlock demon, DK ghoul, Mage elemental) | Unchanged before and after |
| ISO-07 | Missing/bad reference dataset/DBC mapping | Module refuses enable, clear diagnostics, no implicit spell fallback |

## B. Taming, identity and stables

| ID | Test | Expected outcome |
|---|---|---|
| TAM-01 | Level-10 Hunter quest full chain | Legal tame/feeding training sequence survives |
| TAM-02 | Try forbidden/non-beast/non-tameable NPC | Tame denied normally, no metadata row |
| TAM-03 | Tame wild Wolf and record entry | Correct source NPC entry and family linked to native pet number |
| TAM-04 | Tame 2 visually identical beasts with different source IDs | Preserve each identity, don't conflate skins |
| TAM-05 | Tame at native wild creature level then gain XP | No accidental WotLK owner-minus-five catch-up in Vanilla |
| TAM-06 | Pet dismiss/summon and relog | Pet number and tame provenance unchanged |
| TAM-07 | Stable A, summon B, stable B, summon A | Each pet's skill set/loyalty/attack speed intact |
| TAM-08 | Stable is full with preexisting valuable pets | No silent overwrite/deletion/restriction |
| TAM-09 | Owner logs out during pending tame / tame interrupted | No orphan pending identity and no false pet |
| TAM-10 | Pet death, resurrection, worldserver restart | Identity, trained skills and happiness/loyalty safe |
| TAM-11 | Existing unmigrated WotLK Hunter pet | No retroactive forced modification or state loss |

## C. Ability discovery

| ID | Test | Expected outcome |
|---|---|---|
| SKL-01 | Tame legal source knowing Bite rank | Pet has correct innate skill, Hunter does **not** instantly learn unrelated ranks |
| SKL-02 | Pet actually uses teachable rank in eligible state | Hunter-known catalogue updates only under validated use/policy |
| SKL-03 | Pet attempts spell but fails/interrupts | No unjustified acquisition |
| SKL-04 | Pet uses spell without Hunter being its legitimate owner | No cross-character learning |
| SKL-05 | Tamed pet uses ability already Hunter-known | Idempotent no-op, no spam |
| SKL-06 | Source pet abandoned or stabled after valid acquisition | Hunter knowledge persists |
| SKL-07 | Trainer buys Great Stamina rank | Correct rank/level/gold grant, independent of pet having rank |
| SKL-08 | Learning same rank twice / retry after disconnect | No duplicate writes or charges |
| SKL-09 | Future-era BC/WotLK spell appears on pet | Not counted as Vanilla catalogue |
| SKL-10 | Source NPC family/ability data absent | Refuse guessed acquisition; diagnostic |
| SKL-11 | Son of Hakkar Lightning Breath 6 | Acquires **only after valid use**; owner can teach another eligible serpent |
| SKL-12 | Bloodaxe Worg Bite 8, Dash 3, Furious Howl 4 | Each individual skill learned via its own valid use |
| SKL-13 | Logout, server restart after Hunter has learned rank | Hunter catalogue remains correct |

## D. Training points, costs and learned ranks

| ID | Test | Expected outcome |
|---|---|---|
| TP-01 | Level 60 and loyalty 6 | Total TP exactly **300** |
| TP-02 | Level 60 and loyalty 1 | Total TP exactly **0** (before inherited abilities) |
| TP-03 | New tame has original skill cost exceeding TP | Correct negative remaining TP if faithful to chosen historical rules |
| TP-04 | Teach Growl 7 at remaining TP ≤ 0 | Permitted, **0 TP** |
| TP-05 | Teach Lightning Breath 6 to level-59 serpent | Reject with no state change |
| TP-06 | Teach Lightning Breath 6 to cat | Reject wrong family |
| TP-07 | Hunter never learned target rank | Reject even if pet has level and TP |
| TP-08 | Have 4 active abilities, train a **fifth new** active | Reject |
| TP-09 | At 4 active abilities, upgrade one existing active rank | Allow if legal; still 4 |
| TP-10 | Upgrade a rank with lower one trained | Net TP subtracts new cost minus old cost |
| TP-11 | Skip ranks when eligible and knowledge available | Direct high rank allowed |
| TP-12 | Train Great Stamina 10 (185) + Natural Armor 10 (150) at 300 TP | Reject second as **335 > 300** |
| TP-13 | Train passives with four active abilities | Allow if TP/family/level valid |
| TP-14 | Pet unlearn/respec repeatedly | Costs/refunds correct; Hunter learned catalogue not reset |
| TP-15 | Concurrent identical training requests | One logical transaction, no duplicate charges |
| TP-16 | Attempt level/rank data missing deployed Spell.dbc | Reject with clear error |
| TP-17 | Level or loyalty drops below existing spent points | Defined signed balance; do not silently strip all skills |

## E. Core engine conflicts and special combat effects

| ID | Test | Expected outcome |
|---|---|---|
| ENG-01 | Vanilla pet levels up | No illegal 3.3.5 auto-rank appears |
| ENG-02 | Pet stabled and loaded | No 3.3.5 auto-rank appears |
| ENG-03 | Pet talents are reset/inherited | No free abilities overriding TP catalogue |
| ENG-04 | Native pet saved; module ledger refreshed | Exact known/trained spell consistency |
| ENG-05 | Broken Tooth after tame/level/stable/restart | **1.0-second** base swing speed before modifiers |
| ENG-06 | Control same-family cat with 2.0 swing | Faster pet does not get unexplained extra normalized autoattack DPS |
| ENG-07 | Son of Hakkar Lightning Breath 6 | Correct Nature spell, range, focus, rank and cooldown for Vanilla source data |
| ENG-08 | Two wolves Furious Howl in raid group | Historical party buff duration/targets/stacking correctly enforced |
| ENG-09 | Screech, Scorpid Poison, Shell Shield and Thunderstomp | Classic effects rather than WotLK default equivalents |
| ENG-10 | Owner's talents and IP damage scaling | No stacking of duplicate family/happiness damage multipliers |
| ENG-11 | Pet ability autocast toggled then relog | Preference and action bar preserved |
| ENG-12 | Pet tooltip/data on 3.3.5 client | No false display/phantom Rank 80 skill; approved UI fallback |
| ENG-13 | Screech Rank 4 skill mapping and cast on staging | **Classic Rank 4** behaviour from a verified spell; **27051 must not be mapped as Vanilla** (TBC/WotLK variant) |
| ENG-14 | CMaNGOS preset versus historically documented min level | No rank eligibility follows its eight divergent bot levels without primary DBC confirmation |
| ENG-15 | Great Stamina old trainer passive versus WotLK percent aura | Original **flat Stamina rank** restored once, not an unrelated WotLK **+8%** effect; client tooltip accurately reflects it |
| ENG-16 | Spell IDs for all 59 wild and 52 trainer rank entries | Original patch/build mapping plus exact 3.3.5 server/client row confirmed; missing/conflicting mappings cannot activate |
| ENG-17 | Five resistance school Rank 5s during original Vanilla | **Do not offer or grant** Arcane/Fire/Frost/Nature/Shadow Rank 5 based only on Petopia/2705x CMaNGOS spell chain; prove late-1.12 trainer offering and DBC or defer |
| ENG-18 | Growl Rank 1–7 / Great Stamina 1–10 / Natural Armor 1–10 / each school Rank 1–4 | Verify historical Rank 1+ upgrade chain, Hunter-trainer wrapper spell vs actual pet aura/spell, pet level, TP and coin separately |
| ENG-19 | Great Stamina after trained and untrained on same pet | Exact increase/decrease in **Stamina stat AND actual maximum health**, including recalculation after relog/stable; no duplicate WotLK % bonuses |
| ENG-20 | Public `spell_ranks.sql` lacks a candidate ID | Do not conclude the DBC spell is absent; verify deployed **Spell.dbc**, server and client effect rows and core spell loading |
| ENG-21 | Source version audit and provenance | Every enabled pet rank tied to SHA/version of original 1.12 data, deployed 3.3.5 DBC, exact per-Hunter IP rule and documented eligibility |
| ENG-22 | OnPetAddToWorld versus asynchronously loaded native spells | Different order for new tame and reloaded pet, prove module does not reconcile until actual native spell load is complete |
| ENG-23 | Fresh level-60 Hunter tames original level-20 beast | Preserve approved original Vanilla tame level (**20**, not WotLK owner−5 = **55**) including XP and pet save under a genuine module-only path |
| ENG-24 | Stabled low-level Hunter pet resummoned at level 60 | No unwanted owner−5 level catchup; native save/XP remains consistent and IP controls do not regress |
| ENG-25 | Crash after native aura/spell commit and before pet-info commit / module ledger commit | No lost valuable pet, duplicated rank, broken stable slot or silent TP divergence; reconciliation idempotent |
| ENG-26 | Cast-trigger vs actual impact for wild rank discovery | Interrupted, immune, miss, projectile delayed hit, triggered and periodic effects do not accidentally grant a permanent Hunter-known rank |
| ENG-27 | New tame source and pet identity after stable/relog | Native CreatureId, unique PetNumber and visual source persist, including rare model; no inference from nearby NPC |
| ENG-28 | IP-capped human level-60 Hunter killing monsters with original low-level pet | Pet receives the intended XP exactly **once** although human XP capped; no double payment or missed XP through IP `OnPlayerGiveXP` |
| ENG-29 | Capped Random Playerbot / Alt Bot pet XP handoff | Correct account-specific IP branch, legal pet level, no bleed into human/other bots, no duplicate XP |
| ENG-30 | Illegal Vanilla Hunter active pet rank cast manual / autocast / bot / triggered | `OnSpellCheckCast` test proves no illegal effect or cooldown bypass; legal natural/paid ranks work; no other classes affected |
| ENG-31 | Detect actual async native pet load completion | `isBeingLoaded` only false **after** native `PetSpellInitialize` packet; any 'ready' handler documents temporary client exposure and cannot claim strict prevention |
| ENG-32 | Postgrant reconciliation after levelup/stable/relog | No rank regrant from `InitLevelupSpellsForLevel`, no old-rank downgrade loop, passive leak, action bar reset, repeated spam, crash divergence or saved pet damage |
| ENG-33 | Attempt global SpellMgr / DBC / world rank edits | Reject any cross-era/global alteration, shared map const-cast, unofficial core patch or client Spell.dbc mutation; strict module-only isolation maintained |
| ENG-34 | Check both spell level-up sources | `SpellMgr` family-level skill-line and PetDefaultSpells paths each accounted for; no rank receives unexpected auto-grant from other path |
| ENG-35 | Pet passive aura before/after cast denial | No passively granted forbidden Vanilla aura (Stamina/Armor/Resistance/talent) applies despite active-cast denial; fail if leakage |
| ENG-36 | Client packet/log order around summon and levelup | No unapproved transient WotLK `PetLearnedSpell`, `PetSpellInitialize`, or misleading action-bar/autocast without owner-approved compromise |
| ENG-37 | Owner IP transitions with spell gating enabled and disabled | Vanilla-stage human/Alt/Random rules scoped correctly; **post-Vanilla Hunter, non-Hunter, wild NPC** unaffected by spell check or cleanup |
| ENG-38 | Copy IP owner-aware pet rule pattern correctly | Resolve actual Owner Player and installed IP stage; restrict to real Hunter pets; preserve all Vanilla stages, non-Vanilla and other pet classes |
| ENG-39 | Interaction with IP UnitScript pet scaling | No duplicate owner-era melee/spell/periodic damage/healing multiplier; original speed or bonuses remain controlled and testable |
| ENG-40 | Historic IP HunterPetLevelFix assumption | Inspect installed config/SHA and check tame level+stable/relog; do not trust old setting merely because a historical issue describes it |
| ENG-41 | IP OnPlayerSpellCast versus pet AllSpellScript casting | Show original IP Player-cast cancellation **cannot** stop Pet::learnSpell, and validate actual pet caster path separately |
| ENG-42 | Modern IP effective stage vs raw rewarded quest stage | Cap-aware `hasPassedProgression` and config take precedence; `.ip get` is raw stage and can intentionally differ under a cap |
| ENG-43 | Legacy IP header compilation vs modern quest fallback | Stage 7 vs stage 8 boundaries detected; if legacy header unavailable, **no silent Vanilla assumption** from absent modern quests |
| ENG-44 | IP group sync changes Random Bot stage while grouped | Re-evaluate bot pet policy after `OnAddMember` sync without pet spell/TP/loyalty corruption or forced owner level |
| ENG-45 | NT1 human issuer stage vs actual bot/pet owner stage | Do **not** inherit issuer as pet owner; demonstrate correct authorisation for Alt Bot and Random Bot without accidental gear/talent changes |
| ENG-46 | IP battlemaster bot exemption versus Hunter pet training | BG-specific exemption must never allow unrelated pet training/free spell grants or erase the bot's trained state |
| ENG-47 | Safe `.ip get` check on authorised disposable actor | Read-only raw IP stage result matches historical quest reader; no `.ip set`/`.ip setbot` or player data mutation in diagnostic |
| ENG-48 | Source worktree HEAD vs actually running worldserver binary | Record difference where build is stale; never claim hook availability merely from a newer `git pull` or GitHub default |
| ENG-49 | Invalid pet spell ID on saved pet load | Never insert invalid original/Classic IDs into `pet_spell`; source shows global `DELETE FROM pet_spell WHERE spell = ?` without pet-number filter |
| ENG-50 | Original spell rank granted as native 3.3.5 passive | Immediate `Pet::addSpell` triggered aura and native stat effects observed; active cast gate is not counted as strict passive suppression |
| ENG-51 | Native family passives across pet save/load | `PETSPELL_FAMILY` not saved in normal `pet_spell` rows but regenerated via `LearnPetPassives`; no assumption that removing a saved row permanently prevents it |
| ENG-52 | PetNumber reused by a different pet after replacement | Vanilla TP/loyalty/natural skill history not inherited by recycled PetNumber; use validated owner + stable pet identity/generation |
| ENG-53 | `removeSpell(learn_prev=true)` during rank cleanup | Lower rank may be automatically relearned by native rank chain; no unearned predecessor after intended cleanup |
| ENG-54 | Native `addSpell` action bar, autocast and talent impact | Spell selection/rank upgrade does not silently reset pet autocast or native WotLK talent bookkeeping |
| ENG-55 | `ServerScript::CanPacketSend` whole-packet interception | No pet spellbook packets dropped during logging-only probe; packet filtering not claimed to prevent learn/passive changes |
| ENG-56 | `OnPacketSent` instrumentation | Logs represent attempted packets even when no socket or packet later vetoed, never mislabelled 'client received' |
| ENG-57 | `CanUnlearnSpellSet`/`CanUnlearnSpellDefault` returns false for a too-high rank | The native **else** branch executes `learnSpell` and can grant an ineligible rank; these hooks are never used as learn prevention |
| ENG-58 | Actual 3.3.5 family-levelup grant list | For each family, match **both** CreatureFamily skill lines to SkillLineAbility `AcquireMethod`, valid SpellStore `SpellLevel>0`; no assumed ID from empty SQL placeholders |
| ENG-59 | Actual 3.3.5 family-passive grants | Independently enumerate family skill-line `AcquireMethod`, passive flag and `SpellLevel=0`; no passive incorrectly assumed a level-up rank |
| ENG-60 | Actual 3.3.5 creature-specific default grants | Respect `PetSpellDataId`, negative map key, and only summonable-template fallback; deduplicate default spells versus family level-up list |
| ENG-61 | Public base DBC SQL empty but deployed binary has data | Do not report native spells missing based on no base `INSERT` records; compare effective binary-plus-local-DB data and verify file hashes |
| ENG-62 | All 17 Vanilla family identities against loaded 3.3.5 families | Source reference family is matched to verified `CreatureFamily.dbc ID` and its two skill lines, never guessed from name/model alone |
| ENG-63 | All 21 Vanilla ability names / 111 rank records versus loaded 3.3.5 | Each rank classified by **actual** auto-levelup/default/passive status and 3.3.5 server/client spell effects; unresolved ranks remain `UNVERIFIED` |
| ENG-64 | Screech Rank 4 same ID 24579 across public Classic and 3.3.5 | Detect changed damage, debuff duration/AP, required pet level and cooldown; no blind reuse for original Vanilla |
| ENG-65 | Furious Howl Rank 1 same ID 24604 across public sources | Vanilla party next-hit damage buff vs 3.3.5 wolf/master AP buff kept distinct; target and focus/cooldown correct |
| ENG-66 | Lightning Breath Rank 5 same ID 25011 across public sources | Vanilla 50 Focus/no listed cooldown/78–90 Nature vs public 3.3.5 20 Focus/10s cooldown/40–52 base and scaling; Rank 6 separately verified |
| ENG-67 | Growl Rank 2 same ID 14916 threat calculation | Classic 65 threat vs public 3.3.5 reference scaling 65+4.1/level investigated; TP=0 is independent of 15 Focus cast cost |
| ENG-68 | Great Stamina identical name with different spell IDs 4187 vs 61686 | Classic trained +3 flat stat never silently replaced by 3.3.5 pet talent +4% Stamina; no doubled HP/stat benefits |
| ENG-69 | Screech Rank 5 expansion spell 27051 | Never map post-Vanilla 27051 to original Vanilla Screech Rank 4; era and rank provenance checked |
| ENG-70 | Public reference proves a 3.3.5 spell has different effects | Keep native family level-up/default/passive grant flags **UNVERIFIED** until installed 3.3.5 binary+override family map and runtime tame/load are checked |
| ENG-71 | Screech Rank 4 per-target damage, AP debuff, duration | Check exact deployed effect indexes and hook compatibility; no wrong-actor aura or IP double scaling |
| ENG-72 | Screech and Furious Howl native Focus/cooldown | No fake post-cast Focus deduction, no stale client cost/cooldown; restore only with fully proven supported mechanism |
| ENG-73 | Furious Howl Vanilla party next-hit proc versus WotLK AP | A true next-hit +damage proc for party targets proven if attainable; no simultaneous unintended WotLK AP aura |
| ENG-74 | Existing Furious Howl native `-24604` script and Call of the Wild 53434 | Active script binding and filter checked before any module code; post-Vanilla pet+owner targets remain intact; no reliance on unproven filter ordering |
| ENG-75 | Lightning Breath Rank 5 hit damage hook versus IP scaling | Original Nature damage range and target behavior verified across crit/miss/resistance and Vanilla subtiers, no doubled IP output modifiers |
| ENG-76 | Lightning Breath Focus/cooldown client/server checks | Resource availability checked before cast and 3.3.5 cooldown/tooltip aligned; no post-cast penalty pretending to be pre-cast cost |
| ENG-77 | Son of Hakkar Rank 6 distinct from Rank 5 | Only valid tame-source learning enables rank; installed 335 spell/effect and 1.12 candidate independently verified |
| ENG-78 | Growl `SPELL_EFFECT_THREAT` source with effect-hook override | Verify `SetEffectValue` happens before actual ThreatMgr AddThreat for correct effect, no generic damage setter misuse, no false ranks |
| ENG-79 | Great Stamina original flat aura versus WotLK percentage talent | Aura effect type, actual Stamina and maximum HP verified after train/stable/reload without modifying WotLK talent or duplicating IP effects |
| ENG-80 | Client tooltip, Focus and cooldown mismatch despite server effect correction | Partial numeric script success NOT accepted as complete Vanilla spell; document any unfixable display issue and seek explicit owner decision |
| ENG-81 | Restored spell effects without strict native spell eligibility | Do not claim original Beast Training complete if unearned native WotLK spell is still learned, applied as passive or sent to client |
| ENG-82 | Pet Focus cost via `SPELLMOD_COST` owner | Source shows `SpellInfo::CalcPowerCost` calls pet owner's Player `ApplySpellMod`; **specific aura and exact spell-only scope** still need proof |
| ENG-83 | Pet spell/category cooldown via owner `SPELLMOD_COOLDOWN` | `Creature::AddSpellCooldown` correctly evaluates spell AND category recovery; no shared rank/category leftover or unsigned underflow |
| ENG-84 | Manual pet cast versus AI autocast | Both `PetHandler::CheckPetCast` and `PetAI` paths observe same owner modifier, correct native Focus precheck/cooldown and no duplicate charge |
| ENG-85 | SpellModifier family mask versus exact intended pet spell ID | No affected Hunter/non-pet spells, extra pet rank or unrelated family spell; exact 3.3.5 spell-family flags validated |
| ENG-86 | `GlobalScript::OnIsAffectedBySpellModCheck` return semantics | `false` **forces affected** bypassing family mask, not vetoing; no broad false return or cross-module hook interference |
| ENG-87 | Valid modifier aura and owner Player registration | Existing 3.3.5 valid spell/aura and supported module-only registration verified; no guessed DBC ID, native pet_spell insertion or global unsafe DB patch |
| ENG-88 | Modern IP stage transition and bot group sync | Hunter owner at Vanilla 0–7 eligible only as approved; leave TBC/WotLK, unknown Random Bots and stale grouped bots untouched; tear down modifier cleanly |
| ENG-89 | Owner aura persisted after dismissal/stable/logout and module disabled | Owner modifier does not remain after legal scope ends, and native pet spell/action bar/talents survive unchanged |
| ENG-90 | Client cast cost and cooldown after server owner spellmod | Correct actual resource spend, cooldown packet, pet action bar and client presentation; mark reduced fidelity if text/tooltip still disagrees |
| ENG-91 | Triggered pet casts with cost/CD ignore flags | No false success claim from triggered spells that bypass ordinary native power/cooldown checks; wild NPC unaffected |
| ENG-92 | Furious Howl serial target-filter scripts | Public `Spell::CallScriptObjectAreaTargetSelectHandlers` passes one list sequentially; an additive script cannot assume it resurrects earlier removed party members or bypasses area validation |












## F. Loyalty, happiness, food and progression

| ID | Test | Expected outcome |
|---|---|---|
| LOY-01 | Correct species accepts approved food | Happiness changes as expected |
| LOY-02 | Wrong food family or food below effective level | Rejected/effect correctly bounded |
| LOY-03 | Happy, content, unhappy damage samples | Historic **125/100/75%** if confirmed on installed code; no double bonus |
| LOY-04 | Rank progresses 1→6 with elapsed time + required XP | Threshold tests pass; no free AFK-only rank where XP required |
| LOY-05 | Pet at level cap gains loyalty while owner earns qualifying XP | Progress tracks correct requirement |
| LOY-06 | Dismissed/stabled/logged out for extended time | Offline timer/decay consistent with selected historical rules |
| LOY-07 | Happiness loss/neglect | Loyalty declines only per rules, no surprise production permanent desertion |
| LOY-08 | Loyalty changes | Training-point budget changes without signed arithmetic corruption |

## G. IP, bots and backwards compatibility

| ID | Test | Expected outcome |
|---|---|---|
| IP-01 | Human level 60 Vanilla final-stage | Vanilla rules active |
| IP-02 | Human level 60 with TBC stage unlocked | Post-Vanilla policy (requires approval); no arbitrary level-only gate |
| IP-03 | Two Hunters same party with different IP stages | No cross-player state leak |
| IP-04 | Progress from first Vanilla stage to final Vanilla stage | **No pet rules change** |
| IP-05 | TBC entry trigger | No destructive conversion; approved transition only |
| IP-06 | Original IP HunterPetLevelFix and module enabled | No double pet level adjustments |
| BOT-01 | Alt Bot pet persisted through relog/stable | Correct legality and integrity; no gear/talent refresh |
| BOT-02 | Random Bot rerandomization after group raid | Does not corrupt human catalogue or other bot ownership |
| BOT-03 | Bot consumes `.bot consumables mc` Hunter pet buffs | Existing Naxx Core consumables preserved |
| BOT-04 | Bot pet AI casts trained abilities correctly | No inaccessible illegal skill / spam loop |
| BOT-05 | Bot's own era differs from issuing human's IP | Uses correct bot rule or fail-safe, never human inference |
| BOT-06 | Warlock summoned pet in same raid | Completely unaffected |
| ROL-01 | Rollback to pre-module state on staging | Pet counts, stable entries, spell IDs, talent choices and character data intact |
| ROL-02 | Remove standalone after hypothetical Naxx Core merge | No double-processing and no orphaned persistence schema |

## Severity/coverage policy

- **P0:** pet loss, corrupt saved spells, invalid cross-owner skill, upstream edit, row migration irreversibility; blocks any release.
- **P1:** WotLK auto-skill leak, TP bypass, IP leakage, 3.3.5 client unable to use trained ability, bot regression; blocks public release.
- **P2:** incorrect cosmetic display, misreported status text, unnecessary log noise; fix prior to polished release.
- Each failed test has minimal reproduction and owner-visible tracked decision.
- An end-to-end live test is not replaced by 'compiles successfully'.
