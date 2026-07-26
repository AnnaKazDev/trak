# Raport architektoniczny — Moduł 4 (10xArchitect)

**Data:** 2026-07-26 · **Źródła wyłącznie:** artefakty L2–L5 poniżej · Twierdzenia strukturalne z cytowanych plików.

---

## 1. Opisane projekty

| Repo | Stack | Skala (orientacyjnie) | Artefakty |
| --- | --- | --- | --- |
| **TREK** (Travel Planner; open-source) | React 19 + NestJS 11, shared Zod | ~1 338 commitów (Mar–Jul 2026), ~6 254 modułów depcruise; monorepo client/server/shared | **L2** mapa · **L3** research Plugin System · **L4** plan refaktoru |
| **Plant It** (własna app; dokumentacja w tym repo pod `docs/plant-it-app/`) | Astro + React + TS + Supabase + Cloudflare | MVP garden journaling; **BRAK artefaktu** na LOC / liczbę commitów | **L5** domain distillation · invariant/agregat · ACL |

> Ścieżka użytkownika `context/map/repo-map.md` (treść: „Repository Map: TREK Travel Planner”).

---

## 2. Mapa projektu (L2 — TREK)

Źródło: `context/map/repo-map.md`.

1. **Strefy ryzyka (wysokie):** Planner God components (10/10; m.in. `DayPlanSidebar` 2 529 LOC), coupling DB/`migrations.ts` → ~84% projektu (10/10), MCP Tools (9/10; 20+ cykli, 0 testów wg mapy), **Plugin System (9/10; 1 cykl, 0 testów wg mapy)**.
2. **Lokalne centra:** Planner (464 zmian), `tripStore` (131 usages / 10 cykli), services ↔ unit tests (29 co-mods — siła TDD), nowe platformowe: plugins (102) i MCP (100).
3. **Entry pointy (dzień 1):** `TripPlannerPage.tsx` → `tripStore.ts` → `shared` Zod / API client → `trips.service.ts` → `migrations.ts` (danger) → `tests/unit/services/` → MCP `vacay.ts`.
4. **Unknowns / poza mapą:** performance, security audit, user metrics, deployment topology, i18n coupling; okno tylko 4,5 miesiąca historii.
5. **Korekta z L3 (nie z mapy):** mapa „Plugins = 0 tests” jest **nieaktualna** względem researchu L3 (628 testów) — patrz §3.

---

## 3. Analiza ficzera (L3 — TREK)

Źródło: `context/changes/plugin-service/research.md` (repo: trek).

**Przepływ i link do mapy:** Plugin System (`server/src/nest/plugins/`) — strefa ryzyka L2 §3.4 (Risk 9/10, platform pivot lipiec 2026). Wybrany jako młody marketplace z cyklem i god-modulem RPC.

**Feature overview:** Input: HTTP `POST /api/plugins/:id/…` (JWT) → proxy → supervisor IPC → child → SDK capability. Stan: write w `trek.db` (np. places), append `plugin_capability_audit`, broadcast WS. Powrót: `{ok, result}` → HTTP + JSON; błędy grant/schema/permission → wyjątki SDK / `HOST_ERROR`. 2 hop’y IPC na request; 7 warstw host↔plugin.

**Technical debt (top):**

| Ryzyko | Dowód |
| --- | --- |
| **Circular import** `plugins.service` ↔ `create-rpc-host` | L3: importy L8↔L17; **ast-grep/grep CONFIRMED** (§9 research) |
| **God module** `create-rpc-host.ts` | 928 LOC, **41 importów** (ast-grep corrected z 42), ~25–27 serwisów — blast przy każdej zmianie wiring |
| **Luki testowe mimo dużej siatki** | 628 `it()` (mapa „0” skorygowana); luki: Audit API, `plugin-activity.controller`, `signature-status`, audit wiring bez assertów side-effect |

---

## 4. Plan refaktoryzacji (L4 — TREK)

Źródło: `context/changes/refactor-opportunities/plan.md` (+ brief decisions).

**Co refaktoryzujemy (wybrany scope):** (1) framework wersjonowania wire RPC (`version?` / manifest `rpcVersion`, host max, activate hard-fail, missing=v1, gotowość N/N−1 bez handlerów v2); (2) zerwanie cyklu settings + path-scoped depcruise `error` pod `server/src/nest/plugins`; (3) strangler: smoke trip-domain + extract `host/wiring/trip-capabilities.ts`.

**Świadomie NIE:** full split god-module (cost/addon/…), Nest DI dla host/supervisor, flat supervisor, zmiana dual-DB, global `no-circular`→error, prawdziwe handlery v2 / kampania deprecacji, TS `strict`, feature-flagi wersji.

| Faza | Jedna linia | Weryfikacja |
| --- | --- | --- |
| 1 Versioning | Envelope + activate gate, default v1 | Auto: typecheck, vitest, lint · Ręcznie: over-max code + fixture bez `rpcVersion` |
| 2 Cycle + CI | Extract settings helper; `depcruise:plugins` | Auto: depcruise 0, typecheck, unit settings/host · Ręcznie: brak pary cyklu |
| 3 Trip smokes | Minimalny zestaw trip RPC przed extract | Auto: smoke + istniejące wave tests · Ręcznie: coverage trips/places/days/… |
| 4 Extract wiring | `trip-capabilities.ts` composed w factory | Auto: smoke+unit+runtime subset, lint, depcruise · Ręcznie: bloki przeniesione + spot-check RPC |

---

## 5. Domena wg DDD (L5 — Plant It)

Źródła: `docs/plant-it-app/01-…`, `02-…`, `03-…`.

**Ubiquitous language (klucz):** Growth story / visual timeline · Plant (+ display name, grid) · Action (care / planned) · Garden map / unique location · Weather snapshot przy akcji.

**Rozjazdy model↔kod (najważniejsze):** FR-014 „różna lokalizacja” vs multi-occupancy + fan layout; „last action” vs sort max(date) (planned wypycha); moon usunięty z teaserów w PRD vs nadal w UI + brak calendar; weather fail → ciche `null` zamiast retry/save-without.

**Niezmiennik #1:** Unique cell occupancy — każda roślina użytkownika na innej komórce `(user_id, grid_x, grid_y)` (I1 / FR-014). **Agregat-strażnik:** **Garden** (root; plants jako placements; identity MVP = `user_id`), nie samotny Plant.

**ACL:** przecieka **WeatherAPI.com** (HTTP vendor) przez **5 warstw**: types + lib + API + UI + JSONB payload (+ testy/docs) — ~25 plików; intencja roadmap „can swap later” vs kształt vendora jako kontrakt aplikacji.

---

## 6. Decyzje, które należą do mnie

**TREK (L3/L4):** AI podało top-3 (versioning, cykl, pełny split god-module) i chciało zaostrzyć depcruise. Mapa L2 mylnie mówiła o 0 testach pluginów. Ja zawęziłem plan do: versioning + zerwanie cyklu + pierwszy extract trip (bez pełnego splitu / DI / dual-DB). Depcruise na `error` tylko w `plugins/` (globalnie blokuje ~64 innych cykli). Wersje: brak pola = v1; zbyt wysoka `rpcVersion` → fail przy activate; N/N−1 bez prawdziwego v2.

**Plant It (L5):** AI znalazło pojęcia, niezmienniki i przecieki. Moje decyzje (plany, jeszcze bez kodu):

1. **Źródło prawdy = PRD.** Przy konflikcie z kodem lub starymi notatkami wygrywa PRD — także wobec S-05 „allow overlaps”.
2. **#1 niezmiennik: jedna roślina na komórkę** (FR-014). Egzekwujemy. Opcja „wiele roślin w komórce” odpada, dopóki nie zmienimy PRD.
3. **Strażnik = agregat Garden**, nie pojedynczy Plant — reguła dotyczy całego ogrodu (`user_id`).
4. **ACL najpierw dla WeatherAPI**, nie Supabase. Kształt domenowy: `WeatherSnapshot` za portem/adapterem; UI/API nie powinny znać pól vendora.
