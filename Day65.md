# Day 65: Redis Deployment with ConfigMap and Volumes in Kubernetes

## 📋 Task Overview

**Scenario:** The Nautilus application development team observed performance issues with one of their applications deployed in the Kubernetes cluster. After analyzing various factors, the team decided to implement an in-memory caching utility for the database service. They chose Redis for this purpose. Initially, Redis will be deployed on the Kubernetes cluster for testing before moving to production.

**Given Requirements:**
- ConfigMap: `my-redis-config` with `maxmemory 2mb`
- Deployment: `redis-deployment`
- Image: `redis:alpine`
- Container name: `redis-container`
- Replicas: 1
- CPU request: 1 CPU
- Empty directory volume: `data` at `/redis-master-data`
- ConfigMap volume: `redis-config` at `/redis-master`
- Exposed port: 6379

**Your Mission:**
1. Create ConfigMap with Redis configuration
2. Create Redis deployment with proper resource requests
3. Mount EmptyDir and ConfigMap volumes
4. Expose Redis port 6379
5. Verify deployment is running successfully

---

## 🎯 Learning Objectives

By the end of this task, you will understand:
- **Redis Basics:** In-memory caching and data store
- **ConfigMaps:** Storing configuration data in Kubernetes
- **Volume Mounts:** EmptyDir and ConfigMap volumes
- **Resource Requests:** CPU and memory allocation
- **Redis Configuration:** Setting memory limits and policies
- **Deployment Best Practices:** Single replica Redis for testing

---

## 📖 Understanding Redis

### What is Redis?

**Redis** (Remote Dictionary Server) is an open-source, in-memory data structure store used as:
- **Cache:** Fast data access layer
- **Database:** Key-value store
- **Message Broker:** Pub/sub messaging
- **Session Store:** User session management

**Key Features:**
- In-memory storage (extremely fast)
- Data persistence options
- Multiple data structures (strings, hashes, lists, sets)
- Atomic operations
- Replication and clustering support

### Why Use Redis?

**Performance Benefits:**
```
Traditional Database Query: 10-100ms
Redis Cache Query:          <1ms (100x faster!)
```

**Common Use Cases:**
1. **Caching:** Store frequently accessed data
2. **Session Management:** Store user sessions
3. **Real-time Analytics:** Count page views, track metrics
4. **Message Queues:** Pub/sub for microservices
5. **Leaderboards:** Sorted sets for rankings

---

## 🔍 Understanding Kubernetes Components

### 1. ConfigMap

**Purpose:** Store non-sensitive configuration data as key-value pairs

**Benefits:**
- Decouple configuration from container images
- Update configuration without rebuilding images
- Share configuration across multiple pods
- Mount as files or environment variables

**Example Use Cases:**
- Application config files
- Database connection strings
- Feature flags
- Redis configuration parameters

---

### 2. Volume Types

#### EmptyDir Volume

**What it is:** Temporary storage created when pod starts, deleted when pod terminates

**Characteristics:**
- Starts empty (hence the name)
- Shared between containers in same pod
- Stored on node's disk or memory
- Useful for scratch space, caching, checkpoints

**Use Cases:**
- Cache that can be regenerated
- Temporary processing space
- Shared data between containers in pod

---

#### ConfigMap Volume

**What it is:** Mounts ConfigMap data as files in containers

**Characteristics:**
- Each key becomes a file
- File content is the value
- Updates automatically (with propagation delay)
- Read-only by default

**Use Cases:**
- Configuration files (redis.conf, nginx.conf)
- Environment-specific settings
- Application properties

---

### 3. Resource Requests

**Purpose:** Guarantee minimum resources for container

**Requests vs Limits:**
```yaml
resources:
  requests:    # Minimum guaranteed resources
    cpu: "1"
    memory: "256Mi"
  limits:      # Maximum allowed resources
    cpu: "2"
    memory: "512Mi"
```

**CPU Units:**
- `1` or `1000m` = 1 full CPU core
- `500m` = 0.5 CPU core (half core)
- `100m` = 0.1 CPU core (10% of core)

---

## 🛠️ Task Implementation

### Step 1: Verify Cluster Access

**Check kubectl is configured:**
```bash
kubectl cluster-info
```

**Expected output:**
```
Kubernetes control plane is running at https://...
CoreDNS is running at https://...
```

**Check current namespace:**
```bash
kubectl config view --minify | grep namespace
```

**List existing resources:**
```bash
kubectl get all
kubectl get configmap
```

---

### Step 2: Create Redis ConfigMap

**Understanding the Configuration:**

Redis `maxmemory` directive controls maximum memory usage:
- `maxmemory 2mb` = Limit Redis to 2 megabytes
- When limit reached, Redis applies eviction policy
- Critical for preventing memory exhaustion

**Create the ConfigMap:**

**Method 1: Using kubectl create (Recommended)**
```bash
kubectl create configmap my-redis-config \
  --from-literal=maxmemory=2mb
```

**Method 2: Using YAML file**
```bash
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-redis-config
data:
  maxmemory: "2mb"
EOF
```

**Expected output:**
```
configmap/my-redis-config created
```

---

### Step 3: Verify ConfigMap

**View ConfigMap:**
```bash
kubectl get configmap my-redis-config
```

**Expected output:**
```
NAME              DATA   AGE
my-redis-config   1      5s
```

**Describe ConfigMap (see details):**
```bash
kubectl describe configmap my-redis-config
```

**Expected output:**
```yaml
Name:         my-redis-config
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
maxmemory:
----
2mb

Events:  <none>
```

**View ConfigMap YAML:**
```bash
kubectl get configmap my-redis-config -o yaml
```

---

### Step 4: Create Redis Deployment YAML

**Create the deployment file:**
```bash
cat > redis-deployment.yaml <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - name: redis-container
        image: redis:alpine
        ports:
        - containerPort: 6379
        resources:
          requests:
            cpu: "1"
        volumeMounts:
        - name: data
          mountPath: /redis-master-data
        - name: redis-config
          mountPath: /redis-master
      volumes:
      - name: data
        emptyDir: {}
      - name: redis-config
        configMap:
          name: my-redis-config
EOF
```

---

### Step 5: Understanding the Deployment Configuration

**Let's break down each section:**

#### 1. Metadata and Replicas
```yaml
metadata:
  name: redis-deployment  # Deployment name
spec:
  replicas: 1  # Single instance for testing
```

#### 2. Selector and Labels
```yaml
selector:
  matchLabels:
    app: redis  # Deployment manages pods with this label
template:
  metadata:
    labels:
      app: redis  # Pod label (must match selector)
```

#### 3. Container Specification
```yaml
containers:
- name: redis-container  # Container name
  image: redis:alpine    # Lightweight Redis image
  ports:
  - containerPort: 6379  # Redis default port
```

#### 4. Resource Requests
```yaml
resources:
  requests:
    cpu: "1"  # Request 1 full CPU core
```

**Why CPU request matters:**
- Scheduler ensures node has 1 CPU available
- Pod guaranteed 1 CPU core minimum
- Important for performance-critical Redis

#### 5. Volume Mounts (Container perspective)
```yaml
volumeMounts:
- name: data
  mountPath: /redis-master-data  # EmptyDir for data
- name: redis-config
  mountPath: /redis-master       # ConfigMap for config
```

#### 6. Volumes (Pod perspective)
```yaml
volumes:
- name: data
  emptyDir: {}  # Temporary storage
- name: redis-config
  configMap:
    name: my-redis-config  # Reference to ConfigMap
```

---

### Step 6: Apply the Deployment

**Deploy Redis:**
```bash
kubectl apply -f redis-deployment.yaml
```

**Expected output:**
```
deployment.apps/redis-deployment created
```

---

### Step 7: Verify Deployment

**Check deployment status:**
```bash
kubectl get deployment redis-deployment
```

**Expected output:**
```
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
redis-deployment   1/1     1            1           30s
```

**Key indicators:**
- `READY: 1/1` ✅ - One pod ready out of one desired
- `UP-TO-DATE: 1` ✅ - Deployment updated
- `AVAILABLE: 1` ✅ - One pod available

---

### Step 8: Check Pod Status

**List pods:**
```bash
kubectl get pods -l app=redis
```

**Expected output:**
```
NAME                                READY   STATUS    RESTARTS   AGE
redis-deployment-7f8d9c5b4d-xyz12   1/1     Running   0          45s
```

**Describe pod for detailed info:**
```bash
POD_NAME=$(kubectl get pods -l app=redis -o jsonpath='{.items[0].metadata.name}')
kubectl describe pod $POD_NAME
```

**Check for these key sections:**
```yaml
Name:         redis-deployment-7f8d9c5b4d-xyz12
Status:       Running
IP:           10.244.1.5
Containers:
  redis-container:
    Image:          redis:alpine
    Port:           6379/TCP
    State:          Running
    Ready:          True
    Requests:
      cpu:        1
    Mounts:
      /redis-master from redis-config (rw)
      /redis-master-data from data (rw)

Volumes:
  data:
    Type:       EmptyDir
  redis-config:
    Type:       ConfigMap
    Name:       my-redis-config

Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
```

**All should show ✅ True**

---

### Step 9: Verify Redis is Running

**Check Redis logs:**
```bash
kubectl logs $POD_NAME
```

**Expected output (Redis startup logs):**
```
1:C 09 Jan 2026 10:00:00.123 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
1:C 09 Jan 2026 10:00:00.123 # Redis version=7.0.x, bits=64, commit=00000000, modified=0
1:C 09 Jan 2026 10:00:00.123 # Configuration loaded
1:M 09 Jan 2026 10:00:00.124 * monotonic clock: POSIX clock_gettime
1:M 09 Jan 2026 10:00:00.124 * Running mode=standalone, port=6379
1:M 09 Jan 2026 10:00:00.124 # Server initialized
1:M 09 Jan 2026 10:00:00.125 * Ready to accept connections
```

**Key indicators:**
- ✅ "Redis is starting"
- ✅ "Running mode=standalone, port=6379"
- ✅ "Ready to accept connections"

---

### Step 10: Verify Volume Mounts

**Check volumes inside container:**
```bash
# Check EmptyDir volume
kubectl exec $POD_NAME -- ls -la /redis-master-data

# Check ConfigMap volume
kubectl exec $POD_NAME -- ls -la /redis-master

# View ConfigMap content
kubectl exec $POD_NAME -- cat /redis-master/maxmemory
```

**Expected outputs:**
```bash
# EmptyDir (initially empty)
total 8
drwxrwxrwx    2 redis    redis         4096 Jan  9 10:00 .
drwxr-xr-x    1 root     root          4096 Jan  9 10:00 ..

# ConfigMap volume
total 12
drwxrwxrwx    3 root     root          4096 Jan  9 10:00 .
drwxr-xr-x    1 root     root          4096 Jan  9 10:00 ..
drwxr-xr-x    2 root     root          4096 Jan  9 10:00 ..2026_01_09_10_00_00.123456
lrwxrwxrwx    1 root     root            31 Jan  9 10:00 ..data -> ..2026_01_09_10_00_00.123456
lrwxrwxrwx    1 root     root            16 Jan  9 10:00 maxmemory -> ..data/maxmemory

# ConfigMap content
2mb
```

---

### Step 11: Test Redis Connection

**Connect to Redis using redis-cli:**
```bash
kubectl exec -it $POD_NAME -- redis-cli
```

**Inside Redis CLI, test commands:**
```redis
# Test connection
PING

# Expected: PONG

# Check Redis configuration
CONFIG GET maxmemory

# Expected:
# 1) "maxmemory"
# 2) "0"  (Note: maxmemory from ConfigMap not automatically applied)

# Set a test key
SET testkey "Hello from Kubernetes Redis!"

# Get the key
GET testkey

# Expected: "Hello from Kubernetes Redis!"

# Check info
INFO memory

# Exit
exit
```

---

### Step 12: Apply Redis Configuration (Optional)

**Note:** The ConfigMap creates a file but doesn't automatically apply it. To use the config:

**Method 1: Use Redis config file**

**Update deployment to use config file:**
```bash
cat > redis-deployment-with-config.yaml <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - name: redis-container
        image: redis:alpine
        command:
          - redis-server
          - "--maxmemory"
          - "2mb"
        ports:
        - containerPort: 6379
        resources:
          requests:
            cpu: "1"
        volumeMounts:
        - name: data
          mountPath: /redis-master-data
        - name: redis-config
          mountPath: /redis-master
      volumes:
      - name: data
        emptyDir: {}
      - name: redis-config
        configMap:
          name: my-redis-config
EOF

kubectl apply -f redis-deployment-with-config.yaml
```

**Method 2: Set dynamically**
```bash
kubectl exec $POD_NAME -- redis-cli CONFIG SET maxmemory 2mb
```

---

### Step 13: Verify Resource Requests

**Check node resource allocation:**
```bash
kubectl describe node | grep -A 5 "Allocated resources"
```

**Expected output:**
```
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  Resource           Requests      Limits
  --------           --------      ------
  cpu                1050m (52%)   0 (0%)
  memory             256Mi (6%)    512Mi (13%)
```

**Check pod resources:**
```bash
kubectl get pod $POD_NAME -o jsonpath='{.spec.containers[0].resources}' | jq
```

**Expected output:**
```json
{
  "requests": {
    "cpu": "1"
  }
}
```

---

### Step 14: Complete Verification

**Run comprehensive check:**
```bash
echo "=== Deployment Status ==="
kubectl get deployment redis-deployment

echo -e "\n=== Pod Status ==="
kubectl get pods -l app=redis

echo -e "\n=== ConfigMap ==="
kubectl get configmap my-redis-config

echo -e "\n=== Pod Details ==="
kubectl describe pod $POD_NAME | grep -A 10 "Volumes:"

echo -e "\n=== Redis Connection Test ==="
kubectl exec $POD_NAME -- redis-cli PING

echo -e "\n✅ All checks completed!"
```

---

## 📊 Complete Setup Script

**Save and run this script for automated deployment:**

```bash
#!/bin/bash

echo "🚀 Day 65: Redis Deployment with ConfigMap and Volumes"
echo "======================================================"

# Colors
GREEN='\033[0;32m'
RED='\033[0;31m'
YELLOW='\033[1;33m'
NC='\033[0m'

# Step 1: Create ConfigMap
echo -e "\n${YELLOW}1️⃣ Creating ConfigMap...${NC}"
kubectl create configmap my-redis-config \
  --from-literal=maxmemory=2mb \
  --dry-run=client -o yaml | kubectl apply -f -

if [ $? -eq 0 ]; then
    echo -e "${GREEN}✅ ConfigMap created${NC}"
else
    echo -e "${RED}❌ Failed to create ConfigMap${NC}"
    exit 1
fi

# Step 2: Verify ConfigMap
echo -e "\n${YELLOW}2️⃣ Verifying ConfigMap...${NC}"
kubectl get configmap my-redis-config
kubectl describe configmap my-redis-config | grep -A 3 "Data"

# Step 3: Create Deployment
echo -e "\n${YELLOW}3️⃣ Creating Redis Deployment...${NC}"
cat <<EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - name: redis-container
        image: redis:alpine
        command:
          - redis-server
          - "--maxmemory"
          - "2mb"
        ports:
        - containerPort: 6379
        resources:
          requests:
            cpu: "1"
        volumeMounts:
        - name: data
          mountPath: /redis-master-data
        - name: redis-config
          mountPath: /redis-master
      volumes:
      - name: data
        emptyDir: {}
      - name: redis-config
        configMap:
          name: my-redis-config
EOF

if [ $? -eq 0 ]; then
    echo -e "${GREEN}✅ Deployment created${NC}"
else
    echo -e "${RED}❌ Failed to create Deployment${NC}"
    exit 1
fi

# Step 4: Wait for deployment
echo -e "\n${YELLOW}4️⃣ Waiting for deployment to be ready...${NC}"
kubectl rollout status deployment/redis-deployment --timeout=120s

if [ $? -eq 0 ]; then
    echo -e "${GREEN}✅ Deployment ready${NC}"
else
    echo -e "${RED}❌ Deployment failed to become ready${NC}"
    exit 1
fi

# Step 5: Get pod name
POD_NAME=$(kubectl get pods -l app=redis -o jsonpath='{.items[0].metadata.name}')
echo -e "\n${YELLOW}Pod Name: ${POD_NAME}${NC}"

# Step 6: Check deployment
echo -e "\n${YELLOW}5️⃣ Checking Deployment Status...${NC}"
READY=$(kubectl get deployment redis-deployment -o jsonpath='{.status.readyReplicas}')
DESIRED=$(kubectl get deployment redis-deployment -o jsonpath='{.spec.replicas}')

if [ "$READY" == "$DESIRED" ]; then
    echo -e "${GREEN}✅ Deployment: $READY/$DESIRED ready${NC}"
else
    echo -e "${RED}❌ Deployment: $READY/$DESIRED ready${NC}"
fi

# Step 7: Check pod
echo -e "\n${YELLOW}6️⃣ Checking Pod Status...${NC}"
POD_STATUS=$(kubectl get pod $POD_NAME -o jsonpath='{.status.phase}')
POD_READY=$(kubectl get pod $POD_NAME -o jsonpath='{.status.containerStatuses[0].ready}')

if [ "$POD_STATUS" == "Running" ] && [ "$POD_READY" == "true" ]; then
    echo -e "${GREEN}✅ Pod: Running and Ready${NC}"
else
    echo -e "${RED}❌ Pod: Status=$POD_STATUS, Ready=$POD_READY${NC}"
fi

# Step 8: Check volumes
echo -e "\n${YELLOW}7️⃣ Checking Volume Mounts...${NC}"
echo "EmptyDir volume at /redis-master-data:"
kubectl exec $POD_NAME -- ls -la /redis-master-data

echo -e "\nConfigMap volume at /redis-master:"
kubectl exec $POD_NAME -- ls -l /redis-master/maxmemory
kubectl exec $POD_NAME -- cat /redis-master/maxmemory

# Step 9: Test Redis
echo -e "\n${YELLOW}8️⃣ Testing Redis Connection...${NC}"
PING_RESULT=$(kubectl exec $POD_NAME -- redis-cli PING 2>/dev/null)

if [ "$PING_RESULT" == "PONG" ]; then
    echo -e "${GREEN}✅ Redis responding: $PING_RESULT${NC}"
else
    echo -e "${RED}❌ Redis not responding${NC}"
fi

# Step 10: Test Redis commands
echo -e "\n${YELLOW}9️⃣ Testing Redis Commands...${NC}"
kubectl exec $POD_NAME -- redis-cli SET testkey "Hello Kubernetes" >/dev/null
TEST_VALUE=$(kubectl exec $POD_NAME -- redis-cli GET testkey)
echo -e "SET/GET test: ${GREEN}$TEST_VALUE${NC}"

# Step 11: Check maxmemory config
echo -e "\n${YELLOW}🔟 Checking Redis Configuration...${NC}"
MAX_MEM=$(kubectl exec $POD_NAME -- redis-cli CONFIG GET maxmemory | tail -1)
echo -e "Maxmemory configured: ${GREEN}$MAX_MEM bytes (2MB)${NC}"

# Step 12: Check resource requests
echo -e "\n${YELLOW}1️⃣1️⃣ Checking Resource Requests...${NC}"
CPU_REQUEST=$(kubectl get pod $POD_NAME -o jsonpath='{.spec.containers[0].resources.requests.cpu}')
echo -e "CPU Request: ${GREEN}$CPU_REQUEST${NC}"

# Final summary
echo -e "\n${YELLOW}======================================================"
echo "📊 Deployment Summary"
echo "======================================================${NC}"
kubectl get deployment,pods,configmap | grep -E "redis|NAME"

echo -e "\n${GREEN}✅ Redis deployment completed successfully!${NC}"
echo -e "${GREEN}✅ ConfigMap: my-redis-config with maxmemory=2mb${NC}"
echo -e "${GREEN}✅ Deployment: redis-deployment with 1 replica${NC}"
echo -e "${GREEN}✅ Container: redis-container with redis:alpine image${NC}"
echo -e "${GREEN}✅ CPU Request: 1 CPU${NC}"
echo -e "${GREEN}✅ Volumes: EmptyDir (data) and ConfigMap (redis-config)${NC}"
echo -e "${GREEN}✅ Port: 6379 exposed${NC}"
echo -e "${GREEN}✅ Status: Up and running${NC}"
```

**Save and run:**
```bash
chmod +x redis-deploy.sh
./redis-deploy.sh
```

---

## 🧹 Cleanup

**Delete all resources:**
```bash
# Delete deployment
kubectl delete deployment redis-deployment

# Delete ConfigMap
kubectl delete configmap my-redis-config

# Verify deletion
kubectl get deployment,pods,configmap | grep redis
# No resources found

# If you created files
rm -f redis-deployment.yaml redis-deployment-with-config.yaml redis-deploy.sh
```

---

## 🐛 Common Issues and Solutions

### Issue 1: Pod Stuck in Pending

**Symptoms:**
```bash
kubectl get pods
# NAME                                READY   STATUS    RESTARTS   AGE
# redis-deployment-xxx                0/1     Pending   0          2m
```

**Diagnosis:**
```bash
kubectl describe pod $POD_NAME | grep -A 5 Events
# Warning  FailedScheduling  Insufficient cpu
```

**Root Cause:** Not enough CPU available on nodes (requested 1 CPU)

**Solution:**
```bash
# Check node resources
kubectl describe nodes | grep -A 5 "Allocated resources"

# Option 1: Reduce CPU request
kubectl edit deployment redis-deployment
# Change: cpu: "1" → cpu: "500m"

# Option 2: Add more nodes (if cloud/multi-node cluster)
```

---

### Issue 2: ConfigMap Not Found

**Symptoms:**
```bash
kubectl get pods
# redis-deployment-xxx    0/1     CreateContainerConfigError   0          30s
```

**Diagnosis:**
```bash
kubectl describe pod $POD_NAME
# Warning  Failed  Error: configmap "my-redis-config" not found
```

**Solution:**
```bash
# Create the ConfigMap first
kubectl create configmap my-redis-config --from-literal=maxmemory=2mb

# Deployment will automatically recover
kubectl get pods -w
```

---

### Issue 3: Volume Mount Permission Issues

**Symptoms:**
```bash
kubectl logs $POD_NAME
# Error: Can't open or create append only file: Permission denied
```

**Solution:**
```bash
# EmptyDir volumes have correct permissions by default
# If issues persist, check SELinux/AppArmor settings

# Verify volume mounts
kubectl describe pod $POD_NAME | grep -A 10 "Mounts:"
```

---

### Issue 4: Redis Not Applying ConfigMap Values

**Symptoms:**
```bash
kubectl exec $POD_NAME -- redis-cli CONFIG GET maxmemory
# Returns "0" instead of "2mb"
```

**Root Cause:** ConfigMap creates file but doesn't auto-apply to Redis

**Solution:**

**Option 1: Use command in deployment**
```yaml
command:
  - redis-server
  - "--maxmemory"
  - "2mb"
```

**Option 2: Create redis.conf file**
```bash
kubectl create configmap my-redis-config \
  --from-literal=redis.conf="maxmemory 2mb"

# Update deployment to use config file
command:
  - redis-server
  - /redis-master/redis.conf
```

---

### Issue 5: Image Pull Errors

**Symptoms:**
```bash
kubectl get pods
# redis-deployment-xxx    0/1     ImagePullBackOff   0          2m
```

**Diagnosis:**
```bash
kubectl describe pod $POD_NAME
# Failed to pull image "redis:alpine": image not found
```

**Solution:**
```bash
# Check image name spelling
kubectl edit deployment redis-deployment
# Ensure: image: redis:alpine (correct)

# Test image pull manually
docker pull redis:alpine
```

---

## 📖 Key Takeaways

### Concepts Mastered Today:

1. ✅ **Redis Deployment**
   - Deployed Redis with alpine image (lightweight)
   - Configured single replica for testing
   - Exposed Redis port 6379
   - Applied memory limits via ConfigMap

2. ✅ **ConfigMap Usage**
   - Created ConfigMap with literal values
   - Mounted ConfigMap as volume
   - Accessed ConfigMap data as files
   - Used ConfigMap for application configuration

3. ✅ **Volume Types**
   - **EmptyDir:** Temporary storage for Redis data
   - **ConfigMap Volume:** Configuration files
   - Mounted multiple volumes in single pod
   - Understanding volume lifecycle

4. ✅ **Resource Requests**
   - Requested 1 CPU for container
   - Understanding CPU units (cores vs millicores)
   - Resource requests vs limits
   - Impact on pod scheduling

5. ✅ **Volume Mounts**
   - Mounted EmptyDir at `/redis-master-data`
   - Mounted ConfigMap at `/redis-master`
   - Verified mounts inside container
   - Accessed configuration files

6. ✅ **Testing and Verification**
   - Used redis-cli for testing
   - Verified PING/PONG connection
   - Tested SET/GET commands
   - Checked configuration values

---

## 🎯 Real-World Lessons

**Why This Deployment Pattern Matters:**

1. **Configuration Management**
   - ConfigMaps separate config from code
   - Easy to update without rebuilding images
   - Share configuration across environments

2. **Resource Management**
   - CPU requests ensure performance
   - Prevents resource starvation
   - Scheduler makes informed decisions

3. **Storage Strategy**
   - EmptyDir for ephemeral cache data
   - ConfigMap for configuration
   - Different volume types for different needs

4. **Testing Before Production**
   - Single replica for development/testing
   - Later scale to Redis Cluster for production
   - Add persistence (PersistentVolumes) for production

**Production Considerations:**
```yaml
# Development (current)
- Single replica
- EmptyDir storage (data lost on restart)
- No persistence
- Basic configuration

# Production (future)
- Redis Cluster (3+ replicas)
- PersistentVolume storage
- Persistence enabled (RDB/AOF)
- Resource limits set
- Monitoring and alerts
- Backup strategy
```

**Next Steps for Production:**
1. Add PersistentVolume for data persistence
2. Configure Redis replication
3. Set up Redis Cluster for high availability
4. Add monitoring (Prometheus/Grafana)
5. Configure backup and restore
6. Set resource limits (not just requests)
7. Add liveness and readiness probes

---

## 📚 Additional Resources

**Official Documentation:**
- [Redis Documentation](https://redis.io/documentation)
- [Kubernetes ConfigMaps](https://kubernetes.io/docs/concepts/configuration/configmap/)
- [Kubernetes Volumes](https://kubernetes.io/docs/concepts/storage/volumes/)
- [Resource Management](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/)

**Redis Configuration:**
- [Redis Configuration](https://redis.io/docs/management/config/)
- [Redis Memory Optimization](https://redis.io/docs/management/optimization/memory-optimization/)
- [Redis Persistence](https://redis.io/docs/management/persistence/)

**Best Practices:**
- [Redis Best Practices](https://redis.io/docs/management/optimization/)
- [Running Redis on Kubernetes](https://kubernetes.io/docs/tutorials/configuration/configure-redis-using-configmap/)

**Next Steps:**
- **Day 66:** Redis Service and Testing
- **Day 67:** StatefulSets for Redis Cluster
- **Day 68:** Persistent Storage for Redis
- **Day 69:** Redis Monitoring and Metrics

---

## ✅ Task Completion Checklist

- [ ] Created ConfigMap `my-redis-config` with `maxmemory=2mb`
- [ ] Verified ConfigMap creation
- [ ] Created deployment `redis-deployment`
- [ ] Used image `redis:alpine`
- [ ] Set container name to `redis-container`
- [ ] Configured 1 replica
- [ ] Set CPU request to 1
- [ ] Mounted EmptyDir volume `data` at `/redis-master-data`
- [ ] Mounted ConfigMap volume `redis-config` at `/redis-master`
- [ ] Exposed port 6379
- [ ] Verified deployment is Running
- [ ] Tested Redis with PING command
- [ ] Verified volume mounts inside container
- [ ] Checked resource allocation
- [ ] Documented deployment configuration

---

**🎉 Congratulations!** You've successfully deployed Redis with ConfigMap and multiple volume mounts! You now understand how to use ConfigMaps for configuration management, mount different volume types, and set resource requests for optimal scheduling.

**Day 65 Status:** ✅ Complete

**Next:** Day 66 - Creating Services for Redis and External Access 🚀
