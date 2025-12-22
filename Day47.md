# Day 47: Dockerizing a Python Application
**100 Days Cloud DevOps Challenge – KodeKloud**

---

## Objective

Dockerize a Python web application by creating a Dockerfile, building a custom Docker image, and deploying it as a container. Learn the complete workflow from creating a Dockerfile to testing a containerized application.

**Requirements:**
1. Work on Application Server 2 (App Server 2) in Stratos Datacenter
2. Application files located at: `/python_app/src/` (requirements.txt already present)
3. **Create Dockerfile at:** `/python_app/Dockerfile`
4. **Dockerfile Requirements:**
   - Use any Python image as base
   - Install dependencies from `requirements.txt`
   - Expose port `8089`
   - Run `server.py` using CMD
5. **Build Image:** `nautilus/python-app`
6. **Create Container:** `pythonapp_nautilus`
   - Map container port `8089` to host port `8094`
7. **Test:** `curl http://localhost:8094/`

---

## Understanding Dockerizing Applications

**Dockerizing** is the process of packaging an application with all its dependencies into a Docker container, making it portable, consistent, and easy to deploy across different environments.

### Why Dockerize Applications?

```
Traditional Deployment Challenges:
├── "Works on my machine" syndrome
├── Complex dependency management
├── Environment inconsistencies
├── Difficult to scale
└── Long deployment times

Docker Solution:
├── ✅ Consistent across all environments
├── ✅ All dependencies packaged together
├── ✅ Isolated from host system
├── ✅ Easy to scale and deploy
└── ✅ Fast startup and deployment
```

### Docker Application Workflow:

```
1. Create Dockerfile
   ↓
2. Build Docker Image
   ↓
3. Push to Registry (optional)
   ↓
4. Run as Container
   ↓
5. Test Application
```

---

## Understanding Python Docker Images

### Official Python Images:

| Image | Size | Description | Use Case |
|-------|------|-------------|----------|
| **python:3.12** | ~1GB | Full Python 3.12 | **Our task - includes pip, setuptools** |
| python:3.12-slim | ~180MB | Minimal Python 3.12 | Production - smaller size |
| python:3.12-alpine | ~50MB | Alpine-based Python | Minimal footprint |
| python:3.11 | ~1GB | Full Python 3.11 | Stable, widely used |
| python:3.9 | ~900MB | Full Python 3.9 | Legacy compatibility |

### Python Image Comparison:

```
python:3.12 (Full)
├── Size: ~1GB
├── Includes: pip, setuptools, wheel, gcc, build tools
├── Pros: Everything included, easy to use
└── Cons: Large size

python:3.12-slim
├── Size: ~180MB
├── Includes: pip, Python only
├── Pros: 80% smaller, still functional
└── Cons: May need to install build tools

python:3.12-alpine
├── Size: ~50MB
├── Includes: Minimal Python
├── Pros: Smallest size
└── Cons: May have compatibility issues with some packages
```

---

## Understanding Dockerfile Instructions

### Key Dockerfile Instructions:

| Instruction | Purpose | Example |
|-------------|---------|---------|
| **FROM** | Base image | `FROM python:3.12` |
| **WORKDIR** | Set working directory | `WORKDIR /app` |
| **COPY** | Copy files from host to image | `COPY . /app` |
| **RUN** | Execute commands during build | `RUN pip install -r requirements.txt` |
| **EXPOSE** | Document which port to expose | `EXPOSE 8089` |
| **CMD** | Default command when container starts | `CMD ["python", "server.py"]` |
| **ENV** | Set environment variables | `ENV PYTHONUNBUFFERED=1` |
| **ENTRYPOINT** | Configure container as executable | `ENTRYPOINT ["python"]` |

### Dockerfile Best Practices:

```dockerfile
# ✅ Good Dockerfile
FROM python:3.12
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
EXPOSE 8089
CMD ["python", "server.py"]

# ❌ Bad Dockerfile
FROM python:3.12
COPY . /
RUN pip install flask
# No WORKDIR, copies everything to root, hardcoded dependencies
```

---

## Understanding Port Mapping

### Container Port vs Host Port:

```
Port Mapping Syntax: -p HOST_PORT:CONTAINER_PORT

Example: -p 8094:8089
         ↓         ↓
    Host Port  Container Port
    (External) (Internal)

How it works:
External Request (port 8094)
    ↓
Host forwards to Container (port 8089)
    ↓
Application inside container receives request
    ↓
Response sent back through same path
```

### Why Different Ports?

```
Scenario: Multiple applications on same host

Host:
├── App 1 → localhost:8091 → Container A:8080
├── App 2 → localhost:8092 → Container B:8080
├── App 3 → localhost:8093 → Container C:8080
└── Our App → localhost:8094 → Container D:8089

All containers can use same internal port,
but must map to different host ports!
```

---

## Understanding Python Web Applications

### Typical Python Web App Structure:

```
/python_app/
├── src/
│   ├── server.py           # Main application file
│   ├── requirements.txt    # Python dependencies
│   ├── templates/          # HTML templates (optional)
│   └── static/             # CSS, JS files (optional)
├── Dockerfile              # Docker build instructions
├── .dockerignore           # Files to exclude from build
└── README.md               # Documentation
```

### Common Python Web Frameworks:

| Framework | Description | Use Case |
|-----------|-------------|----------|
| **Flask** | Lightweight micro-framework | **Simple APIs, small apps** |
| Django | Full-featured framework | Large web applications |
| FastAPI | Modern, fast framework | High-performance APIs |
| Tornado | Asynchronous framework | Real-time applications |

### Sample Flask Application:

```python
# server.py
from flask import Flask

app = Flask(__name__)

@app.route('/')
def hello():
    return "Welcome to Nautilus Python App!"

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8089)
```

**Key Points:**
- `host='0.0.0.0'` - Listen on all interfaces (required for Docker)
- `port=8089` - Application listens on this port
- Must match EXPOSE in Dockerfile

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
| App Directory | `/python_app/` |
| Source Files | `/python_app/src/` (requirements.txt present) |
| Dockerfile Location | `/python_app/Dockerfile` |
| Image Name | `nautilus/python-app` |
| Container Name | `pythonapp_nautilus` |
| Container Port | 8089 |
| Host Port | 8094 |
| Test URL | `http://localhost:8094/` |

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

### Step 3: Verify Application Directory Structure
```bash
# Navigate to python_app directory
cd /python_app

# List directory contents
ls -la
```

**Expected output:**
```
total 12
drwxr-xr-x  3 root root 4096 Dec 22 10:00 .
dr-xr-xr-x 19 root root 4096 Dec 22 10:00 ..
drwxr-xr-x  2 root root 4096 Dec 22 10:00 src
```

**✅ src directory exists**

### Step 4: Check Application Files
```bash
# List files in src directory
ls -la src/
```

**Expected output:**
```
total 16
drwxr-xr-x 2 root root 4096 Dec 22 10:00 .
drwxr-xr-x 3 root root 4096 Dec 22 10:00 ..
-rw-r--r-- 1 root root   45 Dec 22 10:00 requirements.txt
-rw-r--r-- 1 root root  321 Dec 22 10:00 server.py
```

**✅ Both files present: requirements.txt and server.py**

### Step 5: Examine requirements.txt
```bash
cat src/requirements.txt
```

**Expected output (example):**
```
Flask==3.0.0
Werkzeug==3.0.1
```

**This tells us:**
- Application uses Flask framework
- Specific versions are pinned (good practice)
- Simple application with minimal dependencies

### Step 6: Examine server.py (Optional)
```bash
cat src/server.py
```

**Expected output (example):**
```python
from flask import Flask

app = Flask(__name__)

@app.route('/')
def home():
    return '<h1>Welcome to xFusionCorp Industries!</h1>'

@app.route('/health')
def health():
    return 'OK', 200

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8089, debug=True)
```

**Key observations:**
- Flask application
- Listens on `0.0.0.0:8089`
- Has two routes: `/` and `/health`
- Uses debug mode (good for development)

### Step 7: Create Dockerfile
```bash
# Make sure you're in /python_app directory
cd /python_app

# Create Dockerfile
cat > Dockerfile << 'EOF'
# Use official Python runtime as base image
FROM python:3.12

# Set working directory in container
WORKDIR /app

# Copy requirements file first (better caching)
COPY src/requirements.txt .

# Install Python dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY src/ .

# Expose port 8089 for the application
EXPOSE 8089

# Run the application
CMD ["python", "server.py"]
EOF
```

**Expected output:**
```
(No output means success)
```

**Dockerfile Explanation:**
1. **FROM python:3.12** - Use official Python 3.12 image as base
2. **WORKDIR /app** - Set working directory to /app in container
3. **COPY src/requirements.txt .** - Copy requirements.txt first (Docker layer caching optimization)
4. **RUN pip install...** - Install Python packages during image build
5. **COPY src/ .** - Copy all application files from src/ to /app
6. **EXPOSE 8089** - Document that application uses port 8089
7. **CMD ["python", "server.py"]** - Command to run when container starts

### Step 8: Verify Dockerfile Created
```bash
ls -la
```

**Expected output:**
```
total 16
drwxr-xr-x  3 root root 4096 Dec 22 10:10 .
dr-xr-xr-x 19 root root 4096 Dec 22 10:00 ..
-rw-r--r--  1 root root  456 Dec 22 10:10 Dockerfile
drwxr-xr-x  2 root root 4096 Dec 22 10:00 src
```

**✅ Dockerfile created at /python_app/Dockerfile**

### Step 9: View Dockerfile Contents
```bash
cat Dockerfile
```

**Expected output:**
```dockerfile
# Use official Python runtime as base image
FROM python:3.12

# Set working directory in container
WORKDIR /app

# Copy requirements file first (better caching)
COPY src/requirements.txt .

# Install Python dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY src/ .

# Expose port 8089 for the application
EXPOSE 8089

# Run the application
CMD ["python", "server.py"]
```

**Verify:**
- ✅ FROM instruction uses Python image
- ✅ WORKDIR set to /app
- ✅ COPY requirements.txt before installing
- ✅ RUN pip install command
- ✅ COPY application files
- ✅ EXPOSE 8089
- ✅ CMD runs server.py

### Step 10: Build Docker Image
```bash
# Build image with tag nautilus/python-app
docker build -t nautilus/python-app .
```

**Expected output:**
```
[+] Building 45.2s (10/10) FINISHED
 => [internal] load build definition from Dockerfile                       0.1s
 => => transferring dockerfile: 456B                                       0.0s
 => [internal] load metadata for docker.io/library/python:3.12             2.3s
 => [internal] load .dockerignore                                          0.0s
 => => transferring context: 2B                                            0.0s
 => [1/5] FROM docker.io/library/python:3.12@sha256:abc123...             15.2s
 => => resolve docker.io/library/python:3.12@sha256:abc123...              0.0s
 => => sha256:abc123... 1.86kB / 1.86kB                                    0.0s
 => => sha256:def456... 8.52kB / 8.52kB                                    0.0s
 => => sha256:ghi789... 49.58MB / 49.58MB                                  8.3s
 => => extracting sha256:ghi789...                                         3.2s
 => [internal] load build context                                          0.1s
 => => transferring context: 1.23kB                                        0.0s
 => [2/5] WORKDIR /app                                                     0.5s
 => [3/5] COPY src/requirements.txt .                                      0.1s
 => [4/5] RUN pip install --no-cache-dir -r requirements.txt              25.4s
 => [5/5] COPY src/ .                                                      0.1s
 => exporting to image                                                     1.2s
 => => exporting layers                                                    1.1s
 => => writing image sha256:xyz789...                                      0.0s
 => => naming to docker.io/library/nautilus/python-app                    0.0s
```

**Build Process Breakdown:**
1. **Load Dockerfile** - Reads build instructions
2. **Pull base image** - Downloads python:3.12 (if not cached)
3. **Set WORKDIR** - Creates /app directory
4. **Copy requirements.txt** - Copies dependency file
5. **Install dependencies** - Runs pip install (takes longest)
6. **Copy application** - Copies source code
7. **Create image** - Packages everything into final image

**✅ Image built successfully!**

### Step 11: Verify Image Created
```bash
docker images | grep nautilus
```

**Expected output:**
```
REPOSITORY              TAG       IMAGE ID       CREATED          SIZE
nautilus/python-app     latest    a1b2c3d4e5f6   30 seconds ago   1.02GB
```

**Verify:**
- ✅ Repository: nautilus/python-app
- ✅ Tag: latest (default)
- ✅ Size: ~1GB (Python full image)
- ✅ CREATED: Just now

### Step 12: Inspect Image Details (Optional)
```bash
docker inspect nautilus/python-app
```

**Expected output (excerpt):**
```json
[
    {
        "Id": "sha256:a1b2c3d4e5f6...",
        "RepoTags": [
            "nautilus/python-app:latest"
        ],
        "Config": {
            "ExposedPorts": {
                "8089/tcp": {}
            },
            "Cmd": [
                "python",
                "server.py"
            ],
            "WorkingDir": "/app"
        },
        "Architecture": "amd64",
        "Size": 1020456789
    }
]
```

**Verify:**
- ✅ ExposedPorts: 8089/tcp
- ✅ Cmd: ["python", "server.py"]
- ✅ WorkingDir: /app

### Step 13: Run Container from Image
```bash
# Run container in detached mode
docker run -d --name pythonapp_nautilus -p 8094:8089 nautilus/python-app
```

**Expected output:**
```
a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6q7r8s9t0u1v2w3x4y5z6
```

**Command breakdown:**
- `docker run` - Create and start container
- `-d` - Detached mode (run in background)
- `--name pythonapp_nautilus` - Container name (exact requirement)
- `-p 8094:8089` - Port mapping: host 8094 → container 8089
- `nautilus/python-app` - Image to use

**✅ Container started!**

### Step 14: Verify Container Running
```bash
docker ps
```

**Expected output:**
```
CONTAINER ID   IMAGE                 COMMAND               CREATED          STATUS          PORTS                    NAMES
a1b2c3d4e5f6   nautilus/python-app   "python server.py"    20 seconds ago   Up 18 seconds   0.0.0.0:8094->8089/tcp   pythonapp_nautilus
```

**Verify:**
- ✅ Container ID present
- ✅ IMAGE: nautilus/python-app
- ✅ COMMAND: python server.py
- ✅ STATUS: Up (running)
- ✅ PORTS: 0.0.0.0:8094->8089/tcp (correct mapping)
- ✅ NAMES: pythonapp_nautilus (exact name)

### Step 15: Check Container Logs
```bash
docker logs pythonapp_nautilus
```

**Expected output:**
```
 * Serving Flask app 'server'
 * Debug mode: on
WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
 * Running on all addresses (0.0.0.0)
 * Running on http://127.0.0.1:8089
 * Running on http://172.17.0.2:8089
Press CTRL+C to quit
 * Restarting with stat
 * Debugger is active!
 * Debugger PIN: 123-456-789
```

**Verify:**
- ✅ Flask server started
- ✅ Listening on 0.0.0.0:8089
- ✅ Debug mode active
- ✅ No errors

### Step 16: Test Application with curl
```bash
curl http://localhost:8094/
```

**Expected output:**
```html
<h1>Welcome to xFusionCorp Industries!</h1>
```

**✅ Application responding successfully!**

**Test with verbose output:**
```bash
curl -v http://localhost:8094/
```

**Expected output:**
```
* Connected to localhost (127.0.0.1) port 8094
> GET / HTTP/1.1
> Host: localhost:8094
> User-Agent: curl/7.68.0
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< Server: Werkzeug/3.0.1 Python/3.12.0
< Date: Sun, 22 Dec 2025 10:15:00 GMT
< Content-Type: text/html; charset=utf-8
< Content-Length: 47
< 
<h1>Welcome to xFusionCorp Industries!</h1>
```

**Verify:**
- ✅ Connection successful to localhost:8094
- ✅ HTTP 200 OK response
- ✅ Content received
- ✅ Flask/Werkzeug server responding

### Step 17: Test Health Endpoint (If Available)
```bash
curl http://localhost:8094/health
```

**Expected output:**
```
OK
```

**Or:**
```json
{"status": "healthy"}
```

### Step 18: Verify Port Mapping
```bash
docker port pythonapp_nautilus
```

**Expected output:**
```
8089/tcp -> 0.0.0.0:8094
```

**✅ Port mapping confirmed: Container 8089 → Host 8094**

### Step 19: Check Container Resource Usage
```bash
docker stats --no-stream pythonapp_nautilus
```

**Expected output:**
```
CONTAINER ID   NAME                  CPU %     MEM USAGE / LIMIT     MEM %     NET I/O          BLOCK I/O   PIDS
a1b2c3d4e5f6   pythonapp_nautilus    0.15%     42.5MiB / 7.77GiB    0.53%     1.23kB / 2.34kB  0B / 0B     5
```

**Typical Python Flask app resources:**
- CPU: 0.1-0.5% (idle)
- Memory: 40-80MB (small app)
- PIDs: 3-10 processes

### Step 20: Inspect Container Details
```bash
docker inspect pythonapp_nautilus | grep -A 10 "IPAddress"
```

**Expected output:**
```json
"IPAddress": "172.17.0.2",
"IPPrefixLen": 16,
"Gateway": "172.17.0.1",
```

**Container IP:** 172.17.0.2 (internal Docker network)

### Step 21: Test from Inside Container (Optional)
```bash
docker exec pythonapp_nautilus curl http://localhost:8089/
```

**Expected output:**
```html
<h1>Welcome to xFusionCorp Industries!</h1>
```

**✅ Application accessible from inside container on port 8089**

### Step 22: View Container Processes
```bash
docker top pythonapp_nautilus
```

**Expected output:**
```
UID         PID    PPID   C   STIME   TTY   TIME       CMD
root        1234   5678   0   10:10   ?     00:00:01   python server.py
root        1235   1234   0   10:10   ?     00:00:00   /usr/local/bin/python server.py
```

### Step 23: Final Verification Checklist
```bash
# 1. Dockerfile exists at correct location
ls -la /python_app/Dockerfile

# 2. Image built with correct name
docker images | grep nautilus/python-app

# 3. Container running with correct name
docker ps | grep pythonapp_nautilus

# 4. Port mapping correct
docker port pythonapp_nautilus | grep "8089.*8094"

# 5. Application responding
curl -s http://localhost:8094/ | grep -i "welcome\|fusionCorp\|success"

# 6. No container errors
docker logs pythonapp_nautilus 2>&1 | grep -i "error\|failed" && echo "Errors found" || echo "No errors"
```

**All checks should pass:**
- ✅ Dockerfile at /python_app/Dockerfile
- ✅ Image: nautilus/python-app
- ✅ Container: pythonapp_nautilus
- ✅ Port mapping: 8094:8089
- ✅ Application responding on localhost:8094
- ✅ No errors in logs

---

## Complete Command Summary

### Quick Deployment:
```bash
# SSH and access
ssh steve@stapp02
sudo su -

# Navigate to application directory
cd /python_app

# Verify files
ls -la src/

# Create Dockerfile
cat > Dockerfile << 'EOF'
FROM python:3.12
WORKDIR /app
COPY src/requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY src/ .
EXPOSE 8089
CMD ["python", "server.py"]
EOF

# Build image
docker build -t nautilus/python-app .

# Run container
docker run -d --name pythonapp_nautilus -p 8094:8089 nautilus/python-app

# Test application
curl http://localhost:8094/
```

### Detailed Workflow:
```bash
# 1. SSH and authenticate
ssh steve@stapp02
sudo su -

# 2. Navigate to app directory
cd /python_app

# 3. Check existing files
ls -la src/
cat src/requirements.txt
cat src/server.py

# 4. Create Dockerfile
vi Dockerfile
# (Paste Dockerfile content)

# 5. Verify Dockerfile
cat Dockerfile

# 6. Build Docker image
docker build -t nautilus/python-app .

# 7. Verify image created
docker images | grep nautilus

# 8. Run container
docker run -d --name pythonapp_nautilus -p 8094:8089 nautilus/python-app

# 9. Check container status
docker ps
docker logs pythonapp_nautilus

# 10. Test application
curl http://localhost:8094/
curl -v http://localhost:8094/

# 11. Verify port mapping
docker port pythonapp_nautilus

# 12. Check resource usage
docker stats --no-stream pythonapp_nautilus
```

---

## Understanding Docker Build Process

### Build Context:

```
Docker Build Context = Files sent to Docker daemon

/python_app/              ← Build context root
├── Dockerfile            ← Build instructions
├── src/
│   ├── requirements.txt  ← Dependencies
│   └── server.py         ← Application
└── .dockerignore         ← Files to exclude

When you run: docker build -t nautilus/python-app .
                                                   ↑
                                    Current directory = build context

Docker sends ALL files in build context to daemon
Use .dockerignore to exclude unnecessary files!
```

### Build Layers:

```
Docker Image Layers (Read-Only):

Layer 5: CMD ["python", "server.py"]         ← 0 bytes
Layer 4: COPY src/ .                         ← ~10KB (app files)
Layer 3: RUN pip install...                  ← ~50MB (packages)
Layer 2: COPY src/requirements.txt .         ← ~1KB
Layer 1: WORKDIR /app                        ← 0 bytes
Layer 0: FROM python:3.12                    ← ~950MB (base image)
         ↓
Total: ~1GB

Each instruction creates a new layer
Layers are cached for faster rebuilds!
```

### Layer Caching:

```bash
# First build: Takes 2 minutes
docker build -t nautilus/python-app .

# Change server.py and rebuild
# Docker reuses cached layers 0-3
# Only rebuilds layers 4-5
# Takes 5 seconds!

Best Practice:
Copy requirements.txt BEFORE application code
Changes to code don't invalidate dependency layer
```

---

## Dockerfile Optimization

### Current Dockerfile (Good):

```dockerfile
FROM python:3.12
WORKDIR /app
COPY src/requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY src/ .
EXPOSE 8089
CMD ["python", "server.py"]
```

**Size: ~1GB (Full Python image)**

### Optimized Dockerfile (Better):

```dockerfile
# Use slim image for smaller size
FROM python:3.12-slim
WORKDIR /app

# Copy and install dependencies first (better caching)
COPY src/requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY src/ .

# Create non-root user for security
RUN useradd -m appuser && chown -R appuser:appuser /app
USER appuser

# Expose port
EXPOSE 8089

# Health check
HEALTHCHECK --interval=30s --timeout=3s \
  CMD python -c "import urllib.request; urllib.request.urlopen('http://localhost:8089/health')"

# Run application
CMD ["python", "server.py"]
```

**Size: ~200MB (80% smaller!)**

### Production Dockerfile (Best):

```dockerfile
# Multi-stage build
FROM python:3.12-slim AS builder
WORKDIR /app
COPY src/requirements.txt .
RUN pip install --no-cache-dir --user -r requirements.txt

FROM python:3.12-slim
WORKDIR /app

# Copy installed packages from builder
COPY --from=builder /root/.local /root/.local
ENV PATH=/root/.local/bin:$PATH

# Copy application
COPY src/ .

# Non-root user
RUN useradd -m appuser && chown -R appuser:appuser /app
USER appuser

# Expose port
EXPOSE 8089

# Health check
HEALTHCHECK CMD python -c "import urllib.request; urllib.request.urlopen('http://localhost:8089/health')"

# Run with Gunicorn (production WSGI server)
CMD ["gunicorn", "--bind", "0.0.0.0:8089", "--workers", "4", "server:app"]
```

**Size: ~180MB + Better security + Production-ready**

---

## Docker Commands Reference

### Image Management:

```bash
# Build image
docker build -t NAME:TAG .                  # Build from Dockerfile
docker build -t NAME:TAG -f Dockerfile.prod .  # Use specific Dockerfile
docker build --no-cache -t NAME .           # Build without cache

# List images
docker images                               # List all images
docker images | grep nautilus               # Filter images
docker images -a                            # Include intermediate images

# Remove images
docker rmi IMAGE_NAME                       # Remove image
docker rmi -f IMAGE_NAME                    # Force remove
docker image prune                          # Remove unused images
docker image prune -a                       # Remove all unused images

# Inspect image
docker inspect IMAGE_NAME                   # Detailed info
docker history IMAGE_NAME                   # Show layers
```

### Container Management:

```bash
# Run container
docker run -d --name NAME -p HOST:CONTAINER IMAGE
docker run -it --name NAME IMAGE /bin/bash  # Interactive
docker run --rm IMAGE                       # Remove after exit
docker run -e KEY=VALUE IMAGE               # Set environment variable

# List containers
docker ps                                   # Running containers
docker ps -a                                # All containers
docker ps -q                                # Only container IDs

# Stop/Start containers
docker stop CONTAINER                       # Graceful stop
docker kill CONTAINER                       # Force stop
docker start CONTAINER                      # Start stopped container
docker restart CONTAINER                    # Restart container

# Remove containers
docker rm CONTAINER                         # Remove stopped container
docker rm -f CONTAINER                      # Force remove running
docker container prune                      # Remove stopped containers

# Container info
docker logs CONTAINER                       # View logs
docker logs -f CONTAINER                    # Follow logs
docker stats CONTAINER                      # Resource usage
docker top CONTAINER                        # Running processes
docker port CONTAINER                       # Port mappings
docker inspect CONTAINER                    # Detailed info

# Execute in container
docker exec CONTAINER COMMAND               # Run command
docker exec -it CONTAINER /bin/bash         # Interactive shell
```

---

## Troubleshooting Guide

### Issue 1: Build Fails - Can't Find requirements.txt

**Problem:**
```
ERROR [3/5] COPY src/requirements.txt .:
failed to compute cache key: "/src/requirements.txt" not found
```

**Solution:**
```bash
# Check file exists
ls -la /python_app/src/requirements.txt

# Verify you're in correct directory
pwd  # Should be /python_app

# Check Dockerfile path
cat Dockerfile | grep COPY

# Rebuild
docker build -t nautilus/python-app .
```

### Issue 2: Build Fails - pip Install Error

**Problem:**
```
ERROR: Could not find a version that satisfies the requirement Flask==3.0.0
```

**Solution:**
```bash
# Check requirements.txt format
cat src/requirements.txt

# Try without version pinning
echo "Flask" > src/requirements.txt

# Use specific Python version
# Change Dockerfile FROM line:
FROM python:3.11  # Instead of 3.12

# Rebuild
docker build -t nautilus/python-app .
```

### Issue 3: Container Starts but Exits Immediately

**Problem:**
```bash
docker ps
# Container not shown (exited)

docker ps -a
# STATUS: Exited (1) 5 seconds ago
```

**Diagnosis:**
```bash
# Check logs for error
docker logs pythonapp_nautilus
```

**Common causes:**
```
# Error: server.py not found
# Solution: Check COPY instruction in Dockerfile

# Error: Module not found
# Solution: Verify requirements.txt installed correctly

# Error: Port already in use
# Solution: Container can't bind to port (different issue)
```

**Solution:**
```bash
# Run container interactively to debug
docker run -it --rm nautilus/python-app /bin/bash

# Inside container, manually test:
python server.py
# See actual error

# Fix Dockerfile and rebuild
```

### Issue 4: Can't Access Application on Port 8094

**Problem:**
```bash
curl http://localhost:8094/
# curl: (7) Failed to connect to localhost port 8094
```

**Diagnosis:**
```bash
# 1. Check container is running
docker ps | grep pythonapp_nautilus

# 2. Check port mapping
docker port pythonapp_nautilus

# 3. Check application logs
docker logs pythonapp_nautilus

# 4. Check from inside container
docker exec pythonapp_nautilus curl http://localhost:8089/
```

**Solutions:**

**A. Container not running:**
```bash
docker start pythonapp_nautilus
```

**B. Wrong port mapping:**
```bash
# Remove and recreate with correct ports
docker rm -f pythonapp_nautilus
docker run -d --name pythonapp_nautilus -p 8094:8089 nautilus/python-app
```

**C. Application not listening on 0.0.0.0:**
```python
# server.py must have:
app.run(host='0.0.0.0', port=8089)
# NOT:
app.run(host='127.0.0.1', port=8089)
```

**D. Firewall blocking:**
```bash
# Check firewall
sudo firewall-cmd --list-ports
sudo firewall-cmd --add-port=8094/tcp --permanent
sudo firewall-cmd --reload
```

### Issue 5: Port 8094 Already in Use

**Problem:**
```
docker: Error response from daemon: driver failed programming external 
connectivity: Bind for 0.0.0.0:8094 failed: port is already allocated.
```

**Solution:**
```bash
# Check what's using port 8094
sudo lsof -i :8094
sudo netstat -tuln | grep 8094

# Option 1: Stop the service using port
sudo kill -9 PID

# Option 2: Use different host port
docker run -d --name pythonapp_nautilus -p 8095:8089 nautilus/python-app
curl http://localhost:8095/

# Option 3: Remove existing container
docker rm -f pythonapp_nautilus
docker run -d --name pythonapp_nautilus -p 8094:8089 nautilus/python-app
```

### Issue 6: Container Name Already in Use

**Problem:**
```
docker: Error response from daemon: Conflict. The container name 
"/pythonapp_nautilus" is already in use by container "abc123...".
```

**Solution:**
```bash
# Option 1: Remove existing container
docker rm -f pythonapp_nautilus
docker run -d --name pythonapp_nautilus -p 8094:8089 nautilus/python-app

# Option 2: Use different name (not recommended for this task)
docker run -d --name pythonapp_nautilus_v2 -p 8094:8089 nautilus/python-app

# Option 3: Start existing container
docker start pythonapp_nautilus
```

### Issue 7: Image Build Takes Too Long

**Problem:**
Build stuck at "RUN pip install..." for several minutes

**Solution:**
```bash
# Use pip cache
docker build --build-arg PIP_NO_CACHE_DIR=0 -t nautilus/python-app .

# Use faster mirror
# Add to Dockerfile before RUN pip:
RUN pip config set global.index-url https://pypi.org/simple

# Reduce dependencies
# Check if all packages in requirements.txt are needed

# Use smaller base image
FROM python:3.12-slim  # Instead of python:3.12
```

---

## Advanced Configurations

### Multi-Stage Build for Smaller Images:

```dockerfile
# Stage 1: Builder
FROM python:3.12 AS builder
WORKDIR /app
COPY src/requirements.txt .
RUN pip install --user --no-cache-dir -r requirements.txt

# Stage 2: Runtime
FROM python:3.12-slim
WORKDIR /app
COPY --from=builder /root/.local /root/.local
COPY src/ .
ENV PATH=/root/.local/bin:$PATH
EXPOSE 8089
CMD ["python", "server.py"]
```

**Benefits:**
- Final image doesn't include build tools
- 40-50% smaller than single-stage
- More secure (fewer packages)

### With Environment Variables:

```dockerfile
FROM python:3.12
WORKDIR /app

# Set environment variables
ENV PYTHONUNBUFFERED=1 \
    PYTHONDONTWRITEBYTECODE=1 \
    PORT=8089

COPY src/requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY src/ .

EXPOSE ${PORT}
CMD ["python", "server.py"]
```

**Run with custom env:**
```bash
docker run -d --name pythonapp_nautilus \
  -p 8094:8089 \
  -e DEBUG=True \
  -e LOG_LEVEL=INFO \
  nautilus/python-app
```

### With Volumes for Development:

```bash
# Mount source code as volume for live reload
docker run -d --name pythonapp_nautilus \
  -p 8094:8089 \
  -v /python_app/src:/app \
  nautilus/python-app
```

**Benefits:**
- Edit code on host
- Changes reflect immediately
- No rebuild needed

### With Health Check:

```dockerfile
FROM python:3.12
WORKDIR /app
COPY src/requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY src/ .
EXPOSE 8089

# Add health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD python -c "import urllib.request; urllib.request.urlopen('http://localhost:8089/health')" || exit 1

CMD ["python", "server.py"]
```

**Check health status:**
```bash
docker inspect pythonapp_nautilus | grep -A 10 Health
```

---

## Best Practices Summary

### Dockerfile Best Practices:

1. **Use Official Images:**
   ```dockerfile
   FROM python:3.12  # ✅ Official
   # NOT: FROM random/python  # ❌
   ```

2. **Specify Versions:**
   ```dockerfile
   FROM python:3.12  # ✅ Specific version
   # NOT: FROM python  # ❌ Latest (unpredictable)
   ```

3. **Layer Ordering:**
   ```dockerfile
   # ✅ Dependencies first, code last
   COPY requirements.txt .
   RUN pip install -r requirements.txt
   COPY . .
   
   # ❌ Code first means cache invalidation
   ```

4. **Use .dockerignore:**
   ```
   __pycache__
   *.pyc
   .git
   .env
   venv/
   ```

5. **Run as Non-Root:**
   ```dockerfile
   RUN useradd -m appuser
   USER appuser
   ```

6. **Clean Up in Same Layer:**
   ```dockerfile
   RUN apt-get update && \
       apt-get install -y package && \
       apt-get clean && \
       rm -rf /var/lib/apt/lists/*
   ```

### Container Best Practices:

1. **Use Specific Names:**
   ```bash
   docker run --name pythonapp_nautilus  # ✅
   # NOT: docker run  # ❌ Random name
   ```

2. **Explicit Port Mapping:**
   ```bash
   -p 8094:8089  # ✅ Clear mapping
   # NOT: -P  # ❌ Random host port
   ```

3. **Detached Mode for Services:**
   ```bash
   docker run -d  # ✅ Background
   # NOT: docker run  # ❌ Blocks terminal
   ```

4. **Check Logs Regularly:**
   ```bash
   docker logs -f pythonapp_nautilus
   ```

5. **Clean Up:**
   ```bash
   # Remove stopped containers
   docker container prune
   
   # Remove unused images
   docker image prune
   ```

---

## Real-World Application

### Development Workflow:

```bash
# 1. Develop locally
cd /python_app
vi src/server.py

# 2. Build image
docker build -t nautilus/python-app:dev .

# 3. Run container
docker run -d --name dev_app -p 8094:8089 nautilus/python-app:dev

# 4. Test
curl http://localhost:8094/

# 5. Make changes, rebuild
docker stop dev_app
docker rm dev_app
docker build -t nautilus/python-app:dev .
docker run -d --name dev_app -p 8094:8089 nautilus/python-app:dev
```

### Production Deployment:

```bash
# 1. Build production image
docker build -t nautilus/python-app:1.0.0 .

# 2. Tag for registry
docker tag nautilus/python-app:1.0.0 registry.example.com/nautilus/python-app:1.0.0

# 3. Push to registry
docker push registry.example.com/nautilus/python-app:1.0.0

# 4. Pull and run on production server
docker pull registry.example.com/nautilus/python-app:1.0.0
docker run -d --name pythonapp_prod \
  -p 80:8089 \
  --restart unless-stopped \
  registry.example.com/nautilus/python-app:1.0.0
```

### CI/CD Integration:

```yaml
# .github/workflows/docker-build.yml
name: Build Docker Image

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      
      - name: Build image
        run: docker build -t nautilus/python-app:${{ github.sha }} .
      
      - name: Run tests
        run: |
          docker run -d --name test_app -p 8094:8089 nautilus/python-app:${{ github.sha }}
          sleep 5
          curl http://localhost:8094/
      
      - name: Push to registry
        run: |
          echo ${{ secrets.DOCKER_PASSWORD }} | docker login -u ${{ secrets.DOCKER_USERNAME }} --password-stdin
          docker push nautilus/python-app:${{ github.sha }}
```

---

## Key Commands Reference

| Command | Description |
|---------|-------------|
| `docker build -t NAME .` | Build image from Dockerfile |
| `docker images` | List images |
| `docker run -d --name NAME -p HOST:CONTAINER IMAGE` | Run container |
| `docker ps` | List running containers |
| `docker logs CONTAINER` | View container logs |
| `docker exec CONTAINER COMMAND` | Execute command in container |
| `docker stop CONTAINER` | Stop container |
| `docker rm CONTAINER` | Remove container |
| `docker rmi IMAGE` | Remove image |
| `docker inspect CONTAINER` | Detailed container info |
| `docker port CONTAINER` | Show port mappings |

---

## Completion Checklist

- [ ] SSH into Application Server 2 (stapp02) as steve
- [ ] Switched to root user
- [ ] Navigated to /python_app directory
- [ ] Verified src/requirements.txt and src/server.py exist
- [ ] Created Dockerfile at /python_app/Dockerfile
- [ ] Dockerfile uses Python base image
- [ ] Dockerfile installs dependencies from requirements.txt
- [ ] Dockerfile exposes port 8089
- [ ] Dockerfile runs server.py using CMD
- [ ] Built image named nautilus/python-app
- [ ] Verified image created successfully
- [ ] Created container named pythonapp_nautilus
- [ ] Port mapping: host 8094 → container 8089
- [ ] Container running successfully
- [ ] Application responding on http://localhost:8094/
- [ ] No errors in container logs
- [ ] All requirements met ✅

---

## Completion Details

- **Completion Date:** December 22, 2025
- **Day:** 47 / 100
- **Challenge:** KodeKloud 100 Days Cloud DevOps
- **Topic:** Dockerizing a Python Application
- **Server:** Application Server 2 (stapp02) in Stratos Datacenter
- **User:** steve
- **App Directory:** /python_app/
- **Dockerfile Location:** /python_app/Dockerfile
- **Image Name:** nautilus/python-app
- **Container Name:** pythonapp_nautilus
- **Port Mapping:** 8094:8089 (host:container)
- **Test URL:** http://localhost:8094/
- **Key Skill:** Creating Dockerfiles, building images, running containerized applications
- **Status:** ✅ Successfully Completed

---

## Summary

This task demonstrated **complete application containerization workflow**:

✅ **Created Dockerfile** - Defined build instructions for Python app
✅ **Built Custom Image** - Packaged application with dependencies
✅ **Ran Container** - Deployed application in isolated environment
✅ **Port Mapping** - Made application accessible on host
✅ **Tested Application** - Verified functionality with curl

**Key Insight:** Dockerizing applications provides **consistency, portability, and isolation**:

**The Containerization Process:**
```
Source Code + Dependencies
    ↓
Dockerfile (Build Instructions)
    ↓
Docker Build (Create Image)
    ↓
Docker Image (Immutable Package)
    ↓
Docker Run (Create Container)
    ↓
Running Application (Isolated Process)
```

**Benefits of Dockerizing Applications:**

| Aspect | Traditional | Docker |
|--------|-------------|--------|
| **Setup** | Install Python, deps on host | Pull image, run |
| **Dependencies** | System-wide conflicts | Isolated in container |
| **Consistency** | "Works on my machine" | Works everywhere |
| **Deployment** | Complex setup scripts | Single docker run |
| **Scaling** | Provision new servers | Spin up containers |
| **Isolation** | Shared resources | Separate namespaces |

**Dockerfile Structure:**
```dockerfile
FROM python:3.12              # Base: What to start with
WORKDIR /app                  # Setup: Where to work
COPY requirements.txt .       # Dependencies: What app needs
RUN pip install -r ...        # Install: Prepare environment
COPY src/ .                   # Code: Application files
EXPOSE 8089                   # Document: Which port
CMD ["python", "server.py"]   # Run: Start application
```

**Port Mapping Explained:**
```
External Request → Host Port 8094
                       ↓
            Docker Port Mapping
                       ↓
           Container Port 8089
                       ↓
          Python App Listening
                       ↓
              Response Back
```

**Best Practices Applied:**
- ✅ Used official Python image
- ✅ Copied requirements.txt before app code (layer caching)
- ✅ Used --no-cache-dir to reduce image size
- ✅ Specific WORKDIR for organization
- ✅ Explicit port exposure
- ✅ CMD in exec form for proper signal handling

**Common Python Dockerization Patterns:**
```
Flask/Django Apps:
├── Use python:3.12 or python:3.12-slim
├── Install requirements.txt
├── Run with development server (dev)
└── Run with Gunicorn/uWSGI (production)

FastAPI Apps:
├── Use python:3.12-slim
├── Install requirements.txt
└── Run with Uvicorn (ASGI server)

ML/Data Science:
├── Use python:3.12 or jupyter/datascience
├── Install heavy dependencies (pandas, numpy, tensorflow)
└── Mount data volumes
```

**Real-World Use Cases:**
- **Microservices:** Each service in separate container
- **CI/CD Pipelines:** Build, test, deploy containerized apps
- **Development Environments:** Consistent dev setup across team
- **Multi-version Testing:** Run different Python versions simultaneously
- **Cloud Deployment:** Deploy containers to AWS ECS, Azure Container Instances, GKE

**Remember:** Dockerizing applications makes them **portable, scalable, and cloud-ready**! The same image runs identically on your laptop, test servers, and production cloud environments! 🐳

**Next Steps:**
- Explore Docker Compose for multi-container apps
- Learn Docker networking (bridge, host, overlay)
- Implement multi-stage builds for smaller images
- Add health checks and monitoring
- Deploy to Kubernetes for orchestration

**Next:** Docker networking, Docker volumes, Docker Compose multi-service stacks, and Kubernetes deployments! 🚀
