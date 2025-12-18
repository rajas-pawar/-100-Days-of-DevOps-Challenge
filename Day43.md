# Day 43: Nginx Container with Port Mapping
**100 Days Cloud DevOps Challenge – KodeKloud**

---

## Objective

Deploy an Nginx web server container with custom port mapping to host an application in a containerized environment.

**Requirements:**
1. Work on Application Server 3 (App Server 3)
2. Pull Image: `nginx:stable`
3. Container Name: `beta`
4. Port Mapping: Host port `6400` → Container port `80`
5. State: Keep container running

---

## Understanding Port Mapping

**Port Mapping** is the mechanism that allows external access to services running inside Docker containers. It maps a port on the host machine to a port inside the container.

### What is Port Mapping?

Port mapping creates a bridge between:
- **Host Port** - External access point on the server
- **Container Port** - Internal port where service listens

### How Port Mapping Works:

```
External Client (Browser/curl)
         ↓
    Host Port 6400
         ↓
    Docker Engine (Port Mapping)
         ↓
    Container Port 80
         ↓
    Nginx Web Server
```

### Without Port Mapping:

```
Container (Isolated Network)
├── nginx listening on port 80
└── ❌ Not accessible from outside

External Client → Cannot reach nginx
```

### With Port Mapping (-p 6400:80):

```
Host Machine
├── Port 6400 (publicly accessible)
    ↓ Forwards to
    Container Port 80
    ├── nginx listening
    └── ✅ Accessible via host:6400

External Client → host:6400 → container:80 → nginx
```

---

## Understanding Port Mapping Syntax

### Basic Syntax:

```bash
docker run -p HOST_PORT:CONTAINER_PORT image
```

### Our Task:

```bash
docker run -d --name beta -p 6400:80 nginx:stable

# Breakdown:
-p 6400:80
   ↓     ↓
   │     └─ Container Port (where nginx listens)
   └─ Host Port (external access)
```

### Port Mapping Variations:

**1. Single Port Mapping:**
```bash
docker run -p 8080:80 nginx
# Access: http://host:8080 → container:80
```

**2. Multiple Port Mappings:**
```bash
docker run -p 80:80 -p 443:443 nginx
# HTTP: host:80 → container:80
# HTTPS: host:443 → container:443
```

**3. Bind to Specific Interface:**
```bash
docker run -p 127.0.0.1:8080:80 nginx
# Only accessible from localhost
```

**4. Random Host Port:**
```bash
docker run -p 80 nginx
# Docker assigns random host port
```

**5. All Exposed Ports:**
```bash
docker run -P nginx
# Maps all EXPOSE ports to random host ports
```

### Port Mapping Table:

| Command | Host Port | Container Port | Access |
|---------|-----------|----------------|--------|
| `-p 6400:80` | 6400 | 80 | host:6400 |
| `-p 80:80` | 80 | 80 | host:80 |
| `-p 8080:8080` | 8080 | 8080 | host:8080 |
| `-p 127.0.0.1:8080:80` | 8080 | 80 | localhost:8080 only |
| `-P` | Random | All EXPOSE | host:random_port |

---

## Understanding Nginx Versions

### Nginx Image Variants:

| Tag | Description | Size | Use Case |
|-----|-------------|------|----------|
| **nginx:stable** | Stable release | ~143MB | **Production (our task)** |
| nginx:mainline | Latest features | ~143MB | Testing new features |
| nginx:alpine | Alpine-based | ~23MB | Size-optimized |
| nginx:stable-alpine | Stable + Alpine | ~23MB | Production + small size |
| nginx:latest | Latest mainline | ~143MB | Development |

### Why nginx:stable?

**Stable Release:**
- ✅ Tested and proven
- ✅ Production-ready
- ✅ Long-term support
- ✅ Security updates
- ✅ Fewer breaking changes

**Mainline Release:**
- ⚠️ Latest features
- ⚠️ More updates
- ⚠️ Potential instability
- ✅ New functionality
- ⚠️ More frequent changes

### Version Comparison:

```
nginx:stable (1.24.x)
├── Production-ready
├── LTS support
├── Security patches
└── Recommended for tasks

nginx:mainline (1.25.x)
├── Latest features
├── Active development
└── Use for testing

nginx:alpine
├── Same functionality
├── Smaller size (23MB vs 143MB)
└── Alpine Linux base
```

---

## Understanding the Scenario

### Application Team Requirements:

```
1. Need nginx web server for application hosting
2. Production-stable version required
3. Non-standard port to avoid conflicts
4. Container must remain running
5. Accessible from external clients
```

### Port Selection:

**Why Port 6400?**
- Avoids standard web ports (80, 443, 8080)
- No conflict with existing services
- Above 1024 (no root privileges needed)
- Specific to this application/environment

### Container Naming:

**Why "beta"?**
- Identifies environment (beta testing)
- Easy reference in commands
- Follows naming conventions
- Clear purpose indication

---

## Infrastructure Overview

### Application Servers:
| Server | User | Password | IP |
|--------|------|----------|-----|
| stapp01 | tony | Ir0nM@n | 172.16.238.10 |
| stapp02 | steve | Am3ric@ | 172.16.238.11 |
| **stapp03** | **banner** | **BigGr33n** | **172.16.238.12** |

### Task Details:
| Item | Value |
|------|-------|
| Server | Application Server 3 (stapp03) |
| User | banner |
| Image | `nginx:stable` |
| Container Name | `beta` |
| Host Port | 6400 |
| Container Port | 80 |
| State | Running |

---

## Understanding the Task

### What We're Creating:

```
Application Server 3 (stapp03)
│
└── Docker Container: beta
    ├── Image: nginx:stable
    ├── Internal Port: 80 (nginx default)
    ├── External Port: 6400 (mapped)
    ├── State: Running
    └── Access: http://stapp03:6400
```

### Network Flow:

```
External Request
    ↓
http://172.16.238.12:6400
    ↓
Host Machine (stapp03)
Port 6400
    ↓
Docker Port Mapping
    ↓
Container "beta"
Port 80
    ↓
Nginx Web Server
    ↓
Response (HTML)
```

### Deployment Flow:

```
1. SSH to App Server 3
2. Switch to root
3. Pull nginx:stable image
4. Create container named "beta"
5. Map port 6400 (host) → 80 (container)
6. Verify container running
7. Test nginx accessibility
8. Verify port mapping active
```

---

## Step-by-Step Implementation

### Step 1: SSH into Application Server 3
```bash
ssh banner@stapp03
```

**Expected output:**
```
The authenticity of host 'stapp03 (172.16.238.12)' can't be established.
ED25519 key fingerprint is SHA256:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'stapp03' (ED25519) to the list of known hosts.
banner@stapp03's password:
```

**Enter password:** `BigGr33n`

```
[banner@stapp03 ~]$
```

### Step 2: Switch to Root User
```bash
sudo su -
```

**Expected output:**
```
We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

[sudo] password for banner:
```

**Enter password:** `BigGr33n`

```
[root@stapp03 ~]#
```

### Step 3: Check Existing Docker Images
```bash
docker images
```

**Expected output:**
```
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
```

**Or may show existing images**

### Step 4: Check if nginx:stable Already Exists
```bash
docker images | grep nginx
```

**If empty, nginx not pulled yet**

### Step 5: Pull nginx:stable Image
```bash
docker pull nginx:stable
```

**Expected output:**
```
stable: Pulling from library/nginx
a803e7c4b030: Pull complete
8b625c47d697: Pull complete
4d3239651a63: Pull complete
0f816efa513d: Pull complete
01d159b8db2f: Pull complete
5fb9a81470f3: Pull complete
9b1e1e7164db: Pull complete
Digest: sha256:a5b5b5f8e3c6d0b9c7f8e9d1a2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b1
Status: Downloaded newer image for nginx:stable
docker.io/library/nginx:stable
```

**Pull process breakdown:**
1. Contacts Docker Hub
2. Downloads nginx:stable manifest
3. Pulls each layer (7 layers)
4. Verifies integrity
5. Stores locally

### Step 6: Verify Image Downloaded
```bash
docker images
```

**Expected output:**
```
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
nginx        stable    a1b2c3d4e5f6   2 weeks ago   143MB
```

**Check specific image:**
```bash
docker images nginx:stable
```

**Expected output:**
```
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
nginx        stable    a1b2c3d4e5f6   2 weeks ago   143MB
```

### Step 7: Inspect Image Details (Optional)
```bash
docker inspect nginx:stable
```

**Expected output (partial):**
```json
[
    {
        "Id": "sha256:a1b2c3d4e5f6...",
        "RepoTags": [
            "nginx:stable"
        ],
        "Created": "2025-12-01T10:30:45.123456789Z",
        "Config": {
            "ExposedPorts": {
                "80/tcp": {}
            },
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                "NGINX_VERSION=1.24.0"
            ],
            "Cmd": [
                "nginx",
                "-g",
                "daemon off;"
            ]
        },
        "Architecture": "amd64",
        "Size": 142694567
    }
]
```

**Notice:**
- ExposedPorts: 80/tcp
- Size: ~143MB

### Step 8: Check Existing Containers
```bash
docker ps -a
```

**Expected output:**
```
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

**Or may show existing containers**

### Step 9: Check if "beta" Container Already Exists
```bash
docker ps -a | grep beta
```

**Should be empty if beta doesn't exist**

### Step 10: Create and Run Container with Port Mapping
```bash
docker run -d --name beta -p 6400:80 nginx:stable
```

**Expected output:**
```
d4e5f6a7b8c9d0e1f2a3b4c5d6e7f8a9b0c1d2e3f4e5f6a7b8c9d0e1f2a3b4c5
```

**This is the container ID (yours will differ)**

**Command breakdown:**
- `docker run` - Create and start container
- `-d` - Detached mode (background)
- `--name beta` - Container name
- `-p 6400:80` - Port mapping (host:container)
- `nginx:stable` - Image to use

### Step 11: Verify Container Running
```bash
docker ps
```

**Expected output:**
```
CONTAINER ID   IMAGE          COMMAND                  CREATED         STATUS         PORTS                                       NAMES
d4e5f6a7b8c9   nginx:stable   "/docker-entrypoint.…"   10 seconds ago  Up 9 seconds   0.0.0.0:6400->80/tcp, :::6400->80/tcp      beta
```

**Verify:**
- ✅ Container name: `beta`
- ✅ Image: `nginx:stable`
- ✅ Status: `Up`
- ✅ Ports: `0.0.0.0:6400->80/tcp`

**Filter for beta container:**
```bash
docker ps --filter "name=beta"
```

**Expected output:**
```
CONTAINER ID   IMAGE          COMMAND                  CREATED         STATUS         PORTS                                       NAMES
d4e5f6a7b8c9   nginx:stable   "/docker-entrypoint.…"   15 seconds ago  Up 14 seconds  0.0.0.0:6400->80/tcp, :::6400->80/tcp      beta
```

### Step 12: Check Container Details
```bash
docker inspect beta
```

**Expected output (partial):**
```json
[
    {
        "Id": "d4e5f6a7b8c9d0e1f2a3b4c5d6e7f8a9b0c1d2e3f4e5f6a7b8c9d0e1f2a3b4c5",
        "Name": "/beta",
        "State": {
            "Status": "running",
            "Running": true,
            "Paused": false,
            "Restarting": false,
            "StartedAt": "2025-12-18T10:30:45.123456789Z"
        },
        "Image": "sha256:a1b2c3d4e5f6...",
        "NetworkSettings": {
            "Ports": {
                "80/tcp": [
                    {
                        "HostIp": "0.0.0.0",
                        "HostPort": "6400"
                    },
                    {
                        "HostIp": "::",
                        "HostPort": "6400"
                    }
                ]
            },
            "IPAddress": "172.17.0.2"
        }
    }
]
```

**Key details:**
- State: "running"
- HostPort: "6400"
- Container Port: 80
- IP: 172.17.0.2

### Step 13: Check Port Mapping
```bash
docker port beta
```

**Expected output:**
```
80/tcp -> 0.0.0.0:6400
80/tcp -> :::6400
```

**Confirms:**
- Container port 80 mapped to host port 6400
- Accessible on all interfaces (0.0.0.0)
- IPv6 also mapped (:::6400)

### Step 14: Check Container Logs
```bash
docker logs beta
```

**Expected output:**
```
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
2025/12/18 10:30:45 [notice] 1#1: using the "epoll" event method
2025/12/18 10:30:45 [notice] 1#1: nginx/1.24.0
2025/12/18 10:30:45 [notice] 1#1: built by gcc 12.2.0 (Debian 12.2.0-14)
2025/12/18 10:30:45 [notice] 1#1: OS: Linux 5.4.0-192-generic
2025/12/18 10:30:45 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1048576:1048576
2025/12/18 10:30:45 [notice] 1#1: start worker processes
2025/12/18 10:30:45 [notice] 1#1: start worker process 29
```

**✅ Nginx started successfully!**

### Step 15: Test Nginx from Host
```bash
curl http://localhost:6400
```

**Expected output (partial HTML):**
```html
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

**✅ Nginx responding on port 6400!**

### Step 16: Test with HTTP Headers
```bash
curl -I http://localhost:6400
```

**Expected output:**
```
HTTP/1.1 200 OK
Server: nginx/1.24.0
Date: Wed, 18 Dec 2025 10:35:30 GMT
Content-Type: text/html
Content-Length: 615
Last-Modified: Wed, 01 Dec 2025 10:30:00 GMT
Connection: keep-alive
ETag: "674c1234-267"
Accept-Ranges: bytes
```

**Verify:**
- ✅ HTTP 200 OK
- ✅ Server: nginx/1.24.0
- ✅ Content-Type: text/html

### Step 17: Check Port Listening on Host
```bash
netstat -tuln | grep 6400
```

**Expected output:**
```
tcp        0      0 0.0.0.0:6400            0.0.0.0:*               LISTEN
tcp6       0      0 :::6400                 :::*                    LISTEN
```

**Confirms port 6400 is listening on all interfaces**

**Alternative with ss:**
```bash
ss -tuln | grep 6400
```

**Expected output:**
```
tcp   LISTEN 0      4096         0.0.0.0:6400       0.0.0.0:*
tcp   LISTEN 0      4096            [::]:6400          [::]:*
```

### Step 18: Test from Container IP
```bash
# Get container IP
CONTAINER_IP=$(docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' beta)
echo $CONTAINER_IP

# Test directly to container
curl http://$CONTAINER_IP:80
```

**Expected output:**
```
172.17.0.2

<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
...
```

**Both work:**
- ✅ Host: http://localhost:6400
- ✅ Container IP: http://172.17.0.2:80

### Step 19: Check Container Stats (Optional)
```bash
docker stats beta --no-stream
```

**Expected output:**
```
CONTAINER ID   NAME      CPU %     MEM USAGE / LIMIT     MEM %     NET I/O          BLOCK I/O   PIDS
d4e5f6a7b8c9   beta      0.00%     3.5MiB / 7.77GiB     0.04%     1.23kB / 866B    0B / 0B     3
```

**Shows:**
- CPU usage
- Memory usage
- Network I/O
- Number of processes

### Step 20: Verify All Requirements Met
```bash
# 1. Image pulled
docker images nginx:stable

# 2. Container named beta exists
docker ps --filter "name=beta"

# 3. Port mapping 6400:80 active
docker port beta

# 4. Container running
docker ps | grep beta | grep Up

# 5. Nginx accessible
curl -I http://localhost:6400
```

**All checks should pass:**
- ✅ nginx:stable image present
- ✅ Container "beta" running
- ✅ Port 6400 mapped to 80
- ✅ HTTP 200 OK response

---

## Complete Command Summary

### Quick Deployment:
```bash
# SSH and access
ssh banner@stapp03
sudo su -

# Pull and run
docker pull nginx:stable
docker run -d --name beta -p 6400:80 nginx:stable

# Verify
docker ps
curl http://localhost:6400
```

### Detailed Workflow:
```bash
# 1. SSH and authenticate
ssh banner@stapp03
sudo su -

# 2. Pull nginx:stable image
docker pull nginx:stable

# 3. Verify image
docker images nginx:stable

# 4. Create container with port mapping
docker run -d --name beta -p 6400:80 nginx:stable

# 5. Verify container running
docker ps
docker ps --filter "name=beta"

# 6. Check port mapping
docker port beta

# 7. Check logs
docker logs beta

# 8. Test nginx
curl http://localhost:6400
curl -I http://localhost:6400

# 9. Verify port listening
netstat -tuln | grep 6400

# 10. Check container details
docker inspect beta
```

---

## Understanding Port Mapping in Depth

### Port Mapping Scenarios:

**Scenario 1: Standard Web Server**
```bash
docker run -d -p 80:80 nginx
# Host:80 → Container:80
# Access: http://host/
```

**Scenario 2: Non-Standard Port (Our Task)**
```bash
docker run -d -p 6400:80 nginx
# Host:6400 → Container:80
# Access: http://host:6400/
```

**Scenario 3: Multiple Services**
```bash
# Service 1
docker run -d -p 8001:80 --name web1 nginx

# Service 2
docker run -d -p 8002:80 --name web2 nginx

# Both accessible:
# http://host:8001/ → web1
# http://host:8002/ → web2
```

**Scenario 4: Multiple Ports Same Container**
```bash
docker run -d -p 80:80 -p 443:443 nginx
# HTTP: host:80 → container:80
# HTTPS: host:443 → container:443
```

### Port Mapping Rules:

**✅ Valid:**
```bash
-p 6400:80        # Different host/container ports
-p 80:80          # Same host/container ports
-p 8080:8080      # Any valid port number
-p 127.0.0.1:8080:80  # Localhost only
```

**❌ Invalid:**
```bash
-p 80:6400 -p 80:8080   # Same host port twice
-p 6400 nginx           # No container port specified
-p :80 nginx            # No host port specified
```

### Finding Used Ports:

```bash
# All listening ports
netstat -tuln

# Docker managed ports
docker ps --format "table {{.Names}}\t{{.Ports}}"

# Specific container ports
docker port container_name
```

---

## Advanced Port Mapping

### Binding to Specific Interface:

**Localhost Only:**
```bash
docker run -d -p 127.0.0.1:6400:80 nginx
# Only accessible from host machine
# http://localhost:6400/ ✅
# http://host_ip:6400/ ❌
```

**Specific IP:**
```bash
docker run -d -p 192.168.1.10:6400:80 nginx
# Only accessible via 192.168.1.10
```

**All Interfaces (Default):**
```bash
docker run -d -p 6400:80 nginx
# Same as: -p 0.0.0.0:6400:80
# Accessible from anywhere
```

### Random Port Assignment:

```bash
# Let Docker choose host port
docker run -d -p 80 nginx

# Check assigned port
docker port container_name
# 80/tcp -> 0.0.0.0:32768

# Access via random port
curl http://localhost:32768
```

### Publishing All Exposed Ports:

```bash
# Nginx exposes port 80
docker run -d -P nginx

# Docker assigns random port
docker ps
# PORTS: 0.0.0.0:32769->80/tcp

# Check assignment
docker port container_name
```

### UDP Port Mapping:

```bash
# TCP (default)
docker run -d -p 6400:80/tcp nginx

# UDP
docker run -d -p 6400:80/udp custom-app

# Both TCP and UDP
docker run -d -p 6400:80/tcp -p 6400:80/udp custom-app
```

---

## Port Mapping Best Practices

### 1. Use Non-Standard Ports for Testing
```bash
# ✅ Good: Avoids conflicts
docker run -d -p 8080:80 nginx
docker run -d -p 8081:80 nginx

# ⚠️ Risky: May conflict
docker run -d -p 80:80 nginx
# Error if port 80 already in use
```

### 2. Document Port Assignments
```bash
# Create port mapping file
cat > port-mapping.txt << EOF
Service: beta-nginx
Host Port: 6400
Container Port: 80
Purpose: Beta environment web server
EOF
```

### 3. Use Environment-Specific Ports
```bash
# Development
docker run -d -p 8080:80 --name dev-nginx nginx

# Staging
docker run -d -p 8081:80 --name staging-nginx nginx

# Production
docker run -d -p 80:80 --name prod-nginx nginx
```

### 4. Limit Exposure When Needed
```bash
# Internal only
docker run -d -p 127.0.0.1:6400:80 nginx

# Public
docker run -d -p 0.0.0.0:6400:80 nginx
```

### 5. Check Port Availability First
```bash
# Check if port is free
netstat -tuln | grep 6400

# If empty, port is available
if ! netstat -tuln | grep -q 6400; then
    docker run -d -p 6400:80 --name beta nginx:stable
else
    echo "Port 6400 already in use"
fi
```

---

## Troubleshooting

### Issue 1: Port Already in Use

**Problem:**
```
Error response from daemon: driver failed programming external connectivity: Bind for 0.0.0.0:6400 failed: port is already allocated
```

**Solution:**
```bash
# Check what's using the port
netstat -tuln | grep 6400
lsof -i :6400

# If another container
docker ps | grep 6400

# Stop conflicting container
docker stop container_name

# Or use different port
docker run -d -p 6401:80 --name beta nginx:stable
```

### Issue 2: Cannot Access from External Host

**Problem:**
Can access from localhost but not from other machines

**Solution:**
```bash
# Check if bound to localhost only
docker port beta
# If shows 127.0.0.1:6400, recreate with 0.0.0.0

# Remove and recreate
docker rm -f beta
docker run -d --name beta -p 0.0.0.0:6400:80 nginx:stable

# Check firewall
sudo firewall-cmd --list-ports
sudo firewall-cmd --add-port=6400/tcp --permanent
sudo firewall-cmd --reload
```

### Issue 3: Container Name Already Exists

**Problem:**
```
Error response from daemon: Conflict. The container name "/beta" is already in use
```

**Solution:**
```bash
# Check existing container
docker ps -a | grep beta

# Option 1: Remove old container
docker rm -f beta
docker run -d --name beta -p 6400:80 nginx:stable

# Option 2: Use different name
docker run -d --name beta2 -p 6400:80 nginx:stable
```

### Issue 4: Image Not Found

**Problem:**
```
Unable to find image 'nginx:stable' locally
docker: Error response from daemon: manifest for nginx:stable not found
```

**Solution:**
```bash
# Pull image first
docker pull nginx:stable

# Verify available
docker images nginx

# Check tag exists
# nginx:stable should exist
```

### Issue 5: Container Exits Immediately

**Problem:**
```
docker ps
# Container not listed

docker ps -a
# STATUS: Exited (0) 5 seconds ago
```

**Solution:**
```bash
# Check logs
docker logs beta

# Nginx should run continuously
# If exits, might be resource issue or conflict

# Remove and recreate
docker rm beta
docker run -d --name beta -p 6400:80 nginx:stable
```

### Issue 6: Connection Refused

**Problem:**
```
curl http://localhost:6400
curl: (7) Failed to connect to localhost port 6400: Connection refused
```

**Solution:**
```bash
# Check container running
docker ps | grep beta

# Check port mapping
docker port beta

# Check nginx logs
docker logs beta

# Verify nginx process in container
docker exec beta ps aux | grep nginx

# Test container directly
docker exec beta curl http://localhost:80
```

### Issue 7: Firewall Blocking

**Problem:**
Works locally but not from other machines

**Solution:**
```bash
# Check firewall status
sudo firewall-cmd --state

# Add port to firewall
sudo firewall-cmd --add-port=6400/tcp --permanent
sudo firewall-cmd --reload

# Verify
sudo firewall-cmd --list-ports

# Or disable firewall (testing only)
sudo systemctl stop firewalld
```

---

## Real-World Scenarios

### Scenario 1: Multiple Environment Deployments
```bash
# Development environment
docker run -d --name dev-beta -p 8080:80 nginx:stable

# Staging environment
docker run -d --name staging-beta -p 8081:80 nginx:stable

# Production environment
docker run -d --name prod-beta -p 80:80 nginx:stable

# All running simultaneously on different ports
```

### Scenario 2: Load Balancer Setup
```bash
# Backend servers
docker run -d --name backend1 -p 8001:80 nginx:stable
docker run -d --name backend2 -p 8002:80 nginx:stable
docker run -d --name backend3 -p 8003:80 nginx:stable

# Load balancer (HAProxy/Nginx) distributes to 8001, 8002, 8003
```

### Scenario 3: Microservices Architecture
```bash
# API Gateway
docker run -d --name api-gateway -p 80:80 gateway:latest

# Auth Service
docker run -d --name auth-service -p 8001:80 auth:latest

# User Service
docker run -d --name user-service -p 8002:80 users:latest

# Product Service
docker run -d --name product-service -p 8003:80 products:latest
```

### Scenario 4: Development with Hot Reload
```bash
# Run nginx with custom config mounted
docker run -d \
  --name dev-nginx \
  -p 8080:80 \
  -v $(pwd)/nginx.conf:/etc/nginx/nginx.conf \
  -v $(pwd)/html:/usr/share/nginx/html \
  nginx:stable

# Changes to html reflect immediately
```

### Scenario 5: SSL/TLS Setup
```bash
# HTTP and HTTPS
docker run -d \
  --name secure-nginx \
  -p 80:80 \
  -p 443:443 \
  -v $(pwd)/certs:/etc/nginx/certs \
  nginx:stable

# Access via:
# http://host:80
# https://host:443
```

---

## Testing and Validation

### Complete Test Suite:

```bash
#!/bin/bash
# test-nginx.sh

echo "Testing Nginx Deployment..."

# Test 1: Image exists
echo "1. Checking image..."
if docker images | grep -q "nginx.*stable"; then
    echo "✅ nginx:stable image present"
else
    echo "❌ Image not found"
    exit 1
fi

# Test 2: Container running
echo "2. Checking container..."
if docker ps | grep -q beta; then
    echo "✅ Container 'beta' is running"
else
    echo "❌ Container not running"
    exit 1
fi

# Test 3: Port mapping
echo "3. Checking port mapping..."
if docker port beta | grep -q "6400"; then
    echo "✅ Port 6400 mapped"
else
    echo "❌ Port mapping failed"
    exit 1
fi

# Test 4: HTTP response
echo "4. Testing HTTP response..."
if curl -s -o /dev/null -w "%{http_code}" http://localhost:6400 | grep -q "200"; then
    echo "✅ HTTP 200 OK"
else
    echo "❌ HTTP request failed"
    exit 1
fi

# Test 5: Response content
echo "5. Checking response content..."
if curl -s http://localhost:6400 | grep -q "Welcome to nginx"; then
    echo "✅ Nginx welcome page served"
else
    echo "❌ Unexpected content"
    exit 1
fi

echo ""
echo "All tests passed! ✅"
echo "Access nginx at: http://$(hostname):6400"
```

**Run tests:**
```bash
chmod +x test-nginx.sh
./test-nginx.sh
```

---

## Key Commands Reference

| Command | Description |
|---------|-------------|
| `docker pull nginx:stable` | Pull nginx stable image |
| `docker run -d -p HOST:CONTAINER image` | Run with port mapping |
| `docker run -d --name NAME -p PORT:PORT image` | Run with name and port |
| `docker ps` | List running containers |
| `docker port CONTAINER` | Show port mappings |
| `docker logs CONTAINER` | View container logs |
| `docker inspect CONTAINER` | Detailed container info |
| `docker stop CONTAINER` | Stop container |
| `docker start CONTAINER` | Start stopped container |
| `docker rm CONTAINER` | Remove container |
| `curl http://localhost:PORT` | Test HTTP endpoint |
| `netstat -tuln | grep PORT` | Check port listening |

---

## Port Mapping Quick Reference

### Common Port Mappings:

```bash
# Web servers
-p 80:80          # HTTP
-p 443:443        # HTTPS
-p 8080:80        # HTTP alt
-p 8443:443       # HTTPS alt

# Databases
-p 3306:3306      # MySQL
-p 5432:5432      # PostgreSQL
-p 27017:27017    # MongoDB
-p 6379:6379      # Redis

# Application servers
-p 8080:8080      # Tomcat
-p 9090:9090      # Various apps
-p 3000:3000      # Node.js default

# Custom (like our task)
-p 6400:80        # Custom web port
```

---

## Completion Checklist

- [ ] SSH into Application Server 3 (stapp03) as banner
- [ ] Switched to root user
- [ ] Pulled nginx:stable image
- [ ] Verified image downloaded (~143MB)
- [ ] Created container named "beta"
- [ ] Mapped host port 6400 to container port 80
- [ ] Container running in detached mode
- [ ] Verified container status is "Up"
- [ ] Checked port mapping is active
- [ ] Tested nginx responds on port 6400
- [ ] Verified HTTP 200 OK response
- [ ] Confirmed "Welcome to nginx" page served
- [ ] Port 6400 listening on host
- [ ] All requirements met ✅

---

## Completion Details

- **Completion Date:** December 18, 2025
- **Day:** 43 / 100
- **Challenge:** KodeKloud 100 Days Cloud DevOps
- **Topic:** Nginx Container with Port Mapping
- **Server:** Application Server 3 (stapp03)
- **User:** banner
- **Image:** nginx:stable (~143MB)
- **Container Name:** beta
- **Port Mapping:** 6400 (host) → 80 (container)
- **State:** Running ✅
- **Access URL:** http://stapp03:6400 or http://172.16.238.12:6400
- **Key Skill:** Docker port mapping and container networking
- **Status:** ✅ Successfully Completed

---

## Summary

This task demonstrated **Docker port mapping for exposing containerized services**:

✅ **Pulled nginx:stable** - Production-ready web server image
✅ **Created named container** - "beta" for easy reference
✅ **Mapped ports** - 6400 (host) → 80 (container)
✅ **Running state** - Container up and serving requests
✅ **Verified accessibility** - HTTP 200 OK on port 6400

**Key Insight:** Port mapping is the **bridge between container isolation and external accessibility**:
- **Containers are isolated** - Each has its own network namespace
- **Services need exposure** - Port mapping makes them accessible
- **No port conflicts** - Multiple containers can use port 80 internally
- **Flexible deployment** - Same image, different host ports

**The Port Mapping Magic:**
```
Without Port Mapping:
Container (nginx:80) → ❌ Isolated, not accessible

With Port Mapping (-p 6400:80):
External Client → Host:6400 → Container:80 → Nginx → ✅ Success!
```

**Port Mapping Syntax:**
```bash
-p 6400:80
   ↓    ↓
   │    └─ Container Port (where nginx listens inside)
   └─ Host Port (external access point)
```

**Multiple Containers, No Conflicts:**
```bash
docker run -d -p 8001:80 nginx  # Container 1
docker run -d -p 8002:80 nginx  # Container 2
docker run -d -p 8003:80 nginx  # Container 3

# All use port 80 internally
# No conflicts - isolated networks!
# Access via 8001, 8002, 8003 externally
```

**Why This Matters in Production:**
- **Microservices** - Each service different port
- **Load Balancing** - Multiple backend instances
- **Zero Downtime** - Blue-green deployments
- **Development** - Local testing with production images
- **Security** - Expose only necessary ports

**Common Use Cases:**
```
Web Applications:
└── docker run -p 80:80 webapp
    External: http://host:80

Development:
└── docker run -p 3000:3000 node-app
    Local: http://localhost:3000

Database (careful!):
└── docker run -p 127.0.0.1:3306:3306 mysql
    Only localhost access for security

Multiple Services:
├── docker run -p 8001:80 service1
├── docker run -p 8002:80 service2
└── docker run -p 8003:80 service3
    Microservices architecture
```

**Best Practices Applied:**
- ✅ Used stable image tag (nginx:stable)
- ✅ Named container for clarity (beta)
- ✅ Non-standard port to avoid conflicts (6400)
- ✅ Detached mode for background operation
- ✅ Verified all functionality before completion

**The 0.0.0.0 Binding:**
```
PORTS: 0.0.0.0:6400->80/tcp

0.0.0.0 means:
├── localhost:6400 ✅
├── 127.0.0.1:6400 ✅
├── 172.16.238.12:6400 ✅
└── any_ip:6400 ✅

Accessible from everywhere!
```

**Port Mapping vs Expose:**
```dockerfile
# In Dockerfile
EXPOSE 80
# Documentation only, doesn't publish port

# At runtime
docker run -p 6400:80 nginx
# Actually maps and publishes port
```

**Remember:** `-p HOST:CONTAINER` - The HOST port is what external clients use, the CONTAINER port is what the service listens on inside! 🐳

**Real-World Impact:**
- Deploy nginx in **5 seconds** vs minutes of manual setup
- **Zero configuration** inside container - just map ports
- **Perfect isolation** - No dependency conflicts
- **Easy scaling** - Add more containers with different ports

**Next:** Combine port mapping with custom networks, volumes, and Docker Compose for complete application stacks! 🚀
