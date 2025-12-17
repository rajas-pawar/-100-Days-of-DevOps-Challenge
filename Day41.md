# Day 41: Building Custom Docker Images with Dockerfile
**100 Days Cloud DevOps Challenge – KodeKloud**

---

## Objective

Create a custom Docker image using a Dockerfile to meet application development team requirements for their project testing environment.

**Requirements:**
1. Work on Application Server 3 (App Server 3)
2. Create Dockerfile at: `/opt/docker/Dockerfile`
3. Base Image: `ubuntu:24.04`
4. Install: `apache2` package
5. Configure: Apache to work on port `8084`
6. Keep Dockerfile name with capital 'D'

---

## Understanding Dockerfiles

**Dockerfile** is a text file containing instructions to build Docker images. It's the blueprint for creating reproducible, automated container images.

### What is a Dockerfile?

A Dockerfile contains a series of instructions that Docker uses to:
- Choose a base image
- Install packages
- Copy files
- Configure services
- Set environment variables
- Define startup commands

### Dockerfile vs Docker Commit:

| Aspect | Dockerfile | docker commit |
|--------|-----------|---------------|
| **Reproducibility** | ✅ Fully reproducible | ❌ Manual, not reproducible |
| **Version Control** | ✅ Text file in git | ❌ Binary image |
| **Documentation** | ✅ Self-documenting | ❌ No clear history |
| **Automation** | ✅ CI/CD friendly | ❌ Manual process |
| **Best Practice** | ✅ Production standard | ⚠️ Dev/testing only |
| **Transparency** | ✅ All steps visible | ❌ Black box |

### Dockerfile Workflow:

```
Dockerfile (Instructions)
         ↓
    docker build
         ↓
    Docker Image
         ↓
    docker run
         ↓
    Running Container
```

---

## Understanding the Scenario

### Application Team Requirements:

```
1. Custom image needed for project testing
2. Base: Ubuntu 24.04 (latest LTS)
3. Web server: Apache2
4. Custom port: 8084 (not default 80)
5. Dockerfile location: /opt/docker/Dockerfile
6. Case-sensitive: Dockerfile (capital D)
```

### Why These Requirements?

**Ubuntu 24.04:**
- Latest LTS (Long Term Support)
- Stable, secure base
- Wide package support
- Team familiarity

**Apache2:**
- Popular web server
- Reliable and tested
- Extensive documentation
- Easy configuration

**Port 8084:**
- Avoid port conflicts
- Testing environment standard
- Allows multiple instances
- Non-privileged port

**Dockerfile Location:**
- Organized structure
- Clear separation
- Easy to find
- Version control ready

---

## Understanding Dockerfile Instructions

### Common Dockerfile Instructions:

```dockerfile
FROM        # Base image
RUN         # Execute commands
COPY        # Copy files from host
ADD         # Copy files (with extra features)
WORKDIR     # Set working directory
ENV         # Set environment variables
EXPOSE      # Document port usage
CMD         # Default command
ENTRYPOINT  # Main executable
USER        # Set user context
LABEL       # Add metadata
VOLUME      # Create mount point
ARG         # Build-time variables
```

### Our Dockerfile Instructions:

```dockerfile
FROM ubuntu:24.04
# Sets base image to Ubuntu 24.04

RUN apt-get update && apt-get install -y apache2
# Updates package lists and installs Apache2

RUN sed -i 's/Listen 80/Listen 8084/g' /etc/apache2/ports.conf
# Changes Apache port from 80 to 8084

RUN sed -i 's/<VirtualHost \*:80>/<VirtualHost *:8084>/g' /etc/apache2/sites-available/000-default.conf
# Updates VirtualHost to match port 8084

EXPOSE 8084
# Documents that container listens on port 8084

CMD ["apache2ctl", "-D", "FOREGROUND"]
# Runs Apache in foreground (keeps container alive)
```

### Instruction Breakdown:

**FROM ubuntu:24.04**
- Must be first instruction (except ARG/comments)
- Specifies base image
- Can specify version with tag

**RUN**
- Executes during build time
- Each RUN creates a new layer
- Chain commands with && to reduce layers

**EXPOSE**
- Documentation only (doesn't publish port)
- Indicates which ports container uses
- Actual port mapping done with `docker run -p`

**CMD**
- Provides default command
- Can be overridden at runtime
- Only last CMD is effective

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
| Dockerfile Location | `/opt/docker/Dockerfile` |
| Dockerfile Name | `Dockerfile` (capital D) |
| Base Image | `ubuntu:24.04` |
| Package | apache2 |
| Port | 8084 |
| Apache Config | Listen 8084, VirtualHost *:8084 |

---

## Understanding the Task

### What We're Creating:

```
Application Server 3 (stapp03)
│
├── /opt/docker/
│   └── Dockerfile
│       ├── FROM ubuntu:24.04
│       ├── RUN apt-get update
│       ├── RUN apt-get install apache2
│       ├── RUN configure port 8084
│       ├── EXPOSE 8084
│       └── CMD apache2ctl
│
└── docker build
    ↓
    Docker Image
    └── Ubuntu 24.04 + Apache2 on port 8084
```

### Build Flow:

```
1. SSH to App Server 3
2. Switch to root
3. Create /opt/docker directory
4. Create Dockerfile with requirements
5. Build image from Dockerfile
6. Verify image created
7. Test image (run container)
8. Verify Apache on port 8084
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

### Step 3: Create Directory Structure
```bash
mkdir -p /opt/docker
```

**Verify directory created:**
```bash
ls -la /opt/
```

**Expected output:**
```
total 12
drwxr-xr-x  3 root root 4096 Dec 17 10:30 .
dr-xr-xr-x 18 root root 4096 Dec 17 08:00 ..
drwxr-xr-x  2 root root 4096 Dec 17 10:30 docker
```

### Step 4: Navigate to Directory
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

### Step 5: Create Dockerfile
```bash
cat > Dockerfile << 'EOF'
FROM ubuntu:24.04

RUN apt-get update && apt-get install -y apache2

RUN sed -i 's/Listen 80/Listen 8084/g' /etc/apache2/ports.conf

RUN sed -i 's/<VirtualHost \*:80>/<VirtualHost *:8084>/g' /etc/apache2/sites-available/000-default.conf

EXPOSE 8084

CMD ["apache2ctl", "-D", "FOREGROUND"]
EOF
```

**Expected output:**
```
(No output means success)
```

**Alternative method using vi/nano:**
```bash
vi Dockerfile
```

**Then paste the content:**
```dockerfile
FROM ubuntu:24.04

RUN apt-get update && apt-get install -y apache2

RUN sed -i 's/Listen 80/Listen 8084/g' /etc/apache2/ports.conf

RUN sed -i 's/<VirtualHost \*:80>/<VirtualHost *:8084>/g' /etc/apache2/sites-available/000-default.conf

EXPOSE 8084

CMD ["apache2ctl", "-D", "FOREGROUND"]
```

**Save and exit (in vi: ESC, then :wq)**

### Step 6: Verify Dockerfile Created
```bash
ls -la
```

**Expected output:**
```
total 12
drwxr-xr-x 2 root root 4096 Dec 17 10:35 .
drwxr-xr-x 3 root root 4096 Dec 17 10:30 ..
-rw-r--r-- 1 root root  285 Dec 17 10:35 Dockerfile
```

**✅ Dockerfile exists with capital 'D'**

### Step 7: View Dockerfile Contents
```bash
cat Dockerfile
```

**Expected output:**
```dockerfile
FROM ubuntu:24.04

RUN apt-get update && apt-get install -y apache2

RUN sed -i 's/Listen 80/Listen 8084/g' /etc/apache2/ports.conf

RUN sed -i 's/<VirtualHost \*:80>/<VirtualHost *:8084>/g' /etc/apache2/sites-available/000-default.conf

EXPOSE 8084

CMD ["apache2ctl", "-D", "FOREGROUND"]
```

**Verify:**
- ✅ FROM ubuntu:24.04
- ✅ RUN apt-get install apache2
- ✅ Port configuration to 8084
- ✅ EXPOSE 8084
- ✅ CMD starts Apache

### Step 8: Check Current Docker Images
```bash
docker images
```

**Expected output:**
```
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
```

**Or may show existing images**

### Step 9: Build Docker Image
```bash
docker build -t custom-apache:latest /opt/docker/
```

**Expected output:**
```
[+] Building 45.3s (8/8) FINISHED
 => [internal] load build definition from Dockerfile                       0.1s
 => => transferring dockerfile: 285B                                       0.0s
 => [internal] load .dockerignore                                          0.0s
 => => transferring context: 2B                                            0.0s
 => [internal] load metadata for docker.io/library/ubuntu:24.04            2.3s
 => [1/4] FROM docker.io/library/ubuntu:24.04@sha256:...                  15.2s
 => => resolve docker.io/library/ubuntu:24.04@sha256:...                   0.0s
 => => sha256:... 27.35MB / 27.35MB                                        5.4s
 => => extracting sha256:...                                               8.9s
 => [2/4] RUN apt-get update && apt-get install -y apache2               22.5s
 => [3/4] RUN sed -i 's/Listen 80/Listen 8084/g' /etc/apache2/ports...     0.4s
 => [4/4] RUN sed -i 's/<VirtualHost \*:80>/<VirtualHost *:8084>/g' ...   0.3s
 => exporting to image                                                     4.2s
 => => exporting layers                                                    4.1s
 => => writing image sha256:a1b2c3d4e5f6...                               0.0s
 => => naming to docker.io/library/custom-apache:latest                   0.0s
```

**Build process breakdown:**
1. Loads Dockerfile
2. Pulls ubuntu:24.04 base image
3. Runs apt-get update and install
4. Configures ports.conf
5. Configures VirtualHost
6. Creates final image

**Command breakdown:**
- `docker build` - Build image from Dockerfile
- `-t custom-apache:latest` - Tag image as custom-apache:latest
- `/opt/docker/` - Build context directory

**Alternative build commands:**
```bash
# Build from current directory
cd /opt/docker && docker build -t custom-apache:latest .

# Build with specific Dockerfile name
docker build -f /opt/docker/Dockerfile -t custom-apache:latest /opt/docker/

# Build without cache (fresh build)
docker build --no-cache -t custom-apache:latest /opt/docker/
```

### Step 10: Verify Image Built Successfully
```bash
docker images
```

**Expected output:**
```
REPOSITORY       TAG       IMAGE ID       CREATED          SIZE
custom-apache    latest    a1b2c3d4e5f6   30 seconds ago   245MB
ubuntu           24.04     c3a134f2ace4   2 weeks ago      78.1MB
```

**✅ custom-apache image created!**

**Check specific image:**
```bash
docker images custom-apache
```

**Expected output:**
```
REPOSITORY       TAG       IMAGE ID       CREATED          SIZE
custom-apache    latest    a1b2c3d4e5f6   30 seconds ago   245MB
```

### Step 11: Inspect Image Details
```bash
docker inspect custom-apache:latest
```

**Expected output (partial):**
```json
[
    {
        "Id": "sha256:a1b2c3d4e5f6...",
        "RepoTags": [
            "custom-apache:latest"
        ],
        "Created": "2025-12-17T10:40:15.123456789Z",
        "Container": "",
        "Config": {
            "Hostname": "",
            "ExposedPorts": {
                "8084/tcp": {}
            },
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
            ],
            "Cmd": [
                "apache2ctl",
                "-D",
                "FOREGROUND"
            ],
            "Image": "sha256:...",
            "WorkingDir": ""
        },
        "Architecture": "amd64",
        "Os": "linux",
        "Size": 245123456
    }
]
```

**Verify:**
- ✅ "ExposedPorts": "8084/tcp"
- ✅ "Cmd": ["apache2ctl", "-D", "FOREGROUND"]

### Step 12: View Image History
```bash
docker history custom-apache:latest
```

**Expected output:**
```
IMAGE          CREATED              CREATED BY                                      SIZE
a1b2c3d4e5f6   About a minute ago   CMD ["apache2ctl" "-D" "FOREGROUND"]           0B
<missing>      About a minute ago   EXPOSE map[8084/tcp:{}]                         0B
<missing>      About a minute ago   RUN /bin/sh -c sed -i 's/<VirtualHost \*:8…    1.2kB
<missing>      About a minute ago   RUN /bin/sh -c sed -i 's/Listen 80/Listen 8…    1.1kB
<missing>      About a minute ago   RUN /bin/sh -c apt-get update && apt-get in…   167MB
<missing>      2 weeks ago          /bin/sh -c #(nop)  CMD ["/bin/bash"]            0B
<missing>      2 weeks ago          /bin/sh -c #(nop) ADD file:ddf1aa62235de66…    78.1MB
```

**Shows all layers created during build**

### Step 13: Test Image - Run Container
```bash
docker run -d --name test-apache -p 8084:8084 custom-apache:latest
```

**Expected output:**
```
d4e5f6a7b8c9d0e1f2a3b4c5d6e7f8a9b0c1d2e3f4e5f6a7b8c9d0e1f2a3b4c5
```

**Command breakdown:**
- `docker run` - Run container from image
- `-d` - Detached mode (background)
- `--name test-apache` - Container name
- `-p 8084:8084` - Port mapping (host:container)
- `custom-apache:latest` - Image to use

### Step 14: Verify Container Running
```bash
docker ps
```

**Expected output:**
```
CONTAINER ID   IMAGE                   COMMAND                  CREATED         STATUS         PORTS                                       NAMES
d4e5f6a7b8c9   custom-apache:latest    "apache2ctl -D FOREG…"   10 seconds ago  Up 9 seconds   0.0.0.0:8084->8084/tcp, :::8084->8084/tcp   test-apache
```

**✅ Container running on port 8084!**

### Step 15: Check Container Logs
```bash
docker logs test-apache
```

**Expected output:**
```
AH00558: apache2: Could not reliably determine the server's fully qualified domain name, using 172.17.0.2. Set the 'ServerName' directive globally to suppress this message
AH00558: apache2: Could not reliably determine the server's fully qualified domain name, using 172.17.0.2. Set the 'ServerName' directive globally to suppress this message
[Tue Dec 17 10:42:15.123456 2025] [mpm_event:notice] [pid 1:tid 140123456789] AH00489: Apache/2.4.58 (Ubuntu) configured -- resuming normal operations
[Tue Dec 17 10:42:15.123457 2025] [core:notice] [pid 1:tid 140123456789] AH00094: Command line: 'apache2 -D FOREGROUND'
```

**Apache started successfully!**

### Step 16: Verify Apache Configuration Inside Container
```bash
docker exec test-apache cat /etc/apache2/ports.conf | grep Listen
```

**Expected output:**
```
Listen 8084
```

**✅ Port configured correctly!**

**Check VirtualHost:**
```bash
docker exec test-apache cat /etc/apache2/sites-available/000-default.conf | grep VirtualHost
```

**Expected output:**
```
<VirtualHost *:8084>
</VirtualHost>
```

**✅ VirtualHost configured correctly!**

### Step 17: Test Apache Responding on Port 8084
```bash
curl http://localhost:8084
```

**Expected output (partial HTML):**
```html
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" ...>
<html xmlns="http://www.w3.org/1999/xhtml">
  <head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
    <title>Apache2 Ubuntu Default Page: It works</title>
    ...
  </head>
  <body>
    <div class="main_page">
      <div class="page_header floating_element">
        <img src="/icons/ubuntu-logo.png" alt="Ubuntu Logo" />
        <span class="floating_element">
          Apache2 Default Page
        </span>
      </div>
```

**✅ Apache serving web pages on port 8084!**

**Check HTTP headers:**
```bash
curl -I http://localhost:8084
```

**Expected output:**
```
HTTP/1.1 200 OK
Date: Tue, 17 Dec 2025 10:45:30 GMT
Server: Apache/2.4.58 (Ubuntu)
Last-Modified: Tue, 17 Dec 2025 10:42:10 GMT
ETag: "29af-6281e3b4a8e80"
Accept-Ranges: bytes
Content-Length: 10671
Content-Type: text/html
```

### Step 18: Check Port Listening
```bash
netstat -tuln | grep 8084
```

**Expected output:**
```
tcp        0      0 0.0.0.0:8084            0.0.0.0:*               LISTEN
tcp6       0      0 :::8084                 :::*                    LISTEN
```

**Alternative with ss:**
```bash
ss -tuln | grep 8084
```

**Expected output:**
```
tcp   LISTEN 0      511          0.0.0.0:8084       0.0.0.0:*
tcp   LISTEN 0      511             [::]:8084          [::]:*
```

### Step 19: Clean Up Test Container (Optional)
```bash
docker stop test-apache
docker rm test-apache
```

**Expected output:**
```
test-apache
test-apache
```

**Image still exists for future use:**
```bash
docker images custom-apache
```

### Step 20: Final Verification
```bash
# Verify Dockerfile exists with correct name
ls -la /opt/docker/Dockerfile

# Verify image built successfully
docker images | grep custom-apache

# Verify Dockerfile content
cat /opt/docker/Dockerfile
```

**All requirements met:**
- ✅ Dockerfile at /opt/docker/Dockerfile
- ✅ Capital 'D' in Dockerfile
- ✅ Base image: ubuntu:24.04
- ✅ Apache2 installed
- ✅ Port configured to 8084
- ✅ Image builds successfully
- ✅ Container runs and serves on port 8084

---

## Complete Command Summary

### Quick Dockerfile Creation and Build:
```bash
# SSH and access
ssh banner@stapp03
sudo su -

# Create directory and Dockerfile
mkdir -p /opt/docker
cd /opt/docker

cat > Dockerfile << 'EOF'
FROM ubuntu:24.04

RUN apt-get update && apt-get install -y apache2

RUN sed -i 's/Listen 80/Listen 8084/g' /etc/apache2/ports.conf

RUN sed -i 's/<VirtualHost \*:80>/<VirtualHost *:8084>/g' /etc/apache2/sites-available/000-default.conf

EXPOSE 8084

CMD ["apache2ctl", "-D", "FOREGROUND"]
EOF

# Build image
docker build -t custom-apache:latest /opt/docker/

# Verify
docker images custom-apache

# Test (optional)
docker run -d --name test-apache -p 8084:8084 custom-apache:latest
curl http://localhost:8084
```

### Complete Workflow:
```bash
# 1. SSH and authenticate
ssh banner@stapp03
sudo su -

# 2. Prepare directory
mkdir -p /opt/docker
cd /opt/docker

# 3. Create Dockerfile
cat > Dockerfile << 'EOF'
FROM ubuntu:24.04
RUN apt-get update && apt-get install -y apache2
RUN sed -i 's/Listen 80/Listen 8084/g' /etc/apache2/ports.conf
RUN sed -i 's/<VirtualHost \*:80>/<VirtualHost *:8084>/g' /etc/apache2/sites-available/000-default.conf
EXPOSE 8084
CMD ["apache2ctl", "-D", "FOREGROUND"]
EOF

# 4. Verify Dockerfile
cat Dockerfile

# 5. Build image
docker build -t custom-apache:latest /opt/docker/

# 6. Verify image
docker images custom-apache
docker history custom-apache:latest

# 7. Test image
docker run -d --name test-apache -p 8084:8084 custom-apache:latest
docker ps
docker logs test-apache

# 8. Verify Apache configuration
docker exec test-apache cat /etc/apache2/ports.conf | grep Listen
curl http://localhost:8084

# 9. Clean up (optional)
docker stop test-apache
docker rm test-apache
```

---

## Understanding Dockerfile Best Practices

### 1. Order Instructions Efficiently
```dockerfile
# ✅ Good: Least changing instructions first
FROM ubuntu:24.04
RUN apt-get update && apt-get install -y apache2
COPY config.conf /etc/apache2/
COPY app/ /var/www/html/

# ❌ Bad: Frequently changing instructions first
FROM ubuntu:24.04
COPY app/ /var/www/html/  # Changes often, breaks cache
RUN apt-get update && apt-get install -y apache2
```

### 2. Combine RUN Commands
```dockerfile
# ✅ Good: Single layer
RUN apt-get update && \
    apt-get install -y apache2 curl vim && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# ❌ Bad: Multiple layers
RUN apt-get update
RUN apt-get install -y apache2
RUN apt-get install -y curl
RUN apt-get install -y vim
```

### 3. Use Specific Image Tags
```dockerfile
# ✅ Good: Specific version
FROM ubuntu:24.04

# ⚠️ Risky: Latest tag
FROM ubuntu:latest

# ❌ Bad: No tag (defaults to latest)
FROM ubuntu
```

### 4. Clean Up in Same Layer
```dockerfile
# ✅ Good: Clean up in same RUN
RUN apt-get update && \
    apt-get install -y apache2 && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# ❌ Bad: Clean up in separate layer (doesn't reduce image size)
RUN apt-get update && apt-get install -y apache2
RUN apt-get clean
RUN rm -rf /var/lib/apt/lists/*
```

### 5. Use .dockerignore
```bash
# Create .dockerignore file
cat > /opt/docker/.dockerignore << 'EOF'
.git
.gitignore
README.md
.env
*.log
node_modules
__pycache__
EOF
```

### 6. Document with LABEL
```dockerfile
FROM ubuntu:24.04

LABEL maintainer="devops@nautilus.com"
LABEL version="1.0"
LABEL description="Custom Apache on port 8084"

RUN apt-get update && apt-get install -y apache2
...
```

---

## Advanced Dockerfile Patterns

### Multi-Stage Builds:
```dockerfile
# Build stage
FROM ubuntu:24.04 AS builder
RUN apt-get update && apt-get install -y build-essential
COPY source/ /app/
RUN cd /app && make build

# Runtime stage
FROM ubuntu:24.04
COPY --from=builder /app/binary /usr/local/bin/
CMD ["/usr/local/bin/binary"]
```

### ARG for Build-Time Variables:
```dockerfile
FROM ubuntu:24.04

ARG APACHE_PORT=8084
ARG DEBIAN_FRONTEND=noninteractive

RUN apt-get update && apt-get install -y apache2

RUN sed -i "s/Listen 80/Listen ${APACHE_PORT}/g" /etc/apache2/ports.conf

EXPOSE ${APACHE_PORT}

CMD ["apache2ctl", "-D", "FOREGROUND"]
```

**Build with custom port:**
```bash
docker build --build-arg APACHE_PORT=9090 -t custom-apache:9090 .
```

### ENV for Runtime Variables:
```dockerfile
FROM ubuntu:24.04

ENV APACHE_PORT=8084
ENV APACHE_LOG_DIR=/var/log/apache2

RUN apt-get update && apt-get install -y apache2

EXPOSE ${APACHE_PORT}

CMD ["apache2ctl", "-D", "FOREGROUND"]
```

### HEALTHCHECK:
```dockerfile
FROM ubuntu:24.04

RUN apt-get update && apt-get install -y apache2 curl

RUN sed -i 's/Listen 80/Listen 8084/g' /etc/apache2/ports.conf

EXPOSE 8084

HEALTHCHECK --interval=30s --timeout=3s \
  CMD curl -f http://localhost:8084/ || exit 1

CMD ["apache2ctl", "-D", "FOREGROUND"]
```

### ENTRYPOINT vs CMD:
```dockerfile
# ENTRYPOINT: Always runs, CMD provides default args
FROM ubuntu:24.04
RUN apt-get update && apt-get install -y apache2
ENTRYPOINT ["apache2ctl"]
CMD ["-D", "FOREGROUND"]

# Can override CMD: docker run image -k stop
# But ENTRYPOINT always runs
```

---

## Troubleshooting

### Issue 1: Dockerfile Not Found

**Problem:**
```
unable to prepare context: unable to evaluate symlinks in Dockerfile path: lstat /opt/docker/Dockerfile: no such file or directory
```

**Solution:**
```bash
# Check current directory
pwd

# Verify Dockerfile exists
ls -la /opt/docker/Dockerfile

# Check spelling (capital D)
ls -la /opt/docker/dockerfile  # Wrong
ls -la /opt/docker/Dockerfile  # Correct

# Build with explicit path
docker build -f /opt/docker/Dockerfile -t custom-apache:latest /opt/docker/
```

### Issue 2: Base Image Not Found

**Problem:**
```
failed to solve with frontend dockerfile.v0: failed to resolve source metadata for docker.io/library/ubuntu:24.04: pull access denied, repository does not exist or may require 'docker login'
```

**Solution:**
```bash
# Check image availability
docker pull ubuntu:24.04

# Verify tag exists
# Ubuntu 24.04 is Noble Numbat, released April 2024

# If not available, use existing version
FROM ubuntu:22.04  # or ubuntu:latest
```

### Issue 3: Build Fails During apt-get

**Problem:**
```
E: Unable to locate package apache2
```

**Solution:**
```bash
# Ensure apt-get update runs first
RUN apt-get update && apt-get install -y apache2

# Not separate commands:
RUN apt-get update
RUN apt-get install -y apache2  # May use cached layer without update
```

### Issue 4: Port Configuration Not Applied

**Problem:**
Apache still listens on port 80 instead of 8084

**Solution:**
```bash
# Check sed command syntax
RUN sed -i 's/Listen 80/Listen 8084/g' /etc/apache2/ports.conf

# Verify both files updated
RUN sed -i 's/Listen 80/Listen 8084/g' /etc/apache2/ports.conf && \
    sed -i 's/<VirtualHost \*:80>/<VirtualHost *:8084>/g' /etc/apache2/sites-available/000-default.conf

# Test in container
docker run -it custom-apache:latest bash
cat /etc/apache2/ports.conf
cat /etc/apache2/sites-available/000-default.conf
```

### Issue 5: Container Exits Immediately

**Problem:**
```
docker ps
# Container not listed
docker ps -a
# Shows Exited (0)
```

**Solution:**
```bash
# Ensure Apache runs in foreground
CMD ["apache2ctl", "-D", "FOREGROUND"]

# Not:
CMD ["apache2ctl", "start"]  # Exits after starting

# Check logs
docker logs container_name
```

### Issue 6: Build Cache Issues

**Problem:**
Changes not reflected in new build

**Solution:**
```bash
# Build without cache
docker build --no-cache -t custom-apache:latest /opt/docker/

# Or remove existing image
docker rmi custom-apache:latest
docker build -t custom-apache:latest /opt/docker/
```

### Issue 7: Permission Denied

**Problem:**
```
Got permission denied while trying to connect to the Docker daemon socket
```

**Solution:**
```bash
# Use sudo
sudo docker build -t custom-apache:latest /opt/docker/

# Or switch to root
sudo su -
docker build -t custom-apache:latest /opt/docker/
```

---

## Best Practices

### 1. Use Official Base Images
```dockerfile
# ✅ Official Ubuntu image
FROM ubuntu:24.04

# ✅ Official Apache image (alternative approach)
FROM httpd:2.4

# ⚠️ Unknown source
FROM randomuser/ubuntu
```

### 2. Minimize Layers
```dockerfile
# ✅ Good: Single RUN with multiple commands
RUN apt-get update && \
    apt-get install -y \
        apache2 \
        curl \
        vim && \
    apt-get clean

# ❌ Bad: Multiple RUN commands
RUN apt-get update
RUN apt-get install -y apache2
RUN apt-get install -y curl
RUN apt-get install -y vim
```

### 3. Keep Images Small
```dockerfile
# ✅ Clean up in same layer
RUN apt-get update && \
    apt-get install -y apache2 && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Consider alpine base for smaller images
FROM alpine:latest
RUN apk add --no-cache apache2
```

### 4. Use Specific Versions
```dockerfile
# ✅ Specific version
FROM ubuntu:24.04

# ✅ SHA256 digest (most specific)
FROM ubuntu@sha256:abc123...

# ⚠️ Floating tag
FROM ubuntu:latest
```

### 5. Document Everything
```dockerfile
FROM ubuntu:24.04

# Install Apache web server
RUN apt-get update && apt-get install -y apache2

# Configure Apache to listen on custom port
RUN sed -i 's/Listen 80/Listen 8084/g' /etc/apache2/ports.conf

# Update VirtualHost configuration
RUN sed -i 's/<VirtualHost \*:80>/<VirtualHost *:8084>/g' /etc/apache2/sites-available/000-default.conf

# Expose port for documentation
EXPOSE 8084

# Run Apache in foreground
CMD ["apache2ctl", "-D", "FOREGROUND"]
```

### 6. Leverage Build Cache
```dockerfile
# ✅ Dependencies first (change infrequently)
FROM ubuntu:24.04
RUN apt-get update && apt-get install -y apache2

# Application code last (changes frequently)
COPY app/ /var/www/html/
```

### 7. Don't Run as Root (Production)
```dockerfile
FROM ubuntu:24.04
RUN apt-get update && apt-get install -y apache2

# Create non-root user
RUN useradd -r -u 1001 apache

# Change ownership
RUN chown -R apache:apache /var/www/html

# Switch user
USER apache

CMD ["apache2ctl", "-D", "FOREGROUND"]
```

---

## Real-World Scenarios

### Scenario 1: Multi-Environment Dockerfiles
```dockerfile
# Development
FROM ubuntu:24.04
RUN apt-get update && apt-get install -y apache2 vim curl
EXPOSE 8084
CMD ["apache2ctl", "-D", "FOREGROUND"]

# Production (optimized)
FROM ubuntu:24.04
RUN apt-get update && \
    apt-get install -y --no-install-recommends apache2 && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
EXPOSE 8084
CMD ["apache2ctl", "-D", "FOREGROUND"]
```

### Scenario 2: Application with Dependencies
```dockerfile
FROM ubuntu:24.04

# Install Apache and PHP
RUN apt-get update && \
    apt-get install -y \
        apache2 \
        php \
        libapache2-mod-php \
        php-mysql && \
    apt-get clean

# Copy application files
COPY app/ /var/www/html/

# Configure Apache
RUN sed -i 's/Listen 80/Listen 8084/g' /etc/apache2/ports.conf
RUN sed -i 's/<VirtualHost \*:80>/<VirtualHost *:8084>/g' /etc/apache2/sites-available/000-default.conf

EXPOSE 8084
CMD ["apache2ctl", "-D", "FOREGROUND"]
```

### Scenario 3: Custom Configuration Files
```dockerfile
FROM ubuntu:24.04

RUN apt-get update && apt-get install -y apache2

# Copy custom configurations
COPY apache2.conf /etc/apache2/apache2.conf
COPY ports.conf /etc/apache2/ports.conf
COPY site.conf /etc/apache2/sites-available/000-default.conf

# Enable modules
RUN a2enmod rewrite ssl

EXPOSE 8084
CMD ["apache2ctl", "-D", "FOREGROUND"]
```

### Scenario 4: Static Website
```dockerfile
FROM ubuntu:24.04

RUN apt-get update && apt-get install -y apache2

# Copy website files
COPY website/ /var/www/html/

# Remove default index
RUN rm /var/www/html/index.html

# Configure port
RUN sed -i 's/Listen 80/Listen 8084/g' /etc/apache2/ports.conf
RUN sed -i 's/<VirtualHost \*:80>/<VirtualHost *:8084>/g' /etc/apache2/sites-available/000-default.conf

EXPOSE 8084
CMD ["apache2ctl", "-D", "FOREGROUND"]
```

### Scenario 5: Automated Build Pipeline
```bash
#!/bin/bash
# build.sh

# Variables
IMAGE_NAME="custom-apache"
TAG=$(date +%Y%m%d)

# Build image
docker build -t ${IMAGE_NAME}:${TAG} /opt/docker/

# Tag as latest
docker tag ${IMAGE_NAME}:${TAG} ${IMAGE_NAME}:latest

# Test image
docker run -d --name test -p 8084:8084 ${IMAGE_NAME}:${TAG}
sleep 5
curl -f http://localhost:8084 || exit 1
docker stop test && docker rm test

# Push to registry
docker push ${IMAGE_NAME}:${TAG}
docker push ${IMAGE_NAME}:latest

echo "Build complete: ${IMAGE_NAME}:${TAG}"
```

---

## Key Commands Reference

| Command | Description |
|---------|-------------|
| `docker build -t NAME:TAG PATH` | Build image from Dockerfile |
| `docker build -f DOCKERFILE -t NAME PATH` | Build with specific Dockerfile |
| `docker build --no-cache -t NAME PATH` | Build without cache |
| `docker images` | List images |
| `docker history IMAGE` | View image layers |
| `docker inspect IMAGE` | Inspect image details |
| `docker rmi IMAGE` | Remove image |
| `docker tag SOURCE TARGET` | Tag image |
| `docker save IMAGE -o FILE` | Export image to tar |
| `docker load -i FILE` | Import image from tar |

---

## Dockerfile Instruction Reference

| Instruction | Description | Example |
|-------------|-------------|---------|
| `FROM` | Base image | `FROM ubuntu:24.04` |
| `RUN` | Execute command | `RUN apt-get install apache2` |
| `CMD` | Default command | `CMD ["apache2ctl", "-D", "FOREGROUND"]` |
| `ENTRYPOINT` | Main executable | `ENTRYPOINT ["apache2ctl"]` |
| `EXPOSE` | Document port | `EXPOSE 8084` |
| `ENV` | Set environment | `ENV PORT=8084` |
| `ARG` | Build argument | `ARG VERSION=24.04` |
| `COPY` | Copy files | `COPY app/ /var/www/` |
| `ADD` | Copy with extras | `ADD archive.tar.gz /app/` |
| `WORKDIR` | Set directory | `WORKDIR /var/www` |
| `USER` | Set user | `USER apache` |
| `VOLUME` | Mount point | `VOLUME /var/log` |
| `LABEL` | Add metadata | `LABEL version="1.0"` |
| `HEALTHCHECK` | Health check | `HEALTHCHECK CMD curl localhost` |

---

## Completion Checklist

- [ ] SSH into Application Server 3 (stapp03) as banner
- [ ] Switched to root user
- [ ] Created /opt/docker directory
- [ ] Created Dockerfile with capital 'D'
- [ ] Used ubuntu:24.04 as base image
- [ ] Added apt-get update command
- [ ] Installed apache2 package
- [ ] Configured ports.conf to Listen 8084
- [ ] Configured VirtualHost to *:8084
- [ ] Added EXPOSE 8084
- [ ] Added CMD to run Apache in foreground
- [ ] Built Docker image successfully
- [ ] Verified image exists
- [ ] Tested image by running container
- [ ] Verified Apache serves on port 8084
- [ ] All requirements met ✅

---

## Completion Details

- **Completion Date:** December 17, 2025
- **Day:** 41 / 100
- **Challenge:** KodeKloud 100 Days Cloud DevOps
- **Topic:** Building Custom Docker Images with Dockerfile
- **Server:** Application Server 3 (stapp03)
- **User:** banner
- **Dockerfile Location:** `/opt/docker/Dockerfile`
- **Base Image:** ubuntu:24.04
- **Package Installed:** apache2
- **Port Configured:** 8084
- **Image Name:** custom-apache:latest
- **Image Size:** ~245MB
- **Build Status:** ✅ Successful
- **Test Status:** ✅ Apache serving on port 8084
- **Key Skill:** Dockerfile creation and image building
- **Status:** ✅ Successfully Completed

---

## Summary

This task demonstrated **building custom Docker images using Dockerfiles**:

✅ **Created Dockerfile** - Infrastructure as code for image building
✅ **Used ubuntu:24.04** - Latest LTS base image
✅ **Installed apache2** - Web server package
✅ **Configured custom port** - Apache on port 8084
✅ **Built image successfully** - Automated, reproducible process
✅ **Tested deployment** - Verified Apache serves on correct port

**Key Insight:** Dockerfiles provide **automated, reproducible image building** compared to manual docker commit:
- **Version Control** - Text file can be tracked in git
- **Documentation** - Every step is visible and documented
- **Automation** - CI/CD pipelines can build automatically
- **Reproducibility** - Same Dockerfile = same image every time
- **Collaboration** - Team members can understand and modify
- **Best Practice** - Industry standard for container images

**The Dockerfile Build Process:**
```
Dockerfile (Instructions)
         ↓
    docker build
         ↓
    Read FROM instruction → Pull base image
         ↓
    Execute RUN commands → Create layers
         ↓
    Apply EXPOSE, CMD → Set metadata
         ↓
    Final Image → Ready to run
```

**Our Dockerfile Structure:**
```dockerfile
FROM ubuntu:24.04
↓ Starts with Ubuntu 24.04 LTS base

RUN apt-get update && apt-get install -y apache2
↓ Installs Apache web server

RUN sed -i 's/Listen 80/Listen 8084/g' /etc/apache2/ports.conf
↓ Changes listening port to 8084

RUN sed -i 's/<VirtualHost \*:80>/<VirtualHost *:8084>/g' /etc/apache2/sites-available/000-default.conf
↓ Updates VirtualHost configuration

EXPOSE 8084
↓ Documents port 8084 is used

CMD ["apache2ctl", "-D", "FOREGROUND"]
↓ Runs Apache in foreground (keeps container alive)
```

**Dockerfile vs Docker Commit:**
```
Docker Commit:
✅ Fast for testing
✅ Good for capturing state
❌ Not reproducible
❌ No documentation
❌ Can't version control
❌ Manual process

Dockerfile:
✅ Fully reproducible
✅ Self-documenting
✅ Version controlled
✅ Automated builds
✅ CI/CD friendly
✅ Production standard
```

**Image Layers:**
Each RUN, COPY, ADD creates a new layer. Our image has:
1. Ubuntu 24.04 base (78.1MB)
2. apt-get update + apache2 install (167MB)
3. ports.conf modification (1.1KB)
4. VirtualHost modification (1.2KB)
5. Metadata (EXPOSE, CMD) (0B)

**Total Size: ~245MB**

**Best Practices Applied:**
- ✅ Used specific base image tag (ubuntu:24.04)
- ✅ Combined apt-get update && install in single RUN
- ✅ Added EXPOSE for documentation
- ✅ Used CMD for foreground process
- ✅ Clear, readable Dockerfile structure

**Production Improvements:**
```dockerfile
# Clean up to reduce size
RUN apt-get update && \
    apt-get install -y apache2 && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Add health check
HEALTHCHECK CMD curl -f http://localhost:8084/ || exit 1

# Add labels
LABEL maintainer="team@nautilus.com"
LABEL version="1.0"
```

**Testing the Image:**
```bash
# Build
docker build -t custom-apache:latest /opt/docker/

# Run
docker run -d -p 8084:8084 custom-apache:latest

# Test
curl http://localhost:8084
# ✅ Apache2 Ubuntu Default Page: It works
```

**Remember:** `Dockerfile` = Blueprint for reproducible container images. It's the **standard way** to create Docker images in production! 🐳

**Next:** Push image to registry, use in Kubernetes, implement in CI/CD pipeline! 🚀
