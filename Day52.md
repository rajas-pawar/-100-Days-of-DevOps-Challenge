# Day 52: Kubernetes Rollback - Reverting to Previous Deployment Revision

## Objective
Perform a rollback of the `nginx-deployment` to the previous revision after a customer reported a bug in the newly deployed release. The Nautilus DevOps team needs to quickly revert to the last known working version to restore service quality.

## Task Requirements
- **Environment:** Kubernetes cluster accessible from jump_host
- **Deployment Name:** nginx-deployment
- **Action:** Rollback to previous revision
- **Reason:** Bug reported in current release
- **Success Criteria:** Deployment reverted to previous working version
- **Tool:** kubectl (pre-configured on jump_host)

## Understanding Kubernetes Rollback

### What is a Rollback?
A **rollback** is the process of reverting a Kubernetes deployment to a previous revision. When a new deployment causes issues (bugs, crashes, performance problems), rollback provides a quick way to restore the application to a known working state.

**Key Characteristics:**
- **Quick Recovery:** Revert to previous version in seconds
- **Revision-Based:** Uses Kubernetes revision history
- **Rolling Rollback:** Uses same rolling update mechanism (zero downtime)
- **Automatic:** Kubernetes handles pod recreation automatically
- **Safe Operation:** Previous configuration stored and validated

### Why Rollbacks are Critical

**Common Rollback Scenarios:**
1. **Bugs in New Release:** Application errors not caught in testing
2. **Performance Degradation:** New version slower than previous
3. **Configuration Errors:** Wrong environment variables or settings
4. **Compatibility Issues:** Breaking changes with dependencies
5. **Security Vulnerabilities:** Discovered after deployment
6. **Customer Impact:** User-facing issues requiring immediate fix

**Benefits:**
- **Minimize Downtime:** Restore service quickly
- **Reduce Customer Impact:** Fix issues before widespread complaints
- **Buy Time for Proper Fix:** Revert now, debug later
- **Production Safety Net:** Confidence to deploy knowing you can rollback
- **Incident Response:** Critical tool for production issues

## Revision History Deep Dive

### How Kubernetes Tracks Revisions

Every time you update a deployment, Kubernetes:
1. **Creates New ReplicaSet** with updated configuration
2. **Assigns Revision Number** (sequential: 1, 2, 3...)
3. **Stores Configuration** in ReplicaSet for rollback
4. **Maintains History** up to `revisionHistoryLimit` (default: 10)

**Revision Lifecycle:**
```
Revision 1: Initial deployment (nginx:1.16)
├── ReplicaSet: nginx-deployment-7d9c8f5b4c
├── Pods: 3 replicas
└── Status: Scaled to 0 (retained for rollback)

Revision 2: Updated deployment (nginx:1.17) ← Current
├── ReplicaSet: nginx-deployment-8c6f9d7a3b
├── Pods: 3 replicas
└── Status: Active (3 running pods)

Revision 3: Future update (nginx:1.18)
└── Not yet created
```

### What Gets Stored in Each Revision

Each revision stores:
- **Container Images:** Exact image tags
- **Container Configuration:** Environment variables, commands, args
- **Resource Specifications:** CPU/memory requests and limits
- **Volume Mounts:** Persistent storage configuration
- **Pod Template:** Labels, annotations, init containers
- **Update Strategy:** maxSurge, maxUnavailable settings

### Revision History Limit

```yaml
spec:
  revisionHistoryLimit: 10    # Keep last 10 revisions (default)
```

**Impact:**
- **Default 10:** Can rollback to any of last 10 deployments
- **Set to 0:** No rollback capability (old ReplicaSets deleted)
- **Set to 100:** More rollback options, more storage used
- **Production Recommendation:** Keep at least 10 revisions

## Rollback Process Explained

### How Rollback Works

When you execute rollback:

```
Current State: Revision 2 (nginx:1.17 - buggy version)
┌────────────────────────────────────────┐
│  Pod A     Pod B     Pod C             │
│  nginx:1.17 nginx:1.17 nginx:1.17      │
│  (Bug reported by customer)            │
└────────────────────────────────────────┘

Rollback Initiated: kubectl rollout undo
┌────────────────────────────────────────┐
│ Kubernetes retrieves Revision 1 config │
│ ReplicaSet: nginx-deployment-7d9c8f5b4c│
│ Image: nginx:1.16 (previous version)   │
└────────────────────────────────────────┘

Rolling Rollback in Progress:
┌────────────────────────────────────────┐
│  Pod B     Pod C     Pod D (new)       │
│  nginx:1.17 nginx:1.17 nginx:1.16      │
│  (Gradual replacement begins)          │
└────────────────────────────────────────┘

┌────────────────────────────────────────┐
│  Pod C     Pod D     Pod E (new)       │
│  nginx:1.17 nginx:1.16 nginx:1.16      │
│  (Continues rolling back)              │
└────────────────────────────────────────┘

Final State: Revision 3 (creates new revision from Rev 1 config)
┌────────────────────────────────────────┐
│  Pod D     Pod E     Pod F             │
│  nginx:1.16 nginx:1.16 nginx:1.16      │
│  (Rollback complete - bug resolved)    │
└────────────────────────────────────────┘
```

**Important:** Rollback creates a **new revision** (Revision 3) with the configuration from Revision 1. It does NOT delete Revision 2—history is preserved.

### Rollback vs New Deployment

| Aspect | Rollback | New Deployment |
|--------|----------|----------------|
| **Speed** | Immediate (uses stored config) | Requires creating new YAML/command |
| **Config Source** | Previous ReplicaSet | Manual specification |
| **Revision Number** | Creates new revision | Creates new revision |
| **Use Case** | Quick revert to known state | Intentional update |
| **Risk** | Low (tested configuration) | Higher (new configuration) |

## Infrastructure Overview

### Components Involved in Rollback

**Control Plane:**
- **API Server:** Receives rollback command
- **Controller Manager:** 
  - Deployment controller identifies target revision
  - Initiates rolling update with old configuration
- **etcd:** Retrieves stored revision configuration

**Worker Nodes:**
- **kubelet:** Creates pods with previous image version
- **Container Runtime:** Pulls previous image (may be cached)
- **kube-proxy:** Updates service routing to new pods

### Revision Storage

```
Deployment: nginx-deployment
│
├── ReplicaSet 1 (Revision 1) - replicas: 0
│   └── Configuration: nginx:1.16 ← Rollback target
│
├── ReplicaSet 2 (Revision 2) - replicas: 3 ← Current
│   └── Configuration: nginx:1.17 (buggy)
│
└── ReplicaSet 3 (After rollback) - replicas: 3
    └── Configuration: nginx:1.16 (from Rev 1)
```

## Step-by-Step Implementation

### Method 1: Rollback to Previous Revision (Recommended for Task)

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
# View nginx-deployment current state
kubectl get deployment nginx-deployment

# Expected Output:
# NAME               READY   UP-TO-DATE   AVAILABLE   AGE
# nginx-deployment   3/3     3            3           30m
```

**Note:** Even with a bug, deployment may show all pods ready if the bug doesn't cause crashes.

#### Step 3: Check Current Image Version (Problematic Version)
```bash
# View current container image
kubectl describe deployment nginx-deployment | grep Image

# Expected Output:
# Image:        nginx:1.17
# (This is the buggy version reported by customer)

# Alternative: Check in YAML format
kubectl get deployment nginx-deployment -o jsonpath='{.spec.template.spec.containers[0].image}'
```

#### Step 4: View Rollout History
```bash
# Check revision history
kubectl rollout history deployment/nginx-deployment

# Expected Output:
# deployment.apps/nginx-deployment
# REVISION  CHANGE-CAUSE
# 1         <none>
# 2         <none>
```

**Understanding Output:**
- **REVISION 1:** Initial deployment (working version)
- **REVISION 2:** Current deployment (buggy version) ← We're here
- **CHANGE-CAUSE:** Shows reason for change (if annotated)

#### Step 5: View Specific Revision Details (Optional but Recommended)
```bash
# Check what's in Revision 1 (rollback target)
kubectl rollout history deployment/nginx-deployment --revision=1

# Expected Output:
# deployment.apps/nginx-deployment with revision #1
# Pod Template:
#   Labels:       app=nginx
#                 pod-template-hash=7d9c8f5b4c
#   Containers:
#    nginx:
#     Image:      nginx:1.16
#     Port:       80/TCP
#     Host Port:  0/TCP
#     Environment:        <none>
#     Mounts:     <none>
#   Volumes:      <none>

# Check current Revision 2 (problematic version)
kubectl rollout history deployment/nginx-deployment --revision=2

# Expected Output:
# deployment.apps/nginx-deployment with revision #2
# Pod Template:
#   Labels:       app=nginx
#                 pod-template-hash=8c6f9d7a3b
#   Containers:
#    nginx:
#     Image:      nginx:1.17    ← Bug reported in this version
#     Port:       80/TCP
#     Host Port:  0/TCP
#     Environment:        <none>
#     Mounts:     <none>
#   Volumes:      <none>
```

**Why Check?**
- Verify you're rolling back to correct version
- Confirm image tag and configuration
- Ensure rollback target is truly the "working" version

#### Step 6: Check Current Pods Before Rollback
```bash
# List current pods with images
kubectl get pods -l app=nginx -o wide

# Expected Output:
# NAME                                READY   STATUS    RESTARTS   AGE   IP           NODE
# nginx-deployment-8c6f9d7a3b-abc12   1/1     Running   0          15m   10.244.1.5   node01
# nginx-deployment-8c6f9d7a3b-def34   1/1     Running   0          15m   10.244.2.3   node02
# nginx-deployment-8c6f9d7a3b-ghi56   1/1     Running   0          15m   10.244.1.6   node01

# Note the pod name suffix: 8c6f9d7a3b (ReplicaSet hash for Revision 2)
```

#### Step 7: Execute Rollback to Previous Revision
```bash
# Rollback to previous revision (Revision 1)
kubectl rollout undo deployment/nginx-deployment

# Expected Output:
# deployment.apps/nginx-deployment rolled back
```

**Command Breakdown:**
- `kubectl rollout undo`: Rollback command
- `deployment/nginx-deployment`: Target deployment
- No `--to-revision` flag = defaults to previous revision

**Alternative with explicit revision:**
```bash
# Explicitly specify previous revision (same result)
kubectl rollout undo deployment/nginx-deployment --to-revision=1
```

#### Step 8: Monitor Rollback Progress
```bash
# Watch rollback in real-time
kubectl rollout status deployment/nginx-deployment

# Expected Output (progressive):
# Waiting for deployment "nginx-deployment" rollout to finish: 1 out of 3 new replicas have been updated...
# Waiting for deployment "nginx-deployment" rollout to finish: 1 old replicas are pending termination...
# Waiting for deployment "nginx-deployment" rollout to finish: 2 of 3 updated replicas are available...
# deployment "nginx-deployment" successfully rolled out

# Alternative: Watch pods changing
kubectl get pods -l app=nginx -w
```

**What You'll See:**
1. New pods created with previous image (nginx:1.16)
2. New pods have different ReplicaSet hash (7d9c8f5b4c)
3. Old pods (8c6f9d7a3b) gradually terminated
4. Rolling rollback completes when all pods updated

#### Step 9: Verify Rollback Completion
```bash
# Check deployment status
kubectl get deployment nginx-deployment

# Expected Output:
# NAME               READY   UP-TO-DATE   AVAILABLE   AGE
# nginx-deployment   3/3     3            3           35m
#                    ^^^     ^^^          ^^^
#                     |       |            +-- All pods healthy
#                     |       +-- All pods updated (rolled back)
#                     +-- All desired replicas running
```

**Success Indicators:**
- READY: 3/3 (all replicas running)
- UP-TO-DATE: 3 (all pods rolled back)
- AVAILABLE: 3 (all pods passing health checks)

#### Step 10: Verify Rollback Image Version
```bash
# Check image in deployment
kubectl describe deployment nginx-deployment | grep Image

# Expected Output:
# Image:        nginx:1.16
# (Rollback successful - back to working version!)

# Verify in running pods
kubectl get pods -l app=nginx -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.spec.containers[0].image}{"\n"}{end}'

# Expected Output:
# nginx-deployment-7d9c8f5b4c-xyz01    nginx:1.16
# nginx-deployment-7d9c8f5b4c-xyz02    nginx:1.16
# nginx-deployment-7d9c8f5b4c-xyz03    nginx:1.16
```

**Verification:**
- Image is nginx:1.16 (previous working version)
- Pod names contain new hash (7d9c8f5b4c)
- All pods running same image version

#### Step 11: Check Updated Rollout History
```bash
# View revision history after rollback
kubectl rollout history deployment/nginx-deployment

# Expected Output:
# deployment.apps/nginx-deployment
# REVISION  CHANGE-CAUSE
# 2         <none>
# 3         <none>
```

**Important Observations:**
- **Revision 1 is GONE:** Consumed to create Revision 3
- **Revision 2 remains:** Buggy version kept in history
- **Revision 3 created:** New revision with Revision 1's configuration
- **Can rollback again:** To Revision 2 if needed (though unlikely)

#### Step 12: Verify ReplicaSets
```bash
# Check ReplicaSets status
kubectl get replicasets -l app=nginx

# Expected Output:
# NAME                          DESIRED   CURRENT   READY   AGE
# nginx-deployment-7d9c8f5b4c   3         3         3       40m   (Revision 3 - Active)
# nginx-deployment-8c6f9d7a3b   0         0         0       20m   (Revision 2 - Scaled down)
```

**Analysis:**
- Old ReplicaSet (7d9c8f5b4c) reactivated with 3 pods
- Buggy ReplicaSet (8c6f9d7a3b) scaled to 0
- Both ReplicaSets retained for potential future rollback

#### Step 13: Verify All Pods Running
```bash
# Check pod status
kubectl get pods -l app=nginx

# Expected Output:
# NAME                                READY   STATUS    RESTARTS   AGE
# nginx-deployment-7d9c8f5b4c-xyz01   1/1     Running   0          3m
# nginx-deployment-7d9c8f5b4c-xyz02   1/1     Running   0          3m
# nginx-deployment-7d9c8f5b4c-xyz03   1/1     Running   0          3m
```

**Verification:**
- All pods STATUS: Running
- All pods READY: 1/1
- Recent AGE (created during rollback)
- Pod names match active ReplicaSet

#### Step 14: Test Application Functionality
```bash
# If service exists, test application
kubectl get service nginx-service

# Test application response
curl http://<service-ip>:<port>

# Expected: Application works without the reported bug

# Check nginx version in pod (if applicable)
kubectl exec -it <pod-name> -- nginx -v

# Expected Output:
# nginx version: nginx/1.16.x
# (Previous working version)
```

#### Step 15: Verify Deployment Events
```bash
# Check deployment events for rollback confirmation
kubectl describe deployment nginx-deployment

# Look for events like:
# Events:
#   Type    Reason             Age   From                   Message
#   ----    ------             ----  ----                   -------
#   Normal  ScalingReplicaSet  5m    deployment-controller  Scaled up replica set nginx-deployment-7d9c8f5b4c to 1
#   Normal  ScalingReplicaSet  5m    deployment-controller  Scaled down replica set nginx-deployment-8c6f9d7a3b to 2
#   Normal  ScalingReplicaSet  4m    deployment-controller  Scaled up replica set nginx-deployment-7d9c8f5b4c to 2
#   Normal  ScalingReplicaSet  4m    deployment-controller  Scaled down replica set nginx-deployment-8c6f9d7a3b to 1
#   Normal  ScalingReplicaSet  4m    deployment-controller  Scaled up replica set nginx-deployment-7d9c8f5b4c to 3
#   Normal  ScalingReplicaSet  4m    deployment-controller  Scaled down replica set nginx-deployment-8c6f9d7a3b to 0
```

**Events Confirm:**
- Old ReplicaSet (7d9c8f5b4c) scaled up: 0 → 1 → 2 → 3
- Buggy ReplicaSet (8c6f9d7a3b) scaled down: 3 → 2 → 1 → 0
- Rollback executed successfully

### Method 2: Rollback to Specific Revision

#### Scenario: Need to rollback to a specific revision (not just previous)

```bash
# View all available revisions
kubectl rollout history deployment/nginx-deployment

# Output might show:
# REVISION  CHANGE-CAUSE
# 1         <none>
# 2         <none>
# 3         <none>
# 4         <none>  ← Current (buggy)

# Rollback to specific revision (e.g., Revision 2)
kubectl rollout undo deployment/nginx-deployment --to-revision=2

# Monitor rollback
kubectl rollout status deployment/nginx-deployment
```

**Use Cases:**
- Multiple deployments since last working version
- Need to skip intermediate versions
- Testing different historical versions
- Specific configuration from past revision

#### Step-by-Step for Specific Revision Rollback

```bash
# 1. Identify target revision
kubectl rollout history deployment/nginx-deployment --revision=2

# 2. Execute rollback to that revision
kubectl rollout undo deployment/nginx-deployment --to-revision=2

# 3. Verify (same as Method 1, Steps 8-15)
kubectl rollout status deployment/nginx-deployment
kubectl describe deployment nginx-deployment | grep Image
```

## Complete Command Summary

### Rollback Commands
```bash
# Rollback to previous revision (most common)
kubectl rollout undo deployment/<deployment-name>

# Rollback to specific revision
kubectl rollout undo deployment/<deployment-name> --to-revision=<number>

# Example: Rollback nginx-deployment to previous
kubectl rollout undo deployment/nginx-deployment

# Example: Rollback nginx-deployment to revision 3
kubectl rollout undo deployment/nginx-deployment --to-revision=3
```

### Viewing Revision History
```bash
# View all revisions
kubectl rollout history deployment/<deployment-name>

# View specific revision details
kubectl rollout history deployment/<deployment-name> --revision=<number>

# Example: View nginx-deployment history
kubectl rollout history deployment/nginx-deployment

# Example: View nginx-deployment revision 2 details
kubectl rollout history deployment/nginx-deployment --revision=2
```

### Monitoring Rollback
```bash
# Watch rollback progress
kubectl rollout status deployment/<deployment-name>

# Watch pods during rollback
kubectl get pods -l <label> -w

# Example: Monitor nginx-deployment rollback
kubectl rollout status deployment/nginx-deployment

# Example: Watch nginx pods
kubectl get pods -l app=nginx -w
```

### Verification Commands
```bash
# Check deployment status
kubectl get deployment <deployment-name>

# Check current image
kubectl describe deployment <deployment-name> | grep Image

# Check pod images
kubectl get pods -l <label> -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.spec.containers[0].image}{"\n"}{end}'

# Check ReplicaSets
kubectl get replicasets -l <label>

# Detailed deployment info
kubectl describe deployment <deployment-name>
```

### Pause/Resume Operations
```bash
# Pause rollout (stop in middle of rollback)
kubectl rollout pause deployment/<deployment-name>

# Resume rollout (continue paused rollback)
kubectl rollout resume deployment/<deployment-name>

# Restart deployment (recreate all pods with current config)
kubectl rollout restart deployment/<deployment-name>
```

## Kubectl Rollback Commands Reference

### Undo Operations
```bash
# Undo to previous revision
kubectl rollout undo deployment/<name>

# Undo to specific revision
kubectl rollout undo deployment/<name> --to-revision=<revision-number>

# Undo with dry-run (preview without executing)
kubectl rollout undo deployment/<name> --dry-run=client

# Undo in specific namespace
kubectl rollout undo deployment/<name> -n <namespace>
```

### History Management
```bash
# View rollout history
kubectl rollout history deployment/<name>

# View history with change causes
kubectl rollout history deployment/<name> --show-details

# View specific revision
kubectl rollout history deployment/<name> --revision=<number>

# View history in different output format
kubectl rollout history deployment/<name> -o yaml
```

### Status Checking
```bash
# Check rollout status
kubectl rollout status deployment/<name>

# Check status with timeout
kubectl rollout status deployment/<name> --timeout=5m

# Check status in watch mode
kubectl rollout status deployment/<name> -w

# Check status for specific revision
kubectl get deployment/<name> -o jsonpath='{.metadata.annotations.deployment\.kubernetes\.io/revision}'
```

### Pause/Resume
```bash
# Pause deployment rollout
kubectl rollout pause deployment/<name>

# Resume paused deployment
kubectl rollout resume deployment/<name>

# Check if deployment is paused
kubectl get deployment/<name> -o jsonpath='{.spec.paused}'
```

### Revision Configuration
```bash
# Set revision history limit
kubectl patch deployment/<name> -p '{"spec":{"revisionHistoryLimit":15}}'

# View current revision history limit
kubectl get deployment/<name> -o jsonpath='{.spec.revisionHistoryLimit}'

# Annotate deployment with change cause (for history tracking)
kubectl annotate deployment/<name> kubernetes.io/change-cause="Rollback due to customer bug report"
```

## Troubleshooting Rollback Issues

### Issue 1: No Revision History Available

**Symptoms:**
```bash
kubectl rollout undo deployment/nginx-deployment
# Error: no rollout history found for deployment "nginx-deployment"

kubectl rollout history deployment/nginx-deployment
# No revisions found
```

**Causes:**
- `revisionHistoryLimit` set to 0
- Deployment recently created (no previous revisions)
- Old ReplicaSets manually deleted

**Diagnosis:**
```bash
# Check revision history limit
kubectl get deployment nginx-deployment -o jsonpath='{.spec.revisionHistoryLimit}'

# Output: 0 or null

# Check ReplicaSets
kubectl get replicasets -l app=nginx

# Output: Only one ReplicaSet exists
```

**Solutions:**
```bash
# 1. Set revision history limit for future
kubectl patch deployment nginx-deployment -p '{"spec":{"revisionHistoryLimit":10}}'

# 2. If no history exists, cannot rollback
# Must manually deploy previous version

# Create YAML with previous configuration
kubectl get deployment nginx-deployment -o yaml > current-deployment.yaml

# Edit to change image to previous version
# vim current-deployment.yaml
# Change image: nginx:1.17 to image: nginx:1.16

# Apply updated configuration
kubectl apply -f current-deployment.yaml

# 3. For future, always maintain revision history
# Add to deployment manifest:
# spec:
#   revisionHistoryLimit: 10
```

### Issue 2: Rollback Hangs or Takes Too Long

**Symptoms:**
```bash
kubectl rollout status deployment/nginx-deployment
# Waiting for deployment "nginx-deployment" rollout to finish: 1 out of 3 new replicas have been updated...
# (Hangs for extended period)

kubectl get pods -l app=nginx
# NAME                                READY   STATUS             RESTARTS   AGE
# nginx-deployment-7d9c8f5b4c-xyz01   0/1     ImagePullBackOff   0          5m
```

**Causes:**
- Previous image no longer available (deleted from registry)
- Network issues pulling image
- Node resource constraints
- Image stored in private registry without credentials

**Diagnosis:**
```bash
# Check pod events
kubectl describe pod <pending-pod-name>

# Look for:
# Events:
#   Failed to pull image "nginx:1.16": rpc error: code = Unknown desc = Error response from daemon: manifest for nginx:1.16 not found

# Check node resources
kubectl top nodes

# Check if image needs authentication
kubectl get deployment nginx-deployment -o yaml | grep imagePullSecrets
```

**Solutions:**
```bash
# 1. If image doesn't exist, rollback to different revision
kubectl rollout history deployment/nginx-deployment
# Find revision with available image

kubectl rollout undo deployment/nginx-deployment --to-revision=<available-revision>

# 2. If image in private registry, add pull secret
kubectl create secret docker-registry regcred \
  --docker-server=<registry-url> \
  --docker-username=<username> \
  --docker-password=<password>

kubectl patch deployment nginx-deployment -p '{"spec":{"template":{"spec":{"imagePullSecrets":[{"name":"regcred"}]}}}}'

# 3. If nodes lack resources, scale down temporarily
kubectl scale deployment nginx-deployment --replicas=1
# Then rollback
kubectl rollout undo deployment nginx-deployment
# Scale back up
kubectl scale deployment nginx-deployment --replicas=3

# 4. Cancel stuck rollback and try manual update
kubectl rollout undo deployment/nginx-deployment --to-revision=<working-revision>
```

### Issue 3: Rollback Completes But App Still Broken

**Symptoms:**
```bash
kubectl get pods -l app=nginx
# All pods Running, READY 1/1

curl http://<service-ip>
# Still experiencing the bug OR different error
```

**Causes:**
- Bug exists in multiple versions (not just current)
- Configuration issue separate from image version
- Database migration can't be reverted
- External dependency changed
- Issue with ConfigMap or Secret (not rolled back automatically)

**Diagnosis:**
```bash
# Check which revision you rolled back to
kubectl rollout history deployment/nginx-deployment

# Check current image
kubectl describe deployment nginx-deployment | grep Image

# Check ConfigMaps and Secrets
kubectl get configmaps
kubectl get secrets

# Check application logs
kubectl logs -l app=nginx --tail=100

# Check specific revision that worked
kubectl rollout history deployment/nginx-deployment --revision=<older-revision>
```

**Solutions:**
```bash
# 1. Rollback to older revision (before bug introduced)
kubectl rollout undo deployment/nginx-deployment --to-revision=<older-working-revision>

# 2. Check and update ConfigMaps/Secrets separately
kubectl get configmap <config-name> -o yaml
# Edit and reapply if needed

# 3. For database issues, manual intervention needed
# Connect to database and revert migrations manually

# 4. Check external dependencies
# Ensure APIs, services unchanged

# 5. Rollback to known-good revision from before issue started
kubectl rollout history deployment/nginx-deployment
# Identify last definitely working revision
kubectl rollout undo deployment/nginx-deployment --to-revision=<that-revision>
```

### Issue 4: Wrong Revision Selected for Rollback

**Symptoms:**
```bash
kubectl rollout undo deployment/nginx-deployment --to-revision=2
# deployment.apps/nginx-deployment rolled back

# But this wasn't the working version either!
```

**Causes:**
- Didn't verify revision contents before rollback
- Assumed wrong revision number
- Multiple recent deployments with similar issues

**Diagnosis:**
```bash
# Check current state after bad rollback
kubectl rollout history deployment/nginx-deployment

# Check what's in each revision
kubectl rollout history deployment/nginx-deployment --revision=1
kubectl rollout history deployment/nginx-deployment --revision=2
kubectl rollout history deployment/nginx-deployment --revision=3
```

**Solutions:**
```bash
# 1. Rollback again to correct revision
kubectl rollout undo deployment/nginx-deployment --to-revision=<correct-revision>

# 2. Always verify revision before rollback (preventive)
# Before rollback:
kubectl rollout history deployment/nginx-deployment --revision=<target-revision>
# Verify image, config match expectations

# Then rollback:
kubectl rollout undo deployment/nginx-deployment --to-revision=<target-revision>

# 3. Use change-cause annotations (for future)
# When deploying:
kubectl set image deployment/nginx-deployment nginx=nginx:1.17 \
  --record=true  # Deprecated but useful
# Or manually annotate:
kubectl annotate deployment/nginx-deployment kubernetes.io/change-cause="Update to 1.17 for feature X"
```

### Issue 5: Rollback Creates More Revisions Than Expected

**Symptoms:**
```bash
kubectl rollout history deployment/nginx-deployment
# REVISION  CHANGE-CAUSE
# 5         <none>
# 6         <none>
# 7         <none>
# 8         <none>  ← After multiple rollbacks, many revisions accumulate
```

**Causes:**
- Each rollback creates new revision
- Multiple rollback attempts
- Normal behavior (not actually an issue)

**Understanding:**
```
Initial:     Rev 1 (nginx:1.16)
Update:      Rev 2 (nginx:1.17 - buggy)
Rollback:    Rev 3 (nginx:1.16 from Rev 1)
Rollback:    Rev 4 (nginx:1.17 from Rev 2 - oops!)
Rollback:    Rev 5 (nginx:1.16 from Rev 3)
```

**Solutions:**
```bash
# 1. This is normal Kubernetes behavior
# Each rollback IS a new deployment, hence new revision

# 2. Clean up old revisions by reducing limit
kubectl patch deployment nginx-deployment -p '{"spec":{"revisionHistoryLimit":5}}'

# 3. View revision details to find correct one
kubectl rollout history deployment/nginx-deployment --revision=<number>

# 4. Annotate deployments for clarity (preventive)
kubectl annotate deployment/nginx-deployment \
  kubernetes.io/change-cause="Rollback to nginx:1.16 due to bug in 1.17"

# 5. In future, verify once before rollback
# Check revision content:
kubectl rollout history deployment/nginx-deployment --revision=<target>
# Then rollback:
kubectl rollout undo deployment/nginx-deployment --to-revision=<target>
```

### Issue 6: Deployment Paused During Rollback

**Symptoms:**
```bash
kubectl rollout status deployment/nginx-deployment
# deployment "nginx-deployment" successfully rolled out

# But pods not updated:
kubectl get pods -l app=nginx
# Still showing old pods
```

**Causes:**
- Deployment was paused (manually or by automation)
- Rollback command accepted but not executing

**Diagnosis:**
```bash
# Check if deployment is paused
kubectl get deployment nginx-deployment -o jsonpath='{.spec.paused}'

# Output: true (deployment is paused)

# Check deployment status
kubectl get deployment nginx-deployment
# READY might not match UP-TO-DATE
```

**Solutions:**
```bash
# 1. Resume deployment to allow rollback
kubectl rollout resume deployment/nginx-deployment

# 2. Verify rollback proceeds
kubectl rollout status deployment/nginx-deployment

# 3. Check pods update
kubectl get pods -l app=nginx -w

# 4. For future, avoid pausing during critical operations
# Check pause status before rollback:
kubectl get deployment nginx-deployment -o jsonpath='{.spec.paused}'
```

### Issue 7: Rollback Fails Due to Admission Webhook

**Symptoms:**
```bash
kubectl rollout undo deployment/nginx-deployment
# Error from server: admission webhook "validate.deployment" denied the request: image nginx:1.16 is not approved
```

**Causes:**
- Admission controller blocking old image
- Policy enforcement (security scanning, image approval)
- Organization policies changed since original deployment

**Diagnosis:**
```bash
# Check admission webhooks
kubectl get validatingwebhookconfigurations
kubectl get mutatingwebhookconfigurations

# Check deployment for previous image
kubectl rollout history deployment/nginx-deployment --revision=1

# Check if image violates current policy
# (Organization-specific checks)
```

**Solutions:**
```bash
# 1. Temporarily disable webhook (if permitted)
kubectl delete validatingwebhookconfiguration <webhook-name>
# Perform rollback
kubectl rollout undo deployment/nginx-deployment
# Re-enable webhook
kubectl apply -f <webhook-config.yaml>

# 2. Get exception for old image (organization process)
# Contact security team for approval

# 3. Deploy different working version that passes validation
kubectl set image deployment/nginx-deployment nginx=nginx:1.16.1-approved

# 4. Update policy to allow rollbacks
# (Organization-specific configuration)

# 5. Use emergency override (if available)
kubectl annotate deployment/nginx-deployment \
  security.policy.io/override="emergency-rollback-customer-impact"
kubectl rollout undo deployment/nginx-deployment
```

## Advanced Rollback Scenarios

### Scenario 1: Rollback with Database Migrations

**Challenge:** Application v2.0 included database schema changes. Rolling back code without reverting database causes errors.

**Solution:**

```yaml
# Deployment with pre-stop hook for migration rollback
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-deployment
spec:
  template:
    spec:
      containers:
      - name: app
        image: myapp:2.0
        lifecycle:
          preStop:
            exec:
              command:
              - /bin/sh
              - -c
              - |
                if [ "$ROLLBACK_MIGRATIONS" = "true" ]; then
                  /app/migrate-down.sh
                fi
```

**Rollback Process:**
```bash
# 1. Set environment variable to trigger migration rollback
kubectl set env deployment/app-deployment ROLLBACK_MIGRATIONS=true

# 2. Execute rollback
kubectl rollout undo deployment/app-deployment

# 3. Monitor migration logs
kubectl logs -l app=myapp --tail=100 -f

# 4. Verify database schema reverted
kubectl exec -it <db-pod> -- psql -U user -d mydb -c '\d'

# 5. Remove rollback flag
kubectl set env deployment/app-deployment ROLLBACK_MIGRATIONS-
```

### Scenario 2: Canary Rollback

**Challenge:** Canary deployment (10% traffic) shows issues. Need to rollback canary without affecting main deployment.

**Solution:**

```bash
# Main deployment (90% traffic)
# Name: app-main, replicas: 9

# Canary deployment (10% traffic)
# Name: app-canary, replicas: 1

# Rollback only canary
kubectl rollout undo deployment/app-canary

# Or delete canary entirely
kubectl delete deployment app-canary

# Main deployment unaffected
kubectl get deployment app-main
# Still running stable version
```

### Scenario 3: Multi-Service Rollback

**Challenge:** Microservices deployment with 5 services. One service has issue, but services are interdependent.

**Solution:**

```bash
# Identify problematic service
kubectl rollout history deployment/user-service
kubectl rollout history deployment/auth-service
kubectl rollout history deployment/payment-service
# etc.

# Rollback problematic service first
kubectl rollout undo deployment/payment-service

# Check if other services compatible
kubectl logs -l app=user-service --tail=50
# Look for communication errors

# Rollback dependent services if needed
kubectl rollout undo deployment/user-service --to-revision=5
kubectl rollout undo deployment/order-service --to-revision=3

# Verify inter-service communication
kubectl exec -it <user-service-pod> -- curl http://payment-service:8080/health
```

### Scenario 4: Rollback with ConfigMap/Secret Changes

**Challenge:** Deployment rollback successful, but ConfigMap wasn't rolled back automatically.

**Solution:**

```bash
# Check ConfigMap history (manual tracking needed)
kubectl get configmap app-config -o yaml

# If you have backup YAML files:
git log -- configmap-v1.yaml
git checkout <commit-hash> -- configmap-v1.yaml

# Apply old ConfigMap
kubectl apply -f configmap-v1.yaml

# Restart deployment to pick up ConfigMap changes
kubectl rollout restart deployment/nginx-deployment

# Alternative: Use version-tagged ConfigMaps
# configmap-v1, configmap-v2, etc.
# Deployment references specific version

# In deployment:
spec:
  template:
    spec:
      volumes:
      - name: config
        configMap:
          name: app-config-v1  # Explicit version
```

**Best Practice:**
```yaml
# Version ConfigMaps and Secrets
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config-v2  # Include version in name
data:
  config.yaml: |
    version: "2.0"
    feature_x: enabled

---
# Deployment references specific version
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-deployment
spec:
  template:
    spec:
      containers:
      - name: app
        image: myapp:2.0
        envFrom:
        - configMapRef:
            name: app-config-v2  # Explicit version reference
```

**Rollback Process:**
```bash
# Rollback deployment (automatically uses old ConfigMap version)
kubectl rollout undo deployment/app-deployment

# Deployment spec includes:
# configMapRef: app-config-v1
# No separate ConfigMap rollback needed
```

## Best Practices for Rollback

### 1. Always Maintain Revision History
❌ **Bad:**
```yaml
spec:
  revisionHistoryLimit: 0    # No rollback capability!
```

✅ **Good:**
```yaml
spec:
  revisionHistoryLimit: 10    # Keep last 10 revisions
```

**Why:**
- Enables quick rollback to any recent version
- Default 10 is usually sufficient
- Production minimum: 5 revisions

### 2. Verify Revision Before Rollback
❌ **Bad:**
```bash
# Blind rollback without checking
kubectl rollout undo deployment/nginx-deployment
```

✅ **Good:**
```bash
# Check what you're rolling back to
kubectl rollout history deployment/nginx-deployment --revision=1

# Verify image and configuration
# Then rollback
kubectl rollout undo deployment/nginx-deployment
```

**Why:**
- Ensures rollback target is correct version
- Prevents rolling back to wrong version
- Confirms expected configuration

### 3. Annotate Deployments with Change Cause
❌ **Bad:**
```bash
kubectl set image deployment/nginx-deployment nginx=nginx:1.17

kubectl rollout history deployment/nginx-deployment
# REVISION  CHANGE-CAUSE
# 1         <none>
# 2         <none>
# Hard to identify which revision is which!
```

✅ **Good:**
```bash
kubectl set image deployment/nginx-deployment nginx=nginx:1.17
kubectl annotate deployment/nginx-deployment \
  kubernetes.io/change-cause="Update to nginx 1.17 for security patch CVE-2023-xxxx"

kubectl rollout history deployment/nginx-deployment
# REVISION  CHANGE-CAUSE
# 1         Initial deployment nginx 1.16
# 2         Update to nginx 1.17 for security patch CVE-2023-xxxx
# Easy to identify!
```

**Why:**
- Clear history of changes
- Easy to identify working versions
- Better incident response

### 4. Test Rollback in Non-Production
```bash
# Test rollback procedure in staging
kubectl rollout undo deployment/nginx-deployment --namespace=staging

# Verify application works
curl http://<staging-service-ip>

# Monitor for issues
kubectl logs -l app=nginx -n staging --tail=50

# If successful, apply to production
kubectl rollout undo deployment/nginx-deployment --namespace=production
```

### 5. Monitor Rollback Progress Actively
❌ **Bad:**
```bash
kubectl rollout undo deployment/nginx-deployment
# Walk away, assume it worked
```

✅ **Good:**
```bash
kubectl rollout undo deployment/nginx-deployment

# Watch rollback progress
kubectl rollout status deployment/nginx-deployment -w

# Monitor pods
kubectl get pods -l app=nginx -w

# Check for errors
kubectl describe deployment nginx-deployment

# Verify application functionality
curl http://<service-ip>
```

### 6. Have Rollback Plan Before Deployment
```bash
# Before deploying new version:

# 1. Document current state
kubectl get deployment nginx-deployment -o yaml > backup-pre-deployment.yaml
kubectl rollout history deployment/nginx-deployment > revision-history-pre.txt

# 2. Note current revision number
CURRENT_REV=$(kubectl get deployment nginx-deployment -o jsonpath='{.metadata.annotations.deployment\.kubernetes\.io/revision}')
echo "Current revision: $CURRENT_REV"

# 3. Deploy new version
kubectl set image deployment/nginx-deployment nginx=nginx:1.17

# 4. If issues, immediate rollback
kubectl rollout undo deployment/nginx-deployment
```

### 7. Use Blue-Green or Canary for Critical Apps
```bash
# Instead of direct rollback, use safer deployment strategies

# Blue-Green: Run both versions, switch traffic instantly
kubectl patch service myapp -p '{"spec":{"selector":{"version":"blue"}}}'  # Rollback
kubectl patch service myapp -p '{"spec":{"selector":{"version":"green"}}}' # Deploy

# Canary: Test with small traffic percentage first
# If canary fails, delete it (main deployment unaffected)
kubectl delete deployment myapp-canary
```

### 8. Automate Rollback with Monitoring
```yaml
# Use tools like Flagger for automated rollback based on metrics
apiVersion: flagger.app/v1beta1
kind: Canary
metadata:
  name: nginx
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx-deployment
  analysis:
    metrics:
    - name: error-rate
      thresholdRange:
        max: 5  # If error rate > 5%, automatic rollback
    - name: latency
      thresholdRange:
        max: 500  # If latency > 500ms, automatic rollback
```

### 9. Version ConfigMaps and Secrets
```yaml
# Include version in ConfigMap name
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config-v3  # Version in name
data:
  config.yaml: |
    # Configuration data

---
# Deployment references versioned ConfigMap
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app
spec:
  template:
    spec:
      volumes:
      - name: config
        configMap:
          name: app-config-v3  # Rolls back automatically with deployment
```

### 10. Document Known-Good Revisions
```bash
# Maintain documentation of working revisions
# In git, wiki, or ticket system

# Example documentation:
# Revision 5: nginx:1.16.1 - Stable, production-tested
# Revision 6: nginx:1.17.0 - Bug reported, do not use
# Revision 7: nginx:1.16.1 - Rollback from Rev 6
# Revision 8: nginx:1.17.1 - Bug fixed, safe to use

# Quick reference during incidents
kubectl rollout undo deployment/nginx-deployment --to-revision=5  # Known good
```

## Real-World Rollback Examples

### Example 1: API Service Rollback (Customer-Reported Bug)

**Scenario:** REST API deployed v2.1.0, customers report 500 errors on specific endpoint.

**Rollback Process:**
```bash
# 1. Verify current version (problematic)
kubectl describe deployment api-service | grep Image
# Image: api-service:2.1.0

# 2. Check revision history
kubectl rollout history deployment/api-service
# REVISION  CHANGE-CAUSE
# 7         Update to 2.0.5 for performance improvements
# 8         Update to 2.1.0 for new features (current - buggy)

# 3. Verify previous revision
kubectl rollout history deployment/api-service --revision=7
# Image: api-service:2.0.5 (last known working)

# 4. Execute immediate rollback
kubectl rollout undo deployment/api-service

# 5. Monitor rollback
kubectl rollout status deployment/api-service -w
# deployment "api-service" successfully rolled out

# 6. Verify service restored
kubectl get pods -l app=api-service
# All pods Running with image 2.0.5

# 7. Test endpoint
curl -X POST http://api-service:8080/api/problematic-endpoint
# {"status": "success"}  ← Bug resolved

# 8. Notify customers
# "Issue resolved, API restored to previous stable version"

# 9. Debug v2.1.0 in staging
kubectl set image deployment/api-service api=api-service:2.1.0 -n staging
# Investigate and fix bug
```

**Timeline:**
- T+0: Bug reported
- T+2min: Rollback initiated
- T+5min: Rollback complete, service restored
- T+10min: Customers notified
- Total downtime/impact: ~5 minutes

### Example 2: Frontend Rollback (Performance Regression)

**Scenario:** React app updated from v3.5 to v4.0, monitoring shows increased load times and memory usage.

**Rollback Process:**
```bash
# 1. Monitoring alert received
# Avg response time: 200ms → 2000ms (10x increase)
# Memory usage: 512MB → 2GB per pod

# 2. Check current deployment
kubectl get deployment frontend -o jsonpath='{.spec.template.spec.containers[0].image}'
# frontend:4.0.0

# 3. Decision: Immediate rollback (customer experience degraded)
kubectl rollout undo deployment/frontend

# 4. Watch rollback progress
kubectl get pods -l app=frontend -w
# frontend-5f7d8c9b-abc (4.0.0) Terminating
# frontend-3a6b4e2c-xyz (3.5.0) Running

# 5. Monitor metrics during rollback
kubectl top pods -l app=frontend
# Memory usage dropping: 2GB → 1.5GB → 1GB → 512MB

# 6. Verify performance restored
curl -w "@curl-format.txt" http://frontend-service
# time_total: 0.215s ← Back to normal

# 7. Check all pods rolled back
kubectl get pods -l app=frontend -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.spec.containers[0].image}{"\n"}{end}'
# All showing frontend:3.5.0

# 8. Update status page
# "Performance issue resolved, rollback completed"

# 9. Post-incident review
# Investigate why v4.0 had performance issues
# Add performance testing to CI/CD before production
```

**Metrics:**
- Before rollback: Response time 2000ms, Memory 2GB/pod
- After rollback: Response time 200ms, Memory 512MB/pod
- Rollback duration: 3 minutes
- Customer impact: Minimal (rolling rollback maintained availability)

### Example 3: Database-Connected Service Rollback

**Scenario:** Backend service v5.0 deployed with new ORM, causes database connection pool exhaustion.

**Rollback Process:**
```bash
# 1. Database alerts: Connection pool exhausted
# Connections: 100/100 (max), queries queuing

# 2. Check recent deployments
kubectl rollout history deployment/backend-service
# REVISION  CHANGE-CAUSE
# 12        Update ORM to v3.0 (current)
# 11        Add caching layer (stable)

# 3. Verify v5.0 has ORM change
kubectl rollout history deployment/backend-service --revision=12
# Image: backend-service:5.0.0
# Annotations: "Updated to ORM 3.0"

# 4. Immediate rollback decision
kubectl rollout undo deployment/backend-service

# 5. Monitor database connections during rollback
watch -n 1 'kubectl exec -it postgres-pod -- psql -U admin -c "SELECT count(*) FROM pg_stat_activity;"'
# Connections: 100 → 85 → 70 → 45 → 30 (healthy)

# 6. Verify backend pods healthy
kubectl get pods -l app=backend-service
# All Running with previous image

# 7. Check application logs
kubectl logs -l app=backend-service --tail=50
# No connection timeout errors

# 8. Verify image rolled back
kubectl describe deployment backend-service | grep Image
# Image: backend-service:4.8.2 (previous ORM version)

# 9. Test database operations
kubectl exec -it backend-service-pod -- curl localhost:8080/health/db
# {"database": "connected", "pool": "healthy"}

# 10. Root cause analysis
# ORM 3.0 doesn't close connections properly
# Fix in development, add connection monitoring
```

**Impact:**
- Database connection pool: Exhausted → Normal in 4 minutes
- Application errors: 50% error rate → 0% after rollback
- Zero data loss (connection issue, not data corruption)

### Example 4: Security Patch Rollback (Breaking Change)

**Scenario:** Security patch nginx:1.21.6 → nginx:1.21.7 causes TLS handshake failures with legacy clients.

**Rollback Process:**
```bash
# 1. Customer reports: "Cannot connect via HTTPS"
# Legacy clients (TLS 1.0) unable to connect

# 2. Verify recent deployment
kubectl rollout history deployment/nginx-ingress
# REVISION  CHANGE-CAUSE
# 45        Security patch nginx 1.21.7
# 44        Stable nginx 1.21.6

# 3. Confirm security patch timing matches reports
kubectl rollout history deployment/nginx-ingress --revision=45
# Deployed: 30 minutes ago
# Customer reports started: 25 minutes ago (correlation confirmed)

# 4. Difficult decision: Security vs. Availability
# Decision: Rollback to maintain availability, investigate TLS config

kubectl rollout undo deployment/nginx-ingress

# 5. Monitor connection success rate
kubectl logs -l app=nginx-ingress --tail=100 | grep "TLS handshake"
# Handshake failures dropping

# 6. Verify legacy clients can connect
curl -v --tlsv1.0 https://example.com
# TLS 1.0 connection succeeded (previously failed)

# 7. Temporary resolution
# Rollback complete, all clients can connect

# 8. Proper fix (don't ignore security patch!)
# A. Update nginx config to support TLS 1.0 in 1.21.7
kubectl edit configmap nginx-config
# Add: ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3;

# B. Re-deploy security patch
kubectl set image deployment/nginx-ingress nginx=nginx:1.21.7

# C. Test with legacy clients
curl -v --tlsv1.0 https://example.com
# Success!

# D. Plan to deprecate TLS 1.0 (notify customers)
```

**Lessons:**
- Security patches can have breaking changes
- Test in staging with various client configurations
- Balance security vs. availability
- Document client compatibility requirements

## Completion Checklist

Run these commands to verify successful rollback:

```bash
# 1. Deployment shows all replicas ready
kubectl get deployment nginx-deployment
# READY: X/X, UP-TO-DATE: X, AVAILABLE: X

# 2. All pods running with previous image
kubectl get pods -l app=nginx
# All STATUS: Running, READY: 1/1

# 3. Image version reverted in deployment
kubectl describe deployment nginx-deployment | grep Image
# Should show previous version (e.g., nginx:1.16)

# 4. Image version reverted in all pods
kubectl get pods -l app=nginx -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.spec.containers[0].image}{"\n"}{end}'
# All pods show previous image version

# 5. Rollout status successful
kubectl rollout status deployment/nginx-deployment
# "deployment successfully rolled out"

# 6. Revision history updated
kubectl rollout history deployment/nginx-deployment
# Shows new revision created from previous config

# 7. ReplicaSets show correct scaling
kubectl get replicasets -l app=nginx
# Old RS (previous image): DESIRED=3, CURRENT=3, READY=3
# New RS (buggy image): DESIRED=0, CURRENT=0, READY=0

# 8. No pods in error states
kubectl get pods -l app=nginx --field-selector=status.phase!=Running
# No resources found (all pods healthy)

# 9. Deployment events show rollback
kubectl describe deployment nginx-deployment | grep -A 10 Events
# Shows ScalingReplicaSet events for rollback

# 10. Application functionality verified
curl http://<service-ip>:<port>
# Application responds without reported bug

# 11. No error logs in pods
kubectl logs -l app=nginx --tail=50
# No critical errors related to bug
```

**All checks passing = Rollback successful! ✅**

## Summary

### What We Accomplished
1. ✅ Executed rollback of nginx-deployment to previous revision
2. ✅ Used `kubectl rollout undo` for quick revert
3. ✅ Monitored rollback progress with `kubectl rollout status`
4. ✅ Verified all pods running previous working version
5. ✅ Confirmed bug resolution and service restoration
6. ✅ Maintained revision history for future rollbacks

### Key Concepts Learned
- **Rollback Mechanism:** Kubernetes uses revision history stored in ReplicaSets
- **Rolling Rollback:** Same gradual process as rolling update (zero downtime)
- **Revision History:** Each update creates new revision, kept up to `revisionHistoryLimit`
- **New Revision Created:** Rollback creates new revision with old config (preserves history)
- **Quick Recovery:** Critical tool for production incident response
- **Revision Management:** Can rollback to any stored revision, not just previous

### Critical Commands
```bash
# Rollback to previous revision
kubectl rollout undo deployment/<name>

# Rollback to specific revision
kubectl rollout undo deployment/<name> --to-revision=<number>

# View revision history
kubectl rollout history deployment/<name>

# View specific revision details
kubectl rollout history deployment/<name> --revision=<number>

# Monitor rollback
kubectl rollout status deployment/<name>
```

### Production Best Practices
1. ✅ Maintain adequate revision history (minimum 10)
2. ✅ Verify revision contents before rollback
3. ✅ Annotate deployments with change causes
4. ✅ Test rollback procedures in non-production
5. ✅ Monitor rollback progress actively
6. ✅ Have rollback plan before every deployment
7. ✅ Version ConfigMaps and Secrets with deployments
8. ✅ Document known-good revisions
9. ✅ Use automated rollback with monitoring for critical apps
10. ✅ Balance speed vs. verification during incidents

### Real-World Impact
- **Rapid Incident Response:** Restore service in minutes
- **Customer Impact Minimized:** Quick resolution of reported bugs
- **Production Safety:** Confidence to deploy knowing rollback available
- **Zero Downtime:** Rolling rollback maintains availability
- **Buy Time for Fixes:** Revert immediately, debug properly later

**Rollback is your production safety net—understanding and practicing rollback procedures is essential for any team managing Kubernetes deployments.**

🎉 **Day 52 Complete!** You've mastered Kubernetes rollback, a critical skill for production incident response and maintaining service reliability.
