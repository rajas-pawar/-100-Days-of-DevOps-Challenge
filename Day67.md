# Day 67: Multi-Tier Guestbook Application - Redis Backend + PHP Frontend

## 📋 Task Overview

**Scenario:** The Nautilus Application development team has completed a guestbook application that manages entries for guests/visitors. The DevOps team has finalized the infrastructure design for deployment on the Kubernetes cluster using a multi-tier architecture.

**Application Architecture:**
```
┌─────────────────────────────────────────────────────────┐
│                    FRONT-END TIER                        │
│   PHP Frontend (3 replicas) → NodePort 30009           │
└──────────────────────┬──────────────────────────────────┘
                       │
                       ↓ Reads/Writes
┌─────────────────────────────────────────────────────────┐
│                    BACK-END TIER                         │
│  Redis Master (1 replica) ← Write operations           │
│  Redis Slave (2 replicas)  ← Read operations           │
└─────────────────────────────────────────────────────────┘
```

**Given Requirements:**

**BACK-END TIER:**
1. Redis Master deployment with 1 replica
2. Redis Master service
3. Redis Slave deployment with 2 replicas
4. Redis Slave service

**FRONT-END TIER:**
5. Frontend deployment with 3 replicas
6. Frontend NodePort service on port 30009

**Your Mission:**
1. Deploy Redis master for write operations
2. Deploy Redis slaves for read operations
3. Deploy PHP frontend application
4. Create services for all components
5. Verify guestbook application functionality

---

## 🎯 Learning Objectives

By the end of this task, you will understand:
- **Multi-Tier Architecture:** Frontend, backend separation
- **Redis Master-Slave:** Replication for scalability
- **Service Discovery:** DNS-based service communication
- **Resource Requests:** CPU and memory allocation
- **Horizontal Scaling:** Multiple replicas for availability
- **NodePort Services:** External application access

---

## 📖 Understanding Multi-Tier Architecture

### What is Multi-Tier Architecture?

**Definition:** Application divided into logical layers with specific responsibilities

**Typical Tiers:**
```
┌──────────────────────────────────────┐
│     Presentation Tier (Frontend)      │
│   - User interface                    │
│   - PHP, React, Angular              │
│   - Handles user requests            │
└───────────────┬──────────────────────┘
                │
                ↓
┌──────────────────────────────────────┐
│     Application Tier (Backend)        │
│   - Business logic                    │
│   - APIs, processing                 │
│   - Data validation                  │
└───────────────┬──────────────────────┘
                │
                ↓
┌──────────────────────────────────────┐
│     Data Tier (Database/Cache)        │
│   - Data storage                      │
│   - Redis, MySQL, PostgreSQL         │
│   - Data persistence                 │
└──────────────────────────────────────┘
```

**Benefits:**
- ✅ Separation of concerns
- ✅ Independent scaling
- ✅ Easier maintenance
- ✅ Better security
- ✅ Technology flexibility

---

## 📖 Understanding Redis Master-Slave Replication

### What is Redis Replication?

**Purpose:** Create copies of data for read scalability and redundancy

**Architecture:**
```
                  ┌─────────────────┐
                  │  Redis Master   │
                  │  (Write Only)   │
                  └────────┬────────┘
                           │
            ┌──────────────┼──────────────┐
            │ Replication  │ Replication  │
            ↓              ↓              ↓
    ┌──────────────┐ ┌──────────────┐ ┌──────────────┐
    │ Redis Slave  │ │ Redis Slave  │ │ Redis Slave  │
    │ (Read Only)  │ │ (Read Only)  │ │ (Read Only)  │
    └──────────────┘ └──────────────┘ └──────────────┘
```

### How It Works

1. **Master:** Accepts all write operations
2. **Replication:** Master replicates data to slaves
3. **Slaves:** Handle read operations (load distribution)
4. **Consistency:** Slaves stay synchronized with master

**Traffic Flow:**
```
Write Request → Redis Master → Data written
                    ↓
            Replicate to Slaves
                    ↓
Read Request → Redis Slave → Data returned
```

**Benefits:**
- ✅ Read scalability (distribute reads across slaves)
- ✅ High availability (failover capability)
- ✅ Load distribution (master handles writes, slaves handle reads)
- ✅ Data redundancy (multiple copies)

---

## 📖 Understanding Service Discovery with DNS

### Kubernetes DNS

**What it is:** Built-in DNS server for service discovery

**DNS Format:**
```
<service-name>.<namespace>.svc.cluster.local

Examples:
- redis-master.default.svc.cluster.local
- redis-slave.default.svc.cluster.local
- frontend.default.svc.cluster.local

Short form (same namespace):
- redis-master
- redis-slave
- frontend
```

### Environment Variable: GET_HOSTS_FROM=dns

**Purpose:** Tell application how to discover services

**Options:**
```yaml
GET_HOSTS_FROM=dns     # Use Kubernetes DNS (recommended)
GET_HOSTS_FROM=env     # Use environment variables (legacy)
```

**How Frontend Uses It:**
```php
// PHP Frontend Code
$host = getenv('GET_HOSTS_FROM');

if ($host == 'dns') {
    // Use DNS to find Redis
    $redis_master = 'redis-master';  // DNS name
    $redis_slave = 'redis-slave';    // DNS name
} else {
    // Use environment variables
    $redis_master = getenv('REDIS_MASTER_SERVICE_HOST');
}
```

---

## 📖 Understanding Resource Requests

### CPU and Memory Requests

**Purpose:** Guarantee minimum resources for containers

**Our Configuration:**
```yaml
resources:
  requests:
    cpu: 100m      # 100 millicores = 0.1 CPU
    memory: 100Mi  # 100 Mebibytes
```

**What This Means:**
- **CPU:** 10% of one CPU core guaranteed
- **Memory:** 100Mi guaranteed RAM
- **Scheduler:** Ensures node has resources before placing pod

**Resource Units:**
```
CPU:
- 1 or 1000m = 1 full CPU core
- 500m = 0.5 CPU core
- 100m = 0.1 CPU core (10%)

Memory:
- 100Mi = 100 Mebibytes (104.857 MB)
- 1Gi = 1 Gibibyte (1.074 GB)
```

---

## 🛠️ Task Implementation

### Step 1: Verify Cluster Access

**Check kubectl configuration:**
```bash
kubectl cluster-info
```

**Check current namespace:**
```bash
kubectl config view --minify | grep namespace
```

**List existing resources:**
```bash
kubectl get all
```

---

### Step 2: Create Redis Master Deployment

**Create deployment YAML:**
```bash
cat > redis-master-deployment.yaml <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-master
  labels:
    app: redis
    tier: backend
    role: master
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
      tier: backend
      role: master
  template:
    metadata:
      labels:
        app: redis
        tier: backend
        role: master
    spec:
      containers:
      - name: master-redis-devops
        image: redis
        ports:
        - containerPort: 6379
        resources:
          requests:
            cpu: 100m
            memory: 100Mi
EOF
```

**Understanding the Configuration:**

```yaml
metadata:
  name: redis-master  # Deployment name
  labels:
    app: redis        # Application identifier
    tier: backend     # Backend tier
    role: master      # Master role

spec:
  replicas: 1  # Single master instance

  selector:
    matchLabels:
      app: redis
      tier: backend
      role: master  # Deployment manages pods with these labels

  template:
    metadata:
      labels:  # Pod labels (must match selector)
        app: redis
        tier: backend
        role: master

    spec:
      containers:
      - name: master-redis-devops  # Container name as required
        image: redis                # Official Redis image
        ports:
        - containerPort: 6379       # Redis default port
        resources:
          requests:
            cpu: 100m      # 0.1 CPU core
            memory: 100Mi  # 100 Mebibytes
```

**Apply the deployment:**
```bash
kubectl apply -f redis-master-deployment.yaml
```

**Expected output:**
```
deployment.apps/redis-master created
```

---

### Step 3: Create Redis Master Service

**Create service YAML:**
```bash
cat > redis-master-service.yaml <<EOF
apiVersion: v1
kind: Service
metadata:
  name: redis-master
  labels:
    app: redis
    tier: backend
    role: master
spec:
  selector:
    app: redis
    tier: backend
    role: master
  ports:
  - port: 6379
    targetPort: 6379
  type: ClusterIP
EOF
```

**Understanding the Configuration:**

```yaml
metadata:
  name: redis-master  # Service DNS name

spec:
  selector:
    app: redis
    tier: backend
    role: master  # Routes traffic to pods with these labels

  ports:
  - port: 6379         # Service port (what clients connect to)
    targetPort: 6379   # Container port (Redis listens on)

  type: ClusterIP  # Internal service (not exposed externally)
```

**Apply the service:**
```bash
kubectl apply -f redis-master-service.yaml
```

**Expected output:**
```
service/redis-master created
```

---

### Step 4: Verify Redis Master

**Check deployment:**
```bash
kubectl get deployment redis-master
```

**Expected output:**
```
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
redis-master   1/1     1            1           30s
```

**Check pods:**
```bash
kubectl get pods -l role=master
```

**Expected output:**
```
NAME                            READY   STATUS    RESTARTS   AGE
redis-master-7c8f9d5b4d-xyz12   1/1     Running   0          40s
```

**Check service:**
```bash
kubectl get service redis-master
```

**Expected output:**
```
NAME           TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
redis-master   ClusterIP   10.96.123.45    <none>        6379/TCP   50s
```

**Test Redis Master:**
```bash
POD_NAME=$(kubectl get pods -l role=master -o jsonpath='{.items[0].metadata.name}')
kubectl exec -it $POD_NAME -- redis-cli ping
```

**Expected output:**
```
PONG
```

---

### Step 5: Create Redis Slave Deployment

**Create deployment YAML:**
```bash
cat > redis-slave-deployment.yaml <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-slave
  labels:
    app: redis
    tier: backend
    role: slave
spec:
  replicas: 2
  selector:
    matchLabels:
      app: redis
      tier: backend
      role: slave
  template:
    metadata:
      labels:
        app: redis
        tier: backend
        role: slave
    spec:
      containers:
      - name: slave-redis-devops
        image: gcr.io/google_samples/gb-redisslave:v3
        ports:
        - containerPort: 6379
        env:
        - name: GET_HOSTS_FROM
          value: dns
        resources:
          requests:
            cpu: 100m
            memory: 100Mi
EOF
```

**Understanding the Configuration:**

```yaml
spec:
  replicas: 2  # Two slave replicas for read scaling

  containers:
  - name: slave-redis-devops  # Container name as required
    image: gcr.io/google_samples/gb-redisslave:v3  # Redis slave image
    
    ports:
    - containerPort: 6379  # Redis default port
    
    env:
    - name: GET_HOSTS_FROM
      value: dns  # Use DNS to discover Redis master
    
    resources:
      requests:
        cpu: 100m
        memory: 100Mi
```

**How Slave Finds Master:**
The slave container uses `GET_HOSTS_FROM=dns` to discover the Redis master:
1. Slave starts and reads `GET_HOSTS_FROM=dns`
2. Slave queries Kubernetes DNS for `redis-master`
3. DNS returns master service IP
4. Slave connects to master and begins replication

**Apply the deployment:**
```bash
kubectl apply -f redis-slave-deployment.yaml
```

**Expected output:**
```
deployment.apps/redis-slave created
```

---

### Step 6: Create Redis Slave Service

**Create service YAML:**
```bash
cat > redis-slave-service.yaml <<EOF
apiVersion: v1
kind: Service
metadata:
  name: redis-slave
  labels:
    app: redis
    tier: backend
    role: slave
spec:
  selector:
    app: redis
    tier: backend
    role: slave
  ports:
  - port: 6379
    targetPort: 6379
  type: ClusterIP
EOF
```

**Apply the service:**
```bash
kubectl apply -f redis-slave-service.yaml
```

**Expected output:**
```
service/redis-slave created
```

---

### Step 7: Verify Redis Slave

**Check deployment:**
```bash
kubectl get deployment redis-slave
```

**Expected output:**
```
NAME          READY   UP-TO-DATE   AVAILABLE   AGE
redis-slave   2/2     2            2           30s
```

**Check pods:**
```bash
kubectl get pods -l role=slave
```

**Expected output:**
```
NAME                           READY   STATUS    RESTARTS   AGE
redis-slave-6b9f8c5d7e-abc12   1/1     Running   0          40s
redis-slave-6b9f8c5d7e-def34   1/1     Running   0          40s
```

**Check service:**
```bash
kubectl get service redis-slave
```

**Expected output:**
```
NAME          TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
redis-slave   ClusterIP   10.96.234.56    <none>        6379/TCP   50s
```

**Test Redis Slave:**
```bash
SLAVE_POD=$(kubectl get pods -l role=slave -o jsonpath='{.items[0].metadata.name}')
kubectl exec -it $SLAVE_POD -- redis-cli ping
```

**Expected output:**
```
PONG
```

---

### Step 8: Verify Redis Replication

**Check master replication info:**
```bash
kubectl exec $POD_NAME -- redis-cli info replication
```

**Expected output:**
```
# Replication
role:master
connected_slaves:2  ✅
slave0:ip=10.244.1.5,port=6379,state=online,offset=123,lag=0
slave1:ip=10.244.1.6,port=6379,state=online,offset=123,lag=0
```

**Check slave replication status:**
```bash
kubectl exec $SLAVE_POD -- redis-cli info replication
```

**Expected output:**
```
# Replication
role:slave  ✅
master_host:redis-master  ✅
master_port:6379
master_link_status:up  ✅
```

**Test replication:**
```bash
# Write to master
kubectl exec $POD_NAME -- redis-cli SET test_key "Hello from Master"

# Read from slave
kubectl exec $SLAVE_POD -- redis-cli GET test_key
```

**Expected output:**
```
Hello from Master  ✅
```

**Replication working! ✅**

---

### Step 9: Create Frontend Deployment

**Create deployment YAML:**
```bash
cat > frontend-deployment.yaml <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  labels:
    app: guestbook
    tier: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: guestbook
      tier: frontend
  template:
    metadata:
      labels:
        app: guestbook
        tier: frontend
    spec:
      containers:
      - name: php-redis-devops
        image: gcr.io/google-samples/gb-frontend@sha256:a908df8486ff66f2c4daa0d3d8a2fa09846a1fc8efd65649c0109695c7c5cbff
        ports:
        - containerPort: 80
        env:
        - name: GET_HOSTS_FROM
          value: dns
        resources:
          requests:
            cpu: 100m
            memory: 100Mi
EOF
```

**Understanding the Configuration:**

```yaml
metadata:
  name: frontend
  labels:
    app: guestbook
    tier: frontend

spec:
  replicas: 3  # Three frontend replicas for availability

  containers:
  - name: php-redis-devops  # Container name as required
    image: gcr.io/google-samples/gb-frontend@sha256:a908df8486ff66f2c4daa0d3d8a2fa09846a1fc8efd65649c0109695c7c5cbff
    # ↑ Specific image SHA for consistency
    
    ports:
    - containerPort: 80  # PHP application port
    
    env:
    - name: GET_HOSTS_FROM
      value: dns  # Use DNS to discover Redis services
    
    resources:
      requests:
        cpu: 100m
        memory: 100Mi
```

**How Frontend Uses Redis:**
```php
// Frontend PHP code logic
if ($_GET['cmd'] == 'set') {
    // WRITE operation → Redis Master
    $redis_master = new Redis();
    $redis_master->connect('redis-master', 6379);
    $redis_master->set('guestbook', $value);
} else {
    // READ operation → Redis Slave
    $redis_slave = new Redis();
    $redis_slave->connect('redis-slave', 6379);
    $value = $redis_slave->get('guestbook');
}
```

**Apply the deployment:**
```bash
kubectl apply -f frontend-deployment.yaml
```

**Expected output:**
```
deployment.apps/frontend created
```

---

### Step 10: Create Frontend Service (NodePort)

**Create service YAML:**
```bash
cat > frontend-service.yaml <<EOF
apiVersion: v1
kind: Service
metadata:
  name: frontend
  labels:
    app: guestbook
    tier: frontend
spec:
  type: NodePort
  selector:
    app: guestbook
    tier: frontend
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30009
EOF
```

**Understanding the Configuration:**

```yaml
metadata:
  name: frontend  # Service name

spec:
  type: NodePort  # Expose externally
  
  selector:
    app: guestbook
    tier: frontend  # Routes to frontend pods
  
  ports:
  - port: 80          # Service port (internal)
    targetPort: 80    # Container port (PHP app)
    nodePort: 30009   # External port (as required)
```

**Traffic Flow:**
```
External Browser → Node IP:30009 → Service:80 → Frontend Pod:80
                                                      ↓
                                                   PHP App
                                                      ↓
                                        ┌─────────────┴─────────────┐
                                        ↓                           ↓
                                   Write Request              Read Request
                                        ↓                           ↓
                               redis-master:6379          redis-slave:6379
```

**Apply the service:**
```bash
kubectl apply -f frontend-service.yaml
```

**Expected output:**
```
service/frontend created
```

---

### Step 11: Verify Frontend Deployment

**Check deployment:**
```bash
kubectl get deployment frontend
```

**Expected output:**
```
NAME       READY   UP-TO-DATE   AVAILABLE   AGE
frontend   3/3     3            3           30s
```

**Check pods:**
```bash
kubectl get pods -l tier=frontend
```

**Expected output:**
```
NAME                        READY   STATUS    RESTARTS   AGE
frontend-7d8f9c5b4d-abc12   1/1     Running   0          40s
frontend-7d8f9c5b4d-def34   1/1     Running   0          40s
frontend-7d8f9c5b4d-ghi56   1/1     Running   0          40s
```

**Check service:**
```bash
kubectl get service frontend
```

**Expected output:**
```
NAME       TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
frontend   NodePort   10.96.345.67    <none>        80:30009/TCP   50s
```

**Check endpoints:**
```bash
kubectl get endpoints frontend
```

**Expected output:**
```
NAME       ENDPOINTS                                         AGE
frontend   10.244.1.7:80,10.244.1.8:80,10.244.1.9:80         1m
```

**Three endpoints (one per replica) ✅**

---

### Step 12: View Complete Architecture

**List all resources:**
```bash
kubectl get all
```

**Expected output:**
```
NAME                                READY   STATUS    RESTARTS   AGE
pod/redis-master-xxx                1/1     Running   0          5m
pod/redis-slave-xxx                 1/1     Running   0          4m
pod/redis-slave-yyy                 1/1     Running   0          4m
pod/frontend-xxx                    1/1     Running   0          3m
pod/frontend-yyy                    1/1     Running   0          3m
pod/frontend-zzz                    1/1     Running   0          3m

NAME                   TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
service/kubernetes     ClusterIP   10.96.0.1       <none>        443/TCP        10d
service/redis-master   ClusterIP   10.96.123.45    <none>        6379/TCP       5m
service/redis-slave    ClusterIP   10.96.234.56    <none>        6379/TCP       4m
service/frontend       NodePort    10.96.345.67    <none>        80:30009/TCP   3m

NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/redis-master   1/1     1            1           5m
deployment.apps/redis-slave    2/2     2            2           4m
deployment.apps/frontend       3/3     3            3           3m
```

**Total Pods: 6**
- 1 Redis Master
- 2 Redis Slaves
- 3 Frontend replicas

---

### Step 13: Test Guestbook Application

**Get node IP:**
```bash
kubectl get nodes -o wide
```

**Expected output:**
```
NAME     STATUS   ROLES    AGE   VERSION   INTERNAL-IP    EXTERNAL-IP
node01   Ready    <none>   10d   v1.27.0   172.17.0.2     <none>
```

**Access application:**
```bash
NODE_IP=$(kubectl get nodes -o jsonpath='{.items[0].status.addresses[?(@.type=="InternalIP")].address}')
echo "Guestbook URL: http://$NODE_IP:30009"
```

**Example output:**
```
Guestbook URL: http://172.17.0.2:30009
```

**Test with curl:**
```bash
curl http://$NODE_IP:30009
```

**Expected output (HTML):**
```html
<!DOCTYPE html>
<html>
<head>
<title>Guestbook</title>
<style>
...
</style>
</head>
<body>
<div id="header">
  <h1>Guestbook</h1>
</div>
<div id="guestbook-form">
  <form action="/index.php" method="POST">
    <input type="text" name="message" />
    <input type="submit" value="Submit" />
  </form>
</div>
<div id="guestbook-entries">
  <!-- Entries will appear here -->
</div>
</body>
</html>
```

**Test write operation:**
```bash
curl -X POST http://$NODE_IP:30009/index.php \
  --data "cmd=set&key=messages&value=Hello+Kubernetes"
```

**Test read operation:**
```bash
curl http://$NODE_IP:30009/index.php?cmd=get
```

---

### Step 14: Test Service Discovery

**Test DNS resolution from frontend pod:**
```bash
FRONTEND_POD=$(kubectl get pods -l tier=frontend -o jsonpath='{.items[0].metadata.name}')

# Test Redis Master DNS
kubectl exec $FRONTEND_POD -- nslookup redis-master

# Test Redis Slave DNS
kubectl exec $FRONTEND_POD -- nslookup redis-slave
```

**Expected output:**
```
Server:    10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

Name:      redis-master
Address 1: 10.96.123.45 redis-master.default.svc.cluster.local  ✅

Name:      redis-slave
Address 1: 10.96.234.56 redis-slave.default.svc.cluster.local  ✅
```

**Test connectivity:**
```bash
# Test Redis Master from frontend
kubectl exec $FRONTEND_POD -- nc -zv redis-master 6379

# Test Redis Slave from frontend
kubectl exec $FRONTEND_POD -- nc -zv redis-slave 6379
```

**Expected output:**
```
redis-master (10.96.123.45:6379) open  ✅
redis-slave (10.96.234.56:6379) open   ✅
```

---

### Step 15: Verify Resource Allocation

**Check resource requests:**
```bash
kubectl describe deployment redis-master | grep -A 5 "Requests:"
kubectl describe deployment redis-slave | grep -A 5 "Requests:"
kubectl describe deployment frontend | grep -A 5 "Requests:"
```

**Expected output (each deployment):**
```
Requests:
  cpu:        100m
  memory:     100Mi
```

**Check node resource usage:**
```bash
kubectl top nodes
```

**Expected output:**
```
NAME     CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%
node01   850m         42%    1250Mi          31%
```

**Total resource requests:**
- Redis Master: 100m CPU, 100Mi Memory
- Redis Slaves: 200m CPU, 200Mi Memory (2 replicas)
- Frontend: 300m CPU, 300Mi Memory (3 replicas)
- **Total: 600m CPU (0.6 cores), 600Mi Memory**

---

## 📊 Complete Setup Script

**Save and run this script for automated deployment:**

```bash
#!/bin/bash

echo "🚀 Day 67: Multi-Tier Guestbook Application Deployment"
echo "========================================================"

# Colors
GREEN='\033[0;32m'
RED='\033[0;31m'
YELLOW='\033[1;33m'
NC='\033[0m'

# Step 1: Deploy Redis Master
echo -e "\n${YELLOW}1️⃣ Deploying Redis Master...${NC}"

cat <<EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-master
  labels:
    app: redis
    tier: backend
    role: master
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
      tier: backend
      role: master
  template:
    metadata:
      labels:
        app: redis
        tier: backend
        role: master
    spec:
      containers:
      - name: master-redis-devops
        image: redis
        ports:
        - containerPort: 6379
        resources:
          requests:
            cpu: 100m
            memory: 100Mi
EOF

if [ $? -eq 0 ]; then
    echo -e "${GREEN}✅ Redis Master deployment created${NC}"
else
    echo -e "${RED}❌ Failed to create Redis Master deployment${NC}"
    exit 1
fi

# Step 2: Create Redis Master Service
echo -e "\n${YELLOW}2️⃣ Creating Redis Master Service...${NC}"

cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
  name: redis-master
  labels:
    app: redis
    tier: backend
    role: master
spec:
  selector:
    app: redis
    tier: backend
    role: master
  ports:
  - port: 6379
    targetPort: 6379
  type: ClusterIP
EOF

if [ $? -eq 0 ]; then
    echo -e "${GREEN}✅ Redis Master service created${NC}"
else
    echo -e "${RED}❌ Failed to create Redis Master service${NC}"
    exit 1
fi

# Wait for Redis Master
echo -e "\n${YELLOW}Waiting for Redis Master to be ready...${NC}"
kubectl rollout status deployment/redis-master --timeout=120s

# Step 3: Deploy Redis Slave
echo -e "\n${YELLOW}3️⃣ Deploying Redis Slave...${NC}"

cat <<EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-slave
  labels:
    app: redis
    tier: backend
    role: slave
spec:
  replicas: 2
  selector:
    matchLabels:
      app: redis
      tier: backend
      role: slave
  template:
    metadata:
      labels:
        app: redis
        tier: backend
        role: slave
    spec:
      containers:
      - name: slave-redis-devops
        image: gcr.io/google_samples/gb-redisslave:v3
        ports:
        - containerPort: 6379
        env:
        - name: GET_HOSTS_FROM
          value: dns
        resources:
          requests:
            cpu: 100m
            memory: 100Mi
EOF

if [ $? -eq 0 ]; then
    echo -e "${GREEN}✅ Redis Slave deployment created${NC}"
else
    echo -e "${RED}❌ Failed to create Redis Slave deployment${NC}"
    exit 1
fi

# Step 4: Create Redis Slave Service
echo -e "\n${YELLOW}4️⃣ Creating Redis Slave Service...${NC}"

cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
  name: redis-slave
  labels:
    app: redis
    tier: backend
    role: slave
spec:
  selector:
    app: redis
    tier: backend
    role: slave
  ports:
  - port: 6379
    targetPort: 6379
  type: ClusterIP
EOF

if [ $? -eq 0 ]; then
    echo -e "${GREEN}✅ Redis Slave service created${NC}"
else
    echo -e "${RED}❌ Failed to create Redis Slave service${NC}"
    exit 1
fi

# Wait for Redis Slaves
echo -e "\n${YELLOW}Waiting for Redis Slaves to be ready...${NC}"
kubectl rollout status deployment/redis-slave --timeout=120s

# Step 5: Verify Redis Replication
echo -e "\n${YELLOW}5️⃣ Verifying Redis Replication...${NC}"
sleep 5

MASTER_POD=$(kubectl get pods -l role=master -o jsonpath='{.items[0].metadata.name}')
SLAVE_POD=$(kubectl get pods -l role=slave -o jsonpath='{.items[0].metadata.name}')

echo -e "${YELLOW}Redis Master Pod: ${MASTER_POD}${NC}"
echo -e "${YELLOW}Redis Slave Pod: ${SLAVE_POD}${NC}"

# Check replication
echo -e "\n${YELLOW}Checking replication status...${NC}"
kubectl exec $MASTER_POD -- redis-cli info replication | grep -E "role|connected_slaves"

# Step 6: Deploy Frontend
echo -e "\n${YELLOW}6️⃣ Deploying Frontend...${NC}"

cat <<EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  labels:
    app: guestbook
    tier: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: guestbook
      tier: frontend
  template:
    metadata:
      labels:
        app: guestbook
        tier: frontend
    spec:
      containers:
      - name: php-redis-devops
        image: gcr.io/google-samples/gb-frontend@sha256:a908df8486ff66f2c4daa0d3d8a2fa09846a1fc8efd65649c0109695c7c5cbff
        ports:
        - containerPort: 80
        env:
        - name: GET_HOSTS_FROM
          value: dns
        resources:
          requests:
            cpu: 100m
            memory: 100Mi
EOF

if [ $? -eq 0 ]; then
    echo -e "${GREEN}✅ Frontend deployment created${NC}"
else
    echo -e "${RED}❌ Failed to create Frontend deployment${NC}"
    exit 1
fi

# Step 7: Create Frontend Service
echo -e "\n${YELLOW}7️⃣ Creating Frontend Service...${NC}"

cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
  name: frontend
  labels:
    app: guestbook
    tier: frontend
spec:
  type: NodePort
  selector:
    app: guestbook
    tier: frontend
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30009
EOF

if [ $? -eq 0 ]; then
    echo -e "${GREEN}✅ Frontend service created${NC}"
else
    echo -e "${RED}❌ Failed to create Frontend service${NC}"
    exit 1
fi

# Wait for Frontend
echo -e "\n${YELLOW}Waiting for Frontend to be ready...${NC}"
kubectl rollout status deployment/frontend --timeout=120s

# Step 8: Verification
echo -e "\n${YELLOW}8️⃣ Running Verification Checks...${NC}"

# Check all deployments
echo -e "\n${YELLOW}Deployments:${NC}"
kubectl get deployment

# Check all pods
echo -e "\n${YELLOW}Pods:${NC}"
kubectl get pods

# Check all services
echo -e "\n${YELLOW}Services:${NC}"
kubectl get service

# Get Node IP
NODE_IP=$(kubectl get nodes -o jsonpath='{.items[0].status.addresses[?(@.type=="InternalIP")].address}')

# Test application
echo -e "\n${YELLOW}9️⃣ Testing Guestbook Application...${NC}"
echo -e "${YELLOW}Testing HTTP connection...${NC}"

HTTP_CODE=$(curl -s -o /dev/null -w "%{http_code}" http://$NODE_IP:30009 2>/dev/null)

if [ "$HTTP_CODE" == "200" ]; then
    echo -e "${GREEN}✅ Guestbook accessible! HTTP $HTTP_CODE${NC}"
else
    echo -e "${RED}❌ Guestbook not accessible (HTTP $HTTP_CODE)${NC}"
fi

# Final summary
echo -e "\n${YELLOW}========================================================"
echo "📊 Deployment Summary"
echo "========================================================${NC}"

echo -e "\n${GREEN}✅ BACK-END TIER${NC}"
echo -e "${GREEN}  • Redis Master: 1 replica (write operations)${NC}"
echo -e "${GREEN}  • Redis Slave: 2 replicas (read operations)${NC}"
echo -e "${GREEN}  • Replication: Active${NC}"

echo -e "\n${GREEN}✅ FRONT-END TIER${NC}"
echo -e "${GREEN}  • Frontend: 3 replicas${NC}"
echo -e "${GREEN}  • Service Type: NodePort${NC}"
echo -e "${GREEN}  • NodePort: 30009${NC}"

echo -e "\n${GREEN}✅ RESOURCE ALLOCATION${NC}"
echo -e "${GREEN}  • Total Pods: 6${NC}"
echo -e "${GREEN}  • Total CPU Requests: 600m (0.6 cores)${NC}"
echo -e "${GREEN}  • Total Memory Requests: 600Mi${NC}"

echo -e "\n${YELLOW}🌐 Application Access:${NC}"
echo -e "${GREEN}URL: http://${NODE_IP}:30009${NC}"
echo -e "${YELLOW}Test with: curl http://${NODE_IP}:30009${NC}"

echo -e "\n${GREEN}✅ Guestbook application deployed successfully!${NC}"
```

**Save and run:**
```bash
chmod +x guestbook-deploy.sh
./guestbook-deploy.sh
```

---

## 🧹 Cleanup

**Delete all resources:**
```bash
# Delete frontend
kubectl delete deployment frontend
kubectl delete service frontend

# Delete Redis slaves
kubectl delete deployment redis-slave
kubectl delete service redis-slave

# Delete Redis master
kubectl delete deployment redis-master
kubectl delete service redis-master

# Verify deletion
kubectl get all
# Only default Kubernetes service should remain

# Clean up files
rm -f redis-master-deployment.yaml redis-master-service.yaml \
      redis-slave-deployment.yaml redis-slave-service.yaml \
      frontend-deployment.yaml frontend-service.yaml \
      guestbook-deploy.sh
```

---

## 🐛 Common Issues and Solutions

### Issue 1: Redis Slave Can't Connect to Master

**Symptoms:**
```bash
kubectl logs $SLAVE_POD
# Error connecting to MASTER
# Connection refused
```

**Diagnosis:**
```bash
# Check if master service exists
kubectl get service redis-master

# Check if master is running
kubectl get pods -l role=master

# Check DNS from slave pod
kubectl exec $SLAVE_POD -- nslookup redis-master
```

**Solution:**
```bash
# Ensure master is deployed first
kubectl rollout status deployment/redis-master

# Ensure service exists
kubectl get service redis-master

# Check GET_HOSTS_FROM environment variable
kubectl get deployment redis-slave -o yaml | grep -A 2 "GET_HOSTS_FROM"
# Should be: value: dns
```

---

### Issue 2: Frontend Can't Connect to Redis

**Symptoms:**
```bash
curl http://$NODE_IP:30009
# Error: Could not connect to Redis
```

**Diagnosis:**
```bash
# Check frontend logs
FRONTEND_POD=$(kubectl get pods -l tier=frontend -o jsonpath='{.items[0].metadata.name}')
kubectl logs $FRONTEND_POD

# Test DNS from frontend
kubectl exec $FRONTEND_POD -- nslookup redis-master
kubectl exec $FRONTEND_POD -- nslookup redis-slave
```

**Solution:**
```bash
# Ensure GET_HOSTS_FROM=dns is set
kubectl get deployment frontend -o yaml | grep -A 2 "GET_HOSTS_FROM"

# Ensure Redis services exist
kubectl get service redis-master redis-slave

# Restart frontend if needed
kubectl rollout restart deployment/frontend
```

---

### Issue 3: NodePort Not Accessible

**Symptoms:**
```bash
curl http://$NODE_IP:30009
# Connection refused or timeout
```

**Diagnosis:**
```bash
# Check service type
kubectl get service frontend -o yaml | grep type
# Should be: NodePort

# Check nodePort
kubectl get service frontend -o yaml | grep nodePort
# Should be: 30009

# Check endpoints
kubectl get endpoints frontend
# Should show 3 pod IPs
```

**Solution:**
```bash
# Verify service configuration
kubectl describe service frontend

# Test from inside cluster first
kubectl run test --image=busybox --rm -it --restart=Never -- wget -O- http://frontend

# Check firewall rules (cloud providers)
# May need to open port 30009 in security group
```

---

### Issue 4: Pods Stuck in Pending

**Symptoms:**
```bash
kubectl get pods
# Multiple pods in Pending state
```

**Diagnosis:**
```bash
kubectl describe pod <pod-name>
# Events: Insufficient cpu or Insufficient memory
```

**Root Cause:** Not enough resources on nodes

**Solution:**
```bash
# Check node resources
kubectl describe nodes | grep -A 5 "Allocated resources"

# Option 1: Reduce resource requests
# Edit deployments to request less (e.g., 50m CPU, 50Mi memory)

# Option 2: Add more nodes (multi-node cluster)

# Option 3: Delete other workloads to free resources
```

---

### Issue 5: Image Pull Errors

**Symptoms:**
```bash
kubectl get pods
# Pods in ImagePullBackOff or ErrImagePull
```

**Diagnosis:**
```bash
kubectl describe pod <pod-name>
# Events: Failed to pull image
```

**Solution:**
```bash
# For Redis slave
kubectl set image deployment/redis-slave \
  slave-redis-devops=gcr.io/google_samples/gb-redisslave:v3

# For frontend (check SHA256)
kubectl set image deployment/frontend \
  php-redis-devops=gcr.io/google-samples/gb-frontend@sha256:a908df8486ff66f2c4daa0d3d8a2fa09846a1fc8efd65649c0109695c7c5cbff

# Or use tag instead of SHA
kubectl set image deployment/frontend \
  php-redis-devops=gcr.io/google-samples/gb-frontend:v5
```

---

## 📖 Key Takeaways

### Concepts Mastered Today:

1. ✅ **Multi-Tier Architecture**
   - Frontend tier (PHP application)
   - Backend tier (Redis database)
   - Separation of concerns
   - Independent scaling

2. ✅ **Redis Master-Slave Replication**
   - Master handles writes (1 replica)
   - Slaves handle reads (2 replicas)
   - Automatic replication
   - Load distribution

3. ✅ **Service Discovery with DNS**
   - GET_HOSTS_FROM=dns
   - Services discoverable by name
   - redis-master, redis-slave, frontend
   - No hardcoded IPs

4. ✅ **Resource Management**
   - CPU requests: 100m per container
   - Memory requests: 100Mi per container
   - Total: 600m CPU, 600Mi memory
   - Proper resource allocation

5. ✅ **Horizontal Scaling**
   - Redis Master: 1 replica (single writer)
   - Redis Slave: 2 replicas (read scaling)
   - Frontend: 3 replicas (availability)
   - Total: 6 pods

6. ✅ **NodePort Service**
   - External access to application
   - Port 30009 exposed on all nodes
   - Load balanced across frontend replicas

7. ✅ **Labels and Selectors**
   - app: redis / guestbook
   - tier: backend / frontend
   - role: master / slave
   - Organized resource management

---

## 🎯 Real-World Lessons

**Production Architecture Comparison:**

### Current Implementation (Development)
```yaml
BACK-END:
- Redis Master: 1 replica (no HA)
- Redis Slave: 2 replicas
- No persistence (data lost on restart)

FRONT-END:
- Frontend: 3 replicas
- NodePort service (30009)
```

### Production Implementation
```yaml
BACK-END:
- Redis Sentinel (3+ nodes) for failover
- Redis Cluster (6+ nodes) for sharding
- PersistentVolume for data persistence
- Automated backups
- Monitoring and alerts

FRONT-END:
- Frontend: 5+ replicas
- LoadBalancer service (not NodePort)
- Ingress controller with SSL/TLS
- CDN for static assets
- Horizontal Pod Autoscaler (HPA)

OBSERVABILITY:
- Prometheus monitoring
- Grafana dashboards
- Application logs aggregation
- Distributed tracing
```

**Scaling Strategies:**

1. **Read-Heavy Workload:**
   ```bash
   # Scale Redis slaves for more read capacity
   kubectl scale deployment redis-slave --replicas=5
   ```

2. **High Traffic:**
   ```bash
   # Scale frontend for more concurrent users
   kubectl scale deployment frontend --replicas=10
   ```

3. **Auto-Scaling:**
   ```yaml
   # Add Horizontal Pod Autoscaler
   apiVersion: autoscaling/v2
   kind: HorizontalPodAutoscaler
   metadata:
     name: frontend-hpa
   spec:
     scaleTargetRef:
       apiVersion: apps/v1
       kind: Deployment
       name: frontend
     minReplicas: 3
     maxReplicas: 10
     metrics:
     - type: Resource
       resource:
         name: cpu
         target:
           type: Utilization
           averageUtilization: 70
   ```

---

## 📚 Additional Resources

**Official Documentation:**
- [Kubernetes Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
- [Services](https://kubernetes.io/docs/concepts/services-networking/service/)
- [DNS for Services](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/)
- [Resource Management](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/)

**Redis Documentation:**
- [Redis Replication](https://redis.io/docs/management/replication/)
- [Redis Sentinel](https://redis.io/docs/management/sentinel/)
- [Redis Cluster](https://redis.io/docs/management/scaling/)

**Guestbook Tutorial:**
- [Kubernetes Guestbook Example](https://kubernetes.io/docs/tutorials/stateless-application/guestbook/)

**Next Steps:**
- **Day 68:** StatefulSets for Redis Cluster
- **Day 69:** Persistent Storage for Redis
- **Day 70:** Horizontal Pod Autoscaling
- **Day 71:** Ingress Controllers

---

## ✅ Task Completion Checklist

**BACK-END TIER:**
- [ ] Created Redis Master deployment (1 replica)
- [ ] Container name: master-redis-devops
- [ ] Image: redis
- [ ] CPU request: 100m, Memory request: 100Mi
- [ ] Container port: 6379
- [ ] Created redis-master service (port 6379)
- [ ] Created Redis Slave deployment (2 replicas)
- [ ] Container name: slave-redis-devops
- [ ] Image: gcr.io/google_samples/gb-redisslave:v3
- [ ] CPU request: 100m, Memory request: 100Mi
- [ ] Environment: GET_HOSTS_FROM=dns
- [ ] Container port: 6379
- [ ] Created redis-slave service (port 6379)
- [ ] Verified Redis replication working

**FRONT-END TIER:**
- [ ] Created frontend deployment (3 replicas)
- [ ] Container name: php-redis-devops
- [ ] Image: gcr.io/google-samples/gb-frontend@sha256:...
- [ ] CPU request: 100m, Memory request: 100Mi
- [ ] Environment: GET_HOSTS_FROM=dns
- [ ] Container port: 80
- [ ] Created frontend service (NodePort 30009)
- [ ] Tested guestbook application
- [ ] Verified write operations (Redis Master)
- [ ] Verified read operations (Redis Slave)
- [ ] Application accessible via browser

---

**🎉 Congratulations!** You've successfully deployed a multi-tier guestbook application with Redis master-slave replication and PHP frontend! You now understand how to build scalable, distributed applications on Kubernetes with proper tier separation and service discovery.

**Day 67 Status:** ✅ Complete

**Next:** Day 68 - StatefulSets and Persistent Storage 🚀
