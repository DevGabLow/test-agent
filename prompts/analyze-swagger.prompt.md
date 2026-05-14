---
name: Analyze Swagger
---

description: "Analyze a Swagger/OpenAPI file and extract all endpoints, schemas, and test scenario suggestions for BDD automation."
argument-hint: "Swagger file path or URL"
---

Analyze the provided Swagger/OpenAPI specification and produce a comprehensive analysis document.

## Input
- Swagger file path or URL: `{{swaggerLocation}}`

## Steps
1. Read the Swagger/OpenAPI spec
2. Extract all paths, HTTP methods, parameters, request bodies, and response schemas
3. Resolve all `$ref` schema references
4. Classify each endpoint by CRUD operation type
5. Document authentication/security requirements
6. Generate test scenario suggestions for each endpoint:
   - @happy: minimum valid request → expected success
   - @negative: invalid inputs → error responses
   - @boundary: edge cases → boundary behavior
   - @auth: missing/invalid auth → 401/403
7. Create data type mappings (Swagger → Java → RestAssured)

## Output
Save the analysis to `docs/swagger-analysis.md` with:
- API overview (title, version, base URL)
- Security schemes
- Feature groups (by tag) with endpoint tables
- Schema definitions with field types and required flags
- Per-endpoint scenario suggestions
- Data type mapping table
