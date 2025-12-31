# Day 54: Kubernetes Shared Volumes - Multi-Container Pod with emptyDir

## Objective
Create a multi-container pod with shared storage using emptyDir volume. This demonstrates how containers within the same pod can share data by mounting the same volume at different paths.

## Task Requirements
- **Environment:** Kubernetes cluster accessible from jump_host
- **Pod Name:** volume-share-devops
- **Volume Type:** emptyDir
- **Volume Name:** volume-share
- **Containers:**
  - **Container 1:** volume-container-devops-1 (debian:latest, mounted at /tmp/blog)
  - **Container 2:** volume-container-devops-2 (debian:latest, mounted at /tmp/demo)
- **Test:** Create file in container 1, verify it's accessible in container 2
- **Tool:** kubectl (pre-configured on jump_host)

## Understanding Kubernetes Volumes

### What are Kubernetes Volumes?

**Volumes** in Kubernetes provide persistent or temporary storage that outlives individual containers. They solve several problems:

1. **Data Persistence:** Container file systems are ephemeral (lost on restart)
2. **Data Sharing:** Multiple containers in a pod need to share data
3. **External Storage:** Applications need access to external storage systems

### Volume Lifecycle

```
Pod Created → Volume Created → Containers Mount Volume → Share Data → Pod Deleted → Volume Deleted
```

### Volume vs Container Storage

**Without Volume (Container Storage):**
```
Container 1 File System (Isolated)
├── /tmp/file1.txt
└── /var/data/

Container 2 File System (Isolated)
├── /tmp/file2.txt
└── /var/data/
```
❌ **No sharing between containers**
❌ **Data lost on container restart**

**With Shared Volume:**
```
Shared Volume (volume-share)
├── file1.txt
└── file2.txt
    ↓ Mounted in both containers
Container 1: /tmp/blog → Shared Volume
Container 2: /tmp/demo → Shared Volume
```
✅ **Containers share same data**
✅ **Data persists during container restarts**
✅ **Data lost only when pod is deleted**

## Types of Kubernetes Volumes

### 1. emptyDir (Temporary Storage)

**Characteristics:**
- Created when pod is assigned to a node
- Initially empty (hence "emptyDir")
- Exists as long as pod is running on that node
- Deleted when pod is removed from node
- Shared among all containers in the pod

**Use Cases:**
- Scratch space for computation
- Temporary cache
- Sharing data between containers
- Checkpointing long computation

**Storage Location:**
- By default: Node's disk storage
- Can be RAM-backed: `emptyDir: { medium: "Memory" }`

**Example:**
```yaml
volumes:
- name: cache-volume
  emptyDir: {}
```

### 2. hostPath (Node Storage)

**Characteristics:**
- Mounts file or directory from host node's filesystem
- Data persists even after pod is deleted
- Tied to specific node

**Use Cases:**
- Access Docker internals (e.g., /var/lib/docker)
- Running cAdvisor (container advisor)
- Development/testing scenarios

**Example:**
```yaml
volumes:
- name: host-volume
  hostPath:
    path: /data
    type: Directory
```

### 3. persistentVolumeClaim (Persistent Storage)

**Characteristics:**
- Claims a PersistentVolume
- Data persists beyond pod lifecycle
- Can be reused by other pods
- Supports various storage backends

**Use Cases:**
- Database storage
- User uploads
- Application state
- Any data that must survive pod deletion

**Example:**
```yaml
volumes:
- name: my-pvc
  persistentVolumeClaim:
    claimName: my-claim
```

### 4. configMap (Configuration Data)

**Characteristics:**
- Exposes ConfigMap data as files
- Read-only mount
- Updates automatically (with delay)

**Use Cases:**
- Application configuration files
- Environment-specific settings
- Non-sensitive configuration data

**Example:**
```yaml
volumes:
- name: config-volume
  configMap:
    name: app-config
```

### 5. secret (Sensitive Data)

**Characteristics:**
- Exposes Secret data as files
- Base64 encoded
- More secure than ConfigMaps
- Stored in tmpfs (RAM)

**Use Cases:**
- Passwords
- API keys
- Certificates
- OAuth tokens

**Example:**
```yaml
volumes:
- name: secret-volume
  secret:
    secretName: my-secret
```

### 6. Cloud Storage Volumes

**AWS EBS:**
```yaml
volumes:
- name: aws-volume
  awsElasticBlockStore:
    volumeID: vol-12345678
    fsType: ext4
```

**Azure Disk:**
```yaml
volumes:
- name: azure-volume
  azureDisk:
    diskName: myDisk
    diskURI: /subscriptions/.../myDisk
```

**Google Persistent Disk:**
```yaml
volumes:
- name: gcp-volume
  gcePersistentDisk:
    pdName: my-disk
    fsType: ext4
```

## emptyDir Volume Deep Dive

### emptyDir Creation and Lifecycle

**Pod Creation:**
```bash
kubectl apply -f pod.yaml
# 1. Kubernetes schedules pod to a node
# 2. Kubelet creates emptyDir on node's disk
# 3. Path: /var/lib/kubelet/pods/<pod-uid>/volumes/kubernetes.io~empty-dir/<volume-name>
# 4. Containers mount this directory
```

**During Pod Lifetime:**
```bash
# All containers see same data
Container 1 writes to /tmp/blog/file.txt
    ↓ (writes to emptyDir)
Container 2 reads from /tmp/demo/file.txt
    ↓ (reads from same emptyDir)
Same file content!
```

**Pod Deletion:**
```bash
kubectl delete pod my-pod
# 1. Containers stopped
# 2. emptyDir deleted from node
# 3. All data in emptyDir is lost permanently
```

### emptyDir Storage Backends

**1. Disk-backed (Default):**
```yaml
volumes:
- name: my-volume
  emptyDir: {}
```
- Stored on node's disk
- Subject to disk I/O performance
- Larger capacity available
- Survives container restarts

**2. Memory-backed (tmpfs):**
```yaml
volumes:
- name: my-volume
  emptyDir:
    medium: "Memory"
```
- Stored in RAM (tmpfs)
- Very fast I/O
- Limited by node's memory
- Counts against container memory limit
- Lost on node reboot

**3. Size-limited:**
```yaml
volumes:
- name: my-volume
  emptyDir:
    sizeLimit: "1Gi"
```
- Limits maximum size
- Pod evicted if exceeded
- Good practice for resource control

### emptyDir vs Other Volume Types

| Feature | emptyDir | hostPath | PersistentVolume |
|---------|----------|----------|------------------|
| **Lifetime** | Pod lifetime | Survives pod | Survives pod |
| **Scope** | Single pod | Single node | Cluster-wide |
| **Sharing** | Between containers in pod | Between pods on same node | Between any pods |
| **Persistence** | Temporary | Persistent | Persistent |
| **Use Case** | Scratch space, inter-container sharing | Node-specific data | Application data |
| **Data Loss Risk** | High (pod deletion) | Medium (node issues) | Low (replicated) |

## Multi-Container Pods

### Why Multi-Container Pods?

**Sidecar Pattern:**
```
Main Container (App) + Sidecar Container (Log Shipper)
    ↓ Share Volume
Main writes logs → Shared Volume → Sidecar ships logs
```

**Ambassador Pattern:**
```
Main Container (App) + Ambassador Container (Proxy)
    ↓ Share Configuration
Main makes requests → Ambassador proxies → External Service
```

**Adapter Pattern:**
```
Main Container (App) + Adapter Container (Formatter)
    ↓ Share Data
Main generates data → Adapter formats → Monitoring System
```

### Container Communication in Pods

**1. Shared Volumes (File-based):**
```yaml
volumeMounts:
- name: shared-data
  mountPath: /data
```
✅ Simple file-based communication
✅ Good for logs, configs, temporary data
❌ No real-time communication

**2. Shared Network (localhost):**
```yaml
# Container 1 listens on localhost:8080
# Container 2 connects to localhost:8080
```
✅ Real-time communication
✅ Standard networking
✅ Low latency

**3. Shared Process Namespace:**
```yaml
spec:
  shareProcessNamespace: true
```
✅ Containers see each other's processes
✅ Can send signals between containers
⚠️ Security implications

## Step-by-Step Implementation

### Phase 1: Create the Pod YAML

#### Step 1: Create Pod Manifest

```bash
# Create the YAML file
cat <<EOF > volume-share-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: volume-share-devops
spec:
  containers:
  - name: volume-container-devops-1
    image: debian:latest
    command: ["/bin/sh"]
    args: ["-c", "sleep 10000"]
    volumeMounts:
    - name: volume-share
      mountPath: /tmp/blog
  - name: volume-container-devops-2
    image: debian:latest
    command: ["/bin/sh"]
    args: ["-c", "sleep 10000"]
    volumeMounts:
    - name: volume-share
      mountPath: /tmp/demo
  volumes:
  - name: volume-share
    emptyDir: {}
EOF
```

**YAML Breakdown:**

**Metadata Section:**
```yaml
metadata:
  name: volume-share-devops    # Pod name (must match requirement)
```

**First Container:**
```yaml
- name: volume-container-devops-1         # Container 1 name
  image: debian:latest                    # Debian with latest tag
  command: ["/bin/sh"]                    # Shell to run
  args: ["-c", "sleep 10000"]            # Keep container running
  volumeMounts:                           # Mount specification
  - name: volume-share                    # Reference to volume
    mountPath: /tmp/blog                  # Where to mount in container 1
```

**Second Container:**
```yaml
- name: volume-container-devops-2         # Container 2 name
  image: debian:latest                    # Same image
  command: ["/bin/sh"]                    # Shell to run
  args: ["-c", "sleep 10000"]            # Keep container running
  volumeMounts:                           # Mount specification
  - name: volume-share                    # Same volume (shared!)
    mountPath: /tmp/demo                  # Different path in container 2
```

**Volume Definition:**
```yaml
volumes:                                  # Pod-level volumes
- name: volume-share                      # Volume name
  emptyDir: {}                            # Type: emptyDir, default settings
```

**Key Points:**
- ✅ Two containers in one pod
- ✅ Both mount same volume (`volume-share`)
- ✅ Different mount paths (`/tmp/blog` vs `/tmp/demo`)
- ✅ `sleep 10000` keeps containers running
- ✅ `emptyDir: {}` creates temporary shared storage

#### Step 2: Validate YAML Syntax

```bash
# Check YAML is valid (dry-run)
kubectl apply -f volume-share-pod.yaml --dry-run=client

# Expected Output:
# pod/volume-share-devops created (dry run)

# If errors, check:
# - Indentation (YAML is space-sensitive)
# - Container names match requirements
# - Volume name referenced correctly
```

**Common YAML Errors:**

❌ **Wrong indentation:**
```yaml
  containers:
- name: container1    # Wrong! Should be indented under containers
```

✅ **Correct indentation:**
```yaml
  containers:
  - name: container1  # Correct: 2 spaces for list items
```

❌ **Missing volume reference:**
```yaml
volumeMounts:
- name: wrong-volume-name    # Doesn't match volumes section
  mountPath: /tmp/data
```

✅ **Correct volume reference:**
```yaml
volumeMounts:
- name: volume-share         # Matches volumes section
  mountPath: /tmp/data
```

### Phase 2: Deploy and Verify Pod

#### Step 3: Apply Pod Configuration

```bash
# Create the pod
kubectl apply -f volume-share-pod.yaml

# Expected Output:
# pod/volume-share-devops created
```

#### Step 4: Verify Pod is Running

```bash
# Check pod status
kubectl get pods

# Expected Output:
# NAME                  READY   STATUS    RESTARTS   AGE
# volume-share-devops   2/2     Running   0          10s
```

**Status Interpretation:**
- **READY: 2/2** → Both containers running ✅
- **READY: 1/2** → One container failed ❌
- **READY: 0/2** → Both containers failed ❌
- **STATUS: Running** → Pod is healthy ✅
- **STATUS: Pending** → Waiting to be scheduled
- **STATUS: CrashLoopBackOff** → Containers crashing repeatedly

```bash
# Get detailed pod information
kubectl get pod volume-share-devops -o wide

# Expected Output:
# NAME                  READY   STATUS    RESTARTS   AGE   IP           NODE
# volume-share-devops   2/2     Running   0          20s   10.244.0.5   node01
```

#### Step 5: Verify Containers are Running

```bash
# List containers in the pod
kubectl get pod volume-share-devops -o jsonpath='{.spec.containers[*].name}'

# Expected Output:
# volume-container-devops-1 volume-container-devops-2

# Check container statuses
kubectl get pod volume-share-devops -o jsonpath='{.status.containerStatuses[*].state}'

# Expected Output (both running):
# {"running":{"startedAt":"2025-12-31T10:00:00Z"}} {"running":{"startedAt":"2025-12-31T10:00:00Z"}}
```

#### Step 6: Describe Pod for Detailed Info

```bash
# Get comprehensive pod details
kubectl describe pod volume-share-devops

# Look for these sections:
```

**Expected Output Sections:**

**Containers:**
```
Containers:
  volume-container-devops-1:
    Image:          debian:latest
    State:          Running
    Mounts:
      /tmp/blog from volume-share (rw)
  volume-container-devops-2:
    Image:          debian:latest
    State:          Running
    Mounts:
      /tmp/demo from volume-share (rw)
```

**Volumes:**
```
Volumes:
  volume-share:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:     
    SizeLimit:  <unset>
```

**Events:**
```
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  30s   default-scheduler  Successfully assigned default/volume-share-devops to node01
  Normal  Pulling    29s   kubelet            Pulling image "debian:latest"
  Normal  Pulled     25s   kubelet            Successfully pulled image "debian:latest"
  Normal  Created    25s   kubelet            Created container volume-container-devops-1
  Normal  Started    24s   kubelet            Started container volume-container-devops-1
  Normal  Pulled     23s   kubelet            Successfully pulled image "debian:latest"
  Normal  Created    23s   kubelet            Created container volume-container-devops-2
  Normal  Started    22s   kubelet            Started container volume-container-devops-2
```

### Phase 3: Test Volume Sharing

#### Step 7: Access First Container

```bash
# Exec into first container
kubectl exec -it volume-share-devops -c volume-container-devops-1 -- /bin/bash

# You'll see a prompt like:
# root@volume-share-devops:/#
```

**Command Breakdown:**
- `kubectl exec` - Execute command in container
- `-it` - Interactive terminal
- `volume-share-devops` - Pod name
- `-c volume-container-devops-1` - Specific container (required for multi-container pods)
- `--` - Separator between kubectl args and container command
- `/bin/bash` - Command to run (bash shell)

#### Step 8: Verify Mount Point in First Container

```bash
# Inside container 1, check mount point exists
ls -la /tmp/blog

# Expected Output:
# total 8
# drwxrwxrwx 2 root root 4096 Dec 31 10:00 .
# drwxrwxrwt 3 root root 4096 Dec 31 10:00 ..

# Verify it's empty initially
ls -la /tmp/blog/

# Expected Output:
# total 8
# drwxrwxrwx 2 root root 4096 Dec 31 10:00 .
# drwxrwxrwt 3 root root 4096 Dec 31 10:00 ..
# (No files yet)
```

#### Step 9: Create Test File in First Container

```bash
# Still inside container 1
# Create a test file with content
echo "Hello from Container 1! This is shared storage." > /tmp/blog/blog.txt

# Verify file created
cat /tmp/blog/blog.txt

# Expected Output:
# Hello from Container 1! This is shared storage.

# Check file details
ls -lh /tmp/blog/blog.txt

# Expected Output:
# -rw-r--r-- 1 root root 48 Dec 31 10:05 /tmp/blog/blog.txt

# Exit container 1
exit
```

**Alternative: One-liner without entering container:**
```bash
# Create file without interactive shell
kubectl exec volume-share-devops -c volume-container-devops-1 -- /bin/bash -c "echo 'Hello from Container 1!' > /tmp/blog/blog.txt"

# Verify file created
kubectl exec volume-share-devops -c volume-container-devops-1 -- cat /tmp/blog/blog.txt

# Expected Output:
# Hello from Container 1!
```

#### Step 10: Access Second Container

```bash
# Exec into second container
kubectl exec -it volume-share-devops -c volume-container-devops-2 -- /bin/bash

# You'll see a prompt like:
# root@volume-share-devops:/#
```

#### Step 11: Verify File Exists in Second Container

```bash
# Inside container 2, check the different mount point
ls -la /tmp/demo

# Expected Output:
# total 12
# drwxrwxrwx 2 root root 4096 Dec 31 10:05 .
# drwxrwxrwt 3 root root 4096 Dec 31 10:00 ..
# -rw-r--r-- 1 root root   48 Dec 31 10:05 blog.txt    ← File created in container 1!

# Read the file
cat /tmp/demo/blog.txt

# Expected Output:
# Hello from Container 1! This is shared storage.
```

**✅ Success! The file created in container 1 at `/tmp/blog/blog.txt` is visible in container 2 at `/tmp/demo/blog.txt`**

#### Step 12: Test Bidirectional Sharing

```bash
# Still inside container 2
# Create another file from container 2
echo "Response from Container 2! Sharing works both ways." > /tmp/demo/response.txt

# List files
ls -lh /tmp/demo/

# Expected Output:
# -rw-r--r-- 1 root root 48 Dec 31 10:05 blog.txt
# -rw-r--r-- 1 root root 54 Dec 31 10:06 response.txt

# Exit container 2
exit
```

#### Step 13: Verify Bidirectional Sharing

```bash
# Check from container 1 (should see both files)
kubectl exec volume-share-devops -c volume-container-devops-1 -- ls -lh /tmp/blog/

# Expected Output:
# -rw-r--r-- 1 root root 48 Dec 31 10:05 blog.txt        ← Created by container 1
# -rw-r--r-- 1 root root 54 Dec 31 10:06 response.txt    ← Created by container 2

# Read file created by container 2
kubectl exec volume-share-devops -c volume-container-devops-1 -- cat /tmp/blog/response.txt

# Expected Output:
# Response from Container 2! Sharing works both ways.
```

**✅ Both containers can read and write to the shared volume!**

### Phase 4: Additional Verification

#### Step 14: Verify Volume Type

```bash
# Check volume configuration
kubectl get pod volume-share-devops -o jsonpath='{.spec.volumes[*]}'

# Expected Output:
# {"emptyDir":{},"name":"volume-share"}

# More readable format
kubectl get pod volume-share-devops -o yaml | grep -A 5 "volumes:"

# Expected Output:
# volumes:
# - emptyDir: {}
#   name: volume-share
```

#### Step 15: Check Volume Mounts

```bash
# Check first container's mounts
kubectl get pod volume-share-devops -o jsonpath='{.spec.containers[0].volumeMounts[*]}'

# Expected Output:
# {"mountPath":"/tmp/blog","name":"volume-share"}

# Check second container's mounts
kubectl get pod volume-share-devops -o jsonpath='{.spec.containers[1].volumeMounts[*]}'

# Expected Output:
# {"mountPath":"/tmp/demo","name":"volume-share"}
```

#### Step 16: Verify emptyDir on Node (Advanced)

```bash
# Get node where pod is running
NODE=$(kubectl get pod volume-share-devops -o jsonpath='{.spec.nodeName}')
echo $NODE

# Get pod UID
POD_UID=$(kubectl get pod volume-share-devops -o jsonpath='{.metadata.uid}')
echo $POD_UID

# If you have node access (not in KodeKloud), you could check:
# ssh $NODE
# ls -la /var/lib/kubelet/pods/$POD_UID/volumes/kubernetes.io~empty-dir/volume-share/
# This is where emptyDir is actually stored on the node
```

#### Step 17: Test File Modification

```bash
# Modify file from container 1
kubectl exec volume-share-devops -c volume-container-devops-1 -- /bin/bash -c "echo 'Modified by container 1' >> /tmp/blog/blog.txt"

# Read from container 2 immediately
kubectl exec volume-share-devops -c volume-container-devops-2 -- cat /tmp/demo/blog.txt

# Expected Output:
# Hello from Container 1! This is shared storage.
# Modified by container 1
```

**✅ Changes are immediately visible across containers!**

#### Step 18: Test with Binary Files

```bash
# Create a binary file (simulate image/data)
kubectl exec volume-share-devops -c volume-container-devops-1 -- /bin/bash -c "dd if=/dev/urandom of=/tmp/blog/data.bin bs=1K count=10"

# Verify in container 2
kubectl exec volume-share-devops -c volume-container-devops-2 -- ls -lh /tmp/demo/data.bin

# Expected Output:
# -rw-r--r-- 1 root root 10K Dec 31 10:10 /tmp/demo/data.bin

# Compare checksums (should match)
kubectl exec volume-share-devops -c volume-container-devops-1 -- md5sum /tmp/blog/data.bin
kubectl exec volume-share-devops -c volume-container-devops-2 -- md5sum /tmp/demo/data.bin

# Both should output same MD5 hash
```

## Complete Command Summary

### Pod Creation Commands
```bash
# Create pod YAML file
cat <<EOF > volume-share-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: volume-share-devops
spec:
  containers:
  - name: volume-container-devops-1
    image: debian:latest
    command: ["/bin/sh"]
    args: ["-c", "sleep 10000"]
    volumeMounts:
    - name: volume-share
      mountPath: /tmp/blog
  - name: volume-container-devops-2
    image: debian:latest
    command: ["/bin/sh"]
    args: ["-c", "sleep 10000"]
    volumeMounts:
    - name: volume-share
      mountPath: /tmp/demo
  volumes:
  - name: volume-share
    emptyDir: {}
EOF

# Validate and apply
kubectl apply -f volume-share-pod.yaml --dry-run=client
kubectl apply -f volume-share-pod.yaml

# Verify pod is running
kubectl get pods
kubectl get pod volume-share-devops
kubectl describe pod volume-share-devops
```

### Volume Testing Commands
```bash
# Test from container 1 (create file)
kubectl exec -it volume-share-devops -c volume-container-devops-1 -- /bin/bash
echo "Hello from Container 1!" > /tmp/blog/blog.txt
cat /tmp/blog/blog.txt
exit

# Or one-liner:
kubectl exec volume-share-devops -c volume-container-devops-1 -- /bin/bash -c "echo 'Hello from Container 1!' > /tmp/blog/blog.txt"

# Verify from container 2 (read file)
kubectl exec -it volume-share-devops -c volume-container-devops-2 -- /bin/bash
ls -la /tmp/demo/
cat /tmp/demo/blog.txt
exit

# Or one-liner:
kubectl exec volume-share-devops -c volume-container-devops-2 -- cat /tmp/demo/blog.txt
kubectl exec volume-share-devops -c volume-container-devops-2 -- ls -lh /tmp/demo/
```

### Verification Commands
```bash
# Check pod status
kubectl get pod volume-share-devops -o wide

# List containers
kubectl get pod volume-share-devops -o jsonpath='{.spec.containers[*].name}'

# Check volumes
kubectl get pod volume-share-devops -o jsonpath='{.spec.volumes[*]}'

# Check volume mounts
kubectl get pod volume-share-devops -o jsonpath='{.spec.containers[*].volumeMounts}'

# View full pod YAML
kubectl get pod volume-share-devops -o yaml
```

### Cleanup Commands
```bash
# Delete the pod
kubectl delete pod volume-share-devops

# Verify deletion
kubectl get pods

# Delete YAML file (optional)
rm volume-share-pod.yaml
```

## Troubleshooting Common Issues

### Issue 1: Pod Stuck in Pending State

**Symptoms:**
```bash
kubectl get pods
# NAME                  READY   STATUS    RESTARTS   AGE
# volume-share-devops   0/2     Pending   0          2m
```

**Diagnosis:**
```bash
# Check pod events
kubectl describe pod volume-share-devops | grep -A 10 Events

# Common causes:
# - Insufficient resources on nodes
# - Image pull issues
# - Scheduling constraints
```

**Common Event Messages:**
```
FailedScheduling: 0/1 nodes are available: 1 Insufficient memory
FailedScheduling: 0/1 nodes are available: 1 node had untolerated taint
```

**Solution:**
```bash
# Check node resources
kubectl describe nodes | grep -A 5 "Allocated resources"

# Check for taints
kubectl describe nodes | grep Taints

# If resource issues, delete other pods or add nodes
# If taint issues, add tolerations to pod spec
```

### Issue 2: Container Not Starting (ImagePullBackOff)

**Symptoms:**
```bash
kubectl get pods
# NAME                  READY   STATUS             RESTARTS   AGE
# volume-share-devops   1/2     ImagePullBackOff   0          3m
```

**Diagnosis:**
```bash
# Check which container is failing
kubectl describe pod volume-share-devops

# Look for events like:
# Failed to pull image "debian:latest": rpc error: code = Unknown
```

**Causes:**
- Wrong image name or tag
- Network issues
- Registry authentication required
- Image doesn't exist

**Solution:**
```bash
# Verify image exists
docker pull debian:latest

# Check if image name in YAML is correct
kubectl get pod volume-share-devops -o jsonpath='{.spec.containers[*].image}'

# If wrong, fix YAML and reapply
kubectl delete pod volume-share-devops
kubectl apply -f volume-share-pod.yaml
```

### Issue 3: Container Exits Immediately (CrashLoopBackOff)

**Symptoms:**
```bash
kubectl get pods
# NAME                  READY   STATUS             RESTARTS   AGE
# volume-share-devops   1/2     CrashLoopBackOff   5          10m
```

**Diagnosis:**
```bash
# Check container logs
kubectl logs volume-share-devops -c volume-container-devops-1
kubectl logs volume-share-devops -c volume-container-devops-2

# Check previous container logs (if restarted)
kubectl logs volume-share-devops -c volume-container-devops-1 --previous
```

**Common Cause:**
Container has no long-running process (exits immediately)

**Example of Wrong Configuration:**
```yaml
containers:
- name: my-container
  image: debian:latest
  # No command - container exits immediately!
```

**Solution:**
Ensure sleep command is present:
```yaml
containers:
- name: volume-container-devops-1
  image: debian:latest
  command: ["/bin/sh"]
  args: ["-c", "sleep 10000"]    # Keeps container running
```

### Issue 4: File Not Visible in Second Container

**Symptoms:**
```bash
# File created in container 1
kubectl exec volume-share-devops -c volume-container-devops-1 -- ls /tmp/blog/
# blog.txt

# But not visible in container 2
kubectl exec volume-share-devops -c volume-container-devops-2 -- ls /tmp/demo/
# (empty)
```

**Diagnosis:**
```bash
# Check if containers are mounting same volume
kubectl get pod volume-share-devops -o yaml | grep -A 3 volumeMounts

# Check volume name matches
kubectl get pod volume-share-devops -o yaml | grep -A 2 volumes:
```

**Common Causes:**
1. Volume name mismatch in volumeMounts
2. Containers mounting different volumes
3. File created in wrong path

**Example of Wrong Configuration:**
```yaml
containers:
- name: container-1
  volumeMounts:
  - name: volume-share        # Correct name
    mountPath: /tmp/blog
- name: container-2
  volumeMounts:
  - name: volume-shared       # WRONG! Typo in name
    mountPath: /tmp/demo
```

**Solution:**
Ensure both containers reference same volume name:
```yaml
containers:
- name: container-1
  volumeMounts:
  - name: volume-share        # Same name
- name: container-2
  volumeMounts:
  - name: volume-share        # Same name
volumes:
- name: volume-share          # Matches both containers
  emptyDir: {}
```

### Issue 5: Permission Denied Errors

**Symptoms:**
```bash
kubectl exec volume-share-devops -c volume-container-devops-1 -- /bin/bash -c "echo 'test' > /tmp/blog/file.txt"
# /bin/bash: /tmp/blog/file.txt: Permission denied
```

**Diagnosis:**
```bash
# Check mount permissions
kubectl exec volume-share-devops -c volume-container-devops-1 -- ls -ld /tmp/blog
# Output: drwxr-xr-x 2 root root 4096 Dec 31 10:00 /tmp/blog
#         ↑ Read-only?

# Check if volume mounted as read-only
kubectl get pod volume-share-devops -o yaml | grep -A 2 volumeMounts
```

**Cause:**
Volume mounted as read-only:
```yaml
volumeMounts:
- name: volume-share
  mountPath: /tmp/blog
  readOnly: true              # ← Problem!
```

**Solution:**
Remove readOnly or set to false:
```yaml
volumeMounts:
- name: volume-share
  mountPath: /tmp/blog
  readOnly: false             # Or omit entirely (default is false)
```

### Issue 6: emptyDir Full (No Space Left)

**Symptoms:**
```bash
kubectl exec volume-share-devops -c volume-container-devops-1 -- /bin/bash -c "echo 'test' > /tmp/blog/file.txt"
# /bin/bash: write error: No space left on device
```

**Diagnosis:**
```bash
# Check disk usage in container
kubectl exec volume-share-devops -c volume-container-devops-1 -- df -h /tmp/blog

# Check if sizeLimit is set
kubectl get pod volume-share-devops -o yaml | grep -A 3 "emptyDir"
```

**Causes:**
1. Node disk full
2. sizeLimit exceeded
3. Too many files in emptyDir

**Solution:**
```bash
# Clean up files
kubectl exec volume-share-devops -c volume-container-devops-1 -- rm /tmp/blog/large-file.bin

# Or set appropriate size limit
volumes:
- name: volume-share
  emptyDir:
    sizeLimit: "5Gi"          # Allow up to 5GB
```

### Issue 7: Cannot Exec into Container

**Symptoms:**
```bash
kubectl exec -it volume-share-devops -c volume-container-devops-1 -- /bin/bash
# error: unable to upgrade connection: container not found ("volume-container-devops-1")
```

**Diagnosis:**
```bash
# List actual container names
kubectl get pod volume-share-devops -o jsonpath='{.spec.containers[*].name}'

# Check if container is running
kubectl get pod volume-share-devops -o jsonpath='{.status.containerStatuses[*].state}'
```

**Common Causes:**
1. Wrong container name (typo)
2. Container not running
3. Container has no shell

**Solution:**
```bash
# Use correct container name
kubectl get pod volume-share-devops -o jsonpath='{.spec.containers[*].name}'
# volume-container-devops-1 volume-container-devops-2

# Use exact name from output
kubectl exec -it volume-share-devops -c volume-container-devops-1 -- /bin/bash

# If /bin/bash not available, try /bin/sh
kubectl exec -it volume-share-devops -c volume-container-devops-1 -- /bin/sh
```

## Real-World Use Cases

### Use Case 1: Web Server with Log Shipper (Sidecar Pattern)

**Scenario:**
Nginx web server generates access logs. Fluentd sidecar reads and ships logs to Elasticsearch.

**Implementation:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-with-logging
spec:
  containers:
  # Main application container
  - name: nginx
    image: nginx:latest
    volumeMounts:
    - name: shared-logs
      mountPath: /var/log/nginx
  # Sidecar log shipper
  - name: fluentd
    image: fluent/fluentd:latest
    volumeMounts:
    - name: shared-logs
      mountPath: /var/log/nginx
      readOnly: true              # Read-only access to logs
  volumes:
  - name: shared-logs
    emptyDir: {}
```

**How It Works:**
```
Nginx Container                     Fluentd Container
      ↓ Writes logs                      ↓ Reads logs
/var/log/nginx/access.log ← Shared Volume → /var/log/nginx/access.log
                                           ↓ Ships to
                                      Elasticsearch
```

### Use Case 2: Application with Configuration Updater

**Scenario:**
Application reads configuration from file. Sidecar periodically fetches updated config from API.

**Implementation:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-with-config-updater
spec:
  containers:
  # Main application
  - name: app
    image: my-app:latest
    command: ["./app", "--config=/config/app.conf"]
    volumeMounts:
    - name: config-volume
      mountPath: /config
  # Config updater sidecar
  - name: config-updater
    image: config-fetcher:latest
    command: ["./fetch-config.sh"]
    env:
    - name: CONFIG_API_URL
      value: "https://config-api.example.com"
    - name: CONFIG_FILE
      value: "/config/app.conf"
    volumeMounts:
    - name: config-volume
      mountPath: /config
  volumes:
  - name: config-volume
    emptyDir: {}
```

**How It Works:**
```
Config Updater (every 60s):
1. Fetch config from API
2. Write to /config/app.conf

Application:
1. Reads /config/app.conf
2. Reloads on file change
```

### Use Case 3: Data Processing Pipeline

**Scenario:**
Three-stage data processing: fetch → process → upload

**Implementation:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: data-pipeline
spec:
  # Init container: Download data
  initContainers:
  - name: data-fetcher
    image: curlimages/curl:latest
    command: ["sh", "-c"]
    args:
    - |
      curl https://api.example.com/data.csv -o /data/input.csv
    volumeMounts:
    - name: pipeline-data
      mountPath: /data
  containers:
  # Container 1: Process data
  - name: processor
    image: python:3.9
    command: ["python", "/scripts/process.py"]
    volumeMounts:
    - name: pipeline-data
      mountPath: /data
    - name: scripts
      mountPath: /scripts
  # Container 2: Upload results
  - name: uploader
    image: amazon/aws-cli:latest
    command: ["sh", "-c"]
    args:
    - |
      while [ ! -f /data/output.csv ]; do sleep 5; done
      aws s3 cp /data/output.csv s3://my-bucket/results/
    volumeMounts:
    - name: pipeline-data
      mountPath: /data
  volumes:
  - name: pipeline-data
    emptyDir: {}
  - name: scripts
    configMap:
      name: processing-scripts
```

**Workflow:**
```
1. Init Container: Fetcher
   Downloads input.csv → /data/input.csv

2. Container 1: Processor
   Reads /data/input.csv
   Processes data
   Writes /data/output.csv

3. Container 2: Uploader
   Waits for /data/output.csv
   Uploads to S3
```

### Use Case 4: Multi-Stage Build Artifacts

**Scenario:**
Build container compiles code, runtime container serves the application.

**Implementation:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: build-and-serve
spec:
  initContainers:
  # Build stage
  - name: builder
    image: golang:1.20
    command: ["sh", "-c"]
    args:
    - |
      cd /src
      go build -o /artifacts/app main.go
    volumeMounts:
    - name: source-code
      mountPath: /src
    - name: build-artifacts
      mountPath: /artifacts
  containers:
  # Runtime stage
  - name: app
    image: alpine:latest
    command: ["/app/app"]
    volumeMounts:
    - name: build-artifacts
      mountPath: /app
  volumes:
  - name: source-code
    configMap:
      name: app-source
  - name: build-artifacts
    emptyDir: {}
```

**Workflow:**
```
Build Container:
1. Reads source from ConfigMap
2. Compiles binary
3. Writes to /artifacts/app

Runtime Container:
1. Reads binary from /app/app (same volume)
2. Executes application
```

## Best Practices

### 1. Choose Appropriate Volume Type

✅ **Use emptyDir for:**
- Temporary scratch space
- Caching data that can be regenerated
- Sharing data between containers in a pod
- Short-lived data

✅ **Use PersistentVolume for:**
- Database storage
- User-uploaded files
- Application state
- Any data that must survive pod deletion

✅ **Use ConfigMap/Secret for:**
- Configuration files
- Environment-specific settings
- Credentials and secrets

### 2. Set Size Limits on emptyDir

❌ **Bad (unlimited):**
```yaml
volumes:
- name: cache
  emptyDir: {}
```
Risk: Can fill up node disk

✅ **Good (size-limited):**
```yaml
volumes:
- name: cache
  emptyDir:
    sizeLimit: "2Gi"
```
Benefit: Prevents disk exhaustion

### 3. Use Memory-Backed emptyDir for Performance-Critical Data

✅ **For fast I/O:**
```yaml
volumes:
- name: fast-cache
  emptyDir:
    medium: "Memory"
    sizeLimit: "512Mi"
```

**Note:** Counts against container memory limits!

### 4. Use Read-Only Mounts When Appropriate

✅ **For sidecars that only read:**
```yaml
containers:
- name: log-shipper
  volumeMounts:
  - name: logs
    mountPath: /logs
    readOnly: true        # Prevents accidental modification
```

### 5. Document Volume Purposes

✅ **Good practice:**
```yaml
volumes:
- name: app-logs              # Clear, descriptive name
  emptyDir: {}
  # Purpose: Store application logs for shipper sidecar
  # Lifecycle: Deleted with pod
  # Size: No limit (logs rotated by app)
```

### 6. Clean Up Temporary Files

```bash
# Add cleanup logic in application or sidecar
containers:
- name: cleanup
  image: busybox
  command: ["sh", "-c"]
  args:
  - |
    while true; do
      find /tmp/cache -type f -mtime +1 -delete
      sleep 3600
    done
  volumeMounts:
  - name: cache-volume
    mountPath: /tmp/cache
```

### 7. Test Volume Sharing Before Production

```bash
# Always verify volumes work as expected:
# 1. Create test pod
kubectl apply -f test-pod.yaml

# 2. Write from container 1
kubectl exec test-pod -c container1 -- touch /shared/test.txt

# 3. Verify from container 2
kubectl exec test-pod -c container2 -- ls /shared/
# Should show test.txt

# 4. Clean up
kubectl delete pod test-pod
```

### 8. Monitor emptyDir Usage

```yaml
# Use resource quotas to limit emptyDir usage
apiVersion: v1
kind: ResourceQuota
metadata:
  name: storage-quota
spec:
  hard:
    requests.ephemeral-storage: "10Gi"
```

### 9. Use Proper Naming Conventions

✅ **Good naming:**
```yaml
volumes:
- name: nginx-logs          # Purpose-based
- name: shared-config       # What it contains
- name: temp-processing     # Temporary data
```

❌ **Bad naming:**
```yaml
volumes:
- name: vol1                # Not descriptive
- name: data                # Too generic
- name: temp                # Ambiguous
```

### 10. Handle Volume Mount Failures Gracefully

```yaml
containers:
- name: app
  image: my-app:latest
  command: ["sh", "-c"]
  args:
  - |
    # Check if mount exists
    if [ ! -d /data ]; then
      echo "ERROR: /data volume not mounted"
      exit 1
    fi
    # Continue with application
    ./start-app.sh
  volumeMounts:
  - name: data-volume
    mountPath: /data
```

## Completion Checklist

Verify your setup with these commands:

```bash
# 1. Pod exists and is running
kubectl get pod volume-share-devops
# STATUS: Running, READY: 2/2 ✅

# 2. Pod has correct name
kubectl get pod volume-share-devops -o jsonpath='{.metadata.name}'
# Output: volume-share-devops ✅

# 3. Both containers exist with correct names
kubectl get pod volume-share-devops -o jsonpath='{.spec.containers[*].name}'
# Output: volume-container-devops-1 volume-container-devops-2 ✅

# 4. Both containers use debian:latest
kubectl get pod volume-share-devops -o jsonpath='{.spec.containers[*].image}'
# Output: debian:latest debian:latest ✅

# 5. Volume exists with correct name and type
kubectl get pod volume-share-devops -o jsonpath='{.spec.volumes[*]}'
# Output: {"emptyDir":{},"name":"volume-share"} ✅

# 6. Container 1 mounts at /tmp/blog
kubectl get pod volume-share-devops -o jsonpath='{.spec.containers[0].volumeMounts[*].mountPath}'
# Output: /tmp/blog ✅

# 7. Container 2 mounts at /tmp/demo
kubectl get pod volume-share-devops -o jsonpath='{.spec.containers[1].volumeMounts[*].mountPath}'
# Output: /tmp/demo ✅

# 8. File created in container 1 exists
kubectl exec volume-share-devops -c volume-container-devops-1 -- ls /tmp/blog/blog.txt
# Output: /tmp/blog/blog.txt ✅

# 9. Same file visible in container 2
kubectl exec volume-share-devops -c volume-container-devops-2 -- ls /tmp/demo/blog.txt
# Output: /tmp/demo/blog.txt ✅

# 10. File content is same in both containers
kubectl exec volume-share-devops -c volume-container-devops-1 -- cat /tmp/blog/blog.txt
kubectl exec volume-share-devops -c volume-container-devops-2 -- cat /tmp/demo/blog.txt
# Both outputs match ✅

# 11. Both containers are actively running
kubectl get pod volume-share-devops -o jsonpath='{.status.phase}'
# Output: Running ✅

# 12. No container restarts (indicating stability)
kubectl get pod volume-share-devops -o jsonpath='{.status.containerStatuses[*].restartCount}'
# Output: 0 0 ✅
```

**All checks passing = Task complete! ✅**

## Summary

### What We Accomplished
1. ✅ Created multi-container pod with two Debian containers
2. ✅ Configured emptyDir volume for temporary shared storage
3. ✅ Mounted same volume at different paths in each container
4. ✅ Verified bidirectional file sharing between containers
5. ✅ Tested that files created in one container are immediately visible in the other
6. ✅ Demonstrated practical use of shared volumes in Kubernetes

### Key Concepts Learned

**emptyDir Volumes:**
- Temporary storage with pod lifetime
- Shared among all containers in a pod
- Initially empty, created when pod is scheduled
- Deleted when pod is removed
- Can be disk-backed or memory-backed

**Multi-Container Pods:**
- Multiple containers in single pod share resources
- Containers can communicate via shared volumes
- Each container can mount volumes at different paths
- Useful for sidecar, ambassador, and adapter patterns

**Volume Mounts:**
- Connect pod-level volumes to container paths
- Same volume can be mounted multiple times
- Different containers can have different mount paths
- Supports read-only and read-write access

### Critical Commands
```bash
# Create pod with shared volume
kubectl apply -f volume-share-pod.yaml

# Verify pod and containers
kubectl get pods
kubectl describe pod <pod-name>

# Access containers
kubectl exec -it <pod> -c <container> -- /bin/bash

# Test file sharing
kubectl exec <pod> -c <container1> -- echo "test" > /path/file.txt
kubectl exec <pod> -c <container2> -- cat /path/file.txt

# Check volume configuration
kubectl get pod <pod> -o yaml | grep -A 5 volumes
kubectl get pod <pod> -o jsonpath='{.spec.volumes[*]}'
```

### Volume Type Comparison

| Feature | emptyDir | hostPath | PersistentVolume |
|---------|----------|----------|------------------|
| **Lifetime** | Pod lifetime | Survives pod | Cluster-managed |
| **Scope** | Single pod | Single node | Any pod |
| **Sharing** | Between containers | Between pods on node | Cluster-wide |
| **Use Case** | Temporary data | Node-specific | Persistent data |
| **Data Loss** | On pod delete | On node failure | Replicated/safe |

### Real-World Applications

**Sidecar Pattern:**
```
Main App Container + Log Shipper Container
    ↓ Share Volume
App writes logs → Shared Volume → Shipper sends to central logging
```

**Build + Runtime Pattern:**
```
Build Container (init) + Runtime Container
    ↓ Share Volume
Build creates artifacts → Shared Volume → Runtime executes artifacts
```

**Configuration Pattern:**
```
App Container + Config Updater Container
    ↓ Share Volume
Updater fetches config → Shared Volume → App reads config
```

### Best Practices Summary
1. ✅ Use emptyDir for temporary, inter-container communication
2. ✅ Set size limits to prevent disk exhaustion
3. ✅ Use memory-backed emptyDir for high-performance scenarios
4. ✅ Mount volumes as read-only when containers only need read access
5. ✅ Use descriptive volume names
6. ✅ Document volume purposes and lifecycles
7. ✅ Clean up temporary files to avoid filling volumes
8. ✅ Test volume sharing before production deployment
9. ✅ Use appropriate volume types based on data persistence needs
10. ✅ Monitor volume usage with resource quotas

### What's Next?

**Day 55+:** Continue with Kubernetes topics:
- PersistentVolumes and PersistentVolumeClaims
- StorageClasses and dynamic provisioning
- ConfigMaps and Secrets as volumes
- Volume snapshots and cloning
- StatefulSets with persistent storage
- Volume expansion and management

**Volume Learning Path:**
```
Day 54: emptyDir (temporary sharing) ← YOU ARE HERE
    ↓
Day 55+: PersistentVolumes (permanent storage)
    ↓
Day 56+: StorageClasses (dynamic provisioning)
    ↓
Day 57+: StatefulSets (stable storage)
```

🎉 **Day 54 Complete!** You've mastered Kubernetes shared volumes with emptyDir, enabling containers to collaborate and share data within a pod—a fundamental pattern for building sophisticated Kubernetes applications!
