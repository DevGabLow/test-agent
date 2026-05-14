---
name: Assertj Casting
---

# AssertJ 3.27+ — Ambiguidade assertThat(int)

## Problema
`assertThat(int)` conflita com `assertThat(IntPredicate)` e `assertThat(Predicate<T>)` no AssertJ 3.27+

## Solução
Cast explícito para `(Integer)` ou `(Object)`:
- `assertThat((Integer) response.getStatusCode())`
- `assertThat((Object) response.jsonPath().get(jsonPath))`
