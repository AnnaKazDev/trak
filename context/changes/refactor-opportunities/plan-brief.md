# Plugin System Refactor Opportunities — Plan Brief

> Full plan: `context/changes/refactor-opportunities/plan.md`
> Research: `context/changes/refactor-opportunities/research.md`

## What & Why

Implement the top ranked structural fixes from the plugin-system debt analysis: wire RPC versioning so breaking catalog changes don’t nuke every plugin, break the accidental `plugins.service ↔ create-rpc-host` cycle, and prove a strangler extract of trip HostDeps wiring so the 928-LOC god module can shrink safely later.

## Starting Point

Flat envelopes with no version field; one mutual import on the settings/budget edge; `create-rpc-host` still owns all domain wiring (unit tests exist; global depcruise `no-circular` is warn-only). Dual-DB and supervisor shape stay as intentional architecture.

## Desired End State

Optional request `version` + manifest `rpcVersion` with host max and hard-fail on activate when the plugin asks for more than the host supports (missing = v1). Cycle gone and CI-guarded under `server/src/nest/plugins`. Trip capabilities composed from `host/wiring/trip-capabilities.ts` with smoke coverage.

## Key Decisions Made

| Decision | Choice | Why (1 sentence) | Source |
| -------- | ------ | ---------------- | ------ |
| Plan scope | Versioning + cycle break + first trip strangler | Full top-3 XL split deferred; prove pattern | Plan |
| Version declaration | Envelope `version?` + manifest `rpcVersion` | Ecosystem opt-in + wire default v1 | Plan |
| Mismatch policy | Hard-fail activate if rpcVersion > host max; omit = v1 | Clear errors; old plugins safe | Plan |
| Compat window | N and N−1 ready | Migration window without N−2 cost | Plan |
| God-module depth | Smoke + extract trip wiring only | Validates strangler without XL risk | Plan |
| Test strategy | Targeted trip smoke + keep existing suites | Not full 110-method gate | Plan |
| Depcruise | Path-scoped `error` under plugins + CI | Global error blocked by ~64 other cycles | Plan (refines Research) |
| Rollback | Additive defaults + git revert | No feature-flag debt | Plan |
| Ranking order | Version → cycle → god extract | Research ROI / ecosystem first | Research |

## Scope

**In scope:** Protocol versioning framework; settings extract + plugins depcruise CI; trip smoke tests; `wiring/trip-capabilities.ts`.

**Out of scope:** Full domain split; DI; flat supervisor; dual-DB change; global depcruise error; real v2 handlers; TS strict mode.

## Architecture / Approach

Additive envelope + activate gate first. Extract `readUserSettingDecrypted` to a third module to make the import graph acyclic. Raise circular detection to error only under `server/src/nest/plugins`. Then smoke the trip RPC surface and move that HostDeps wiring into `host/wiring/`, leaving `create-rpc-host` as composer for that domain.

## Phases at a Glance

| Phase | What it delivers | Key risk |
| ----- | ---------------- | -------- |
| 1. Protocol versioning | `version?`, `rpcVersion`, host max, activate fail | Persistence/discovery seam for activate read |
| 2. Cycle + depcruise CI | Acyclic plugins graph; CI guard | tsconfig.depcruise / path rule tuning |
| 3. Trip smoke tests | Regression net for extract | Choosing minimal method set |
| 4. Trip wiring extract | `wiring/trip-capabilities.ts` composed in | Behavior drift on HostDeps merge |

**Prerequisites:** Research verified; decisions locked in planning session.
**Estimated effort:** ~1.5–2.5 weeks across 4 phases (versioning + cycle quick; smoke + extract medium).

## Open Risks & Assumptions

- Manifest `rpcVersion` may need a DB column if activate only reads SQL fields — confirm discovery shape during Phase 1.
- Path-scoped depcruise must not false-positive on intentional type-only or test paths.
- Existing create-rpc-host unit coverage is better than research assumed; smokes still required before extract.

## Success Criteria (Summary)

- Old plugins (no version fields) still activate and RPC as v1.
- Over-max `rpcVersion` fails activate with a stable code.
- Plugins tree has no service↔host cycle; CI enforces it.
- Trip wiring lives in `wiring/`; smokes and unit tests green.
