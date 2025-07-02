---

# **Module 10: CockroachDB Troubleshooting & Maintenance**  

## **1. Debugging Node Failures**  

### **Common Causes of Node Failures**  
- **Network Issues** (e.g., loss of connectivity between nodes)  
- **Hardware Failures** (e.g., disk failures, memory corruption)  
- **Configuration Issues** (e.g., incorrect cluster settings, time sync issues)  
- **Overload & Resource Constraints** (e.g., CPU/RAM exhaustion)  
- **Process Crashes** (e.g., out-of-memory errors, segmentation faults)  

### **Steps to Debug Node Failures**  

#### **1. Check Node Status**  
Run the following command to check the cluster status:  
```sh
cockroach node status --certs-dir=<certs-dir> --host=<node-ip>
```

#### **2. Inspect Logs**  
Check CockroachDB logs for errors:  
```sh
cat /var/log/cockroachdb/cockroach.log | grep ERROR
```

For Kubernetes clusters:  
```sh
kubectl logs <pod-name> -n <namespace>
```

#### **3. Verify Disk Space & Resource Usage**  
```sh
df -h   # Check disk usage
free -m # Check memory usage
top     # Monitor CPU usage
```

#### **4. Check Network Connectivity Between Nodes**  
```sh
ping <other-node-ip>
nc -zv <other-node-ip> 26257  # Check if the CockroachDB port is accessible
```

#### **5. Restart the Node**  
```sh
systemctl restart cockroachdb
```

For Kubernetes:  
```sh
kubectl delete pod <pod-name> -n <namespace>
```

### **Hands-on Lab: Simulating & Recovering from a Node Failure**  
1. **Step 1:** Stop a CockroachDB node  
```sh
systemctl stop cockroachdb
```
2. **Step 2:** Check cluster health  
```sh
cockroach node status --certs-dir=<certs-dir> --host=<any-other-node-ip>
```
3. **Step 3:** Restart the node  
```sh
systemctl start cockroachdb
```
4. **Step 4:** Verify recovery  
```sh
cockroach sql --execute="SHOW CLUSTER SETTING cluster.organization;"
```

---

## **2. Handling Schema Migrations**  

### **Best Practices for Schema Migrations**
- **Use transactions** to ensure atomicity.  
- **Minimize downtime** by using `ALTER TABLE` instead of `DROP & CREATE`.  
- **Use `SHOW JOBS`** to track schema migration jobs.  
- **Perform migrations during off-peak hours.**  
- **Backup before applying schema changes.**  

### **Performing Schema Migrations**  

#### **1. Creating a New Table**  
```sql
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name STRING NOT NULL,
    email STRING UNIQUE NOT NULL,
    created_at TIMESTAMP DEFAULT now()
);
```

#### **2. Adding a Column**  
```sql
ALTER TABLE users ADD COLUMN phone_number STRING;
```

#### **3. Renaming a Column**  
```sql
ALTER TABLE users RENAME COLUMN phone_number TO contact_number;
```

#### **4. Dropping a Column**  
```sql
ALTER TABLE users DROP COLUMN contact_number;
```

#### **5. Rolling Back Schema Changes**
```sql
SHOW JOBS;
CANCEL JOB <job-id>;  -- Only possible if the job is still running
```

### **Hands-on Lab: Schema Migration Simulation**
1. **Step 1:** Create a sample table  
```sql
CREATE TABLE employees (
    emp_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name STRING NOT NULL,
    department STRING
);
```
2. **Step 2:** Add a new column  
```sql
ALTER TABLE employees ADD COLUMN salary DECIMAL(10,2);
```
3. **Step 3:** Rename a column  
```sql
ALTER TABLE employees RENAME COLUMN salary TO base_salary;
```
4. **Step 4:** Verify schema changes  
```sql
SHOW CREATE TABLE employees;
```

---

## **3. Common Errors & Resolutions**  

### **1. Node Not Found / Unavailable**
**Error:**  
```sh
ERROR: node is unavailable; check the status of the cluster
```
**Solution:**  
- Check if the node process is running  
  ```sh
  systemctl status cockroachdb
  ```
- Restart the node  
  ```sh
  systemctl restart cockroachdb
  ```
- Verify cluster connectivity  
  ```sh
  cockroach node status --host=<node-ip>
  ```

### **2. Transaction Deadlock**
**Error:**  
```sql
ERROR: restart transaction: TransactionRetryWithProtoRefreshError
```
**Solution:**  
- Use **transaction retries**  
```sql
BEGIN;
  UPDATE users SET email = 'new_email@example.com' WHERE id = '123';
COMMIT;
```
- Add **retry logic** in applications  

### **3. Insufficient Disk Space**
**Error:**  
```sh
ERROR: disk space below minimum threshold
```
**Solution:**  
- Check disk space  
  ```sh
  df -h
  ```
- Free up space  
  ```sh
  rm -rf /var/log/cockroachdb/*.log
  ```

### **Hands-on Lab: Debugging Errors**
1. **Step 1:** Simulate an error by running an invalid query  
```sql
SELECT * FROM non_existent_table;
```
2. **Step 2:** Check logs  
```sh
cat /var/log/cockroachdb/cockroach.log
```
3. **Step 3:** Fix errors and retry the queries.  

---

## **4. Regular Maintenance Best Practices**  

### **1. Monitoring Cluster Health**
```sh
cockroach node status --host=<node-ip>
```
Or via SQL:  
```sql
SELECT node_id, address, is_live FROM crdb_internal.gossip_nodes;
```

### **2. Backup & Restore**
**Perform a full backup:**  
```sh
cockroach backup bank TO 'nodelocal://1/backup';
```
**Restore from backup:**  
```sh
cockroach restore bank FROM 'nodelocal://1/backup';
```

### **3. Index & Table Optimization**
- **Rebuild indexes** for performance:  
```sql
ALTER TABLE users EXPERIMENTAL_RELOCATE VALUES (ARRAY[1,2], id);
```
- **Vacuum & Analyze tables**:  
```sql
ANALYZE users;
```

### **4. Automating Maintenance with Cron Jobs**
Example: Auto-backup every night  
```sh
echo "0 2 * * * cockroach backup bank TO 'nodelocal://1/backup'" | crontab -
```

### **Hands-on Lab: Automating Backups & Health Checks**
1. **Step 1:** Schedule a cron job for backups  
```sh
crontab -e
```
Add the following entry:  
```sh
0 3 * * * cockroach backup bank TO 'nodelocal://1/backup'
```
2. **Step 2:** Verify the scheduled job  
```sh
crontab -l
```
3. **Step 3:** Test a manual backup & restore.  

---

## **Conclusion**
In this module, you learned:
✅ How to debug node failures  
✅ How to handle schema migrations  
✅ Common CockroachDB errors & resolutions  
✅ Best practices for regular maintenance  

This ensures your CockroachDB cluster remains **highly available, optimized, and resilient** in production. 🚀  

Would you like additional hands-on scenarios or troubleshooting exercises?
