# Day 39: Docker Commit - Creating Images from Containers
**100 Days Cloud DevOps Challenge – KodeKloud**

---

## Objective

A Nautilus developer has made changes to a running container and wants to preserve those changes. The DevOps team needs to create a new Docker image from the modified container to keep a backup of the changes.

**Requirements:**
1. Work on Application Server 3
2. Source: Running container `ubuntu_latest`
3. Create new image: `demo:nautilus`
4. Preserve all changes made in the container

---

## Understanding Docker Commit

**Docker Commit** creates a new image from a container's changes. It captures the current state of a container (including file system changes) and saves it as a new Docker image.

### What is Docker Commit?

The `docker commit` command takes a running or stopped container and creates a new image layer containing all changes made since the container was created from its base image.

### Container → Image Flow:

```
Base Image (ubuntu:latest)
         ↓
    Container Created
         ↓
    Changes Made:
    - Files modified
    - Packages installed
    - Configs updated
         ↓
    docker commit
         ↓
    New Image (demo:nautilus)
```

### Why Create Images from Containers?

- **Backup Changes** - Preserve container modifications
- **Version Control** - Save different states
- **Share Work** - Distribute customized images
- **Quick Testing** - Save experimental configurations
- **Development** - Create development environments
- **Debugging** - Capture problematic state

### Docker Commit vs Dockerfile:

| Aspect | Docker Commit | Dockerfile |
|--------|--------------|------------|
| **Reproducibility** | ❌ Manual, not reproducible | ✅ Automated, reproducible |
| **Speed** | ✅ Fast, immediate | Slower, builds from scratch |
| **Documentation** | ❌ Changes not documented | ✅ All steps documented |
| **Best For** | Quick backups, testing | Production, version control |
| **Automation** | ❌ Manual process | ✅ CI/CD friendly |
| **Transparency** | ❌ Black box | ✅ Clear instructions |

---

## Understanding the Scenario

### Developer Workflow:

```
1. Developer starts ubuntu_latest container
2. Makes changes:
   - Installs packages
   - Modifies configuration
   - Creates files
   - Tests application
3. Wants to save these changes
4. DevOps creates image from container
5. Image can be used to recreate container with changes
```

### Use Case Example:

**Developer's Container Changes:**
```bash
# Started with ubuntu:latest
docker run -it --name ubuntu_latest ubuntu:latest

# Inside container, made changes:
apt-get update
apt-get install -y nginx vim curl
echo "Custom config" > /etc/myapp.conf
mkdir /app && cd /app

# Now wants to save this state
```

**DevOps Action:**
```bash
# Create image from container
docker commit ubuntu_latest demo:nautilus

# Image now contains all changes
# Can be used to start new containers with same setup
```

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
| Source Container | `ubuntu_latest` (running) |
| New Image Name | `demo:nautilus` |
| Operation | docker commit |
| State | Preserve all changes |

---

## Understanding the Task

### What We're Creating:

```
Application Server 3 (stapp03)
│
├── Running Container: ubuntu_latest
│   ├── Based on: ubuntu:latest
│   ├── Changes: Developer's modifications
│   └── State: Running
│
└── docker commit
    ↓
    New Image: demo:nautilus
    ├── Contains: All container changes
    ├── Type: Custom image
    └── Use: Can create new containers
```

### Image Creation Flow:

```
1. Verify ubuntu_latest container is running
2. Check container changes/modifications
3. Execute docker commit command
4. Verify new image created
5. Test new image (optional)
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
ED25519 key fingerprint is SHA256:4Ydosz8hwqwmULScJMpoT6V2Fry028FHTOxtFB9Vd80.
This key is not known by any other names
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

### Step 3: List Running Containers
```bash
docker ps
```

**Expected output:**
```
CONTAINER ID   IMAGE     COMMAND       CREATED         STATUS         PORTS     NAMES
e3b9df43a5fe   ubuntu    "/bin/bash"   4 minutes ago   Up 4 minutes             ubuntu_latest
```

**Verify:**
- ✅ Container `ubuntu_latest` is present
- ✅ STATUS shows "Up" (running)
- ✅ Based on `ubuntu` image (ubuntu:latest)

### Step 4: Inspect Container Details
```bash
docker inspect ubuntu_latest
```

**Expected output (partial):**
```json
[
    {
        "Id": "a1b2c3d4e5f6...",
        "Name": "/ubuntu_latest",
        "State": {
            "Status": "running",
            "Running": true,
            ...
        },
        "Image": "sha256:3b418d7b466a...",
        "Config": {
            "Image": "ubuntu:latest",
            ...
        }
    }
]
```

### Step 5: Check Container Changes (Optional)
```bash
docker diff ubuntu_latest
```

**Expected output (example):**
```
C /usr
C /usr/src
A /usr/src/welcome.txt
C /var
C /var/lib
C /var/lib/apt
C /var/lib/apt/lists
A /var/lib/apt/lists/archive.ubuntu.com_ubuntu_dists_noble-updates_universe_binary-amd64_Packages.lz4
A /var/lib/apt/lists/archive.ubuntu.com_ubuntu_dists_noble_restricted_binary-amd64_Packages.lz4
A /var/lib/apt/lists/security.ubuntu.com_ubuntu_dists_noble-security_main_binary-amd64_Packages.lz4
A /var/lib/apt/lists/security.ubuntu.com_ubuntu_dists_noble-security_multiverse_binary-amd64_Packages.lz4
...
```

**Legend:**
- `A` = Added file/directory
- `C` = Changed file/directory
- `D` = Deleted file/directory

### Step 6: View Container Processes (Optional)
```bash
docker top ubuntu_latest
```

**Expected output:**
```
UID                 PID                 PPID                C                   STIME               TTY                 TIME                CMD
root                2338                2317                0                   17:07               pts/0               00:00:00            /bin/bash
```

### Step 7: Check Current Images
```bash
docker images
```

**Expected output (may vary):**
```
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
ubuntu       latest    c3a134f2ace4   8 weeks ago   78.1MB
```

**Check if demo:nautilus already exists:**
```bash
docker images | grep demo
```

**Should be empty initially**

### Step 8: Create Image from Container
```bash
docker commit ubuntu_latest demo:nautilus
```

**Expected output:**
```
sha256:c8615be5c5676911ec5569f6c5620360e543f3c1452734dc67ff9d3f7ecaf2e2
```

**This is the new image ID (yours will differ)**

**Command breakdown:**
- `docker commit` - Create image from container
- `ubuntu_latest` - Source container name
- `demo:nautilus` - New image name (repository:tag)

### Step 9: Verify New Image Created
```bash
docker images
```

**Expected output:**
```
REPOSITORY   TAG       IMAGE ID       CREATED          SIZE
demo         nautilus  c8615be5c567   10 seconds ago   135MB
ubuntu       latest    c3a134f2ace4   8 weeks ago      78.1MB
```

**Verify:**
- ✅ Repository: `demo`
- ✅ Tag: `nautilus`
- ✅ Just created: "10 seconds ago"
- ✅ Size: 135MB (larger than ubuntu base due to changes)

### Step 10: Check Specific Image
```bash
docker images demo:nautilus
```

**Expected output:**
```
REPOSITORY   TAG        IMAGE ID       CREATED          SIZE
demo         nautilus   c8615be5c567   21 seconds ago   135MB
```

### Step 11: Inspect New Image
```bash
docker inspect demo:nautilus
```

**Expected output (partial):**
```json
[
    {
        "Id": "sha256:c8615be5c5676911ec5569f6c5620360e543f3c1452734dc67ff9d3f7ecaf2e2",
        "RepoTags": [
            "demo:nautilus"
        ],
        "Parent": "sha256:c3a134f2ace4f6d480733efcfef27c60ea8ed48be1cd36f2c17ec0729775b2c8",
        "Created": "2025-12-14T17:12:13.833438697Z",
        "Size": 135352262,
        "Config": {
            "Image": "ubuntu",
            ...
        }
    }
]
```

**Notice the "Parent" field - links to ubuntu:latest**

### Step 12: View Image History
```bash
docker history demo:nautilus
```

**Expected output:**
```
IMAGE          CREATED          CREATED BY                                      SIZE
c8615be5c567   41 seconds ago   /bin/bash                                       57.2MB    
c3a134f2ace4   8 weeks ago      /bin/sh -c #(nop)  CMD ["/bin/bash"]            0B        
<missing>      8 weeks ago      /bin/sh -c #(nop) ADD file:ddf1aa62235de665…   78.1MB    
<missing>      8 weeks ago      /bin/sh -c #(nop)  LABEL org.opencontainers…   0B        
<missing>      8 weeks ago      /bin/sh -c #(nop)  LABEL org.opencontainers…   0B        
<missing>      8 weeks ago      /bin/sh -c #(nop)  ARG LAUNCHPAD_BUILD_ARCH     0B        
<missing>      8 weeks ago      /bin/sh -c #(nop)  ARG RELEASE                  0B        
```

**Top layer (57.2MB) is the committed changes**

### Step 13: Compare Image Sizes
```bash
docker images --format "table {{.Repository}}:{{.Tag}}\t{{.Size}}" | grep -E "demo|ubuntu"
```

**Expected output:**
```
demo:nautilus           135MB
ubuntu:latest           78.1MB
```

**Size is larger because of the apt updates and welcome.txt file added**

### Step 14: Test New Image (Optional)
```bash
docker run --rm demo:nautilus cat /etc/os-release
```

**Expected output:**
```
PRETTY_NAME="Ubuntu 24.04.3 LTS"
NAME="Ubuntu"
VERSION_ID="24.04"
VERSION="24.04.3 LTS (Noble Numbat)"
VERSION_CODENAME=noble
ID=ubuntu
ID_LIKE=debian
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
UBUNTU_CODENAME=noble
LOGO=ubuntu-logo
```

**Confirms the image works! Ubuntu 24.04.3 LTS (Noble Numbat)**

### Step 15: Test with Interactive Shell (Optional)
```bash
docker run -it --rm demo:nautilus /bin/bash
```

**Inside container:**
```bash
# Check if developer's changes are present
ls -la /

# Check the welcome file that was created
cat /usr/src/welcome.txt

# Exit container
exit
```

### Step 16: Final Verification
```bash
# List all images
docker images

# Check demo:nautilus specifically
docker images demo

# Verify tag
docker images --filter "reference=demo:nautilus"
```

**Expected output:**
```
REPOSITORY   TAG        IMAGE ID       CREATED          SIZE
demo         nautilus   c8615be5c567   5 minutes ago    135MB
```

**Perfect! Image created successfully ✅**

**Key Details:**
- Image ID: `c8615be5c567`
- Size: 135MB (57.2MB of changes added to 78.1MB base)
- Contains: apt package lists + /usr/src/welcome.txt

---

## Complete Command Summary

### Quick Image Creation:
```bash
# SSH to server
ssh banner@stapp03

# Switch to root
sudo su -

# Verify container running
docker ps | grep ubuntu_latest

# Create image from container
docker commit ubuntu_latest demo:nautilus

# Verify
docker images demo:nautilus
```

### Detailed Workflow:
```bash
# SSH and authenticate
ssh banner@stapp03
sudo su -

# Check running containers
docker ps

# View container details
docker inspect ubuntu_latest

# Check what changed (optional)
docker diff ubuntu_latest

# View current images
docker images

# Create new image
docker commit ubuntu_latest demo:nautilus

# Verify creation
docker images

# Check new image details
docker inspect demo:nautilus

# View image history
docker history demo:nautilus

# Test image (optional)
docker run --rm demo:nautilus echo "Image works!"
```

---

## Understanding Docker Commit Command

### Syntax:
```bash
docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]
```

### Basic Usage:

**1. Simple Commit:**
```bash
docker commit container_name new_image
# Creates: new_image:latest
```

**2. With Tag:**
```bash
docker commit ubuntu_latest demo:nautilus
# Creates: demo:nautilus
```

**3. With Message:**
```bash
docker commit -m "Added nginx and custom config" ubuntu_latest demo:nautilus
```

**4. With Author:**
```bash
docker commit -a "Developer Name <dev@example.com>" ubuntu_latest demo:nautilus
```

**5. With Changes:**
```bash
docker commit --change='CMD ["nginx", "-g", "daemon off;"]' ubuntu_latest demo:nginx
```

**6. Pause Container During Commit:**
```bash
docker commit -p ubuntu_latest demo:nautilus
# -p pauses container during commit (default)
```

**7. Don't Pause:**
```bash
docker commit --pause=false ubuntu_latest demo:nautilus
```

### Common Options:

| Option | Description | Example |
|--------|-------------|---------|
| `-a, --author` | Author info | `--author "John <john@example.com>"` |
| `-m, --message` | Commit message | `--message "Added nginx"` |
| `-p, --pause` | Pause during commit | `--pause=true` (default) |
| `-c, --change` | Apply Dockerfile instruction | `--change "EXPOSE 80"` |

---

## Docker Commit Advanced Usage

### Commit with Message and Author:
```bash
docker commit \
  -a "John Doe <john@nautilus.com>" \
  -m "Installed nginx, vim, and custom configs" \
  ubuntu_latest \
  demo:nautilus
```

**Output:**
```
sha256:7c8d9e1f2a3b4c5d6e7f8a9b0c1d2e3f4a5b6c7d8e9f0a1b2c3d4e5f6a7b8c9
```

### Commit with Dockerfile Instructions:
```bash
docker commit \
  --change='ENV DEBUG=true' \
  --change='EXPOSE 80' \
  --change='CMD ["nginx", "-g", "daemon off;"]' \
  ubuntu_latest \
  demo:webserver
```

**This adds metadata to the image**

### Commit Stopped Container:
```bash
# Stop container
docker stop ubuntu_latest

# Commit still works
docker commit ubuntu_latest demo:nautilus

# Start container again
docker start ubuntu_latest
```

---

## Understanding Container Changes

### View Changes:
```bash
docker diff ubuntu_latest
```

**Output Explanation:**
```
A /app                    # Added directory
A /app/config.json        # Added file
C /etc                    # Changed directory
C /etc/nginx              # Changed directory
A /etc/nginx/nginx.conf   # Added file
D /tmp/old-file.txt       # Deleted file
```

### Change Types:

| Symbol | Meaning | Description |
|--------|---------|-------------|
| `A` | Added | New file or directory created |
| `C` | Changed | Modified file or directory |
| `D` | Deleted | Removed file or directory |

### What Gets Committed:

**✅ Included:**
- File system changes
- Installed packages
- Modified configurations
- Created files/directories
- Environment variables (if changed with --change)

**❌ Not Included:**
- Running processes
- Port mappings
- Volume mounts
- Network settings
- Container name
- Container logs

---

## Troubleshooting

### Issue 1: Container Not Found

**Problem:**
```
Error: No such container: ubuntu_latest
```

**Solution:**
```bash
# List all containers (including stopped)
docker ps -a

# Check correct name
docker ps -a --format "{{.Names}}"

# If stopped, you can still commit
docker commit ubuntu_latest demo:nautilus

# If doesn't exist at all
docker run -d --name ubuntu_latest ubuntu:latest sleep 3600
docker commit ubuntu_latest demo:nautilus
```

### Issue 2: Image Name Already Exists

**Problem:**
```
# No error, but want to replace
```

**Solution:**
```bash
# docker commit overwrites by default

# Or remove old image first
docker rmi demo:nautilus

# Then commit
docker commit ubuntu_latest demo:nautilus
```

### Issue 3: Invalid Image Name

**Problem:**
```
Error parsing reference: "Demo:Nautilus" is not a valid repository/tag
```

**Solution:**
```bash
# ❌ Uppercase not allowed
docker commit ubuntu_latest Demo:Nautilus

# ✅ Use lowercase
docker commit ubuntu_latest demo:nautilus

# ✅ Also valid
docker commit ubuntu_latest demo:nautilus-v1
docker commit ubuntu_latest demo:nautilus_backup
```

### Issue 4: Permission Denied

**Problem:**
```
Got permission denied while trying to connect to the Docker daemon socket
```

**Solution:**
```bash
# Use sudo
sudo docker commit ubuntu_latest demo:nautilus

# Or switch to root
sudo su -
docker commit ubuntu_latest demo:nautilus
```

### Issue 5: Container is Paused

**Problem:**
Container paused during commit takes too long

**Solution:**
```bash
# Don't pause during commit (faster but may be inconsistent)
docker commit --pause=false ubuntu_latest demo:nautilus

# Or ensure container is not busy
docker stats ubuntu_latest --no-stream
```

### Issue 6: Large Image Size

**Problem:**
Committed image is very large

**Solution:**
```bash
# Check what changed
docker diff ubuntu_latest

# Remove unnecessary files in container first
docker exec ubuntu_latest rm -rf /tmp/* /var/tmp/*
docker exec ubuntu_latest apt-get clean

# Then commit
docker commit ubuntu_latest demo:nautilus

# Check size
docker images demo:nautilus

# Consider using Dockerfile instead for production
```

---

## Docker Commit vs Dockerfile

### When to Use Docker Commit:

**✅ Good For:**
- Quick backups
- Saving experimental changes
- Development/testing
- Capturing problematic state
- One-off customizations
- Learning and exploration

**Example Scenarios:**
```bash
# Scenario 1: Developer testing
docker run -it ubuntu:latest
# Install and test packages
# Exit and save
docker commit container_id demo:test

# Scenario 2: Backup before changes
docker commit prod_container prod_backup:$(date +%Y%m%d)

# Scenario 3: Capture bug state
docker commit buggy_container debug:issue-123
```

### When to Use Dockerfile:

**✅ Good For:**
- Production images
- Reproducible builds
- Version control
- Documentation
- CI/CD pipelines
- Team collaboration

**Example Dockerfile:**
```dockerfile
FROM ubuntu:latest

RUN apt-get update && \
    apt-get install -y nginx vim curl && \
    apt-get clean

COPY config.json /etc/myapp.conf

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

### Comparison:

| Aspect | docker commit | Dockerfile |
|--------|--------------|------------|
| **Reproducibility** | ❌ Manual | ✅ Automated |
| **Documentation** | ❌ No history | ✅ All steps visible |
| **Version Control** | ❌ Binary blob | ✅ Text file in git |
| **Speed** | ✅ Instant | ⚠️ Build time |
| **Best Practice** | ⚠️ Dev only | ✅ Production |
| **Debugging** | ✅ Easy to capture | ⚠️ Need to rebuild |

---

## Image Management After Commit

### Tagging Committed Images:
```bash
# Create additional tags
docker tag demo:nautilus demo:latest
docker tag demo:nautilus demo:v1.0
docker tag demo:nautilus demo:backup-$(date +%Y%m%d)
```

### Pushing to Registry:
```bash
# Tag for registry
docker tag demo:nautilus myregistry.com/demo:nautilus

# Push to registry
docker push myregistry.com/demo:nautilus
```

### Creating Containers from New Image:
```bash
# Run new container from committed image
docker run -d --name test_demo demo:nautilus

# Interactive session
docker run -it --rm demo:nautilus /bin/bash

# With custom command
docker run --rm demo:nautilus ls -la /app
```

### Exporting Image:
```bash
# Save image to tar file
docker save demo:nautilus -o demo-nautilus.tar

# Load on another system
docker load -i demo-nautilus.tar
```

---

## Best Practices

### 1. Add Commit Messages
```bash
# Document what changed
docker commit -m "Installed nginx v1.25, added custom config" ubuntu_latest demo:nautilus
```

### 2. Include Author Information
```bash
docker commit \
  -a "Developer Name <dev@nautilus.com>" \
  -m "Development environment setup" \
  ubuntu_latest demo:nautilus
```

### 3. Clean Before Committing
```bash
# Remove temporary files
docker exec ubuntu_latest bash -c "rm -rf /tmp/* /var/tmp/*"
docker exec ubuntu_latest bash -c "apt-get clean"

# Then commit
docker commit ubuntu_latest demo:nautilus
```

### 4. Use Descriptive Names
```bash
# ✅ Good
docker commit ubuntu_latest demo:nautilus-dev-v1
docker commit ubuntu_latest demo:nginx-configured
docker commit ubuntu_latest demo:backup-20251214

# ❌ Vague
docker commit ubuntu_latest demo:new
docker commit ubuntu_latest demo:test
```

### 5. Document Changes
```bash
# Create log file
echo "Changes made: installed nginx, vim, curl" > /tmp/commit-log.txt
echo "Date: $(date)" >> /tmp/commit-log.txt
echo "By: Developer" >> /tmp/commit-log.txt

docker commit -m "$(cat /tmp/commit-log.txt)" ubuntu_latest demo:nautilus
```

### 6. Test Before Production
```bash
# Create committed image
docker commit ubuntu_latest demo:nautilus

# Test it
docker run --rm demo:nautilus nginx -v

# If good, tag for production
docker tag demo:nautilus demo:production
```

### 7. Regular Backups
```bash
# Daily backup script
#!/bin/bash
DATE=$(date +%Y%m%d)
docker commit ubuntu_latest demo:backup-$DATE
docker images | grep demo:backup | tail -n +8 | awk '{print $3}' | xargs docker rmi
```

---

## Real-World Scenarios

### Scenario 1: Developer Workflow
```bash
# Day 1: Start fresh
docker run -d --name dev_container ubuntu:latest

# Day 2-5: Make changes
docker exec dev_container apt-get update
docker exec dev_container apt-get install -y python3 pip

# End of Week: Save progress
docker commit dev_container demo:dev-week1
```

### Scenario 2: Emergency Backup
```bash
# Production container having issues
docker ps | grep prod

# Quick backup before troubleshooting
docker commit prod_container prod:backup-emergency-$(date +%Y%m%d-%H%M)

# Now safe to troubleshoot
docker exec prod_container systemctl restart service
```

### Scenario 3: A/B Testing
```bash
# Original version
docker commit app_container demo:version-a

# Make experimental changes
docker exec app_container configure_new_feature.sh

# Save experimental version
docker commit app_container demo:version-b

# Test both
docker run -p 8080:80 demo:version-a
docker run -p 8081:80 demo:version-b
```

### Scenario 4: Share Configuration
```bash
# Developer configures environment
docker run -it --name setup ubuntu:latest
# ... manual setup ...

# Create shareable image
docker commit setup demo:team-env

# Push to registry
docker tag demo:team-env registry.company.com/demo:team-env
docker push registry.company.com/demo:team-env

# Team members pull and use
docker pull registry.company.com/demo:team-env
docker run -it registry.company.com/demo:team-env
```

### Scenario 5: Incremental Development
```bash
# Base setup
docker commit dev_container demo:base

# Add database
docker exec dev_container apt-get install -y postgresql
docker commit dev_container demo:base-db

# Add web server
docker exec dev_container apt-get install -y nginx
docker commit dev_container demo:base-db-web

# Final with app
docker exec dev_container deploy_app.sh
docker commit dev_container demo:complete
```

---

## Key Commands Reference

| Command | Description |
|---------|-------------|
| `docker commit CONTAINER IMAGE` | Create image from container |
| `docker commit -m "msg" CONTAINER IMAGE` | With commit message |
| `docker commit -a "author" CONTAINER IMAGE` | With author info |
| `docker commit -p CONTAINER IMAGE` | Pause during commit |
| `docker commit --change CMD CONTAINER IMAGE` | Apply Dockerfile instruction |
| `docker diff CONTAINER` | View container changes |
| `docker inspect IMAGE` | Image details |
| `docker history IMAGE` | Image layer history |
| `docker images IMAGE` | List specific image |
| `docker tag IMAGE NEW_IMAGE` | Create additional tag |

---

## Completion Checklist

- [ ] SSH into Application Server 3 (stapp03) as banner
- [ ] Switched to root user
- [ ] Verified ubuntu_latest container is running
- [ ] Checked container status and details
- [ ] Viewed container changes (optional)
- [ ] Checked existing images
- [ ] Executed docker commit command
- [ ] Created image named demo:nautilus
- [ ] Verified new image exists in image list
- [ ] Confirmed image repository is "demo"
- [ ] Confirmed image tag is "nautilus"
- [ ] Inspected new image details
- [ ] Viewed image history
- [ ] Tested new image (optional)
- [ ] Image successfully created ✅

---

## Completion Details

- **Completion Date:** December 14, 2025
- **Day:** 39 / 100
- **Challenge:** KodeKloud 100 Days Cloud DevOps
- **Topic:** Docker Commit - Creating Images from Containers
- **Server:** Application Server 3 (stapp03)
- **User:** banner
- **Source Container:** `ubuntu_latest` (running)
- **New Image:** `demo:nautilus`
- **Image ID:** `c8615be5c567`
- **Base Image:** ubuntu:latest (Ubuntu 24.04.3 LTS)
- **Image Size:** 135MB
- **Changes Size:** 57.2MB
- **Operation:** docker commit
- **Key Skill:** Container state preservation and custom image creation
- **Status:** ✅ Successfully Completed

---

## Summary

This task demonstrated **creating Docker images from running containers**:

✅ **Verified container running** - ubuntu_latest container active
✅ **Used docker commit** - Preserved all container changes
✅ **Created custom image** - demo:nautilus with all modifications
✅ **Verified creation** - Image appears in docker images list
✅ **Understood workflow** - Container → Image conversion

**Key Insight:** Docker commit provides a way to **capture container state** as an image. This is useful for:
- **Quick backups** - Save work before risky changes
- **Development** - Preserve environment setup
- **Debugging** - Capture problematic state
- **Sharing** - Distribute customized environments
- **Testing** - Save experimental configurations

**The Process:**
```
Container with Changes → docker commit → New Image → Can create new containers
```

**When Developer Makes Changes:**
```bash
1. Container runs (ubuntu_latest)
2. Developer installs packages, configures apps
3. DevOps commits: docker commit ubuntu_latest demo:nautilus
4. Image saved with all changes
5. Can recreate environment: docker run demo:nautilus
```

**Important Considerations:**
- ✅ Fast and easy for backups
- ✅ Preserves all file system changes
- ⚠️ Not reproducible (manual process)
- ⚠️ Changes not documented
- ⚠️ Not ideal for production (use Dockerfile instead)

**Best Use Cases:**
- Development environment snapshots
- Emergency backups
- Debugging containers
- Quick testing
- Learning Docker

**Production Alternative:**
For production, convert manual changes to Dockerfile:
```dockerfile
FROM ubuntu:latest
RUN apt-get update && apt-get install -y package1 package2
COPY config.conf /etc/app/
CMD ["application"]
```

**Remember:** `docker commit container image:tag` = Instant Backup of Container State! 🐳

**The Developer's Friend:** When a developer says "I've configured everything perfectly in this container," docker commit captures that perfect state for reuse!

**Next:** Building images with Dockerfiles for reproducible, production-ready images! 🚀
