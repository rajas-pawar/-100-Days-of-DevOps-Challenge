# Day 49: Creating Kubernetes Deployments
**100 Days Cloud DevOps Challenge – KodeKloud**

---

## Objective

Create a Kubernetes Deployment to manage application replicas with automated rollout, scaling, and self-healing capabilities. Learn the difference between Pods and Deployments, and why Deployments are the production standard.

**Requirements:**
1. Work on **jump_host** (kubectl already configured)
2. **Deployment Name:** `httpd`
3. **Application:** httpd (Apache HTTP Server)
4. **Image:** `httpd:latest` (explicitly specify tag)
5. **Method:** Use kubectl or YAML manifest
6. **Note:** kubectl is pre-configured for the cluster

---

## Understanding Kubernetes Deployments

**Deployment** is a higher-level Kubernetes object that provides declarative updates for Pods and ReplicaSets. It's the standard way to run stateless applications in production.

### What is a Deployment?

```
Pod (Day 48):
┌─────────────────┐
│   Single Pod    │
│   Manual mgmt   │
│   No scaling    │
│   No updates    │
└─────────────────┘

Deployment (Day 49):
┌─────────────────────────────────┐
│        Deployment               │
│   ┌─────────────────────┐      │
│   │    ReplicaSet       │      │
│   │  ┌─────┬─────┬─────┐│      │
│   │  │Pod 1│Pod 2│Pod 3││      │
│   │  └─────┴─────┴─────┘│      │
│   └─────────────────────┘      │
│                                 │
│  ✅ Automated scaling           │
│  ✅ Rolling updates             │
│  ✅ Self-healing                │
│  ✅ Rollback capability         │
└─────────────────────────────────┘
```

### Deployment vs Pod:

| Feature | Pod (Day 48) | Deployment (Day 49) |
|---------|-------------|---------------------|
| **Management** | Manual | Automated |
| **Replicas** | Single instance | Multiple replicas |
| **Scaling** | Manual recreation | Declarative scaling |
| **Updates** | Delete and recreate | Rolling updates |
| **Rollback** | Not possible | Built-in rollback |
| **Self-Healing** | Manual restart | Automatic restart |
| **Use Case** | Testing, debugging | **Production apps** |
| **Recommended** | ❌ Not for prod | ✅ Production standard |

### Why Use Deployments?

```
Without Deployment (Pod only):
├── Pod crashes → Manual restart needed
├── Update app → Delete pod, create new one (downtime!)
├── Scale to 3 pods → Create 3 separate pods manually
├── One pod fails → Application partially down
└── Rollback → No history, manual recreation

With Deployment:
├── ✅ Pod crashes → Auto-recreated in seconds
├── ✅ Update app → Rolling update (zero downtime!)
├── ✅ Scale to 3 pods → `kubectl scale --replicas=3`
├── ✅ One pod fails → Others continue serving traffic
└── ✅ Rollback → `kubectl rollout undo` (one command!)
```

---

## Understanding Deployment Architecture

### Three-Layer Architecture:

```
Layer 1: Deployment
  │
  ├── Manages lifecycle
  ├── Defines desired state
  ├── Handles updates
  │
  ↓
Layer 2: ReplicaSet
  │
  ├── Maintains replica count
  ├── Creates/deletes pods
  ├── Ensures availability
  │
  ↓
Layer 3: Pods
  │
  ├── Run containers
  ├── Execute application
  └── Serve traffic
```

### How Deployments Work:

```
1. You create Deployment YAML
   ↓
2. kubectl apply -f deployment.yaml
   ↓
3. Deployment Controller receives spec
   ↓
4. Creates ReplicaSet with pod template
   ↓
5. ReplicaSet Controller creates pods
   ↓
6. Scheduler assigns pods to nodes
   ↓
7. Kubelet starts containers
   ↓
8. Pods start running
   ↓
9. Deployment monitors health
   ↓
10. Self-healing if pods fail
```

### Deployment Lifecycle:

```
Create Deployment
  ↓
Progressing (Creating ReplicaSet & Pods)
  ↓
Available (All replicas running)
  ↓
[Running - Self-healing active]
  ↓
Update (Trigger rolling update)
  ↓
Progressing (Creating new ReplicaSet)
  ↓
Available (Update complete)
  ↓
[Optional: Rollback if issues]
```

---

## Understanding ReplicaSets

**ReplicaSet** ensures that a specified number of pod replicas are running at any given time. Deployments create and manage ReplicaSets automatically.

### ReplicaSet Role:

```
ReplicaSet Responsibilities:
├── Maintain desired replica count
├── Create pods from template
├── Monitor pod health
├── Replace failed pods
└── Scale up/down as needed

Example:
Desired: 3 replicas
Current: 2 replicas (1 crashed)
Action: Create 1 new pod automatically
```

### Deployment vs ReplicaSet:

```
ReplicaSet (Lower Level):
├── Maintains pod count
├── No update strategy
├── Manual version management
└── Direct use: Rare

Deployment (Higher Level):
├── Creates/manages ReplicaSets
├── Rolling update strategy
├── Automated version management
└── Direct use: Standard practice ✅
```

**Best Practice:** Always use Deployments, not ReplicaSets directly!

---

## Understanding Declarative vs Imperative

### Imperative Approach:

```bash
# Tell Kubernetes HOW to do it
kubectl create deployment httpd --image=httpd:latest
kubectl scale deployment httpd --replicas=3
kubectl set image deployment/httpd httpd=httpd:2.4.58
```

**Pros:**
- ✅ Quick for simple tasks
- ✅ No YAML knowledge needed
- ✅ Good for learning

**Cons:**
- ❌ Not version controlled
- ❌ Hard to reproduce
- ❌ No audit trail

### Declarative Approach:

```yaml
# Tell Kubernetes WHAT you want
apiVersion: apps/v1
kind: Deployment
metadata:
  name: httpd
spec:
  replicas: 3
  selector:
    matchLabels:
      app: httpd
  template:
    metadata:
      labels:
        app: httpd
    spec:
      containers:
      - name: httpd
        image: httpd:2.4.58
```

```bash
kubectl apply -f deployment.yaml
```

**Pros:**
- ✅ Version controlled (Git)
- ✅ Reproducible
- ✅ Audit trail
- ✅ Production standard

**Cons:**
- ❌ Requires YAML knowledge
- ❌ More initial setup

**Recommendation:** Use declarative (YAML) for production!

---

## Understanding Deployment YAML

### Basic Deployment Structure:

```yaml
apiVersion: apps/v1        # API version for Deployments
kind: Deployment           # Object type
metadata:
  name: httpd             # Deployment name
spec:
  replicas: 1             # Number of pod replicas
  selector:               # How to find pods to manage
    matchLabels:
      app: httpd
  template:               # Pod template
    metadata:
      labels:             # Pod labels (must match selector)
        app: httpd
    spec:
      containers:         # Container spec
      - name: httpd
        image: httpd:latest
```

### Field Explanations:

| Field | Purpose | Required | Example |
|-------|---------|----------|---------|
| **apiVersion** | API version | ✅ Yes | `apps/v1` |
| **kind** | Object type | ✅ Yes | `Deployment` |
| **metadata.name** | Deployment name | ✅ Yes | `httpd` |
| **spec.replicas** | Desired pod count | ❌ No (default: 1) | `3` |
| **spec.selector** | Pod selector | ✅ Yes | `matchLabels: app: httpd` |
| **spec.template** | Pod template | ✅ Yes | Pod specification |
| **template.metadata.labels** | Pod labels | ✅ Yes | Must match selector |
| **template.spec.containers** | Container list | ✅ Yes | Image, name, ports |

### Critical: Selector and Labels:

```yaml
spec:
  selector:
    matchLabels:
      app: httpd        # ← Deployment finds pods with this label
  template:
    metadata:
      labels:
        app: httpd      # ← Pods must have matching label
```

**⚠️ Important:** Labels in `template.metadata.labels` **MUST match** `selector.matchLabels`!

```yaml
# ✅ Correct - Labels match
selector:
  matchLabels:
    app: httpd
template:
  metadata:
    labels:
      app: httpd

# ❌ Wrong - Labels don't match
selector:
  matchLabels:
    app: httpd
template:
  metadata:
    labels:
      app: apache      # Mismatch! Deployment won't work
```

---

## Understanding Deployment Strategies

### Rolling Update (Default):

```
Old Version (v1):  [Pod1] [Pod2] [Pod3]
                       ↓
Start Update:      [Pod1] [Pod2] [Pod3] [Pod4-v2]
                       ↓
Continue:          [Pod1] [Pod2] [Pod4-v2] [Pod5-v2]
                       ↓
Complete:          [Pod4-v2] [Pod5-v2] [Pod6-v2]

Benefits:
✅ Zero downtime
✅ Gradual rollout
✅ Can pause/resume
✅ Automatic rollback if failures
```

### Recreate Strategy:

```
Old Version:  [Pod1] [Pod2] [Pod3]
                  ↓
Delete All:   [ ]    [ ]    [ ]     ← Downtime!
                  ↓
Create New:   [Pod1-v2] [Pod2-v2] [Pod3-v2]

Use Case:
- Apps that can't run multiple versions
- Database migrations
- Acceptable downtime
```

### Strategy Configuration:

```yaml
spec:
  strategy:
    type: RollingUpdate    # or Recreate
    rollingUpdate:
      maxSurge: 1          # Max extra pods during update
      maxUnavailable: 0    # Max unavailable pods during update
```

---

## Infrastructure Overview

### KodeKloud Environment:

| Component | Details |
|-----------|---------|
| **jump_host** | Client machine with kubectl configured |
| **Kubernetes Cluster** | Pre-configured K8s cluster |
| **kubectl** | Already installed and configured |
| **Context** | Default namespace |
| **Access** | Full cluster access |

### Task Details:

| Item | Value |
|------|-------|
| **Host** | jump_host (kubectl pre-configured) |
| **Deployment Name** | `httpd` (exact name) |
| **Application** | Apache HTTP Server |
| **Image** | `httpd:latest` (explicit tag required) |
| **Replicas** | 1 (default, not specified) |
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

#### Step 3: Check Existing Deployments (Optional)
```bash
# List current deployments
kubectl get deployments
```

**Expected output (if no deployments exist):**
```
No resources found in default namespace.
```

**Or:**
```
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
```

#### Step 4: Create Deployment Using Imperative Command
```bash
# Create deployment with httpd:latest
kubectl create deployment httpd --image=httpd:latest
```

**Expected output:**
```
deployment.apps/httpd created
```

**✅ Deployment created successfully!**

**Command breakdown:**
- `kubectl create deployment` - Create a deployment
- `httpd` - Name of the deployment
- `--image=httpd:latest` - Container image to use

---

### Method 2: Declarative YAML (Recommended)

#### Step 1: Create Deployment YAML Manifest
```bash
# Create httpd-deployment.yaml file
cat > httpd-deployment.yaml << 'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: httpd
  labels:
    app: httpd
spec:
  replicas: 1
  selector:
    matchLabels:
      app: httpd
  template:
    metadata:
      labels:
        app: httpd
    spec:
      containers:
      - name: httpd
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
cat httpd-deployment.yaml
```

**Expected output:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: httpd
  labels:
    app: httpd
spec:
  replicas: 1
  selector:
    matchLabels:
      app: httpd
  template:
    metadata:
      labels:
        app: httpd
    spec:
      containers:
      - name: httpd
        image: httpd:latest
        ports:
        - containerPort: 80
```

**Verify:**
- ✅ apiVersion: apps/v1 (for Deployment)
- ✅ kind: Deployment
- ✅ metadata.name: httpd
- ✅ spec.replicas: 1
- ✅ selector.matchLabels.app: httpd
- ✅ template.metadata.labels.app: httpd (matches selector!)
- ✅ containers.image: httpd:latest
- ✅ Proper YAML indentation (2 spaces)

#### Step 3: Validate YAML (Optional)
```bash
# Dry-run to validate without creating
kubectl apply -f httpd-deployment.yaml --dry-run=client
```

**Expected output:**
```
deployment.apps/httpd created (dry run)
```

**✅ YAML is valid**

#### Step 4: Create Deployment from YAML
```bash
# Apply the manifest
kubectl apply -f httpd-deployment.yaml
```

**Expected output:**
```
deployment.apps/httpd created
```

**Alternative command:**
```bash
kubectl create -f httpd-deployment.yaml
```

**Expected output:**
```
deployment.apps/httpd created
```

**✅ Deployment created successfully!**

---

## Verification Steps

### Step 5: List All Deployments
```bash
kubectl get deployments
```

**Expected output:**
```
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
httpd   1/1     1            1           30s
```

**Column meanings:**
- **NAME:** Deployment name
- **READY:** Ready replicas / Desired replicas
- **UP-TO-DATE:** Replicas at latest version
- **AVAILABLE:** Replicas available to serve traffic
- **AGE:** Time since creation

**✅ Deployment is running with 1/1 pods ready**

### Step 6: List Deployments with More Details
```bash
kubectl get deployments -o wide
```

**Expected output:**
```
NAME    READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES         SELECTOR
httpd   1/1     1            1           1m    httpd        httpd:latest   app=httpd
```

**Additional columns:**
- **CONTAINERS:** Container names
- **IMAGES:** Container images
- **SELECTOR:** Label selector

**✅ Image is httpd:latest, selector is app=httpd**

### Step 7: Check ReplicaSets
```bash
kubectl get replicasets
```

**Expected output:**
```
NAME               DESIRED   CURRENT   READY   AGE
httpd-6f7b8c9d5f   1         1         1       2m
```

**✅ Deployment automatically created a ReplicaSet**

**ReplicaSet name format:** `{deployment-name}-{pod-template-hash}`

### Step 8: List Pods Created by Deployment
```bash
kubectl get pods
```

**Expected output:**
```
NAME                     READY   STATUS    RESTARTS   AGE
httpd-6f7b8c9d5f-abcde   1/1     Running   0          2m
```

**✅ Deployment created a pod automatically**

**Pod name format:** `{deployment-name}-{replicaset-hash}-{random-id}`

### Step 9: List Pods with Labels
```bash
kubectl get pods --show-labels
```

**Expected output:**
```
NAME                     READY   STATUS    RESTARTS   AGE   LABELS
httpd-6f7b8c9d5f-abcde   1/1     Running   0          3m    app=httpd,pod-template-hash=6f7b8c9d5f
```

**✅ Pod has label app=httpd (matches selector)**

### Step 10: Filter Pods by Deployment Label
```bash
# Get pods managed by httpd deployment
kubectl get pods -l app=httpd
```

**Expected output:**
```
NAME                     READY   STATUS    RESTARTS   AGE
httpd-6f7b8c9d5f-abcde   1/1     Running   0          3m
```

**✅ Pod found using deployment's label selector**

### Step 11: Describe Deployment
```bash
kubectl describe deployment httpd
```

**Expected output:**
```
Name:                   httpd
Namespace:              default
CreationTimestamp:      Tue, 24 Dec 2025 11:00:00 +0000
Labels:                 app=httpd
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=httpd
Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=httpd
  Containers:
   httpd:
    Image:        httpd:latest
    Port:         80/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   httpd-6f7b8c9d5f (1/1 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  3m    deployment-controller  Scaled up replica set httpd-6f7b8c9d5f to 1
```

**Key details to verify:**
- ✅ Name: httpd
- ✅ Selector: app=httpd
- ✅ Replicas: 1 desired | 1 available
- ✅ Image: httpd:latest
- ✅ Strategy: RollingUpdate
- ✅ Conditions: Available=True
- ✅ NewReplicaSet created
- ✅ Events show successful scaling

### Step 12: Check Deployment Status
```bash
kubectl rollout status deployment/httpd
```

**Expected output:**
```
deployment "httpd" successfully rolled out
```

**✅ Deployment rollout completed successfully**

### Step 13: Get Deployment YAML
```bash
# View complete deployment configuration
kubectl get deployment httpd -o yaml
```

**Expected output (excerpt):**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "1"
  creationTimestamp: "2025-12-24T11:00:00Z"
  generation: 1
  labels:
    app: httpd
  name: httpd
  namespace: default
  resourceVersion: "12345"
  uid: abc123-def4-5678-90ab-cdef12345678
spec:
  progressDeadlineSeconds: 600
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: httpd
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: httpd
    spec:
      containers:
      - image: httpd:latest
        imagePullPolicy: Always
        name: httpd
        ports:
        - containerPort: 80
          protocol: TCP
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
status:
  availableReplicas: 1
  conditions:
  - lastTransitionTime: "2025-12-24T11:00:15Z"
    lastUpdateTime: "2025-12-24T11:00:15Z"
    message: Deployment has minimum availability.
    reason: MinimumReplicasAvailable
    status: "True"
    type: Available
  - lastTransitionTime: "2025-12-24T11:00:00Z"
    lastUpdateTime: "2025-12-24T11:00:15Z"
    message: ReplicaSet "httpd-6f7b8c9d5f" has successfully progressed.
    reason: NewReplicaSetAvailable
    status: "True"
    type: Progressing
  observedGeneration: 1
  readyReplicas: 1
  replicas: 1
  updatedReplicas: 1
```

**Key observations:**
- ✅ Complete deployment configuration with defaults
- ✅ Strategy: RollingUpdate with 25% surge/unavailable
- ✅ revisionHistoryLimit: 10 (keeps 10 old ReplicaSets for rollback)
- ✅ Status shows Available and Progressing conditions
- ✅ 1/1 replicas available

### Step 14: Check Pod Logs
```bash
# Get pod name first
POD_NAME=$(kubectl get pods -l app=httpd -o jsonpath='{.items[0].metadata.name}')

# Check logs
kubectl logs $POD_NAME
```

**Or directly:**
```bash
kubectl logs -l app=httpd
```

**Expected output:**
```
AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 10.244.0.5. Set the 'ServerName' directive globally to suppress this message
AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 10.244.0.5. Set the 'ServerName' directive globally to suppress this message
[Tue Dec 24 11:00:15.123456 2025] [mpm_event:notice] [pid 1:tid 140123456789] AH00489: Apache/2.4.58 (Unix) configured -- resuming normal operations
[Tue Dec 24 11:00:15.123789 2025] [core:notice] [pid 1:tid 140123456789] AH00094: Command line: 'httpd -D FOREGROUND'
```

**✅ Apache HTTP Server started successfully in pod**

### Step 15: Verify Deployment Hierarchy
```bash
# Show all related objects
echo "=== Deployment ==="
kubectl get deployment httpd

echo -e "\n=== ReplicaSet ==="
kubectl get replicaset -l app=httpd

echo -e "\n=== Pods ==="
kubectl get pods -l app=httpd
```

**Expected output:**
```
=== Deployment ===
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
httpd   1/1     1            1           5m

=== ReplicaSet ===
NAME               DESIRED   CURRENT   READY   AGE
httpd-6f7b8c9d5f   1         1         1       5m

=== Pods ===
NAME                     READY   STATUS    RESTARTS   AGE
httpd-6f7b8c9d5f-abcde   1/1     Running   0          5m
```

**✅ Complete hierarchy visible: Deployment → ReplicaSet → Pod**

### Step 16: Test Self-Healing (Optional)
```bash
# Delete the pod
kubectl delete pod -l app=httpd

# Immediately check pods
kubectl get pods -l app=httpd
```

**Expected output:**
```
NAME                     READY   STATUS              RESTARTS   AGE
httpd-6f7b8c9d5f-xyz12   0/1     ContainerCreating   0          2s
```

**Wait a few seconds and check again:**
```bash
kubectl get pods -l app=httpd
```

**Expected output:**
```
NAME                     READY   STATUS    RESTARTS   AGE
httpd-6f7b8c9d5f-xyz12   1/1     Running   0          15s
```

**✅ Deployment automatically created a new pod! (Self-healing in action)**

### Step 17: Final Verification Checklist
```bash
# 1. Deployment exists with correct name
kubectl get deployment httpd &> /dev/null && echo "✅ Deployment exists" || echo "❌ Deployment not found"

# 2. Deployment has correct image
kubectl get deployment httpd -o jsonpath='{.spec.template.spec.containers[0].image}' | grep -q "httpd:latest" && echo "✅ Image correct" || echo "❌ Image incorrect"

# 3. Deployment is available
kubectl get deployment httpd -o jsonpath='{.status.conditions[?(@.type=="Available")].status}' | grep -q "True" && echo "✅ Deployment available" || echo "❌ Deployment not available"

# 4. All replicas are ready
DESIRED=$(kubectl get deployment httpd -o jsonpath='{.spec.replicas}')
READY=$(kubectl get deployment httpd -o jsonpath='{.status.readyReplicas}')
[ "$DESIRED" == "$READY" ] && echo "✅ All replicas ready ($READY/$DESIRED)" || echo "❌ Not all replicas ready ($READY/$DESIRED)"

# 5. ReplicaSet exists
kubectl get replicaset -l app=httpd &> /dev/null && echo "✅ ReplicaSet exists" || echo "❌ ReplicaSet not found"

# 6. Pods are running
kubectl get pods -l app=httpd -o jsonpath='{.items[0].status.phase}' | grep -q "Running" && echo "✅ Pods running" || echo "❌ Pods not running"
```

**All checks should pass:**
```
✅ Deployment exists
✅ Image correct
✅ Deployment available
✅ All replicas ready (1/1)
✅ ReplicaSet exists
✅ Pods running
```

---

## Complete Command Summary

### Quick Deployment (Imperative):
```bash
# Create deployment
kubectl create deployment httpd --image=httpd:latest

# Verify
kubectl get deployments
kubectl get pods
kubectl describe deployment httpd
```

### Recommended Deployment (Declarative):
```bash
# Create YAML manifest
cat > httpd-deployment.yaml << 'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: httpd
  labels:
    app: httpd
spec:
  replicas: 1
  selector:
    matchLabels:
      app: httpd
  template:
    metadata:
      labels:
        app: httpd
    spec:
      containers:
      - name: httpd
        image: httpd:latest
        ports:
        - containerPort: 80
EOF

# Create deployment
kubectl apply -f httpd-deployment.yaml

# Verify
kubectl get deployments
kubectl get replicasets
kubectl get pods
kubectl rollout status deployment/httpd
```

### Detailed Workflow:
```bash
# 1. Verify kubectl access
kubectl version --client
kubectl cluster-info

# 2. Check current state
kubectl get deployments
kubectl get all

# 3. Create deployment (choose one method)
# Method A: Imperative
kubectl create deployment httpd --image=httpd:latest

# Method B: Declarative
kubectl apply -f httpd-deployment.yaml

# 4. Watch deployment creation
kubectl get deployments -w

# 5. Check deployment status
kubectl rollout status deployment/httpd
kubectl get deployments
kubectl get replicasets
kubectl get pods

# 6. Describe deployment
kubectl describe deployment httpd

# 7. Check logs
kubectl logs -l app=httpd

# 8. View complete config
kubectl get deployment httpd -o yaml

# 9. Test self-healing (optional)
kubectl delete pod -l app=httpd
kubectl get pods -l app=httpd
```

---

## Deployment Management Commands

### Scaling Deployments:

```bash
# Scale to 3 replicas (imperative)
kubectl scale deployment httpd --replicas=3

# Verify scaling
kubectl get deployments
kubectl get pods
```

**Expected output:**
```
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
httpd   3/3     3            3           5m

NAME                     READY   STATUS    RESTARTS   AGE
httpd-6f7b8c9d5f-abcde   1/1     Running   0          5m
httpd-6f7b8c9d5f-fghij   1/1     Running   0          10s
httpd-6f7b8c9d5f-klmno   1/1     Running   0          10s
```

**Declarative scaling:**
```yaml
# Edit deployment.yaml
spec:
  replicas: 3  # Change from 1 to 3

# Apply changes
kubectl apply -f httpd-deployment.yaml
```

### Updating Deployments:

```bash
# Update image (imperative)
kubectl set image deployment/httpd httpd=httpd:2.4.58

# Check rollout status
kubectl rollout status deployment/httpd

# View rollout history
kubectl rollout history deployment/httpd
```

**Expected output:**
```
deployment.apps/httpd image updated

deployment "httpd" successfully rolled out

REVISION  CHANGE-CAUSE
1         <none>
2         <none>
```

**Declarative update:**
```yaml
# Edit deployment.yaml
spec:
  template:
    spec:
      containers:
      - name: httpd
        image: httpd:2.4.58  # Change from httpd:latest

# Apply changes
kubectl apply -f httpd-deployment.yaml
```

### Rolling Back Deployments:

```bash
# Rollback to previous version
kubectl rollout undo deployment/httpd

# Rollback to specific revision
kubectl rollout undo deployment/httpd --to-revision=1

# Check rollout status
kubectl rollout status deployment/httpd
```

### Pausing and Resuming:

```bash
# Pause deployment (no new rollouts)
kubectl rollout pause deployment/httpd

# Make multiple changes
kubectl set image deployment/httpd httpd=httpd:2.4.58
kubectl set resources deployment/httpd -c=httpd --limits=cpu=200m,memory=512Mi

# Resume deployment (apply all changes together)
kubectl rollout resume deployment/httpd
```

---

## kubectl Deployment Commands Reference

### Deployment Management:

```bash
# Create deployment
kubectl create deployment NAME --image=IMAGE           # Imperative
kubectl apply -f deployment.yaml                       # Declarative

# List deployments
kubectl get deployments                                # All deployments
kubectl get deployments -o wide                        # With more details
kubectl get deployments -n namespace                   # Specific namespace
kubectl get deployments -A                             # All namespaces

# Deployment details
kubectl describe deployment NAME                       # Detailed info
kubectl get deployment NAME -o yaml                    # YAML format
kubectl get deployment NAME -o json                    # JSON format

# Scale deployment
kubectl scale deployment NAME --replicas=N             # Set replica count
kubectl autoscale deployment NAME --min=2 --max=10    # Auto-scaling

# Update deployment
kubectl set image deployment/NAME CONTAINER=IMAGE      # Update image
kubectl edit deployment NAME                           # Edit live config
kubectl apply -f deployment.yaml                       # Apply changes

# Rollout management
kubectl rollout status deployment/NAME                 # Check status
kubectl rollout history deployment/NAME                # View history
kubectl rollout undo deployment/NAME                   # Rollback
kubectl rollout undo deployment/NAME --to-revision=N   # Rollback to revision
kubectl rollout pause deployment/NAME                  # Pause rollout
kubectl rollout resume deployment/NAME                 # Resume rollout
kubectl rollout restart deployment/NAME                # Restart pods

# Delete deployment
kubectl delete deployment NAME                         # Delete by name
kubectl delete -f deployment.yaml                      # Delete by file
kubectl delete deployments --all                       # Delete all
```

### Related Objects:

```bash
# ReplicaSets
kubectl get replicasets                                # All ReplicaSets
kubectl get replicasets -l app=httpd                   # By label
kubectl describe replicaset NAME                       # Detailed info

# Pods (managed by deployment)
kubectl get pods                                       # All pods
kubectl get pods -l app=httpd                          # Deployment's pods
kubectl logs -l app=httpd                              # Logs from all pods
kubectl delete pod NAME                                # Delete pod (will be recreated)
```

---

## Troubleshooting Guide

### Issue 1: Deployment Stuck in Progressing

**Problem:**
```bash
kubectl get deployments
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
httpd   0/1     1            0           5m
```

**Diagnosis:**
```bash
kubectl describe deployment httpd
kubectl get pods -l app=httpd
kubectl describe pod <pod-name>
```

**Common causes:**

**A. Image pull error:**
```
Events:
  Warning  Failed  2m  kubelet  Failed to pull image "httpd:latest"
```

**Solution:**
```bash
# Check image name
# Verify image exists in registry
# Check image pull secrets for private registries

# Fix image in deployment
kubectl set image deployment/httpd httpd=httpd:latest
```

**B. Insufficient resources:**
```
Events:
  Warning  FailedScheduling  2m  default-scheduler  0/3 nodes available: insufficient cpu
```

**Solution:**
```bash
# Check node resources
kubectl top nodes
kubectl describe nodes

# Reduce resource requests or add nodes
```

**C. Pod crashes on startup:**
```bash
kubectl get pods -l app=httpd
NAME                     READY   STATUS             RESTARTS   AGE
httpd-6f7b8c9d5f-abcde   0/1     CrashLoopBackOff   5          5m
```

**Solution:**
```bash
# Check pod logs
kubectl logs <pod-name>
kubectl logs <pod-name> --previous

# Fix application or image
```

### Issue 2: Selector/Label Mismatch

**Problem:**
```
The Deployment "httpd" is invalid: spec.template.metadata.labels: Invalid value: map[string]string{"app":"apache"}: `selector` does not match template `labels`
```

**Cause:**
```yaml
# Mismatched labels
spec:
  selector:
    matchLabels:
      app: httpd        # ← Looking for this
  template:
    metadata:
      labels:
        app: apache     # ← But pods have this (mismatch!)
```

**Solution:**
```yaml
# Fix: Make labels match
spec:
  selector:
    matchLabels:
      app: httpd
  template:
    metadata:
      labels:
        app: httpd      # ← Now they match!
```

### Issue 3: Deployment Update Not Rolling Out

**Problem:**
```bash
# Changed image but pods not updating
kubectl set image deployment/httpd httpd=httpd:2.4.58
# Pods still running old version
```

**Diagnosis:**
```bash
# Check if deployment is paused
kubectl get deployment httpd -o jsonpath='{.spec.paused}'
```

**Solution:**
```bash
# Resume if paused
kubectl rollout resume deployment/httpd

# Check status
kubectl rollout status deployment/httpd
```

### Issue 4: Old ReplicaSets Not Cleaned Up

**Problem:**
```bash
kubectl get replicasets
NAME               DESIRED   CURRENT   READY   AGE
httpd-6f7b8c9d5f   1         1         1       10m
httpd-abc123def    0         0         0       8m
httpd-xyz789ghi    0         0         0       6m
httpd-mno456pqr    0         0         0       4m
# Too many old ReplicaSets
```

**Explanation:**
Kubernetes keeps old ReplicaSets for rollback (default: 10)

**Solution:**
```yaml
# Set revision history limit in deployment
spec:
  revisionHistoryLimit: 3  # Keep only 3 old ReplicaSets
```

**Or manually delete:**
```bash
# Delete specific ReplicaSet
kubectl delete replicaset httpd-abc123def

# Clean up automatically
kubectl apply -f deployment.yaml  # With revisionHistoryLimit: 3
```

### Issue 5: Can't Delete Deployment

**Problem:**
```bash
kubectl delete deployment httpd
# Deployment stuck in Terminating state
```

**Diagnosis:**
```bash
kubectl describe deployment httpd
kubectl get pods -l app=httpd
```

**Solution:**
```bash
# Force delete if stuck
kubectl delete deployment httpd --grace-period=0 --force

# Or delete finalizers
kubectl patch deployment httpd -p '{"metadata":{"finalizers":[]}}' --type=merge
```

### Issue 6: Deployment Replicas Not Matching

**Problem:**
```bash
kubectl get deployment httpd
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
httpd   2/3     3            2           5m
```

**Diagnosis:**
```bash
kubectl describe deployment httpd
kubectl get pods -l app=httpd
```

**Common causes:**

**A. Pods not ready:**
```bash
kubectl get pods -l app=httpd
NAME                     READY   STATUS    RESTARTS   AGE
httpd-6f7b8c9d5f-abc     1/1     Running   0          5m
httpd-6f7b8c9d5f-def     1/1     Running   0          5m
httpd-6f7b8c9d5f-ghi     0/1     Running   0          30s  # Not ready yet
```

**Solution:** Wait for pod to become ready, or check pod logs/events

**B. Insufficient resources:**
```bash
kubectl describe pods -l app=httpd
# Check Events for scheduling issues
```

**Solution:** Scale down or add resources

### Issue 7: Wrong Image Version Deployed

**Problem:**
Deployment created with wrong image version

**Solution:**
```bash
# Update image (imperative)
kubectl set image deployment/httpd httpd=httpd:latest

# Or update YAML and apply
kubectl apply -f httpd-deployment.yaml

# Verify update
kubectl get deployment httpd -o jsonpath='{.spec.template.spec.containers[0].image}'
```

---

## Advanced Deployment Concepts

### Deployment with Resource Limits:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: httpd
spec:
  replicas: 3
  selector:
    matchLabels:
      app: httpd
  template:
    metadata:
      labels:
        app: httpd
    spec:
      containers:
      - name: httpd
        image: httpd:latest
        ports:
        - containerPort: 80
        
        # Resource limits
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
          limits:
            memory: "128Mi"
            cpu: "500m"
```

### Deployment with Health Checks:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: httpd
spec:
  replicas: 3
  selector:
    matchLabels:
      app: httpd
  template:
    metadata:
      labels:
        app: httpd
    spec:
      containers:
      - name: httpd
        image: httpd:latest
        ports:
        - containerPort: 80
        
        # Liveness probe
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 10
          periodSeconds: 10
        
        # Readiness probe
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
```

### Deployment with Environment Variables:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: httpd
spec:
  replicas: 1
  selector:
    matchLabels:
      app: httpd
  template:
    metadata:
      labels:
        app: httpd
    spec:
      containers:
      - name: httpd
        image: httpd:latest
        ports:
        - containerPort: 80
        
        # Environment variables
        env:
        - name: APACHE_LOG_LEVEL
          value: "info"
        - name: APACHE_RUN_USER
          value: "www-data"
        - name: APACHE_RUN_GROUP
          value: "www-data"
```

### Deployment with ConfigMap:

```yaml
# ConfigMap
apiVersion: v1
kind: ConfigMap
metadata:
  name: httpd-config
data:
  log-level: "info"
  server-name: "example.com"

---
# Deployment using ConfigMap
apiVersion: apps/v1
kind: Deployment
metadata:
  name: httpd
spec:
  replicas: 1
  selector:
    matchLabels:
      app: httpd
  template:
    metadata:
      labels:
        app: httpd
    spec:
      containers:
      - name: httpd
        image: httpd:latest
        ports:
        - containerPort: 80
        
        # Use ConfigMap values
        envFrom:
        - configMapRef:
            name: httpd-config
```

### Deployment with Volumes:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: httpd
spec:
  replicas: 1
  selector:
    matchLabels:
      app: httpd
  template:
    metadata:
      labels:
        app: httpd
    spec:
      containers:
      - name: httpd
        image: httpd:latest
        ports:
        - containerPort: 80
        
        # Volume mounts
        volumeMounts:
        - name: html
          mountPath: /usr/local/apache2/htdocs
        - name: config
          mountPath: /usr/local/apache2/conf/httpd.conf
          subPath: httpd.conf
      
      # Volumes
      volumes:
      - name: html
        emptyDir: {}
      - name: config
        configMap:
          name: httpd-config
```

---

## Best Practices Summary

### Deployment Best Practices:

1. **Always Use Deployments for Stateless Apps:**
   ```
   ✅ Deployment (for stateless apps like web servers)
   ✅ StatefulSet (for stateful apps like databases)
   ❌ Standalone Pods (only for testing)
   ```

2. **Specify Image Tags:**
   ```yaml
   image: httpd:2.4.58  # ✅ Specific version (production)
   image: httpd:latest  # ⚠️ Latest (dev/test only)
   ```

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
   kubectl apply -f deployment.yaml  # ✅ Reproducible
   ```

6. **Label Everything:**
   ```yaml
   metadata:
     labels:
       app: httpd
       env: production
       version: v1.0
   ```

7. **Set Rollout Strategy:**
   ```yaml
   strategy:
     type: RollingUpdate
     rollingUpdate:
       maxSurge: 1
       maxUnavailable: 0  # Zero downtime
   ```

8. **Limit Revision History:**
   ```yaml
   spec:
     revisionHistoryLimit: 10  # Keep 10 old versions
   ```

9. **Use Update Strategies:**
   ```bash
   # For zero-downtime updates
   strategy:
     rollingUpdate:
       maxUnavailable: 0
       maxSurge: 1
   ```

10. **Test Rollbacks:**
    ```bash
    # Always test rollback procedure
    kubectl rollout undo deployment/httpd
    kubectl rollout status deployment/httpd
    ```

---

## Real-World Applications

### Production Deployment Example:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: httpd
  labels:
    app: httpd
    env: production
    version: v1.0
  annotations:
    description: "Production Apache web server"
    owner: "devops-team@example.com"
spec:
  replicas: 3  # High availability
  
  # Deployment strategy
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0  # Zero downtime
  
  revisionHistoryLimit: 10
  
  selector:
    matchLabels:
      app: httpd
      env: production
  
  template:
    metadata:
      labels:
        app: httpd
        env: production
        version: v1.0
    spec:
      containers:
      - name: httpd
        image: httpd:2.4.58  # Specific version
        
        ports:
        - containerPort: 80
          name: http
        
        # Resource limits
        resources:
          requests:
            memory: "128Mi"
            cpu: "250m"
          limits:
            memory: "256Mi"
            cpu: "500m"
        
        # Health checks
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3
        
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
          timeoutSeconds: 3
          failureThreshold: 3
        
        # Environment variables
        env:
        - name: APACHE_LOG_LEVEL
          value: "info"
        
        # Volume mounts
        volumeMounts:
        - name: html
          mountPath: /usr/local/apache2/htdocs
      
      volumes:
      - name: html
        persistentVolumeClaim:
          claimName: httpd-pvc
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
      
      - name: Deploy application
        run: |
          kubectl apply -f k8s/deployment.yaml
          kubectl rollout status deployment/httpd
      
      - name: Verify deployment
        run: |
          kubectl get deployment httpd
          kubectl get pods -l app=httpd
      
      - name: Run tests
        run: |
          kubectl port-forward deployment/httpd 8080:80 &
          sleep 5
          curl http://localhost:8080
```

### Blue-Green Deployment:

```yaml
# Blue deployment (current)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: httpd-blue
spec:
  replicas: 3
  selector:
    matchLabels:
      app: httpd
      version: blue
  template:
    metadata:
      labels:
        app: httpd
        version: blue
    spec:
      containers:
      - name: httpd
        image: httpd:2.4.57

---
# Green deployment (new)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: httpd-green
spec:
  replicas: 3
  selector:
    matchLabels:
      app: httpd
      version: green
  template:
    metadata:
      labels:
        app: httpd
        version: green
    spec:
      containers:
      - name: httpd
        image: httpd:2.4.58

---
# Service (switch between blue and green)
apiVersion: v1
kind: Service
metadata:
  name: httpd-service
spec:
  selector:
    app: httpd
    version: blue  # Change to 'green' when ready
  ports:
  - port: 80
    targetPort: 80
```

---

## Key Commands Reference

| Command | Description |
|---------|-------------|
| `kubectl create deployment NAME --image=IMAGE` | Create deployment |
| `kubectl apply -f deployment.yaml` | Create/update from YAML |
| `kubectl get deployments` | List deployments |
| `kubectl describe deployment NAME` | Detailed info |
| `kubectl scale deployment NAME --replicas=N` | Scale deployment |
| `kubectl set image deployment/NAME CONTAINER=IMAGE` | Update image |
| `kubectl rollout status deployment/NAME` | Check rollout status |
| `kubectl rollout history deployment/NAME` | View history |
| `kubectl rollout undo deployment/NAME` | Rollback deployment |
| `kubectl delete deployment NAME` | Delete deployment |

---

## Completion Checklist

- [ ] Verified kubectl is configured
- [ ] Connected to Kubernetes cluster
- [ ] Created deployment named `httpd` (exact name)
- [ ] Used image `httpd:latest` (explicit tag)
- [ ] Deployment created successfully
- [ ] Deployment status is Available
- [ ] ReplicaSet created automatically
- [ ] Pod(s) created and running
- [ ] All replicas ready (1/1)
- [ ] Rollout completed successfully
- [ ] Verified deployment hierarchy (Deployment → ReplicaSet → Pod)
- [ ] No errors in events
- [ ] All requirements met ✅

---

## Completion Details

- **Completion Date:** December 24, 2025
- **Day:** 49 / 100
- **Challenge:** KodeKloud 100 Days Cloud DevOps
- **Topic:** Creating Kubernetes Deployments
- **Platform:** Kubernetes
- **Host:** jump_host (kubectl pre-configured)
- **Deployment Name:** httpd
- **Image:** httpd:latest (Apache HTTP Server)
- **Replicas:** 1 (default)
- **Method:** Imperative OR Declarative
- **Key Skill:** Kubernetes Deployments, automated rollouts, self-healing
- **Status:** ✅ Successfully Completed

---

## Summary

This task demonstrated **Kubernetes Deployments** - the production standard for running stateless applications:

✅ **Created Deployment** - Higher-level abstraction over Pods
✅ **Automated Management** - Self-healing, scaling, and updates
✅ **Understood Architecture** - Deployment → ReplicaSet → Pods
✅ **Verified Rollout** - Deployment successfully rolled out
✅ **Production Ready** - Learned production best practices

**Key Insight:** Deployments provide **automation, reliability, and zero-downtime updates** that standalone Pods cannot!

**Deployment Journey:**
```
YAML/Command
    ↓
Deployment Created
    ↓
Deployment Controller creates ReplicaSet
    ↓
ReplicaSet Controller creates Pods
    ↓
Scheduler assigns Pods to Nodes
    ↓
Kubelet starts Containers
    ↓
Pods Running & Monitored
    ↓
Continuous Self-Healing
```

**Pod vs Deployment:**

| Scenario | Pod (Day 48) | Deployment (Day 49) |
|----------|-------------|---------------------|
| **Pod crashes** | Manual restart | Auto-recreated ✅ |
| **Update app** | Delete & recreate (downtime) | Rolling update (zero downtime) ✅ |
| **Scale to 3** | Create 3 separate pods | `replicas: 3` ✅ |
| **Rollback** | Not possible | One command ✅ |
| **Production** | ❌ Not recommended | ✅ Standard practice |

**Three-Layer Architecture:**
```
Deployment: httpd
├── Manages: Updates, scaling, rollback
├── Creates: ReplicaSet
│
ReplicaSet: httpd-6f7b8c9d5f
├── Manages: Replica count
├── Creates: Pods
│
Pods: httpd-6f7b8c9d5f-xxxxx
├── Runs: Containers
└── Executes: Application
```

**Deployment Benefits:**
```
1. Self-Healing:
   Pod dies → New pod auto-created

2. Declarative Updates:
   Change image → Rolling update with zero downtime

3. Scaling:
   replicas: 1 → replicas: 3 → Automatic

4. Rollback:
   Bad update → Rollback to previous version instantly

5. History:
   10 revisions kept for easy rollback

6. Health Monitoring:
   Readiness/Liveness probes → Auto-recovery
```

**Rolling Update Process:**
```
Old Pods: [v1] [v1] [v1]
            ↓
Step 1:   [v1] [v1] [v1] [v2]  ← Create new pod
Step 2:   [v1] [v1] [v2] [v2]  ← Another new pod, delete old
Step 3:   [v1] [v2] [v2] [v2]  ← Continue
Step 4:   [v2] [v2] [v2]       ← Complete (zero downtime!)
```

**Critical YAML Concept:**
```yaml
spec:
  selector:
    matchLabels:
      app: httpd      # Deployment finds pods with this label
  template:
    metadata:
      labels:
        app: httpd    # Pods MUST have matching label!
```

**Real-World Use Cases:**
- **Web Servers:** Apache, Nginx (stateless)
- **APIs:** REST APIs, microservices
- **Frontend:** React, Angular apps
- **Background Workers:** Job processors
- **Caching:** Redis, Memcached (stateless mode)

**Production Checklist:**
- ✅ Use Deployments, not Pods
- ✅ Set resource limits
- ✅ Add health checks
- ✅ Specify image versions (not :latest in prod)
- ✅ Configure rolling update strategy
- ✅ Set replicas ≥ 3 for high availability
- ✅ Use labels for organization
- ✅ Test rollback procedures
- ✅ Monitor deployment status

**Remember:** Deployments are the **foundation of production Kubernetes**! They provide the automation and reliability needed for running applications at scale with zero downtime! 🚀

**Next Steps:**
- Expose Deployment with Services (ClusterIP, NodePort, LoadBalancer)
- Add Horizontal Pod Autoscaler (HPA)
- Implement ConfigMaps and Secrets
- Set up Ingress for routing
- Deploy multi-tier applications

**Next:** Kubernetes Services, ConfigMaps, Persistent Volumes, StatefulSets, and complete microservices deployments! ☸️
