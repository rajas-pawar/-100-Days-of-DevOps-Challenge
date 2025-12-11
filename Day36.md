# Day 36: Docker Container Deployment - Nginx
**100 Days Cloud DevOps Challenge – KodeKloud**

---

## Objective

The Nautilus DevOps team is conducting application deployment tests and requires an nginx container deployment on Application Server 3. Deploy a containerized nginx web server using Docker.

**Requirements:**
1. Work on Application Server 3
2. Create a container named `nginx_3`
3. Use nginx image with `alpine` tag
4. Ensure container is in running state

---

## Understanding Docker Containers

**Docker Container** is a lightweight, standalone, executable package that includes everything needed to run an application: code, runtime, system tools, libraries, and settings.

### Containers vs Virtual Machines

| Aspect | Containers | Virtual Machines |
|--------|-----------|------------------|
| **Size** | Megabytes | Gigabytes |
| **Startup** | Seconds | Minutes |
| **Isolation** | Process-level | Hardware-level |
| **OS** | Shares host kernel | Full OS per VM |
| **Performance** | Near-native | Overhead from hypervisor |
| **Portability** | Highly portable | Less portable |

### Why Use Containers?

- **Lightweight:** Minimal overhead compared to VMs
- **Fast:** Start in seconds
- **Portable:** Run anywhere Docker runs
- **Consistent:** Same environment dev to prod
- **Efficient:** Share host OS kernel
- **Scalable:** Easy horizontal scaling
- **Isolated:** Process and file system separation

---

## Understanding Nginx Alpine

### What is Nginx?

**Nginx** (pronounced "engine-x") is a high-performance web server, reverse proxy, and load balancer.

**Common Uses:**
- Static content serving
- Reverse proxy
- Load balancing
- SSL/TLS termination
- HTTP caching

### What is Alpine Linux?

**Alpine Linux** is a security-oriented, lightweight Linux distribution based on musl libc and busybox.

**Characteristics:**
- Very small size (~5MB base)
- Security-focused
- Simple and resource-efficient
- Ideal for containers

### nginx:alpine Image

| Image Tag | Size | Use Case |
|-----------|------|----------|
| `nginx:latest` | ~140MB | Full-featured |
| `nginx:alpine` | ~23MB | Lightweight production |
| `nginx:stable` | ~140MB | Stable releases |
| `nginx:mainline` | ~140MB | Latest features |

**Why nginx:alpine?**
- ✅ 85% smaller than standard nginx
- ✅ Faster downloads and deployments
- ✅ Less attack surface (security)
- ✅ Lower resource consumption
- ✅ Production-ready

---

## Infrastructure Overview

### Application Servers:
| Server | User | Password | IP |
|--------|------|----------|-----|
| stapp01 | tony | Ir0nM@n | 172.16.238.10 |
| stapp02 | steve | Am3ric@ | 172.16.238.11 |
| **stapp03** | **banner** | **BigGr33n** | **172.16.238.12** |

### Container Details:
| Item | Value |
|------|-------|
| Server | Application Server 3 (stapp03) |
| Container Name | `nginx_3` |
| Image | `nginx:alpine` |
| Required State | Running |
| Port | Default 80 (not exposed in this task) |

---

## Understanding the Task

### What We're Creating:

```
Application Server 3 (stapp03)
│
├── Docker Daemon (running)
│
└── Container: nginx_3
    ├── Image: nginx:alpine
    ├── State: Running
    └── Nginx web server (listening on port 80 inside container)
```

### Docker Container Lifecycle:

```
1. Pull Image: nginx:alpine → Local registry
2. Create Container: nginx_3 from image
3. Start Container: nginx_3 enters running state
4. Verify: Container is running
```

---

## Step-by-Step Implementation

### Step 1: SSH into Application Server 3
```bash
ssh banner@stapp03
```

**Enter password:** `BigGr33n`

**Expected output:**
```
banner@stapp03's password:
Last login: ...
[banner@stapp03 ~]$
```

### Step 2: Verify Docker is Installed
```bash
docker --version
```

**Expected output:**
```
Docker version 24.0.7, build afdd53b
```

**If Docker not installed:**
```bash
# Install Docker (from Day 35)
sudo yum install -y docker-ce docker-ce-cli containerd.io
```

### Step 3: Check Docker Service Status
```bash
sudo systemctl status docker
```

**Expected output:**
```
● docker.service - Docker Application Container Engine
   Loaded: loaded (/usr/lib/systemd/system/docker.service; enabled)
   Active: active (running) since ...
```

**If not running:**
```bash
sudo systemctl start docker
sudo systemctl enable docker
```

### Step 4: Verify Docker is Working
```bash
sudo docker info
```

**Expected output:**
```
Client:
 Version:    24.0.7
Server:
 Containers: 0
  Running: 0
  Paused: 0
  Stopped: 0
 Images: 0
```

### Step 5: Check Existing Containers
```bash
sudo docker ps -a
```

**Expected output (if no containers):**
```
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

### Step 6: Search for Nginx Image (Optional)
```bash
sudo docker search nginx
```

**Expected output:**
```
NAME                DESCRIPTION                     STARS     OFFICIAL
nginx               Official build of Nginx.        19000+    [OK]
nginx/nginx-ingress Nginx Ingress Controller...     100+
```

### Step 7: Pull nginx:alpine Image
```bash
sudo docker pull nginx:alpine
```

**Expected output:**
```
alpine: Pulling from library/nginx
31e352740f53: Pull complete
...
Digest: sha256:a5127...
Status: Downloaded newer image for nginx:alpine
docker.io/library/nginx:alpine
```

**Note:** This downloads the image from Docker Hub

### Step 8: Verify Image Downloaded
```bash
sudo docker images
```

**Expected output:**
```
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
nginx        alpine    a5127a7414...  2 weeks ago   23.5MB
```

### Step 9: Create and Start Container
```bash
sudo docker run -d --name nginx_3 nginx:alpine
```

**Command breakdown:**
- `docker run` - Create and start container
- `-d` - Detached mode (run in background)
- `--name nginx_3` - Name the container
- `nginx:alpine` - Image to use

**Expected output:**
```
a7b9c8d3e4f5g6h7i8j9k0l1m2n3o4p5q6r7s8t9u0v1w2x3y4z5
```
(This is the container ID)

### Step 10: Verify Container is Running
```bash
sudo docker ps
```

**Expected output:**
```
CONTAINER ID   IMAGE          COMMAND                  CREATED         STATUS         PORTS     NAMES
a7b9c8d3e4f5   nginx:alpine   "/docker-entrypoint.…"   5 seconds ago   Up 4 seconds   80/tcp    nginx_3
```

**Key points to verify:**
- ✅ Container ID present
- ✅ IMAGE: nginx:alpine
- ✅ STATUS: Up X seconds
- ✅ NAMES: nginx_3

### Step 11: Check Container Details
```bash
sudo docker inspect nginx_3
```

**Expected output (partial):**
```json
[
    {
        "Id": "a7b9c8d3e4f5...",
        "Name": "/nginx_3",
        "State": {
            "Status": "running",
            "Running": true,
            "Paused": false,
            "Restarting": false,
            "Dead": false,
            ...
        },
        "Image": "sha256:a5127...",
        "Config": {
            "Image": "nginx:alpine",
            ...
        }
    }
]
```

### Step 12: Check Container Logs
```bash
sudo docker logs nginx_3
```

**Expected output:**
```
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
...
2025/12/10 10:00:00 [notice] 1#1: start worker processes
```

### Step 13: View Running Processes in Container
```bash
sudo docker top nginx_3
```

**Expected output:**
```
UID       PID       PPID      C     STIME     TTY     TIME        CMD
root      12345     12320     0     10:00     ?       00:00:00    nginx: master process
nginx     12346     12345     0     10:00     ?       00:00:00    nginx: worker process
```

### Step 14: Check Container Stats (Optional)
```bash
sudo docker stats nginx_3 --no-stream
```

**Expected output:**
```
CONTAINER ID   NAME      CPU %     MEM USAGE / LIMIT   MEM %     NET I/O     BLOCK I/O   PIDS
a7b9c8d3e4f5   nginx_3   0.00%     3.5MiB / 7.77GiB    0.04%     656B / 0B   0B / 0B     3
```

### Step 15: Test Nginx Inside Container
```bash
sudo docker exec nginx_3 nginx -t
```

**Expected output:**
```
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

### Step 16: Check Nginx Version
```bash
sudo docker exec nginx_3 nginx -v
```

**Expected output:**
```
nginx version: nginx/1.25.3
```

### Step 17: View Nginx Configuration
```bash
sudo docker exec nginx_3 cat /etc/nginx/nginx.conf
```

**Expected output (partial):**
```nginx
user  nginx;
worker_processes  auto;

error_log  /var/log/nginx/error.log notice;
pid        /var/run/nginx.pid;

events {
    worker_connections  1024;
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;
    ...
}
```

### Step 18: List All Containers (Including Stopped)
```bash
sudo docker ps -a
```

**Expected output:**
```
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS          PORTS     NAMES
a7b9c8d3e4f5   nginx:alpine   "/docker-entrypoint.…"   2 minutes ago    Up 2 minutes    80/tcp    nginx_3
```

### Step 19: Final Verification Checklist

**Check 1: Container exists and named correctly**
```bash
sudo docker ps --filter "name=nginx_3"
```

**Check 2: Using nginx:alpine image**
```bash
sudo docker ps --filter "ancestor=nginx:alpine"
```

**Check 3: Container is running (not stopped)**
```bash
sudo docker ps --filter "status=running" | grep nginx_3
```

**All checks should show the nginx_3 container**

---

## Complete Command Summary

### Quick Deployment:
```bash
# SSH to server
ssh banner@stapp03

# Verify Docker is running
sudo systemctl status docker

# Pull and run nginx container in one command
sudo docker run -d --name nginx_3 nginx:alpine

# Verify container is running
sudo docker ps

# Check logs
sudo docker logs nginx_3
```

### Detailed Verification:
```bash
# SSH and prepare
ssh banner@stapp03
sudo systemctl start docker

# Pull image explicitly
sudo docker pull nginx:alpine

# Verify image
sudo docker images | grep nginx

# Create container
sudo docker run -d --name nginx_3 nginx:alpine

# Comprehensive checks
sudo docker ps
sudo docker inspect nginx_3 | grep -i status
sudo docker logs nginx_3
sudo docker exec nginx_3 nginx -t

# View container stats
sudo docker stats nginx_3 --no-stream
```

---

## Understanding Docker Run Command

### Basic Syntax:
```bash
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
```

### Common Options:

| Option | Description | Example |
|--------|-------------|---------|
| `-d` | Detached (background) | `docker run -d nginx` |
| `--name` | Container name | `docker run --name web nginx` |
| `-p` | Port mapping | `docker run -p 8080:80 nginx` |
| `-v` | Volume mount | `docker run -v /data:/app nginx` |
| `-e` | Environment variable | `docker run -e "VAR=value" nginx` |
| `--rm` | Auto-remove on stop | `docker run --rm nginx` |
| `-it` | Interactive terminal | `docker run -it ubuntu bash` |
| `--restart` | Restart policy | `docker run --restart=always nginx` |

### Our Command Explained:
```bash
sudo docker run -d --name nginx_3 nginx:alpine
```

- `sudo` - Run with elevated privileges
- `docker run` - Create and start container
- `-d` - Detached mode (background)
- `--name nginx_3` - Name it nginx_3
- `nginx:alpine` - Use nginx with alpine tag

---

## Docker Container States

### State Lifecycle:

```
Created → Starting → Running → Pausing → Paused
   ↓                    ↓           ↓
Stopped ← Stopping ← Running ← Unpausing
   ↓
Removed
```

### State Commands:

| State | Command | Description |
|-------|---------|-------------|
| **Running** | `docker start` | Container is active |
| **Stopped** | `docker stop` | Container is inactive |
| **Paused** | `docker pause` | Processes frozen |
| **Restarting** | `docker restart` | Stop then start |
| **Removing** | `docker rm` | Delete container |
| **Dead** | - | Container failed |

### Check Container State:
```bash
# View state in ps output
sudo docker ps -a

# Detailed state info
sudo docker inspect nginx_3 | grep -A 10 "State"
```

**Expected state output:**
```json
"State": {
    "Status": "running",
    "Running": true,
    "Paused": false,
    "Restarting": false,
    "Dead": false,
    "Pid": 12345,
    "StartedAt": "2025-12-10T10:00:00Z"
}
```

---

## Troubleshooting

### Issue 1: Permission Denied

**Problem:**
```
Got permission denied while trying to connect to the Docker daemon socket
```

**Solution:**
```bash
# Use sudo
sudo docker ps

# OR add user to docker group (requires logout)
sudo usermod -aG docker banner
# Logout and login again
```

### Issue 2: Container Name Already Exists

**Problem:**
```
Error response from daemon: Conflict. The container name "/nginx_3" is already in use
```

**Solution:**
```bash
# Check existing containers
sudo docker ps -a | grep nginx_3

# Remove existing container
sudo docker rm nginx_3

# Or use force remove if running
sudo docker rm -f nginx_3

# Now create new one
sudo docker run -d --name nginx_3 nginx:alpine
```

### Issue 3: Image Not Found Locally

**Problem:**
```
Unable to find image 'nginx:alpine' locally
```

**Solution:**
```bash
# This is normal - Docker will automatically pull
# But you can pull explicitly first
sudo docker pull nginx:alpine

# Then run
sudo docker run -d --name nginx_3 nginx:alpine
```

### Issue 4: Container Exits Immediately

**Problem:**
```
sudo docker ps
# nginx_3 not showing in running containers
```

**Solution:**
```bash
# Check all containers including stopped
sudo docker ps -a

# View logs to see why it stopped
sudo docker logs nginx_3

# Check exit code
sudo docker inspect nginx_3 | grep -i exitcode

# Common causes:
# - Wrong command
# - Configuration error
# - Missing dependencies

# For nginx:alpine, this should not happen
# If it does, recreate:
sudo docker rm nginx_3
sudo docker run -d --name nginx_3 nginx:alpine
```

### Issue 5: Docker Service Not Running

**Problem:**
```
Cannot connect to the Docker daemon at unix:///var/run/docker.sock
```

**Solution:**
```bash
# Check Docker status
sudo systemctl status docker

# Start Docker
sudo systemctl start docker

# Enable on boot
sudo systemctl enable docker

# Verify
sudo docker info
```

### Issue 6: Port Already in Use

**Problem:**
```
Error starting userland proxy: listen tcp 0.0.0.0:80: bind: address already in use
```

**Solution:**
```bash
# This task doesn't expose ports, but if you do:

# Check what's using port 80
sudo netstat -tulpn | grep :80
sudo lsof -i :80

# Use different host port
sudo docker run -d --name nginx_3 -p 8080:80 nginx:alpine
```

---

## Docker Container Management

### Starting and Stopping:

```bash
# Stop container
sudo docker stop nginx_3

# Start stopped container
sudo docker start nginx_3

# Restart container
sudo docker restart nginx_3

# Pause container (freeze processes)
sudo docker pause nginx_3

# Unpause container
sudo docker unpause nginx_3

# Kill container (force stop)
sudo docker kill nginx_3
```

### Viewing Information:

```bash
# List running containers
sudo docker ps

# List all containers
sudo docker ps -a

# Container details (JSON)
sudo docker inspect nginx_3

# Container logs
sudo docker logs nginx_3

# Follow logs (real-time)
sudo docker logs -f nginx_3

# Last 100 lines
sudo docker logs --tail 100 nginx_3

# Resource usage
sudo docker stats nginx_3
```

### Executing Commands:

```bash
# Run command in container
sudo docker exec nginx_3 ls /etc/nginx

# Interactive shell
sudo docker exec -it nginx_3 /bin/sh

# As specific user
sudo docker exec -u nginx nginx_3 whoami

# Check nginx status
sudo docker exec nginx_3 nginx -t
```

### Copying Files:

```bash
# Copy from container to host
sudo docker cp nginx_3:/etc/nginx/nginx.conf ./nginx.conf

# Copy from host to container
sudo docker cp ./index.html nginx_3:/usr/share/nginx/html/
```

### Removing Containers:

```bash
# Remove stopped container
sudo docker rm nginx_3

# Force remove running container
sudo docker rm -f nginx_3

# Remove all stopped containers
sudo docker container prune

# Remove container and volumes
sudo docker rm -v nginx_3
```

---

## Nginx Container Testing

### Test 1: Nginx Process Running
```bash
sudo docker exec nginx_3 ps aux | grep nginx
```

**Expected output:**
```
    1 root      0:00 nginx: master process nginx -g daemon off;
    7 nginx     0:00 nginx: worker process
```

### Test 2: Nginx Listening on Port 80
```bash
sudo docker exec nginx_3 netstat -tulpn | grep :80
```

**Expected output:**
```
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      1/nginx: master pro
```

### Test 3: Nginx Configuration Valid
```bash
sudo docker exec nginx_3 nginx -t
```

**Expected output:**
```
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

### Test 4: Default Welcome Page Exists
```bash
sudo docker exec nginx_3 cat /usr/share/nginx/html/index.html
```

**Expected output:**
```html
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
...
```

### Test 5: Curl Inside Container
```bash
sudo docker exec nginx_3 wget -q -O - http://localhost
```

**Expected:** HTML content of welcome page

---

## Understanding Docker Images

### Image Layers:

```
nginx:alpine
    ↓
[Layer 1] Alpine Linux base (~5MB)
[Layer 2] Nginx binaries (~15MB)
[Layer 3] Configuration files (~3MB)
    ↓
Total: ~23MB
```

### Image Management:

```bash
# List images
sudo docker images

# Remove image
sudo docker rmi nginx:alpine

# Pull specific version
sudo docker pull nginx:1.25-alpine

# Image history
sudo docker history nginx:alpine

# Image details
sudo docker inspect nginx:alpine
```

### Image Tags:

```bash
# Latest (default)
docker pull nginx

# Specific version
docker pull nginx:1.25

# Alpine variant
docker pull nginx:alpine

# Specific version + alpine
docker pull nginx:1.25-alpine
```

---

## Best Practices

### 1. Use Specific Tags
```bash
# ❌ Avoid
docker run nginx

# ✅ Better
docker run nginx:alpine

# ✅ Best (for production)
docker run nginx:1.25.3-alpine
```

### 2. Name Your Containers
```bash
# ❌ Auto-generated name
docker run -d nginx:alpine

# ✅ Descriptive name
docker run -d --name nginx_3 nginx:alpine
```

### 3. Use Health Checks
```bash
docker run -d --name nginx_3 \
  --health-cmd="wget -q --spider http://localhost || exit 1" \
  --health-interval=30s \
  --health-timeout=3s \
  --health-retries=3 \
  nginx:alpine
```

### 4. Set Restart Policy
```bash
# Restart on failure
docker run -d --name nginx_3 --restart=unless-stopped nginx:alpine
```

### 5. Limit Resources
```bash
# Limit CPU and memory
docker run -d --name nginx_3 \
  --cpus="0.5" \
  --memory="256m" \
  nginx:alpine
```

### 6. Use Read-Only Root Filesystem (Security)
```bash
docker run -d --name nginx_3 \
  --read-only \
  --tmpfs /var/cache/nginx \
  --tmpfs /var/run \
  nginx:alpine
```

### 7. Don't Run as Root
```bash
# Check user
docker exec nginx_3 whoami
# nginx (not root - nginx:alpine already does this)
```

---

## Real-World Applications

### 1. Development Environment
```bash
# Quick local web server
docker run -d --name dev_nginx \
  -p 8080:80 \
  -v $(pwd):/usr/share/nginx/html:ro \
  nginx:alpine

# Access at http://localhost:8080
```

### 2. Reverse Proxy
```bash
# Nginx as reverse proxy
docker run -d --name proxy \
  -p 80:80 \
  -v /path/to/nginx.conf:/etc/nginx/nginx.conf:ro \
  nginx:alpine
```

### 3. Load Balancer
```bash
# Nginx load balancing multiple backends
docker run -d --name lb \
  -p 80:80 \
  -v /path/to/lb.conf:/etc/nginx/nginx.conf:ro \
  --link backend1 \
  --link backend2 \
  nginx:alpine
```

### 4. Static Website Hosting
```bash
# Host static site
docker run -d --name website \
  -p 80:80 \
  -v /var/www/html:/usr/share/nginx/html:ro \
  nginx:alpine
```

### 5. SSL/TLS Termination
```bash
# Nginx with SSL
docker run -d --name ssl_nginx \
  -p 443:443 \
  -v /etc/ssl/certs:/etc/nginx/certs:ro \
  -v /path/to/nginx-ssl.conf:/etc/nginx/nginx.conf:ro \
  nginx:alpine
```

---

## Docker vs Traditional Deployment

### Traditional Nginx Installation:
```bash
# Install nginx
sudo yum install nginx

# Configure
sudo vi /etc/nginx/nginx.conf

# Start service
sudo systemctl start nginx
sudo systemctl enable nginx

# Issues:
# - OS-specific
# - System-wide installation
# - Version conflicts
# - Dependency hell
# - Hard to reproduce
```

### Docker Nginx Deployment:
```bash
# One command
docker run -d --name nginx_3 nginx:alpine

# Benefits:
# ✅ Consistent across environments
# ✅ Isolated from host
# ✅ Version controlled
# ✅ Easy rollback
# ✅ Reproducible
# ✅ Portable
```

---

## Key Docker Commands Reference

| Command | Description |
|---------|-------------|
| `docker run` | Create and start container |
| `docker ps` | List running containers |
| `docker ps -a` | List all containers |
| `docker start` | Start stopped container |
| `docker stop` | Stop running container |
| `docker restart` | Restart container |
| `docker rm` | Remove container |
| `docker logs` | View container logs |
| `docker exec` | Run command in container |
| `docker inspect` | Detailed container info |
| `docker stats` | Resource usage |
| `docker pull` | Download image |
| `docker images` | List images |

---

## Verification Checklist

- [ ] SSH into Application Server 3 (stapp03) as banner
- [ ] Verified Docker service is running
- [ ] Pulled nginx:alpine image
- [ ] Created container named nginx_3
- [ ] Container using nginx:alpine image
- [ ] Container in running state (docker ps shows it)
- [ ] Verified container logs show nginx started
- [ ] Tested nginx configuration valid
- [ ] Confirmed nginx processes running
- [ ] Container accessible and responding

---

## Completion Details

- **Completion Date:** December 10, 2025
- **Day:** 36 / 100
- **Challenge:** KodeKloud 100 Days Cloud DevOps
- **Topic:** Docker Container Deployment - Nginx
- **Server:** Application Server 3 (stapp03)
- **User:** banner
- **Container:** nginx_3
- **Image:** nginx:alpine
- **State:** Running ✅
- **Port:** 80 (internal)
- **Key Skill:** Docker container basics and nginx deployment
- **Status:** ✅ Successfully Completed

---

## Summary

This task demonstrated **Docker container deployment** fundamentals:

✅ **Pulled nginx:alpine image** - Lightweight 23MB image
✅ **Created named container** - nginx_3 with descriptive name
✅ **Started in detached mode** - Running in background
✅ **Verified running state** - Container healthy and active
✅ **Tested nginx service** - Configuration valid, processes running

**Key Insight:** Containers revolutionize application deployment by providing:
- **Consistency** across environments
- **Isolation** from host system
- **Portability** - run anywhere
- **Efficiency** - lightweight and fast
- **Simplicity** - one command deployment

Unlike traditional installations requiring multiple steps and OS-specific configurations, Docker containers deploy in seconds with guaranteed consistency. The nginx:alpine image demonstrates container efficiency - a full web server in just 23MB!

**Remember:** `docker run -d --name nginx_3 nginx:alpine` = Production-Ready Web Server in Seconds! 🐳

This marks the beginning of your containerization journey - from here, you'll explore Docker networking, volumes, compose, and orchestration!