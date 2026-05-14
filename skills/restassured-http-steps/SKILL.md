---
name: restassured-http-steps
description: "Create and maintain CommonSteps.java with RestAssured HTTP methods for Cucumber BDD tests. Use when: setting up HTTP step definitions, configuring base URL, request building, response handling, authentication setup in CommonSteps."
argument-hint: "HTTP method or feature to add steps for"
user-invocable: true
---

# RestAssured HTTP Common Steps

## What This Skill Does
Creates and maintains the `CommonSteps.java` file — the shared step definitions that handle all HTTP communication via RestAssured. This is the core infrastructure that all feature-specific step classes depend on.

## When to Use
- Initial project setup (creates the skeleton)
- First feature implementation (adds ~16 core HTTP steps)
- Adding new HTTP methods or patterns
- Adding authentication handling
- Adding multipart/form-data support

## Prerequisites
- `TestContext.java` must exist in `context` package
- Maven with `rest-assured` dependency (5.5.7+ or 6.0.0+ for Java 17)
- `jackson-databind` for JSON serialization

## Procedure

### 1. Initial Skeleton (Project Setup)
Create the minimal CommonSteps with only base URL support:
```java
package steps;

import context.TestContext;
import io.cucumber.java.pt.Dado;
import io.restassured.RestAssured;

public class CommonSteps {

    @Dado("a URL base da API é {string}")
    public void setBaseUrl(String url) {
        TestContext.get().setBaseUrl(url);
    }
}
```

### 2. First Feature — Add Core HTTP Steps
When the first feature is implemented, add these ~16 core steps:

#### Request Building Steps
```java
@Dado("eu tenha os headers:")
public void setHeaders(List<Map<String, String>> rows) {
    Map<String, String> headers = new HashMap<>();
    for (Map<String, String> row : rows) {
        headers.put(row.get("name"), row.get("value"));
    }
    TestContext.get().setHeaders(headers);
}

@Dado("eu tenha o payload:")
public void setPayload(String payload) {
    TestContext.get().setRequestPayload(payload);
}

@Dado("eu tenha o payload do arquivo {string}")
public void setPayloadFromFile(String fileName) {
    String payload = new String(Files.readAllBytes(
        Paths.get("src/test/resources/payloads/" + fileName)));
    TestContext.get().setRequestPayload(payload);
}

@Dado("eu tenha os form params:")
public void setFormParams(List<Map<String, String>> rows) {
    TestContext.get().setFormParams(rows);
}

@Dado("eu esteja autenticado com token {string}")
public void setAuthToken(String token) {
    Map<String, String> headers = TestContext.get().getHeaders();
    headers.put("Authorization", "Bearer " + token);
}

@Dado("eu tenha a api key {string}")
public void setApiKey(String apiKey) {
    Map<String, String> headers = TestContext.get().getHeaders();
    headers.put("api_key", apiKey);
}
```

#### Request Execution Steps
```java
@Quando("envio uma requisição {string} para {string}")
public void sendRequest(String method, String endpoint) {
    RequestSpecification request = RestAssured.given()
        .baseUri(TestContext.get().getBaseUrl())
        .headers(TestContext.get().getHeaders());
    Response response = execute(request, method, endpoint);
    TestContext.get().setResponse(response);
}

@Quando("envio uma requisição {string} para {string} com payload")
public void sendRequestWithPayload(String method, String endpoint) {
    RequestSpecification request = RestAssured.given()
        .baseUri(TestContext.get().getBaseUrl())
        .headers(TestContext.get().getHeaders())
        .contentType(ContentType.JSON)
        .body(TestContext.get().getRequestPayload());
    Response response = execute(request, method, endpoint);
    TestContext.get().setResponse(response);
}

@Quando("envio uma requisição {string} para {string} com form params")
public void sendRequestWithFormParams(String method, String endpoint, List<Map<String, String>> rows) {
    RequestSpecification request = RestAssured.given()
        .baseUri(TestContext.get().getBaseUrl())
        .headers(TestContext.get().getHeaders());
    for (Map<String, String> row : rows) {
        request.formParam(row.get("name"), row.get("value"));
    }
    Response response = execute(request, method, endpoint);
    TestContext.get().setResponse(response);
}
```

#### Response Assertion Steps
```java
@Entao("o status code da resposta deve ser {int}")
public void verifyStatusCode(int expectedStatus) {
    Response response = TestContext.get().getResponse();
    assertThat(response.getStatusCode()).isEqualTo(expectedStatus);
}

@Entao("o campo {string} deve ser {int}")
public void verifyIntField(String jsonPath, int expected) {
    Response response = TestContext.get().getResponse();
    assertThat(response.jsonPath().getInt(jsonPath)).isEqualTo(expected);
}

@Entao("o campo {string} deve ser {long}")
public void verifyLongField(String jsonPath, long expected) {
    Response response = TestContext.get().getResponse();
    assertThat(response.jsonPath().getLong(jsonPath)).isEqualTo(expected);
}

@Entao("o campo {string} deve ser {string}")
public void verifyStringField(String jsonPath, String expected) {
    Response response = TestContext.get().getResponse();
    assertThat(response.jsonPath().getString(jsonPath)).isEqualTo(expected);
}

@Entao("o campo {string} deve ser verdadeiro")
public void verifyBooleanFieldTrue(String jsonPath) {
    Response response = TestContext.get().getResponse();
    assertThat(response.jsonPath().getBoolean(jsonPath)).isTrue();
}

@Entao("o campo {string} deve conter {string}")
public void verifyFieldContains(String jsonPath, String substring) {
    Response response = TestContext.get().getResponse();
    assertThat(response.jsonPath().getString(jsonPath)).contains(substring);
}

@Entao("o array {string} deve ter tamanho {int}")
public void verifyArraySize(String jsonPath, int expectedSize) {
    Response response = TestContext.get().getResponse();
    List<?> list = response.jsonPath().getList(jsonPath);
    assertThat(list).hasSize(expectedSize);
}

@Entao("o array {string} não deve estar vazio")
public void verifyArrayNotEmpty(String jsonPath) {
    Response response = TestContext.get().getResponse();
    List<?> list = response.jsonPath().getList(jsonPath);
    assertThat(list).isNotEmpty();
}

@Entao("o header {string} deve ser {string}")
public void verifyHeader(String headerName, String expectedValue) {
    Response response = TestContext.get().getResponse();
    assertThat(response.getHeader(headerName)).isEqualTo(expectedValue);
}
```

#### Helper Method
```java
private Response execute(RequestSpecification request, String method, String endpoint) {
    String fullUrl = TestContext.get().getBaseUrl() + endpoint;
    return switch (method.toUpperCase()) {
        case "GET" -> request.get(fullUrl);
        case "POST" -> request.post(fullUrl);
        case "PUT" -> request.put(fullUrl);
        case "PATCH" -> request.patch(fullUrl);
        case "DELETE" -> request.delete(fullUrl);
        default -> throw new IllegalArgumentException("Método HTTP não suportado: " + method);
    };
}
```

### 3. Incremental Additions (Subsequent Features)
Later features may add 0-2 steps each:
- File upload (multipart)
- Query parameter handling
- Response time assertions
- Response schema validation
- Specific authentication flows (OAuth2, Basic Auth)

### 4. Maven Dependencies (in pom.xml)
```xml
<dependency>
    <groupId>io.rest-assured</groupId>
    <artifactId>rest-assured</artifactId>
    <version>${rest-assured.version}</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>${jackson.version}</version>
    <scope>test</scope>
</dependency>
```

For Java 17+, use RestAssured 5.5.7+ or 6.0.0+.

## Type Safety Reminders
- `int64` → use `getLong()`, not `getInt()`
- `number` → use `getDouble()` or `getFloat()` depending on format
- `array` → use `getList()` with proper generic type
- Mixing `get()` + cast is error-prone — prefer typed getters
- `response.jsonPath().get("id")` on int64 returns `Long` → `Long petId = response.jsonPath().getLong("id")`

## Output
- Primary: `src/test/java/steps/CommonSteps.java`
- May update: `pom.xml` (if dependencies added)
