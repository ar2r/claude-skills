# SQL Review Policy

Правила ревью для SQL кода, запросов и работы с базами данных. Применимо к raw SQL, ORM queries, и database migrations.

---

## Query Performance

| ID | Rule | Severity | Description |
|----|------|----------|-------------|
| QP-1 | SELECT * in production | MAJOR | Selecting all columns when only few needed. Wastes bandwidth, breaks when schema changes |
| QP-2 | Missing WHERE clause | CRITICAL | UPDATE/DELETE without WHERE. Can destroy entire table data |
| QP-3 | N+1 queries | CRITICAL | Query in loop. Use JOIN or bulk fetch instead |
| QP-4 | Missing indexes | MAJOR | WHERE/JOIN/ORDER BY on unindexed columns. Slow queries, full table scan |
| QP-5 | Inefficient JOIN order | MAJOR | Large table joined first. Join smaller tables first, filter early |
| QP-6 | Function in WHERE clause | MAJOR | `WHERE YEAR(date) = 2024` prevents index use. Use `WHERE date >= '2024-01-01' AND date < '2025-01-01'` |
| QP-7 | OR in WHERE prevents index | MAJOR | `WHERE col1 = x OR col2 = y` often prevents index use. Consider UNION or separate queries |
| QP-8 | Missing LIMIT on large result sets | MAJOR | Query returning thousands of rows without LIMIT. Memory issues, slow API |
| QP-9 | Wildcard at start of LIKE | MAJOR | `LIKE '%term'` prevents index use. Can't use index with leading wildcard |
| QP-10 | Subquery in SELECT list | MAJOR | `SELECT (SELECT...) FROM` - runs subquery per row. Use JOIN instead |

## Correctness

| ID | Rule | Severity | Description |
|----|------|----------|-------------|
| CR-1 | SQL injection vulnerability | CRITICAL | String concatenation in SQL. Use parameterized queries |
| CR-2 | Implicit type conversion | MAJOR | Comparing int column to string. Prevents index use, unexpected results |
| CR-3 | NULL comparison with = | MAJOR | `WHERE col = NULL` always false. Use `IS NULL` or `IS NOT NULL` |
| CR-4 | Missing transaction | MAJOR | Multiple related writes without transaction. Partial updates on error |
| CR-5 | Lost updates | CRITICAL | Read-modify-write without lock or version check. Race condition |
| CR-6 | COUNT(*) vs COUNT(column) | MINOR | Different semantics with NULLs. COUNT(*) counts all rows, COUNT(col) skips NULLs |
| CR-7 | Division by zero | MAJOR | No check before division. Use NULLIF or CASE |
| CR-8 | Wrong JOIN type | MAJOR | INNER JOIN losing data where LEFT JOIN needed, or vice versa |
| CR-9 | Aggregate without GROUP BY | MAJOR | Mixing aggregates and columns without GROUP BY. Wrong results or error |

## Schema Design

| ID | Rule | Severity | Description |
|----|------|----------|-------------|
| SD-1 | Missing primary key | CRITICAL | Table without PK. Can't identify unique rows, replication issues |
| SD-2 | Missing foreign key constraints | MAJOR | Referential integrity not enforced. Orphaned records, data inconsistency |
| SD-3 | Wrong data type | MAJOR | Storing dates as VARCHAR, booleans as INT. Query issues, storage waste |
| SD-4 | No default value for NOT NULL | MINOR | NOT NULL column without DEFAULT. All INSERTs must specify value |
| SD-5 | ENUM misuse | MINOR | Using ENUM for values that change often. Schema change needed to add values |
| SD-6 | Missing unique constraint | MAJOR | Columns that should be unique without constraint. Duplicate data possible |
| SD-7 | Over-normalization | MINOR | Too many JOINs needed for simple queries. Denormalize common read patterns |
| SD-8 | Under-normalization | MAJOR | Duplicated data, update anomalies. Normalize to remove redundancy |
| SD-9 | Missing indexes on FK | MAJOR | Foreign key columns without index. Slow JOINs and deletes |

## Transactions & Concurrency

| ID | Rule | Severity | Description |
|----|------|----------|-------------|
| TC-1 | Long-running transaction | CRITICAL | Transaction holding locks for long time. Blocks other queries, deadlocks |
| TC-2 | Wrong isolation level | MAJOR | READ UNCOMMITTED allowing dirty reads. Use READ COMMITTED or higher |
| TC-3 | No deadlock handling | MAJOR | No retry logic for deadlocks. Operations fail permanently |
| TC-4 | SELECT FOR UPDATE missing | MAJOR | Reading data for update without lock. Lost updates possible |
| TC-5 | Nested transactions misuse | MAJOR | Assuming nested transactions work like they don't. Use savepoints |
| TC-6 | Transaction per row | CRITICAL | Committing after each row in loop. Massive overhead, slow bulk operations |

## ORM Anti-patterns

| ID | Rule | Severity | Description |
|----|------|----------|-------------|
| ORM-1 | Lazy loading in loop | CRITICAL | N+1 queries from lazy-loaded relationships. Use eager loading |
| ORM-2 | Loading full object for update | MAJOR | Loading entire record to update one field. Use UPDATE query |
| ORM-3 | ORM for bulk operations | MAJOR | ORM methods for inserting thousands of rows. Use bulk INSERT |
| ORM-4 | No query inspection | MAJOR | Not checking generated SQL. ORM may generate inefficient queries |
| ORM-5 | Missing select_related/prefetch | MAJOR | Django/SQLAlchemy queries without optimization. N+1 queries |

## Data Types

| ID | Rule | Severity | Description |
|----|------|----------|-------------|
| DT-1 | VARCHAR without length | MINOR | VARCHAR without explicit length. May use max length (memory waste) |
| DT-2 | FLOAT for money | CRITICAL | Using FLOAT/DOUBLE for currency. Rounding errors, wrong calculations |
| DT-3 | Wrong string type | MINOR | TEXT for short strings, CHAR for variable length. Storage inefficiency |
| DT-4 | Timestamp without timezone | MAJOR | TIMESTAMP instead of TIMESTAMPTZ. Timezone confusion, bugs |
| DT-5 | BIGINT everywhere | NIT | Using BIGINT when INT sufficient. Wastes 4 bytes per row |
| DT-6 | JSON in SQL database | MINOR | Storing structured data as JSON. Hard to query, no validation |

## Migrations

| ID | Rule | Severity | Description |
|----|------|----------|-------------|
| MG-1 | No rollback migration | MAJOR | Migration without down/rollback script. Can't undo deployment |
| MG-2 | Blocking schema change | CRITICAL | ALTER TABLE on large table without chunking. Locks table for minutes |
| MG-3 | Data loss in migration | CRITICAL | Dropping column/table without backup. Irreversible data loss |
| MG-4 | Missing index creation | MAJOR | Adding index on large table without CONCURRENTLY (PostgreSQL). Locks table |
| MG-5 | Renaming instead of aliasing | MAJOR | Renaming column immediately. Old code breaks. Add new, migrate, remove old |
| MG-6 | Default value for existing rows | MAJOR | Adding NOT NULL column without default on large table. Long lock |

## PostgreSQL Specific

| ID | Rule | Severity | Description |
|----|------|----------|-------------|
| PG-1 | Missing VACUUM | MAJOR | No VACUUM on high-churn tables. Bloat, slow queries |
| PG-2 | Inefficient LIKE | MINOR | LIKE without trigram index (pg_trgm). Use GIN index for pattern matching |
| PG-3 | JSON vs JSONB | MINOR | Using JSON instead of JSONB. JSONB faster, supports indexing |
| PG-4 | Missing partial index | MINOR | Index on boolean or sparse column without WHERE. Large unnecessary index |
| PG-5 | EXPLAIN not used | MAJOR | Complex query without EXPLAIN ANALYZE. Unknown performance characteristics |

## Security

| ID | Rule | Severity | Description |
|----|------|----------|-------------|
| SEC-1 | SQL injection | CRITICAL | String concatenation or format in query. Use parameterized queries |
| SEC-2 | Over-privileged database user | MAJOR | Application using admin account. Principle of least privilege |
| SEC-3 | Passwords in plain text | CRITICAL | Storing passwords without hashing. Security breach waiting to happen |
| SEC-4 | Missing row-level security | MAJOR | Multi-tenant database without RLS. Users can access other tenants' data |
| SEC-5 | Sensitive data in logs | CRITICAL | Logging queries with passwords, tokens, PII. Compliance violation |

## Examples

### QP-1: SELECT * in production

**Bad:**
```sql
-- Returns all 50 columns even though only need 3
SELECT * FROM users WHERE id = 123;
```

**Good:**
```sql
SELECT id, name, email FROM users WHERE id = 123;
```

### QP-3: N+1 queries

**Bad:**
```python
# N+1 queries
users = User.query.all()
for user in users:
    # Query per user!
    orders = Order.query.filter_by(user_id=user.id).all()
    process(orders)
```

**Good:**
```python
# Single query with JOIN
users = User.query.options(
    joinedload(User.orders)
).all()
for user in users:
    process(user.orders)
```

### QP-6: Function in WHERE clause

**Bad:**
```sql
-- Can't use index on created_at
SELECT * FROM orders
WHERE YEAR(created_at) = 2024
  AND MONTH(created_at) = 2;
```

**Good:**
```sql
-- Can use index
SELECT * FROM orders
WHERE created_at >= '2024-02-01'
  AND created_at < '2024-03-01';
```

### CR-1: SQL injection

**Bad:**
```python
# CRITICAL - SQL injection!
query = f"SELECT * FROM users WHERE id = {user_id}"
cursor.execute(query)

# Also bad
query = "SELECT * FROM users WHERE name = '" + username + "'"
```

**Good:**
```python
# Parameterized query
query = "SELECT * FROM users WHERE id = %s"
cursor.execute(query, (user_id,))

# Or with ORM
user = User.query.filter_by(id=user_id).first()
```

### CR-3: NULL comparison

**Bad:**
```sql
-- Always returns nothing
SELECT * FROM users WHERE deleted_at = NULL;
```

**Good:**
```sql
SELECT * FROM users WHERE deleted_at IS NULL;
```

### CR-5: Lost updates (race condition)

**Bad:**
```python
# Race condition!
user = User.query.get(user_id)
user.balance -= 100
user.save()
```

**Good:**
```python
# Atomic update
db.execute(
    "UPDATE users SET balance = balance - %s WHERE id = %s",
    (100, user_id)
)

# Or with pessimistic lock
user = User.query.filter_by(id=user_id).with_for_update().first()
user.balance -= 100
user.save()
```

### SD-2: Missing foreign key

**Bad:**
```sql
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    user_id INTEGER,  -- No FK constraint!
    total DECIMAL(10, 2)
);
```

**Good:**
```sql
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    total DECIMAL(10, 2) NOT NULL
);

-- Index on FK for performance
CREATE INDEX idx_orders_user_id ON orders(user_id);
```

### TC-1: Long-running transaction

**Bad:**
```python
with transaction():
    # Long transaction holding locks
    users = fetch_all_users()
    for user in users:
        # External API call!
        status = check_user_status_api(user)
        user.update(status=status)
```

**Good:**
```python
# Fetch data outside transaction
users = fetch_all_users()
user_statuses = {}
for user in users:
    user_statuses[user.id] = check_user_status_api(user)

# Quick transaction
with transaction():
    for user_id, status in user_statuses.items():
        User.query.filter_by(id=user_id).update(status=status)
```

### DT-2: FLOAT for money

**Bad:**
```sql
CREATE TABLE products (
    price FLOAT  -- Rounding errors!
);

-- 0.1 + 0.2 = 0.30000000000000004
```

**Good:**
```sql
CREATE TABLE products (
    price DECIMAL(10, 2)  -- Exact precision
);

-- Or store cents as integer
CREATE TABLE products (
    price_cents INTEGER  -- 1050 = $10.50
);
```

### MG-2: Blocking schema change

**Bad:**
```sql
-- Locks entire table for hours on large table!
ALTER TABLE orders ADD COLUMN priority INTEGER;
```

**Good:**
```sql
-- PostgreSQL: Add column with default efficiently
ALTER TABLE orders
ADD COLUMN priority INTEGER DEFAULT 0 NOT NULL;

-- Or for other databases: add nullable first, backfill, then add constraint
ALTER TABLE orders ADD COLUMN priority INTEGER;
-- Backfill in batches
UPDATE orders SET priority = 0 WHERE priority IS NULL AND id < 1000000;
-- ... more batches ...
ALTER TABLE orders ALTER COLUMN priority SET NOT NULL;
```

### ORM-1: Lazy loading in loop

**Bad (SQLAlchemy):**
```python
# N+1 queries!
users = session.query(User).all()
for user in users:
    print(user.profile.bio)  # Lazy load per user
```

**Good:**
```python
# Eager loading
users = session.query(User).options(
    joinedload(User.profile)
).all()
for user in users:
    print(user.profile.bio)  # Already loaded
```

**Bad (Django):**
```python
# N+1 queries!
for order in Order.objects.all():
    print(order.user.name)  # Query per order
```

**Good:**
```python
# select_related for ForeignKey
for order in Order.objects.select_related('user'):
    print(order.user.name)
```

### PG-5: Missing EXPLAIN

**Bad:**
```python
# Complex query, no idea if it's efficient
results = db.execute("""
    SELECT u.name, COUNT(o.id)
    FROM users u
    LEFT JOIN orders o ON o.user_id = u.id
    WHERE u.created_at > '2024-01-01'
    GROUP BY u.id
    HAVING COUNT(o.id) > 10
""")
```

**Good:**
```python
# Check query plan first
explain = db.execute("""
    EXPLAIN ANALYZE
    SELECT u.name, COUNT(o.id)
    FROM users u
    LEFT JOIN orders o ON o.user_id = u.id
    WHERE u.created_at > '2024-01-01'
    GROUP BY u.id
    HAVING COUNT(o.id) > 10
""")
print(explain)
# Look for: Seq Scan (bad), Index Scan (good), join costs, row estimates
```

## Query Optimization Checklist

When reviewing slow queries:

1. **Run EXPLAIN ANALYZE** - understand the query plan
2. **Check for**:
   - Sequential scans on large tables → add index
   - High row estimates vs actual → update statistics
   - Nested loops on large result sets → improve indexes or rewrite
   - Sort/hash operations → consider indexes
3. **Verify indexes exist** on:
   - WHERE clause columns
   - JOIN columns (especially foreign keys)
   - ORDER BY columns
   - GROUP BY columns
4. **Consider query rewrite**:
   - Subquery → JOIN
   - OR → UNION
   - Function call → pre-computed column
5. **Check data distribution** - indexes useless if low cardinality
6. **Monitor in production** - synthetic tests don't show real data patterns

## Detection Tips

- Search for: raw SQL strings with string concatenation or f-strings
- Look for: loops containing database queries
- Find: SELECT * in application code
- Check: migrations without rollback
- Verify: foreign keys have indexes
- Confirm: money fields use DECIMAL not FLOAT
- Review: UPDATE/DELETE for WHERE clause
