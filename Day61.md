# Day 61: Kubernetes Init Containers - Pre-deployment Configuration

## 📋 Task Overview

**Scenario:** The xFusion DevOps team needs to deploy applications that require pre-configuration before the main container starts. Some configurations cannot be baked into container images, so they'll use **Init Containers** to perform setup tasks during deployment.

**Requirements:**
1. Create a Deployment with init containers
2. Init container writes configuration message to shared volume
3. Main container reads and displays the message continuously
4. Use emptyDir volume for data sharing between containers

---

## 🎯 Learning Objectives

By the end of this task, you will understand:
- **Init Containers:** Special containers that run before app containers
- **Container Lifecycle:** Init containers → Main containers
- **Volume Sharing:** How containers share data via volumes
- **Use Cases:** Configuration, setup, dependency checking
- **Execution Order:** Sequential init container execution
- **Failure Handling:** Init container failures block deployment

---

## 📖 Understanding Init Containers

### What are Init Containers?

**Definition:** Specialized containers that run and complete **before** the main application containers start.

**Key Characteristics:**
- ✅ Always run to completion
- ✅ Run sequentially (one at a time)
- ✅ Must succeed before next init container or main container starts
- ✅ Have separate images from app containers
- ✅ Share volumes with main containers

### Init Containers vs Regular Containers

```
┌─────────────────────────────────────────────────────────────────┐
│                        Pod Lifecycle                             │
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │ Phase 1: Init Containers (Sequential)                    │  │
│  │                                                           │  │
│  │  ┌────────────┐      ┌────────────┐      ┌────────────┐ │  │
│  │  │ Init 1     │ ───→ │ Init 2     │ ───→ │ Init 3     │ │  │
│  │  │ (Setup)    │      │ (Config)   │      │ (Verify)   │ │  │
│  │  └────────────┘      └────────────┘      └────────────┘ │  │
│  │       ↓                   ↓                   ↓          │  │
│  │    Success             Success             Success       │  │
│  └────────────────────────────────┬───────────────────────────┘
│                                   │
│                                   ↓
│  ┌──────────────────────────────────────────────────────────┐  │
│  │ Phase 2: Main Containers (Parallel)                      │  │
│  │                                                           │  │
│  │  ┌────────────┐      ┌────────────┐      ┌────────────┐ │  │
│  │  │ App        │      │ Sidecar    │      │ Logger     │ │  │
│  │  │ Container  │      │ Container  │      │ Container  │ │  │
│  │  └────────────┘      └────────────┘      └────────────┘ │  │
│  │       ↓                   ↓                   ↓          │  │
│  │    Running             Running             Running       │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

| Aspect | Init Containers | Regular Containers |
|--------|----------------|-------------------|
| **Execution** | Sequential (one at a time) | Parallel (all at once) |
| **Timing** | Before app containers | After init containers |
| **Purpose** | Setup, configuration, prerequisites | Run application |
| **Lifecycle** | Run to completion, then exit | Long-running (usually) |
| **Failure Impact** | Blocks pod from starting | Pod may restart |
| **Resource Limits** | Highest init container used | Sum of all containers |
| **Restart Policy** | Always retries on failure | Follows pod policy |

---

## 🔧 Init Container Use Cases

### 1. **Wait for Dependencies**

```yaml
initContainers:
- name: wait-for-db
  image: busybox
  command: ['sh', '-c', 'until nslookup mysql-service; do echo waiting for mysql; sleep 2; done']
```

**Use Case:** Wait for database service to be available before starting app

### 2. **Download Configuration**

```yaml
initContainers:
- name: fetch-config
  image: curlimages/curl
  command: ['sh', '-c', 'curl -o /config/app.conf https://config-server/app.conf']
  volumeMounts:
  - name: config-volume
    mountPath: /config
```

**Use Case:** Download config files from external sources

### 3. **Generate Certificates**

```yaml
initContainers:
- name: generate-certs
  image: alpine/openssl
  command: ['sh', '-c', 'openssl req -x509 -newkey rsa:4096 -keyout /certs/key.pem -out /certs/cert.pem -days 365 -nodes']
  volumeMounts:
  - name: cert-volume
    mountPath: /certs
```

**Use Case:** Generate SSL/TLS certificates before app starts

### 4. **Database Migration**

```yaml
initContainers:
- name: db-migrate
  image: myapp-migrate:latest
  command: ['python', 'manage.py', 'migrate']
  env:
  - name: DATABASE_URL
    value: "postgresql://db:5432/myapp"
```

**Use Case:** Run database migrations before app deployment

### 5. **Clone Git Repository**

```yaml
initContainers:
- name: git-clone
  image: alpine/git
  command: ['sh', '-c', 'git clone https://github.com/user/repo.git /data']
  volumeMounts:
  - name: data-volume
    mountPath: /data
```

**Use Case:** Clone code or data from Git repository

### 6. **File Permission Setup**

```yaml
initContainers:
- name: fix-permissions
  image: busybox
  command: ['sh', '-c', 'chown -R 1000:1000 /data && chmod -R 755 /data']
  volumeMounts:
  - name: data-volume
    mountPath: /data
```

**Use Case:** Fix file ownership and permissions

---

## 🛠️ Task Implementation

### Understanding Today's Scenario

**Objective:** Use init container to write a welcome message that the main container reads continuously.

**Flow:**
1. Init container starts first
2. Writes message to `/ic/news` file in shared volume
3. Init container completes and exits
4. Main container starts
5. Main container reads and displays message every 5 seconds

### Step 1: Create the Deployment with Init Container

**Complete Deployment YAML:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ic-deploy-xfusion
  labels:
    app: ic-xfusion
spec:
  replicas: 1
  selector:
    matchLabels:
      app: ic-xfusion
  template:
    metadata:
      labels:
        app: ic-xfusion
    spec:
      # Init Containers - Run first, sequentially
      initContainers:
      - name: ic-msg-xfusion
        image: fedora:latest
        command: ['/bin/bash', '-c', 'echo Init Done - Welcome to xFusionCorp Industries > /ic/news']
        volumeMounts:
        - name: ic-volume-xfusion
          mountPath: /ic
      
      # Main Containers - Run after init containers succeed
      containers:
      - name: ic-main-xfusion
        image: fedora:latest
        command: ['/bin/bash', '-c', 'while true; do cat /ic/news; sleep 5; done']
        volumeMounts:
        - name: ic-volume-xfusion
          mountPath: /ic
      
      # Shared Volume - Accessible by all containers
      volumes:
      - name: ic-volume-xfusion
        emptyDir: {}
```

**Understanding Each Component:**

**1. Deployment Metadata:**
```yaml
metadata:
  name: ic-deploy-xfusion
  labels:
    app: ic-xfusion
```
- Deployment name: `ic-deploy-xfusion`
- Label for identification: `ic-xfusion`

**2. Deployment Spec:**
```yaml
spec:
  replicas: 1
  selector:
    matchLabels:
      app: ic-xfusion
```
- Creates 1 pod replica
- Selector matches pods with label `app: ic-xfusion`

**3. Init Container:**
```yaml
initContainers:
- name: ic-msg-xfusion
  image: fedora:latest
  command: ['/bin/bash', '-c', 'echo Init Done - Welcome to xFusionCorp Industries > /ic/news']
  volumeMounts:
  - name: ic-volume-xfusion
    mountPath: /ic
```

**Breakdown:**
- **name:** `ic-msg-xfusion` (init container identifier)
- **image:** `fedora:latest` (base Linux image)
- **command:** Writes welcome message to `/ic/news` file
- **volumeMount:** Mounts shared volume at `/ic` directory
- **Purpose:** Setup task (create config file) before app starts

**4. Main Container:**
```yaml
containers:
- name: ic-main-xfusion
  image: fedora:latest
  command: ['/bin/bash', '-c', 'while true; do cat /ic/news; sleep 5; done']
  volumeMounts:
  - name: ic-volume-xfusion
    mountPath: /ic
```

**Breakdown:**
- **name:** `ic-main-xfusion` (main app container)
- **image:** `fedora:latest` (same base image)
- **command:** Infinite loop that reads and displays `/ic/news` every 5 seconds
- **volumeMount:** Mounts same shared volume at `/ic`
- **Purpose:** Application container that uses init container's output

**5. Shared Volume:**
```yaml
volumes:
- name: ic-volume-xfusion
  emptyDir: {}
```

**Breakdown:**
- **type:** `emptyDir` (temporary storage)
- **lifecycle:** Created when pod starts, deleted when pod terminates
- **scope:** Shared across all containers in the pod
- **purpose:** Data exchange between init and main containers

---

### Step 2: Create the Deployment

**Method 1: Using YAML File (Recommended)**

```bash
# Create deployment YAML file
cat > ic-deploy-xfusion.yaml <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ic-deploy-xfusion
  labels:
    app: ic-xfusion
spec:
  replicas: 1
  selector:
    matchLabels:
      app: ic-xfusion
  template:
    metadata:
      labels:
        app: ic-xfusion
    spec:
      initContainers:
      - name: ic-msg-xfusion
        image: fedora:latest
        command: ['/bin/bash', '-c', 'echo Init Done - Welcome to xFusionCorp Industries > /ic/news']
        volumeMounts:
        - name: ic-volume-xfusion
          mountPath: /ic
      containers:
      - name: ic-main-xfusion
        image: fedora:latest
        command: ['/bin/bash', '-c', 'while true; do cat /ic/news; sleep 5; done']
        volumeMounts:
        - name: ic-volume-xfusion
          mountPath: /ic
      volumes:
      - name: ic-volume-xfusion
        emptyDir: {}
EOF

# Apply the deployment
kubectl apply -f ic-deploy-xfusion.yaml
```

**Method 2: Direct kubectl Command (One-liner)**

```bash
kubectl create deployment ic-deploy-xfusion --image=fedora:latest --dry-run=client -o yaml | \
kubectl set initcontainers - ic-msg-xfusion=fedora:latest --dry-run=client -o yaml | \
kubectl apply -f -

# Note: This method is complex for init containers, YAML file is preferred
```

---

### Step 3: Verify Deployment

**1. Check Deployment Status:**

```bash
kubectl get deployment ic-deploy-xfusion
```

**Expected Output:**
```
NAME                 READY   UP-TO-DATE   AVAILABLE   AGE
ic-deploy-xfusion    1/1     1            1           30s
```

**Status Indicators:**
- **READY: 1/1** → 1 out of 1 replicas are running
- **UP-TO-DATE: 1** → Latest pod spec applied
- **AVAILABLE: 1** → Pod is ready to serve

---

**2. Check Pod Status:**

```bash
kubectl get pods -l app=ic-xfusion
```

**Expected Output:**
```
NAME                                 READY   STATUS    RESTARTS   AGE
ic-deploy-xfusion-7b8c9d5f6b-x2k4p   1/1     Running   0          45s
```

**During Init Container Phase (if you catch it):**
```
NAME                                 READY   STATUS     RESTARTS   AGE
ic-deploy-xfusion-7b8c9d5f6b-x2k4p   0/1     Init:0/1   0          5s
```

**Status Meanings:**
- **Init:0/1** → 0 out of 1 init containers completed
- **Running** → Init containers completed, main container running
- **READY: 1/1** → 1 out of 1 containers ready

---

**3. Describe the Pod (Detailed View):**

```bash
# Get pod name
POD_NAME=$(kubectl get pods -l app=ic-xfusion -o jsonpath='{.items[0].metadata.name}')

# Describe pod
kubectl describe pod $POD_NAME
```

**Key Sections to Review:**

```yaml
Init Containers:
  ic-msg-xfusion:
    Container ID:  containerd://abc123...
    Image:         fedora:latest
    Command:
      /bin/bash
      -c
      echo Init Done - Welcome to xFusionCorp Industries > /ic/news
    State:          Terminated
      Reason:       Completed
      Exit Code:    0
    Mounts:
      /ic from ic-volume-xfusion (rw)

Containers:
  ic-main-xfusion:
    Container ID:  containerd://def456...
    Image:         fedora:latest
    Command:
      /bin/bash
      -c
      while true; do cat /ic/news; sleep 5; done
    State:          Running
    Ready:          True
    Mounts:
      /ic from ic-volume-xfusion (rw)

Volumes:
  ic-volume-xfusion:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)

Events:
  Type     Reason     Age   Message
  ----     ------     ----  -------
  Normal   Scheduled  1m    Successfully assigned default/ic-deploy-xfusion-xxx
  Normal   Pulling    1m    Pulling image "fedora:latest"
  Normal   Pulled     50s   Successfully pulled image "fedora:latest"
  Normal   Created    50s   Created container ic-msg-xfusion
  Normal   Started    50s   Started container ic-msg-xfusion
  Normal   Pulled     45s   Container image "fedora:latest" already present
  Normal   Created    45s   Created container ic-main-xfusion
  Normal   Started    45s   Started container ic-main-xfusion
```

**Event Timeline:**
1. **Scheduled** → Pod assigned to node
2. **Pulling** → Download init container image
3. **Pulled** → Init container image downloaded
4. **Created** → Init container created
5. **Started** → Init container started (and completed)
6. **Pulled** → Main container image (already cached)
7. **Created** → Main container created
8. **Started** → Main container started

---

**4. Check Init Container Logs:**

```bash
# View init container logs
kubectl logs $POD_NAME -c ic-msg-xfusion
```

**Expected Output:**
```
(Empty output - the command redirects to file, no stdout)
```

**Note:** The init container's command writes to a file (`> /ic/news`), so there's no console output.

---

**5. Check Main Container Logs:**

```bash
# View main container logs
kubectl logs $POD_NAME -c ic-main-xfusion
```

**Expected Output:**
```
Init Done - Welcome to xFusionCorp Industries
Init Done - Welcome to xFusionCorp Industries
Init Done - Welcome to xFusionCorp Industries
Init Done - Welcome to xFusionCorp Industries
...
```

**What's Happening:**
- Main container reads `/ic/news` every 5 seconds
- Displays the message written by init container
- Runs in infinite loop

**Follow Logs in Real-Time:**
```bash
# Watch logs continuously
kubectl logs -f $POD_NAME -c ic-main-xfusion

# Press Ctrl+C to stop watching
```

---

**6. Verify File Inside Container:**

```bash
# Exec into main container
kubectl exec -it $POD_NAME -c ic-main-xfusion -- bash

# Inside container:
cat /ic/news
# Output: Init Done - Welcome to xFusionCorp Industries

# Check file details
ls -lh /ic/news
# -rw-r--r-- 1 root root 49 Jan 5 12:34 /ic/news

# Exit container
exit
```

---

**7. Check Resource Usage:**

```bash
# View pod resource usage
kubectl top pod $POD_NAME

# Expected output:
# NAME                                 CPU(cores)   MEMORY(bytes)
# ic-deploy-xfusion-7b8c9d5f6b-x2k4p   1m           25Mi
```

---

### Step 4: Understanding the Volume Mount

**Volume Mount Diagram:**

```
┌─────────────────────────────────────────────────────────────┐
│                         Pod                                  │
│                                                              │
│  ┌────────────────────────────────────────────────────────┐ │
│  │ Init Container: ic-msg-xfusion                         │ │
│  │                                                        │ │
│  │  /ic/news ← writes message                            │ │
│  │     ↓                                                  │ │
│  │  [Mount: /ic]                                          │ │
│  └──────────────┬─────────────────────────────────────────┘ │
│                 │                                            │
│                 ↓                                            │
│  ┌─────────────────────────────────────────────────────────┐│
│  │         Volume: ic-volume-xfusion (emptyDir)            ││
│  │                 /ic/news file                           ││
│  └─────────────────────────────────────────────────────────┘│
│                 ↓                                            │
│  ┌──────────────┴─────────────────────────────────────────┐ │
│  │ Main Container: ic-main-xfusion                        │ │
│  │                                                        │ │
│  │  [Mount: /ic]                                          │ │
│  │     ↓                                                  │ │
│  │  /ic/news ← reads message every 5s                    │ │
│  └────────────────────────────────────────────────────────┘ │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

**Key Points:**
- Both containers mount the **same volume** at the **same path** (`/ic`)
- Init container **writes** to `/ic/news`
- Main container **reads** from `/ic/news`
- Data persists in emptyDir for pod's lifetime
- If pod restarts, emptyDir is recreated (data lost)

---

## 🔄 Init Container Execution Flow

### Detailed Timeline

```
┌─────────────────────────────────────────────────────────────┐
│                    Pod Startup Timeline                      │
└─────────────────────────────────────────────────────────────┘

Time: T+0s
  ↓
┌─────────────────────────────────────────────────────────────┐
│ 1. Pod Scheduled to Node                                    │
│    - Kubernetes finds suitable node                         │
│    - Pod assigned to node                                   │
└─────────────────────────────────────────────────────────────┘
  ↓
Time: T+1s
  ↓
┌─────────────────────────────────────────────────────────────┐
│ 2. Pull Init Container Image                               │
│    - Download fedora:latest (if not cached)                │
│    - Image layers downloaded                                │
└─────────────────────────────────────────────────────────────┘
  ↓
Time: T+10s (depends on image size)
  ↓
┌─────────────────────────────────────────────────────────────┐
│ 3. Create emptyDir Volume                                   │
│    - Create temporary directory on node                     │
│    - Mount to init container at /ic                         │
└─────────────────────────────────────────────────────────────┘
  ↓
Time: T+11s
  ↓
┌─────────────────────────────────────────────────────────────┐
│ 4. Start Init Container: ic-msg-xfusion                    │
│    - Run: echo Init Done... > /ic/news                     │
│    - File created in emptyDir                               │
│    - Command completes                                      │
│    - Container exits with code 0 (success)                  │
│    Status: Init:0/1 → Completed                            │
└─────────────────────────────────────────────────────────────┘
  ↓
Time: T+12s
  ↓
┌─────────────────────────────────────────────────────────────┐
│ 5. Pull Main Container Image                               │
│    - Image already cached (same as init)                   │
│    - Skip download                                          │
└─────────────────────────────────────────────────────────────┘
  ↓
Time: T+13s
  ↓
┌─────────────────────────────────────────────────────────────┐
│ 6. Start Main Container: ic-main-xfusion                   │
│    - Mount same emptyDir volume at /ic                     │
│    - Run: while true; do cat /ic/news; sleep 5; done      │
│    - Read file created by init container                    │
│    - Display message every 5 seconds                        │
│    Status: Running, Ready: 1/1                             │
└─────────────────────────────────────────────────────────────┘
  ↓
Time: T+14s onwards
  ↓
┌─────────────────────────────────────────────────────────────┐
│ 7. Pod Running                                              │
│    - Main container outputs message continuously            │
│    - Pod remains running until deleted                      │
└─────────────────────────────────────────────────────────────┘
```

---

## 🐛 Common Issues and Troubleshooting

### Issue 1: Init Container Stuck in Pending

**Symptoms:**
```bash
kubectl get pods -l app=ic-xfusion
# NAME                                 READY   STATUS    RESTARTS   AGE
# ic-deploy-xfusion-xxx                0/1     Pending   0          2m
```

**Root Causes:**

**A. Image Pull Failure**
```bash
kubectl describe pod $POD_NAME
# Events:
#   Failed to pull image "fedora:latest": ... connection timeout
```

**Solution:**
```bash
# Check if image exists and is accessible
docker pull fedora:latest

# Or use alternative image
kubectl set image deployment/ic-deploy-xfusion \
  ic-msg-xfusion=alpine:latest \
  ic-main-xfusion=alpine:latest

# Update commands for alpine (uses sh instead of bash)
```

**B. Insufficient Resources**
```bash
kubectl describe pod $POD_NAME
# Events:
#   Warning  FailedScheduling  Pod  0/3 nodes are available: insufficient memory
```

**Solution:**
```bash
# Add resource limits to deployment
kubectl edit deployment ic-deploy-xfusion

# Add under each container:
resources:
  requests:
    memory: "128Mi"
    cpu: "100m"
  limits:
    memory: "256Mi"
    cpu: "200m"
```

---

### Issue 2: Init Container Fails (Exit Code != 0)

**Symptoms:**
```bash
kubectl get pods -l app=ic-xfusion
# NAME                                 READY   STATUS                  RESTARTS   AGE
# ic-deploy-xfusion-xxx                0/1     Init:Error              0          1m
# ic-deploy-xfusion-xxx                0/1     Init:CrashLoopBackOff   3          2m
```

**Diagnosis:**
```bash
# Check init container logs
kubectl logs $POD_NAME -c ic-msg-xfusion

# Describe pod for events
kubectl describe pod $POD_NAME
```

**Common Causes:**

**A. Command Syntax Error**
```yaml
# ❌ Wrong:
command: ['/bin/bash', '-c', 'echo test > /ic/news && exit 1']  # Forces error

# ✅ Correct:
command: ['/bin/bash', '-c', 'echo test > /ic/news']  # Exits 0 on success
```

**B. Volume Mount Issues**
```yaml
# ❌ Wrong: volume doesn't exist
volumeMounts:
- name: wrong-volume-name
  mountPath: /ic

# ✅ Correct: matches volume name
volumeMounts:
- name: ic-volume-xfusion
  mountPath: /ic
```

**C. Permission Denied**
```bash
# Check logs
kubectl logs $POD_NAME -c ic-msg-xfusion
# /bin/bash: /ic/news: Permission denied
```

**Solution:**
```yaml
# Add securityContext to init container
initContainers:
- name: ic-msg-xfusion
  image: fedora:latest
  command: ['/bin/bash', '-c', 'echo test > /ic/news']
  securityContext:
    runAsUser: 0  # Run as root
  volumeMounts:
  - name: ic-volume-xfusion
    mountPath: /ic
```

---

### Issue 3: Main Container Can't Read File

**Symptoms:**
```bash
kubectl logs $POD_NAME -c ic-main-xfusion
# cat: /ic/news: No such file or directory
```

**Root Causes:**

**A. Init Container Didn't Complete Successfully**
```bash
# Check init container status
kubectl describe pod $POD_NAME | grep -A10 "Init Containers"
# State: Terminated
# Reason: Error  ← Init container failed
```

**Solution:** Fix init container error (see Issue 2)

**B. Different Volume Mounts**
```yaml
# ❌ Wrong: different volumes
initContainers:
- volumeMounts:
  - name: init-volume
    mountPath: /ic

containers:
- volumeMounts:
  - name: app-volume  # Different volume!
    mountPath: /ic

# ✅ Correct: same volume
# Both use: ic-volume-xfusion
```

**C. Wrong File Path**
```yaml
# ❌ Wrong: init writes to /ic/news, main reads /ic/message
initContainers:
- command: ['sh', '-c', 'echo test > /ic/news']

containers:
- command: ['sh', '-c', 'cat /ic/message']  # Wrong path!

# ✅ Correct: same path in both
```

---

### Issue 4: Pod Restart Loop

**Symptoms:**
```bash
kubectl get pods -l app=ic-xfusion
# NAME                                 READY   STATUS             RESTARTS   AGE
# ic-deploy-xfusion-xxx                0/1     CrashLoopBackOff   5          3m
```

**Root Cause:** Main container command exits/fails

```bash
# Check main container logs
kubectl logs $POD_NAME -c ic-main-xfusion

# Check for:
# - Command not found
# - File not readable
# - Syntax errors
```

**Common Issues:**

**A. Command Exits Immediately**
```yaml
# ❌ Wrong: command exits after one read
command: ['/bin/bash', '-c', 'cat /ic/news']  # Exits after one cat

# ✅ Correct: infinite loop keeps container running
command: ['/bin/bash', '-c', 'while true; do cat /ic/news; sleep 5; done']
```

**B. Bash Not Available in Image**
```yaml
# ❌ Wrong: alpine doesn't have bash by default
image: alpine:latest
command: ['/bin/bash', '-c', '...']  # bash not found

# ✅ Correct: use sh for alpine
image: alpine:latest
command: ['/bin/sh', '-c', '...']  # sh available
```

---

### Issue 5: Init Container Runs Every Restart

**Behavior:** Init container re-runs when pod restarts

**This is expected!**

```bash
# Delete pod (deployment recreates it)
kubectl delete pod $POD_NAME

# New pod runs init container again
kubectl get pods -l app=ic-xfusion -w
# ic-deploy-xfusion-new   0/1   Init:0/1   0   1s
# ic-deploy-xfusion-new   0/1   Init:0/1   0   2s
# ic-deploy-xfusion-new   1/1   Running    0   3s
```

**Why This Matters:**
- Init containers ALWAYS run on pod creation
- emptyDir is fresh for each pod
- Init container setup is re-executed
- Design init containers to be **idempotent** (safe to run multiple times)

**Best Practice:**
```yaml
# ❌ Risky: assumes file doesn't exist
command: ['sh', '-c', 'echo data >> /ic/news']  # Appends on each restart

# ✅ Safe: overwrites file
command: ['sh', '-c', 'echo data > /ic/news']  # Replaces on each restart
```

---

## 📚 Advanced Init Container Patterns

### Pattern 1: Multiple Init Containers (Sequential Setup)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-init-pod
spec:
  initContainers:
  # Step 1: Check database connectivity
  - name: check-db
    image: busybox
    command: ['sh', '-c', 'until nslookup mysql-service; do echo waiting for db; sleep 2; done']
  
  # Step 2: Download configuration
  - name: fetch-config
    image: curlimages/curl
    command: ['sh', '-c', 'curl -o /config/app.yaml https://config-server/app.yaml']
    volumeMounts:
    - name: config
      mountPath: /config
  
  # Step 3: Validate configuration
  - name: validate-config
    image: myapp-validator:latest
    command: ['python', 'validate.py', '/config/app.yaml']
    volumeMounts:
    - name: config
      mountPath: /config
  
  containers:
  - name: app
    image: myapp:latest
    volumeMounts:
    - name: config
      mountPath: /config
  
  volumes:
  - name: config
    emptyDir: {}
```

**Execution Order:**
1. `check-db` runs → waits for database → completes
2. `fetch-config` runs → downloads config → completes
3. `validate-config` runs → validates config → completes
4. `app` container starts → uses validated config

---

### Pattern 2: Git Clone with Init Container

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: git-app-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: git-app
  template:
    metadata:
      labels:
        app: git-app
    spec:
      initContainers:
      - name: git-clone
        image: alpine/git
        command:
        - sh
        - -c
        - |
          git clone https://github.com/username/repo.git /data
          cd /data
          git checkout main
          echo "Cloned at: $(date)" > /data/.clone-info
        volumeMounts:
        - name: app-data
          mountPath: /data
      
      containers:
      - name: web-server
        image: nginx:alpine
        volumeMounts:
        - name: app-data
          mountPath: /usr/share/nginx/html
          readOnly: true
        ports:
        - containerPort: 80
      
      volumes:
      - name: app-data
        emptyDir: {}
```

**Use Case:** Serve static website from Git repository

---

### Pattern 3: Database Migration Init Container

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: django-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: django
  template:
    metadata:
      labels:
        app: django
    spec:
      initContainers:
      - name: db-migrate
        image: myapp:latest
        command:
        - python
        - manage.py
        - migrate
        - --noinput
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: url
        - name: DJANGO_SETTINGS_MODULE
          value: "myapp.settings.production"
      
      containers:
      - name: django-app
        image: myapp:latest
        command: ["gunicorn", "myapp.wsgi:application"]
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: url
        ports:
        - containerPort: 8000
```

**Use Case:** Run database migrations before starting web app

---

### Pattern 4: Certificate Generation

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secure-app
spec:
  initContainers:
  - name: generate-certs
    image: alpine/openssl
    command:
    - sh
    - -c
    - |
      # Generate private key
      openssl genrsa -out /certs/server.key 2048
      
      # Generate certificate signing request
      openssl req -new -key /certs/server.key \
        -out /certs/server.csr \
        -subj "/CN=myapp.example.com"
      
      # Generate self-signed certificate
      openssl x509 -req -days 365 \
        -in /certs/server.csr \
        -signkey /certs/server.key \
        -out /certs/server.crt
      
      # Set permissions
      chmod 644 /certs/server.crt
      chmod 600 /certs/server.key
      
      echo "Certificates generated successfully"
    volumeMounts:
    - name: certs
      mountPath: /certs
  
  containers:
  - name: app
    image: nginx:alpine
    volumeMounts:
    - name: certs
      mountPath: /etc/nginx/certs
      readOnly: true
    ports:
    - containerPort: 443
  
  volumes:
  - name: certs
    emptyDir: {}
```

**Use Case:** Generate SSL certificates before starting HTTPS server

---

### Pattern 5: Dependency Waiter (Service Readiness)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend-api
spec:
  replicas: 2
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      initContainers:
      # Wait for database
      - name: wait-db
        image: busybox:latest
        command:
        - sh
        - -c
        - |
          echo "Waiting for database..."
          until nc -z mysql-service 3306; do
            echo "Database not ready, waiting..."
            sleep 2
          done
          echo "Database is ready!"
      
      # Wait for Redis
      - name: wait-redis
        image: busybox:latest
        command:
        - sh
        - -c
        - |
          echo "Waiting for Redis..."
          until nc -z redis-service 6379; do
            echo "Redis not ready, waiting..."
            sleep 2
          done
          echo "Redis is ready!"
      
      # Wait for external API
      - name: wait-api
        image: curlimages/curl
        command:
        - sh
        - -c
        - |
          echo "Waiting for external API..."
          until curl -f http://api.example.com/health; do
            echo "API not ready, waiting..."
            sleep 5
          done
          echo "API is ready!"
      
      containers:
      - name: backend
        image: mybackend:latest
        env:
        - name: DB_HOST
          value: mysql-service
        - name: REDIS_HOST
          value: redis-service
        - name: API_URL
          value: http://api.example.com
        ports:
        - containerPort: 8080
```

**Use Case:** Ensure all dependencies are available before starting app

---

## 🎯 Best Practices

### 1. Init Container Design

```yaml
# ✅ Good: Simple, focused, idempotent
initContainers:
- name: setup-config
  image: busybox
  command: ['sh', '-c', 'echo "production" > /config/env']  # Overwrites

# ❌ Bad: Complex, stateful, non-idempotent
initContainers:
- name: complex-setup
  image: my-bloated-image:latest
  command: ['sh', '-c', 'if [ ! -f /config/env ]; then echo "production" >> /config/env; fi']  # Conditional logic
```

**Principles:**
- Keep init containers **simple** and **focused**
- Make them **idempotent** (safe to run multiple times)
- Use **lightweight images** (busybox, alpine)
- Avoid **stateful logic**

---

### 2. Error Handling

```yaml
# ✅ Good: Clear error messages
initContainers:
- name: check-db
  image: busybox
  command:
  - sh
  - -c
  - |
    echo "Checking database connectivity..."
    if ! nc -z db-service 5432; then
      echo "ERROR: Cannot connect to database at db-service:5432"
      exit 1
    fi
    echo "Database connection successful"

# ❌ Bad: Silent failures
initContainers:
- name: check-db
  image: busybox
  command: ['nc', '-z', 'db-service', '5432']  # No error context
```

---

### 3. Resource Limits

```yaml
# ✅ Good: Set resource limits for init containers
initContainers:
- name: setup
  image: busybox
  resources:
    requests:
      memory: "64Mi"
      cpu: "50m"
    limits:
      memory: "128Mi"
      cpu: "100m"

# Note: Pod's effective resource is the MAXIMUM of:
# - Sum of all main containers
# - Highest init container resource
```

---

### 4. Volume Strategy

```yaml
# ✅ Good: Use emptyDir for temporary init data
volumes:
- name: init-data
  emptyDir: {}

# ✅ Good: Use ConfigMap for read-only config
volumes:
- name: config
  configMap:
    name: app-config

# ⚠️ Careful: hostPath ties pod to specific node
volumes:
- name: host-data
  hostPath:
    path: /mnt/data
    type: Directory

# ❌ Bad: PVC for init-only data (waste of persistent storage)
volumes:
- name: init-data
  persistentVolumeClaim:
    claimName: init-pvc  # Unnecessary persistence
```

---

### 5. Security

```yaml
# ✅ Good: Run as non-root when possible
initContainers:
- name: setup
  image: busybox
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    readOnlyRootFilesystem: true
  volumeMounts:
  - name: data
    mountPath: /data

# ✅ Good: Drop unnecessary capabilities
initContainers:
- name: network-check
  image: busybox
  securityContext:
    capabilities:
      drop:
      - ALL
      add:
      - NET_RAW  # Only add what's needed
```

---

### 6. Logging and Debugging

```yaml
# ✅ Good: Verbose logging for debugging
initContainers:
- name: setup
  image: busybox
  command:
  - sh
  - -c
  - |
    set -x  # Enable command tracing
    echo "Starting setup at $(date)"
    echo "Current directory: $(pwd)"
    echo "Available disk space:"
    df -h
    echo "Running setup command..."
    # actual setup command
    echo "Setup complete at $(date)"
```

---

### 7. Timeout Configuration

```yaml
# ✅ Good: Set activeDeadlineSeconds for long-running init containers
apiVersion: v1
kind: Pod
metadata:
  name: app-with-timeout
spec:
  activeDeadlineSeconds: 300  # Pod fails if not ready in 5 minutes
  
  initContainers:
  - name: slow-setup
    image: mysetup:latest
    command: ['sh', '-c', 'sleep 60 && echo done']
```

---

## 📋 Complete Verification Checklist

```bash
#!/bin/bash

echo "🔍 Init Container Deployment - Complete Verification"
echo "====================================================="

# Colors
GREEN='\033[0;32m'
RED='\033[0;31m'
YELLOW='\033[1;33m'
NC='\033[0m'

# 1. Check Deployment
echo -e "\n${YELLOW}1️⃣ Checking Deployment...${NC}"
if kubectl get deployment ic-deploy-xfusion &>/dev/null; then
    READY=$(kubectl get deployment ic-deploy-xfusion -o jsonpath='{.status.readyReplicas}')
    DESIRED=$(kubectl get deployment ic-deploy-xfusion -o jsonpath='{.spec.replicas}')
    if [ "$READY" == "$DESIRED" ]; then
        echo -e "${GREEN}✅ Deployment ic-deploy-xfusion: $READY/$DESIRED ready${NC}"
    else
        echo -e "${RED}❌ Deployment ic-deploy-xfusion: $READY/$DESIRED ready${NC}"
    fi
else
    echo -e "${RED}❌ Deployment ic-deploy-xfusion not found${NC}"
    exit 1
fi

# 2. Check Pod
echo -e "\n${YELLOW}2️⃣ Checking Pod Status...${NC}"
POD_NAME=$(kubectl get pods -l app=ic-xfusion -o jsonpath='{.items[0].metadata.name}')
if [ -z "$POD_NAME" ]; then
    echo -e "${RED}❌ No pods found with label app=ic-xfusion${NC}"
    exit 1
fi

POD_STATUS=$(kubectl get pod $POD_NAME -o jsonpath='{.status.phase}')
POD_READY=$(kubectl get pod $POD_NAME -o jsonpath='{.status.containerStatuses[0].ready}')

if [ "$POD_STATUS" == "Running" ] && [ "$POD_READY" == "true" ]; then
    echo -e "${GREEN}✅ Pod $POD_NAME: Running and Ready${NC}"
else
    echo -e "${RED}❌ Pod $POD_NAME: Status=$POD_STATUS, Ready=$POD_READY${NC}"
fi

# 3. Check Init Container
echo -e "\n${YELLOW}3️⃣ Checking Init Container...${NC}"
INIT_STATE=$(kubectl get pod $POD_NAME -o jsonpath='{.status.initContainerStatuses[0].state.terminated.reason}')
INIT_EXIT=$(kubectl get pod $POD_NAME -o jsonpath='{.status.initContainerStatuses[0].state.terminated.exitCode}')

if [ "$INIT_STATE" == "Completed" ] && [ "$INIT_EXIT" == "0" ]; then
    echo -e "${GREEN}✅ Init Container ic-msg-xfusion: Completed (Exit Code: 0)${NC}"
else
    echo -e "${RED}❌ Init Container ic-msg-xfusion: $INIT_STATE (Exit Code: $INIT_EXIT)${NC}"
fi

# 4. Check Main Container Logs
echo -e "\n${YELLOW}4️⃣ Checking Main Container Logs...${NC}"
LOG_OUTPUT=$(kubectl logs $POD_NAME -c ic-main-xfusion --tail=1 2>/dev/null)
EXPECTED_MESSAGE="Init Done - Welcome to xFusionCorp Industries"

if [ "$LOG_OUTPUT" == "$EXPECTED_MESSAGE" ]; then
    echo -e "${GREEN}✅ Main Container Logs: Correct message${NC}"
    echo "   Message: $LOG_OUTPUT"
else
    echo -e "${RED}❌ Main Container Logs: Incorrect message${NC}"
    echo "   Expected: $EXPECTED_MESSAGE"
    echo "   Got: $LOG_OUTPUT"
fi

# 5. Verify File Inside Container
echo -e "\n${YELLOW}5️⃣ Verifying File in Container...${NC}"
FILE_CONTENT=$(kubectl exec $POD_NAME -c ic-main-xfusion -- cat /ic/news 2>/dev/null)

if [ "$FILE_CONTENT" == "$EXPECTED_MESSAGE" ]; then
    echo -e "${GREEN}✅ File /ic/news: Correct content${NC}"
else
    echo -e "${RED}❌ File /ic/news: Incorrect content${NC}"
    echo "   Expected: $EXPECTED_MESSAGE"
    echo "   Got: $FILE_CONTENT"
fi

# 6. Check Volume Mount
echo -e "\n${YELLOW}6️⃣ Checking Volume Mount...${NC}"
VOLUME_NAME=$(kubectl get pod $POD_NAME -o jsonpath='{.spec.volumes[0].name}')
MOUNT_PATH=$(kubectl get pod $POD_NAME -o jsonpath='{.spec.containers[0].volumeMounts[0].mountPath}')

if [ "$VOLUME_NAME" == "ic-volume-xfusion" ] && [ "$MOUNT_PATH" == "/ic" ]; then
    echo -e "${GREEN}✅ Volume Mount: Correct (volume: $VOLUME_NAME, path: $MOUNT_PATH)${NC}"
else
    echo -e "${RED}❌ Volume Mount: Incorrect${NC}"
    echo "   Volume: $VOLUME_NAME (expected: ic-volume-xfusion)"
    echo "   Mount Path: $MOUNT_PATH (expected: /ic)"
fi

# 7. Summary
echo -e "\n${YELLOW}====================================================${NC}"
echo -e "${YELLOW}📊 Summary:${NC}"
kubectl get deployment,pods -l app=ic-xfusion

echo -e "\n${GREEN}✅ Task Complete! All checks passed.${NC}"
```

**Save and run:**
```bash
chmod +x verify-init-containers.sh
./verify-init-containers.sh
```

---

## 🧹 Cleanup

```bash
# Delete deployment (also deletes pods)
kubectl delete deployment ic-deploy-xfusion

# Verify deletion
kubectl get deployment,pods -l app=ic-xfusion
# No resources found

# If you created YAML files
rm -f ic-deploy-xfusion.yaml verify-init-containers.sh
```

---

## 📖 Key Takeaways

### Concepts Mastered Today:

1. ✅ **Init Containers**
   - Run before main containers
   - Execute sequentially
   - Must complete successfully
   - Can have different images from app

2. ✅ **Container Communication**
   - Shared volumes (emptyDir)
   - Data exchange between init and main containers
   - Volume mount paths

3. ✅ **Use Cases**
   - Configuration setup
   - Dependency checking
   - Data preparation
   - File generation

4. ✅ **Execution Flow**
   - Pod scheduling
   - Init container sequence
   - Main container startup
   - Error handling

5. ✅ **Best Practices**
   - Idempotent init containers
   - Lightweight images
   - Clear error messages
   - Resource limits
   - Security context

---

## 🎯 Real-World Scenarios

| Scenario | Init Container Task | Main Container Task |
|----------|-------------------|-------------------|
| **Web App** | Download config from S3 | Serve web application |
| **Database App** | Run migrations | Start database connections |
| **ML Service** | Download model files | Serve predictions |
| **API Gateway** | Validate upstream services | Proxy requests |
| **Monitoring** | Setup Prometheus config | Collect metrics |
| **Data Pipeline** | Download datasets | Process data |

---

## 📚 Additional Resources

**Official Documentation:**
- [Init Containers](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/)
- [Pod Lifecycle](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/)
- [emptyDir Volumes](https://kubernetes.io/docs/concepts/storage/volumes/#emptydir)

**Next Steps:**
- **Day 62:** StatefulSets with Init Containers
- **Day 63:** ConfigMaps and Secrets with Init Containers
- **Day 64:** Advanced multi-stage init patterns
- **Day 65:** Init containers with persistent volumes

---

## ✅ Task Completion Checklist

- [ ] Deployment `ic-deploy-xfusion` created
- [ ] Replica count set to 1
- [ ] Labels configured correctly (`app: ic-xfusion`)
- [ ] Init container `ic-msg-xfusion` using `fedora:latest`
- [ ] Init container writes message to `/ic/news`
- [ ] Main container `ic-main-xfusion` using `fedora:latest`
- [ ] Main container reads and displays message every 5 seconds
- [ ] Volume `ic-volume-xfusion` configured as emptyDir
- [ ] Both containers mount volume at `/ic`
- [ ] Pod reaches Running state
- [ ] Main container logs show correct message
- [ ] All verification checks pass

---

**🎉 Congratulations!** You've successfully implemented Init Containers in Kubernetes! You now understand how to perform pre-deployment tasks and share data between init and main containers using shared volumes.

**Day 61 Status:** ✅ Complete

**Next:** Day 62 - StatefulSets and Persistent Applications 🚀
