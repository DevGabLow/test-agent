# Regras Críticas — BDD com Cucumber + RestAssured

## Gherkin
- `Feature:` obrigatório 1ª linha após tags (senão FeatureParserException)
- Keywords EN (Given/When/Then), texto PT-BR
- Tags: `@{tag}` `@regression` + `@happy`/`@negative`/`@boundary`/`@auth` + `@ctNNNN`
- DataTable no step IMEDIATAMENTE acima (arity): mesmo step OU two-step (Given→TestContext, When lê)

## JUnit 5 (NUNCA JUnit 4)
- `cucumber-junit-platform-engine` + `junit-platform-suite`
- Runner: `@Suite` + `@IncludeEngines("cucumber")` + `@SelectClasspathResource("features")`
- PROIBIDO: `cucumber-junit`, `@RunWith`, `@CucumberOptions`

## Tipagem
- Swagger int32→Integer, int64→Long, number→Double
- DataTable dup keys (name|value): `List<Map<String,String>>`, NUNCA `Map<String,String>`

## Pipeline SSD (sequencial, 1 feature por vez)
1. Setup: plan.md → swagger-analysis.md → infra (pom.xml, TestContext, Hooks, Runner, CommonSteps skeleton)
2. Por feature: Tasks → Gherkin → CommonSteps+ → Steps → Mini-Gate (BLOQUEANTE)
3. Wrap-up: Regressão → quality-report.md

## RestAssured Armadilhas
- `RequestSpecification.get(path)` dispara HTTP GET, NÃO é getter
- `RequestSpecification.getHeaders()` NÃO EXISTE
- Para form params/multipart: use `.spec(existingSpec)` + `.contentType(...)` — NUNCA copie headers de Response
- `ProtocolException: Transfer-encoding header already present` = copiou headers de Response para Request

