# Day 63: Kubernetes Multi-Tier Application Deployment - Iron Gallery

## 📋 Task Overview

**Scenario:** The Nautilus DevOps team has developed the Iron Gallery application - a web-based photo gallery with a MariaDB backend. After customization, they need to deploy it to their Kubernetes cluster with proper resource management, storage, and networking.

**Application Architecture:**
- **Frontend:** Iron Gallery (nginx-based web app)
- **Backend:** MariaDB database
- **Services:** NodePort for frontend, ClusterIP for database

**Requirements:**
1. Create dedicated namespace `iron-namespace-nautilus`
2. Deploy Iron Gallery frontend with resource limits and volumes
3. Deploy MariaDB database with environment variables and persistent storage
4. Create ClusterIP service for database (internal access)
5. Create NodePort service for frontend (external access on port 32678)

---

## 🎯 Learning Objectives

By the end of this task, you will understand:
- **Namespaces:** Logical cluster partitioning and resource isolation
- **Multi-Container Apps:** Frontend and backend deployment patterns
- **Resource Limits:** CPU and memory constraints
- **EmptyDir Volumes:** Temporary storage for pods
- **Service Types:** ClusterIP vs NodePort
- **Environment Variables:** Database configuration
- **Label Selectors:** Service-to-pod routing

---

## 📖 Understanding Kubernetes Namespaces

### What are Namespaces?

**Definition:** Virtual clusters within a physical Kubernetes cluster that provide scope for resource names and isolation.

**Key Characteristics:**
- ✅ Logical resource separation
- ✅ Resource quotas and limits per namespace
- ✅ RBAC policies per namespace
- ✅ Network policies for isolation
- ✅ Multiple teams/projects in one cluster

### Namespace Use Cases

```
┌─────────────────────────────────────────────────────────────────┐
│                    Kubernetes Cluster                            │
│                                                                  │
│  ┌────────────────┐  ┌────────────────┐  ┌────────────────┐   │
│  │  Namespace:    │  │  Namespace:    │  │  Namespace:    │   │
│  │  development   │  │  staging       │  │  production    │   │
│  │                │  │                │  │                │   │
│  │  • dev-app     │  │  • staging-app │  │  • prod-app    │   │
│  │  • dev-db      │  │  • staging-db  │  │  • prod-db     │   │
│  │  • dev-cache   │  │  • staging-cache│ │  • prod-cache  │   │
│  └────────────────┘  └────────────────┘  └────────────────┘   │
│                                                                  │
│  ┌────────────────┐  ┌────────────────┐  ┌────────────────┐   │
│  │  Namespace:    │  │  Namespace:    │  │  Namespace:    │   │
│  │  team-alpha    │  │  team-beta     │  │  monitoring    │   │
│  └────────────────┘  └────────────────┘  └────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

| Use Case | Example | Benefits |
|----------|---------|----------|
| **Environment Separation** | dev, staging, prod | Isolate environments |
| **Team Isolation** | team-a, team-b | Separate team resources |
| **Application Grouping** | app-frontend, app-backend | Organize components |
| **Resource Quotas** | limit CPU/memory per namespace | Prevent resource hogging |
| **Access Control** | RBAC per namespace | Security boundaries |

---

## 🏗️ Application Architecture

### Iron Gallery Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                  iron-namespace-nautilus                         │
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  External Traffic (Internet/Users)                       │  │
│  └────────────────────────┬─────────────────────────────────┘  │
│                           │                                     │
│                           ↓ NodePort 32678                      │
│  ┌────────────────────────────────────────────────────────┐    │
│  │  Service: iron-gallery-service-nautilus               │    │
│  │  Type: NodePort                                        │    │
│  │  Port: 80 → 32678                                      │    │
│  │  Selector: run=iron-gallery                            │    │
│  └────────────────────┬───────────────────────────────────┘    │
│                       │                                         │
│                       ↓                                         │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  Deployment: iron-gallery-deployment-nautilus          │   │
│  │  Replicas: 1                                           │   │
│  │  Labels: run=iron-gallery                              │   │
│  │                                                         │   │
│  │  ┌───────────────────────────────────────────────┐     │   │
│  │  │  Pod: iron-gallery-container-nautilus        │     │   │
│  │  │  Image: kodekloud/irongallery:2.0            │     │   │
│  │  │  Resources:                                   │     │   │
│  │  │    CPU: 50m, Memory: 100Mi                   │     │   │
│  │  │                                               │     │   │
│  │  │  Volumes:                                     │     │   │
│  │  │  • config → /usr/share/nginx/html/data       │     │   │
│  │  │  • images → /usr/share/nginx/html/uploads    │     │   │
│  │  └───────────────────┬───────────────────────────┘     │   │
│  └────────────────────────────────────────────────────────┘   │
│                        │ Connects to                          │
│                        ↓                                       │
│  ┌────────────────────────────────────────────────────────┐   │
│  │  Service: iron-db-service-nautilus                    │   │
│  │  Type: ClusterIP (internal only)                      │   │
│  │  Port: 3306                                            │   │
│  │  Selector: db=mariadb                                  │   │
│  └────────────────────┬───────────────────────────────────┘   │
│                       │                                        │
│                       ↓                                        │
│  ┌─────────────────────────────────────────────────────────┐  │
│  │  Deployment: iron-db-deployment-nautilus               │  │
│  │  Replicas: 1                                           │  │
│  │  Labels: db=mariadb                                    │  │
│  │                                                         │  │
│  │  ┌───────────────────────────────────────────────┐     │  │
│  │  │  Pod: iron-db-container-nautilus             │     │  │
│  │  │  Image: kodekloud/irondb:2.0                 │     │  │
│  │  │                                               │     │  │
│  │  │  Environment:                                 │     │  │
│  │  │  • MYSQL_DATABASE=database_host              │     │  │
│  │  │  • MYSQL_ROOT_PASSWORD=<secure>              │     │  │
│  │  │  • MYSQL_PASSWORD=<secure>                   │     │  │
│  │  │  • MYSQL_USER=<custom>                       │     │  │
│  │  │                                               │     │  │
│  │  │  Volume:                                      │     │  │
│  │  │  • db → /var/lib/mysql                       │     │  │
│  │  └───────────────────────────────────────────────┘     │  │
│  └─────────────────────────────────────────────────────────┘  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🚀 Task Implementation

### Step 1: Create Namespace

**What is it?**
A namespace provides logical isolation for the Iron Gallery application and all its resources.

**Command:**
```bash
kubectl create namespace iron-namespace-nautilus
```

**Expected Output:**
```
namespace/iron-namespace-nautilus created
```

**Verify Namespace:**
```bash
# List all namespaces
kubectl get namespaces

# Check specific namespace
kubectl get namespace iron-namespace-nautilus

# Describe namespace
kubectl describe namespace iron-namespace-nautilus
```

**Expected Output:**
```
NAME                      STATUS   AGE
iron-namespace-nautilus   Active   10s
```

**Alternative: YAML Method**
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: iron-namespace-nautilus
  labels:
    app: iron-gallery
    team: nautilus
```

```bash
kubectl apply -f namespace.yaml
```

---

### Step 2: Create Iron Gallery Deployment

**Complete Deployment YAML:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: iron-gallery-deployment-nautilus
  namespace: iron-namespace-nautilus
  labels:
    run: iron-gallery
spec:
  replicas: 1
  selector:
    matchLabels:
      run: iron-gallery
  template:
    metadata:
      labels:
        run: iron-gallery
    spec:
      containers:
      - name: iron-gallery-container-nautilus
        image: kodekloud/irongallery:2.0
        resources:
          limits:
            memory: "100Mi"
            cpu: "50m"
        volumeMounts:
        - name: config
          mountPath: /usr/share/nginx/html/data
        - name: images
          mountPath: /usr/share/nginx/html/uploads
      volumes:
      - name: config
        emptyDir: {}
      - name: images
        emptyDir: {}
```

**Understanding Each Component:**

**1. Metadata Section:**
```yaml
metadata:
  name: iron-gallery-deployment-nautilus
  namespace: iron-namespace-nautilus
  labels:
    run: iron-gallery
```
- **name:** Deployment identifier
- **namespace:** Deploys to iron-namespace-nautilus
- **labels:** `run: iron-gallery` for identification

**2. Spec Section:**
```yaml
spec:
  replicas: 1
  selector:
    matchLabels:
      run: iron-gallery
```
- **replicas:** 1 pod instance
- **selector:** Matches pods with label `run: iron-gallery`

**3. Template Section:**
```yaml
template:
  metadata:
    labels:
      run: iron-gallery
```
- **labels:** Pod template labels (must match selector)

**4. Container Configuration:**
```yaml
containers:
- name: iron-gallery-container-nautilus
  image: kodekloud/irongallery:2.0
  resources:
    limits:
      memory: "100Mi"
      cpu: "50m"
```
- **name:** Container identifier
- **image:** Exact image with tag `2.0`
- **resources.limits:**
  - **memory:** Maximum 100 MiB
  - **cpu:** Maximum 50 millicores (0.05 CPU)

**5. Volume Mounts:**
```yaml
volumeMounts:
- name: config
  mountPath: /usr/share/nginx/html/data
- name: images
  mountPath: /usr/share/nginx/html/uploads
```
- **config volume:** Mounted at `/usr/share/nginx/html/data`
- **images volume:** Mounted at `/usr/share/nginx/html/uploads`

**6. Volumes:**
```yaml
volumes:
- name: config
  emptyDir: {}
- name: images
  emptyDir: {}
```
- **emptyDir:** Temporary storage, deleted when pod terminates
- **config:** For application configuration
- **images:** For uploaded gallery images

**Create the Deployment:**
```bash
# Save to file
cat > iron-gallery-deployment.yaml <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: iron-gallery-deployment-nautilus
  namespace: iron-namespace-nautilus
  labels:
    run: iron-gallery
spec:
  replicas: 1
  selector:
    matchLabels:
      run: iron-gallery
  template:
    metadata:
      labels:
        run: iron-gallery
    spec:
      containers:
      - name: iron-gallery-container-nautilus
        image: kodekloud/irongallery:2.0
        resources:
          limits:
            memory: "100Mi"
            cpu: "50m"
        volumeMounts:
        - name: config
          mountPath: /usr/share/nginx/html/data
        - name: images
          mountPath: /usr/share/nginx/html/uploads
      volumes:
      - name: config
        emptyDir: {}
      - name: images
        emptyDir: {}
EOF

# Apply deployment
kubectl apply -f iron-gallery-deployment.yaml
```

**Expected Output:**
```
deployment.apps/iron-gallery-deployment-nautilus created
```

**Verify Deployment:**
```bash
# Check deployment
kubectl get deployment -n iron-namespace-nautilus

# Check pods
kubectl get pods -n iron-namespace-nautilus

# Describe deployment
kubectl describe deployment iron-gallery-deployment-nautilus -n iron-namespace-nautilus
```

---

### Step 3: Create MariaDB Deployment

**Complete Deployment YAML:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: iron-db-deployment-nautilus
  namespace: iron-namespace-nautilus
  labels:
    db: mariadb
spec:
  replicas: 1
  selector:
    matchLabels:
      db: mariadb
  template:
    metadata:
      labels:
        db: mariadb
    spec:
      containers:
      - name: iron-db-container-nautilus
        image: kodekloud/irondb:2.0
        env:
        - name: MYSQL_DATABASE
          value: "database_host"
        - name: MYSQL_ROOT_PASSWORD
          value: "R00tP@ssw0rd#2026"
        - name: MYSQL_PASSWORD
          value: "Us3rP@ssw0rd#2026"
        - name: MYSQL_USER
          value: "iron_user"
        volumeMounts:
        - name: db
          mountPath: /var/lib/mysql
      volumes:
      - name: db
        emptyDir: {}
```

**Understanding Each Component:**

**1. Metadata Section:**
```yaml
metadata:
  name: iron-db-deployment-nautilus
  namespace: iron-namespace-nautilus
  labels:
    db: mariadb
```
- **labels:** `db: mariadb` (different from frontend)

**2. Selector:**
```yaml
selector:
  matchLabels:
    db: mariadb
```
- Matches pods with label `db: mariadb`

**3. Container Configuration:**
```yaml
containers:
- name: iron-db-container-nautilus
  image: kodekloud/irondb:2.0
```
- **image:** MariaDB database with tag `2.0`

**4. Environment Variables:**
```yaml
env:
- name: MYSQL_DATABASE
  value: "database_host"
- name: MYSQL_ROOT_PASSWORD
  value: "R00tP@ssw0rd#2026"
- name: MYSQL_PASSWORD
  value: "Us3rP@ssw0rd#2026"
- name: MYSQL_USER
  value: "iron_user"
```

**Environment Variable Details:**

| Variable | Purpose | Example Value |
|----------|---------|---------------|
| `MYSQL_DATABASE` | Database name to create | `database_host` |
| `MYSQL_ROOT_PASSWORD` | Root user password | `R00tP@ssw0rd#2026` |
| `MYSQL_USER` | Non-root username | `iron_user` |
| `MYSQL_PASSWORD` | Non-root user password | `Us3rP@ssw0rd#2026` |

**Password Requirements:**
- ✅ Complex passwords (uppercase, lowercase, numbers, symbols)
- ✅ Minimum 12 characters recommended
- ✅ Different for root and user
- ❌ Don't use simple passwords like `password123`

**5. Volume Mount:**
```yaml
volumeMounts:
- name: db
  mountPath: /var/lib/mysql
```
- MariaDB data directory mount point

**6. Volume:**
```yaml
volumes:
- name: db
  emptyDir: {}
```
- Temporary database storage

**Create the Deployment:**
```bash
# Save to file
cat > iron-db-deployment.yaml <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: iron-db-deployment-nautilus
  namespace: iron-namespace-nautilus
  labels:
    db: mariadb
spec:
  replicas: 1
  selector:
    matchLabels:
      db: mariadb
  template:
    metadata:
      labels:
        db: mariadb
    spec:
      containers:
      - name: iron-db-container-nautilus
        image: kodekloud/irondb:2.0
        env:
        - name: MYSQL_DATABASE
          value: "database_host"
        - name: MYSQL_ROOT_PASSWORD
          value: "R00tP@ssw0rd#2026"
        - name: MYSQL_PASSWORD
          value: "Us3rP@ssw0rd#2026"
        - name: MYSQL_USER
          value: "iron_user"
        volumeMounts:
        - name: db
          mountPath: /var/lib/mysql
      volumes:
      - name: db
        emptyDir: {}
EOF

# Apply deployment
kubectl apply -f iron-db-deployment.yaml
```

**Expected Output:**
```
deployment.apps/iron-db-deployment-nautilus created
```

**Verify Database Deployment:**
```bash
# Check deployment
kubectl get deployment iron-db-deployment-nautilus -n iron-namespace-nautilus

# Check pods
kubectl get pods -l db=mariadb -n iron-namespace-nautilus

# Check logs
kubectl logs -l db=mariadb -n iron-namespace-nautilus
```

---

### Step 4: Create Database Service (ClusterIP)

**Complete Service YAML:**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: iron-db-service-nautilus
  namespace: iron-namespace-nautilus
spec:
  selector:
    db: mariadb
  type: ClusterIP
  ports:
  - protocol: TCP
    port: 3306
    targetPort: 3306
```

**Understanding Each Component:**

**1. Service Type: ClusterIP**
```yaml
type: ClusterIP
```
- **Internal only** - Not accessible from outside cluster
- Default service type
- Provides stable internal IP
- Used for backend services (databases, caches)

**2. Selector:**
```yaml
selector:
  db: mariadb
```
- Routes traffic to pods with label `db: mariadb`
- Matches the database deployment pods

**3. Ports:**
```yaml
ports:
- protocol: TCP
  port: 3306
  targetPort: 3306
```
- **protocol:** TCP (MySQL/MariaDB uses TCP)
- **port:** Service port (3306 - MySQL default)
- **targetPort:** Container port (3306)

**Service Access Pattern:**
```
Frontend Pod
     ↓
iron-db-service-nautilus:3306
     ↓
ClusterIP (internal IP)
     ↓
iron-db-container-nautilus:3306
```

**Create the Service:**
```bash
# Save to file
cat > iron-db-service.yaml <<EOF
apiVersion: v1
kind: Service
metadata:
  name: iron-db-service-nautilus
  namespace: iron-namespace-nautilus
spec:
  selector:
    db: mariadb
  type: ClusterIP
  ports:
  - protocol: TCP
    port: 3306
    targetPort: 3306
EOF

# Apply service
kubectl apply -f iron-db-service.yaml
```

**Expected Output:**
```
service/iron-db-service-nautilus created
```

**Verify Service:**
```bash
# Check service
kubectl get svc iron-db-service-nautilus -n iron-namespace-nautilus

# Describe service
kubectl describe svc iron-db-service-nautilus -n iron-namespace-nautilus
```

**Expected Output:**
```
NAME                        TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
iron-db-service-nautilus    ClusterIP   10.96.45.123    <none>        3306/TCP   15s
```

---

### Step 5: Create Frontend Service (NodePort)

**Complete Service YAML:**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: iron-gallery-service-nautilus
  namespace: iron-namespace-nautilus
spec:
  selector:
    run: iron-gallery
  type: NodePort
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
    nodePort: 32678
```

**Understanding Each Component:**

**1. Service Type: NodePort**
```yaml
type: NodePort
```
- **External access** - Accessible from outside cluster
- Opens port on all nodes
- Port range: 30000-32767
- Used for frontend services

**2. Selector:**
```yaml
selector:
  run: iron-gallery
```
- Routes traffic to pods with label `run: iron-gallery`
- Matches the frontend deployment pods

**3. Ports:**
```yaml
ports:
- protocol: TCP
  port: 80
  targetPort: 80
  nodePort: 32678
```
- **protocol:** TCP (HTTP uses TCP)
- **port:** Service port (80 - HTTP default)
- **targetPort:** Container port (80)
- **nodePort:** External access port (32678)

**Service Access Pattern:**
```
External User (Browser)
     ↓
http://node-ip:32678
     ↓
NodePort Service
     ↓
iron-gallery-service-nautilus:80
     ↓
iron-gallery-container-nautilus:80
```

**Create the Service:**
```bash
# Save to file
cat > iron-gallery-service.yaml <<EOF
apiVersion: v1
kind: Service
metadata:
  name: iron-gallery-service-nautilus
  namespace: iron-namespace-nautilus
spec:
  selector:
    run: iron-gallery
  type: NodePort
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
    nodePort: 32678
EOF

# Apply service
kubectl apply -f iron-gallery-service.yaml
```

**Expected Output:**
```
service/iron-gallery-service-nautilus created
```

**Verify Service:**
```bash
# Check service
kubectl get svc iron-gallery-service-nautilus -n iron-namespace-nautilus

# Describe service
kubectl describe svc iron-gallery-service-nautilus -n iron-namespace-nautilus
```

**Expected Output:**
```
NAME                            TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
iron-gallery-service-nautilus   NodePort   10.96.123.45    <none>        80:32678/TCP   20s
```

---

### Step 6: Verify Complete Deployment

**Check All Resources:**
```bash
# List all resources in namespace
kubectl get all -n iron-namespace-nautilus

# Expected output:
# NAME                                                 READY   STATUS    RESTARTS   AGE
# pod/iron-gallery-deployment-nautilus-xxxxx-xxxxx    1/1     Running   0          5m
# pod/iron-db-deployment-nautilus-xxxxx-xxxxx         1/1     Running   0          4m
#
# NAME                                    TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
# service/iron-db-service-nautilus        ClusterIP   10.96.45.123    <none>        3306/TCP       3m
# service/iron-gallery-service-nautilus   NodePort    10.96.123.45    <none>        80:32678/TCP   2m
#
# NAME                                           READY   UP-TO-DATE   AVAILABLE   AGE
# deployment.apps/iron-gallery-deployment-nautilus   1/1     1            1           5m
# deployment.apps/iron-db-deployment-nautilus        1/1     1            1           4m
```

**Check Pods Status:**
```bash
kubectl get pods -n iron-namespace-nautilus -o wide
```

**Check Services:**
```bash
kubectl get svc -n iron-namespace-nautilus -o wide
```

**Check Deployments:**
```bash
kubectl get deployments -n iron-namespace-nautilus
```

---

### Step 7: Test Application Access

**Get Node IP:**
```bash
# Get node IP
kubectl get nodes -o wide

# Example output:
# NAME           STATUS   ROLES    AGE   VERSION   INTERNAL-IP    EXTERNAL-IP
# node01         Ready    <none>   10d   v1.27.0   172.17.0.2     <none>
```

**Access Application:**
```bash
# Using curl
curl http://<node-ip>:32678

# Or in browser
# http://<node-ip>:32678
```

**Expected Response:**
- Installation page for Iron Gallery
- Form to configure database connection
- No connection errors (database service is reachable internally)

**Check Application Logs:**
```bash
# Frontend logs
kubectl logs -l run=iron-gallery -n iron-namespace-nautilus

# Database logs
kubectl logs -l db=mariadb -n iron-namespace-nautilus

# Follow logs
kubectl logs -f -l run=iron-gallery -n iron-namespace-nautilus
```

---

## 📊 Complete All-in-One YAML

For convenience, here's a single YAML file with all resources:

```yaml
# Namespace
apiVersion: v1
kind: Namespace
metadata:
  name: iron-namespace-nautilus
  labels:
    app: iron-gallery

---
# Iron Gallery Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: iron-gallery-deployment-nautilus
  namespace: iron-namespace-nautilus
  labels:
    run: iron-gallery
spec:
  replicas: 1
  selector:
    matchLabels:
      run: iron-gallery
  template:
    metadata:
      labels:
        run: iron-gallery
    spec:
      containers:
      - name: iron-gallery-container-nautilus
        image: kodekloud/irongallery:2.0
        resources:
          limits:
            memory: "100Mi"
            cpu: "50m"
        volumeMounts:
        - name: config
          mountPath: /usr/share/nginx/html/data
        - name: images
          mountPath: /usr/share/nginx/html/uploads
      volumes:
      - name: config
        emptyDir: {}
      - name: images
        emptyDir: {}

---
# MariaDB Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: iron-db-deployment-nautilus
  namespace: iron-namespace-nautilus
  labels:
    db: mariadb
spec:
  replicas: 1
  selector:
    matchLabels:
      db: mariadb
  template:
    metadata:
      labels:
        db: mariadb
    spec:
      containers:
      - name: iron-db-container-nautilus
        image: kodekloud/irondb:2.0
        env:
        - name: MYSQL_DATABASE
          value: "database_host"
        - name: MYSQL_ROOT_PASSWORD
          value: "R00tP@ssw0rd#2026"
        - name: MYSQL_PASSWORD
          value: "Us3rP@ssw0rd#2026"
        - name: MYSQL_USER
          value: "iron_user"
        volumeMounts:
        - name: db
          mountPath: /var/lib/mysql
      volumes:
      - name: db
        emptyDir: {}

---
# Database Service (ClusterIP)
apiVersion: v1
kind: Service
metadata:
  name: iron-db-service-nautilus
  namespace: iron-namespace-nautilus
spec:
  selector:
    db: mariadb
  type: ClusterIP
  ports:
  - protocol: TCP
    port: 3306
    targetPort: 3306

---
# Frontend Service (NodePort)
apiVersion: v1
kind: Service
metadata:
  name: iron-gallery-service-nautilus
  namespace: iron-namespace-nautilus
spec:
  selector:
    run: iron-gallery
  type: NodePort
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
    nodePort: 32678
```

**Deploy Everything:**
```bash
# Save to file
kubectl apply -f iron-gallery-complete.yaml

# Or deploy all files at once
kubectl apply -f .
```

---

## 🐛 Common Issues and Troubleshooting

### Issue 1: Namespace Not Found

**Symptoms:**
```bash
kubectl apply -f iron-gallery-deployment.yaml
# Error: namespace "iron-namespace-nautilus" not found
```

**Solution:**
```bash
# Create namespace first
kubectl create namespace iron-namespace-nautilus

# Or apply all resources including namespace
kubectl apply -f iron-gallery-complete.yaml
```

---

### Issue 2: Image Pull Errors

**Symptoms:**
```bash
kubectl get pods -n iron-namespace-nautilus
# NAME                                          READY   STATUS         RESTARTS   AGE
# iron-gallery-deployment-nautilus-xxx-xxx      0/1     ErrImagePull   0          30s
```

**Diagnosis:**
```bash
kubectl describe pod <pod-name> -n iron-namespace-nautilus
# Events:
#   Failed to pull image "kodekloud/irongallery:2.0": rpc error: code = Unknown desc = Error response from daemon: manifest for kodekloud/irongallery:2.0 not found
```

**Solution:**
```bash
# Verify exact image name and tag
docker pull kodekloud/irongallery:2.0

# Check if image exists on Docker Hub
# Ensure exact spelling: kodekloud/irongallery:2.0
```

---

### Issue 3: Pod Stuck in ContainerCreating

**Symptoms:**
```bash
kubectl get pods -n iron-namespace-nautilus
# NAME                                          READY   STATUS              RESTARTS   AGE
# iron-gallery-deployment-nautilus-xxx-xxx      0/1     ContainerCreating   0          2m
```

**Diagnosis:**
```bash
kubectl describe pod <pod-name> -n iron-namespace-nautilus

# Common causes:
# - Volume mount issues
# - Resource constraints
# - Image pull in progress
```

**Solution:**
```bash
# Wait for image pull to complete (check events)
kubectl get events -n iron-namespace-nautilus --sort-by='.lastTimestamp'

# Check node resources
kubectl top nodes

# If volume issue, verify volume configuration
kubectl get pod <pod-name> -n iron-namespace-nautilus -o yaml | grep -A10 volumes
```

---

### Issue 4: Resource Limit Exceeded

**Symptoms:**
```bash
kubectl describe pod <pod-name> -n iron-namespace-nautilus
# Events:
#   Warning  FailedScheduling  pod  0/1 nodes are available: 1 Insufficient memory
```

**Solution:**
```bash
# Check node resources
kubectl describe nodes

# Adjust resource limits in deployment
kubectl edit deployment iron-gallery-deployment-nautilus -n iron-namespace-nautilus

# Reduce limits:
resources:
  limits:
    memory: "50Mi"  # Reduced from 100Mi
    cpu: "25m"      # Reduced from 50m
```

---

### Issue 5: Service Not Routing Traffic

**Symptoms:**
```bash
curl http://<node-ip>:32678
# Connection refused or timeout
```

**Diagnosis:**
```bash
# Check service
kubectl get svc iron-gallery-service-nautilus -n iron-namespace-nautilus

# Check endpoints
kubectl get endpoints iron-gallery-service-nautilus -n iron-namespace-nautilus

# Should show pod IPs:
# NAME                            ENDPOINTS         AGE
# iron-gallery-service-nautilus   10.244.1.5:80     5m
```

**Solution:**
```bash
# If no endpoints, check selector
kubectl get svc iron-gallery-service-nautilus -n iron-namespace-nautilus -o yaml | grep -A2 selector

# Verify pods have matching labels
kubectl get pods -n iron-namespace-nautilus --show-labels

# Fix selector mismatch
kubectl edit svc iron-gallery-service-nautilus -n iron-namespace-nautilus
```

---

### Issue 6: Database Connection Errors

**Symptoms:**
```bash
# Frontend shows database connection error
kubectl logs -l run=iron-gallery -n iron-namespace-nautilus
# Error: Can't connect to MySQL server on 'iron-db-service-nautilus' (111)
```

**Diagnosis:**
```bash
# Check database pod status
kubectl get pods -l db=mariadb -n iron-namespace-nautilus

# Check database logs
kubectl logs -l db=mariadb -n iron-namespace-nautilus

# Check service
kubectl get svc iron-db-service-nautilus -n iron-namespace-nautilus

# Check if database is listening
kubectl exec -it <db-pod-name> -n iron-namespace-nautilus -- mysql -u root -p
```

**Solution:**
```bash
# Verify environment variables
kubectl get pod <db-pod-name> -n iron-namespace-nautilus -o yaml | grep -A10 env

# Check service DNS resolution from frontend
kubectl exec -it <gallery-pod-name> -n iron-namespace-nautilus -- nslookup iron-db-service-nautilus

# Restart database pod if needed
kubectl delete pod <db-pod-name> -n iron-namespace-nautilus
```

---

### Issue 7: NodePort Not Accessible

**Symptoms:**
```bash
curl http://<node-ip>:32678
# Connection timeout
```

**Solution:**
```bash
# Check firewall rules (if applicable)
# Ensure port 32678 is open

# Check service type and port
kubectl get svc iron-gallery-service-nautilus -n iron-namespace-nautilus -o yaml

# Verify nodePort is 32678
# ports:
# - nodePort: 32678
#   port: 80
#   protocol: TCP
#   targetPort: 80

# Test from inside cluster
kubectl run -it --rm debug --image=busybox --restart=Never -- wget -O- http://iron-gallery-service-nautilus.iron-namespace-nautilus.svc.cluster.local
```

---

## 🎯 Best Practices

### 1. Use Secrets for Sensitive Data

**❌ Bad: Hardcoded passwords in deployment**
```yaml
env:
- name: MYSQL_ROOT_PASSWORD
  value: "R00tP@ssw0rd#2026"  # Visible in YAML
```

**✅ Good: Use Kubernetes Secrets**
```yaml
# Create secret
kubectl create secret generic db-secrets \
  --from-literal=root-password=R00tP@ssw0rd#2026 \
  --from-literal=user-password=Us3rP@ssw0rd#2026 \
  -n iron-namespace-nautilus

# Reference in deployment
env:
- name: MYSQL_ROOT_PASSWORD
  valueFrom:
    secretKeyRef:
      name: db-secrets
      key: root-password
- name: MYSQL_PASSWORD
  valueFrom:
    secretKeyRef:
      name: db-secrets
      key: user-password
```

---

### 2. Use ConfigMaps for Configuration

**✅ Good: Separate configuration from code**
```yaml
# Create ConfigMap
apiVersion: v1
kind: ConfigMap
metadata:
  name: db-config
  namespace: iron-namespace-nautilus
data:
  database-name: "database_host"
  database-user: "iron_user"

# Reference in deployment
env:
- name: MYSQL_DATABASE
  valueFrom:
    configMapKeyRef:
      name: db-config
      key: database-name
```

---

### 3. Add Resource Requests

**✅ Good: Set both requests and limits**
```yaml
resources:
  requests:
    memory: "64Mi"
    cpu: "25m"
  limits:
    memory: "100Mi"
    cpu: "50m"
```

**Benefits:**
- Better pod scheduling
- Prevents resource starvation
- QoS (Quality of Service) guarantees

---

### 4. Use PersistentVolumes for Database

**❌ Bad: emptyDir (data lost on pod restart)**
```yaml
volumes:
- name: db
  emptyDir: {}
```

**✅ Good: PersistentVolumeClaim**
```yaml
volumes:
- name: db
  persistentVolumeClaim:
    claimName: mariadb-pvc
```

---

### 5. Add Readiness and Liveness Probes

**✅ Good: Health checks**
```yaml
containers:
- name: iron-gallery-container-nautilus
  image: kodekloud/irongallery:2.0
  livenessProbe:
    httpGet:
      path: /
      port: 80
    initialDelaySeconds: 30
    periodSeconds: 10
  readinessProbe:
    httpGet:
      path: /
      port: 80
    initialDelaySeconds: 5
    periodSeconds: 5
```

---

### 6. Add Labels and Annotations

**✅ Good: Comprehensive metadata**
```yaml
metadata:
  name: iron-gallery-deployment-nautilus
  namespace: iron-namespace-nautilus
  labels:
    app: iron-gallery
    tier: frontend
    version: v2.0
    team: nautilus
  annotations:
    description: "Iron Gallery photo application"
    owner: "nautilus-devops-team"
    contact: "devops@nautilus.local"
```

---

### 7. Use Network Policies

**✅ Good: Restrict traffic**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-network-policy
  namespace: iron-namespace-nautilus
spec:
  podSelector:
    matchLabels:
      db: mariadb
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          run: iron-gallery
    ports:
    - protocol: TCP
      port: 3306
```

---

## 📋 Complete Verification Script

```bash
#!/bin/bash

echo "🔍 Iron Gallery Application - Complete Verification"
echo "====================================================="

# Colors
GREEN='\033[0;32m'
RED='\033[0;31m'
YELLOW='\033[1;33m'
NC='\033[0m'

NAMESPACE="iron-namespace-nautilus"

# 1. Check Namespace
echo -e "\n${YELLOW}1️⃣ Checking Namespace...${NC}"
if kubectl get namespace $NAMESPACE &>/dev/null; then
    echo -e "${GREEN}✅ Namespace '$NAMESPACE' exists${NC}"
else
    echo -e "${RED}❌ Namespace '$NAMESPACE' not found${NC}"
    exit 1
fi

# 2. Check Deployments
echo -e "\n${YELLOW}2️⃣ Checking Deployments...${NC}"

# Gallery Deployment
if kubectl get deployment iron-gallery-deployment-nautilus -n $NAMESPACE &>/dev/null; then
    GALLERY_READY=$(kubectl get deployment iron-gallery-deployment-nautilus -n $NAMESPACE -o jsonpath='{.status.readyReplicas}')
    GALLERY_DESIRED=$(kubectl get deployment iron-gallery-deployment-nautilus -n $NAMESPACE -o jsonpath='{.spec.replicas}')
    
    if [ "$GALLERY_READY" == "$GALLERY_DESIRED" ]; then
        echo -e "${GREEN}✅ Gallery Deployment: $GALLERY_READY/$GALLERY_DESIRED ready${NC}"
    else
        echo -e "${RED}❌ Gallery Deployment: $GALLERY_READY/$GALLERY_DESIRED ready${NC}"
    fi
else
    echo -e "${RED}❌ Gallery deployment not found${NC}"
fi

# DB Deployment
if kubectl get deployment iron-db-deployment-nautilus -n $NAMESPACE &>/dev/null; then
    DB_READY=$(kubectl get deployment iron-db-deployment-nautilus -n $NAMESPACE -o jsonpath='{.status.readyReplicas}')
    DB_DESIRED=$(kubectl get deployment iron-db-deployment-nautilus -n $NAMESPACE -o jsonpath='{.spec.replicas}')
    
    if [ "$DB_READY" == "$DB_DESIRED" ]; then
        echo -e "${GREEN}✅ Database Deployment: $DB_READY/$DB_DESIRED ready${NC}"
    else
        echo -e "${RED}❌ Database Deployment: $DB_READY/$DB_DESIRED ready${NC}"
    fi
else
    echo -e "${RED}❌ Database deployment not found${NC}"
fi

# 3. Check Pods
echo -e "\n${YELLOW}3️⃣ Checking Pods...${NC}"

GALLERY_POD=$(kubectl get pods -l run=iron-gallery -n $NAMESPACE -o jsonpath='{.items[0].metadata.name}' 2>/dev/null)
if [ -n "$GALLERY_POD" ]; then
    POD_STATUS=$(kubectl get pod $GALLERY_POD -n $NAMESPACE -o jsonpath='{.status.phase}')
    POD_READY=$(kubectl get pod $GALLERY_POD -n $NAMESPACE -o jsonpath='{.status.containerStatuses[0].ready}')
    
    if [ "$POD_STATUS" == "Running" ] && [ "$POD_READY" == "true" ]; then
        echo -e "${GREEN}✅ Gallery Pod: Running and Ready${NC}"
    else
        echo -e "${RED}❌ Gallery Pod: Status=$POD_STATUS, Ready=$POD_READY${NC}"
    fi
else
    echo -e "${RED}❌ Gallery pod not found${NC}"
fi

DB_POD=$(kubectl get pods -l db=mariadb -n $NAMESPACE -o jsonpath='{.items[0].metadata.name}' 2>/dev/null)
if [ -n "$DB_POD" ]; then
    DB_POD_STATUS=$(kubectl get pod $DB_POD -n $NAMESPACE -o jsonpath='{.status.phase}')
    DB_POD_READY=$(kubectl get pod $DB_POD -n $NAMESPACE -o jsonpath='{.status.containerStatuses[0].ready}')
    
    if [ "$DB_POD_STATUS" == "Running" ] && [ "$DB_POD_READY" == "true" ]; then
        echo -e "${GREEN}✅ Database Pod: Running and Ready${NC}"
    else
        echo -e "${RED}❌ Database Pod: Status=$DB_POD_STATUS, Ready=$DB_POD_READY${NC}"
    fi
else
    echo -e "${RED}❌ Database pod not found${NC}"
fi

# 4. Check Services
echo -e "\n${YELLOW}4️⃣ Checking Services...${NC}"

# Gallery Service
if kubectl get svc iron-gallery-service-nautilus -n $NAMESPACE &>/dev/null; then
    SVC_TYPE=$(kubectl get svc iron-gallery-service-nautilus -n $NAMESPACE -o jsonpath='{.spec.type}')
    NODE_PORT=$(kubectl get svc iron-gallery-service-nautilus -n $NAMESPACE -o jsonpath='{.spec.ports[0].nodePort}')
    
    if [ "$SVC_TYPE" == "NodePort" ] && [ "$NODE_PORT" == "32678" ]; then
        echo -e "${GREEN}✅ Gallery Service: Type=$SVC_TYPE, NodePort=$NODE_PORT${NC}"
    else
        echo -e "${RED}❌ Gallery Service: Type=$SVC_TYPE, NodePort=$NODE_PORT (expected: NodePort/32678)${NC}"
    fi
else
    echo -e "${RED}❌ Gallery service not found${NC}"
fi

# DB Service
if kubectl get svc iron-db-service-nautilus -n $NAMESPACE &>/dev/null; then
    DB_SVC_TYPE=$(kubectl get svc iron-db-service-nautilus -n $NAMESPACE -o jsonpath='{.spec.type}')
    DB_PORT=$(kubectl get svc iron-db-service-nautilus -n $NAMESPACE -o jsonpath='{.spec.ports[0].port}')
    
    if [ "$DB_SVC_TYPE" == "ClusterIP" ] && [ "$DB_PORT" == "3306" ]; then
        echo -e "${GREEN}✅ Database Service: Type=$DB_SVC_TYPE, Port=$DB_PORT${NC}"
    else
        echo -e "${RED}❌ Database Service: Type=$DB_SVC_TYPE, Port=$DB_PORT (expected: ClusterIP/3306)${NC}"
    fi
else
    echo -e "${RED}❌ Database service not found${NC}"
fi

# 5. Check Resource Limits
echo -e "\n${YELLOW}5️⃣ Checking Resource Limits...${NC}"

MEMORY_LIMIT=$(kubectl get pod $GALLERY_POD -n $NAMESPACE -o jsonpath='{.spec.containers[0].resources.limits.memory}')
CPU_LIMIT=$(kubectl get pod $GALLERY_POD -n $NAMESPACE -o jsonpath='{.spec.containers[0].resources.limits.cpu}')

if [ "$MEMORY_LIMIT" == "100Mi" ] && [ "$CPU_LIMIT" == "50m" ]; then
    echo -e "${GREEN}✅ Resource Limits: Memory=$MEMORY_LIMIT, CPU=$CPU_LIMIT${NC}"
else
    echo -e "${YELLOW}⚠️  Resource Limits: Memory=$MEMORY_LIMIT, CPU=$CPU_LIMIT (expected: 100Mi/50m)${NC}"
fi

# 6. Check Volumes
echo -e "\n${YELLOW}6️⃣ Checking Volumes...${NC}"

VOLUME_COUNT=$(kubectl get pod $GALLERY_POD -n $NAMESPACE -o jsonpath='{.spec.volumes}' | grep -o emptyDir | wc -l)

if [ "$VOLUME_COUNT" -eq 2 ]; then
    echo -e "${GREEN}✅ EmptyDir Volumes: $VOLUME_COUNT (config, images)${NC}"
else
    echo -e "${RED}❌ EmptyDir Volumes: $VOLUME_COUNT (expected: 2)${NC}"
fi

# 7. Test Application Access
echo -e "\n${YELLOW}7️⃣ Testing Application Access...${NC}"

NODE_IP=$(kubectl get nodes -o jsonpath='{.items[0].status.addresses[?(@.type=="InternalIP")].address}')
HTTP_CODE=$(curl -s -o /dev/null -w "%{http_code}" http://$NODE_IP:32678 2>/dev/null || echo "000")

if [ "$HTTP_CODE" == "200" ]; then
    echo -e "${GREEN}✅ Application accessible at http://$NODE_IP:32678${NC}"
    echo "   HTTP Status: $HTTP_CODE"
elif [ "$HTTP_CODE" != "000" ]; then
    echo -e "${YELLOW}⚠️  Application responding but not 200 OK: HTTP $HTTP_CODE${NC}"
    echo "   URL: http://$NODE_IP:32678"
else
    echo -e "${RED}❌ Cannot access application at http://$NODE_IP:32678${NC}"
fi

# 8. Summary
echo -e "\n${YELLOW}====================================================${NC}"
echo -e "${YELLOW}📊 Summary:${NC}"
kubectl get all -n $NAMESPACE

echo -e "\n${GREEN}✅ Verification Complete!${NC}"
echo -e "\n${YELLOW}Access application at: http://$NODE_IP:32678${NC}"
```

**Save and run:**
```bash
chmod +x verify-iron-gallery.sh
./verify-iron-gallery.sh
```

---

## 🧹 Cleanup

```bash
# Delete entire namespace (deletes all resources inside)
kubectl delete namespace iron-namespace-nautilus

# Verify deletion
kubectl get namespace iron-namespace-nautilus
# Error from server (NotFound): namespaces "iron-namespace-nautilus" not found

# Or delete individual resources
kubectl delete deployment iron-gallery-deployment-nautilus -n iron-namespace-nautilus
kubectl delete deployment iron-db-deployment-nautilus -n iron-namespace-nautilus
kubectl delete service iron-gallery-service-nautilus -n iron-namespace-nautilus
kubectl delete service iron-db-service-nautilus -n iron-namespace-nautilus

# If you created YAML files
rm -f *.yaml verify-iron-gallery.sh
```

---

## 📖 Key Takeaways

### Concepts Mastered Today:

1. ✅ **Namespaces**
   - Logical resource isolation
   - Multiple applications in one cluster
   - Resource quotas and RBAC per namespace

2. ✅ **Multi-Tier Applications**
   - Frontend and backend separation
   - Service discovery via DNS
   - ClusterIP for internal, NodePort for external

3. ✅ **Resource Management**
   - CPU and memory limits
   - Container resource requests
   - Pod scheduling based on resources

4. ✅ **Storage**
   - EmptyDir for temporary storage
   - Volume mounts in containers
   - Data persistence patterns

5. ✅ **Networking**
   - Service types (ClusterIP, NodePort)
   - Port mapping (port, targetPort, nodePort)
   - Label selectors for routing

6. ✅ **Configuration**
   - Environment variables for database
   - Image tags and versions
   - Deployment replicas

---

## 🎯 Real-World Scenarios

| Scenario | Frontend Service | Backend Service | Storage |
|----------|-----------------|-----------------|---------|
| **Web App + DB** | NodePort/LoadBalancer | ClusterIP | PVC for DB |
| **Microservices** | Ingress | ClusterIP | ConfigMap/Secret |
| **API + Cache** | LoadBalancer | ClusterIP | EmptyDir for cache |
| **CMS + Media** | NodePort | ClusterIP | PVC for media |

---

## 📚 Additional Resources

**Official Documentation:**
- [Namespaces](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)
- [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
- [Services](https://kubernetes.io/docs/concepts/services-networking/service/)
- [Resource Management](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/)
- [EmptyDir Volumes](https://kubernetes.io/docs/concepts/storage/volumes/#emptydir)

**Next Steps:**
- **Day 64:** StatefulSets for stateful applications
- **Day 65:** Ingress controllers and routing
- **Day 66:** Persistent Volumes and Claims
- **Day 67:** Helm charts for package management

---

## ✅ Task Completion Checklist

- [ ] Namespace `iron-namespace-nautilus` created
- [ ] Gallery deployment created with correct labels
- [ ] Gallery replica count is 1
- [ ] Gallery container name is `iron-gallery-container-nautilus`
- [ ] Gallery image is `kodekloud/irongallery:2.0`
- [ ] Resource limits: 100Mi memory, 50m CPU
- [ ] Two volume mounts: config at `/usr/share/nginx/html/data`, images at `/usr/share/nginx/html/uploads`
- [ ] Two emptyDir volumes: config and images
- [ ] Database deployment created with correct labels
- [ ] Database replica count is 1
- [ ] Database container name is `iron-db-container-nautilus`
- [ ] Database image is `kodekloud/irondb:2.0`
- [ ] Environment variables set: MYSQL_DATABASE, MYSQL_ROOT_PASSWORD, MYSQL_PASSWORD, MYSQL_USER
- [ ] Database volume mount at `/var/lib/mysql`
- [ ] Database service (ClusterIP) created on port 3306
- [ ] Gallery service (NodePort) created on port 80:32678
- [ ] All pods are in Running state
- [ ] Application accessible at `http://<node-ip>:32678`
- [ ] Installation page loads successfully

---

**🎉 Congratulations!** You've successfully deployed a multi-tier application on Kubernetes with proper namespace isolation, resource management, and service networking!

**Day 63 Status:** ✅ Complete

**Next:** Day 64 - StatefulSets and Persistent Storage 🚀
