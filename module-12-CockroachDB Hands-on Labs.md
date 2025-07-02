# Module 12: CockroachDB Hands-on Labs & Capstone Project

## 1. Deploying a CockroachDB Cluster

### **Objective**
Set up a multi-node CockroachDB cluster using Docker and Kubernetes.

### **Hands-on Lab: Deploying CockroachDB Locally (Docker)**
#### **Prerequisites**
- Install Docker

#### **Steps**
1. **Create a Docker network**
   ```bash
   docker network create cockroachdb
   ```
2. **Start a three-node CockroachDB cluster**
   ```bash
   docker run -d --name=roach1 --hostname=roach1 --net=cockroachdb -p 26257:26257 -p 8080:8080 cockroachdb/cockroach:v23.1 start --join=roach1,roach2,roach3
   docker run -d --name=roach2 --hostname=roach2 --net=cockroachdb cockroachdb/cockroach:v23.1 start --join=roach1,roach2,roach3
   docker run -d --name=roach3 --hostname=roach3 --net=cockroachdb cockroachdb/cockroach:v23.1 start --join=roach1,roach2,roach3
   ```
3. **Initialize the cluster**
   ```bash
   docker exec -it roach1 ./cockroach init --insecure
   ```
4. **Verify cluster status**
   ```bash
   docker exec -it roach1 ./cockroach node status --insecure
   ```
5. **Access the Admin UI**: Open `http://localhost:8080` in a browser.

### **Hands-on Lab: Deploying on Kubernetes**
1. Deploy a StatefulSet using Kubernetes.
2. Configure persistent storage.
3. Verify cluster health with `cockroach node status`.

---

## 2. Implementing HA & Failover Scenarios

### **Objective**
Test CockroachDB’s high availability features by simulating node failures.

### **Hands-on Lab: Simulating Node Failures**
#### **Steps**
1. **Stop a node**
   ```bash
   docker stop roach2
   ```
2. **Check cluster health**
   ```bash
   docker exec -it roach1 ./cockroach node status --insecure
   ```
3. **Restart the node and verify recovery**
   ```bash
   docker start roach2
   ```

---

## 3. Optimize Queries & Performance

### **Objective**
Optimize query performance using indexing, partitions, and query tuning.

### **Hands-on Lab: Performance Tuning**
#### **Steps**
1. **Create an index**
   ```sql
   CREATE INDEX idx_user_email ON users(email);
   ```
2. **Analyze query execution plan**
   ```sql
   EXPLAIN ANALYZE SELECT * FROM users WHERE email='test@example.com';
   ```
3. **Optimize queries by reducing scans**
   ```sql
   SET cluster_setting sql.defaults.experimental_follower_reads.enabled = true;
   ```

---

## 4. Secure the Database

### **Objective**
Implement authentication, authorization, and encryption.

### **Hands-on Lab: Securing CockroachDB**
#### **Steps**
1. **Create an admin user**
   ```sql
   CREATE USER admin WITH PASSWORD 'securepassword';
   GRANT ADMIN TO admin;
   ```
2. **Enable TLS encryption** (Modify `cockroach start` to include `--certs-dir=certs`)
3. **Restrict database access**
   ```sql
   REVOKE ALL ON database mydb FROM public;
   ```

---

## 5. Final Project: Deploying a Production-Ready CockroachDB Cluster

### **Objective**
Deploy a secure, high-performance CockroachDB cluster in production.

### **Tasks**
1. **Deploy CockroachDB on Kubernetes with StatefulSets**
2. **Implement HA and load balancing using HAProxy**
3. **Optimize schema design for real-world workloads**
4. **Set up automated backups and monitoring**
5. **Secure the cluster with TLS and RBAC policies**

This capstone project integrates all key learnings to create a robust CockroachDB deployment.

