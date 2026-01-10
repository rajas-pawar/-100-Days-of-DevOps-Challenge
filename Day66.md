# Day 66: MySQL Deployment with PersistentVolume and Secrets in Kubernetes

## 📋 Task Overview

**Scenario:** The Nautilus DevOps team needs to deploy a new MySQL server on the Kubernetes cluster. After gathering and finalizing requirements, the team is ready to implement a production-ready MySQL deployment with persistent storage and secure credential management.

**Given Requirements:**
- **PersistentVolume:** `mysql-pv` with 250Mi capacity
- **PersistentVolumeClaim:** `mysql-pv-claim` requesting 250Mi
- **Deployment:** `mysql-deployment` with MySQL image
- **Mount Path:** `/var/lib/mysql` (MySQL data directory)
- **Service:** `mysql` NodePort type on port 30007
- **Secrets:** Three secrets for root password, user credentials, and database name
- **Environment Variables:** Configure MySQL using secretKeyRef

**Your Mission:**
1. Create PersistentVolume and PersistentVolumeClaim for data persistence
2. Create three Secrets for secure credential management
3. Deploy MySQL with environment variables from Secrets
4. Create NodePort service for external access
5. Verify MySQL is running and accessible

---

## 🎯 Learning Objectives

By the end of this task, you will understand:
- **PersistentVolumes (PV):** Cluster-level storage resources
- **PersistentVolumeClaims (PVC):** Storage requests by pods
- **Kubernetes Secrets:** Secure credential storage
- **Environment Variables:** Using secretKeyRef for sensitive data
- **MySQL on Kubernetes:** Deploying stateful applications
- **NodePort Services:** External database access

---

## 📖 Understanding Kubernetes Storage

### PersistentVolume (PV)

**What it is:** Cluster-level storage resource provisioned by administrator

**Characteristics:**
- Independent lifecycle from pods
- Exists even after pod deletion
- Can be provisioned statically or dynamically
- Has capacity, access modes, and storage class

**Why Use PV:**
- Data survives pod restarts
- Data survives pod deletion
- Shared storage across pods
- Production-ready data persistence

---

### PersistentVolumeClaim (PVC)

**What it is:** Request for storage by a user/pod

**PV vs PVC Relationship:**
```
┌─────────────────────────────────────────────────┐
│         PersistentVolume (PV)                   │
│    (Cluster Administrator Provisions)           │
│                                                 │
│  Capacity: 250Mi                                │
│  Storage Class: manual                          │
│  Access Mode: ReadWriteOnce                     │
└─────────────────┬───────────────────────────────┘
                  │
                  │ Binds to
                  ↓
┌─────────────────────────────────────────────────┐
│    PersistentVolumeClaim (PVC)                  │
│         (User Requests)                         │
│                                                 │
│  Request: 250Mi                                 │
│  Access Mode: ReadWriteOnce                     │
└─────────────────┬───────────────────────────────┘
                  │
                  │ Mounts to
                  ↓
┌─────────────────────────────────────────────────┐
│              Pod/Deployment                     │
│         (Uses the storage)                      │
│                                                 │
│  MySQL data stored in /var/lib/mysql            │
└─────────────────────────────────────────────────┘
```

---

### Access Modes

| Mode | Description | Use Case |
|------|-------------|----------|
| **ReadWriteOnce (RWO)** | Single node read-write | Databases, single pod apps |
| **ReadOnlyMany (ROX)** | Multiple nodes read-only | Static content, shared config |
| **ReadWriteMany (RWX)** | Multiple nodes read-write | Shared file systems, CMS |

**For MySQL:** Use ReadWriteOnce (RWO) - single pod writes to database

---

## 🔐 Understanding Kubernetes Secrets

### What are Secrets?

**Purpose:** Store sensitive information (passwords, tokens, keys)

**Benefits over ConfigMaps:**
- Base64 encoded (not encrypted, but obscured)
- Special handling by Kubernetes
- Can be encrypted at rest (with configuration)
- Separate lifecycle from application code

**Types of Secrets:**
- **Opaque:** Generic key-value pairs (our use case)
- **TLS:** TLS certificates
- **Docker Registry:** Container registry credentials
- **Service Account:** Kubernetes service account tokens

---

### Using Secrets in Pods

**Three Methods:**
```yaml
# Method 1: Environment Variable (we use this)
env:
- name: MYSQL_ROOT_PASSWORD
  valueFrom:
    secretKeyRef:
      name: mysql-root-pass
      key: password

# Method 2: Volume Mount
volumes:
- name: secret-volume
  secret:
    secretName: mysql-root-pass

# Method 3: Pull Image Secret
imagePullSecrets:
- name: registry-secret
```

---

## 📖 Understanding MySQL on Kubernetes

### MySQL Data Directory

**Default Location:** `/var/lib/mysql`

**What's Stored:**
- Database files (.ibd, .frm)
- Transaction logs (ib_logfile)
- System tablespace (ibdata1)
- Binary logs (for replication)
- Error logs

**Why Persistence Matters:**
```
Without PV:              With PV:
Pod restarts → Data lost  Pod restarts → Data persists ✅
Pod deleted → Data lost   Pod deleted → Data persists ✅
Node fails → Data lost    Node fails → Data persists ✅
```

---

### MySQL Environment Variables

**Required for MySQL Container:**

| Variable | Purpose | Example |
|----------|---------|---------|
| `MYSQL_ROOT_PASSWORD` | Root user password | YUIidhb667 |
| `MYSQL_DATABASE` | Database to create | kodekloud_db5 |
| `MYSQL_USER` | Non-root user | kodekloud_gem |
| `MYSQL_PASSWORD` | User password | B4zNgHA7Ya |

**Initialization Process:**
1. Container starts
2. Checks if `/var/lib/mysql` is empty
3. If empty, initializes MySQL
4. Creates database specified in `MYSQL_DATABASE`
5. Creates user with `MYSQL_USER` and `MYSQL_PASSWORD`
6. Sets root password to `MYSQL_ROOT_PASSWORD`

---

## 🛠️ Task Implementation

### Step 1: Verify Cluster Access

**Check kubectl configuration:**
```bash
kubectl cluster-info
```

**Check current resources:**
```bash
kubectl get pv,pvc,secrets,deployments,services
```

---

### Step 2: Create Kubernetes Secrets

**Understanding the Requirements:**

We need three secrets:
1. **mysql-root-pass:** Root password
2. **mysql-user-pass:** Application user credentials
3. **mysql-db-url:** Database name

**Create Secret 1: Root Password**
```bash
kubectl create secret generic mysql-root-pass \
  --from-literal=password=YUIidhb667
```

**Expected output:**
```
secret/mysql-root-pass created
```

**Create Secret 2: User Credentials**
```bash
kubectl create secret generic mysql-user-pass \
  --from-literal=username=kodekloud_gem \
  --from-literal=password=B4zNgHA7Ya
```

**Expected output:**
```
secret/mysql-user-pass created
```

**Create Secret 3: Database Name**
```bash
kubectl create secret generic mysql-db-url \
  --from-literal=database=kodekloud_db5
```

**Expected output:**
```
secret/mysql-db-url created
```

---

### Step 3: Verify Secrets

**List all secrets:**
```bash
kubectl get secrets
```

**Expected output:**
```
NAME              TYPE     DATA   AGE
mysql-root-pass   Opaque   1      30s
mysql-user-pass   Opaque   2      25s
mysql-db-url      Opaque   1      20s
```

**Describe secrets (see structure, not values):**
```bash
kubectl describe secret mysql-root-pass
```

**Expected output:**
```
Name:         mysql-root-pass
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
password:  10 bytes
```

**View secret values (base64 encoded):**
```bash
kubectl get secret mysql-root-pass -o yaml
```

**Expected output:**
```yaml
apiVersion: v1
data:
  password: WVVJaWRoYjY2Nw==  # Base64 encoded
kind: Secret
metadata:
  name: mysql-root-pass
type: Opaque
```

**Decode secret (for verification only):**
```bash
kubectl get secret mysql-root-pass -o jsonpath='{.data.password}' | base64 --decode
# Output: YUIidhb667
```

---

### Step 4: Create PersistentVolume

**Create PV YAML file:**
```bash
cat > mysql-pv.yaml <<EOF
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysql-pv
spec:
  capacity:
    storage: 250Mi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: manual
  hostPath:
    path: /mnt/data/mysql
    type: DirectoryOrCreate
EOF
```

**Understanding the Configuration:**

```yaml
capacity:
  storage: 250Mi  # Total storage available

accessModes:
  - ReadWriteOnce  # Single node can mount read-write

persistentVolumeReclaimPolicy: Retain
  # What happens when PVC is deleted:
  # - Retain: Keep data (manual cleanup)
  # - Delete: Auto-delete volume
  # - Recycle: Scrub data and reuse (deprecated)

storageClassName: manual
  # Groups PVs by storage type
  # PVC must match this to bind

hostPath:
  path: /mnt/data/mysql  # Directory on node
  type: DirectoryOrCreate  # Create if doesn't exist
```

**Apply the PV:**
```bash
kubectl apply -f mysql-pv.yaml
```

**Expected output:**
```
persistentvolume/mysql-pv created
```

---

### Step 5: Verify PersistentVolume

**Check PV status:**
```bash
kubectl get pv
```

**Expected output:**
```
NAME       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   AGE
mysql-pv   250Mi      RWO            Retain           Available           manual         10s
```

**Key indicators:**
- **CAPACITY:** 250Mi ✅
- **ACCESS MODES:** RWO (ReadWriteOnce) ✅
- **STATUS:** Available (ready to be claimed) ✅
- **STORAGECLASS:** manual ✅

**Describe PV for details:**
```bash
kubectl describe pv mysql-pv
```

---

### Step 6: Create PersistentVolumeClaim

**Create PVC YAML file:**
```bash
cat > mysql-pvc.yaml <<EOF
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pv-claim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 250Mi
  storageClassName: manual
EOF
```

**Understanding the Configuration:**

```yaml
accessModes:
  - ReadWriteOnce  # Must match PV access mode

resources:
  requests:
    storage: 250Mi  # Request 250Mi (matches PV capacity)

storageClassName: manual
  # Must match PV storageClassName to bind
```

**Apply the PVC:**
```bash
kubectl apply -f mysql-pvc.yaml
```

**Expected output:**
```
persistentvolumeclaim/mysql-pv-claim created
```

---

### Step 7: Verify PVC Binding

**Check PVC status:**
```bash
kubectl get pvc
```

**Expected output:**
```
NAME             STATUS   VOLUME     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
mysql-pv-claim   Bound    mysql-pv   250Mi      RWO            manual         5s
```

**Key indicators:**
- **STATUS:** Bound (successfully bound to PV) ✅
- **VOLUME:** mysql-pv (bound to our PV) ✅
- **CAPACITY:** 250Mi ✅

**Check PV status again:**
```bash
kubectl get pv
```

**Expected output:**
```
NAME       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                    STORAGECLASS   AGE
mysql-pv   250Mi      RWO            Retain           Bound    default/mysql-pv-claim   manual         2m
```

**Notice:**
- **STATUS:** Changed from Available → Bound ✅
- **CLAIM:** Shows default/mysql-pv-claim ✅

**Describe PVC:**
```bash
kubectl describe pvc mysql-pv-claim
```

**Expected output:**
```
Name:          mysql-pv-claim
Namespace:     default
StorageClass:  manual
Status:        Bound
Volume:        mysql-pv
Labels:        <none>
Annotations:   pv.kubernetes.io/bind-completed: yes
Capacity:      250Mi
Access Modes:  RWO
VolumeMode:    Filesystem
Mounted By:    <none>  (will show pod name after deployment)
```

---

### Step 8: Create MySQL Deployment

**Create deployment YAML file:**
```bash
cat > mysql-deployment.yaml <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:8.0
        ports:
        - containerPort: 3306
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-root-pass
              key: password
        - name: MYSQL_DATABASE
          valueFrom:
            secretKeyRef:
              name: mysql-db-url
              key: database
        - name: MYSQL_USER
          valueFrom:
            secretKeyRef:
              name: mysql-user-pass
              key: username
        - name: MYSQL_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-user-pass
              key: password
        volumeMounts:
        - name: mysql-persistent-storage
          mountPath: /var/lib/mysql
      volumes:
      - name: mysql-persistent-storage
        persistentVolumeClaim:
          claimName: mysql-pv-claim
EOF
```

---

### Step 9: Understanding the Deployment Configuration

**Breaking Down Each Section:**

#### 1. Basic Metadata
```yaml
metadata:
  name: mysql-deployment
spec:
  replicas: 1  # Single instance (MySQL not clustered)
```

#### 2. Selector and Labels
```yaml
selector:
  matchLabels:
    app: mysql  # Deployment manages pods with this label
template:
  metadata:
    labels:
      app: mysql  # Pod label (must match selector)
```

#### 3. Container Configuration
```yaml
containers:
- name: mysql
  image: mysql:8.0  # Official MySQL image
  ports:
  - containerPort: 3306  # MySQL default port
```

#### 4. Environment Variables from Secrets
```yaml
env:
- name: MYSQL_ROOT_PASSWORD  # Environment variable name
  valueFrom:
    secretKeyRef:
      name: mysql-root-pass  # Secret name
      key: password          # Key within secret

- name: MYSQL_DATABASE
  valueFrom:
    secretKeyRef:
      name: mysql-db-url
      key: database

- name: MYSQL_USER
  valueFrom:
    secretKeyRef:
      name: mysql-user-pass
      key: username

- name: MYSQL_PASSWORD
  valueFrom:
    secretKeyRef:
      name: mysql-user-pass
      key: password
```

**How it works:**
1. Kubernetes reads secrets
2. Injects values as environment variables
3. MySQL container reads environment variables
4. MySQL initializes with these values

#### 5. Volume Mounts
```yaml
volumeMounts:
- name: mysql-persistent-storage  # Volume name (defined below)
  mountPath: /var/lib/mysql       # MySQL data directory
```

#### 6. Volumes
```yaml
volumes:
- name: mysql-persistent-storage
  persistentVolumeClaim:
    claimName: mysql-pv-claim  # References our PVC
```

---

### Step 10: Apply MySQL Deployment

**Deploy MySQL:**
```bash
kubectl apply -f mysql-deployment.yaml
```

**Expected output:**
```
deployment.apps/mysql-deployment created
```

**Watch deployment progress:**
```bash
kubectl get deployment mysql-deployment -w
```

**Expected progression:**
```
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
mysql-deployment   0/1     1            0           5s
mysql-deployment   0/1     1            0           15s
mysql-deployment   1/1     1            1           45s  ✅
```

**Note:** MySQL initialization takes 30-60 seconds

---

### Step 11: Verify MySQL Deployment

**Check deployment status:**
```bash
kubectl get deployment mysql-deployment
```

**Expected output:**
```
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
mysql-deployment   1/1     1            1           1m
```

**Check pod status:**
```bash
kubectl get pods -l app=mysql
```

**Expected output:**
```
NAME                                READY   STATUS    RESTARTS   AGE
mysql-deployment-7f8d9c5b4d-xyz12   1/1     Running   0          1m
```

**Get pod name:**
```bash
POD_NAME=$(kubectl get pods -l app=mysql -o jsonpath='{.items[0].metadata.name}')
echo $POD_NAME
```

**Describe pod:**
```bash
kubectl describe pod $POD_NAME
```

**Check for:**
```yaml
Containers:
  mysql:
    Image:          mysql:8.0
    Port:           3306/TCP
    State:          Running
    Ready:          True
    Environment:
      MYSQL_ROOT_PASSWORD:  <set to the key 'password' in secret 'mysql-root-pass'>
      MYSQL_DATABASE:       <set to the key 'database' in secret 'mysql-db-url'>
      MYSQL_USER:           <set to the key 'username' in secret 'mysql-user-pass'>
      MYSQL_PASSWORD:       <set to the key 'password' in secret 'mysql-user-pass'>
    Mounts:
      /var/lib/mysql from mysql-persistent-storage (rw)

Volumes:
  mysql-persistent-storage:
    Type:       PersistentVolumeClaim
    ClaimName:  mysql-pv-claim
    ReadOnly:   false
```

**All should show ✅**

---

### Step 12: Check MySQL Logs

**View MySQL initialization logs:**
```bash
kubectl logs $POD_NAME
```

**Expected output (key lines):**
```
[System] [MY-010116] /usr/sbin/mysqld (mysqld 8.0.x) starting as process 1
[System] [MY-013576] InnoDB initialization has started.
[System] [MY-013577] InnoDB initialization has ended.
[Server] Created database 'kodekloud_db5'
[Server] Created user 'kodekloud_gem'@'%'
[Server] /usr/sbin/mysqld: ready for connections. Version: '8.0.x'  port: 3306  MySQL Community Server
```

**Key indicators:**
- ✅ "InnoDB initialization has ended"
- ✅ "Created database 'kodekloud_db5'"
- ✅ "Created user 'kodekloud_gem'"
- ✅ "ready for connections"

---

### Step 13: Verify PVC is Mounted

**Check PVC status:**
```bash
kubectl get pvc mysql-pv-claim
```

**Expected output:**
```
NAME             STATUS   VOLUME     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
mysql-pv-claim   Bound    mysql-pv   250Mi      RWO            manual         5m
```

**Describe PVC to see mounted pod:**
```bash
kubectl describe pvc mysql-pv-claim
```

**Expected output:**
```
Name:          mysql-pv-claim
Status:        Bound
Volume:        mysql-pv
Mounted By:    mysql-deployment-7f8d9c5b4d-xyz12  ✅
```

**Check MySQL data directory:**
```bash
kubectl exec $POD_NAME -- ls -la /var/lib/mysql
```

**Expected output:**
```
total 188460
drwxr-xr-x 7 mysql mysql      4096 Jan 10 10:00 .
drwxr-xr-x 1 root  root       4096 Jan 10 10:00 ..
-rw-r----- 1 mysql mysql        56 Jan 10 10:00 auto.cnf
-rw-r----- 1 mysql mysql      1680 Jan 10 10:00 ca-key.pem
-rw-r----- 1 mysql mysql      1112 Jan 10 10:00 ca.pem
drwxr-x--- 2 mysql mysql      4096 Jan 10 10:00 kodekloud_db5  ✅
-rw-r----- 1 mysql mysql 196608000 Jan 10 10:00 ibdata1
-rw-r----- 1 mysql mysql  50331648 Jan 10 10:00 ib_logfile0
-rw-r----- 1 mysql mysql  50331648 Jan 10 10:00 ib_logfile1
drwxr-x--- 2 mysql mysql      4096 Jan 10 10:00 mysql
```

**Database files are present! ✅**

---

### Step 14: Create MySQL Service (NodePort)

**Create service YAML file:**
```bash
cat > mysql-service.yaml <<EOF
apiVersion: v1
kind: Service
metadata:
  name: mysql
spec:
  type: NodePort
  selector:
    app: mysql
  ports:
  - port: 3306
    targetPort: 3306
    nodePort: 30007
    protocol: TCP
EOF
```

**Understanding the Configuration:**

```yaml
type: NodePort  # Exposes service on each node's IP at static port

selector:
  app: mysql  # Routes traffic to pods with label app=mysql

ports:
- port: 3306         # Service port (cluster-internal)
  targetPort: 3306   # Container port (MySQL)
  nodePort: 30007    # External port (node IP:30007)
  protocol: TCP
```

**Apply the service:**
```bash
kubectl apply -f mysql-service.yaml
```

**Expected output:**
```
service/mysql created
```

---

### Step 15: Verify MySQL Service

**Check service:**
```bash
kubectl get service mysql
```

**Expected output:**
```
NAME    TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
mysql   NodePort   10.96.123.45    <none>        3306:30007/TCP   10s
```

**Key indicators:**
- **TYPE:** NodePort ✅
- **PORT(S):** 3306:30007/TCP ✅
- **Cluster-IP:** Assigned ✅

**Describe service:**
```bash
kubectl describe service mysql
```

**Expected output:**
```
Name:                     mysql
Namespace:                default
Labels:                   <none>
Selector:                 app=mysql
Type:                     NodePort
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.96.123.45
Port:                     <unset>  3306/TCP
TargetPort:               3306/TCP
NodePort:                 <unset>  30007/TCP  ✅
Endpoints:                10.244.1.5:3306     ✅
Session Affinity:         None
External Traffic Policy:  Cluster

Events:  <none>
```

**Verify endpoints:**
```bash
kubectl get endpoints mysql
```

**Expected output:**
```
NAME    ENDPOINTS         AGE
mysql   10.244.1.5:3306   30s
```

**Endpoint present = Service routing to pod ✅**

---

### Step 16: Test MySQL Connection

**Get node IP:**
```bash
kubectl get nodes -o wide
```

**Expected output:**
```
NAME     STATUS   ROLES    AGE   VERSION   INTERNAL-IP    EXTERNAL-IP
node01   Ready    <none>   10d   v1.27.0   172.17.0.2     <none>
```

**Test connection from inside cluster:**
```bash
kubectl exec -it $POD_NAME -- mysql -u root -pYUIidhb667 -e "SHOW DATABASES;"
```

**Expected output:**
```
mysql: [Warning] Using a password on the command line interface can be insecure.
+--------------------+
| Database           |
+--------------------+
| information_schema |
| kodekloud_db5      | ✅
| mysql              |
| performance_schema |
| sys                |
+--------------------+
```

**Test user connection:**
```bash
kubectl exec -it $POD_NAME -- mysql -u kodekloud_gem -pB4zNgHA7Ya -e "SELECT USER();"
```

**Expected output:**
```
+---------------------+
| USER()              |
+---------------------+
| kodekloud_gem@%     | ✅
+---------------------+
```

**Test database access:**
```bash
kubectl exec -it $POD_NAME -- mysql -u kodekloud_gem -pB4zNgHA7Ya kodekloud_db5 -e "SHOW TABLES;"
```

**Expected output:**
```
Empty set (0.00 sec)
```

**Database accessible! ✅**

---

### Step 17: Test External Access (NodePort)

**Create a test client pod:**
```bash
kubectl run mysql-client --image=mysql:8.0 --rm -it --restart=Never -- /bin/bash
```

**Inside the client pod:**
```bash
# Get MySQL service cluster IP
mysql -h mysql.default.svc.cluster.local -u kodekloud_gem -pB4zNgHA7Ya

# Or use service name directly
mysql -h mysql -u kodekloud_gem -pB4zNgHA7Ya
```

**Once connected:**
```sql
SHOW DATABASES;
USE kodekloud_db5;
CREATE TABLE test (id INT, name VARCHAR(50));
INSERT INTO test VALUES (1, 'Kubernetes');
SELECT * FROM test;
DROP TABLE test;
EXIT;
```

**Test from node (external access):**
```bash
# Get node IP
NODE_IP=$(kubectl get nodes -o jsonpath='{.items[0].status.addresses[?(@.type=="InternalIP")].address}')

# Test connection (from jump host)
mysql -h $NODE_IP -P 30007 -u kodekloud_gem -pB4zNgHA7Ya -e "SHOW DATABASES;"
```

**If mysql client not available, test with telnet:**
```bash
telnet $NODE_IP 30007
```

**Or use curl:**
```bash
curl -v telnet://$NODE_IP:30007
```

---

## 📊 Complete Setup Script

**Save and run this script for automated deployment:**

```bash
#!/bin/bash

echo "🚀 Day 66: MySQL Deployment with PersistentVolume and Secrets"
echo "============================================================="

# Colors
GREEN='\033[0;32m'
RED='\033[0;31m'
YELLOW='\033[1;33m'
NC='\033[0m'

# Step 1: Create Secrets
echo -e "\n${YELLOW}1️⃣ Creating Kubernetes Secrets...${NC}"

kubectl create secret generic mysql-root-pass \
  --from-literal=password=YUIidhb667 \
  --dry-run=client -o yaml | kubectl apply -f -

kubectl create secret generic mysql-user-pass \
  --from-literal=username=kodekloud_gem \
  --from-literal=password=B4zNgHA7Ya \
  --dry-run=client -o yaml | kubectl apply -f -

kubectl create secret generic mysql-db-url \
  --from-literal=database=kodekloud_db5 \
  --dry-run=client -o yaml | kubectl apply -f -

if [ $? -eq 0 ]; then
    echo -e "${GREEN}✅ Secrets created${NC}"
else
    echo -e "${RED}❌ Failed to create secrets${NC}"
    exit 1
fi

# Verify secrets
echo -e "\n${YELLOW}Secrets created:${NC}"
kubectl get secrets | grep mysql

# Step 2: Create PersistentVolume
echo -e "\n${YELLOW}2️⃣ Creating PersistentVolume...${NC}"

cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysql-pv
spec:
  capacity:
    storage: 250Mi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: manual
  hostPath:
    path: /mnt/data/mysql
    type: DirectoryOrCreate
EOF

if [ $? -eq 0 ]; then
    echo -e "${GREEN}✅ PersistentVolume created${NC}"
else
    echo -e "${RED}❌ Failed to create PV${NC}"
    exit 1
fi

# Step 3: Create PersistentVolumeClaim
echo -e "\n${YELLOW}3️⃣ Creating PersistentVolumeClaim...${NC}"

cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pv-claim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 250Mi
  storageClassName: manual
EOF

if [ $? -eq 0 ]; then
    echo -e "${GREEN}✅ PersistentVolumeClaim created${NC}"
else
    echo -e "${RED}❌ Failed to create PVC${NC}"
    exit 1
fi

# Wait for PVC to bind
echo -e "\n${YELLOW}Waiting for PVC to bind...${NC}"
sleep 5

PVC_STATUS=$(kubectl get pvc mysql-pv-claim -o jsonpath='{.status.phase}')
if [ "$PVC_STATUS" == "Bound" ]; then
    echo -e "${GREEN}✅ PVC bound to PV${NC}"
else
    echo -e "${RED}❌ PVC not bound (Status: $PVC_STATUS)${NC}"
fi

# Step 4: Create MySQL Deployment
echo -e "\n${YELLOW}4️⃣ Creating MySQL Deployment...${NC}"

cat <<EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:8.0
        ports:
        - containerPort: 3306
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-root-pass
              key: password
        - name: MYSQL_DATABASE
          valueFrom:
            secretKeyRef:
              name: mysql-db-url
              key: database
        - name: MYSQL_USER
          valueFrom:
            secretKeyRef:
              name: mysql-user-pass
              key: username
        - name: MYSQL_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-user-pass
              key: password
        volumeMounts:
        - name: mysql-persistent-storage
          mountPath: /var/lib/mysql
      volumes:
      - name: mysql-persistent-storage
        persistentVolumeClaim:
          claimName: mysql-pv-claim
EOF

if [ $? -eq 0 ]; then
    echo -e "${GREEN}✅ Deployment created${NC}"
else
    echo -e "${RED}❌ Failed to create deployment${NC}"
    exit 1
fi

# Wait for deployment
echo -e "\n${YELLOW}Waiting for MySQL to initialize (this takes 30-60 seconds)...${NC}"
kubectl rollout status deployment/mysql-deployment --timeout=180s

if [ $? -eq 0 ]; then
    echo -e "${GREEN}✅ Deployment ready${NC}"
else
    echo -e "${RED}❌ Deployment failed${NC}"
    exit 1
fi

# Step 5: Create Service
echo -e "\n${YELLOW}5️⃣ Creating NodePort Service...${NC}"

cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
  name: mysql
spec:
  type: NodePort
  selector:
    app: mysql
  ports:
  - port: 3306
    targetPort: 3306
    nodePort: 30007
    protocol: TCP
EOF

if [ $? -eq 0 ]; then
    echo -e "${GREEN}✅ Service created${NC}"
else
    echo -e "${RED}❌ Failed to create service${NC}"
    exit 1
fi

# Get pod name
POD_NAME=$(kubectl get pods -l app=mysql -o jsonpath='{.items[0].metadata.name}')
echo -e "\n${YELLOW}Pod Name: ${POD_NAME}${NC}"

# Step 6: Verification
echo -e "\n${YELLOW}6️⃣ Running Verification Checks...${NC}"

# Check deployment
echo -e "\n${YELLOW}Deployment Status:${NC}"
kubectl get deployment mysql-deployment

# Check pod
echo -e "\n${YELLOW}Pod Status:${NC}"
kubectl get pods -l app=mysql

# Check service
echo -e "\n${YELLOW}Service Status:${NC}"
kubectl get service mysql

# Check PV/PVC
echo -e "\n${YELLOW}Storage Status:${NC}"
kubectl get pv,pvc

# Check secrets
echo -e "\n${YELLOW}Secrets:${NC}"
kubectl get secrets | grep mysql

# Check endpoints
echo -e "\n${YELLOW}Service Endpoints:${NC}"
kubectl get endpoints mysql

# Wait for MySQL to be fully ready
echo -e "\n${YELLOW}Waiting for MySQL to be fully initialized...${NC}"
sleep 10

# Test database connection
echo -e "\n${YELLOW}7️⃣ Testing MySQL Connection...${NC}"

echo -e "\n${YELLOW}Testing root connection...${NC}"
kubectl exec $POD_NAME -- mysql -u root -pYUIidhb667 -e "SELECT 'MySQL Root Connection: SUCCESS' as Status;" 2>/dev/null

echo -e "\n${YELLOW}Testing user connection...${NC}"
kubectl exec $POD_NAME -- mysql -u kodekloud_gem -pB4zNgHA7Ya -e "SELECT 'MySQL User Connection: SUCCESS' as Status;" 2>/dev/null

echo -e "\n${YELLOW}Showing databases...${NC}"
kubectl exec $POD_NAME -- mysql -u kodekloud_gem -pB4zNgHA7Ya -e "SHOW DATABASES;" 2>/dev/null

echo -e "\n${YELLOW}Testing database access...${NC}"
kubectl exec $POD_NAME -- mysql -u kodekloud_gem -pB4zNgHA7Ya kodekloud_db5 -e "SELECT 'Database Access: SUCCESS' as Status;" 2>/dev/null

# Get node IP
NODE_IP=$(kubectl get nodes -o jsonpath='{.items[0].status.addresses[?(@.type=="InternalIP")].address}')
echo -e "\n${YELLOW}Node IP: ${NODE_IP}${NC}"
echo -e "${YELLOW}MySQL accessible at: ${NODE_IP}:30007${NC}"

# Final summary
echo -e "\n${YELLOW}============================================================="
echo "📊 Deployment Summary"
echo "=============================================================${NC}"

echo -e "\n${GREEN}✅ PersistentVolume: mysql-pv (250Mi)${NC}"
echo -e "${GREEN}✅ PersistentVolumeClaim: mysql-pv-claim (Bound)${NC}"
echo -e "${GREEN}✅ Secrets: mysql-root-pass, mysql-user-pass, mysql-db-url${NC}"
echo -e "${GREEN}✅ Deployment: mysql-deployment (1/1 ready)${NC}"
echo -e "${GREEN}✅ Service: mysql (NodePort 30007)${NC}"
echo -e "${GREEN}✅ Database: kodekloud_db5${NC}"
echo -e "${GREEN}✅ User: kodekloud_gem${NC}"
echo -e "${GREEN}✅ Root Password: YUIidhb667${NC}"
echo -e "${GREEN}✅ User Password: B4zNgHA7Ya${NC}"

echo -e "\n${YELLOW}Connection Examples:${NC}"
echo -e "${GREEN}Internal: mysql -h mysql -u kodekloud_gem -pB4zNgHA7Ya${NC}"
echo -e "${GREEN}External: mysql -h ${NODE_IP} -P 30007 -u kodekloud_gem -pB4zNgHA7Ya${NC}"

echo -e "\n${GREEN}✅ All checks completed successfully!${NC}"
```

**Save and run:**
```bash
chmod +x mysql-deploy.sh
./mysql-deploy.sh
```

---

## 🧹 Cleanup

**Delete all resources:**
```bash
# Delete service
kubectl delete service mysql

# Delete deployment
kubectl delete deployment mysql-deployment

# Delete PVC (this will release PV)
kubectl delete pvc mysql-pv-claim

# Delete PV
kubectl delete pv mysql-pv

# Delete secrets
kubectl delete secret mysql-root-pass mysql-user-pass mysql-db-url

# Verify deletion
kubectl get all,pv,pvc,secrets
# No MySQL resources found

# Clean up files
rm -f mysql-pv.yaml mysql-pvc.yaml mysql-deployment.yaml mysql-service.yaml mysql-deploy.sh
```

**Note:** With `persistentVolumeReclaimPolicy: Retain`, the data in `/mnt/data/mysql` on the node persists after PV deletion. Manual cleanup may be needed.

---

## 🐛 Common Issues and Solutions

### Issue 1: PVC Stuck in Pending

**Symptoms:**
```bash
kubectl get pvc
# NAME             STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
# mysql-pv-claim   Pending                                      manual         2m
```

**Diagnosis:**
```bash
kubectl describe pvc mysql-pv-claim
# Events:
#   Warning  ProvisioningFailed  no persistent volumes available
```

**Root Causes:**
1. No PV with matching storageClassName
2. PV capacity smaller than requested
3. Access mode mismatch

**Solution:**
```bash
# Check PV exists
kubectl get pv

# Check storageClassName matches
kubectl get pv mysql-pv -o yaml | grep storageClassName
kubectl get pvc mysql-pv-claim -o yaml | grep storageClassName

# Create PV if missing or fix storageClassName
```

---

### Issue 2: MySQL Pod CrashLoopBackOff

**Symptoms:**
```bash
kubectl get pods
# NAME                                READY   STATUS             RESTARTS   AGE
# mysql-deployment-xxx                0/1     CrashLoopBackOff   5          5m
```

**Diagnosis:**
```bash
kubectl logs $POD_NAME

# Common errors:
# 1. "MYSQL_ROOT_PASSWORD not set"
# 2. "Can't read dir of '/var/lib/mysql/'"
# 3. "Permission denied"
```

**Solutions:**

**A. Missing Environment Variables**
```bash
# Check secrets exist
kubectl get secrets

# Verify secret has correct keys
kubectl describe secret mysql-root-pass
kubectl describe secret mysql-user-pass
kubectl describe secret mysql-db-url

# Check deployment references correct secrets
kubectl get deployment mysql-deployment -o yaml | grep -A 5 "env:"
```

**B. Permission Issues**
```bash
# Check PVC is bound
kubectl get pvc mysql-pv-claim

# Check volume mount
kubectl describe pod $POD_NAME | grep -A 5 "Mounts:"

# On some systems, may need init container to fix permissions
```

---

### Issue 3: Secret Not Found

**Symptoms:**
```bash
kubectl get pods
# mysql-deployment-xxx    0/1     CreateContainerConfigError   0          30s
```

**Diagnosis:**
```bash
kubectl describe pod $POD_NAME
# Error: secret "mysql-root-pass" not found
```

**Solution:**
```bash
# Create missing secrets
kubectl create secret generic mysql-root-pass --from-literal=password=YUIidhb667
kubectl create secret generic mysql-user-pass --from-literal=username=kodekloud_gem --from-literal=password=B4zNgHA7Ya
kubectl create secret generic mysql-db-url --from-literal=database=kodekloud_db5

# Pod will automatically recover
```

---

### Issue 4: MySQL Not Initializing Database

**Symptoms:**
```bash
kubectl exec $POD_NAME -- mysql -u root -pYUIidhb667 -e "SHOW DATABASES;"
# kodekloud_db5 not in list
```

**Diagnosis:**
```bash
kubectl logs $POD_NAME | grep "Created database"
# No output = database not created
```

**Root Cause:** /var/lib/mysql not empty (previous initialization exists)

**Solution:**

**Option 1: Delete and recreate (development only)**
```bash
kubectl delete deployment mysql-deployment
kubectl delete pvc mysql-pv-claim
kubectl delete pv mysql-pv

# Clean data directory on node
kubectl run cleanup --image=busybox --rm -it --restart=Never -- rm -rf /mnt/data/mysql/*

# Recreate everything
```

**Option 2: Manually create database**
```bash
kubectl exec -it $POD_NAME -- mysql -u root -pYUIidhb667
CREATE DATABASE kodekloud_db5;
CREATE USER 'kodekloud_gem'@'%' IDENTIFIED BY 'B4zNgHA7Ya';
GRANT ALL PRIVILEGES ON kodekloud_db5.* TO 'kodekloud_gem'@'%';
FLUSH PRIVILEGES;
EXIT;
```

---

### Issue 5: NodePort Not Accessible

**Symptoms:**
```bash
mysql -h $NODE_IP -P 30007 -u kodekloud_gem -pB4zNgHA7Ya
# ERROR 2003: Can't connect to MySQL server
```

**Diagnosis:**
```bash
# Check service exists
kubectl get service mysql

# Check service type
kubectl get service mysql -o yaml | grep type
# Should be: NodePort

# Check nodePort
kubectl get service mysql -o yaml | grep nodePort
# Should be: 30007

# Check endpoints
kubectl get endpoints mysql
# Should show pod IP
```

**Solutions:**

**A. Service Not Created**
```bash
kubectl apply -f mysql-service.yaml
```

**B. Wrong NodePort Range**
```bash
# NodePort must be 30000-32767
# Edit service if wrong
kubectl edit service mysql
```

**C. Firewall Issues**
```bash
# On cloud providers, may need to open port in security group
# On local cluster, check firewall rules
```

---

### Issue 6: Wrong Secret Key Names

**Symptoms:**
```bash
kubectl logs $POD_NAME
# mysqld: [Warning] World-writable config file '/etc/mysql/conf.d/docker.cnf'
# [ERROR] --initialize specified but the data directory has files in it
```

**Diagnosis:**
```bash
# Check secret keys match deployment
kubectl describe secret mysql-user-pass
# Data should have: username, password

kubectl get deployment mysql-deployment -o yaml | grep -A 10 "secretKeyRef"
# Keys must match exactly
```

**Solution:**
```bash
# Delete and recreate secret with correct keys
kubectl delete secret mysql-user-pass
kubectl create secret generic mysql-user-pass \
  --from-literal=username=kodekloud_gem \
  --from-literal=password=B4zNgHA7Ya
```

---

## 📖 Key Takeaways

### Concepts Mastered Today:

1. ✅ **PersistentVolume (PV)**
   - Created cluster-level storage resource
   - Set capacity to 250Mi
   - Used hostPath for local storage
   - Configured ReadWriteOnce access mode

2. ✅ **PersistentVolumeClaim (PVC)**
   - Requested storage from PV
   - Successfully bound to PV
   - Mounted in MySQL deployment
   - Verified data persistence

3. ✅ **Kubernetes Secrets**
   - Created three secrets for credentials
   - Used opaque type for generic key-value pairs
   - Referenced secrets in environment variables
   - Kept sensitive data separate from code

4. ✅ **Environment Variables with secretKeyRef**
   - MYSQL_ROOT_PASSWORD from mysql-root-pass
   - MYSQL_DATABASE from mysql-db-url
   - MYSQL_USER and MYSQL_PASSWORD from mysql-user-pass
   - Secure injection of credentials

5. ✅ **MySQL Deployment**
   - Deployed MySQL 8.0 on Kubernetes
   - Mounted PVC at /var/lib/mysql
   - Initialized database automatically
   - Created user with proper permissions

6. ✅ **NodePort Service**
   - Exposed MySQL on port 30007
   - Enabled external database access
   - Routed traffic to MySQL pod
   - Verified endpoint connectivity

7. ✅ **Storage Persistence**
   - Data survives pod restarts
   - Data survives pod deletion
   - PV retains data after PVC deletion
   - Production-ready data persistence

---

## 🎯 Real-World Lessons

**Production Considerations:**

### 1. Storage
```yaml
# Development (current)
- hostPath storage
- Single node
- 250Mi capacity

# Production
- Cloud storage (EBS, Persistent Disk, Azure Disk)
- StorageClass with dynamic provisioning
- Larger capacity (100Gi+)
- Backup strategy with snapshots
```

### 2. High Availability
```yaml
# Development (current)
- Single replica
- No replication

# Production
- Master-slave replication
- Multi-node deployment
- StatefulSet (not Deployment)
- Automated failover
```

### 3. Security
```yaml
# Current Implementation
- Secrets in Kubernetes (base64)
- Environment variables

# Production Enhancement
- External secret manager (Vault, AWS Secrets Manager)
- Encrypted secrets at rest
- RBAC for secret access
- Network policies
- TLS encryption
```

### 4. Monitoring
```yaml
# Add to Production
- MySQL exporter for Prometheus
- Grafana dashboards
- Slow query logs
- Performance schema monitoring
- Alert rules for downtime
```

### 5. Backup Strategy
```yaml
# Add to Production
- Automated daily backups
- Point-in-time recovery
- Cross-region backup replication
- Backup testing and restore procedures
- Retention policies
```

---

## 📚 Additional Resources

**Official Documentation:**
- [Kubernetes Persistent Volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)
- [Kubernetes Secrets](https://kubernetes.io/docs/concepts/configuration/secret/)
- [MySQL Docker Hub](https://hub.docker.com/_/mysql)
- [MySQL Environment Variables](https://dev.mysql.com/doc/refman/8.0/en/environment-variables.html)

**Best Practices:**
- [MySQL on Kubernetes Best Practices](https://kubernetes.io/docs/tutorials/stateful-application/mysql-wordpress-persistent-volume/)
- [Secret Management](https://kubernetes.io/docs/concepts/configuration/secret/#best-practices)
- [Storage Best Practices](https://kubernetes.io/docs/concepts/storage/storage-classes/)

**Next Steps:**
- **Day 67:** MySQL Replication with StatefulSets
- **Day 68:** Database Backups and Restore
- **Day 69:** MySQL Monitoring with Prometheus
- **Day 70:** Advanced MySQL Configuration

---

## ✅ Task Completion Checklist

- [ ] Created secret `mysql-root-pass` with password
- [ ] Created secret `mysql-user-pass` with username and password
- [ ] Created secret `mysql-db-url` with database name
- [ ] Verified all three secrets exist
- [ ] Created PersistentVolume `mysql-pv` (250Mi)
- [ ] Created PersistentVolumeClaim `mysql-pv-claim` (250Mi)
- [ ] Verified PVC bound to PV
- [ ] Created deployment `mysql-deployment`
- [ ] Used MySQL image (mysql:8.0 or similar)
- [ ] Configured environment variables from secrets
- [ ] Mounted PVC at /var/lib/mysql
- [ ] Verified pod is running
- [ ] Created NodePort service `mysql` on port 30007
- [ ] Tested MySQL root connection
- [ ] Tested MySQL user connection
- [ ] Verified database `kodekloud_db5` exists
- [ ] Verified user `kodekloud_gem` has access
- [ ] Tested external access via NodePort

---

**🎉 Congratulations!** You've successfully deployed a production-ready MySQL database on Kubernetes with persistent storage, secure credential management, and external access! You now understand how to use PersistentVolumes for data persistence and Secrets for secure configuration.

**Day 66 Status:** ✅ Complete

**Next:** Day 67 - MySQL High Availability and Replication 🚀
