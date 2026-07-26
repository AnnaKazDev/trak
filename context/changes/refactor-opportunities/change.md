---
change_id: refactor-opportunities
title: Refactor opportunities analysis and prioritization
status: plan_reviewed
created: 2026-07-26
updated: 2026-07-26
archived_at: null
---

## Notes

Intencja: mamy analizę modułu, która dokumentuje dług techniczny
i ryzyka strukturalne: context/changes/plugin-service/research.md. Ta zmiana odpowiada na pytanie, które tamta analiza celowo zostawiła otwarte:
KTÓRE z tych problemów warto naprawić, w jakim docelowym kształcie i w jakiej kolejności. Eksplorujemy każdy zapisany problem w kodzie i historii, a potem porządkujemy jako refactor opportunities.
Zmiana przebiega etapami: eksploracja → decyzja i plan → implementacja. Na etapie eksploracji nie dzieje się żaden refaktor i nie zapada żadna decyzja.
Wynik eksploracji: research.md tej zmiany, zakończony rankingiem opcji z trade-offami. Plan: plan.md / plan-brief.md — scope: protocol versioning + circular-dep break + trip strangler (pierwszy).
