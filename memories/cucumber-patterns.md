---
name: Cucumber Patterns
---

# Cucumber Patterns — Ambiguidades

## {int} vs {long} — Ambiguidade
- Step definitions com `{int}` e `{long}` no mesmo padrão causam `AmbiguousStepDefinitionsException`
- Ambos casam com literais inteiros
- **Solução**: Padrões distintos:
  - `"o campo {string} deve ser {int}"`
  - `"o campo {string} deve ser o numero {long}"`

## -Dcucumber.filter.tags ignorado com cucumber.features
- Quando Cucumber descobre features via `cucumber.features`, `-Dcucumber.filter.tags` é ignorado
- Regressão completa é o comportamento padrão
