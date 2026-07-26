---
date: 2026-07-26T14:41:50+0000
researcher: AI Agent
git_commit: c439b93
branch: dev_metrics
repository: trek
topic: "Plugin System - Refactor Opportunities Analysis"
tags: [research, refactoring, technical-debt, plugin-system, verified]
status: complete
last_updated: 2026-07-26
last_updated_by: AI Agent
verification_commit: c439b93
---

# Research: Plugin System - Refactor Opportunities Analysis

**Date**: 2026-07-26T14:41:50+0000  
**Researcher**: AI Agent  
**Git Commit**: `c439b93`  
**Branch**: `dev_metrics`  
**Repository**: trek (TREK Travel Planner)

## Research Question

Given the technical debt and structural risks documented in `context/changes/plugin-service/research.md`, which problems warrant refactoring, in what target shape, and in what order?

This analysis explores each documented problem in code and history, then ranks refactor opportunities by cost of debt vs cost of change, with concrete first-step prerequisites.

## Summary

**Scope**: Evaluated 12 documented issues from plugin-service research. **6 are structural refactor candidates**, 5 are test/process gaps, and 1 is correct architecture.

### Top 3 Refactor Opportunities (Ranked)

1. **Protocol Versioning** (Risk: LOW, Effort: M, Impact: HIGH)
   - Enable breaking changes without breaking all plugins
   - Prerequisite: Version negotiation policy (1-2 days)
   - First step: Add `version?: number` to RpcRequest interface

2. **Circular Dependency** (Risk: LOW, Effort: S, Impact: MEDIUM)
   - Break `plugins.service ↔ create-rpc-host` cycle
   - Reduces merge conflicts from 2.4/sprint to ~0
   - Prerequisite: Circular dependency linter (0.5 day)
   - First step: Extract `plugin-settings.interface.ts`

3. **God Module Split** (Risk: HIGH, Effort: XL, Impact: HIGH)
   - Split 928-LOC `create-rpc-host.ts` by capability domain
   - 8x cost multiplier on service changes → ~2x
   - Prerequisite: Integration tests for RPC surface (2-3 days)
   - First step: Extract `wiring/trip-capabilities.ts` (20-30% of module)

### Rejected Candidates

- **Dual Database Model**: CORRECT architecture for security isolation (no change needed)
- **Flat Supervisor**: LOW urgency, unclear benefit vs cost
- **No Dependency Injection**: Blocked by god module split (do that first)

### Evidence Quality

All findings based on:
- [Current code shape analysis](d7670384-81a6-4da6-8918-dcdf9bb957c0) - file:line citations from source
- [Git archaeology](166ed3b2-d5cc-4d95-8786-b954bb10448e) - intentionality verdicts
- [Migration feasibility analysis](0c75236e-f5a1-4e3b-9ec9-b6bd962f7582) - blast radius, safety nets, prerequisites
- Source: `context/changes/plugin-service/research.md` (technical debt baseline)
- Source: `context/changes/plugin-service/blast-radius-analysis.md` (co-change patterns)

---

## Problem Classification

From plugin-service research.md, all 12 documented issues classified:

### CANDIDATES (Structural Code Changes)

| # | Problem | Current State | Intentionality | Priority |
|---|---------|---------------|----------------|----------|
| 1 | **Circular Dependency** | `plugins.service.ts:8` ↔ `create-rpc-host.ts:17` | ACCIDENTAL | HIGH |
| 2 | **God Module** | 41 imports, 928 LOC, 25-27 services | ACCIDENTAL | HIGH |
| 3 | **No Dependency Injection** | Direct service imports, manual `new` | ACCIDENTAL | MEDIUM |
| 4 | **No Protocol Versioning** | Flat `KNOWN_METHODS` array, no wire version | INTENTIONAL → DEBT | HIGH |
| 5 | **Flat Supervisor** | Single 781-LOC class, no hierarchy | UNKNOWN | LOW |
| 6 | **Dual Database Model** | `trek.db` + `plugin.db`, no JOIN | **INTENTIONAL (CORRECT)** | **N/A** |

### NOT CANDIDATES (Non-Structural Issues)

| # | Problem | Fix Type | Notes |
|---|---------|----------|-------|
| 7 | Test Gaps (Audit API, activity controller) | Add tests | Not refactor |
| 8 | Permission Gate Mocks | Improve assertions | Test quality |
| 9 | Audit Wiring Not Asserted | Add side-effect checks | Test coverage |
| 10 | Child Bootstrap Indirect Coverage | Add unit tests | Test isolation |
| 11 | Young System Instability (102 commits/1.5mo) | Time + maturity | Temporal |
| 12 | 2 IPC Hops Per Request | Performance characteristic | Trade-off |

**Verdict**: 5 actionable refactor candidates (excluding #6, which is correct).

---

## Candidate 1: Circular Dependency

### Current Shape

**Evidence from [code shape analysis](d7670384-81a6-4da6-8918-dcdf9bb957c0)**:

**Location:**
- `server/src/nest/plugins/plugins.service.ts:8` — imports `pluginBudgetUsage` from `./host/create-rpc-host`
- `server/src/nest/plugins/host/create-rpc-host.ts:17` — imports `readUserSettingDecrypted` from `../plugins.service`

**Responsibilities:**
- `PluginsService` (Nest service) provides admin API: list, configs, audit, budget views
- `create-rpc-host` (factory function) wires privileged HostDeps for live plugin RPC
- **Cycle edge A**: Admin budget view (`plugins.service.ts:287-288`) needs host budget counters from `pluginBudgetUsage`
- **Cycle edge B**: RPC `ctx.settings.get` (`create-rpc-host.ts:424`) needs decrypted user settings from `readUserSettingDecrypted`

**Existing Abstractions:**
- Both sides export standalone functions (not class methods)
- `readUserSettingDecrypted` explicitly marked "Standalone (no Nest DI)" (L339-340)
- Suggests authors knew this was awkward but chose expediency

**Evidence Quality:**
- [EVIDENCE] Mutual imports exist at L8 and L17
- [EVIDENCE] Runtime call sites close the cycle (L287-288, L424)
- [INFERENCE] Standalone functions chosen to allow cross-module use without Nest DI

### Intentionality

**Verdict**: **ACCIDENTAL COMPLEXITY** (Confidence: HIGH)

**Evidence from [git archaeology](166ed3b2-d5cc-4d95-8786-b954bb10448e)**:
- No ADR or design doc found justifying circular pattern
- Blast-radius-analysis.md calls this "fragile coupling that amplifies blast radius"
- Research.md documents pain: **2.4 merge conflicts per sprint** from co-changes
- Zero attempts in git history to break the cycle
- No inline comments defending the pattern

**Why accidental**:
- Circular imports for utility functions that could easily be extracted
- Pain metrics show this wasn't a deliberate trade-off
- Pattern emerged from "aggressive delivery timeline" (102 commits/1.5 months)

### Migration Feasibility

**Evidence from [feasibility analysis](0c75236e-f5a1-4e3b-9ec9-b6bd962f7582)**:

**Incremental Path** (Extract Interface):
1. Create `plugin-settings.interface.ts` with `readUserSettingDecrypted` function
2. Move shared logic to new file
3. Both modules import from interface, breaking the cycle
4. Run tests to verify no behavior change

**Reversibility**: ✅ YES - can revert by moving function back

**Blast Radius**: 3 files (atomic change)
- `plugins.service.ts` (remove export)
- `create-rpc-host.ts` (change import)
- `plugin-settings.interface.ts` (new)

**Safety Nets**:
- ✅ Unit tests: `plugins-service.test.ts` (275 lines, covers settings encryption)
- ✅ Integration: `plugin-runtime.test.ts` (720 lines, full activate/deactivate)
- ✅ CI: typecheck, lint, tests on every PR
- ❌ TypeScript strict mode OFF (`strict: false`)
- ❌ No circular dependency linter rule

**First Step Prerequisite**:
- Add architecture test to detect circular imports
- Tool: `dependency-cruiser` (already in devDependencies)
- Effort: **XS** (0.5 day)
- Reason: Without this, cycle can be reintroduced silently

### Cost-Benefit Analysis

**Cost of Debt**:
- 2.4 merge conflicts/sprint × 30 min resolution = **1.2 hours/sprint** lost per developer
- Hot reload races (file A saved → reload triggered → file B stale → runtime error)
- Tight coupling makes unit testing harder (need both modules)

**Cost of Change**:
- Effort: **S** (1 day)
- Risk: **LOW** (3-file change, well-tested)
- Prerequisite: 0.5 day (architecture test)
- **Total**: 1.5 days

**ROI**: HIGH - Quick win that unblocks future refactors and eliminates recurring pain.

---

## Candidate 2: God Module (`create-rpc-host.ts`)

### Current Shape

**Evidence from [code shape analysis](d7670384-81a6-4da6-8918-dcdf9bb957c0)**:

**Location:**
- `server/src/nest/plugins/host/create-rpc-host.ts` — **928 LOC**, **41 import statements** (L1-41)
- Factory entry: `createRealRpcHost` at L209-927
- Module comment claims uniqueness: "ONLY plugin file that imports db/websocket" (L145-149)

**Responsibilities** (single file mixes):
- Module-level helpers: packing privacy, `canEditTripAs`, addon gates (L49-137)
- Plugin DB handle cache: `getPluginDataDb` / `closePluginDataDb` (L152-172)
- Daily AI/notify budget seeding from audit table (L174-201)
- Entire `HostDeps` object wiring: domain reads/writes across trips, packing, files, collab, journal, atlas, vacay, costs, reservations, meta, OAuth, AI, notify, scheduler, inter-plugin calls (L209-927)
- Direct service imports: `../../../services/*`, Nest services (`BudgetService`, `ReservationsService`), LLM, websocket, crypto

**Existing Abstractions**:
- `PluginRpcHost` + `HostDeps` split: factory fills deps, router enforces grants
- Sibling helpers: `plugin-data.service.ts`, `daily-budget.ts`, `plugin-audit.ts`, `rate-limit.ts`
- Error mapping helpers: `ForbiddenResource`, `BadParams`, `mapCollectionError`

**Evidence Quality**:
- [EVIDENCE] 41 imports, 928 lines verified via wc
- [EVIDENCE] One function body (~L209-927) wires all host capabilities
- [INFERENCE] "God module" is the wiring layer; enforcement lives in `rpc-host.ts` (also 1332 LOC)
- [UNKNOWN] Why not split by domain (no comments describing planned splits)

### Intentionality

**Verdict**: **ACCIDENTAL COMPLEXITY** (Confidence: HIGH)

**Evidence from [git archaeology](166ed3b2-d5cc-4d95-8786-b954bb10448e)**:
- No ADR explaining monolithic RPC host choice
- Research.md labels this with "❌ God module (928 LOC), tight coupling"
- Blast-radius-analysis.md rates **RED (HIGH)** risk with "40+ service dependencies"
- Zero commits attempting to decompose this module
- Pattern suggests incremental growth: services added one-by-one as imports pile up

**Why accidental**:
- The 928-line file has comments explaining security gates (L43-55) but NOT why all services are in one file
- Absence of "why monolithic" justification in such a large file is damning
- Timeline pressure: "102 commits in 1.5 months" suggests shortcuts were taken

### Migration Feasibility

**Evidence from [feasibility analysis](0c75236e-f5a1-4e3b-9ec9-b6bd962f7582)**:

**Incremental Path** (Domain-Based Splitting via Strangler Fig):
1. Create `wiring/trip-capabilities.ts` (first domain)
2. Move trip-related service wiring (20-30% of create-rpc-host)
3. Import and compose in create-rpc-host
4. Repeat for other domains: budget, addons, notifications, files
5. Eventually, create-rpc-host becomes just a composition layer

**Reversibility**: ✅ YES - can inline functions back if pattern doesn't work

**Blast Radius**: Per migration cycle
- 1 new wiring file (e.g., `wiring/trip-capabilities.ts`)
- 1 existing `create-rpc-host.ts` (remove moved code, add import)
- 0 changes to `rpc-host.ts` (consumer)
- **2-3 files per cycle**

**Safety Nets**:
- ❌ **ZERO tests** for create-rpc-host.ts directly
- ✅ Integration coverage via plugin-runtime.test.ts (720 lines)
- Tests exercise RPC capabilities end-to-end but not wiring layer
- ✅ CI: typecheck, lint, tests
- ❌ TypeScript strict mode OFF
- ❌ No complexity metrics (cyclomatic, lines-per-function)

**First Step Prerequisite**:
- **Add integration test for RPC capability surface**
- Test that each KNOWN_METHOD works with minimal fixture
- Reason: Without this, refactoring breaks capabilities silently
- Effort: **L** (2-3 days to cover 110 methods)
- **Alternative**: Add smoke tests for each capability domain first (trips, budget, etc.)
- Alternative effort: **M** (1 day per domain)

### Cost-Benefit Analysis

**Cost of Debt**:
- Service signature change → **8x cost multiplier** (research.md L360-362)
  - Expected: 2 hours (update service + tests)
  - Actual: **16 hours** (service + create-rpc-host wiring + 8 affected controller tests + plugin SDK types)
- Example: `budgetService` signature change cascaded through 720 LOC of wiring
- Context loss: "No one understands God module wiring" (research.md L374)

**Cost of Change**:
- Effort: **XL** (2-3 weeks for full split)
- Risk: **HIGH** (large file, no direct tests)
- Prerequisite: 2-3 days (integration tests)
- **Total**: 4-5 weeks including tests and migration cycles

**ROI**: HIGH - Eliminates 8x cost multiplier, but requires substantial upfront investment. Should be done in phases (one domain at a time) to spread risk.

---

## Candidate 3: No Dependency Injection

### Current Shape

**Evidence from [code shape analysis](d7670384-81a6-4da6-8918-dcdf9bb957c0)**:

**Location:**
- Nest module providers: `plugins.module.ts:33` — only 4 services: `PluginsService`, `PluginRuntimeService`, `PluginRegistryService`, `PluginOAuthService`
- `@Injectable()` decorator only on those four
- `create-rpc-host.ts:1-41` — 41 direct imports
- Manual instantiation: `new BudgetService()` / `new ReservationsService()` (L59-60)
- Per-call `new PluginOAuthService()` at L426 (despite OAuth being a Nest provider)
- `new PluginSupervisor(...)` at `plugin-runtime.service.ts:133` (not Nest-injected)
- `new PluginDataDb(...)` in `getPluginDataDb` (L162)

**Responsibilities**:
- Nest DI covers admin/runtime/registry/oauth controllers ↔ services
- Live RPC host bypasses Nest: factory closes over imports and constructs collaborators itself
- Supervisor constructed manually, receives RPC-host factory callback

**Existing Abstractions**:
- Manual DI via `HostDeps` interface (`rpc-host.ts:46-260`) — constructor-injected into `PluginRpcHost`
- `PluginCallRouter` interface for inter-plugin calls
- `SupervisorHooks` / `SupervisorTuning` for supervisor collaboration
- Comment: "it has no injected deps" on `BudgetService` (L57-58)

**Evidence Quality**:
- [EVIDENCE] Exactly 4 Nest providers in module (L33)
- [EVIDENCE] 41 direct imports + multiple `new` instantiations
- [INFERENCE] `HostDeps` is intentional manual DI for child-isolation boundary
- [UNKNOWN] Whether all Nest services could be constructor-injected without breaking isolation

### Intentionality

**Verdict**: **ACCIDENTAL COMPLEXITY** (Confidence: HIGH)

**Evidence from [git archaeology](166ed3b2-d5cc-4d95-8786-b954bb10448e)**:
- No ADR explaining choice of manual imports over DI container
- Server uses NestJS (which has DI) - see `server/src/nest/README.md`
- Plugin system built OUTSIDE NestJS DI - manual instantiation throughout
- Research.md lines 624-627: "**3. Dependency injection (partial)** - **Problem**: 40+ direct imports instead of DI container - **Result**: God module, tight coupling"
- The word "Problem" and "Result: tight coupling" indicates this wasn't desired
- Other modules (weather, packing, budget) migrated TO NestJS DI, but plugins didn't follow

**Why accidental**:
- Timeline pressure: "102 commits in 1.5 months" suggests shortcuts
- The REST of the server IS adopting DI, making this an outlier
- No architectural justification comments

### Migration Feasibility

**Evidence from [feasibility analysis](0c75236e-f5a1-4e3b-9ec9-b6bd962f7582)**:

**Incremental Path** (Service Locator Pattern):
1. Create `ServiceContainer` interface
2. Move service instantiation to caller (plugin-runtime.service)
3. Pass container to createRpcHost instead of importing services
4. Gradually replace direct imports with container lookups

**Alternative**: Full Constructor Injection (requires updating all call sites - higher risk)

**Reversibility**: ✅ YES - can keep both patterns during transition

**Blast Radius**: Per service moved to DI
- `create-rpc-host.ts` (change import → container access)
- `plugin-runtime.service.ts` (provide service in container)
- Tests using that service (update fixtures)
- **3-5 files per service**

**Safety Nets**:
- ✅ Integration: plugin-runtime.test.ts covers full lifecycle
- ✅ Unit tests mock individual services
- ❌ No test for DI container itself
- ❌ Services not designed for injection (no interfaces)

**First Step Prerequisite**:
- **Define HostDeps interface fully**
- Document every service method used by RPC host
- Create TypeScript interface (not just inline type)
- Reason: Without clear contract, DI refactor becomes ad-hoc and fragile
- Effort: **M** (1 day to catalog 40+ service methods)

### Cost-Benefit Analysis

**Cost of Debt**:
- Testing difficulty: Hard to mock services in isolation
- Tight coupling: Service changes ripple through god module
- Onboarding friction: New developers confused by mix of Nest DI and manual instantiation

**Cost of Change**:
- Effort: **L** (1-2 weeks for incremental migration)
- Risk: **MEDIUM** (interface stability uncertain, system only 1 month old)
- Prerequisite: 1 day (HostDeps interface)
- **Total**: 2-3 weeks

**ROI**: MEDIUM - Enables better testing and loose coupling, but BLOCKED by god module split (should do that first to reduce blast radius).

**Recommendation**: DEFER until after god module split (#2). Splitting the god module will naturally create smaller surfaces to inject.

---

## Candidate 4: No Protocol Versioning

### Current Shape

**Evidence from [code shape analysis](d7670384-81a6-4da6-8918-dcdf9bb957c0)**:

**Location:**
- `server/src/nest/plugins/protocol/envelope.ts:1-8` — "PURE TYPES ONLY", dependency-free wire protocol
- Envelope kinds: `RpcRequest | RpcResponse | RpcError | RpcEvent` (L19-42)
- Fields: `k`, `id`, `method`, `params` / `ok` / `topic`
- **No `version` field**
- `KNOWN_METHODS` flat const — **110 methods** (L49-160)
- `METHOD_PERMISSION` maps method → permission (L164 (raport: 163-277))
- Separate: `PLUGIN_API_VERSION = 1` in SDK (`runtime/plugin-sdk.ts:12`), manifest `apiVersion` (L87, 217) — not on envelope

**Responsibilities**:
- Envelope defines message shapes + global method/permission catalogs
- Method set is single static list
- Adding method = append to `KNOWN_METHODS` + `METHOD_PERMISSION` + host handler registration
- No negotiate/handshake version in IPC envelope types

**Existing Abstractions**:
- Typed `Envelope` union and `PluginErrCode`
- `KnownMethod` derived from `KNOWN_METHODS`
- Permission catalog with prefix escape hatch (`http:outbound:`)
- SDK/manifest API versioning exists **separately** from wire envelope

**Evidence Quality**:
- [EVIDENCE] No `version` field on envelope interfaces (L19-42)
- [EVIDENCE] Single flat `KNOWN_METHODS` array of 110 strings
- [EVIDENCE] `PLUGIN_API_VERSION` exists outside envelope
- [INFERENCE] "No protocol versioning" refers to wire/RPC catalog, not plugin package semver
- [UNKNOWN] Whether child and host compare `PLUGIN_API_VERSION` at load time

### Intentionality

**Verdict**: **INTENTIONAL CONSTRAINT** (initially) → **ACCIDENTAL COMPLEXITY** (now) (Confidence: MEDIUM)

**Evidence from [git archaeology](166ed3b2-d5cc-4d95-8786-b954bb10448e)**:
- Research.md L680-682: "**Chosen: No protocol versioning** - ✅ Simple (one KNOWN_METHODS array) - ❌ Breaking changes break all plugins"
- The "✅ Simple" and "Chosen:" suggest deliberate MVP decision
- HOWEVER: No inline comments marking this as temporary technical debt
- Research.md L568: "Mitigation needed: Protocol versioning or compatibility shim"
- Appears to be "ship fast now, add versioning later" that never got the "later" part

**Why it became debt**:
- Research.md documents "3 breaking changes in 6 weeks" (L351)
- Young system (1.5 months) made 3 breaking changes → plugin adoption slows
- No migration path for plugins when methods change signature

### Migration Feasibility

**Evidence from [feasibility analysis](0c75236e-f5a1-4e3b-9ec9-b6bd962f7582)**:

**Incremental Path** (Additive Versioning):
1. Add optional `version?: number` to RpcRequest interface
2. Update rpc-host to check version field (default to 1)
3. When changing method, create v2 handler, keep v1
4. Plugin manifest declares `rpcVersion: 2` to opt in
5. Eventually deprecate v1 after migration window

**Reversibility**: ✅ YES - can remove versioning by always using latest

**Blast Radius**: Initial setup
- `protocol/envelope.ts` (add version field)
- `rpc-host.ts` (add version routing logic)
- `manifest.ts` (add rpcVersion to schema)
- Plugin SDK types (frontend)
- **4 files (one-time setup)**

**Per method change**: 1 file (rpc-host.ts - add new version handler)

**Safety Nets**:
- ✅ Integration: plugin-runtime.test.ts exercises RPC invoke path
- ❌ No tests for protocol versioning (doesn't exist yet)
- ❌ No compatibility test suite (old plugins on new host)
- ❌ No version mismatch detection tests

**First Step Prerequisite**:
- **Design version negotiation protocol**
- Define how plugin declares supported versions
- Define host's backward compatibility window
- Define deprecation policy (how long to support old versions)
- Reason: Without policy, versioning becomes maintenance burden
- Effort: **S** (1-2 days for design doc + review)

### Cost-Benefit Analysis

**Cost of Debt**:
- Research.md L351: "3 breaking changes in 6 weeks → plugin authors complain, adoption slows"
- Immediate plugin breakage on RPC signature change (no deprecation window)
- Marketplace trust erosion: plugins break without warning

**Cost of Change**:
- Effort: **M** (1 week for initial versioning framework)
- Risk: **LOW** (4-file change, stable interface)
- Prerequisite: 1-2 days (version policy)
- **Total**: 1.5-2 weeks

**ROI**: VERY HIGH - Unblocks future evolution without breaking existing plugins. Should be done BEFORE any more breaking changes.

**Recommendation**: **TOP PRIORITY**. Do this first to prevent further ecosystem damage.

---

## Candidate 5: Flat Supervisor

### Current Shape

**Evidence from [code shape analysis](d7670384-81a6-4da6-8918-dcdf9bb957c0)**:

**Location:**
- `server/src/nest/plugins/supervisor/plugin-supervisor.ts` — **781 LOC**, single class `PluginSupervisor` (L113)
- Instantiated in `plugin-runtime.service.ts:133` with factory + hooks
- `Supervised` state bag: child, rpcHost, routes, jobs, hooks, events, exports, subscriptions, pending RPC, rate limiters, activation/respawn timers (L42-68)

**Responsibilities** (one class):
- Public surface: `activate`, `disable`, `isActive`, `activeIds`, `statusOf`, `routesOf`, `providersOf`, `exportsOf`, `subscribersOf`, `deliverEvent`, `deliverScheduled`, `deliverUserErasure`, `collectUserExport`, `invoke`, `shutdownAll`, `reapStale`
- Internals: `spawn`, `onMessage` (RPC + lifecycle), `onExit` (crash/backoff), `recordLog`, event buffer/replay, heartbeat/RSS sweep, kill/grace
- Also owns: hook permission filtering (`HOOK_PERMISSION` L99-111), job scheduling via imported helpers, IPC envelope routing

**Existing Abstractions**:
- Constructor injection: `createRpcHost` factory + `SupervisorHooks` + `SupervisorTuning`
- Extracted helpers: `scheduleJobs` / `stopJobs` (`host/plugin-jobs.ts`), `RpcRateLimiter` / `TokenBucket` (`host/rate-limit.ts`), path/fork helpers (`paths.ts`)
- Types: `PluginStatus`, `PluginRouteInfo`, `Supervised` (private)

**Evidence Quality**:
- [EVIDENCE] One class file, 781 LOC, no subclass/strategy split
- [EVIDENCE] `Supervised` aggregates lifecycle + RPC + jobs + hooks + rate limits (L42-68)
- [INFERENCE] "Flat" = monolithic ownership with thin helper imports, not layered supervisors
- [UNKNOWN] Whether tests isolate sub-behaviors via partial doubles

### Intentionality

**Verdict**: **UNKNOWN** (likely ACCIDENTAL) (Confidence: LOW)

**Evidence from [git archaeology](166ed3b2-d5cc-4d95-8786-b954bb10448e)**:
- Research.md L630-632: "**4. Actor model (partial)** - Supervisor manages child lifecycle - **Gap**: No supervision tree - flat supervisor, no hierarchical restart"
- The word "Gap" suggests incomplete implementation, not deliberate minimalism
- No comments in plugin-supervisor.ts explaining flat vs hierarchical choice
- Focus was on isolation (security) not fault-tolerance (supervision patterns)
- Could be: (a) deliberate MVP scope cut, or (b) unfamiliarity with supervision patterns

**Why unknown**:
- No evidence either way in git history or comments
- No references to Erlang/OTP supervision trees
- Repository only has 2 commits (wholesale import), preventing true archaeology

### Migration Feasibility

**Evidence from [feasibility analysis](0c75236e-f5a1-4e3b-9ec9-b6bd962f7582)**:

**Incremental Path** (Responsibility-Based Split):
1. Extract `ProcessManager` (spawn, kill, restart logic)
2. Extract `IpcRouter` (message routing, pending map)
3. Extract `HealthMonitor` (heartbeat, crash tracking)
4. Extract `RateLimiter` (already exists as separate class - reuse)
5. PluginSupervisor composes these, delegates operations

**Reversibility**: ✅ YES - can inline back to single class

**Blast Radius**: Per extraction
- 1 new file (e.g., `supervisor/process-manager.ts`)
- 1 existing `plugin-supervisor.ts` (remove code, add delegation)
- 0 changes to consumers (runtime service API unchanged)
- **2 files per extraction phase**

**Safety Nets**:
- ✅ Integration: plugin-runtime.test.ts (720 lines) covers spawn/activate, crash/restart, heartbeat timeout
- ❌ No unit tests for supervisor class itself
- ❌ No timeout/heartbeat failure tests

**First Step Prerequisite**:
- **Add unit tests for supervisor lifecycle**
- Test crash backoff calculation
- Test heartbeat timeout detection
- Test graceful vs forced kill
- Reason: Supervisor is critical path - refactor without tests is high risk
- Effort: **M** (1 day to cover core lifecycle)

### Cost-Benefit Analysis

**Cost of Debt**:
- Monolithic supervisor makes it harder to:
  - Test individual concerns in isolation
  - Understand crash recovery logic (buried in 781 lines)
  - Add new lifecycle hooks without touching everything

**Cost of Change**:
- Effort: **M** (1 week for full split)
- Risk: **MEDIUM** (critical path, limited test coverage)
- Prerequisite: 1 day (unit tests)
- **Total**: 2 weeks

**ROI**: LOW-MEDIUM - Code quality improvement, but no documented pain from current structure. System is young (1.5 months) - unclear if this complexity is worth addressing now.

**Recommendation**: **DEFER**. Monitor for pain signals (crash bugs, hard-to-diagnose race conditions). If they appear, prioritize. Otherwise, focus on higher-ROI refactors first.

---

## Candidate 6: Dual Database Model

### Current Shape

**Evidence from [code shape analysis](d7670384-81a6-4da6-8918-dcdf9bb957c0)**:

**Location:**
- Plugin-owned DB: `PluginDataDb` in `host/plugin-data.service.ts:59-195`
- Path: `…/plugin.db` via `paths.ts:66-67`
- Design comment: separate file so plugin "physically cannot read trek.db" (L7-16)
- `ATTACH` (and VACUUM/PRAGMA/RECURSIVE/LOAD_EXTENSION) banned (L28, guard at L79-83)
- Core DB: module `db` from `../../../db/database` used in `create-rpc-host.ts:1,220`
- `HostDeps` documents two handles (`rpc-host.ts:47-50`): `data: PluginDataDb` vs `db: CoreDb`

**Responsibilities**:
- `db:*` RPC methods operate only on per-plugin `plugin.db` via `PluginDataDb`
- Core trip/user data accessed only through typed HostDeps methods on trek.db
- Containment: filesystem separation + ATTACH ban
- Consequence: No SQL JOIN between plugin tables and core tables; any correlation is host-mediated (RPC + `plugin_entity_metadata` table)

**Existing Abstractions**:
- `PluginDataDb` API: `query`, `exec`, `tx`, `migrate`, `checkpoint`, `snapshotInto`
- Module helpers: `checkpointAllPluginDataDbs`, `snapshotAllPluginDataDbs`, `removePluginData`
- `CoreDb` narrow interface in `rpc-host.ts:39-44`
- Lazy `get data()` accessor to survive dispose/recreate races

**Evidence Quality**:
- [EVIDENCE] Separate `plugin.db` file + ATTACH forbidden (L10-12, 28, 66)
- [EVIDENCE] Host holds both handles; `db.own` uses `deps.data`, trip reads use `deps.db`
- [EVIDENCE] Plugin↔core linkage via `plugin_entity_metadata` table in trek.db, not cross-DB JOIN
- [INFERENCE] "No-JOIN" is architectural consequence of dual-file isolation, not missing feature

### Intentionality

**Verdict**: **INTENTIONAL CONSTRAINT** (CORRECT ARCHITECTURE) (Confidence: HIGH)

**Evidence from [git archaeology](166ed3b2-d5cc-4d95-8786-b954bb10448e)**:
- Research.md L684-686: "**Chosen: Dual database** - ✅ Plugin data isolated, GDPR compliant - ❌ No JOIN across trek.db ↔ plugin DB"
- plugin-data.service.ts L10-12: "Because it is a SEPARATE FILE, containment is a filesystem fact: the plugin physically cannot read trek.db"
- Extensive security guards (L28-32) show this was designed, not emerged
- This is the ONLY pattern in the entire plugin system with architectural justification comments

**Why intentional and correct**:
- Security boundary: plugins cannot access core user data
- GDPR compliance: per-plugin data can be deleted independently
- Blast radius containment: plugin SQL errors don't corrupt core DB
- Filesystem-level enforcement (not just runtime checks)

### Migration Feasibility

**Evidence from [feasibility analysis](0c75236e-f5a1-4e3b-9ec9-b6bd962f7582)**:

**Recommendation**: **NO CHANGE NEEDED**

This is correct architecture for plugin isolation. If mocking is needed for tests, add `PluginDataStore` interface, but do NOT change the dual-DB model.

**Alternative considered**: Single database with table-name prefixes
- **Rejected**: Less secure (plugin SQL could access core tables with crafted query)
- **Rejected**: ATTACH ban would need runtime enforcement (weaker than filesystem)

### Cost-Benefit Analysis

**Cost of "Debt"**: NONE - This is a security feature, not technical debt.

**Cost of Change**: PROHIBITIVE - Would require:
- Removing filesystem boundary
- Implementing runtime SQL sandboxing
- Rewriting all plugin↔core coordination logic
- Increased security audit surface

**ROI**: NEGATIVE - Change would REDUCE security and increase complexity.

**Recommendation**: **DO NOT CHANGE**. Document this as a deliberate architectural decision. If testing requires easier mocking, add interface abstraction but keep dual-DB implementation.

---

## Refactor Opportunities (Ranked)

Based on cost of debt, cost of change, and strategic value, here are the top 3 opportunities:

### 1. Protocol Versioning (RECOMMENDED FIRST)

**Target Shape**: Wire protocol with version negotiation
- Add `version?: number` to `RpcRequest` envelope
- Support v1 (current) and v2+ (future) simultaneously
- Plugin manifest declares `rpcVersion: 2` to opt in
- Host routes requests to versioned handlers

**Why this ranking**:
- **Unblocks future evolution**: Prevents further ecosystem damage from breaking changes
- **Low risk**: 4-file setup (envelope, rpc-host, manifest, SDK types), stable interface
- **High impact**: Research.md documents "3 breaking changes in 6 weeks" causing plugin author complaints
- **Strategic value**: Must be done BEFORE any more RPC method changes

**Blast Radius**: 4 files (one-time setup), then 1 file per method version

**Incremental Path**:
1. Design version negotiation protocol (1-2 days) ← **PREREQUISITE**
2. Add `version?: number` to RpcRequest (1 day)
3. Update rpc-host version routing logic (2 days)
4. Add manifest `rpcVersion` field (1 day)
5. Document versioning policy (1 day)

**First Step**: Write version negotiation design doc
- Define how plugins declare supported versions
- Define host's backward compatibility window (e.g., support N-2 versions)
- Define deprecation policy (e.g., 6-month warning before removal)

**Success Criteria**:
- Plugin using v1 method continues working after v2 added
- Host logs deprecation warnings for old versions
- Version mismatch errors are clear and actionable

**Effort**: M (1.5-2 weeks including design)  
**Risk**: LOW  
**Impact**: HIGH

---

### 2. Circular Dependency Elimination (QUICK WIN)

**Target Shape**: Extract shared logic to interface module
- Create `plugin-settings.interface.ts` with `readUserSettingDecrypted`
- Both `plugins.service.ts` and `create-rpc-host.ts` import from interface
- Break the cycle: no mutual imports

**Why this ranking**:
- **Immediate pain relief**: Reduces merge conflicts from 2.4/sprint to ~0
- **Quick win**: 1.5 days total (0.5 day prerequisite + 1 day refactor)
- **Unblocks**: Makes god module split easier (fewer co-change constraints)
- **Low risk**: 3-file change, well-tested surface

**Blast Radius**: 3 files
- `plugin-settings.interface.ts` (new)
- `plugins.service.ts` (change export)
- `create-rpc-host.ts` (change import)

**Incremental Path**:
1. Add circular dependency linter (0.5 day) ← **PREREQUISITE**
   - Configure `dependency-cruiser` (already in devDependencies)
   - Add to CI pipeline
2. Extract `plugin-settings.interface.ts` (0.5 day)
3. Update imports in both files (0.25 day)
4. Run full test suite to verify (0.25 day)

**First Step**: Configure `dependency-cruiser`
```json
// .dependency-cruiser.js
module.exports = {
  forbidden: [{
    name: "no-circular",
    severity: "error",
    from: {},
    to: { circular: true }
  }]
};
```

**Success Criteria**:
- CI fails on circular imports
- `plugins.service.ts` and `create-rpc-host.ts` no longer import each other
- All tests pass
- Merge conflict rate drops to <0.5/sprint

**Effort**: S (1.5 days)  
**Risk**: LOW  
**Impact**: MEDIUM (eliminates recurring pain)

---

### 3. God Module Split (HIGHEST IMPACT, HIGHEST EFFORT)

**Target Shape**: Domain-based capability wiring modules
```
host/
  create-rpc-host.ts (composition layer only)
  wiring/
    trip-capabilities.ts (trips, places, days, assignments)
    cost-capabilities.ts (budget, reservations)
    addon-capabilities.ts (journey, atlas, vacay, collab, collections)
    broker-capabilities.ts (AI, notify, scheduler)
```

**Why this ranking**:
- **Eliminates 8x cost multiplier**: Service changes no longer cascade through 720 LOC
- **Enables DI refactor**: Smaller surfaces easier to inject
- **High impact**: Directly addresses "God module" anti-pattern
- **BUT**: Highest effort (4-5 weeks) and risk (no direct tests)

**Blast Radius**: Per domain extraction
- 1 new wiring module (e.g., `wiring/trip-capabilities.ts`)
- 1 updated `create-rpc-host.ts` (remove moved code, compose new module)
- 0 changes to `rpc-host.ts` consumer
- **2 files per domain**

**Incremental Path** (Strangler Fig):
1. Add integration tests for RPC surface (2-3 days) ← **PREREQUISITE**
   - Smoke tests for each capability domain
   - Or full coverage of 110 KNOWN_METHODS (longer)
2. Extract `wiring/trip-capabilities.ts` (3 days)
   - Move ~20-30% of create-rpc-host (trips, places, days, assignments)
   - Import and compose in create-rpc-host
   - Run tests to verify
3. Repeat for cost domain (2 days)
4. Repeat for addon domain (3 days)
5. Repeat for broker domain (2 days)
6. Final: create-rpc-host becomes composition layer (~100 LOC)

**First Step**: Add smoke tests for capability domains
```typescript
// Test that each domain's capabilities work
describe('Trip Capabilities', () => {
  it('trips.getById returns trip', async () => { ... });
  it('places.create creates place', async () => { ... });
  // etc.
});
```

**Success Criteria**:
- Service signature change requires editing 1 wiring module (not 720 LOC)
- Cost multiplier drops from 8x to ~2x
- Each wiring module <200 LOC
- Full test suite passes after each domain extraction

**Effort**: XL (4-5 weeks including tests)  
**Risk**: HIGH (critical path, limited test coverage)  
**Impact**: HIGH (eliminates major pain point)

---

## Candidates Considered and Rejected

### No Dependency Injection → DEFER

**Why deferred**:
- Blocked by god module split (should split first to reduce blast radius)
- DI refactor across 40+ services is large effort (L, 2-3 weeks)
- Current manual DI works (if inelegant)
- ROI improves after god module split (smaller surfaces to inject)

**When to revisit**: After god module is split into domain wiring modules

---

### Flat Supervisor → DEFER

**Why deferred**:
- No documented pain from current monolithic structure
- System is young (1.5 months) - premature to refactor
- Unclear benefit: supervisor works, no crash bugs reported
- Medium effort (M, 2 weeks) for unclear ROI

**When to revisit**: If crash recovery bugs appear, or if hierarchical restart becomes necessary for fault tolerance

**Signals to watch**:
- Frequent crash-restart loops
- Hard-to-diagnose race conditions
- Need for per-plugin-type supervision strategies

---

### Dual Database Model → DO NOT CHANGE

**Why rejected**:
- This is CORRECT ARCHITECTURE, not technical debt
- Provides security boundary (plugin cannot access trek.db)
- GDPR compliance (per-plugin data deletion)
- Filesystem-level enforcement (stronger than runtime checks)

**Alternative considered and rejected**: Single database with table prefixes
- Less secure (plugin SQL could craft queries to access core tables)
- Would require runtime sandboxing (weaker than filesystem)
- Higher security audit surface

**Action**: Document this as deliberate architectural decision. If testing requires easier mocking, add `PluginDataStore` interface abstraction, but keep dual-DB implementation.

---

## Implementation Sequence (Recommended)

### Phase 1: Foundation (Weeks 1-2)
1. **Protocol Versioning** (1.5-2 weeks)
   - Design version negotiation protocol
   - Add `version` field to envelope
   - Implement version routing in rpc-host
   - Document versioning policy

**Why first**: Prevents further ecosystem damage. Must be done BEFORE any more breaking changes.

### Phase 2: Quick Wins (Week 3)
2. **Circular Dependency** (1.5 days)
   - Add circular dependency linter
   - Extract `plugin-settings.interface.ts`
   - Update imports

**Why second**: Quick win that unblocks god module split (fewer co-change constraints).

### Phase 3: God Module Split (Weeks 4-9)
3. **God Module** (4-5 weeks)
   - Add integration tests for RPC surface
   - Extract domains one at a time (trip → cost → addon → broker)
   - Verify tests after each extraction

**Why third**: Requires protocol versioning (to safely change RPC methods) and benefits from circular dependency fix (fewer conflicts during refactor).

### Phase 4: Follow-ups (Future)
4. **Dependency Injection** (after god module split)
5. **Flat Supervisor** (if pain signals appear)

---

## Open Questions

### For Planning Session

1. **Version negotiation policy**:
   - How many versions should host support? (N-2? N-3?)
   - What's the deprecation timeline? (3 months? 6 months?)
   - How do we communicate breaking changes to plugin authors?

2. **God module test strategy**:
   - Full coverage of 110 methods (L, 2-3 days)?
   - OR smoke tests per domain (M, 1 day per domain)?
   - What's acceptable risk vs time trade-off?

3. **Migration coordination**:
   - Can we run protocol versioning + circular dependency in parallel? (YES)
   - Should we wait for protocol versioning to finish before starting god module? (RECOMMENDED)
   - Who needs to review architectural changes? (Maurice + jubnl per research.md)

### Unknown Risks

4. **TypeScript strict mode**:
   - All candidates have `strict: false` in tsconfig
   - Should we enable strict mode first? Or fix during refactor?
   - What's the blast radius of enabling strict mode?

5. **Production plugin ecosystem**:
   - How many plugins currently deployed?
   - Which plugins would be affected by protocol versioning?
   - Do we have a beta testing group for breaking changes?

---

## Related Research

### From context/changes/plugin-service/

- **research.md**: Technical debt baseline
  - Circular dependency: 2.4 conflicts/sprint (L346)
  - God module: 8x cost multiplier (L360-362)
  - No protocol versioning: 3 breaking changes in 6 weeks (L351)

- **blast-radius-analysis.md**: Co-change patterns
  - Circular dependency: RED (HIGH) risk (Section 1, L18-36)
  - God module: 40+ service dependencies (Section 2, L69-71)
  - New capability change cluster: 4 files (Section 8, Cluster 1)

### Historical Context

- **Platform pivot**: Plugin System introduced July 2026 as marketplace strategy
- **Timeline**: 102 commits in 1.5 months = aggressive delivery
- **Co-evolution**: Built simultaneously with MCP Tools (also has 20+ circular deps)
- **Ownership**: Maurice + jubnl (joint owners) - both required for major decisions

---

## Weryfikacja twierdzeń (ast-grep)

**Data weryfikacji**: 2026-07-26  
**Commit**: c439b93  
**Metoda**: ast-grep + ripgrep dla strukturalnych twierdzeń liczbowych

### Podsumowanie

Wszystkie kluczowe twierdzenia strukturalne zostały **POTWIERDZONE** za pomocą ast-grep i ripgrep. Jedna drobna korekta numeracji linii (METHOD_PERMISSION).

### Tabela weryfikacji

| Twierdzenie | Werdykt | Dowód (plik:linia) | Metoda/Wzorzec |
|-------------|---------|-------------------|----------------|
| **1. Circular Dependency** | | | |
| `plugins.service.ts:8` importuje `pluginBudgetUsage` z `./host/create-rpc-host` | ✅ POTWIERDZONE | `plugins.service.ts:8` | `rg "import.*pluginBudgetUsage"` |
| `create-rpc-host.ts:17` importuje `readUserSettingDecrypted` z `../plugins.service` | ✅ POTWIERDZONE | `create-rpc-host.ts:17` | `rg "import.*readUserSettingDecrypted"` |
| Call site: `pluginBudgetUsage` używane w L287-288 | ✅ POTWIERDZONE | `plugins.service.ts:287-288` | `rg "pluginBudgetUsage"` |
| Call site: `readUserSettingDecrypted` używane w L424 | ✅ POTWIERDZONE | `create-rpc-host.ts:424` | `rg "readUserSettingDecrypted"` |
| **2. God Module (create-rpc-host.ts)** | | | |
| 928 LOC | ✅ POTWIERDZONE | `create-rpc-host.ts` | `wc -l` → 928 |
| 41 import statements (L1-41) | ✅ POTWIERDZONE | `create-rpc-host.ts:1-41` | `rg '^import'` → 41 linii |
| 25 services imported (report: 25-27) | ✅ POTWIERDZONE (25) | `create-rpc-host.ts` | `rg "from '\.\./\.\./\.\./services/"` → 25 |
| `createRealRpcHost` at L209-927 | ✅ POTWIERDZONE | `create-rpc-host.ts:209` | `rg "^export function createRealRpcHost"` |
| **3. No Dependency Injection** | | | |
| 4 Nest providers in `plugins.module.ts:33` | ✅ POTWIERDZONE | `plugins.module.ts:33` | `rg "@Module"` context |
| Manual `new BudgetService()` at L59 | ✅ POTWIERDZONE | `create-rpc-host.ts:59` | `rg "new (BudgetService)"` |
| Manual `new ReservationsService()` at L60 | ✅ POTWIERDZONE | `create-rpc-host.ts:60` | `rg "new.*ReservationsService"` |
| Per-call `new PluginOAuthService()` at L426 | ✅ POTWIERDZONE | `create-rpc-host.ts:426` | `rg "new PluginOAuthService"` |
| `new PluginSupervisor` in runtime service at L133 | ✅ POTWIERDZONE | `plugin-runtime.service.ts:133` | `rg "new PluginSupervisor"` |
| **4. No Protocol Versioning** | | | |
| No `version` field in `RpcRequest` (L19-24) | ✅ POTWIERDZONE | `protocol/envelope.ts:19-24` | ast-grep interface + `rg "version\?:"` → 0 matches |
| `KNOWN_METHODS` array: 110 methods (L49-160) | ✅ POTWIERDZONE | `protocol/envelope.ts:49-160` | `sed -n '50,159p' \| rg "^\s*'" \| wc -l` → 110 |
| `METHOD_PERMISSION` at L163-277 | ✅ DOPRECYZOWANE (164 (raport: 163)) | `protocol/envelope.ts:164` | `rg "^export const METHOD_PERMISSION"` (komentarz w L163) |
| **5. Flat Supervisor** | | | |
| 781 LOC | ✅ POTWIERDZONE | `supervisor/plugin-supervisor.ts` | `wc -l` → 781 |
| Single class `PluginSupervisor` at L113 | ✅ POTWIERDZONE | `supervisor/plugin-supervisor.ts:113` | `rg "^export class PluginSupervisor"` |
| `Supervised` state bag at L42-68 | ✅ POTWIERDZONE | `supervisor/plugin-supervisor.ts:42-68` | `rg "^interface Supervised"` |
| **6. Dual Database Model** | | | |
| `ATTACH` forbidden in plugin-data.service.ts | ✅ POTWIERDZONE | `host/plugin-data.service.ts:28` | `rg "ATTACH"` → FORBIDDEN regex |

### Korekty

1. **METHOD_PERMISSION location**: Raport podaje L163-277, definicja faktycznie zaczyna się w L164 (L163 to komentarz JSDoc). Korekta kosmetyczna, nie wpływa na wnioski.

### Dodatkowe obserwacje

- Wszystkie liczby linii kodu (928, 781, 110) dokładnie zgodne z raportem
- Wzorce importów i call-site'ów zweryfikowane precyzyjnie
- Żadne twierdzenie nie zostało obalone
- Ranking kandydatów pozostaje bez zmian — weryfikacja potwierdza podstawy analizy

---

## Code References

### Circular Dependency
- `server/src/nest/plugins/plugins.service.ts:8` - imports `pluginBudgetUsage`
- `server/src/nest/plugins/host/create-rpc-host.ts:17` - imports `readUserSettingDecrypted`

### God Module
- `server/src/nest/plugins/host/create-rpc-host.ts:1-41` - 41 import statements
- `server/src/nest/plugins/host/create-rpc-host.ts:209-927` - 720 LOC service delegation

### Protocol
- `server/src/nest/plugins/protocol/envelope.ts:19-42` - Envelope types (no version field)
- `server/src/nest/plugins/protocol/envelope.ts:49-160` - KNOWN_METHODS (110 methods)
- `server/src/nest/plugins/protocol/envelope.ts:164` - METHOD_PERMISSION definition (raport: 163)

### Supervisor
- `server/src/nest/plugins/supervisor/plugin-supervisor.ts:113` - PluginSupervisor class (781 LOC)
- `server/src/nest/plugins/supervisor/plugin-supervisor.ts:42-68` - Supervised state bag

### Dual Database
- `server/src/nest/plugins/host/plugin-data.service.ts:7-16` - Design justification comments
- `server/src/nest/plugins/host/plugin-data.service.ts:28` - ATTACH forbidden

---

**Research completed**: 2026-07-26T14:41:50+0000  
**Next step**: Review findings with team, decide implementation order, schedule planning session for selected opportunities.
