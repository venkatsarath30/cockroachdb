I've already included **Module 1: Introduction to CockroachDB** with detailed notes, step-by-step installation, and code examples. Here’s an expanded breakdown for better clarity:  

---

# **Module 1: Introduction to CockroachDB**  

## **What is CockroachDB?**  
CockroachDB is a **distributed SQL database** designed for resilience, horizontal scalability, and high availability. It is compatible with **PostgreSQL** and optimized for **global-scale deployments**.  

### **Key Features**  
✅ **Distributed SQL** – ACID transactions across multiple nodes.  
✅ **Horizontal Scalability** – Easily scale by adding nodes.  
✅ **Automatic Failover** – Handles node failures without downtime.  
✅ **Geo-Partitioning** – Data remains close to users to reduce latency.  
✅ **PostgreSQL-Compatible** – Supports PostgreSQL drivers and syntax.  

---

## **CockroachDB Installation & Setup**  

### **1. Install CockroachDB**  
#### **On Linux/macOS**  
```sh
curl https://binaries.cockroachdb.com/cockroach-latest.linux-amd64.tgz | tar xvz
sudo cp -i cockroach-latest.linux-amd64/cockroach /usr/local/bin/
cockroach version
```

#### **On Windows (Using WSL)**
```sh
wget https://binaries.cockroachdb.com/cockroach-latest.windows-amd64.zip
unzip cockroach-latest.windows-amd64.zip
move cockroach-latest.windows-amd64/cockroach.exe C:\cockroach\
cockroach version
```

---

### **2. Start a Single-Node CockroachDB Cluster**
```sh
cockroach start-single-node --insecure --listen-addr=localhost
```
> This command starts a **single-node** instance running locally.

---

### **3. Open CockroachDB SQL Shell**
```sh
cockroach sql --insecure --host=localhost
```
> This launches the **interactive SQL shell** to execute queries.

---

### **4. Create a Database and Table**
```sql
CREATE DATABASE testdb;
USE testdb;

CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name STRING,
    age INT
);

INSERT INTO users (name, age) VALUES ('Alice', 25), ('Bob', 30);

SELECT * FROM users;
```
> This creates a database, a `users` table, inserts records, and retrieves them.

---

### **5. Stop CockroachDB Cluster**
```sh
cockroach quit --insecure
```
> Gracefully stops the CockroachDB server.

---

