---
name: quality-gate
description: "Run Cucumber tests, validate pass rates, and generate quality reports. Use when: running test suite, checking test results, generating quality metrics, validating mini-gate before proceeding, creating quality-report.md."
argument-hint: "Feature tag or 'all' for full regression"
user-invocable: true
---

# Quality Gate

## What This Skill Does
Runs Cucumber tests via Maven, validates results against quality thresholds, and generates comprehensive quality reports. Acts as the Mini-Gate (per feature) and Final Quality Gate (post-iteration).

## When to Use
- After implementing step definitions for a feature (Mini-Gate)
- After all features are implemented (Final Quality Gate)
- Before committing/pushing code
- When diagnosing test failures
- When generating quality metrics

## Procedure

### 1. Mini-Gate (Per Feature)
Run tests for a specific feature tag:

```bash
mvn test -Dcucumber.filter.tags="@{tag} and not @ignore"
```

#### Pass Criteria
- ALL scenarios pass (0 failures, 0 errors)
- No pending/undefined steps
- Build succeeds

#### Failure Handling
If Mini-Gate fails:
1. Read the Cucumber report at `target/cucumber-reports/cucumber.json`
2. Identify failing scenarios
3. Diagnose root cause:
   - **Arity mismatch**: Step definition has wrong number of parameters
   - **Type mismatch**: Using `Integer` where `Long` is needed (int64 vs int32)
   - **Missing step**: No matching step definition found
   - **Assertion failure**: Expected vs actual mismatch
   - **HTTP error**: API returned unexpected status code
   - **Connection error**: Base URL wrong or API unreachable
4. Fix the issue
5. **Re-run Mini-Gate** — DO NOT proceed until all pass

### 2. Full Regression
```bash
mvn test
```

Runs ALL features. Generates reports at:
- HTML: `target/cucumber-reports/cucumber.html`
- JSON: `target/cucumber-reports/cucumber.json`

### 3. Generate Quality Report
Create `quality-report.md`:

```markdown
# Quality Report — {Project Name}
**Date**: {date}
**Environment**: {baseUrl}

## Summary
| Metric | Value |
|--------|-------|
| Total Scenarios | {N} |
| Passed | {N} |
| Failed | {N} |
| Skipped | {N} |
| Pass Rate | {X}% |

## Per Feature
| Feature | Tag | Total | Passed | Failed | Pass Rate |
|---------|-----|-------|--------|--------|-----------|
| Pets | @pet | 15 | 15 | 0 | 100% |
| Store | @store | 10 | 9 | 1 | 90% |

## Tag Coverage
| Tag Type | Count | Pass Rate |
|----------|-------|-----------|
| @happy | 20 | 100% |
| @negative | 12 | 100% |
| @boundary | 8 | 100% |
| @auth | 5 | 80% |

## Failed Scenarios
| Scenario | Feature | Error |
|----------|---------|-------|
| CT0010 | @store | 500 Internal Server Error |
| CT0018 | @auth | Expected 401 but was 200 |

## Traceability
| Swagger Endpoint | Feature | Scenario ID |
|-----------------|---------|-------------|
| GET /pet/{petId} | @pet | CT0001, CT0005 |
| POST /pet | @pet | CT0002, CT0003, CT0004 |

## Recommendations
- Fix CT0010: API returns 500 for invalid order quantity
- Fix CT0018: Auth endpoint does not enforce authentication
```

### 4. Quality Thresholds
| Gate | Threshold | Action on Failure |
|------|-----------|-------------------|
| Mini-Gate | 100% pass | BLOCK — must fix before next feature |
| Final Gate | ≥ 95% pass | Review failures, document known issues |
| @happy | 100% pass | MUST pass — critical path |
| @negative | 100% pass | MUST pass — error handling |
| @auth | 100% pass | MUST pass — security |

### 5. Interpreting Cucumber JSON Report
Parse `target/cucumber-reports/cucumber.json` to extract:
- `elements[].steps[].result.status`: `passed`, `failed`, `skipped`, `undefined`, `pending`
- `elements[].steps[].result.error_message`: Stack trace for failures
- `elements[].tags[].name`: Tags for filtering

### 6. Common Failures and Fixes

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| `ClassCastException: Long cannot be cast to Integer` | int64 schema mapped to Integer | Use `Long` / `getLong()` |
| `UndefinedStepException` | Missing step definition | Add step to CommonSteps or FeatureSteps |
| `CucumberDataTableException: duplicate key` | Used `Map` for form params DataTable | Use `List<Map<String, String>>` |
| `Arity mismatch` | Step has different param count than Gherkin | Match params exactly; check DataTable ownership |
| `FeatureParserException` | Missing `Feature:` line in .feature | Add `Feature:` as first non-tag line |
| `NoSuchMethodError: RunWith` | Using JUnit 4 dependencies | Use `cucumber-junit-platform-engine` |

## Output
- Console: test execution results
- File: `quality-report.md` (final gate)
- Reports: `target/cucumber-reports/` (HTML + JSON)
