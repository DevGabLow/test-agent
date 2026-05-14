---
name: Test Automation Patterns
---

# Padrões e Lições: Test Automation BDD/Cucumber Java

## Falhas Comuns em APIs Públicas (petstore e similares)

### API_BEHAVIOR_MISMATCH — APIs públicas compartilhadas não se comportam como a spec descreve
- `POST /pet` com dados inválidos → retorna 200 em vez de 405
- `PUT /resource` com ID inexistente → cria recurso em vez de retornar 404
- `GET /resource?status=INVALIDO` → retorna array vazio `[]` em vez de 400
- **Fix**: adaptar cenários `@negative` para refletir comportamento real da API pública;
  ou usar ambiente controlado/mock para testes negativos estritos

### TEST_DATA_ISOLATION — APIs públicas compartilhadas apagam dados entre requests
- Recurso criado no `@Before` pode ser deletado antes do step usar o ID
- **Fix**: criar o recurso no próprio cenário (não no Hooks) OU verificar existência antes de consultar

### CONTRACT_SCHEMA_MISMATCH — Schema de objeto vs. array
- Endpoint retorna array mas schema define `type: object` → erro de validação
- **Fix**: usar `pet-array-schema.json` separado para endpoints que retornam lista;
  ou validar cada item individualmente no step

### CODE_ERROR — RestAssured multipart upload
- `.body(bytes).contentType("image/jpeg")` NÃO funciona para upload de arquivo
- **Fix**: `.multiPart(new File(path))` ou `.multiPart("file", bytes, "image/jpeg")`

## Contract Testing — Padrões Aprendidos
- Schema `login-response-schema.json` deve ter `"required": ["message"]` para reforçar contrato
- Usar `pet-array-schema.json` distinto do `pet-schema.json` para endpoints `/findByStatus`, `/findByTags`
- Schemas em `src/test/resources/schemas/` são copiados para `target/test-classes/schemas/` automaticamente pelo Maven

## Rastreabilidade SDD
- Cobertura medida por operações da spec, não por linhas de código
- Meta: 100% das operações com ao menos 1 cenário no `.feature`
- Meta mínima: operações críticas com `@contract` test

## Tags Recomendadas por Tipo de Cenário
| Tipo | Tags |
|------|------|
| Happy path básico | `@smoke @regression` |
| Validação de schema/contrato | `@contract @regression` |
| Fluxo negativo/erro | `@negative @regression` |
| Smoke rápido | `@smoke` |

## Awaitility — Template de Polling
```java
await().atMost(10, SECONDS)
       .pollInterval(500, MILLISECONDS)
       .until(() -> client.getStatus(id).equals("available"));
```

## TestContext — Compartilhamento entre Steps
- Usar `@ScenarioScoped` (via Cucumber PicoContainer) ou mapa estático com cleanup em `@After`
- Guardar: `latestResponse`, `latestPetId`, `latestOrderId`, `latestUsername`

## Runners — Template mínimo Allure
```java
@Suite
@IncludeEngines("cucumber")
@SelectClasspathResource("features/<domain>")
@ConfigurationParameter(key = PLUGIN_PROPERTY_NAME, value = "io.qameta.allure.cucumber7jvm.AllureCucumber7Jvm")
@ConfigurationParameter(key = FILTER_TAGS_PROPERTY_NAME, value = "not @wip")
public class <Domain>Runner {}
```
