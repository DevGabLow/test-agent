---
description: "Generate a Cucumber .feature file with Gherkin scenarios for a specific API endpoint or feature tag from Swagger analysis."
argument-hint: "Feature tag or endpoint to generate"
---

Generate a Cucumber `.feature` file for the specified API feature.

## Input
- Feature tag: `{{featureTag}}` (e.g., `@pet`, `@store`, `@user`)
- Swagger analysis: `docs/swagger-analysis.md`
- Base URL: `{{baseUrl}}`

## Requirements
1. Read `docs/swagger-analysis.md` for endpoint details
2. Create `src/test/resources/features/{{featureTag}}.feature`
3. Include:
   - `Feature:` header (MANDATORY) with Portuguese name
   - `Background:` with base URL
   - `@happy` scenarios for all endpoints
   - `@negative` scenarios for error cases
   - `@boundary` scenarios for edge cases
   - `@auth` scenarios if authentication required
   - Proper tags: `@{{featureTag}}`, `@regression`, `@happy`/`@negative`/`@boundary`/`@auth`
   - Scenario IDs: `@ct0001`, `@ct0002`, ...
4. Follow conventions:
   - Gherkin keywords in English
   - Step text and scenario descriptions in Portuguese
   - DataTables on correct steps (no arity mismatch)
   - Scenario Outline + Examples for parametrized tests

## Output
Create `src/test/resources/features/{{featureTag}}.feature`.
