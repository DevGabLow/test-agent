---
name: Restassured Jsonpath
---

# RestAssured JSONPath — Sem `$` no início

## Problema
`$[0].id` não funciona no RestAssured

## Solução
Usar `[0].id` para acessar primeiro elemento do array raiz.
O `$` prefixo não é reconhecido pelo JSONPath do RestAssured.
