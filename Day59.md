# Day 59: Kubernetes Troubleshooting - Debugging Redis Deployment

**100 Days Cloud DevOps Challenge – KodeKloud**

---

## 📋 Task Overview

**Scenario:** A Redis deployment that was working fine went down after a team member made changes. Need to identify and fix the issue quickly.

**Environment:**
- **Cluster:** Kubernetes on jump_host
- **Deployment Name:** `redis-deployment`
- **Status:** Pods not running
- **Goal:** Identify the problem and restore the application

---

## 🎯 Learning Objectives

By the end of this task, you will:
- ✅ Master systematic troubleshooting methodology
- ✅ Use kubectl describe, logs, and get events effectively
- ✅ Identify common deployment configuration errors
- ✅ Fix image pull errors, configuration mistakes, and resource issues
- ✅ Understand Redis deployment on Kubernetes
- ✅ Apply production debugging techniques

---

## 📚 Understanding Redis on Kubernetes

### What is Redis?

**Redis (Remote Dictionary Server)** is an open-source, in-memory data structure store used as:
- **Cache:** Fast data retrieval
- **Database:** Key-value storage
- **Message Broker:** Pub/sub patterns
- **Session Store:** Web application sessions

**Key Features:**
- In-memory storage (extremely fast)
- Persistence options (RDB snapshots, AOF logs)
- Multiple data structures (strings, lists, sets, hashes, sorted sets)
- Replication and clustering
- Default port: 6379

### Redis Deployment Patterns

**Standalone Redis:**
```yaml
- Single pod deployment
- No replication
- Good for: Development, caching, non-critical data
```

**Redis Master-Slave:**
```yaml
- One master (read/write)
- Multiple replicas (read-only)
- Good for: Read-heavy workloads, high availability
```

**Redis Cluster:**
```yaml
- Multiple masters (sharding)
- Each master has replicas
- Good for: Large datasets, horizontal scaling
```

**Redis Sentinel:**
```yaml
- Monitoring and automatic failover
- Multiple sentinel processes
- Good for: High availability, automatic recovery
```

---

## 🔍 Systematic Troubleshooting Methodology

### The 5-Step Debugging Process

```
1. OBSERVE → What's the current state?
2. IDENTIFY → What's the problem?
3. ANALYZE → Why did it happen?
4. FIX → How to resolve it?
5. VERIFY → Is it working now?
```

### Troubleshooting Command Hierarchy

```
Level 1: Overview
├── kubectl get deployments
├── kubectl get pods
└── kubectl get events

Level 2: Details
├── kubectl describe deployment <name>
├── kubectl describe pod <name>
└── kubectl get pod <name> -o yaml

Level 3: Deep Dive
├── kubectl logs <pod>
├── kubectl logs <pod> --previous
└── kubectl exec -it <pod> -- /bin/sh

Level 4: System Level
├── kubectl get nodes
├── kubectl describe node <name>
└── kubectl top nodes/pods
```

---

## 🛠️ Step-by-Step Solution

### Phase 1: Initial Investigation

#### Step 1: Access the Cluster

```bash
# SSH to jump host (if needed)
ssh thor@jump_host

# Verify kubectl is configured
kubectl version --short
kubectl cluster-info
```

**Expected Output:**
```
Client Version: v1.28.0
Server Version: v1.28.0
Kubernetes control plane is running at https://...
```

---

#### Step 2: Check Deployment Status

```bash
# List all deployments
kubectl get deployments

# Specific deployment details
kubectl get deployment redis-deployment

# Wide output for more info
kubectl get deployment redis-deployment -o wide
```

**Expected Output (Problem State):**
```
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
redis-deployment   0/1     1            0           15m
```

**What This Tells Us:**
- `READY: 0/1` → None of the desired replicas are ready
- `AVAILABLE: 0` → No pods available to serve traffic
- Deployment exists but pods aren't running

---

#### Step 3: Check Pod Status

```bash
# List all pods
kubectl get pods

# Filter redis pods
kubectl get pods | grep redis

# Detailed pod status
kubectl get pods -l app=redis

# Wide output
kubectl get pods -o wide
```

**Common Problem States:**

| Status | Meaning | Likely Cause |
|--------|---------|--------------|
| `Pending` | Not scheduled | Resource constraints, node issues |
| `ImagePullBackOff` | Can't pull image | Wrong image name/tag, registry issue |
| `CrashLoopBackOff` | Container keeps crashing | App error, wrong command, missing config |
| `ErrImagePull` | Image pull failed | Image doesn't exist |
| `CreateContainerConfigError` | Config problem | ConfigMap/Secret missing |
| `RunContainerError` | Can't start container | Command error, permission issue |

**Expected Output (One of these):**
```
NAME                               READY   STATUS             RESTARTS   AGE
redis-deployment-7d6b5c8f9-abcde   0/1     ImagePullBackOff   0          5m
```

---

#### Step 4: Describe the Deployment

```bash
# Get detailed deployment information
kubectl describe deployment redis-deployment
```

**What to Look For:**

```yaml
# Check these sections:
1. Replicas: Desired: 1, Updated: 1, Available: 0  ← Mismatch indicates problem
2. StrategyType: RollingUpdate (check maxSurge/maxUnavailable)
3. Pod Template:
   - Labels: app=redis  ← Used for service selector
   - Containers:
     - Image: redis:latest  ← Check image name/tag
     - Port: 6379  ← Redis default port
     - Environment: Any env variables
     - Volume Mounts: Any persistent storage
4. Conditions:
   - Available: False  ← Why false?
   - Progressing: True/False
5. Events: ← CRITICAL - Shows what went wrong
   - Normal   ScalingReplicaSet  Scaled up to 1
   - Warning  FailedCreate        Error creating pod
```

**Common Issues Found Here:**
- ❌ Wrong image name: `redi:latest` instead of `redis:latest`
- ❌ Wrong tag: `redis:abc` (doesn't exist)
- ❌ Missing ports
- ❌ Wrong resource limits (exceeds node capacity)

---

#### Step 5: Describe the Pod

```bash
# Get pod name first
POD_NAME=$(kubectl get pods -l app=redis -o jsonpath='{.items[0].metadata.name}')

# Describe the pod
kubectl describe pod $POD_NAME

# Or directly
kubectl describe pod <pod-name>
```

**Critical Sections to Check:**

```yaml
# 1. Status: Shows current state
Status: Pending / Running / Failed

# 2. Containers: Configuration details
Containers:
  redis:
    Image:          redis:latest  ← Check spelling/tag
    Port:           6379/TCP
    State:          Waiting  ← Why waiting?
    Reason:         ImagePullBackOff  ← ROOT CAUSE
    Ready:          False

# 3. Conditions: Health checks
Conditions:
  Type              Status
  Initialized       True   ← Init containers completed
  Ready             False  ← Pod not ready to serve traffic
  ContainersReady   False  ← Containers not running
  PodScheduled      True   ← Pod assigned to node

# 4. Events: Timeline of what happened
Events:
  Type     Reason     Age                From               Message
  ----     ------     ----               ----               -------
  Normal   Scheduled  5m                 default-scheduler  Successfully assigned default/redis-...
  Normal   Pulling    3m (x4 over 5m)    kubelet            Pulling image "redi:latest"
  Warning  Failed     3m (x4 over 5m)    kubelet            Failed to pull image "redi:latest"
  Warning  Failed     3m (x4 over 5m)    kubelet            Error: ErrImagePull
  Normal   BackOff    2m (x6 over 5m)    kubelet            Back-off pulling image "redi:latest"
  Warning  Failed     2m (x6 over 5m)    kubelet            Error: ImagePullBackOff
```

**🔴 FOUND THE PROBLEM:**
```
Image name is "redi:latest" but should be "redis:latest"
Typo in image name!
```

---

#### Step 6: Check Events

```bash
# Get recent cluster events
kubectl get events --sort-by='.lastTimestamp'

# Filter for redis
kubectl get events --field-selector involvedObject.name=redis-deployment

# Last 30 events
kubectl get events --sort-by='.lastTimestamp' | tail -30
```

**Expected Output (Problem State):**
```
LAST SEEN   TYPE      REASON              OBJECT                                MESSAGE
2m          Warning   Failed              pod/redis-deployment-...              Failed to pull image "redi:latest": rpc error: code = NotFound desc = failed to pull and unpack image "docker.io/library/redi:latest": failed to resolve reference "docker.io/library/redi:latest": docker.io/library/redi:latest: not found
2m          Warning   Failed              pod/redis-deployment-...              Error: ErrImagePull
2m          Normal    BackOff             pod/redis-deployment-...              Back-off pulling image "redi:latest"
2m          Warning   Failed              pod/redis-deployment-...              Error: ImagePullBackOff
```

---

#### Step 7: Check Logs (If Pod Started)

```bash
# Current logs
kubectl logs $POD_NAME

# Previous container logs (if crashed)
kubectl logs $POD_NAME --previous

# Follow logs in real-time
kubectl logs -f $POD_NAME

# Logs from specific container (if multi-container pod)
kubectl logs $POD_NAME -c redis
```

**Note:** If pod is in `ImagePullBackOff`, there are no logs yet because the container never started.

---

### Phase 2: Identify the Issue

#### Common Issues and How to Spot Them

**Issue 1: Wrong Image Name (Typo)**

**Symptoms:**
```
Status: ImagePullBackOff / ErrImagePull
Events: Failed to pull image "redi:latest": not found
```

**How to Verify:**
```bash
# Check current image
kubectl get deployment redis-deployment -o jsonpath='{.spec.template.spec.containers[0].image}'

# Output: redi:latest  ← Should be "redis:latest"
```

**Fix:**
```bash
# Method 1: Set image directly
kubectl set image deployment/redis-deployment redis=redis:latest

# Method 2: Edit deployment
kubectl edit deployment redis-deployment
# Change line: image: redi:latest → image: redis:latest

# Method 3: Patch
kubectl patch deployment redis-deployment -p '{"spec":{"template":{"spec":{"containers":[{"name":"redis","image":"redis:latest"}]}}}}'
```

---

**Issue 2: Wrong Image Tag**

**Symptoms:**
```
Status: ImagePullBackOff
Events: manifest for redis:abc not found
```

**How to Verify:**
```bash
kubectl get deployment redis-deployment -o jsonpath='{.spec.template.spec.containers[0].image}'
# Output: redis:abc  ← Tag doesn't exist
```

**Fix:**
```bash
kubectl set image deployment/redis-deployment redis=redis:latest
# Or specific version: redis=redis:7.2
```

---

**Issue 3: Missing Container Name**

**Symptoms:**
```
Status: CreateContainerConfigError
Events: container name cannot be empty
```

**How to Verify:**
```bash
kubectl get deployment redis-deployment -o yaml | grep -A10 "containers:"
```

**Fix:**
```bash
kubectl edit deployment redis-deployment
# Add name field:
containers:
- name: redis  # ← Add this
  image: redis:latest
```

---

**Issue 4: Wrong Port Configuration**

**Symptoms:**
```
Pod running but service not accessible
Redis clients can't connect
```

**How to Verify:**
```bash
kubectl get deployment redis-deployment -o jsonpath='{.spec.template.spec.containers[0].ports}'
# Check if port 6379 is exposed
```

**Fix:**
```bash
kubectl edit deployment redis-deployment
# Ensure:
containers:
- name: redis
  image: redis:latest
  ports:
  - containerPort: 6379
    protocol: TCP
```

---

**Issue 5: Insufficient Resources**

**Symptoms:**
```
Status: Pending
Events: 0/3 nodes are available: insufficient cpu/memory
```

**How to Verify:**
```bash
# Check node resources
kubectl describe nodes

# Check deployment resource requests
kubectl get deployment redis-deployment -o yaml | grep -A5 resources
```

**Fix:**
```bash
kubectl edit deployment redis-deployment
# Reduce resource requests or add more nodes
resources:
  requests:
    memory: "64Mi"   # ← Lower these
    cpu: "100m"
  limits:
    memory: "128Mi"
    cpu: "200m"
```

---

**Issue 6: ConfigMap or Secret Missing**

**Symptoms:**
```
Status: CreateContainerConfigError
Events: configmap "redis-config" not found
```

**How to Verify:**
```bash
# Check if ConfigMap exists
kubectl get configmaps

# Check deployment references
kubectl get deployment redis-deployment -o yaml | grep -A10 configMap
```

**Fix:**
```bash
# Option 1: Create missing ConfigMap
kubectl create configmap redis-config --from-literal=maxmemory=2mb

# Option 2: Remove ConfigMap reference
kubectl edit deployment redis-deployment
# Delete the envFrom or volumeMounts section referencing ConfigMap
```

---

**Issue 7: Wrong Command/Args**

**Symptoms:**
```
Status: CrashLoopBackOff
Logs: Error: unknown command 'rediserver'
```

**How to Verify:**
```bash
kubectl get deployment redis-deployment -o yaml | grep -A5 "command:"
```

**Fix:**
```bash
kubectl edit deployment redis-deployment
# Fix command:
command: ["redis-server"]  # Not "rediserver"
args: ["--appendonly", "yes"]
```

---

### Phase 3: Fix the Issue

#### Example: Fixing Image Name Typo

**Step 8: Correct the Image**

```bash
# Method 1: Using kubectl set image (Recommended)
kubectl set image deployment/redis-deployment redis=redis:latest

# Wait for rollout
kubectl rollout status deployment/redis-deployment

# Expected output:
# deployment "redis-deployment" successfully rolled out
```

**Step 9: Verify Fix**

```bash
# Check deployment image
kubectl get deployment redis-deployment -o jsonpath='{.spec.template.spec.containers[0].image}'

# Output: redis:latest ✅

# Check pod status
kubectl get pods -l app=redis

# Expected output:
NAME                               READY   STATUS    RESTARTS   AGE
redis-deployment-7d6b5c8f9-xyz12   1/1     Running   0          30s
```

---

#### Alternative: Edit Deployment YAML

**Step 10: Edit Directly**

```bash
# Open deployment in editor
kubectl edit deployment redis-deployment
```

**Find and fix the issue:**

```yaml
# BEFORE (wrong):
spec:
  containers:
  - name: redis
    image: redi:latest  # ← Typo here
    ports:
    - containerPort: 6379

# AFTER (correct):
spec:
  containers:
  - name: redis
    image: redis:latest  # ← Fixed
    ports:
    - containerPort: 6379
```

**Save and exit:**
- `vi/vim`: Press `Esc`, type `:wq`, press `Enter`
- Kubernetes will automatically apply changes and recreate pods

---

### Phase 4: Verification

#### Step 11: Watch Rollout Progress

```bash
# Watch rollout in real-time
kubectl rollout status deployment/redis-deployment

# Watch pods being created
kubectl get pods -l app=redis -w

# Expected progression:
NAME                               READY   STATUS              RESTARTS   AGE
redis-deployment-new-xyz12         0/1     ContainerCreating   0          5s
redis-deployment-new-xyz12         1/1     Running             0          15s
redis-deployment-old-abcde         1/1     Terminating         0          5m
```

---

#### Step 12: Verify Pod is Running

```bash
# Check pod status
kubectl get pods -l app=redis

# Expected output:
NAME                               READY   STATUS    RESTARTS   AGE
redis-deployment-7d6b5c8f9-xyz12   1/1     Running   0          2m

# Describe pod to confirm no errors
kubectl describe pod <pod-name> | tail -20
```

**Healthy Pod Should Show:**
```
Status: Running
Ready: True
Restarts: 0
Events:
  Normal  Scheduled  2m    Successfully assigned...
  Normal  Pulling    2m    Pulling image "redis:latest"
  Normal  Pulled     2m    Successfully pulled image
  Normal  Created    2m    Created container redis
  Normal  Started    2m    Started container redis
```

---

#### Step 13: Test Redis Functionality

```bash
# Method 1: Port forward to access Redis locally
kubectl port-forward deployment/redis-deployment 6379:6379 &

# Test with redis-cli (if installed locally)
redis-cli ping
# Expected: PONG

# Method 2: Exec into pod
kubectl exec -it <pod-name> -- redis-cli

# Inside redis-cli:
127.0.0.1:6379> ping
PONG
127.0.0.1:6379> set test "Hello Redis"
OK
127.0.0.1:6379> get test
"Hello Redis"
127.0.0.1:6379> exit
```

---

#### Step 14: Check Deployment Health

```bash
# Overall deployment status
kubectl get deployment redis-deployment

# Expected output:
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
redis-deployment   1/1     1            1           20m
```

**Health Indicators:**
- ✅ `READY: 1/1` → All replicas ready
- ✅ `AVAILABLE: 1` → Pod available to serve traffic
- ✅ `UP-TO-DATE: 1` → Latest configuration applied

---

#### Step 15: Check Events (Should be Clean)

```bash
# Recent events for redis
kubectl get events --field-selector involvedObject.name=redis-deployment --sort-by='.lastTimestamp' | tail -10

# Expected (healthy):
LAST SEEN   TYPE     REASON              OBJECT                   MESSAGE
1m          Normal   ScalingReplicaSet   deployment/redis...      Scaled up replica set to 1
1m          Normal   Scheduled           pod/redis...             Successfully assigned pod
1m          Normal   Pulled              pod/redis...             Successfully pulled image "redis:latest"
1m          Normal   Created             pod/redis...             Created container redis
1m          Normal   Started             pod/redis...             Started container redis
```

---

#### Step 16: Final Validation Checklist

```bash
# Run all checks
echo "1. Deployment Status:"
kubectl get deployment redis-deployment

echo -e "\n2. Pod Status:"
kubectl get pods -l app=redis

echo -e "\n3. Pod Details:"
kubectl describe pod $(kubectl get pods -l app=redis -o jsonpath='{.items[0].metadata.name}') | grep -A5 "Status:\|Ready:\|Restart"

echo -e "\n4. Image Verification:"
kubectl get deployment redis-deployment -o jsonpath='{.spec.template.spec.containers[0].image}'

echo -e "\n5. Redis Connectivity:"
kubectl exec $(kubectl get pods -l app=redis -o jsonpath='{.items[0].metadata.name}') -- redis-cli ping

echo -e "\n✅ All checks passed!"
```

---

## � Real-World Troubleshooting Case: Multiple Issues in One Deployment

### Your Actual Day 59 Experience

This section documents the **actual troubleshooting process** you went through during Day 59, where you encountered **three distinct issues** in the same Redis deployment.

#### Initial State

**Deployment Check:**
```bash
kubectl get deployments
# NAME               READY   UP-TO-DATE   AVAILABLE   AGE
# redis-deployment   0/1     1            0           13m
```

**Pod Check:**
```bash
kubectl get pods | grep redis
# redis-deployment-6fd9d5fcb-kghnn   0/1     ContainerCreating   0   13m
```

**🚨 Problem:** Pod stuck in `ContainerCreating` state for 13+ minutes. Something is definitely wrong!

---

#### Investigation Phase

**Step 1: Describe the Deployment**
```bash
kubectl describe deployment redis-deployment
```

**Critical Findings:**

```yaml
# Issue 1: Image Tag Typo
Containers:
  redis-container:
    Image:        redis:alpin    # ❌ Missing 'e' - should be 'alpine'
    Port:         6379/TCP

# Issue 2: ConfigMap Name Typo
Volumes:
  config:
    Type:      ConfigMap (a volume populated by a ConfigMap)
    Name:      redis-cofig      # ❌ Typo - should be 'redis-config'
    Optional:  false
```

**Step 2: Describe the Pod**
```bash
kubectl describe pod redis-deployment-6fd9d5fcb-kghnn
```

**Events Revealed the Problem:**
```
Events:
  Type     Reason       Age                  From     Message
  ----     ------       ----                 ----     -------
  Normal   Scheduled    13m                  default-scheduler  Successfully assigned default/redis-deployment-6fd9d5fcb-kghnn
  Warning  FailedMount  2m (x50 over 13m)    kubelet   MountVolume.SetUp failed for volume "config": configmap "redis-cofig" not found
```

**🔍 Root Cause Identified:** 
1. **Image tag typo:** `redis:alpin` instead of `redis:alpine`
2. **ConfigMap name typo:** `redis-cofig` instead of `redis-config`

**Verification:**
```bash
# Confirm the correct ConfigMap exists
kubectl get configmap | grep redis
# redis-config   1      20m    ✅ ConfigMap exists with correct name
```

---

#### Fix Attempt #1: Correct Both Typos

**Applied Fix:**
```bash
kubectl edit deployment redis-deployment

# Changed:
# 1. Image: redis:alpin → redis:alpine
# 2. ConfigMap name: redis-cofig → redis-config
```

**Result:**
```bash
kubectl get pods -w
# redis-deployment-6fd9d5fcb-kghnn   0/1   Terminating         0   15m    ← Old pod terminating
# redis-deployment-7c8d4f6ddf-7qhjz  0/1   ContainerCreating   0   5s     ← New pod created
# redis-deployment-7c8d4f6ddf-7qhjz  0/1   ErrImagePull        0   10s    ← ❌ New issue!
# redis-deployment-7c8d4f6ddf-7qhjz  0/1   ImagePullBackOff    0   25s    ← Backoff started
```

**🚨 Issue 3 Emerged:** Even after fixing both typos, the pod still won't start!

---

#### Investigation Phase #2: Network/Registry Issue

**Step 3: Describe the New Pod**
```bash
kubectl describe pod redis-deployment-7c8d4f6ddf-7qhjz
```

**New Error in Events:**
```
Events:
  Type     Reason     Age               From     Message
  ----     ------     ----              ----     -------
  Normal   Scheduled  30s               default-scheduler  Successfully assigned default/redis-deployment-7c8d4f6ddf-7qhjz
  Normal   Pulling    10s (x2 over 30s) kubelet  Pulling image "redis:alpine"
  Warning  Failed     10s (x2 over 25s) kubelet  Failed to pull image "redis:alpine": rpc error: code = Unknown desc = 
           failed to pull and unpack image "docker.io/library/redis:alpine": failed to read expected number of bytes: unexpected EOF
  Warning  Failed     10s (x2 over 25s) kubelet  Error: ErrImagePull
  Normal   BackOff    5s (x3 over 25s)  kubelet  Back-off pulling image "redis:alpine"
  Warning  Failed     5s (x3 over 25s)  kubelet  Error: ImagePullBackOff
```

**🔍 Root Cause #3:** Network/registry connectivity issue causing image pull failure
- The image name is NOW correct (`redis:alpine`)
- ConfigMap is NOW correct (`redis-config`)
- BUT: Docker Hub is experiencing connectivity issues or the image pull is interrupted

---

#### Fix Attempt #2: Use Alternative Image Tag

**Solution:** Switch to `redis:latest` tag (more likely to be cached on the node)

```bash
kubectl set image deployment/redis-deployment redis-container=redis:latest
```

**Result:**
```bash
kubectl get pods -w
# redis-deployment-7c8d4f6ddf-7qhjz  0/1   ImagePullBackOff   0   2m     ← Old pod (alpine tag)
# redis-deployment-7c8d4f6ddf-7qhjz  0/1   Terminating        0   2m     ← Terminating
# redis-deployment-68b5dcf7b8-mxk4z  0/1   Pending            0   0s     ← New pod created
# redis-deployment-68b5dcf7b8-mxk4z  0/1   ContainerCreating  0   1s     
# redis-deployment-68b5dcf7b8-mxk4z  1/1   Running            0   5s     ← ✅ SUCCESS!
```

---

#### Final Verification

```bash
# 1. Check deployment status
kubectl get deployment redis-deployment
# NAME               READY   UP-TO-DATE   AVAILABLE   AGE
# redis-deployment   1/1     1            1           25m    ✅

# 2. Check pod status
kubectl get pods -l app=redis
# NAME                                READY   STATUS    RESTARTS   AGE
# redis-deployment-68b5dcf7b8-mxk4z   1/1     Running   0          2m    ✅

# 3. Test Redis functionality
kubectl exec -it redis-deployment-68b5dcf7b8-mxk4z -- redis-cli ping
# PONG    ✅

# 4. Verify final configuration
kubectl get deployment redis-deployment -o yaml | grep -A2 "image:\|configMap:"
#     image: redis:latest          ✅ Correct image
#   configMap:
#     name: redis-config           ✅ Correct ConfigMap
```

---

### 📊 Summary of Three Issues

| Issue # | Type | Problem | Impact | Fix |
|---------|------|---------|--------|-----|
| **1** | Configuration | `redis:alpin` (typo) | Would cause ErrImagePull | Changed to `redis:alpine` |
| **2** | Configuration | ConfigMap `redis-cofig` (typo) | Pod stuck ContainerCreating | Changed to `redis-config` |
| **3** | Network | Image pull failure for alpine | ImagePullBackOff | Changed to `redis:latest` |

---

### 🎓 Key Lessons Learned

1. **Multiple Issues Can Exist Simultaneously**
   - Don't stop after finding the first issue
   - Check both deployment spec AND events thoroughly
   - Verify all resource names match actual resources

2. **Events Are Your Best Friend**
   - `kubectl describe pod` events section tells the full story
   - `FailedMount` events point to volume/ConfigMap issues
   - `Failed to pull image` events indicate registry/network problems

3. **Fix Strategy Matters**
   - Fixed both configuration typos in one edit (efficient)
   - When one fix doesn't work, investigate the NEW error
   - Network issues may require alternative approaches (cached images, different tags)

4. **Troubleshooting Timeline**
   - Issue 1 & 2: Found through `kubectl describe` (configuration review)
   - Issue 3: Appeared AFTER fixing 1 & 2 (cascading discovery)
   - Total resolution time: ~20 minutes with systematic approach

5. **Image Tag Selection**
   - `alpine` tags are smaller but require network pull
   - `latest` tags are more likely to be cached on nodes
   - In network-constrained environments, consider pre-pulling images

---

## �🐛 Common Issues and Solutions

### Issue 1: ImagePullBackOff - Wrong Image Name

**Problem:**
```
Status: ImagePullBackOff
Message: Failed to pull image "redi:latest": not found
```

**Root Cause:**
- Typo in image name (`redi` instead of `redis`)
- Common mistake during manual edits

**Diagnosis:**
```bash
kubectl describe pod <pod-name> | grep -A5 "Events:"
# Look for: Failed to pull image "redi:latest"

kubectl get deployment redis-deployment -o jsonpath='{.spec.template.spec.containers[0].image}'
# Output: redi:latest ← Wrong!
```

**Solution:**
```bash
# Fix the image name
kubectl set image deployment/redis-deployment redis=redis:latest

# Verify
kubectl get pods -l app=redis
# Status should change to Running
```

---

### Issue 2: ErrImagePull - Invalid Tag

**Problem:**
```
Status: ErrImagePull
Message: manifest for redis:abc not found
```

**Root Cause:**
- Image tag doesn't exist on Docker Hub
- Common tags: `latest`, `7.2`, `7.2-alpine`, `7.0`

**Diagnosis:**
```bash
kubectl describe pod <pod-name> | grep "image:"
# Output: redis:abc

# Check available tags: https://hub.docker.com/_/redis/tags
```

**Solution:**
```bash
# Use valid tag
kubectl set image deployment/redis-deployment redis=redis:latest
# Or specific version: redis=redis:7.2
```

---

### Issue 3: CrashLoopBackOff - Wrong Command

**Problem:**
```
Status: CrashLoopBackOff
Logs: Error: unknown command 'rediserver'
```

**Root Cause:**
- Wrong command specified (should be `redis-server`)
- Application exits immediately

**Diagnosis:**
```bash
# Check command
kubectl get deployment redis-deployment -o yaml | grep -A3 "command:"

# Check logs
kubectl logs <pod-name>
```

**Solution:**
```bash
kubectl edit deployment redis-deployment

# Fix:
containers:
- name: redis
  image: redis:latest
  command: ["redis-server"]  # Fix typo
  args: ["--appendonly", "yes"]
```

---

### Issue 4: Pending - Insufficient Resources

**Problem:**
```
Status: Pending
Events: 0/3 nodes are available: insufficient cpu (3), insufficient memory (3)
```

**Root Cause:**
- Resource requests exceed available node resources
- Need to reduce requests or add nodes

**Diagnosis:**
```bash
# Check node resources
kubectl describe nodes | grep -A10 "Allocated resources:"

# Check deployment resources
kubectl get deployment redis-deployment -o yaml | grep -A10 "resources:"
```

**Solution:**
```bash
kubectl edit deployment redis-deployment

# Reduce resources:
resources:
  requests:
    memory: "64Mi"   # Lower from 1Gi
    cpu: "100m"      # Lower from 1000m
  limits:
    memory: "128Mi"
    cpu: "200m"
```

---

### Issue 5: CreateContainerConfigError - Missing ConfigMap

**Problem:**
```
Status: CreateContainerConfigError
Events: configmap "redis-config" not found
```

**Root Cause:**
- Deployment references non-existent ConfigMap
- ConfigMap was deleted accidentally

**Diagnosis:**
```bash
# Check if ConfigMap exists
kubectl get configmaps | grep redis

# Check deployment references
kubectl get deployment redis-deployment -o yaml | grep -A5 configMapRef
```

**Solution:**

**Option 1: Create Missing ConfigMap**
```bash
kubectl create configmap redis-config \
  --from-literal=maxmemory=2mb \
  --from-literal=maxmemory-policy=allkeys-lru
```

**Option 2: Remove ConfigMap Reference**
```bash
kubectl edit deployment redis-deployment

# Remove this section:
envFrom:
- configMapRef:
    name: redis-config  # Delete this entire section
```

---

### Issue 6: Port Configuration Error

**Problem:**
- Pod running but can't connect to Redis
- Service not routing traffic

**Root Cause:**
- Wrong containerPort specified
- Redis uses port 6379 by default

**Diagnosis:**
```bash
# Check configured port
kubectl get deployment redis-deployment -o yaml | grep -A3 "ports:"

# Check if port 6379 is exposed
kubectl exec <pod-name> -- netstat -tulpn | grep 6379
```

**Solution:**
```bash
kubectl edit deployment redis-deployment

# Ensure correct port:
containers:
- name: redis
  image: redis:latest
  ports:
  - containerPort: 6379  # Must be 6379
    protocol: TCP
```

---

### Issue 7: Deployment Selector Mismatch

**Problem:**
```
Error: Deployment.spec.selector does not match template labels
```

**Root Cause:**
- Selector labels don't match pod template labels
- Critical for ReplicaSet to manage pods

**Diagnosis:**
```bash
kubectl get deployment redis-deployment -o yaml | grep -A3 "selector:\|labels:"
```

**Solution:**
```bash
kubectl edit deployment redis-deployment

# Ensure these match:
selector:
  matchLabels:
    app: redis      # ← These must match
template:
  metadata:
    labels:
      app: redis    # ← These must match
```

---

## 📖 Complete Command Summary

### Quick Troubleshooting Script

```bash
#!/bin/bash
# Redis Deployment Troubleshooting Script

DEPLOYMENT="redis-deployment"
POD=$(kubectl get pods -l app=redis -o jsonpath='{.items[0].metadata.name}')

echo "=== Redis Deployment Troubleshooting ==="

echo -e "\n1. Deployment Status:"
kubectl get deployment $DEPLOYMENT

echo -e "\n2. Pod Status:"
kubectl get pods -l app=redis

echo -e "\n3. Pod Description:"
kubectl describe pod $POD | grep -A10 "Status:\|Conditions:\|Events:"

echo -e "\n4. Image Configuration:"
kubectl get deployment $DEPLOYMENT -o jsonpath='{.spec.template.spec.containers[0].image}'

echo -e "\n5. Recent Events:"
kubectl get events --field-selector involvedObject.name=$DEPLOYMENT --sort-by='.lastTimestamp' | tail -10

echo -e "\n6. Container Logs (if available):"
kubectl logs $POD --tail=50 2>/dev/null || echo "No logs available (container not started)"

echo -e "\n=== Troubleshooting Complete ==="
```

---

### Detailed Workflow

**Investigation:**
```bash
# Step 1: Check overall status
kubectl get deployments
kubectl get pods

# Step 2: Get detailed info
kubectl describe deployment redis-deployment
kubectl describe pod <pod-name>

# Step 3: Check events and logs
kubectl get events --sort-by='.lastTimestamp' | tail -20
kubectl logs <pod-name>
```

**Fix:**
```bash
# Option 1: Set image directly
kubectl set image deployment/redis-deployment redis=redis:latest

# Option 2: Edit deployment
kubectl edit deployment redis-deployment

# Option 3: Apply fixed YAML
kubectl apply -f redis-deployment-fixed.yaml
```

**Verify:**
```bash
# Watch rollout
kubectl rollout status deployment/redis-deployment

# Check status
kubectl get deployment redis-deployment
kubectl get pods -l app=redis

# Test Redis
kubectl exec -it <pod-name> -- redis-cli ping
```

---

## 🎓 Best Practices

### Troubleshooting Best Practices

1. **Always Start with `kubectl get`**
   - Get high-level overview first
   - Identify which component is failing

2. **Use `kubectl describe` Next**
   - Shows detailed state and events
   - Often reveals root cause immediately

3. **Check Logs for Application Errors**
   - `kubectl logs` shows application output
   - Use `--previous` for crashed containers

4. **Verify Configuration**
   - Image name and tag correct?
   - Ports properly configured?
   - Resource limits reasonable?

5. **Check Dependencies**
   - ConfigMaps and Secrets exist?
   - Services and Ingress configured?
   - Network policies allow traffic?

6. **Use Events Timeline**
   - Shows what happened in order
   - Helps understand failure sequence

7. **Test Incrementally**
   - Fix one issue at a time
   - Verify before moving to next issue

8. **Document the Issue**
   - What went wrong?
   - How did you fix it?
   - How to prevent it?

---

### Deployment Best Practices

1. **Use Specific Image Tags**
   ```yaml
   # ✅ Good
   image: redis:7.2-alpine
   
   # ❌ Avoid in production
   image: redis:latest
   ```

2. **Set Resource Limits**
   ```yaml
   resources:
     requests:
       memory: "64Mi"
       cpu: "100m"
     limits:
       memory: "128Mi"
       cpu: "200m"
   ```

3. **Add Health Checks**
   ```yaml
   livenessProbe:
     tcpSocket:
       port: 6379
     initialDelaySeconds: 30
     periodSeconds: 10
   
   readinessProbe:
     exec:
       command: ["redis-cli", "ping"]
     initialDelaySeconds: 5
     periodSeconds: 5
   ```

4. **Use Labels Consistently**
   ```yaml
   metadata:
     labels:
       app: redis
       version: "7.2"
       environment: production
   ```

5. **Add Annotations for Documentation**
   ```yaml
   metadata:
     annotations:
       description: "Redis cache for application"
       owner: "devops-team"
       last-updated: "2025-01-01"
   ```

6. **Configure Persistent Storage (Production)**
   ```yaml
   volumes:
   - name: redis-data
     persistentVolumeClaim:
       claimName: redis-pvc
   ```

7. **Set Restart Policy Appropriately**
   ```yaml
   # For long-running services
   restartPolicy: Always
   
   # For jobs
   restartPolicy: OnFailure
   ```

8. **Use ConfigMaps for Configuration**
   ```yaml
   env:
   - name: REDIS_MAXMEMORY
     valueFrom:
       configMapKeyRef:
         name: redis-config
         key: maxmemory
   ```

---

## 🌍 Real-World Applications

### Production Redis Deployment

**Complete production-ready Redis:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-deployment
  labels:
    app: redis
    tier: cache
    environment: production
  annotations:
    description: "Production Redis cache"
    prometheus.io/scrape: "true"
    prometheus.io/port: "9121"
spec:
  replicas: 1  # For standalone; use StatefulSet for clustering
  selector:
    matchLabels:
      app: redis
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0  # Zero downtime
  template:
    metadata:
      labels:
        app: redis
        version: "7.2"
    spec:
      securityContext:
        fsGroup: 1000  # Redis user group
      containers:
      - name: redis
        image: redis:7.2-alpine  # Specific version
        imagePullPolicy: IfNotPresent
        
        command: ["redis-server"]
        args: 
        - "--appendonly yes"  # Persistence
        - "--maxmemory 512mb"  # Memory limit
        - "--maxmemory-policy allkeys-lru"  # Eviction policy
        
        ports:
        - name: redis
          containerPort: 6379
          protocol: TCP
        
        env:
        - name: REDIS_PASSWORD
          valueFrom:
            secretKeyRef:
              name: redis-secret
              key: password
        
        resources:
          requests:
            memory: "256Mi"
            cpu: "200m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        
        livenessProbe:
          tcpSocket:
            port: 6379
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3
        
        readinessProbe:
          exec:
            command: ["redis-cli", "ping"]
          initialDelaySeconds: 5
          periodSeconds: 5
          timeoutSeconds: 3
          failureThreshold: 3
        
        volumeMounts:
        - name: redis-data
          mountPath: /data
        - name: redis-config
          mountPath: /usr/local/etc/redis/redis.conf
          subPath: redis.conf
      
      volumes:
      - name: redis-data
        persistentVolumeClaim:
          claimName: redis-pvc
      - name: redis-config
        configMap:
          name: redis-config
---
apiVersion: v1
kind: Service
metadata:
  name: redis-service
  labels:
    app: redis
spec:
  type: ClusterIP  # Internal only for cache
  selector:
    app: redis
  ports:
  - name: redis
    port: 6379
    targetPort: 6379
    protocol: TCP
```

---

### Redis with Monitoring (Redis Exporter)

```yaml
# Add sidecar for Prometheus metrics
- name: redis-exporter
  image: oliver006/redis_exporter:latest
  ports:
  - name: metrics
    containerPort: 9121
  env:
  - name: REDIS_ADDR
    value: "localhost:6379"
  resources:
    requests:
      memory: "32Mi"
      cpu: "50m"
    limits:
      memory: "64Mi"
      cpu: "100m"
```

---

## ✅ Completion Checklist

Run these commands to verify everything is working:

```bash
# 1. Deployment is healthy
kubectl get deployment redis-deployment
# ✅ READY: 1/1, AVAILABLE: 1

# 2. Pod is running
kubectl get pods -l app=redis
# ✅ STATUS: Running, READY: 1/1

# 3. Image is correct
kubectl get deployment redis-deployment -o jsonpath='{.spec.template.spec.containers[0].image}'
# ✅ Output: redis:latest (or specific version)

# 4. No error events
kubectl get events --field-selector type=Warning --sort-by='.lastTimestamp' | grep redis
# ✅ No recent warnings

# 5. Redis responds to ping
kubectl exec $(kubectl get pods -l app=redis -o jsonpath='{.items[0].metadata.name}') -- redis-cli ping
# ✅ Output: PONG

# 6. Can set and get data
kubectl exec $(kubectl get pods -l app=redis -o jsonpath='{.items[0].metadata.name}') -- redis-cli set test "Hello"
kubectl exec $(kubectl get pods -l app=redis -o jsonpath='{.items[0].metadata.name}') -- redis-cli get test
# ✅ Output: "Hello"

# 7. Port is correct
kubectl get deployment redis-deployment -o jsonpath='{.spec.template.spec.containers[0].ports[0].containerPort}'
# ✅ Output: 6379

# 8. Container name is set
kubectl get deployment redis-deployment -o jsonpath='{.spec.template.spec.containers[0].name}'
# ✅ Output: redis

# 9. Deployment has correct labels
kubectl get deployment redis-deployment -o jsonpath='{.spec.selector.matchLabels}'
# ✅ Output: {"app":"redis"}

# 10. Rollout completed successfully
kubectl rollout status deployment/redis-deployment
# ✅ Output: deployment "redis-deployment" successfully rolled out
```

---

## 📊 Troubleshooting Flowchart

```
START: Redis Deployment Not Running
    ↓
1. kubectl get deployment redis-deployment
    ↓
    ├─ READY: 0/1? → Continue
    └─ READY: 1/1? → Check if accessible
        ↓
2. kubectl get pods -l app=redis
    ↓
    ├─ ImagePullBackOff? → Check image name/tag
    │   ↓
    │   kubectl set image deployment/redis-deployment redis=redis:latest
    │
    ├─ CrashLoopBackOff? → Check logs
    │   ↓
    │   kubectl logs <pod-name>
    │   └─ Fix command/config
    │
    ├─ Pending? → Check resources
    │   ↓
    │   kubectl describe nodes
    │   └─ Reduce resource requests
    │
    └─ CreateContainerConfigError? → Check ConfigMap/Secret
        ↓
        kubectl get configmaps
        └─ Create missing ConfigMap or remove reference
        ↓
3. Verify Fix
    ↓
    kubectl rollout status deployment/redis-deployment
    ↓
4. Test Redis
    ↓
    kubectl exec -it <pod-name> -- redis-cli ping
    ↓
END: Redis Deployment Running ✅
```

---

## 🎉 Summary

**What You Learned:**
- ✅ Systematic Kubernetes troubleshooting methodology
- ✅ How to use kubectl describe, logs, and events effectively
- ✅ Common deployment configuration errors and fixes
- ✅ Redis deployment patterns on Kubernetes
- ✅ Production-ready deployment best practices
- ✅ Resource management and health checks
- ✅ Real-world debugging scenarios

**Key Takeaways:**
1. **Start with `kubectl get`** for overview
2. **Use `kubectl describe`** for detailed state
3. **Check events** for failure timeline
4. **Fix one issue at a time** and verify
5. **Always use specific image tags** in production
6. **Set resource limits** to prevent resource exhaustion
7. **Add health checks** for automatic recovery
8. **Test incrementally** after each change

**Day 59 Complete!** 🎊 You can now confidently debug Kubernetes deployments and fix common issues quickly!

---

**Next Steps:**
- Day 60: Kubernetes ConfigMaps and Secrets
- Day 61: Persistent Volumes and Storage
- Day 62: StatefulSets for Stateful Applications
- Day 63: DaemonSets and Node Management

---

**Tags:** #Kubernetes #Troubleshooting #Redis #Debugging #DevOps #K8s #ProductionIssues #kubectl #ContainerOrchestration