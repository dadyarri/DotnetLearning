---
stage: 2
status: done
---
Многие методы LINQ не выполняются сразу, а дают `IEnumerable`, который можно материализовать позднее (в `foreach` или `.ToList()`/`.ToArray()`).

- `Where`
- `Take`
- `Select`

Материализующие методы:

- `ToList`
- `ToArray`
- `Count`
- `Sum`