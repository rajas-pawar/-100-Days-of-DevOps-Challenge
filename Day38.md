# Day 38: Docker Image Tagging
**100 Days Cloud DevOps Challenge – KodeKloud**

---

## Objective

Nautilus project developers want to test containerized application features. The DevOps team needs to prepare a Docker image with a custom tag for their testing environment.

**Requirements:**
1. Work on App Server 3 in Stratos Datacenter
2. Pull `busybox:musl` image from Docker Hub
3. Re-tag the image as `busybox:media`
4. Verify the new tag exists

---

## Understanding Docker Image Tags

**Docker Image Tag** is a label applied to a Docker image to identify specific versions or variants. Tags enable version control and help manage multiple versions of the same image.

### Image Naming Convention:

```
[registry/][username/]repository:tag

Examples:
nginx:latest               → Official nginx, latest tag
nginx:1.25-alpine         → Nginx version 1.25, alpine variant
docker.io/library/nginx:latest → Full reference
myregistry.com/myapp:v2.0 → Private registry
```

### Tag Components:

| Component | Description | Example |
|-----------|-------------|---------|
| **Repository** | Image name | `busybox`, `nginx`, `ubuntu` |
| **Tag** | Version/variant label | `latest`, `musl`, `alpine`, `v1.0` |
| **Registry** | Image source | `docker.io` (default), `gcr.io`, `quay.io` |

### Why Use Tags?

- **Version Control** - Track different versions
- **Variant Management** - Different builds (alpine, musl, slim)
- **Environment Separation** - dev, staging, prod tags
- **Rollback Capability** - Revert to previous versions
- **Testing** - Separate test and production images
- **Documentation** - Self-documenting image versions

---

## Understanding BusyBox

**BusyBox** is a single executable that combines tiny versions of many common UNIX utilities. It's designed for embedded systems and containers.

### What is BusyBox?

- **Size:** ~1-2MB (extremely lightweight)
- **Purpose:** Minimal utility collection
- **Contains:** 300+ common UNIX commands
- **Use Cases:** Debugging, testing, minimal containers

### BusyBox Variants:

| Tag | Size | C Library | Use Case |
|-----|------|-----------|----------|
| `busybox:latest` | ~4MB | glibc | Default, most compatible |
| `busybox:musl` | ~1.5MB | musl libc | Smaller, Alpine-like |
| `busybox:glibc` | ~4MB | glibc | GNU C library |
| `busybox:uclibc` | ~1MB | uClibc | Ultra-minimal |

### BusyBox:musl Specifics:

**musl libc** is a lightweight, fast, and standards-compliant C library.

**Characteristics:**
- ✅ Small size (~1.5MB)
- ✅ Fast execution
- ✅ Simple and secure
- ✅ Used in Alpine Linux
- ✅ Good for containers

**Common Uses:**
- Debugging containers
- Testing network connectivity
- Minimal base images
- Init containers in Kubernetes
- Utility sidecar containers

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
| Server | App Server 3 (stapp03) |
| Source Image | `busybox:musl` |
| New Tag | `busybox:media` |
| Registry | Docker Hub (default) |
| Operation | Pull + Re-tag |

---

## Understanding the Task

### Image Tagging Flow:

```
Docker Hub                    App Server 3
┌──────────────────┐         ┌────────────────────────────┐
│                  │         │                            │
│ busybox:musl     │────────→│ busybox:musl (pulled)      │
│ (official)       │  pull   │         ↓                  │
│                  │         │    docker tag               │
└──────────────────┘         │         ↓                  │
                             │ busybox:media (new tag)    │
                             │                            │
                             │ Same image, two tags       │
                             └────────────────────────────┘
```

### What Happens When Re-Tagging:

1. **Pull** `busybox:musl` from Docker Hub
2. **Create reference** with new tag name
3. **Same image ID** - No duplication
4. **Two tags** pointing to same image layers
5. **No storage overhead** - Just metadata

---

## Step-by-Step Implementation

### Step 1: SSH into App Server 3
```bash
ssh banner@stapp03
```

**Expected output:**
```
The authenticity of host 'stapp03 (172.16.238.12)' can't be established.
ED25519 key fingerprint is SHA256:V+3Rx9jebLVfBIWIXvd5osfwUnXzpalzgfQLd0Gdt0M.
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

**Note:** Docker commands will now work without `sudo` prefix

### Step 3: Check Current Docker Images
```bash
docker images
```

**Expected output (may be empty initially):**
```
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
```

**Check if busybox already exists:**
```bash
docker images | grep busybox
```

### Step 4: Pull busybox:musl Image
```bash
docker pull busybox:musl
```

**Expected output:**
```
musl: Pulling from library/busybox
5bfa213ad291: Pull complete 
Digest: sha256:b259afe60d4b88dbdb31908ca9524ef5308afd01aea17f4ce44ddb3c6a882929
Status: Downloaded newer image for busybox:musl
docker.io/library/busybox:musl
```

**Note the digest** - This uniquely identifies the image

### Step 5: Verify Image Downloaded
```bash
docker images busybox
```

**Expected output:**
```
REPOSITORY   TAG    IMAGE ID       CREATED         SIZE
busybox      musl   1e2006b46d8b   14 months ago   1.51MB
```

**Key information:**
- Repository: `busybox`
- Tag: `musl`
- Size: ~1.51MB
- Image ID: `1e2006b46d8b` (yours may differ)

### Step 6: Inspect the Image
```bash
sudo docker inspect busybox:musl
```

**Expected output (partial):**
```json
[
    {
        "Id": "sha256:3d24ee258efc...",
        "RepoTags": [
            "busybox:musl"
        ],
        "RepoDigests": [
            "busybox@sha256:5acba83a746c..."
        ],
        "Size": 1458176,
        "Architecture": "amd64",
        "Os": "linux",
        ...
    }
]
```

### Step 6: Get Image ID
```bash
docker images busybox:musl --format "{{.ID}}"
```

**Expected output:**
```
1e2006b46d8b
```

**Save this ID - we'll verify it matches after tagging**

### Step 7: Tag the Image
```bash
docker tag busybox:musl busybox:media
```

**No output means success!**

**Command breakdown:**
- `docker tag` - Create new tag
- `busybox:musl` - Source image
- `busybox:media` - New tag name

### Step 8: Verify New Tag Created
```bash
docker images
```

**Expected output:**
```
REPOSITORY   TAG       IMAGE ID       CREATED         SIZE
busybox      media     1e2006b46d8b   14 months ago   1.51MB
busybox      musl      1e2006b46d8b   14 months ago   1.51MB
```

**Key observations:**
- ✅ Two tags: `musl` and `media`
- ✅ Same Image ID: `1e2006b46d8b`
- ✅ Same size: 1.51MB
- ✅ Same creation date: 14 months ago
- ✅ No storage duplication

### Step 9: List All busybox Tags
```bash
docker images --filter "reference=busybox:*"
```

**Expected output:**
```
REPOSITORY   TAG     IMAGE ID       CREATED         SIZE
busybox      media   1e2006b46d8b   14 months ago   1.51MB
busybox      musl    1e2006b46d8b   14 months ago   1.51MB
```

### Step 10: Compare Image IDs
```bash
echo "busybox:musl ID:"
docker images busybox:musl --format "{{.ID}}"

echo "busybox:media ID:"
docker images busybox:media --format "{{.ID}}"
```

**Expected output:**
```
busybox:musl ID:
1e2006b46d8b

busybox:media ID:
1e2006b46d8b
```

**IDs match = Same image! ✅**

### Step 11: Test Both Tags (Optional)
```bash
# Test original tag
docker run --rm busybox:musl echo "Testing musl tag"

# Test new tag
docker run --rm busybox:media echo "Testing media tag"
```

**Expected output:**
```
Testing musl tag
Testing media tag
```

**Both work identically!**

### Step 12: Verify with Docker Info
```bash
docker images busybox --format "table {{.Repository}}\t{{.Tag}}\t{{.ID}}\t{{.Size}}"
```

**Expected output:**
```
REPOSITORY   TAG     ID            SIZE
busybox      media   1e2006b46d8b  1.51MB
busybox      musl    1e2006b46d8b  1.51MB
```

**Perfect! Both tags pointing to same image ✅**

---

## Complete Command Summary

### Quick Tagging Operation:
```bash
# SSH to server
ssh banner@stapp03

# Switch to root
sudo su -

# Pull source image
docker pull busybox:musl

# Tag image
docker tag busybox:musl busybox:media

# Verify
docker images
```

### Detailed Verification:
```bash
# SSH
ssh banner@stapp03

# Switch to root
sudo su -

# View current images
docker images

# Pull busybox:musl
docker pull busybox:musl

# Get image ID before tagging
echo "Original image ID:"
docker images busybox:musl --format "{{.ID}}"

# Create new tag
docker tag busybox:musl busybox:media

# Verify both tags exist
echo "All busybox images:"
docker images busybox

# Compare IDs
echo "musl ID: $(docker images busybox:musl --format '{{.ID}}')"
echo "media ID: $(docker images busybox:media --format '{{.ID}}')"

# Test both (optional)
docker run --rm busybox:musl echo "musl works"
docker run --rm busybox:media echo "media works"
```

---

## Understanding Docker Tag Command

### Syntax:
```bash
docker tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG]
```

### Tagging Patterns:

**1. Same Repository, Different Tag:**
```bash
docker tag busybox:musl busybox:media
```

**2. Different Repository, Same Tag:**
```bash
docker tag busybox:musl mybox:musl
```

**3. Add Tag to Latest:**
```bash
docker tag nginx:latest nginx:prod
```

**4. Version to Latest:**
```bash
docker tag myapp:1.0.0 myapp:latest
```

**5. Add Registry:**
```bash
docker tag myapp:latest myregistry.com/myapp:latest
```

**6. Full Path:**
```bash
docker tag nginx:alpine registry.example.com:5000/web/nginx:alpine
```

### Tag Rules:

| Rule | Valid | Invalid |
|------|-------|---------|
| Characters | `a-z`, `0-9`, `.`, `-`, `_` | Uppercase, spaces, special chars |
| Start with | Letter or number | Dash, dot |
| Length | Up to 128 chars | Longer than 128 |
| Examples | `v1.0`, `prod-2025` | `Prod`, `v1.0!` |

---

## Image Storage and Tags

### Storage Model:

```
Docker Image Storage
│
├── Image Layers (content-addressed)
│   ├── Layer 1: base filesystem
│   ├── Layer 2: add packages
│   └── Layer 3: configuration
│
└── Tags (pointers to image)
    ├── busybox:musl ────┐
    └── busybox:media ───┴─→ Same Image ID
```

### Storage Facts:

**No Duplication:**
```bash
# Single image, two tags
busybox:musl   1.51MB  1e2006b46d8b
busybox:media  1.51MB  1e2006b46d8b
# Total storage: 1.51MB (not 3.02MB)
```

**Tag is Metadata:**
- Tags are just labels
- No data duplication
- Instant operation
- Can create unlimited tags

**Image Layers:**
- Content-addressed storage
- Shared between images
- Immutable layers
- Deduplicated automatically

---

## Troubleshooting

### Issue 1: Image Not Found

**Problem:**
```
Error response from daemon: pull access denied for busybox, repository does not exist or may require 'docker login'
```

**Solution:**
```bash
# Check spelling
sudo docker pull busybox:musl

# Not busybox:muscle or busybox:msl

# If authentication needed
docker login

# Try again
sudo docker pull busybox:musl
```

### Issue 2: Tag Already Exists

**Problem:**
```
# No error, but you want to replace
```

**Solution:**
```bash
# Docker overwrites by default
sudo docker tag busybox:musl busybox:media

# This replaces the media tag if it exists

# To be safe, remove old tag first
sudo docker rmi busybox:media
sudo docker tag busybox:musl busybox:media
```

### Issue 3: Invalid Tag Name

**Problem:**
```
Error parsing reference: "busybox:Media" is not a valid repository/tag
```

**Solution:**
```bash
# ❌ Uppercase not allowed
sudo docker tag busybox:musl busybox:Media

# ✅ Use lowercase
sudo docker tag busybox:musl busybox:media

# ❌ Special characters
sudo docker tag busybox:musl busybox:media!

# ✅ Use alphanumeric and dashes
sudo docker tag busybox:musl busybox:media-v1
```

### Issue 4: Source Image Doesn't Exist

**Problem:**
```
Error: No such image: busybox:musl
```

**Solution:**
```bash
# Check what images exist
sudo docker images busybox

# Pull the source image first
sudo docker pull busybox:musl

# Then tag
sudo docker tag busybox:musl busybox:media
```

### Issue 5: Permission Denied

**Problem:**
```
Got permission denied while trying to connect to the Docker daemon socket
```

**Solution:**
```bash
# Use sudo
sudo docker tag busybox:musl busybox:media

# Or add user to docker group
sudo usermod -aG docker banner
# Logout and login again
```

### Issue 6: Cannot Remove Tagged Image

**Problem:**
```
Error: conflict: unable to remove repository reference "busybox:musl" (must force)
```

**Solution:**
```bash
# Can't remove if it's the only tag
# This is fine - keep both tags

# To remove one tag
sudo docker rmi busybox:media

# Original busybox:musl remains

# To remove image completely, remove all tags
sudo docker rmi busybox:musl busybox:media
```

---

## Docker Image Management

### Listing Images:

```bash
# All images
sudo docker images

# Specific repository
sudo docker images busybox

# With filters
sudo docker images --filter "reference=busybox:*"

# Specific format
sudo docker images --format "{{.Repository}}:{{.Tag}}"

# Show digests
sudo docker images --digests

# Only image IDs
sudo docker images -q
```

### Removing Tags:

```bash
# Remove one tag
sudo docker rmi busybox:media

# busybox:musl still exists

# Remove both
sudo docker rmi busybox:musl busybox:media

# Force remove (with containers)
sudo docker rmi -f busybox:media

# Remove unused images
sudo docker image prune

# Remove all images
sudo docker rmi $(sudo docker images -q)
```

### Inspecting Images:

```bash
# Full details
sudo docker inspect busybox:media

# Specific fields
sudo docker inspect busybox:media --format='{{.Size}}'

# Layer information
sudo docker history busybox:media

# Image digest
sudo docker images --digests busybox:media
```

---

## Tag Naming Conventions

### Version Tags:

```bash
# Semantic versioning
myapp:1.0.0
myapp:1.0
myapp:1

# With variant
myapp:1.0.0-alpine
myapp:2.1-slim

# Date-based
myapp:2025.12.13
myapp:20251213
```

### Environment Tags:

```bash
myapp:dev
myapp:staging
myapp:prod
myapp:test

# With version
myapp:v1.0-prod
myapp:v2.0-staging
```

### Feature Tags:

```bash
myapp:feature-login
myapp:bugfix-123
myapp:hotfix-auth
```

### Best Practices:

```bash
# ✅ Good
nginx:1.25.3-alpine
myapp:v2.1.0
database:prod-2025.12

# ❌ Avoid
nginx:latest          # Too vague
myapp:new            # Not descriptive
db:final             # Ambiguous
test:v1              # Environment + version mixed
```

---

## Real-World Tag Scenarios

### Scenario 1: Promote Dev to Prod
```bash
# Tag dev image as prod
docker tag myapp:dev myapp:prod

# Push to registry
docker push myapp:prod
```

### Scenario 2: Create Backup Tag
```bash
# Before updating
docker tag myapp:latest myapp:backup-$(date +%Y%m%d)

# Update latest
docker pull myapp:latest
```

### Scenario 3: Multi-Environment Deployment
```bash
# Build once
docker build -t myapp:v1.0.0 .

# Tag for environments
docker tag myapp:v1.0.0 myapp:dev
docker tag myapp:v1.0.0 myapp:staging
docker tag myapp:v1.0.0 myapp:prod
```

### Scenario 4: Registry Migration
```bash
# Tag for new registry
docker tag myregistry.com/app:v1 newregistry.com/app:v1

# Push to new registry
docker push newregistry.com/app:v1
```

### Scenario 5: Version Aliases
```bash
# Create multiple version tags
docker tag myapp:1.2.3 myapp:1.2
docker tag myapp:1.2.3 myapp:1
docker tag myapp:1.2.3 myapp:latest
```

### Scenario 6: CI/CD Pipeline
```bash
# Build with git commit
docker build -t myapp:${GIT_COMMIT} .

# Also tag as branch
docker tag myapp:${GIT_COMMIT} myapp:main

# And latest if main branch
if [ "$BRANCH" = "main" ]; then
  docker tag myapp:${GIT_COMMIT} myapp:latest
fi
```

---

## Docker Registry Tagging

### Local Registry:

```bash
# Default (Docker Hub)
docker pull busybox:musl
docker tag busybox:musl busybox:media
```

### Private Registry:

```bash
# Pull from private registry
docker pull myregistry.com:5000/busybox:musl

# Tag for different registry
docker tag myregistry.com:5000/busybox:musl myregistry.com:5000/busybox:media

# Push to registry
docker push myregistry.com:5000/busybox:media
```

### Multi-Registry:

```bash
# Pull from Docker Hub
docker pull nginx:alpine

# Tag for Google Container Registry
docker tag nginx:alpine gcr.io/myproject/nginx:alpine

# Tag for AWS ECR
docker tag nginx:alpine 123456789.dkr.ecr.us-east-1.amazonaws.com/nginx:alpine

# Push to both
docker push gcr.io/myproject/nginx:alpine
docker push 123456789.dkr.ecr.us-east-1.amazonaws.com/nginx:alpine
```

---

## Understanding Image Layers

### View Layers:

```bash
sudo docker history busybox:media
```

**Output:**
```
IMAGE          CREATED       CREATED BY                           SIZE
3d24ee258efc   2 weeks ago   /bin/sh -c #(nop)  CMD ["sh"]        0B
<missing>      2 weeks ago   /bin/sh -c #(nop) ADD file:...       1.46MB
```

### Layer Sharing:

```
Image 1: nginx:alpine          Image 2: nginx:latest
    ↓                               ↓
[Alpine base layer]            [Debian base layer]
[Nginx binaries]  ←── Shared ───→ [Nginx binaries]
[Config layer]                 [Config layer]
```

**Tags point to top layer, share underlying layers**

---

## Best Practices

### 1. Use Descriptive Tags
```bash
# ✅ Good
docker tag myapp:musl myapp:media-testing-v1

# ❌ Vague
docker tag myapp:musl myapp:new
```

### 2. Document Tag Purpose
```bash
# Add label with purpose
docker tag myapp:musl myapp:media

# Document in README
echo "busybox:media - Testing tag for media project" >> DOCKER_TAGS.md
```

### 3. Keep Tags Consistent
```bash
# Use consistent naming scheme
myapp:v1.0-prod
myapp:v1.0-staging
myapp:v1.0-dev

# Not mixing styles
myapp:prod
myapp:v1.0-staging
myapp:development
```

### 4. Don't Rely on 'latest'
```bash
# ❌ Ambiguous
docker pull busybox:latest

# ✅ Specific
docker pull busybox:1.36.1-musl
```

### 5. Tag Before Pushing
```bash
# Tag for registry
docker tag myapp:local myregistry.com/myapp:v1.0

# Then push
docker push myregistry.com/myapp:v1.0
```

### 6. Clean Up Old Tags
```bash
# Remove unused tags
docker images | grep "<none>" | awk '{print $3}' | xargs docker rmi

# Or
docker image prune
```

### 7. Verify After Tagging
```bash
# Always verify
docker images busybox

# Check IDs match
docker images busybox --format "{{.Tag}}: {{.ID}}"
```

---

## Common Use Cases

### 1. Development Workflow
```bash
# Developer builds image
docker build -t myapp:dev .

# Test locally
docker run myapp:dev

# Promote to staging
docker tag myapp:dev myapp:staging

# After testing, promote to prod
docker tag myapp:staging myapp:prod
```

### 2. Version Management
```bash
# Release version 2.0
docker build -t myapp:2.0 .

# Also tag as latest
docker tag myapp:2.0 myapp:latest

# Keep old version available
# myapp:1.0 still exists
```

### 3. Testing Variants
```bash
# Original
busybox:musl

# Test variant
docker tag busybox:musl busybox:media

# A/B testing
docker tag myapp:current myapp:variant-a
docker tag myapp:experimental myapp:variant-b
```

### 4. Backup Before Update
```bash
# Current production
docker tag myapp:prod myapp:prod-backup-$(date +%Y%m%d)

# Deploy new version
docker pull myapp:latest
docker tag myapp:latest myapp:prod
```

---

## Key Commands Reference

| Command | Description |
|---------|-------------|
| `docker tag SOURCE TARGET` | Create new tag |
| `docker images` | List all images |
| `docker images REPO` | List specific repository |
| `docker pull IMAGE:TAG` | Download image |
| `docker rmi IMAGE:TAG` | Remove tag/image |
| `docker inspect IMAGE:TAG` | Image details |
| `docker history IMAGE:TAG` | Layer history |
| `docker images --digests` | Show digests |
| `docker images -q` | Only IDs |
| `docker image prune` | Remove unused images |

---

## Completion Checklist

- [ ] SSH into App Server 3 (stapp03) as banner
- [ ] Verified Docker service is running
- [ ] Checked current Docker images
- [ ] Pulled busybox:musl from Docker Hub
- [ ] Verified busybox:musl downloaded successfully
- [ ] Noted original image ID
- [ ] Tagged busybox:musl as busybox:media
- [ ] Verified both tags exist
- [ ] Confirmed both tags have same Image ID
- [ ] Tested both tags work (optional)
- [ ] Verified no storage duplication
- [ ] Both tags point to same image layers ✅

---

## Completion Details

- **Completion Date:** December 13, 2025
- **Day:** 38 / 100
- **Challenge:** KodeKloud 100 Days Cloud DevOps
- **Topic:** Docker Image Tagging
- **Server:** App Server 3 (stapp03)
- **User:** banner
- **Source Image:** `busybox:musl` (1.51MB)
- **New Tag:** `busybox:media`
- **Image ID:** `1e2006b46d8b`
- **Same Image ID:** ✅ Verified
- **Storage Used:** 1.51MB (no duplication)
- **Key Skill:** Docker image management and version control
- **Status:** ✅ Successfully Completed

---

## Summary

This task demonstrated **Docker image tagging** fundamentals:

✅ **Pulled busybox:musl** - Minimal 1.51MB image from Docker Hub
✅ **Created new tag** - Re-tagged as busybox:media
✅ **Verified no duplication** - Same Image ID (1e2006b46d8b) for both tags
✅ **Tested functionality** - Both tags work identically
✅ **Understood storage** - Tags are metadata, no data duplication

**Key Insight:** Docker tags are **pointers to images**, not copies. When you tag an image:
- **No data duplication** - Tags are just labels
- **Instant operation** - No copying or moving data
- **Same Image ID** - Multiple tags point to same layers
- **Storage efficient** - 10 tags = same storage as 1 tag

This is crucial for:
- **Version control** - Track different versions (v1.0, v2.0, latest)
- **Environment management** - Same image, different names (dev, staging, prod)
- **Testing** - Create test tags without duplicating storage
- **CI/CD** - Tag based on git commits, branches, environments
- **Registry operations** - Tag for different registries before pushing

**Remember:** `docker tag source:tag new:tag` = New Label, Same Image! 🐳

**The Pattern:**
```
Pull → Tag → Verify (same ID) → Use
```

**Why This Matters:**
In production, you might have:
- `myapp:v1.2.3` (specific version)
- `myapp:1.2` (minor version)
- `myapp:1` (major version)
- `myapp:latest` (current)
- `myapp:prod` (production)
- `myapp:stable` (stable release)

All pointing to the **same image**, zero storage overhead! This enables sophisticated deployment strategies without wasting space.

**Next:** Building custom images with Dockerfiles! 🚀
