# Development Metrics

Ten katalog zawiera analizy i metryki dotyczące aktywności developmentu w projekcie TREK.

## 📊 Dostępne Dokumenty

### [Service Overview](./service-overview.md)
Kompleksowy przewodnik po aplikacji TREK:
- Co to jest TREK i do czego służy
- Pełna lista funkcjonalności (trip planning, collaboration, AI, etc.)
- Szczegółowy tech stack (Frontend, Backend, Database, DevOps)
- Architektura systemu i data flow
- Deployment options i system requirements
- Use cases i porównanie z alternatywami

**Ostatnia aktualizacja:** 26 lipca 2026

### [Timeline Analysis 2026](./timeline-analysis-2026.md)
Miesięczna analiza ewolucji projektu od początku:
- Pierwszy commit: 18 marca 2026 (projekt ma 4.5 miesiąca!)
- Breakdown commitów po miesiącach (1,338 total)
- Focus shift analysis - jak zmieniał się nacisk pracy
- TOP obszary i pliki dla każdego miesiąca
- Trendy i wzorce rozwoju
- Prognozy i strategic insights
- **Kluczowa obserwacja:** Ewolucja od MVP → Platform (Plugin System w lipcu!)

**Ostatnia aktualizacja:** 26 lipca 2026

### [Coupling Analysis](./coupling-analysis.md)
Analiza sprzężeń między obszarami kodu:
- TOP 30 par katalogów modyfikowanych razem
- Deep dive dla TOP 3 obszarów (Planner, Tests, Map)
- TOP 20 trójek (3-way coupling)
- Wzorce coupling: Expected, Test, Cross-Stack, Database
- **Kluczowe problemy:** Brak DTO layer (DB-Frontend coupling), Planner jako god component
- **Mocne strony:** Excellent TDD culture, Clean layer boundaries
- Rekomendacje i action items (Priority 1-3)

**Ostatnia aktualizacja:** 26 lipca 2026

### [Universal Connectors & Validation](./universal-connectors-validation.md) 🆕
Analiza "wspólnych mianowników" i weryfikacja plików:
- TOP 30 universal connectors (pliki zmieniające się z wieloma obszarami)
- **Wiki files touches 341 areas** - Excellent living documentation!
- **migrations.ts touches 320 areas (84%)** - Potwierdza problem coupling
- Weryfikacja istnienia 31 kluczowych plików (30/31 exists = 96.7%)
- **routes → nest migration** (31 maja 2026) - Architectural upgrade
- Breakdown by type: Documentation, Database, API, Services, Core Components

**Ostatnia aktualizacja:** 26 lipca 2026

### [Activity Analysis (Aggregate)](./activity-analysis-2025-2026.md)
Zagregowana analiza całego okresu:
- TOP 10 najczęściej modyfikowanych folderów/modułów
- TOP 10 najczęściej modyfikowanych plików
- Hot spots i obszary intensywnej aktywności
- Analizę według kategorii (Frontend, Backend, Testing)
- Insights i rekomendacje

**Ostatnia aktualizacja:** 26 lipca 2026

## 🔄 Jak Odświeżyć Metryki

Aby wygenerować aktualne metryki, użyj następujących komend w katalogu głównym projektu:

```bash
# Najpopularniejsze foldery (3-4 poziomy, filtrowane)
git log --since="12 months ago" --pretty=format: --name-only | \
  grep -v '^$' | \
  grep -v 'package-lock.json' | \
  grep -v 'package.json' | \
  grep -v '/translations/' | \
  grep -v '/i18n/' | \
  awk -F'/' '{
    if (NF >= 4) print $1"/"$2"/"$3"/"$4
    else if (NF == 3) print $1"/"$2"/"$3
    else if (NF == 2) print $1"/"$2
    else print $1
  }' | \
  sort | uniq -c | sort -rn | head -30

# Najpopularniejsze pliki (filtrowane)
git log --since="12 months ago" --pretty=format: --name-only | \
  grep -v '^$' | \
  grep -v 'package-lock.json' | \
  grep -v 'package.json' | \
  grep -v '\.snap$' | \
  grep -v '\.env' | \
  grep -v 'Chart.yaml' | \
  grep -v '/translations/' | \
  grep -v '/i18n/' | \
  sort | uniq -c | sort -rn | head -50
```

## 📅 Częstotliwość Aktualizacji

**Rekomendowana:** Kwartalnie (co 3 miesiące)

Analiza pomaga zidentyfikować:
- Obszary wymagające refactoringu (zbyt często modyfikowane)
- Hot spots w rozwoju produktu
- Potencjalne bottlenecki
- Trendy w aktywności zespołu

## 🎯 Zastosowania

1. **Planning** - Identyfikacja obszarów do refactoringu
2. **Onboarding** - Pokazanie nowym członkom zespołu, gdzie dzieje się akcja
3. **Architecture Reviews** - Wsparcie dla decyzji architektonicznych
4. **Sprint Planning** - Zrozumienie velocity w różnych obszarach kodu
5. **Technical Debt** - Monitoring obszarów wymagających uwagi

## 📝 Konwencje

- Pliki nazywamy według schematu: `activity-analysis-YYYY-YYYY.md`
- Zawsze dokumentujemy okres analizy i datę wygenerowania
- Filtrujemy szum: lock files, translations, config files
- Focus na code changes, nie na infrastructure/tooling changes
