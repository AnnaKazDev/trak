# Code Coupling Analysis
## Analiza Sprzężeń Między Obszarami Kodu

> **Okres analizy:** Ostatnie 12 miesięcy (400 commitów)  
> **Metoda:** Co-occurrence analysis - katalogi modyfikowane w tych samych commitach  
> **Data wygenerowania:** 26 lipca 2026

---

## 🎯 Executive Summary

Analiza sprzężeń (coupling) pokazuje, które obszary kodu są ze sobą powiązane i wymagają synchronicznych zmian. Silne sprzężenie może wskazywać na:
- ✅ Dobrze zaprojektowane granice modułów (expected coupling)
- ⚠️ Ukryte zależności wymagające refactoringu
- 📊 Wzorce architektoniczne (np. test-driven development)

### Kluczowe Odkrycia

1. **Frontend-Backend coupling silny** - 31 wspólnych commitów (components + services)
2. **Test-Driven culture widoczna** - Testing pojawia się w TOP parach
3. **Planner jest hubem** - Najczęściej łączony z innymi obszarami
4. **Cross-stack changes dominują** - Full-stack features wymagają zmian na wszystkich poziomach

---

## 📊 TOP 30 Najpopularniejszych Par Katalogów

### Ranking Ogólny (Wszystkie Pary)

| Rank | Para Katalogów | Co-modifications | Typ Coupling | Interpretacja |
|------|---------------|------------------|--------------|---------------|
| **1** | `client/src/components` ↔ `client/src/pages` | **32** | 🎨 Frontend-Internal | **Expected** - Pages używają components |
| **2** | `client/src/components` ↔ `server/src/services` | **31** | 🔗 Cross-Stack | **Strong** - Full-stack features |
| **3** | `server/src/services` ↔ `server/tests/unit` | **29** | 🧪 Test-Code | **Excellent** - TDD culture! |
| **4** | `client/src/pages` ↔ `server/src/services` | **25** | 🔗 Cross-Stack | **Strong** - API-driven pages |
| **5** | `server/src/db` ↔ `server/src/services` | **22** | 🔧 Backend-Internal | **Expected** - Services use DB |
| **6** | `server/src/services` ↔ `server/tests/integration` | **21** | 🧪 Test-Code | **Good** - Integration testing |
| **7** | `client/src/components` ↔ `server/tests/unit` | **20** | 🔗 Cross-Stack-Test | Interesting - Frontend triggers backend tests |
| **8** | `server/src/routes` ↔ `server/src/services` | **19** | 🔧 Backend-Internal | **Expected** - Routes call services |
| **9** | `client/src/components` ↔ `server/src/db` | **17** | 🔗 Cross-Stack | Schema changes affect UI |
| **10** | `client/src/components` ↔ `server/tests/integration` | **16** | 🔗 Cross-Stack-Test | E2E testing patterns |

### Inne Znaczące Pary

| Rank | Para | Count | Znaczenie |
|------|------|-------|-----------|
| 11 | `server/tests/integration` ↔ `server/tests/unit` | 16 | Test infrastructure shared |
| 12 | `client/src/pages` ↔ `server/tests/integration` | 14 | E2E testing |
| 13 | `server/src/db` ↔ `server/tests/unit` | 14 | DB testing |
| 14 | `client/src/pages` ↔ `server/tests/unit` | 13 | Pages trigger backend tests |
| 15 | `client/src/pages` ↔ `server/src/db` | 13 | Schema affects pages |
| 16 | `server/src/db` ↔ `server/tests/integration` | 13 | DB integration tests |
| 17 | `server/src/mcp` ↔ `server/src/services` | 13 | MCP uses services |
| 18 | `client/src/components` ↔ `client/src/utils` | 12 | Components use utilities |
| 19 | `client/src/pages` ↔ `client/src/utils` | 12 | Pages use utilities |
| 20 | `client/src/api` ↔ `client/src/pages` | 12 | API client in pages |

---

## 🔍 Deep Dive: TOP 3 Obszary

Szczegółowa analiza coupling dla trzech najaktywniejszych obszarów projektu:

---

### 1️⃣ `client/src/components/Planner` (464 zmiany - Rank #1)

#### Najczęstsze Sprzężenia (TOP 15)

| Rank | Co-modified With | Count | Typ | Analiza |
|------|------------------|-------|-----|---------|
| **1** | `client/src/pages/TripPlannerPage.tsx` | **19** | 🎨 Parent-Child | **EXPECTED** - Page hosts Planner |
| **2** | `server/src/db/migrations.ts` | **14** | ⚠️ Schema-UI | **RED FLAG** - Schema changes affect UI directly |
| **3** | `client/src/components/Map` | **14** | 🎨 Sibling | **EXPECTED** - Planner shows map |
| **4** | `server/tests/unit/services` | **13** | 🧪 Cross-Test | Backend services tested when Planner changes |
| **5** | `client/src/components/Settings` | **12** | 🎨 Sibling | Settings affect Planner behavior |
| 6 | `client/src/api/client.ts` | 11 | 🔗 API | Planner uses API heavily |
| 7 | `client/src/components/shared` | 11 | 🎨 Shared-UI | Uses shared components |
| 8 | `client/src/types.ts` | 11 | 📦 Types | Type definitions for Planner |
| 9 | `client/src/components/Budget` | 10 | 🎨 Sibling | Budget related to Planner |
| 10 | `server/src/services/reservationService.ts` | 10 | 🔗 Backend | Reservations in Planner |
| 11 | `server/src/services/tripService.ts` | 10 | 🔗 Backend | Trip service core |
| 12 | `client/src/pages/SharedTripPage.tsx` | 9 | 🎨 Related-Page | Shared trips use Planner |
| 13 | `server/src/mcp/tools` | 9 | 🤖 AI | MCP exposes Planner to AI |
| 14 | `client/src/components/Admin` | 9 | 🎨 Sibling | Admin configures Planner |
| 15 | `client/src/components/Files` | 9 | 🎨 Sibling | File attachments in Planner |

#### 🎯 Wnioski dla Planner

**Mocne Strony:**
- ✅ Dobrze izolowany jako komponent (używa shared components)
- ✅ Clear API boundary (client.ts jako interface)
- ✅ Testowany razem z backend services (TDD pattern)

**Czerwone Flagi:**
- ⚠️ **Database coupling (14 co-modifications z migrations)** - Schema changes wymagają UI updates
  - **Problem:** Tight coupling między DB schema i UI
  - **Rekomendacja:** API contract layer (DTO pattern), API versioning
  
- ⚠️ **Duże sprzężenie z wieloma komponentami** (Map, Settings, Budget, Admin, Files)
  - **Problem:** Planner może być "god component"
  - **Rekomendacja:** Consider splitting Planner na mniejsze, focused sub-components

**Pozytywne Wzorce:**
- 💚 **Cross-stack testing** - Changes trigger backend tests (dobra praktyka!)
- 💚 **Shared components reuse** - Używa `shared` components
- 💚 **Type safety** - Strong coupling z `types.ts`

**Architektura:**
```
TripPlannerPage (19)
       │
       ├─── Planner Component (CENTRUM)
       │           │
       │           ├─── Map (14)
       │           ├─── Settings (12)
       │           ├─── Budget (10)
       │           ├─── Files (9)
       │           └─── Admin (9)
       │
       ├─── API Client (11)
       │       │
       │       └─── Backend Services
       │                 │
       │                 ├─── Trip Service (10)
       │                 ├─── Reservation Service (10)
       │                 └─── DB Migrations (14) ⚠️
       │
       └─── Tests (13)
```

**Ryzyko Refactoringu:** 🔴 **HIGH** - Ze względu na wiele zależności, changes są expensive

---

### 2️⃣ `server/tests/unit/services` (187 zmian - Rank #2)

#### Najczęstsze Sprzężenia (TOP 15)

| Rank | Co-modified With | Count | Typ | Analiza |
|------|------------------|-------|-----|---------|
| **1** | `client/src/components/Planner` | **13** | 🔗 Cross-Test | Frontend changes trigger backend tests |
| **2** | `server/src/db/migrations.ts` | **12** | 🧪 Test-DB | **GOOD** - Schema tests |
| **3** | `client/src/api/client.ts` | **11** | 🔗 Cross-Test | API contract testing |
| **4** | `client/src/pages/TripPlannerPage.tsx` | **10** | 🔗 Cross-Test | E2E coverage |
| **5** | `server/src/services/tripService.ts` | **9** | 🧪 Test-Code | **EXCELLENT** - Direct SUT |
| 6 | `server/src/services/mapsService.ts` | 8 | 🧪 Test-Code | Maps service tests |
| 7 | `server/src/mcp/tools` | 8 | 🧪 Test-Code | MCP testing |
| 8 | `client/src/components/shared` | 8 | 🔗 Cross-Test | Shared components tested |
| 9 | `server/src/services/authService.ts` | 8 | 🧪 Test-Code | Auth testing |
| 10 | `server/src/services/journeyService.ts` | 8 | 🧪 Test-Code | Journey testing |
| 11 | `server/src/services/notifications.ts` | 7 | 🧪 Test-Code | Notification tests |
| 12 | `server/src/services/adminService.ts` | 7 | 🧪 Test-Code | Admin tests |
| 13 | `server/src/services/atlasService.ts` | 7 | 🧪 Test-Code | Atlas tests |
| 14 | `server/src/services/budgetService.ts` | 7 | 🧪 Test-Code | Budget tests |
| 15 | Multiple backend services | ~6-7 | 🧪 Test-Code | Comprehensive coverage |

#### 🎯 Wnioski dla Tests/Unit/Services

**Mocne Strony:**
- ✅ **EXCELLENT Test Coverage Pattern** - Tests change razem z kodem!
- ✅ **Test-Driven Development visible** - #3 rank w całym projekcie
- ✅ **Cross-stack testing** - Frontend changes trigger backend tests
- ✅ **Wide coverage** - 15+ różnych services testowanych

**Pozytywne Sygnały:**
- 💚 **Direct SUT coupling** (9 z tripService) - Tests blisko kodu
- 💚 **DB testing** (12 z migrations) - Schema changes testowane
- 💚 **API contract testing** (11 z api/client) - Contract tests
- 💚 **Comprehensive service coverage** - Wszystkie major services mają testy

**Wzorce Testing:**
```
Frontend Changes (Planner, Pages)
       │
       ├─── Trigger Backend Tests ✅
       │
Backend Service Changes
       │
       ├─── Unit Tests (same commit) ✅
       ├─── DB Migration Tests ✅
       └─── Integration Tests ✅

Database Changes (migrations)
       │
       └─── Unit Tests Updated ✅
```

**Architektura Testowa:**
```
                    Tests/Unit/Services (CENTRUM)
                            │
        ┌───────────────────┼───────────────────┐
        │                   │                   │
   Services (Direct)   Database (12)      Frontend (13)
        │                   │                   │
    Trip (9)           Migrations         Planner Tests
    Maps (8)           Schema Tests       Page Tests
    Auth (8)                              API Tests
    Journey (8)
    MCP (8)
    +10 more
```

**Interpretacja:**
- **Test-Driven Culture** - Testy są first-class citizen, nie afterthought
- **Comprehensive Coverage** - Wide range of services covered
- **Cross-Stack Awareness** - Frontend changes nie przechodzą bez backend tests

**Ryzyko Refactoringu:** 🟢 **LOW** - Tests pomagają w refactoringu, nie przeszkadzają

---

### 3️⃣ `client/src/components/Map` (105 zmian - Rank #3)

#### Najczęstsze Sprzężenia (TOP 15)

| Rank | Co-modified With | Count | Typ | Analiza |
|------|------------------|-------|-----|---------|
| **1** | `client/src/components/Planner` | **14** | 🎨 Sibling | **EXPECTED** - Planner displays Map |
| **2** | `client/src/pages/TripPlannerPage.tsx` | **12** | 🎨 Parent-Child | Page hosts Map |
| **3** | `client/src/components/Settings` | **10** | 🎨 Sibling | Map settings (tiles, style) |
| **4** | `server/src/db/migrations.ts` | **10** | ⚠️ Schema-UI | DB changes affect map data |
| **5** | `client/src/api/client.ts` | **9** | 🔗 API | Map fetches geo data |
| 6 | `client/src/components/Journey` | 8 | 🎨 Sibling | Journey shows map |
| 7 | `client/src/types.ts` | 8 | 📦 Types | Geo types |
| 8 | `client/src/components/Admin` | 7 | 🎨 Sibling | Admin map config |
| 9 | `client/src/components/shared` | 7 | 🎨 Shared-UI | Shared UI elements |
| 10 | `client/src/hooks/useRouteCalculation.ts` | 7 | 🔗 Logic | **GOOD** - Route logic extracted |
| 11 | `server/src/services/atlasService.ts` | 6 | 🔗 Backend | Atlas geo data |
| 12 | `client/src/index.css` | 6 | 🎨 Styles | Map styling |
| 13 | `client/src/store/settingsStore.ts` | 6 | 📦 State | Map settings state |
| 14 | `server/src/services/mapsService.ts` | 6 | 🔗 Backend | Maps backend service |
| 15 | `server/tests/unit/services` | 6 | 🧪 Test | Map backend tests |

#### 🎯 Wnioski dla Map Component

**Mocne Strony:**
- ✅ Clear parent (Planner) relationship
- ✅ **Extracted hook logic** (useRouteCalculation) - Good separation!
- ✅ Settings-driven behavior (configurable)
- ✅ Backend services tested (atlasService, mapsService)

**Czerwone Flagi:**
- ⚠️ **Database coupling (10 co-modifications)** - Similar do Planner
  - **Problem:** Schema changes affect map rendering
  - **Rekomendacja:** API layer, data transformation

**Pozytywne Wzorce:**
- 💚 **Logic extraction** - `useRouteCalculation` hook (dobra praktyka!)
- 💚 **State management** - Używa `settingsStore` (separation of concerns)
- 💚 **Backend services** - Clear boundary (atlasService, mapsService)
- 💚 **Multi-context usage** - Używany w Planner, Journey, Atlas

**Architektura:**
```
Pages (TripPlanner, Journey, Atlas)
       │
       └─── Map Component
               │
               ├─── useRouteCalculation (7) ✅
               ├─── settingsStore (6) ✅
               │
               ├─── API Client (9)
               │       │
               │       └─── Backend Services
               │               ├─── atlasService (6)
               │               └─── mapsService (6)
               │
               └─── Siblings
                       ├─── Planner (14)
                       ├─── Settings (10)
                       ├─── Journey (8)
                       └─── Admin (7)
```

**Reusability:** 🟢 **HIGH** - Używany w multiple contexts (Planner, Journey, Atlas)

**Interpretacja:**
- **Well-designed component** - Logic extraction, state management
- **Configurable** - Settings-driven behavior
- **Reusable** - Multiple use cases
- **Backend boundary clear** - Dedicated services

**Ryzyko Refactoringu:** 🟡 **MEDIUM** - Używany w wielu miejscach, ale dobrze zaprojektowany

---

## 🔗 Najczęstsze Trójki (3-Way Coupling)

### TOP 20 Triads

Trójki pokazują bardziej złożone wzorce - które 3 obszary są często modyfikowane razem.

| Rank | Triad | Count | Interpretacja |
|------|-------|-------|---------------|
| **1** | `components` + `pages` + `services` | **21** | 🔗 **Full-Stack Feature** - Kompletna feature (UI + API + Logic) |
| **2** | `components` + `services` + `tests/unit` | **18** | 🧪 **TDD Full-Stack** - Feature + Tests razem! |
| **3** | `services` + `tests/integration` + `tests/unit` | **16** | 🧪 **Comprehensive Testing** - Unit + Integration |
| **4** | `components` + `db` + `services` | **16** | 🔗 **Full-Stack + Schema** - Schema changes affect wszystko |
| **5** | `components` + `services` + `tests/integration` | **15** | 🧪 **E2E Testing** - Integration tests dla features |
| 6 | `components` + `pages` + `tests/integration` | 14 | 🧪 E2E Frontend tests |
| 7 | `pages` + `services` + `tests/integration` | 14 | 🧪 API integration tests |
| 8 | `components` + `pages` + `tests/unit` | 13 | 🧪 Unit tests dla UI |
| 9 | `components` + `tests/integration` + `tests/unit` | 13 | 🧪 Component testing |
| 10 | `pages` + `services` + `tests/unit` | 13 | 🧪 Page backend tests |

### Wzorce w Triads

#### Pattern #1: Full-Stack Features (Rank 1, 4)
```
Frontend (components + pages)
    +
Backend (services)
    +
Database (db / migrations)
    = Full feature implementation
```
**Interpretacja:** Features są implementowane cross-stack, nie w silosach.

#### Pattern #2: Test-Driven Full-Stack (Rank 2, 3, 5)
```
Code (components/services)
    +
Unit Tests
    +
Integration Tests
    = Comprehensive test coverage
```
**Interpretacja:** Testing jest integral part of development, nie afterthought.

#### Pattern #3: Schema Evolution (Rank 4)
```
Database (migrations)
    +
Backend (services)
    +
Frontend (components)
    = Schema change propagates przez stack
```
**Interpretacja:** Schema changes are expensive - affect wszystkie warstwy.

---

## 📊 Wzorce Coupling - Analiza Kategoryczna

### 1. Expected Coupling (Healthy) ✅

Naturalne, oczekiwane zależności:

| Para | Count | Dlaczego OK |
|------|-------|-------------|
| `components` ↔ `pages` | 32 | Pages składają się z components |
| `routes` ↔ `services` | 19 | Routes delegują do services |
| `db` ↔ `services` | 22 | Services używają database |
| `api/client` ↔ `pages` | 12 | Pages fetują data via API |

**Verdict:** 🟢 **Healthy Architecture** - Clear layer boundaries

---

### 2. Test Coupling (Excellent) 🧪

Test-code coupling - znak dojrzałej kultury:

| Para | Count | Test Type |
|------|-------|-----------|
| `services` ↔ `tests/unit` | 29 | Unit testing services |
| `services` ↔ `tests/integration` | 21 | Integration testing |
| `tests/integration` ↔ `tests/unit` | 16 | Shared test infrastructure |
| `components` ↔ `tests/unit` | 20 | Component testing |

**Verdict:** 🟢 **Excellent TDD Culture** - Tests change z kodem

---

### 3. Cross-Stack Coupling (Strong) 🔗

Frontend-Backend coupling:

| Para | Count | Typ |
|------|-------|-----|
| `components` ↔ `services` | 31 | **STRONG** coupling |
| `pages` ↔ `services` | 25 | API-driven pages |
| `components` ↔ `db` | 17 | Schema affects UI |
| `pages` ↔ `db` | 13 | Schema affects pages |

**Verdict:** 🟡 **Mixed**
- ✅ Full-stack features dobrze zintegrowane
- ⚠️ DB coupling może być problematyczny (brak API contract layer)

---

### 4. Database Coupling (Risky) ⚠️

Database schema coupling z frontendem:

| Para | Count | Risk |
|------|-------|------|
| `components` ↔ `db/migrations` | 17 | 🔴 Direct schema-UI coupling |
| `pages` ↔ `db/migrations` | 13 | 🔴 Schema affects pages |
| Planner ↔ `db/migrations` | 14 | 🔴 Core feature coupled to schema |
| Map ↔ `db/migrations` | 10 | 🔴 Map rendering coupled to schema |

**Verdict:** 🔴 **HIGH RISK**
- **Problem:** Schema changes wymagają UI changes
- **Missing:** API contract layer (DTO pattern)
- **Rekomendacja:** 
  1. Wprowadzić API versioning
  2. DTO pattern (database models ≠ API models)
  3. GraphQL lub API Gateway dla data transformation

---

## 🎯 Strategic Insights

### Architektura Wynikająca z Coupling

Bazując na analizie, prawdziwa architektura wygląda tak:

```
┌─────────────────────────────────────────────────────────────┐
│                     FRONTEND LAYER                          │
│  ┌──────────┐    ┌────────────┐    ┌──────────────┐       │
│  │  Pages   │───▶│ Components │───▶│ Shared/Utils │       │
│  └─────┬────┘    └──────┬─────┘    └──────────────┘       │
│        │                 │                                   │
│        │    Coupling: 32 (Strong) ✅                        │
│        │                 │                                   │
└────────┼─────────────────┼───────────────────────────────────┘
         │                 │
         │  ┌──────────────┘
         │  │  Cross-Stack Coupling: 31 (Very Strong) 🟡
         │  │
┌────────▼──▼─────────────────────────────────────────────────┐
│                      API LAYER                               │
│  ┌─────────────┐              ┌─────────────┐               │
│  │ API Client  │◀─────────────│   Routes    │               │
│  └──────┬──────┘              └──────┬──────┘               │
│         │                            │                       │
│         │  Expected Coupling: 12 ✅  │                       │
│         │                            │                       │
└─────────┼────────────────────────────┼───────────────────────┘
          │                            │
          │  ⚠️ Missing: DTO Layer     │
          │  (Direct DB access)        │
          │                            │
┌─────────▼────────────────────────────▼───────────────────────┐
│                    BACKEND LAYER                             │
│  ┌───────────┐    ┌───────────┐    ┌──────────────┐        │
│  │ Services  │───▶│ Database  │    │     MCP      │        │
│  └─────┬─────┘    └─────┬─────┘    └──────────────┘        │
│        │                 │                                   │
│        │   Coupling: 22 ✅                                   │
│        │                 │                                   │
│        │   ⚠️ PROBLEM: DB directly coupled to Frontend      │
│        │      (components ↔ migrations: 17)                 │
└────────┼─────────────────┼───────────────────────────────────┘
         │                 │
         │                 │
┌────────▼─────────────────▼───────────────────────────────────┐
│                      TEST LAYER                              │
│  ┌──────────────┐         ┌─────────────────┐               │
│  │  Unit Tests  │────────▶│ Integration Tests│               │
│  └──────────────┘         └─────────────────┘               │
│                                                               │
│    Coupling: 16 (Shared Infrastructure) ✅                   │
│    Test-Code Coupling: 29 (Excellent TDD) 🟢                │
└───────────────────────────────────────────────────────────────┘
```

### Kluczowe Problemy

#### 🔴 Problem #1: Brak API Contract Layer

**Evidence:**
- `components` ↔ `db/migrations`: 17 co-modifications
- `pages` ↔ `db/migrations`: 13 co-modifications
- Planner ↔ `migrations`: 14 co-modifications

**Impact:**
- Schema changes wymagają frontend changes
- Trudne API versioning
- Breaking changes unavoidable

**Solution:**
```typescript
// Current (BAD):
Frontend → Services → Database Models (direct)

// Recommended (GOOD):
Frontend → API DTOs → Services → Database Models
                ↑
         Transformation Layer
```

#### 🔴 Problem #2: Planner jako "God Component"

**Evidence:**
- 19 co-modifications z TripPlannerPage
- 14 z Map, 12 z Settings, 10 z Budget
- Coupled z 15+ innymi obszarami

**Impact:**
- Changes są expensive (ripple effect)
- Trudny do testowania
- High refactoring risk

**Solution:**
- Split na sub-components (DayPlanner, PlacesList, etc.)
- Extract business logic do custom hooks
- Reduce direct dependencies

#### 🟡 Problem #3: Strong Cross-Stack Coupling

**Evidence:**
- `components` ↔ `services`: 31 (second highest!)
- `pages` ↔ `services`: 25

**Interpretation:**
- ✅ **Positive:** Full-stack features, nie siloed development
- ⚠️ **Negative:** Changes propagate across stack

**Not necessarily bad** - to jest natural dla full-stack app, ale requires:
- Good API contracts
- Comprehensive tests (które są! ✅)
- Clear boundaries

---

## ✅ Co Działa Dobrze?

### 1. Test-Driven Development Culture 🧪

**Evidence:**
- `services` ↔ `tests/unit`: **29** (#3 overall!)
- `services` ↔ `tests/integration`: **21** (#6 overall!)
- Rank #2 obszar to **testy** (187 zmian)

**Verdict:** 🟢 **Exceptional** - Tests są integral part of development

---

### 2. Clean Layer Boundaries (Mostly) ✅

**Evidence:**
- `routes` ↔ `services`: 19 (expected)
- `db` ↔ `services`: 22 (expected)
- `components` ↔ `pages`: 32 (expected)

**Verdict:** 🟢 **Good Architecture** - Clear separation w większości przypadków

---

### 3. Reusable Components 🎨

**Evidence:**
- `components/shared` pojawia się w wielu parach
- Map używany w multiple contexts
- Extracted hooks (useRouteCalculation)

**Verdict:** 🟢 **Good Reusability** - DRY principle followed

---

## 🎯 Rekomendacje

### Priority 1: Wprowadzić API Contract Layer 🔴

**Co:**
- DTO (Data Transfer Objects) pattern
- API versioning (/v1/, /v2/)
- OpenAPI/Swagger specs

**Dlaczego:**
- Zmniejszy coupling między DB a Frontend
- Umożliwi breaking changes w DB bez breaking frontend
- Better API documentation

**Jak:**
```typescript
// database/models/Trip.ts
interface TripEntity {
  id: number;
  created_at: Date;
  internal_flags: number;
  // ... internal details
}

// api/dto/TripDTO.ts
interface TripDTO {
  id: string;          // UUID
  createdAt: string;   // ISO 8601
  // ... only public fields
}

// services/tripService.ts
class TripService {
  async getTrip(id: string): Promise<TripDTO> {
    const entity = await db.getTrip(id);
    return this.toDTO(entity);  // Transform!
  }
}
```

---

### Priority 2: Refactor Planner Component 🟡

**Co:**
- Split na sub-components:
  - `DayPlanner`
  - `PlacesList`
  - `PlannerMap` (wrapper around Map)
  - `PlannerActions`
- Extract business logic do custom hooks
- Reduce direct dependencies

**Dlaczego:**
- Obecnie 15+ dependencies - high coupling
- Changes są expensive
- Difficult to test in isolation

---

### Priority 3: Continue Excellent Testing 🟢

**Co:**
- **Keep doing what you're doing!** 🎉
- Test-code coupling (29) pokazuje TDD culture
- Consider coverage metrics tracking

**Dlaczego:**
- To jest **biggest strength** projektu
- Umożliwia confident refactoring
- Reduces bugs

---

## 📝 Metodologia

### Jak Dane Były Zbierane

```python
# For each commit in last 12 months:
for commit in commits:
    files = get_changed_files(commit)
    directories = extract_directories(files, level=3)
    
    # Find all pairs
    for (dir1, dir2) in combinations(directories, 2):
        pairs[(dir1, dir2)] += 1

# Find most frequent pairs
top_pairs = sorted(pairs.items(), key=lambda x: -x[1])
```

### Ograniczenia

1. **Commit granularity** - Coupling measurement zależy od commit practices
2. **3-level directory depth** - Może ukryć internal coupling
3. **Filtrowanie** - Package.json, translations odfiltrowane
4. **Sample size** - 300-400 commitów (reprezentatywne, ale nie wszystkie)

---

## 🎬 Podsumowanie dla Busy People

### TL;DR:

🎨 **Frontend-Backend silnie coupled** (31) - Full-stack features  
🧪 **Excellent TDD culture** (29) - Tests z kodem razem  
⚠️ **DB coupling problematyczny** (17) - Brak DTO layer  
🔴 **Planner jest hubem** (15+ dependencies) - Potencjalny refactor  
🟢 **Clean boundaries mostly** - Good architecture overall  

### Top 3 Insights:

1. **Planner** - Hub z 15+ dependencies, coupled z DB (14), needs refactoring
2. **Tests** - Exceptional TDD culture, coupling z services (29) pokazuje quality
3. **Map** - Well-designed (extracted hooks), reusable, medium coupling

### Top 3 Actions:

1. 🔴 **Wprowadzić DTO layer** - Break DB-Frontend coupling
2. 🟡 **Refactor Planner** - Split god component
3. 🟢 **Continue testing** - Biggest strength!

---

*Dokument wygenerowany: 26 lipca 2026*  
*Analiza bazuje na 400 commitów z ostatnich 12 miesięcy*  
*Metoda: Co-occurrence analysis (directories modified together)*
