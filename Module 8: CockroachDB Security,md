# Module 8: CockroachDB Security & Access Control

## Overview
Security is a crucial aspect of managing a CockroachDB cluster. This module covers:

- **Authentication & Authorization**
- **Role-Based Access Control (RBAC)**
- **Data Encryption (At Rest & In Transit)**
- **Audit Logging & Compliance**

---

## 1. Authentication & Authorization
### Authentication
CockroachDB supports various authentication methods:
- **Password-based authentication**
- **Certificate-based authentication (TLS)**
- **JWT authentication**

#### Setting Up Password Authentication
1. Enable authentication in the cluster:
   ```sh
   cockroach start --insecure --listen-addr=localhost:26257
   ```
2. Create a user with a password:
   ```sql
   CREATE USER admin WITH PASSWORD 'StrongPassword123';
   ```
3. Log in using the new user:
   ```sh
   cockroach sql --user=admin --host=localhost --port=26257
   ```

#### Enabling TLS Authentication
1. Generate TLS certificates:
   ```sh
   cockroach cert create-ca --certs-dir=certs --ca-key=ca.key
   cockroach cert create-node localhost --certs-dir=certs --ca-key=ca.key
   cockroach cert create-client root --certs-dir=certs --ca-key=ca.key
   ```
2. Start CockroachDB with TLS:
   ```sh
   cockroach start --certs-dir=certs --listen-addr=localhost:26257
   ```
3. Connect using TLS authentication:
   ```sh
   cockroach sql --certs-dir=certs --user=root
   ```

### Authorization
Authorization in CockroachDB is managed using SQL privileges. Users must be granted specific permissions.

#### Granting Permissions
```sql
GRANT SELECT, INSERT ON TABLE users TO admin;
```

#### Revoking Permissions
```sql
REVOKE INSERT ON TABLE users FROM admin;
```

---

## 2. Role-Based Access Control (RBAC)
RBAC allows you to manage access at a role level instead of per user.

### Creating a Role
```sql
CREATE ROLE read_only;
```

### Assigning Permissions to a Role
```sql
GRANT SELECT ON DATABASE mydb TO read_only;
```

### Assigning Users to a Role
```sql
GRANT read_only TO admin;
```

### Checking Role Membership
```sql
SHOW GRANTS FOR admin;
```

**Best Practices:**
- Use **roles instead of direct grants** for easier management.
- Assign **least privilege access** to users.
- Regularly review access levels.

---

## 3. Data Encryption (At Rest & In Transit)

### Encryption At Rest
CockroachDB encrypts data at rest using **AES-256 encryption** when running in **enterprise mode**.

#### Enabling Encryption at Rest
1. Start CockroachDB with encryption flags:
   ```sh
   cockroach start --store=path=store,attrs=ssd,encryption=aes128-ctr:key=your-encryption-key
   ```
2. Verify encryption:
   ```sh
   cockroach debug encryption-status --stores=store
   ```

### Encryption In Transit
By default, CockroachDB supports **TLS encryption** for secure communication.

#### Enforcing TLS for Clients
```sh
cockroach sql --certs-dir=certs --user=root
```

#### Enforcing TLS for Nodes
Start nodes with TLS enabled:
```sh
cockroach start --certs-dir=certs
```

**Best Practices:**
- Use **TLS encryption** for all client connections.
- Store encryption keys securely.
- Regularly rotate encryption keys.

---

## 4. Audit Logging & Compliance
Audit logging helps track database activity for security and compliance.

### Enabling Audit Logs
```sh
cockroach start --log-dir=audit_logs --sql-audit-dir=audit_logs
```

### Viewing Audit Logs
```sh
cat audit_logs/cockroach.log
```

### Creating an Audit Policy
```sql
CREATE EVENT TRIGGER audit_users ON ALL TABLES IN SCHEMA public EXECUTE FUNCTION log_activity();
```

### Checking System Events
```sql
SELECT * FROM crdb_internal.cluster_events;
```

**Best Practices:**
- Regularly review audit logs.
- Enable logging for sensitive tables.
- Integrate logs with a SIEM system.

---

## Hands-on Lab: Securing CockroachDB
### Step 1: Start a Secure CockroachDB Cluster
```sh
cockroach start-single-node --certs-dir=certs --listen-addr=localhost:26257
```

### Step 2: Create Users and Roles
```sql
CREATE USER dba WITH PASSWORD 'SecureDBA123';
CREATE ROLE db_readonly;
GRANT SELECT ON DATABASE mydb TO db_readonly;
GRANT db_readonly TO dba;
```

### Step 3: Enforce TLS Authentication
```sh
cockroach sql --certs-dir=certs --user=dba
```

### Step 4: Enable and Check Encryption
```sh
cockroach debug encryption-status --stores=store
```

### Step 5: Enable and View Audit Logs
```sh
cat audit_logs/cockroach.log
```

---

## Summary
- **Authentication**: Use passwords or TLS certificates.
- **RBAC**: Assign roles instead of direct permissions.
- **Encryption**: Enable at-rest and in-transit encryption.
- **Audit Logging**: Track user activity and system events.

By implementing these security measures, you can protect your CockroachDB cluster from unauthorized access and ensure compliance with security policies.

