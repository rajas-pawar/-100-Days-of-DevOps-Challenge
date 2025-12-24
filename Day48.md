# Day 48: Creating Kubernetes Pods
**100 Days Cloud DevOps Challenge – KodeKloud**

---

## Objective

Create a Kubernetes pod using kubectl command or YAML manifest. Learn fundamental Kubernetes concepts including pods, containers, labels, and kubectl operations.

**Requirements:**
1. Work on **jump_host** (kubectl already configured)
2. **Pod Name:** `pod-httpd`
3. **Image:** `httpd:latest` (Apache HTTP Server)
4. **Container Name:** `httpd-container`
5. **Label:** `app=httpd_app`
6. **Note:** Explicitly specify tag as `httpd:latest`

---

## Understanding Kubernetes

**Kubernetes (K8s)** is an open-source container orchestration platform that automates deployment, scaling, and management of containerized applications.

### What is Kubernetes?

```
Traditional Deployment:
├── Manual container management
├── Single host limitations
├── No automatic scaling
├── No self-healing
└── Complex networking

Kubernetes Solution:
├── ✅ Automated container orchestration
├── ✅ Multi-node cluster management
├── ✅ Automatic scaling (horizontal & vertical)
├── ✅ Self-healing (restart failed containers)
├── ✅ Service discovery & load balancing
├── ✅ Rolling updates & rollbacks
└── ✅ Declarative configuration
```

### Kubernetes Architecture:

```
Kubernetes Cluster
│
├── Control Plane (Master Node)
│   ├── API Server        ← Entry point for all requests
│   ├── etcd              ← Cluster state storage
│   ├── Scheduler         ← Assigns pods to nodes
│   ├── Controller Mgr    ← Maintains desired state
│   └── Cloud Controller  ← Cloud provider integration
│
└── Worker Nodes
    ├── Node 1
    │   ├── kubelet       ← Node agent
    │   ├── kube-proxy    ← Network proxy
    │   ├── Container Runtime (Docker/containerd)
    │   └── Pods          ← Running containers
    ├── Node 2
    └── Node 3
```

### Why Kubernetes?

| Feature | Docker Alone | Kubernetes |
|---------|-------------|------------|
| **Container Management** | Manual | Automated |
| **Scaling** | Manual | Automatic |
| **High Availability** | Manual setup | Built-in |
| **Load Balancing** | External tool | Native service |
| **Self-Healing** | Manual restart | Automatic |
| **Rolling Updates** | Manual | Declarative |
| **Multi-Host** | Complex | Native support |
| **Service Discovery** | Manual | Automatic |

---

## Understanding Kubernetes Pods

**Pod** is the smallest deployable unit in Kubernetes. A pod represents a single instance of a running process in your cluster.

### What is a Pod?

```
Pod = Wrapper around Container(s)

Single-Container Pod (Most Common):
┌─────────────────────────────┐
│         Pod                 │
│  ┌─────────────────────┐   │
│  │   Container         │   │
│  │   (httpd:latest)    │   │
│  │   Port: 80          │   │
│  └─────────────────────┘   │
│  Shared: Network, Storage  │
└─────────────────────────────┘

Multi-Container Pod (Sidecar Pattern):
┌─────────────────────────────┐
│         Pod                 │
│  ┌──────────┐ ┌──────────┐ │
│  │ Main App │ │ Logging  │ │
│  │          │ │ Sidecar  │ │
│  └──────────┘ └──────────┘ │
│  Shared: Network, Storage  │
└─────────────────────────────┘
```

### Pod Characteristics:

1. **Smallest Unit:** Can't split a pod across nodes
2. **Shared Network:** All containers share same IP and port space
3. **Shared Storage:** Can mount shared volumes
4. **Ephemeral:** Pods are temporary and replaceable
5. **Single Host:** All containers in pod run on same node

### Pod Lifecycle:

```
Pod Lifecycle States:

Pending
  ↓
  Scheduler assigns to node
  ↓
ContainerCreating
  ↓
  Pull image, create containers
  ↓
Running
  ↓
  All containers running
  ↓
Succeeded/Failed
  ↓
  Pod completed or failed
  ↓
Terminating
  ↓
Pod deleted
```

---

## Understanding Kubernetes Objects

### Core Kubernetes Objects:

| Object | Purpose | Example |
|--------|---------|---------|
| **Pod** | Run containers | Single httpd container |
| **ReplicaSet** | Maintain pod replicas | Keep 3 pods running |
| **Deployment** | Manage ReplicaSets | Rolling updates |
| **Service** | Expose pods | Load balancer |
| **ConfigMap** | Configuration data | App settings |
| **Secret** | Sensitive data | Passwords, tokens |
| **Volume** | Persistent storage | Database data |
| **Namespace** | Logical isolation | dev, staging, prod |

### Object Hierarchy:

```
Deployment
  ↓
  Creates & Manages
  ↓
ReplicaSet
  ↓
  Creates & Manages
  ↓
Pods
  ↓
  Run
  ↓
Containers
```

**For this task:** We're creating a standalone **Pod** (simplest object)

---

## Understanding Labels and Selectors

**Labels** are key-value pairs attached to Kubernetes objects for identification and organization.

### What are Labels?

```
Labels = Metadata Tags

Example Pod with Labels:
┌─────────────────────────────┐
│  Pod: pod-httpd             │
│                             │
│  Labels:                    │
│  ├── app: httpd_app         │ ← Our task requirement
│  ├── env: production        │
│  ├── version: v1.0          │
│  └── tier: frontend         │
│                             │
│  Container: httpd-container │
└─────────────────────────────┘
```

### Why Labels?

```
Use Cases:

1. Selection:
   kubectl get pods -l app=httpd_app
   → Returns only pods with app=httpd_app label

2. Grouping:
   Group all frontend pods: tier=frontend
   Group all production: env=production

3. Service Routing:
   Service routes traffic to pods matching labels

4. Organization:
   Filter by team, project, environment, version
```

### Label Best Practices:

```
Good Labels:
✅ app: httpd_app              (Application name)
✅ env: production             (Environment)
✅ version: v1.0               (Version)
✅ component: web              (Component type)
✅ managed-by: helm            (Management tool)

Bad Labels:
❌ label: value                (Too generic)
❌ 123: test                   (Numbers as keys)
❌ very-long-label-name-that-exceeds-63-characters: value
```

---

## Understanding kubectl

**kubectl** is the Kubernetes command-line tool for interacting with clusters.

### kubectl Syntax:

```
kubectl [COMMAND] [TYPE] [NAME] [FLAGS]

Examples:
kubectl get pods                    ← Get all pods
kubectl get pod pod-httpd          ← Get specific pod
kubectl describe pod pod-httpd     ← Detailed info
kubectl delete pod pod-httpd       ← Delete pod
kubectl logs pod-httpd             ← View logs
```

### Common kubectl Commands:

| Command | Purpose | Example |
|---------|---------|---------|
| **create** | Create resource | `kubectl create -f pod.yaml` |
| **run** | Run pod quickly | `kubectl run mypod --image=nginx` |
| **get** | List resources | `kubectl get pods` |
| **describe** | Detailed info | `kubectl describe pod mypod` |
| **logs** | Container logs | `kubectl logs mypod` |
| **exec** | Execute command | `kubectl exec mypod -- ls` |
| **delete** | Delete resource | `kubectl delete pod mypod` |
| **apply** | Apply config | `kubectl apply -f pod.yaml` |
| **edit** | Edit resource | `kubectl edit pod mypod` |

### kubectl Configuration:

```
kubectl uses kubeconfig file for cluster access:

Default location: ~/.kube/config

Contains:
├── Cluster info (API server URL, certificate)
├── User credentials (token, certificate)
└── Context (cluster + user + namespace)

Check configuration:
kubectl config view
kubectl cluster-info
kubectl version
```

---

## Creating Pods: Two Methods

### Method 1: Imperative (kubectl run)

**Pros:**
- ✅ Fast for simple pods
- ✅ No YAML knowledge needed
- ✅ Good for testing/development

**Cons:**
- ❌ Limited customization
- ❌ Not version controlled
- ❌ Hard to reproduce

```bash
kubectl run pod-httpd \
  --image=httpd:latest \
  --labels=app=httpd_app \
  --port=80
```

### Method 2: Declarative (YAML manifest)

**Pros:**
- ✅ Full control over configuration
- ✅ Version controlled (Git)
- ✅ Reproducible and shareable
- ✅ Production best practice

**Cons:**
- ❌ Requires YAML knowledge
- ❌ More initial setup

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-httpd
  labels:
    app: httpd_app
spec:
  containers:
  - name: httpd-container
    image: httpd:latest
```

**For this task:** Both methods work, but YAML is recommended for production!

---

## Understanding YAML for Kubernetes

### YAML Basics:

```yaml
# Key-value pairs
name: pod-httpd

# Lists
containers:
  - name: container1
  - name: container2

# Nested structures
metadata:
  name: pod-httpd
  labels:
    app: httpd_app
```

### Kubernetes Pod YAML Structure:

```yaml
apiVersion: v1              # API version (v1 for core objects)
kind: Pod                   # Object type (Pod, Deployment, Service)
metadata:                   # Object metadata
  name: pod-httpd          # Required: Pod name
  labels:                  # Optional: Labels
    app: httpd_app         # Key-value pairs
spec:                      # Pod specification
  containers:              # List of containers
  - name: httpd-container  # Container name
    image: httpd:latest    # Docker image
    ports:                 # Optional: Exposed ports
    - containerPort: 80    # Port number
```

### Required vs Optional Fields:

```yaml
Required:
├── apiVersion    # Which API version to use
├── kind          # Type of object
├── metadata
│   └── name      # Unique name
└── spec
    └── containers
        ├── name  # Container name
        └── image # Container image

Optional:
├── metadata.labels        # Labels for selection
├── spec.containers.ports  # Port definitions
├── spec.restartPolicy     # Restart behavior
└── spec.volumes           # Storage volumes
```

---

## Infrastructure Overview

### KodeKloud Kubernetes Environment:

| Component | Details |
|-----------|---------|
| **jump_host** | Client machine with kubectl configured |
| **Kubernetes Cluster** | Pre-configured K8s cluster |
| **kubectl** | Already installed and configured |
| **Context** | Default namespace |

### Task Details:

| Item | Value |
|------|-------|
| **Host** | jump_host (kubectl pre-configured) |
| **Pod Name** | `pod-httpd` (exact name) |
| **Image** | `httpd:latest` (Apache HTTP Server) |
| **Container Name** | `httpd-container` (exact name) |
| **Label** | `app=httpd_app` (exact key-value) |
| **Method** | Imperative OR Declarative |
| **Namespace** | default (not specified) |

---

## Step-by-Step Implementation

### Method 1: Imperative Command (Quick)

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

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

**✅ Connected to Kubernetes cluster**

#### Step 3: Check Current Pods (Optional)
```bash
# List existing pods
kubectl get pods
```

**Expected output (if no pods exist):**
```
No resources found in default namespace.
```

**Or (if pods exist):**
```
NAME          READY   STATUS    RESTARTS   AGE
example-pod   1/1     Running   0          5m
```

#### Step 4: Create Pod Using Imperative Command
```bash
# Create pod with all requirements
kubectl run pod-httpd \
  --image=httpd:latest \
  --labels=app=httpd_app \
  --dry-run=client -o yaml > pod-definition.yaml
```

**Wait!** The above generates YAML. Let's use direct creation:

```bash
# Create pod directly (one-line version)
kubectl run pod-httpd --image=httpd:latest --labels=app=httpd_app
```

**Expected output:**
```
pod/pod-httpd created
```

**✅ Pod created successfully!**

**Note:** The `kubectl run` command doesn't allow specifying container name directly. To set container name as `httpd-container`, we need to use YAML method (Method 2).

---

### Method 2: Declarative YAML (Recommended)

#### Step 1: Create Pod YAML Manifest
```bash
# Create pod-httpd.yaml file
cat > pod-httpd.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: pod-httpd
  labels:
    app: httpd_app
spec:
  containers:
  - name: httpd-container
    image: httpd:latest
    ports:
    - containerPort: 80
EOF
```

**Expected output:**
```
(No output means success)
```

**✅ YAML manifest created**

#### Step 2: Verify YAML File
```bash
cat pod-httpd.yaml
```

**Expected output:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-httpd
  labels:
    app: httpd_app
spec:
  containers:
  - name: httpd-container
    image: httpd:latest
    ports:
    - containerPort: 80
```

**Verify:**
- ✅ name: pod-httpd
- ✅ labels.app: httpd_app
- ✅ containers.name: httpd-container
- ✅ image: httpd:latest
- ✅ Proper YAML indentation

#### Step 3: Validate YAML (Optional)
```bash
# Dry-run to validate without creating
kubectl apply -f pod-httpd.yaml --dry-run=client
```

**Expected output:**
```
pod/pod-httpd created (dry run)
```

**✅ YAML is valid**

#### Step 4: Create Pod from YAML
```bash
# Apply the manifest
kubectl apply -f pod-httpd.yaml
```

**Expected output:**
```
pod/pod-httpd created
```

**Alternative command:**
```bash
kubectl create -f pod-httpd.yaml
```

**Expected output:**
```
pod/pod-httpd created
```

**Difference between apply and create:**
- `create` - Creates new resource (fails if exists)
- `apply` - Creates or updates (idempotent)

**✅ Pod created successfully!**

---

## Verification Steps

### Step 5: List All Pods
```bash
kubectl get pods
```

**Expected output:**
```
NAME        READY   STATUS    RESTARTS   AGE
pod-httpd   1/1     Running   0          30s
```

**Status meanings:**
- **Pending** - Waiting for scheduler
- **ContainerCreating** - Pulling image, starting container
- **Running** - Pod is running
- **Succeeded** - Pod completed successfully
- **Failed** - Pod failed
- **CrashLoopBackOff** - Container keeps crashing

**✅ Pod is Running with 1/1 containers ready**

### Step 6: Get Pod with Labels
```bash
kubectl get pods --show-labels
```

**Expected output:**
```
NAME        READY   STATUS    RESTARTS   AGE   LABELS
pod-httpd   1/1     Running   0          1m    app=httpd_app
```

**✅ Label app=httpd_app is present**

### Step 7: Filter Pods by Label
```bash
# Get pods with specific label
kubectl get pods -l app=httpd_app
```

**Expected output:**
```
NAME        READY   STATUS    RESTARTS   AGE
pod-httpd   1/1     Running   0          2m
```

**✅ Pod found using label selector**

### Step 8: Describe Pod Details
```bash
kubectl describe pod pod-httpd
```

**Expected output:**
```
Name:             pod-httpd
Namespace:        default
Priority:         0
Service Account:  default
Node:             node01/192.168.1.10
Start Time:       Tue, 24 Dec 2025 10:00:00 +0000
Labels:           app=httpd_app
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
      Started:      Tue, 24 Dec 2025 10:00:15 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
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
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  2m    default-scheduler  Successfully assigned default/pod-httpd to node01
  Normal  Pulling    2m    kubelet            Pulling image "httpd:latest"
  Normal  Pulled     2m    kubelet            Successfully pulled image "httpd:latest" in 10.5s
  Normal  Created    2m    kubelet            Created container httpd-container
  Normal  Started    2m    kubelet            Started container httpd-container
```

**Verify key details:**
- ✅ Name: pod-httpd
- ✅ Labels: app=httpd_app
- ✅ Container Name: httpd-container
- ✅ Image: httpd:latest
- ✅ Status: Running
- ✅ Events show successful pull and start

### Step 9: Check Pod Logs
```bash
kubectl logs pod-httpd
```

**Expected output:**
```
AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 10.244.0.5. Set the 'ServerName' directive globally to suppress this message
AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 10.244.0.5. Set the 'ServerName' directive globally to suppress this message
[Tue Dec 24 10:00:15.123456 2025] [mpm_event:notice] [pid 1:tid 140123456789] AH00489: Apache/2.4.58 (Unix) configured -- resuming normal operations
[Tue Dec 24 10:00:15.123789 2025] [core:notice] [pid 1:tid 140123456789] AH00094: Command line: 'httpd -D FOREGROUND'
```

**✅ Apache HTTP Server started successfully**

**Note:** The "Could not reliably determine" warning is normal and harmless.

### Step 10: Get Pod YAML
```bash
# View complete pod configuration
kubectl get pod pod-httpd -o yaml
```

**Expected output (excerpt):**
```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2025-12-24T10:00:00Z"
  labels:
    app: httpd_app
  name: pod-httpd
  namespace: default
  resourceVersion: "12345"
  uid: abc123-def4-5678-90ab-cdef12345678
spec:
  containers:
  - image: httpd:latest
    imagePullPolicy: Always
    name: httpd-container
    ports:
    - containerPort: 80
      protocol: TCP
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-xxxxx
      readOnly: true
  dnsPolicy: ClusterFirst
  enableServiceLinks: true
  nodeName: node01
  preemptionPolicy: PreemptLowerPriority
  priority: 0
  restartPolicy: Always
  schedulerName: default-scheduler
  securityContext: {}
  serviceAccount: default
  serviceAccountName: default
  terminationGracePeriodSeconds: 30
  tolerations:
  - effect: NoExecute
    key: node.kubernetes.io/not-ready
    operator: Exists
    tolerationSeconds: 300
  - effect: NoExecute
    key: node.kubernetes.io/unreachable
    operator: Exists
    tolerationSeconds: 300
  volumes:
  - name: kube-api-access-xxxxx
    projected:
      defaultMode: 420
      sources:
      - serviceAccountToken:
          expirationSeconds: 3607
          path: token
      - configMap:
          items:
          - key: ca.crt
            path: ca.crt
          name: kube-root-ca.crt
      - downwardAPI:
          items:
          - fieldRef:
              apiVersion: v1
              fieldPath: metadata.namespace
            path: namespace
status:
  conditions:
  - lastProbeTime: null
    lastTransitionTime: "2025-12-24T10:00:00Z"
    status: "True"
    type: Initialized
  - lastProbeTime: null
    lastTransitionTime: "2025-12-24T10:00:15Z"
    status: "True"
    type: Ready
  - lastProbeTime: null
    lastTransitionTime: "2025-12-24T10:00:15Z"
    status: "True"
    type: ContainersReady
  - lastProbeTime: null
    lastTransitionTime: "2025-12-24T10:00:00Z"
    status: "True"
    type: PodScheduled
  containerStatuses:
  - containerID: containerd://abc123def456789...
    image: docker.io/library/httpd:latest
    imageID: docker.io/library/httpd@sha256:789abc123def...
    lastState: {}
    name: httpd-container
    ready: true
    restartCount: 0
    started: true
    state:
      running:
        startedAt: "2025-12-24T10:00:15Z"
  hostIP: 192.168.1.10
  phase: Running
  podIP: 10.244.0.5
  podIPs:
  - ip: 10.244.0.5
  qosClass: BestEffort
  startTime: "2025-12-24T10:00:00Z"
```

**Key observations:**
- ✅ Complete pod configuration with defaults
- ✅ Container name: httpd-container
- ✅ Image: httpd:latest
- ✅ Label: app=httpd_app
- ✅ Pod IP assigned: 10.244.0.5
- ✅ Running on node01

### Step 11: Get Pod in JSON Format
```bash
# View as JSON
kubectl get pod pod-httpd -o json
```

**Or get specific fields:**
```bash
# Get pod name and image
kubectl get pod pod-httpd -o jsonpath='{.metadata.name}{"\t"}{.spec.containers[0].image}{"\n"}'
```

**Expected output:**
```
pod-httpd       httpd:latest
```

### Step 12: Execute Command in Pod
```bash
# Test Apache is running
kubectl exec pod-httpd -- httpd -v
```

**Expected output:**
```
Server version: Apache/2.4.58 (Unix)
Server built:   Dec 15 2025 00:00:00
```

**✅ Apache HTTP Server running inside container**

### Step 13: Access Pod Shell (Optional)
```bash
# Interactive shell
kubectl exec -it pod-httpd -- /bin/bash
```

**Expected output:**
```
root@pod-httpd:/usr/local/apache2#
```

**Inside container, you can:**
```bash
# Check Apache process
ps aux | grep httpd

# Check listening ports
netstat -tuln | grep 80

# View Apache config
cat /usr/local/apache2/conf/httpd.conf

# Exit container
exit
```

### Step 14: Port Forward to Access Locally (Optional)
```bash
# Forward local port 8080 to pod port 80
kubectl port-forward pod-httpd 8080:80
```

**Expected output:**
```
Forwarding from 127.0.0.1:8080 -> 80
Forwarding from [::1]:8080 -> 80
```

**In another terminal:**
```bash
curl http://localhost:8080
```

**Expected output:**
```html
<html><body><h1>It works!</h1></body></html>
```

**✅ Apache HTTP Server accessible!**

**Press Ctrl+C to stop port forwarding**

### Step 15: Final Verification Checklist
```bash
# 1. Pod exists with correct name
kubectl get pod pod-httpd &> /dev/null && echo "✅ Pod exists" || echo "❌ Pod not found"

# 2. Pod is running
kubectl get pod pod-httpd -o jsonpath='{.status.phase}' | grep -q "Running" && echo "✅ Pod running" || echo "❌ Pod not running"

# 3. Label is correct
kubectl get pod pod-httpd -o jsonpath='{.metadata.labels.app}' | grep -q "httpd_app" && echo "✅ Label correct" || echo "❌ Label incorrect"

# 4. Image is correct
kubectl get pod pod-httpd -o jsonpath='{.spec.containers[0].image}' | grep -q "httpd:latest" && echo "✅ Image correct" || echo "❌ Image incorrect"

# 5. Container name is correct
kubectl get pod pod-httpd -o jsonpath='{.spec.containers[0].name}' | grep -q "httpd-container" && echo "✅ Container name correct" || echo "❌ Container name incorrect"

# 6. Container is ready
kubectl get pod pod-httpd -o jsonpath='{.status.containerStatuses[0].ready}' | grep -q "true" && echo "✅ Container ready" || echo "❌ Container not ready"
```

**All checks should pass:**
```
✅ Pod exists
✅ Pod running
✅ Label correct
✅ Image correct
✅ Container name correct
✅ Container ready
```

---

## Complete Command Summary

### Quick Deployment (Imperative):
```bash
# Create pod directly
kubectl run pod-httpd --image=httpd:latest --labels=app=httpd_app

# Verify
kubectl get pods
kubectl describe pod pod-httpd
```

**Note:** This method doesn't allow custom container name. Use YAML for full control.

### Recommended Deployment (Declarative):
```bash
# Create YAML manifest
cat > pod-httpd.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: pod-httpd
  labels:
    app: httpd_app
spec:
  containers:
  - name: httpd-container
    image: httpd:latest
    ports:
    - containerPort: 80
EOF

# Create pod
kubectl apply -f pod-httpd.yaml

# Verify
kubectl get pods
kubectl get pods --show-labels
kubectl describe pod pod-httpd
kubectl logs pod-httpd
```

### Detailed Workflow:
```bash
# 1. Verify kubectl access
kubectl version --client
kubectl cluster-info

# 2. Check current state
kubectl get pods
kubectl get all

# 3. Create pod manifest
vi pod-httpd.yaml
# (Paste YAML content)

# 4. Validate YAML
kubectl apply -f pod-httpd.yaml --dry-run=client

# 5. Create pod
kubectl apply -f pod-httpd.yaml

# 6. Watch pod creation
kubectl get pods -w

# 7. Check pod details
kubectl get pods
kubectl get pods --show-labels
kubectl get pods -l app=httpd_app
kubectl describe pod pod-httpd

# 8. Check logs
kubectl logs pod-httpd
kubectl logs pod-httpd -f  # Follow logs

# 9. Test pod
kubectl exec pod-httpd -- httpd -v
kubectl port-forward pod-httpd 8080:80
curl http://localhost:8080

# 10. View complete config
kubectl get pod pod-httpd -o yaml
```

---

## Understanding Pod YAML Deep Dive

### Complete Pod YAML with All Options:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-httpd
  namespace: default
  labels:
    app: httpd_app
    env: production
    version: v1
  annotations:
    description: "Apache HTTP Server pod"
    owner: "devops-team"
spec:
  # Container specification
  containers:
  - name: httpd-container
    image: httpd:latest
    imagePullPolicy: Always  # Always, IfNotPresent, Never
    
    # Ports
    ports:
    - containerPort: 80
      protocol: TCP
      name: http
    
    # Resource limits
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
    
    # Environment variables
    env:
    - name: APACHE_LOG_LEVEL
      value: "info"
    
    # Health checks
    livenessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 10
      periodSeconds: 10
    
    readinessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 5
    
    # Volume mounts
    volumeMounts:
    - name: html
      mountPath: /usr/local/apache2/htdocs
  
  # Pod-level settings
  restartPolicy: Always  # Always, OnFailure, Never
  
  # Volumes
  volumes:
  - name: html
    emptyDir: {}
  
  # Node selection
  nodeSelector:
    disktype: ssd
  
  # Tolerations
  tolerations:
  - key: "example-key"
    operator: "Exists"
    effect: "NoSchedule"
```

### Field Explanations:

| Field | Purpose | Common Values |
|-------|---------|---------------|
| **apiVersion** | API version | `v1` for core objects |
| **kind** | Object type | `Pod`, `Deployment`, `Service` |
| **metadata.name** | Unique identifier | `pod-httpd` |
| **metadata.labels** | Key-value tags | `app: httpd_app` |
| **spec.containers** | Container list | Array of containers |
| **containers.name** | Container name | `httpd-container` |
| **containers.image** | Docker image | `httpd:latest` |
| **containers.ports** | Exposed ports | `containerPort: 80` |
| **spec.restartPolicy** | Restart behavior | `Always`, `OnFailure`, `Never` |
| **spec.volumes** | Storage volumes | EmptyDir, PersistentVolume |

---

## kubectl Commands Reference

### Pod Management:

```bash
# Create pod
kubectl run NAME --image=IMAGE                    # Imperative
kubectl create -f pod.yaml                        # Declarative (create)
kubectl apply -f pod.yaml                         # Declarative (apply)

# List pods
kubectl get pods                                  # All pods
kubectl get pods -o wide                          # With more details
kubectl get pods --show-labels                    # With labels
kubectl get pods -l app=httpd_app                 # Filter by label
kubectl get pods -A                               # All namespaces
kubectl get pods -n namespace-name                # Specific namespace

# Pod details
kubectl describe pod NAME                         # Detailed info
kubectl get pod NAME -o yaml                      # YAML format
kubectl get pod NAME -o json                      # JSON format

# Pod logs
kubectl logs NAME                                 # View logs
kubectl logs NAME -f                              # Follow logs
kubectl logs NAME --previous                      # Previous container
kubectl logs NAME -c container-name               # Specific container

# Execute commands
kubectl exec NAME -- COMMAND                      # Execute command
kubectl exec -it NAME -- /bin/bash                # Interactive shell

# Port forwarding
kubectl port-forward NAME LOCAL:REMOTE            # Forward ports
kubectl port-forward pod-httpd 8080:80            # Example

# Delete pod
kubectl delete pod NAME                           # Delete by name
kubectl delete -f pod.yaml                        # Delete by file
kubectl delete pods --all                         # Delete all pods
```

### Troubleshooting Commands:

```bash
# Check pod status
kubectl get pod NAME
kubectl describe pod NAME
kubectl get events

# Debug failing pods
kubectl logs NAME
kubectl logs NAME --previous
kubectl get pod NAME -o yaml
kubectl describe pod NAME | grep -A 10 Events

# Network debugging
kubectl exec NAME -- ping 8.8.8.8
kubectl exec NAME -- curl http://example.com
kubectl exec NAME -- nslookup kubernetes.default

# Resource usage
kubectl top pods                    # Requires metrics-server
kubectl top pod NAME
```

### Advanced Operations:

```bash
# Edit running pod
kubectl edit pod NAME

# Replace pod
kubectl replace -f pod.yaml --force

# Patch pod
kubectl patch pod NAME -p '{"spec":{"containers":[{"name":"httpd-container","image":"httpd:2.4"}]}}'

# Label operations
kubectl label pod NAME key=value                  # Add label
kubectl label pod NAME key=value --overwrite      # Update label
kubectl label pod NAME key-                       # Remove label

# Annotation operations
kubectl annotate pod NAME key=value               # Add annotation
kubectl annotate pod NAME key-                    # Remove annotation

# Watch pods
kubectl get pods -w                               # Watch changes
kubectl get pods -w -o wide                       # Watch with details
```

---

## Troubleshooting Guide

### Issue 1: Pod Stuck in Pending State

**Problem:**
```bash
kubectl get pods
NAME        READY   STATUS    RESTARTS   AGE
pod-httpd   0/1     Pending   0          2m
```

**Diagnosis:**
```bash
kubectl describe pod pod-httpd
```

**Common causes:**

**A. Insufficient resources:**
```
Events:
  Warning  FailedScheduling  2m  default-scheduler  0/3 nodes available: insufficient cpu
```

**Solution:**
```bash
# Check node resources
kubectl top nodes
kubectl describe nodes

# Reduce resource requests in pod spec
```

**B. No available nodes:**
```
Events:
  Warning  FailedScheduling  2m  default-scheduler  no nodes available to schedule pods
```

**Solution:**
```bash
# Check nodes
kubectl get nodes
kubectl describe nodes

# Ensure nodes are Ready
```

**C. Image pull errors:**
```
Events:
  Warning  Failed  2m  kubelet  Failed to pull image "httpd:latest"
```

**Solution:**
```bash
# Check image name
# Check registry access
# Check image pull secrets
```

### Issue 2: Pod in CrashLoopBackOff

**Problem:**
```bash
kubectl get pods
NAME        READY   STATUS             RESTARTS   AGE
pod-httpd   0/1     CrashLoopBackOff   5          5m
```

**Diagnosis:**
```bash
# Check logs
kubectl logs pod-httpd
kubectl logs pod-httpd --previous

# Check events
kubectl describe pod pod-httpd
```

**Common causes:**

**A. Application error:**
```
Logs show: Error: Failed to start application
```

**Solution:**
```bash
# Fix application code
# Check environment variables
# Check configuration
```

**B. Wrong command:**
```yaml
# Pod spec with wrong command
spec:
  containers:
  - name: httpd-container
    image: httpd:latest
    command: ["/bin/bash"]  # Wrong, exits immediately
```

**Solution:**
```yaml
# Remove command or use correct command
spec:
  containers:
  - name: httpd-container
    image: httpd:latest
    # Let image default command run
```

### Issue 3: Image Pull Error

**Problem:**
```bash
kubectl get pods
NAME        READY   STATUS         RESTARTS   AGE
pod-httpd   0/1     ErrImagePull   0          1m
```

**Diagnosis:**
```bash
kubectl describe pod pod-httpd
```

**Events:**
```
Warning  Failed     1m    kubelet  Failed to pull image "httpd:latest": rpc error: code = Unknown desc = Error response from daemon: pull access denied for httpd, repository does not exist or may require 'docker login'
```

**Common causes:**

**A. Wrong image name:**
```yaml
image: httpd:latests  # Typo
```

**Solution:**
```yaml
image: httpd:latest  # Correct
```

**B. Private registry without credentials:**
```yaml
image: private-registry.com/httpd:latest
```

**Solution:**
```bash
# Create secret
kubectl create secret docker-registry regcred \
  --docker-server=private-registry.com \
  --docker-username=user \
  --docker-password=pass

# Use in pod
spec:
  imagePullSecrets:
  - name: regcred
  containers:
  - name: httpd-container
    image: private-registry.com/httpd:latest
```

**C. Network issues:**
```
Failed to pull image: dial tcp: lookup registry-1.docker.io: no such host
```

**Solution:**
```bash
# Check node network
# Check DNS resolution
# Check registry accessibility
```

### Issue 4: Container Not Ready

**Problem:**
```bash
kubectl get pods
NAME        READY   STATUS    RESTARTS   AGE
pod-httpd   0/1     Running   0          5m
```

**Pod is Running but not Ready**

**Diagnosis:**
```bash
kubectl describe pod pod-httpd
```

**Common causes:**

**A. Readiness probe failing:**
```
Readiness probe failed: HTTP probe failed with statuscode: 404
```

**Solution:**
```bash
# Check readiness probe configuration
# Ensure app is healthy
# Adjust probe settings
```

**B. Application starting slowly:**
```yaml
# Increase initialDelaySeconds
readinessProbe:
  httpGet:
    path: /
    port: 80
  initialDelaySeconds: 30  # Give more time
  periodSeconds: 10
```

### Issue 5: Can't Delete Pod

**Problem:**
```bash
kubectl delete pod pod-httpd
# Pod stuck in Terminating state
```

**Diagnosis:**
```bash
kubectl describe pod pod-httpd
```

**Solution:**
```bash
# Force delete
kubectl delete pod pod-httpd --grace-period=0 --force
```

**Warning:** Only use force delete when necessary!

### Issue 6: Label Not Applied

**Problem:**
```bash
kubectl get pods -l app=httpd_app
# No resources found
```

**Diagnosis:**
```bash
kubectl get pods --show-labels
# Label is missing or different
```

**Solution:**
```bash
# Add label to running pod
kubectl label pod pod-httpd app=httpd_app

# Or delete and recreate with correct label
kubectl delete pod pod-httpd
kubectl apply -f pod-httpd.yaml
```

### Issue 7: Wrong Container Name

**Problem:**
You created pod with `kubectl run` but container name is wrong.

**Diagnosis:**
```bash
kubectl get pod pod-httpd -o jsonpath='{.spec.containers[0].name}'
# Shows: pod-httpd (not httpd-container)
```

**Solution:**
```bash
# kubectl run doesn't support custom container names
# Must use YAML manifest

# Delete pod
kubectl delete pod pod-httpd

# Create with YAML
kubectl apply -f pod-httpd.yaml
```

---

## Advanced Pod Concepts

### Pod Resource Management:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-httpd
  labels:
    app: httpd_app
spec:
  containers:
  - name: httpd-container
    image: httpd:latest
    
    # Resource requests (minimum guaranteed)
    resources:
      requests:
        memory: "64Mi"     # 64 Megabytes
        cpu: "250m"        # 250 millicores (0.25 CPU)
      
      # Resource limits (maximum allowed)
      limits:
        memory: "128Mi"    # 128 Megabytes
        cpu: "500m"        # 500 millicores (0.5 CPU)
```

**Benefits:**
- Ensures pod gets minimum resources
- Prevents pod from consuming all node resources
- Helps scheduler make better placement decisions

### Pod Health Checks:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-httpd
  labels:
    app: httpd_app
spec:
  containers:
  - name: httpd-container
    image: httpd:latest
    
    # Liveness Probe: Is container alive?
    livenessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 10
      periodSeconds: 10
      failureThreshold: 3
    
    # Readiness Probe: Is container ready to serve?
    readinessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 5
      failureThreshold: 3
    
    # Startup Probe: Has container started?
    startupProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 0
      periodSeconds: 10
      failureThreshold: 30
```

**Probe Types:**
- **Liveness:** Restarts container if unhealthy
- **Readiness:** Removes from service if not ready
- **Startup:** Waits for slow-starting containers

### Pod Restart Policies:

```yaml
spec:
  restartPolicy: Always  # Always, OnFailure, Never
```

| Policy | Behavior | Use Case |
|--------|----------|----------|
| **Always** | Always restart on exit | Long-running services |
| **OnFailure** | Restart only if failed | Batch jobs |
| **Never** | Never restart | One-time tasks |

### Multi-Container Pods:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-container-pod
spec:
  containers:
  # Main application container
  - name: app
    image: myapp:latest
    ports:
    - containerPort: 8080
  
  # Sidecar: Log aggregator
  - name: log-aggregator
    image: fluentd:latest
    volumeMounts:
    - name: logs
      mountPath: /var/log
  
  # Sidecar: Metrics exporter
  - name: metrics
    image: prometheus-exporter:latest
    ports:
    - containerPort: 9090
  
  volumes:
  - name: logs
    emptyDir: {}
```

**Common Multi-Container Patterns:**
1. **Sidecar:** Helper container (logging, monitoring)
2. **Ambassador:** Proxy for external services
3. **Adapter:** Normalize output for monitoring

---

## Best Practices Summary

### Pod Best Practices:

1. **Always Use Labels:**
   ```yaml
   metadata:
     labels:
       app: httpd_app
       env: production
       version: v1
   ```

2. **Specify Image Tags:**
   ```yaml
   image: httpd:2.4.58  # ✅ Specific version
   # NOT: httpd:latest  # ❌ Unpredictable
   ```
   
   **Exception:** This task requires `httpd:latest`

3. **Set Resource Limits:**
   ```yaml
   resources:
     requests:
       memory: "64Mi"
       cpu: "250m"
     limits:
       memory: "128Mi"
       cpu: "500m"
   ```

4. **Add Health Checks:**
   ```yaml
   livenessProbe:
     httpGet:
       path: /healthz
       port: 8080
   readinessProbe:
     httpGet:
       path: /ready
       port: 8080
   ```

5. **Use Declarative Configuration:**
   ```bash
   kubectl apply -f pod.yaml  # ✅ Version controlled
   # NOT: kubectl run ...     # ❌ Hard to track
   ```

6. **Meaningful Names:**
   ```yaml
   metadata:
     name: httpd-web-server  # ✅ Descriptive
   # NOT: pod1               # ❌ Generic
   ```

7. **Document with Annotations:**
   ```yaml
   metadata:
     annotations:
       description: "Apache HTTP Server"
       owner: "devops-team@example.com"
       runbook: "https://wiki.example.com/httpd"
   ```

### YAML Best Practices:

1. **Use 2-space indentation** (YAML standard)
2. **Validate before applying** (`--dry-run=client`)
3. **Version control** (Git)
4. **Add comments** for complex configurations
5. **Use templates** (Helm, Kustomize)

### kubectl Best Practices:

1. **Use declarative commands** (`apply` vs `create`)
2. **Check before deleting** (`describe` first)
3. **Use labels for filtering** (`-l app=httpd_app`)
4. **Watch for changes** (`-w` flag)
5. **Use contexts** for multiple clusters

---

## Real-World Applications

### Development Workflow:

```bash
# 1. Create pod manifest
cat > pod.yaml << EOF
apiVersion: v1
kind: Pod
metadata:
  name: dev-app
  labels:
    app: dev-app
    env: development
spec:
  containers:
  - name: app
    image: myapp:dev
    ports:
    - containerPort: 8080
EOF

# 2. Create pod
kubectl apply -f pod.yaml

# 3. Test changes
kubectl port-forward dev-app 8080:8080
curl http://localhost:8080

# 4. Check logs
kubectl logs dev-app -f

# 5. Update and redeploy
kubectl delete -f pod.yaml
kubectl apply -f pod.yaml
```

### Production Considerations:

**Don't use standalone pods in production!**

```
Use instead:
├── Deployments      ← For stateless apps
├── StatefulSets     ← For stateful apps
├── DaemonSets       ← For node agents
└── Jobs/CronJobs    ← For batch tasks

Why?
├── ✅ Automatic scaling
├── ✅ Rolling updates
├── ✅ Self-healing
└── ✅ Declarative management
```

### Migration from Docker to Kubernetes:

```bash
# Docker command:
docker run -d --name my-app -p 8080:80 httpd:latest

# Equivalent Kubernetes pod:
apiVersion: v1
kind: Pod
metadata:
  name: my-app
spec:
  containers:
  - name: my-app
    image: httpd:latest
    ports:
    - containerPort: 80

# Better: Use Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-app
        image: httpd:latest
        ports:
        - containerPort: 80
```

### CI/CD Integration:

```yaml
# .github/workflows/deploy.yml
name: Deploy to Kubernetes

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      
      - name: Set up kubectl
        uses: azure/setup-kubectl@v1
      
      - name: Configure kubectl
        run: |
          echo "${{ secrets.KUBECONFIG }}" > kubeconfig
          export KUBECONFIG=kubeconfig
      
      - name: Deploy pod
        run: kubectl apply -f pod-httpd.yaml
      
      - name: Verify deployment
        run: |
          kubectl wait --for=condition=Ready pod/pod-httpd --timeout=60s
          kubectl get pod pod-httpd
```

---

## Key Commands Reference

| Command | Description |
|---------|-------------|
| `kubectl run NAME --image=IMAGE` | Create pod imperatively |
| `kubectl apply -f pod.yaml` | Create/update pod from YAML |
| `kubectl get pods` | List pods |
| `kubectl get pods -l app=httpd_app` | Filter by label |
| `kubectl describe pod NAME` | Detailed pod info |
| `kubectl logs NAME` | View container logs |
| `kubectl exec NAME -- COMMAND` | Execute command in pod |
| `kubectl port-forward NAME LOCAL:REMOTE` | Forward ports |
| `kubectl delete pod NAME` | Delete pod |
| `kubectl get pod NAME -o yaml` | View pod YAML |

---

## Completion Checklist

- [ ] Verified kubectl is configured
- [ ] Connected to Kubernetes cluster
- [ ] Created pod-httpd.yaml manifest
- [ ] Pod name is `pod-httpd` (exact)
- [ ] Container name is `httpd-container` (exact)
- [ ] Image is `httpd:latest` (explicit tag)
- [ ] Label `app=httpd_app` is set (exact)
- [ ] Applied YAML to create pod
- [ ] Pod status is Running
- [ ] Container is Ready (1/1)
- [ ] Label visible with `--show-labels`
- [ ] Pod selectable with `-l app=httpd_app`
- [ ] Apache logs show successful start
- [ ] No errors in pod events
- [ ] All requirements met ✅

---

## Completion Details

- **Completion Date:** December 24, 2025
- **Day:** 48 / 100
- **Challenge:** KodeKloud 100 Days Cloud DevOps
- **Topic:** Creating Kubernetes Pods
- **Platform:** Kubernetes
- **Host:** jump_host (kubectl pre-configured)
- **Pod Name:** pod-httpd
- **Image:** httpd:latest (Apache HTTP Server)
- **Container Name:** httpd-container
- **Label:** app=httpd_app
- **Method:** Declarative (YAML manifest)
- **Key Skill:** Kubernetes pod creation, kubectl operations, YAML manifests
- **Status:** ✅ Successfully Completed

---

## Summary

This task introduced **Kubernetes fundamentals** through pod creation:

✅ **Understood Kubernetes Architecture** - Control plane and worker nodes
✅ **Created Pod** - Smallest deployable unit in Kubernetes
✅ **Used Labels** - Organized and selected pods
✅ **Applied YAML Manifest** - Declarative configuration
✅ **Verified Deployment** - Checked pod status and logs

**Key Insight:** Kubernetes provides **orchestration, automation, and self-healing** for containerized applications!

**Pod Creation Journey:**
```
YAML Manifest
    ↓
kubectl apply
    ↓
API Server receives request
    ↓
Scheduler assigns to node
    ↓
Kubelet pulls image
    ↓
Container Runtime creates container
    ↓
Pod Running
    ↓
Ready to serve traffic
```

**Kubernetes vs Docker:**

| Aspect | Docker | Kubernetes |
|--------|--------|-----------|
| **Unit** | Container | Pod |
| **Orchestration** | Manual | Automatic |
| **Scaling** | Manual | Declarative |
| **Discovery** | Manual linking | Service DNS |
| **Updates** | Stop/Start | Rolling update |
| **Healing** | Manual restart | Automatic |

**Pod Components:**
```yaml
Pod (pod-httpd)
├── Metadata
│   ├── Name: pod-httpd
│   └── Label: app=httpd_app
├── Spec
│   └── Container: httpd-container
│       ├── Image: httpd:latest
│       └── Port: 80
└── Status
    ├── Phase: Running
    ├── IP: 10.244.0.5
    └── Ready: True
```

**kubectl Workflow:**
```
Create:  kubectl apply -f pod.yaml
List:    kubectl get pods
Details: kubectl describe pod pod-httpd
Logs:    kubectl logs pod-httpd
Test:    kubectl exec pod-httpd -- httpd -v
Delete:  kubectl delete pod pod-httpd
```

**Why Labels Matter:**
```
Without Labels:
kubectl get pods → Returns all pods (hard to filter)

With Labels:
kubectl get pods -l app=httpd_app → Returns only httpd pods
kubectl get pods -l env=production → Returns only production pods
kubectl get pods -l app=httpd_app,env=production → Multiple filters
```

**YAML Structure:**
```yaml
apiVersion: v1              # Which API (v1 = core)
kind: Pod                   # What to create
metadata:                   # Identification
  name: pod-httpd          # Unique name
  labels:                  # Tags
    app: httpd_app         # app=httpd_app
spec:                      # Desired state
  containers:              # List of containers
  - name: httpd-container  # Container name
    image: httpd:latest    # Docker image
```

**Imperative vs Declarative:**

| Approach | Command | Pros | Cons |
|----------|---------|------|------|
| **Imperative** | `kubectl run` | Fast, simple | Not reproducible |
| **Declarative** | `kubectl apply -f` | Reproducible, version controlled | More setup |

**Production Best Practices:**
- ✅ Use Deployments, not standalone pods
- ✅ Set resource limits
- ✅ Add health checks
- ✅ Use specific image tags (not :latest)
- ✅ Label everything
- ✅ Version control YAML files

**Real-World Use Cases:**
- **Microservices:** Each service in separate pods
- **Batch Jobs:** One-time tasks in pods
- **Debugging:** Test containers in isolated pods
- **Development:** Quick testing with kubectl run
- **CI/CD:** Deploy apps via YAML manifests

**Remember:** Pods are **ephemeral** - they can be deleted and recreated anytime. For production, use **Deployments** which manage pods and provide rolling updates, scaling, and self-healing! 🚀

**Next Steps:**
- Learn Deployments for production workloads
- Explore Services for pod networking
- Study ConfigMaps and Secrets for configuration
- Practice with ReplicaSets for scaling
- Deploy multi-tier applications

**Next:** Kubernetes Deployments, Services, ConfigMaps, Persistent Volumes, and Helm charts! ☸️
