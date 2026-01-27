# Day 71: Jenkins Parameterized Job for Package Installation Automation

## 📋 Task Overview

**Scenario:** The Nautilus DevOps team has set up a new Jenkins server and wants to automate package installation on infrastructure servers. You need to create a parameterized Jenkins job that can install any package on the storage server.

**Given Requirements:**
- **Jenkins UI Access:** Click Jenkins button on top bar
- **Login Credentials:**
  - Username: `admin`
  - Password: `Adm!n321`
- **Job Configuration:**
  - Job Name: `install-packages`
  - Job Type: Freestyle project
  - Parameter: String parameter named `PACKAGE`
  - Target: Storage server in Stratos Datacenter
  - Action: Install package specified in `$PACKAGE` parameter

**Your Mission:**
1. Log in to Jenkins
2. Create new freestyle job named `install-packages`
3. Add string parameter `PACKAGE`
4. Configure build steps to install package on storage server
5. Test job execution with different packages
6. Verify repeatability and reliability

---

## 🎯 Learning Objectives

By the end of this task, you will understand:
- **Parameterized Builds:** Creating reusable jobs with dynamic inputs
- **Build Steps:** Executing shell commands and SSH operations
- **Remote Execution:** Running commands on remote servers from Jenkins
- **Job Configuration:** Best practices for job setup
- **Testing & Validation:** Ensuring job reliability
- **Plugin Management:** SSH plugins and credential handling

---

## 📖 Understanding Parameterized Jenkins Jobs

### What are Parameterized Builds?

**Parameterized builds** allow you to create flexible, reusable Jenkins jobs that accept input values at runtime.

**Benefits:**
```
Single Job → Multiple Use Cases
- Same job, different parameters
- Reduces job proliferation
- Easier maintenance
- Dynamic behavior based on input
```

**Example Use Cases:**
```
Deploy Job:
- Parameter: ENVIRONMENT (dev/staging/prod)
- Same job deploys to different environments

Build Job:
- Parameter: BRANCH_NAME (main/develop/feature)
- Same job builds different branches

Package Installation:
- Parameter: PACKAGE (nginx/mysql/redis)
- Same job installs different packages
```

---

### Parameter Types in Jenkins

**1. String Parameter**
```
Most common type
Accepts text input
Use case: Package names, branch names, URLs
```

**2. Choice Parameter**
```
Dropdown selection
Predefined options
Use case: Environment selection, version selection
```

**3. Boolean Parameter**
```
Checkbox (true/false)
Use case: Enable/disable features, debug mode
```

**4. Password Parameter**
```
Masked input field
Secure credential input
Use case: API keys, passwords (better to use Credentials plugin)
```

**5. File Parameter**
```
Upload file for build
Use case: Configuration files, deployment packages
```

**6. Multi-line String Parameter**
```
Text area input
Use case: Scripts, long configurations
```

---

### How Parameterized Builds Work

**Build Flow:**
```
1. User triggers job
   ↓
2. Jenkins displays parameter form
   ↓
3. User enters parameter values
   ↓
4. Jenkins runs job with parameters
   ↓
5. Parameters available as environment variables
   ↓
6. Build steps use parameter values
```

**Accessing Parameters:**
```bash
# In shell build steps:
echo "Package name: $PACKAGE"
yum install -y $PACKAGE

# In Jenkins Pipeline:
echo "Installing ${params.PACKAGE}"
sh "yum install -y ${params.PACKAGE}"
```

---

## 📖 Remote Command Execution from Jenkins

### Method 1: SSH Plugin (Recommended)

**Publish Over SSH Plugin:**
- Configures SSH connections to remote servers
- Stores credentials securely
- Supports multiple remote hosts
- Easy configuration in job

**Setup:**
```
1. Install "Publish Over SSH" plugin
2. Configure SSH server in Manage Jenkins → Configure System
3. Add SSH Server with:
   - Hostname
   - Username
   - Private key or password
   - Remote directory
4. Use "Send files over SSH" or "Execute shell script on remote host" build step
```

---

### Method 2: SSH Command in Shell (Simple)

**Direct SSH from Jenkins:**
```bash
# In Execute Shell build step
ssh user@storage-server "yum install -y $PACKAGE"

# With password (not recommended)
sshpass -p 'password' ssh user@storage-server "command"

# With key (recommended)
ssh -i /path/to/key user@storage-server "command"
```

**Requirements:**
- SSH client installed on Jenkins server
- SSH access configured (keys or password)
- Known_hosts file configured

---

### Method 3: SSH Agent Plugin

**SSH Agent Plugin:**
- Uses SSH credentials stored in Jenkins
- Provides SSH key in build environment
- Clean credential management

**Configuration:**
```groovy
// In Pipeline
sshagent(['storage-server-credentials']) {
    sh 'ssh user@storage "yum install -y ${PACKAGE}"'
}
```

---

## 🛠️ Task Implementation

### Step 1: Access Jenkins and Log In

**Click the "Jenkins" button on the top bar.**

**Or navigate to:**
```
http://<jenkins-server-ip>:8080
```

**Log in with admin credentials:**
- **Username:** `admin`
- **Password:** `Adm!n321`

**Expected:** Jenkins Dashboard appears

---

### Step 2: Check Required Plugins

Before creating the job, verify/install required plugins for SSH execution.

**Option A: Publish Over SSH Plugin (Recommended)**

1. Go to **Manage Jenkins** → **Plugins**
2. Click **Available plugins**
3. Search for: `Publish Over SSH`
4. Check the checkbox
5. Click **Install**
6. Select: **Restart Jenkins when installation is complete and no jobs are running**
7. Wait 30-60 seconds for restart
8. Login again with `admin` / `Adm!n321`

**Option B: SSH Agent Plugin (Alternative)**

1. Search for: `SSH Agent`
2. Install and restart

**Option C: No plugin (Use direct SSH)**

If SSH access is already configured and keys are in place, you can skip plugin installation and use direct SSH commands in shell build steps.

---

### Step 3: Configure SSH Access (If Using Plugin)

**If you installed Publish Over SSH plugin:**

1. Go to **Manage Jenkins** → **Configure System**
2. Scroll down to **Publish over SSH** section
3. Click **Add** under "SSH Servers"

**Configure SSH Server:**
```
Name: storage-server
Hostname: <storage-server-hostname-or-ip>
Username: <storage-server-username>
Remote Directory: /tmp

Authentication:
  Method 1 - Password:
    - Check "Use password authentication"
    - Password: <storage-server-password>
  
  Method 2 - Key (Recommended):
    - Path to key: /var/lib/jenkins/.ssh/id_rsa
    - Or paste key content in "Key" field
```

4. Click **Test Configuration** to verify connection
5. Expected: "Success" message
6. Click **Save**

**Note:** If you're not using the plugin, skip this step and use direct SSH in shell commands.

---

### Step 4: Create New Jenkins Job

**From Jenkins Dashboard:**

1. Click **New Item** (left sidebar)
2. **Enter job name:** `install-packages`
3. **Select:** Freestyle project
4. Click **OK**

**You'll be redirected to job configuration page.**

---

### Step 5: Configure String Parameter

**In the job configuration page:**

1. Scroll to **General** section
2. Check the box: **☑ This project is parameterized**

**Add String Parameter:**

3. Click **Add Parameter** dropdown
4. Select **String Parameter**

**Configure the parameter:**
```
┌──────────────────────────────────────────────┐
│  String Parameter                             │
├──────────────────────────────────────────────┤
│                                              │
│  Name:                                       │
│  [PACKAGE___________________________]        │
│                                              │
│  Default Value:                              │
│  [________________________________]          │
│                                              │
│  Description:                                │
│  [Name of the package to install on_]       │
│  [storage server (e.g., nginx, git,_]       │
│  [wget, curl, etc.)__________________]       │
│                                              │
│  ☐ Trim the string                          │
│                                              │
└──────────────────────────────────────────────┘
```

**Fill in:**
- **Name:** `PACKAGE`
- **Default Value:** (leave empty or set default like `vim`)
- **Description:** `Name of the package to install on storage server (e.g., nginx, git, wget, curl)`
- **Trim the string:** Check this box (removes leading/trailing whitespace)

**Note:** Parameter name must match exactly: `PACKAGE` (case-sensitive)

---

### Step 6: Configure Build Steps

Scroll down to the **Build** section.

**Option A: Using Publish Over SSH Plugin**

If you installed and configured the SSH plugin:

1. Click **Add build step** → **Send files or execute commands over SSH**

**Configure:**
```
SSH Server: storage-server (select from dropdown)

Transfers:
  Source files: (leave empty)
  Remove prefix: (leave empty)
  Remote directory: (leave empty)
  
  Exec command:
    sudo yum install -y $PACKAGE
```

**Alternative command (with error handling):**
```bash
#!/bin/bash
echo "Installing package: $PACKAGE"
sudo yum install -y $PACKAGE || sudo apt-get install -y $PACKAGE
echo "Package $PACKAGE installation completed"
```

2. Click **Save**

---

**Option B: Using Execute Shell (Direct SSH)**

If using direct SSH without plugin:

1. Click **Add build step** → **Execute shell**

**Enter the following script:**

```bash
#!/bin/bash

# Package installation script
echo "=========================================="
echo "Starting package installation"
echo "Package to install: $PACKAGE"
echo "Target: Storage Server"
echo "=========================================="

# SSH details (adjust these to your environment)
STORAGE_SERVER="<storage-server-ip-or-hostname>"
STORAGE_USER="<username>"
SSH_KEY="/var/lib/jenkins/.ssh/id_rsa"  # Or use password authentication

# Method 1: Using SSH key
ssh -i $SSH_KEY -o StrictHostKeyChecking=no $STORAGE_USER@$STORAGE_SERVER << EOF
    echo "Connected to storage server"
    echo "Installing package: $PACKAGE"
    
    # Try yum first (RHEL/CentOS)
    if command -v yum &> /dev/null; then
        sudo yum install -y $PACKAGE
        echo "Package installed successfully using yum"
    # Try apt if yum not available (Debian/Ubuntu)
    elif command -v apt-get &> /dev/null; then
        sudo apt-get update
        sudo apt-get install -y $PACKAGE
        echo "Package installed successfully using apt-get"
    else
        echo "Error: No supported package manager found"
        exit 1
    fi
    
    # Verify installation
    echo "Verifying installation..."
    if command -v $PACKAGE &> /dev/null; then
        echo "SUCCESS: $PACKAGE is installed and available"
        $PACKAGE --version 2>/dev/null || echo "$PACKAGE installed"
    else
        echo "Package installed but command not found (may be library/dependency)"
    fi
EOF

# Check SSH command exit status
if [ $? -eq 0 ]; then
    echo "=========================================="
    echo "Package installation completed successfully"
    echo "=========================================="
else
    echo "=========================================="
    echo "ERROR: Package installation failed"
    echo "=========================================="
    exit 1
fi
```

**Adjust the script:**
- Replace `<storage-server-ip-or-hostname>` with actual storage server address
- Replace `<username>` with SSH username
- Adjust SSH key path if needed
- Or use `sshpass` for password authentication (not recommended)

---

**Option C: Using sshpass (Password Authentication)**

If using password authentication:

```bash
#!/bin/bash

echo "Installing package: $PACKAGE on storage server"

# Storage server details
STORAGE_SERVER="<storage-server-ip>"
STORAGE_USER="<username>"
STORAGE_PASS="<password>"

# Install package using sshpass
sshpass -p "$STORAGE_PASS" ssh -o StrictHostKeyChecking=no $STORAGE_USER@$STORAGE_SERVER << EOF
    sudo yum install -y $PACKAGE || sudo apt-get install -y $PACKAGE
    echo "Package $PACKAGE installed successfully"
EOF

if [ $? -eq 0 ]; then
    echo "SUCCESS: Package installation completed"
else
    echo "ERROR: Package installation failed"
    exit 1
fi
```

**Security Note:** Using passwords in scripts is not recommended for production. Use SSH keys or Jenkins Credentials plugin instead.

---

### Step 7: Add Post-Build Actions (Optional but Recommended)

Scroll down to **Post-build Actions** section.

**Add email notification (if configured):**

1. Click **Add post-build action** → **E-mail Notification**
2. Recipients: `devops-team@example.com`
3. Check: **Send e-mail for every unstable build**
4. Check: **Send separate e-mails to individuals who broke the build**

**Or add build status notification:**

1. Click **Add post-build action** → **Editable Email Notification**
2. Configure custom email template with build parameters and status

---

### Step 8: Save the Job Configuration

1. Scroll to the bottom of the page
2. Click **Save**

**You'll be redirected to the job page.**

---

### Step 9: Test the Job - First Execution

**Build the job with a test package:**

1. On the job page, click **Build with Parameters** (left sidebar)

**You'll see a parameter form:**
```
┌──────────────────────────────────────────────┐
│  Build with Parameters                        │
├──────────────────────────────────────────────┤
│                                              │
│  PACKAGE                                     │
│  [vim_______________________________]        │
│                                              │
│  Name of the package to install on           │
│  storage server (e.g., nginx, git,           │
│  wget, curl)                                 │
│                                              │
│              [Build]                         │
│                                              │
└──────────────────────────────────────────────┘
```

2. **Enter test package name:** `vim`
3. Click **Build**

**Monitor the build:**

4. In **Build History** (left sidebar), you'll see **#1** appear
5. Click on the build number **#1**
6. Click **Console Output** to view real-time logs

**Expected output:**
```
Started by user admin
Running as SYSTEM
Building in workspace /var/lib/jenkins/workspace/install-packages
[install-packages] $ /bin/sh -xe /tmp/jenkins1234567890.sh
+ echo ==========================================
==========================================
+ echo Starting package installation
Starting package installation
+ echo Package to install: vim
Package to install: vim
+ echo Target: Storage Server
Target: Storage Server
+ echo ==========================================
==========================================
+ ssh storage-user@storage-server
Connected to storage server
Installing package: vim
Loaded plugins: fastestmirror, ovl
Loading mirror speeds from cached hostfile
Resolving Dependencies
--> Running transaction check
---> Package vim-enhanced.x86_64 2:7.4.629-8.el7_9 will be installed
--> Finished Dependency Resolution

Dependencies Resolved
Package installed successfully using yum
Verifying installation...
SUCCESS: vim is installed and available
VIM - Vi IMproved 7.4 (2013 Aug 10, compiled Jun 23 2020)
==========================================
Package installation completed successfully
==========================================
Finished: SUCCESS
```

**Result:** Build should show blue ball (success) ✅

---

### Step 10: Test with Different Packages

**Test job repeatability with various packages:**

**Test 2: Install Git**
1. Click **Build with Parameters**
2. Enter: `git`
3. Click **Build**
4. Verify: #2 succeeds

**Test 3: Install wget**
1. Click **Build with Parameters**
2. Enter: `wget`
3. Click **Build**
4. Verify: #3 succeeds

**Test 4: Install curl**
1. Click **Build with Parameters**
2. Enter: `curl`
3. Click **Build**
4. Verify: #4 succeeds

**Test 5: Install nginx**
1. Click **Build with Parameters**
2. Enter: `nginx`
3. Click **Build**
4. Verify: #5 succeeds

**Build History should show:**
```
#5  ● (blue) - nginx
#4  ● (blue) - curl
#3  ● (blue) - wget
#2  ● (blue) - git
#1  ● (blue) - vim
```

All builds should succeed (blue) ✅

---

### Step 11: Verify on Storage Server

**SSH to storage server and verify packages are installed:**

```bash
# Connect to storage server
ssh user@storage-server

# Check installed packages
rpm -qa | grep -E "vim|git|wget|curl|nginx"

# Or for Debian/Ubuntu
dpkg -l | grep -E "vim|git|wget|curl|nginx"

# Verify commands are available
which vim
which git
which wget
which curl
which nginx

# Check versions
vim --version
git --version
wget --version
curl --version
nginx -v
```

**Expected:** All packages should be installed and available ✅

---

### Step 12: Test Error Handling

**Test with invalid package name:**

1. Click **Build with Parameters**
2. Enter: `this-package-does-not-exist-12345`
3. Click **Build**

**Expected:** Build should fail (red ball) with error message ❌
```
No package this-package-does-not-exist-12345 available.
Error: Nothing to do
ERROR: Package installation failed
Build step 'Execute shell' marked build as failure
Finished: FAILURE
```

**This confirms error handling works correctly.**

---

### Step 13: Configure Job Description (Optional)

**Add helpful description to the job:**

1. Go to job page: **install-packages**
2. Click **Configure** (left sidebar)
3. In **Description** field, add:

```
Automated Package Installation Job

This job installs packages on the storage server in Stratos Datacenter.

Parameters:
- PACKAGE: Name of the package to install (e.g., nginx, git, vim)

Usage:
1. Click "Build with Parameters"
2. Enter package name
3. Click "Build"
4. Monitor console output for installation status

Supported package managers:
- yum (RHEL/CentOS)
- apt-get (Debian/Ubuntu)

Target Server: Storage Server (ststor01)

Note: Ensure the package name is correct and available in the repository.
```

4. Click **Save**

**Now the job page displays helpful information for other team members.**

---

## 📊 Complete Job Configuration Script

**Full Execute Shell script with comprehensive error handling and logging:**

```bash
#!/bin/bash

#############################################
# Jenkins Job: install-packages
# Description: Install packages on storage server
# Parameter: PACKAGE (string)
#############################################

# Color codes for output
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m' # No Color

# Configuration
STORAGE_SERVER="ststor01.stratos.xfusioncorp.com"  # Adjust to your environment
STORAGE_USER="natasha"  # Adjust to your environment
SSH_KEY="/var/lib/jenkins/.ssh/id_rsa"  # Adjust if using different key
TIMEOUT=300  # 5 minutes timeout

# Logging function
log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1"
}

# Error handling
set -e
trap 'log "ERROR: Script failed at line $LINENO"' ERR

log "=========================================="
log "Jenkins Job: Package Installation"
log "=========================================="

# Validate parameter
if [ -z "$PACKAGE" ]; then
    log "${RED}ERROR: PACKAGE parameter is empty${NC}"
    log "Please provide a package name when building the job"
    exit 1
fi

log "Package to install: ${GREEN}$PACKAGE${NC}"
log "Target server: $STORAGE_SERVER"
log "SSH user: $STORAGE_USER"
log "=========================================="

# Check if SSH key exists
if [ ! -f "$SSH_KEY" ]; then
    log "${YELLOW}WARNING: SSH key not found at $SSH_KEY${NC}"
    log "Attempting connection without explicit key specification..."
fi

# Test SSH connection
log "Testing SSH connection to storage server..."
if timeout 10 ssh -o StrictHostKeyChecking=no -o ConnectTimeout=5 $STORAGE_USER@$STORAGE_SERVER "echo 'Connection successful'" 2>/dev/null; then
    log "${GREEN}SSH connection test passed${NC}"
else
    log "${RED}ERROR: Cannot connect to storage server${NC}"
    log "Please verify:"
    log "  - Storage server is reachable"
    log "  - SSH credentials are correct"
    log "  - Firewall allows SSH connection"
    exit 1
fi

log "=========================================="
log "Starting package installation on storage server..."
log "=========================================="

# Execute installation on remote server
ssh -o StrictHostKeyChecking=no $STORAGE_USER@$STORAGE_SERVER << 'ENDSSH'
    #!/bin/bash
    
    # Export parameter to remote session
    PACKAGE="'"$PACKAGE"'"
    
    echo "Connected to storage server: $(hostname)"
    echo "Current user: $(whoami)"
    echo "----------------------------------------"
    
    # Detect package manager
    if command -v yum &> /dev/null; then
        PKG_MANAGER="yum"
        INSTALL_CMD="sudo yum install -y"
        CHECK_CMD="rpm -q"
    elif command -v apt-get &> /dev/null; then
        PKG_MANAGER="apt-get"
        INSTALL_CMD="sudo apt-get install -y"
        CHECK_CMD="dpkg -l"
        # Update package lists for apt
        echo "Updating package lists..."
        sudo apt-get update -qq
    else
        echo "ERROR: No supported package manager found"
        echo "Supported: yum (RHEL/CentOS) or apt-get (Debian/Ubuntu)"
        exit 1
    fi
    
    echo "Detected package manager: $PKG_MANAGER"
    echo "----------------------------------------"
    
    # Check if package is already installed
    echo "Checking if package is already installed..."
    if $CHECK_CMD $PACKAGE &> /dev/null; then
        echo "Package $PACKAGE is already installed"
        echo "Attempting to update to latest version..."
    fi
    
    # Install/update package
    echo "Installing package: $PACKAGE"
    echo "Command: $INSTALL_CMD $PACKAGE"
    echo "----------------------------------------"
    
    if $INSTALL_CMD $PACKAGE; then
        echo "----------------------------------------"
        echo "Package installation command completed"
    else
        echo "----------------------------------------"
        echo "ERROR: Package installation command failed"
        exit 1
    fi
    
    # Verify installation
    echo "----------------------------------------"
    echo "Verifying installation..."
    
    if $CHECK_CMD $PACKAGE &> /dev/null; then
        echo "SUCCESS: Package $PACKAGE is installed"
        
        # Try to show version if command exists
        if command -v $PACKAGE &> /dev/null; then
            echo "Command location: $(which $PACKAGE)"
            echo "Attempting to show version..."
            $PACKAGE --version 2>/dev/null || $PACKAGE -v 2>/dev/null || $PACKAGE version 2>/dev/null || echo "Version information not available"
        else
            echo "Note: $PACKAGE command not found in PATH (may be a library or daemon)"
        fi
    else
        echo "WARNING: Package verification failed"
        echo "Package may not have installed correctly"
        exit 1
    fi
    
    echo "----------------------------------------"
    echo "Package installation completed successfully"
ENDSSH

# Capture SSH exit status
SSH_EXIT=$?

log "=========================================="

if [ $SSH_EXIT -eq 0 ]; then
    log "${GREEN}✓ SUCCESS: Package installation completed${NC}"
    log "Package: $PACKAGE"
    log "Server: $STORAGE_SERVER"
    log "Status: Installed and verified"
    log "=========================================="
    exit 0
else
    log "${RED}✗ FAILURE: Package installation failed${NC}"
    log "Package: $PACKAGE"
    log "Server: $STORAGE_SERVER"
    log "Exit code: $SSH_EXIT"
    log "=========================================="
    log "Troubleshooting steps:"
    log "1. Verify package name is correct"
    log "2. Check package is available in repository"
    log "3. Ensure sudo privileges for $STORAGE_USER"
    log "4. Check storage server disk space"
    log "5. Review console output above for detailed error"
    log "=========================================="
    exit 1
fi
```

**This script provides:**
- ✅ Parameter validation
- ✅ SSH connection testing
- ✅ Package manager detection
- ✅ Already-installed check
- ✅ Installation verification
- ✅ Comprehensive logging
- ✅ Error handling
- ✅ Exit codes
- ✅ Troubleshooting hints

---

## 🧪 Testing Scenarios

### Test Case 1: Common Package Installation

**Packages to test:**
```bash
# Development tools
vim, git, wget, curl, nano

# Web servers
nginx, httpd, apache2

# Databases
mysql, mariadb, postgresql

# System utilities
htop, net-tools, bind-utils, telnet

# Programming languages
python3, nodejs, ruby, golang
```

**Expected:** All should install successfully ✅

---

### Test Case 2: Already Installed Package

**Steps:**
1. Build with `PACKAGE=vim` (first time)
2. Build with `PACKAGE=vim` again (second time)

**Expected:**
- First build: Installs vim
- Second build: Reports already installed, may update

Both should succeed ✅

---

### Test Case 3: Invalid Package Name

**Test with:**
- `this-package-does-not-exist`
- `invalid_pkg_name_12345`
- `random-package-xyz`

**Expected:**
- Build fails (red ball) ❌
- Error message: "No package available"
- Job returns exit code 1

---

### Test Case 4: Empty Parameter

**Steps:**
1. Click "Build with Parameters"
2. Leave PACKAGE field empty
3. Click Build

**Expected:**
- Build fails immediately ❌
- Error: "PACKAGE parameter is empty"
- Validation catches error before SSH

---

### Test Case 5: Package with Dependencies

**Test with packages that have dependencies:**
- `nginx` (requires many libs)
- `docker` (requires containerd, etc.)
- `postgresql` (requires libs and tools)

**Expected:**
- Package manager resolves dependencies
- All dependencies installed automatically
- Build succeeds ✅

---

### Test Case 6: Rapid Successive Builds

**Steps:**
1. Start build with `PACKAGE=vim`
2. Immediately start another with `PACKAGE=git`
3. Start another with `PACKAGE=wget`

**Expected:**
- All builds queue properly
- Each build executes in order
- No conflicts or race conditions
- All succeed ✅

---

## 📊 Build History Analysis

**After multiple test runs, your build history should look like:**

```
Build History:
#12 ● wget (10 sec) - SUCCESS
#11 ● git (8 sec) - SUCCESS
#10 ● vim (12 sec) - SUCCESS (re-run)
#9  ● nginx (45 sec) - SUCCESS (many dependencies)
#8  ● curl (7 sec) - SUCCESS
#7  ● invalid-pkg ❌ (3 sec) - FAILURE (expected)
#6  ● htop (15 sec) - SUCCESS
#5  ● nginx (40 sec) - SUCCESS
#4  ● curl (8 sec) - SUCCESS
#3  ● wget (9 sec) - SUCCESS
#2  ● git (10 sec) - SUCCESS
#1  ● vim (11 sec) - SUCCESS

Success Rate: 91% (10/11 successful, 1 intentional failure)
Average Duration: 13 seconds (excluding failures)
```

**Metrics to track:**
- Success rate (should be high)
- Average build time
- Failed builds (analyze why)
- Most installed packages

---

## 🐛 Common Issues and Solutions

### Issue 1: SSH Connection Refused

**Symptoms:**
```
ssh: connect to host storage-server port 22: Connection refused
ERROR: Cannot connect to storage server
Build failed
```

**Diagnosis:**
- Storage server is down
- SSH service not running
- Firewall blocking port 22
- Wrong hostname/IP

**Solution:**

```bash
# Test from Jenkins server command line
ssh user@storage-server

# Check if server is reachable
ping storage-server

# Check if SSH port is open
telnet storage-server 22
nc -zv storage-server 22

# Verify hostname resolution
nslookup storage-server
dig storage-server

# If hostname doesn't resolve, use IP address instead
# Update job configuration with IP
```

---

### Issue 2: Permission Denied (SSH)

**Symptoms:**
```
Permission denied (publickey,password)
ERROR: SSH authentication failed
```

**Diagnosis:**
- SSH key not configured
- Wrong username
- Password authentication disabled
- Key not in authorized_keys

**Solution A: Fix SSH Key**

```bash
# On Jenkins server
# Generate SSH key if not exists
sudo -u jenkins ssh-keygen -t rsa -b 4096 -N "" -f /var/lib/jenkins/.ssh/id_rsa

# Copy public key to storage server
sudo -u jenkins ssh-copy-id user@storage-server

# Or manually add to authorized_keys
cat /var/lib/jenkins/.ssh/id_rsa.pub
# Copy output and add to storage server's ~/.ssh/authorized_keys
```

**Solution B: Use Password Authentication**

```bash
# In job configuration, use sshpass
sshpass -p 'password' ssh user@storage-server "command"

# Note: Less secure, not recommended for production
```

---

### Issue 3: Sudo Password Required

**Symptoms:**
```
[sudo] password for user:
sudo: no tty present and no askpass program specified
ERROR: Package installation failed
```

**Diagnosis:**
- User doesn't have passwordless sudo
- TTY required for sudo
- Sudo configuration not set up

**Solution:**

```bash
# On storage server, add user to sudoers with NOPASSWD
sudo visudo

# Add this line:
username ALL=(ALL) NOPASSWD: /usr/bin/yum, /usr/bin/apt-get

# Or for all commands (less secure):
username ALL=(ALL) NOPASSWD: ALL

# Save and exit
```

**Verify:**
```bash
# Should not prompt for password
sudo yum list installed
```

---

### Issue 4: Package Not Found

**Symptoms:**
```
No package nginx available.
Error: Nothing to do
```

**Diagnosis:**
- Package name is incorrect
- Package not in enabled repositories
- Repository needs update
- Package name differs between distros

**Solution:**

```bash
# Search for correct package name
yum search nginx
apt-cache search nginx

# Enable additional repositories (RHEL/CentOS)
sudo yum install epel-release
sudo yum update

# Update package lists (Debian/Ubuntu)
sudo apt-get update

# Check available repositories
yum repolist
apt-cache policy
```

**Common package name differences:**
```
RHEL/CentOS    | Debian/Ubuntu
---------------|---------------
httpd          | apache2
mariadb-server | mariadb-server
net-tools      | net-tools
bind-utils     | dnsutils
```

---

### Issue 5: Disk Space Full

**Symptoms:**
```
Error: Disk Requirements:
  At least XXX MB more space needed on the / filesystem.
Transaction failed
```

**Diagnosis:**
- Storage server disk is full
- /var partition full
- Cache taking up space

**Solution:**

```bash
# Check disk space
df -h

# Clean yum cache
sudo yum clean all

# Clean apt cache
sudo apt-get clean
sudo apt-get autoclean

# Remove old kernels (be careful!)
sudo package-cleanup --oldkernels --count=1

# Find large files
sudo du -sh /* | sort -hr | head -10

# Remove old logs if needed
sudo journalctl --vacuum-size=100M
```

---

### Issue 6: Job Always Succeeds Even When Package Fails

**Symptoms:**
- Build shows SUCCESS (blue ball)
- But package is not actually installed
- Console output shows errors, but job passes

**Diagnosis:**
- Script doesn't have `set -e`
- Exit codes not checked
- SSH command doesn't propagate errors

**Solution:**

```bash
# Add to beginning of script
set -e  # Exit on any error

# Check SSH exit code
ssh user@server "command"
if [ $? -ne 0 ]; then
    echo "ERROR: Command failed"
    exit 1
fi

# Use && for command chaining
ssh user@server "command1 && command2 && command3"

# Or in heredoc, add set -e
ssh user@server << 'EOF'
    set -e
    command1
    command2
    command3
EOF
```

---

### Issue 7: Parameter Not Being Passed to Remote Server

**Symptoms:**
- Parameter works in Jenkins script
- But is empty on remote server
- Package name shows as blank

**Diagnosis:**
- Environment variable not exported to SSH session
- Single quotes prevent variable expansion
- Variable scope issue

**Solution:**

```bash
# Method 1: Pass variable explicitly
ssh user@server "PACKAGE=$PACKAGE; sudo yum install -y \$PACKAGE"

# Method 2: Use heredoc with variable substitution
ssh user@server << ENDSSH
    PACKAGE="$PACKAGE"
    sudo yum install -y \$PACKAGE
ENDSSH

# Method 3: Pass as SSH command
ssh user@server bash -s << 'EOF'
    PACKAGE="$1"
    sudo yum install -y $PACKAGE
EOF
```

**Note:** Quote handling is critical - double quotes for outer, single/escaped for inner.

---

## 📖 Key Takeaways

### Concepts Mastered Today:

1. ✅ **Parameterized Builds**
   - Creating flexible, reusable Jenkins jobs
   - String parameters for dynamic input
   - Parameter validation and error handling
   - Best practices for parameter naming

2. ✅ **Remote Command Execution**
   - SSH from Jenkins to remote servers
   - Multiple methods (plugin vs direct SSH)
   - SSH key management and authentication
   - Secure credential handling

3. ✅ **Build Steps Configuration**
   - Execute Shell build steps
   - Publish Over SSH plugin usage
   - Script development in Jenkins
   - Error handling and exit codes

4. ✅ **Job Reliability**
   - Testing with multiple packages
   - Error scenarios and handling
   - Repeatability verification
   - Build history analysis

5. ✅ **Automation Best Practices**
   - Comprehensive logging
   - Parameter validation
   - Connection testing
   - Verification steps

---

## 🎯 Real-World Applications

### Production Use Cases

**1. Infrastructure Package Management**
```
Use Case: Maintain consistent packages across fleet
Job Parameters:
- PACKAGE: Package name
- SERVERS: Comma-separated server list
- ACTION: install/update/remove

Example: Install monitoring agents on all app servers
```

**2. Compliance Automation**
```
Use Case: Ensure security patches installed
Job Parameters:
- SECURITY_PACKAGE: CVE patch package
- ENVIRONMENT: prod/staging/dev
- SCHEDULE: Automated daily run

Example: Auto-install critical security updates
```

**3. Development Environment Setup**
```
Use Case: Standardize dev environments
Job Parameters:
- PACKAGE_LIST: dev-tools/web-stack/data-stack
- DEVELOPER: Username

Example: Install standard developer tools on new workstation
```

**4. Dependency Installation for Deployments**
```
Use Case: Pre-install dependencies before app deployment
Job Parameters:
- APPLICATION: app name
- VERSION: app version
- DEPENDENCIES: Required system packages

Example: Install nginx + php-fpm before deploying PHP app
```

---

### Enterprise Patterns

**Pattern 1: Cascading Jobs**
```
Job Chain:
1. install-packages (system packages)
   ↓
2. configure-packages (config files)
   ↓
3. start-services (systemctl start)
   ↓
4. verify-services (health checks)

Use Trigger: "Build other projects" post-build action
```

**Pattern 2: Multi-Server Installation**
```yaml
# Enhanced job with server list
Parameters:
  - PACKAGE: Package name
  - SERVER_GROUP: all/app-servers/db-servers/web-servers

Script:
for server in $(get_servers $SERVER_GROUP); do
    ssh $server "yum install -y $PACKAGE"
done
```

**Pattern 3: Version-Specific Installation**
```yaml
# Install specific package version
Parameters:
  - PACKAGE: Base package name
  - VERSION: Specific version (optional)

Example Build:
  PACKAGE=nginx
  VERSION=1.20.1

Command:
  yum install -y $PACKAGE-$VERSION
```

**Pattern 4: Rollback Capability**
```bash
# Before installation, backup current version
yum list installed $PACKAGE > /backup/before.txt

# Install new version
yum install -y $PACKAGE

# If failure, rollback
yum history rollback last
```

---

### Integration with CI/CD Pipeline

**Full Pipeline Integration:**
```
Developer commits code
    ↓
Jenkins detects commit (webhook)
    ↓
Run build job
    ↓
Run test job
    ↓
Trigger install-packages (dependencies)
    ↓
Deploy application
    ↓
Run smoke tests
    ↓
Notify team (Slack/Email)
```

**Parametrized Deployment:**
```groovy
pipeline {
    stages {
        stage('Install Dependencies') {
            steps {
                build job: 'install-packages',
                    parameters: [
                        string(name: 'PACKAGE', value: 'nginx')
                    ]
            }
        }
        stage('Deploy App') {
            steps {
                // Deploy application code
            }
        }
    }
}
```

---

## 📚 Additional Resources

**Official Documentation:**
- [Jenkins Parameterized Builds](https://www.jenkins.io/doc/book/pipeline/syntax/#parameters)
- [Publish Over SSH Plugin](https://plugins.jenkins.io/publish-over-ssh/)
- [SSH Agent Plugin](https://plugins.jenkins.io/ssh-agent/)
- [Jenkins Build Environment](https://www.jenkins.io/doc/book/pipeline/jenkinsfile/#using-environment-variables)

**Tutorials:**
- [Creating Parameterized Jobs](https://www.jenkins.io/doc/book/pipeline/syntax/#string)
- [Remote Execution Best Practices](https://www.jenkins.io/doc/book/managing/security/#ssh-agents)
- [Error Handling in Jenkins](https://www.jenkins.io/doc/book/pipeline/syntax/#post)

**Security Best Practices:**
- [Jenkins Credentials Plugin](https://plugins.jenkins.io/credentials/)
- [SSH Key Management](https://www.jenkins.io/doc/book/using/using-credentials/)
- [Securing Jenkins](https://www.jenkins.io/doc/book/security/securing-jenkins/)

**Next Steps:**
- **Day 72:** Jenkins Pipeline as Code (Jenkinsfile)
- **Day 73:** Jenkins + GitHub Webhook Integration
- **Day 74:** Jenkins Multi-Branch Pipeline
- **Day 75:** Jenkins + Docker Integration

---

## ✅ Task Completion Checklist

**Job Creation:**
- [ ] Logged into Jenkins (admin / Adm!n321)
- [ ] Created new job named `install-packages`
- [ ] Selected Freestyle project type
- [ ] Job created successfully

**Parameter Configuration:**
- [ ] Checked "This project is parameterized"
- [ ] Added String Parameter
- [ ] Parameter name set to `PACKAGE`
- [ ] Description added for clarity
- [ ] "Trim the string" option enabled

**Plugin Installation (if needed):**
- [ ] Checked for Publish Over SSH plugin
- [ ] Installed plugin if missing
- [ ] Restarted Jenkins service
- [ ] Plugin verified in installed list

**SSH Configuration:**
- [ ] SSH access to storage server verified
- [ ] SSH keys configured or password auth set up
- [ ] Sudo privileges confirmed for package installation
- [ ] Connection tested successfully

**Build Steps Configuration:**
- [ ] Added build step (SSH or Execute Shell)
- [ ] Script configured to install package on remote server
- [ ] Package manager detection added (yum/apt)
- [ ] Error handling implemented
- [ ] Verification steps included

**Testing:**
- [ ] First test build executed (e.g., vim)
- [ ] Build succeeded with blue ball
- [ ] Console output reviewed
- [ ] Package verified on storage server

**Multiple Test Runs:**
- [ ] Tested with vim - SUCCESS ✅
- [ ] Tested with git - SUCCESS ✅
- [ ] Tested with wget - SUCCESS ✅
- [ ] Tested with curl - SUCCESS ✅
- [ ] Tested with nginx - SUCCESS ✅
- [ ] Tested with invalid package - EXPECTED FAILURE ❌
- [ ] All successful builds show blue ball
- [ ] Build history shows multiple successful runs

**Error Handling Verification:**
- [ ] Tested with invalid package name
- [ ] Build failed as expected (red ball)
- [ ] Error message displayed clearly
- [ ] Job returned non-zero exit code

**Reliability Verification:**
- [ ] Re-ran same package multiple times
- [ ] Job handles already-installed packages gracefully
- [ ] No race conditions with rapid successive builds
- [ ] Success rate is high (>90%)

**Documentation:**
- [ ] Screenshots captured of:
  - Job configuration page
  - Parameter configuration
  - Build step configuration
  - Successful build console output
  - Build history with multiple runs
  - Parameter input form
- [ ] Or screen recording created (loom.com)
- [ ] Documentation includes all configurations

**Verification on Storage Server:**
- [ ] SSHed to storage server
- [ ] Verified packages are installed
- [ ] Checked package versions
- [ ] Commands available in PATH

---

**🎉 Congratulations!** You've successfully created a parameterized Jenkins job for automated package installation! You've mastered:
- Parameterized builds for flexible automation
- Remote command execution via SSH
- Build step configuration and scripting
- Error handling and validation
- Job testing and reliability verification

This job can now be used by your team to quickly install packages on infrastructure servers without manual SSH access!

**Day 71 Status:** ✅ Complete

**Next:** Day 72 - Jenkins Pipeline as Code with Jenkinsfile! 🚀
