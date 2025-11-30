# Day 25: Git Branch, Commit, and Merge Workflow
**100 Days Cloud DevOps Challenge – KodeKloud**

---

## Objective

The Nautilus application development team has been working on a project repository and needs the DevOps team to perform a complete Git workflow involving branching, committing, merging, and pushing changes.

**Requirements:**
1. Create a new branch `nautilus` from `master` in `/usr/src/kodekloudrepos/games`
2. Copy `/tmp/index.html` file into the repository
3. Add and commit the file in the new branch
4. Merge the `nautilus` branch back into `master`
5. Push changes to origin for both branches

---

## Infrastructure Overview

### Storage Server:
| Server   | User    | Password | IP            |
|----------|---------|----------|---------------|
| ststor01 | natasha | Bl@kW    | 172.16.238.15 |

### Repository Details:
| Item | Value |
|------|-------|
| Original Repository | `/opt/games.git` |
| Cloned Repository | `/usr/src/kodekloudrepos/games` |
| Source Branch | `master` |
| New Branch | `nautilus` |
| File to Add | `/tmp/index.html` |

---

## Understanding the Git Workflow

This task demonstrates a complete feature development workflow:
```
1. Create Branch (nautilus from master)
2. Make Changes (copy and add index.html)
3. Commit Changes (save to branch)
4. Merge Branch (nautilus → master)
5. Push to Remote (both branches)
```

### Why This Workflow?

- **Isolation:** Changes made in separate branch
- **Safety:** Master remains stable during development
- **Review:** Changes can be reviewed before merging
- **History:** Clear record of feature development
- **Collaboration:** Multiple developers can work simultaneously

---

## Step-by-Step Implementation

### Step 1: SSH into Storage Server
```bash
ssh natasha@ststor01
```

**Enter password:** `Bl@kW`

### Step 2: Navigate to Repository
```bash
cd /usr/src/kodekloudrepos/games
```

### Step 3: Verify Current Status
```bash
# Check current branch
git branch

# Check repository status
git status

# View current files
ls -la
```

**Expected output:**
```
* master

On branch master
Your branch is up to date with 'origin/master'.

nothing to commit, working tree clean
```

### Step 4: Create New Branch
```bash
git checkout -b nautilus
```

**Expected output:**
```
Switched to a new branch 'nautilus'
```

### Step 5: Verify Branch Creation
```bash
git branch
```

**Expected output:**
```
  master
* nautilus
```

### Step 6: Verify Source File Exists
```bash
ls -la /tmp/index.html
```

**Expected:** File should exist
```bash
cat /tmp/index.html
```

**Verify:** Check file contents

### Step 7: Copy File to Repository
```bash
cp /tmp/index.html .
```

Or with full path:
```bash
cp /tmp/index.html /usr/src/kodekloudrepos/games/
```

### Step 8: Verify File Was Copied
```bash
ls -la index.html
```

**Expected output:**
```
-rw-r--r--. 1 natasha natasha [size] [date] index.html
```

### Step 9: Check Git Status
```bash
git status
```

**Expected output:**
```
On branch nautilus
Untracked files:
  (use "git add <file>..." to include in what will be committed)
        index.html

nothing added to commit but untracked files present (use "git add" to track)
```

### Step 10: Add File to Staging Area
```bash
git add index.html
```

Or add all changes:
```bash
git add .
```

### Step 11: Verify File is Staged
```bash
git status
```

**Expected output:**
```
On branch nautilus
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        new file:   index.html
```

### Step 12: Commit the Changes
```bash
git commit -m "Add index.html file"
```

**Expected output:**
```
[nautilus 1a2b3c4] Add index.html file
 1 file changed, X insertions(+)
 create mode 100644 index.html
```

### Step 13: Verify Commit
```bash
git log --oneline -n 3
```

**Expected output:**
```
1a2b3c4 (HEAD -> nautilus) Add index.html file
...previous commits...
```

### Step 14: Switch to Master Branch
```bash
git checkout master
```

**Expected output:**
```
Switched to branch 'master'
Your branch is up to date with 'origin/master'.
```

### Step 15: Verify Master Branch (Before Merge)
```bash
# Check you're on master
git branch

# Verify index.html doesn't exist yet
ls -la index.html
```

**Expected:** File should not exist in master yet

### Step 16: Merge Nautilus Branch into Master
```bash
git merge nautilus
```

**Expected output:**
```
Updating abc1234..def5678
Fast-forward
 index.html | X ++++++++++
 1 file changed, X insertions(+)
 create mode 100644 index.html
```

### Step 17: Verify Merge
```bash
# Check file now exists in master
ls -la index.html

# Check commit history
git log --oneline -n 3

# View merge status
git status
```

**Expected output:**
```
On branch master
Your branch is ahead of 'origin/master' by 1 commit.
  (use "git push" to publish your local commits)

nothing to commit, working tree clean
```

### Step 18: Push Master Branch to Origin
```bash
git push origin master
```

**Expected output:**
```
Counting objects: 3, done.
Delta compression using up to 2 threads.
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), XXX bytes | XXX KiB/s, done.
Total 3 (delta 0), reused 0 (delta 0)
To /opt/games.git
   abc1234..def5678  master -> master
```

### Step 19: Push Nautilus Branch to Origin
```bash
git push origin nautilus
```

**Expected output:**
```
Total 0 (delta 0), reused 0 (delta 0)
To /opt/games.git
 * [new branch]      nautilus -> nautilus
```

### Step 20: Verify All Changes Pushed
```bash
# Check remote branches
git branch -r

# Check status
git status
```

**Expected output:**
```
  origin/HEAD -> origin/master
  origin/master
  origin/nautilus

On branch master
Your branch is up to date with 'origin/master'.

nothing to commit, working tree clean
```

---

## Complete Command Summary

### Full Workflow (Copy-Paste):
```bash
# SSH into storage server
ssh natasha@ststor01

# Navigate to repository
cd /usr/src/kodekloudrepos/games

# Check current status
git status
git branch

# Create new branch
git checkout -b nautilus

# Verify branch
git branch

# Copy file
cp /tmp/index.html .

# Verify file copied
ls -la index.html

# Add file to staging
git add index.html

# Commit changes
git commit -m "Add index.html file"

# Switch to master
git checkout master

# Merge nautilus into master
git merge nautilus

# Push master to origin
git push origin master

# Push nautilus branch to origin
git push origin nautilus

# Verify
git branch -r
git status
```

---

## Understanding Each Git Command

### git checkout -b nautilus

**What it does:**
- Creates new branch named `nautilus`
- Switches to that branch immediately
- Branch is based on current branch (master)

### cp /tmp/index.html .

**What it does:**
- Copies file from `/tmp/` to current directory
- `.` represents current directory
- Creates new file in working directory

### git add index.html

**What it does:**
- Stages the file for commit
- Moves file to staging area
- Prepares file to be committed

### git commit -m "message"

**What it does:**
- Creates a commit with staged changes
- Saves snapshot of repository
- Records changes with message

### git merge nautilus

**What it does:**
- Merges `nautilus` branch into current branch (master)
- Combines commit histories
- Creates merge commit (if needed)

### git push origin master

**What it does:**
- Uploads `master` branch to remote repository
- Syncs local changes with remote
- Updates `origin/master`

### git push origin nautilus

**What it does:**
- Uploads `nautilus` branch to remote
- Creates branch on remote if doesn't exist
- Allows others to access the branch

---

## Git Workflow Visualization

### Before Starting:
```
master:  A---B---C (HEAD)
         
origin/master: A---B---C
```

### After Creating Branch:
```
master:  A---B---C
                  \
nautilus:          C (HEAD)
```

### After Adding & Committing:
```
master:  A---B---C
                  \
nautilus:          C---D (HEAD)
                      (index.html added)
```

### After Merging:
```
master:  A---B---C---D (HEAD)
                  \  /
nautilus:          C---D
```

### After Pushing:
```
Local:
master:  A---B---C---D (HEAD)
nautilus:         C---D

Remote (origin):
master:  A---B---C---D
nautilus:         C---D
```

---

## Verification Steps

### Verify 1: Both Branches Exist Locally
```bash
git branch
```

**Expected output:**
```
* master
  nautilus
```

### Verify 2: File Exists in Master
```bash
git checkout master
ls -la index.html
cat index.html
```

**Expected:** File should exist and have content

### Verify 3: Both Branches on Remote
```bash
git branch -r
```

**Expected output:**
```
  origin/HEAD -> origin/master
  origin/master
  origin/nautilus
```

### Verify 4: Commits Match
```bash
# Check master commits
git log --oneline master -n 3

# Check nautilus commits
git log --oneline nautilus -n 3
```

Both should show the commit with "Add index.html file"

### Verify 5: Remote is Updated
```bash
git fetch origin
git status
```

**Expected output:**
```
On branch master
Your branch is up to date with 'origin/master'.

nothing to commit, working tree clean
```

---

## Troubleshooting

### Issue 1: File Not Found

**Problem:**
```
cp: cannot stat '/tmp/index.html': No such file or directory
```

**Solution:**
```bash
# Check if file exists
ls -la /tmp/index.html

# Check file location
find /tmp -name "index.html"

# If in different location, use correct path
cp /path/to/index.html .
```

### Issue 2: Branch Already Exists

**Problem:**
```
fatal: A branch named 'nautilus' already exists.
```

**Solution:**
```bash
# Delete existing branch
git branch -D nautilus

# Recreate
git checkout -b nautilus
```

### Issue 3: Merge Conflict

**Problem:**
```
CONFLICT (content): Merge conflict in index.html
Automatic merge failed; fix conflicts and then commit the result.
```

**Solution:**
```bash
# View conflicted files
git status

# Edit files to resolve conflicts
vi index.html

# After resolving
git add index.html
git commit -m "Resolve merge conflict"
```

### Issue 4: Push Rejected

**Problem:**
```
! [rejected]        master -> master (fetch first)
error: failed to push some refs to '/opt/games.git'
```

**Solution:**
```bash
# Fetch latest changes
git fetch origin

# Merge or rebase
git merge origin/master

# Push again
git push origin master
```

### Issue 5: Nothing to Commit

**Problem:**
```
nothing to commit, working tree clean
```

**Cause:** File not added

**Solution:**
```bash
# Verify file exists
ls -la index.html

# If missing, copy again
cp /tmp/index.html .

# Add file
git add index.html

# Commit
git commit -m "Add index.html file"
```

### Issue 6: Permission Denied on Push

**Problem:**
```
error: insufficient permission for adding an object to repository database .git/objects
```

**Solution:**
```bash
# Check repository ownership
ls -la /usr/src/kodekloudrepos/games/.git

# Fix permissions
sudo chown -R natasha:natasha /usr/src/kodekloudrepos/games
```

---

## Understanding Git Merge

### Types of Merges:

#### 1. Fast-Forward Merge (Most Common)
```bash
# When target branch has no new commits
master:  A---B---C
                  \
nautilus:          C---D

# After merge
master:  A---B---C---D (fast-forward)
```

**Command:**
```bash
git merge nautilus
```

**Result:** Simply moves master pointer forward

#### 2. Three-Way Merge
```bash
# When both branches have new commits
master:  A---B---C---E
                  \
nautilus:          C---D

# After merge
master:  A---B---C---E---M (merge commit)
                  \     /
nautilus:          C---D
```

**Command:**
```bash
git merge nautilus
```

**Result:** Creates new merge commit

#### 3. Squash Merge
```bash
# Combines all commits into one
git merge --squash nautilus
git commit -m "Add feature"
```

**Result:** Single commit instead of merge commit

---

## Git Add vs Commit vs Push

### git add (Staging):
```bash
Working Directory → Staging Area
```

- Prepares changes for commit
- Can add specific files or all changes
- Reversible with `git restore --staged`

### git commit (Local Repository):
```bash
Staging Area → Local Repository
```

- Saves snapshot of staged changes
- Creates commit with unique hash
- Changes only local repository

### git push (Remote Repository):
```bash
Local Repository → Remote Repository
```

- Uploads commits to remote server
- Makes changes available to team
- Updates remote branch

### Complete Flow:
```
Working Directory
     |
     | git add
     ↓
Staging Area
     |
     | git commit
     ↓
Local Repository
     |
     | git push
     ↓
Remote Repository
```

---

## Best Practices Applied

### 1. Descriptive Commit Messages
```bash
# Good
git commit -m "Add index.html file"
git commit -m "Fix navigation bug"
git commit -m "Update documentation"

# Avoid
git commit -m "update"
git commit -m "changes"
git commit -m "fix"
```

### 2. Verify Before Committing
```bash
# Check what's being committed
git status
git diff

# Then commit
git add .
git commit -m "Descriptive message"
```

### 3. Test Before Merging
```bash
# On feature branch
# ... test changes ...

# Then merge
git checkout master
git merge feature-branch
```

### 4. Push Both Branches
```bash
# Push main branch
git push origin master

# Push feature branch (for backup/collaboration)
git push origin nautilus
```

### 5. Clean Workflow
```bash
# Create → Develop → Commit → Merge → Push
# Each step clearly defined
# No skipping steps
```

---

## Common Git Workflow Patterns

### Pattern 1: Feature Development (This Task)
```bash
git checkout -b feature
# ... make changes ...
git add .
git commit -m "Add feature"
git checkout master
git merge feature
git push origin master
git push origin feature
```

### Pattern 2: Hotfix
```bash
git checkout master
git checkout -b hotfix
# ... fix bug ...
git commit -am "Fix critical bug"
git checkout master
git merge hotfix
git push origin master
git branch -d hotfix
```

### Pattern 3: Multiple Features
```bash
# Feature 1
git checkout -b feature-1
# ... work ...
git commit -am "Feature 1"

# Feature 2
git checkout master
git checkout -b feature-2
# ... work ...
git commit -am "Feature 2"

# Merge both
git checkout master
git merge feature-1
git merge feature-2
git push origin master
```

---

## Key Git Commands Reference

### Branch Operations:

| Command | Description |
|---------|-------------|
| `git branch` | List branches |
| `git branch -r` | List remote branches |
| `git branch -a` | List all branches |
| `git checkout -b <name>` | Create and switch to branch |
| `git checkout <name>` | Switch to branch |
| `git branch -d <name>` | Delete branch |
| `git merge <branch>` | Merge branch into current |

### File Operations:

| Command | Description |
|---------|-------------|
| `git status` | Show working tree status |
| `git add <file>` | Stage file |
| `git add .` | Stage all changes |
| `git commit -m "<msg>"` | Commit with message |
| `git commit -am "<msg>"` | Add and commit (tracked files) |
| `git log` | Show commit history |

### Remote Operations:

| Command | Description |
|---------|-------------|
| `git push origin <branch>` | Push branch to remote |
| `git push origin --all` | Push all branches |
| `git fetch origin` | Fetch from remote |
| `git pull origin <branch>` | Fetch and merge |
| `git remote -v` | Show remote URLs |

---

## Understanding Git States

### Working Directory:

- Untracked files
- Modified tracked files
- Not yet staged

### Staging Area (Index):

- Files marked with `git add`
- Ready to be committed
- Preview of next commit

### Local Repository (.git):

- Committed changes
- Full project history
- Local branches

### Remote Repository (origin):

- Shared with team
- Backup of local work
- Source of truth

---

## Key Takeaways

- **Branch workflow** isolates feature development
- **Staging area** allows selective commits
- **Commits** save snapshots of changes
- **Merging** integrates features into main branch
- **Pushing** shares changes with team
- **Both branches pushed** for backup and collaboration
- **Clean history** with descriptive messages
- **Verify at each step** prevents errors

---

## Real-World Application

This workflow is used for:

### 1. Feature Development
```
Create branch → Develop → Test → Merge → Deploy
```

### 2. Bug Fixes
```
Create branch → Fix → Verify → Merge → Release
```

### 3. Code Review
```
Create branch → Develop → Push → Review → Merge
```

### 4. Experimentation
```
Create branch → Experiment → Evaluate → Merge or Discard
```

### 5. Collaboration
```
Create branch → Develop → Share → Collaborate → Merge
```

---

## Completion Checklist

- [x] SSH into Storage Server (ststor01)
- [x] Navigated to repository `/usr/src/kodekloudrepos/games`
- [x] Created new branch `nautilus` from `master`
- [x] Verified branch creation
- [x] Copied `/tmp/index.html` to repository
- [x] Verified file copied successfully
- [x] Staged file with `git add`
- [x] Committed file with descriptive message
- [x] Switched back to `master` branch
- [x] Merged `nautilus` branch into `master`
- [x] Verified merge completed
- [x] Pushed `master` branch to origin
- [x] Pushed `nautilus` branch to origin
- [x] Verified both branches on remote
- [x] Confirmed working tree clean

---

## Completion Details

- **Completion Date:** November 30, 2025
- **Day:** 25 / 100
- **Challenge:** KodeKloud 100 Days Cloud DevOps
- **Topic:** Complete Git Branch, Commit, and Merge Workflow
- **Server:** Storage Server (ststor01)
- **Repository:** `/usr/src/kodekloudrepos/games`
- **Original Repo:** `/opt/games.git`
- **New Branch:** nautilus
- **File Added:** index.html
- **Branches Pushed:** master, nautilus
- **Key Skills:** Branching, staging, committing, merging, pushing
- **Status:** ✅ Successfully Completed


---

## Summary

This task demonstrated a **complete Git workflow** from start to finish:

✅ **Branch:** Create isolated development environment
✅ **Modify:** Add new files to repository
✅ **Stage:** Prepare changes for commit
✅ **Commit:** Save changes with message
✅ **Merge:** Integrate changes into main branch
✅ **Push:** Share changes with remote repository

This is the **fundamental workflow** used in professional software development every day. Understanding this process is essential for:
- Team collaboration
- Code review workflows
- Feature development
- Bug fixing
- Release management

**Remember:** Branch → Add → Commit → Merge → Push = Complete Feature Delivery! 🚀