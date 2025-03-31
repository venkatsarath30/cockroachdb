# **Module 2: CockroachDB Architecture (With Hands-on Lab)**

## **1. Clustered Architecture Overview**
### **1.1 CockroachDB Cluster Structure**
A CockroachDB cluster consists of multiple **nodes** that work together to provide:
- **High availability** (no single point of failure).
- **Automatic data distribution** (data is divided into smaller chunks called **ranges**).
- **Fault tolerance** (if a node fails, other nodes take over).
  
### **1.2 Key Components**
| Component  | Description |
|------------|------------|
| **Nodes**  | Independent servers running CockroachDB. |
| **Ranges** | Data is split into **512 MiB** chunks (**ranges**) for distribution. |
| **Leases** | Each range has a **leaseholder node** that coordinates reads/writes. |
| **Replicas** | Each range is replicated across multiple nodes for fault tolerance. |
| **KV Store** | Data is stored as key-value pairs but accessed via SQL. |

### **1.3 Hands-on: Setting Up a Local Cluster**
We will set up a **3-node CockroachDB cluster** on a local machine.

📌 **Step 1: Download and Install CockroachDB**
```sh
curl https://binaries.cockroachdb.com/cockroach-v23.1.9.linux-amd64.tgz | tar -xz
sudo cp -i cockroach-v23.1.9.linux-amd64/cockroach /usr/local/bin/
cockroach version
```

📌 **Step 2: Start a 3-Node Cluster Locally**
```sh
cockroach start --insecure --store=node1 --listen-addr=localhost:26257 --http-addr=localhost:8080 --join=localhost:26257,localhost:26258,localhost:26259 --background
cockroach start --insecure --store=node2 --listen-addr=localhost:26258 --http-addr=localhost:8081 --join=localhost:26257,localhost:26258,localhost:26259 --background
cockroach start --insecure --store=node3 --listen-addr=localhost:26259 --http-addr=localhost:8082 --join=localhost:26257,localhost:26258,localhost:26259 --background
```
- This starts a **3-node CockroachDB cluster**.
- Each node runs on a different **port**.

📌 **Step 3: Verify Cluster Status**
```sh
cockroach node status --insecure
```
- This will show the **nodes, addresses, and replica counts**.

📌 **Step 4: Open CockroachDB SQL Shell**
```sh
cockroach sql --insecure
```
- This starts an interactive SQL shell connected to the cluster.

📌 **Step 5: Create a Test Database and Table**
```sql
CREATE DATABASE testdb;
USE testdb;
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name STRING NOT NULL,
    email STRING UNIQUE NOT NULL
);
INSERT INTO users (name, email) VALUES ('Alice', 'alice@example.com'), ('Bob', 'bob@example.com');
SELECT * FROM users;
```
- We just created a **distributed SQL database** with a **users table**.

---

## **2. Raft Consensus Algorithm & Replication**
### **2.1 What is Raft?**
- Raft is a **consensus algorithm** that ensures all nodes have the same data.
- Uses a **Leader-Follower model**:
  - **Leader**: Handles all writes.
  - **Followers**: Replicate data and vote for a new leader if needed.

### **2.2 Hands-on: Checking Data Replication**
📌 **Step 1: Check Data Distribution**
```sql
SHOW RANGES FROM TABLE testdb.users;
```
- This shows **where the table's data is stored**.

📌 **Step 2: Simulate a Node Failure**
```sh
cockroach node decommission 2 --insecure
```
- **Node 2 is removed**, but data remains **available**.

📌 **Step 3: Add Node 2 Back**
```sh
cockroach node recommission 2 --insecure
```
- **Node 2 rejoins** the cluster and starts replicating data again.

---

## **3. Distributed Transactions & Data Distribution**
### **3.1 How Transactions Work**
CockroachDB supports **ACID transactions** across multiple nodes.

📌 **Example: Atomic Bank Transfer**
```sql
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;
```
- Ensures **both updates happen together** or not at all.

### **3.2 Hands-on: Checking Data Distribution**
📌 **Step 1: Show Where Data is Stored**
```sql
SELECT node_id, store_id, range_id, replicas FROM crdb_internal.ranges;
```
- Lists **which nodes store which data**.

📌 **Step 2: Balance Data Across Nodes**
```sh
cockroach debug compact-engine --insecure
```
- Forces CockroachDB to **re-balance data** across nodes.

---

## **4. Multi-Region & Multi-Cloud Deployments**
### **4.1 Why Multi-Region?**
- **Faster query response times** (keep data close to users).
- **High availability** (survives regional failures).
- **Data compliance** (store user data in the right region).

### **4.2 Hands-on: Deploying Multi-Region Locally**
📌 **Step 1: Start Multi-Region Nodes**
```sh
cockroach start --insecure --advertise-addr=us-east-1 --join=us-east-1,us-west-1,eu-central-1 --store=us-east
cockroach start --insecure --advertise-addr=us-west-1 --join=us-east-1,us-west-1,eu-central-1 --store=us-west
cockroach start --insecure --advertise-addr=eu-central-1 --join=us-east-1,us-west-1,eu-central-1 --store=eu-central
```
- **3 nodes** are deployed in **different regions**.

📌 **Step 2: Partition Data by Region**
```sql
ALTER TABLE users PARTITION BY LIST (region) (
    PARTITION us_users VALUES IN ('us-east', 'us-west'),
    PARTITION eu_users VALUES IN ('eu-central')
);
```
- Queries **stay within the user's region**, improving performance.

📌 **Step 3: Verify Multi-Region Setup**
```sh
cockroach sql --insecure -e "SHOW REGIONS FROM CLUSTER"
```
- Confirms that data is **distributed across multiple regions**.

---

## **5. Summary**
✅ **CockroachDB uses a shared-nothing, distributed architecture**.  
✅ **The Raft consensus algorithm ensures strong consistency**.  
✅ **Data is automatically replicated and balanced across nodes**.  
✅ **Multi-region deployments improve performance and resilience**.  

---

## **6. Lab Exercises (Practice)**
1️⃣ **Deploy a 5-node CockroachDB cluster locally**.  
2️⃣ **Check where data is stored using `SHOW RANGES FROM TABLE`**.  
3️⃣ **Decommission a node and verify if the cluster remains available**.  
4️⃣ **Create a multi-region partitioned table and verify queries remain local**.  

🚀 **Next: Advanced Query Execution & Performance Optimization**  
Would you like more detailed troubleshooting steps or automation with Terraform? 😊
