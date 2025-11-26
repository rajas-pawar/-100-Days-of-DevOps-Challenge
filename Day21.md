# Day 21: Setting Up Git Repository on Storage Server
**100 Days Cloud DevOps Challenge – KodeKloud**

---

## Objective

The Nautilus development team has requested the DevOps team to establish a Git repository for a new application development project. We need to create a Git repository on the Storage server in Stratos DC.

**Requirements:**
- Install `git` package using `yum` on the Storage Server
- Create a bare repository named `/opt/ecommerce.git`
- Ensure exact naming convention is followed

---

## Infrastructure Overview

### Storage Server:
| Server | User   | Password     | IP            |
|--------|--------|--------------|---------------|
| ststor01 | natasha | Bl@kW | 172.16.238.15 |

**Note:** Storage server details may vary. Check your lab environment for exact credentials.

---

## What is a Bare Git Repository?

A **bare repository** is a Git repository that doesn't have a working directory. It only contains the version control information (the `.git` directory contents).

**Key Differences:**

| Feature | Normal Repository | Bare Repository |
|---------|------------------|-----------------|
| Working Directory | Yes | No |
| Can commit directly | Yes | No |
| Used for | Development | Central repository/sharing |
| Contains | Files + .git folder | Only .git contents |
| Extension | None | Usually .git |

**Use Cases:**
- Central repositories (like GitHub/GitLab servers)
- Shared repositories for team collaboration
- Repository hosting on servers

---

## Step-by-Step Implementation

### Step 1: SSH into Storage Server
```bash
ssh natasha@ststor01
```

**Enter password when prompted** (typically: `Bl@kW` or check your lab details)

### Step 2: Switch to Root User
```bash
sudo su -
```

### Step 3: Install Git
```bash
yum install -y git
```

**Expected output:**
```
Installed:
  git-x.x.x...

Complete!
```

### Step 4: Verify Git Installation
```bash
git --version
```

**Expected output:**
```
git version 2.x.x
```

### Step 5: Create Bare Repository
```bash
git init --bare /opt/ecommerce.git
```

**Expected output:**
```
Initialized empty Git repository in /opt/ecommerce.git/
```

### Step 6: Verify Repository Structure
```bash
ls -la /opt/ecommerce.git/
```

**Expected output:**
```
total 16
drwxr-xr-x. 7 root root  119 Nov 25 10:00 .
drwxr-xr-x. 3 root root   28 Nov 25 10:00 ..
drwxr-xr-x. 2 root root    6 Nov 25 10:00 branches
-rw-r--r--. 1 root root   66 Nov 25 10:00 config
-rw-r--r--. 1 root root   73 Nov 25 10:00 description
-rw-r--r--. 1 root root   23 Nov 25 10:00 HEAD
drwxr-xr-x. 2 root root 4096 Nov 25 10:00 hooks
drwxr-xr-x. 2 root root   21 Nov 25 10:00 info
drwxr-xr-x. 4 root root   30 Nov 25 10:00 objects
drwxr-xr-x. 4 root root   31 Nov 25 10:00 refs
```

### Step 7: Verify Repository Configuration
```bash
cat /opt/ecommerce.git/config
```

**Expected output:**
```
[core]
        repositoryformatversion = 0
        filemode = true
        bare = true
```

**Note:** The `bare = true` confirms it's a bare repository.

### Step 8: Check HEAD Reference
```bash
cat /opt/ecommerce.git/HEAD
```

**Expected output:**
```
ref: refs/heads/master
```

Or for newer Git versions:
```
ref: refs/heads/main
```

---

## Complete Command Summary

### All Commands in One Block:
```bash
# SSH into storage server
ssh natasha@ststor01

# Switch to root
sudo su -

# Install git
yum install -y git

# Verify installation
git --version

# Create bare repository
git init --bare /opt/ecommerce.git

# Verify repository
ls -la /opt/ecommerce.git/
cat /opt/ecommerce.git/config

# Confirm it's a bare repo
grep "bare = true" /opt/ecommerce.git/config
```

---

## Verification Steps

### Verify 1: Check Git Installation
```bash
which git
```

**Expected output:**
```
/usr/bin/git
```

### Verify 2: Check Repository Type
```bash
git -C /opt/ecommerce.git config --get core.bare
```

**Expected output:**
```
true
```

### Verify 3: Check Repository Location and Permissions
```bash
ls -ld /opt/ecommerce.git
```

**Expected output:**
```
drwxr-xr-x. 7 root root 119 Nov 25 10:00 /opt/ecommerce.git
```

### Verify 4: List Repository Contents
```bash
tree /opt/ecommerce.git
```

**Expected structure:**
```
/opt/ecommerce.git
├── branches
├── config
├── description
├── HEAD
├── hooks
│   ├── applypatch-msg.sample
│   ├── commit-msg.sample
│   └── ...
├── info
│   └── exclude
├── objects
│   ├── info
│   └── pack
└── refs
    ├── heads
    └── tags
```

---

## Testing the Repository

### Test 1: Clone the Repository (Optional)

From another location or the jump host:
```bash
# Create a test directory
mkdir -p /tmp/test-clone
cd /tmp/test-clone

# Clone the bare repository
git clone /opt/ecommerce.git
```

**Expected output:**
```
Cloning into 'ecommerce'...
warning: You appear to have cloned an empty repository.
done.
```

### Test 2: Add a Test File and Push
```bash
# Enter the cloned directory
cd ecommerce

# Configure git (if needed)
git config user.name "Test User"
git config user.email "test@example.com"

# Create a test file
echo "# Ecommerce Project" > README.md

# Add and commit
git add README.md
git commit -m "Initial commit"

# Push to bare repository
git push origin master
```

Or for newer Git versions:
```bash
git push origin main
```

### Test 3: Verify Objects in Bare Repository
```bash
ls -la /opt/ecommerce.git/objects/
```

After pushing, you should see new object directories.

---

## Troubleshooting

### Issue 1: Git Command Not Found

**Problem:**
```
-bash: git: command not found
```

**Solution:**
```bash
# Install git
yum install -y git

# Verify installation
which git
git --version
```

### Issue 2: Permission Denied

**Problem:**
```
fatal: could not create work tree dir 'ecommerce': Permission denied
```

**Solution:**
```bash
# Ensure you're root or have proper permissions
sudo su -

# Or set proper permissions
chmod 755 /opt/ecommerce.git
```

### Issue 3: Repository Already Exists

**Problem:**
```
fatal: destination path '/opt/ecommerce.git' already exists
```

**Solution:**
```bash
# Check if repository exists
ls -la /opt/ecommerce.git

# If you need to recreate, remove existing one
rm -rf /opt/ecommerce.git

# Create new bare repository
git init --bare /opt/ecommerce.git
```

### Issue 4: Not a Bare Repository

**Problem:** Repository created without `--bare` flag

**Verification:**
```bash
cat /opt/ecommerce.git/config | grep bare
```

**Solution:**
```bash
# Remove incorrectly created repository
rm -rf /opt/ecommerce.git

# Create bare repository correctly
git init --bare /opt/ecommerce.git

# Verify
git -C /opt/ecommerce.git config --get core.bare
```

### Issue 5: Wrong Directory Name

**Problem:** Repository created with wrong name (e.g., `ecommerce` instead of `ecommerce.git`)

**Solution:**
```bash
# Remove incorrect directory
rm -rf /opt/ecommerce

# Create with correct name
git init --bare /opt/ecommerce.git
```

---

## Setting Up Repository Permissions (Optional)

For team access, you might want to set up proper permissions:
```bash
# Create a git group
groupadd git

# Add users to git group
usermod -aG git natasha

# Change repository ownership
chown -R root:git /opt/ecommerce.git

# Set permissions
chmod -R 775 /opt/ecommerce.git

# Set SGID bit so new files inherit group
find /opt/ecommerce.git -type d -exec chmod g+s {} \;
```

---

## Git Repository Configuration

### View Repository Configuration
```bash
cat /opt/ecommerce.git/config
```

### Common Configuration Options
```bash
# Set repository description
echo "Ecommerce Application Repository" > /opt/ecommerce.git/description

# Enable shared repository mode (for multi-user access)
git -C /opt/ecommerce.git config core.sharedRepository group

# Set default branch name
git -C /opt/ecommerce.git symbolic-ref HEAD refs/heads/main
```

---

## Understanding Bare Repository Structure

### Key Directories and Files:

| Path | Description |
|------|-------------|
| `HEAD` | Points to current branch (default: refs/heads/master or main) |
| `config` | Repository-specific configuration |
| `description` | Repository description (for GitWeb) |
| `hooks/` | Server-side Git hooks |
| `info/` | Global exclude file and other info |
| `objects/` | Stores all Git objects (commits, trees, blobs) |
| `refs/` | References to commits (branches, tags) |
| `refs/heads/` | Branch references |
| `refs/tags/` | Tag references |

### Important Files:
```bash
# View HEAD reference
cat /opt/ecommerce.git/HEAD

# View config
cat /opt/ecommerce.git/config

# View description
cat /opt/ecommerce.git/description

# List hooks
ls -la /opt/ecommerce.git/hooks/
```

---

## Git Commands Reference

### Basic Git Commands

| Command | Description |
|---------|-------------|
| `git --version` | Show Git version |
| `git init` | Initialize regular repository |
| `git init --bare` | Initialize bare repository |
| `git clone <repo>` | Clone a repository |
| `git config --list` | List all configurations |
| `git config <key> <value>` | Set configuration |

### Repository Management

| Command | Description |
|---------|-------------|
| `git -C <path> <command>` | Run git command in specific directory |
| `git config --get core.bare` | Check if repository is bare |
| `git symbolic-ref HEAD` | Show current HEAD reference |
| `git show-ref` | List all references |

---

## Key Takeaways

- **Bare repositories** don't have working directories - they only store Git history
- **Naming convention**: Bare repositories typically use `.git` extension
- **Purpose**: Bare repositories are used as central/shared repositories
- **Installation**: Git can be easily installed using `yum install git`
- **Verification**: Always verify with `git config --get core.bare` to confirm it's bare
- **Structure**: Bare repositories contain the same contents as `.git` folder in normal repos
- **Use case**: Perfect for server-side repositories where no direct commits are made
- **Permissions**: Consider setting proper group permissions for team access

---

## Common Use Cases

### 1. Team Development
```bash
# On server (already done)
git init --bare /opt/ecommerce.git

# Developers clone from server
git clone natasha@ststor01:/opt/ecommerce.git
cd ecommerce
# ... make changes ...
git push origin main
```

### 2. Backup Repository
```bash
# Clone existing repository as bare backup
git clone --bare <source-repo> /opt/ecommerce.git
```

### 3. Mirror Repository
```bash
# Create mirror of existing repository
git clone --mirror <source-repo> /opt/ecommerce.git
```

---

## Security Considerations

### 1. Access Control
```bash
# Limit access to specific users/groups
chown -R root:developers /opt/ecommerce.git
chmod -R 770 /opt/ecommerce.git
```

### 2. SSH Access

For remote access, ensure SSH is configured:
```bash
# Check SSH service
systemctl status sshd

# Allow SSH through firewall
firewall-cmd --add-service=ssh --permanent
firewall-cmd --reload
```

### 3. Repository Hooks

Set up hooks for security checks:
```bash
# Example: pre-receive hook to validate commits
vi /opt/ecommerce.git/hooks/pre-receive
chmod +x /opt/ecommerce.git/hooks/pre-receive
```

---

## Completion Checklist

- [x] SSH access to Storage Server established
- [x] Root/sudo access obtained
- [x] Git package installed via yum
- [x] Git version verified
- [x] Bare repository created at `/opt/ecommerce.git`
- [x] Repository structure verified
- [x] Configuration confirmed as bare repository
- [x] Repository name exactly matches requirement (ecommerce.git)
- [x] Repository located in correct directory (/opt/)
- [x] Repository accessible and functional

---

## Completion Details

- **Completion Date:** November 26, 2025
- **Day:** 21 / 100
- **Challenge:** KodeKloud 100 Days Cloud DevOps
- **Topic:** Setting Up Bare Git Repository on Storage Server
- **Repository Name:** `/opt/ecommerce.git`
- **Repository Type:** Bare Repository
- **Status:** ✅ Successfully Completed

---

