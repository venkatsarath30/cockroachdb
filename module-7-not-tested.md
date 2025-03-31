# Module 7: CockroachDB Performance Tuning & Monitoring

## Overview
Performance tuning and monitoring in CockroachDB are essential for optimizing query execution, improving indexing strategies, and ensuring the system runs efficiently under load. This module covers:

- **Query Optimization & EXPLAIN Plans**
- **Indexing Best Practices**
- **Monitoring Tools & Logging**
- **Performance Tuning Techniques**

---

## 1. Query Optimization & EXPLAIN Plans
### Understanding Query Execution
CockroachDB provides the `EXPLAIN` and `EXPLAIN ANALYZE` commands to analyze query plans and optimize performance.

### Using `EXPLAIN`
The `EXPLAIN` command shows how a query is planned and executed.

```sql
EXPLAIN SELECT * FROM users WHERE email = 'test@example.com';
```

Example output:
```
  tree    |    field    |      description
----------+------------+-------------------
 scan     | table      | users
          | filter     | email = 'test@example.com'
```

### Using `EXPLAIN ANALYZE`
This command executes the query and provides execution statistics.

```sql
EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'test@example.com';
```

It helps identify slow operations and optimize indexes or query structure.

**Optimization Tips:**
- Avoid full table scans by adding proper indexes.
- Use indexed columns in WHERE clauses.
- Use `LIMIT` to reduce result set size.
- Rewrite queries to use efficient joins.

---

## 2. Indexing Best Practices
Indexes improve query performance by allowing CockroachDB to quickly locate data.

### Types of Indexes
1. **Primary Index:** Automatically created for the primary key.
2. **Secondary Index:** Created to optimize query performance on non-primary key columns.
3. **Unique Index:** Ensures uniqueness of a column.
4. **Composite Index:** Indexing multiple columns together.

### Creating an Index
```sql
CREATE INDEX idx_email ON users(email);
```

### Using Composite Indexes
```sql
CREATE INDEX idx_name_email ON users(name, email);
```

### Dropping an Index
```sql
DROP INDEX idx_email;
```

**Indexing Tips:**
- Avoid excessive indexing to reduce write overhead.
- Use composite indexes for multi-column queries.
- Regularly monitor index usage using `SHOW CREATE TABLE`.

---

## 3. Monitoring Tools & Logging
### Built-in Monitoring Tools
CockroachDB provides several tools for monitoring cluster health and performance.

1. **CockroachDB Web UI** (Available at `http://localhost:8080`)
2. **Metrics API** (`/debug/metrics`)
3. **Query Performance Logs**
4. **Cluster Events (`cockroach sql -e 'SHOW JOBS'`)**

### Checking Node Status
```sh
cockroach node status --insecure
```

### Viewing SQL Query Metrics
```sh
cockroach sql --insecure -e "SELECT * FROM crdb_internal.node_queries ORDER BY execution_time DESC LIMIT 10;"
```

### Enabling Statement Logging
```sh
cockroach start --insecure --log-dir=logs --vmodule=exec_log=2
```

**Monitoring Best Practices:**
- Regularly check slow queries in the Web UI.
- Enable logs for performance tuning.
- Use monitoring dashboards like Prometheus and Grafana.

---

## 4. Performance Tuning Techniques
### Connection Pooling
Use a connection pooler like **pgbouncer** to manage multiple client connections efficiently.

### Memory & Cache Tuning
Adjust memory limits in the `cockroach start` command:
```sh
cockroach start --cache=25% --max-sql-memory=25%
```

### Load Balancing
Distribute queries using a load balancer like **HAProxy** or **CockroachDB's SQL Proxy**.

### Query Optimization with Prepared Statements
```sql
PREPARE stmt AS SELECT * FROM users WHERE email = $1;
EXECUTE stmt ('test@example.com');
```

**Performance Tuning Best Practices:**
- Use `EXPLAIN ANALYZE` to detect slow queries.
- Optimize indexes based on query patterns.
- Distribute load across multiple nodes.
- Monitor cluster resource usage.

---

## Hands-on Lab: Performance Tuning & Monitoring
### Step 1: Setup CockroachDB Cluster
```sh
cockroach start-single-node --insecure --listen-addr=localhost:26257
```

### Step 2: Create Sample Data
```sql
CREATE DATABASE perf_test;
USE perf_test;
CREATE TABLE users (id SERIAL PRIMARY KEY, name STRING, email STRING UNIQUE, age INT);
INSERT INTO users (name, email, age) VALUES ('Alice', 'alice@example.com', 30), ('Bob', 'bob@example.com', 25);
```

### Step 3: Analyze Query Performance
```sql
EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'alice@example.com';
```

### Step 4: Optimize with Indexing
```sql
CREATE INDEX idx_users_email ON users(email);
```

### Step 5: Monitor System Performance
```sh
cockroach node status --insecure
```

### Step 6: Enable Query Logging
```sh
cockroach sql --insecure -e "SHOW JOBS;"
```

### Step 7: Tune Memory and Cache
```sh
cockroach start --cache=50% --max-sql-memory=50%
```

---

## Summary
- **Query Optimization**: Use `EXPLAIN ANALYZE` to analyze and optimize queries.
- **Indexing Best Practices**: Create efficient indexes to improve query performance.
- **Monitoring Tools**: Use CockroachDB Web UI, logs, and `SHOW JOBS` to monitor activity.
- **Performance Tuning**: Optimize memory, caching, and load balancing for scalability.

By implementing these best practices, you can ensure optimal performance and scalability for your CockroachDB cluster.

