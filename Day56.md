# Day 56: Kubernetes Deployments and Services - High Availability Nginx

## Objective
Deploy a highly available and scalable static website using Kubernetes Deployment with multiple replicas and expose it using a NodePort service. This demonstrates production-ready application deployment patterns with built-in scalability and high availability.

## Task Requirements
- **Environment:** Kubernetes cluster accessible from jump_host
- **Deployment Name:** nginx-deployment
- **Image:** nginx:latest (must specify tag)
- **Container Name:** nginx-container
- **Replicas:** 3 (for high availability)
- **Service Type:** NodePort
- **Service Name:** nginx-service
- **NodePort:** 30011
- **Tool:** kubectl (pre-configured on jump_host)

## Understanding Kubernetes Deployments

### What is a Deployment?

A **Deployment** is a Kubernetes resource that provides declarative updates for Pods and ReplicaSets. It manages the lifecycle of your application, ensuring the desired number of replicas are always running.

**Key Characteristics:**
- **Declarative:** You declare the desired state, Kubernetes maintains it
- **Self-Healing:** Automatically replaces failed pods
- **Scalable:** Easy to scale up or down
- **Rolling Updates:** Update with zero downtime
- **Rollback:** Revert to previous versions if needed

### Deployment vs Pod vs ReplicaSet

**Pod (Lowest Level):**
```
Single Pod
├── Container(s)
└── If it dies, it's gone forever ❌
```

**ReplicaSet (Mid Level):**
```
ReplicaSet
├── Pod 1
├── Pod 2
├── Pod 3
└── Maintains desired replica count ✅
    But no rolling updates ❌
```

**Deployment (Highest Level - Best Practice):**
```
Deployment
└── ReplicaSet
    ├── Pod 1
    ├── Pod 2
    └── Pod 3
        ✅ Maintains replica count
        ✅ Rolling updates
        ✅ Rollback capability
        ✅ Self-healing
```

### Why Use Deployments?

**Without Deployment (Manual Pods):**
```
Problems:
❌ Pod dies → Manual recreation needed
❌ Update → Delete all pods, create new ones (downtime!)
❌ No history → Can't rollback
❌ Scaling → Manually create/delete pods
```

**With Deployment:**
```
Benefits:
✅ Pod dies → Automatically recreated
✅ Update → Rolling update (zero downtime)
✅ Revision history → Easy rollback
✅ Scaling → One command: kubectl scale
✅ Health checks → Auto-restart unhealthy pods
```

### Deployment Architecture

```
User Request
    ↓
kubectl apply -f deployment.yaml
    ↓
API Server (stores desired state)
    ↓
Deployment Controller
    ↓
Creates/Manages ReplicaSet
    ↓
ReplicaSet Controller
    ↓
Creates/Manages Pods
    ↓
Scheduler (assigns pods to nodes)
    ↓
Kubelet (runs containers)
    ↓
Running Application (3 replicas)
```

### High Availability with Multiple Replicas

**Single Replica (Not HA):**
```
Node 1
└── Pod 1 (nginx)

Problem: If Pod 1 or Node 1 fails → Service down ❌
```

**Three Replicas (High Availability):**
```
Node 1              Node 2              Node 3
└── Pod 1 (nginx)   └── Pod 2 (nginx)   └── Pod 3 (nginx)

Benefits:
✅ If Pod 1 fails → Pod 2 and Pod 3 still serve traffic
✅ If Node 1 fails → Pods on Node 2 and Node 3 available
✅ Load distributed across 3 pods
✅ Can handle more traffic
✅ Updates possible without downtime
```

## Understanding Kubernetes Services

### What is a Service?

A **Service** is a Kubernetes resource that provides stable networking for Pods. Since Pods are ephemeral (can be destroyed and recreated), Services provide a consistent way to access them.

**Problems Without Services:**
```
Pod 1 (IP: 10.244.1.5) ← Client connects here
    ↓ Pod restarts
Pod 1 (IP: 10.244.1.8) ← Different IP! Client loses connection ❌
```

**With Service:**
```
Service (nginx-service, IP: 10.96.100.50) ← Client connects here
    ↓ Routes to healthy pod
Pod 1, Pod 2, or Pod 3 (IPs change, service IP doesn't) ✅
```

### Service Types

**1. ClusterIP (Default - Internal Only):**
```
Cluster
├── Service: nginx-service (ClusterIP: 10.96.100.50)
│   └── Accessible only within cluster
└── External Client ❌ Cannot access
```

**Use Case:** Internal microservices communication

**2. NodePort (External Access via Node):**
```
Node 1 (192.168.1.10)
├── NodePort: 30011 ← External client connects here
│   ↓
├── Service: nginx-service (ClusterIP: 10.96.100.50)
│   └── Routes to any pod
└── Pods (on any node)
```

**Use Case:** Development, testing, on-premises clusters

**3. LoadBalancer (Cloud Provider):**
```
Cloud Load Balancer (External IP: 203.0.113.5)
    ↓
NodePort on all nodes
    ↓
Service
    ↓
Pods
```

**Use Case:** Production on cloud platforms (AWS, GCP, Azure)

**4. ExternalName (DNS CNAME):**
```
Service: database-service
    ↓ CNAME
External Database: db.example.com
```

**Use Case:** Access external services via DNS

### NodePort Service Details

**Port Concepts:**
```
Client → NodePort (30011) → Service Port (80) → Container Port (80)
         [Node Level]        [Service Level]     [Container Level]
```

**How It Works:**
1. **Client** makes request to any node IP on port 30011
2. **kube-proxy** intercepts traffic on NodePort 30011
3. **Service** routes to one of the healthy pods
4. **Pod** receives traffic on container port 80
5. **Response** returns through same path

**NodePort Range:**
- Default: 30000-32767
- Configured in API server with `--service-node-port-range`
- Our task uses: 30011 ✅

## Step-by-Step Implementation

### Phase 1: Create Deployment

#### Step 1: Create Deployment YAML

```bash
# Create deployment manifest
vi nginx-deployment.yaml
```

**Complete Deployment Configuration:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx-container
        image: nginx:latest
        ports:
        - containerPort: 80
```

**YAML Breakdown:**

**1. API Version and Kind:**
```yaml
apiVersion: apps/v1        # API group for deployments
kind: Deployment           # Resource type
```

**2. Metadata:**
```yaml
metadata:
  name: nginx-deployment   # Deployment name (required)
  labels:
    app: nginx             # Label for deployment (organizational)
```

**3. Deployment Spec:**
```yaml
spec:
  replicas: 3              # Number of pod replicas (high availability)
```

**4. Selector (Critical!):**
```yaml
  selector:
    matchLabels:
      app: nginx           # Deployment manages pods with this label
```

**Why Selector is Important:**
- Deployment uses this to identify which pods it manages
- Must match the labels in template.metadata.labels
- If mismatch → Deployment won't manage any pods ❌

**5. Pod Template:**
```yaml
  template:                # Template for pods
    metadata:
      labels:
        app: nginx         # Must match selector.matchLabels!
    spec:
      containers:
      - name: nginx-container     # Container name (required)
        image: nginx:latest       # Image with tag (required)
        ports:
        - containerPort: 80       # Port nginx listens on
```

**Label Matching Requirement:**
```yaml
selector:
  matchLabels:
    app: nginx           # This...

template:
  metadata:
    labels:
      app: nginx         # ...must match this!
```

#### Step 2: Validate Deployment YAML

```bash
# Dry-run validation
kubectl apply -f nginx-deployment.yaml --dry-run=client

# Expected Output:
# deployment.apps/nginx-deployment created (dry run)

# Check YAML syntax
cat nginx-deployment.yaml
```

**Validation Checklist:**
- ✅ apiVersion: apps/v1
- ✅ kind: Deployment
- ✅ name: nginx-deployment
- ✅ replicas: 3
- ✅ selector.matchLabels matches template.metadata.labels
- ✅ container name: nginx-container
- ✅ image: nginx:latest (tag specified)
- ✅ containerPort: 80

#### Step 3: Apply Deployment

```bash
# Create deployment
kubectl apply -f nginx-deployment.yaml

# Expected Output:
# deployment.apps/nginx-deployment created
```

#### Step 4: Verify Deployment Status

```bash
# Check deployment
kubectl get deployments

# Expected Output:
# NAME               READY   UP-TO-DATE   AVAILABLE   AGE
# nginx-deployment   3/3     3            3           30s
```

**Status Columns:**
- **READY:** 3/3 → All 3 replicas are running ✅
- **UP-TO-DATE:** 3 → All replicas have latest spec ✅
- **AVAILABLE:** 3 → All replicas are available to serve traffic ✅
- **AGE:** 30s → Time since deployment created

**If Not Ready:**
```bash
# NAME               READY   UP-TO-DATE   AVAILABLE   AGE
# nginx-deployment   1/3     3            1           10s
# ↑ Still creating replicas, wait a moment

# Check again after 20-30 seconds
kubectl get deployments
# Should show 3/3
```

#### Step 5: View Deployment Details

```bash
# Detailed deployment information
kubectl describe deployment nginx-deployment
```

**Expected Output (Key Sections):**
```
Name:                   nginx-deployment
Namespace:              default
CreationTimestamp:      Thu, 02 Jan 2026 10:00:00 +0000
Labels:                 app=nginx
Selector:               app=nginx
Replicas:               3 desired | 3 updated | 3 total | 3 available
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=nginx
  Containers:
   nginx-container:
    Image:        nginx:latest
    Port:         80/TCP
    Host Port:    0/TCP
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  40s   deployment-controller  Scaled up replica set nginx-deployment-xxxx to 3
```

#### Step 6: Verify ReplicaSet Created

```bash
# List ReplicaSets
kubectl get replicasets

# Expected Output:
# NAME                          DESIRED   CURRENT   READY   AGE
# nginx-deployment-7d6d8cfc8b   3         3         3       1m

# Or use shorter alias
kubectl get rs
```

**ReplicaSet Name:**
- Format: `<deployment-name>-<pod-template-hash>`
- Example: `nginx-deployment-7d6d8cfc8b`
- Hash ensures unique identification

#### Step 7: Verify Pods Created

```bash
# List pods
kubectl get pods

# Expected Output:
# NAME                                READY   STATUS    RESTARTS   AGE
# nginx-deployment-7d6d8cfc8b-abc12   1/1     Running   0          1m
# nginx-deployment-7d6d8cfc8b-def34   1/1     Running   0          1m
# nginx-deployment-7d6d8cfc8b-ghi56   1/1     Running   0          1m
```

**Pod Naming:**
- Format: `<deployment-name>-<replicaset-hash>-<random-string>`
- Example: `nginx-deployment-7d6d8cfc8b-abc12`
- Each pod has unique name

**Check Pod Distribution Across Nodes:**
```bash
# Show which node each pod is on
kubectl get pods -o wide

# Expected Output:
# NAME                                READY   STATUS    RESTARTS   AGE   IP           NODE
# nginx-deployment-7d6d8cfc8b-abc12   1/1     Running   0          2m    10.244.1.5   node01
# nginx-deployment-7d6d8cfc8b-def34   1/1     Running   0          2m    10.244.2.8   node02
# nginx-deployment-7d6d8cfc8b-ghi56   1/1     Running   0          2m    10.244.1.9   node01
```

**High Availability Verified:**
- Pods spread across multiple nodes (if multi-node cluster)
- Each pod has unique IP
- All pods running and ready

#### Step 8: Check Pod Labels

```bash
# View pod labels
kubectl get pods --show-labels

# Expected Output:
# NAME                                READY   STATUS    RESTARTS   AGE   LABELS
# nginx-deployment-7d6d8cfc8b-abc12   1/1     Running   0          3m    app=nginx,pod-template-hash=7d6d8cfc8b
# nginx-deployment-7d6d8cfc8b-def34   1/1     Running   0          3m    app=nginx,pod-template-hash=7d6d8cfc8b
# nginx-deployment-7d6d8cfc8b-ghi56   1/1     Running   0          3m    app=nginx,pod-template-hash=7d6d8cfc8b
```

**Important Labels:**
- `app=nginx` - Matches deployment selector ✅
- `pod-template-hash=7d6d8cfc8b` - ReplicaSet identifier

### Phase 2: Create NodePort Service

#### Step 9: Create Service YAML

```bash
# Create service manifest
vi nginx-service.yaml
```

**Complete Service Configuration:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
    nodePort: 30011
```

**YAML Breakdown:**

**1. API Version and Kind:**
```yaml
apiVersion: v1             # Core API (v1 for services)
kind: Service              # Resource type
```

**2. Metadata:**
```yaml
metadata:
  name: nginx-service      # Service name (required)
```

**3. Service Spec:**
```yaml
spec:
  type: NodePort           # Service type: NodePort for external access
```

**4. Selector (Routes Traffic):**
```yaml
  selector:
    app: nginx             # Routes to pods with label app=nginx
```

**Critical:** Service selector must match pod labels!
```yaml
# Service selector
selector:
  app: nginx

# Pod labels (from deployment template)
labels:
  app: nginx               # Must match! ✅
```

**5. Ports Configuration:**
```yaml
  ports:
  - protocol: TCP          # Protocol (TCP or UDP)
    port: 80               # Service port (ClusterIP port)
    targetPort: 80         # Container port (where nginx listens)
    nodePort: 30011        # External port on nodes (30000-32767)
```

**Port Flow:**
```
External Client
    ↓
Node IP:30011 (nodePort)
    ↓
Service ClusterIP:80 (port)
    ↓
Pod Container:80 (targetPort)
    ↓
Nginx Application
```

#### Step 10: Validate Service YAML

```bash
# Dry-run validation
kubectl apply -f nginx-service.yaml --dry-run=client

# Expected Output:
# service/nginx-service created (dry run)

# Check YAML syntax
cat nginx-service.yaml
```

**Validation Checklist:**
- ✅ apiVersion: v1
- ✅ kind: Service
- ✅ name: nginx-service
- ✅ type: NodePort
- ✅ selector: app=nginx (matches pod labels)
- ✅ port: 80
- ✅ targetPort: 80
- ✅ nodePort: 30011 (within 30000-32767 range)

#### Step 11: Apply Service

```bash
# Create service
kubectl apply -f nginx-service.yaml

# Expected Output:
# service/nginx-service created
```

#### Step 12: Verify Service Created

```bash
# List services
kubectl get services

# Expected Output:
# NAME            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
# kubernetes      ClusterIP   10.96.0.1       <none>        443/TCP        5d
# nginx-service   NodePort    10.96.100.50    <none>        80:30011/TCP   10s
```

**Service Details:**
- **NAME:** nginx-service ✅
- **TYPE:** NodePort ✅
- **CLUSTER-IP:** 10.96.100.50 (internal service IP)
- **PORT(S):** 80:30011/TCP
  - 80 = Service port
  - 30011 = NodePort ✅
- **EXTERNAL-IP:** <none> (NodePort doesn't get external IP, uses node IPs)

```bash
# Alternative short form
kubectl get svc

# Same output
```

#### Step 13: Describe Service

```bash
# Detailed service information
kubectl describe service nginx-service
```

**Expected Output:**
```
Name:                     nginx-service
Namespace:                default
Labels:                   <none>
Annotations:              <none>
Selector:                 app=nginx
Type:                     NodePort
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.96.100.50
IPs:                      10.96.100.50
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  30011/TCP
Endpoints:                10.244.1.5:80,10.244.1.9:80,10.244.2.8:80
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```

**Key Fields:**
- **Selector:** app=nginx → Matches pod labels ✅
- **Type:** NodePort ✅
- **IP:** 10.96.100.50 → ClusterIP for internal access
- **NodePort:** 30011/TCP ✅
- **Endpoints:** 10.244.1.5:80,10.244.1.9:80,10.244.2.8:80
  - These are the 3 pod IPs! ✅
  - Service automatically discovered pods via label selector

#### Step 14: Verify Service Endpoints

```bash
# Check endpoints (backend pods)
kubectl get endpoints nginx-service

# Expected Output:
# NAME            ENDPOINTS                                      AGE
# nginx-service   10.244.1.5:80,10.244.1.9:80,10.244.2.8:80     2m

# Or shorter
kubectl get ep nginx-service
```

**Endpoints Verification:**
- Should show 3 IP:port combinations (one per replica)
- IPs match pod IPs from `kubectl get pods -o wide`
- If no endpoints → Selector doesn't match any pods ❌

**Cross-Reference with Pod IPs:**
```bash
# Get pod IPs
kubectl get pods -o wide

# NAME                                READY   STATUS    RESTARTS   AGE   IP
# nginx-deployment-7d6d8cfc8b-abc12   1/1     Running   0          5m    10.244.1.5
# nginx-deployment-7d6d8cfc8b-def34   1/1     Running   0          5m    10.244.2.8
# nginx-deployment-7d6d8cfc8b-ghi56   1/1     Running   0          5m    10.244.1.9

# Compare with endpoints
kubectl get ep nginx-service
# ENDPOINTS: 10.244.1.5:80,10.244.1.9:80,10.244.2.8:80
# Perfect match! ✅
```

### Phase 3: Testing and Verification

#### Step 15: Get Node Information

```bash
# List nodes with IPs
kubectl get nodes -o wide

# Expected Output:
# NAME     STATUS   ROLES           AGE   VERSION   INTERNAL-IP    EXTERNAL-IP
# node01   Ready    control-plane   5d    v1.28.0   192.168.1.10   <none>
# node02   Ready    <none>          5d    v1.28.0   192.168.1.11   <none>
```

**Note Node IPs:**
- node01: 192.168.1.10
- node02: 192.168.1.11

**NodePort Service:** Accessible on ANY node IP at port 30011
- http://192.168.1.10:30011 ✅
- http://192.168.1.11:30011 ✅

#### Step 16: Test Service from Within Cluster

```bash
# Get service ClusterIP
kubectl get svc nginx-service -o jsonpath='{.spec.clusterIP}'
# Output: 10.96.100.50

# Test from a temporary pod
kubectl run test-pod --image=busybox --rm -it --restart=Never -- wget -O- http://10.96.100.50

# Expected Output:
# <!DOCTYPE html>
# <html>
# <head>
# <title>Welcome to nginx!</title>
# ...
# pod "test-pod" deleted
```

**What This Tests:**
- ✅ Service ClusterIP is reachable within cluster
- ✅ Service routes to pods correctly
- ✅ Nginx is serving content

#### Step 17: Test Service via NodePort

**Method 1: From Jump Host (if accessible):**
```bash
# Get node IP
NODE_IP=$(kubectl get nodes -o jsonpath='{.items[0].status.addresses[?(@.type=="InternalIP")].address}')
echo $NODE_IP

# Test via NodePort
curl http://$NODE_IP:30011

# Expected Output:
# <!DOCTYPE html>
# <html>
# <head>
# <title>Welcome to nginx!</title>
# ...
```

**Method 2: Port-forward for Testing:**
```bash
# Forward local port to service
kubectl port-forward service/nginx-service 8080:80 &

# Test
curl http://localhost:8080

# Expected Output:
# <!DOCTYPE html>
# <html>
# <head>
# <title>Welcome to nginx!</title>
```

**Method 3: Access from Browser:**
```
http://<node-ip>:30011
```

**In KodeKloud Environment:**
- Click "Select Port" button
- Enter: 30011
- Opens nginx welcome page in browser ✅

#### Step 18: Test Load Balancing

```bash
# Make multiple requests and check which pod serves each
for i in {1..10}; do
  kubectl run test-$i --image=busybox --rm -it --restart=Never -- wget -qO- http://nginx-service 2>/dev/null | grep -i "welcome"
done

# Each request may be served by different pod (round-robin)
```

**Check pod logs to see distribution:**
```bash
# Pod 1 logs
kubectl logs nginx-deployment-7d6d8cfc8b-abc12 --tail=5

# Pod 2 logs
kubectl logs nginx-deployment-7d6d8cfc8b-def34 --tail=5

# Pod 3 logs
kubectl logs nginx-deployment-7d6d8cfc8b-ghi56 --tail=5

# You should see requests distributed across all 3 pods
```

#### Step 19: Test High Availability

```bash
# Delete one pod
kubectl delete pod nginx-deployment-7d6d8cfc8b-abc12

# Immediately check pods
kubectl get pods

# Expected Output:
# NAME                                READY   STATUS              RESTARTS   AGE
# nginx-deployment-7d6d8cfc8b-abc12   1/1     Terminating         0          10m
# nginx-deployment-7d6d8cfc8b-def34   1/1     Running             0          10m
# nginx-deployment-7d6d8cfc8b-ghi56   1/1     Running             0          10m
# nginx-deployment-7d6d8cfc8b-xyz99   0/1     ContainerCreating   0          2s

# Wait a few seconds
kubectl get pods

# Expected Output:
# NAME                                READY   STATUS    RESTARTS   AGE
# nginx-deployment-7d6d8cfc8b-def34   1/1     Running   0          10m
# nginx-deployment-7d6d8cfc8b-ghi56   1/1     Running   0          10m
# nginx-deployment-7d6d8cfc8b-xyz99   1/1     Running   0          30s

# New pod automatically created to maintain 3 replicas! ✅
```

**During Pod Deletion:**
```bash
# Service continues working with remaining 2 pods
curl http://$NODE_IP:30011
# Still returns nginx page ✅ (served by other pods)
```

**Self-Healing Verified:**
- Deleted pod: abc12
- New pod created automatically: xyz99
- Replica count maintained: 3
- Service never went down: High Availability ✅

#### Step 20: Verify Service Discovery

```bash
# Check endpoints updated after pod recreation
kubectl get endpoints nginx-service

# Expected Output (new pod IP included):
# NAME            ENDPOINTS                                      AGE
# nginx-service   10.244.1.9:80,10.244.2.8:80,10.244.2.12:80    15m
#                 ↑ Old pod     ↑ Old pod     ↑ NEW pod
```

**Automatic Service Discovery:**
- Service automatically detects new pod
- Adds new pod IP to endpoints
- Removes terminated pod IP
- No manual intervention needed ✅

### Phase 4: Advanced Verification

#### Step 21: Check Deployment Rollout Status

```bash
# Check rollout status
kubectl rollout status deployment/nginx-deployment

# Expected Output:
# deployment "nginx-deployment" successfully rolled out
```

#### Step 22: View Deployment Revision History

```bash
# Check revision history
kubectl rollout history deployment/nginx-deployment

# Expected Output:
# deployment.apps/nginx-deployment
# REVISION  CHANGE-CAUSE
# 1         <none>
```

#### Step 23: Scale Deployment (Test Scalability)

```bash
# Scale up to 5 replicas
kubectl scale deployment nginx-deployment --replicas=5

# Expected Output:
# deployment.apps/nginx-deployment scaled

# Verify scaling
kubectl get deployment nginx-deployment

# Expected Output:
# NAME               READY   UP-TO-DATE   AVAILABLE   AGE
# nginx-deployment   5/5     5            5           20m

# Check pods
kubectl get pods

# Expected Output (5 pods now):
# NAME                                READY   STATUS    RESTARTS   AGE
# nginx-deployment-7d6d8cfc8b-abc12   1/1     Running   0          20m
# nginx-deployment-7d6d8cfc8b-def34   1/1     Running   0          20m
# nginx-deployment-7d6d8cfc8b-ghi56   1/1     Running   0          20m
# nginx-deployment-7d6d8cfc8b-jkl78   1/1     Running   0          10s
# nginx-deployment-7d6d8cfc8b-mno90   1/1     Running   0          10s
```

**Scale Back to 3:**
```bash
# Scale down to 3 replicas
kubectl scale deployment nginx-deployment --replicas=3

# Verify
kubectl get pods
# Back to 3 pods
```

#### Step 24: Check Resource Usage

```bash
# View resource usage
kubectl top pods

# Expected Output:
# NAME                                CPU(cores)   MEMORY(bytes)
# nginx-deployment-7d6d8cfc8b-abc12   1m           3Mi
# nginx-deployment-7d6d8cfc8b-def34   1m           3Mi
# nginx-deployment-7d6d8cfc8b-ghi56   1m           3Mi
```

#### Step 25: Test Service from All Nodes

```bash
# Get all node IPs
kubectl get nodes -o jsonpath='{.items[*].status.addresses[?(@.type=="InternalIP")].address}'

# Test service on each node IP
for node_ip in $(kubectl get nodes -o jsonpath='{.items[*].status.addresses[?(@.type=="InternalIP")].address}'); do
  echo "Testing node: $node_ip"
  curl -s http://$node_ip:30011 | grep -i "welcome to nginx"
done

# Expected: All nodes respond successfully ✅
```

## Complete Command Summary

### Deployment Commands
```bash
# Create deployment YAML
vi nginx-deployment.yaml

# Paste content:
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx-container
        image: nginx:latest
        ports:
        - containerPort: 80

# Apply deployment
kubectl apply -f nginx-deployment.yaml

# Verify deployment
kubectl get deployments
kubectl describe deployment nginx-deployment
kubectl get replicasets
kubectl get pods
kubectl get pods -o wide
kubectl get pods --show-labels
```

### Service Commands
```bash
# Create service YAML
vi nginx-service.yaml

# Paste content:
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
    nodePort: 30011

# Apply service
kubectl apply -f nginx-service.yaml

# Verify service
kubectl get services
kubectl describe service nginx-service
kubectl get endpoints nginx-service
```

### Testing Commands
```bash
# Test ClusterIP (internal)
kubectl run test-pod --image=busybox --rm -it --restart=Never -- wget -O- http://nginx-service

# Test NodePort (external)
NODE_IP=$(kubectl get nodes -o jsonpath='{.items[0].status.addresses[?(@.type=="InternalIP")].address}')
curl http://$NODE_IP:30011

# Test high availability
kubectl delete pod <pod-name>
kubectl get pods

# Scale deployment
kubectl scale deployment nginx-deployment --replicas=5
kubectl scale deployment nginx-deployment --replicas=3

# Check rollout status
kubectl rollout status deployment/nginx-deployment
kubectl rollout history deployment/nginx-deployment
```

### Monitoring Commands
```bash
# Watch pods
kubectl get pods -w

# Check logs
kubectl logs deployment/nginx-deployment
kubectl logs <pod-name>

# Check resource usage
kubectl top pods
kubectl top nodes

# Check events
kubectl get events --sort-by='.lastTimestamp'
```

### Cleanup Commands
```bash
# Delete service
kubectl delete service nginx-service

# Delete deployment (also deletes replicaset and pods)
kubectl delete deployment nginx-deployment

# Verify deletion
kubectl get all

# Delete YAML files (optional)
rm nginx-deployment.yaml nginx-service.yaml
```

## Troubleshooting Common Issues

### Issue 1: Deployment Not Creating Pods

**Symptoms:**
```bash
kubectl get deployment nginx-deployment
# NAME               READY   UP-TO-DATE   AVAILABLE   AGE
# nginx-deployment   0/3     0            0           2m

kubectl get pods
# No pods listed
```

**Diagnosis:**
```bash
# Check deployment events
kubectl describe deployment nginx-deployment

# Look for errors in Events section
```

**Common Causes:**

**1. Selector Mismatch:**
```yaml
selector:
  matchLabels:
    app: nginx        # This...

template:
  metadata:
    labels:
      app: web        # ...doesn't match this! ❌
```

**Fix:**
```yaml
selector:
  matchLabels:
    app: nginx

template:
  metadata:
    labels:
      app: nginx      # Must match! ✅
```

**2. Invalid Image:**
```yaml
containers:
- name: nginx-container
  image: nginx:lates  # Typo! (lates instead of latest)
```

**Fix:**
```yaml
containers:
- name: nginx-container
  image: nginx:latest  # Correct spelling ✅
```

### Issue 2: Pods Stuck in Pending

**Symptoms:**
```bash
kubectl get pods
# NAME                                READY   STATUS    RESTARTS   AGE
# nginx-deployment-7d6d8cfc8b-abc12   0/1     Pending   0          5m
```

**Diagnosis:**
```bash
# Check pod details
kubectl describe pod nginx-deployment-7d6d8cfc8b-abc12

# Look for scheduling errors
```

**Common Causes:**

**1. Insufficient Resources:**
```
Events:
  Warning  FailedScheduling  1m  default-scheduler  0/1 nodes are available: 1 Insufficient cpu
```

**Fix:**
- Reduce resource requests in deployment
- Add more nodes to cluster
- Remove resource-heavy pods

**2. Node Taints:**
```
Events:
  Warning  FailedScheduling  1m  default-scheduler  0/1 nodes are available: 1 node had untolerated taint
```

**Fix:**
Add tolerations or remove taints

### Issue 3: Service Has No Endpoints

**Symptoms:**
```bash
kubectl get endpoints nginx-service
# NAME            ENDPOINTS   AGE
# nginx-service   <none>      2m
```

**Diagnosis:**
```bash
# Check service selector
kubectl describe service nginx-service | grep Selector
# Selector: app=nginx

# Check pod labels
kubectl get pods --show-labels
# Check if any pods have label app=nginx
```

**Cause:**
Service selector doesn't match any pod labels

**Example:**
```yaml
# Service
selector:
  app: nginx        # Looking for this label...

# Pods (from deployment)
labels:
  app: web          # ...but pods have this! ❌ Mismatch
```

**Fix:**
Update service selector to match pod labels:
```yaml
selector:
  app: web          # Match pod labels ✅
```

Or update deployment labels to match service selector.

### Issue 4: Cannot Access via NodePort

**Symptoms:**
```bash
curl http://192.168.1.10:30011
# Connection refused or timeout
```

**Diagnosis Steps:**

**1. Check Service Type:**
```bash
kubectl get svc nginx-service
# TYPE should be NodePort, not ClusterIP
```

**2. Check NodePort Value:**
```bash
kubectl describe svc nginx-service | grep NodePort
# NodePort: <unset>  30011/TCP
```

**3. Check Firewall:**
```bash
# On node, check if port is open
sudo netstat -tlnp | grep 30011

# Check firewall rules
sudo iptables -L -n | grep 30011
```

**4. Test from Within Cluster:**
```bash
# If this works, issue is external access
kubectl run test --image=busybox --rm -it --restart=Never -- wget -O- http://nginx-service
```

**Common Solutions:**
- Open NodePort (30011) in firewall
- Verify kube-proxy is running on nodes
- Check network policies aren't blocking traffic

### Issue 5: Pods Crashing (CrashLoopBackOff)

**Symptoms:**
```bash
kubectl get pods
# NAME                                READY   STATUS             RESTARTS   AGE
# nginx-deployment-7d6d8cfc8b-abc12   0/1     CrashLoopBackOff   5          5m
```

**Diagnosis:**
```bash
# Check pod logs
kubectl logs nginx-deployment-7d6d8cfc8b-abc12

# Check previous logs
kubectl logs nginx-deployment-7d6d8cfc8b-abc12 --previous

# Describe pod
kubectl describe pod nginx-deployment-7d6d8cfc8b-abc12
```

**For Nginx, Common Causes:**
- Configuration file syntax error
- Port already in use
- Missing files or directories
- Permission issues

**For this task (nginx:latest):**
Should not crash unless node issues or resource constraints.

### Issue 6: Wrong Number of Replicas

**Symptoms:**
```bash
kubectl get deployment nginx-deployment
# NAME               READY   UP-TO-DATE   AVAILABLE   AGE
# nginx-deployment   2/3     3            2           5m
# Only 2/3 replicas ready
```

**Diagnosis:**
```bash
# Check pod status
kubectl get pods

# Check events
kubectl describe deployment nginx-deployment
```

**Common Causes:**
- Insufficient resources for all replicas
- Image pull failures
- Node issues

**Fix:**
Wait for pods to become ready, or check pod errors with `kubectl describe pod`

## Deployment Patterns and Best Practices

### 1. Replica Count Selection

**Development:**
```yaml
replicas: 1  # Single replica for dev/testing
```

**Staging:**
```yaml
replicas: 2  # Minimal HA for testing
```

**Production:**
```yaml
replicas: 3  # Minimum for HA (odd number preferred)
```

**High-Traffic Production:**
```yaml
replicas: 5+  # Scale based on load
```

**Why Odd Numbers?**
- Better for distributed consensus (if applicable)
- Avoids split-brain scenarios
- Examples: 3, 5, 7

### 2. Resource Requests and Limits

✅ **Good (With Resources):**
```yaml
containers:
- name: nginx-container
  image: nginx:latest
  resources:
    requests:
      memory: "64Mi"
      cpu: "100m"
    limits:
      memory: "128Mi"
      cpu: "200m"
```

**Why:**
- Ensures predictable scheduling
- Prevents resource starvation
- Better cluster utilization

### 3. Health Checks (Probes)

✅ **Good (With Probes):**
```yaml
containers:
- name: nginx-container
  image: nginx:latest
  livenessProbe:
    httpGet:
      path: /
      port: 80
    initialDelaySeconds: 10
    periodSeconds: 5
  readinessProbe:
    httpGet:
      path: /
      port: 80
    initialDelaySeconds: 5
    periodSeconds: 3
```

**Benefits:**
- **Liveness:** Restarts unhealthy containers
- **Readiness:** Removes unhealthy pods from service endpoints

### 4. Rolling Update Strategy

✅ **Good (Controlled Updates):**
```yaml
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1        # Max 1 extra pod during update
      maxUnavailable: 1  # Max 1 pod down during update
```

**What This Does:**
- Updates pods gradually (1 at a time)
- Ensures at least 2 pods always available
- Zero-downtime updates

### 5. Labels and Annotations

✅ **Good (Well-Labeled):**
```yaml
metadata:
  name: nginx-deployment
  labels:
    app: nginx
    tier: frontend
    environment: production
    version: v1.0
  annotations:
    description: "Nginx web server for static site"
    owner: "devops-team"
```

**Benefits:**
- Better organization
- Easier filtering: `kubectl get pods -l tier=frontend`
- Documentation

### 6. NodePort vs LoadBalancer vs Ingress

**Use NodePort When:**
- Development/testing
- On-premises without load balancer
- Direct node access is acceptable

**Use LoadBalancer When:**
- Production on cloud (AWS, GCP, Azure)
- Need dedicated external IP
- Single service exposure

**Use Ingress When:**
- Multiple services (HTTP/HTTPS)
- Path-based routing (/api, /web)
- SSL/TLS termination
- Cost optimization (single LB for multiple services)

### 7. Service Naming Convention

✅ **Good Names:**
```yaml
nginx-service       # Service for nginx deployment
api-service         # API backend service
database-service    # Database service
```

❌ **Bad Names:**
```yaml
svc1                # Non-descriptive
my-service          # Too generic
test                # Unclear purpose
```

### 8. Deployment Update Commands

```bash
# Update image
kubectl set image deployment/nginx-deployment nginx-container=nginx:1.21

# Update with YAML
kubectl apply -f nginx-deployment.yaml

# Rollback to previous version
kubectl rollout undo deployment/nginx-deployment

# Rollback to specific revision
kubectl rollout undo deployment/nginx-deployment --to-revision=2

# Pause rollout
kubectl rollout pause deployment/nginx-deployment

# Resume rollout
kubectl rollout resume deployment/nginx-deployment
```

### 9. Monitoring and Observability

```bash
# Watch deployment changes
kubectl get deployment nginx-deployment -w

# Monitor pod status
kubectl get pods -w

# View real-time logs
kubectl logs -f deployment/nginx-deployment

# Check pod metrics
kubectl top pods

# View all resources
kubectl get all -l app=nginx
```

### 10. Production Checklist

Before deploying to production:

- ✅ Replicas ≥ 3
- ✅ Resource requests and limits defined
- ✅ Liveness and readiness probes configured
- ✅ Rolling update strategy defined
- ✅ Labels and annotations documented
- ✅ Service type appropriate (LoadBalancer or Ingress for prod)
- ✅ Monitoring and logging configured
- ✅ Backup and disaster recovery plan
- ✅ Security policies applied
- ✅ Testing completed

## Completion Checklist

Verify your setup with these commands:

```bash
# 1. Deployment exists with correct name
kubectl get deployment nginx-deployment
# NAME: nginx-deployment ✅

# 2. Deployment has 3 replicas ready
kubectl get deployment nginx-deployment
# READY: 3/3 ✅

# 3. Deployment uses correct image
kubectl get deployment nginx-deployment -o jsonpath='{.spec.template.spec.containers[0].image}'
# Output: nginx:latest ✅

# 4. Container has correct name
kubectl get deployment nginx-deployment -o jsonpath='{.spec.template.spec.containers[0].name}'
# Output: nginx-container ✅

# 5. ReplicaSet created
kubectl get replicasets
# Shows nginx-deployment-xxxx with 3 desired/current/ready ✅

# 6. Pods created (3 replicas)
kubectl get pods -l app=nginx
# Shows 3 pods in Running status ✅

# 7. Service exists with correct name
kubectl get service nginx-service
# NAME: nginx-service ✅

# 8. Service type is NodePort
kubectl get service nginx-service -o jsonpath='{.spec.type}'
# Output: NodePort ✅

# 9. Service has correct NodePort
kubectl get service nginx-service -o jsonpath='{.spec.ports[0].nodePort}'
# Output: 30011 ✅

# 10. Service selector matches pod labels
kubectl get service nginx-service -o jsonpath='{.spec.selector}'
# Output: {"app":"nginx"} ✅

# 11. Service has 3 endpoints
kubectl get endpoints nginx-service
# Shows 3 IP:port entries ✅

# 12. Service is accessible via NodePort
NODE_IP=$(kubectl get nodes -o jsonpath='{.items[0].status.addresses[?(@.type=="InternalIP")].address}')
curl -s http://$NODE_IP:30011 | grep -i "welcome to nginx"
# Shows nginx welcome page ✅

# 13. High availability works (self-healing)
POD_NAME=$(kubectl get pods -l app=nginx -o jsonpath='{.items[0].metadata.name}')
kubectl delete pod $POD_NAME
sleep 10
kubectl get pods -l app=nginx
# Still shows 3 pods (new one created) ✅

# 14. Load balancing works
for i in {1..5}; do curl -s http://$NODE_IP:30011 | grep -i title; done
# All requests succeed ✅
```

**All checks passing = Task complete! ✅**

## Summary

### What We Accomplished
1. ✅ Created Kubernetes Deployment with 3 replicas
2. ✅ Used nginx:latest image with specified tag
3. ✅ Named container nginx-container
4. ✅ Created NodePort Service for external access
5. ✅ Configured NodePort 30011
6. ✅ Verified high availability (self-healing)
7. ✅ Tested load balancing across replicas
8. ✅ Demonstrated scalability (scale up/down)

### Key Concepts Learned

**Deployments:**
- Declarative management of application lifecycle
- Self-healing (automatic pod recreation)
- Scalability (easy replica management)
- Rolling updates (zero-downtime updates)
- Rollback capability

**High Availability:**
- Multiple replicas prevent single points of failure
- Pods distributed across nodes
- Service continues during pod failures
- Automatic pod replacement

**Services:**
- Stable networking for ephemeral pods
- Label-based pod selection
- NodePort for external access
- Automatic endpoint management
- Load balancing across healthy pods

### Critical Commands
```bash
# Create deployment (3 replicas)
kubectl apply -f nginx-deployment.yaml

# Create NodePort service (port 30011)
kubectl apply -f nginx-service.yaml

# Verify deployment
kubectl get deployments
kubectl get pods
kubectl describe deployment nginx-deployment

# Verify service
kubectl get services
kubectl get endpoints nginx-service
kubectl describe service nginx-service

# Test access
curl http://<node-ip>:30011

# Test high availability
kubectl delete pod <pod-name>
kubectl get pods  # New pod auto-created

# Scale deployment
kubectl scale deployment nginx-deployment --replicas=5
```

### Production Deployment Pattern

```
High Availability Setup:
├── Deployment: nginx-deployment
│   └── ReplicaSet
│       ├── Pod 1 (node01)
│       ├── Pod 2 (node02)
│       └── Pod 3 (node03)
├── Service: nginx-service (NodePort 30011)
│   └── Routes to all 3 pods
└── Benefits:
    ✅ Self-healing
    ✅ Load balancing
    ✅ Zero-downtime updates
    ✅ Easy scaling
    ✅ External access via NodePort
```

### Best Practices Applied
1. ✅ Multiple replicas (3) for high availability
2. ✅ Specified image tag (nginx:latest)
3. ✅ Named resources clearly
4. ✅ Used label selectors for service discovery
5. ✅ Tested self-healing and load balancing
6. ✅ Verified endpoint connectivity
7. ✅ Documented all steps
8. ✅ Provided troubleshooting guidance

### Real-World Applications

**Static Website Hosting:**
```
User Request
    ↓
Node IP:30011 (any node)
    ↓
nginx-service (load balances)
    ↓
One of 3 nginx pods
    ↓
Serve static HTML/CSS/JS
```

**Production Enhancements:**
- Add Ingress for domain-based access
- Use LoadBalancer service on cloud
- Add SSL/TLS certificates
- Configure resource limits
- Add monitoring and logging
- Implement autoscaling (HPA)

### What's Next?

**Day 57+:** Continue with Kubernetes topics:
- Ingress controllers (domain-based routing)
- ConfigMaps (application configuration)
- Secrets (sensitive data)
- Persistent storage (databases)
- StatefulSets (stateful applications)
- Horizontal Pod Autoscaler (auto-scaling)
- Network Policies (security)

**Learning Path:**
```
Day 56: Deployments + NodePort Service ← YOU ARE HERE
    ↓
Day 57+: Ingress (HTTP/HTTPS routing)
    ↓
Day 58+: ConfigMaps & Secrets
    ↓
Day 59+: Persistent Volumes
    ↓
Day 60+: StatefulSets
    ↓
Day 61+: Auto-scaling
```

🎉 **Day 56 Complete!** You've mastered Kubernetes Deployments and Services, creating a highly available, scalable web application with external access—essential skills for production Kubernetes deployments!
