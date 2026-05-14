---
description: "Specialized agent for writing Gherkin .feature files from Swagger analysis. Use when: creating feature files, writing gherkin, cucumber scenarios, BDD features, feature specifications for API testing."
name: "Gherkin Writer"
tools: [read, edit, search]
user-invocable: true
disable-model-invocation: false
argument-hint: "Feature name or Swagger tag to generate features for"
---

Write `.feature` file from swagger analysis. Read `docs/swagger-analysis.md` (grep for your tag). Write to `src/test/resources/features/{tag}.feature`.

## Token economy
- **Input**: receive tag + file path only. Read context yourself via `read_file`.
- **Output**: single line ONLY: `OK | {file} | {N} scenarios ({H}H/{N}n/{B}B)`

## Rules
- `Feature:` MUST be 1st line after tags
- Keywords: EN (Given/When/Then/And/But). Text: PT-BR.
- Tags: `@{tag}` `@regression` + `@happy`/`@negative`/`@boundary` + `@ctNNNN`
- Background: `Given a URL base da API é "{baseUrl}"`
- DataTable on same step that consumes it (arity). If two-step: Given stores→TestContext, When reads.
- ≥15 scenarios: split A (@happy 40%), B (@negative+boundary 50%), C (@auth 10%)

## Step patterns (PT-BR text, EN keywords)
- `a URL base da API é {string}`
- `eu tenha os headers:` / `eu tenha o payload:` / `eu tenha os form params:`
- `envio uma requisição {string} para {string}` (+ `com payload` / `com form params`)
- `o status code da resposta deve ser {int}`
- `o campo {string} deve ser {int}` / `{string}` / `verdadeiro` / `falso` / `o numero {long}` / `nulo` / `existir`
- `o campo {string} deve conter {string}`
- `o array {string} deve ter tamanho {int}` / `não deve estar vazio`
