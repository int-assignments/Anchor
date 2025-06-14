# Anchor – Home Task

This assignment asks you to build a small HTTP service that manages spreadsheet-like data. Each operation must be exposed through a dedicated HTTP endpoint.

---

## Table of Contents

1. [Project Goals](#project-goals)
2. [API Specification](#api-specification)

   * [Create a Sheet](#1-create-a-sheet)
   * [Set a Cell](#2-set-a-cell)
   * [Retrieve a Sheet](#3-retrieve-a-sheet)
3. [Lookup Function](#lookup-function)
4. [Validation Rules](#validation-rules)
5. [Quality Expectations](#quality-expectations)
6. [Assumptions](#assumptions)
7. [Running the Server](#running-the-server)
8. [Executing the Tests](#executing-the-tests)

---

## Project Goals

* Build a lightweight HTTP server that can

  * Create spreadsheets from a supplied JSON schema.
  * Update individual cells while enforcing type safety.
  * Retrieve a full spreadsheet by its ID.
  * Evaluate *lookup* expressions that reference other cells.
* Deliver clean, well-structured, and fully-tested code that reflects your best engineering practices.

---

## API Specification

All endpoints are rooted at `/sheet`.

### 1. Create a Sheet

```
POST /sheet
Content-Type: application/json
```

#### Request Body

```jsonc
{
  "columns": [
    { "name": "A", "type": "boolean" },
    { "name": "B", "type": "int"     },
    { "name": "C", "type": "double"  },
    { "name": "D", "type": "string"  }
  ]
}
```

* `type` supports: `boolean`, `int`, `double`, `string`.

#### Response

```json
{ "sheetId": "123e4567-e89b-12d3-a456-426614174000" }
```

---

### 2. Set a Cell

```
PUT /sheet/{sheetId}/cell
Content-Type: application/json
```

#### Request Body

```jsonc
{
  "column": "A",
  "row":    10,
  "value":  "hello" // or "lookup(B,5)"
}
```

* Accepts a literal value **or** a `lookup(columnName,rowIndex)` expression.

#### Errors

* **400 Bad Request** – Type mismatch, invalid column/row, or a cycle in lookup references.

---

### 3. Retrieve a Sheet

```
GET /sheet/{sheetId}
```

Returns the full sheet, including resolved lookup values.
The exact JSON structure is up to your design; it simply needs to convey cell positions and their current values.

---

## Lookup Function

```
lookup(columnName, rowIndex)
```

* Returns the value stored at `(columnName, rowIndex)`.
* Evaluated lazily or eagerly—implementation choice is yours.
* Must obey all validation rules below.

#### Example Session

```text
POST  /sheet               → sheetId = 42
PUT   /sheet/42/cell (A,10) = "hello"
PUT   /sheet/42/cell (B,11) = true
PUT   /sheet/42/cell (C, 1) = "lookup(A,10)"

GET   /sheet/42
→
(A,10) = "hello"
(B,11) = true
(C, 1) = "hello"
```

---

## Validation Rules

1. **Type Safety**

   * The evaluated value must match the column’s declared type.

     * Example: assigning `lookup(A,10)` to column **B** (type = boolean) is invalid if **A** is a string column.

2. **No Cycles**

   * Direct or indirect cycles in lookup chains are disallowed.
   * A self-reference (`lookup(C,1)` inside `(C,1)`) is also a cycle.

   ```
   A1 → lookup(C1)
   C1 → lookup(A1)  // ❌ cycle of length 2
   ```

---

## Quality Expectations

1. **Code Style** – Idiomatic, modular, and easy to navigate.
2. **Unit Tests** – Test business logic functions directly (no HTTP layer).
3. **Integration Tests** – Spin up the server and exercise endpoints via an HTTP client.
4. **Packaging** – Organize code into coherent packages/modules.

---

## Assumptions

* Column names are unique within a sheet and case-sensitive.
* Row indices are non-negative integers.
* Lookups reference cells **within the same sheet** only.
* Sheets are stored in memory for simplicity; persistence is optional.

Feel free to add or modify assumptions—just document them here.

---

## Running the Server

```bash
# Using Maven (example)
mvn spring-boot:run
# Or with Gradle
./gradlew bootRun
```

The service starts on `http://localhost:8080/` (configurable via environment variables or application-yaml).

### Configuration

| Variable            | Default | Description              |
| ------------------- | ------- | ------------------------ |
| `SERVER_PORT`       | `8080`  | HTTP listen port         |
| `ANCHOR_DATA_STORE` | `mem`   | `mem` or `postgres` etc. |

---

## Executing the Tests

```bash
# Unit tests
mvn test            # or ./gradlew test

# Integration tests
mvn verify          # or ./gradlew integrationTest
```

Integration tests automatically start an ephemeral server instance on a random port.

---

**Good luck!**
When you’re finished, please push your solution to a public repository, grant us read access, and share the link. We’ll review the implementation and discuss it in a follow-up interview.
