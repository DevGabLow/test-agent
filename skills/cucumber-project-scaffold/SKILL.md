---
name: cucumber-project-scaffold
description: "Scaffold a complete Cucumber + RestAssured JUnit 5 project from scratch. Use when: creating new BDD testing project, setting up cucumber maven project, initializing API test automation workspace, scaffolding test infrastructure."
argument-hint: "Project name and base package"
user-invocable: true
---

# Cucumber Project Scaffolder

## What This Skill Does
Creates a complete, ready-to-run Maven project for Cucumber BDD + RestAssured API testing with JUnit 5, Java 17+, following all AutomatoBCB conventions.

## When to Use
- Starting a new API test automation project
- Setting up the initial project structure before any features are written
- Need a standardized project template

## Procedure

### 1. Gather Project Info
Ask the user:
- **GroupId**: e.g., `com.example`
- **ArtifactId**: e.g., `petstore-tests`
- **Base package**: e.g., `com.example.petstore`
- **Java version**: 17+ (default)

### 2. Create Maven pom.xml
Use `./assets/pom.xml.template` as the base. Key dependencies:
| Dependency | Version | Scope |
|-----------|---------|-------|
| `cucumber-java` | 7.20.1+ | test |
| `cucumber-junit-platform-engine` | 7.20.1+ | test |
| `junit-platform-suite` | 1.11.0+ | test |
| `rest-assured` | 5.5.7+ | test |
| `jackson-databind` | 2.18.0+ | test |
| `assertj-core` | 3.26.0+ | test |

**CRITICAL**: NEVER use `cucumber-junit` (JUnit 4) — use `cucumber-junit-platform-engine` (JUnit 5).

### 3. Create Directory Structure
```
{project}/
├── pom.xml
├── src/
│   └── test/
│       ├── java/
│       │   ├── runner/
│       │   │   └── CucumberTestSuite.java
│       │   ├── steps/
│       │   │   └── CommonSteps.java
│       │   ├── hooks/
│       │   │   └── Hooks.java
│       │   └── context/
│       │       └── TestContext.java
│       └── resources/
│           └── features/
│               └── .gitkeep
├── docs/
│   └── swagger-analysis.md
├── tasks/
└── plan.md
```

### 4. Create CucumberTestSuite.java
```java
package runner;

import org.junit.platform.suite.api.*;
import static io.cucumber.junit.platform.engine.Constants.*;

@Suite
@IncludeEngines("cucumber")
@SelectClasspathResource("features")
@ConfigurationParameter(key = GLUE_PROPERTY_NAME, value = "steps, hooks, context, runner")
@ConfigurationParameter(key = PLUGIN_PROPERTY_NAME,
    value = "pretty, html:target/cucumber-reports/cucumber.html, json:target/cucumber-reports/cucumber.json")
@ConfigurationParameter(key = FILTER_TAGS_PROPERTY_NAME, value = "not @ignore")
public class CucumberTestSuite {
    // Empty class — configuration via annotations
}
```

### 5. Create TestContext.java
Thread-safe context using `ThreadLocal`:
- `baseUrl`: String
- `request`: RequestSpecification
- `response`: Response
- `storage`: Map<String, Object> (generic context storage)
- `formParams`: List<Map<String, String>>
- `requestPayload`: String
- `headers`: Map<String, String>

```java
package context;

import io.restassured.response.Response;
import io.restassured.specification.RequestSpecification;
import java.util.*;

public class TestContext {

    private static final ThreadLocal<TestContext> INSTANCE =
        ThreadLocal.withInitial(TestContext::new);

    private String baseUrl;
    private RequestSpecification request;
    private Response response;
    private final Map<String, Object> storage = new HashMap<>();
    private List<Map<String, String>> formParams;
    private String requestPayload;
    private Map<String, String> headers = new HashMap<>();

    public static TestContext get() { return INSTANCE.get(); }
    public static void clear() { INSTANCE.remove(); }

    // Getters and setters...
    public String getBaseUrl() { return baseUrl; }
    public void setBaseUrl(String baseUrl) { this.baseUrl = baseUrl; }
    public RequestSpecification getRequest() { return request; }
    public void setRequest(RequestSpecification request) { this.request = request; }
    public Response getResponse() { return response; }
    public void setResponse(Response response) { this.response = response; }
    public void set(String key, Object value) { storage.put(key, value); }
    @SuppressWarnings("unchecked")
    public <T> T get(String key) { return (T) storage.get(key); }
    public List<Map<String, String>> getFormParams() { return formParams; }
    public void setFormParams(List<Map<String, String>> formParams) { this.formParams = formParams; }
    public String getRequestPayload() { return requestPayload; }
    public void setRequestPayload(String requestPayload) { this.requestPayload = requestPayload; }
    public Map<String, String> getHeaders() { return headers; }
    public void setHeaders(Map<String, String> headers) { this.headers = headers; }
}
```

### 6. Create Hooks.java
```java
package hooks;

import context.TestContext;
import io.cucumber.java.After;
import io.cucumber.java.Before;

public class Hooks {

    @Before
    public void setUp() {
        // TestContext is auto-initialized via ThreadLocal
    }

    @After
    public void tearDown() {
        TestContext.clear();
    }
}
```

### 7. Create CommonSteps.java (SKELETON)
```java
package steps;

import context.TestContext;
import io.cucumber.java.pt.Dado;

public class CommonSteps {

    @Dado("a URL base da API é {string}")
    public void setBaseUrl(String url) {
        TestContext.get().setBaseUrl(url);
    }
}
```

### 8. Create plan.md
```markdown
# Plano de Testes Automatizados — {Project Name}

## Visão Geral
- **API**: {Swagger title}
- **Base URL**: {baseUrl}
- **Framework**: Cucumber + RestAssured + JUnit 5
- **Java**: 17+

## Features
| Feature | Tag | Endpoints | Estimativa |
|---------|-----|-----------|------------|
| {Name} | @{tag} | N | X cenários |

## Pipeline
1. Pré-iteração: Infraestrutura + Análise Swagger
2. Iteração por Feature: SPEC → Tasks → Gherkin → Steps → Mini-Gate
3. Pós-iteração: Quality Gate + Rastreabilidade
```

### 9. Verify
Run `mvn test` to verify:
- Project compiles
- Cucumber finds 0 scenarios (no .feature files yet)
- BUILD SUCCESS

## Output
Complete project directory with all files ready for feature generation.

## Templates
- [pom.xml template](./assets/pom.xml.template)
- [CucumberTestSuite.java template](./assets/CucumberTestSuite.java.template)
- [TestContext.java template](./assets/TestContext.java.template)
- [Hooks.java template](./assets/Hooks.java.template)
- [CommonSteps.java template](./assets/CommonSteps.java.template)
