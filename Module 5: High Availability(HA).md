# **Module 5: High Availability & Fault Tolerance (With Hands-on Lab)**  

This module covers:  
✅ **Node Failures & Auto-Rebalancing**  
✅ **Distributed Replication & Data Availability**  
✅ **Multi-Region Deployment Best Practices**  
✅ **Load Balancing Techniques**  

---

## **1. Node Failures & Auto-Rebalancing**  

### **1.1 What Happens When a Node Fails?**  
In CockroachDB, **automatic rebalancing** ensures that:  
- **Data remains available** as long as a majority of replicas are up.  
- **A new leader is elected** if the previous leader node goes down.  
- **Ranges are redistributed** across remaining nodes to balance the load.  

### **1.2 Hands-on: Simulating a Node Failure & Auto-Recovery**  

📌 **Step 1: Start a 3-Node Cluster Locally**  
```sh
cockroach start --insecure --store=node1 --listen-addr=localhost:26257 --join=localhost:26257,localhost:26258,localhost:26259 --background
cockroach start --insecure --store=node2 --listen-addr=localhost:26258 --join=localhost:26257,localhost:26258,localhost:26259 --background
cockroach start --insecure --store=node3 --listen-addr=localhost:26259 --join=localhost:26257,localhost:26258,localhost:26259 --background
```

📌 **Step 2: Check Cluster Health**  
```sh
cockroach node status --insecure
```
- Displays **all active nodes**.

📌 **Step 3: Decommission a Node (Simulate Node Failure)**  
```sh
cockroach node decommission 2 --insecure
```
- Node **2** is removed from the cluster.  
- CockroachDB **rebalances data** across the remaining nodes.  

📌 **Step 4: Verify Auto-Rebalancing**  
```sh
cockroach node status --insecure
SHOW RANGES;
```
- Checks **which nodes store which ranges**.  

📌 **Step 5: Recommission Node 2 (Bring it Back)**  
```sh
cockroach node recommission 2 --insecure
```
- Node **2 rejoins** the cluster.  

---

## **2. Distributed Replication & Data Availability**  

### **2.1 How CockroachDB Handles Replication**  
- Every **range has 3+ replicas** stored across different nodes.  
- Uses **Raft consensus algorithm** to maintain consistency.  
- If a node **fails**, other replicas take over automatically.  

### **2.2 Hands-on: Viewing Replication Status**  

📌 **Step 1: Check Replication Factor**  
```sh
cockroach sql --insecure -e "SHOW ZONE CONFIGURATION FOR RANGE default;"
```
- Displays **default replication settings** (usually `num_replicas: 3`).  

📌 **Step 2: Manually Set Replication Factor**  
```sh
cockroach zone set company.employees --insecure --file=- <<EOF
num_replicas: 5
EOF
```
- Increases replication to **5 replicas per range**.  

📌 **Step 3: Verify Data Availability After a Node Failure**  
```sh
cockroach node decommission 3 --insecure
cockroach sql --insecure -e "SELECT * FROM company.employees;"
```
- Data remains accessible, **even with a failed node**.  

---

## **3. Multi-Region Deployment Best Practices**  

### **3.1 Why Deploy CockroachDB Across Multiple Regions?**  
✅ **High availability** – Survives regional failures.  
✅ **Low latency** – Keeps data close to users.  
✅ **Regulatory compliance** – Stores user data in designated regions.  

### **3.2 Hands-on: Deploying a Multi-Region Cluster (Local Simulation)**  

📌 **Step 1: Start Nodes in Different Regions**  
```sh
cockroach start --insecure --advertise-addr=us-east-1 --join=us-east-1,us-west-1,eu-central-1 --store=us-east --background
cockroach start --insecure --advertise-addr=us-west-1 --join=us-east-1,us-west-1,eu-central-1 --store=us-west --background
cockroach start --insecure --advertise-addr=eu-central-1 --join=us-east-1,us-west-1,eu-central-1 --store=eu-central --background
```
- Simulates a **3-region cluster** (US-East, US-West, Europe).  

📌 **Step 2: Set Up Geo-Partitioning (Keep Data in Specific Regions)**  
```sql
ALTER TABLE customers PARTITION BY LIST (region) (
    PARTITION us_customers VALUES IN ('us-east', 'us-west'),
    PARTITION eu_customers VALUES IN ('eu-central')
);
```
- Ensures **US customers’ data stays in US nodes**.  

📌 **Step 3: Verify Regional Data Distribution**  
```sql
SHOW RANGES FROM TABLE customers;
```

📌 **Step 4: Test Regional Availability After a Node Failure**  
```sh
cockroach node decommission 1 --insecure
cockroach sql --insecure -e "SELECT * FROM customers;"
```
- **Ensures data remains available** in US & Europe.  

---

## **4. Load Balancing Techniques**  

### **4.1 Why Load Balancing is Important**  
- **Distributes queries** evenly across nodes.  
- **Reduces latency** by directing requests to the nearest node.  
- **Prevents overload** on a single node.  

### **4.2 Hands-on: Setting Up Load Balancing**  

📌 **Step 1: Install HAProxy (Load Balancer)**  
```sh
sudo apt install haproxy -y
```

📌 **Step 2: Configure HAProxy**  
Edit `/etc/haproxy/haproxy.cfg`:
```sh
frontend cockroachdb
    bind *:26257
    mode tcp
    default_backend cockroachdb_nodes

backend cockroachdb_nodes
    mode tcp
    balance roundrobin
    server node1 localhost:26257 check
    server node2 localhost:26258 check
    server node3 localhost:26259 check
```

📌 **Step 3: Restart HAProxy**  
```sh
sudo systemctl restart haproxy
```

📌 **Step 4: Test Load Balancing**  
```sh
cockroach sql --url="postgresql://root@localhost:26257/defaultdb?sslmode=disable"
```
- Queries are automatically **distributed across nodes**.  

---

## **5. Hands-on Lab: Deploy a Resilient Multi-Region Cluster**  

### **💡 Goal:**
Set up a **3-region CockroachDB cluster** with automatic failover & load balancing.

📌 **Step 1: Start a 3-Node Cluster**  
```sh
cockroach start --insecure --store=node1 --listen-addr=localhost:26257 --join=localhost:26257,localhost:26258,localhost:26259 --background
cockroach start --insecure --store=node2 --listen-addr=localhost:26258 --join=localhost:26257,localhost:26258,localhost:26259 --background
cockroach start --insecure --store=node3 --listen-addr=localhost:26259 --join=localhost:26257,localhost:26258,localhost:26259 --background
```

📌 **Step 2: Create & Partition a Table**  
```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    region STRING NOT NULL,
    name STRING NOT NULL
) PARTITION BY LIST (region) (
    PARTITION us_users VALUES IN ('us-east', 'us-west'),
    PARTITION eu_users VALUES IN ('eu-central')
);
```

📌 **Step 3: Test Auto-Failover & Load Balancing**  
```sh
cockroach node decommission 3 --insecure
cockroach sql --insecure -e "SELECT * FROM users;"
```
- **Ensures queries continue running smoothly**.

---

## **6. Summary**  
✅ **CockroachDB automatically rebalances when a node fails**  
✅ **Replication ensures data remains available**  
✅ **Multi-region deployments improve performance & reliability**  
✅ **Load balancing distributes queries efficiently**  

