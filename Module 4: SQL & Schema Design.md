# **Module 4: SQL & Schema Design (With Hands-on Lab & Compatibility Chart)**  

This module covers:  
✅ **CockroachDB SQL vs. PostgreSQL vs. MySQL Compatibility**  
✅ **Creating & Managing Databases & Tables**  
✅ **Indexing Strategies & Performance Considerations**  
✅ **Primary Keys, Foreign Keys, and Unique Constraints**  
✅ **Partitioning & Sharding**  

---

## **1. CockroachDB SQL vs. PostgreSQL vs. MySQL Compatibility**  

CockroachDB is wire-compatible with PostgreSQL but has differences compared to both **PostgreSQL** and **MySQL**. Below is a comparison:

### **1.1 Compatibility Chart**  

| Feature | CockroachDB | PostgreSQL | MySQL |
|---------|------------|------------|-------|
| **ACID Transactions** | ✅ Yes (Distributed) | ✅ Yes | ⚠️ Limited (with InnoDB) |
| **Joins (INNER, OUTER, etc.)** | ✅ Yes | ✅ Yes | ✅ Yes |
| **Stored Procedures** | ❌ No | ✅ Yes | ✅ Yes |
| **Triggers** | ❌ No | ✅ Yes | ✅ Yes |
| **Materialized Views** | ❌ No | ✅ Yes | ❌ No |
| **Full-Text Search** | ⚠️ Limited | ✅ Yes | ✅ Yes |
| **Partitioning** | ✅ Yes (Table Partitioning) | ✅ Yes | ✅ Yes |
| **Sharding** | ✅ Automatic | ❌ No (manual sharding) | ❌ No (manual sharding) |
| **Foreign Keys** | ✅ Yes | ✅ Yes | ⚠️ Limited (only with InnoDB) |
| **JSON Support** | ✅ Yes | ✅ Yes (jsonb) | ✅ Yes (JSON) |
| **Indexes** | ✅ Yes (B-Tree, Unique, Partial, etc.) | ✅ Yes (B-Tree, Hash, GIN) | ✅ Yes (B-Tree, Full-Text) |
| **Multi-Region Support** | ✅ Yes | ❌ No (manual setup) | ❌ No |
| **Horizontal Scaling** | ✅ Yes (Native) | ❌ No | ❌ No |

### **1.2 Hands-on: Test PostgreSQL/MySQL Compatibility in CockroachDB**  

📌 **Step 1: Start a CockroachDB SQL Shell**  
```sh
cockroach sql --insecure
```

📌 **Step 2: Create a PostgreSQL/MySQL-Compatible Table**  
```sql
CREATE DATABASE testdb;
USE testdb;

CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    name STRING NOT NULL,
    email STRING UNIQUE NOT NULL
);
```

📌 **Step 3: Insert Data & Query**  
```sql
INSERT INTO employees (name, email) VALUES ('Alice', 'alice@example.com');
SELECT * FROM employees;
```

---

## **2. Creating & Managing Databases & Tables**  

### **2.1 Creating a Database**  
```sql
CREATE DATABASE company;
SHOW DATABASES;
```

### **2.2 Creating Tables**  
```sql
USE company;

CREATE TABLE departments (
    id SERIAL PRIMARY KEY,
    name STRING NOT NULL UNIQUE
);

CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    name STRING NOT NULL,
    email STRING UNIQUE NOT NULL,
    department_id INT REFERENCES departments(id) ON DELETE CASCADE
);
```

### **2.3 Modifying Tables**  
```sql
ALTER TABLE employees ADD COLUMN salary DECIMAL(10,2);
ALTER TABLE employees DROP COLUMN salary;
```

### **2.4 Dropping Tables & Databases**  
```sql
DROP TABLE employees;
DROP DATABASE company CASCADE;
```

---

## **3. Indexing Strategies & Performance Considerations**  

Indexes improve query performance by speeding up searches.

### **3.1 Primary Index (Implicit)**  
Every table in CockroachDB has a **default primary index**.  
```sql
CREATE TABLE orders (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    customer_id INT NOT NULL,
    order_date TIMESTAMP DEFAULT now()
);
```

### **3.2 Secondary Index**  
```sql
CREATE INDEX idx_orders_customer ON orders(customer_id);
```
- Speeds up lookups by `customer_id`.

### **3.3 Unique Index**  
```sql
CREATE UNIQUE INDEX idx_unique_email ON employees(email);
```
- Ensures emails are **always unique**.

### **3.4 Partial Index**  
```sql
CREATE INDEX idx_active_employees ON employees(name) WHERE active = TRUE;
```
- **Only indexes active employees** for better performance.

### **3.5 Hands-on: Query Performance Testing**  
📌 **Step 1: Create a Large Table**  
```sql
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name STRING NOT NULL,
    category STRING NOT NULL,
    price DECIMAL(10,2)
);
```

📌 **Step 2: Insert 100,000 Sample Rows**  
```sh
cockroach workload init tpcc --warehouses=10
```

📌 **Step 3: Compare Indexed vs. Non-Indexed Queries**  
```sql
EXPLAIN ANALYZE SELECT * FROM products WHERE category = 'Electronics';
CREATE INDEX idx_category ON products(category);
EXPLAIN ANALYZE SELECT * FROM products WHERE category = 'Electronics';
```
- **Compare execution times**.

---

## **4. Primary Keys, Foreign Keys, and Unique Constraints**  

### **4.1 Primary Keys**  
```sql
CREATE TABLE customers (
    id SERIAL PRIMARY KEY,
    name STRING NOT NULL
);
```

### **4.2 Foreign Keys**  
```sql
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    customer_id INT NOT NULL REFERENCES customers(id) ON DELETE CASCADE
);
```

### **4.3 Unique Constraints**  
```sql
ALTER TABLE customers ADD CONSTRAINT unique_email UNIQUE (email);
```

### **4.4 Hands-on: Testing Constraints**  
📌 **Step 1: Insert Data**  
```sql
INSERT INTO customers (id, name) VALUES (1, 'Alice');
INSERT INTO orders (id, customer_id) VALUES (1, 1);
```

📌 **Step 2: Test Foreign Key Constraint**  
```sql
DELETE FROM customers WHERE id = 1;
```
- **Fails** unless `ON DELETE CASCADE` is set.

📌 **Step 3: Test Unique Constraint**  
```sql
INSERT INTO customers (id, name, email) VALUES (2, 'Bob', 'bob@example.com');
INSERT INTO customers (id, name, email) VALUES (3, 'Charlie', 'bob@example.com'); -- Should fail!
```

---

## **5. Partitioning & Sharding**  

Partitioning **stores specific rows** in different locations for performance.

### **5.1 RANGE Partitioning**  
```sql
CREATE TABLE orders_by_region (
    id SERIAL PRIMARY KEY,
    region STRING NOT NULL,
    amount DECIMAL(10,2)
) PARTITION BY LIST (region) (
    PARTITION us_orders VALUES IN ('US-East', 'US-West'),
    PARTITION eu_orders VALUES IN ('EU-Central')
);
```
- **Keeps US & EU orders separate** for performance.

### **5.2 HASH Sharding**  
Automatically **distributes data** across nodes.  
```sql
CREATE TABLE transactions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id INT NOT NULL,
    amount DECIMAL(10,2) NOT NULL
) USING HASH WITH (BUCKET_COUNT = 4);
```
- **Prevents hot spots** in distributed systems.

### **5.3 Hands-on: Data Partitioning**  
📌 **Step 1: Create a Partitioned Table**  
```sql
CREATE TABLE sales (
    id SERIAL PRIMARY KEY,
    region STRING NOT NULL,
    total DECIMAL(10,2) NOT NULL
) PARTITION BY LIST (region) (
    PARTITION north_america VALUES IN ('USA', 'Canada'),
    PARTITION europe VALUES IN ('Germany', 'France')
);
```

📌 **Step 2: Insert Data & Check Partitions**  
```sql
INSERT INTO sales (region, total) VALUES ('USA', 1000.50);
INSERT INTO sales (region, total) VALUES ('France', 500.75);
SHOW RANGES FROM TABLE sales;
```
- **Verifies correct data distribution**.

---

## **6. Summary**  
✅ **CockroachDB supports PostgreSQL-like SQL but has differences from MySQL**  
✅ **Indexes significantly improve query performance**  
✅ **Constraints (PK, FK, UNIQUE) enforce data integrity**  
✅ **Partitioning & sharding optimize performance for large datasets**  

🚀 **Next: Module 5 - Query Optimization & Performance Tuning!**  
Would you like deeper insights into cost-based optimization & query execution plans? 😊
