# Module 9: CockroachDB Automation & Scripting

## Overview
Automating CockroachDB deployments and management helps streamline operations, improve consistency, and reduce manual errors. This module covers:

- **Automating Deployments with Terraform & Ansible (AWS & GCP)**
- **Scripting with CockroachDB CLI & SQL**
- **CI/CD Integration for CockroachDB**

---

## 1. Automating Deployments with Terraform & Ansible (AWS & GCP)
### Deploying CockroachDB with Terraform
Terraform is used to provision CockroachDB clusters in AWS and GCP.

#### Example: Terraform for AWS
```hcl
provider "aws" {
  region = "us-west-2"
}

resource "aws_instance" "cockroach_node" {
  count         = 3
  ami          = "ami-12345678"
  instance_type = "t3.medium"
  key_name      = "my-key"

  tags = {
    Name = "cockroach-node-${count.index}"
  }
}
```

#### Example: Terraform for GCP
```hcl
provider "google" {
  project = "my-gcp-project"
  region  = "us-central1"
}

resource "google_compute_instance" "cockroach_node" {
  count        = 3
  name         = "cockroach-node-${count.index}"
  machine_type = "n1-standard-2"
  zone         = "us-central1-a"

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-10"
    }
  }
}
```

### Automating Deployment with Ansible
Ansible helps automate the configuration and startup of CockroachDB nodes.

#### Ansible Playbook for CockroachDB Setup
```yaml
- name: Install CockroachDB
  hosts: all
  become: yes
  tasks:
    - name: Download CockroachDB
      get_url:
        url: "https://binaries.cockroachdb.com/cockroach-v23.1.0.linux-amd64.tgz"
        dest: "/tmp/cockroach.tgz"

    - name: Extract CockroachDB
      ansible.builtin.unarchive:
        src: "/tmp/cockroach.tgz"
        dest: "/usr/local/bin/"
        remote_src: yes

    - name: Start CockroachDB
      command: cockroach start --insecure --listen-addr={{ ansible_host }}
```

---

## 2. Scripting with CockroachDB CLI & SQL
### Using CockroachDB CLI
CockroachDB provides a CLI for managing the database.

#### Common CockroachDB CLI Commands
```sh
# Start a single-node cluster
cockroach start-single-node --insecure --listen-addr=localhost:26257

# Create a database
cockroach sql --insecure -e "CREATE DATABASE mydb;"

# List databases
cockroach sql --insecure -e "SHOW DATABASES;"

# Backup database
cockroach sql --insecure -e "BACKUP mydb TO 'nodelocal://1/backup';"
```

### Automating with SQL Scripts
#### Example SQL Script for User & Table Setup
```sql
CREATE USER admin WITH PASSWORD 'StrongPassword123';
CREATE DATABASE company;
GRANT ALL ON DATABASE company TO admin;

USE company;
CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    name STRING,
    role STRING,
    salary DECIMAL
);

INSERT INTO employees (name, role, salary) VALUES ('Alice', 'Engineer', 90000);
```

Run the script using:
```sh
cockroach sql --insecure -f setup.sql
```

---

## 3. CI/CD Integration for CockroachDB
### Setting Up CI/CD with GitHub Actions
GitHub Actions can automate database schema migrations and testing.

#### Example GitHub Actions Workflow
```yaml
name: CockroachDB CI/CD
on: [push]

jobs:
  test:
    runs-on: ubuntu-latest
    services:
      cockroach:
        image: cockroachdb/cockroach:v23.1.0
        ports:
          - 26257:26257
    steps:
      - name: Checkout Code
        uses: actions/checkout@v3

      - name: Run Migrations
        run: |
          cockroach sql --host=localhost --insecure -e "CREATE DATABASE testdb;"
          cockroach sql --host=localhost --insecure -f migrations.sql

      - name: Run Tests
        run: pytest tests/
```

### Deploying Schema Changes with Liquibase
Liquibase allows versioned database migrations.

#### Example Liquibase Changeset
```xml
<databaseChangeLog>
    <changeSet id="1" author="dev">
        <createTable tableName="users">
            <column name="id" type="SERIAL" primaryKey="true"/>
            <column name="name" type="STRING"/>
            <column name="email" type="STRING" unique="true"/>
        </createTable>
    </changeSet>
</databaseChangeLog>
```

Run migration:
```sh
liquibase update
```

---

## Hands-on Lab: Automating CockroachDB
### Step 1: Deploy CockroachDB with Terraform
```sh
terraform init
terraform apply
```

### Step 2: Configure CockroachDB with Ansible
```sh
ansible-playbook setup-cockroachdb.yml
```

### Step 3: Automate Database Setup with SQL Script
```sh
cockroach sql --insecure -f setup.sql
```

### Step 4: Enable CI/CD for Schema Changes
1. Add Liquibase changeset.
2. Run CI/CD pipeline using GitHub Actions.

---

## Summary
- **Terraform & Ansible**: Automate deployments on AWS/GCP.
- **CockroachDB CLI & SQL**: Script database setup and backups.
- **CI/CD Integration**: Automate schema changes with GitHub Actions & Liquibase.

By implementing automation, you can efficiently manage and scale CockroachDB with minimal manual effort.

