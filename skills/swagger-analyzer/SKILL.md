---
name: swagger-analyzer
description: "Analyze Swagger/OpenAPI specs (JSON/YAML) to extract endpoints, HTTP methods, parameters, schemas, auth requirements, and generate scenario suggestions. Use when: analyzing swagger, openapi spec, API documentation, extracting API structure for test generation."
argument-hint: "Swagger file path or URL"
user-invocable: true
---

# Swagger/OpenAPI Analyzer

## What This Skill Does
Reads a Swagger/OpenAPI specification file (JSON or YAML) and produces a structured analysis document that feeds into the Gherkin generator and step definition generator.

## When to Use
- Starting test automation for a new API
- Analyzing a Swagger spec to understand API structure
- Preparing input for BDD scenario generation
- Documenting API endpoints and their data types

## Procedure

### 1. Read the Spec
Read the Swagger file from:
- Local file: `read_file` tool
- URL: `fetch_webpage` tool
- GitHub raw: `fetch_webpage` with raw URL

### 2. Parse and Extract
For each path in the spec, document:
- **HTTP Method**: GET, POST, PUT, PATCH, DELETE
- **Summary/Description**: What the endpoint does
- **OperationId**: Unique identifier
- **Tags**: Feature grouping
- **Parameters**: Path, query, header, body — with names, types, required flag
- **Request Body**: Schema reference, required fields
- **Responses**: Status codes (200, 201, 400, 401, 403, 404, 422, 500) with schemas
- **Security**: Which security schemes apply

### 3. Resolve Schema References
Follow all `$ref` pointers:
- `#/definitions/` (Swagger 2.0)
- `#/components/schemas/` (OpenAPI 3.x)
- Resolve nested objects and arrays
- Document `required` field arrays

### 4. Classify Endpoints
- **CRUD mapping**: POST→Create, GET→Read, PUT→Update, PATCH→PartialUpdate, DELETE→Delete
- **Auth required**: Which security scheme applies
- **Idempotency**: PUT/DELETE are idempotent, POST is not

### 5. Generate Data Type Mappings
| Swagger/OpenAPI Type | Java Type | RestAssured Getter |
|---------------------|-----------|-------------------|
| `integer` (int32) | `Integer` | `.getInt()` |
| `integer` (int64) | `Long` | `.getLong()` |
| `number` (float) | `Float` | `.getFloat()` |
| `number` (double) | `Double` | `.getDouble()` |
| `string` | `String` | `.getString()` |
| `string` (date-time) | `String` | `.getString()` |
| `boolean` | `Boolean` | `.getBoolean()` |
| `array` | `List<T>` | `.getList()` |
| `object` / `$ref` | `Map<String, ?>` | `.getMap()` |

**CRITICAL**: `integer` with `format: int64` MUST map to `Long`, never `Integer`.

### 6. Suggest Test Scenarios
For each endpoint, suggest:
- **@happy**: Minimum valid request → expected success response
- **@negative**: Missing required fields, invalid types, invalid values → error responses
- **@boundary**: Edge cases — empty strings, max lengths, boundary numeric values, special characters
- **@auth**: Missing/expired/invalid auth token → 401/403

### 7. Output the Analysis
Save to `docs/swagger-analysis.md` with the structure defined in the agent instructions.

## Resources
- [Swagger Specification v2.0](https://swagger.io/specification/v2/)
- [OpenAPI Specification v3.x](https://spec.openapis.org/oas/latest.html)
- [RestAssured Usage Guide](https://github.com/rest-assured/rest-assured/wiki/Usage)
