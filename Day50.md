# Day 50: Kubernetes Resource Management - Requests and Limits
**100 Days Cloud DevOps Challenge – KodeKloud**

---

## Objective

Configure resource requests and limits for Kubernetes pods to ensure predictable performance, prevent resource starvation, and optimize cluster utilization. Learn how Kubernetes manages CPU and memory resources.

**Requirements:**
1. Work on **jump_host** (kubectl already configured)
2. **Pod Name:** `httpd-pod`
3. **Container Name:** `httpd-container`
4. **Image:** `httpd:latest` (explicitly specify tag)
5. **Resource Requests:**
   - Memory: `15Mi`
   - CPU: `100m`
6. **Resource Limits:**
   - Memory: `20Mi`
   - CPU: `100m`

---

## Understanding Kubernetes Resource Management

**Resources** in Kubernetes refer to compute resources like CPU and memory that containers need to run. Proper resource management ensures applications run reliably without affecting other workloads.

### What are Resource Requests and Limits?

```
Resource Requests:
├── Minimum resources guaranteed to container
├── Used by scheduler to choose nodes
├── Container always gets at least this amount
└── Node must have available resources

Resource Limits:
├── Maximum resources container can use
├── Container cannot exceed this amount
├── Enforced by kubelet
└── Container throttled/killed if exceeded
```

### Requests vs Limits:

```
Resource Allocation:

Requests (Minimum Guaranteed):
┌─────────────────────────────────┐
│  Container gets AT LEAST this   │
│  Scheduler: "Node must have     │
│  this much available"           │
└─────────────────────────────────┘

Limits (Maximum Allowed):
┌─────────────────────────────────┐
│  Container can use UP TO this   │
│  Kubelet: "Stop if exceeds!"    │
└─────────────────────────────────┘

Example:
Requests: 15Mi memory → Guaranteed 15Mi
Limits:   20Mi memory → Can burst to 20Mi
```

### Why Resource Management Matters:

```
Without Resource Management:
├── ❌ One app can starve others
├── ❌ Unpredictable performance
├── ❌ Node overcommitment
├── ❌ Random pod evictions
└── ❌ Cluster instability

With Resource Management:
├── ✅ Predictable performance
├── ✅ Fair resource sharing
├── ✅ Efficient scheduling
├── ✅ Prevents resource starvation
└── ✅ Cluster stability
```

---

## Understanding CPU Resources

**CPU** in Kubernetes is measured in cores (millicores).

### CPU Units:

```
CPU Measurement:
├── 1 CPU = 1 core = 1000 millicores (m)
├── 100m = 0.1 CPU = 10% of one core
├── 500m = 0.5 CPU = 50% of one core
└── 1000m = 1 CPU = 100% of one core = 1 full core

Examples:
100m   → 10% of one CPU core
250m   → 25% of one CPU core
500m   → 50% of one CPU core (half a core)
1000m  → 100% of one CPU core (1 full core)
2      → 2 full CPU cores
2000m  → 2 full CPU cores
```

### CPU as Compressible Resource:

```
CPU is COMPRESSIBLE:
├── Container can be throttled
├── Exceeding limit → CPU throttling (slower execution)
├── Does NOT kill container
└── Container continues running (just slower)

Example:
Request: 100m → Guaranteed 10% of CPU
Limit:   100m → Maximum 10% of CPU
         ↓
Container tries to use more → Throttled to 100m
         ↓
Container keeps running (slower performance)
```

---

## Understanding Memory Resources

**Memory** in Kubernetes is measured in bytes (with standard units).

### Memory Units:

```
Memory Measurement:
├── Ki = Kibibyte = 1024 bytes
├── Mi = Mebibyte = 1024 Ki = 1,048,576 bytes
├── Gi = Gibibyte = 1024 Mi
├── Ti = Tebibyte = 1024 Gi

Examples:
15Mi   → 15 Mebibytes = 15,728,640 bytes
20Mi   → 20 Mebibytes = 20,971,520 bytes
128Mi  → 128 Mebibytes = ~134 MB
1Gi    → 1 Gibibyte = 1024 Mi = ~1.07 GB
2Gi    → 2 Gibibytes = ~2.15 GB

Note: Ki/Mi/Gi (binary) vs KB/MB/GB (decimal)
1Mi = 1,048,576 bytes
1MB = 1,000,000 bytes
```

### Memory as Incompressible Resource:

```
Memory is INCOMPRESSIBLE:
├── Cannot be throttled
├── Exceeding limit → Container KILLED (OOMKilled)
├── Container terminated and restarted
└── Must stay within limit

Example:
Request: 15Mi → Guaranteed 15Mi memory
Limit:   20Mi → Maximum 20Mi memory
         ↓
Container tries to use 21Mi → KILLED by kubelet
         ↓
Container terminated → Restarted (if restart policy allows)
         ↓
Status: OOMKilled (Out Of Memory)
```

---

## Understanding Resource Behavior

### CPU Behavior:

| Scenario | Request | Limit | Behavior |
|----------|---------|-------|----------|
| **Normal usage** | 100m | 100m | Uses up to 100m, throttled if exceeds |
| **Low usage** | 100m | 500m | Uses less than 100m, can burst to 500m |
| **Burst usage** | 100m | 500m | Temporarily uses 500m (allowed) |
| **Exceeded** | 100m | 100m | **Throttled** (slower execution) |

### Memory Behavior:

| Scenario | Request | Limit | Behavior |
|----------|---------|-------|----------|
| **Normal usage** | 15Mi | 20Mi | Uses 15Mi, can burst to 20Mi |
| **Low usage** | 15Mi | 20Mi | Uses less than 15Mi (normal) |
| **Burst usage** | 15Mi | 20Mi | Temporarily uses 20Mi (allowed) |
| **Exceeded** | 15Mi | 20Mi | **OOMKilled** (container terminated!) |

### QoS Classes (Quality of Service):

```
Kubernetes assigns QoS class based on requests/limits:

1. Guaranteed (Highest Priority):
   ├── Requests = Limits (for all resources)
   ├── Example: Request 100m, Limit 100m
   └── Last to be evicted

2. Burstable (Medium Priority):
   ├── Requests < Limits OR only requests set
   ├── Example: Request 100m, Limit 500m
   └── Evicted after BestEffort

3. BestEffort (Lowest Priority):
   ├── No requests or limits set
   ├── Can use any available resources
   └── First to be evicted
```

**Our Task:**
```
Requests: Memory 15Mi, CPU 100m
Limits:   Memory 20Mi, CPU 100m

CPU: Request = Limit (100m = 100m) → Guaranteed for CPU
Memory: Request < Limit (15Mi < 20Mi) → Burstable for Memory

Overall QoS Class: Burstable
(At least one resource has Request < Limit)
```

---

## Understanding Scheduling with Resources

### How Scheduler Uses Requests:

```
Node Selection Process:

1. User creates pod with requests:
   CPU: 100m, Memory: 15Mi

2. Scheduler checks all nodes:
   Node1: 500m CPU, 100Mi memory available ✅
   Node2: 50m CPU, 200Mi memory available ❌ (insufficient CPU)
   Node3: 200m CPU, 10Mi memory available ❌ (insufficient memory)

3. Scheduler assigns pod to Node1 ✅

4. Node resources updated:
   Node1: 400m CPU, 85Mi memory available (500-100=400, 100-15=85)
```

### Node Capacity Example:

```
Node with 2 CPU cores and 4Gi memory:

Total Capacity:
├── CPU: 2000m (2 cores)
└── Memory: 4Gi

After system pods:
├── CPU: 1500m available
└── Memory: 3Gi available

Can fit pods with:
├── 15 pods × 100m CPU = 1500m ✅
├── OR 10 pods × 150m CPU = 1500m ✅
├── BUT NOT 20 pods × 100m = 2000m ❌ (only 1500m available)
```

---

## Understanding Resource Constraints

### What Happens When Resources Insufficient?

```
Insufficient CPU (Compressible):
├── Scheduler cannot place pod → Pending
├── Pod waits for node with available CPU
├── Once scheduled, CPU throttling if exceeds
└── Status: Pending (not enough CPU)

Insufficient Memory (Incompressible):
├── Scheduler cannot place pod → Pending
├── Pod waits for node with available memory
├── If scheduled and exceeds → OOMKilled
└── Status: Pending (not enough memory)

Example:
All nodes have: 50m CPU available
Pod needs: 100m CPU
Result: Pod stuck in Pending state
```

### Resource Pressure Scenarios:

```
Memory Pressure (Node running out of memory):
├── Kubelet starts evicting pods
├── Eviction order: BestEffort → Burstable → Guaranteed
├── Pods exceeding requests evicted first
└── Critical pods with Guaranteed QoS last

CPU Pressure (Node running out of CPU):
├── Kubelet throttles containers
├── Each container gets proportional CPU time
├── No eviction (CPU is compressible)
└── Slower execution for all containers
```

---

## YAML Structure for Resource Management

### Basic Pod YAML with Resources:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: httpd-pod
spec:
  containers:
  - name: httpd-container
    image: httpd:latest
    
    # Resource management
    resources:
      requests:         # Minimum guaranteed
        memory: "15Mi"  # 15 Mebibytes
        cpu: "100m"     # 100 millicores (0.1 CPU)
      limits:           # Maximum allowed
        memory: "20Mi"  # 20 Mebibytes
        cpu: "100m"     # 100 millicores (0.1 CPU)
```

### Field Explanations:

| Field | Purpose | Unit | Example |
|-------|---------|------|---------|
| **resources.requests.memory** | Minimum memory guaranteed | Mi, Gi | `15Mi` |
| **resources.requests.cpu** | Minimum CPU guaranteed | m (millicores) | `100m` |
| **resources.limits.memory** | Maximum memory allowed | Mi, Gi | `20Mi` |
| **resources.limits.cpu** | Maximum CPU allowed | m (millicores) | `100m` |

### Resource Validation:

```yaml
# ✅ Valid - Requests ≤ Limits
resources:
  requests:
    memory: "15Mi"
    cpu: "100m"
  limits:
    memory: "20Mi"   # 20Mi ≥ 15Mi ✅
    cpu: "100m"      # 100m ≥ 100m ✅

# ❌ Invalid - Requests > Limits
resources:
  requests:
    memory: "25Mi"   # 25Mi > 20Mi ❌
    cpu: "200m"      # 200m > 100m ❌
  limits:
    memory: "20Mi"
    cpu: "100m"
```

---

## Infrastructure Overview

### Task Details:

| Item | Value |
|------|-------|
| **Host** | jump_host (kubectl pre-configured) |
| **Pod Name** | `httpd-pod` (exact name) |
| **Container Name** | `httpd-container` (exact name) |
| **Image** | `httpd:latest` (explicit tag) |
| **Memory Request** | `15Mi` (minimum guaranteed) |
| **CPU Request** | `100m` (0.1 CPU, minimum) |
| **Memory Limit** | `20Mi` (maximum allowed) |
| **CPU Limit** | `100m` (0.1 CPU, maximum) |
| **QoS Class** | Burstable (Memory: 15Mi < 20Mi) |
| **Namespace** | default |

---

## Step-by-Step Implementation

### Method 1: Declarative YAML (Recommended)

#### Step 1: Verify kubectl Access
```bash
# Check kubectl is configured
kubectl version --client
```

**Expected output:**
```
Client Version: v1.28.0
Kustomize Version: v5.0.4-0.20230601165947-6ce0bf390ce3
```

**✅ kubectl installed and accessible**

#### Step 2: Check Cluster Connection
```bash
# Verify cluster connectivity
kubectl cluster-info
```

**Expected output:**
```
Kubernetes control plane is running at https://10.0.0.1:6443
CoreDNS is running at https://10.0.0.1:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
```

**✅ Connected to Kubernetes cluster**

#### Step 3: Check Node Resources (Optional)
```bash
# View node capacity and allocatable resources
kubectl describe nodes
```

**Expected output (excerpt):**
```
Capacity:
  cpu:                2
  memory:             4Gi
Allocatable:
  cpu:                1900m
  memory:             3.5Gi
```

**Note:** This shows available resources on nodes.

#### Step 4: Create Pod YAML with Resource Constraints
```bash
# Create httpd-pod.yaml file
cat > httpd-pod.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: httpd-pod
  labels:
    app: httpd
spec:
  containers:
  - name: httpd-container
    image: httpd:latest
    ports:
    - containerPort: 80
    resources:
      requests:
        memory: "15Mi"
        cpu: "100m"
      limits:
        memory: "20Mi"
        cpu: "100m"
EOF
```

**Expected output:**
```
(No output means success)
```

**✅ YAML manifest created**

#### Step 5: Verify YAML File
```bash
cat httpd-pod.yaml
```

**Expected output:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: httpd-pod
  labels:
    app: httpd
spec:
  containers:
  - name: httpd-container
    image: httpd:latest
    ports:
    - containerPort: 80
    resources:
      requests:
        memory: "15Mi"
        cpu: "100m"
      limits:
        memory: "20Mi"
        cpu: "100m"
```

**Verify all requirements:**
- ✅ metadata.name: `httpd-pod`
- ✅ containers[0].name: `httpd-container`
- ✅ image: `httpd:latest`
- ✅ resources.requests.memory: `15Mi`
- ✅ resources.requests.cpu: `100m`
- ✅ resources.limits.memory: `20Mi`
- ✅ resources.limits.cpu: `100m`
- ✅ Proper YAML indentation

#### Step 6: Validate YAML (Optional)
```bash
# Dry-run to validate without creating
kubectl apply -f httpd-pod.yaml --dry-run=client
```

**Expected output:**
```
pod/httpd-pod created (dry run)
```

**✅ YAML is valid**

#### Step 7: Create Pod from YAML
```bash
# Apply the manifest
kubectl apply -f httpd-pod.yaml
```

**Expected output:**
```
pod/httpd-pod created
```

**✅ Pod created successfully!**

---

## Verification Steps

### Step 8: Check Pod Status
```bash
kubectl get pods
```

**Expected output:**
```
NAME        READY   STATUS    RESTARTS   AGE
httpd-pod   1/1     Running   0          30s
```

**Status meanings:**
- **Pending** - Waiting for scheduling (insufficient resources?)
- **ContainerCreating** - Pulling image, starting container
- **Running** - Pod is running ✅
- **OOMKilled** - Exceeded memory limit (killed)
- **CrashLoopBackOff** - Container keeps crashing

**✅ Pod is Running**

#### Step 9: Get Pod with Resource Info
```bash
kubectl get pod httpd-pod -o wide
```

**Expected output:**
```
NAME        READY   STATUS    RESTARTS   AGE   IP            NODE     NOMINATED NODE   READINESS GATES
httpd-pod   1/1     Running   0          1m    10.244.0.5    node01   <none>           <none>
```

**✅ Pod assigned to node and running**

### Step 10: Describe Pod to View Resources
```bash
kubectl describe pod httpd-pod
```

**Expected output:**
```
Name:             httpd-pod
Namespace:        default
Priority:         0
Service Account:  default
Node:             node01/192.168.1.10
Start Time:       Mon, 30 Dec 2025 10:00:00 +0000
Labels:           app=httpd
Annotations:      <none>
Status:           Running
IP:               10.244.0.5
IPs:
  IP:  10.244.0.5
Containers:
  httpd-container:
    Container ID:   containerd://abc123def456...
    Image:          httpd:latest
    Image ID:       docker.io/library/httpd@sha256:789abc...
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Mon, 30 Dec 2025 10:00:15 +0000
    Ready:          True
    Restart Count:  0
    Limits:
      cpu:     100m
      memory:  20Mi
    Requests:
      cpu:        100m
      memory:     15Mi
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-xxxxx (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  kube-api-access-xxxxx:
    Type:                    Projected (a volume that contains injected data from multiple sources)
QoS Class:       Burstable
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                 node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  1m    default-scheduler  Successfully assigned default/httpd-pod to node01
  Normal  Pulling    1m    kubelet            Pulling image "httpd:latest"
  Normal  Pulled     1m    kubelet            Successfully pulled image "httpd:latest" in 10.5s
  Normal  Created    1m    kubelet            Created container httpd-container
  Normal  Started    1m    kubelet            Started container httpd-container
```

**Key details to verify:**
- ✅ Name: httpd-pod
- ✅ Container Name: httpd-container
- ✅ Image: httpd:latest
- ✅ **Limits:** cpu: 100m, memory: 20Mi
- ✅ **Requests:** cpu: 100m, memory: 15Mi
- ✅ **QoS Class: Burstable**
- ✅ Status: Running
- ✅ Events show successful scheduling and start

### Step 11: Get Pod YAML to Verify Resources
```bash
# View complete pod configuration
kubectl get pod httpd-pod -o yaml | grep -A 10 resources
```

**Expected output:**
```yaml
    resources:
      limits:
        cpu: 100m
        memory: 20Mi
      requests:
        cpu: 100m
        memory: 15Mi
```

**✅ Resources correctly configured**

### Step 12: Check QoS Class
```bash
# Get QoS class
kubectl get pod httpd-pod -o jsonpath='{.status.qosClass}'
echo
```

**Expected output:**
```
Burstable
```

**✅ QoS Class is Burstable (Memory request < limit)**

**QoS Explanation:**
```
CPU:    Request 100m = Limit 100m → Guaranteed
Memory: Request 15Mi < Limit 20Mi → Burstable

Overall QoS: Burstable
(At least one resource has request < limit)
```

### Step 13: Verify Resource Allocation on Node
```bash
# Check how much resources pod is using from node
kubectl describe node | grep -A 15 "Allocated resources"
```

**Expected output (excerpt):**
```
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  Resource           Requests      Limits
  --------           --------      ------
  cpu                100m (5%)     100m (5%)
  memory             15Mi (0%)     20Mi (0%)
```

**✅ Pod's resources allocated from node capacity**

### Step 14: Check Container Resource Usage (Requires Metrics Server)
```bash
# Real-time resource usage
kubectl top pod httpd-pod
```

**Expected output:**
```
NAME        CPU(cores)   MEMORY(bytes)
httpd-pod   10m          12Mi
```

**Note:** Requires metrics-server to be installed in cluster.

**Analysis:**
- CPU: 10m (using 10% of 100m limit) ✅
- Memory: 12Mi (within 15Mi-20Mi range) ✅

### Step 15: Check Pod Logs
```bash
kubectl logs httpd-pod
```

**Expected output:**
```
AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 10.244.0.5. Set the 'ServerName' directive globally to suppress this message
AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 10.244.0.5. Set the 'ServerName' directive globally to suppress this message
[Mon Dec 30 10:00:15.123456 2025] [mpm_event:notice] [pid 1:tid 140123456789] AH00489: Apache/2.4.58 (Unix) configured -- resuming normal operations
[Mon Dec 30 10:00:15.123789 2025] [core:notice] [pid 1:tid 140123456789] AH00094: Command line: 'httpd -D FOREGROUND'
```

**✅ Apache HTTP Server started successfully**

### Step 16: Test Container Resource Limits (Optional)
```bash
# Execute command to check memory inside container
kubectl exec httpd-pod -- free -m
```

**Expected output:**
```
              total        used        free      shared  buff/cache   available
Mem:             20           12           5           0           2           7
```

**Note:** Container sees limit (20Mi) as total memory.

### Step 17: Final Verification Checklist
```bash
# 1. Pod exists with correct name
kubectl get pod httpd-pod &> /dev/null && echo "✅ Pod exists" || echo "❌ Pod not found"

# 2. Pod is running
kubectl get pod httpd-pod -o jsonpath='{.status.phase}' | grep -q "Running" && echo "✅ Pod running" || echo "❌ Pod not running"

# 3. Container name is correct
kubectl get pod httpd-pod -o jsonpath='{.spec.containers[0].name}' | grep -q "httpd-container" && echo "✅ Container name correct" || echo "❌ Container name incorrect"

# 4. Image is correct
kubectl get pod httpd-pod -o jsonpath='{.spec.containers[0].image}' | grep -q "httpd:latest" && echo "✅ Image correct" || echo "❌ Image incorrect"

# 5. Memory request is correct
kubectl get pod httpd-pod -o jsonpath='{.spec.containers[0].resources.requests.memory}' | grep -q "15Mi" && echo "✅ Memory request correct" || echo "❌ Memory request incorrect"

# 6. CPU request is correct
kubectl get pod httpd-pod -o jsonpath='{.spec.containers[0].resources.requests.cpu}' | grep -q "100m" && echo "✅ CPU request correct" || echo "❌ CPU request incorrect"

# 7. Memory limit is correct
kubectl get pod httpd-pod -o jsonpath='{.spec.containers[0].resources.limits.memory}' | grep -q "20Mi" && echo "✅ Memory limit correct" || echo "❌ Memory limit incorrect"

# 8. CPU limit is correct
kubectl get pod httpd-pod -o jsonpath='{.spec.containers[0].resources.limits.cpu}' | grep -q "100m" && echo "✅ CPU limit correct" || echo "❌ CPU limit incorrect"

# 9. QoS class is Burstable
kubectl get pod httpd-pod -o jsonpath='{.status.qosClass}' | grep -q "Burstable" && echo "✅ QoS class correct" || echo "❌ QoS class incorrect"
```

**All checks should pass:**
```
✅ Pod exists
✅ Pod running
✅ Container name correct
✅ Image correct
✅ Memory request correct (15Mi)
✅ CPU request correct (100m)
✅ Memory limit correct (20Mi)
✅ CPU limit correct (100m)
✅ QoS class correct (Burstable)
```

---

## Complete Command Summary

### Quick Deployment:
```bash
# Create YAML manifest
cat > httpd-pod.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: httpd-pod
  labels:
    app: httpd
spec:
  containers:
  - name: httpd-container
    image: httpd:latest
    ports:
    - containerPort: 80
    resources:
      requests:
        memory: "15Mi"
        cpu: "100m"
      limits:
        memory: "20Mi"
        cpu: "100m"
EOF

# Create pod
kubectl apply -f httpd-pod.yaml

# Verify
kubectl get pods
kubectl describe pod httpd-pod
```

### Detailed Workflow:
```bash
# 1. Verify kubectl access
kubectl version --client
kubectl cluster-info

# 2. Check node resources (optional)
kubectl describe nodes | grep -A 10 "Allocatable"

# 3. Create pod manifest
cat > httpd-pod.yaml << 'EOF'
[YAML content]
EOF

# 4. Validate YAML
kubectl apply -f httpd-pod.yaml --dry-run=client

# 5. Create pod
kubectl apply -f httpd-pod.yaml

# 6. Watch pod creation
kubectl get pods -w

# 7. Check pod status
kubectl get pods
kubectl get pod httpd-pod -o wide

# 8. Describe pod (view resources)
kubectl describe pod httpd-pod

# 9. Verify resources in YAML
kubectl get pod httpd-pod -o yaml | grep -A 10 resources

# 10. Check QoS class
kubectl get pod httpd-pod -o jsonpath='{.status.qosClass}'

# 11. Check resource usage (if metrics-server installed)
kubectl top pod httpd-pod

# 12. Check logs
kubectl logs httpd-pod

# 13. Verify all settings
kubectl get pod httpd-pod -o jsonpath='{.spec.containers[0].resources}'
```

---

## Understanding Resource Units Reference

### CPU Units:

| Notation | Meaning | Percentage | Example Use Case |
|----------|---------|------------|------------------|
| **100m** | 0.1 CPU | 10% | Light app, sidecar |
| **250m** | 0.25 CPU | 25% | Small web service |
| **500m** | 0.5 CPU | 50% | Medium workload |
| **1 or 1000m** | 1 CPU | 100% | CPU-intensive app |
| **2 or 2000m** | 2 CPUs | 200% | Heavy processing |

### Memory Units:

| Notation | Bytes | Approximate | Example Use Case |
|----------|-------|-------------|------------------|
| **15Mi** | 15,728,640 | ~15 MB | Minimal app |
| **20Mi** | 20,971,520 | ~20 MB | Small container |
| **128Mi** | 134,217,728 | ~128 MB | Small web app |
| **256Mi** | 268,435,456 | ~256 MB | Medium web app |
| **512Mi** | 536,870,912 | ~512 MB | Standard app |
| **1Gi** | 1,073,741,824 | ~1 GB | Large app |
| **2Gi** | 2,147,483,648 | ~2 GB | Database, cache |

---

## kubectl Resource Commands Reference

### Pod Resource Management:

```bash
# View pod resources
kubectl describe pod POD_NAME                           # Full details
kubectl get pod POD_NAME -o yaml | grep -A 10 resources # YAML format
kubectl get pod POD_NAME -o jsonpath='{.spec.containers[0].resources}' # JSON path

# View QoS class
kubectl get pod POD_NAME -o jsonpath='{.status.qosClass}'

# View resource usage (requires metrics-server)
kubectl top pod POD_NAME                                # Single pod
kubectl top pods                                        # All pods
kubectl top pods --sort-by=memory                       # Sort by memory
kubectl top pods --sort-by=cpu                          # Sort by CPU

# View node resources
kubectl describe nodes                                  # Full node details
kubectl top nodes                                       # Node usage
kubectl get nodes -o json | jq '.items[] | {name:.metadata.name, allocatable:.status.allocatable}' # Allocatable resources
```

### Resource Debugging:

```bash
# Check why pod is pending (insufficient resources?)
kubectl describe pod POD_NAME | grep -A 10 Events

# Check node pressure
kubectl describe node NODE_NAME | grep -A 5 Conditions

# Check resource quotas (if set)
kubectl describe resourcequota

# Check limit ranges (if set)
kubectl describe limitrange
```

---

## Troubleshooting Guide

### Issue 1: Pod Stuck in Pending - Insufficient CPU

**Problem:**
```bash
kubectl get pods
NAME        READY   STATUS    RESTARTS   AGE
httpd-pod   0/1     Pending   0          2m
```

**Diagnosis:**
```bash
kubectl describe pod httpd-pod
```

**Events:**
```
Warning  FailedScheduling  2m  default-scheduler  0/3 nodes are available: 3 Insufficient cpu.
```

**Cause:**
```
Pod requests: 100m CPU
All nodes have: < 100m CPU available
Result: Cannot schedule pod
```

**Solutions:**

**A. Reduce CPU request:**
```yaml
resources:
  requests:
    cpu: "50m"  # Reduced from 100m
```

**B. Add more nodes to cluster**

**C. Delete other pods to free resources:**
```bash
kubectl delete pod OTHER_POD
```

**D. Check what's consuming resources:**
```bash
kubectl top nodes
kubectl top pods --all-namespaces
```

### Issue 2: Pod Stuck in Pending - Insufficient Memory

**Problem:**
```
Warning  FailedScheduling  2m  default-scheduler  0/3 nodes are available: 3 Insufficient memory.
```

**Cause:**
```
Pod requests: 15Mi memory
All nodes have: < 15Mi memory available
Result: Cannot schedule pod
```

**Solutions:**

**A. Reduce memory request:**
```yaml
resources:
  requests:
    memory: "10Mi"  # Reduced from 15Mi
```

**B. Free memory on nodes:**
```bash
# Delete unused pods
kubectl delete pod UNUSED_POD

# Check memory hogs
kubectl top pods --all-namespaces --sort-by=memory
```

### Issue 3: Pod OOMKilled (Out Of Memory)

**Problem:**
```bash
kubectl get pods
NAME        READY   STATUS      RESTARTS   AGE
httpd-pod   0/1     OOMKilled   3          5m
```

**Diagnosis:**
```bash
kubectl describe pod httpd-pod
```

**Events:**
```
Warning  BackOff  2m  kubelet  Back-off restarting failed container
```

**Container status:**
```
Last State:     Terminated
  Reason:       OOMKilled
  Exit Code:    137
```

**Cause:**
```
Container tried to use > 20Mi memory (limit)
Kubelet killed container to protect node
Container restarted (and likely killed again)
```

**Solutions:**

**A. Increase memory limit:**
```yaml
resources:
  limits:
    memory: "50Mi"  # Increased from 20Mi
```

**B. Optimize application to use less memory**

**C. Check application logs:**
```bash
kubectl logs httpd-pod --previous  # Logs from killed container
```

**D. Check actual memory usage:**
```bash
kubectl top pod httpd-pod
```

### Issue 4: CPU Throttling (Slow Performance)

**Problem:**
Application running very slowly, high latency.

**Diagnosis:**
```bash
# Check if CPU usage is at limit
kubectl top pod httpd-pod
```

**Output:**
```
NAME        CPU(cores)   MEMORY(bytes)
httpd-pod   100m         15Mi
```

**Analysis:**
```
CPU usage: 100m (exactly at limit!)
Container is being throttled
Cannot use more CPU even if available on node
```

**Solutions:**

**A. Increase CPU limit:**
```yaml
resources:
  limits:
    cpu: "200m"  # Increased from 100m
```

**B. Check if CPU is actually bottleneck:**
```bash
# Enter container and check
kubectl exec -it httpd-pod -- top
```

**C. Profile application to reduce CPU usage**

### Issue 5: Request > Limit Error

**Problem:**
```
The Pod "httpd-pod" is invalid: spec.containers[0].resources.requests: Invalid value: "200m": must be less than or equal to cpu limit
```

**Cause:**
```yaml
# Invalid configuration
resources:
  requests:
    cpu: "200m"     # Request > Limit ❌
  limits:
    cpu: "100m"
```

**Solution:**
```yaml
# Fix: Ensure Requests ≤ Limits
resources:
  requests:
    cpu: "100m"     # Request ≤ Limit ✅
  limits:
    cpu: "200m"
```

### Issue 6: Invalid Resource Units

**Problem:**
```
The Pod "httpd-pod" is invalid: spec.containers[0].resources.requests.memory: Invalid value: "15MB": quantities must match the regular expression
```

**Cause:**
```yaml
# Wrong unit - used MB instead of Mi
resources:
  requests:
    memory: "15MB"  # ❌ Wrong
```

**Solution:**
```yaml
# Use correct Kubernetes units
resources:
  requests:
    memory: "15Mi"  # ✅ Correct (Mebibytes)
    # OR
    memory: "15728640"  # ✅ Bytes
```

**Valid units:**
- Memory: `Ki`, `Mi`, `Gi`, `Ti` (binary) or bytes
- CPU: `m` (millicores) or whole numbers

### Issue 7: Pod Evicted Due to Resource Pressure

**Problem:**
```bash
kubectl get pods
NAME        READY   STATUS    RESTARTS   AGE
httpd-pod   0/1     Evicted   0          5m
```

**Diagnosis:**
```bash
kubectl describe pod httpd-pod
```

**Message:**
```
Status:  Failed
Reason:  Evicted
Message: The node was low on resource: memory. Container httpd-container was using 22Mi, which exceeds its request of 15Mi.
```

**Cause:**
```
Node ran out of memory
Kubelet evicted pods to free resources
Pods with QoS Burstable evicted first (our case)
```

**Solutions:**

**A. Increase resource requests:**
```yaml
resources:
  requests:
    memory: "20Mi"  # Closer to actual usage
```

**B. Recreate pod (it will be scheduled again):**
```bash
kubectl delete pod httpd-pod
kubectl apply -f httpd-pod.yaml
```

**C. Add more nodes to cluster**

**D. Set higher QoS (Guaranteed):**
```yaml
resources:
  requests:
    memory: "20Mi"
  limits:
    memory: "20Mi"  # Request = Limit → Guaranteed QoS
```

---

## Advanced Resource Management

### Resource Quotas (Namespace Level):

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
  namespace: default
spec:
  hard:
    requests.cpu: "10"       # Total CPU requests in namespace
    requests.memory: "10Gi"  # Total memory requests
    limits.cpu: "20"         # Total CPU limits
    limits.memory: "20Gi"    # Total memory limits
    pods: "50"               # Max number of pods
```

**Purpose:**
- Limit total resources in namespace
- Prevent resource hogging
- Fair sharing among teams

### LimitRange (Default Values):

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: limit-range
  namespace: default
spec:
  limits:
  - max:
      cpu: "2"
      memory: "2Gi"
    min:
      cpu: "100m"
      memory: "10Mi"
    default:
      cpu: "500m"
      memory: "512Mi"
    defaultRequest:
      cpu: "250m"
      memory: "256Mi"
    type: Container
```

**Purpose:**
- Set default requests/limits for containers
- Enforce min/max constraints
- Prevent overly large/small resource specifications

### Vertical Pod Autoscaler (VPA):

```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: httpd-vpa
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: httpd
  updatePolicy:
    updateMode: "Auto"  # Auto, Initial, Off
```

**Purpose:**
- Automatically adjust resource requests
- Based on actual usage patterns
- Right-size containers over time

### Best Practices for Setting Resources:

```yaml
# Development Environment
resources:
  requests:
    memory: "128Mi"
    cpu: "100m"
  limits:
    memory: "256Mi"
    cpu: "500m"  # Allow bursting

# Production Environment
resources:
  requests:
    memory: "512Mi"
    cpu: "500m"
  limits:
    memory: "512Mi"  # Same as request (Guaranteed QoS)
    cpu: "1000m"     # Some burst capacity
```

---

## Resource Management Best Practices

### 1. Always Set Requests and Limits:

```yaml
# ✅ Good - Both set
resources:
  requests:
    memory: "15Mi"
    cpu: "100m"
  limits:
    memory: "20Mi"
    cpu: "100m"

# ❌ Bad - Nothing set (BestEffort QoS)
# No resources specified
```

### 2. Requests Based on Minimum Needs:

```
Strategy:
1. Start with conservative estimates
2. Monitor actual usage (kubectl top)
3. Adjust based on real data
4. Requests = Average usage
5. Limits = Peak usage + buffer
```

### 3. Set Limits to Prevent Resource Hogging:

```yaml
# ✅ Good - Limits prevent runaway consumption
resources:
  limits:
    memory: "20Mi"  # Killed if exceeds
    cpu: "100m"     # Throttled if exceeds

# ❌ Bad - No limits (can consume entire node)
resources:
  requests:
    memory: "15Mi"
  # No limits set!
```

### 4. Use Guaranteed QoS for Critical Workloads:

```yaml
# Critical app - Guaranteed QoS
resources:
  requests:
    memory: "1Gi"
    cpu: "1000m"
  limits:
    memory: "1Gi"   # Same as request
    cpu: "1000m"    # Same as request
```

### 5. Monitor Resource Usage:

```bash
# Regular monitoring
kubectl top pods
kubectl top nodes

# Set up alerts for:
# - High CPU usage (>80%)
# - High memory usage (>80%)
# - OOMKilled events
# - Pending pods due to resources
```

### 6. Consider Application Type:

```yaml
# Web Server (bursty traffic)
resources:
  requests:
    cpu: "100m"
  limits:
    cpu: "500m"  # Allow burst

# Database (consistent load)
resources:
  requests:
    cpu: "1000m"
  limits:
    cpu: "1000m"  # No burst needed

# Batch Job (resource-intensive)
resources:
  requests:
    cpu: "2000m"
    memory: "4Gi"
  limits:
    cpu: "4000m"
    memory: "8Gi"
```

### 7. Start Conservative, Scale Up:

```
Process:
1. Start with small requests (100m CPU, 128Mi memory)
2. Deploy and monitor
3. Check kubectl top pod regularly
4. If consistently hitting limits → Increase
5. If using much less → Decrease
```

### 8. Document Resource Decisions:

```yaml
metadata:
  annotations:
    resource-reasoning: |
      CPU: 100m - Average usage 50m, peaks to 80m
      Memory: 15Mi request, 20Mi limit - App uses 12-18Mi typically
      Last updated: 2025-12-30
      Based on: 1 week of monitoring data
```

---

## Real-World Resource Examples

### Nginx Web Server:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:latest
    resources:
      requests:
        memory: "64Mi"
        cpu: "100m"
      limits:
        memory: "128Mi"
        cpu: "500m"  # Allow bursting for traffic spikes
```

### Redis Cache:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: redis
spec:
  containers:
  - name: redis
    image: redis:latest
    resources:
      requests:
        memory: "512Mi"  # Cache needs memory
        cpu: "250m"
      limits:
        memory: "512Mi"  # Guaranteed QoS for cache
        cpu: "500m"
```

### Node.js API:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nodejs-api
spec:
  containers:
  - name: api
    image: node:18
    resources:
      requests:
        memory: "256Mi"
        cpu: "200m"
      limits:
        memory: "512Mi"
        cpu: "1000m"  # Node.js can benefit from more CPU
```

### Java Application:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: java-app
spec:
  containers:
  - name: app
    image: openjdk:17
    resources:
      requests:
        memory: "1Gi"    # JVM needs memory
        cpu: "500m"
      limits:
        memory: "2Gi"    # Allow for heap growth
        cpu: "2000m"
```

---

## Key Commands Reference

| Command | Description |
|---------|-------------|
| `kubectl apply -f pod.yaml` | Create pod with resources |
| `kubectl describe pod POD` | View resource config |
| `kubectl top pod POD` | View resource usage |
| `kubectl top nodes` | View node resource usage |
| `kubectl get pod POD -o yaml \| grep -A 10 resources` | View resources in YAML |
| `kubectl get pod POD -o jsonpath='{.status.qosClass}'` | Check QoS class |

---

## Completion Checklist

- [ ] Verified kubectl is configured
- [ ] Connected to Kubernetes cluster
- [ ] Created pod named `httpd-pod` (exact name)
- [ ] Container named `httpd-container` (exact name)
- [ ] Used image `httpd:latest` (explicit tag)
- [ ] Set memory request to `15Mi`
- [ ] Set CPU request to `100m`
- [ ] Set memory limit to `20Mi`
- [ ] Set CPU limit to `100m`
- [ ] Pod created successfully
- [ ] Pod status is Running
- [ ] Resources verified in describe output
- [ ] QoS class is Burstable
- [ ] No OOMKilled or resource issues
- [ ] All requirements met ✅

---

## Completion Details

- **Completion Date:** December 30, 2025
- **Day:** 50 / 100
- **Challenge:** KodeKloud 100 Days Cloud DevOps
- **Topic:** Kubernetes Resource Management - Requests and Limits
- **Platform:** Kubernetes
- **Host:** jump_host (kubectl pre-configured)
- **Pod Name:** httpd-pod
- **Container Name:** httpd-container
- **Image:** httpd:latest
- **Memory:** Request 15Mi, Limit 20Mi
- **CPU:** Request 100m, Limit 100m
- **QoS Class:** Burstable
- **Key Skill:** Resource management, capacity planning, QoS understanding
- **Status:** ✅ Successfully Completed

---

## Summary

This task demonstrated **Kubernetes resource management** - essential for production cluster stability and performance:

✅ **Set Resource Requests** - Minimum guaranteed resources (15Mi memory, 100m CPU)
✅ **Set Resource Limits** - Maximum allowed resources (20Mi memory, 100m CPU)
✅ **Understood QoS Classes** - Burstable (request < limit for memory)
✅ **Learned Resource Behavior** - CPU throttling vs Memory OOMKill
✅ **Production Best Practice** - Always specify resources!

**Key Insight:** Resource management is **critical for cluster stability**, preventing the "noisy neighbor" problem where one app starves others!

**Resource Management Fundamentals:**
```
Requests (Minimum Guaranteed):
├── Used by scheduler for placement
├── Guarantees container gets at least this
├── Node must have this available
└── Basis for QoS classification

Limits (Maximum Allowed):
├── Enforced by kubelet
├── CPU: Throttled if exceeded (slower)
├── Memory: OOMKilled if exceeded (terminated!)
└── Protects node from resource exhaustion
```

**CPU vs Memory Behavior:**

| Resource | Type | Exceeded Behavior |
|----------|------|-------------------|
| **CPU** | Compressible | Throttled (slower) ✅ Container keeps running |
| **Memory** | Incompressible | OOMKilled (terminated) ❌ Container restarted |

**QoS Classes:**
```
Our Configuration:
├── CPU: Request 100m = Limit 100m → Guaranteed
└── Memory: Request 15Mi < Limit 20Mi → Burstable

Overall: Burstable (at least one resource has request < limit)

Eviction Priority:
1. BestEffort (no requests/limits) - Evicted FIRST
2. Burstable (request < limit) - Evicted SECOND ← Our pod
3. Guaranteed (request = limit) - Evicted LAST
```

**Real-World Impact:**
```
Without Resources:
├── Pod A: Uses 90% CPU → Other pods starved
├── Pod B: Uses 3Gi memory → Node OOM
├── Scheduler: No idea where to place pods
└── Result: Cluster chaos! ❌

With Resources:
├── Pod A: Limited to 100m CPU → Fair sharing
├── Pod B: Limited to 20Mi memory → Protected node
├── Scheduler: Places pods on nodes with capacity
└── Result: Stable cluster! ✅
```

**Resource Units Quick Reference:**
```
CPU:
├── 100m = 0.1 CPU = 10% of one core
├── 500m = 0.5 CPU = 50% of one core
└── 1000m = 1 CPU = One full core

Memory:
├── 15Mi = 15,728,640 bytes ≈ 15 MB
├── 20Mi = 20,971,520 bytes ≈ 20 MB
└── 1Gi = 1,073,741,824 bytes ≈ 1 GB
```

**Scheduling Decision:**
```
Pod Needs: 15Mi memory, 100m CPU

Scheduler checks nodes:
Node1: 50Mi available, 200m CPU → ✅ Can fit
Node2: 10Mi available, 500m CPU → ❌ Insufficient memory
Node3: 100Mi available, 50m CPU → ❌ Insufficient CPU

Result: Pod placed on Node1 ✅
```

**Best Practices Applied:**
- ✅ Set both requests and limits (not just one)
- ✅ Requests based on minimum needs
- ✅ Limits prevent resource hogging
- ✅ Monitor actual usage to tune values
- ✅ Consider QoS for critical workloads

**Remember:** Resource management is **NOT optional** for production! Always set requests and limits to ensure predictable performance, fair resource sharing, and cluster stability! 🎯

**Next Steps:**
- Learn about ResourceQuotas (namespace limits)
- Explore LimitRanges (default values)
- Study Vertical Pod Autoscaler (VPA)
- Implement Horizontal Pod Autoscaler (HPA)
- Monitor resource usage patterns

**Next:** Advanced resource management, autoscaling, multi-container pods with resource sharing, and production capacity planning! ☸️
