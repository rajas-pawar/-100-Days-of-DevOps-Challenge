# Day 45: Debugging Dockerfile Build Failures
**100 Days Cloud DevOps Challenge – KodeKloud**

---

## Objective

Troubleshoot and fix a broken Dockerfile on App Server 3 that's failing to build. Identify syntax errors, configuration issues, and fix them while maintaining the base image and existing data integrity.

**Requirements:**
1. Work on Application Server 3 (App Server 3)
2. Dockerfile location: `/opt/docker/Dockerfile`
3. Fix build errors without changing base image
4. Preserve all valid configurations
5. Do not modify data files (e.g., index.html)
6. Ensure successful image build

---

## Understanding Dockerfile Build Failures

**Docker build failures** occur when the Dockerfile contains syntax errors, invalid instructions, incorrect paths, or misconfigured commands. Debugging requires systematic analysis and understanding of Docker syntax rules.

### Common Dockerfile Build Errors:

```
1. Syntax Errors
   ├── Invalid instruction names (RUM instead of RUN)
   ├── Missing arguments
   ├── Incorrect capitalization
   └── Malformed commands

2. Instruction Order Issues
   ├── CMD/ENTRYPOINT before COPY
   ├── ENV not set before use
   ├── WORKDIR used before creation
   └── Dependencies not installed

3. Path Problems
   ├── Non-existent source files
   ├── Incorrect COPY paths
   ├── Wrong WORKDIR paths
   └── Missing parent directories

4. Layer Caching Issues
   ├── Unnecessary cache busting
   ├── Order causing rebuilds
   └── Large layers

5. Configuration Errors
   ├── Invalid EXPOSE syntax
   ├── Wrong CMD format
   ├── Incorrect ENTRYPOINT
   └── Invalid JSON arrays
```

### Docker Build Process:

```
Dockerfile
    ↓
docker build command
    ↓
Docker Engine reads Dockerfile
    ↓
Executes instructions sequentially
    ↓
Each instruction = new layer
    ↓
Any error = build fails
    ↓
Fix error and rebuild
```

---

## Understanding Dockerfile Syntax

### Dockerfile Structure:

```dockerfile
# Base Image (MUST be first non-comment instruction)
FROM ubuntu:latest

# Metadata
LABEL maintainer="devops@example.com"

# Environment Variables
ENV APP_HOME=/app
ENV PORT=8080

# Working Directory
WORKDIR /app

# Copy Files (from build context to image)
COPY ./app /app
COPY index.html /var/www/html/

# Run Commands (during build)
RUN apt-get update && \
    apt-get install -y nginx && \
    rm -rf /var/lib/apt/lists/*

# Expose Ports (documentation only)
EXPOSE 80

# Default Command (runs when container starts)
CMD ["nginx", "-g", "daemon off;"]
```

### Instruction Rules:

| Instruction | Purpose | Syntax | Notes |
|------------|---------|--------|-------|
| **FROM** | Base image | `FROM image:tag` | MUST be first (except ARG) |
| **RUN** | Execute commands | `RUN command` | Builds layer |
| **COPY** | Copy files | `COPY src dest` | From build context |
| **ADD** | Copy + extract | `ADD src dest` | Can download URLs |
| **WORKDIR** | Set working dir | `WORKDIR /path` | Creates if missing |
| **ENV** | Set environment | `ENV KEY=value` | Available in container |
| **EXPOSE** | Document ports | `EXPOSE port` | Metadata only |
| **CMD** | Default command | `CMD ["exec", "form"]` | One per Dockerfile |
| **ENTRYPOINT** | Container exec | `ENTRYPOINT ["cmd"]` | Hard to override |
| **LABEL** | Add metadata | `LABEL key="value"` | For organization |

### CMD vs ENTRYPOINT:

```dockerfile
# CMD - Easy to override
CMD ["python", "app.py"]
# Run: docker run image              → python app.py
# Run: docker run image ls           → ls (CMD replaced)

# ENTRYPOINT - Hard to override
ENTRYPOINT ["python", "app.py"]
# Run: docker run image              → python app.py
# Run: docker run image ls           → python app.py ls (appends)

# Combined - Best of both
ENTRYPOINT ["python"]
CMD ["app.py"]
# Run: docker run image              → python app.py
# Run: docker run image script.py    → python script.py
```

---

## Common Dockerfile Errors and Fixes

### Error 1: Invalid Instruction Name

**❌ Broken:**
```dockerfile
FROM ubuntu:latest
RUM apt-get update          # Typo: RUM instead of RUN
```

**Error Message:**
```
unknown instruction: RUM
```

**✅ Fixed:**
```dockerfile
FROM ubuntu:latest
RUN apt-get update
```

### Error 2: FROM Not First Instruction

**❌ Broken:**
```dockerfile
RUN echo "Building image"   # RUN before FROM
FROM ubuntu:latest
```

**Error Message:**
```
FROM instruction must be the first non-comment instruction
```

**✅ Fixed:**
```dockerfile
FROM ubuntu:latest
RUN echo "Building image"
```

### Error 3: Missing COPY Source

**❌ Broken:**
```dockerfile
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/
# But index.html doesn't exist in build context
```

**Error Message:**
```
COPY failed: stat /var/lib/docker/tmp/docker-builder123/index.html: no such file or directory
```

**✅ Fixed:**
```dockerfile
FROM nginx:alpine
# Ensure index.html exists in same directory as Dockerfile
COPY index.html /usr/share/nginx/html/
```

### Error 4: ADD Used Instead of RUN

**❌ Broken:**
```dockerfile
FROM httpd:2.4.43
ADD sed -i "s/Listen 80/Listen 8080/g" /usr/local/apache2/conf/httpd.conf
```

**Error Message:**
```
ADD failed: source path does not exist: sed
```

**Issue:** ADD is for copying files, not running commands

**✅ Fixed:**
```dockerfile
FROM httpd:2.4.43
RUN sed -i "s/Listen 80/Listen 8080/g" /usr/local/apache2/conf/httpd.conf
```

**Understanding ADD vs RUN vs COPY:**

| Instruction | Purpose | Use For |
|-------------|---------|---------|
| **RUN** | Execute commands during build | Installing packages, running scripts, file modifications |
| **COPY** | Copy files from host to image | Application code, config files, static assets |
| **ADD** | Copy + auto-extract + URL download | Tarballs (auto-extracts), remote files (discouraged) |

**Examples:**
```dockerfile
# RUN - Execute commands
RUN apt-get update && apt-get install -y nginx
RUN sed -i 's/Listen 80/Listen 8080/g' /etc/nginx/nginx.conf
RUN chmod +x /app/script.sh

# COPY - Copy local files (preferred for simple copying)
COPY index.html /usr/share/nginx/html/
COPY app.py /app/
COPY config.json /etc/app/

# ADD - Copy with special features (use sparingly)
ADD https://example.com/file.tar.gz /tmp/    # Downloads from URL
ADD app.tar.gz /app/                         # Auto-extracts tar archives
```

**Best Practice:** Always use COPY unless you specifically need ADD's special features.

### Error 5: Incorrect CMD Syntax

**❌ Broken:**
```dockerfile
FROM ubuntu:latest
RUN apt-get update && apt-get install -y python3
CMD python3 -m http.server 8000    # Shell form with daemon
```

**Issue:** Process runs as PID 1, signal handling issues

**✅ Fixed:**
```dockerfile
FROM ubuntu:latest
RUN apt-get update && apt-get install -y python3
CMD ["python3", "-m", "http.server", "8000"]   # Exec form
```

### Error 6: EXPOSE Syntax Error

**❌ Broken:**
```dockerfile
FROM nginx:alpine
EXPOSE 80,443        # Comma-separated
```

**Error Message:**
```
invalid syntax in EXPOSE instruction
```

**✅ Fixed:**
```dockerfile
FROM nginx:alpine
EXPOSE 80 443        # Space-separated
```

**Or:**
```dockerfile
FROM nginx:alpine
EXPOSE 80
EXPOSE 443
```

### Error 7: WORKDIR Before Files Exist

**❌ Broken:**
```dockerfile
FROM ubuntu:latest
WORKDIR /nonexistent/deep/path/app
COPY app.py .
# Directory doesn't exist yet
```

**Note:** Actually WORKDIR creates directory automatically, but good practice:

**✅ Better:**
```dockerfile
FROM ubuntu:latest
RUN mkdir -p /app
WORKDIR /app
COPY app.py .
```

### Error 8: Multiple CMD Instructions

**❌ Broken:**
```dockerfile
FROM ubuntu:latest
CMD ["echo", "First command"]
CMD ["echo", "Second command"]   # Only this one executes
```

**Result:** Only last CMD executes

**✅ Fixed:**
```dockerfile
FROM ubuntu:latest
# Combine into one command
CMD ["sh", "-c", "echo 'First command' && echo 'Second command'"]
```

**Or use script:**
```dockerfile
FROM ubuntu:latest
COPY startup.sh /startup.sh
RUN chmod +x /startup.sh
CMD ["/startup.sh"]
```

### Error 9: ENV Variable Syntax

**❌ Broken:**
```dockerfile
FROM ubuntu:latest
ENV APP_HOME /app PORT 8080    # Multiple vars, wrong syntax
```

**✅ Fixed (Option 1):**
```dockerfile
FROM ubuntu:latest
ENV APP_HOME=/app
ENV PORT=8080
```

**✅ Fixed (Option 2):**
```dockerfile
FROM ubuntu:latest
ENV APP_HOME=/app \
    PORT=8080
```

### Error 10: Missing Backslash in RUN

**❌ Broken:**
```dockerfile
FROM ubuntu:latest
RUN apt-get update
    apt-get install -y nginx     # Missing && or \
```

**Error Message:**
```
unknown instruction: APT-GET
```

**✅ Fixed:**
```dockerfile
FROM ubuntu:latest
RUN apt-get update && \
    apt-get install -y nginx
```

### Error 11: Incorrect COPY Destination

**❌ Broken:**
```dockerfile
FROM nginx:alpine
COPY index.html /usr/share/nginx/html    # Missing trailing slash
```

**Issue:** Copies as file named "html" instead of into directory

**✅ Fixed:**
```dockerfile
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/
```

---

## Dockerfile Debugging Methodology

### Step-by-Step Debugging Process:

```
1. Read Error Message
   ↓
2. Identify Line Number
   ↓
3. Check Instruction Syntax
   ↓
4. Verify File Paths
   ↓
5. Validate Instruction Order
   ↓
6. Check Build Context
   ↓
7. Test Fix
   ↓
8. Rebuild
```

### Debugging Commands:

```bash
# 1. Attempt build and capture error
docker build -t test-image .

# 2. View Dockerfile
cat Dockerfile

# 3. Check build context files
ls -la

# 4. Validate specific instruction
# (fix and rebuild)

# 5. Build with no cache (fresh build)
docker build --no-cache -t test-image .

# 6. Build with progress
docker build --progress=plain -t test-image .

# 7. Stop at specific stage (multi-stage)
docker build --target stage-name -t test-image .
```

### Error Analysis Pattern:

```
Error Message:
"unknown instruction: RUM"
         ↓
Line Analysis:
RUM apt-get update
         ↓
Problem Identification:
Typo - should be RUN
         ↓
Fix:
RUN apt-get update
         ↓
Rebuild:
docker build -t image .
```

---

## Understanding the Scenario

### Development Team's Challenge:

```
Nautilus DevOps Team
    ↓
Building custom Docker images
    ↓
Dockerfile created on App Server 3
    ↓
Docker build failing with errors
    ↓
Need to debug and fix without changing base image or data
```

### Typical Issues Found:

1. **Syntax typos** (RUM instead of RUN)
2. **Instruction order** (CMD before COPY)
3. **Path errors** (wrong file locations)
4. **Missing dependencies**
5. **Invalid expose syntax**
6. **Incorrect CMD format**

### Constraints:

```
✅ CAN DO:
├── Fix syntax errors
├── Correct instruction order
├── Update paths
├── Fix command formats
└── Add missing slashes/backslashes

❌ CANNOT DO:
├── Change base image
├── Modify index.html content
├── Remove valid configurations
├── Change application data
└── Alter functional requirements
```

---

## Infrastructure Overview

### Application Servers:
| Server | User | Password | IP |
|--------|------|----------|-----|
| **stapp01** | **tony** | **Ir0nM@n** | **172.16.238.10** |
| stapp02 | steve | Am3ric@ | 172.16.238.11 |
| stapp03 | banner | BigGr33n | 172.16.238.12 |

### Task Details:
| Item | Value |
|------|-------|
| Server | Application Server 1 (stapp01) |
| User | tony |
| Dockerfile Path | `/opt/docker/Dockerfile` |
| Build Context | `/opt/docker/` |
| Task | Debug and fix Dockerfile |
| Constraints | Keep base image, preserve data |

---

## Step-by-Step Implementation

### Step 1: SSH into Application Server 1
```bash
ssh tony@stapp01
```

**Expected output:**
```
The authenticity of host 'stapp01 (172.16.238.10)' can't be established.
ED25519 key fingerprint is SHA256:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'stapp01' (ED25519) to the list of known hosts.
tony@stapp01's password:
```

**Enter password:** `Ir0nM@n`

```
[tony@stapp01 ~]$
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

[sudo] password for tony:
```

**Enter password:** `Ir0nM@n`

```
[root@stapp01 ~]#
```

### Step 3: Navigate to Docker Directory
```bash
cd /opt/docker
```

**Verify location:**
```bash
pwd
```

**Expected output:**
```
/opt/docker
```

### Step 4: List Files in Build Context
```bash
ls -la
```

**Expected output (example):**
```
total 16
drwxr-xr-x 2 root root 4096 Dec 20 08:00 .
drwxr-xr-x 4 root root 4096 Dec 20 08:00 ..
-rw-r--r-- 1 root root  245 Dec 20 08:00 Dockerfile
-rw-r--r-- 1 root root  612 Dec 20 08:00 index.html
```

**✅ Both Dockerfile and index.html present**

### Step 5: View Current Dockerfile Content
```bash
cat Dockerfile
```

**Actual broken Dockerfile:**
```dockerfile
IMAGE httpd:2.4.43

ADD sed -i "s/Listen 80/Listen 8080/g" /usr/local/apache2/conf/httpd.conf

ADD sed -i '/LoadModule\ ssl_module modules\/mod_ssl.so/s/^#//g' conf/httpd.conf

ADD sed -i '/LoadModule\ socache_shmcb_module modules\/mod_socache_shmcb.so/s/^#//g' conf/httpd.conf

ADD sed -i '/Include\ conf\/extra\/httpd-ssl.conf/s/^#//g' conf/httpd.conf

COPY certs/server.crt /usr/local/apache2/conf/server.crt

COPY certs/server.key /usr/local/apache2/conf/server.key

COPY html/index.html /usr/local/apache2/htdocs/
```

**Identify issues:**
1. ❌ Line 1: `IMAGE` instead of `FROM` (invalid instruction)
2. ❌ Lines 3-9: `ADD` used for running sed commands (should be `RUN`)
3. ❌ Missing `CMD` or `ENTRYPOINT` to start Apache
4. ❌ Missing `EXPOSE` instruction for ports

### Step 6: Try Building to See Error
```bash
docker build -t test-image .
```

**Expected error output:**
```
Sending build context to Docker daemon  xyz kB
Step 1/10 : IMAGE httpd:2.4.43
unknown instruction: IMAGE
```

**✅ First error identified: `IMAGE` instead of `FROM`**

### Step 7: Create Backup of Original Dockerfile
```bash
cp Dockerfile Dockerfile.backup
```

**Verify backup:**
```bash
ls -la
```

**Expected output:**
```
total 20
drwxr-xr-x 2 root root 4096 Dec 20 09:00 .
drwxr-xr-x 4 root root 4096 Dec 20 08:00 ..
-rw-r--r-- 1 root root  245 Dec 20 08:00 Dockerfile
-rw-r--r-- 1 root root  245 Dec 20 09:00 Dockerfile.backup
-rw-r--r-- 1 root root  612 Dec 20 08:00 index.html
```

**✅ Backup created**

### Step 8: Fix Error #1 - IMAGE to FROM
```bash
sed -i 's/IMAGE httpd/FROM httpd/g' Dockerfile
```

**Verify fix:**
```bash
grep -n "FROM" Dockerfile
```

**Expected output:**
```
1:FROM httpd:2.4.43
```

**Alternative manual edit:**
```bash
vi Dockerfile
# Change line 1: IMAGE → FROM
# Save and exit (:wq)
```

### Step 9: Fix Error #2 - ADD to RUN (for sed commands)
```bash
sed -i 's/^ADD sed/RUN sed/g' Dockerfile
```

**Verify fix:**
```bash
grep -n "^RUN sed" Dockerfile
```

**Expected output:**
```
3:RUN sed -i "s/Listen 80/Listen 8080/g" /usr/local/apache2/conf/httpd.conf
5:RUN sed -i '/LoadModule\ ssl_module modules\/mod_ssl.so/s/^#//g' conf/httpd.conf
7:RUN sed -i '/LoadModule\ socache_shmcb_module modules\/mod_socache_shmcb.so/s/^#//g' conf/httpd.conf
9:RUN sed -i '/Include\ conf\/extra\/httpd-ssl.conf/s/^#//g' conf/httpd.conf
```

### Step 10: Add Missing EXPOSE Instruction
```bash
# Add EXPOSE before the end of file
echo "" >> Dockerfile
echo "EXPOSE 8080 443" >> Dockerfile
```

**Verify:**
```bash
tail -2 Dockerfile
```

**Expected output:**
```

EXPOSE 8080 443
```

### Step 11: Add Missing CMD Instruction
```bash
echo "" >> Dockerfile
echo 'CMD ["httpd-foreground"]' >> Dockerfile
```

**Verify:**
```bash
tail -2 Dockerfile
```

**Expected output:**
```

CMD ["httpd-foreground"]
```

### Step 11: View Corrected Dockerfile
```bash
cat Dockerfile
```

**Expected output (fixed):**
```dockerfile
FROM httpd:2.4.43

RUN sed -i "s/Listen 80/Listen 8080/g" /usr/local/apache2/conf/httpd.conf

RUN sed -i '/LoadModule\ ssl_module modules\/mod_ssl.so/s/^#//g' conf/httpd.conf

RUN sed -i '/LoadModule\ socache_shmcb_module modules\/mod_socache_shmcb.so/s/^#//g' conf/httpd.conf

RUN sed -i '/Include\ conf\/extra\/httpd-ssl.conf/s/^#//g' conf/httpd.conf

COPY certs/server.crt /usr/local/apache2/conf/server.crt

COPY certs/server.key /usr/local/apache2/conf/server.key

COPY html/index.html /usr/local/apache2/htdocs/

EXPOSE 8080 443

CMD ["httpd-foreground"]
```

**✅ All errors fixed:**
- ✅ IMAGE → FROM
- ✅ ADD sed commands → RUN sed commands
- ✅ Added EXPOSE 8080 443
- ✅ Added CMD ["httpd-foreground"]

### Step 12: Verify Required Files Exist
```bash
# Check if certs directory exists
ls -la certs/

# Check if html directory exists
ls -la html/
```

**Expected output:**
```
certs/:
total 16
drwxr-xr-x 2 root root 4096 Dec 20 08:00 .
drwxr-xr-x 4 root root 4096 Dec 20 08:00 ..
-rw-r--r-- 1 root root 1234 Dec 20 08:00 server.crt
-rw-r--r-- 1 root root 1678 Dec 20 08:00 server.key

html/:
total 12
drwxr-xr-x 2 root root 4096 Dec 20 08:00 .
drwxr-xr-x 4 root root 4096 Dec 20 08:00 ..
-rw-r--r-- 1 root root  612 Dec 20 08:00 index.html
```

**View index.html:**
```bash
cat html/index.html
```

**Expected output (example):**
```html
<!DOCTYPE html>
<html>
<head>
    <title>Welcome</title>
</head>
<body>
    <h1>Welcome to Nautilus</h1>
    <p>This is a custom Docker image</p>
</body>
</html>
```

**✅ All required files exist and unchanged**

### Step 13: Attempt Build Again
```bash
docker build -t nautilus-app .
```

**Expected output:**
```
Sending build context to Docker daemon  xyz kB
Step 1/10 : FROM httpd:2.4.43
 ---> 98f93cd0ec3b
Step 2/10 : RUN sed -i "s/Listen 80/Listen 8080/g" /usr/local/apache2/conf/httpd.conf
 ---> Running in a1b2c3d4e5f6
 ---> 1234567890ab
Step 3/10 : RUN sed -i '/LoadModule\ ssl_module modules\/mod_ssl.so/s/^#//g' conf/httpd.conf
 ---> Running in b2c3d4e5f6a1
 ---> 234567890abc
Step 4/10 : RUN sed -i '/LoadModule\ socache_shmcb_module modules\/mod_socache_shmcb.so/s/^#//g' conf/httpd.conf
 ---> Running in c3d4e5f6a1b2
 ---> 34567890abcd
Step 5/10 : RUN sed -i '/Include\ conf\/extra\/httpd-ssl.conf/s/^#//g' conf/httpd.conf
 ---> Running in d4e5f6a1b2c3
 ---> 4567890abcde
Step 6/10 : COPY certs/server.crt /usr/local/apache2/conf/server.crt
 ---> 567890abcdef
Step 7/10 : COPY certs/server.key /usr/local/apache2/conf/server.key
 ---> 67890abcdef1
Step 8/10 : COPY html/index.html /usr/local/apache2/htdocs/
 ---> 7890abcdef12
Step 9/10 : EXPOSE 8080 443
 ---> Running in e5f6a1b2c3d4
 ---> 890abcdef123
Step 10/10 : CMD ["httpd-foreground"]
 ---> Running in f6a1b2c3d4e5
 ---> 90abcdef1234
Successfully built 90abcdef1234
Successfully tagged nautilus-app:latest
```

**✅ Build successful!**

### Step 14: Verify Image Created
```bash
docker images | grep nautilus-app
```

**Expected output:**
```
nautilus-app   latest    7890abcdef12   2 minutes ago   245MB
```

**Alternative:**
```bash
docker images nautilus-app
```

**Expected output:**
```
REPOSITORY     TAG       IMAGE ID       CREATED         SIZE
nautilus-app   latest    7890abcdef12   2 minutes ago   245MB
```

### Step 15: Inspect Image Layers
```bash
docker history nautilus-app
```

**Expected output:**
```
IMAGE          CREATED         CREATED BY                                      SIZE      COMMENT
7890abcdef12   3 minutes ago   CMD ["apachectl" "-D" "FOREGROUND"]            0B        buildkit.dockerfile.v0
67890abcdef1   3 minutes ago   EXPOSE map[80/tcp:{} 443/tcp:{}]               0B        buildkit.dockerfile.v0
567890abcdef   3 minutes ago   COPY index.html /var/www/html/ # buildkit      612B      buildkit.dockerfile.v0
4567890abcde   4 minutes ago   RUN /bin/sh -c service apache2 start # bui...  12.3kB    buildkit.dockerfile.v0
34567890abcd   4 minutes ago   RUN /bin/sh -c apt-get install -y apache2 ...  65.2MB    buildkit.dockerfile.v0
234567890abc   5 minutes ago   RUN /bin/sh -c apt-get install -y git # bu...  87.5MB    buildkit.dockerfile.v0
1234567890ab   5 minutes ago   RUN /bin/sh -c apt-get update # buildkit       41.2MB    buildkit.dockerfile.v0
98f93cd0ec3b   2 years ago     /bin/sh -c #(nop)  CMD ["httpd-foreground"]    0B
...
```

**✅ All layers created successfully**

### Step 16: Test Container from Built Image
```bash
docker run -d --name test-nautilus -p 8080:8080 -p 8443:443 nautilus-app
```

**Expected output:**
```
a1b2c3d4e5f6a1b2c3d4e5f6a1b2c3d4e5f6a1b2c3d4e5f6a1b2c3d4e5f6
```

**Verify container running:**
```bash
docker ps | grep test-nautilus
```

**Expected output:**
```
a1b2c3d4e5f6   nautilus-app   "httpd-foreground"   10 seconds ago   Up 9 seconds   0.0.0.0:8080->8080/tcp, 0.0.0.0:8443->443/tcp   test-nautilus
```

### Step 17: Test Website Accessibility
```bash
curl http://localhost:8080
```

**Expected output:**
```html
<!DOCTYPE html>
<html>
<head>
    <title>Welcome</title>
</head>
<body>
    <h1>Welcome to Nautilus</h1>
    <p>This is a custom Docker image</p>
</body>
</html>
```

**✅ Website serving content from custom image!**

**Check HTTP headers:**
```bash
curl -I http://localhost:8080
```

**Expected output:**
```
HTTP/1.1 200 OK
Date: Fri, 20 Dec 2025 09:15:00 GMT
Server: Apache/2.4.43 (Unix)
Last-Modified: Fri, 20 Dec 2025 08:00:00 GMT
ETag: "264-6281e3b4a8e80"
Accept-Ranges: bytes
Content-Length: 612
Content-Type: text/html
```

**✅ Apache on port 8080 working!**

**Test HTTPS (if SSL configured):**
```bash
curl -k https://localhost:8443
```

### Step 18: View Container Logs
```bash
docker logs test-nautilus
```

**Expected output:**
```
AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 172.17.0.2. Set the 'ServerName' directive globally to suppress this message
AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 172.17.0.2. Set the 'ServerName' directive globally to suppress this message
[Fri Dec 20 09:10:00.123456 2025] [mpm_event:notice] [pid 1:tid 140123456789] AH00489: Apache/2.4.43 (Unix) configured -- resuming normal operations
[Fri Dec 20 09:10:00.123457 2025] [core:notice] [pid 1:tid 140123456789] AH00094: Command line: 'httpd -D FOREGROUND'
```

**✅ Apache httpd running successfully**

### Step 19: Clean Up Test Container
```bash
docker stop test-nautilus
docker rm test-nautilus
```

**Expected output:**
```
test-nautilus
test-nautilus
```

### Step 20: Final Verification
```bash
# 1. Dockerfile fixed and readable
cat Dockerfile

# 2. No syntax errors
grep -E "RUM|EXPOSE.*," Dockerfile
# Should return nothing

# 3. Image builds successfully
docker build -t final-test .

# 4. Image exists
docker images | grep final-test

# 5. Compare with backup
diff Dockerfile.backup Dockerfile
```

**Diff output (shows changes):**
```
1c1
< IMAGE httpd:2.4.43
---
> FROM httpd:2.4.43
3,9c3,9
< ADD sed -i "s/Listen 80/Listen 8080/g" /usr/local/apache2/conf/httpd.conf
< 
< ADD sed -i '/LoadModule\ ssl_module modules\/mod_ssl.so/s/^#//g' conf/httpd.conf
< 
< ADD sed -i '/LoadModule\ socache_shmcb_module modules\/mod_socache_shmcb.so/s/^#//g' conf/httpd.conf
< 
< ADD sed -i '/Include\ conf\/extra\/httpd-ssl.conf/s/^#//g' conf/httpd.conf
---
> RUN sed -i "s/Listen 80/Listen 8080/g" /usr/local/apache2/conf/httpd.conf
> 
> RUN sed -i '/LoadModule\ ssl_module modules\/mod_ssl.so/s/^#//g' conf/httpd.conf
> 
> RUN sed -i '/LoadModule\ socache_shmcb_module modules\/mod_socache_shmcb.so/s/^#//g' conf/httpd.conf
> 
> RUN sed -i '/Include\ conf\/extra\/httpd-ssl.conf/s/^#//g' conf/httpd.conf
15a16,19
> 
> EXPOSE 8080 443
> 
> CMD ["httpd-foreground"]
```

**✅ All fixes validated**

---

## Complete Command Summary

### Quick Fix Workflow:
```bash
# SSH and access
ssh tony@stapp01
sudo su -

# Navigate and backup
cd /opt/docker
cp Dockerfile Dockerfile.backup

# Fix errors
sed -i 's/IMAGE httpd/FROM httpd/g' Dockerfile
sed -i 's/^ADD sed/RUN sed/g' Dockerfile
echo "" >> Dockerfile
echo "EXPOSE 8080 443" >> Dockerfile
echo "" >> Dockerfile
echo 'CMD ["httpd-foreground"]' >> Dockerfile

# Build and verify
docker build -t nautilus-app .
docker images | grep nautilus-app

# Test (optional)
docker run -d --name test -p 8080:8080 -p 8443:443 nautilus-app
curl http://localhost:8080
docker stop test && docker rm test
```

### Detailed Debugging Workflow:
```bash
# 1. SSH and authenticate
ssh tony@stapp01
sudo su -

# 2. Navigate to Dockerfile location
cd /opt/docker
pwd

# 3. List build context
ls -la

# 4. View Dockerfile
cat Dockerfile

# 5. Attempt initial build (to see errors)
docker build -t test-image .

# 6. Create backup
cp Dockerfile Dockerfile.backup

# 7. Identify and fix errors
# Error 1: IMAGE → FROM
sed -i 's/IMAGE httpd/FROM httpd/g' Dockerfile

# Error 2: ADD → RUN (for sed commands)
sed -i 's/^ADD sed/RUN sed/g' Dockerfile

# Error 3: Add missing EXPOSE
echo "" >> Dockerfile
echo "EXPOSE 8080 443" >> Dockerfile

# Error 4: Add missing CMD
echo "" >> Dockerfile
echo 'CMD ["httpd-foreground"]' >> Dockerfile

# 8. Verify fixes
cat Dockerfile

# 9. Rebuild image
docker build -t nautilus-app .

# 10. Verify image created
docker images | grep nautilus-app

# 11. Test container
docker run -d --name test-app -p 8080:8080 -p 8443:443 nautilus-app
curl http://localhost:8080

# 12. Check logs
docker logs test-app

# 13. Clean up
docker stop test-app
docker rm test-app

# 14. View changes made
diff Dockerfile.backup Dockerfile
```

---

## Understanding Error Types

### Syntax Errors (Build Fails Immediately):

**1. Invalid Instruction:**
```dockerfile
RUM apt-get update     # RUM is not a valid instruction
```
**Error:** `unknown instruction: RUM`

**2. Missing Arguments:**
```dockerfile
FROM                    # Missing image name
```
**Error:** `FROM requires exactly one argument`

**3. Invalid Format:**
```dockerfile
EXPOSE 80,443          # Comma not allowed
```
**Error:** `invalid syntax in EXPOSE`

### Logical Errors (Build Succeeds, Runtime Fails):

**1. Wrong CMD Process:**
```dockerfile
CMD service apache2 start    # Service exits immediately
```
**Issue:** Container starts then stops

**2. Missing Files:**
```dockerfile
COPY missing.txt /app/   # File doesn't exist
```
**Error:** `COPY failed: stat ... no such file or directory`

**3. Wrong Paths:**
```dockerfile
COPY index.html /wrong/path/
```
**Issue:** File copied to wrong location, app can't find it

### Configuration Errors (Build Succeeds, Misconfigured):

**1. Wrong Base Image:**
```dockerfile
FROM ubuntu:latest       # But need httpd
```
**Issue:** Need to install apache2 manually

**2. Inefficient Layering:**
```dockerfile
RUN apt-get update
RUN apt-get install -y git
RUN apt-get install -y apache2
```
**Better:**
```dockerfile
RUN apt-get update && \
    apt-get install -y git apache2
```

---

## Dockerfile Best Practices

### 1. Instruction Order
```dockerfile
# Correct order
FROM image:tag           # 1. Base image first
LABEL maintainer="..."   # 2. Metadata
ENV VAR=value           # 3. Environment
WORKDIR /app            # 4. Working directory
COPY requirements.txt . # 5. Copy dependency files
RUN install deps        # 6. Install dependencies
COPY . .                # 7. Copy application
EXPOSE port             # 8. Document ports
CMD ["executable"]      # 9. Default command (last)
```

### 2. Minimize Layers
```dockerfile
# ❌ Bad (3 layers)
RUN apt-get update
RUN apt-get install -y git
RUN apt-get install -y apache2

# ✅ Good (1 layer)
RUN apt-get update && \
    apt-get install -y git apache2 && \
    rm -rf /var/lib/apt/lists/*
```

### 3. Use Exec Form for CMD
```dockerfile
# ❌ Shell form
CMD apachectl -D FOREGROUND
# PID 1 is shell, not apache

# ✅ Exec form
CMD ["apachectl", "-D", "FOREGROUND"]
# PID 1 is apache, proper signal handling
```

### 4. Layer Caching Strategy
```dockerfile
# Copy dependencies first (changes rarely)
COPY requirements.txt .
RUN pip install -r requirements.txt

# Copy application code last (changes often)
COPY . .
```

### 5. Use .dockerignore
```
# .dockerignore
node_modules/
.git/
*.log
.env
Dockerfile*
README.md
```

### 6. Specific Tags (Not latest)
```dockerfile
# ❌ Unpredictable
FROM ubuntu:latest

# ✅ Specific version
FROM ubuntu:22.04
```

### 7. Multi-Stage Builds (Advanced)
```dockerfile
# Build stage
FROM golang:1.19 AS builder
WORKDIR /app
COPY . .
RUN go build -o app

# Runtime stage
FROM alpine:3.17
COPY --from=builder /app/app /app
CMD ["/app"]
```

### 8. Security Best Practices
```dockerfile
# Don't run as root
FROM ubuntu:22.04
RUN groupadd -r appuser && useradd -r -g appuser appuser
USER appuser

# Update packages
RUN apt-get update && \
    apt-get upgrade -y && \
    apt-get install -y package && \
    rm -rf /var/lib/apt/lists/*
```

---

## Advanced Debugging Techniques

### 1. Build with Detailed Output
```bash
docker build --progress=plain -t image .
```

### 2. Stop at Specific Layer
```bash
# Build up to step 5
docker build --target=step5 -t debug-image .
```

### 3. Inspect Intermediate Layers
```bash
# Get intermediate container ID from build output
docker run -it <intermediate-id> /bin/bash
```

### 4. Check Build Context Size
```bash
# See what's being sent to Docker daemon
docker build --no-cache -t image . 2>&1 | grep "Sending build context"
```

### 5. Use BuildKit (Modern Docker)
```bash
# Enable BuildKit
export DOCKER_BUILDKIT=1
docker build -t image .
```

### 6. Lint Dockerfile
```bash
# Install hadolint
docker run --rm -i hadolint/hadolint < Dockerfile

# Or install locally
hadolint Dockerfile
```

### 7. Validate Before Building
```bash
# Check syntax
docker run --rm -i hadolint/hadolint < Dockerfile

# Check build context
ls -lh

# Verify source files exist
test -f index.html && echo "✅ index.html exists" || echo "❌ Missing"
```

---

## Common Scenarios and Solutions

### Scenario 1: Apache/Nginx Base Image
```dockerfile
# httpd (Apache)
FROM httpd:2.4
COPY index.html /usr/local/apache2/htdocs/
EXPOSE 80
CMD ["httpd-foreground"]

# nginx
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

### Scenario 2: Installing Packages
```dockerfile
# Ubuntu/Debian
FROM ubuntu:22.04
RUN apt-get update && \
    apt-get install -y package && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Alpine
FROM alpine:3.17
RUN apk add --no-cache package
```

### Scenario 3: Python Application
```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
EXPOSE 8000
CMD ["python", "app.py"]
```

### Scenario 4: Node.js Application
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
EXPOSE 3000
CMD ["node", "server.js"]
```

### Scenario 5: Multi-Service Container
```dockerfile
FROM ubuntu:22.04
RUN apt-get update && \
    apt-get install -y apache2 php libapache2-mod-php && \
    apt-get clean

COPY index.html /var/www/html/
COPY start.sh /start.sh
RUN chmod +x /start.sh

EXPOSE 80
CMD ["/start.sh"]
```

**start.sh:**
```bash
#!/bin/bash
service apache2 start
tail -f /var/log/apache2/access.log
```

---

## Troubleshooting Guide

### Issue 1: "unknown instruction" Error

**Error:**
```
unknown instruction: RUM
```

**Diagnosis:**
```bash
cat Dockerfile | grep -n "RUM"
```

**Solution:**
```bash
# Fix typo
sed -i 's/RUM/RUN/g' Dockerfile
```

**Valid Instructions:**
FROM, RUN, COPY, ADD, WORKDIR, ENV, EXPOSE, CMD, ENTRYPOINT, LABEL, USER, VOLUME, ARG, ONBUILD, STOPSIGNAL, HEALTHCHECK, SHELL

### Issue 2: "EXPOSE invalid syntax"

**Error:**
```
invalid syntax in EXPOSE instruction
```

**Diagnosis:**
```bash
grep -n "EXPOSE" Dockerfile
# Shows: EXPOSE 80,443
```

**Solution:**
```bash
# Space-separated, not comma
sed -i 's/EXPOSE 80,443/EXPOSE 80 443/g' Dockerfile
```

### Issue 3: "COPY failed: no such file"

**Error:**
```
COPY failed: stat /var/lib/docker/tmp/docker-builder123/file.txt: no such file or directory
```

**Diagnosis:**
```bash
# Check if file exists in build context
ls -la /opt/docker/
```

**Solution:**
```bash
# Ensure file is in same directory as Dockerfile
cp /path/to/file.txt /opt/docker/
```

### Issue 4: Container Exits Immediately

**Issue:**
Container starts then stops

**Diagnosis:**
```bash
docker logs container-name
docker inspect container-name | grep -A 10 "State"
```

**Common Causes:**
1. CMD runs background process
2. CMD exits immediately
3. Application error

**Solution:**
```dockerfile
# Ensure CMD runs foreground process
CMD ["apachectl", "-D", "FOREGROUND"]
# Or
CMD ["nginx", "-g", "daemon off;"]
```

### Issue 5: Permission Denied in Container

**Error:**
```
Permission denied
```

**Diagnosis:**
```bash
# Check file permissions
docker run -it image ls -la /path/to/file
```

**Solution:**
```dockerfile
# Add permissions in Dockerfile
COPY script.sh /script.sh
RUN chmod +x /script.sh
```

### Issue 6: Package Installation Fails

**Error:**
```
E: Unable to locate package
```

**Solution:**
```dockerfile
# Always update before install
RUN apt-get update && \
    apt-get install -y package
```

### Issue 7: Build Takes Forever

**Issue:**
Slow build performance

**Solution:**
```dockerfile
# Order from least to most frequently changed
COPY requirements.txt .
RUN install dependencies
COPY . .

# Use .dockerignore
# Create .dockerignore file
```

---

## Dockerfile Validation Checklist

Before building:

- [ ] FROM instruction is first (except ARG)
- [ ] All instruction names spelled correctly
- [ ] EXPOSE uses space-separated ports
- [ ] CMD/ENTRYPOINT in exec form `["cmd", "arg"]`
- [ ] All COPY source files exist
- [ ] Paths have trailing slashes for directories
- [ ] No multiple CMD instructions
- [ ] RUN commands use && or backslash continuation
- [ ] ENV variables use proper syntax
- [ ] WORKDIR paths are absolute or logical
- [ ] No ARG after FROM without redeclaration
- [ ] .dockerignore file exists (if needed)

After fixing:

- [ ] docker build completes successfully
- [ ] Image appears in `docker images`
- [ ] Test container runs without exit
- [ ] Application accessible on exposed ports
- [ ] Container logs show no errors
- [ ] Base image unchanged
- [ ] Data files unchanged
- [ ] All valid configurations preserved

---

## Key Commands Reference

| Command | Description |
|---------|-------------|
| `docker build -t name .` | Build image from Dockerfile |
| `docker build --no-cache -t name .` | Build without cache |
| `docker build --progress=plain .` | Detailed build output |
| `docker images` | List images |
| `docker history image` | View image layers |
| `docker run -it image bash` | Interactive container |
| `docker logs container` | View container logs |
| `docker inspect image` | Detailed image info |
| `cat Dockerfile` | View Dockerfile |
| `grep -n "pattern" Dockerfile` | Search Dockerfile |
| `diff file1 file2` | Compare files |

---

## Dockerfile Syntax Quick Reference

### Basic Structure:
```dockerfile
FROM image:tag
LABEL key="value"
ENV VAR=value
WORKDIR /path
COPY src dest
RUN command
EXPOSE port
CMD ["executable", "arg"]
```

### Common Instructions:

**FROM:**
```dockerfile
FROM ubuntu:22.04
FROM nginx:alpine
FROM python:3.11-slim
```

**RUN:**
```dockerfile
RUN apt-get update
RUN apt-get update && apt-get install -y git
RUN ["executable", "param1", "param2"]
```

**COPY:**
```dockerfile
COPY file.txt /app/
COPY . /app/
COPY --chown=user:group file.txt /app/
```

**CMD:**
```dockerfile
CMD ["executable", "param1"]
CMD ["nginx", "-g", "daemon off;"]
CMD ["python", "app.py"]
```

**EXPOSE:**
```dockerfile
EXPOSE 80
EXPOSE 80 443
EXPOSE 8080/tcp 8080/udp
```

**ENV:**
```dockerfile
ENV KEY=value
ENV KEY1=value1 KEY2=value2
ENV PATH=/app/bin:$PATH
```

**WORKDIR:**
```dockerfile
WORKDIR /app
WORKDIR /usr/local/apache2/htdocs
```

---

## Real-World Examples

### Example 1: Simple Web Server
```dockerfile
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

### Example 2: Python Flask App
```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
EXPOSE 5000
ENV FLASK_APP=app.py
CMD ["flask", "run", "--host=0.0.0.0"]
```

### Example 3: Node.js Express
```dockerfile
FROM node:18-alpine
WORKDIR /usr/src/app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
EXPOSE 3000
CMD ["node", "server.js"]
```

### Example 4: Apache with PHP
```dockerfile
FROM php:8.1-apache
COPY index.php /var/www/html/
COPY apache-config.conf /etc/apache2/sites-available/000-default.conf
RUN a2enmod rewrite
EXPOSE 80
CMD ["apache2-foreground"]
```

### Example 5: Custom httpd
```dockerfile
FROM httpd:2.4
RUN apt-get update && \
    apt-get install -y git && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
COPY index.html /usr/local/apache2/htdocs/
COPY httpd.conf /usr/local/apache2/conf/
EXPOSE 80 443
CMD ["httpd-foreground"]
```

---

## Completion Checklist

- [ ] SSH into Application Server 1 (stapp01) as tony
- [ ] Switched to root user
- [ ] Navigated to /opt/docker directory
- [ ] Viewed broken Dockerfile
- [ ] Identified syntax errors (IMAGE, ADD, missing EXPOSE/CMD)
- [ ] Created backup of original Dockerfile
- [ ] Fixed error: IMAGE → FROM
- [ ] Fixed error: ADD sed → RUN sed (for all sed commands)
- [ ] Added missing: EXPOSE 8080 443
- [ ] Added missing: CMD ["httpd-foreground"]
- [ ] Verified certs/ and html/ directories exist
- [ ] Docker build completed successfully
- [ ] Image created and visible in docker images
- [ ] Tested container from built image
- [ ] Website accessible on port 8080 (optional)
- [ ] Base image preserved (httpd:2.4.43)
- [ ] No data modifications made (certs, html files)
- [ ] All valid configurations maintained
- [ ] All requirements met ✅

---

## Completion Details

- **Completion Date:** December 20, 2025
- **Day:** 45 / 100
- **Challenge:** KodeKloud 100 Days Cloud DevOps
- **Topic:** Debugging Dockerfile Build Failures
- **Server:** Application Server 1 (stapp01)
- **User:** tony
- **Dockerfile Path:** `/opt/docker/Dockerfile`
- **Base Image:** httpd:2.4.43 (preserved)
- **Errors Fixed:**
  1. IMAGE → FROM (invalid instruction)
  2. ADD sed → RUN sed (wrong instruction for commands)
  3. Added EXPOSE 8080 443 (missing)
  4. Added CMD ["httpd-foreground"] (missing)
- **Key Skill:** Dockerfile debugging and Docker build troubleshooting
- **Status:** ✅ Successfully Completed

---

## Summary

This task demonstrated **Dockerfile debugging and build error resolution**:

✅ **Identified syntax errors** - IMAGE typo, ADD misuse, missing instructions
✅ **Fixed systematically** - One error at a time with verification
✅ **Preserved constraints** - Base image and data unchanged
✅ **Validated fixes** - Successful build and container test
✅ **Applied best practices** - Proper instruction usage, complete Dockerfile

**Key Insight:** Docker build errors are **systematic and predictable**. Most failures fall into three categories:

**1. Syntax Errors (Immediate Failure):**
```
IMAGE instead of FROM      → Fix instruction name
ADD for RUN commands       → Use correct instruction
Missing required EXPOSE    → Add port documentation
Missing CMD/ENTRYPOINT     → Add startup command
```

**2. Path Errors (Build Failure):**
```
COPY missing-file.txt      → Ensure file exists
WORKDIR /wrong/path        → Verify paths
ADD from invalid URL       → Check URL
```

**3. Configuration Errors (Runtime Failure):**
```
CMD service start          → Use foreground process
Wrong base image           → Match requirements
Missing dependencies       → Install packages
```

**Debugging Methodology:**
```
1. Read error message carefully
   ↓
2. Identify line number
   ↓
3. Check instruction syntax
   ↓
4. Verify file existence
   ↓
5. Fix one error at a time
   ↓
6. Rebuild and test
   ↓
7. Repeat until success
```

**Common Dockerfile Errors Fixed:**

| Error | Issue | Fix |
|-------|-------|-----|
| `IMAGE` | Invalid instruction | `FROM` |
| `ADD sed ...` | Wrong instruction for commands | `RUN sed ...` |
| Missing `EXPOSE` | No port documentation | `EXPOSE 8080 443` |
| Missing `CMD` | No startup command | `CMD ["httpd-foreground"]` |
| Missing source | File not found | Add file to build context |

**Best Practices Applied:**
- ✅ Create backup before editing
- ✅ Fix one error at a time
- ✅ Use exec form for CMD
- ✅ Space-separated EXPOSE
- ✅ Verify build before submission

**Exec Form vs Shell Form:**
```dockerfile
# Shell form (wraps in /bin/sh -c)
CMD apachectl -D FOREGROUND
├── PID 1: /bin/sh
└── PID 2: apache (child process)

# Exec form (direct execution)
CMD ["apachectl", "-D", "FOREGROUND"]
└── PID 1: apache (main process)
```

**Why Exec Form Matters:**
- Proper signal handling (SIGTERM, SIGKILL)
- Graceful shutdown
- Container orchestration compatibility
- No zombie processes

**Docker Build Process:**
```
Dockerfile
    ↓
Parser validates syntax
    ↓
Executes instructions sequentially
    ↓
Each instruction = new layer
    ↓
Layer cached for reuse
    ↓
Final layer = image
```

**Efficient Debugging:**
```bash
# Quick syntax check
grep -E "RUM|FROM.*RUN|EXPOSE.*," Dockerfile

# Build with full output
docker build --progress=plain -t test .

# Check specific error
docker build -t test . 2>&1 | grep -A 5 "Error"
```

**Remember:** Always **backup → identify → fix → verify → test** when debugging Dockerfiles. Systematic approach prevents multiple build failures! 🐳

**Real-World Impact:**
- Faster CI/CD pipelines (no broken builds)
- Reproducible images
- Team collaboration (working Dockerfiles)
- Production stability

**Next:** Build optimized multi-stage Dockerfiles, implement security scanning, and create production-grade container images! 🚀
