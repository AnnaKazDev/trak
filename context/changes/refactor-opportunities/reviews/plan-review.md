<!-- PLAN-REVIEW-REPORT -->
# Plan Review: Plugin System Refactor Opportunities

- **Plan**: context/changes/refactor-opportunities/plan.md
- **Mode**: Deep
- **Date**: 2026-07-26
- **Verdict**: REVISE
- **Findings**: 1 critical 4 warnings 1 observation

## Verdicts

| Dimension | Verdict |
|-----------|---------|
| End-State Alignment | PASS |
| Lean Execution | WARNING |
| Architectural Fitness | PASS |
| Blind Spots | WARNING |
| Plan Completeness | FAIL |

## Grounding

12/12 existing paths ✓, tsconfig.depcruise.json missing (acknowledged in plan), 5/5 symbols ✓, brief↔plan ✓

## Findings

### F1 — Progress Phase 4 name mismatches body heading

- **Severity**: ❌ CRITICAL
- **Impact**: 🏃 LOW — quick decision; fix is obvious and narrowly scoped
- **Dimension**: Plan Completeness
- **Location**: ## Phase 4 vs ## Progress → ### Phase 4
- **Detail**: Body heading is `## Phase 4: Extract `wiring/trip-capabilities.ts`` (with backticks). Progress is `### Phase 4: Extract wiring/trip-capabilities.ts` (no backticks). /10x-implement matches phase names between body and Progress; this breaks the Progress↔Phase contract.
- **Fix**: Align both headings to the same string (prefer without nested backticks): `Phase 4: Extract wiring/trip-capabilities.ts`.
- **Decision**: PENDING

### F2 — rpcVersion persistence path left open for activate

- **Severity**: ⚠️ WARNING
- **Impact**: 🔎 MEDIUM — real tradeoff; pause to reason through it
- **Dimension**: Blind Spots
- **Location**: Phase 1 §4 Manifest rpcVersion; Critical Implementation Details
- **Detail**: `assertActivatable` reads SQL columns only (`permissions`, `granted_permissions`, `dependencies`, `trek_range`) — not on-disk manifest and not a full-manifest blob. `api_version` is persisted in discovery but unused at activate. Plan says “mirror apiVersion persistence only if activate reads from DB; otherwise parse from stored manifest JSON” — there is no stored manifest JSON. Brief flags this as an open risk; without a locked choice, Phase 1 can stall mid-implement.
- **Fix A ⭐ Recommended**: Persist `rpc_version` (default 1) in discovery upsert, mirroring `api_version`; SELECT it in `assertActivatable`.
  - Strength: Matches existing activate pattern (DB-only gate); no FS at activate.
  - Tradeoff: Schema/migration touch; discovery write path changes.
  - Confidence: HIGH — `api_version` / `trek_range` already follow this path.
  - Blind spot: Exact migration tooling for plugins table not re-verified here.
- **Fix B**: Re-parse `trek-plugin.json` from `pluginCodeDir(id)` inside assertActivatable.
  - Strength: No new column; version always matches on-disk package.
  - Tradeoff: New FS dependency on activate hot path; diverges from trek_range pattern.
  - Confidence: MEDIUM — install/dev-link already parse manifest elsewhere.
  - Blind spot: Missing/corrupt package at activate edge cases.
- **Decision**: PENDING

### F3 — Trip extract line range mixes non-trip HostDeps

- **Severity**: ⚠️ WARNING
- **Impact**: 🏃 LOW — quick decision; fix is obvious and narrowly scoped
- **Dimension**: Plan Completeness
- **Location**: Phase 4 §1; Current State Analysis
- **Detail**: Plan cites `create-rpc-host.ts` ~363–637 as trip wiring. Code shows members at 365–378, then notify/AI/settings/OAuth/scheduler/costs at 383–487, then trip-core places/days/itinerary/trips/accommodations at 488–637, plus `tripMembers` ~844. Phase 4 contract text is correct; the line range will mislead a mechanical extract.
- **Fix**: Replace “~363–637” with explicit ranges: members 365–378, trip-core 488–637, tripMembers ~844–845. Keep canEditTripAs in create-rpc-host (already preferred).
- **Decision**: PENDING

### F4 — plugin-sdk does not construct wire envelopes

- **Severity**: ⚠️ WARNING
- **Impact**: 🏃 LOW — quick decision; fix is obvious and narrowly scoped
- **Dimension**: Lean Execution
- **Location**: Phase 1 §3 Child transport
- **Detail**: Plan says update `plugin-host-entry.ts` “and published plugin-sdk transport mirror if it constructs the same shape.” Published `plugin-sdk/` has no `k:'req'` / `RpcRequest` construction; child transport builds envelopes only in `runtime/plugin-host-entry.ts` (~42–48). Supervisor also builds host→child reqs. The SDK clause invites unnecessary scope.
- **Fix**: Scope Phase 1 transport to `plugin-host-entry.ts` (+ supervisor only if host→child version is required). Drop published plugin-sdk from this phase.
- **Decision**: PENDING

### F5 — Trip smoke method set not enumerated

- **Severity**: ⚠️ WARNING
- **Impact**: 🏃 LOW — quick decision; fix is obvious and narrowly scoped
- **Dimension**: Plan Completeness
- **Location**: Phase 3 §1
- **Detail**: Phase 3 defers the method list to “research / HostDeps trip cluster.” Existing `create-rpc-host.test.ts` already covers most write paths (places/days/itinerary/accommodations/create/update/members) but not `trips.getById`, `trips.getPlaces`, or `trips.removeMember`. Without an explicit smoke list, Phase 3 can either duplicate wave tests or miss the extract-critical reads.
- **Fix**: List the smoke methods explicitly, including at least getById, getPlaces, removeMember plus one write each for places/days/itinerary/accommodations.
- **Decision**: PENDING

### F6 — safeParse is shared with settings helpers that stay

- **Severity**: ℹ️ OBSERVATION
- **Impact**: 🏃 LOW — quick decision; fix is obvious and narrowly scoped
- **Dimension**: Blind Spots
- **Location**: Phase 2 §1
- **Detail**: `readUserSettingDecrypted` uses private `safeParse`, also used by `readUserSettingsDecrypted` / config methods that remain in `plugins.service`. Moving the singular reader without duplicating or extracting `safeParse` breaks the service. Plan hedges (“any private helpers… e.g. safeParse”) but doesn’t pick duplicate vs shared helper module.
- **Fix**: State explicitly: extract `safeParse` (+ decrypt path deps) into the new settings-read module, or duplicate the tiny helper; leave plural readers importing from the shared module if extracted.
- **Decision**: PENDING
