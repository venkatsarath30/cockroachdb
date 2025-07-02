# Module 6: CockroachDB Backup & Recovery

## Overview
Backup and recovery are crucial aspects of managing a CockroachDB cluster, ensuring data durability, resilience, and quick recovery from failures. This module covers the following key topics:

- **Point-in-Time Recovery (PITR)**
- **Full & Incremental Backups**
- **Restoring Databases & Disaster Recovery Strategies**

---

## 1. Point-in-Time Recovery (PITR)
### Concept
PITR allows restoring data to a specific timestamp within a defined retention window. This is achieved using incremental backups and revision history.

### Enabling PITR
To enable PITR, set a **GC TTL (Garbage Collection Time-To-Live)** period on tables to retain historical data for a defined duration.

```sql
ALTER TABLE my_table CONFIGURE ZONE USING gc.ttlseconds = 86400; -- Retain data for 1 day
```

### Performing a PITR Recovery
1. **Initiate an incremental backup with revision history**:
   ```sql
   BACKUP mydb TO 'nodelocal://1/backups/mydb' WITH revision_history;
   ```
2. **Restore to a specific timestamp**:
   ```sql
   RESTORE mydb FROM 'nodelocal://1/backups/mydb' AS OF SYSTEM TIME '2025-03-30 10:00:00';
   ```

---

## 2. Full & Incremental Backups
### Full Backup
A full backup captures the entire database state at a point in time.

```sql
BACKUP DATABASE mydb TO 'nodelocal://1/full_backups/mydb';
```

### Incremental Backup
An incremental backup only captures changes made since the last backup.

```sql
BACKUP DATABASE mydb TO 'nodelocal://1/incremental_backups/mydb' INCREMENTAL FROM 'nodelocal://1/full_backups/mydb';
```

**Best Practices:**
- Perform **full backups periodically** (e.g., daily or weekly).
- Perform **incremental backups frequently** (e.g., every hour) to minimize recovery time.
- Store backups in a **remote location** (e.g., Google Cloud Storage, AWS S3, NFS).

---

## 3. Restoring Databases & Disaster Recovery Strategies
### Restoring a Full Backup
To restore a full database backup:

```sql
RESTORE DATABASE mydb FROM 'nodelocal://1/full_backups/mydb';
```

### Restoring an Incremental Backup
To restore the database including incremental backups:

```sql
RESTORE DATABASE mydb FROM ('nodelocal://1/full_backups/mydb', 'nodelocal://1/incremental_backups/mydb');
```

### Disaster Recovery Strategy
1. **Automate Backups**: Use a scheduled job to run backups at regular intervals.
   ```sql
   CREATE SCHEDULE FOR BACKUP DATABASE mydb INTO 'nodelocal://1/scheduled_backups/mydb' RECURRING '1h';
   ```
2. **Test Recovery Procedures**: Periodically restore backups in a test environment.
3. **Monitor Backup Status**: Check recent backups using:
   ```sql
   SHOW BACKUPS IN 'nodelocal://1/backups';
   ```
4. **Multi-Region Backup Storage**: Store copies in different geographical locations for redundancy.

---

## Hands-on Lab: CockroachDB Backup & Recovery
### Prerequisites
- A running CockroachDB cluster
- CockroachDB client installed
- Sufficient storage space for backups

### Step 1: Setup Backup Location
1. Start a local cluster:
   ```sh
   cockroach start-single-node --insecure --listen-addr=localhost:26257 --store=node1
   ```
2. Open SQL shell:
   ```sh
   cockroach sql --insecure
   ```
3. Create a database and table:
   ```sql
   CREATE DATABASE testdb;
   CREATE TABLE testdb.users (id SERIAL PRIMARY KEY, name STRING);
   INSERT INTO testdb.users (name) VALUES ('Alice'), ('Bob');
   ```

### Step 2: Perform a Full Backup
```sql
BACKUP DATABASE testdb TO 'nodelocal://1/backups/full';
```

### Step 3: Make Changes and Perform Incremental Backup
```sql
INSERT INTO testdb.users (name) VALUES ('Charlie');
BACKUP DATABASE testdb TO 'nodelocal://1/backups/incremental' INCREMENTAL FROM 'nodelocal://1/backups/full';
```

### Step 4: Simulate Data Loss and Restore
```sql
DROP DATABASE testdb;
RESTORE DATABASE testdb FROM ('nodelocal://1/backups/full', 'nodelocal://1/backups/incremental');
```

### Step 5: Verify Data Restoration
```sql
SELECT * FROM testdb.users;
```

---

## Summary
- **PITR** allows recovery to a specific point in time.
- **Full backups** capture the entire database, while **incremental backups** store only changes.
- **Restoration** ensures business continuity during failures.
- **Disaster recovery strategies** include automation, monitoring, and multi-region storage.
