# Day 58: Deploying Grafana on Kubernetes - Monitoring and Analytics Platform

## Objective
Deploy Grafana, a popular monitoring and analytics platform, on Kubernetes cluster using a Deployment and expose it via NodePort service. This demonstrates deploying real-world applications with persistent configuration and external access patterns.

## Task Requirements
- **Environment:** Kubernetes cluster accessible from jump_host
- **Deployment Name:** grafana-deployment-devops
- **Image:** Any grafana image (we'll use grafana/grafana:latest)
- **Service Type:** NodePort
- **NodePort:** 32000
- **Goal:** Access Grafana login page
- **Tool:** kubectl (pre-configured on jump_host)

## Understanding Grafana

### What is Grafana?

**Grafana** is an open-source analytics and interactive visualization platform. It provides:
- **Dashboards:** Beautiful, customizable dashboards
- **Data Sources:** Connects to Prometheus, InfluxDB, MySQL, PostgreSQL, etc.
- **Alerting:** Alert notifications to Slack, email, PagerDuty
- **Visualization:** Graphs, charts, tables, heatmaps
- **Monitoring:** Infrastructure, application, and business metrics

### Why Grafana?

**Use Cases:**
```
Infrastructure Monitoring:
├── Kubernetes cluster metrics
├── Server CPU, memory, disk usage
├── Network traffic
└── Application performance

Application Monitoring:
├── API response times
├── Request rates
├── Error rates
└── User analytics

Business Metrics:
├── Sales dashboards
├── User engagement
├── Revenue tracking
└── KPI visualization
```

### Grafana Architecture

**Standalone Deployment:**
```
User Browser
    ↓
Grafana Web UI (Port 3000)
    ↓
Grafana Server
    ├── Query Data Sources
    │   ├── Prometheus
    │   ├── InfluxDB
    │   ├── MySQL
    │   └── PostgreSQL
    └── Render Dashboards
```

**Our Kubernetes Deployment:**
```
External User
    ↓
Node IP:32000 (NodePort)
    ↓
Grafana Service (ClusterIP)
    ↓
Grafana Pod (Deployment)
    └── Container (grafana/grafana:latest)
        └── Listening on port 3000
```

### Grafana Default Configuration

**Default Settings:**
- **Port:** 3000 (HTTP)
- **Default Credentials:**
  - Username: `admin`
  - Password: `admin` (prompts to change on first login)
- **Data Directory:** `/var/lib/grafana`
- **Logs:** `/var/log/grafana`
- **Config:** `/etc/grafana/grafana.ini`

## Understanding Kubernetes Deployment for Applications

### Deployment vs Pod for Applications

**Using Just a Pod (Not Recommended):**
```yaml
kind: Pod
Problems:
❌ Pod dies → Manual recreation needed
❌ No scaling
❌ No rolling updates
❌ No self-healing
```

**Using Deployment (Recommended):**
```yaml
kind: Deployment
Benefits:
✅ Pod dies → Automatically recreated
✅ Easy scaling (1 to N replicas)
✅ Rolling updates (zero downtime)
✅ Rollback capability
✅ Self-healing
```

### StatefulSet vs Deployment

**For Grafana, we use Deployment because:**
```
Deployment (Stateless-friendly):
✅ Single instance sufficient for testing
✅ Configuration can be in ConfigMap
✅ Data can be in PersistentVolume
✅ Simple to deploy and manage
✅ Easy to update/rollback

StatefulSet (For stateful apps):
- Stable network identities
- Ordered deployment/scaling
- Stable persistent storage
- More complex (overkill for single Grafana)
```

### NodePort for External Access

**Why NodePort for Grafana?**

**NodePort Advantages:**
```
✅ Simple external access
✅ No cloud provider needed
✅ Works on-premises
✅ Good for dev/testing
✅ Direct node access
```

**NodePort vs Other Options:**

**1. ClusterIP (Internal Only):**
```
Use for: Internal services only
Access: Only within cluster ❌
Grafana: Need external access, so NO
```

**2. NodePort (External via Node):**
```
Use for: Dev, testing, on-prem
Access: <NodeIP>:32000 ✅
Grafana: Perfect for this task ✅
```

**3. LoadBalancer (Cloud):**
```
Use for: Production on cloud
Access: External IP from cloud LB
Grafana: Overkill for this task
Requires: Cloud provider (AWS, GCP, Azure)
```

**4. Ingress (HTTP/HTTPS Routing):**
```
Use for: Multiple services, domain-based routing
Access: grafana.example.com
Grafana: Production setup
Requires: Ingress controller
```

## Step-by-Step Implementation

### Phase 1: Create Grafana Deployment

#### Step 1: Create Deployment YAML

```bash
# Create deployment manifest
vi grafana-deployment-devops.yaml
```

**Complete Deployment Configuration:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: grafana-deployment-devops
  labels:
    app: grafana
spec:
  replicas: 1
  selector:
    matchLabels:
      app: grafana
  template:
    metadata:
      labels:
        app: grafana
    spec:
      containers:
      - name: grafana
        image: grafana/grafana:latest
        ports:
        - containerPort: 3000
          name: grafana-http
          protocol: TCP
```

**YAML Breakdown:**

**1. Deployment Metadata:**
```yaml
apiVersion: apps/v1              # Deployment API version
kind: Deployment                 # Resource type
metadata:
  name: grafana-deployment-devops  # Deployment name (required)
  labels:
    app: grafana                 # Label for organization
```

**2. Replica Configuration:**
```yaml
spec:
  replicas: 1                    # Single replica (sufficient for monitoring tool)
```

**Why 1 Replica?**
- Grafana is a dashboard tool (not high-traffic)
- Single instance sufficient for dev/testing
- Can scale later if needed: `kubectl scale deployment grafana-deployment-devops --replicas=3`

**3. Selector (Pod Selection):**
```yaml
  selector:
    matchLabels:
      app: grafana               # Deployment manages pods with this label
```

**4. Pod Template:**
```yaml
  template:
    metadata:
      labels:
        app: grafana             # Must match selector.matchLabels
    spec:
      containers:
      - name: grafana            # Container name
        image: grafana/grafana:latest  # Official Grafana image
```

**Image Options:**
- `grafana/grafana:latest` - Latest version (we'll use this)
- `grafana/grafana:10.2.3` - Specific version
- `grafana/grafana:10.2.3-ubuntu` - Ubuntu-based

**5. Port Configuration:**
```yaml
        ports:
        - containerPort: 3000    # Grafana listens on port 3000
          name: grafana-http     # Named port (optional but good practice)
          protocol: TCP          # Protocol (TCP for HTTP)
```

**Named Ports Benefits:**
- Service can reference by name instead of number
- Self-documenting
- Easier to change port later

**Enhanced Configuration (Optional but Recommended):**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: grafana-deployment-devops
  labels:
    app: grafana
    component: monitoring
spec:
  replicas: 1
  selector:
    matchLabels:
      app: grafana
  template:
    metadata:
      labels:
        app: grafana
        component: monitoring
    spec:
      containers:
      - name: grafana
        image: grafana/grafana:latest
        ports:
        - containerPort: 3000
          name: grafana-http
          protocol: TCP
        env:
        - name: GF_SERVER_ROOT_URL
          value: "http://localhost:3000"
        - name: GF_SECURITY_ADMIN_PASSWORD
          value: "admin"
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /api/health
            port: 3000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /api/health
            port: 3000
          initialDelaySeconds: 10
          periodSeconds: 5
```

**Enhanced Features Explained:**

**Environment Variables:**
```yaml
env:
- name: GF_SERVER_ROOT_URL
  value: "http://localhost:3000"  # Base URL for Grafana
- name: GF_SECURITY_ADMIN_PASSWORD
  value: "admin"                   # Set admin password (avoid first-login prompt)
```

**Resource Limits:**
```yaml
resources:
  requests:              # Minimum resources guaranteed
    memory: "128Mi"      # Needs at least 128MB RAM
    cpu: "100m"          # Needs at least 0.1 CPU core
  limits:                # Maximum resources allowed
    memory: "512Mi"      # Can use up to 512MB RAM
    cpu: "500m"          # Can use up to 0.5 CPU core
```

**Liveness Probe (Restart if unhealthy):**
```yaml
livenessProbe:
  httpGet:
    path: /api/health    # Grafana health check endpoint
    port: 3000
  initialDelaySeconds: 30  # Wait 30s before first check
  periodSeconds: 10        # Check every 10s
```

**Readiness Probe (Remove from service if not ready):**
```yaml
readinessProbe:
  httpGet:
    path: /api/health
    port: 3000
  initialDelaySeconds: 10
  periodSeconds: 5
```

**For This Task:** Basic configuration is sufficient, but enhanced version is production-ready.

#### Step 2: Validate Deployment YAML

```bash
# Dry-run validation
kubectl apply -f grafana-deployment-devops.yaml --dry-run=client

# Expected Output:
# deployment.apps/grafana-deployment-devops created (dry run)

# Check YAML syntax
cat grafana-deployment-devops.yaml
```

**Validation Checklist:**
- ✅ apiVersion: apps/v1
- ✅ kind: Deployment
- ✅ name: grafana-deployment-devops
- ✅ replicas: 1 (or any number)
- ✅ selector.matchLabels matches template.metadata.labels
- ✅ container name: grafana (or any name)
- ✅ image: grafana/grafana:latest (or any grafana image)
- ✅ containerPort: 3000
- ✅ No YAML syntax errors

#### Step 3: Apply Deployment

```bash
# Create deployment
kubectl apply -f grafana-deployment-devops.yaml

# Expected Output:
# deployment.apps/grafana-deployment-devops created
```

#### Step 4: Verify Deployment Status

```bash
# Check deployment
kubectl get deployments

# Expected Output:
# NAME                        READY   UP-TO-DATE   AVAILABLE   AGE
# grafana-deployment-devops   1/1     1            1           30s
```

**Status Indicators:**
- **READY: 1/1** → All replicas running ✅
- **UP-TO-DATE: 1** → All replicas have latest spec ✅
- **AVAILABLE: 1** → All replicas available to serve traffic ✅

**If Not Ready:**
```bash
# NAME                        READY   UP-TO-DATE   AVAILABLE   AGE
# grafana-deployment-devops   0/1     1            0           15s

# Wait 30-60 seconds for image pull and container startup
kubectl get deployments -w  # Watch mode

# Should transition to 1/1 after container starts
```

#### Step 5: Check ReplicaSet and Pods

```bash
# List ReplicaSets
kubectl get replicasets

# Expected Output:
# NAME                                   DESIRED   CURRENT   READY   AGE
# grafana-deployment-devops-7d6d8cfc8b   1         1         1       1m

# List pods
kubectl get pods

# Expected Output:
# NAME                                         READY   STATUS    RESTARTS   AGE
# grafana-deployment-devops-7d6d8cfc8b-abc12   1/1     Running   0          1m

# Check pod with labels
kubectl get pods -l app=grafana

# Show pod details
kubectl get pods -o wide

# Expected Output:
# NAME                                         READY   STATUS    RESTARTS   AGE   IP           NODE
# grafana-deployment-devops-7d6d8cfc8b-abc12   1/1     Running   0          2m    10.244.1.5   node01
```

#### Step 6: Describe Deployment

```bash
# Detailed deployment information
kubectl describe deployment grafana-deployment-devops
```

**Expected Output (Key Sections):**
```
Name:                   grafana-deployment-devops
Namespace:              default
Labels:                 app=grafana
Selector:               app=grafana
Replicas:               1 desired | 1 updated | 1 total | 1 available
Pod Template:
  Labels:  app=grafana
  Containers:
   grafana:
    Image:        grafana/grafana:latest
    Port:         3000/TCP
    Host Port:    0/TCP
Events:
  Type    Reason             Age    From                   Message
  ----    ------             ----   ----                   -------
  Normal  ScalingReplicaSet  2m30s  deployment-controller  Scaled up replica set grafana-deployment-devops-xxx to 1
```

#### Step 7: Check Pod Logs

```bash
# View Grafana startup logs
kubectl logs deployment/grafana-deployment-devops

# Or specify pod name
POD_NAME=$(kubectl get pods -l app=grafana -o jsonpath='{.items[0].metadata.name}')
kubectl logs $POD_NAME

# Follow logs (real-time)
kubectl logs -f $POD_NAME
```

**Expected Log Output (Key Lines):**
```
logger=settings t=2026-01-02T10:00:00+0000 lvl=info msg="Starting Grafana" version=10.2.3
logger=server t=2026-01-02T10:00:01+0000 lvl=info msg="HTTP Server Listen" address=[::]:3000 protocol=http
logger=infra.usagestats t=2026-01-02T10:00:02+0000 lvl=info msg="Usage stats are ready to report"
```

**What to Look For:**
- ✅ "Starting Grafana" - Grafana started successfully
- ✅ "HTTP Server Listen" address=[::]:3000 - Server listening on port 3000
- ✅ No error messages
- ❌ If errors about permissions or missing files → Check pod describe

### Phase 2: Create NodePort Service

#### Step 8: Create Service YAML

```bash
# Create service manifest
vi grafana-service.yaml
```

**Complete Service Configuration:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: grafana-service
  labels:
    app: grafana
spec:
  type: NodePort
  selector:
    app: grafana
  ports:
  - port: 3000
    targetPort: 3000
    nodePort: 32000
    protocol: TCP
    name: grafana-http
```

**YAML Breakdown:**

**1. Service Metadata:**
```yaml
apiVersion: v1                   # Core API
kind: Service                    # Resource type
metadata:
  name: grafana-service          # Service name
  labels:
    app: grafana                 # Label for organization
```

**2. Service Type:**
```yaml
spec:
  type: NodePort                 # External access via node port
```

**3. Pod Selector:**
```yaml
  selector:
    app: grafana                 # Routes to pods with label app=grafana
```

**Critical:** Selector must match pod labels from deployment!
```yaml
# Service selector
selector:
  app: grafana

# Deployment pod labels
template:
  metadata:
    labels:
      app: grafana               # Must match! ✅
```

**4. Port Configuration:**
```yaml
  ports:
  - port: 3000                   # Service port (ClusterIP port)
    targetPort: 3000             # Container port (where Grafana listens)
    nodePort: 32000              # External port on nodes (30000-32767)
    protocol: TCP                # Protocol
    name: grafana-http           # Named port (optional)
```

**Port Flow:**
```
External Client
    ↓
Node IP:32000 (nodePort)
    ↓
Service ClusterIP:3000 (port)
    ↓
Pod Container:3000 (targetPort)
    ↓
Grafana Application
```

**Alternative: Using Named Port:**
```yaml
# In Deployment:
ports:
- containerPort: 3000
  name: grafana-http             # Named port

# In Service (reference by name):
ports:
- port: 3000
  targetPort: grafana-http       # Reference named port instead of number
  nodePort: 32000
```

#### Step 9: Validate Service YAML

```bash
# Dry-run validation
kubectl apply -f grafana-service.yaml --dry-run=client

# Expected Output:
# service/grafana-service created (dry run)

# Check YAML syntax
cat grafana-service.yaml
```

**Validation Checklist:**
- ✅ apiVersion: v1
- ✅ kind: Service
- ✅ name: grafana-service
- ✅ type: NodePort
- ✅ selector: app=grafana (matches pod labels)
- ✅ port: 3000
- ✅ targetPort: 3000
- ✅ nodePort: 32000 (within 30000-32767 range)
- ✅ No YAML syntax errors

#### Step 10: Apply Service

```bash
# Create service
kubectl apply -f grafana-service.yaml

# Expected Output:
# service/grafana-service created
```

#### Step 11: Verify Service Created

```bash
# List services
kubectl get services

# Expected Output:
# NAME              TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
# kubernetes        ClusterIP   10.96.0.1       <none>        443/TCP          5d
# grafana-service   NodePort    10.96.100.50    <none>        3000:32000/TCP   10s
```

**Service Details:**
- **NAME:** grafana-service ✅
- **TYPE:** NodePort ✅
- **CLUSTER-IP:** 10.96.100.50 (internal service IP)
- **PORT(S):** 3000:32000/TCP
  - 3000 = Service port
  - 32000 = NodePort ✅
- **EXTERNAL-IP:** <none> (NodePort doesn't get external IP)

```bash
# Alternative short form
kubectl get svc

# Specific service
kubectl get svc grafana-service
```

#### Step 12: Describe Service

```bash
# Detailed service information
kubectl describe service grafana-service
```

**Expected Output:**
```
Name:                     grafana-service
Namespace:                default
Labels:                   app=grafana
Selector:                 app=grafana
Type:                     NodePort
IP Family Policy:         SingleStack
IP:                       10.96.100.50
IPs:                      10.96.100.50
Port:                     grafana-http  3000/TCP
TargetPort:               3000/TCP
NodePort:                 grafana-http  32000/TCP
Endpoints:                10.244.1.5:3000
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```

**Key Verification Points:**
- **Selector:** app=grafana ✅
- **Type:** NodePort ✅
- **NodePort:** 32000/TCP ✅
- **Endpoints:** 10.244.1.5:3000 ✅ (pod IP, should have 1 entry)

#### Step 13: Verify Service Endpoints

```bash
# Check endpoints (backend pods)
kubectl get endpoints grafana-service

# Expected Output:
# NAME              ENDPOINTS         AGE
# grafana-service   10.244.1.5:3000   2m

# Or shorter
kubectl get ep grafana-service
```

**Endpoint Verification:**
- Should show 1 IP:port (10.244.1.5:3000)
- IP matches pod IP from `kubectl get pods -o wide`
- If no endpoints → Selector doesn't match pod labels ❌

**Cross-Reference:**
```bash
# Get pod IP
kubectl get pods -l app=grafana -o wide
# IP: 10.244.1.5

# Get service endpoints
kubectl get ep grafana-service
# ENDPOINTS: 10.244.1.5:3000

# Perfect match! ✅
```

### Phase 3: Access Grafana

#### Step 14: Get Node Information

```bash
# List nodes with IPs
kubectl get nodes -o wide

# Expected Output:
# NAME     STATUS   ROLES           AGE   VERSION   INTERNAL-IP    EXTERNAL-IP
# node01   Ready    control-plane   5d    v1.28.0   192.168.1.10   <none>
```

**Note Node IP:**
- Example: 192.168.1.10
- Grafana accessible at: http://192.168.1.10:32000

#### Step 15: Test Service from Within Cluster

```bash
# Get service ClusterIP
kubectl get svc grafana-service -o jsonpath='{.spec.clusterIP}'
# Output: 10.96.100.50

# Test from temporary pod
kubectl run test-grafana --image=busybox --rm -it --restart=Never -- wget -O- http://10.96.100.50:3000/api/health

# Expected Output:
# {
#   "commit": "abc123",
#   "database": "ok",
#   "version": "10.2.3"
# }
# pod "test-grafana" deleted
```

#### Step 16: Access Grafana via NodePort

**Method 1: Using curl (from jump_host):**
```bash
# Get node IP
NODE_IP=$(kubectl get nodes -o jsonpath='{.items[0].status.addresses[?(@.type=="InternalIP")].address}')
echo $NODE_IP

# Test Grafana health endpoint
curl http://$NODE_IP:32000/api/health

# Expected Output:
# {
#   "commit": "abc123def456",
#   "database": "ok",
#   "version": "10.2.3"
# }

# Test Grafana login page
curl -I http://$NODE_IP:32000/login

# Expected Output:
# HTTP/1.1 200 OK
# Content-Type: text/html; charset=UTF-8
# ...
```

**Method 2: Port-forward (alternative for testing):**
```bash
# Forward local port to service
kubectl port-forward service/grafana-service 8080:3000 &

# Test
curl http://localhost:8080/api/health

# Stop port-forward
kill %1
```

**Method 3: Access from Browser:**

**In KodeKloud/Similar Environment:**
1. Click "Select Port" or "Open Port" button
2. Enter port: **32000**
3. Browser opens Grafana login page ✅

**Direct Access (if node IP accessible):**
- Open browser
- Navigate to: `http://<node-ip>:32000`
- Example: `http://192.168.1.10:32000`

**Expected Grafana Login Page:**
```
┌────────────────────────────────────┐
│         Grafana Logo               │
│                                    │
│  Email or username                 │
│  [__________________]              │
│                                    │
│  Password                          │
│  [__________________]              │
│                                    │
│  [ Log in ]                        │
│                                    │
│  Forgot your password?             │
└────────────────────────────────────┘
```

#### Step 17: Login to Grafana

**Default Credentials:**
- **Username:** admin
- **Password:** admin

**First Login Steps:**
1. Enter username: `admin`
2. Enter password: `admin`
3. Click "Log in"
4. Prompted to change password (optional for this task)
5. Skip or set new password
6. Welcome to Grafana! ✅

**After Login, You'll See:**
- Grafana Home Dashboard
- Left sidebar with menu options:
  - Dashboards
  - Explore
  - Alerting
  - Configuration
  - Data Sources
- Welcome message and getting started guides

#### Step 18: Verify Grafana is Working

**Test from Command Line:**
```bash
# Check login API (should require authentication)
curl -u admin:admin http://$NODE_IP:32000/api/org

# Expected Output:
# {
#   "id": 1,
#   "name": "Main Org.",
#   "address": {
#     "address1": "",
#     "address2": "",
#     "city": "",
#     "zipCode": "",
#     "state": "",
#     "country": ""
#   }
# }

# Check datasources (empty initially)
curl -u admin:admin http://$NODE_IP:32000/api/datasources

# Expected Output:
# []
```

**Visual Verification:**
- ✅ Login page loads
- ✅ Can login with admin/admin
- ✅ Dashboard interface visible
- ✅ Can navigate menus
- ✅ No error messages

### Phase 4: Verification and Testing

#### Step 19: Check All Resources

```bash
# View all Grafana-related resources
kubectl get all -l app=grafana

# Expected Output:
# NAME                                             READY   STATUS    RESTARTS   AGE
# pod/grafana-deployment-devops-7d6d8cfc8b-abc12   1/1     Running   0          10m
#
# NAME                      TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
# service/grafana-service   NodePort   10.96.100.50   <none>        3000:32000/TCP   8m
#
# NAME                                        READY   UP-TO-DATE   AVAILABLE   AGE
# deployment.apps/grafana-deployment-devops   1/1     1            1           10m
#
# NAME                                                   DESIRED   CURRENT   READY   AGE
# replicaset.apps/grafana-deployment-devops-7d6d8cfc8b   1         1         1       10m
```

**All Resources Present:**
- ✅ Pod: grafana-deployment-devops-xxx (Running)
- ✅ Service: grafana-service (NodePort)
- ✅ Deployment: grafana-deployment-devops (1/1)
- ✅ ReplicaSet: grafana-deployment-devops-xxx (1/1)

#### Step 20: Test High Availability

```bash
# Delete Grafana pod
kubectl delete pod -l app=grafana

# Immediately check pods
kubectl get pods -l app=grafana

# Expected Output (pod recreating):
# NAME                                         READY   STATUS              RESTARTS   AGE
# grafana-deployment-devops-7d6d8cfc8b-xyz99   0/1     ContainerCreating   0          2s

# Wait a moment
kubectl get pods -l app=grafana

# Expected Output (new pod running):
# NAME                                         READY   STATUS    RESTARTS   AGE
# grafana-deployment-devops-7d6d8cfc8b-xyz99   1/1     Running   0          30s

# New pod automatically created! ✅
# Service still works (access http://<node-ip>:32000)
```

#### Step 21: Test Service Discovery

```bash
# Check if service automatically updated endpoints
kubectl get endpoints grafana-service

# Expected Output (new pod IP):
# NAME              ENDPOINTS          AGE
# grafana-service   10.244.1.8:3000    12m
#                   ↑ New pod IP (different from before)

# Service automatically discovered new pod ✅
```

#### Step 22: Verify Port Mapping

```bash
# Check NodePort configuration
kubectl get svc grafana-service -o jsonpath='{.spec.ports[0].nodePort}'
# Output: 32000 ✅

# Check service port
kubectl get svc grafana-service -o jsonpath='{.spec.ports[0].port}'
# Output: 3000 ✅

# Check target port
kubectl get svc grafana-service -o jsonpath='{.spec.ports[0].targetPort}'
# Output: 3000 ✅
```

#### Step 23: Check Grafana Health

```bash
# Health check endpoint
curl http://$NODE_IP:32000/api/health

# Metrics endpoint (if enabled)
curl http://$NODE_IP:32000/metrics

# API endpoint
curl http://$NODE_IP:32000/api/org
# Requires authentication (returns 401 if not logged in)
```

#### Step 24: Test from Multiple Nodes (if multi-node cluster)

```bash
# Get all node IPs
kubectl get nodes -o jsonpath='{.items[*].status.addresses[?(@.type=="InternalIP")].address}'

# Test service on each node
for node_ip in $(kubectl get nodes -o jsonpath='{.items[*].status.addresses[?(@.type=="InternalIP")].address}'); do
  echo "Testing node: $node_ip"
  curl -s http://$node_ip:32000/api/health | grep -q "ok" && echo "✅ Healthy" || echo "❌ Failed"
done

# Expected: All nodes respond ✅
```

## Complete Command Summary

### Deployment Commands
```bash
# Create deployment YAML
vi grafana-deployment-devops.yaml

# Paste content:
apiVersion: apps/v1
kind: Deployment
metadata:
  name: grafana-deployment-devops
  labels:
    app: grafana
spec:
  replicas: 1
  selector:
    matchLabels:
      app: grafana
  template:
    metadata:
      labels:
        app: grafana
    spec:
      containers:
      - name: grafana
        image: grafana/grafana:latest
        ports:
        - containerPort: 3000
          name: grafana-http
          protocol: TCP

# Apply deployment
kubectl apply -f grafana-deployment-devops.yaml

# Verify deployment
kubectl get deployments
kubectl get pods
kubectl get pods -o wide
kubectl describe deployment grafana-deployment-devops

# Check logs
kubectl logs deployment/grafana-deployment-devops
kubectl logs -f deployment/grafana-deployment-devops
```

### Service Commands
```bash
# Create service YAML
vi grafana-service.yaml

# Paste content:
apiVersion: v1
kind: Service
metadata:
  name: grafana-service
  labels:
    app: grafana
spec:
  type: NodePort
  selector:
    app: grafana
  ports:
  - port: 3000
    targetPort: 3000
    nodePort: 32000
    protocol: TCP
    name: grafana-http

# Apply service
kubectl apply -f grafana-service.yaml

# Verify service
kubectl get services
kubectl get svc grafana-service
kubectl describe service grafana-service
kubectl get endpoints grafana-service
```

### Testing Commands
```bash
# Get node IP
NODE_IP=$(kubectl get nodes -o jsonpath='{.items[0].status.addresses[?(@.type=="InternalIP")].address}')
echo "Grafana URL: http://$NODE_IP:32000"

# Test health endpoint
curl http://$NODE_IP:32000/api/health

# Test login page
curl -I http://$NODE_IP:32000/login

# Test with authentication
curl -u admin:admin http://$NODE_IP:32000/api/org

# Test from within cluster
kubectl run test-pod --image=busybox --rm -it --restart=Never -- wget -O- http://grafana-service:3000/api/health
```

### Monitoring Commands
```bash
# Watch deployment
kubectl get deployment grafana-deployment-devops -w

# Watch pods
kubectl get pods -l app=grafana -w

# View logs
kubectl logs -f deployment/grafana-deployment-devops

# Check resource usage
kubectl top pod -l app=grafana

# View events
kubectl get events --sort-by='.lastTimestamp' | grep grafana
```

### One-Liner Alternative (Without YAML Files)
```bash
# Create deployment
kubectl create deployment grafana-deployment-devops \
  --image=grafana/grafana:latest \
  --port=3000

# Expose as NodePort service
kubectl expose deployment grafana-deployment-devops \
  --name=grafana-service \
  --type=NodePort \
  --port=3000 \
  --target-port=3000

# Update NodePort to 32000 (requires patch)
kubectl patch service grafana-service \
  --type='json' \
  -p='[{"op": "replace", "path": "/spec/ports/0/nodePort", "value":32000}]'

# Verify
kubectl get all
```

### Cleanup Commands
```bash
# Delete service
kubectl delete service grafana-service

# Delete deployment (also deletes replicaset and pods)
kubectl delete deployment grafana-deployment-devops

# Verify deletion
kubectl get all -l app=grafana

# Delete YAML files (optional)
rm grafana-deployment-devops.yaml grafana-service.yaml
```

## Troubleshooting Common Issues

### Issue 1: Pod Stuck in "ImagePullBackOff"

**Symptoms:**
```bash
kubectl get pods
# NAME                                         READY   STATUS             RESTARTS   AGE
# grafana-deployment-devops-7d6d8cfc8b-abc12   0/1     ImagePullBackOff   0          2m
```

**Diagnosis:**
```bash
kubectl describe pod <pod-name>

# Look for:
# Events:
#   Failed to pull image "grafana/grafana:latest": rpc error: image not found
```

**Common Causes:**

**1. Wrong Image Name:**
```yaml
❌ WRONG:
image: grafanna/grafana:latest  # Typo: grafanna

✅ CORRECT:
image: grafana/grafana:latest
```

**2. Network Issues:**
- Node can't reach Docker Hub
- Check: `kubectl describe node` for network issues

**3. Private Image Without Credentials:**
- If using private registry, add imagePullSecrets

**Fix:**
1. Correct image name in YAML
2. Reapply: `kubectl apply -f grafana-deployment-devops.yaml`
3. Check pod status again

### Issue 2: Pod Stuck in "CrashLoopBackOff"

**Symptoms:**
```bash
kubectl get pods
# NAME                                         READY   STATUS             RESTARTS   AGE
# grafana-deployment-devops-7d6d8cfc8b-abc12   0/1     CrashLoopBackOff   5          3m
```

**Diagnosis:**
```bash
# Check pod logs
kubectl logs <pod-name>

# Check previous logs (from crashed container)
kubectl logs <pod-name> --previous

# Describe pod
kubectl describe pod <pod-name>
```

**Common Causes for Grafana:**

**1. Permission Issues:**
```
Error: mkdir /var/lib/grafana/plugins: permission denied
```

**Fix:** Add securityContext to allow write access
```yaml
spec:
  containers:
  - name: grafana
    image: grafana/grafana:latest
    securityContext:
      runAsUser: 472  # Grafana user ID
      fsGroup: 472
```

**2. Configuration Errors:**
```
Error: invalid configuration in grafana.ini
```

**Fix:** Check environment variables or mounted config

**3. Resource Constraints:**
```
Events:
  OOMKilled  # Out of memory
```

**Fix:** Increase memory limits
```yaml
resources:
  limits:
    memory: "512Mi"  # Increase from default
```

### Issue 3: Service Has No Endpoints

**Symptoms:**
```bash
kubectl get endpoints grafana-service
# NAME              ENDPOINTS   AGE
# grafana-service   <none>      2m
```

**Diagnosis:**
```bash
# Check service selector
kubectl describe service grafana-service | grep Selector
# Selector: app=grafana

# Check pod labels
kubectl get pods --show-labels | grep grafana

# If pod doesn't have label app=grafana → Mismatch!
```

**Cause:**
Service selector doesn't match pod labels

**Example:**
```yaml
# Service
selector:
  app: grafana        # Looking for this...

# Deployment pod labels
labels:
  app: grafana-app    # ...but pods have this! ❌ Mismatch
```

**Fix:**
Update service selector to match pod labels:
```yaml
selector:
  app: grafana-app    # Match pod labels ✅
```

Or update deployment labels to match service selector.

### Issue 4: Cannot Access via NodePort

**Symptoms:**
```bash
curl http://<node-ip>:32000
# Connection refused or timeout
```

**Diagnosis Steps:**

**1. Check Service Type:**
```bash
kubectl get svc grafana-service -o jsonpath='{.spec.type}'
# Should output: NodePort
```

**2. Check NodePort Value:**
```bash
kubectl get svc grafana-service -o jsonpath='{.spec.ports[0].nodePort}'
# Should output: 32000
```

**3. Check Pod is Running:**
```bash
kubectl get pods -l app=grafana
# Should show Running status
```

**4. Check Service Endpoints:**
```bash
kubectl get endpoints grafana-service
# Should show pod IP:port
```

**5. Test from Within Cluster:**
```bash
kubectl run test --image=busybox --rm -it --restart=Never -- wget -O- http://grafana-service:3000/api/health
# If this works, issue is external access
```

**Common Solutions:**

**A. Firewall Blocking Port 32000:**
```bash
# On node, check if port is open
sudo netstat -tlnp | grep 32000

# Check firewall rules (Ubuntu/Debian)
sudo ufw status
sudo ufw allow 32000/tcp

# Check firewall rules (CentOS/RHEL)
sudo firewall-cmd --list-ports
sudo firewall-cmd --add-port=32000/tcp --permanent
sudo firewall-cmd --reload
```

**B. kube-proxy Not Running:**
```bash
# Check kube-proxy pods
kubectl get pods -n kube-system | grep kube-proxy

# Should see kube-proxy pods on each node
```

**C. Wrong Node IP:**
```bash
# Get correct node IP
kubectl get nodes -o wide
# Use INTERNAL-IP column
```

### Issue 5: Grafana Login Page Not Loading (Blank Page)

**Symptoms:**
- URL accessible but shows blank page
- Or shows error: "Cannot read property..."

**Diagnosis:**
```bash
# Check Grafana logs
kubectl logs deployment/grafana-deployment-devops

# Look for errors like:
# Error: failed to load static files
# Error: plugin not found
```

**Common Causes:**

**1. Incomplete Container Startup:**
- Grafana still initializing
- Wait 30-60 seconds and refresh

**2. Browser Cache:**
- Clear browser cache
- Open in incognito/private window

**3. Wrong Base URL:**
- Check if accessing via correct URL
- Should be: http://<node-ip>:32000 (not https)

**Fix:**
```bash
# Check if Grafana is actually ready
kubectl logs deployment/grafana-deployment-devops | grep "HTTP Server Listen"

# Should see:
# logger=server msg="HTTP Server Listen" address=[::]:3000

# Test with curl
curl http://<node-ip>:32000/api/health
```

### Issue 6: "Admin User Already Exists" Error

**Symptoms:**
- Can't login with admin/admin
- Error message about admin user

**Cause:**
- Persistent volume retaining old data
- Previous Grafana installation

**Fix:**
```bash
# Delete deployment and recreate
kubectl delete deployment grafana-deployment-devops
kubectl apply -f grafana-deployment-devops.yaml

# If using persistent volume, delete PVC:
kubectl delete pvc grafana-pvc  # If you created one

# Then recreate deployment
```

### Issue 7: Port Already in Use (NodePort 32000)

**Symptoms:**
```bash
kubectl apply -f grafana-service.yaml
# Error: port 32000 is already allocated
```

**Diagnosis:**
```bash
# Check which service is using port 32000
kubectl get svc --all-namespaces | grep 32000
```

**Fix:**

**Option 1: Use Different NodePort**
```yaml
ports:
- port: 3000
  targetPort: 3000
  nodePort: 32001  # Change to 32001 or any free port
```

**Option 2: Delete Conflicting Service**
```bash
kubectl delete svc <conflicting-service-name>
```

**Option 3: Let Kubernetes Assign Port Automatically**
```yaml
ports:
- port: 3000
  targetPort: 3000
  # No nodePort specified → Kubernetes assigns random port
```

```bash
# After applying, check assigned port
kubectl get svc grafana-service
# PORT(S): 3000:31234/TCP  ← Random port assigned
```

## Production Best Practices

### 1. Use Specific Image Versions

**❌ Not Recommended (latest tag):**
```yaml
image: grafana/grafana:latest
# Problems:
# - Unpredictable updates
# - Hard to debug issues
# - Can break on restart
```

**✅ Recommended (specific version):**
```yaml
image: grafana/grafana:10.2.3
# Benefits:
# - Predictable behavior
# - Easy to rollback
# - Consistent across environments
```

### 2. Add Resource Limits

**✅ Always Set Resources:**
```yaml
containers:
- name: grafana
  image: grafana/grafana:10.2.3
  resources:
    requests:
      memory: "128Mi"
      cpu: "100m"
    limits:
      memory: "512Mi"
      cpu: "500m"
```

**Why:**
- Prevents one pod from consuming all resources
- Enables predictable scheduling
- Better cluster utilization

### 3. Use Persistent Storage

**Problem Without Persistence:**
- Pod restarts → All dashboards lost ❌
- Configuration resets ❌
- Data sources disappear ❌

**Solution: PersistentVolume**
```yaml
spec:
  containers:
  - name: grafana
    image: grafana/grafana:10.2.3
    volumeMounts:
    - name: grafana-storage
      mountPath: /var/lib/grafana
  volumes:
  - name: grafana-storage
    persistentVolumeClaim:
      claimName: grafana-pvc
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: grafana-pvc
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

### 4. Add Health Checks

**✅ Liveness and Readiness Probes:**
```yaml
containers:
- name: grafana
  image: grafana/grafana:10.2.3
  livenessProbe:
    httpGet:
      path: /api/health
      port: 3000
    initialDelaySeconds: 30
    periodSeconds: 10
    failureThreshold: 3
  readinessProbe:
    httpGet:
      path: /api/health
      port: 3000
    initialDelaySeconds: 10
    periodSeconds: 5
    failureThreshold: 3
```

**Benefits:**
- Automatic restart of unhealthy pods
- Remove unhealthy pods from service
- Better reliability

### 5. Use ConfigMaps for Configuration

**Externalize Configuration:**
```bash
# Create ConfigMap with grafana.ini
kubectl create configmap grafana-config \
  --from-file=grafana.ini

# Use in deployment
```

```yaml
containers:
- name: grafana
  volumeMounts:
  - name: config
    mountPath: /etc/grafana
volumes:
- name: config
  configMap:
    name: grafana-config
```

### 6. Use Secrets for Passwords

**❌ Don't Hardcode Passwords:**
```yaml
env:
- name: GF_SECURITY_ADMIN_PASSWORD
  value: "admin"  # ❌ Visible in pod spec!
```

**✅ Use Secrets:**
```bash
# Create secret
kubectl create secret generic grafana-secret \
  --from-literal=admin-password=SuperSecretPassword123

# Use in deployment
```

```yaml
env:
- name: GF_SECURITY_ADMIN_PASSWORD
  valueFrom:
    secretKeyRef:
      name: grafana-secret
      key: admin-password
```

### 7. Use Ingress for Production

**Development: NodePort**
```yaml
type: NodePort  # Good for dev/testing
```

**Production: Ingress**
```yaml
# Use Ingress for:
# - Domain-based access (grafana.example.com)
# - SSL/TLS termination
# - Path-based routing
# - Better security
```

**Example Ingress:**
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: grafana-ingress
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
spec:
  tls:
  - hosts:
    - grafana.example.com
    secretName: grafana-tls
  rules:
  - host: grafana.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: grafana-service
            port:
              number: 3000
```

### 8. Enable Monitoring

**Monitor Grafana Itself:**
```yaml
env:
- name: GF_METRICS_ENABLED
  value: "true"
- name: GF_METRICS_BASIC_AUTH_USERNAME
  value: "metrics"
- name: GF_METRICS_BASIC_AUTH_PASSWORD
  valueFrom:
    secretKeyRef:
      name: grafana-secret
      key: metrics-password
```

**Prometheus ServiceMonitor:**
```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: grafana
spec:
  selector:
    matchLabels:
      app: grafana
  endpoints:
  - port: grafana-http
    path: /metrics
```

### 9. Use Multiple Replicas (with Shared Storage)

**High Availability:**
```yaml
spec:
  replicas: 2  # Multiple replicas
```

**Note:** Requires shared storage (ReadWriteMany PV) or external database for session storage.

### 10. Security Hardening

**Security Best Practices:**
```yaml
containers:
- name: grafana
  image: grafana/grafana:10.2.3
  securityContext:
    runAsNonRoot: true
    runAsUser: 472
    readOnlyRootFilesystem: true
    allowPrivilegeEscalation: false
    capabilities:
      drop:
      - ALL
  env:
  - name: GF_SECURITY_ADMIN_PASSWORD
    valueFrom:
      secretKeyRef:
        name: grafana-secret
        key: admin-password
  - name: GF_SERVER_ROOT_URL
    value: "https://grafana.example.com"
  - name: GF_SECURITY_COOKIE_SECURE
    value: "true"
  - name: GF_SECURITY_STRICT_TRANSPORT_SECURITY
    value: "true"
```

## Completion Checklist

Verify your setup with these commands:

```bash
# 1. Deployment exists with correct name
kubectl get deployment grafana-deployment-devops
# NAME: grafana-deployment-devops ✅

# 2. Deployment has 1 replica ready
kubectl get deployment grafana-deployment-devops -o jsonpath='{.status.readyReplicas}'
# Output: 1 ✅

# 3. Pod is running
kubectl get pods -l app=grafana
# STATUS: Running ✅

# 4. Pod uses Grafana image
kubectl get deployment grafana-deployment-devops -o jsonpath='{.spec.template.spec.containers[0].image}'
# Output: grafana/grafana:latest (or any grafana image) ✅

# 5. Service exists
kubectl get service grafana-service
# NAME: grafana-service ✅

# 6. Service type is NodePort
kubectl get service grafana-service -o jsonpath='{.spec.type}'
# Output: NodePort ✅

# 7. Service has correct NodePort
kubectl get service grafana-service -o jsonpath='{.spec.ports[0].nodePort}'
# Output: 32000 ✅

# 8. Service selector matches pod labels
kubectl get service grafana-service -o jsonpath='{.spec.selector}'
# Output: {"app":"grafana"} ✅

# 9. Service has endpoint
kubectl get endpoints grafana-service
# Shows 1 IP:port ✅

# 10. Grafana health check works
NODE_IP=$(kubectl get nodes -o jsonpath='{.items[0].status.addresses[?(@.type=="InternalIP")].address}')
curl -s http://$NODE_IP:32000/api/health | grep -q "ok"
# Exit code 0 ✅

# 11. Grafana login page accessible
curl -I http://$NODE_IP:32000/login | grep "200 OK"
# Shows HTTP/1.1 200 OK ✅

# 12. Can access via browser
# Open: http://<node-ip>:32000
# Shows Grafana login page ✅

# 13. Can login with default credentials
# Username: admin, Password: admin
# Successfully logs in ✅

# 14. Grafana dashboard loads
# After login, see Grafana home page ✅
```

**All checks passing = Task complete! ✅**

## Summary

### What We Accomplished
1. ✅ Deployed Grafana using Kubernetes Deployment
2. ✅ Used grafana/grafana:latest image
3. ✅ Named deployment: grafana-deployment-devops
4. ✅ Created NodePort service with port 32000
5. ✅ Successfully accessed Grafana login page
6. ✅ Verified self-healing (pod recreation)
7. ✅ Demonstrated service discovery
8. ✅ Logged in and verified functionality

### Key Concepts Learned

**Grafana:**
- Open-source monitoring and analytics platform
- Default port: 3000
- Default credentials: admin/admin
- Used for dashboards, alerts, and data visualization
- Connects to various data sources (Prometheus, InfluxDB, etc.)

**Kubernetes Deployment:**
- Manages application lifecycle
- Self-healing (automatic pod recreation)
- Easy scaling and updates
- Declarative configuration
- ReplicaSet management

**NodePort Service:**
- External access via node IPs
- Port range: 30000-32767
- Works on any node in cluster
- Good for dev/testing/on-premises
- Simpler than LoadBalancer/Ingress

**Application Deployment Pattern:**
```
Deployment
└── ReplicaSet
    └── Pod
        └── Container (Grafana)

Service (NodePort 32000)
└── Routes to Pod (port 3000)
```

### Critical Commands
```bash
# Create deployment
kubectl apply -f grafana-deployment-devops.yaml

# Create service
kubectl apply -f grafana-service.yaml

# Verify deployment
kubectl get deployments
kubectl get pods
kubectl get svc

# Access Grafana
NODE_IP=$(kubectl get nodes -o jsonpath='{.items[0].status.addresses[?(@.type=="InternalIP")].address}')
echo "Grafana URL: http://$NODE_IP:32000"

# Test health
curl http://$NODE_IP:32000/api/health

# View logs
kubectl logs deployment/grafana-deployment-devops

# Delete resources
kubectl delete deployment grafana-deployment-devops
kubectl delete service grafana-service
```

### Real-World Applications

**Infrastructure Monitoring:**
```
Grafana + Prometheus:
├── Monitor Kubernetes cluster
├── Track pod CPU/memory usage
├── Alert on high resource usage
└── Visualize cluster health
```

**Application Performance:**
```
Grafana + Application Metrics:
├── API response times
├── Request rates
├── Error rates
└── User activity
```

**Business Dashboards:**
```
Grafana + Database:
├── Sales metrics
├── User signups
├── Revenue tracking
└── KPI visualization
```

### Best Practices Applied
1. ✅ Used Deployment (not bare Pod)
2. ✅ Clear, descriptive resource names
3. ✅ Label-based service discovery
4. ✅ Named ports for better documentation
5. ✅ NodePort for external access
6. ✅ Verified self-healing behavior
7. ✅ Checked service endpoints
8. ✅ Tested from multiple methods

### What's Next?

**Day 59+:** Continue with Kubernetes applications:
- **ConfigMaps:** External configuration files
- **Secrets:** Sensitive data (passwords, certificates)
- **Persistent Volumes:** Data persistence for Grafana
- **StatefulSets:** Stateful applications (databases)
- **Ingress:** Domain-based routing (grafana.example.com)
- **Helm Charts:** Package management (install Grafana with Helm)
- **Monitoring Stack:** Grafana + Prometheus + AlertManager

**Learning Path:**
```
Day 58: Grafana Deployment ← YOU ARE HERE
    ↓
Day 59+: ConfigMaps & Secrets
    ↓
Day 60+: Persistent Volumes (persist Grafana data)
    ↓
Day 61+: StatefulSets
    ↓
Day 62+: Ingress Controllers
    ↓
Day 63+: Helm (deploy complete monitoring stack)
```

### Production Grafana Stack

**Complete Monitoring Setup:**
```
Grafana (Visualization)
    ↑
Prometheus (Metrics Collection)
    ↑
Node Exporter (System Metrics)
    ↑
Application Exporters (App Metrics)
```

**Next Steps:**
1. Add Persistent Volume for Grafana data
2. Deploy Prometheus as data source
3. Configure Grafana dashboards
4. Set up alerting rules
5. Add SSL/TLS with Ingress
6. Implement backup strategy

🎉 **Day 58 Complete!** You've successfully deployed Grafana on Kubernetes with external access—a crucial skill for building monitoring and observability platforms in production environments!
