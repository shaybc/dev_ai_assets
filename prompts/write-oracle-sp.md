---
name: Write Oracle SP Class
dcc_uri: dev/prompts/write-oracle-sp
version: '1.0.0'
schema: 'v1'
description: Generates a complete, production-ready Java class and method for executing an Oracle stored procedure with full error handling, security, and resource management.
dcc_definition_type: prompt
dcc_tags:
  - database
  - backend
  - Oracle
  - stored_procedure
invokable: true
---

Generate a production-ready Java class that executes an Oracle stored procedure.

## Context & Requirements

Apply ALL of the following rules without exception:

### Connection & Resource Management
- Use `javax.sql.DataSource` injected via constructor (not `DriverManager`)
- Use **try-with-resources** for every `Connection`, `CallableStatement`, and `ResultSet`
- Close REF CURSOR `ResultSet` objects as a **separate** try-with-resources block
- Never store `Connection` as a field

### SQL Construction — SECURITY CRITICAL
- Use JDBC escape syntax ONLY: `{call SCHEMA.PACKAGE.PROC(?,?,?)}` 
- **Zero** string concatenation of user-supplied values into the call string
- Bind every parameter via typed `cs.setXxx()` / `cs.registerOutParameter()`
- For null values use `cs.setNull(index, Types.XXX)` — never pass raw Java null

### Type Mapping (Oracle → Java)
- NUMBER (integer) → `int` / `long` via `setInt`/`setLong`
- NUMBER (decimal/currency) → `BigDecimal` via `setBigDecimal` — **NEVER `double`**
- VARCHAR2 → `String` with length validation before binding
- DATE → `java.sql.Date`
- TIMESTAMP → `java.sql.Timestamp`
- REF CURSOR → `OracleTypes.CURSOR` → cast OUT param to `ResultSet`
- BLOB/CLOB → stream, never load entire content into memory

### Transaction Management
- Set `conn.setAutoCommit(false)` for procedures containing DML
- Call `conn.commit()` on success; `conn.rollback()` in catch
- If Spring is available, prefer `@Transactional`

### Error Handling
- Catch `SQLException`, wrap in a domain `StoredProcedureException` preserving the cause
- Log: procedure name + `e.getErrorCode()` + `e.getSQLState()` + correlation ID
- **NEVER** log bind parameter values (PII risk)
- Map known ORA codes: ORA-01403 → `NoSuchRecordException`, ORA-00001 → `DuplicateRecordException`, ORA-04068 → single retry
- Use `setQueryTimeout(30)` on the statement

### Security
- No hardcoded credentials — DataSource must be externally configured
- Add input validation (null check + length check) before any binding
- Privilege note: DataSource user should only have EXECUTE on this procedure

### Code Structure
- Class name: `{ProcedureName}Executor` (e.g., `CreateOrderExecutor`)
- Method name: `execute{ProcedureName}(...)` returning a domain DTO or void
- Method throws: `StoredProcedureException` (not raw `SQLException`)
- Constants for procedure call string and each parameter index
- Separate mapper method: `private MyDto mapRow(ResultSet rs)`
- Include SLF4J logging

---

## Input Information Needed

Please provide:
1. **Procedure full name** (SCHEMA.PACKAGE.PROCEDURE or SCHEMA.PROCEDURE)
2. **IN parameters**: name, Oracle type, Java type, nullable?
3. **OUT parameters**: name, Oracle type, Java type
4. **IN/OUT parameters** (if any)
5. **Does the procedure return a REF CURSOR?** If yes, which OUT param index?
6. **Does it perform DML?** (commit/rollback needed?)
7. **Target Java version** (8, 11, 17, 21)
8. **Framework** (plain JDBC, Spring JDBC, Spring Boot)
9. **DTO class name** to map results into (if applicable)

---

## Expected Output Structure

```
{ProcedureName}Executor.java
├── Constants block (PROC_CALL, PARAM_* indices)
├── Constructor (DataSource injection)
├── Public execute method
│   ├── Input validation
│   ├── try-with-resources (Connection → CallableStatement)
│   │   ├── registerOutParameter calls
│   │   ├── setXxx calls for IN params
│   │   ├── cs.setQueryTimeout(30)
│   │   ├── cs.execute()
│   │   ├── REF CURSOR try-with-resources (if applicable)
│   │   ├── conn.commit() (if DML)
│   │   └── return DTO / collect results
│   └── catch blocks with rollback + wrap + rethrow
└── Private mapRow(ResultSet) method
```

Also generate:
- `StoredProcedureException.java` — runtime exception wrapping SQLException
- `{ProcedureName}Result.java` — immutable DTO (record or final class) for OUT params / cursor rows
- Skeleton JUnit 5 test class with Mockito, covering: happy path, no-data-found, duplicate key, timeout, connection failure

---

## Example Instantiation

**Procedure:** `ORDER_SCHEMA.ORDER_PKG.CREATE_ORDER`  
**IN:** `p_customer_id NUMBER`, `p_amount NUMBER(15,2)`, `p_description VARCHAR2(500)`  
**OUT:** `p_order_id NUMBER`, `p_status_code VARCHAR2(10)`  
**DML:** YES  

Expected class skeleton:

```java
public class CreateOrderExecutor {

    private static final String PROC_CALL = 
        "{call ORDER_SCHEMA.ORDER_PKG.CREATE_ORDER(?,?,?,?,?)}";
    private static final int PARAM_CUSTOMER_ID  = 1;
    private static final int PARAM_AMOUNT       = 2;
    private static final int PARAM_DESCRIPTION  = 3;
    private static final int PARAM_ORDER_ID_OUT = 4;
    private static final int PARAM_STATUS_OUT   = 5;
    private static final int QUERY_TIMEOUT_SEC  = 30;

    private final DataSource dataSource;
    private static final Logger log = LoggerFactory.getLogger(CreateOrderExecutor.class);

    public CreateOrderExecutor(DataSource dataSource) {
        this.dataSource = Objects.requireNonNull(dataSource, "dataSource must not be null");
    }

    public CreateOrderResult executeCreateOrder(long customerId, 
                                                BigDecimal amount, 
                                                String description) throws StoredProcedureException {
        // 1. Input validation
        if (amount == null || amount.compareTo(BigDecimal.ZERO) <= 0)
            throw new IllegalArgumentException("amount must be positive");
        if (description != null && description.length() > 500)
            throw new IllegalArgumentException("description exceeds 500 characters");

        Connection conn = null;
        try {
            conn = dataSource.getConnection();
            conn.setAutoCommit(false);

            try (CallableStatement cs = conn.prepareCall(PROC_CALL)) {
                cs.setQueryTimeout(QUERY_TIMEOUT_SEC);
                cs.setLong(PARAM_CUSTOMER_ID, customerId);
                cs.setBigDecimal(PARAM_AMOUNT, amount);
                if (description != null) {
                    cs.setString(PARAM_DESCRIPTION, description);
                } else {
                    cs.setNull(PARAM_DESCRIPTION, Types.VARCHAR);
                }
                cs.registerOutParameter(PARAM_ORDER_ID_OUT, Types.NUMERIC);
                cs.registerOutParameter(PARAM_STATUS_OUT,   Types.VARCHAR);

                cs.execute();

                long orderId    = cs.getLong(PARAM_ORDER_ID_OUT);
                String status   = cs.getString(PARAM_STATUS_OUT);

                conn.commit();
                log.info("CREATE_ORDER executed: orderId={}", orderId);
                return new CreateOrderResult(orderId, status);
            }
        } catch (SQLException e) {
            safeRollback(conn, e);
            handleOraError(e); // maps known ORA codes
            throw new StoredProcedureException("CREATE_ORDER failed [ORA-" + e.getErrorCode() + "]", e);
        } finally {
            closeQuietly(conn);
        }
    }

    private void handleOraError(SQLException e) throws StoredProcedureException {
        switch (e.getErrorCode()) {
            case 1403: throw new StoredProcedureException("No data found", e);
            case    1: throw new StoredProcedureException("Duplicate record", e);
        }
    }

    private void safeRollback(Connection conn, SQLException cause) {
        if (conn != null) {
            try { conn.rollback(); }
            catch (SQLException ex) { log.error("Rollback failed after ORA-{}", cause.getErrorCode(), ex); }
        }
    }

    private void closeQuietly(Connection conn) {
        if (conn != null) {
            try { conn.close(); }
            catch (SQLException ex) { log.warn("Connection close failed", ex); }
        }
    }
}
```
