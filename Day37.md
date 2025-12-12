# Day 37: Docker File Operations - Host to Container
**100 Days Cloud DevOps Challenge – KodeKloud**

---

## Objective

The Nautilus DevOps team has confidential data on App Server 1 that needs to be transferred to a running container. Copy an encrypted file from the Docker host to a container without modifying its contents.

**Requirements:**
1. Work on App Server 1 in Stratos Datacenter
2. Container `ubuntu_latest` is already running
3. Copy encrypted file `/tmp/nautilus.txt.gpg` from host
4. Destination: `/opt/` directory inside container
5. Ensure file integrity (no modifications)

---

## Understanding Docker File Operations

**Docker CP** is the command for copying files and directories between the Docker host and containers, similar to the Linux `cp` command but works across container boundaries.

### Why Copy Files to Containers?

- **Configuration files** - Deploy app configs
- **Data files** - Import datasets
- **Secrets** - Transfer encrypted credentials
- **Logs** - Extract logs for analysis
- **Backups** - Copy data for backup
- **Development** - Hot-reload code changes

### Docker CP vs Volumes:

| Method | Use Case | Persistence | Performance |
|--------|----------|-------------|-------------|
| **docker cp** | One-time transfers | Manual | Good |
| **Volumes** | Persistent data | Automatic | Better |
| **Bind mounts** | Development | Real-time sync | Best |

### File Transfer Scenarios:

```
Host → Container:  docker cp /host/file container:/path/
Container → Host:  docker cp container:/path/file /host/
Container ↔ Container: Copy to host, then to container
```

---

## Understanding Encrypted Files

### GPG Encryption:

**GPG (GNU Privacy Guard)** is a free implementation of the OpenPGP standard for encrypting and signing data.

**File Extension:** `.gpg` indicates encrypted file

**Characteristics:**
- Binary format (not human-readable)
- Requires key/passphrase to decrypt
- Maintains file integrity
- Used for secure data transfer

### Why Transfer Encrypted Files?

- **Security:** Data encrypted at rest
- **Compliance:** Regulatory requirements
- **Confidentiality:** Protect sensitive information
- **Integrity:** Verify file hasn't been tampered with
- **Defense-in-depth:** Multiple security layers

### File Integrity Verification:

```bash
# Before copy
md5sum /tmp/nautilus.txt.gpg

# After copy (in container)
docker exec container md5sum /opt/nautilus.txt.gpg

# Checksums should match
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
| Server | App Server 1 (stapp01) |
| Container | ubuntu_latest (running) |
| Source File | `/tmp/nautilus.txt.gpg` (host) |
| Destination | `/opt/nautilus.txt.gpg` (container) |
| File Type | GPG encrypted |
| Requirement | No modifications |

---

## Understanding the Task

### File Transfer Flow:

```
App Server 1 (Host)                    Container: ubuntu_latest
┌───────────────────────┐             ┌──────────────────────┐
│                       │             │                      │
│  /tmp/nautilus.txt.gpg│────────────→│  /opt/nautilus.txt.gpg│
│  (encrypted file)     │   docker cp │  (same content)      │
│                       │             │                      │
└───────────────────────┘             └──────────────────────┘
```

### What Docker CP Does:

1. **Reads** file from host filesystem
2. **Streams** data to container
3. **Writes** file to container filesystem
4. **Preserves** permissions and timestamps (by default)
5. **Maintains** file integrity (binary-safe)

---

## Step-by-Step Implementation

### Step 1: SSH into App Server 1
```bash
ssh tony@stapp01
```

**Enter password:** `Ir0nM@n`

**Expected output:**
```
tony@stapp01's password:
Last login: ...
[tony@stapp01 ~]$
```

### Step 2: Verify Docker is Running
```bash
sudo systemctl status docker
```

**Expected output:**
```
● docker.service - Docker Application Container Engine
   Loaded: loaded (/usr/lib/systemd/system/docker.service; enabled)
   Active: active (running) since ...
```

### Step 3: List Running Containers
```bash
sudo docker ps
```

**Expected output:**
```
CONTAINER ID   IMAGE          COMMAND       CREATED       STATUS       PORTS     NAMES
abc123def456   ubuntu:latest  "/bin/bash"   2 hours ago   Up 2 hours             ubuntu_latest
```

**Verify:**
- ✅ Container named `ubuntu_latest` is present
- ✅ STATUS shows "Up" (running)

### Step 4: Verify Source File Exists
```bash
ls -lh /tmp/nautilus.txt.gpg
```

**Expected output:**
```
-rw-r--r-- 1 root root 2.3K Dec 10 10:00 /tmp/nautilus.txt.gpg
```

**Check file details:**
```bash
file /tmp/nautilus.txt.gpg
```

**Expected output:**
```
/tmp/nautilus.txt.gpg: GPG symmetrically encrypted data (AES256 cipher)
```

### Step 5: Check File Checksum (Before Copy)
```bash
sudo md5sum /tmp/nautilus.txt.gpg
```

**Expected output:**
```
a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6  /tmp/nautilus.txt.gpg
```

**Save this checksum for verification later!**

### Step 6: View File Permissions
```bash
stat /tmp/nautilus.txt.gpg
```

**Expected output:**
```
  File: /tmp/nautilus.txt.gpg
  Size: 2345      	Blocks: 8          IO Block: 4096   regular file
Device: fd00h/64768d	Inode: 12345       Links: 1
Access: (0644/-rw-r--r--)  Uid: (    0/    root)   Gid: (    0/    root)
Access: 2025-12-10 10:00:00.000000000 +0000
Modify: 2025-12-10 10:00:00.000000000 +0000
Change: 2025-12-10 10:00:00.000000000 +0000
```

### Step 7: Check Container's /opt Directory
```bash
sudo docker exec ubuntu_latest ls -la /opt
```

**Expected output:**
```
total 8
drwxr-xr-x 2 root root 4096 Oct 15 10:00 .
drwxr-xr-x 1 root root 4096 Dec 10 09:00 ..
```

**The /opt directory exists and is empty**

### Step 8: Copy File from Host to Container
```bash
sudo docker cp /tmp/nautilus.txt.gpg ubuntu_latest:/opt/
```

**No output means success!**

**Command breakdown:**
- `docker cp` - Copy command
- `/tmp/nautilus.txt.gpg` - Source (host path)
- `ubuntu_latest:/opt/` - Destination (container:path)

### Step 9: Verify File Copied Successfully
```bash
sudo docker exec ubuntu_latest ls -lh /opt/
```

**Expected output:**
```
total 4.0K
-rw-r--r-- 1 root root 2.3K Dec 10 10:00 nautilus.txt.gpg
```

**Verify:**
- ✅ File `nautilus.txt.gpg` is present
- ✅ Size matches original (2.3K)
- ✅ Permissions preserved (0644)

### Step 10: Verify File Type in Container
```bash
sudo docker exec ubuntu_latest file /opt/nautilus.txt.gpg
```

**Expected output:**
```
/opt/nautilus.txt.gpg: GPG symmetrically encrypted data (AES256 cipher)
```

**Same file type as source - good!**

### Step 11: Check File Checksum (After Copy)
```bash
sudo docker exec ubuntu_latest md5sum /opt/nautilus.txt.gpg
```

**Expected output:**
```
a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6  /opt/nautilus.txt.gpg
```

**Compare with Step 5 checksum - they must match!**

### Step 12: Compare Checksums Directly
```bash
# Host checksum
echo "Host checksum:"
sudo md5sum /tmp/nautilus.txt.gpg

# Container checksum
echo "Container checksum:"
sudo docker exec ubuntu_latest md5sum /opt/nautilus.txt.gpg
```

**Expected output:**
```
Host checksum:
a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6  /tmp/nautilus.txt.gpg

Container checksum:
a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6  /opt/nautilus.txt.gpg
```

**✅ Checksums match = File integrity verified!**

### Step 13: Check File Size Match
```bash
# Host file size
ls -l /tmp/nautilus.txt.gpg | awk '{print $5}'

# Container file size
sudo docker exec ubuntu_latest bash -c "ls -l /opt/nautilus.txt.gpg | awk '{print \$5}'"
```

**Both should show the same byte count**

### Step 14: Verify File Permissions
```bash
sudo docker exec ubuntu_latest stat /opt/nautilus.txt.gpg
```

**Expected output:**
```
  File: /opt/nautilus.txt.gpg
  Size: 2345      	Blocks: 8          IO Block: 4096   regular file
Device: 33h/51d	Inode: 78901       Links: 1
Access: (0644/-rw-r--r--)  Uid: (    0/    root)   Gid: (    0/    root)
Access: 2025-12-10 10:00:00.000000000 +0000
Modify: 2025-12-10 10:00:00.000000000 +0000
Change: 2025-12-10 10:15:00.000000000 +0000
```

**Permissions preserved (0644)**

### Step 15: Final Verification - List Contents
```bash
# Host
echo "Host file:"
ls -lh /tmp/nautilus.txt.gpg

# Container
echo "Container file:"
sudo docker exec ubuntu_latest ls -lh /opt/nautilus.txt.gpg
```

**Expected output:**
```
Host file:
-rw-r--r-- 1 root root 2.3K Dec 10 10:00 /tmp/nautilus.txt.gpg

Container file:
-rw-r--r-- 1 root root 2.3K Dec 10 10:00 /opt/nautilus.txt.gpg
```

**Perfect match! ✅**

---

## Complete Command Summary

### Quick Copy Operation:
```bash
# SSH to server
ssh tony@stapp01

# Verify container running
sudo docker ps | grep ubuntu_latest

# Verify source file
ls -lh /tmp/nautilus.txt.gpg

# Get source checksum
sudo md5sum /tmp/nautilus.txt.gpg

# Copy file
sudo docker cp /tmp/nautilus.txt.gpg ubuntu_latest:/opt/

# Verify copy
sudo docker exec ubuntu_latest ls -lh /opt/nautilus.txt.gpg

# Verify checksum
sudo docker exec ubuntu_latest md5sum /opt/nautilus.txt.gpg
```

### With Complete Verification:
```bash
# SSH
ssh tony@stapp01

# Check container
sudo docker ps --filter "name=ubuntu_latest"

# Pre-copy verification
echo "=== Source File ==="
ls -lh /tmp/nautilus.txt.gpg
file /tmp/nautilus.txt.gpg
sudo md5sum /tmp/nautilus.txt.gpg

# Copy operation
sudo docker cp /tmp/nautilus.txt.gpg ubuntu_latest:/opt/

# Post-copy verification
echo "=== Destination File ==="
sudo docker exec ubuntu_latest ls -lh /opt/nautilus.txt.gpg
sudo docker exec ubuntu_latest file /opt/nautilus.txt.gpg
sudo docker exec ubuntu_latest md5sum /opt/nautilus.txt.gpg

# Compare
echo "=== Comparison ==="
echo "Host:"
sudo md5sum /tmp/nautilus.txt.gpg
echo "Container:"
sudo docker exec ubuntu_latest md5sum /opt/nautilus.txt.gpg
```

---

## Understanding Docker CP Command

### Syntax:
```bash
docker cp [OPTIONS] SRC_PATH CONTAINER:DEST_PATH
docker cp [OPTIONS] CONTAINER:SRC_PATH DEST_PATH
```

### Direction Patterns:

**Host to Container:**
```bash
docker cp /host/path/file.txt container_name:/container/path/
docker cp /host/path/file.txt container_id:/container/path/
```

**Container to Host:**
```bash
docker cp container_name:/container/path/file.txt /host/path/
docker cp container_id:/container/path/file.txt /host/path/
```

### Path Behaviors:

| Source | Destination | Result |
|--------|-------------|--------|
| `/path/file.txt` | `container:/dir/` | File copied to `/dir/file.txt` |
| `/path/file.txt` | `container:/dir/newname.txt` | File copied as `/dir/newname.txt` |
| `/path/dir/` | `container:/dest/` | Directory contents copied |
| `/path/dir/.` | `container:/dest/` | All contents of dir copied |

### Important Options:

```bash
# Follow symbolic links (copy target, not link)
docker cp -L /path/symlink container:/dest/

# Archive mode (preserve attributes)
docker cp -a /path/file container:/dest/
```

---

## Docker CP vs Other Methods

### 1. Docker CP (This Task)
```bash
docker cp /host/file container:/path/
```

**Pros:**
- ✅ Simple one-time transfers
- ✅ Works with running or stopped containers
- ✅ No volume setup needed
- ✅ Preserves file attributes

**Cons:**
- ❌ Manual operation
- ❌ Not real-time
- ❌ No automatic sync

**Best For:** One-time file transfers, config deployment, log extraction

### 2. Docker Volumes
```bash
docker run -v /host/path:/container/path image
```

**Pros:**
- ✅ Automatic synchronization
- ✅ Persistent data
- ✅ Shared across containers
- ✅ Performance optimized

**Cons:**
- ❌ Must configure at container creation
- ❌ More complex setup

**Best For:** Databases, persistent storage, shared data

### 3. Bind Mounts
```bash
docker run -v /host/file:/container/file:ro image
```

**Pros:**
- ✅ Real-time sync
- ✅ Direct file access
- ✅ Read-only option

**Cons:**
- ❌ Host filesystem dependency
- ❌ Security considerations

**Best For:** Development, configuration files, hot-reload

### 4. Docker Build (COPY/ADD)
```dockerfile
COPY file.txt /app/
ADD archive.tar.gz /app/
```

**Pros:**
- ✅ Baked into image
- ✅ Version controlled
- ✅ Immutable

**Cons:**
- ❌ Requires rebuild
- ❌ Image size increase

**Best For:** Application code, static resources

### 5. Network Transfer (SCP/SFTP)
```bash
docker exec container curl -O http://host/file
```

**Pros:**
- ✅ Remote transfers
- ✅ Works across hosts

**Cons:**
- ❌ Complex setup
- ❌ Network dependency

**Best For:** Multi-host scenarios, remote data

---

## File Integrity Verification Methods

### 1. MD5 Checksum (Fast)
```bash
# Host
md5sum /tmp/nautilus.txt.gpg

# Container
docker exec ubuntu_latest md5sum /opt/nautilus.txt.gpg
```

**Output:** 32-character hexadecimal hash

### 2. SHA256 Checksum (More Secure)
```bash
# Host
sha256sum /tmp/nautilus.txt.gpg

# Container
docker exec ubuntu_latest sha256sum /opt/nautilus.txt.gpg
```

**Output:** 64-character hexadecimal hash

### 3. File Size Comparison
```bash
# Host
stat -c%s /tmp/nautilus.txt.gpg

# Container
docker exec ubuntu_latest stat -c%s /opt/nautilus.txt.gpg
```

**Must be identical**

### 4. Byte-by-Byte Comparison
```bash
# Copy file back from container
docker cp ubuntu_latest:/opt/nautilus.txt.gpg /tmp/nautilus-copied.txt.gpg

# Compare
diff /tmp/nautilus.txt.gpg /tmp/nautilus-copied.txt.gpg

# No output = identical
```

### 5. File Type Check
```bash
# Host
file /tmp/nautilus.txt.gpg

# Container
docker exec ubuntu_latest file /opt/nautilus.txt.gpg
```

**Must match**

---

## Troubleshooting

### Issue 1: Container Not Found

**Problem:**
```
Error: No such container: ubuntu_latest
```

**Solution:**
```bash
# List all containers
sudo docker ps -a

# Check correct container name
sudo docker ps --format "table {{.Names}}\t{{.Status}}"

# If stopped, start it
sudo docker start ubuntu_latest

# Verify running
sudo docker ps | grep ubuntu_latest
```

### Issue 2: Source File Not Found

**Problem:**
```
Error: No such file or directory: /tmp/nautilus.txt.gpg
```

**Solution:**
```bash
# Check if file exists
ls -la /tmp/nautilus.txt.gpg

# Check permissions
stat /tmp/nautilus.txt.gpg

# If missing, check different location
find / -name "nautilus.txt.gpg" 2>/dev/null

# Verify you're in correct directory
pwd
```

### Issue 3: Permission Denied

**Problem:**
```
Error response from daemon: ... permission denied
```

**Solution:**
```bash
# Use sudo
sudo docker cp /tmp/nautilus.txt.gpg ubuntu_latest:/opt/

# Check file permissions
ls -l /tmp/nautilus.txt.gpg

# Make readable if needed
sudo chmod 644 /tmp/nautilus.txt.gpg
```

### Issue 4: Destination Directory Doesn't Exist

**Problem:**
```
Error: No such container path
```

**Solution:**
```bash
# Check if /opt exists in container
sudo docker exec ubuntu_latest ls -la /opt

# Create directory if missing
sudo docker exec ubuntu_latest mkdir -p /opt

# Then copy
sudo docker cp /tmp/nautilus.txt.gpg ubuntu_latest:/opt/
```

### Issue 5: File Already Exists

**Problem:**
File exists in container, overwrite needed

**Solution:**
```bash
# docker cp overwrites by default

# But if you want to backup first
sudo docker exec ubuntu_latest mv /opt/nautilus.txt.gpg /opt/nautilus.txt.gpg.bak

# Then copy
sudo docker cp /tmp/nautilus.txt.gpg ubuntu_latest:/opt/
```

### Issue 6: Checksum Mismatch

**Problem:**
Checksums don't match after copy

**Solution:**
```bash
# This should NOT happen with docker cp
# If it does:

# 1. Verify source file integrity
md5sum /tmp/nautilus.txt.gpg

# 2. Remove corrupted file
sudo docker exec ubuntu_latest rm /opt/nautilus.txt.gpg

# 3. Copy again
sudo docker cp /tmp/nautilus.txt.gpg ubuntu_latest:/opt/

# 4. Verify again
sudo docker exec ubuntu_latest md5sum /opt/nautilus.txt.gpg

# If still fails, check disk space
df -h
sudo docker exec ubuntu_latest df -h
```

---

## Advanced Docker CP Scenarios

### Scenario 1: Copy Entire Directory
```bash
# Copy directory with contents
sudo docker cp /tmp/mydir ubuntu_latest:/opt/

# Verify
sudo docker exec ubuntu_latest ls -la /opt/mydir
```

### Scenario 2: Copy Multiple Files
```bash
# Create archive on host
tar -czf /tmp/files.tar.gz /tmp/file1 /tmp/file2 /tmp/file3

# Copy archive
sudo docker cp /tmp/files.tar.gz ubuntu_latest:/opt/

# Extract in container
sudo docker exec ubuntu_latest tar -xzf /opt/files.tar.gz -C /opt/

# Cleanup archive
sudo docker exec ubuntu_latest rm /opt/files.tar.gz
```

### Scenario 3: Copy with Different Name
```bash
# Specify new filename
sudo docker cp /tmp/nautilus.txt.gpg ubuntu_latest:/opt/renamed-file.txt.gpg
```

### Scenario 4: Copy from Container to Host
```bash
# Extract logs
sudo docker cp ubuntu_latest:/var/log/app.log /tmp/app.log

# Extract config
sudo docker cp ubuntu_latest:/etc/app/config.json ./config-backup.json
```

### Scenario 5: Copy Between Containers
```bash
# Copy from container1 to host
sudo docker cp container1:/path/file /tmp/file

# Copy from host to container2
sudo docker cp /tmp/file container2:/path/file

# Cleanup
rm /tmp/file
```

### Scenario 6: Copy with Archive Mode
```bash
# Preserve all attributes
sudo docker cp -a /tmp/nautilus.txt.gpg ubuntu_latest:/opt/
```

### Scenario 7: Copy Symbolic Links
```bash
# Follow symlink (copy target)
sudo docker cp -L /tmp/symlink ubuntu_latest:/opt/

# Copy symlink as-is (not recommended for cross-platform)
sudo docker cp /tmp/symlink ubuntu_latest:/opt/
```

---

## Working with Encrypted Files

### GPG File Operations:

**Encrypt a file:**
```bash
gpg -c file.txt
# Creates file.txt.gpg
```

**Decrypt a file:**
```bash
gpg file.txt.gpg
# Creates file.txt
```

**View encrypted file info:**
```bash
gpg --list-packets file.txt.gpg
```

### Handling GPG Files in Containers:

**Copy encrypted file:**
```bash
docker cp secret.txt.gpg container:/opt/
```

**Decrypt inside container:**
```bash
# Install GPG in container if needed
docker exec container apt-get update
docker exec container apt-get install -y gnupg

# Decrypt
docker exec -it container gpg /opt/secret.txt.gpg
```

**Security Best Practices:**
- ✅ Keep files encrypted at rest
- ✅ Use strong passphrases
- ✅ Limit file permissions (600 or 400)
- ✅ Decrypt only when needed
- ✅ Securely delete decrypted files
- ✅ Use volume encryption for sensitive data

---

## Docker Container File System

### Container File System Layers:

```
┌─────────────────────────────────┐
│  Read-Write Container Layer     │  ← docker cp writes here
├─────────────────────────────────┤
│  Read-Only Image Layers         │
│  (Application files)            │
├─────────────────────────────────┤
│  Read-Only Base Image Layer     │
│  (Ubuntu, Alpine, etc.)         │
└─────────────────────────────────┘
```

### Common Container Directories:

| Directory | Purpose |
|-----------|---------|
| `/opt` | Optional software/packages |
| `/tmp` | Temporary files |
| `/var/log` | Log files |
| `/etc` | Configuration files |
| `/home` | User home directories |
| `/usr/local` | Locally installed programs |
| `/app` | Application directory (convention) |

### File Persistence:

**Container Layer (Non-Persistent):**
```bash
docker cp file.txt container:/opt/
# Lost when container is removed
```

**Volume (Persistent):**
```bash
docker run -v myvolume:/opt ubuntu
# Data survives container removal
```

---

## Real-World Use Cases

### 1. Deploy Configuration Files
```bash
# Copy app config
docker cp app.config.json web_app:/etc/app/

# Restart app to load config
docker restart web_app
```

### 2. Extract Application Logs
```bash
# Get logs from container
docker cp app_container:/var/log/app/error.log ./logs/

# Analyze locally
grep ERROR ./logs/error.log
```

### 3. Backup Database
```bash
# Create backup in container
docker exec db_container mysqldump -u root -p database > /tmp/backup.sql

# Copy to host
docker cp db_container:/tmp/backup.sql ./backups/db-$(date +%Y%m%d).sql
```

### 4. Deploy SSL Certificates
```bash
# Copy certificates
docker cp server.crt nginx:/etc/nginx/ssl/
docker cp server.key nginx:/etc/nginx/ssl/

# Set permissions
docker exec nginx chmod 600 /etc/nginx/ssl/server.key

# Reload nginx
docker exec nginx nginx -s reload
```

### 5. Update Application Code
```bash
# Copy new version
docker cp app-v2.jar tomcat:/usr/local/tomcat/webapps/

# Restart container
docker restart tomcat
```

### 6. Transfer Secrets
```bash
# Copy encrypted secrets
docker cp secrets.gpg app:/opt/

# Decrypt in container
docker exec -it app gpg --decrypt /opt/secrets.gpg > /opt/secrets.json
docker exec app rm /opt/secrets.gpg
```

---

## Best Practices

### 1. Verify Before and After
```bash
# Always check checksums
md5sum original
docker exec container md5sum copied
```

### 2. Use Absolute Paths
```bash
# ✅ Good
docker cp /tmp/file.txt container:/opt/

# ❌ Risky
docker cp file.txt container:/opt/
```

### 3. Set Appropriate Permissions
```bash
# Copy file
docker cp secret.txt container:/opt/

# Restrict permissions
docker exec container chmod 600 /opt/secret.txt
docker exec container chown app:app /opt/secret.txt
```

### 4. Handle Large Files Carefully
```bash
# For large files, consider volumes instead
# But if using cp:

# Monitor copy
docker cp large-file.tar.gz container:/opt/

# Verify space
docker exec container df -h /opt
```

### 5. Clean Up After Transfer
```bash
# If file was temporary
docker exec container rm /opt/temp-file.txt

# Or on host
rm /tmp/temp-file.txt
```

### 6. Document Transfer Operations
```bash
# Keep audit trail
echo "$(date): Copied nautilus.txt.gpg to ubuntu_latest:/opt/" >> /var/log/docker-ops.log
```

### 7. Test with Non-Critical Files First
```bash
# Test with dummy file
echo "test" > /tmp/test.txt
docker cp /tmp/test.txt container:/opt/
docker exec container cat /opt/test.txt
```

---

## Docker CP Alternatives Comparison

### When to Use Each Method:

**Use `docker cp` when:**
- ✅ One-time file transfer needed
- ✅ Container already exists
- ✅ No volume setup possible
- ✅ Extracting logs/backups
- ✅ Emergency file deployment

**Use Volumes when:**
- ✅ Persistent data required
- ✅ Multiple containers need access
- ✅ Database storage
- ✅ Frequent updates
- ✅ Production deployments

**Use Bind Mounts when:**
- ✅ Development environment
- ✅ Source code hot-reload
- ✅ Real-time file sync
- ✅ Host filesystem integration

**Use COPY in Dockerfile when:**
- ✅ Building images
- ✅ Application code
- ✅ Static resources
- ✅ Version controlled files

---

## Key Commands Reference

| Command | Description |
|---------|-------------|
| `docker cp host:file container:path` | Copy host to container |
| `docker cp container:path host:file` | Copy container to host |
| `docker cp -L` | Follow symbolic links |
| `docker cp -a` | Archive mode (preserve) |
| `docker exec container ls -la /path` | List container directory |
| `docker exec container cat /path/file` | View container file |
| `docker exec container md5sum /path/file` | Checksum in container |
| `md5sum /path/file` | Checksum on host |
| `stat /path/file` | File details |
| `file /path/file` | File type |

---

## Completion Checklist

- [ ] SSH into App Server 1 (stapp01) as tony
- [ ] Verified Docker service running
- [ ] Confirmed ubuntu_latest container running
- [ ] Located source file `/tmp/nautilus.txt.gpg`
- [ ] Verified file is GPG encrypted
- [ ] Generated source file checksum (md5sum)
- [ ] Executed docker cp command
- [ ] Verified file exists in container's /opt/
- [ ] Confirmed file type matches (GPG encrypted)
- [ ] Generated destination file checksum
- [ ] Compared checksums (must match)
- [ ] Verified file size matches
- [ ] Confirmed file permissions preserved
- [ ] No file modifications detected
- [ ] File integrity maintained ✅

---

## Completion Details

- **Completion Date:** December 12, 2025
- **Day:** 37 / 100
- **Challenge:** KodeKloud 100 Days Cloud DevOps
- **Topic:** Docker File Operations - Host to Container
- **Server:** App Server 1 (stapp01)
- **User:** tony
- **Container:** ubuntu_latest
- **Source:** `/tmp/nautilus.txt.gpg` (host)
- **Destination:** `/opt/nautilus.txt.gpg` (container)
- **File Type:** GPG encrypted
- **Integrity:** Verified (checksums match) ✅
- **Key Skill:** Docker file transfer and integrity verification
- **Status:** ✅ Successfully Completed

---

## Summary

This task demonstrated **Docker file transfer operations** with integrity verification:

✅ **Copied encrypted file** - From host to running container
✅ **Used docker cp** - Simple and effective file transfer
✅ **Maintained integrity** - No modifications during transfer
✅ **Verified with checksums** - MD5 hashes matched perfectly
✅ **Preserved attributes** - Permissions and file type maintained

**Key Insight:** Docker's `cp` command provides a reliable way to transfer files between host and containers while maintaining file integrity. This is essential for:
- **Configuration deployment** - Push config files to containers
- **Data migration** - Move data into containerized apps
- **Log extraction** - Pull logs for analysis
- **Secret deployment** - Transfer encrypted credentials
- **Backup operations** - Extract data from containers

Unlike manual methods that might corrupt binary files, `docker cp` handles all file types correctly, preserving content and attributes. For encrypted files like GPG, this is critical - any modification would make decryption impossible!

**Remember:** `docker cp source container:destination` = Safe File Transfer with Zero Modifications! 🐳

**The Pattern:**
1. **Verify source** (checksum)
2. **Copy with docker cp**
3. **Verify destination** (checksum)
4. **Compare** (must match)

This establishes trust in the transfer process - essential for production operations!

**Next:** Volume mounting, persistent storage, and advanced container data management! 🚀
