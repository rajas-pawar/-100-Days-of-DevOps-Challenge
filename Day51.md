# Day 51: Kubernetes Rolling Updates - Updating Deployment Images

## Objective
Execute a rolling update for the `nginx-deployment` application on the Kubernetes cluster, updating from the current nginx image to `nginx:1.17` to deploy recent changes made by the Nautilus application development team. Ensure zero downtime and verify all pods are operational post-update.

## Task Requirements
- **Environment:** Kubernetes cluster accessible from jump_host
- **Deployment Name:** nginx-deployment
- **Current Image:** nginx (existing version)
- **New Image:** nginx:1.17
- **Update Strategy:** Rolling update (zero downtime)
- **Success Criteria:** All pods running with new image version
- **Tool:** kubectl (pre-configured on jump_host)

## Understanding Rolling Updates

### What is a Rolling Update?
A **rolling update** is a deployment strategy that gradually replaces old pods with new pods, ensuring the application remains available throughout the update process. Kubernetes updates pods incrementally, maintaining service continuity.

**Key Characteristics:**
- **Zero Downtime:** Application remains available during updates
- **Gradual Rollout:** Pods updated in controlled batches
- **Automatic Health Checks:** New pods must pass readiness checks
- **Rollback Capability:** Can revert to previous version if issues occur
- **Version Control:** Maintains revision history for all deployments

### Rolling Update vs Other Strategies

| Strategy | Downtime | Resource Usage | Rollback Speed | Use Case |
|----------|----------|----------------|----------------|----------|
| **Rolling Update** | None | Moderate (old + new pods) | Fast | Production apps (default) |
| **Recreate** | Yes (brief) | Low (terminate then create) | N/A | Development, stateful apps |
| **Blue-Green** | None | High (2x resources) | Instant | Critical apps, testing |
| **Canary** | None | Moderate | Fast | Risk-averse deployments |

### Why Rolling Updates Matter

**Benefits:**
1. **Zero Downtime:** Users unaffected during deployment
2. **Safe Updates:** Gradual rollout catches issues early
3. **Easy Rollback:** Quick revert if problems detected
4. **Production Standard:** Industry best practice for updates
5. **Declarative:** Version-controlled configuration

**Use Cases:**
- Deploying new application versions
- Updating container images (security patches, bug fixes)
- Changing container configurations
- Scaling application resources
- Rolling back problematic deployments

## Rolling Update Process

### How It Works

```
Initial State: 3 Pods running nginx:latest
┌─────────────────────────────────────────┐
│  Pod 1 (old)  Pod 2 (old)  Pod 3 (old)  │
│  nginx:latest nginx:latest nginx:latest │
└─────────────────────────────────────────┘

Step 1: Create new pod, terminate one old pod
┌─────────────────────────────────────────┐
│  Pod 2 (old)  Pod 3 (old)  Pod 4 (new)  │
│  nginx:latest nginx:latest nginx:1.17   │
└─────────────────────────────────────────┘

Step 2: Create new pod, terminate one old pod
┌─────────────────────────────────────────┐
│  Pod 3 (old)  Pod 4 (new)  Pod 5 (new)  │
│  nginx:latest nginx:1.17   nginx:1.17   │
└─────────────────────────────────────────┘

Step 3: Create new pod, terminate last old pod
┌─────────────────────────────────────────┐
│  Pod 4 (new)  Pod 5 (new)  Pod 6 (new)  │
│  nginx:1.17   nginx:1.17   nginx:1.17   │
└─────────────────────────────────────────┘

Final State: All pods running nginx:1.17
```

### Update Strategy Parameters

Kubernetes Deployments support two key parameters for controlling rolling updates:

**1. maxSurge:**
- Maximum number of pods that can be created above desired replica count
- Can be absolute number (e.g., 2) or percentage (e.g., 25%)
- Default: 25%
- Example: With 4 replicas and maxSurge=1, max 5 pods during update

**2. maxUnavailable:**
- Maximum number of pods that can be unavailable during update
- Can be absolute number (e.g., 1) or percentage (e.g., 25%)
- Default: 25%
- Example: With 4 replicas and maxUnavailable=1, min 3 pods available

**Strategy Configuration:**
```yaml
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1        # Max 4 pods during update (3 + 1)
      maxUnavailable: 1  # Min 2 pods available (3 - 1)
```

### Deployment Revisions

Kubernetes maintains **revision history** for deployments:

- **Revision 1:** Initial deployment (nginx:latest)
- **Revision 2:** After rolling update (nginx:1.17)
- **Revision 3:** Subsequent update (nginx:1.18)
- **Revision History Limit:** Default 10 (configurable)

Each revision stores:
- ReplicaSet configuration
- Pod template
- Image version
- Resource specifications
- Rollback capability

## Infrastructure Overview

### Kubernetes Cluster Components

**Control Plane:**
- **API Server:** Receives kubectl update commands
- **Controller Manager:** Deployment controller manages rolling updates
- **Scheduler:** Places new pods on appropriate nodes
- **etcd:** Stores deployment revisions and state

**Worker Nodes:**
- **kubelet:** Creates/terminates pods during update
- **Container Runtime:** Pulls nginx:1.17 image, runs containers
- **kube-proxy:** Ensures service routes to available pods only

### Deployment → ReplicaSet → Pods Hierarchy

```
nginx-deployment (Deployment)
├── nginx-deployment-7d9c8f5b4c (Old ReplicaSet - Revision 1)
│   ├── Pod 1 (nginx:latest) ─── Terminating
│   ├── Pod 2 (nginx:latest) ─── Terminating
│   └── Pod 3 (nginx:latest) ─── Terminating
│
└── nginx-deployment-8c6f9d7a3b (New ReplicaSet - Revision 2)
    ├── Pod 4 (nginx:1.17) ─── Running
    ├── Pod 5 (nginx:1.17) ─── Running
    └── Pod 6 (nginx:1.17) ─── Running
```

**During Rolling Update:**
- Deployment creates new ReplicaSet with nginx:1.17
- Old ReplicaSet scales down gradually (replicas: 3 → 2 → 1 → 0)
- New ReplicaSet scales up gradually (replicas: 0 → 1 → 2 → 3)
- Both ReplicaSets exist temporarily during transition
- Old ReplicaSet retained for rollback capability (replicas: 0)

## Step-by-Step Implementation

### Method 1: Imperative Update (Recommended for Task)

#### Step 1: Access the Kubernetes Cluster
```bash
# Connect to jump_host (if not already connected)
ssh username@jump_host

# Verify kubectl is configured
kubectl cluster-info

# Expected Output:
# Kubernetes control plane is running at https://x.x.x.x:6443
# CoreDNS is running at https://x.x.x.x:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
```

#### Step 2: Check Current Deployment Status
```bash
# View the nginx-deployment details
kubectl get deployment nginx-deployment

# Expected Output:
# NAME               READY   UP-TO-DATE   AVAILABLE   AGE
# nginx-deployment   3/3     3            3           10m
```

**Column Meanings:**
- **READY:** Actual running pods / Desired replicas
- **UP-TO-DATE:** Pods updated with latest configuration
- **AVAILABLE:** Pods passing readiness checks
- **AGE:** Deployment creation time

#### Step 3: Check Current Image Version
```bash
# View current container image
kubectl describe deployment nginx-deployment | grep Image

# Expected Output:
# Image:        nginx:latest
# (or another nginx version)

# Alternative: View full deployment details
kubectl get deployment nginx-deployment -o yaml | grep image:
```

#### Step 4: View Current Pods and Their Images
```bash
# List all pods with their images
kubectl get pods -l app=nginx -o wide

# Expected Output:
# NAME                                READY   STATUS    RESTARTS   AGE   IP           NODE
# nginx-deployment-7d9c8f5b4c-abc12   1/1     Running   0          5m    10.244.1.5   node01
# nginx-deployment-7d9c8f5b4c-def34   1/1     Running   0          5m    10.244.2.3   node02
# nginx-deployment-7d9c8f5b4c-ghi56   1/1     Running   0          5m    10.244.1.6   node01

# Check specific pod image
kubectl get pod <pod-name> -o jsonpath='{.spec.containers[0].image}'
```

#### Step 5: Check Deployment Rollout History (Pre-Update)
```bash
# View revision history before update
kubectl rollout history deployment/nginx-deployment

# Expected Output:
# deployment.apps/nginx-deployment
# REVISION  CHANGE-CAUSE
# 1         <none>
```

**Note:** CHANGE-CAUSE shows annotation from `--record` flag (deprecated) or manual annotation.

#### Step 6: Execute Rolling Update (Primary Method)
```bash
# Update deployment to nginx:1.17 image
kubectl set image deployment/nginx-deployment nginx=nginx:1.17

# Expected Output:
# deployment.apps/nginx-deployment image updated

# Alternative: If container name is different
kubectl set image deployment/nginx-deployment <container-name>=nginx:1.17
```

**Command Breakdown:**
- `kubectl set image`: Update container image in deployment
- `deployment/nginx-deployment`: Target deployment
- `nginx=nginx:1.17`: Update container named "nginx" to image "nginx:1.17"

**Note:** Container name must match the name in pod template. Verify with:
```bash
kubectl get deployment nginx-deployment -o jsonpath='{.spec.template.spec.containers[0].name}'
```

#### Step 7: Watch Rolling Update in Real-Time
```bash
# Monitor update progress (live updates)
kubectl rollout status deployment/nginx-deployment

# Expected Output (progressive):
# Waiting for deployment "nginx-deployment" rollout to finish: 1 out of 3 new replicas have been updated...
# Waiting for deployment "nginx-deployment" rollout to finish: 1 old replicas are pending termination...
# Waiting for deployment "nginx-deployment" rollout to finish: 2 of 3 updated replicas are available...
# deployment "nginx-deployment" successfully rolled out

# Alternative: Watch pods changing (separate terminal)
kubectl get pods -l app=nginx -w
```

**What You'll See:**
1. New pods created with `-<new-hash>` suffix
2. New pods transition: Pending → ContainerCreating → Running
3. Old pods transition: Running → Terminating → Deleted
4. Process repeats until all pods updated

#### Step 8: Verify Update Completion
```bash
# Check deployment status post-update
kubectl get deployment nginx-deployment

# Expected Output:
# NAME               READY   UP-TO-DATE   AVAILABLE   AGE
# nginx-deployment   3/3     3            3           15m
#                    ^^^     ^^^          ^^^
#                     |       |            +-- All pods passing health checks
#                     |       +-- All pods updated with new image
#                     +-- All desired replicas running
```

**Success Indicators:**
- READY shows all replicas (e.g., 3/3)
- UP-TO-DATE equals total replicas (all pods updated)
- AVAILABLE equals total replicas (all pods healthy)

#### Step 9: Verify New Image in Pods
```bash
# Check image version in deployment
kubectl describe deployment nginx-deployment | grep Image

# Expected Output:
# Image:        nginx:1.17

# Verify in running pods
kubectl get pods -l app=nginx -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.spec.containers[0].image}{"\n"}{end}'

# Expected Output:
# nginx-deployment-8c6f9d7a3b-xyz01    nginx:1.17
# nginx-deployment-8c6f9d7a3b-xyz02    nginx:1.17
# nginx-deployment-8c6f9d7a3b-xyz03    nginx:1.17
```

#### Step 10: Verify All Pods are Running
```bash
# Check pod status
kubectl get pods -l app=nginx

# Expected Output: All pods with STATUS = Running
# NAME                                READY   STATUS    RESTARTS   AGE
# nginx-deployment-8c6f9d7a3b-xyz01   1/1     Running   0          2m
# nginx-deployment-8c6f9d7a3b-xyz02   1/1     Running   0          2m
# nginx-deployment-8c6f9d7a3b-xyz03   1/1     Running   0          2m
```

**Verify:**
- All pods show `STATUS: Running`
- All pods show `READY: 1/1`
- All pods have low RESTARTS count (0 or minimal)
- All pods have recent AGE (created during update)

#### Step 11: Check Rollout History (Post-Update)
```bash
# View updated revision history
kubectl rollout history deployment/nginx-deployment

# Expected Output:
# deployment.apps/nginx-deployment
# REVISION  CHANGE-CAUSE
# 1         <none>
# 2         <none>
```

**Revision 2 = Current deployment with nginx:1.17**

#### Step 12: Verify ReplicaSets
```bash
# View ReplicaSets created by deployment
kubectl get replicasets -l app=nginx

# Expected Output:
# NAME                          DESIRED   CURRENT   READY   AGE
# nginx-deployment-7d9c8f5b4c   0         0         0       15m   (Old - Revision 1)
# nginx-deployment-8c6f9d7a3b   3         3         3       5m    (New - Revision 2)
```

**Analysis:**
- Old ReplicaSet scaled to 0 (kept for rollback)
- New ReplicaSet managing 3 pods (current deployment)
- Both ReplicaSets retained in history

#### Step 13: Detailed Deployment Verification
```bash
# Check deployment events for update confirmation
kubectl describe deployment nginx-deployment

# Look for events like:
# Events:
#   Type    Reason             Age   From                   Message
#   ----    ------             ----  ----                   -------
#   Normal  ScalingReplicaSet  5m    deployment-controller  Scaled up replica set nginx-deployment-8c6f9d7a3b to 1
#   Normal  ScalingReplicaSet  5m    deployment-controller  Scaled down replica set nginx-deployment-7d9c8f5b4c to 2
#   Normal  ScalingReplicaSet  4m    deployment-controller  Scaled up replica set nginx-deployment-8c6f9d7a3b to 2
#   Normal  ScalingReplicaSet  4m    deployment-controller  Scaled down replica set nginx-deployment-7d9c8f5b4c to 1
#   Normal  ScalingReplicaSet  4m    deployment-controller  Scaled up replica set nginx-deployment-8c6f9d7a3b to 3
#   Normal  ScalingReplicaSet  4m    deployment-controller  Scaled down replica set nginx-deployment-7d9c8f5b4c to 0
```

**Events Show:**
- New ReplicaSet scaled up: 0 → 1 → 2 → 3
- Old ReplicaSet scaled down: 3 → 2 → 1 → 0
- Rolling update executed successfully

#### Step 14: Test Application Functionality (Optional)
```bash
# If service exists, get service details
kubectl get service nginx-service

# Test nginx response (if service is exposed)
curl http://<service-ip>:<port>

# Expected: Nginx default page or application response

# Check nginx version in pod
kubectl exec -it <pod-name> -- nginx -v

# Expected Output:
# nginx version: nginx/1.17.x
```

#### Step 15: Validate Pod Health and Readiness
```bash
# Check pod conditions
kubectl get pods -l app=nginx -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.status.conditions[?(@.type=="Ready")].status}{"\n"}{end}'

# Expected Output: All pods show "True" for Ready condition
# nginx-deployment-8c6f9d7a3b-xyz01    True
# nginx-deployment-8c6f9d7a3b-xyz02    True
# nginx-deployment-8c6f9d7a3b-xyz03    True

# Detailed pod health check
kubectl describe pod <pod-name> | grep -A 5 Conditions
```

### Method 2: Declarative Update (Alternative Method)

#### Step 1: Export Current Deployment Configuration
```bash
# Save current deployment to YAML file
kubectl get deployment nginx-deployment -o yaml > nginx-deployment.yaml
```

#### Step 2: Edit Deployment YAML
```bash
# Edit the YAML file
nano nginx-deployment.yaml

# OR
vim nginx-deployment.yaml
```

**Find and update the image field:**
```yaml
spec:
  containers:
  - name: nginx
    image: nginx:latest    # Change this line
    # TO:
    image: nginx:1.17      # Updated line
```

**Complete Example:**
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
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.17    # Updated image version
        ports:
        - containerPort: 80
```

#### Step 3: Apply Updated Configuration
```bash
# Apply changes to cluster
kubectl apply -f nginx-deployment.yaml

# Expected Output:
# deployment.apps/nginx-deployment configured
```

#### Step 4: Follow Steps 7-15 from Method 1
Continue with verification steps to ensure rolling update succeeded.

### Method 3: Edit Deployment Directly (Quick Method)

#### Step 1: Edit Deployment in Real-Time
```bash
# Open deployment in default editor
kubectl edit deployment nginx-deployment

# This opens vim/nano with deployment YAML
```

#### Step 2: Modify Image Field
Find the `spec.template.spec.containers[].image` field and change to `nginx:1.17`:
```yaml
spec:
  template:
    spec:
      containers:
      - image: nginx:1.17    # Change this line
        name: nginx
```

#### Step 3: Save and Exit
```bash
# In vim: Press ESC, type :wq, press ENTER
# In nano: Press CTRL+X, then Y, then ENTER

# Expected Output after saving:
# deployment.apps/nginx-deployment edited
```

Rolling update starts automatically upon saving.

#### Step 4: Follow Steps 7-15 from Method 1
Continue with verification steps.

## Complete Command Summary

### Rolling Update Commands
```bash
# Execute rolling update (imperative - recommended)
kubectl set image deployment/nginx-deployment nginx=nginx:1.17

# Execute rolling update (declarative)
kubectl apply -f nginx-deployment.yaml

# Execute rolling update (direct edit)
kubectl edit deployment nginx-deployment
```

### Monitoring Commands
```bash
# Watch rollout progress
kubectl rollout status deployment/nginx-deployment

# Watch pods in real-time
kubectl get pods -l app=nginx -w

# View rollout history
kubectl rollout history deployment/nginx-deployment

# View specific revision details
kubectl rollout history deployment/nginx-deployment --revision=2
```

### Verification Commands
```bash
# Check deployment status
kubectl get deployment nginx-deployment

# Check pods with images
kubectl get pods -l app=nginx -o wide

# Verify image in deployment
kubectl describe deployment nginx-deployment | grep Image

# Verify image in pods
kubectl get pods -l app=nginx -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.spec.containers[0].image}{"\n"}{end}'

# Check ReplicaSets
kubectl get replicasets -l app=nginx

# Detailed deployment info
kubectl describe deployment nginx-deployment
```

### Rollback Commands (If Needed)
```bash
# Rollback to previous revision
kubectl rollout undo deployment/nginx-deployment

# Rollback to specific revision
kubectl rollout undo deployment/nginx-deployment --to-revision=1

# Verify rollback
kubectl rollout status deployment/nginx-deployment
```

## Kubectl Rolling Update Commands Reference

### Update Operations
```bash
# Update image (single container)
kubectl set image deployment/<name> <container>=<image>:<tag>

# Update image (multiple containers)
kubectl set image deployment/<name> container1=image1:tag1 container2=image2:tag2

# Update using YAML
kubectl apply -f deployment.yaml

# Direct edit
kubectl edit deployment/<name>

# Patch deployment (JSON)
kubectl patch deployment/<name> -p '{"spec":{"template":{"spec":{"containers":[{"name":"nginx","image":"nginx:1.17"}]}}}}'
```

### Rollout Management
```bash
# Check rollout status
kubectl rollout status deployment/<name>

# View rollout history
kubectl rollout history deployment/<name>

# View specific revision
kubectl rollout history deployment/<name> --revision=<number>

# Pause rollout (stop update temporarily)
kubectl rollout pause deployment/<name>

# Resume rollout (continue paused update)
kubectl rollout resume deployment/<name>

# Rollback to previous revision
kubectl rollout undo deployment/<name>

# Rollback to specific revision
kubectl rollout undo deployment/<name> --to-revision=<number>

# Restart deployment (recreate all pods)
kubectl rollout restart deployment/<name>
```

### Strategy Configuration
```bash
# View current strategy
kubectl get deployment/<name> -o jsonpath='{.spec.strategy}'

# Update strategy via patch
kubectl patch deployment/<name> -p '{"spec":{"strategy":{"type":"RollingUpdate","rollingUpdate":{"maxSurge":"1","maxUnavailable":"1"}}}}'
```

### Verification
```bash
# Get deployment details
kubectl get deployment/<name>
kubectl describe deployment/<name>

# Get pods with labels
kubectl get pods -l <label-key>=<label-value>
kubectl get pods -l app=nginx -o wide

# Check pod images
kubectl get pods -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.spec.containers[0].image}{"\n"}{end}'

# Get ReplicaSets
kubectl get replicasets -l <label>
kubectl describe replicaset/<replicaset-name>

# Check deployment conditions
kubectl get deployment/<name> -o jsonpath='{.status.conditions[*].type}{"\n"}{.status.conditions[*].status}'
```

## Troubleshooting Rolling Updates

### Issue 1: Rolling Update Stuck - Pods Not Starting

**Symptoms:**
```bash
kubectl rollout status deployment/nginx-deployment
# Output: Waiting for deployment "nginx-deployment" rollout to finish: 1 out of 3 new replicas have been updated...
# (Hangs indefinitely)

kubectl get pods
# NAME                                READY   STATUS             RESTARTS   AGE
# nginx-deployment-8c6f9d7a3b-xyz01   0/1     ImagePullBackOff   0          5m
```

**Causes:**
- Image doesn't exist (typo in image name)
- Image pull authentication failure
- Network issues downloading image

**Diagnosis:**
```bash
# Check pod events
kubectl describe pod <pod-name>

# Look for:
# Events:
#   Failed to pull image "nginx:1.17": rpc error: code = Unknown desc = Error response from daemon: manifest for nginx:1.17 not found
```

**Solutions:**
```bash
# 1. Verify image exists
docker pull nginx:1.17
# If this fails, image name is incorrect

# 2. Check correct image name/tag
# Visit Docker Hub or check with development team

# 3. Update to correct image
kubectl set image deployment/nginx-deployment nginx=nginx:1.17.0

# 4. If private registry, add image pull secret
kubectl create secret docker-registry regcred \
  --docker-server=<registry-url> \
  --docker-username=<username> \
  --docker-password=<password>

# Add secret to deployment
kubectl patch deployment nginx-deployment -p '{"spec":{"template":{"spec":{"imagePullSecrets":[{"name":"regcred"}]}}}}'
```

### Issue 2: Pods Crash After Update (CrashLoopBackOff)

**Symptoms:**
```bash
kubectl get pods
# NAME                                READY   STATUS             RESTARTS   AGE
# nginx-deployment-8c6f9d7a3b-xyz01   0/1     CrashLoopBackOff   5          5m

kubectl rollout status deployment/nginx-deployment
# Waiting for deployment "nginx-deployment" rollout to finish: 1 out of 3 new replicas have been updated...
```

**Causes:**
- Application error in new version
- Configuration incompatibility
- Missing environment variables
- Port conflicts

**Diagnosis:**
```bash
# Check pod logs
kubectl logs <pod-name>

# Check previous container logs (if restarted)
kubectl logs <pod-name> --previous

# Check pod events
kubectl describe pod <pod-name>
```

**Solutions:**
```bash
# 1. Immediate rollback to previous working version
kubectl rollout undo deployment/nginx-deployment

# Verify rollback
kubectl rollout status deployment/nginx-deployment

# 2. Check rollout history
kubectl rollout history deployment/nginx-deployment

# 3. Fix configuration issue, then retry update
kubectl set image deployment/nginx-deployment nginx=nginx:1.17

# 4. If persistent, check deployment configuration
kubectl get deployment nginx-deployment -o yaml > debug-deployment.yaml
# Review for configuration errors
```

### Issue 3: Rolling Update Taking Too Long

**Symptoms:**
```bash
kubectl rollout status deployment/nginx-deployment
# Waiting for deployment "nginx-deployment" rollout to finish: 2 out of 10 new replicas have been updated...
# (Very slow progress, 1 pod every 5+ minutes)
```

**Causes:**
- Large image size (slow download)
- Conservative update strategy (maxSurge=0, maxUnavailable=1)
- Resource constraints on nodes
- Slow readiness probes

**Diagnosis:**
```bash
# Check update strategy
kubectl get deployment nginx-deployment -o jsonpath='{.spec.strategy}'

# Check pod resource requests
kubectl get deployment nginx-deployment -o jsonpath='{.spec.template.spec.containers[0].resources}'

# Check node resources
kubectl top nodes

# Check image pull time
kubectl describe pod <new-pod-name> | grep -A 10 Events
# Look for "Pulling image" and "Successfully pulled image" timestamps
```

**Solutions:**
```bash
# 1. Adjust rolling update strategy for faster updates
kubectl patch deployment nginx-deployment -p '{"spec":{"strategy":{"rollingUpdate":{"maxSurge":2,"maxUnavailable":1}}}}'

# 2. Pre-pull image on nodes (if possible)
# SSH to each node:
docker pull nginx:1.17

# 3. Adjust readiness probe (if too conservative)
kubectl edit deployment nginx-deployment
# Reduce initialDelaySeconds or periodSeconds

# 4. Resume if paused
kubectl rollout resume deployment/nginx-deployment
```

### Issue 4: Old Pods Not Terminating

**Symptoms:**
```bash
kubectl get pods
# NAME                                READY   STATUS        RESTARTS   AGE
# nginx-deployment-7d9c8f5b4c-old01   1/1     Terminating   0          15m
# nginx-deployment-7d9c8f5b4c-old02   1/1     Terminating   0          15m
# nginx-deployment-8c6f9d7a3b-new01   1/1     Running       0          5m
# nginx-deployment-8c6f9d7a3b-new02   1/1     Running       0          5m
# nginx-deployment-8c6f9d7a3b-new03   1/1     Running       0          5m

# Old pods stuck in Terminating for 10+ minutes
```

**Causes:**
- Application not handling SIGTERM gracefully
- Long terminationGracePeriodSeconds
- Finalizers preventing deletion
- Storage volume not releasing

**Diagnosis:**
```bash
# Check termination grace period
kubectl get deployment nginx-deployment -o jsonpath='{.spec.template.spec.terminationGracePeriodSeconds}'

# Check pod details
kubectl describe pod <terminating-pod-name>

# Check for finalizers
kubectl get pod <terminating-pod-name> -o jsonpath='{.metadata.finalizers}'
```

**Solutions:**
```bash
# 1. Wait for graceful termination (default 30 seconds)
# Pods should terminate automatically

# 2. Force delete stuck pods (use cautiously)
kubectl delete pod <pod-name> --grace-period=0 --force

# 3. Remove finalizers if blocking deletion
kubectl patch pod <pod-name> -p '{"metadata":{"finalizers":null}}'

# 4. Reduce termination grace period (for future updates)
kubectl patch deployment nginx-deployment -p '{"spec":{"template":{"spec":{"terminationGracePeriodSeconds":10}}}}'
```

### Issue 5: Wrong Container Name in Update Command

**Symptoms:**
```bash
kubectl set image deployment/nginx-deployment nginx-container=nginx:1.17
# Error from server (NotFound): containers "nginx-container" not found in deployment spec
```

**Cause:**
- Container name mismatch between command and deployment spec

**Diagnosis:**
```bash
# Check actual container name in deployment
kubectl get deployment nginx-deployment -o jsonpath='{.spec.template.spec.containers[*].name}'

# Expected Output:
# nginx
# (Not "nginx-container")
```

**Solution:**
```bash
# Use correct container name
kubectl set image deployment/nginx-deployment nginx=nginx:1.17
#                                               ^^^^^ Correct container name

# If multiple containers, specify all
kubectl set image deployment/nginx-deployment container1=image1:tag container2=image2:tag
```

### Issue 6: Rollback Doesn't Work

**Symptoms:**
```bash
kubectl rollout undo deployment/nginx-deployment
# Error: no rollout history found for deployment "nginx-deployment"
```

**Cause:**
- Revision history limit set to 0 or very low
- Revision history pruned

**Diagnosis:**
```bash
# Check revision history limit
kubectl get deployment nginx-deployment -o jsonpath='{.spec.revisionHistoryLimit}'

# Check existing revisions
kubectl rollout history deployment/nginx-deployment
```

**Solutions:**
```bash
# 1. Set revision history limit (for future)
kubectl patch deployment nginx-deployment -p '{"spec":{"revisionHistoryLimit":10}}'

# 2. If no history, manually update to previous image
kubectl set image deployment/nginx-deployment nginx=<previous-image>:<previous-tag>

# 3. Restore from backup YAML (if available)
kubectl apply -f nginx-deployment-backup.yaml
```

### Issue 7: Update Succeeds But Application Doesn't Work

**Symptoms:**
```bash
kubectl get pods
# All pods Running with READY 1/1

curl http://<service-ip>
# Connection refused or 503 Service Unavailable
```

**Causes:**
- Readiness probe not configured (pods marked ready prematurely)
- Service selector mismatch
- Application listening on wrong port
- Configuration missing in new version

**Diagnosis:**
```bash
# Check service selector
kubectl get service nginx-service -o jsonpath='{.spec.selector}'

# Check pod labels
kubectl get pods -l app=nginx --show-labels

# Check container port
kubectl get deployment nginx-deployment -o jsonpath='{.spec.template.spec.containers[0].ports}'

# Test pod directly (bypass service)
kubectl port-forward <pod-name> 8080:80
curl http://localhost:8080

# Check readiness probe
kubectl get deployment nginx-deployment -o jsonpath='{.spec.template.spec.containers[0].readinessProbe}'
```

**Solutions:**
```bash
# 1. Add/fix readiness probe
kubectl edit deployment nginx-deployment
# Add:
# readinessProbe:
#   httpGet:
#     path: /
#     port: 80
#   initialDelaySeconds: 5
#   periodSeconds: 5

# 2. Fix service selector (if mismatch)
kubectl edit service nginx-service
# Ensure selector matches pod labels

# 3. Update port configuration
kubectl patch deployment nginx-deployment -p '{"spec":{"template":{"spec":{"containers":[{"name":"nginx","ports":[{"containerPort":80}]}]}}}}'

# 4. Check application logs
kubectl logs <pod-name>
```

## Advanced Rolling Update Strategies

### 1. Canary Deployments

Deploy new version to small subset of pods first:

```yaml
# Create separate deployment for canary
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment-canary
spec:
  replicas: 1    # Only 1 pod initially
  selector:
    matchLabels:
      app: nginx
      version: canary
  template:
    metadata:
      labels:
        app: nginx
        version: canary
    spec:
      containers:
      - name: nginx
        image: nginx:1.17    # New version
        ports:
        - containerPort: 80
```

**Process:**
1. Deploy canary with 1 replica (new version)
2. Monitor metrics, logs, errors
3. If successful, gradually increase canary replicas
4. Decrease main deployment replicas
5. Eventually replace main deployment

**Service routes to both:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx    # Matches both main and canary
  ports:
  - port: 80
    targetPort: 80
```

### 2. Blue-Green Deployments

Maintain two complete environments:

```yaml
# Blue deployment (current production)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-blue
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
      version: blue
  template:
    metadata:
      labels:
        app: nginx
        version: blue
    spec:
      containers:
      - name: nginx
        image: nginx:latest

---
# Green deployment (new version)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-green
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
      version: green
  template:
    metadata:
      labels:
        app: nginx
        version: green
    spec:
      containers:
      - name: nginx
        image: nginx:1.17
```

**Switch traffic by updating service:**
```bash
# Initially pointing to blue
kubectl patch service nginx-service -p '{"spec":{"selector":{"version":"blue"}}}'

# Test green deployment
kubectl port-forward deployment/nginx-green 8080:80

# Switch to green (instant cutover)
kubectl patch service nginx-service -p '{"spec":{"selector":{"version":"green"}}}'

# Rollback to blue if issues
kubectl patch service nginx-service -p '{"spec":{"selector":{"version":"blue"}}}'
```

### 3. Custom maxSurge and maxUnavailable

Fine-tune rolling update behavior:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 10
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 3           # Max 13 pods during update (10 + 3)
      maxUnavailable: 2     # Min 8 pods available (10 - 2)
  # ... rest of spec
```

**Scenarios:**

**Fast Update (more resources):**
```yaml
maxSurge: 50%          # Can create 5 extra pods (50% of 10)
maxUnavailable: 0      # Always keep 10 pods available
# Result: Creates all new pods first, then terminates old
```

**Resource-Constrained (slower but less resource use):**
```yaml
maxSurge: 0            # No extra pods created
maxUnavailable: 1      # Can have 9 pods available (10 - 1)
# Result: Terminates 1 old pod, creates 1 new pod, repeat
```

**Balanced (default):**
```yaml
maxSurge: 25%          # Can create 2-3 extra pods
maxUnavailable: 25%    # Can have 7-8 pods available
# Result: Moderate speed, moderate resource usage
```

### 4. Progressive Delivery with Flagger

Automate canary deployments with metrics:

```yaml
apiVersion: flagger.app/v1beta1
kind: Canary
metadata:
  name: nginx
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx-deployment
  service:
    port: 80
  analysis:
    interval: 1m
    threshold: 5
    maxWeight: 50
    stepWeight: 10
    metrics:
    - name: request-success-rate
      thresholdRange:
        min: 99
      interval: 1m
```

**Progressive rollout:**
- 10% traffic to new version (monitor 1 min)
- If success rate > 99%, increase to 20%
- Continue until 50% traffic
- If metrics fail, automatic rollback

## Best Practices for Rolling Updates

### 1. Always Use Deployments (Not Bare Pods)
❌ **Bad:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx
    image: nginx:latest
```
No rolling update capability, no self-healing.

✅ **Good:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  # ... rolling update enabled automatically
```

### 2. Tag Images with Specific Versions (Not :latest)
❌ **Bad:**
```yaml
spec:
  containers:
  - name: nginx
    image: nginx:latest    # Ambiguous, no version control
```

✅ **Good:**
```yaml
spec:
  containers:
  - name: nginx
    image: nginx:1.17.0    # Specific version, reproducible
```

**Why:**
- `:latest` is ambiguous (what version is running?)
- Can't rollback reliably (`:latest` changes)
- Inconsistent across nodes (pulled at different times)
- Production deployments need reproducibility

### 3. Configure Readiness Probes
❌ **Bad:**
```yaml
spec:
  containers:
  - name: nginx
    image: nginx:1.17
    # No readiness probe - pod marked ready immediately
```

✅ **Good:**
```yaml
spec:
  containers:
  - name: nginx
    image: nginx:1.17
    readinessProbe:
      httpGet:
        path: /health
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 5
```

**Why:**
- Ensures pods are actually ready to serve traffic
- Prevents routing traffic to broken pods
- Rolling update waits for readiness before continuing
- Critical for zero-downtime deployments

### 4. Set Resource Requests and Limits
❌ **Bad:**
```yaml
spec:
  containers:
  - name: nginx
    image: nginx:1.17
    # No resources - unpredictable behavior
```

✅ **Good:**
```yaml
spec:
  containers:
  - name: nginx
    image: nginx:1.17
    resources:
      requests:
        memory: "128Mi"
        cpu: "100m"
      limits:
        memory: "256Mi"
        cpu: "200m"
```

**Why:**
- Scheduler can place pods appropriately
- Prevents resource starvation during updates
- Ensures both old and new pods can run simultaneously
- Critical when maxSurge > 0 (extra pods during update)

### 5. Test Updates in Non-Production First
```bash
# Test in dev/staging environment
kubectl set image deployment/nginx-deployment nginx=nginx:1.17 --namespace=staging

# Monitor for issues
kubectl rollout status deployment/nginx-deployment --namespace=staging

# If successful, deploy to production
kubectl set image deployment/nginx-deployment nginx=nginx:1.17 --namespace=production
```

### 6. Maintain Revision History
```yaml
spec:
  revisionHistoryLimit: 10    # Keep last 10 revisions
  # Default is 10, but explicitly set for clarity
```

**Why:**
- Enables rollback to any previous version
- Useful for auditing deployment history
- Balance between rollback capability and storage

### 7. Use Declarative Configuration (YAML in Git)
```bash
# Store deployment in version control
git add nginx-deployment.yaml
git commit -m "Update nginx to 1.17"
git push

# Apply from git repository
kubectl apply -f nginx-deployment.yaml
```

**Why:**
- Version control for infrastructure
- Code review for deployment changes
- Audit trail of who changed what
- Easy rollback (git revert)
- GitOps workflows

### 8. Monitor Rollout Progress
```bash
# Don't just run update and walk away
kubectl set image deployment/nginx-deployment nginx=nginx:1.17

# ALWAYS watch the rollout
kubectl rollout status deployment/nginx-deployment -w

# Monitor pod status
kubectl get pods -l app=nginx -w

# Check for errors
kubectl describe deployment nginx-deployment
```

### 9. Plan for Rollback
```bash
# Before update, document current state
kubectl get deployment nginx-deployment -o yaml > nginx-deployment-backup.yaml

# Note current image version
kubectl describe deployment nginx-deployment | grep Image

# After update, test rollback procedure in staging
kubectl rollout undo deployment/nginx-deployment --namespace=staging
```

### 10. Use Appropriate Update Strategy
```yaml
# For critical, high-traffic services
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0    # Never reduce capacity

# For batch jobs, non-critical services
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 100%       # Fast updates
      maxUnavailable: 50%

# For stateful apps requiring full restart
spec:
  strategy:
    type: Recreate    # Terminate all, then create all
```

## Real-World Rolling Update Examples

### Example 1: Web Application (Nginx)

**Scenario:** Update nginx web server from 1.16 to 1.17, zero downtime required, high traffic.

**Deployment Configuration:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-web
  labels:
    app: nginx
    tier: frontend
spec:
  replicas: 5    # High availability
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 2           # Can handle extra load
      maxUnavailable: 1     # Maintain capacity
  selector:
    matchLabels:
      app: nginx
      tier: frontend
  template:
    metadata:
      labels:
        app: nginx
        tier: frontend
    spec:
      containers:
      - name: nginx
        image: nginx:1.17.0    # Specific version
        ports:
        - containerPort: 80
          name: http
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "256Mi"
            cpu: "200m"
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
          successThreshold: 1
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 15
          periodSeconds: 10
```

**Update Process:**
```bash
# 1. Update image
kubectl set image deployment/nginx-web nginx=nginx:1.17.0

# 2. Monitor (expect ~2-3 minutes for 5 replicas)
kubectl rollout status deployment/nginx-web

# 3. Verify
kubectl get pods -l app=nginx,tier=frontend
kubectl describe deployment nginx-web | grep Image

# 4. Test application
curl http://<service-ip>
```

**Expected Behavior:**
- Creates 2 new pods (maxSurge=2): 5 + 2 = 7 total
- Terminates 1 old pod: 6 total (5 desired + 1 surge)
- Creates 1 new pod, terminates 1 old pod
- Repeat until all 5 pods running nginx:1.17.0
- Zero downtime (always ≥4 pods available)

### Example 2: API Backend (Node.js)

**Scenario:** Update Node.js API from v2.0.0 to v2.1.0, needs database migrations, careful rollout.

**Deployment Configuration:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-backend
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0    # Never reduce capacity (API must stay available)
  selector:
    matchLabels:
      app: api
  template:
    metadata:
      labels:
        app: api
    spec:
      initContainers:
      - name: db-migrate
        image: myapp:2.1.0
        command: ['npm', 'run', 'migrate']    # Run migrations before starting
      containers:
      - name: api
        image: myapp:2.1.0
        ports:
        - containerPort: 3000
        env:
        - name: DB_HOST
          value: postgres-service
        resources:
          requests:
            memory: "256Mi"
            cpu: "200m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        readinessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 10    # App needs startup time
          periodSeconds: 5
          failureThreshold: 3
        livenessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 30
          periodSeconds: 10
```

**Update Process:**
```bash
# 1. Backup database (critical!)
kubectl exec -it postgres-pod -- pg_dump mydb > backup.sql

# 2. Update deployment
kubectl apply -f api-deployment.yaml

# 3. Watch carefully (migrations run in initContainer)
kubectl rollout status deployment/api-backend -w

# 4. Check for migration errors
kubectl logs -l app=api -c db-migrate

# 5. Verify API functionality
curl http://<api-service>/health
curl http://<api-service>/api/v2/endpoint

# 6. Monitor application logs
kubectl logs -l app=api --tail=50 -f
```

**Expected Behavior:**
- Creates 1 new pod (maxSurge=1): 4 total pods
- Runs db-migrate initContainer (database migrations)
- Starts new pod with v2.1.0
- Waits for readiness probe (health check passes)
- Terminates 1 old pod: 3 total pods
- Repeat twice more
- Always maintains 3 pods available (maxUnavailable=0)

### Example 3: Microservice with Canary (Redis Cache)

**Scenario:** Update Redis from 6.2 to 7.0, test with canary first due to major version change.

**Main Deployment:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-main
  labels:
    app: redis
    version: stable
spec:
  replicas: 2
  selector:
    matchLabels:
      app: redis
      version: stable
  template:
    metadata:
      labels:
        app: redis
        version: stable
    spec:
      containers:
      - name: redis
        image: redis:6.2-alpine
        ports:
        - containerPort: 6379
        resources:
          requests:
            memory: "512Mi"
            cpu: "250m"
          limits:
            memory: "1Gi"
            cpu: "500m"
        readinessProbe:
          exec:
            command:
            - redis-cli
            - ping
          initialDelaySeconds: 5
          periodSeconds: 3
```

**Canary Deployment:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-canary
  labels:
    app: redis
    version: canary
spec:
  replicas: 1    # Only 1 pod for canary testing
  selector:
    matchLabels:
      app: redis
      version: canary
  template:
    metadata:
      labels:
        app: redis
        version: canary
    spec:
      containers:
      - name: redis
        image: redis:7.0-alpine    # New version
        ports:
        - containerPort: 6379
        resources:
          requests:
            memory: "512Mi"
            cpu: "250m"
          limits:
            memory: "1Gi"
            cpu: "500m"
        readinessProbe:
          exec:
            command:
            - redis-cli
            - ping
          initialDelaySeconds: 5
          periodSeconds: 3
```

**Service (routes to both):**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: redis-service
spec:
  selector:
    app: redis    # Matches both stable and canary
  ports:
  - port: 6379
    targetPort: 6379
```

**Update Process:**
```bash
# 1. Deploy canary (1 pod with redis:7.0)
kubectl apply -f redis-canary.yaml

# 2. Monitor canary pod
kubectl get pods -l version=canary -w

# 3. Test canary directly
kubectl port-forward -n default deployment/redis-canary 6379:6379
redis-cli -p 6379 ping    # Should return PONG

# 4. Monitor metrics (memory, CPU, response time)
kubectl top pods -l version=canary

# 5. Check logs for errors
kubectl logs -l version=canary --tail=100

# 6. If canary successful, update main deployment
kubectl set image deployment/redis-main redis=redis:7.0-alpine

# 7. Monitor main deployment rollout
kubectl rollout status deployment/redis-main

# 8. Once stable, remove canary
kubectl delete deployment redis-canary

# 9. Verify all pods running new version
kubectl get pods -l app=redis -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.spec.containers[0].image}{"\n"}{end}'
```

**Traffic Distribution (approximate):**
- 2 stable pods (redis:6.2) + 1 canary pod (redis:7.0) = 3 total
- Service routes ~33% traffic to canary, ~67% to stable
- Monitor canary for issues before full rollout

### Example 4: Batch Processing (Data Pipeline)

**Scenario:** Update data processing job from v1.0 to v2.0, can tolerate brief downtime, processes large batches.

**Deployment Configuration:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: data-processor
spec:
  replicas: 1    # Single replica for batch job
  strategy:
    type: Recreate    # Terminate old, then create new (ensures no parallel processing)
  selector:
    matchLabels:
      app: data-processor
  template:
    metadata:
      labels:
        app: data-processor
    spec:
      containers:
      - name: processor
        image: data-processor:2.0.0
        env:
        - name: BATCH_SIZE
          value: "1000"
        - name: DATA_SOURCE
          value: "s3://mybucket/data"
        resources:
          requests:
            memory: "2Gi"
            cpu: "1000m"
          limits:
            memory: "4Gi"
            cpu: "2000m"
        livenessProbe:
          httpGet:
            path: /healthz
            port: 8080
          initialDelaySeconds: 60
          periodSeconds: 30
      terminationGracePeriodSeconds: 300    # Allow 5 minutes for graceful shutdown
```

**Update Process:**
```bash
# 1. Update image (uses Recreate strategy)
kubectl set image deployment/data-processor processor=data-processor:2.0.0

# 2. Watch rollout (will show termination then creation)
kubectl rollout status deployment/data-processor

# Expected:
# Waiting for deployment spec update to be observed...
# Waiting for deployment "data-processor" rollout to finish: 0 out of 1 new replicas have been updated...
# Waiting for deployment "data-processor" rollout to finish: 0 of 1 updated replicas are available...
# deployment "data-processor" successfully rolled out

# 3. Verify new pod running
kubectl get pods -l app=data-processor

# 4. Check processing logs
kubectl logs -l app=data-processor --tail=100 -f

# 5. Verify batch processing working
kubectl exec -it <processor-pod> -- curl localhost:8080/status
```

**Expected Behavior:**
- Terminates existing pod immediately (Recreate strategy)
- Waits 300 seconds for graceful shutdown (finishes current batch)
- Creates new pod with v2.0.0
- Brief downtime acceptable for batch processing
- No parallel processing of batches (avoids data inconsistency)

## Completion Checklist

Run these commands to verify successful rolling update:

```bash
# 1. Deployment status shows all replicas ready
kubectl get deployment nginx-deployment
# READY should show X/X (e.g., 3/3)
# UP-TO-DATE should equal READY
# AVAILABLE should equal READY

# 2. All pods running with new image
kubectl get pods -l app=nginx
# All pods STATUS: Running
# All pods READY: 1/1

# 3. Image version correct in deployment
kubectl describe deployment nginx-deployment | grep Image
# Should show: Image: nginx:1.17

# 4. Image version correct in all pods
kubectl get pods -l app=nginx -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.spec.containers[0].image}{"\n"}{end}'
# All pods should show: nginx:1.17

# 5. Rollout completed successfully
kubectl rollout status deployment/nginx-deployment
# Should show: deployment "nginx-deployment" successfully rolled out

# 6. Rollout history shows new revision
kubectl rollout history deployment/nginx-deployment
# Should show at least 2 revisions

# 7. ReplicaSets show old scaled to 0, new scaled to desired
kubectl get replicasets -l app=nginx
# Old RS: DESIRED=0, CURRENT=0, READY=0
# New RS: DESIRED=3, CURRENT=3, READY=3

# 8. No pods in error states
kubectl get pods -l app=nginx --field-selector=status.phase!=Running
# Should return: No resources found

# 9. Recent events show successful scaling
kubectl describe deployment nginx-deployment | grep -A 10 Events
# Should show: ScalingReplicaSet events for new RS up and old RS down

# 10. Application responds correctly (if service exists)
curl http://<service-ip>:<port>
# Should return expected response

# 11. Nginx version in pod confirms update
kubectl exec -it <pod-name> -- nginx -v
# Should show: nginx version: nginx/1.17.x
```

**All checks passing = Rolling update successful! ✅**

## Summary

### What We Accomplished
1. ✅ Executed rolling update for nginx-deployment from existing version to nginx:1.17
2. ✅ Used `kubectl set image` command for seamless image update
3. ✅ Monitored rollout progress with `kubectl rollout status`
4. ✅ Verified all pods operational with new image version
5. ✅ Confirmed zero downtime during update process
6. ✅ Maintained deployment revision history for rollback capability

### Key Concepts Learned
- **Rolling Updates:** Gradual pod replacement for zero-downtime deployments
- **Update Strategies:** RollingUpdate (default) vs Recreate for different scenarios
- **Rollout Control:** maxSurge and maxUnavailable parameters control update speed
- **Revision History:** Kubernetes maintains deployment history for easy rollback
- **ReplicaSet Management:** Old RS scaled to 0, new RS created during update
- **Health Checks:** Readiness probes ensure new pods ready before receiving traffic

### Critical Commands
```bash
# Execute rolling update
kubectl set image deployment/<name> <container>=<image>:<tag>

# Monitor update progress
kubectl rollout status deployment/<name>

# Verify success
kubectl get deployment <name>
kubectl get pods -l <label>

# Rollback if needed
kubectl rollout undo deployment/<name>
```

### Production Best Practices
1. ✅ Always use specific image tags (never :latest in production)
2. ✅ Configure readiness probes for safe updates
3. ✅ Set resource requests/limits for predictable behavior
4. ✅ Test updates in non-production environments first
5. ✅ Maintain revision history for rollback capability
6. ✅ Monitor rollout progress actively
7. ✅ Use declarative configuration (YAML in Git) for version control
8. ✅ Plan rollback strategy before every update

### Real-World Impact
- **Zero Downtime:** Applications remain available during updates
- **Safe Deployments:** Gradual rollout catches issues early
- **Easy Rollback:** Revert to previous version in seconds
- **Production Standard:** Industry best practice for updates
- **Operational Efficiency:** Automated, declarative, version-controlled

**Rolling updates are the foundation of modern continuous deployment practices, enabling teams to ship code changes safely and frequently to production Kubernetes clusters.**

🎉 **Day 51 Complete!** You've mastered Kubernetes rolling updates, a critical skill for maintaining production applications with zero downtime and safe deployment practices.
