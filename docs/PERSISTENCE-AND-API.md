# Persistence model, runtime API and migration blueprint

**Status:** Unapproved, implementation-agnostic design. Schema names/types are proposals only. No SQL files have been created or executed. Native AzerothCore table names and constraints must be audited on the deployed installation.

## 1. Ownership of state

**Native core is authoritative for live pet spell/action bar/inventory state**. The module may own *additional* Vanilla-specific state. A separate table cannot make the native engine persist a spell safely by itself.

Proposed entities:

| Logical entity | Proposed key | Responsibility |
|---|---|---|
| `naxhp_hunter_known_rank` | (owner_guid, canonical_ability_key, rank) | Hunter has permanently learned a wild or trainer rank; source + timestamp |
| `naxhp_pet_identity` | (owner_guid, native_pet_number) | Original NPC entry, family, native identity, original speed, creation metadata |
| `naxhp_pet_progress` | (owner_guid, native_pet_number) | Loyalty rank, points, XP requirement, elapsed-time remainder, last update, schema version |
| `naxhp_pet_trained_rank` | (owner_guid, native_pet_number, canonical_ability_key) | Current rank per ability; TP cost metadata |
| `naxhp_operation_journal` | (owner_guid, nonce) or generated op ID | Train/respec/knowledge transaction idempotence and audit for recovery |
| `naxhp_owner_schema_state` | owner_guid | Migration status: untouched, eligible, pending, migrated, rolled back, blocked |

**Use real persistent native pet number** after verifying which field survives native stable operations. Do *not* use runtime object GUID for long-lived identity or assume pet name is unique. Every table is character-DB-owned and indexed for hot-path lookup.

## 2. Ownership and invariants

- `owner_guid` is a Hunter character GUID; no account-global catalogue unless expressly approved.
- `native_pet_number` must be verified to belong to `owner_guid` before any mutation.
- `source_creature_entry` records original creature identity at tame time. Do not overwrite on login with current family template.
- `canonical_ability_key` is internal stable string/small ID independent of spell ID or localized display text.
- Ability `rank` validated against Vanilla catalogue and server/client Spell.dbc mapping.
- No duplicate trained rank for same ability; rank replacement must atomically replace old rank.
- Remaining points computed as `level × (loyalty − 1) − sum(current trained cost)`; allow explicitly historic negative value rather than signed→unsigned wrap.
- Every mutable record carries `schema_version` or a central version and optional optimistic lock `revision`.
- Pet spell list and training ledger must agree after checkpoint; mismatches create audit warning instead of blind mass mutation.

## 3. Transaction outline (proposal)

```text
train(owner, nativePetId, ability, newRank, nonce):
  load current pet and owner, authoritative IP era; assert eligibility
  reject stale pet instance and stale revision
  lookup checked rule from versioned mapping dataset
  reject wrong family/level/unknown Hunter rank/active limit
  compute newRemainingTP with signed arithmetic
  validate ability grant/remove API and potential failures
  begin character DB transaction if actual core interfaces support it
    idempotence guard for (owner, nonce)
    update module training ledger + journal / revision
    persist any supported native pet state through the documented core API
  commit only after consistency checks
  refresh client action bar/spells in valid engine order
  emit event once
```

**Important:** A module transaction and AzerothCore internal pet saving are not necessarily one database transaction. **The developer must investigate actual save lifecycle**, and if atomic native+module updates cannot be achieved, must propose a rigorous recovery protocol or mark as blocked. Pseudocode is not proof that these operations are available.

## 4. Skill-acquisition event design

Suggested event inputs:
`pet_number, owner_guid, source_creature_entry, cast_spell_id, cast_stage, occurred_at, server_build, acquisition_policy_version`.

- Only report a successful cast **after its completion/effect as applicable**; no grant on unsuccessful attempted casts.
- Map installed spell IDs to canonical ability+rank with versioned lookup.
- Verify ability existed as a natural tame source or approved trainer rank; reject arbitrary AI-injected spell.
- If the Hunter already knows the rank, no money/TP side effect.
- If learning probability is retained from documented Vanilla reference, source and random decision must be **auditable/testable** without exposing a combat spam log.
- Notify Hunter and refresh appropriate UI only after durable recording.

## 5. Legacy pet migration plan

**Never** convert all existing Hunter pets without explicit owner approval. Staged phases:

1. Enumerate existing pets and snapshot `character_pet`, `pet_spell`, `pet_spell_cooldown`, character talent/known spells, all stable slot indices and active pet GUIDs; use deployed schema as truth.
2. Produce **read-only migration preview** per Hunter: can map source entry? extra modern abilities? potential loss of pet talent UI? full stable? conflicting IP stage?
3. Safe dispositions: `skip`, `opt-in_convert`, `needs_manual_review` (exact choice requires owner decision). A pet without reliable historical source identity must **never** be assigned one by guessing its appearance.
4. Preserve a reversible per-pet snapshot and provenance; do not delete legacy learned spells on migration before approved design and proven backup restore.
5. Convert a disposable character first; validate full stable/relog/restore and no future spell regrant.
6. Consider modern→Vanilla→modern non-destructive cycles and the owner's future TBC migration. If impossible, explicitly block multi-era conversion.

## 6. Versioned data contract

Reference datasets (future production):
- Family profile, diet and allowed abilities
- Ability/rank training catalogue and TP costs
- Wild source creature/rank list
- Original creature attack speed
- Classic→deployed spell ID map
- IP stage/feature availability policy (from installed public API)

Every table carries a `dataset_version`, source URL and patch/version. Runtime must validate referential integrity at startup and refuse to enable if incompatible with loaded DBC or database.

## 7. Configuration and operational controls

All names below are **proposed**:
```ini
NaxxHunterPets.Enabled = 0
NaxxHunterPets.VanillaOnly = 1
NaxxHunterPets.IncludeAltBots = 0
NaxxHunterPets.IncludeRandomBots = 0
NaxxHunterPets.ExperimentalIrreversibleDesertion = 0
NaxxHunterPets.MigrationMode = 0
NaxxHunterPets.AuditLogging = 0
```

Potential GM *read-only* diagnostics (not yet implemented):
```text
.naxhp inspect <player> [pet_number]
.naxhp catalogue <player>
.naxhp validate <player> [pet_number]
.naxhp migrate preview <player>
```

Do not install raw GM commands which bypass ownership, TP or world/IP gates. Mutating repair/debug tools would need stronger access restrictions and audit.

## 8. Error handling and observability

- Mismatched source/owner; unknown IP era; stale revision; bad rank mapping; native DB save mismatch; duplicate cast event: **fail safe**.
- Rate-limit warnings and remove GUID/name/character data from public logs as appropriate.
- Useful metrics: eligible Hunters/pets; valid learnt skill events; rejected train attempts by reason; skills mismatches; safe rollbacks; latency per handler; auto-learn bypass detections; bot pet-AI mismatches.
- Write failure must not become a silent success in client UI.
- A disabled feature must not execute hidden migration or uninstall handlers that cause loss of native pet behaviour.

## 9. Data rollback and transfer into Naxx Core

Integrate *ownership of tables* with Naxx Core, not just C++ calls. Plan table namespace, migration version monotonicity, data/read-path compatibility, and disable the standalone module before enabling the merged module. Roll back only after a verified restore and confirmation that neither implementation will silently teach/strip spells. Preserve source database backups.
