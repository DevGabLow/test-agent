---
name: step-definition-generator
description: "Generate Java step definition classes from .feature files using Cucumber JUnit 5 + RestAssured. Use when: implementing cucumber steps, creating step definitions, writing glue code for feature files, Java BDD test implementation."
argument-hint: "Feature file to implement step definitions for"
user-invocable: true
---

# Step Definition Generator

## What This Skill Does
Generates Java step definition classes that connect Gherkin feature files to RestAssured HTTP calls. Produces JUnit 5 + Cucumber Platform Engine compatible code.

## When to Use
- After `.feature` files are created
- Need to implement glue code for Cucumber scenarios
- Building the Java layer between Gherkin and HTTP calls
- Adding steps for a new feature to existing test suite

## Prerequisites
- `docs/swagger-analysis.md` (for schema/type information)
- `.feature` files in `src/test/resources/features/`
- Project infrastructure (TestContext, Hooks, CucumberTestSuite, CommonSteps skeleton)
- Maven with `cucumber-junit-platform-engine` (not `cucumber-junit`)

## Procedure

### 1. Read Feature File
Read the `.feature` file to identify all steps that need implementation.

### 2. Classify Steps
- **Reusable (CommonSteps)**: Base URL, headers, generic HTTP requests, status code assertions, JSON field assertions
- **Feature-specific (FeatureSteps)**: Steps unique to this feature (e.g., specific payloads, business logic assertions)

### 3. Check CommonSteps Coverage
Read `CommonSteps.java` to see which steps already exist:
- `setBaseUrl` — "a URL base da API é {string}"
- `setHeaders` — "eu tenha os headers:"
- `setPayload` — "eu tenha o payload:"
- `setFormParams` — "eu tenha os form params:"
- `sendRequest` — "envio uma requisição {string} para {string}"
- `sendRequestWithPayload` — "envio uma requisição {string} para {string} com payload"
- `sendRequestWithFormParams` — "envio uma requisição {string} para {string} com form params"
- `verifyStatusCode` — "o status code da resposta deve ser {int}"
- `verifyIntField` / `verifyLongField` / `verifyStringField` / `verifyBooleanField`
- `verifyFieldContains` / `verifyArraySize` / `verifyHeader`

### 4. Add Missing Common Steps
If a step is reusable across features, add it to `CommonSteps.java`.

### 5. Create Feature-Specific Steps Class
File: `src/test/java/steps/{Tag}Steps.java`
Package: `steps`

Template:
```java
package steps;

import context.TestContext;
import io.cucumber.java.pt.Dado;
import io.cucumber.java.pt.Quando;
import io.cucumber.java.pt.Entao;
import io.restassured.response.Response;
import static org.assertj.core.api.Assertions.assertThat;

public class {Tag}Steps {

    // Feature-specific step definitions here
}
```

**Note**: Use `io.cucumber.java.pt.*` (Portuguese locale) if you enable Cucumber i18n. Otherwise, use `io.cucumber.java.en.*` (default).

### 6. Implement Each Step

#### Pattern: Given — Set context data
```java
@Given("eu tenha um {string} existente com id {int}")
public void iHaveAnExistingResource(String resource, int id) {
    TestContext.get().set("resourceType", resource);
    TestContext.get().set("resourceId", id);
}
```

#### Pattern: When — Make HTTP request
```java
@Quando("envio uma requisição {string} para {string}")
public void sendRequest(String method, String endpoint) {
    // Substituted by CommonSteps — this is the reference pattern
}
```

#### Pattern: Then — Assert response
```java
@Entao("o campo {string} deve ser maior que {int}")
public void verifyFieldGreaterThan(String jsonPath, int minValue) {
    Response response = TestContext.get().getResponse();
    Number value = response.jsonPath().get(jsonPath);
    assertThat(value.longValue()).isGreaterThan(minValue);
}
```

### 7. Type Safety — Use Correct Getters
| JSON Path returns | Use |
|------------------|-----|
| Integer/int32 | `response.jsonPath().getInt("field")` |
| Long/int64 | `response.jsonPath().getLong("field")` |
| Double/number | `response.jsonPath().getDouble("field")` |
| String | `response.jsonPath().getString("field")` |
| Boolean | `response.jsonPath().getBoolean("field")` |
| List | `response.jsonPath().getList("field")` |
| Map | `response.jsonPath().getMap("field")` |

**NEVER** use raw `get()` + cast — always use typed getters.

### 8. Handle DataTables Correctly

#### Form Params with duplicates (CRITICAL)
```java
// DataTable on WHEN step — receives List<Map<String, String>> directly
@Quando("envio uma requisição {string} para {string} com form params")
public void sendRequestWithFormParams(String method, String endpoint, List<Map<String, String>> rows) {
    for (Map<String, String> row : rows) {
        String name = row.get("name");
        String value = row.get("value");
        // Build form params...
    }
}
```

#### Headers (unique names — can use Map)
```java
@Given("eu tenha os headers:")
public void setHeaders(List<Map<String, String>> rows) {
    Map<String, String> headers = new HashMap<>();
    for (Map<String, String> row : rows) {
        headers.put(row.get("name"), row.get("value"));
    }
    TestContext.get().setHeaders(headers);
}
```

### 9. Arity Verification
For every step definition, verify:
- Number of `{string}` + `{int}` + `{float}` + `{word}` placeholders = number of non-DataTable parameters
- +1 parameter if step has an immediate DataTable or docstring
- The DataTable/docstring parameter is LAST in the method signature

### 10. Update Glue Configuration
Ensure `CucumberTestSuite.java` has:
```java
@ConfigurationParameter(key = GLUE_PROPERTY_NAME, value = "steps, hooks, context, runner")
```

## Common Step Patterns Reference

See [CommonSteps Template](./assets/CommonSteps.java.template) for the full implementation of shared HTTP steps.

## Output
- New file: `src/test/java/steps/{Tag}Steps.java`
- Updated: `src/test/java/steps/CommonSteps.java` (if new reusable steps added)
