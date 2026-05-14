---
name: Gherkin Conventions
description: "Gherkin writing conventions for Cucumber .feature files in Brazilian Portuguese. Use when: writing feature files, creating scenarios, gherkin syntax, BDD specifications, cucumber feature format."
applyTo: "src/test/resources/features/**/*.feature"
---

# Gherkin Conventions (PT-BR)

## Mandatory
- `Feature:` MUST be 1st line after tags (else `FeatureParserException`)
- Order: `@tags` → `Feature:` → `Background:` → `Scenario:` / `Scenario Outline:`
- Keywords: **EN** (Given/When/Then/And/But). Text: **PT-BR**.
- Scenario descriptions: PT-BR

## Tags
| Tag | Purpose |
|-----|---------|
| `@{swagger_tag}` | Feature group |
| `@happy` / `@negative` / `@boundary` | Test type |
| `@auth` | Auth scenarios |
| `@regression` | Full suite |
| `@ignore` | Skip |
| `@ctNNNN` | Scenario ID |

## DataTable
- `name \| value` (unique) → headers, query params
- `name \| value` (dup names) → form params → `List<Map<String,String>>`, NOT `Map<String,String>`
- DataTable belongs to step IMMEDIATELY above it → arity: same step OR two-step (Given→TestContext, When reads)

## Step patterns (EN keywords, PT-BR text)
```
Given a URL base da API é "{url}"
Given eu tenha os headers:          ← DataTable name|value
Given eu tenha o payload:           ← multiline JSON
Given eu tenha os form params:      ← DataTable name|value (dup ok)
When envio uma requisição "{METHOD}" para "{endpoint}"
When envio uma requisição "{METHOD}" para "{endpoint}" com payload
When envio uma requisição "{METHOD}" para "{endpoint}" com form params
Then o status code da resposta deve ser {int}
Then o campo "{jsonPath}" deve ser {int} / "{string}" / verdadeiro / falso / o numero {long}
Then o campo "{jsonPath}" deve conter "{string}"
Then o array "{jsonPath}" deve ter tamanho {int} / não deve estar vazio
```

## Batch split (≥15 scenarios)
- Lote A: @happy (~40%) → Lote B: @negative+@boundary (~50%) → Lote C: @auth (~10%)
