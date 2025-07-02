# **Module 11: CockroachDB Advanced Features & Use Cases**  

## **1. Geo-Partitioning for Low Latency**  

### **What is Geo-Partitioning?**  
Geo-partitioning allows you to **store data closer to users** based on geographic locations. This reduces latency and improves performance for multi-region deployments.

### **How Geo-Partitioning Works in CockroachDB**
- **Partitions data by region** using `PARTITION BY LIST`.  
- **Assigns specific nodes to each region** using `ALTER PARTITION`.  
- **Ensures fast queries by keeping data local** to the user’s region.  

### **Example Use Case**  
A global e-commerce application wants to store user data **in the nearest region** to reduce latency.  

### **Step-by-Step Implementation**  

#### **1. Create a Multi-Region Table**  
```sql
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name STRING NOT NULL,
    email STRING UNIQUE NOT NULL,
    region STRING NOT NULL,
    created_at TIMESTAMP DEFAULT now()
) LOCALITY REGIONAL BY ROW;
```

#### **2. Partition the Table by Region**  
```sql
ALTER TABLE users PARTITION BY LIST (region) (
    PARTITION us_east VALUES IN ('us-east'),
    PARTITION us_west VALUES IN ('us-west'),
    PARTITION europe VALUES IN ('europe')
);
```

#### **3. Assign Data to Specific Nodes (Replication Constraints)**  
```sql
ALTER PARTITION us_east OF TABLE users CONFIGURE ZONE USING constraints='[+region=us-east]';
ALTER PARTITION us_west OF TABLE users CONFIGURE ZONE USING constraints='[+region=us-west]';
ALTER PARTITION europe OF TABLE users CONFIGURE ZONE USING constraints='[+region=europe]';
```

#### **4. Verify Geo-Partitioning**
```sql
SHOW CREATE TABLE users;
```

### **Hands-on Lab: Simulating Multi-Region Queries**
1. **Step 1:** Insert test data  
```sql
INSERT INTO users (name, email, region) VALUES ('Alice', 'alice@example.com', 'us-east');
INSERT INTO users (name, email, region) VALUES ('Bob', 'bob@example.com', 'europe');
```
2. **Step 2:** Run queries from different regions and measure latency  
```sql
SELECT * FROM users WHERE region = 'us-east';
```
3. **Step 3:** Verify data placement  
```sql
SHOW PARTITIONS FROM users;
```

---

## **2. Change Data Capture (CDC)**  

### **What is CDC?**
Change Data Capture (CDC) **streams real-time database changes** (INSERT, UPDATE, DELETE) to external systems like **Kafka, Google Pub/Sub, or Webhooks**.

### **Common Use Cases**
- **Real-time analytics** (e.g., updating dashboards)  
- **Event-driven architectures** (e.g., triggering notifications)  
- **Database replication** (e.g., syncing to a data warehouse)  

### **How to Enable CDC in CockroachDB**  

#### **1. Create a Table for CDC**  
```sql
CREATE TABLE orders (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    customer_id UUID NOT NULL,
    amount DECIMAL(10,2) NOT NULL,
    status STRING CHECK (status IN ('pending', 'shipped', 'delivered')),
    created_at TIMESTAMP DEFAULT now()
);
```

#### **2. Enable CDC for the Table**
```sql
CREATE CHANGEFEED FOR orders 
WITH format=json, sink='kafka://localhost:9092';
```

#### **3. Verify CDC Stream**
```sh
kafka-console-consumer --bootstrap-server localhost:9092 --topic orders
```

### **Hands-on Lab: Streaming Data to Kafka**
1. **Step 1:** Insert test data  
```sql
INSERT INTO orders (customer_id, amount, status) VALUES ('123', 100.00, 'pending');
```
2. **Step 2:** Check if Kafka receives the event  
```sh
kafka-console-consumer --bootstrap-server localhost:9092 --topic orders
```
3. **Step 3:** Update data and verify CDC stream  
```sql
UPDATE orders SET status = 'shipped' WHERE id = '123';
```

---

## **3. JSON & Vector Data Support**  

### **1. JSON Support in CockroachDB**  
CockroachDB **supports JSONB** for semi-structured data, similar to PostgreSQL.

#### **Example Use Case**  
A social media app stores **user preferences** as JSON instead of separate columns.

#### **Creating a Table with JSON Data**  
```sql
CREATE TABLE profiles (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name STRING NOT NULL,
    data JSONB
);
```

#### **Inserting JSON Data**  
```sql
INSERT INTO profiles (name, data) 
VALUES ('Alice', '{"likes": ["sports", "music"], "age": 30}');
```

#### **Querying JSON Data**  
```sql
SELECT name, data->>'likes' FROM profiles WHERE data->>'age' = '30';
```

---

### **2. Vector Data Support in CockroachDB**  
CockroachDB supports **vector embeddings** for **AI/ML workloads** (e.g., similarity search).  

#### **Example Use Case**  
A recommendation system stores **vector embeddings** for user preferences.

#### **Creating a Table with Vector Data**  
```sql
CREATE TABLE embeddings (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    item_name STRING NOT NULL,
    embedding VECTOR(3) -- Vector with 3 dimensions
);
```

#### **Inserting Vector Data**  
```sql
INSERT INTO embeddings (item_name, embedding) 
VALUES ('Item A', '[0.1, 0.8, 0.3]');
```

#### **Finding Similar Vectors**  
```sql
SELECT * FROM embeddings ORDER BY embedding <-> '[0.1, 0.7, 0.3]' LIMIT 5;
```

### **Hands-on Lab: Storing and Querying JSON & Vector Data**
1. **Step 1:** Insert user preferences as JSON  
```sql
INSERT INTO profiles (name, data) VALUES ('Bob', '{"likes": ["movies", "tech"], "age": 25}');
```
2. **Step 2:** Query JSON fields  
```sql
SELECT name FROM profiles WHERE data->>'likes' @> '["movies"]';
```
3. **Step 3:** Insert and search vector data  
```sql
INSERT INTO embeddings (item_name, embedding) VALUES ('Product X', '[0.5, 0.2, 0.9]');
SELECT * FROM embeddings ORDER BY embedding <-> '[0.5, 0.3, 0.9]' LIMIT 1;
```

---

## **4. Serverless CockroachDB**  

### **What is Serverless CockroachDB?**  
- Fully **managed** CockroachDB  
- **Auto-scales** based on demand  
- **Free tier available** for small workloads  
- No manual infrastructure management  

### **How to Deploy a Serverless Database**
1. **Sign up on CockroachDB Serverless**  
   - Visit [https://www.cockroachlabs.com](https://www.cockroachlabs.com)  
   - Create a free account  

2. **Create a Database**  
```sql
CREATE DATABASE my_serverless_db;
```

3. **Create a Table**  
```sql
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name STRING NOT NULL
);
```

4. **Connect to Serverless CockroachDB**  
Use `cockroach sql` with your **connection string**:
```sh
cockroach sql --url "your_connection_string"
```

### **Hands-on Lab: Deploying a Serverless Database**
1. **Step 1:** Create a free CockroachDB Serverless cluster.  
2. **Step 2:** Connect via `cockroach sql`.  
3. **Step 3:** Create a table and insert data.  
4. **Step 4:** Verify auto-scaling by running **load tests**.

---

## **Conclusion**
In this module, you learned:  
✅ How to **geo-partition data** for low-latency queries  
✅ How to **stream real-time data** using **Change Data Capture (CDC)**  
✅ How to **store and query JSON & vector embeddings**  
✅ How to deploy **Serverless CockroachDB**  
