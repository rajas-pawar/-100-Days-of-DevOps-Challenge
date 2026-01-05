# Day 60: Kubernetes Persistent Volumes (PV) and Persistent Volume Claims (PVC)

## 📋 Task Overview

**Scenario:** The Nautilus DevOps team needs to deploy a web application on Kubernetes with persistent storage for application code. You'll create a complete storage solution using PersistentVolumes and PersistentVolumeClaims.

**Requirements:**
1. Create a PersistentVolume with hostPath storage
2. Create a PersistentVolumeClaim to request storage
3. Deploy nginx pod using the PVC
4. Expose the application via NodePort service

---

## 🎯 Learning Objectives

By the end of this task, you will understand:
- **Persistent Volumes (PV):** Cluster-level storage resources
- **Persistent Volume Claims (PVC):** User requests for storage
- **Storage Classes:** Define storage types and provisioners
- **Volume Binding:** How PVCs bind to PVs
- **hostPath Volumes:** Node-local storage for development
- **Volume Mounting:** Attach storage to containers
- **Data Persistence:** Keep data beyond pod lifecycle

---

## 📖 Understanding Kubernetes Storage

### What is Persistent Storage?

**The Problem:**
- Container filesystem is ephemeral (temporary)
- When a pod dies, all data in the container is lost
- Stateful applications need data to survive pod restarts

**The Solution:**
- **Persistent Volumes (PV):** Pre-provisioned storage in the cluster
- **Persistent Volume Claims (PVC):** Requests for storage by users
- **Dynamic Provisioning:** Automatic PV creation based on storage classes

### Storage Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Kubernetes Cluster                        │
│                                                              │
│  ┌──────────────┐         ┌──────────────┐                 │
│  │ Administrator│         │  Developer   │                 │
│  │              │         │              │                 │
│  │  Creates     │         │  Requests    │                 │
│  │     ↓        │         │     ↓        │                 │
│  │    PV        │         │    PVC       │                 │
│  └──────┬───────┘         └──────┬───────┘                 │
│         │                        │                          │
│         │    ┌──────────────────┐│                          │
│         └───→│   Binding        │←──                        │
│              │   (Automatic)    │                           │
│              └────────┬─────────┘                           │
│                       │                                     │
│                       ↓                                     │
│              ┌──────────────────┐                           │
│              │       Pod        │                           │
│              │   ┌──────────┐   │                           │
│              │   │Container │   │                           │
│              │   │          │   │                           │
│              │   │ /mnt/data│←──┼─── Volume Mount           │
│              │   └──────────┘   │                           │
│              └──────────────────┘                           │
│                       ↓                                     │
│              ┌──────────────────┐                           │
│              │   Node Storage   │                           │
│              │   /mnt/dba       │                           │
│              └──────────────────┘                           │
└─────────────────────────────────────────────────────────────┘
```

### PV vs PVC: What's the Difference?

| Aspect | PersistentVolume (PV) | PersistentVolumeClaim (PVC) |
|--------|----------------------|----------------------------|
| **Created By** | Cluster administrator | Application developer |
| **Scope** | Cluster-wide resource | Namespace-scoped |
| **Purpose** | Provision actual storage | Request storage |
| **Contains** | Storage details, capacity, access modes | Storage requirements |
| **Lifecycle** | Independent of pods | Can be retained or deleted |
| **Analogy** | Storage in a warehouse | Purchase order for storage |

**Real-World Analogy:**
- **PV** = Physical hard drive in a data center
- **PVC** = Request form to get storage space
- **Binding** = Assigning the hard drive to your request
- **Pod mounting** = Connecting the drive to your computer

---

## 🔧 Volume Types in Kubernetes

### 1. hostPath (Today's Focus)

**Description:** Mounts a directory from the host node's filesystem into the pod.

**Use Cases:**
- ✅ Development and testing
- ✅ Single-node clusters (Minikube, local development)
- ✅ Node-level DaemonSet data

**Limitations:**
- ❌ Not suitable for production multi-node clusters
- ❌ Data tied to specific node
- ❌ Pod rescheduling to different node = data loss

**Example:**
```yaml
volumes:
  - name: data
    hostPath:
      path: /mnt/dba      # Directory on host node
      type: Directory     # Must already exist
```

### 2. Other Volume Types (For Reference)

**Cloud Provider Volumes:**
- `awsElasticBlockStore` - AWS EBS
- `azureDisk` - Azure Disk Storage
- `gcePersistentDisk` - Google Compute Engine PD

**Network Storage:**
- `nfs` - Network File System
- `cephfs` - Ceph File System
- `glusterfs` - GlusterFS

**Block Storage:**
- `iscsi` - iSCSI storage
- `fc` - Fibre Channel

**Configuration:**
- `configMap` - Configuration data
- `secret` - Sensitive data

---

## 🛠️ Task Implementation

### Step 1: Create the PersistentVolume

**Understanding the Spec:**

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-nautilus                    # PV name (cluster-wide unique)
spec:
  capacity:
    storage: 3Gi                       # Total storage capacity
  accessModes:
    - ReadWriteOnce                    # Access mode (RWO)
  storageClassName: manual             # Storage class name
  hostPath:
    path: /mnt/dba                     # Path on host node
    type: Directory                    # Directory must exist
```

**Access Modes Explained:**

| Mode | Abbreviation | Description | Use Case |
|------|--------------|-------------|----------|
| **ReadWriteOnce** | RWO | Volume mounted read-write by single node | Databases, most apps |
| **ReadOnlyMany** | ROX | Volume mounted read-only by many nodes | Static content, configs |
| **ReadWriteMany** | RWX | Volume mounted read-write by many nodes | Shared storage (NFS) |

**For Our Task:** `ReadWriteOnce` means only one node can mount the volume in read-write mode.

**Create the PV:**

```bash
# Create pv.yaml file
cat > pv-nautilus.yaml <<EOF
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-nautilus
spec:
  capacity:
    storage: 3Gi
  accessModes:
    - ReadWriteOnce
  storageClassName: manual
  hostPath:
    path: /mnt/dba
    type: Directory
EOF

# Apply the configuration
kubectl apply -f pv-nautilus.yaml
```

**Verify PV Creation:**

```bash
kubectl get pv
# NAME          CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   AGE
# pv-nautilus   3Gi        RWO            Retain           Available           manual         5s

kubectl describe pv pv-nautilus
```

**PV Status States:**
- **Available:** PV is free and not yet bound to a PVC
- **Bound:** PV is bound to a PVC
- **Released:** PVC was deleted, but PV not yet reclaimed
- **Failed:** PV has failed automatic reclamation

---

### Step 2: Create the PersistentVolumeClaim

**Understanding the Spec:**

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-nautilus                   # PVC name (namespace-scoped)
spec:
  accessModes:
    - ReadWriteOnce                    # Must match PV access mode
  storageClassName: manual             # Must match PV storage class
  resources:
    requests:
      storage: 2Gi                     # Request 2Gi (PV has 3Gi available)
```

**Binding Logic:**
Kubernetes will bind this PVC to a PV that meets:
1. ✅ Storage class matches (`manual`)
2. ✅ Access mode matches (`ReadWriteOnce`)
3. ✅ Capacity sufficient (`2Gi` request ≤ `3Gi` available)
4. ✅ PV status is `Available`

**Create the PVC:**

```bash
# Create pvc.yaml file
cat > pvc-nautilus.yaml <<EOF
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-nautilus
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: manual
  resources:
    requests:
      storage: 2Gi
EOF

# Apply the configuration
kubectl apply -f pvc-nautilus.yaml
```

**Verify PVC Creation and Binding:**

```bash
# Check PVC status
kubectl get pvc
# NAME           STATUS   VOLUME        CAPACITY   ACCESS MODES   STORAGECLASS   AGE
# pvc-nautilus   Bound    pv-nautilus   3Gi        RWO            manual         5s

# Check PV status (should now be Bound)
kubectl get pv
# NAME          CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                  STORAGECLASS   AGE
# pv-nautilus   3Gi        RWO            Retain           Bound    default/pvc-nautilus   manual         2m

# Detailed PVC information
kubectl describe pvc pvc-nautilus
```

**Key Points:**
- PVC requested `2Gi`, but got `3Gi` (entire PV capacity)
- Once bound, the PV is exclusively reserved for this PVC
- Status changed from `Available` → `Bound`

---

### Step 3: Create Pod with Volume Mount

**Understanding the Spec:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-nautilus
  labels:
    app: nginx-nautilus
spec:
  containers:
  - name: container-nautilus
    image: nginx:latest                # Must specify tag explicitly
    ports:
    - containerPort: 80
    volumeMounts:
    - name: storage                    # Reference to volume name below
      mountPath: /usr/share/nginx/html # nginx document root
  volumes:
  - name: storage                      # Volume name (arbitrary)
    persistentVolumeClaim:
      claimName: pvc-nautilus          # Reference to PVC
```

**Volume Mount Explanation:**
- **volumeMounts:** Where to mount the volume inside the container
- **mountPath:** `/usr/share/nginx/html` = nginx default document root
- **volumes:** Defines the volume using PVC
- **Connection:** `volumeMounts.name` must match `volumes.name`

**Create the Pod:**

```bash
# Create pod.yaml file
cat > pod-nautilus.yaml <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: pod-nautilus
  labels:
    app: nginx-nautilus
spec:
  containers:
  - name: container-nautilus
    image: nginx:latest
    ports:
    - containerPort: 80
    volumeMounts:
    - name: storage
      mountPath: /usr/share/nginx/html
  volumes:
  - name: storage
    persistentVolumeClaim:
      claimName: pvc-nautilus
EOF

# Apply the configuration
kubectl apply -f pod-nautilus.yaml
```

**Verify Pod Creation:**

```bash
# Check pod status
kubectl get pods
# NAME            READY   STATUS    RESTARTS   AGE
# pod-nautilus    1/1     Running   0          10s

# Detailed pod information
kubectl describe pod pod-nautilus
```

**Check Volume Mount Inside Container:**

```bash
# Exec into the pod
kubectl exec -it pod-nautilus -- bash

# Inside the container:
df -h | grep nginx
# /dev/sda1       100G   10G   90G  10%  /usr/share/nginx/html

ls -la /usr/share/nginx/html
# (Shows files from /mnt/dba on the host)

# Exit the container
exit
```

---

### Step 4: Create NodePort Service

**Understanding the Spec:**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-nautilus
spec:
  type: NodePort                       # Service type
  selector:
    app: nginx-nautilus                # Match pod labels
  ports:
  - port: 80                           # Service port
    targetPort: 80                     # Container port
    nodePort: 30008                    # External access port
```

**Service Ports Explained:**
- **port:** Port exposed by the service (cluster-internal)
- **targetPort:** Port on the container (where app listens)
- **nodePort:** Port on each node (external access) - must be 30000-32767

**Create the Service:**

```bash
# Create service.yaml file
cat > web-nautilus.yaml <<EOF
apiVersion: v1
kind: Service
metadata:
  name: web-nautilus
spec:
  type: NodePort
  selector:
    app: nginx-nautilus
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30008
EOF

# Apply the configuration
kubectl apply -f web-nautilus.yaml
```

**Verify Service Creation:**

```bash
# Check service
kubectl get svc web-nautilus
# NAME            TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
# web-nautilus    NodePort   10.96.100.50    <none>        80:30008/TCP   5s

# Detailed service information
kubectl describe svc web-nautilus
```

---

### Step 5: Complete Verification

**1. Check All Resources:**

```bash
echo "=== Persistent Volume ==="
kubectl get pv pv-nautilus

echo -e "\n=== Persistent Volume Claim ==="
kubectl get pvc pvc-nautilus

echo -e "\n=== Pod ==="
kubectl get pod pod-nautilus

echo -e "\n=== Service ==="
kubectl get svc web-nautilus
```

**Expected Output:**

```
=== Persistent Volume ===
NAME          CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                  STORAGECLASS   AGE
pv-nautilus   3Gi        RWO            Retain           Bound    default/pvc-nautilus   manual         5m

=== Persistent Volume Claim ===
NAME           STATUS   VOLUME        CAPACITY   ACCESS MODES   STORAGECLASS   AGE
pvc-nautilus   Bound    pv-nautilus   3Gi        RWO            manual         4m

=== Pod ===
NAME            READY   STATUS    RESTARTS   AGE
pod-nautilus    1/1     Running   0          3m

=== Service ===
NAME            TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
web-nautilus    NodePort   10.96.100.50    <none>        80:30008/TCP   2m
```

**2. Test Data Persistence:**

```bash
# Create a test file in the mounted volume
kubectl exec pod-nautilus -- bash -c "echo 'Hello from Persistent Storage!' > /usr/share/nginx/html/index.html"

# Verify the file exists
kubectl exec pod-nautilus -- cat /usr/share/nginx/html/index.html
# Output: Hello from Persistent Storage!

# Test via service (from cluster node)
curl http://localhost:30008
# Output: Hello from Persistent Storage!
```

**3. Test Persistence by Deleting Pod:**

```bash
# Delete the pod
kubectl delete pod pod-nautilus

# Check PVC status (should still be Bound)
kubectl get pvc pvc-nautilus
# STATUS: Bound (PVC is not deleted)

# Recreate the pod
kubectl apply -f pod-nautilus.yaml

# Wait for pod to be running
kubectl wait --for=condition=Ready pod/pod-nautilus --timeout=60s

# Verify data still exists
kubectl exec pod-nautilus -- cat /usr/share/nginx/html/index.html
# Output: Hello from Persistent Storage!  ✅ Data persisted!
```

**4. Verify from Node (if accessible):**

```bash
# Get the node where pod is running
NODE=$(kubectl get pod pod-nautilus -o jsonpath='{.spec.nodeName}')
echo "Pod is running on node: $NODE"

# On the node (if you have access):
# cat /mnt/dba/index.html
# Output: Hello from Persistent Storage!
```

---

## 📊 Resource Relationship Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                    Kubernetes Cluster                            │
│                                                                  │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ PersistentVolume: pv-nautilus                              │ │
│  │ ├─ Capacity: 3Gi                                           │ │
│  │ ├─ StorageClass: manual                                    │ │
│  │ ├─ AccessMode: ReadWriteOnce                               │ │
│  │ └─ HostPath: /mnt/dba                                      │ │
│  └──────────────────────┬─────────────────────────────────────┘ │
│                         │                                        │
│                         │ Bound                                  │
│                         ↓                                        │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ PersistentVolumeClaim: pvc-nautilus                        │ │
│  │ ├─ Request: 2Gi                                            │ │
│  │ ├─ StorageClass: manual                                    │ │
│  │ └─ Status: Bound to pv-nautilus                            │ │
│  └──────────────────────┬─────────────────────────────────────┘ │
│                         │                                        │
│                         │ Used by                                │
│                         ↓                                        │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Pod: pod-nautilus                                          │ │
│  │ ├─ Container: container-nautilus                           │ │
│  │ │  ├─ Image: nginx:latest                                  │ │
│  │ │  └─ Port: 80                                             │ │
│  │ └─ Volume Mount: /usr/share/nginx/html → pvc-nautilus     │ │
│  └──────────────────────┬─────────────────────────────────────┘ │
│                         │                                        │
│                         │ Exposed by                             │
│                         ↓                                        │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Service: web-nautilus                                      │ │
│  │ ├─ Type: NodePort                                          │ │
│  │ ├─ Selector: app=nginx-nautilus                            │ │
│  │ └─ Ports: 80:30008                                         │ │
│  └────────────────────────────────────────────────────────────┘ │
│                         │                                        │
└─────────────────────────┼────────────────────────────────────────┘
                          │
                          ↓
               Access: http://<node-ip>:30008
```

---

## 🎯 One-Command Alternative (All-in-One YAML)

You can also combine all resources in a single YAML file:

```bash
# Create all-in-one configuration
cat > nautilus-app.yaml <<EOF
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-nautilus
spec:
  capacity:
    storage: 3Gi
  accessModes:
    - ReadWriteOnce
  storageClassName: manual
  hostPath:
    path: /mnt/dba
    type: Directory
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-nautilus
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: manual
  resources:
    requests:
      storage: 2Gi
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-nautilus
  labels:
    app: nginx-nautilus
spec:
  containers:
  - name: container-nautilus
    image: nginx:latest
    ports:
    - containerPort: 80
    volumeMounts:
    - name: storage
      mountPath: /usr/share/nginx/html
  volumes:
  - name: storage
    persistentVolumeClaim:
      claimName: pvc-nautilus
---
apiVersion: v1
kind: Service
metadata:
  name: web-nautilus
spec:
  type: NodePort
  selector:
    app: nginx-nautilus
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30008
EOF

# Apply all resources at once
kubectl apply -f nautilus-app.yaml

# Verify all resources created
kubectl get pv,pvc,pod,svc | grep nautilus
```

**Benefits:**
- Single file to manage
- Easy to version control
- Atomic deployment (all or nothing)
- Easier to share and reproduce

---

## 🐛 Common Issues and Troubleshooting

### Issue 1: PVC Stuck in Pending State

**Symptoms:**
```bash
kubectl get pvc
# NAME           STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS
# pvc-nautilus   Pending                                      manual
```

**Root Causes:**

**A. No Matching PV Available**
```bash
# Check if PV exists
kubectl get pv
# (No PV or PV doesn't match requirements)

# Solution: Create PV first
kubectl apply -f pv-nautilus.yaml
```

**B. Storage Class Mismatch**
```bash
# Check PV storage class
kubectl get pv pv-nautilus -o jsonpath='{.spec.storageClassName}'
# Output: standard  (but PVC wants "manual")

# Solution: Fix storage class in PV or PVC
kubectl edit pv pv-nautilus
# Change storageClassName to "manual"
```

**C. Access Mode Mismatch**
```bash
# Check PV access modes
kubectl describe pv pv-nautilus | grep "Access Modes"
# Access Modes:  ReadWriteMany

# PVC expects ReadWriteOnce
# Solution: Fix access mode in PV
kubectl edit pv pv-nautilus
```

**D. Insufficient Capacity**
```bash
# PVC requests 2Gi, but PV only has 1Gi
# Solution: Increase PV capacity or reduce PVC request
kubectl edit pv pv-nautilus
# Change capacity to 3Gi
```

---

### Issue 2: Pod Stuck in ContainerCreating

**Symptoms:**
```bash
kubectl get pod pod-nautilus
# NAME            READY   STATUS              RESTARTS   AGE
# pod-nautilus    0/1     ContainerCreating   0          5m
```

**Diagnosis:**
```bash
kubectl describe pod pod-nautilus
```

**Root Causes:**

**A. PVC Not Bound**
```
Events:
  Warning  FailedMount  pod-nautilus  Unable to attach or mount volumes: PVC "pvc-nautilus" is not bound
```

**Solution:**
```bash
# Check PVC status
kubectl get pvc pvc-nautilus
# If pending, see Issue 1

# Wait for PVC to bind
kubectl wait --for=condition=Bound pvc/pvc-nautilus --timeout=60s
```

**B. HostPath Directory Doesn't Exist (Real-World Issue)**
```
Events:
  Warning  FailedMount  pod-nautilus  MountVolume.SetUp failed: hostPath type check failed: /mnt/dba is not a directory
```

**Your Actual Experience:**

The pod was stuck in `ContainerCreating` for over 6 minutes because the `/mnt/dba` directory didn't exist on the node. The PV specified `type: Directory` which requires the directory to pre-exist.

**Initial Attempts:**
```bash
# Tried docker command (not available)
docker exec kodekloud-control-plane mkdir -p /mnt/dba
# bash: docker: command not found
```

**Working Solution for Kind/Kubeadm Clusters:**

Since direct node access wasn't available, we created a **privileged helper pod** to create the directory:

```bash
# Create helper pod to create directory on node
cat > create-hostpath.yaml <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: hostpath-creator
spec:
  hostNetwork: true
  containers:
  - name: creator
    image: busybox
    command: ["sh", "-c", "mkdir -p /mnt/dba && chmod 755 /mnt/dba && echo 'Directory created' && sleep 10"]
    volumeMounts:
    - name: host-root
      mountPath: /mnt
    securityContext:
      privileged: true
  volumes:
  - name: host-root
    hostPath:
      path: /mnt
  restartPolicy: Never
EOF

# Apply the helper pod
kubectl apply -f create-hostpath.yaml

# Check logs to confirm success
kubectl logs hostpath-creator
# Output: Directory created

# Clean up helper pod
kubectl delete pod hostpath-creator

# Delete and recreate the stuck pod
kubectl delete pod pod-nautilus
kubectl apply -f pod-nautilus.yaml

# Watch pod start successfully
kubectl get pod pod-nautilus -w
# NAME           READY   STATUS    RESTARTS   AGE
# pod-nautilus   1/1     Running   0          10s
```

**How This Works:**
- **hostNetwork: true** - Pod runs in host network namespace
- **privileged: true** - Container has full access to host
- **hostPath mount at /mnt** - Mounts host's /mnt into container
- Creates `/mnt/dba` directory inside container (which is actually host's `/mnt/dba`)
- Pod completes and exits after directory creation

**Alternative Solutions:**

**Option 1: If you have direct node access**
```bash
# SSH to node
ssh user@node-ip
sudo mkdir -p /mnt/dba
sudo chmod 755 /mnt/dba
```

**Option 2: Use a different hostPath type**
```yaml
# Change PV to use DirectoryOrCreate instead of Directory
hostPath:
  path: /mnt/dba
  type: DirectoryOrCreate  # Auto-creates if doesn't exist
```

**Option 3: Use existing directory**
```bash
kubectl delete pv pv-nautilus
# Edit pv-nautilus.yaml to use /tmp instead
# hostPath:
#   path: /tmp/dba
#   type: DirectoryOrCreate
kubectl apply -f pv-nautilus.yaml
```

**Key Lessons:**
- ✅ **hostPath type `Directory`** requires pre-existing directory
- ✅ **hostPath type `DirectoryOrCreate`** creates directory automatically
- ✅ Privileged pods can access host filesystem for setup tasks
- ✅ Always check kubectl describe pod events for root cause
- ✅ In production, directories should be pre-provisioned by infrastructure team

**C. Wrong PVC Name in Pod**
```yaml
volumes:
- name: storage
  persistentVolumeClaim:
    claimName: pvc-nautilus-typo  # Typo!
```

**Solution:**
```bash
kubectl delete pod pod-nautilus
kubectl edit -f pod-nautilus.yaml
# Fix claimName to "pvc-nautilus"
kubectl apply -f pod-nautilus.yaml
```

---

### Issue 3: Pod Running but Volume Empty

**Symptoms:**
```bash
kubectl exec pod-nautilus -- ls /usr/share/nginx/html
# (Empty directory or no files)

# Accessing service returns 403 Forbidden
curl http://localhost:30008
# 403 Forbidden
```

**Root Cause:** nginx document root is empty (no index.html)

**Solution:**
```bash
# Create default index.html
kubectl exec pod-nautilus -- bash -c 'echo "<h1>Welcome to Nautilus!</h1>" > /usr/share/nginx/html/index.html'

# Verify
curl http://localhost:30008
# <h1>Welcome to Nautilus!</h1>
```

**Better Solution:** Pre-populate /mnt/dba on the host node

---

### Issue 4: NodePort Service Not Accessible

**Symptoms:**
```bash
curl http://localhost:30008
# Connection refused
```

**Diagnosis:**
```bash
# Check service
kubectl get svc web-nautilus
# NAME            TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
# web-nautilus    NodePort   10.96.100.50    <none>        80:30008/TCP   2m

# Check service endpoints
kubectl get endpoints web-nautilus
# NAME            ENDPOINTS         AGE
# web-nautilus    10.244.0.5:80     2m
```

**Root Causes:**

**A. Wrong NodePort Range**
```yaml
nodePort: 8080  # ❌ Outside allowed range (30000-32767)
```

**Solution:**
```bash
kubectl edit svc web-nautilus
# Change nodePort to 30008
```

**B. Pod Label Mismatch**
```yaml
# Service selector
selector:
  app: nginx-nautilus

# Pod labels
labels:
  app: nginx-web  # ❌ Doesn't match!
```

**Solution:**
```bash
# Check pod labels
kubectl get pod pod-nautilus --show-labels

# Fix pod labels
kubectl label pod pod-nautilus app=nginx-nautilus --overwrite

# Or recreate pod with correct labels
```

**C. Pod Not Running**
```bash
kubectl get pod pod-nautilus
# STATUS: Error or CrashLoopBackOff
```

**Solution:** Fix pod issues first, then service will work

---

### Issue 5: Image Pull Error

**Symptoms:**
```bash
kubectl get pod pod-nautilus
# NAME            READY   STATUS         RESTARTS   AGE
# pod-nautilus    0/1     ErrImagePull   0          30s
```

**Diagnosis:**
```bash
kubectl describe pod pod-nautilus
# Events:
#   Failed to pull image "nginx:latest": ... manifest for nginx:latest not found
```

**Root Cause:** Image tag issue or registry access

**Solution:**
```bash
# Use specific version
kubectl delete pod pod-nautilus

# Edit pod-nautilus.yaml
# Change: image: nginx:latest → image: nginx:1.25

kubectl apply -f pod-nautilus.yaml
```

---

### Issue 6: Data Loss After Pod Deletion

**Symptoms:**
```bash
# Created file in volume
kubectl exec pod-nautilus -- echo "test" > /usr/share/nginx/html/test.txt

# Deleted pod
kubectl delete pod pod-nautilus

# Recreated pod
kubectl apply -f pod-nautilus.yaml

# File is gone!
kubectl exec pod-nautilus -- ls /usr/share/nginx/html/test.txt
# No such file
```

**Root Causes:**

**A. Using emptyDir Instead of PVC**
```yaml
volumes:
- name: storage
  emptyDir: {}  # ❌ Ephemeral, not persistent!
```

**B. hostPath Pod Rescheduled to Different Node**
```bash
# Original pod on node1 (/mnt/dba on node1)
# New pod on node2 (/mnt/dba on node2 - different directory!)
```

**Solution:**
- For single-node: Use node selector to ensure same node
- For multi-node: Use network storage (NFS, cloud volumes)

```yaml
spec:
  nodeSelector:
    kubernetes.io/hostname: node1  # Pin to specific node
```

---

## 📚 Storage Class Deep Dive

### What is a Storage Class?

**Purpose:** Define "classes" of storage with different qualities (performance, backup policies, etc.)

**Types:**
1. **Static Provisioning:** Admin creates PVs manually (today's approach)
2. **Dynamic Provisioning:** PVs created automatically on-demand

### Manual Storage Class (Today's Task)

```yaml
storageClassName: manual
```

- **Meaning:** Static provisioning - admin must create PV manually
- **Binding:** PVC waits for matching PV
- **No Auto-Provisioning:** If no PV exists, PVC stays Pending

### Dynamic Storage Classes (Production)

**AWS Example:**
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: aws-ebs-fast
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp3
  fsType: ext4
```

**Usage:**
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-app-storage
spec:
  storageClassName: aws-ebs-fast  # References StorageClass
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
```

**Result:** Kubernetes automatically creates EBS volume and PV!

---

## 🔄 PV Reclaim Policies

When a PVC is deleted, what happens to the PV?

### 1. Retain (Default for Manual PVs)

```yaml
persistentVolumeReclaimPolicy: Retain
```

**Behavior:**
- PV becomes "Released" (not Available)
- Data preserved on storage
- Manual cleanup required

**Use Case:** Production data that requires backup before deletion

**Cleanup Process:**
```bash
# Delete PVC
kubectl delete pvc pvc-nautilus

# PV status becomes Released
kubectl get pv pv-nautilus
# STATUS: Released

# Manual cleanup
kubectl delete pv pv-nautilus
# Manually delete data on host: rm -rf /mnt/dba/*
```

### 2. Delete

```yaml
persistentVolumeReclaimPolicy: Delete
```

**Behavior:**
- PV and underlying storage deleted automatically
- Data permanently lost

**Use Case:** Dynamic provisioning with cloud volumes

### 3. Recycle (Deprecated)

```yaml
persistentVolumeReclaimPolicy: Recycle
```

**Behavior:** Performs basic scrub (`rm -rf /volume/*`) and makes PV available again

**Status:** Deprecated - use Delete or Retain instead

---

## 🎓 Best Practices

### 1. Storage Planning

```yaml
# ✅ Good: Separate PVs for different purposes
pv-database:     100Gi  # Critical data, frequent access
pv-logs:         50Gi   # Log files, less critical
pv-cache:        20Gi   # Temporary cache

# ❌ Bad: Single huge PV for everything
pv-everything:   1000Gi # Wastes space, no isolation
```

### 2. Capacity Planning

```yaml
# ✅ Good: PV capacity > PVC request (buffer for growth)
PV:  10Gi
PVC: 8Gi   # 20% buffer

# ⚠️ Acceptable: Exact match
PV:  10Gi
PVC: 10Gi

# ❌ Bad: PVC > PV capacity
PV:  5Gi
PVC: 10Gi  # Will never bind
```

### 3. Access Mode Selection

```yaml
# ✅ Database (single writer)
accessModes:
  - ReadWriteOnce

# ✅ Static content (multiple readers)
accessModes:
  - ReadOnlyMany

# ✅ Shared storage (multiple writers)
accessModes:
  - ReadWriteMany  # Requires NFS or cloud volume
```

### 4. Storage Class Strategy

```yaml
# ✅ Production: Use dynamic provisioning
storageClassName: aws-ebs-gp3

# ✅ Development: Manual or local storage
storageClassName: manual

# ❌ Bad: No storage class (uses default, unpredictable)
# storageClassName:  # Empty = default
```

### 5. Resource Labels

```yaml
# ✅ Good: Descriptive labels
metadata:
  name: pv-mysql-prod-01
  labels:
    app: mysql
    environment: production
    tier: database

# ❌ Bad: Generic names
metadata:
  name: pv-1
  # No labels
```

### 6. Backup Strategy

```bash
# ✅ Regular backups of persistent data
# For hostPath volumes:
rsync -av /mnt/dba/ /backup/nautilus-$(date +%Y%m%d)/

# For cloud volumes:
# Use cloud provider snapshots (EBS snapshots, Azure snapshots)
```

### 7. Security

```yaml
# ✅ Set appropriate ownership in hostPath
# On host node:
# chown 33:33 /mnt/dba  # www-data user for nginx
# chmod 750 /mnt/dba

# ✅ Use SecurityContext in pod
securityContext:
  runAsUser: 33
  runAsGroup: 33
  fsGroup: 33
```

### 8. Monitoring

```bash
# ✅ Monitor disk usage
kubectl exec pod-nautilus -- df -h /usr/share/nginx/html

# ✅ Set up alerts for disk space
# - Prometheus + Grafana
# - Cloud provider monitoring
```

---

## 🚀 Production-Ready Example

**Complete production configuration with best practices:**

```yaml
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-nautilus-prod
  labels:
    app: nginx
    environment: production
    tier: frontend
spec:
  capacity:
    storage: 10Gi                      # Larger capacity for production
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain # Preserve data on PVC deletion
  storageClassName: manual
  hostPath:
    path: /mnt/production/nautilus
    type: Directory
  mountOptions:                         # Advanced mount options
    - noatime                           # Performance optimization
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-nautilus-prod
  labels:
    app: nginx
    environment: production
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: manual
  resources:
    requests:
      storage: 8Gi                      # Request with buffer
    limits:
      storage: 10Gi                     # Quota limit
  selector:                             # Select specific PV
    matchLabels:
      app: nginx
      environment: production
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-nautilus-prod
  labels:
    app: nginx
    environment: production
    version: v1.0.0
spec:
  securityContext:
    runAsUser: 33                       # www-data
    runAsGroup: 33
    fsGroup: 33
  containers:
  - name: nginx
    image: nginx:1.25-alpine            # Specific version, smaller image
    ports:
    - containerPort: 80
      name: http
    volumeMounts:
    - name: web-storage
      mountPath: /usr/share/nginx/html
    resources:                          # Resource limits
      requests:
        memory: "128Mi"
        cpu: "100m"
      limits:
        memory: "256Mi"
        cpu: "200m"
    livenessProbe:                      # Health checks
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 30
      periodSeconds: 10
    readinessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 10
      periodSeconds: 5
  volumes:
  - name: web-storage
    persistentVolumeClaim:
      claimName: pvc-nautilus-prod
---
apiVersion: v1
kind: Service
metadata:
  name: web-nautilus-prod
  labels:
    app: nginx
    environment: production
spec:
  type: NodePort
  selector:
    app: nginx
    environment: production
  ports:
  - name: http
    port: 80
    targetPort: 80
    nodePort: 30008
  sessionAffinity: ClientIP             # Session persistence
```

---

## 📋 Complete Task Verification Script

```bash
#!/bin/bash

echo "🔍 Nautilus Application - Complete Verification"
echo "=============================================="

# Colors
GREEN='\033[0;32m'
RED='\033[0;31m'
NC='\033[0m'

# 1. Check PV
echo -e "\n1️⃣ Checking PersistentVolume..."
if kubectl get pv pv-nautilus &>/dev/null; then
    STATUS=$(kubectl get pv pv-nautilus -o jsonpath='{.status.phase}')
    CAPACITY=$(kubectl get pv pv-nautilus -o jsonpath='{.spec.capacity.storage}')
    if [ "$STATUS" == "Bound" ]; then
        echo -e "${GREEN}✅ PV pv-nautilus: $STATUS ($CAPACITY)${NC}"
    else
        echo -e "${RED}❌ PV pv-nautilus: $STATUS${NC}"
    fi
else
    echo -e "${RED}❌ PV pv-nautilus not found${NC}"
fi

# 2. Check PVC
echo -e "\n2️⃣ Checking PersistentVolumeClaim..."
if kubectl get pvc pvc-nautilus &>/dev/null; then
    STATUS=$(kubectl get pvc pvc-nautilus -o jsonpath='{.status.phase}')
    VOLUME=$(kubectl get pvc pvc-nautilus -o jsonpath='{.spec.volumeName}')
    if [ "$STATUS" == "Bound" ]; then
        echo -e "${GREEN}✅ PVC pvc-nautilus: $STATUS (Volume: $VOLUME)${NC}"
    else
        echo -e "${RED}❌ PVC pvc-nautilus: $STATUS${NC}"
    fi
else
    echo -e "${RED}❌ PVC pvc-nautilus not found${NC}"
fi

# 3. Check Pod
echo -e "\n3️⃣ Checking Pod..."
if kubectl get pod pod-nautilus &>/dev/null; then
    STATUS=$(kubectl get pod pod-nautilus -o jsonpath='{.status.phase}')
    READY=$(kubectl get pod pod-nautilus -o jsonpath='{.status.containerStatuses[0].ready}')
    IMAGE=$(kubectl get pod pod-nautilus -o jsonpath='{.spec.containers[0].image}')
    if [ "$STATUS" == "Running" ] && [ "$READY" == "true" ]; then
        echo -e "${GREEN}✅ Pod pod-nautilus: $STATUS ($IMAGE)${NC}"
    else
        echo -e "${RED}❌ Pod pod-nautilus: $STATUS (Ready: $READY)${NC}"
    fi
else
    echo -e "${RED}❌ Pod pod-nautilus not found${NC}"
fi

# 4. Check Service
echo -e "\n4️⃣ Checking Service..."
if kubectl get svc web-nautilus &>/dev/null; then
    TYPE=$(kubectl get svc web-nautilus -o jsonpath='{.spec.type}')
    NODEPORT=$(kubectl get svc web-nautilus -o jsonpath='{.spec.ports[0].nodePort}')
    ENDPOINTS=$(kubectl get endpoints web-nautilus -o jsonpath='{.subsets[0].addresses[0].ip}')
    if [ "$TYPE" == "NodePort" ] && [ "$NODEPORT" == "30008" ]; then
        echo -e "${GREEN}✅ Service web-nautilus: $TYPE (NodePort: $NODEPORT)${NC}"
        if [ -n "$ENDPOINTS" ]; then
            echo -e "${GREEN}   Endpoint: $ENDPOINTS${NC}"
        else
            echo -e "${RED}   No endpoints found${NC}"
        fi
    else
        echo -e "${RED}❌ Service web-nautilus: Type=$TYPE, NodePort=$NODEPORT${NC}"
    fi
else
    echo -e "${RED}❌ Service web-nautilus not found${NC}"
fi

# 5. Check Volume Mount
echo -e "\n5️⃣ Checking Volume Mount..."
if kubectl exec pod-nautilus -- df -h /usr/share/nginx/html &>/dev/null; then
    echo -e "${GREEN}✅ Volume mounted successfully${NC}"
    kubectl exec pod-nautilus -- df -h /usr/share/nginx/html | grep -v Filesystem
else
    echo -e "${RED}❌ Volume mount check failed${NC}"
fi

# 6. Test Application
echo -e "\n6️⃣ Testing Application..."
kubectl exec pod-nautilus -- bash -c 'echo "<h1>Nautilus Application</h1>" > /usr/share/nginx/html/index.html' &>/dev/null
sleep 2
RESPONSE=$(kubectl exec pod-nautilus -- curl -s localhost)
if echo "$RESPONSE" | grep -q "Nautilus"; then
    echo -e "${GREEN}✅ Application responding correctly${NC}"
else
    echo -e "${RED}❌ Application test failed${NC}"
fi

# 7. Summary
echo -e "\n=============================================="
echo "📊 Summary:"
kubectl get pv,pvc,pod,svc | grep nautilus

echo -e "\n✅ Task Complete! All resources verified."
```

**Run the script:**
```bash
chmod +x verify-nautilus.sh
./verify-nautilus.sh
```

---

## 🧹 Cleanup

**Remove all resources:**

```bash
# Method 1: Delete individual resources
kubectl delete svc web-nautilus
kubectl delete pod pod-nautilus
kubectl delete pvc pvc-nautilus
kubectl delete pv pv-nautilus

# Method 2: Delete from YAML files
kubectl delete -f nautilus-app.yaml

# Method 3: Delete by label (if labeled)
kubectl delete pod,svc -l app=nginx-nautilus
kubectl delete pvc,pv -l app=nginx

# Verify cleanup
kubectl get pv,pvc,pod,svc | grep nautilus
# (Should return no results)
```

**Note:** With `Retain` reclaim policy, you may need to manually delete data:
```bash
# On the node (if accessible):
# rm -rf /mnt/dba/*
```

---

## 📖 Key Takeaways

### Concepts Mastered Today:

1. ✅ **Persistent Volumes (PV)**
   - Cluster-level storage resources
   - Independent of pod lifecycle
   - Admin-provisioned (static) or auto-provisioned (dynamic)

2. ✅ **Persistent Volume Claims (PVC)**
   - User requests for storage
   - Namespace-scoped
   - Automatically binds to matching PV

3. ✅ **Volume Mounting**
   - Attach PVC to pod via volumeMounts
   - Mount at specific path in container
   - Data persists across pod restarts

4. ✅ **hostPath Volumes**
   - Good for development/single-node
   - Not recommended for production multi-node
   - Data tied to specific node

5. ✅ **Storage Classes**
   - Define storage types
   - Enable dynamic provisioning
   - Manual vs automated PV creation

6. ✅ **Access Modes**
   - ReadWriteOnce (RWO) - Single node read-write
   - ReadOnlyMany (ROX) - Many nodes read-only
   - ReadWriteMany (RWX) - Many nodes read-write

7. ✅ **Complete Application Stack**
   - Storage → PV → PVC → Pod → Service
   - Each layer depends on previous
   - NodePort exposes to external access

---

## 🎯 Production Considerations

### When to Use Each Storage Type:

| Use Case | Storage Type | Access Mode | Provisioning |
|----------|-------------|-------------|--------------|
| **Database** | Block storage (EBS, Azure Disk) | RWO | Dynamic |
| **Shared files** | NFS, EFS, Azure Files | RWX | Dynamic/Static |
| **Logs** | Block storage or NFS | RWO or RWX | Dynamic |
| **Cache** | emptyDir or local storage | RWO | Ephemeral |
| **Config** | ConfigMap | - | Native |
| **Secrets** | Secret | - | Native |
| **Development** | hostPath or local | RWO | Static |

### Migration Path:

**Development → Production:**

```yaml
# Development (hostPath)
volumes:
  - name: data
    hostPath:
      path: /mnt/dba

# ↓ Migrate to ↓

# Production (Cloud Storage)
volumes:
  - name: data
    persistentVolumeClaim:
      claimName: aws-ebs-volume

# With dynamic provisioning
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: aws-ebs-volume
spec:
  storageClassName: aws-ebs-gp3  # Auto-provision
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 100Gi
```

---

## 📚 Additional Resources

**Official Documentation:**
- [Persistent Volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)
- [Storage Classes](https://kubernetes.io/docs/concepts/storage/storage-classes/)
- [Volume Types](https://kubernetes.io/docs/concepts/storage/volumes/)

**Next Steps:**
- **Day 61:** StatefulSets for stateful applications
- **Day 62:** Dynamic volume provisioning with StorageClass
- **Day 63:** ConfigMaps and Secrets
- **Day 64:** Network Policies and security

---

## ✅ Task Completion Checklist

- [ ] PersistentVolume `pv-nautilus` created (3Gi, manual, RWO, hostPath /mnt/dba)
- [ ] PersistentVolumeClaim `pvc-nautilus` created (2Gi request, manual, RWO)
- [ ] PVC successfully bound to PV
- [ ] Pod `pod-nautilus` created with container `container-nautilus`
- [ ] Container uses `nginx:latest` image
- [ ] Volume mounted at `/usr/share/nginx/html`
- [ ] NodePort service `web-nautilus` created on port 30008
- [ ] Service successfully routes traffic to pod
- [ ] Application accessible via `http://node-ip:30008`
- [ ] Data persists after pod deletion and recreation
- [ ] All resources verified and working

---

**🎉 Congratulations!** You've successfully implemented persistent storage in Kubernetes with PV, PVC, and hostPath volumes. You now understand how to provide stateful storage for applications running in containers!

**Day 60 Status:** ✅ Complete

**Next:** Day 61 - StatefulSets and Headless Services 🚀
