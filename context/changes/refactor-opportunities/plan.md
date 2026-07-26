# Plugin System Refactor Opportunities — Implementation Plan

## Overview

Turn the verified ranking in `research.md` into incremental, reversible code changes: scaffold wire RPC versioning (N / N−1 ready), break the `plugins.service ↔ create-rpc-host` cycle and lock it with path-scoped dependency-cruiser in CI, then prove the god-module strangler with trip-domain smoke tests and a first `wiring/trip-capabilities.ts` extraction.

## Current State Analysis

- Flat wire protocol: `RpcRequest` has `k`, `id`, `method`, `params` only (`protocol/envelope.ts:19-24`). No envelope version; host does not compare `PLUGIN_API_VERSION` / manifest `apiVersion` at activate.
- Circular import: `plugins.service.ts:8` → `pluginBudgetUsage`; `create-rpc-host.ts:17` → `readUserSettingDecrypted` (call at `:424`). Verified.
- `create-rpc-host.ts` is 928 LOC / 41 imports; trip HostDeps wiring roughly L363–637 (+ `tripMembers` ~844). Unit tests already exist in `server/tests/unit/plugins/create-rpc-host.test.ts` (wave suites); integration coverage in `plugin-runtime.test.ts` and supervisor tests.
- `.dependency-cruiser.js` already has global `no-circular` at **warn**. ~64 cycles exist repo-wide — global `error` is unsafe. Plugins pair is the one Nest-plugins cycle to eliminate.
- Dual DB, flat supervisor, and full DI are out of scope (research: correct / defer).

### Key Discoveries

- Activate hard-fail seam: `assertActivatable` in `plugin-runtime.service.ts` (mirror `TREK_VERSION_INCOMPATIBLE`).
- Child constructs envelopes in `runtime/plugin-host-entry.ts` (~42–48); host dispatches in `plugin-supervisor.ts` → `rpcHost.dispatch`.
- `METHOD_PERMISSION` is audit-oriented; runtime routing is grant-map in `rpc-host.ts`.
- Cycle breaks by moving `readUserSettingDecrypted` alone (one remaining one-way import is fine).
- Path-scoped depcruise `error` under `server/src/nest/plugins/**` is the implementable form of the agreed “raise to error + CI” decision.

## Desired End State

1. Plugins without `rpcVersion` / request `version` keep working as v1. Host advertises a max RPC version; activate fails clearly if manifest `rpcVersion` > host max. Envelope may carry optional `version`; missing = v1. Framework ready for N−1 handlers later (no v2 methods required in this change).
2. No circular import between `plugins.service` and `create-rpc-host`. CI fails if a cycle reappears under the plugins tree.
3. Trip HostDeps wiring lives in `host/wiring/trip-capabilities.ts`; `create-rpc-host` composes it. Trip smoke tests + existing unit/integration suites stay green.

### How to verify

- Automated: server typecheck, lint, targeted vitest suites, depcruise on plugins path.
- Manual: activate an old-style plugin (no `rpcVersion`); confirm activate fails with a clear code when `rpcVersion` is artificially high; spot-check one trip RPC after the extract.

## What We're NOT Doing

- Full god-module split (cost / addon / broker domains)
- Nest DI for RPC host / supervisor
- Flat supervisor hierarchy
- Changing dual-database isolation
- Raising global `no-circular` to error for the whole repo
- Implementing actual v2 method handlers or a 6-month deprecation campaign
- Enabling TypeScript `strict` mode
- Feature-flagging version checks (rollback = git revert; additive defaults)

## Implementation Approach

Sequence matches research ROI and locked decisions (scope B, version src B, mismatch B, compat N/N−1, god split B, tests B, depcruise path-scoped error, rollback additive/revert):

1. Versioning framework first (prevents further ecosystem breakage; additive).
2. Cycle break + CI guard (quick win; unblocks cleaner host edits).
3. Trip smoke net, then strangler extract (prove pattern without XL full split).

## Critical Implementation Details

- **Activate vs RPC mismatch:** Manifest `rpcVersion` > host max → fail in `assertActivatable` with a dedicated code (e.g. `RPC_VERSION_INCOMPATIBLE`), not a vague `HOST_ERROR`. Per-request unknown/future version after activate → `HOST_ERROR` (or explicit refusal) on dispatch; requests omitting `version` must behave as v1.
- **Depcruise:** Keep global `no-circular` as `warn`. Add a second forbidden rule (or path-constrained rule) at `error` for `server/src/nest/plugins`. Fix or replace missing `tsconfig.depcruise.json` reference so the script runs in CI.
- **Extract order:** Land smoke tests (Phase 3) before moving wiring (Phase 4). Do not extract trip wiring without the smoke net green on mainline tests.

## Phase 1: Protocol Versioning Framework

### Overview

Add optional wire `version`, manifest `rpcVersion`, host max constant, activate hard-fail, and default-v1 behavior — without introducing v2 method implementations.

### Changes Required

#### 1. Envelope types

**File**: `server/src/nest/plugins/protocol/envelope.ts`

**Intent**: Allow optional per-request RPC catalog version on the wire; keep pure types / no runtime imports.

**Contract**: `RpcRequest` gains optional `version?: number`. Document that omitted means v1. Do not add version to response/event unless needed for symmetry (prefer request-only).

#### 2. Host max + dispatch default

**Files**: `server/src/nest/plugins/host/rpc-host.ts` (and/or a tiny `protocol/rpc-version.ts` if a shared constant is cleaner)

**Intent**: Centralize `HOST_RPC_VERSION` (or equivalent) = 1; treat missing request version as 1; refuse request versions the host cannot serve (today: anything other than 1, until N−1 exists).

**Contract**: Dispatch path that receives `RpcRequest` applies defaulting before method lookup. No v2 handler map required yet — structure comments/extension point so a future N−1 dual-handler is additive.

#### 3. Child transport

**File**: `server/src/nest/plugins/runtime/plugin-host-entry.ts` (and published `plugin-sdk` transport mirror if it constructs the same shape)

**Intent**: When the SDK/context knows an RPC version, include it on outbound `k:'req'` envelopes; omit or send `1` for current plugins so behavior is unchanged.

**Contract**: Envelope construction site (~req object with `k/id/method/params`) may add `version`. Host→child invokes from supervisor can omit version (host-originated) unless a clear need appears.

#### 4. Manifest `rpcVersion`

**Files**: `server/src/nest/plugins/install/manifest.ts`, `plugin-sdk/src/manifest.ts`, discovery persistence if `api_version` is stored and an analogous column is required for activate checks

**Intent**: Plugins may declare `rpcVersion` (number, default 1 when absent). Persist enough for `assertActivatable` to read without re-parsing the package every time — mirror `apiVersion` persistence pattern only if activate reads from DB; otherwise parse from stored manifest JSON if already available.

**Contract**: Validate type when present; default 1. Activate compares declared version to host max.

#### 5. Activate hard-fail

**File**: `server/src/nest/plugins/plugin-runtime.service.ts`

**Intent**: Refuse activation when plugin `rpcVersion` > host max, with a stable error code for admins/CI.

**Contract**: Extend `assertActivatable` (alongside trek range checks). Error shape mirrors existing dependency/version errors (`PluginDependencyError`-style code string, e.g. `RPC_VERSION_INCOMPATIBLE`).

#### 6. Tests

**Files**: unit/integration under `server/tests/**/plugins/`

**Intent**: Lock default-v1 and activate mismatch behavior.

**Contract**: (1) Request without `version` still dispatches. (2) Activate with `rpcVersion` > host max fails with the new code. (3) `rpcVersion` omitted or `1` still activates (fixture permitting).

### Success Criteria

#### Automated Verification

- Server typecheck passes: `npm run typecheck` (in `server/`)
- Plugin-related unit/integration tests for versioning pass: `npm test --` filtered to the new/updated suites
- Lint passes for touched files: `npm run lint:check` (in `server/`)

#### Manual Verification

- Confirm error message/code for over-max `rpcVersion` is actionable (read test output or local activate attempt)
- Confirm an existing test plugin / fixture without `rpcVersion` still activates in the usual integration path

**Implementation Note**: After automated verification passes, pause for manual confirmation before Phase 2.

---

## Phase 2: Circular Dependency Break + Depcruise CI

### Overview

Extract shared settings read helper to break the cycle; enforce no new cycles under the plugins tree via dependency-cruiser in CI.

### Changes Required

#### 1. Extract settings helper

**Files**: new `server/src/nest/plugins/plugin-settings.interface.ts` (name may be `plugin-user-settings-read.ts` if “interface” is misleading — pick one and stay consistent), `plugins.service.ts`, `create-rpc-host.ts`

**Intent**: Move `readUserSettingDecrypted` (and any private helpers it needs, e.g. `safeParse`) out of `plugins.service` so `create-rpc-host` no longer imports `../plugins.service`.

**Contract**: Same function signature: `(pluginId, userId, key) => unknown`. `plugins.service` may re-export for backward compatibility or update call sites; `create-rpc-host` imports from the new module. Update mocks in `create-rpc-host.test.ts` and imports in `settings-isolation.test.ts` / `plugin-user-settings.test.ts`.

#### 2. Path-scoped no-circular error

**File**: `.dependency-cruiser.js`

**Intent**: Fail CI on circular deps inside the plugins Nest tree without failing the whole monorepo.

**Contract**: Global `no-circular` stays `warn`. Add rule at `error` constrained to `server/src/nest/plugins` (from/to path). Ensure config’s `tsConfig` points at an existing file (add `tsconfig.depcruise.json` or retarget).

#### 3. Script + CI

**Files**: root `package.json`, `.github/workflows/test.yml` (server-tests job)

**Intent**: Make the check runnable locally and on every PR.

**Contract**: Script e.g. `depcruise:plugins` running depcruise against `server/src/nest/plugins`. CI step after server lint/typecheck. Must pass once the cycle is gone.

### Success Criteria

#### Automated Verification

- `npm run depcruise:plugins` (or chosen script name) exits 0
- Server typecheck + `create-rpc-host` / settings unit tests pass
- No remaining import of `../plugins.service` from `create-rpc-host.ts` for settings

#### Manual Verification

- Spot-check: `rg` / depcruise output shows no circular pair between `plugins.service` and `create-rpc-host`

**Implementation Note**: Pause for manual confirmation before Phase 3.

---

## Phase 3: Trip-Domain Smoke Tests

### Overview

Add a focused smoke suite for trip capability RPCs so Phase 4 extraction has a regression net beyond existing wave tests.

### Changes Required

#### 1. Smoke suite

**File**: new or extended test under `server/tests/` (prefer unit host wiring style alongside `create-rpc-host.test.ts`, or a thin integration slice — choose the layer that can call the same `createRealRpcHost` + grants pattern already used)

**Intent**: Cover a minimal trip-domain set: trips read/write, places, days, itinerary assign/unassign, accommodations (methods listed in research / HostDeps trip cluster). Skip reservations as a separate domain unless already trivial in the same helper.

**Contract**: Each smoke asserts granted path returns without throw / expected shape for the fixture trip. Ungranted path remains permission-denied where the existing harness already checks that pattern. Do not attempt all 110 `KNOWN_METHODS`.

### Success Criteria

#### Automated Verification

- New smoke suite passes: `npm test --` filtered to the smoke file
- Existing `create-rpc-host.test.ts` still passes

#### Manual Verification

- Review smoke list vs HostDeps trip keys — no accidental omission of places/days/itinerary/accommodations/trips core

**Implementation Note**: Pause for manual confirmation before Phase 4.

---

## Phase 4: Extract `wiring/trip-capabilities.ts`

### Overview

Move trip-related HostDeps wiring into a dedicated module and compose it from `createRealRpcHost`.

### Changes Required

#### 1. New wiring module

**File**: `server/src/nest/plugins/host/wiring/trip-capabilities.ts`

**Intent**: Own construction of trip-domain HostDeps fields (members, places, days, itinerary, trip create/update, hydrated trip reads used by trips.*, accommodations, `tripMembers`) currently inline in `create-rpc-host.ts` (~363–637, ~844).

**Contract**: Export a pure(ish) factory, e.g. `createTripCapabilities(depsCtx) => Partial<HostDeps>` or a named bag merged into the HostDeps object. No change to `rpc-host.ts` registration map in this phase unless a type import is required. Keep security helpers (`canEditTripAs`, etc.) either imported from existing module-level helpers or moved only if they are trip-exclusive and unused elsewhere — prefer leave shared gates in `create-rpc-host` if still shared.

#### 2. Compose in factory

**File**: `server/src/nest/plugins/host/create-rpc-host.ts`

**Intent**: Replace inlined trip wiring with spread/merge from the new module; file becomes thinner composition for that domain.

**Contract**: Behavior identical for trip RPCs. All Phase 3 smokes + existing create-rpc-host wave suites green.

### Success Criteria

#### Automated Verification

- Trip smoke + `create-rpc-host` unit tests + relevant `plugin-runtime` integration subset pass
- Server typecheck + lint pass
- `depcruise:plugins` still exits 0 (no new cycle via wiring imports)

#### Manual Verification

- Confirm `create-rpc-host.ts` no longer contains the moved trip write/read blocks (composition call only)
- Spot-check one trip RPC path in integration logs or a manual plugin invoke if available

**Implementation Note**: After this phase, the change is complete for the agreed scope. Further domains are a follow-up change.

---

## Testing Strategy

### Unit Tests

- Version defaulting and dispatch refusal for unsupported request versions
- Activate `rpcVersion` > host max
- Settings helper after extract (existing settings isolation tests updated)
- Trip smoke via `createRealRpcHost` grants

### Integration Tests

- Existing `plugin-runtime` activate/deactivate paths remain green
- Optional: one runtime test that installs/activates a fixture with high `rpcVersion` and expects failure

### Manual Testing Steps

1. Activate fixture plugin without `rpcVersion` — success
2. Temporarily set `rpcVersion` above host max — activate fails with `RPC_VERSION_INCOMPATIBLE` (or chosen code)
3. After Phase 4, exercise `trips.getById` / `places.create` via existing test harness

## Performance Considerations

None material — version field is optional metadata; wiring extract is organizational.

## Migration Notes

- Additive protocol: old plugins need no changes.
- No data migration required unless persisting `rpc_version` column; if added, default existing rows to `1`.
- Rollback: git revert; omitted `version` / `rpcVersion` keeps v1 behavior.

## References

- Research: `context/changes/refactor-opportunities/research.md`
- Baseline debt: `context/changes/plugin-service/research.md`
- Blast radius: `context/changes/plugin-service/blast-radius-analysis.md`
- Envelope: `server/src/nest/plugins/protocol/envelope.ts`
- Host factory: `server/src/nest/plugins/host/create-rpc-host.ts`
- Activate gate: `server/src/nest/plugins/plugin-runtime.service.ts`
- Depcruise: `.dependency-cruiser.js`

## Progress

> Convention: `- [ ]` pending, `- [x]` done. Append ` — <commit sha>` when a step lands. Do not rename step titles. See `references/progress-format.md`.

### Phase 1: Protocol Versioning Framework

#### Automated

- [ ] 1.1 Server typecheck passes
- [ ] 1.2 Versioning unit/integration tests pass
- [ ] 1.3 Server lint passes for touched files

#### Manual

- [ ] 1.4 Over-max rpcVersion error is actionable
- [ ] 1.5 Fixture without rpcVersion still activates

### Phase 2: Circular Dependency Break + Depcruise CI

#### Automated

- [ ] 2.1 depcruise:plugins exits 0
- [ ] 2.2 Server typecheck + settings/create-rpc-host unit tests pass
- [ ] 2.3 create-rpc-host no longer imports plugins.service for settings

#### Manual

- [ ] 2.4 Confirm no circular pair plugins.service ↔ create-rpc-host

### Phase 3: Trip-Domain Smoke Tests

#### Automated

- [ ] 3.1 Trip smoke suite passes
- [ ] 3.2 Existing create-rpc-host.test.ts still passes

#### Manual

- [ ] 3.3 Smoke list covers trips/places/days/itinerary/accommodations core

### Phase 4: Extract wiring/trip-capabilities.ts

#### Automated

- [ ] 4.1 Trip smoke + create-rpc-host unit + relevant runtime tests pass
- [ ] 4.2 Server typecheck + lint pass
- [ ] 4.3 depcruise:plugins still exits 0

#### Manual

- [ ] 4.4 Trip wiring blocks moved; create-rpc-host composes only
- [ ] 4.5 Spot-check one trip RPC path after extract
