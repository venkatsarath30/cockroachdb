# **Module 3: Installation & Setup (With Hands-on Lab)**

This module covers:
✅ **System requirements & prerequisites**  
✅ **Installation methods** (Binary, Docker, Kubernetes, Cloud)  
✅ **Cluster initialization & verification**  
✅ **Node management & scaling**  

---

## **1. System Requirements & Prerequisites**
### **1.1 Hardware Requirements**
| Component  | Minimum Requirements  | Recommended  |
|------------|-----------------------|-------------|
| **CPU**    | 2 vCPUs                 | 4+ vCPUs  |
| **RAM**    | 4 GB                     | 8+ GB  |
| **Storage**| SSD with 20 GB+          | SSD with 100+ GB |
| **Network**| 1 Gbps                    | 10 Gbps |

### **1.2 Software Requirements**
- **Linux:** Ubuntu 20.04+, CentOS 7+, Oracle Linux 9+
- **MacOS:** 10.15+  
- **Windows:** Use **WSL 2**  
- **Dependencies:** `tar`, `curl`, `bash`, `jq`  
- **Ports:** 
  - **TCP 26257** (SQL & Gossip)
  - **TCP 8080** (Admin UI)

---

## **2. Installation Methods**
### **2.1 Manual Installation (Binary)**
📌 **Step 1: Download CockroachDB Binary**
```sh
curl https://binaries.cockroachdb.com/cockroach-v23.1.9.linux-amd64.tgz | tar -xz
sudo cp -i cockroach-v23.1.9.linux-amd64/cockroach /usr/local/bin/
cockroach version
```

📌 **Step 2: Verify Installation**
```sh
cockroach version
```

---

### **2.2 Install Using Docker**
📌 **Step 1: Pull the CockroachDB Image**
```sh
docker pull cockroachdb/cockroach:v23.1.9
```

📌 **Step 2: Start a Single-Node Cluster**
```sh
docker run -d --name=cockroach-single -p 26257:26257 -p 8080:8080 cockroachdb/cockroach:v23.1.9 start-single-node --insecure
```

📌 **Step 3: Connect to SQL Shell**
```sh
docker exec -it cockroach-single cockroach sql --insecure
```

---

### **2.3 Kubernetes Deployment**
📌 **Step 1: Install kubectl & Helm**
```sh
curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl && sudo mv kubectl /usr/local/bin/
helm repo add cockroachdb https://charts.cockroachdb.com/
helm repo update
```

📌 **Step 2: Deploy CockroachDB**
```sh
kubectl create namespace cockroachdb
helm install cockroachdb cockroachdb/cockroachdb --namespace cockroachdb
```

📌 **Step 3: Check Cluster Status**
```sh
kubectl get pods -n cockroachdb
```

---

### **2.4 Cloud Deployment (AWS, GCP)**
#### **AWS (EC2) Deployment**
📌 **Step 1: Launch EC2 Instances**  
- 3 Instances with **Ubuntu 22.04 LTS**
- **Ports:** 26257, 8080

📌 **Step 2: Install CockroachDB on Each Node**
```sh
curl https://binaries.cockroachdb.com/cockroach-v23.1.9.linux-amd64.tgz | tar -xz
sudo cp -i cockroach-v23.1.9.linux-amd64/cockroach /usr/local/bin/
```

📌 **Step 3: Start Cluster**
```sh
cockroach start --insecure --advertise-addr=<PRIVATE_IP> --join=<PRIVATE_IP_1>,<PRIVATE_IP_2>,<PRIVATE_IP_3> --store=node1 --background
```

📌 **Step 4: Verify Cluster**
```sh
cockroach node status --insecure
```

#### **GCP Deployment**
📌 **Step 1: Create GCE Instances**
```sh
gcloud compute instances create cockroach-node-1 --zone=us-central1-a --machine-type=e2-standard-4 --image-family=ubuntu-2204-lts --image-project=ubuntu-os-cloud
```
- Repeat for `cockroach-node-2` & `cockroach-node-3`.

📌 **Step 2: Install CockroachDB**
```sh
gcloud compute ssh cockroach-node-1 -- sudo apt update && sudo apt install -y cockroachdb
```
- Repeat on all nodes.

📌 **Step 3: Start Cluster**
```sh
cockroach start --insecure --advertise-addr=$(hostname -I | awk '{print $1}') --join=$(gcloud compute instances list --format="value(EXTERNAL_IP)") --store=node1 --background
```

---

## **3. Cluster Initialization & Verification**
📌 **Step 1: Initialize the Cluster**
```sh
cockroach init --insecure --host=localhost:26257
```

📌 **Step 2: Verify Cluster Status**
```sh
cockroach node status --insecure
```

📌 **Step 3: Connect to SQL Shell**
```sh
cockroach sql --insecure
```
```sql
SHOW DATABASES;
```

---

## **4. Node Management & Scaling**
### **4.1 Adding a New Node**
📌 **Step 1: Start a New Node**
```sh
cockroach start --insecure --store=node4 --listen-addr=localhost:26260 --join=localhost:26257,localhost:26258,localhost:26259 --background
```

📌 **Step 2: Verify Cluster Expansion**
```sh
cockroach node status --insecure
```

---

### **4.2 Decommissioning a Node**
📌 **Step 1: Remove a Node**
```sh
cockroach node decommission 2 --insecure
```

📌 **Step 2: Verify Cluster After Decommissioning**
```sh
cockroach node status --insecure
```

---

## **5. Hands-on Lab: Local Multi-Node Cluster**
### **Goal:** Deploy a **3-node cluster** locally and verify data distribution.

📌 **Step 1: Install CockroachDB**
```sh
curl https://binaries.cockroachdb.com/cockroach-v23.1.9.linux-amd64.tgz | tar -xz
sudo cp -i cockroach-v23.1.9.linux-amd64/cockroach /usr/local/bin/
```

📌 **Step 2: Start a 3-Node Cluster**
```sh
cockroach start --insecure --store=node1 --listen-addr=localhost:26257 --http-addr=localhost:8080 --join=localhost:26257,localhost:26258,localhost:26259 --background
cockroach start --insecure --store=node2 --listen-addr=localhost:26258 --http-addr=localhost:8081 --join=localhost:26257,localhost:26258,localhost:26259 --background
cockroach start --insecure --store=node3 --listen-addr=localhost:26259 --http-addr=localhost:8082 --join=localhost:26257,localhost:26258,localhost:26259 --background
```

📌 **Step 3: Initialize Cluster**
```sh
cockroach init --insecure --host=localhost:26257
```

📌 **Step 4: Verify Cluster**
```sh
cockroach node status --insecure
```

📌 **Step 5: Create a Sample Database**
```sh
cockroach sql --insecure -e "CREATE DATABASE testdb;"
cockroach sql --insecure -e "SHOW DATABASES;"
```

📌 **Step 6: Check Data Distribution**
```sh
cockroach sql --insecure -e "SHOW RANGES FROM TABLE testdb.users;"
```

---

## **6. Summary**
✅ **Multiple ways to install CockroachDB (Binary, Docker, Kubernetes, Cloud).**  
✅ **Cluster initialization ensures proper setup before use.**  
✅ **Node management allows scaling up/down as needed.**  
✅ **Hands-on labs reinforce real-world CockroachDB deployment.**  

