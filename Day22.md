# Day 22: Cloning Git Repository on Storage Server
**100 Days Cloud DevOps Challenge – KodeKloud**

---

## Objective

The DevOps team established a new Git repository last week, which remains unused at present. However, the Nautilus application development team now requires a copy of this repository on the Storage Server in Stratos DC.

**Requirements:**
- Clone the Git repository located at `/opt/apps.git`
- Clone to `/usr/src/kodekloudrepos` directory
- Perform the task as `natasha` user
- Do not modify the repository or existing directories
- Do not change permissions or make unauthorized alterations

---

## Infrastructure Overview

### Storage Server:
| Server   | User    | Password | IP            |
|----------|---------|----------|---------------|
| ststor01 | natasha | Bl@kW    | 172.16.238.15 |

### Repository Details:
- **Source Repository:** `/opt/apps.git` (bare repository)
- **Destination:** `/usr/src/kodekloudrepos`
- **User:** natasha

---

## Understanding the Requirement

**Key Point:** The task requires cloning the repository so that the repository contents are in `/usr/src/kodekloudrepos/`, NOT in a subdirectory.

**Wrong Structure:** ❌
```
/usr/src/kodekloudrepos/apps/.git
```

**Correct Structure:** ✅
```
/usr/src/kodekloudrepos/.git
```

---

## Step-by-Step Implementation

### Step 1: SSH into Storage Server
```bash
ssh natasha@ststor01
```

**Enter password:** `Bl@kW`

### Step 2: Verify Source Repository Exists
```bash
ls -la /opt/apps.git
```

**Expected output:**
```
total 40
drwxr-xr-x.  7 root root  119 Nov 27 10:00 .
drwxr-xr-x.  4 root root   30 Nov 27 10:00 ..
drwxr-xr-x.  2 root root    6 Nov 27 10:00 branches
-rw-r--r--.  1 root root   66 Nov 27 10:00 config
-rw-r--r--.  1 root root   73 Nov 27 10:00 description
-rw-r--r--.  1 root root   23 Nov 27 10:00 HEAD
drwxr-xr-x.  2 root root 4096 Nov 27 10:00 hooks
drwxr-xr-x.  2 root root   21 Nov 27 10:00 info
drwxr-xr-x.  4 root root   30 Nov 27 10:00 objects
drwxr-xr-x.  4 root root   31 Nov 27 10:00 refs
```

### Step 3: Create Destination Directory
```bash
sudo mkdir -p /usr/src/kodekloudrepos
```

### Step 4: Navigate to Destination Directory
```bash
cd /usr/src/kodekloudrepos
```

### Step 5: Clone Repository into Current Directory

**CRITICAL:** Use the dot (`.`) at the end to clone into the current directory:
```bash
sudo git clone /opt/apps.git .
```

**Note the dot (`.`)** - this is the most important part!

**Expected output:**
```
Cloning into '.'...
done.
```

Or if the repository is empty:
```
Cloning into '.'...
warning: You appear to have cloned an empty repository.
done.
```

### Step 6: Verify the Clone
```bash
ls -la /usr/src/kodekloudrepos/
```

**Expected output:**
```
total 12
drwxr-xr-x. 3 root    root      17 Nov 27 10:05 .
drwxr-xr-x. 4 root    root      34 Nov 27 10:05 ..
drwxr-xr-x. 8 natasha natasha 4096 Nov 27 10:05 .git
```

### Step 7: Verify Git Configuration
```bash
pwd
```

**Should show:** `/usr/src/kodekloudrepos`
```bash
git remote -v
```

**Expected output:**
```
origin  /opt/apps.git (fetch)
origin  /opt/apps.git (push)
```

### Step 8: Check Repository Status
```bash
git status
```

**Expected output:**
```
On branch master
Your branch is up to date with 'origin/master'.

nothing to commit, working tree clean
```

Or for newer Git versions:
```
On branch main
Your branch is up to date with 'origin/main'.

nothing to commit, working tree clean
```

---

## Complete Command Summary

### Quick Copy-Paste Solution:
```bash
# SSH into storage server
ssh natasha@ststor01

# Verify source repository
ls -la /opt/apps.git

# Create destination directory
sudo mkdir -p /usr/src/kodekloudrepos

# Navigate to destination
cd /usr/src/kodekloudrepos

# Clone repository (note the dot!)
sudo git clone /opt/apps.git .

# Verify
ls -la
pwd
git remote -v
git status
```

---

## Why the Dot (`.`) is Essential

### Without Dot:
```bash
git clone /opt/apps.git /usr/src/kodekloudrepos
```
**Result:** Creates `/usr/src/kodekloudrepos/apps/` ❌

### With Dot:
```bash
cd /usr/src/kodekloudrepos
git clone /opt/apps.git .
```
**Result:** Creates `/usr/src/kodekloudrepos/.git` ✅

**The dot (`.`) tells Git:** "Clone the repository contents into the **current directory**" instead of creating a new subdirectory named after the repository.

---

## Verification Steps

### Verify 1: Check .git Location
```bash
ls -la /usr/src/kodekloudrepos/.git
```

**Should exist and show:**
```
drwxr-xr-x. 8 natasha natasha 4096 Nov 27 10:05 .git
```

### Verify 2: Confirm Git Repository
```bash
cd /usr/src/kodekloudrepos
git rev-parse --is-inside-work-tree
```

**Expected output:**
```
true
```

### Verify 3: Check Remote URL
```bash
git config --get remote.origin.url
```

**Expected output:**
```
/opt/apps.git
```

### Verify 4: No Subdirectories Created
```bash
ls /usr/src/kodekloudrepos/
```

**Should show:** Only `.git` directory (and any repository files if present), NO `apps` subdirectory

### Verify 5: Test Git Commands
```bash
cd /usr/src/kodekloudrepos
git log --oneline
git branch
```

Both commands should work without errors.

---

## Troubleshooting

### Issue 1: Directory Not Empty Error

**Problem:**
```
fatal: destination path '.' already exists and is not an empty directory.
```

**Solution:**
```bash
# Option 1: Remove existing content
cd /usr/src/kodekloudrepos
sudo rm -rf * .[^.]*

# Then clone again
sudo git clone /opt/apps.git .
```

**Alternative:**
```bash
# Option 2: Remove and recreate directory
sudo rm -rf /usr/src/kodekloudrepos
sudo mkdir -p /usr/src/kodekloudrepos
cd /usr/src/kodekloudrepos
sudo git clone /opt/apps.git .
```

### Issue 2: Permission Denied

**Problem:**
```
fatal: could not create work tree dir: Permission denied
```

**Solution:**
```bash
# Use sudo for the clone operation
sudo git clone /opt/apps.git .
```

### Issue 3: Repository Not Found

**Problem:**
```
fatal: repository '/opt/apps.git' does not exist
```

**Solution:**
```bash
# Verify the repository path
ls -la /opt/ | grep apps
ls -la /opt/*.git

# Use the correct path found
sudo git clone /opt/apps.git .
```

### Issue 4: Wrong Directory Structure

**Problem:** Files are in `/usr/src/kodekloudrepos/apps/` instead of `/usr/src/kodekloudrepos/`

**Solution:**
```bash
# You forgot the dot! Remove and redo:
sudo rm -rf /usr/src/kodekloudrepos/apps
cd /usr/src/kodekloudrepos
sudo git clone /opt/apps.git .
# Don't forget the dot!
```

### Issue 5: Git Not Installed

**Problem:**
```
-bash: git: command not found
```

**Solution:**
```bash
sudo yum install -y git
git --version
```

---

## Final Verification Checklist

Before submitting, verify all these:
```bash
# 1. Check current directory
pwd
# Output should be: /usr/src/kodekloudrepos

# 2. Verify .git exists
ls -la | grep .git
# Should show: drwxr-xr-x ... .git

# 3. Test git status
git status
# Should work without errors

# 4. Check remote
git remote -v
# Should show: origin /opt/apps.git

# 5. Verify no apps subdirectory
ls -la /usr/src/kodekloudrepos/apps 2>/dev/null
# Should return: No such file or directory

# 6. Confirm you can run git commands
git branch
git log
# Both should work
```

---

## Understanding Git Clone Behavior

### Standard Clone (Creates Subdirectory):
```bash
git clone /opt/apps.git /destination/path
```
**Result:** `/destination/path/apps/` (named after repository)

### Clone into Directory (No Subdirectory):
```bash
cd /destination/path
git clone /opt/apps.git .
```
**Result:** `/destination/path/.git` (directly in specified location)

### Alternative Syntax:
```bash
git clone /opt/apps.git /usr/src/kodekloudrepos/apps
```
**Result:** `/usr/src/kodekloudrepos/apps/` (explicitly named subdirectory)

---

## Key Git Commands Used

| Command | Purpose |
|---------|---------|
| `git clone <source> .` | Clone into current directory |
| `git remote -v` | Show remote repository URLs |
| `git status` | Check repository status |
| `git config --get remote.origin.url` | Get remote URL |
| `git rev-parse --is-inside-work-tree` | Verify if in Git repository |
| `git branch` | Show current branch |
| `git log` | Show commit history |

---

## What KodeKloud Validates

The validation script likely checks:

1. ✅ `/usr/src/kodekloudrepos/.git` directory exists
2. ✅ Git commands work in `/usr/src/kodekloudrepos/`
3. ✅ Remote origin points to `/opt/apps.git`
4. ✅ No `/usr/src/kodekloudrepos/apps/` subdirectory exists
5. ✅ Repository is functional (git status returns successfully)
6. ✅ Performed as natasha user

---

## Common Mistakes to Avoid

### ❌ Mistake 1: Forgetting the Dot
```bash
cd /usr/src/kodekloudrepos
git clone /opt/apps.git
# Creates subdirectory: /usr/src/kodekloudrepos/apps/
```

### ✅ Correct:
```bash
cd /usr/src/kodekloudrepos
git clone /opt/apps.git .
# Creates: /usr/src/kodekloudrepos/.git
```

### ❌ Mistake 2: Wrong Repository Path
```bash
git clone /opt/games.git .  # Wrong repo name!
```

### ✅ Correct:
```bash
git clone /opt/apps.git .  # Correct repo name
```

### ❌ Mistake 3: Not Using sudo
```bash
git clone /opt/apps.git .
# Permission denied error
```

### ✅ Correct:
```bash
sudo git clone /opt/apps.git .
# Works correctly
```

### ❌ Mistake 4: Wrong Destination
```bash
cd /usr/src
git clone /opt/apps.git kodekloudrepos
# Creates: /usr/src/kodekloudrepos/apps/
```

### ✅ Correct:
```bash
cd /usr/src/kodekloudrepos
git clone /opt/apps.git .
# Creates: /usr/src/kodekloudrepos/.git
```

---

## Best Practices

### 1. Always Verify Source First
```bash
ls -la /opt/apps.git
```

### 2. Create Destination Before Cloning
```bash
sudo mkdir -p /usr/src/kodekloudrepos
```

### 3. Navigate Then Clone
```bash
cd /usr/src/kodekloudrepos
sudo git clone /opt/apps.git .
```

### 4. Verify After Cloning
```bash
ls -la
git status
git remote -v
```

### 5. Use Absolute Paths
```bash
# Good
git clone /opt/apps.git .

# Avoid
git clone ../apps.git .
```

---

## Repository Structure Comparison

### Bare Repository (Source):
```
/opt/apps.git/
├── branches/
├── config
├── description
├── HEAD
├── hooks/
├── info/
├── objects/
└── refs/
```

### Cloned Repository (Destination):
```
/usr/src/kodekloudrepos/
├── .git/              # Hidden directory
│   ├── config
│   ├── HEAD
│   ├── hooks/
│   ├── objects/
│   └── refs/
└── [project files]    # Working files (if any)
```

---

## Key Takeaways

- **The dot (`.`) is critical** for cloning into current directory
- **Navigate first, clone second** ensures correct location
- **Use sudo** if you encounter permission issues
- **Verify the structure** after cloning
- **No subdirectories** should be created
- **Remote URL** should point to `/opt/apps.git`
- **Git commands** should work in `/usr/src/kodekloudrepos/`

---

## Quick Reference

### The One Command That Matters:
```bash
cd /usr/src/kodekloudrepos && sudo git clone /opt/apps.git .
```

**Remember:** The dot (`.`) is what makes it work! 🎯

---

## Completion Checklist

- [x] SSH into ststor01 as natasha
- [x] Verified source repository at `/opt/apps.git`
- [x] Created `/usr/src/kodekloudrepos` directory
- [x] Navigated to destination directory
- [x] Executed: `sudo git clone /opt/apps.git .`
- [x] Verified `.git` exists in `/usr/src/kodekloudrepos/`
- [x] Confirmed no `apps` subdirectory created
- [x] Tested `git status` successfully
- [x] Verified remote URL is correct
- [x] No unauthorized modifications made

---

## Completion Details

- **Completion Date:** November 27, 2025
- **Day:** 22 / 100
- **Challenge:** KodeKloud 100 Days Cloud DevOps
- **Topic:** Cloning Git Repository into Specific Directory Structure
- **Source Repository:** `/opt/apps.git`
- **Destination:** `/usr/src/kodekloudrepos` (direct, no subdirectory)
- **Key Command:** `git clone /opt/apps.git .`
- **Critical Learning:** The dot (`.`) prevents subdirectory creation
- **Status:** ✅ Successfully Completed

---

