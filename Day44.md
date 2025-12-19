# Day 44: Docker Compose - httpd Web Server with Volumes
**100 Days Cloud DevOps Challenge – KodeKloud**

---

## Objective

Deploy an Apache httpd web server using Docker Compose with volume mounting for static website content and custom port mapping.

**Requirements:**
1. Work on Application Server 2 (App Server 2)
2. Create Docker Compose file: `/opt/docker/docker-compose.yml`
3. Image: `httpd:latest`
4. Container Name: `httpd`
5. Port Mapping: Host `3001` → Container `80`
6. Volume Mapping: Host `/opt/devops` → Container `/usr/local/apache2/htdocs`

---

## Understanding Docker Compose

**Docker Compose** is a tool for defining and running multi-container Docker applications using a YAML configuration file. It simplifies container orchestration and management.

### What is Docker Compose?

Docker Compose allows you to:
- Define services in YAML format
- Manage multiple containers as one application
- Configure networks, volumes, and dependencies
- Start/stop entire application with single command
- Version control infrastructure configuration

### Docker Run vs Docker Compose:

```
Docker Run (Manual):
docker run -d --name httpd -p 3001:80 -v /opt/devops:/usr/local/apache2/htdocs httpd:latest
↓ Long command, error-prone, hard to reproduce

Docker Compose (Automated):
docker-compose up -d
↓ One command, reads docker-compose.yml, reproducible
```

| Aspect | docker run | docker-compose |
|--------|-----------|----------------|
| **Configuration** | Command-line flags | YAML file |
| **Reproducibility** | Manual, error-prone | Automated, reliable |
| **Multi-container** | Multiple commands | Single file |
| **Version Control** | Difficult | Easy (YAML in git) |
| **Complexity** | Simple for one container | Better for multiple |
| **Best For** | Quick testing | Production, teams |

---

## Understanding Docker Compose Architecture

### Docker Compose Workflow:

```
docker-compose.yml (Configuration)
         ↓
    docker-compose up
         ↓
    Reads YAML file
         ↓
    Creates networks, volumes
         ↓
    Starts containers
         ↓
    Running Application
```

### Compose File Structure:

```yaml
version: '3'              # Compose file format version

services:                 # Container definitions
  service-name:           # Custom service name
    image: httpd:latest   # Docker image
    container_name: httpd # Container name
    ports:                # Port mappings
      - "3001:80"
    volumes:              # Volume mounts
      - /opt/devops:/usr/local/apache2/htdocs
```

### Components:

**1. Version**
- Specifies compose file format
- Version 3+ recommended
- Determines available features

**2. Services**
- Each service = one container
- Define image, ports, volumes, etc.
- Can have multiple services

**3. Networks** (optional)
- Custom network configuration
- Service discovery
- Isolation

**4. Volumes** (optional)
- Named volume definitions
- Persistent data storage

---

## Understanding httpd Web Server

### Apache httpd Container:

**httpd** is the official Apache HTTP Server Docker image. It's lightweight, production-ready, and widely used for serving static and dynamic web content.

### httpd Image Details:

| Tag | Description | Size | Use Case |
|-----|-------------|------|----------|
| **httpd:latest** | Latest stable release | ~145MB | **Production (our task)** |
| httpd:2.4 | Specific version | ~145MB | Version pinning |
| httpd:alpine | Alpine-based | ~55MB | Size-optimized |
| httpd:2.4-alpine | Version + Alpine | ~55MB | Production + small |

### httpd Directory Structure:

```
httpd Container
│
├── /usr/local/apache2/
│   ├── conf/               # Configuration files
│   │   └── httpd.conf      # Main config
│   ├── htdocs/             # Website root (our mount point)
│   │   └── index.html      # Default page
│   ├── logs/               # Access/error logs
│   └── bin/                # Apache binaries
```

### Why httpd?

- ✅ Official Apache image
- ✅ Production-ready
- ✅ Well-documented
- ✅ Active maintenance
- ✅ Secure defaults

---

## Understanding Volume Mounting

**Volume Mounting** connects host directories to container directories, allowing containers to access host files and persist data beyond container lifecycle.

### Volume Mount Flow:

```
Host Machine
├── /opt/devops/
│   ├── index.html
│   └── style.css
         ↓
    Volume Mount
         ↓
Container
├── /usr/local/apache2/htdocs/
│   ├── index.html (from host)
│   └── style.css (from host)
         ↓
    Apache serves files
         ↓
    Web Browser
```

### Volume Mount Syntax:

```yaml
volumes:
  - /opt/devops:/usr/local/apache2/htdocs
    ↓           ↓
    │           └─ Container path (where Apache looks)
    └─ Host path (where files actually are)
```

### Volume Types:

**1. Bind Mounts (Our Task):**
```yaml
volumes:
  - /opt/devops:/usr/local/apache2/htdocs
# Direct host directory mount
```

**2. Named Volumes:**
```yaml
volumes:
  - web-data:/usr/local/apache2/htdocs
# Docker-managed volume
```

**3. Anonymous Volumes:**
```yaml
volumes:
  - /usr/local/apache2/htdocs
# Temporary, removed with container
```

### Why Volume Mounting?

- **Persistent Data** - Survives container restarts
- **Easy Updates** - Edit files on host
- **Development** - Live code changes
- **Backup** - Host files are backed up
- **Sharing** - Multiple containers access same data

---

## Understanding the Scenario

### Development Team Requirements:

```
1. Static website content ready in /opt/devops
2. Need httpd web server to host content
3. Must use Docker Compose for management
4. Custom port 3001 to avoid conflicts
5. Content must be editable on host
6. Container orchestration for easy deployment
```

### Current State:

```
App Server 2
│
├── /opt/devops/
│   ├── index.html (static website)
│   ├── css/
│   ├── js/
│   └── images/
│
└── Need: httpd container to serve these files
```

### Target State:

```
App Server 2
│
├── /opt/devops/ (website content)
│   └── Mounted to container
│
├── /opt/docker/docker-compose.yml (config)
│
└── httpd container
    ├── Port 3001 exposed
    ├── Serving /opt/devops content
    └── Managed by Docker Compose
```

---

## Infrastructure Overview

### Application Servers:
| Server | User | Password | IP |
|--------|------|----------|-----|
| stapp01 | tony | Ir0nM@n | 172.16.238.10 |
| **stapp02** | **steve** | **Am3ric@** | **172.16.238.11** |
| stapp03 | banner | BigGr33n | 172.16.238.12 |

### Task Details:
| Item | Value |
|------|-------|
| Server | Application Server 2 (stapp02) |
| User | steve |
| Compose File | `/opt/docker/docker-compose.yml` |
| Image | `httpd:latest` |
| Container Name | `httpd` |
| Host Port | 3001 |
| Container Port | 80 |
| Host Volume | `/opt/devops` |
| Container Volume | `/usr/local/apache2/htdocs` |

---

## Understanding the Task

### What We're Creating:

```
Application Server 2 (stapp02)
│
├── /opt/docker/
│   └── docker-compose.yml
│       ├── version: '3'
│       ├── services:
│       │   └── web:
│       │       ├── image: httpd:latest
│       │       ├── container_name: httpd
│       │       ├── ports: 3001:80
│       │       └── volumes: /opt/devops:/usr/local/apache2/htdocs
│
├── /opt/devops/
│   └── Website content (already exists)
│
└── docker-compose up -d
    ↓
    httpd container serving content on port 3001
```

### Deployment Flow:

```
1. SSH to App Server 2
2. Switch to root
3. Create /opt/docker directory
4. Create docker-compose.yml
5. Define httpd service
6. Configure ports and volumes
7. Run docker-compose up -d
8. Verify container running
9. Test website accessibility
```

---

## Step-by-Step Implementation

### Step 1: SSH into Application Server 2
```bash
ssh steve@stapp02
```

**Expected output:**
```
The authenticity of host 'stapp02 (172.16.238.11)' can't be established.
ED25519 key fingerprint is SHA256:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'stapp02' (ED25519) to the list of known hosts.
steve@stapp02's password:
```

**Enter password:** `Am3ric@`

```
[steve@stapp02 ~]$
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

[sudo] password for steve:
```

**Enter password:** `Am3ric@`

```
[root@stapp02 ~]#
```

### Step 3: Verify /opt/devops Directory Exists
```bash
ls -la /opt/devops/
```

**Expected output:**
```
total 16
drwxr-xr-x 2 root root 4096 Dec 19 10:00 .
drwxr-xr-x 4 root root 4096 Dec 19 10:00 ..
-rw-r--r-- 1 root root  612 Dec 19 10:00 index.html
```

**Or similar content - website files already present**

**View content:**
```bash
cat /opt/devops/index.html
```

**Sample output:**
```html
<!DOCTYPE html>
<html>
<head>
    <title>Welcome</title>
</head>
<body>
    <h1>Welcome to Nautilus DevOps</h1>
    <p>Static website content</p>
</body>
</html>
```

**✅ Content directory exists with website files**

### Step 4: Create /opt/docker Directory
```bash
mkdir -p /opt/docker
```

**Verify directory created:**
```bash
ls -la /opt/
```

**Expected output:**
```
total 16
drwxr-xr-x  4 root root 4096 Dec 19 10:05 .
dr-xr-xr-x 18 root root 4096 Dec 19 08:00 ..
drwxr-xr-x  2 root root 4096 Dec 19 10:00 devops
drwxr-xr-x  2 root root 4096 Dec 19 10:05 docker
```

### Step 5: Navigate to /opt/docker
```bash
cd /opt/docker
```

**Verify current location:**
```bash
pwd
```

**Expected output:**
```
/opt/docker
```

### Step 6: Create docker-compose.yml File
```bash
cat > docker-compose.yml << 'EOF'
version: '3'

services:
  web:
    image: httpd:latest
    container_name: httpd
    ports:
      - "3001:80"
    volumes:
      - /opt/devops:/usr/local/apache2/htdocs
EOF
```

**Expected output:**
```
(No output means success)
```

**Alternative method using vi/nano:**
```bash
vi docker-compose.yml
```

**Then paste the content:**
```yaml
version: '3'

services:
  web:
    image: httpd:latest
    container_name: httpd
    ports:
      - "3001:80"
    volumes:
      - /opt/devops:/usr/local/apache2/htdocs
```

**Save and exit (in vi: ESC, then :wq)**

### Step 7: Verify docker-compose.yml Created
```bash
ls -la
```

**Expected output:**
```
total 12
drwxr-xr-x 2 root root 4096 Dec 19 10:10 .
drwxr-xr-x 4 root root 4096 Dec 19 10:05 ..
-rw-r--r-- 1 root root  182 Dec 19 10:10 docker-compose.yml
```

**✅ docker-compose.yml exists**

### Step 8: View docker-compose.yml Contents
```bash
cat docker-compose.yml
```

**Expected output:**
```yaml
version: '3'

services:
  web:
    image: httpd:latest
    container_name: httpd
    ports:
      - "3001:80"
    volumes:
      - /opt/devops:/usr/local/apache2/htdocs
```

**Verify:**
- ✅ version: '3'
- ✅ service name: web (any name is fine)
- ✅ image: httpd:latest
- ✅ container_name: httpd
- ✅ ports: "3001:80"
- ✅ volumes: /opt/devops:/usr/local/apache2/htdocs

### Step 9: Validate docker-compose.yml Syntax
```bash
docker-compose config
```

**Expected output:**
```yaml
services:
  web:
    container_name: httpd
    image: httpd:latest
    ports:
    - published: 3001
      target: 80
    volumes:
    - /opt/devops:/usr/local/apache2/htdocs:rw
version: '3'
```

**✅ No errors means syntax is valid**

**If errors appear, fix YAML indentation (use spaces, not tabs)**

### Step 10: Check Current Docker Containers
```bash
docker ps -a
```

**Expected output:**
```
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

**Or may show existing containers**

### Step 11: Pull httpd Image (Optional)
```bash
docker pull httpd:latest
```

**Expected output:**
```
latest: Pulling from library/httpd
a803e7c4b030: Pull complete
1234567890ab: Pull complete
abcdef123456: Pull complete
7890abcdef12: Pull complete
3456789012ab: Pull complete
Digest: sha256:abc123def456...
Status: Downloaded newer image for httpd:latest
docker.io/library/httpd:latest
```

**Note:** `docker-compose up` will pull automatically if not present

### Step 12: Start Container with Docker Compose
```bash
docker-compose up -d
```

**Expected output:**
```
Creating network "docker_default" with the default driver
Pulling web (httpd:latest)...
latest: Pulling from library/httpd
a803e7c4b030: Pull complete
1234567890ab: Pull complete
abcdef123456: Pull complete
7890abcdef12: Pull complete
3456789012ab: Pull complete
Digest: sha256:abc123def456...
Status: Downloaded newer image for httpd:latest
Creating httpd ... done
```

**Command breakdown:**
- `docker-compose up` - Start services defined in docker-compose.yml
- `-d` - Detached mode (background)

**If image already pulled:**
```
Creating network "docker_default" with the default driver
Creating httpd ... done
```

### Step 13: Verify Container Running
```bash
docker ps
```

**Expected output:**
```
CONTAINER ID   IMAGE          COMMAND              CREATED          STATUS          PORTS                                       NAMES
a1b2c3d4e5f6   httpd:latest   "httpd-foreground"   15 seconds ago   Up 14 seconds   0.0.0.0:3001->80/tcp, :::3001->80/tcp      httpd
```

**Verify:**
- ✅ IMAGE: httpd:latest
- ✅ COMMAND: httpd-foreground
- ✅ STATUS: Up
- ✅ PORTS: 0.0.0.0:3001->80/tcp
- ✅ NAMES: httpd

**Filter for httpd container:**
```bash
docker ps --filter "name=httpd"
```

### Step 14: Check Docker Compose Services
```bash
docker-compose ps
```

**Expected output:**
```
Name               Command              State                    Ports                  
---------------------------------------------------------------------------------------
httpd   httpd-foreground         Up      0.0.0.0:3001->80/tcp,:::3001->80/tcp
```

**Shows services managed by current docker-compose.yml**

### Step 15: Inspect Container Details
```bash
docker inspect httpd
```

**Expected output (partial):**
```json
[
    {
        "Id": "a1b2c3d4e5f6...",
        "Name": "/httpd",
        "State": {
            "Status": "running",
            "Running": true
        },
        "Image": "httpd:latest",
        "Mounts": [
            {
                "Type": "bind",
                "Source": "/opt/devops",
                "Destination": "/usr/local/apache2/htdocs",
                "Mode": "",
                "RW": true
            }
        ],
        "NetworkSettings": {
            "Ports": {
                "80/tcp": [
                    {
                        "HostIp": "0.0.0.0",
                        "HostPort": "3001"
                    }
                ]
            }
        }
    }
]
```

**Verify:**
- ✅ Status: "running"
- ✅ Source: "/opt/devops"
- ✅ Destination: "/usr/local/apache2/htdocs"
- ✅ HostPort: "3001"

### Step 16: Check Volume Mount
```bash
docker exec httpd ls -la /usr/local/apache2/htdocs/
```

**Expected output:**
```
total 16
drwxr-xr-x 2 root root 4096 Dec 19 10:00 .
drwxr-xr-x 1 root root 4096 Dec 19 10:15 ..
-rw-r--r-- 1 root root  612 Dec 19 10:00 index.html
```

**✅ Host files visible inside container!**

**View content inside container:**
```bash
docker exec httpd cat /usr/local/apache2/htdocs/index.html
```

**Should show same content as host file**

### Step 17: Check Container Logs
```bash
docker-compose logs
```

**Expected output:**
```
Attaching to httpd
httpd    | AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 172.18.0.2. Set the 'ServerName' directive globally to suppress this message
httpd    | AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 172.18.0.2. Set the 'ServerName' directive globally to suppress this message
httpd    | [Thu Dec 19 10:15:30.123456 2025] [mpm_event:notice] [pid 1:tid 140123456789] AH00489: Apache/2.4.58 (Unix) configured -- resuming normal operations
httpd    | [Thu Dec 19 10:15:30.123457 2025] [core:notice] [pid 1:tid 140123456789] AH00094: Command line: 'httpd -D FOREGROUND'
```

**Alternative:**
```bash
docker logs httpd
```

### Step 18: Verify Port Mapping
```bash
docker port httpd
```

**Expected output:**
```
80/tcp -> 0.0.0.0:3001
80/tcp -> :::3001
```

**✅ Port 3001 mapped to container port 80**

**Check port listening on host:**
```bash
netstat -tuln | grep 3001
```

**Expected output:**
```
tcp        0      0 0.0.0.0:3001            0.0.0.0:*               LISTEN
tcp6       0      0 :::3001                 :::*                    LISTEN
```

### Step 19: Test Website from Host
```bash
curl http://localhost:3001
```

**Expected output:**
```html
<!DOCTYPE html>
<html>
<head>
    <title>Welcome</title>
</head>
<body>
    <h1>Welcome to Nautilus DevOps</h1>
    <p>Static website content</p>
</body>
</html>
```

**✅ Website served successfully!**

**Test with HTTP headers:**
```bash
curl -I http://localhost:3001
```

**Expected output:**
```
HTTP/1.1 200 OK
Date: Thu, 19 Dec 2025 10:20:00 GMT
Server: Apache/2.4.58 (Unix)
Last-Modified: Thu, 19 Dec 2025 10:00:00 GMT
ETag: "264-6281e3b4a8e80"
Accept-Ranges: bytes
Content-Length: 612
Content-Type: text/html
```

### Step 20: Test Volume Persistence
```bash
# Create new file on host
echo "<h2>Test content</h2>" > /opt/devops/test.html

# Verify file exists on host
ls -la /opt/devops/test.html

# Check if visible in container
docker exec httpd ls -la /usr/local/apache2/htdocs/test.html

# Access via web server
curl http://localhost:3001/test.html
```

**Expected output:**
```
-rw-r--r-- 1 root root 22 Dec 19 10:25 /opt/devops/test.html
-rw-r--r-- 1 root root 22 Dec 19 10:25 /usr/local/apache2/htdocs/test.html
<h2>Test content</h2>
```

**✅ Volume mount working - changes on host reflect in container!**

### Step 21: Check Docker Compose Network
```bash
docker network ls | grep docker
```

**Expected output:**
```
a1b2c3d4e5f6   docker_default   bridge    local
```

**Docker Compose creates default network automatically**

### Step 22: View Complete Service Status
```bash
docker-compose ps -a
```

**Expected output:**
```
Name               Command              State                    Ports                  
---------------------------------------------------------------------------------------
httpd   httpd-foreground         Up      0.0.0.0:3001->80/tcp,:::3001->80/tcp
```

### Step 23: Final Verification
```bash
# 1. Compose file exists
ls -la /opt/docker/docker-compose.yml

# 2. Container named httpd running
docker ps --filter "name=httpd" --format "{{.Names}}"

# 3. Using httpd:latest image
docker ps --filter "name=httpd" --format "{{.Image}}"

# 4. Port 3001 mapped
docker port httpd | grep 3001

# 5. Volume mounted correctly
docker inspect httpd | grep -A 5 "Mounts"

# 6. Website accessible
curl -s http://localhost:3001 | grep -i "welcome"
```

**All checks should pass:**
- ✅ /opt/docker/docker-compose.yml exists
- ✅ httpd container running
- ✅ httpd:latest image used
- ✅ Port 3001:80 mapped
- ✅ Volume /opt/devops mounted
- ✅ Website serving content

---

## Complete Command Summary

### Quick Deployment:
```bash
# SSH and access
ssh steve@stapp02
sudo su -

# Create compose file
mkdir -p /opt/docker
cd /opt/docker

cat > docker-compose.yml << 'EOF'
version: '3'

services:
  web:
    image: httpd:latest
    container_name: httpd
    ports:
      - "3001:80"
    volumes:
      - /opt/devops:/usr/local/apache2/htdocs
EOF

# Start container
docker-compose up -d

# Verify
docker ps
curl http://localhost:3001
```

### Detailed Workflow:
```bash
# 1. SSH and authenticate
ssh steve@stapp02
sudo su -

# 2. Verify content directory
ls -la /opt/devops/
cat /opt/devops/index.html

# 3. Create docker directory
mkdir -p /opt/docker
cd /opt/docker

# 4. Create docker-compose.yml
cat > docker-compose.yml << 'EOF'
version: '3'

services:
  web:
    image: httpd:latest
    container_name: httpd
    ports:
      - "3001:80"
    volumes:
      - /opt/devops:/usr/local/apache2/htdocs
EOF

# 5. Validate syntax
docker-compose config

# 6. Start services
docker-compose up -d

# 7. Verify container
docker ps
docker-compose ps

# 8. Check logs
docker-compose logs

# 9. Verify volume mount
docker exec httpd ls -la /usr/local/apache2/htdocs/

# 10. Test website
curl http://localhost:3001
curl -I http://localhost:3001

# 11. Check port mapping
docker port httpd
netstat -tuln | grep 3001
```

---

## Understanding Docker Compose Commands

### Essential Commands:

**1. Start Services:**
```bash
docker-compose up           # Foreground (shows logs)
docker-compose up -d        # Background (detached)
docker-compose up --build   # Rebuild images first
```

**2. Stop Services:**
```bash
docker-compose stop         # Stop containers (preserves them)
docker-compose down         # Stop and remove containers
docker-compose down -v      # Also remove volumes
```

**3. View Status:**
```bash
docker-compose ps           # List containers
docker-compose ps -a        # Include stopped containers
docker-compose logs         # View logs
docker-compose logs -f      # Follow logs (live)
docker-compose logs web     # Logs for specific service
```

**4. Manage Services:**
```bash
docker-compose start        # Start stopped services
docker-compose restart      # Restart services
docker-compose pause        # Pause services
docker-compose unpause      # Unpause services
```

**5. Execute Commands:**
```bash
docker-compose exec web sh              # Interactive shell
docker-compose exec web ls -la /app     # Run command
docker-compose run web /bin/bash        # Run one-off command
```

**6. Configuration:**
```bash
docker-compose config       # Validate and view config
docker-compose config -q    # Validate only (quiet)
docker-compose version      # Show version
```

---

## Docker Compose File Deep Dive

### Complete docker-compose.yml Structure:

```yaml
version: '3'                          # Compose file version

services:                             # Container definitions
  web:                                # Service name (custom)
    image: httpd:latest               # Docker image
    container_name: httpd             # Container name
    restart: always                   # Restart policy
    ports:                            # Port mappings
      - "3001:80"                     # host:container
    volumes:                          # Volume mounts
      - /opt/devops:/usr/local/apache2/htdocs
    environment:                      # Environment variables
      - ENV_VAR=value
    networks:                         # Network connections
      - frontend
    depends_on:                       # Service dependencies
      - database

networks:                             # Network definitions
  frontend:
    driver: bridge

volumes:                              # Named volume definitions
  data:
    driver: local
```

### Our Minimal Configuration:

```yaml
version: '3'

services:
  web:                                # Service name (can be anything)
    image: httpd:latest               # ✅ Required: httpd:latest
    container_name: httpd             # ✅ Required: httpd
    ports:
      - "3001:80"                     # ✅ Required: 3001:80
    volumes:
      - /opt/devops:/usr/local/apache2/htdocs  # ✅ Required: mount
```

### Key Points:

**Service Name (web):**
- Can be any name (web, app, server, etc.)
- Used in docker-compose commands
- Creates network aliases

**Container Name (httpd):**
- Must be exactly "httpd" per requirements
- Used in docker commands
- Must be unique on host

**Image:**
- httpd:latest for latest stable
- Could use httpd:2.4 for specific version
- Docker pulls if not present

**Ports:**
- Format: "HOST:CONTAINER"
- Can have multiple port mappings
- Quoted to avoid YAML parsing issues

**Volumes:**
- Format: HOST_PATH:CONTAINER_PATH
- Bind mount (direct host directory)
- Changes on host reflect in container instantly

---

## Advanced Docker Compose Patterns

### Multi-Container Application:

```yaml
version: '3'

services:
  web:
    image: httpd:latest
    container_name: httpd
    ports:
      - "3001:80"
    volumes:
      - /opt/devops:/usr/local/apache2/htdocs
    depends_on:
      - db
    networks:
      - frontend
      - backend

  db:
    image: mysql:8.0
    container_name: mysql
    environment:
      MYSQL_ROOT_PASSWORD: secret
      MYSQL_DATABASE: myapp
    volumes:
      - db-data:/var/lib/mysql
    networks:
      - backend

networks:
  frontend:
  backend:

volumes:
  db-data:
```

### Environment Variables:

```yaml
version: '3'

services:
  web:
    image: httpd:latest
    container_name: httpd
    environment:
      - APACHE_LOG_LEVEL=debug
      - TZ=America/New_York
    env_file:
      - .env
    ports:
      - "3001:80"
    volumes:
      - /opt/devops:/usr/local/apache2/htdocs
```

### Health Checks:

```yaml
version: '3'

services:
  web:
    image: httpd:latest
    container_name: httpd
    ports:
      - "3001:80"
    volumes:
      - /opt/devops:/usr/local/apache2/htdocs
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:80"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
```

### Restart Policies:

```yaml
version: '3'

services:
  web:
    image: httpd:latest
    container_name: httpd
    restart: always          # always, unless-stopped, on-failure, no
    ports:
      - "3001:80"
    volumes:
      - /opt/devops:/usr/local/apache2/htdocs
```

### Resource Limits:

```yaml
version: '3'

services:
  web:
    image: httpd:latest
    container_name: httpd
    ports:
      - "3001:80"
    volumes:
      - /opt/devops:/usr/local/apache2/htdocs
    deploy:
      resources:
        limits:
          cpus: '0.50'
          memory: 512M
        reservations:
          cpus: '0.25'
          memory: 256M
```

---

## Troubleshooting

### Issue 1: Syntax Error in docker-compose.yml

**Problem:**
```
ERROR: yaml.scanner.ScannerError: mapping values are not allowed here
```

**Solution:**
```bash
# Common issues:
# 1. Tabs instead of spaces (use spaces only)
# 2. Incorrect indentation
# 3. Missing colons

# Validate syntax
docker-compose config

# Fix indentation (2 spaces per level)
version: '3'

services:
  web:                    # 2 spaces
    image: httpd:latest   # 4 spaces
    ports:                # 4 spaces
      - "3001:80"         # 6 spaces (dash counts as part of indent)
```

### Issue 2: Port Already in Use

**Problem:**
```
ERROR: for httpd  Cannot start service web: driver failed programming external connectivity: Bind for 0.0.0.0:3001 failed: port is already allocated
```

**Solution:**
```bash
# Check what's using port 3001
netstat -tuln | grep 3001
lsof -i :3001

# Stop conflicting service
docker ps | grep 3001
docker stop container_name

# Or use different port
# Edit docker-compose.yml
ports:
  - "3002:80"
```

### Issue 3: Container Name Already Exists

**Problem:**
```
ERROR: for httpd  Conflict. The container name "/httpd" is already in use
```

**Solution:**
```bash
# Remove existing container
docker ps -a | grep httpd
docker rm -f httpd

# Or change container name in docker-compose.yml
container_name: httpd2

# Then start again
docker-compose up -d
```

### Issue 4: Volume Mount Permission Issues

**Problem:**
```
Permission denied when accessing files
```

**Solution:**
```bash
# Check host directory permissions
ls -la /opt/devops/

# Ensure readable
chmod -R 755 /opt/devops/

# Check ownership
chown -R root:root /opt/devops/

# Restart container
docker-compose restart
```

### Issue 5: docker-compose Command Not Found

**Problem:**
```
bash: docker-compose: command not found
```

**Solution:**
```bash
# Install docker-compose
sudo curl -L "https://github.com/docker/compose/releases/download/v2.23.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose

# Verify
docker-compose --version

# Alternative: use docker compose (v2 plugin)
docker compose version
```

### Issue 6: Cannot Access Website

**Problem:**
```
curl: (7) Failed to connect to localhost port 3001: Connection refused
```

**Solution:**
```bash
# Check container running
docker ps | grep httpd

# Check logs
docker-compose logs

# Verify port mapping
docker port httpd

# Check httpd is listening inside container
docker exec httpd netstat -tuln | grep 80

# Test from inside container
docker exec httpd curl http://localhost:80

# Check firewall
sudo firewall-cmd --list-ports
sudo firewall-cmd --add-port=3001/tcp --permanent
sudo firewall-cmd --reload
```

### Issue 7: Volume Mount Not Working

**Problem:**
Host files not visible in container

**Solution:**
```bash
# Verify host directory exists
ls -la /opt/devops/

# Check mount inside container
docker exec httpd ls -la /usr/local/apache2/htdocs/

# Verify mount in inspect
docker inspect httpd | grep -A 10 "Mounts"

# Restart container
docker-compose down
docker-compose up -d

# Check if SELinux is blocking (CentOS/RHEL)
getenforce
# If Enforcing, add :z to volume
volumes:
  - /opt/devops:/usr/local/apache2/htdocs:z
```

---

## Best Practices

### 1. Use Version Control for docker-compose.yml
```bash
# Initialize git repository
cd /opt/docker
git init
git add docker-compose.yml
git commit -m "Add httpd docker-compose configuration"
```

### 2. Use Environment-Specific Compose Files
```bash
# docker-compose.yml (base)
# docker-compose.dev.yml (development overrides)
# docker-compose.prod.yml (production overrides)

# Run with override
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up -d
```

### 3. Add Restart Policy
```yaml
version: '3'

services:
  web:
    image: httpd:latest
    container_name: httpd
    restart: unless-stopped    # Auto-restart on failure
    ports:
      - "3001:80"
    volumes:
      - /opt/devops:/usr/local/apache2/htdocs
```

### 4. Use Named Volumes for Persistence
```yaml
version: '3'

services:
  web:
    image: httpd:latest
    container_name: httpd
    ports:
      - "3001:80"
    volumes:
      - /opt/devops:/usr/local/apache2/htdocs
      - httpd-logs:/usr/local/apache2/logs

volumes:
  httpd-logs:
```

### 5. Document Configuration
```yaml
version: '3'

services:
  web:
    # Apache httpd web server for static content
    # Serves content from /opt/devops on host port 3001
    image: httpd:latest
    container_name: httpd
    ports:
      - "3001:80"    # External:Internal
    volumes:
      - /opt/devops:/usr/local/apache2/htdocs    # Static content mount
```

### 6. Use .env Files
```bash
# Create .env file
cat > .env << 'EOF'
HOST_PORT=3001
CONTAINER_PORT=80
CONTENT_PATH=/opt/devops
EOF

# Reference in docker-compose.yml
version: '3'

services:
  web:
    image: httpd:latest
    container_name: httpd
    ports:
      - "${HOST_PORT}:${CONTAINER_PORT}"
    volumes:
      - "${CONTENT_PATH}:/usr/local/apache2/htdocs"
```

### 7. Add Health Checks
```yaml
version: '3'

services:
  web:
    image: httpd:latest
    container_name: httpd
    ports:
      - "3001:80"
    volumes:
      - /opt/devops:/usr/local/apache2/htdocs
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:80"]
      interval: 30s
      timeout: 3s
      retries: 3
```

---

## Real-World Scenarios

### Scenario 1: Development Environment
```yaml
version: '3'

services:
  web:
    image: httpd:latest
    container_name: dev-httpd
    ports:
      - "8080:80"
    volumes:
      - ./app:/usr/local/apache2/htdocs
      - ./config/httpd.conf:/usr/local/apache2/conf/httpd.conf
    environment:
      - APACHE_LOG_LEVEL=debug
```

### Scenario 2: Multi-Site Hosting
```yaml
version: '3'

services:
  site1:
    image: httpd:latest
    container_name: site1
    ports:
      - "8001:80"
    volumes:
      - /opt/sites/site1:/usr/local/apache2/htdocs

  site2:
    image: httpd:latest
    container_name: site2
    ports:
      - "8002:80"
    volumes:
      - /opt/sites/site2:/usr/local/apache2/htdocs

  site3:
    image: httpd:latest
    container_name: site3
    ports:
      - "8003:80"
    volumes:
      - /opt/sites/site3:/usr/local/apache2/htdocs
```

### Scenario 3: Production with SSL
```yaml
version: '3'

services:
  web:
    image: httpd:latest
    container_name: httpd-prod
    restart: always
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - /opt/websites/prod:/usr/local/apache2/htdocs
      - /etc/letsencrypt:/usr/local/apache2/conf/ssl
      - httpd-logs:/usr/local/apache2/logs
    environment:
      - TZ=UTC

volumes:
  httpd-logs:
```

### Scenario 4: Microservices with API Gateway
```yaml
version: '3'

services:
  gateway:
    image: httpd:latest
    container_name: api-gateway
    ports:
      - "80:80"
    volumes:
      - ./gateway-config:/usr/local/apache2/conf
    networks:
      - frontend
      - backend

  service1:
    image: myapp:service1
    container_name: service1
    networks:
      - backend

  service2:
    image: myapp:service2
    container_name: service2
    networks:
      - backend

networks:
  frontend:
  backend:
    internal: true
```

### Scenario 5: CI/CD Pipeline
```yaml
version: '3'

services:
  web:
    image: httpd:${VERSION:-latest}
    container_name: httpd-${ENVIRONMENT:-dev}
    restart: unless-stopped
    ports:
      - "${PORT:-3001}:80"
    volumes:
      - ${CONTENT_PATH:-./build}:/usr/local/apache2/htdocs
    labels:
      - "com.example.environment=${ENVIRONMENT}"
      - "com.example.version=${VERSION}"
```

**Deploy with:**
```bash
VERSION=v1.2.3 ENVIRONMENT=staging PORT=8080 docker-compose up -d
```

---

## Key Commands Reference

| Command | Description |
|---------|-------------|
| `docker-compose up -d` | Start services in background |
| `docker-compose down` | Stop and remove containers |
| `docker-compose ps` | List containers |
| `docker-compose logs` | View logs |
| `docker-compose logs -f` | Follow logs |
| `docker-compose restart` | Restart services |
| `docker-compose stop` | Stop services |
| `docker-compose start` | Start stopped services |
| `docker-compose exec SERVICE CMD` | Execute command in service |
| `docker-compose config` | Validate and view config |
| `docker-compose pull` | Pull service images |
| `docker-compose build` | Build services |

---

## docker-compose.yml Reference

### Minimal Configuration:
```yaml
version: '3'

services:
  web:
    image: httpd:latest
    container_name: httpd
    ports:
      - "3001:80"
    volumes:
      - /opt/devops:/usr/local/apache2/htdocs
```

### Complete Configuration:
```yaml
version: '3'

services:
  web:
    image: httpd:latest
    container_name: httpd
    restart: unless-stopped
    ports:
      - "3001:80"
      - "3443:443"
    volumes:
      - /opt/devops:/usr/local/apache2/htdocs:ro
      - logs:/usr/local/apache2/logs
    environment:
      - APACHE_LOG_LEVEL=warn
      - TZ=UTC
    networks:
      - webnet
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost"]
      interval: 30s
      timeout: 3s
      retries: 3
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 512M

networks:
  webnet:
    driver: bridge

volumes:
  logs:
    driver: local
```

---

## Completion Checklist

- [ ] SSH into Application Server 2 (stapp02) as steve
- [ ] Switched to root user
- [ ] Verified /opt/devops directory exists with content
- [ ] Created /opt/docker directory
- [ ] Created docker-compose.yml at /opt/docker/docker-compose.yml
- [ ] Used version: '3' in compose file
- [ ] Defined service (can use any name like 'web')
- [ ] Used image: httpd:latest
- [ ] Set container_name: httpd
- [ ] Configured ports: "3001:80"
- [ ] Configured volumes: /opt/devops:/usr/local/apache2/htdocs
- [ ] Validated docker-compose.yml syntax
- [ ] Ran docker-compose up -d
- [ ] Container named httpd is running
- [ ] Port 3001 mapped to container port 80
- [ ] Volume mounted correctly (host files visible in container)
- [ ] Website accessible on port 3001
- [ ] HTTP 200 OK response received
- [ ] All requirements met ✅

---

## Completion Details

- **Completion Date:** December 19, 2025
- **Day:** 44 / 100
- **Challenge:** KodeKloud 100 Days Cloud DevOps
- **Topic:** Docker Compose - httpd Web Server with Volumes
- **Server:** Application Server 2 (stapp02)
- **User:** steve
- **Compose File:** `/opt/docker/docker-compose.yml`
- **Service Name:** web (or custom name)
- **Container Name:** httpd
- **Image:** httpd:latest (~145MB)
- **Port Mapping:** 3001 (host) → 80 (container)
- **Volume Mount:** /opt/devops → /usr/local/apache2/htdocs
- **Access URL:** http://stapp02:3001 or http://172.16.238.11:3001
- **Key Skill:** Docker Compose configuration and volume management
- **Status:** ✅ Successfully Completed

---

## Summary

This task demonstrated **Docker Compose for simplified container orchestration**:

✅ **Created docker-compose.yml** - Infrastructure as code
✅ **Used httpd:latest** - Apache web server
✅ **Named container "httpd"** - Easy identification
✅ **Mapped port 3001:80** - Custom external access
✅ **Mounted volume** - Host content in container
✅ **One-command deployment** - docker-compose up -d

**Key Insight:** Docker Compose transforms **complex docker run commands into simple, version-controlled configuration files**:

**Before (docker run):**
```bash
docker run -d --name httpd -p 3001:80 -v /opt/devops:/usr/local/apache2/htdocs httpd:latest
# Long, error-prone, hard to remember
```

**After (docker-compose):**
```yaml
version: '3'
services:
  web:
    image: httpd:latest
    container_name: httpd
    ports: ["3001:80"]
    volumes: ["/opt/devops:/usr/local/apache2/htdocs"]
```
```bash
docker-compose up -d
# Simple, reproducible, version controlled
```

**The Docker Compose Advantage:**
```
Single Configuration File
├── All container settings in one place
├── Version controlled (git)
├── Easy to share with team
├── Reproducible deployments
├── Simple updates (edit YAML, docker-compose up -d)
└── Multi-container orchestration
```

**Volume Mount Magic:**
```
Host: /opt/devops/index.html
         ↓ (mounted to)
Container: /usr/local/apache2/htdocs/index.html
         ↓
Apache serves file
         ↓
User accesses: http://host:3001/index.html
```

**Why This Matters:**
- **Live Updates** - Edit files on host, immediately served
- **Persistence** - Content survives container restarts
- **Backup** - Host directory is backed up normally
- **Development** - Easy local development workflow
- **Team Collaboration** - Shared content directory

**Service Name vs Container Name:**
```yaml
services:
  web:                    # Service name (Docker Compose uses this)
    container_name: httpd # Container name (Docker uses this)

# Commands:
docker-compose logs web       # Use service name
docker logs httpd             # Use container name
```

**Port Mapping in Compose:**
```yaml
ports:
  - "3001:80"
    ↓      ↓
    │      └─ Container port (where httpd listens)
    └─ Host port (external access)
```

**Best Practices Applied:**
- ✅ Used quotes for port mapping ("3001:80")
- ✅ Proper YAML indentation (2 spaces)
- ✅ Descriptive service names
- ✅ Specific image tags (httpd:latest explicit)
- ✅ Absolute paths for volume mounts

**Common Docker Compose Workflows:**
```bash
# Start
docker-compose up -d

# View logs
docker-compose logs -f

# Stop
docker-compose stop

# Remove
docker-compose down

# Restart after config change
docker-compose up -d --force-recreate

# Execute command
docker-compose exec web sh
```

**Production Enhancements:**
```yaml
# Add restart policy
restart: unless-stopped

# Add health check
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost"]

# Add resource limits
deploy:
  resources:
    limits:
      memory: 512M
```

**Remember:** `docker-compose.yml` = Infrastructure as Code for containers! It's the standard for multi-container applications and production deployments! 🐳

**Real-World Impact:**
- Define once, deploy anywhere
- Team members use same configuration
- CI/CD pipelines use same compose file
- Dev, staging, prod consistency guaranteed

**Next:** Scale to multi-container applications, add databases, implement networking, and build complete application stacks with Docker Compose! 🚀
