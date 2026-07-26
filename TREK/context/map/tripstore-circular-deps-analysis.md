# tripStore Circular Dependencies Analysis

## 🎯 Problem Statement

**Pytanie:** Dlaczego tripStore jest niemożliwy do unit testowania?

**Odpowiedź:** 10 circular dependencies między store a slices.

---

## 📊 Wizualizacja Circular Dependencies

```
                    ┌─────────────────────┐
                    │   tripStore.ts      │
                    │  (266 linii, 20 deps)│
                    │  Używany: 131x       │
                    └──────────┬──────────┘
                               │
                ┌──────────────┼──────────────┐
                │              │              │
                │ creates      │ uses         │
     ┌──────────▼───┐   ┌─────▼────┐   ┌────▼─────┐
     │ assignments  │   │   days   │   │  places  │
     │   Slice      │   │  Slice   │   │  Slice   │
     └──────┬───────┘   └────┬─────┘   └────┬─────┘
            │                │              │
            │ imports        │ imports      │ imports
            │ tripStore      │ tripStore    │ tripStore
            └────────────────┴──────────────┘
                         ⚠️ CIRCULAR!


           KAŻDY SLICE IMPORTUJE TRIPSTORE
                        ↓
              tripStore importuje slices
                        ↓
                  10 CYKLI
```

---

## 🔴 All 10 Circular Dependencies

### Pattern: `tripStore.ts` ↔ `slices/*.ts`

1. **tripStore.ts** ↔ **assignmentsSlice.ts**
   - tripStore imports: `createAssignmentsSlice`
   - assignmentsSlice imports: `TripStoreState`

2. **tripStore.ts** ↔ **budgetSlice.ts**
   - tripStore imports: `createBudgetSlice`
   - budgetSlice imports: `TripStoreState`

3. **tripStore.ts** ↔ **daysSlice.ts**
   - tripStore imports: `createDaysSlice`
   - daysSlice imports: `TripStoreState`

4. **tripStore.ts** ↔ **dayNotesSlice.ts**
   - tripStore imports: `createDayNotesSlice`
   - dayNotesSlice imports: `TripStoreState`

5. **tripStore.ts** ↔ **filesSlice.ts**
   - tripStore imports: `createFilesSlice`
   - filesSlice imports: `TripStoreState`

6. **tripStore.ts** ↔ **packingSlice.ts**
   - tripStore imports: `createPackingSlice`
   - packingSlice imports: `TripStoreState`

7. **tripStore.ts** ↔ **placesSlice.ts**
   - tripStore imports: `createPlacesSlice`
   - placesSlice imports: `TripStoreState`

8. **tripStore.ts** ↔ **remoteEventHandler.ts**
   - tripStore imports: `handleRemoteEvent`
   - remoteEventHandler imports: `TripStoreState`

9. **tripStore.ts** ↔ **reservationsSlice.ts**
   - tripStore imports: `createReservationsSlice`
   - reservationsSlice imports: `TripStoreState`

10. **tripStore.ts** ↔ **todoSlice.ts**
    - tripStore imports: `createTodoSlice`
    - todoSlice imports: `TripStoreState`

---

## 🧪 Impact on Testability

### Why Unit Tests Are Impossible

```typescript
// Próba unit testu dla assignmentsSlice:

import { assignmentsSlice } from './assignmentsSlice'

// ❌ PROBLEM: assignmentsSlice importuje TripStoreState
// TripStoreState importuje wszystkie 10 slices
// Więc test dla 1 slice wymaga załadowania wszystkich 10!

test('should add assignment', () => {
  // Wymaga:
  // ✗ Mock tripStore (który importuje 10 slices)
  // ✗ Mock 9 innych slices (które też importują tripStore)
  // ✗ Break circular deps (niemożliwe w runtime)
  
  // Result: Musisz załadować cały store
  // = to już nie jest unit test, to integration test
})
```

### What This Means in Practice

**Unit Test:** ❌ Niemożliwy
- Nie możesz testować 1 slice w izolacji
- Musisz załadować cały store (266 linii + 10 slices)
- Każdy test jest kosztowny (setup + teardown)

**Integration Test:** ✅ Jedyna opcja
- Test z prawdziwym store
- Test wszystkich slices razem
- Nie mockuj store, użyj go

**E2E Test:** ✅ Najlepsze dla komponentów
- Komponenty używają store przez hooks
- E2E testuje całość: store + API + UI
- Nie walcz z circular deps

---

## 📈 Metrics

| Metric | Value | Impact |
|--------|-------|--------|
| **Circular Deps** | 10 | 🔴 Critical |
| **Store LOC** | 266 | 🟡 Medium |
| **Slices** | 10 | 🟡 Medium |
| **Store Usages** | 131 | 🔴 Very High |
| **Unit Tests** | 0 | 🔴 None |

**Z artifact-1-territory.md:**
- tripStore obsługuje Planner (#1, 464 zmiany)
- Components ↔ services (#2 coupling, 31 co-mods)
- Zmiana w store = potencjalnie 131 miejsc do sprawdzenia

---

## 🔧 Solution Paths

### Option A: Accept & Test Integration (Recommended)

```typescript
// Nie walcz z circular deps
// Użyj prawdziwego store w testach

import { renderHook, act, waitFor } from '@testing-library/react'
import { useTripStore } from './tripStore'

describe('tripStore integration', () => {
  beforeEach(() => {
    const store = useTripStore.getState()
    store.resetTrip()
  })
  
  it('loads trip and populates slices', async () => {
    const { result } = renderHook(() => useTripStore())
    
    await act(async () => {
      await result.current.loadTrip(123)
    })
    
    await waitFor(() => {
      expect(result.current.trip).not.toBeNull()
      expect(result.current.days.length).toBeGreaterThan(0)
      expect(result.current.places.length).toBeGreaterThan(0)
    })
  })
})
```

**Pros:**
- ✅ Działa z circular deps
- ✅ Testuje rzeczywiste zachowanie
- ✅ Catch side effects między slices

**Cons:**
- ❌ Wolniejsze niż unit tests
- ❌ Trzeba mockować API

---

### Option B: Refactor to Break Cycles (Long-term)

```typescript
// BEFORE (circular):
// tripStore.ts
import { createAssignmentsSlice } from './slices/assignmentsSlice'
export const useTripStore = create<TripStoreState>((set, get) => ({
  ...createAssignmentsSlice(set, get),
}))

// assignmentsSlice.ts
import type { TripStoreState } from '../tripStore'  // ← CIRCULAR!

// AFTER (acyclic):
// tripStore.ts
import { createAssignmentsSlice } from './slices/assignmentsSlice'
export const useTripStore = create<TripStoreState>((set, get) => ({
  ...createAssignmentsSlice(set, get),
}))

// assignmentsSlice.ts - NO IMPORT!
export function createAssignmentsSlice(
  set: SetState<TripStoreState>,
  get: GetState<TripStoreState>
) {
  return {
    // Use set/get directly, don't import TripStoreState type
  }
}

// types.ts (new file)
export interface TripStoreState {
  // All types here
}
```

**Pros:**
- ✅ Breaks circular deps
- ✅ Enables unit tests
- ✅ Better architecture

**Cons:**
- ❌ Big refactor (10 slices)
- ❌ Risk of breaking existing code
- ❌ Need extensive testing after

---

## 🎯 Recommendation

**For TREK project:**

1. **Short-term (Q3 2026):** Integration tests
   - Accept circular deps as-is
   - Test store as a whole
   - Focus on critical paths (Planner, Admin)

2. **Long-term (Q4 2026+):** Consider refactor
   - Only if adding new slices becomes painful
   - Only if test performance becomes issue
   - Use type-only imports pattern

**Priority:** 🟡 MEDIUM
- Store działa dobrze (131 usages bez problemów)
- Circular deps nie powodują runtime errors
- Główny problem to testability, nie functionality

---

## 📁 Files

**DOT Graph:** `context/map/tripstore-circular-deps.dot`

**To render SVG (requires Graphviz):**
```bash
# Install graphviz (if not installed)
brew install graphviz  # macOS
# or
apt-get install graphviz  # Linux

# Render to SVG
cat context/map/tripstore-circular-deps.dot | \
  dot -Tsvg -Grankdir=TB -Gsplines=ortho > \
  context/map/tripstore-circular-deps.svg

# Open in browser
open context/map/tripstore-circular-deps.svg
```

**Alternative (online):**
1. Copy content of `tripstore-circular-deps.dot`
2. Paste to: https://dreampuf.github.io/GraphvizOnline/
3. View interactive graph

---

**Generated:** 26 lipca 2026, 13:17 PM (UTC+2)  
**Tool:** dependency-cruiser + manual analysis  
**Related:** artifact-1-territory.md (store impact analysis)
