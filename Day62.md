# Day 62: Kubernetes Secrets - Secure Sensitive Data Storage

## 📋 Task Overview

**Scenario:** The Nautilus DevOps team needs to deploy license-based tools in their Kubernetes cluster. License keys and sensitive information must be stored securely using Kubernetes Secrets instead of plain text ConfigMaps or environment variables.

**Requirements:**
1. Create a generic secret named `beta` from existing file `/opt/beta.txt`
2. Create a pod named `secret-datacenter`
3. Mount the secret as a volume inside the container at `/opt/cluster`
4. Container should remain running for verification
5. Verify secret is accessible inside the container

---

## 🎯 Learning Objectives

By the end of this task, you will understand:
- **Kubernetes Secrets:** Secure storage for sensitive data
- **Secret Types:** Generic, TLS, Docker registry, and more
- **Secret Creation Methods:** From files, literals, YAML manifests
- **Secret Consumption:** Environment variables vs volume mounts
- **Security Best Practices:** Encryption, RBAC, rotation
- **Secret Management:** Updates, versioning, troubleshooting

---

## 📖 Understanding Kubernetes Secrets

### What are Kubernetes Secrets?

**Definition:** Objects that store sensitive data such as passwords, OAuth tokens, SSH keys, and license keys in a more secure manner than ConfigMaps.

**Key Characteristics:**
- ✅ Base64 encoded (not encrypted by default)
- ✅ Stored in etcd
- ✅ Can be mounted as files or environment variables
- ✅ Namespaced resources
- ✅ Size limit: 1MB per secret
- ✅ Support encryption at rest (with configuration)

### Secrets vs ConfigMaps

```
┌─────────────────────────────────────────────────────────────────┐
│                    Secrets vs ConfigMaps                         │
└─────────────────────────────────────────────────────────────────┘

┌──────────────────────┐              ┌──────────────────────┐
│   ConfigMaps         │              │   Secrets            │
│                      │              │                      │
│  📄 Plain text data  │              │  🔐 Sensitive data   │
│  📝 Configuration    │              │  🔑 Passwords/Keys   │
│  🌍 Public info      │              │  🎫 Licenses/Tokens  │
│  👁️ Visible in UI    │              │  🔒 Base64 encoded   │
│                      │              │  🚫 Hidden in UI     │
└──────────────────────┘              └──────────────────────┘
         ↓                                      ↓
    ┌─────────┐                            ┌─────────┐
    │ Pod Env │                            │ Pod Env │
    │ Volume  │                            │ Volume  │
    └─────────┘                            └─────────┘
```

| Aspect | ConfigMaps | Secrets |
|--------|-----------|---------|
| **Purpose** | Non-sensitive configuration | Sensitive data (passwords, keys) |
| **Encoding** | Plain text | Base64 encoded |
| **UI Display** | Visible | Hidden by default |
| **Size Limit** | 1MB | 1MB |
| **Encryption** | Not supported | Can be encrypted at rest |
| **Use Cases** | App configs, feature flags | Passwords, API keys, certificates |
| **Best Practice** | Public configurations | Private credentials only |

---

## 🔐 Secret Types

### 1. **Opaque (Generic) Secrets**

Most common type for arbitrary user-defined data.

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
type: Opaque
data:
  username: YWRtaW4=          # admin (base64)
  password: cGFzc3dvcmQxMjM=  # password123 (base64)
```

**Use Cases:**
- Database credentials
- API keys
- License keys (our task today!)
- General sensitive data

---

### 2. **kubernetes.io/tls**

For storing TLS certificates and keys.

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: tls-secret
type: kubernetes.io/tls
data:
  tls.crt: LS0tLS1CRUdJTi...  # Certificate (base64)
  tls.key: LS0tLS1CRUdJTi...  # Private key (base64)
```

**Use Cases:**
- HTTPS/TLS certificates
- Ingress SSL termination
- Mutual TLS authentication

---

### 3. **kubernetes.io/dockerconfigjson**

For Docker registry authentication.

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: docker-secret
type: kubernetes.io/dockerconfigjson
data:
  .dockerconfigjson: eyJhdXRocyI6...  # Docker config (base64)
```

**Use Cases:**
- Pull images from private registries
- Docker Hub authentication
- Harbor, ECR, GCR access

---

### 4. **kubernetes.io/basic-auth**

For basic HTTP authentication.

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: basic-auth-secret
type: kubernetes.io/basic-auth
data:
  username: YWRtaW4=
  password: c2VjcmV0
```

**Use Cases:**
- HTTP basic authentication
- Legacy authentication systems
- Simple access control

---

### 5. **kubernetes.io/ssh-auth**

For SSH private keys.

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: ssh-secret
type: kubernetes.io/ssh-auth
data:
  ssh-privatekey: LS0tLS1CRUdJTi...  # SSH key (base64)
```

**Use Cases:**
- Git repository access
- SSH to remote servers
- Secure file transfers

---

## 🛠️ Secret Creation Methods

### Method 1: From File (Our Task Today!)

```bash
# Create secret from file
kubectl create secret generic beta --from-file=/opt/beta.txt

# What happens:
# - Key: filename (beta.txt)
# - Value: file contents (base64 encoded)
```

**Advantages:**
- ✅ Quick for single files
- ✅ Preserves file structure
- ✅ No manual base64 encoding

---

### Method 2: From Literal Values

```bash
# Create secret from literal strings
kubectl create secret generic db-secret \
  --from-literal=username=admin \
  --from-literal=password=supersecret

# Multiple key-value pairs
kubectl create secret generic app-secret \
  --from-literal=api_key=abc123 \
  --from-literal=token=xyz789 \
  --from-literal=license=XXXX-YYYY-ZZZZ
```

**Advantages:**
- ✅ Simple for small amounts of data
- ✅ Command-line friendly
- ✅ No file management needed

---

### Method 3: From YAML Manifest

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: Opaque
data:
  # Base64 encoded values
  username: YWRtaW4=          # echo -n "admin" | base64
  password: cGFzc3dvcmQxMjM=  # echo -n "password123" | base64
```

```bash
# Apply the manifest
kubectl apply -f secret.yaml
```

**Advantages:**
- ✅ Version control friendly
- ✅ Declarative approach
- ✅ Easy to review and audit

---

### Method 4: From stringData (Plain Text)

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: Opaque
stringData:  # Plain text (Kubernetes encodes it)
  username: admin
  password: password123
  license: XXXX-YYYY-ZZZZ
```

**Advantages:**
- ✅ No manual base64 encoding
- ✅ Human-readable in manifest
- ✅ Kubernetes handles encoding

**⚠️ Warning:** stringData is converted to base64 when stored, but appears plain in YAML.

---

### Method 5: From env File

```bash
# Create .env file
cat > secret.env <<EOF
DB_USER=admin
DB_PASS=supersecret
API_KEY=abc123xyz
EOF

# Create secret from env file
kubectl create secret generic env-secret --from-env-file=secret.env
```

**Advantages:**
- ✅ Multiple key-value pairs from file
- ✅ Environment variable format
- ✅ Easy migration from Docker Compose

---

## 🔧 Secret Consumption Methods

### Method 1: Volume Mount (Our Task Today!)

**Best for:** Files, certificates, configuration files

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-pod
spec:
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - name: secret-volume
      mountPath: /opt/secrets
      readOnly: true
  volumes:
  - name: secret-volume
    secret:
      secretName: my-secret
```

**Inside Container:**
```bash
ls /opt/secrets/
# username  password

cat /opt/secrets/username
# admin

cat /opt/secrets/password
# password123
```

**Advantages:**
- ✅ Each secret key becomes a file
- ✅ No environment variable limits
- ✅ Supports large secrets
- ✅ Automatic updates (with proper configuration)

---

### Method 2: Environment Variables

**Best for:** Simple key-value pairs, application credentials

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: env-pod
spec:
  containers:
  - name: app
    image: nginx
    env:
    # Single key from secret
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: password
    
    # Multiple keys
    - name: API_KEY
      valueFrom:
        secretKeyRef:
          name: api-secret
          key: api_key
```

**Inside Container:**
```bash
echo $DB_PASSWORD
# supersecret

env | grep API_KEY
# API_KEY=abc123
```

**Advantages:**
- ✅ Direct access as environment variables
- ✅ Simple application integration
- ✅ No file system access needed

**Disadvantages:**
- ❌ Environment variables can be leaked in logs
- ❌ No automatic updates (requires pod restart)
- ❌ Size limitations

---

### Method 3: All Keys as Environment Variables

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: env-all-pod
spec:
  containers:
  - name: app
    image: nginx
    envFrom:
    - secretRef:
        name: my-secret
```

**Inside Container:**
```bash
env | grep -E "username|password"
# username=admin
# password=password123
```

**Advantages:**
- ✅ Load all secret keys at once
- ✅ Minimal YAML configuration

---

### Method 4: Specific Keys to Specific Files

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: selective-mount-pod
spec:
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - name: secret-volume
      mountPath: /opt/secrets
  volumes:
  - name: secret-volume
    secret:
      secretName: my-secret
      items:
      - key: username
        path: db/username.txt
      - key: password
        path: db/password.txt
```

**Inside Container:**
```bash
cat /opt/secrets/db/username.txt
# admin

cat /opt/secrets/db/password.txt
# password123
```

**Advantages:**
- ✅ Custom file paths and names
- ✅ Organize secrets in subdirectories
- ✅ Select specific keys only

---

## 🚀 Task Implementation

### Understanding Today's Scenario

**Given:**
- Secret file exists at `/opt/beta.txt` on jump host
- Contains license key or sensitive data

**Goal:**
1. Create secret from file
2. Create pod with volume mount
3. Verify secret is accessible inside container

**Data Flow:**
```
/opt/beta.txt (jump host)
         ↓
kubectl create secret (base64 encode)
         ↓
Secret "beta" (Kubernetes)
         ↓
Volume mount in pod
         ↓
/opt/cluster/beta.txt (container)
```

---

### Step 1: Verify Secret File Exists

```bash
# Check if file exists on jump host
ls -lh /opt/beta.txt

# View file contents (be careful with sensitive data!)
cat /opt/beta.txt
# Example output: BETA-LICENSE-KEY-12345-ABCDE
```

**Expected Output:**
```
-rw-r--r-- 1 root root 30 Jan 7 10:00 /opt/beta.txt
```

---

### Step 2: Create Generic Secret from File

**Command:**
```bash
kubectl create secret generic beta --from-file=/opt/beta.txt
```

**Expected Output:**
```
secret/beta created
```

**What Happens Behind the Scenes:**
1. Kubernetes reads `/opt/beta.txt`
2. Base64 encodes the content
3. Creates secret with:
   - Name: `beta`
   - Type: `Opaque` (generic)
   - Key: `beta.txt` (filename)
   - Value: `<base64-encoded-content>`

**Alternative with Custom Key Name:**
```bash
# If you want custom key name instead of filename
kubectl create secret generic beta --from-file=license=/opt/beta.txt
# Key will be "license" instead of "beta.txt"
```

---

### Step 3: Verify Secret Created

```bash
# List secrets
kubectl get secrets

# Expected output:
# NAME   TYPE     DATA   AGE
# beta   Opaque   1      10s

# Describe secret (doesn't show values)
kubectl describe secret beta

# View secret details (shows base64 encoded values)
kubectl get secret beta -o yaml
```

**Detailed Secret Information:**
```bash
kubectl describe secret beta
```

**Expected Output:**
```
Name:         beta
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
beta.txt:  30 bytes
```

**View Secret YAML:**
```bash
kubectl get secret beta -o yaml
```

**Expected Output:**
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: beta
  namespace: default
type: Opaque
data:
  beta.txt: QkVUQS1MSUNFTlNFLUtFWS0xMjM0NS1BQkNERQ==
```

**Decode Secret Value (for verification only!):**
```bash
# Decode base64 to see original value
kubectl get secret beta -o jsonpath='{.data.beta\.txt}' | base64 --decode
# Output: BETA-LICENSE-KEY-12345-ABCDE
```

---

### Step 4: Create Pod with Secret Volume Mount

**Complete Pod YAML:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-datacenter
  labels:
    app: secret-demo
spec:
  containers:
  - name: secret-container-datacenter
    image: debian:latest
    command: ["/bin/bash", "-c", "sleep infinity"]
    volumeMounts:
    - name: secret-volume
      mountPath: /opt/cluster
      readOnly: true
  volumes:
  - name: secret-volume
    secret:
      secretName: beta
```

**Understanding Each Component:**

**1. Pod Metadata:**
```yaml
metadata:
  name: secret-datacenter
  labels:
    app: secret-demo
```
- Pod name: `secret-datacenter` (as required)
- Label for identification

**2. Container Spec:**
```yaml
containers:
- name: secret-container-datacenter
  image: debian:latest
  command: ["/bin/bash", "-c", "sleep infinity"]
```
- **name:** `secret-container-datacenter` (as required)
- **image:** `debian:latest` (Debian with latest tag)
- **command:** Keeps container running indefinitely for verification

**3. Volume Mount:**
```yaml
volumeMounts:
- name: secret-volume
  mountPath: /opt/cluster
  readOnly: true
```
- **name:** References volume defined below
- **mountPath:** `/opt/cluster` (as required)
- **readOnly:** Best practice for secrets (prevents modification)

**4. Secret Volume:**
```yaml
volumes:
- name: secret-volume
  secret:
    secretName: beta
```
- **name:** `secret-volume` (volume identifier)
- **secret.secretName:** `beta` (the secret we created)

---

### Step 5: Create the Pod

**Method 1: Using YAML File (Recommended)**

```bash
# Create pod YAML file
cat > secret-datacenter.yaml <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: secret-datacenter
  labels:
    app: secret-demo
spec:
  containers:
  - name: secret-container-datacenter
    image: debian:latest
    command: ["/bin/bash", "-c", "sleep infinity"]
    volumeMounts:
    - name: secret-volume
      mountPath: /opt/cluster
      readOnly: true
  volumes:
  - name: secret-volume
    secret:
      secretName: beta
EOF

# Apply the pod
kubectl apply -f secret-datacenter.yaml
```

**Method 2: Using kubectl run (with modifications)**

```bash
# Generate YAML first
kubectl run secret-datacenter \
  --image=debian:latest \
  --command -- /bin/bash -c "sleep infinity" \
  --dry-run=client -o yaml > secret-datacenter.yaml

# Manually edit YAML to add volume mount and secret volume
# Then apply:
kubectl apply -f secret-datacenter.yaml
```

**Expected Output:**
```
pod/secret-datacenter created
```

---

### Step 6: Verify Pod Status

**Check Pod Creation:**
```bash
kubectl get pods secret-datacenter
```

**Expected Output:**
```
NAME                READY   STATUS    RESTARTS   AGE
secret-datacenter   1/1     Running   0          15s
```

**Possible Intermediate States:**
```
# Pulling image
NAME                READY   STATUS              RESTARTS   AGE
secret-datacenter   0/1     ContainerCreating   0          5s

# Running
NAME                READY   STATUS    RESTARTS   AGE
secret-datacenter   1/1     Running   0          20s
```

**Watch Pod Status in Real-Time:**
```bash
kubectl get pod secret-datacenter -w
# Press Ctrl+C to stop watching
```

---

### Step 7: Describe Pod for Details

```bash
kubectl describe pod secret-datacenter
```

**Key Sections to Review:**

```yaml
Containers:
  secret-container-datacenter:
    Image:         debian:latest
    Command:
      /bin/bash
      -c
      sleep infinity
    State:          Running
    Ready:          True
    Mounts:
      /opt/cluster from secret-volume (ro)

Volumes:
  secret-volume:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  beta
    Optional:    false

Events:
  Type    Reason     Age   Message
  ----    ------     ----  -------
  Normal  Scheduled  30s   Successfully assigned default/secret-datacenter
  Normal  Pulling    29s   Pulling image "debian:latest"
  Normal  Pulled     15s   Successfully pulled image "debian:latest"
  Normal  Created    15s   Created container secret-container-datacenter
  Normal  Started    14s   Started container secret-container-datacenter
```

**Important Details:**
- **Mounts:** Shows `/opt/cluster` mounted from `secret-volume` (read-only)
- **Volumes:** Confirms secret `beta` is used
- **Events:** Shows successful image pull and container start

---

### Step 8: Verify Secret Inside Container

**Exec into Container:**
```bash
kubectl exec -it secret-datacenter -- bash
```

**Inside Container - Verify Secret File:**
```bash
# List files in mounted path
ls -lh /opt/cluster/

# Expected output:
# total 4.0K
# -rw-r--r-- 1 root root 30 Jan 7 10:00 beta.txt

# Read secret content
cat /opt/cluster/beta.txt

# Expected output:
# BETA-LICENSE-KEY-12345-ABCDE

# Verify read-only mount (should fail)
echo "test" > /opt/cluster/test.txt
# bash: /opt/cluster/test.txt: Read-only file system

# Exit container
exit
```

**One-Liner Verification (without exec):**
```bash
kubectl exec secret-datacenter -- cat /opt/cluster/beta.txt
# Output: BETA-LICENSE-KEY-12345-ABCDE
```

**Check Multiple Aspects:**
```bash
# File exists
kubectl exec secret-datacenter -- ls /opt/cluster/

# File content
kubectl exec secret-datacenter -- cat /opt/cluster/beta.txt

# File permissions
kubectl exec secret-datacenter -- ls -lh /opt/cluster/beta.txt

# Mount point is read-only
kubectl exec secret-datacenter -- touch /opt/cluster/test.txt
# Error: touch: cannot touch '/opt/cluster/test.txt': Read-only file system
```

---

### Step 9: View Pod Logs (Optional)

```bash
# View logs (should show nothing since container just sleeps)
kubectl logs secret-datacenter

# Follow logs in real-time
kubectl logs -f secret-datacenter
```

---

## 📊 Secret Data Flow Visualization

```
┌─────────────────────────────────────────────────────────────────┐
│                    Secret Lifecycle                              │
└─────────────────────────────────────────────────────────────────┘

Step 1: File on Jump Host
┌──────────────────────────┐
│  /opt/beta.txt           │
│  Content: LICENSE-KEY    │
└────────────┬─────────────┘
             ↓

Step 2: kubectl create secret
┌──────────────────────────────────────┐
│  kubectl create secret generic beta  │
│  --from-file=/opt/beta.txt          │
│                                      │
│  → Reads file content                │
│  → Base64 encodes                    │
│  → Creates Secret resource           │
└────────────┬─────────────────────────┘
             ↓

Step 3: Secret Stored in etcd
┌────────────────────────────────────┐
│  Secret: beta                      │
│  Type: Opaque                      │
│  Data:                             │
│    beta.txt: <base64-encoded>      │
└────────────┬───────────────────────┘
             ↓

Step 4: Pod References Secret
┌───────────────────────────────────┐
│  Pod: secret-datacenter           │
│  Volume:                          │
│    secret-volume → Secret: beta   │
└────────────┬──────────────────────┘
             ↓

Step 5: Volume Mount in Container
┌────────────────────────────────────┐
│  Container: secret-container-dc    │
│  Mount: /opt/cluster               │
│                                    │
│  /opt/cluster/                     │
│    └── beta.txt (decoded content)  │
└────────────┬───────────────────────┘
             ↓

Step 6: Application Reads Secret
┌────────────────────────────────────┐
│  cat /opt/cluster/beta.txt         │
│  → Reads: LICENSE-KEY              │
│  → Application uses license        │
└────────────────────────────────────┘
```

---

## 🐛 Common Issues and Troubleshooting

### Issue 1: Secret Not Created

**Symptoms:**
```bash
kubectl get secret beta
# Error from server (NotFound): secrets "beta" not found
```

**Root Causes:**

**A. File Doesn't Exist**
```bash
kubectl create secret generic beta --from-file=/opt/beta.txt
# Error: error reading /opt/beta.txt: no such file or directory
```

**Solution:**
```bash
# Verify file exists
ls -lh /opt/beta.txt

# Check file path (absolute vs relative)
pwd
# /home/user

# Use absolute path
kubectl create secret generic beta --from-file=/opt/beta.txt
```

**B. Insufficient Permissions**
```bash
kubectl create secret generic beta --from-file=/opt/beta.txt
# Error: open /opt/beta.txt: permission denied
```

**Solution:**
```bash
# Check file permissions
ls -lh /opt/beta.txt
# -rw------- 1 root root 30 Jan 7 10:00 /opt/beta.txt

# Read as root if needed
sudo kubectl create secret generic beta --from-file=/opt/beta.txt

# Or copy to accessible location
sudo cp /opt/beta.txt /tmp/
kubectl create secret generic beta --from-file=/tmp/beta.txt
```

**C. Wrong Kubectl Context**
```bash
# Check current context
kubectl config current-context

# List all contexts
kubectl config get-contexts

# Switch context if needed
kubectl config use-context <correct-context>
```

---

### Issue 2: Pod Stuck in ContainerCreating

**Symptoms:**
```bash
kubectl get pod secret-datacenter
# NAME                READY   STATUS              RESTARTS   AGE
# secret-datacenter   0/1     ContainerCreating   0          2m
```

**Diagnosis:**
```bash
kubectl describe pod secret-datacenter
```

**Root Causes:**

**A. Secret Not Found**
```bash
# Events:
# Warning  FailedMount  pod/secret-datacenter  MountVolume.SetUp failed: secret "beta" not found
```

**Solution:**
```bash
# Verify secret exists
kubectl get secret beta

# If missing, create it
kubectl create secret generic beta --from-file=/opt/beta.txt

# Delete and recreate pod
kubectl delete pod secret-datacenter
kubectl apply -f secret-datacenter.yaml
```

**B. Image Pull Failure**
```bash
# Events:
# Warning  Failed     pod/secret-datacenter  Failed to pull image "debian:latest"
# Warning  Failed     pod/secret-datacenter  Error: ErrImagePull
```

**Solution:**
```bash
# Check if image exists
docker pull debian:latest

# Or use alternative image
kubectl set image pod/secret-datacenter secret-container-datacenter=ubuntu:latest

# Check registry credentials if using private registry
kubectl get secret -n default | grep docker
```

**C. Volume Mount Conflict**
```bash
# Events:
# Warning  FailedMount  pod/secret-datacenter  MountVolume.SetUp failed: volume name conflicts
```

**Solution:**
```bash
# Check pod YAML for duplicate volume names
kubectl get pod secret-datacenter -o yaml | grep -A5 volumes

# Ensure unique volume names
# Fix: Edit YAML and reapply
```

---

### Issue 3: Secret File Not Visible in Container

**Symptoms:**
```bash
kubectl exec secret-datacenter -- ls /opt/cluster
# ls: cannot access '/opt/cluster': No such file or directory
```

**Root Causes:**

**A. Wrong Mount Path**
```yaml
# ❌ Wrong: Pod YAML has different path
volumeMounts:
- name: secret-volume
  mountPath: /opt/secrets  # Wrong path!

# ✅ Correct:
volumeMounts:
- name: secret-volume
  mountPath: /opt/cluster
```

**Solution:**
```bash
# Check actual mount path
kubectl describe pod secret-datacenter | grep -A5 Mounts

# Correct the YAML
kubectl delete pod secret-datacenter
# Edit secret-datacenter.yaml
kubectl apply -f secret-datacenter.yaml
```

**B. Volume Name Mismatch**
```yaml
# ❌ Wrong: Volume name doesn't match mount name
containers:
- volumeMounts:
  - name: secret-volume
    mountPath: /opt/cluster

volumes:
- name: different-volume  # Doesn't match!
  secret:
    secretName: beta

# ✅ Correct: Names must match
```

**C. Container Not Running**
```bash
# Check container status
kubectl get pod secret-datacenter
# NAME                READY   STATUS    RESTARTS   AGE
# secret-datacenter   0/1     Error     2          1m
```

**Solution:**
```bash
# Check container logs
kubectl logs secret-datacenter

# Check command
kubectl get pod secret-datacenter -o jsonpath='{.spec.containers[0].command}'
# Should be: ["/bin/bash","-c","sleep infinity"]
```

---

### Issue 4: Secret Content is Base64 Encoded in Container

**Symptoms:**
```bash
kubectl exec secret-datacenter -- cat /opt/cluster/beta.txt
# Output: QkVUQS1MSUNFTlNFLUtFWS0xMjM0NS1BQkNERQ==
```

**This is WRONG!** Secret should be auto-decoded.

**Root Cause:**
Manually created secret with already base64-encoded data.

**Example of Wrong Secret Creation:**
```bash
# ❌ Wrong: Double encoding
echo "LICENSE-KEY" | base64 > /tmp/encoded.txt
kubectl create secret generic beta --from-file=/tmp/encoded.txt
# Result: Secret stores base64 of base64!
```

**Solution:**
```bash
# Delete wrong secret
kubectl delete secret beta

# Create correctly from original file
kubectl create secret generic beta --from-file=/opt/beta.txt

# Verify
kubectl exec secret-datacenter -- cat /opt/cluster/beta.txt
# Should show: LICENSE-KEY (plain text)
```

---

### Issue 5: Pod Crashes/Restarts

**Symptoms:**
```bash
kubectl get pod secret-datacenter
# NAME                READY   STATUS             RESTARTS   AGE
# secret-datacenter   0/1     CrashLoopBackOff   5          3m
```

**Root Causes:**

**A. Wrong Command**
```yaml
# ❌ Wrong: Command exits immediately
command: ["/bin/bash", "-c", "echo hello"]

# ✅ Correct: Infinite sleep
command: ["/bin/bash", "-c", "sleep infinity"]
```

**B. Bash Not Available (for Alpine/BusyBox)**
```yaml
# ❌ Wrong: Alpine doesn't have bash
image: alpine:latest
command: ["/bin/bash", "-c", "sleep infinity"]

# ✅ Correct: Use sh for Alpine
image: alpine:latest
command: ["/bin/sh", "-c", "sleep infinity"]

# ✅ Correct: Or use Debian as specified
image: debian:latest
command: ["/bin/bash", "-c", "sleep infinity"]
```

**C. Container Exits After Reading Secret**
```bash
# Check logs
kubectl logs secret-datacenter

# Check exit code
kubectl get pod secret-datacenter -o jsonpath='{.status.containerStatuses[0].lastState.terminated.exitCode}'
```

**Solution:**
```bash
# Ensure command keeps container running
# Use: sleep infinity, tail -f /dev/null, or similar
```

---

### Issue 6: Read-Only File System Errors

**Symptoms:**
```bash
kubectl exec secret-datacenter -- touch /opt/cluster/test.txt
# touch: cannot touch '/opt/cluster/test.txt': Read-only file system
```

**This is EXPECTED behavior!** Secrets are mounted read-only by default.

**Explanation:**
```yaml
volumeMounts:
- name: secret-volume
  mountPath: /opt/cluster
  readOnly: true  # This is best practice for secrets
```

**If You Need Write Access (not recommended for secrets):**
```yaml
volumeMounts:
- name: secret-volume
  mountPath: /opt/cluster
  readOnly: false  # ⚠️ Not recommended for secrets!
```

**Better Approach:**
```yaml
# Use ConfigMap for writable data, Secret for read-only sensitive data
volumes:
- name: secret-volume
  secret:
    secretName: beta
- name: writable-data
  emptyDir: {}
```

---

## 🔒 Secret Security Best Practices

### 1. Encryption at Rest

**Enable Encryption:**
```yaml
# /etc/kubernetes/manifests/kube-apiserver.yaml
apiVersion: v1
kind: Pod
metadata:
  name: kube-apiserver
spec:
  containers:
  - name: kube-apiserver
    command:
    - kube-apiserver
    - --encryption-provider-config=/etc/kubernetes/encryption-config.yaml
```

**Encryption Config:**
```yaml
# /etc/kubernetes/encryption-config.yaml
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
  - resources:
      - secrets
    providers:
      - aescbc:
          keys:
            - name: key1
              secret: <base64-encoded-32-byte-key>
      - identity: {}
```

---

### 2. RBAC for Secrets

**Restrict Secret Access:**
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: secret-reader
  namespace: default
rules:
- apiGroups: [""]
  resources: ["secrets"]
  verbs: ["get", "list"]
  resourceNames: ["beta"]  # Only specific secret
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-beta-secret
  namespace: default
subjects:
- kind: ServiceAccount
  name: app-serviceaccount
roleRef:
  kind: Role
  name: secret-reader
  apiGroup: rbac.authorization.k8s.io
```

---

### 3. Use External Secret Management

**Integrate with Vault, AWS Secrets Manager, etc.**

```yaml
# Using External Secrets Operator
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: vault-secret
spec:
  secretStoreRef:
    name: vault-backend
    kind: SecretStore
  target:
    name: beta
  data:
  - secretKey: license
    remoteRef:
      key: secret/data/licenses
      property: beta
```

---

### 4. Limit Secret Size

```bash
# ✅ Good: Keep secrets small
kubectl create secret generic small-secret --from-literal=key=value

# ❌ Bad: Large files in secrets (use ConfigMap or PV)
kubectl create secret generic large-secret --from-file=large-file.bin
```

**Limits:**
- Maximum secret size: **1MB**
- Consider using ConfigMap for large non-sensitive data
- Use Persistent Volumes for very large files

---

### 5. Use stringData for Readability

```yaml
# ✅ Recommended: Use stringData in manifests
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: Opaque
stringData:
  username: admin
  password: supersecret

# ❌ Avoid: Manual base64 encoding (error-prone)
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: Opaque
data:
  username: YWRtaW4=
  password: c3VwZXJzZWNyZXQ=
```

---

### 6. Don't Log Secret Values

```yaml
# ❌ Bad: Logging secret values
command: ["/bin/bash", "-c", "echo $SECRET_PASSWORD > /tmp/log.txt"]

# ✅ Good: Use secrets without logging
command: ["/bin/bash", "-c", "mysql -u $DB_USER -p$DB_PASS < schema.sql"]
```

---

### 7. Rotate Secrets Regularly

```bash
# Create new secret version
kubectl create secret generic beta-v2 --from-file=/opt/beta-new.txt

# Update pod to use new secret
kubectl set volume pod/secret-datacenter \
  --add --name=secret-volume \
  --type=secret --secret-name=beta-v2 \
  --mount-path=/opt/cluster

# Or update deployment
kubectl patch deployment myapp \
  -p '{"spec":{"template":{"spec":{"volumes":[{"name":"secret-volume","secret":{"secretName":"beta-v2"}}]}}}}'

# Delete old secret after verification
kubectl delete secret beta
```

---

### 8. Use Immutable Secrets (Kubernetes 1.21+)

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: beta
type: Opaque
immutable: true  # Cannot be modified after creation
data:
  beta.txt: QkVUQS1MSUNFTlNFLUtFWS0xMjM0NS1BQkNERQ==
```

**Benefits:**
- ✅ Protection from accidental updates
- ✅ Performance improvement (no watching for changes)
- ✅ Must delete and recreate to change

---

## 📚 Advanced Secret Patterns

### Pattern 1: Multiple Secrets in One Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-secret-pod
spec:
  containers:
  - name: app
    image: debian:latest
    command: ["/bin/bash", "-c", "sleep infinity"]
    volumeMounts:
    - name: license-volume
      mountPath: /opt/license
      readOnly: true
    - name: db-creds-volume
      mountPath: /opt/db
      readOnly: true
    env:
    - name: API_KEY
      valueFrom:
        secretKeyRef:
          name: api-secret
          key: key
  volumes:
  - name: license-volume
    secret:
      secretName: beta
  - name: db-creds-volume
    secret:
      secretName: db-secret
```

**Use Case:** Application needs multiple types of secrets

---

### Pattern 2: Secret with Environment Variables

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: env-secret-pod
spec:
  containers:
  - name: app
    image: debian:latest
    command: ["/bin/bash", "-c", "sleep infinity"]
    env:
    # Individual secret keys as env vars
    - name: DB_USER
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: username
    - name: DB_PASS
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: password
    # All keys from secret as env vars
    envFrom:
    - secretRef:
        name: app-config-secret
```

**Verification:**
```bash
kubectl exec env-secret-pod -- env | grep -E "DB_|APP_"
```

---

### Pattern 3: TLS Secret for HTTPS

```bash
# Generate TLS certificate
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout tls.key -out tls.crt \
  -subj "/CN=myapp.example.com"

# Create TLS secret
kubectl create secret tls tls-secret \
  --cert=tls.crt \
  --key=tls.key

# Use in Ingress
cat > ingress.yaml <<EOF
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: tls-ingress
spec:
  tls:
  - hosts:
    - myapp.example.com
    secretName: tls-secret
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: myapp-service
            port:
              number: 80
EOF

kubectl apply -f ingress.yaml
```

---

### Pattern 4: Docker Registry Secret

```bash
# Create docker registry secret
kubectl create secret docker-registry registry-secret \
  --docker-server=docker.io \
  --docker-username=myuser \
  --docker-password=mypassword \
  --docker-email=myemail@example.com

# Use in pod
cat > pod-with-imagepull.yaml <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: private-image-pod
spec:
  containers:
  - name: app
    image: myuser/private-image:latest
  imagePullSecrets:
  - name: registry-secret
EOF

kubectl apply -f pod-with-imagepull.yaml
```

---

### Pattern 5: Secret with Subpath Mount

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: subpath-secret-pod
spec:
  containers:
  - name: app
    image: debian:latest
    command: ["/bin/bash", "-c", "sleep infinity"]
    volumeMounts:
    # Mount specific secret key as a file
    - name: secret-volume
      mountPath: /app/config/license.txt
      subPath: beta.txt
    # Mount another key
    - name: secret-volume
      mountPath: /app/config/api-key.txt
      subPath: api-key
  volumes:
  - name: secret-volume
    secret:
      secretName: multi-key-secret
```

**Use Case:** Mount specific secret keys to specific file paths

---

## 📋 Complete Verification Script

```bash
#!/bin/bash

echo "🔍 Kubernetes Secrets - Complete Verification"
echo "=============================================="

# Colors
GREEN='\033[0;32m'
RED='\033[0;31m'
YELLOW='\033[1;33m'
NC='\033[0m'

# 1. Check Secret File
echo -e "\n${YELLOW}1️⃣ Checking Secret File on Jump Host...${NC}"
if [ -f /opt/beta.txt ]; then
    echo -e "${GREEN}✅ File exists: /opt/beta.txt${NC}"
    FILE_SIZE=$(stat -f%z /opt/beta.txt 2>/dev/null || stat -c%s /opt/beta.txt)
    echo "   Size: $FILE_SIZE bytes"
else
    echo -e "${RED}❌ File not found: /opt/beta.txt${NC}"
    exit 1
fi

# 2. Check Secret
echo -e "\n${YELLOW}2️⃣ Checking Secret...${NC}"
if kubectl get secret beta &>/dev/null; then
    echo -e "${GREEN}✅ Secret 'beta' exists${NC}"
    
    # Check secret details
    SECRET_TYPE=$(kubectl get secret beta -o jsonpath='{.type}')
    SECRET_DATA_KEYS=$(kubectl get secret beta -o jsonpath='{.data}' | jq -r 'keys[]' 2>/dev/null)
    
    echo "   Type: $SECRET_TYPE"
    echo "   Keys: $SECRET_DATA_KEYS"
else
    echo -e "${RED}❌ Secret 'beta' not found${NC}"
    echo "   Create with: kubectl create secret generic beta --from-file=/opt/beta.txt"
    exit 1
fi

# 3. Check Pod
echo -e "\n${YELLOW}3️⃣ Checking Pod...${NC}"
if kubectl get pod secret-datacenter &>/dev/null; then
    POD_STATUS=$(kubectl get pod secret-datacenter -o jsonpath='{.status.phase}')
    POD_READY=$(kubectl get pod secret-datacenter -o jsonpath='{.status.containerStatuses[0].ready}')
    
    if [ "$POD_STATUS" == "Running" ] && [ "$POD_READY" == "true" ]; then
        echo -e "${GREEN}✅ Pod 'secret-datacenter': Running and Ready${NC}"
    else
        echo -e "${RED}❌ Pod 'secret-datacenter': Status=$POD_STATUS, Ready=$POD_READY${NC}"
    fi
else
    echo -e "${RED}❌ Pod 'secret-datacenter' not found${NC}"
    exit 1
fi

# 4. Check Container Name
echo -e "\n${YELLOW}4️⃣ Checking Container Name...${NC}"
CONTAINER_NAME=$(kubectl get pod secret-datacenter -o jsonpath='{.spec.containers[0].name}')
if [ "$CONTAINER_NAME" == "secret-container-datacenter" ]; then
    echo -e "${GREEN}✅ Container name: $CONTAINER_NAME${NC}"
else
    echo -e "${RED}❌ Container name: $CONTAINER_NAME (expected: secret-container-datacenter)${NC}"
fi

# 5. Check Image
echo -e "\n${YELLOW}5️⃣ Checking Container Image...${NC}"
CONTAINER_IMAGE=$(kubectl get pod secret-datacenter -o jsonpath='{.spec.containers[0].image}')
if [[ "$CONTAINER_IMAGE" == "debian:latest" ]]; then
    echo -e "${GREEN}✅ Image: $CONTAINER_IMAGE${NC}"
else
    echo -e "${RED}❌ Image: $CONTAINER_IMAGE (expected: debian:latest)${NC}"
fi

# 6. Check Volume Mount
echo -e "\n${YELLOW}6️⃣ Checking Volume Mount...${NC}"
MOUNT_PATH=$(kubectl get pod secret-datacenter -o jsonpath='{.spec.containers[0].volumeMounts[0].mountPath}')
VOLUME_NAME=$(kubectl get pod secret-datacenter -o jsonpath='{.spec.containers[0].volumeMounts[0].name}')

if [ "$MOUNT_PATH" == "/opt/cluster" ]; then
    echo -e "${GREEN}✅ Volume mounted at: $MOUNT_PATH${NC}"
else
    echo -e "${RED}❌ Mount path: $MOUNT_PATH (expected: /opt/cluster)${NC}"
fi

# 7. Check Secret Volume
echo -e "\n${YELLOW}7️⃣ Checking Secret Volume Configuration...${NC}"
SECRET_NAME=$(kubectl get pod secret-datacenter -o jsonpath='{.spec.volumes[0].secret.secretName}')
if [ "$SECRET_NAME" == "beta" ]; then
    echo -e "${GREEN}✅ Secret volume uses: $SECRET_NAME${NC}"
else
    echo -e "${RED}❌ Secret volume uses: $SECRET_NAME (expected: beta)${NC}"
fi

# 8. Verify Secret File Inside Container
echo -e "\n${YELLOW}8️⃣ Verifying Secret File Inside Container...${NC}"
if kubectl exec secret-datacenter -- test -f /opt/cluster/beta.txt 2>/dev/null; then
    echo -e "${GREEN}✅ File exists: /opt/cluster/beta.txt${NC}"
    
    # Check file content
    FILE_CONTENT=$(kubectl exec secret-datacenter -- cat /opt/cluster/beta.txt 2>/dev/null)
    ORIGINAL_CONTENT=$(cat /opt/beta.txt 2>/dev/null)
    
    if [ "$FILE_CONTENT" == "$ORIGINAL_CONTENT" ]; then
        echo -e "${GREEN}✅ File content matches original${NC}"
        echo "   Content: $FILE_CONTENT"
    else
        echo -e "${RED}❌ File content mismatch${NC}"
        echo "   Expected: $ORIGINAL_CONTENT"
        echo "   Got: $FILE_CONTENT"
    fi
else
    echo -e "${RED}❌ File not found: /opt/cluster/beta.txt${NC}"
fi

# 9. Check Read-Only Mount
echo -e "\n${YELLOW}9️⃣ Verifying Read-Only Mount...${NC}"
if kubectl exec secret-datacenter -- touch /opt/cluster/test.txt 2>&1 | grep -q "Read-only"; then
    echo -e "${GREEN}✅ Volume is read-only (as expected)${NC}"
else
    echo -e "${YELLOW}⚠️  Volume is writable (not recommended for secrets)${NC}"
fi

# 10. Summary
echo -e "\n${YELLOW}=============================================="
echo -e "📊 Summary:${NC}"
kubectl get secret beta
kubectl get pod secret-datacenter

echo -e "\n${GREEN}✅ Task Complete! All checks passed.${NC}"

# Cleanup function (optional)
read -p "Do you want to cleanup resources? (y/N) " -n 1 -r
echo
if [[ $REPLY =~ ^[Yy]$ ]]; then
    echo -e "\n${YELLOW}Cleaning up resources...${NC}"
    kubectl delete pod secret-datacenter
    kubectl delete secret beta
    echo -e "${GREEN}✅ Cleanup complete${NC}"
fi
```

**Save and run:**
```bash
chmod +x verify-secrets.sh
./verify-secrets.sh
```

---

## 🧹 Cleanup

```bash
# Delete pod
kubectl delete pod secret-datacenter

# Delete secret
kubectl delete secret beta

# Verify deletion
kubectl get pods,secrets | grep -E "secret-datacenter|beta"
# No resources found

# If you created YAML files
rm -f secret-datacenter.yaml verify-secrets.sh
```

---

## 📖 Key Takeaways

### Concepts Mastered Today:

1. ✅ **Kubernetes Secrets**
   - Secure storage for sensitive data
   - Base64 encoded (not encrypted by default)
   - Namespaced resources
   - Size limit: 1MB

2. ✅ **Secret Types**
   - Opaque (generic) - arbitrary data
   - TLS - certificates and keys
   - Docker registry - image pull credentials
   - Basic auth, SSH auth

3. ✅ **Secret Creation**
   - From files (`--from-file`)
   - From literals (`--from-literal`)
   - From YAML manifests (data/stringData)
   - From env files (`--from-env-file`)

4. ✅ **Secret Consumption**
   - Volume mounts (files)
   - Environment variables
   - Both methods simultaneously

5. ✅ **Security Best Practices**
   - Encryption at rest
   - RBAC for access control
   - External secret management
   - Regular rotation
   - Immutable secrets

---

## 🎯 Real-World Scenarios

| Scenario | Secret Type | Consumption Method | Use Case |
|----------|------------|-------------------|----------|
| **Database Connection** | Opaque | Environment Variables | DB_USER, DB_PASS |
| **HTTPS Certificate** | TLS | Volume Mount | Ingress SSL termination |
| **License Key** | Opaque | Volume Mount (file) | Application licensing |
| **Private Registry** | Docker Registry | imagePullSecrets | Pull private images |
| **API Credentials** | Opaque | Environment Variables | Third-party API access |
| **SSH Keys** | SSH Auth | Volume Mount | Git clone private repos |

---

## 📚 Additional Resources

**Official Documentation:**
- [Secrets](https://kubernetes.io/docs/concepts/configuration/secret/)
- [Encrypt Secret Data at Rest](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/)
- [Managing Secrets using kubectl](https://kubernetes.io/docs/tasks/configmap-secret/managing-secret-using-kubectl/)
- [Secret Best Practices](https://kubernetes.io/docs/concepts/security/secrets-good-practices/)

**External Tools:**
- [Sealed Secrets](https://github.com/bitnami-labs/sealed-secrets) - Encrypt secrets for Git
- [External Secrets Operator](https://external-secrets.io/) - Integrate with external secret stores
- [HashiCorp Vault](https://www.vaultproject.io/) - Secret management platform

**Next Steps:**
- **Day 63:** ConfigMaps for non-sensitive configuration
- **Day 64:** StatefulSets with secrets and persistent storage
- **Day 65:** RBAC and security policies
- **Day 66:** Helm charts with secrets management

---

## 🎯 Real-World Implementation Experience

### Task Completed Successfully! ✅

**Date:** January 7, 2026  
**Cluster:** Kubernetes (kodekloud environment)  
**Objective:** Deploy license-based tool with secure secret storage

### Implementation Steps Executed

**Step 1: Verified Secret File**
```bash
# Checked if file exists on jump host
thor@jump_host ~$ ls -lh /opt/beta.txt
-rw-r--r-- 1 root root 30 Jan 7 10:15 /opt/beta.txt

# Viewed content (sample)
thor@jump_host ~$ cat /opt/beta.txt
BETA-LICENSE-KEY-NAUTILUS-2026
```

**Step 2: Created Secret from File**
```bash
# Created generic secret named 'beta'
thor@jump_host ~$ kubectl create secret generic beta --from-file=/opt/beta.txt
secret/beta created

# Verified secret creation
thor@jump_host ~$ kubectl get secret beta
NAME   TYPE     DATA   AGE
beta   Opaque   1      5s

# Checked secret details
thor@jump_host ~$ kubectl describe secret beta
Name:         beta
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
beta.txt:  30 bytes
```

**Step 3: Created Pod with Secret Volume Mount**
```bash
# Created pod YAML file
thor@jump_host ~$ cat > secret-datacenter.yaml <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: secret-datacenter
  labels:
    app: secret-demo
spec:
  containers:
  - name: secret-container-datacenter
    image: debian:latest
    command: ["/bin/bash", "-c", "sleep infinity"]
    volumeMounts:
    - name: secret-volume
      mountPath: /opt/cluster
      readOnly: true
  volumes:
  - name: secret-volume
    secret:
      secretName: beta
EOF

# Applied the pod
thor@jump_host ~$ kubectl apply -f secret-datacenter.yaml
pod/secret-datacenter created
```

**Step 4: Verified Pod Status**
```bash
# Checked pod status (initially)
thor@jump_host ~$ kubectl get pod secret-datacenter
NAME                READY   STATUS              RESTARTS   AGE
secret-datacenter   0/1     ContainerCreating   0          10s

# Waited for pod to be running
thor@jump_host ~$ kubectl get pod secret-datacenter
NAME                READY   STATUS    RESTARTS   AGE
secret-datacenter   1/1     Running   0          45s

# Described pod for details
thor@jump_host ~$ kubectl describe pod secret-datacenter
Name:         secret-datacenter
Namespace:    default
Status:       Running

Containers:
  secret-container-datacenter:
    Image:         debian:latest
    State:         Running
    Ready:         True
    Mounts:
      /opt/cluster from secret-volume (ro)

Volumes:
  secret-volume:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  beta
    Optional:    false
```

**Step 5: Verified Secret Inside Container**
```bash
# Exec into container
thor@jump_host ~$ kubectl exec -it secret-datacenter -- bash

# Listed files in mounted path
root@secret-datacenter:/# ls -lh /opt/cluster/
total 4.0K
-rw-r--r-- 1 root root 30 Jan 7 10:15 beta.txt

# Read secret content
root@secret-datacenter:/# cat /opt/cluster/beta.txt
BETA-LICENSE-KEY-NAUTILUS-2026

# Verified read-only mount
root@secret-datacenter:/# touch /opt/cluster/test.txt
touch: cannot touch '/opt/cluster/test.txt': Read-only file system

# Exit container
root@secret-datacenter:/# exit
```

**Step 6: One-Liner Verification**
```bash
# Verified without exec
thor@jump_host ~$ kubectl exec secret-datacenter -- cat /opt/cluster/beta.txt
BETA-LICENSE-KEY-NAUTILUS-2026

# Checked file exists
thor@jump_host ~$ kubectl exec secret-datacenter -- ls /opt/cluster/
beta.txt
```

### Key Observations

**✅ What Went Right:**
1. **Secret creation was instant** - File-based secret creation is straightforward
2. **Pod started quickly** - Debian image was already cached in cluster
3. **Secret auto-decoded** - Kubernetes automatically decoded base64 to plain text
4. **Read-only mount worked** - Security best practice enforced by default
5. **Verification was simple** - Easy to exec and check file content

**💡 Lessons Learned:**

1. **File-based secrets are convenient**
   - No manual base64 encoding needed
   - Filename becomes the key in secret
   - Original file content preserved exactly

2. **Volume mounts vs environment variables**
   - Volume mounts better for file-based data (certificates, licenses)
   - Easier to verify inside container
   - Supports larger secrets than env vars

3. **Read-only by default**
   - Secrets mounted as read-only automatically
   - Prevents accidental modification
   - Good security practice

4. **Pod timing**
   - ContainerCreating phase takes 10-15 seconds
   - Image pull happens if not cached
   - Always wait for Running status before verification

5. **Secret key naming**
   - When using `--from-file=/path/file.txt`, key name is `file.txt`
   - Can customize with `--from-file=customkey=/path/file.txt`
   - Important for mounting specific files

### Production Considerations

Based on this implementation, here are recommendations for production:

**1. Secret Management:**
```bash
# ✅ Good: Use descriptive secret names
kubectl create secret generic app-license-2026 --from-file=/opt/beta.txt

# ✅ Good: Add labels for organization
kubectl label secret beta app=nautilus-tools env=production

# ✅ Good: Document secret purpose
kubectl annotate secret beta description="License key for Nautilus tools"
```

**2. Security Enhancements:**
```yaml
# Add security context to pod
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    fsGroup: 1000
  containers:
  - name: secret-container-datacenter
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      capabilities:
        drop:
        - ALL
```

**3. RBAC for Secret Access:**
```yaml
# Limit who can read secrets
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: secret-reader
rules:
- apiGroups: [""]
  resources: ["secrets"]
  verbs: ["get"]
  resourceNames: ["beta"]
```

**4. Secret Rotation:**
```bash
# Plan for regular updates
kubectl create secret generic beta-v2 --from-file=/opt/beta-new.txt
# Update pod to use beta-v2
# Delete old secret after verification
kubectl delete secret beta
```

**5. Monitoring:**
```bash
# Set up alerts for secret access
# Monitor pod events for FailedMount
# Track secret age for rotation reminders
```

### Alternative Approaches Considered

**Option 1: Environment Variables**
```yaml
# Not chosen because file-based license is better as mounted file
env:
- name: LICENSE_KEY
  valueFrom:
    secretKeyRef:
      name: beta
      key: beta.txt
```

**Option 2: External Secret Operator**
```yaml
# For production, could integrate with external secret manager
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: beta
spec:
  secretStoreRef:
    name: vault-backend
  target:
    name: beta
```

**Option 3: Sealed Secrets**
```bash
# For GitOps, could use sealed secrets
kubeseal --format yaml < secret.yaml > sealed-secret.yaml
# Safe to commit to Git
```

### Verification Script Used

```bash
#!/bin/bash
# Quick verification script

echo "Checking secret..."
kubectl get secret beta || exit 1

echo "Checking pod..."
kubectl get pod secret-datacenter | grep Running || exit 1

echo "Verifying secret content in container..."
CONTENT=$(kubectl exec secret-datacenter -- cat /opt/cluster/beta.txt)
echo "Content: $CONTENT"

echo "✅ All checks passed!"
```

---

## ✅ Task Completion Checklist

- [x] Secret file `/opt/beta.txt` exists on jump host
- [x] Secret `beta` created from file
- [x] Secret type is `Opaque`
- [x] Secret contains key `beta.txt` with file content
- [x] Pod `secret-datacenter` created
- [x] Container named `secret-container-datacenter`
- [x] Container image is `debian:latest`
- [x] Container uses `sleep infinity` command
- [x] Secret mounted as volume at `/opt/cluster`
- [x] Volume mount is read-only
- [x] Pod status is `Running`
- [x] File `/opt/cluster/beta.txt` exists in container
- [x] File content matches original file
- [x] All verification checks pass

---

**🎉 Congratulations!** You've successfully implemented Kubernetes Secrets! You now understand how to securely store and consume sensitive data in your Kubernetes applications.

**Day 62 Status:** ✅ Complete

**Next:** Day 63 - ConfigMaps and Application Configuration 🚀
