# Day 53: Kubernetes Troubleshooting - Debugging Nginx and PHP-FPM Pod

## Objective
Investigate and resolve issues with a non-functional Nginx and PHP-FPM setup running in a Kubernetes pod. The pod name is `nginx-phpfpm` and uses a ConfigMap named `nginx-config`. After fixing the issue, deploy the PHP application file to make the website accessible.

## Task Requirements
- **Environment:** Kubernetes cluster accessible from jump_host
- **Pod Name:** nginx-phpfpm
- **ConfigMap Name:** nginx-config
- **Issue:** Nginx and PHP-FPM setup halted this morning
- **Tasks:**
  1. Identify and fix the problem with the pod
  2. Copy `/home/thor/index.php` from jump_host to nginx-container
  3. Copy to nginx document root (typically `/usr/share/nginx/html` or `/var/www/html`)
  4. Verify website accessibility via Website button
- **Tool:** kubectl (pre-configured on jump_host)

## Understanding Nginx + PHP-FPM on Kubernetes

### What is Nginx + PHP-FPM?

**Nginx:**
- Web server for serving static content
- Reverse proxy for dynamic content
- Handles HTTP requests from clients

**PHP-FPM (FastCGI Process Manager):**
- PHP processor for dynamic content
- Runs PHP scripts
- Communicates with Nginx via FastCGI protocol

**Architecture:**
```
Client Request → Nginx Container
                    ├── Static files (.html, .css, .js) → Serve directly
                    └── PHP files (.php) → Forward to PHP-FPM Container
                                              ├── Process PHP
                                              └── Return result to Nginx
```

### Common Nginx + PHP-FPM Setup in Kubernetes

**Two Approaches:**

**1. Multi-Container Pod (Sidecar Pattern):**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-phpfpm
spec:
  containers:
  - name: nginx-container
    image: nginx:latest
    volumeMounts:
    - name: shared-files
      mountPath: /usr/share/nginx/html
  - name: php-fpm-container
    image: php:fpm
    volumeMounts:
    - name: shared-files
      mountPath: /var/www/html
  volumes:
  - name: shared-files
    emptyDir: {}
```

**2. Single Container (Nginx + PHP-FPM combined):**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-phpfpm
spec:
  containers:
  - name: nginx-phpfpm-container
    image: richarvey/nginx-php-fpm:latest
```

### ConfigMap for Nginx Configuration

ConfigMaps store Nginx configuration files:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-config
data:
  nginx.conf: |
    server {
      listen 80;
      root /usr/share/nginx/html;
      location ~ \.php$ {
        fastcgi_pass 127.0.0.1:9000;  # PHP-FPM address
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
      }
    }
```

## Common Issues with Nginx + PHP-FPM Pods

### Issue 1: Invalid apiVersion in ConfigMap
- **Symptom:** ConfigMap validation errors, pod fails to start
- **Cause:** Wrong apiVersion specified (e.g., v6 instead of v1)
- **Example:** `apiVersion: v6` should be `apiVersion: v1`

### Issue 2: ConfigMap Nginx Syntax Errors
- **Symptom:** Pod crashes or CrashLoopBackOff
- **Cause:** Invalid nginx.conf syntax
- **Example:** Missing semicolon, wrong directive

### Issue 2: Wrong FastCGI Pass Address
- **Symptom:** 502 Bad Gateway or 504 Gateway Timeout
- **Cause:** Nginx can't reach PHP-FPM
- **Example:** `fastcgi_pass php-fpm:9000;` but should be `localhost:9000`

### Issue 3: Incorrect Document Root
- **Symptom:** 404 Not Found errors
- **Cause:** Nginx and PHP-FPM using different paths
- **Example:** Nginx uses `/usr/share/nginx/html`, PHP-FPM uses `/var/www/html`

### Issue 4: Missing Volume Mounts
- **Symptom:** Files not shared between containers
- **Cause:** No shared volume between Nginx and PHP-FPM
- **Fix:** Add shared emptyDir or PersistentVolume

### Issue 5: Port Conflicts
- **Symptom:** Pod won't start, port binding errors
- **Cause:** Multiple containers trying to use same port
- **Fix:** Ensure proper port configuration

### Issue 6: Missing FastCGI Parameters
- **Symptom:** PHP scripts download instead of execute
- **Cause:** Missing `fastcgi_params` or `SCRIPT_FILENAME`
- **Fix:** Include proper FastCGI configuration

### Issue 7: File Permissions
- **Symptom:** 403 Forbidden errors
- **Cause:** Wrong ownership or permissions on PHP files
- **Fix:** Adjust file permissions in containers

## Kubernetes Troubleshooting Tools and Techniques

### Essential Kubectl Commands for Debugging

**Check Pod Status:**
```bash
kubectl get pods                           # List all pods
kubectl get pod <pod-name>                 # Specific pod status
kubectl describe pod <pod-name>            # Detailed pod info + events
```

**View Logs:**
```bash
kubectl logs <pod-name>                    # Single container pod
kubectl logs <pod-name> -c <container>     # Multi-container pod
kubectl logs <pod-name> --previous         # Previous container logs (if crashed)
kubectl logs <pod-name> -f                 # Follow logs in real-time
kubectl logs <pod-name> --tail=50          # Last 50 lines
```

**Inspect ConfigMaps:**
```bash
kubectl get configmap                      # List ConfigMaps
kubectl describe configmap <name>          # ConfigMap details
kubectl get configmap <name> -o yaml       # View ConfigMap YAML
```

**Execute Commands in Pod:**
```bash
kubectl exec -it <pod-name> -- /bin/bash   # Interactive shell
kubectl exec <pod-name> -- <command>       # Run single command
kubectl exec <pod-name> -c <container> -- <command>  # Multi-container
```

**Copy Files:**
```bash
kubectl cp <local-file> <pod>:<remote-path>           # Copy to pod
kubectl cp <pod>:<remote-path> <local-file>           # Copy from pod
kubectl cp <file> <pod>:<path> -c <container>         # Multi-container
```

**Edit Resources:**
```bash
kubectl edit pod <pod-name>                # Edit pod (limited changes)
kubectl edit configmap <name>              # Edit ConfigMap
kubectl apply -f <file>                    # Apply updated YAML
```

## Step-by-Step Troubleshooting and Resolution

### Phase 1: Initial Investigation

#### Step 1: Access the Kubernetes Cluster
```bash
# Connect to jump_host (if not already connected)
ssh username@jump_host

# Verify kubectl is configured
kubectl cluster-info

# Expected Output:
# Kubernetes control plane is running at https://x.x.x.x:6443
```

#### Step 2: Check Pod Status
```bash
# List all pods
kubectl get pods

# Expected Output (example of problem):
# NAME            READY   STATUS             RESTARTS   AGE
# nginx-phpfpm    1/2     CrashLoopBackOff   5          10m
# OR
# nginx-phpfpm    2/2     Running            0          10m  (but not working)
```

**Interpreting Status:**
- **CrashLoopBackOff:** Container repeatedly crashing
- **Running but not working:** Configuration issue
- **ImagePullBackOff:** Image not found
- **Pending:** Resource constraints or scheduling issues
- **Error:** Container exited with error

#### Step 3: Get Detailed Pod Information
```bash
# Describe pod for detailed info and events
kubectl describe pod nginx-phpfpm

# Look for:
# 1. Events section (recent errors/warnings)
# 2. Container statuses
# 3. Volume mounts
# 4. ConfigMap references
```

**Example Events to Look For:**
```
Events:
  Type     Reason     Age                From               Message
  ----     ------     ----               ----               -------
  Warning  Failed     2m (x5 over 10m)   kubelet            Error: failed to start container "nginx-container": Error response from daemon: OCI runtime create failed
  Warning  BackOff    1m (x10 over 9m)   kubelet            Back-off restarting failed container
```

#### Step 4: Check ConfigMap
```bash
# View ConfigMap content
kubectl get configmap nginx-config -o yaml

# Expected Output: Full ConfigMap with nginx configuration
```

**Look for Common Issues:**
- Missing semicolons in nginx.conf
- Wrong fastcgi_pass address
- Incorrect root directive
- Syntax errors in configuration

#### Step 5: Check Pod Logs
```bash
# If single container pod:
kubectl logs nginx-phpfpm

# If multi-container pod, check each container:
kubectl logs nginx-phpfpm -c nginx-container
kubectl logs nginx-phpfpm -c php-fpm-container

# Check previous logs if container crashed:
kubectl logs nginx-phpfpm --previous
```

**Common Error Messages:**
```
# Nginx errors:
nginx: [emerg] unexpected "}" in /etc/nginx/nginx.conf:25
nginx: [emerg] invalid number of arguments in "fastcgi_pass" directive
nginx: [error] connect() failed (111: Connection refused) while connecting to upstream

# PHP-FPM errors:
ERROR: failed to open error_log (/var/log/php-fpm/error.log): No such file or directory
WARNING: [pool www] child 10 exited with code 1 after 0.123456 seconds from start
```

### Phase 2: Identify the Issue

#### Step 6: Analyze ConfigMap for Errors
```bash
# Extract and review nginx configuration
kubectl get configmap nginx-config -o jsonpath='{.data}'

# Common issues to check:
```

**Issue Checklist:**

1. **Syntax Errors:**
```nginx
# WRONG (missing semicolon):
server {
    listen 80
    root /usr/share/nginx/html;
}

# CORRECT:
server {
    listen 80;
    root /usr/share/nginx/html;
}
```

2. **Wrong FastCGI Pass:**
```nginx
# WRONG (if PHP-FPM in same pod):
fastcgi_pass php-fpm:9000;

# CORRECT (localhost for same-pod containers):
fastcgi_pass 127.0.0.1:9000;
# OR
fastcgi_pass localhost:9000;
```

3. **Missing FastCGI Parameters:**
```nginx
# WRONG (incomplete):
location ~ \.php$ {
    fastcgi_pass 127.0.0.1:9000;
}

# CORRECT (complete):
location ~ \.php$ {
    fastcgi_pass 127.0.0.1:9000;
    fastcgi_index index.php;
    include fastcgi_params;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
}
```

4. **Wrong Root Path:**
```nginx
# Check consistency between nginx and PHP-FPM
# Both should use same path or shared volume
root /usr/share/nginx/html;  # Nginx
# PHP-FPM should access same path
```

#### Step 7: Check Container Status in Pod
```bash
# Get detailed container status
kubectl get pod nginx-phpfpm -o jsonpath='{.status.containerStatuses[*].name}'
# Output: nginx-container php-fpm-container (or similar)

# Check each container state
kubectl get pod nginx-phpfpm -o jsonpath='{.status.containerStatuses[*].state}'
```

#### Step 8: Test Nginx Configuration Syntax (if accessible)
```bash
# If pod is running, test nginx config
kubectl exec nginx-phpfpm -c nginx-container -- nginx -t

# Expected Output:
# nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
# nginx: configuration file /etc/nginx/nginx.conf test is successful

# OR (if error):
# nginx: [emerg] unexpected "}" in /etc/nginx/nginx.conf:25
# nginx: configuration file /etc/nginx/nginx.conf test failed
```

### Phase 3: Fix the Issue

#### Step 9: Edit ConfigMap to Fix Configuration
```bash
# Method 1: Interactive edit (recommended)
kubectl edit configmap nginx-config

# This opens editor (vim/nano) with ConfigMap YAML
# Make necessary corrections
# Save and exit
```

**Example Corrections:**

**Before (with errors):**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-config
data:
  nginx.conf: |
    server {
        listen 80
        root /usr/share/nginx/html;
        index index.php index.html;
        
        location ~ \.php$ {
            fastcgi_pass php-fpm:9000;
            fastcgi_index index.php;
            include fastcgi_params;
        }
    }
```

**After (fixed):**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-config
data:
  nginx.conf: |
    server {
        listen 80;                              # Added semicolon
        root /usr/share/nginx/html;
        index index.php index.html;
        
        location ~ \.php$ {
            fastcgi_pass 127.0.0.1:9000;        # Changed to localhost
            fastcgi_index index.php;
            include fastcgi_params;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;  # Added
        }
    }
```

**Common Fixes:**

1. **Fix Missing Semicolons:**
```nginx
# Find lines missing semicolons and add them
listen 80;           # Not: listen 80
root /var/www/html;  # Not: root /var/www/html
```

2. **Fix FastCGI Pass Address:**
```nginx
# For multi-container pod (same pod):
fastcgi_pass 127.0.0.1:9000;  # Use localhost

# For separate service:
fastcgi_pass php-fpm-service:9000;  # Use service name
```

3. **Add Missing Parameters:**
```nginx
location ~ \.php$ {
    fastcgi_pass 127.0.0.1:9000;
    fastcgi_index index.php;
    include fastcgi_params;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;  # Critical!
}
```

#### Step 10: Restart Pod to Apply ConfigMap Changes
```bash
# Method 1: Delete pod (Deployment will recreate it)
kubectl delete pod nginx-phpfpm

# Wait for pod to be recreated
kubectl get pods -w

# Method 2: If managed by Deployment
kubectl rollout restart deployment <deployment-name>

# Method 3: Force pod restart by scaling (if Deployment)
kubectl scale deployment <deployment-name> --replicas=0
kubectl scale deployment <deployment-name> --replicas=1
```

**Note:** ConfigMap changes don't automatically restart pods. Pod must be restarted to pick up new configuration.

#### Step 11: Verify Pod is Running
```bash
# Check pod status after restart
kubectl get pod nginx-phpfpm

# Expected Output:
# NAME            READY   STATUS    RESTARTS   AGE
# nginx-phpfpm    2/2     Running   0          1m

# If still failing, check logs again
kubectl logs nginx-phpfpm -c nginx-container
kubectl logs nginx-phpfpm -c php-fpm-container
```

#### Step 12: Verify Nginx Configuration Inside Pod
```bash
# Test nginx configuration
kubectl exec nginx-phpfpm -c nginx-container -- nginx -t

# Expected Output:
# nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
# nginx: configuration file /etc/nginx/nginx.conf test is successful

# Check nginx is running
kubectl exec nginx-phpfpm -c nginx-container -- ps aux | grep nginx

# Expected Output:
# root         1  0.0  0.0  nginx: master process
# nginx       10  0.0  0.0  nginx: worker process
```

### Phase 4: Deploy PHP Application

#### Step 13: Identify Nginx Document Root
```bash
# Check nginx configuration for document root
kubectl exec nginx-phpfpm -c nginx-container -- cat /etc/nginx/nginx.conf | grep root

# Common paths:
# /usr/share/nginx/html  (default nginx)
# /var/www/html          (common alternative)
# /usr/local/nginx/html  (custom builds)

# Or check from ConfigMap
kubectl get configmap nginx-config -o yaml | grep root
```

**Typical Output:**
```nginx
root /usr/share/nginx/html;
```

#### Step 14: Verify Source File Exists on Jump Host
```bash
# Check if index.php exists
ls -lh /home/thor/index.php

# Expected Output:
# -rw-r--r-- 1 thor thor 1.2K Dec 30 10:00 /home/thor/index.php

# View file contents (optional)
cat /home/thor/index.php

# Typical PHP file:
# <?php
# phpinfo();
# ?>
```

#### Step 15: Copy index.php to Nginx Container
```bash
# Copy file from jump_host to pod
kubectl cp /home/thor/index.php nginx-phpfpm:/usr/share/nginx/html/index.php -c nginx-container

# If multi-container pod and shared volume, copy to nginx container
# The volume should be shared with php-fpm container

# Expected Output:
# (No output if successful)
```

**Command Breakdown:**
- `kubectl cp`: Copy command
- `/home/thor/index.php`: Source file on jump_host
- `nginx-phpfpm`: Pod name
- `:/usr/share/nginx/html/index.php`: Destination path in pod
- `-c nginx-container`: Target container (for multi-container pods)

**Alternative if shared volume:**
```bash
# If both containers share volume at /usr/share/nginx/html
# Copy to nginx container (it's accessible to both)
kubectl cp /home/thor/index.php nginx-phpfpm:/usr/share/nginx/html/index.php -c nginx-container

# OR copy to php-fpm container
kubectl cp /home/thor/index.php nginx-phpfpm:/var/www/html/index.php -c php-fpm-container
```

#### Step 16: Verify File Copied Successfully
```bash
# Check file exists in nginx container
kubectl exec nginx-phpfpm -c nginx-container -- ls -lh /usr/share/nginx/html/index.php

# Expected Output:
# -rw-r--r-- 1 root root 1.2K Dec 30 10:05 /usr/share/nginx/html/index.php

# View file contents (verify integrity)
kubectl exec nginx-phpfpm -c nginx-container -- cat /usr/share/nginx/html/index.php

# Should match source file
```

#### Step 17: Fix File Permissions (if needed)
```bash
# If file permissions incorrect, fix them
kubectl exec nginx-phpfpm -c nginx-container -- chmod 644 /usr/share/nginx/html/index.php

# Ensure nginx user can read
kubectl exec nginx-phpfpm -c nginx-container -- chown nginx:nginx /usr/share/nginx/html/index.php

# Or for www-data user (common in PHP containers)
kubectl exec nginx-phpfpm -c php-fpm-container -- chown www-data:www-data /var/www/html/index.php
```

### Phase 5: Verification and Testing

#### Step 18: Check Pod Service
```bash
# Find service exposing the pod
kubectl get services

# Expected Output:
# NAME                TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
# nginx-phpfpm-svc    NodePort    10.96.100.50    <none>        80:30080/TCP   1h
# OR
# nginx-phpfpm-svc    LoadBalancer 10.96.100.50   <pending>     80:30080/TCP   1h

# Describe service
kubectl describe service nginx-phpfpm-svc
```

#### Step 19: Get Service Access URL
```bash
# For NodePort service
kubectl get service nginx-phpfpm-svc -o jsonpath='{.spec.ports[0].nodePort}'
# Output: 30080 (example)

# Access via NodePort
# http://<node-ip>:30080

# For LoadBalancer (if cloud provider)
kubectl get service nginx-phpfpm-svc -o jsonpath='{.status.loadBalancer.ingress[0].ip}'

# For ClusterIP (port-forward for testing)
kubectl port-forward pod/nginx-phpfpm 8080:80
# Access via http://localhost:8080
```

#### Step 20: Test Website Access
```bash
# Method 1: From within cluster (if curl available)
kubectl run test-pod --image=curlimages/curl --rm -it --restart=Never -- curl http://nginx-phpfpm-svc

# Method 2: Port-forward and test locally
kubectl port-forward pod/nginx-phpfpm 8080:80 &
curl http://localhost:8080/index.php

# Expected Output:
# HTML output with PHP info or application content

# Method 3: From jump_host (if service exposed)
curl http://<service-ip>:80/index.php
```

#### Step 21: Verify PHP Processing
```bash
# Test if PHP is being processed (not downloaded)
kubectl port-forward pod/nginx-phpfpm 8080:80 &
curl -I http://localhost:8080/index.php

# Expected Headers:
# HTTP/1.1 200 OK
# Content-Type: text/html; charset=UTF-8    ← PHP processed
# X-Powered-By: PHP/8.x.x                   ← PHP is working

# If you see:
# Content-Type: application/octet-stream    ← PHP NOT processed (downloading)
# Then nginx is not passing to PHP-FPM correctly
```

#### Step 22: Check Application Logs
```bash
# Check nginx access logs
kubectl logs nginx-phpfpm -c nginx-container --tail=20

# Expected Output:
# 10.244.0.1 - - [30/Dec/2025:10:15:30 +0000] "GET /index.php HTTP/1.1" 200 1234

# Check PHP-FPM logs
kubectl logs nginx-phpfpm -c php-fpm-container --tail=20

# Expected Output:
# [30-Dec-2025 10:15:30] NOTICE: ready to handle connections
```

#### Step 23: Test via Website Button (KodeKloud Interface)
```bash
# In KodeKloud environment, click "Website" button on top bar
# This typically accesses the service via pre-configured URL

# If website loads successfully:
# ✅ Task complete!

# If website doesn't load:
# - Check service is exposed correctly
# - Verify pod is running
# - Check firewall/network policies
```

#### Step 24: Verify Complete Functionality
```bash
# Run comprehensive checks
kubectl get pod nginx-phpfpm
# STATUS: Running, READY: 2/2

kubectl logs nginx-phpfpm -c nginx-container --tail=5
# No errors, showing successful requests

kubectl logs nginx-phpfpm -c php-fpm-container --tail=5
# PHP-FPM handling requests

curl http://<service-endpoint>/index.php
# Returns PHP-processed content (HTML)

# Website button shows working application
# ✅ All checks passed
```

## Complete Command Summary

### Investigation Commands
```bash
# Check pod status
kubectl get pods
kubectl get pod nginx-phpfpm
kubectl describe pod nginx-phpfpm

# Check logs
kubectl logs nginx-phpfpm
kubectl logs nginx-phpfpm -c nginx-container
kubectl logs nginx-phpfpm -c php-fpm-container
kubectl logs nginx-phpfpm --previous

# Check ConfigMap
kubectl get configmap nginx-config
kubectl get configmap nginx-config -o yaml
kubectl describe configmap nginx-config

# Check services
kubectl get services
kubectl describe service <service-name>
```

### Troubleshooting Commands
```bash
# Test nginx configuration
kubectl exec nginx-phpfpm -c nginx-container -- nginx -t

# Check nginx processes
kubectl exec nginx-phpfpm -c nginx-container -- ps aux | grep nginx

# Check file system
kubectl exec nginx-phpfpm -c nginx-container -- ls -lh /usr/share/nginx/html/
kubectl exec nginx-phpfpm -c nginx-container -- cat /etc/nginx/nginx.conf

# Interactive shell for debugging
kubectl exec -it nginx-phpfpm -c nginx-container -- /bin/bash
kubectl exec -it nginx-phpfpm -c php-fpm-container -- /bin/sh
```

### Fix Commands
```bash
# Edit ConfigMap
kubectl edit configmap nginx-config

# Restart pod
kubectl delete pod nginx-phpfpm

# Reload nginx configuration (if supported)
kubectl exec nginx-phpfpm -c nginx-container -- nginx -s reload

# Copy file to pod
kubectl cp /home/thor/index.php nginx-phpfpm:/usr/share/nginx/html/index.php -c nginx-container

# Fix permissions
kubectl exec nginx-phpfpm -c nginx-container -- chmod 644 /usr/share/nginx/html/index.php
kubectl exec nginx-phpfpm -c nginx-container -- chown nginx:nginx /usr/share/nginx/html/index.php
```

### Verification Commands
```bash
# Verify file copied
kubectl exec nginx-phpfpm -c nginx-container -- ls -lh /usr/share/nginx/html/index.php
kubectl exec nginx-phpfpm -c nginx-container -- cat /usr/share/nginx/html/index.php

# Port forward for testing
kubectl port-forward pod/nginx-phpfpm 8080:80

# Test endpoints
curl http://localhost:8080/index.php
curl -I http://localhost:8080/index.php

# Check pod status
kubectl get pod nginx-phpfpm
kubectl describe pod nginx-phpfpm
```

## Troubleshooting Common Issues

### Issue 1: ConfigMap Changes Not Applied

**Symptoms:**
```bash
kubectl edit configmap nginx-config
# Made changes, but pod still has old config

kubectl exec nginx-phpfpm -c nginx-container -- cat /etc/nginx/nginx.conf
# Still showing old configuration
```

**Cause:**
- ConfigMap changes don't automatically update mounted volumes
- Pod needs restart to pick up new ConfigMap

**Solution:**
```bash
# Delete pod to restart (will be recreated if managed by Deployment)
kubectl delete pod nginx-phpfpm

# Wait for new pod
kubectl get pods -w

# Verify new config loaded
kubectl exec nginx-phpfpm -c nginx-container -- cat /etc/nginx/nginx.conf
# Should show updated configuration
```

### Issue 2: File Copy Fails - Container Not Found

**Symptoms:**
```bash
kubectl cp /home/thor/index.php nginx-phpfpm:/usr/share/nginx/html/index.php
# Error: error: unable to upgrade connection: container not found ("nginx-container")
```

**Cause:**
- Multi-container pod requires `-c` flag to specify container
- Container name might be different than expected

**Diagnosis:**
```bash
# List containers in pod
kubectl get pod nginx-phpfpm -o jsonpath='{.spec.containers[*].name}'

# Output might be:
# nginx php-fpm
# (Not "nginx-container")
```

**Solution:**
```bash
# Use correct container name
kubectl cp /home/thor/index.php nginx-phpfpm:/usr/share/nginx/html/index.php -c nginx

# Or if single container, no -c flag needed
kubectl cp /home/thor/index.php nginx-phpfpm:/usr/share/nginx/html/index.php
```

### Issue 3: 502 Bad Gateway Error

**Symptoms:**
```bash
curl http://localhost:8080/index.php
# HTTP/1.1 502 Bad Gateway
```

**Cause:**
- Nginx can't connect to PHP-FPM
- Wrong fastcgi_pass address
- PHP-FPM not running

**Diagnosis:**
```bash
# Check PHP-FPM is running
kubectl exec nginx-phpfpm -c php-fpm-container -- ps aux | grep php-fpm

# Check PHP-FPM logs
kubectl logs nginx-phpfpm -c php-fpm-container

# Check nginx error logs
kubectl logs nginx-phpfpm -c nginx-container | grep error
```

**Solution:**
```bash
# Fix fastcgi_pass in ConfigMap
kubectl edit configmap nginx-config

# Change to correct address:
# For same-pod containers:
fastcgi_pass 127.0.0.1:9000;

# Restart pod
kubectl delete pod nginx-phpfpm

# Verify connection
curl http://localhost:8080/index.php
# Should return 200 OK
```

### Issue 4: PHP File Downloads Instead of Executing

**Symptoms:**
```bash
curl http://localhost:8080/index.php
# Downloads index.php file instead of executing it
```

**Cause:**
- Missing or incomplete PHP location block in nginx config
- Missing SCRIPT_FILENAME parameter
- PHP-FPM not properly configured

**Diagnosis:**
```bash
# Check nginx configuration
kubectl get configmap nginx-config -o yaml

# Look for location ~ \.php$ block
# Check if SCRIPT_FILENAME is set
```

**Solution:**
```bash
# Edit ConfigMap
kubectl edit configmap nginx-config

# Ensure complete PHP block:
location ~ \.php$ {
    fastcgi_pass 127.0.0.1:9000;
    fastcgi_index index.php;
    include fastcgi_params;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;  # Critical!
}

# Restart pod
kubectl delete pod nginx-phpfpm

# Test
curl -I http://localhost:8080/index.php
# Content-Type: text/html; charset=UTF-8  (correct - PHP processed)
```

### Issue 5: 404 Not Found Error

**Symptoms:**
```bash
curl http://localhost:8080/index.php
# HTTP/1.1 404 Not Found
```

**Cause:**
- File not in correct location
- Wrong document root in nginx config
- File permissions preventing access

**Diagnosis:**
```bash
# Check nginx document root
kubectl exec nginx-phpfpm -c nginx-container -- cat /etc/nginx/nginx.conf | grep root

# Check if file exists
kubectl exec nginx-phpfpm -c nginx-container -- ls -lh /usr/share/nginx/html/

# Check file permissions
kubectl exec nginx-phpfpm -c nginx-container -- ls -lh /usr/share/nginx/html/index.php
```

**Solution:**
```bash
# If file in wrong location, copy to correct location
kubectl cp /home/thor/index.php nginx-phpfpm:/usr/share/nginx/html/index.php -c nginx-container

# Fix permissions if needed
kubectl exec nginx-phpfpm -c nginx-container -- chmod 644 /usr/share/nginx/html/index.php

# Verify
curl http://localhost:8080/index.php
# Should return 200 OK
```

### Issue 6: Permission Denied Errors

**Symptoms:**
```bash
kubectl logs nginx-phpfpm -c nginx-container
# [error] 10#10: *1 open() "/usr/share/nginx/html/index.php" failed (13: Permission denied)
```

**Cause:**
- Wrong file ownership or permissions
- SELinux context issues
- Pod security policies

**Diagnosis:**
```bash
# Check file permissions
kubectl exec nginx-phpfpm -c nginx-container -- ls -lh /usr/share/nginx/html/index.php

# Check nginx user
kubectl exec nginx-phpfpm -c nginx-container -- ps aux | grep nginx
```

**Solution:**
```bash
# Fix ownership (use nginx user or www-data)
kubectl exec nginx-phpfpm -c nginx-container -- chown nginx:nginx /usr/share/nginx/html/index.php

# Fix permissions
kubectl exec nginx-phpfpm -c nginx-container -- chmod 644 /usr/share/nginx/html/index.php

# Verify
kubectl exec nginx-phpfpm -c nginx-container -- ls -lh /usr/share/nginx/html/index.php
# -rw-r--r-- 1 nginx nginx 1.2K Dec 30 10:00 index.php
```

### Issue 7: Pod Stuck in CrashLoopBackOff

**Symptoms:**
```bash
kubectl get pod nginx-phpfpm
# NAME            READY   STATUS             RESTARTS   AGE
# nginx-phpfpm    0/2     CrashLoopBackOff   10         15m
```

**Cause:**
- Container repeatedly failing to start
- Configuration error causing crash
- Missing dependencies

**Diagnosis:**
```bash
# Check pod events
kubectl describe pod nginx-phpfpm | grep -A 20 Events

# Check logs of crashed container
kubectl logs nginx-phpfpm -c nginx-container --previous
kubectl logs nginx-phpfpm -c php-fpm-container --previous

# Look for error messages
```

**Solution:**
```bash
# Fix the underlying issue (usually ConfigMap)
kubectl edit configmap nginx-config

# Common fixes:
# 1. Fix nginx.conf syntax errors
# 2. Correct fastcgi_pass address
# 3. Fix document root paths

# Delete pod to restart with fixed config
kubectl delete pod nginx-phpfpm

# Monitor pod startup
kubectl get pods -w
```

## Best Practices for Nginx + PHP-FPM on Kubernetes

### 1. Use Shared Volumes for Multi-Container Pods
✅ **Good:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-phpfpm
spec:
  containers:
  - name: nginx
    image: nginx:latest
    volumeMounts:
    - name: shared-files
      mountPath: /usr/share/nginx/html
  - name: php-fpm
    image: php:fpm
    volumeMounts:
    - name: shared-files
      mountPath: /usr/share/nginx/html  # Same path for sharing
  volumes:
  - name: shared-files
    emptyDir: {}
```

**Why:**
- Both containers can access PHP files
- Nginx serves static files directly
- PHP-FPM processes PHP files from same location

### 2. Use ConfigMaps for Nginx Configuration
✅ **Good:**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-config
data:
  default.conf: |
    server {
        listen 80;
        root /usr/share/nginx/html;
        index index.php index.html;
        
        location ~ \.php$ {
            fastcgi_pass 127.0.0.1:9000;
            fastcgi_index index.php;
            include fastcgi_params;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        }
    }
---
apiVersion: v1
kind: Pod
metadata:
  name: nginx-phpfpm
spec:
  containers:
  - name: nginx
    image: nginx:latest
    volumeMounts:
    - name: nginx-config-volume
      mountPath: /etc/nginx/conf.d/default.conf
      subPath: default.conf
  volumes:
  - name: nginx-config-volume
    configMap:
      name: nginx-config
```

**Why:**
- Easy to update configuration without rebuilding images
- Version control for configuration
- Separate configuration from application code

### 3. Use Correct FastCGI Pass Address
✅ **Good:**
```nginx
# For same-pod containers (sidecar pattern):
location ~ \.php$ {
    fastcgi_pass 127.0.0.1:9000;  # localhost
}

# For separate PHP-FPM service:
location ~ \.php$ {
    fastcgi_pass php-fpm-service.default.svc.cluster.local:9000;  # Service DNS
}
```

❌ **Bad:**
```nginx
# Wrong for same-pod containers:
fastcgi_pass php-fpm:9000;  # DNS lookup will fail
```

### 4. Include All Required FastCGI Parameters
✅ **Good:**
```nginx
location ~ \.php$ {
    fastcgi_pass 127.0.0.1:9000;
    fastcgi_index index.php;
    include fastcgi_params;                                        # Include standard params
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;  # Critical!
    fastcgi_param PATH_INFO $fastcgi_path_info;                    # For frameworks
}
```

❌ **Bad:**
```nginx
location ~ \.php$ {
    fastcgi_pass 127.0.0.1:9000;
    # Missing parameters - PHP won't execute properly
}
```

### 5. Set Appropriate Resource Limits
✅ **Good:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-phpfpm
spec:
  containers:
  - name: nginx
    image: nginx:latest
    resources:
      requests:
        memory: "64Mi"
        cpu: "100m"
      limits:
        memory: "128Mi"
        cpu: "200m"
  - name: php-fpm
    image: php:fpm
    resources:
      requests:
        memory: "256Mi"
        cpu: "200m"
      limits:
        memory: "512Mi"
        cpu: "500m"
```

**Why:**
- Prevents resource starvation
- Ensures predictable performance
- Enables proper scheduling

### 6. Use Liveness and Readiness Probes
✅ **Good:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-phpfpm
spec:
  containers:
  - name: nginx
    image: nginx:latest
    livenessProbe:
      httpGet:
        path: /health.html
        port: 80
      initialDelaySeconds: 10
      periodSeconds: 5
    readinessProbe:
      httpGet:
        path: /health.html
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 3
```

**Why:**
- Kubernetes knows when pod is healthy
- Automatic restart if unhealthy
- Traffic only routed to ready pods

### 7. Use Persistent Volumes for Application Files (Production)
✅ **Good (Production):**
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: app-files-pvc
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
---
apiVersion: v1
kind: Pod
metadata:
  name: nginx-phpfpm
spec:
  containers:
  - name: nginx
    volumeMounts:
    - name: app-files
      mountPath: /usr/share/nginx/html
  - name: php-fpm
    volumeMounts:
    - name: app-files
      mountPath: /usr/share/nginx/html
  volumes:
  - name: app-files
    persistentVolumeClaim:
      claimName: app-files-pvc
```

**Why:**
- Files persist across pod restarts
- Can be backed up
- Suitable for production workloads

### 8. Log to stdout/stderr (Not Files)
✅ **Good:**
```nginx
# In nginx.conf:
error_log /dev/stderr warn;
access_log /dev/stdout main;
```

```ini
# In php-fpm.conf:
error_log = /proc/self/fd/2
access.log = /proc/self/fd/1
```

**Why:**
- Kubernetes can collect logs via `kubectl logs`
- Works with log aggregation systems
- No disk space issues from log files

### 9. Test Configuration Before Applying
```bash
# Before editing ConfigMap in production:

# 1. Export current ConfigMap
kubectl get configmap nginx-config -o yaml > nginx-config-backup.yaml

# 2. Create test environment
kubectl create namespace test

# 3. Apply to test namespace first
kubectl apply -f nginx-config-new.yaml -n test

# 4. Test thoroughly
kubectl port-forward pod/nginx-phpfpm 8080:80 -n test
curl http://localhost:8080/index.php

# 5. If successful, apply to production
kubectl apply -f nginx-config-new.yaml

# 6. Keep backup for rollback
```

### 10. Monitor and Set Up Alerts
```yaml
# Use monitoring tools to track:
# - Pod restart count
# - Error rate in logs
# - Response times
# - PHP-FPM process count
# - Memory usage

# Example: Prometheus alerts
apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-alerts
data:
  alerts.yaml: |
    groups:
    - name: nginx_phpfpm
      rules:
      - alert: HighErrorRate
        expr: rate(nginx_http_requests_total{status=~"5.."}[5m]) > 0.05
        annotations:
          summary: "High error rate detected in nginx-phpfpm"
```

## Real-World Troubleshooting Scenarios

### Scenario 1: Invalid apiVersion in ConfigMap

**Problem:**
```yaml
apiVersion: v6    # Wrong! Should be v1
kind: ConfigMap
metadata:
  name: nginx-config
data:
  nginx.conf: |
    events {}
    http {
      server {
        listen 8099;
        root /var/www/html;
      }
    }
```

**Symptoms:**
```bash
kubectl get pod nginx-phpfpm
# STATUS: Error or CreateContainerConfigError

kubectl describe pod nginx-phpfpm
# Error: unable to retrieve ConfigMap: ConfigMap "nginx-config" is invalid: apiVersion: Invalid value: "v6": must be "v1"
```

**Solution:**
```bash
kubectl edit configmap nginx-config
# Change: apiVersion: v6
# To:     apiVersion: v1

# Restart pod
kubectl delete pod nginx-phpfpm

# Verify
kubectl get pod nginx-phpfpm
# STATUS: Running
```

**Root Cause:**
- ConfigMaps in Kubernetes use `apiVersion: v1` (part of core API)
- `v6` is not a valid API version
- This is a typo or copy-paste error

**Prevention:**
- Always use `apiVersion: v1` for ConfigMaps
- Use `kubectl create configmap` which sets correct apiVersion automatically
- Validate YAML before applying: `kubectl apply --dry-run=client -f configmap.yaml`

### Scenario 2: Missing Semicolon in nginx.conf

**Problem:**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-config
data:
  nginx.conf: |
    server {
        listen 80          # Missing semicolon!
        root /usr/share/nginx/html;
    }
```

**Symptoms:**
```bash
kubectl get pod nginx-phpfpm
# STATUS: CrashLoopBackOff

kubectl logs nginx-phpfpm -c nginx-container
# nginx: [emerg] invalid number of arguments in "listen" directive in /etc/nginx/nginx.conf:2
```

**Solution:**
```bash
kubectl edit configmap nginx-config
# Add semicolon: listen 80;

kubectl delete pod nginx-phpfpm
kubectl get pods -w
# Pod restarts successfully
```

### Scenario 3: Wrong FastCGI Pass Address

**Problem:**
```yaml
data:
  nginx.conf: |
    location ~ \.php$ {
        fastcgi_pass php-fpm:9000;  # Wrong for same-pod containers!
    }
```

**Symptoms:**
```bash
curl http://localhost:8080/index.php
# 502 Bad Gateway

kubectl logs nginx-phpfpm -c nginx-container
# [error] 10#10: *1 connect() failed (111: Connection refused) while connecting to upstream
# [error] 10#10: *1 no live upstreams while connecting to upstream
```

**Solution:**
```bash
kubectl edit configmap nginx-config
# Change to: fastcgi_pass 127.0.0.1:9000;

kubectl delete pod nginx-phpfpm
curl http://localhost:8080/index.php
# 200 OK - Working!
```

### Scenario 4: Missing SCRIPT_FILENAME Parameter

**Problem:**
```yaml
data:
  nginx.conf: |
    location ~ \.php$ {
        fastcgi_pass 127.0.0.1:9000;
        include fastcgi_params;
        # Missing: fastcgi_param SCRIPT_FILENAME
    }
```

**Symptoms:**
```bash
curl http://localhost:8080/index.php
# File not found.

kubectl logs nginx-phpfpm -c php-fpm-container
# [error] 10#10: *1 FastCGI sent in stderr: "Primary script unknown"
```

**Solution:**
```bash
kubectl edit configmap nginx-config
# Add: fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;

kubectl delete pod nginx-phpfpm
curl http://localhost:8080/index.php
# PHP content displayed correctly
```

### Scenario 5: File in Wrong Directory

**Problem:**
```bash
# File copied to wrong location
kubectl cp /home/thor/index.php nginx-phpfpm:/var/www/html/index.php -c nginx-container

# But nginx configured with:
# root /usr/share/nginx/html;
```

**Symptoms:**
```bash
curl http://localhost:8080/index.php
# 404 Not Found
```

**Solution:**
```bash
# Copy to correct location
kubectl cp /home/thor/index.php nginx-phpfpm:/usr/share/nginx/html/index.php -c nginx-container

# Verify
kubectl exec nginx-phpfpm -c nginx-container -- ls -lh /usr/share/nginx/html/
# -rw-r--r-- 1 root root 1.2K Dec 30 10:00 index.php

curl http://localhost:8080/index.php
# 200 OK - Working!
```

### Scenario 6: Pod Configuration Issue Requiring Redeployment

**Problem:**
```bash
# Pod exists but not functioning correctly
kubectl get pod nginx-phpfpm
# STATUS: Running but website not accessible

# ConfigMap looks correct
kubectl get configmap nginx-config -o yaml
# nginx.conf has correct configuration (listen 8099, fastcgi_pass 127.0.0.1:9000, etc.)
```

**Symptoms:**
- Pod shows Running status
- ConfigMap configuration is valid
- Website still not accessible
- Issue is in pod specification itself

**Real-World Solution Steps:**
```bash
# Step 1: Export pod configuration
kubectl get pod nginx-phpfpm -o yaml > pod.yml

# Step 2: Review and edit pod YAML file
vi pod.yml
# Look for:
# - Volume mount issues
# - Container configuration problems
# - Port mappings
# - Environment variables
# - Init container issues

# Step 3: Delete existing pod
kubectl delete pod nginx-phpfpm
# pod "nginx-phpfpm" deleted

# Step 4: Reapply corrected configuration
kubectl apply -f pod.yml
# pod/nginx-phpfpm created

# Step 5: Verify pod is running correctly
kubectl get pod nginx-phpfpm
# STATUS: Running, READY: 2/2

# Step 6: Copy application file to correct location
kubectl cp /home/thor/index.php nginx-phpfpm:/var/www/html -c nginx-container

# Step 7: Verify file copied successfully
kubectl exec nginx-phpfpm -c nginx-container -- ls -lh /var/www/html/
# -rw-r--r-- 1 root root 1.2K Dec 31 14:30 index.php

# Step 8: Test website accessibility
# Click "Website" button on top bar
# Website should now be accessible on port 8099
```

**Root Cause:**
Pod specification had configuration issues that couldn't be fixed by editing the ConfigMap alone. Common pod-level issues include:
- Incorrect volume mounts (not mounting ConfigMap properly)
- Wrong container ports exposed
- Missing or incorrect init containers
- Environment variable issues
- Resource constraints causing startup problems
- Security context preventing proper operation

**Key ConfigMap Details (Verified Correct):**
```yaml
data:
  nginx.conf: |
    events {}
    http {
      server {
        listen 8099 default_server;              # Custom port 8099
        listen [::]:8099 default_server;         # IPv6 support
        root /var/www/html;                      # Document root
        index index.html index.htm index.php;
        server_name _;
        location / {
          try_files $uri $uri/ =404;
        }
        location ~ \.php$ {
          include fastcgi_params;
          fastcgi_param REQUEST_METHOD $request_method;
          fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;  # Correct!
          fastcgi_pass 127.0.0.1:9000;           # Correct localhost address
        }
      }
    }
```

**Prevention:**
- Use Deployments instead of standalone Pods for easier updates
- Validate pod YAML before applying: `kubectl apply --dry-run=client -f pod.yml`
- Test pod configuration in development environment first
- Keep backup copies of working configurations
- Use `kubectl diff -f pod.yml` to see changes before applying
- Document known-good configurations in version control

**Lesson Learned:**
Not all Kubernetes issues are in the ConfigMap. Sometimes the pod specification itself needs correction, requiring pod deletion and redeployment with corrected YAML. Always check both ConfigMap and pod configuration when troubleshooting.

## Completion Checklist

Run these commands to verify task completion:

```bash
# 1. Pod is running successfully
kubectl get pod nginx-phpfpm
# STATUS: Running, READY: 2/2

# 2. ConfigMap has valid configuration
kubectl get configmap nginx-config -o yaml
# Check for syntax errors, correct fastcgi_pass

# 3. Nginx configuration syntax valid
kubectl exec nginx-phpfpm -c nginx-container -- nginx -t
# nginx: configuration file test is successful

# 4. Both containers running
kubectl get pod nginx-phpfpm -o jsonpath='{.status.containerStatuses[*].state.running}'
# Should show both containers in running state

# 5. index.php file exists in correct location
kubectl exec nginx-phpfpm -c nginx-container -- ls -lh /usr/share/nginx/html/index.php
# File exists with proper permissions

# 6. No errors in nginx logs
kubectl logs nginx-phpfpm -c nginx-container --tail=20
# No [error] or [emerg] messages

# 7. No errors in PHP-FPM logs
kubectl logs nginx-phpfpm -c php-fpm-container --tail=20
# PHP-FPM ready to handle connections

# 8. Website accessible via service
kubectl get service
# Service exists and has endpoints

# 9. PHP processing works
curl -I http://<service-endpoint>/index.php
# Content-Type: text/html; charset=UTF-8
# X-Powered-By: PHP/8.x.x

# 10. Website button works (KodeKloud)
# Click "Website" button on top bar
# Application loads successfully

# 11. Application returns expected content
curl http://<service-endpoint>/index.php
# Returns processed PHP content (HTML)
```

**All checks passing = Task complete! ✅**

## Summary

### What We Accomplished
1. ✅ Investigated nginx-phpfpm pod issues using kubectl commands
2. ✅ Identified ConfigMap configuration errors (syntax, fastcgi_pass, parameters)
3. ✅ Fixed nginx configuration in ConfigMap
4. ✅ Restarted pod to apply configuration changes
5. ✅ Copied index.php from jump_host to pod's document root
6. ✅ Verified website functionality and PHP processing
7. ✅ Confirmed accessibility via Website button

### Key Concepts Learned
- **Multi-Container Pods:** Nginx and PHP-FPM as sidecars with shared volumes
- **ConfigMap Usage:** Store and mount nginx configuration
- **Troubleshooting Workflow:** Check status → logs → configuration → fix → verify
- **FastCGI Configuration:** Critical parameters for PHP processing
- **File Management:** Copy files to pods and set proper permissions
- **Service Access:** Multiple methods to test pod accessibility

### Critical Commands
```bash
# Investigation
kubectl describe pod <pod-name>
kubectl logs <pod-name> -c <container-name>
kubectl get configmap <name> -o yaml

# Fix
kubectl edit configmap <name>
kubectl delete pod <pod-name>
kubectl exec <pod-name> -c <container> -- nginx -t

# Deploy
kubectl cp <local-file> <pod>:<remote-path> -c <container>
kubectl exec <pod> -c <container> -- chmod 644 <file>

# Verify
kubectl get pod <pod-name>
curl http://<service-endpoint>/index.php
```

### Production Best Practices
1. ✅ Use ConfigMaps for configuration management
2. ✅ Share volumes between related containers
3. ✅ Use correct FastCGI pass address (127.0.0.1 for same-pod)
4. ✅ Include all required FastCGI parameters (especially SCRIPT_FILENAME)
5. ✅ Test nginx configuration before applying
6. ✅ Monitor logs for errors and performance issues
7. ✅ Set proper file permissions in containers
8. ✅ Use liveness and readiness probes
9. ✅ Set resource limits for predictable performance
10. ✅ Back up ConfigMaps before making changes

### Troubleshooting Methodology
1. **Check Status:** `kubectl get pods` - Identify problem pods
2. **View Details:** `kubectl describe pod` - Read events and status
3. **Check Logs:** `kubectl logs` - Find error messages
4. **Inspect Config:** `kubectl get configmap -o yaml` - Review configuration
5. **Test Syntax:** `kubectl exec -- nginx -t` - Validate nginx config
6. **Fix Issue:** `kubectl edit` - Correct configuration
7. **Restart Pod:** `kubectl delete pod` - Apply changes
8. **Verify:** Check logs, test endpoints, confirm functionality

**Kubernetes troubleshooting is a systematic process: gather information, identify root cause, apply fix, verify resolution.**

🎉 **Day 53 Complete!** You've mastered Kubernetes troubleshooting for multi-container pods, ConfigMap debugging, and PHP application deployment—essential skills for maintaining production Kubernetes clusters.
