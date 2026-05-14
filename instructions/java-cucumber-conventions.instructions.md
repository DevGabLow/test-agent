---
name: Java Cucumber Conventions
---

description: "Java, Cucumber, and RestAssured coding conventions for BDD API testing. Use when: writing Java test code, cucumber step definitions, RestAssured HTTP calls, BDD test implementation, Java 17+ testing patterns."
applyTo: "src/test/java/**/*.java"
---

# Java + Cucumber + RestAssured Conventions

## JUnit 5 ONLY (NEVER JUnit 4)
- Runner: `@Suite` + `@IncludeEngines("cucumber")` + `@SelectClasspathResource("features")`
- Deps: `cucumber-junit-platform-engine` + `junit-platform-suite`
- NEVER: `cucumber-junit`, `@RunWith`, `@CucumberOptions`, `org.junit.runner`

## Type map (Swagger → Java → RestAssured)
| Swagger | Java | Getter |
|---------|------|--------|
| int32 | Integer/int | `.getInt()` |
| int64 | Long/long | `.getLong()` |
| number | Double | `.getDouble()` |
| string | String | `.getString()` |
| boolean | Boolean | `.getBoolean()` |
| array | List<?> | `.getList()` |

## Critical rules
- **int64 → Long**, NEVER `Integer` (ClassCastException)
- **DataTable dup keys** → `List<Map<String,String>>`, NEVER `Map<String,String>`
- **Arity**: Gherkin args == Java params. +1 for DataTable/docstring on same step.
- **TestContext**: ThreadLocal singleton for state sharing
- **Step imports**: `io.cucumber.java.en.*` (English annotations), NEVER `io.cucumber.java.pt.*`

## RestAssured execute pattern
```java
private Response execute(RequestSpecification req, String method, String endpoint) {
    String url = TestContext.get().getBaseUrl() + endpoint;
    return switch (method.toUpperCase()) {
        case "GET" -> req.get(url);
        case "POST" -> req.post(url);
        case "PUT" -> req.put(url);
        case "DELETE" -> req.delete(url);
        default -> throw new IllegalArgumentException("Unsupported: " + method);
    };
}
```
