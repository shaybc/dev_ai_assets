---
name: Create Oracle SP
dcc_uri: dev/rules/create-oracle-sp
description: Oracle Stored Procedure Java Development Rules
version: '1.0.0'
dcc_definition_type: rule
dcc_tags:
  - dev
  - backend
  - oracle
  - database
  - stored_procedure
---

# Oracle Stored Procedure Java Development Rules

## Overview
These rules govern the development of Java classes and methods that execute Oracle stored procedures via JDBC. They enforce correctness, security, and maintainability standards.

---

## 1. Connection Management

- **ALWAYS** obtain connections from a `DataSource` or connection pool (e.g., HikariCP, UCP). Never create raw `DriverManager` connections in production code.
- **ALWAYS** use try-with-resources for `Connection`, `CallableStatement`, and `ResultSet` to guarantee closure even on exceptions.
- **NEVER** store a `Connection` as an instance variable in a shared (singleton/service) bean — connections are not thread-safe.
- **ALWAYS** return connections to the pool promptly; avoid long-lived transactions unless explicitly required.
- Set `connection.setAutoCommit(false)` explicitly when the procedure performs DML so you control commit/rollback.

```java
// CORRECT pattern
try (Connection conn = dataSource.getConnection();
     CallableStatement cs = conn.prepareCall("{call MY_SCHEMA.MY_PROC(?,?,?)}")) {
    // ...
} catch (SQLException e) {
    // handle
}
```

---

## 2. CallableStatement Construction

- **ALWAYS** use the JDBC escape syntax `{call SCHEMA.PACKAGE.PROCEDURE(?,?)}` — never build the call string by concatenating user input.
- Include the schema and, where applicable, the package name explicitly to avoid ambiguous resolution.
- **NEVER** use `Statement.execute(String sql)` to call stored procedures — always use `CallableStatement`.
- For functions (which return a value), use `{? = call SCHEMA.PACKAGE.FUNCTION(?)}` and register the return parameter first.

---

## 3. Parameter Binding — Security & Correctness

- **ALWAYS** bind all input values with `cs.setXxx(index, value)` typed setters — NEVER concatenate values into the call string (SQL injection).
- **ALWAYS** register OUT and IN/OUT parameters with `cs.registerOutParameter(index, Types.XXX)` before executing.
- Match Java types to Oracle types precisely:

  | Oracle Type | Java Setter/Getter |
  |---|---|
  | VARCHAR2 / CHAR | `setString` / `getString` |
  | NUMBER (integer) | `setInt` / `getInt` or `setLong` / `getLong` |
  | NUMBER (decimal) | `setBigDecimal` / `getBigDecimal` — never use `double` for financial data |
  | DATE | `setDate` / `getDate` (use `java.sql.Date`) |
  | TIMESTAMP | `setTimestamp` / `getTimestamp` |
  | CLOB | `setClob` / `getClob` — stream large text, do not load into String |
  | BLOB | `setBlob` / `getBlob` — stream large binary, do not load into byte[] |
  | REF CURSOR | `registerOutParameter(index, OracleTypes.CURSOR)` then `(ResultSet) cs.getObject(index)` |
  | ARRAY (TABLE type) | Use `oracle.jdbc.OracleConnection.createARRAY` |

- **NEVER** pass `null` as a raw Java null for typed parameters — use `cs.setNull(index, Types.XXX)` instead to avoid `NullPointerException` or JDBC driver misbehavior.

---

## 4. REF CURSOR Handling

- Cast the OUT parameter to `ResultSet` using `(ResultSet) cs.getObject(index)`.
- **ALWAYS** close the REF CURSOR `ResultSet` explicitly in a finally block or try-with-resources — it is a separate resource from the `CallableStatement`.
- **NEVER** return a live `ResultSet` outside the try-with-resources scope — map it to a DTO/List inside the block.

```java
cs.registerOutParameter(3, OracleTypes.CURSOR);
cs.execute();
try (ResultSet rs = (ResultSet) cs.getObject(3)) {
    while (rs.next()) {
        // map to DTO
    }
}
```

---

## 5. Transaction Management

- After calling a procedure that performs DML, explicitly call `conn.commit()` on success and `conn.rollback()` in the catch block.
- If using Spring, prefer `@Transactional` with a `PlatformTransactionManager` rather than manual commit/rollback.
- **NEVER** swallow `SQLException` without at minimum logging it and rethrowing as a domain exception.
- Handle `ORA-` error codes where specific recovery is possible (e.g., `ORA-00001` unique constraint violation → map to a domain-level `DuplicateKeyException`).

---

## 6. Error Handling

- Wrap `SQLException` in a meaningful application exception (e.g., `StoredProcedureException`) that preserves the original cause.
- Log the Oracle error code (`e.getErrorCode()`), SQL state (`e.getSQLState()`), and the procedure name — **NEVER** log bind parameter values that may contain PII or secrets.
- For batch/loop scenarios, use `e.getNextException()` to iterate the full exception chain.
- Map well-known ORA codes to application exceptions:
  - `ORA-01403` (no data found) → `NoSuchRecordException`
  - `ORA-00001` (unique constraint) → `DuplicateRecordException`
  - `ORA-04068` (package state discarded) → retry once, then fail
  - `ORA-01000` (max cursors exceeded) → resource leak; fix close logic

---

## 7. Security Rules

- **NO SQL injection**: All parameters must go through `setXxx` bind methods — zero exceptions.
- **Principle of least privilege**: The DB user used for the connection must have `EXECUTE` privilege only on the required procedures — no `SELECT/INSERT/UPDATE/DELETE` on underlying tables directly.
- **No credentials in code**: Connection strings, usernames, and passwords must come from environment variables, Vault, or an encrypted config source — never hardcoded or committed to source control.
- **No PII in logs**: Never log parameter values; log only procedure names, error codes, and correlation IDs.
- **Input length validation**: Validate string parameter lengths in Java before binding to prevent `ORA-06502` (value too large) and as a defense-in-depth against misuse.
- **Audit trail**: For sensitive procedures (account changes, financial transactions), log the calling user identity and timestamp at the application level before invocation.
- **Connection encryption**: Ensure the JDBC URL specifies Oracle's native network encryption or TLS (`TCPS` protocol) in non-development environments.

---

## 8. Performance & Resource Rules

- Reuse `DataSource` — never create a new pool per request.
- For procedures called in tight loops, use a single `CallableStatement` and rebind parameters each iteration — do not prepare a new statement per iteration.
- Fetch `ResultSet` rows in batches using `rs.setFetchSize(n)` to reduce round-trips.
- Set statement timeout with `cs.setQueryTimeout(seconds)` to avoid runaway queries holding connections.
- Monitor and alert on connection pool exhaustion metrics.

---

## 9. Code Structure Rules

- Each stored procedure invocation belongs in its own method, named `execute<ProcedureName>`.
- The method must declare `throws StoredProcedureException` (your domain exception) — do not leak `SQLException` to callers.
- Separate JDBC plumbing (DAO layer) from business logic — no procedure calls in service/controller layers directly.
- Use constants for procedure names and parameter indices:

```java
private static final String PROC_CREATE_ORDER = "{call ORDER_PKG.CREATE_ORDER(?,?,?,?)}";
private static final int PARAM_CUSTOMER_ID = 1;
private static final int PARAM_AMOUNT      = 2;
private static final int PARAM_STATUS_OUT  = 3;
private static final int PARAM_ORDER_ID_OUT = 4;
```

---

## 10. Testing Rules

- Unit-test DAO methods using an embedded database or Testcontainers with a real Oracle image.
- Mock `DataSource`/`Connection`/`CallableStatement` with Mockito only for testing error-handling paths (not for validating SQL correctness).
- Integration tests must cover: happy path, `ORA-01403` no-data, `ORA-00001` duplicate key, connection failure, and timeout.
- Assert that resources are closed (use `verify(mockCs).close()` in Mockito tests).

---

## 11. Common Pitfalls Checklist

| Pitfall | Prevention |
|---|---|
| Resource leak (unclosed cursor/connection) | Always use try-with-resources |
| Wrong parameter index (off-by-one) | Use named constants for indices |
| Double for currency/financial values | Use `BigDecimal` always |
| Null NPE on OUT parameter before execute | Register OUT params before `execute()` |
| Swallowed exception hiding ORA errors | Always log and rethrow |
| Hardcoded schema name | Use configurable schema prefix |
| Thread-unsafe shared Connection | Never share connections across threads |
| Missing rollback on exception | Always rollback in catch when autoCommit=false |
| REF CURSOR not closed | Close RS separately inside try-with-resources |
| Package state discarded (ORA-04068) | Implement single-retry logic |
