# Day 64: Kubernetes Troubleshooting - Python Flask Application Deployment

## 📋 Task Overview

**Scenario:** A DevOps engineer attempted to deploy a Python Flask application on the Kubernetes cluster but encountered misconfiguration issues. The application is not accessible, and you need to troubleshoot and fix the deployment.

**Given Information:**
- Deployment name: `python-deployment-nautilus`
- Image: `poroko/flask-demo-app`
- Deployment and service already exist (but misconfigured)
- Required nodePort: `32345`
- Required targetPort: Flask default port (5000)

**Your Mission:**
1. Identify misconfigurations in deployment and service
2. Fix the image name in deployment
3. Fix the targetPort in service to 5000
4. Ensure application is accessible on nodePort 32345
5. Verify Flask app responds correctly

---

## 🎯 Learning Objectives

By the end of this task, you will understand:
- **Systematic Troubleshooting:** Step-by-step debugging methodology
- **Common Misconfigurations:** Typical deployment and service errors
- **Port Mapping:** Understanding port, targetPort, and nodePort relationships
- **Label Selectors:** How services route traffic to pods
- **Flask Applications:** Default port and container configuration
- **Verification Techniques:** Testing and validating fixes

---

## 📖 Understanding Flask Applications

### What is Flask?

**Flask** is a lightweight Python web framework commonly used for building web applications and APIs.

**Default Configuration:**
- **Default Port:** 5000
- **Default Host:** 127.0.0.1 (localhost only)
- **Production Port:** Often runs on 5000 or 8080
- **Health Check:** Usually `GET /` or `GET /health`

### Flask in Containers

```python
# Typical Flask app entry point
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello():
    return 'Hello from Flask!'

if __name__ == '__main__':
    # Container must listen on 0.0.0.0 to accept external traffic
    app.run(host='0.0.0.0', port=5000)
```

**Key Point:** Flask must bind to `0.0.0.0` (all interfaces) in containers, not `127.0.0.1` (localhost only).

---

## 🔍 Troubleshooting Methodology

### The Systematic Approach

```
┌─────────────────────────────────────────────────────────────┐
│          Kubernetes Troubleshooting Workflow                 │
└─────────────────────────────────────────────────────────────┘

Step 1: Get Overview
   ↓
kubectl get all
   ↓
Identify resource status

Step 2: Check Deployment
   ↓
kubectl get deployment
kubectl describe deployment
   ↓
Check replicas, image, labels

Step 3: Check Pods
   ↓
kubectl get pods
kubectl describe pod
kubectl logs pod
   ↓
Check status, events, application logs

Step 4: Check Service
   ↓
kubectl get svc
kubectl describe svc
   ↓
Check type, ports, selector, endpoints

Step 5: Test Connectivity
   ↓
curl http://node-ip:nodePort
   ↓
Verify application responds

Step 6: Fix Issues
   ↓
kubectl edit deployment
kubectl edit svc
   ↓
Apply corrections

Step 7: Verify Fix
   ↓
kubectl rollout status
kubectl get pods
curl test
   ↓
Confirm application works
```

---

## 🛠️ Task Implementation

### Step 1: Initial Assessment

**Check all resources:**
```bash
# Get all resources in default namespace
kubectl get all

# Expected output (with issues):
# NAME                                            READY   STATUS    RESTARTS   AGE
# pod/python-deployment-nautilus-xxxxx-xxxxx      0/1     Running   0          5m
#
# NAME                                TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
# service/kubernetes                  ClusterIP   10.96.0.1       <none>        443/TCP          10d
# service/python-service-nautilus     NodePort    10.96.123.45    <none>        8080:32345/TCP   5m
#
# NAME                                     READY   UP-TO-DATE   AVAILABLE   AGE
# deployment.apps/python-deployment-nautilus   0/1     1            0           5m
```

**Initial Observations:**
- ✅ Deployment exists
- ✅ Pod is running
- ✅ Service exists
- ❌ Deployment shows 0/1 READY
- ❌ Deployment shows 0 AVAILABLE
- ⚠️ Service port might be misconfigured

---

### Step 2: Check Deployment Details

**Describe deployment:**
```bash
kubectl describe deployment python-deployment-nautilus
```

**Expected output (with issues):**
```yaml
Name:                   python-deployment-nautilus
Namespace:              default
CreationTimestamp:      Thu, 09 Jan 2026 10:00:00 +0000
Labels:                 app=python-app
Selector:               app=python-app
Replicas:               1 desired | 1 updated | 1 total | 0 available
StrategyType:           RollingUpdate
Pod Template:
  Labels:  app=python-app
  Containers:
   python-container:
    Image:        poroko/flask-demo-app
    Port:         5000/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      False   MinimumReplicasUnavailable
  Progressing    True    ReplicaSetUpdated
Events:
  Type    Reason             Age   Message
  ----    ------             ----  -------
  Normal  ScalingReplicaSet  5m    Scaled up replica set python-deployment-nautilus-xxx to 1
```

**Key Information:**
- Image: `poroko/flask-demo-app` ✅
- Container Port: `5000` ✅ (Flask default)
- Labels: `app=python-app` ✅
- Replicas: 1 desired, 0 available ❌

---

### Step 3: Check Pod Status

**Get pods:**
```bash
kubectl get pods -l app=python-app
```

**Expected output:**
```
NAME                                         READY   STATUS    RESTARTS   AGE
python-deployment-nautilus-7b8f9d5-xyz12    0/1     Running   0          5m
```

**Describe pod:**
```bash
POD_NAME=$(kubectl get pods -l app=python-app -o jsonpath='{.items[0].metadata.name}')
kubectl describe pod $POD_NAME
```

**Key sections to check:**
```yaml
Name:         python-deployment-nautilus-7b8f9d5-xyz12
Status:       Running
IP:           10.244.1.5
Containers:
  python-container:
    Container ID:   containerd://abc123...
    Image:          poroko/flask-demo-app
    Port:           5000/TCP
    State:          Running
    Ready:          False  ❌
    Restart Count:  0
    
Conditions:
  Type              Status
  Initialized       True
  Ready             False   ❌
  ContainersReady   False   ❌
  PodScheduled      True

Events:
  Type     Reason     Age   Message
  ----     ------     ----  -------
  Normal   Scheduled  5m    Successfully assigned default/python-deployment-nautilus-xxx
  Normal   Pulling    5m    Pulling image "poroko/flask-demo-app"
  Normal   Pulled     4m    Successfully pulled image
  Normal   Created    4m    Created container python-container
  Normal   Started    4m    Started container python-container
  Warning  Unhealthy  3m    Readiness probe failed: Get http://10.244.1.5:5000/: dial tcp 10.244.1.5:5000: connect: connection refused
```

**Critical Finding:** Readiness probe failing! ❌

---

### Step 4: Check Pod Logs

**View application logs:**
```bash
kubectl logs $POD_NAME
```

**Expected output:**
```
 * Serving Flask app 'app'
 * Debug mode: off
WARNING: This is a development server. Do not use it in a production deployment.
 * Running on http://0.0.0.0:5000
Press CTRL+C to quit
```

**Analysis:**
- ✅ Flask is running
- ✅ Listening on port 5000
- ✅ Bound to 0.0.0.0 (all interfaces)
- ⚠️ But readiness probe is failing

---

### Step 5: Check Service Configuration

**Get service details:**
```bash
kubectl get svc python-service-nautilus
```

**Expected output (with issues):**
```
NAME                      TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
python-service-nautilus   NodePort   10.96.123.45    <none>        8080:32345/TCP   5m
```

**Describe service:**
```bash
kubectl describe svc python-service-nautilus
```

**Expected output (with issues):**
```yaml
Name:                     python-service-nautilus
Namespace:                default
Labels:                   <none>
Selector:                 app=python-app
Type:                     NodePort
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.96.123.45
Port:                     <unset>  8080/TCP  ❌ Wrong!
TargetPort:               8080/TCP  ❌ Wrong! (Should be 5000)
NodePort:                 <unset>  32345/TCP  ✅
Endpoints:                10.244.1.5:8080  ❌ Wrong port!
Session Affinity:         None
External Traffic Policy:  Cluster

Events:  <none>
```

**Critical Issues Found:**

| Component | Current | Expected | Status |
|-----------|---------|----------|--------|
| **Service Port** | 8080 | 80 or 5000 | ❌ Wrong |
| **TargetPort** | 8080 | 5000 (Flask default) | ❌ Wrong |
| **NodePort** | 32345 | 32345 | ✅ Correct |
| **Endpoints** | 10.244.1.5:8080 | 10.244.1.5:5000 | ❌ Wrong |

**Root Cause:** Service is trying to connect to pods on port 8080, but Flask is running on port 5000!

---

### Step 6: Check Deployment Configuration Issues

**Get deployment YAML:**
```bash
kubectl get deployment python-deployment-nautilus -o yaml
```

**Look for readiness probe:**
```yaml
spec:
  containers:
  - name: python-container
    image: poroko/flask-demo-app
    ports:
    - containerPort: 5000
    readinessProbe:
      httpGet:
        path: /
        port: 5000
      initialDelaySeconds: 5
      periodSeconds: 10
```

**Issue:** If readiness probe exists and is failing, pod shows as not ready.

---

### Step 7: Fix the Issues

## 🔧 The Two Fixes Required

Based on troubleshooting, two changes were needed:

### Fix 1: Update Deployment Image

**Issue:** Wrong or misconfigured image in deployment

**Solution:**
```bash
# Edit the deployment
kubectl edit deployment python-deployment-nautilus

# Update the image name to the correct one
# Change: image: <wrong-image>
# To: image: poroko/flask-demo-app
# Save and exit
```

**Alternative - Using set image command:**
```bash
kubectl set image deployment/python-deployment-nautilus python-container=poroko/flask-demo-app
```

**Expected output:**
```
deployment.apps/python-deployment-nautilus image updated
```

---

### Fix 2: Update Service targetPort to 5000

**Issue:** Service targetPort was not set to Flask's default port (5000)

**Current service (broken):**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: python-service-nautilus
spec:
  selector:
    app: python-app
  type: NodePort
  ports:
  - port: 8080        # Wrong
    targetPort: 8080  # ❌ Wrong (Flask runs on 5000!)
    nodePort: 32345   # ✅ Correct
```

**Solution:**
```bash
# Edit the service
kubectl edit svc python-service-nautilus

# Change in the editor:
# targetPort: 8080 → targetPort: 5000
# Save and exit
```

**Corrected service:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: python-service-nautilus
spec:
  selector:
    app: python-app
  type: NodePort
  ports:
  - port: 80          # Updated
    targetPort: 5000  # ✅ Fixed - Flask default port
    nodePort: 32345   # ✅ Correct
    protocol: TCP
```

**Expected output:**
```
service/python-service-nautilus edited
```

**Alternative methods to update service:**

**Method 2: Apply YAML file**
```bash
# Create corrected service YAML
cat > python-service-fixed.yaml <<EOF
apiVersion: v1
kind: Service
metadata:
  name: python-service-nautilus
spec:
  selector:
    app: python-app
  type: NodePort
  ports:
  - port: 80
    targetPort: 5000
    nodePort: 32345
    protocol: TCP
EOF

# Apply the fix
kubectl apply -f python-service-fixed.yaml
```

**Method 3: Patch command**
```bash
kubectl patch svc python-service-nautilus -p '{"spec":{"ports":[{"port":80,"targetPort":5000,"nodePort":32345,"protocol":"TCP"}]}}'
```

**That's it! Only these two changes were needed:**
1. ✅ Update deployment image name
2. ✅ Update service targetPort to 5000

---

### Step 8: Verify the Fixes

**Wait for deployment to roll out:**
```bash
# Watch deployment rollout
kubectl rollout status deployment/python-deployment-nautilus
```

**Expected output:**
```
Waiting for deployment "python-deployment-nautilus" rollout to finish: 0 of 1 updated replicas are available...
deployment "python-deployment-nautilus" successfully rolled out
```

**Check updated service:**
```bash
kubectl get svc python-service-nautilus
```

**Expected output (fixed):**
```
NAME                      TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
python-service-nautilus   NodePort   10.96.123.45    <none>        80:32345/TCP   10m
```

**Describe service to verify:**
```bash
kubectl describe svc python-service-nautilus
```

**Expected output (fixed):**
```yaml
Name:                     python-service-nautilus
Namespace:                default
Selector:                 app=python-app
Type:                     NodePort
IP:                       10.96.123.45
Port:                     <unset>  80/TCP  ✅
TargetPort:               5000/TCP  ✅
NodePort:                 <unset>  32345/TCP  ✅
Endpoints:                10.244.1.5:5000  ✅
Session Affinity:         None
```

**Key Verifications:**
- ✅ Port: 80
- ✅ TargetPort: 5000 (Flask default)
- ✅ NodePort: 32345 (as required)
- ✅ Endpoints: Shows pod IP with port 5000

---

### Step 9: Check Pod Readiness

**Wait for pods to become ready:**
```bash
kubectl get pods -l app=python-app -w
```

**Expected progression:**
```
NAME                                         READY   STATUS    RESTARTS   AGE
python-deployment-nautilus-7b8f9d5-xyz12    0/1     Running   0          10m
python-deployment-nautilus-7b8f9d5-xyz12    1/1     Running   0          10m20s  ✅
```

**Check deployment status:**
```bash
kubectl get deployment python-deployment-nautilus
```

**Expected output (fixed):**
```
NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
python-deployment-nautilus   1/1     1            1           15m  ✅
```

---

### Step 10: Test Application Access

**Get node IP:**
```bash
kubectl get nodes -o wide
```

**Expected output:**
```
NAME     STATUS   ROLES    AGE   VERSION   INTERNAL-IP    EXTERNAL-IP
node01   Ready    <none>   10d   v1.27.0   172.17.0.2     <none>
```

**Test application using curl:**
```bash
# Test from jump host
curl http://172.17.0.2:32345

# Or using node name
NODE_IP=$(kubectl get nodes -o jsonpath='{.items[0].status.addresses[?(@.type=="InternalIP")].address}')
curl http://$NODE_IP:32345
```

**Expected output:**
```html
<!DOCTYPE html>
<html>
<head>
    <title>Flask Demo App</title>
</head>
<body>
    <h1>Hello from Flask!</h1>
    <p>This is a demo Python Flask application running on Kubernetes.</p>
</body>
</html>
```

**Or simpler output:**
```
Hello from Flask!
```

**Test with verbose output:**
```bash
curl -v http://$NODE_IP:32345
```

**Expected response:**
```
* Trying 172.17.0.2:32345...
* Connected to 172.17.0.2 (172.17.0.2) port 32345 (#0)
> GET / HTTP/1.1
> Host: 172.17.0.2:32345
> User-Agent: curl/7.68.0
> Accept: */*
>
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< Server: Werkzeug/2.0.1 Python/3.9.5
< Date: Thu, 09 Jan 2026 10:15:00 GMT
< Content-Type: text/html; charset=utf-8
< Content-Length: 150
<
Hello from Flask!
* Connection #0 to host 172.17.0.2 left intact
```

**Key indicators:**
- ✅ HTTP 200 OK status
- ✅ Response body received
- ✅ Connection successful

---

### Step 12: Test from Browser (Optional)

**Access in browser:**
```
http://<node-ip>:32345
```

**Example:**
```
http://172.17.0.2:32345
```

**Expected:** Web page loads with Flask application content

---

## 🐛 Common Issues and Solutions

### Issue 1: Wrong Image Name

**Symptoms:**
```bash
kubectl get pods
# NAME                                         READY   STATUS             RESTARTS   AGE
# python-deployment-nautilus-xxx               0/1     ErrImagePull       0          2m
# Or
# python-deployment-nautilus-xxx               0/1     ImagePullBackOff   0          3m
```

**Diagnosis:**
```bash
kubectl describe pod <pod-name>

# Events:
#   Failed to pull image "wrong/image-name": manifest not found
#   Or: image pull failed for <image-name>
```

**Solution:**
```bash
# Update deployment with correct image
kubectl set image deployment/python-deployment-nautilus python-container=poroko/flask-demo-app

# Or edit directly
kubectl edit deployment python-deployment-nautilus
# Change image to: poroko/flask-demo-app
```

---

### Issue 2: Wrong targetPort in Service

**Symptoms:**
```bash
curl http://$NODE_IP:32345
# curl: (52) Empty reply from server
# Or: Connection refused
```

**Diagnosis:**
```bash
# Check service targetPort
kubectl describe svc python-service-nautilus | grep TargetPort

# TargetPort: 8080/TCP  ❌ (Flask runs on 5000!)

# Check what port Flask is actually listening on
kubectl logs <pod-name>
# Output: Running on http://0.0.0.0:5000
```

**Solution:**
```bash
# Update service targetPort to 5000
kubectl edit svc python-service-nautilus

# Change targetPort to 5000
spec:
  ports:
  - port: 80
    targetPort: 5000  # ✅ Flask default port
    nodePort: 32345
```

---

### Issue 3: Service Still Not Working After Fix

**Symptoms:**
```bash
curl http://$NODE_IP:32345
# curl: (7) Failed to connect to 172.17.0.2 port 32345: Connection refused
```

**Diagnosis:**
```bash
# Check if endpoints exist
kubectl get endpoints python-service-nautilus

# Empty or wrong port?
# NAME                      ENDPOINTS          AGE
# python-service-nautilus   <none>             15m  ❌
```

**Root Causes:**

**A. Selector Mismatch**
```bash
# Check service selector
kubectl get svc python-service-nautilus -o yaml | grep -A2 selector

# Check pod labels
kubectl get pods --show-labels
```

**Solution:**
```bash
# Fix selector in service
kubectl edit svc python-service-nautilus

# Ensure selector matches pod labels:
spec:
  selector:
    app: python-app  # Must match pod labels
```

---

**B. Pod Not Ready**
```bash
kubectl get pods -l app=python-app

# NAME                                         READY   STATUS    RESTARTS   AGE
# python-deployment-nautilus-xxx               0/1     Running   0          5m  ❌
```

**Solution:**
```bash
# Check why pod is not ready
kubectl describe pod <pod-name>

# Look for readiness probe failures
# Fix or remove problematic readiness probe
kubectl edit deployment python-deployment-nautilus
```

---

### Issue 2: Wrong Image or Image Pull Errors

**Symptoms:**
```bash
kubectl get pods
# NAME                                         READY   STATUS         RESTARTS   AGE
# python-deployment-nautilus-xxx               0/1     ErrImagePull   0          2m
```

**Diagnosis:**
```bash
kubectl describe pod <pod-name>

# Events:
#   Failed to pull image "poroko/flask-demo-app": manifest not found
```

**Solution:**
```bash
# Verify correct image name
docker pull poroko/flask-demo-app

# If image doesn't exist, find correct image
# Example alternatives:
# - kodekloud/webapp-color
# - nginx
# - httpd

# Update deployment with correct image
kubectl set image deployment/python-deployment-nautilus python-container=<correct-image>
```

---

### Issue 3: Port Mismatch

**Symptoms:**
```bash
curl http://$NODE_IP:32345
# curl: (52) Empty reply from server
```

**Diagnosis:**
```bash
# Check what port the application is actually listening on
kubectl logs <pod-name>

# Output: Running on http://0.0.0.0:8080
# But targetPort is set to 5000!
```

**Solution:**
```bash
# Update service targetPort to match application port
kubectl edit svc python-service-nautilus

# Change targetPort to match application
spec:
  ports:
  - port: 80
    targetPort: 8080  # Match application's actual port
    nodePort: 32345
```

---

### Issue 4: Readiness Probe Blocking Traffic

**Symptoms:**
```bash
kubectl get pods
# NAME                                         READY   STATUS    RESTARTS   AGE
# python-deployment-nautilus-xxx               0/1     Running   0          5m

kubectl describe pod <pod-name>
# Warning  Unhealthy  Readiness probe failed: HTTP probe failed with statuscode: 404
```

**Root Cause:** Readiness probe checking non-existent endpoint

**Solution:**

**Option 1: Fix probe endpoint**
```bash
kubectl edit deployment python-deployment-nautilus

# Change probe path to valid endpoint
readinessProbe:
  httpGet:
    path: /       # or /health if app has it
    port: 5000
```

**Option 2: Use TCP probe instead**
```bash
kubectl edit deployment python-deployment-nautilus

# Replace httpGet with tcpSocket
readinessProbe:
  tcpSocket:
    port: 5000
  initialDelaySeconds: 5
  periodSeconds: 10
```

**Option 3: Remove readiness probe (temporary)**
```bash
kubectl edit deployment python-deployment-nautilus

# Delete entire readinessProbe section
# Save and exit
```

---

### Issue 5: NodePort Not Accessible

**Symptoms:**
```bash
curl http://$NODE_IP:32345
# Connection timeout or refused
```

**Diagnosis:**
```bash
# Check if nodePort is correct
kubectl get svc python-service-nautilus -o yaml | grep nodePort

# Check firewall rules (if applicable)
# Check if service type is NodePort
kubectl get svc python-service-nautilus -o yaml | grep type
```

**Solution:**
```bash
# Ensure service type is NodePort
kubectl edit svc python-service-nautilus

spec:
  type: NodePort
  ports:
  - port: 80
    targetPort: 5000
    nodePort: 32345

# Test from inside cluster first
kubectl run test-pod --image=busybox --rm -it --restart=Never -- wget -O- http://python-service-nautilus
```

---

### Issue 6: Multiple Pods in CrashLoopBackOff

**Symptoms:**
```bash
kubectl get pods
# NAME                                         READY   STATUS             RESTARTS   AGE
# python-deployment-nautilus-xxx               0/1     CrashLoopBackOff   5          5m
```

**Diagnosis:**
```bash
# Check logs for errors
kubectl logs <pod-name>

# Common errors:
# - Module not found
# - Configuration error
# - Port already in use
# - Permission denied
```

**Solution:**
```bash
# View full error
kubectl logs <pod-name> --previous

# If image is broken, use known working image
kubectl set image deployment/python-deployment-nautilus python-container=kodekloud/webapp-color
```

---

## 📊 Complete Troubleshooting Summary

### Issue Checklist

| Check | Command | Expected | Common Issue |
|-------|---------|----------|--------------|
| **Deployment Status** | `kubectl get deployment` | 1/1 READY | 0/1 available |
| **Pod Status** | `kubectl get pods` | Running, 1/1 READY | Not ready |
| **Pod Events** | `kubectl describe pod` | No errors | Readiness probe failed |
| **Service Type** | `kubectl get svc` | NodePort | ClusterIP |
| **Service Ports** | `kubectl describe svc` | targetPort: 5000 | Wrong targetPort |
| **Endpoints** | `kubectl get endpoints` | Pod IPs present | Empty endpoints |
| **App Logs** | `kubectl logs` | App running | Errors present |
| **Connectivity** | `curl http://ip:32345` | 200 OK response | Connection refused |

---

## 🎯 Best Practices

### 1. Always Check Logs First

```bash
# ✅ Good: Check logs for application errors
kubectl logs <pod-name>
kubectl logs <pod-name> --previous  # Previous container logs
kubectl logs -f <pod-name>          # Follow logs

# Check all containers if multi-container pod
kubectl logs <pod-name> -c <container-name>
```

---

### 2. Use Describe for Events

```bash
# ✅ Good: Events show timeline of issues
kubectl describe pod <pod-name>
kubectl describe deployment <deployment-name>
kubectl describe svc <service-name>

# Look for Warning events
kubectl get events --sort-by='.lastTimestamp'
```

---

### 3. Verify Port Consistency

```bash
# ✅ Good: Ensure port chain is correct
# Container Port (5000) → TargetPort (5000) → Service Port (80) → NodePort (32345)

# Check container port
kubectl get pod <pod-name> -o yaml | grep containerPort

# Check service ports
kubectl get svc <service-name> -o yaml | grep -A5 ports
```

---

### 4. Test Internal Connectivity First

```bash
# ✅ Good: Test from inside cluster
kubectl run test-pod --image=busybox --rm -it --restart=Never -- wget -O- http://python-service-nautilus:80

# Then test NodePort externally
curl http://$NODE_IP:32345
```

---

### 5. Use Labels Consistently

```bash
# ✅ Good: Consistent labels
# Deployment labels → Pod labels → Service selector

# Check deployment labels
kubectl get deployment <name> -o yaml | grep -A5 labels

# Check service selector
kubectl get svc <name> -o yaml | grep -A5 selector

# They must match!
```

---

### 6. Document Changes

```bash
# ✅ Good: Keep track of what you changed
kubectl annotate deployment python-deployment-nautilus \
  troubleshooting="Fixed service targetPort from 8080 to 5000 on $(date)"

# View annotations
kubectl describe deployment python-deployment-nautilus | grep Annotations
```

---

## 📋 Complete Verification Script

```bash
#!/bin/bash

echo "🔍 Python Flask Application - Complete Troubleshooting"
echo "======================================================"

# Colors
GREEN='\033[0;32m'
RED='\033[0;31m'
YELLOW='\033[1;33m'
NC='\033[0m'

DEPLOYMENT="python-deployment-nautilus"
SERVICE="python-service-nautilus"
APP_LABEL="app=python-app"

# 1. Check Deployment
echo -e "\n${YELLOW}1️⃣ Checking Deployment...${NC}"
if kubectl get deployment $DEPLOYMENT &>/dev/null; then
    READY=$(kubectl get deployment $DEPLOYMENT -o jsonpath='{.status.readyReplicas}')
    DESIRED=$(kubectl get deployment $DEPLOYMENT -o jsonpath='{.spec.replicas}')
    
    if [ "$READY" == "$DESIRED" ]; then
        echo -e "${GREEN}✅ Deployment: $READY/$DESIRED ready${NC}"
    else
        echo -e "${RED}❌ Deployment: $READY/$DESIRED ready (FIX NEEDED)${NC}"
    fi
else
    echo -e "${RED}❌ Deployment not found${NC}"
    exit 1
fi

# 2. Check Pods
echo -e "\n${YELLOW}2️⃣ Checking Pods...${NC}"
POD_NAME=$(kubectl get pods -l $APP_LABEL -o jsonpath='{.items[0].metadata.name}' 2>/dev/null)

if [ -n "$POD_NAME" ]; then
    POD_STATUS=$(kubectl get pod $POD_NAME -o jsonpath='{.status.phase}')
    POD_READY=$(kubectl get pod $POD_NAME -o jsonpath='{.status.containerStatuses[0].ready}')
    
    if [ "$POD_STATUS" == "Running" ] && [ "$POD_READY" == "true" ]; then
        echo -e "${GREEN}✅ Pod: $POD_NAME - Running and Ready${NC}"
    else
        echo -e "${RED}❌ Pod: $POD_NAME - Status=$POD_STATUS, Ready=$POD_READY${NC}"
        
        # Show events if not ready
        echo -e "${YELLOW}   Recent events:${NC}"
        kubectl describe pod $POD_NAME | grep -A10 Events:
    fi
else
    echo -e "${RED}❌ No pods found${NC}"
    exit 1
fi

# 3. Check Application Logs
echo -e "\n${YELLOW}3️⃣ Checking Application Logs...${NC}"
LOG_OUTPUT=$(kubectl logs $POD_NAME --tail=5 2>/dev/null)
if echo "$LOG_OUTPUT" | grep -q "Running on"; then
    echo -e "${GREEN}✅ Application running${NC}"
    echo "$LOG_OUTPUT" | grep "Running on"
else
    echo -e "${RED}❌ Application issues detected${NC}"
    echo "$LOG_OUTPUT"
fi

# 4. Check Service Configuration
echo -e "\n${YELLOW}4️⃣ Checking Service...${NC}"
if kubectl get svc $SERVICE &>/dev/null; then
    SVC_TYPE=$(kubectl get svc $SERVICE -o jsonpath='{.spec.type}')
    TARGET_PORT=$(kubectl get svc $SERVICE -o jsonpath='{.spec.ports[0].targetPort}')
    NODE_PORT=$(kubectl get svc $SERVICE -o jsonpath='{.spec.ports[0].nodePort}')
    
    echo -e "   Type: $SVC_TYPE"
    echo -e "   TargetPort: $TARGET_PORT"
    echo -e "   NodePort: $NODE_PORT"
    
    # Validate
    if [ "$SVC_TYPE" == "NodePort" ]; then
        echo -e "${GREEN}✅ Service type: NodePort${NC}"
    else
        echo -e "${RED}❌ Service type: $SVC_TYPE (should be NodePort)${NC}"
    fi
    
    if [ "$TARGET_PORT" == "5000" ]; then
        echo -e "${GREEN}✅ TargetPort: 5000 (Flask default)${NC}"
    else
        echo -e "${RED}❌ TargetPort: $TARGET_PORT (should be 5000)${NC}"
    fi
    
    if [ "$NODE_PORT" == "32345" ]; then
        echo -e "${GREEN}✅ NodePort: 32345 (correct)${NC}"
    else
        echo -e "${RED}❌ NodePort: $NODE_PORT (should be 32345)${NC}"
    fi
else
    echo -e "${RED}❌ Service not found${NC}"
    exit 1
fi

# 5. Check Endpoints
echo -e "\n${YELLOW}5️⃣ Checking Service Endpoints...${NC}"
ENDPOINTS=$(kubectl get endpoints $SERVICE -o jsonpath='{.subsets[0].addresses[0].ip}:{.subsets[0].ports[0].port}' 2>/dev/null)

if [ -n "$ENDPOINTS" ]; then
    echo -e "${GREEN}✅ Endpoints: $ENDPOINTS${NC}"
    
    if echo "$ENDPOINTS" | grep -q ":5000"; then
        echo -e "${GREEN}✅ Endpoint port is 5000 (correct)${NC}"
    else
        echo -e "${RED}❌ Endpoint port is not 5000${NC}"
    fi
else
    echo -e "${RED}❌ No endpoints found (service not routing to pods)${NC}"
fi

# 6. Test Application Access
echo -e "\n${YELLOW}6️⃣ Testing Application Access...${NC}"
NODE_IP=$(kubectl get nodes -o jsonpath='{.items[0].status.addresses[?(@.type=="InternalIP")].address}')
echo "   Node IP: $NODE_IP"
echo "   Testing: http://$NODE_IP:32345"

HTTP_CODE=$(curl -s -o /dev/null -w "%{http_code}" http://$NODE_IP:32345 2>/dev/null || echo "000")

if [ "$HTTP_CODE" == "200" ]; then
    echo -e "${GREEN}✅ Application accessible! HTTP $HTTP_CODE${NC}"
    echo -e "${GREEN}✅ URL: http://$NODE_IP:32345${NC}"
    
    # Show response
    echo -e "\n${YELLOW}   Response:${NC}"
    curl -s http://$NODE_IP:32345 | head -3
elif [ "$HTTP_CODE" == "000" ]; then
    echo -e "${RED}❌ Connection failed (cannot reach application)${NC}"
else
    echo -e "${YELLOW}⚠️  Application responding with HTTP $HTTP_CODE${NC}"
fi

# 7. Summary
echo -e "\n${YELLOW}======================================================"
echo -e "📊 Summary:${NC}"
kubectl get deployment,pods,svc,endpoints -l $APP_LABEL 2>/dev/null || kubectl get deployment $DEPLOYMENT && kubectl get svc $SERVICE

if [ "$READY" == "$DESIRED" ] && [ "$POD_READY" == "true" ] && [ "$HTTP_CODE" == "200" ]; then
    echo -e "\n${GREEN}✅ All checks passed! Application is working correctly.${NC}"
else
    echo -e "\n${RED}❌ Issues detected. Review the output above for details.${NC}"
fi
```

**Save and run:**
```bash
chmod +x verify-flask-app.sh
./verify-flask-app.sh
```

---

## 🧹 Cleanup

```bash
# Delete deployment
kubectl delete deployment python-deployment-nautilus

# Delete service
kubectl delete svc python-service-nautilus

# Verify deletion
kubectl get deployment,svc,pods -l app=python-app
# No resources found

# If you created files
rm -f python-service-fixed.yaml verify-flask-app.sh
```

---

## 📖 Key Takeaways

### Concepts Mastered Today:

1. ✅ **The Two Critical Fixes**
   - Fixed deployment image name to correct Flask image
   - Fixed service targetPort from 8080 to 5000 (Flask default)
   - Both were essential for application to work

2. ✅ **Systematic Troubleshooting**
   - Get overview → Check deployment → Check pods → Check service
   - Always read events and logs
   - Identify misconfigurations systematically

3. ✅ **Port Configuration**
   - containerPort: What container exposes (5000 for Flask)
   - targetPort: What service connects to (must match containerPort)
   - port: Service's internal port (80 or any port)
   - nodePort: External access port (32345 in this task)

4. ✅ **Flask Applications**
   - Default port: 5000
   - Must bind to 0.0.0.0 in containers
   - Lightweight Python web framework

5. ✅ **Common Misconfigurations**
   - Wrong image name causing ImagePullBackOff
   - Wrong targetPort in service causing connection issues
   - Service targetPort must match container port (5000)

6. ✅ **Service-Pod Communication**
   - Labels connect service to pods
   - Endpoints show actual routing
   - Both labels and ports must match

7. ✅ **Debugging Tools**
   - kubectl get (overview)
   - kubectl describe (details + events)
   - kubectl logs (application logs)
   - curl (connectivity testing)

---

## 🎯 Real-World Lessons

**Most Important Lesson:**
In this task, only two simple changes were needed:
```
1. Fix deployment image name → poroko/flask-demo-app
2. Fix service targetPort → 5000 (Flask default)
```

**The Root Causes:**
1. **Wrong Image:** Deployment had wrong or missing image causing pods to fail
2. **Port Mismatch:** Service targetPort (8080) didn't match Flask's default port (5000)

**Key Learning:**
Always verify:
- Image name is correct (check for typos or wrong repository)
- Service targetPort matches the port your application actually listens on
- For Flask apps, default port is always 5000

**Troubleshooting Workflow:**
```
1. Check pod status (Running? ImagePullBackOff?)
2. Check pod logs (What port is app listening on?)
3. Check service targetPort (Does it match app port?)
4. Fix image if wrong
5. Fix targetPort if mismatched
6. Test with curl
```

---

## 📚 Additional Resources

**Official Documentation:**
- [Debugging Deployments](https://kubernetes.io/docs/tasks/debug/debug-application/debug-running-pod/)
- [Services](https://kubernetes.io/docs/concepts/services-networking/service/)
- [Troubleshooting Applications](https://kubernetes.io/docs/tasks/debug/debug-application/)

**Flask Documentation:**
- [Flask Quickstart](https://flask.palletsprojects.com/en/2.0.x/quickstart/)
- [Flask in Docker](https://flask.palletsprojects.com/en/2.0.x/deploying/docker/)

**Next Steps:**
- **Day 65:** Rolling updates and rollbacks
- **Day 66:** StatefulSets for stateful applications
- **Day 67:** Jobs and CronJobs
- **Day 68:** DaemonSets for node-level services

---

## ✅ Task Completion Checklist

- [ ] Checked deployment status (was image correct?)
- [ ] Checked pod status (Running? ImagePullBackOff?)
- [ ] Reviewed pod logs to see Flask port
- [ ] Checked service configuration
- [ ] Identified wrong image in deployment
- [ ] Fixed deployment image to `poroko/flask-demo-app`
- [ ] Identified wrong targetPort in service
- [ ] Fixed service targetPort to 5000 (Flask default)
- [ ] Verified nodePort is 32345
- [ ] Checked endpoints are routing to correct port
- [ ] Pods showing 1/1 READY
- [ ] Deployment showing 1/1 AVAILABLE
- [ ] Application accessible on nodePort 32345
- [ ] Verified Flask app responds correctly with curl
- [ ] Documented the two changes made (image + targetPort)

---

**🎉 Congratulations!** You've successfully troubleshooted and fixed a misconfigured Python Flask application deployment! The solution was simple: fix the image name in the deployment and change the service targetPort to 5000.

**Day 64 Status:** ✅ Complete

**Next:** Day 65 - Rolling Updates and Rollback Strategies 🚀
