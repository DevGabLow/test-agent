---
name: Sdd Harness
---

# SDD Test Automation Harness

## Pipeline Obrigatório (sequência fixa)
```
SPEC → architecture-expert → gherkin-generator → cucumber-java-automator
     → architecture-validator → test-quality-validator → test-executor
```
- Nunca pular passos; cada agente atualiza `pipeline.state.json`
- test-executor roda **feature por feature**, nunca tudo junto
- Features com `status: "PASSED"` não são re-executadas
- 2 falhas consecutivas no mesmo passo → bloquear e pedir intervenção humana

## Stack Canônico
- Java 17, Cucumber 7.18.0, RestAssured 5.4.0, AssertJ 3.25.3
- JUnit Platform 1.10.2, Awaitility 4.2.1
- Allure 2.27+ (`allure-cucumber7-jvm` + `allure-maven`) — único reporter tolerado
- Maven build manager

## Estrutura de Pastas (invariável)
```
src/test/
  java/
    steps/      → <Domain>Steps.java
    runners/    → <Domain>Runner.java
    support/    → Hooks.java, TestContext.java
    clients/    → <Domain>Client.java
  resources/
    features/   → <domain>/snake_case.feature
    schemas/    → JSON schemas para contract testing
    specs/      → OpenAPI/Swagger specs (1 por feature/domínio)
    fixtures/   → dados de teste JSON (não hardcoded nos steps)
```

## Convenções de Nomenclatura
- Feature files: `snake_case.feature`
- Step classes: `<Domain>Steps.java` (PascalCase)
- Runners: `<Domain>Runner.java`
- Clients: `<Domain>Client.java`
- Tags Gherkin: `@smoke`, `@regression`, `@contract`, `@negative`, `@<domain>`

## Arquivos de Memória do Pipeline (`.github/harness-memory/`)
| Arquivo | Gerado por | Conteúdo |
|---------|-----------|----------|
| `pipeline.state.json` | todos | step atual + steps_completed + blocked |
| `spec.lock.json` | architecture-expert | endpoints + operationIds + schemas |
| `architecture.snapshot.json` | architecture-expert | pacotes + stack + conventions |
| `coverage.map.json` | gherkin-generator | cobertura de operações por feature |
| `validation.report.json` | arch-validator + quality-validator | violações + score |
| `execution.report.json` | test-executor | resultados por feature |
| `feature-sessions/<feature>.session.json` | test-executor | detalhes de falhas por feature |

## Regras de Qualidade Rígidas
- Step definitions NÃO contêm lógica de negócio → delegar para `clients/`
- Sem `Thread.sleep()` → usar Awaitility
- Dados de teste via `Examples:` ou fixtures JSON, jamais hardcoded nos steps
- Cada cenário tem exatamente uma responsabilidade
- URLs hardcoded em steps são WARNING → mover para fixtures/

## Comando Maven por Feature
```bash
mvn clean test -Dtest=<Domain>Runner -Dsurefire.failIfNoSpecifiedTests=false
```
