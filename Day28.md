# Day 28: Git Cherry-Pick - Selective Commit Merging
**100 Days Cloud DevOps Challenge – KodeKloud**

---

## Objective

The Nautilus application development team is working on a project repository with multiple branches. A developer needs to merge a specific commit from the feature branch into master while their work is still in progress.

**Requirements:**
1. Work with repository `/opt/news.git` cloned at `/usr/src/kodekloudrepos` on storage server
2. Two branches exist: `master` and `feature`
3. Cherry-pick the commit with message "Update info.txt" from `feature` branch
4. Merge this specific commit into `master` branch
5. Push changes to remote repository

---

## Understanding Git Cherry-Pick

**Git cherry-pick** allows you to select specific commits from one branch and apply them to another branch. Unlike merge or rebase which apply multiple commits, cherry-pick targets individual commits.

### Cherry-Pick vs Merge vs Rebase

| Command | What It Does | Use Case | History Impact |
|---------|-------------|----------|----------------|
| `git cherry-pick` | Copy single commit | Selective changes | Creates new commit |
| `git merge` | Combine branches | Full branch integration | Merge commit |
| `git rebase` | Rewrite commit history | Clean linear history | Rewrites commits |

### Why Use Cherry-Pick?

- **Selective Integration:** Pick specific features without merging everything
- **Bug Fixes:** Apply critical fixes from one branch to another
- **Work in Progress:** Take completed work while rest is incomplete
- **Hotfixes:** Apply production fixes to multiple branches
- **Flexibility:** Control what gets merged and when

---

## Infrastructure Overview

### Storage Server:
| Server   | User    | Password | IP            |
|----------|---------|----------|---------------|
| ststor01 | natasha | Bl@kW    | 172.16.238.15 |

### Repository Details:
| Item | Value |
|------|-------|
| Repository | `/opt/news.git` |
| Clone Location | `/usr/src/kodekloudrepos` |
| Branches | `master` and `feature` |
| Target Commit Message | "Update info.txt" |
| Action | Cherry-pick from `feature` to `master` |
| Final Step | Push changes |

---

## Understanding the Scenario

### Current State:
```
master:  A --- B --- C
                      ↑
                    master

feature: A --- B --- D --- E --- F
                          ↑
                   "Update info.txt"
```

### After Cherry-Pick:
```
master:  A --- B --- C --- E'
                           ↑
                        master

feature: A --- B --- D --- E --- F
                          ↑
                   "Update info.txt"
                   (original still exists)
```

**Note:** E' is a new commit with same changes as E but different hash

---

## Step-by-Step Implementation

### Step 1: SSH into Storage Server
```bash
ssh natasha@ststor01
```

**Enter password:** `Bl@kW`

### Step 2: Navigate to Repository
```bash
cd /usr/src/kodekloudrepos
```

### Step 3: Check Repository Name
```bash
ls -la
```

**Look for the news repository directory**

```bash
cd news
```

Or if it's named differently:
```bash
cd <repository-name>
```

### Step 4: Verify Repository Status
```bash
git status
```

**Expected output:**
```
On branch master
Your branch is up to date with 'origin/master'.

nothing to commit, working tree clean
```

### Step 5: List All Branches
```bash
git branch -a
```

**Expected output:**
```
* master
  feature
  remotes/origin/HEAD -> origin/master
  remotes/origin/master
  remotes/origin/feature
```

### Step 6: Ensure You're on Master Branch
```bash
git checkout master
```

**Expected output:**
```
Already on 'master'
```

Or:
```
Switched to branch 'master'
Your branch is up to date with 'origin/master'.
```

### Step 7: Update Master Branch
```bash
git pull origin master
```

**Expected output:**
```
Already up to date.
```

### Step 8: View Feature Branch Commits
```bash
git log feature --oneline
```

**Expected output (example):**
```
abc1234 Some other work
def5678 Update info.txt
789abcd Initial feature work
```

### Step 9: Find the Specific Commit
```bash
git log feature --oneline --grep="Update info.txt"
```

**Expected output:**
```
def5678 Update info.txt
```

**Alternative - View detailed log:**
```bash
git log feature --all --grep="Update info.txt"
```

### Step 10: Note the Commit Hash
From the output above, identify the commit hash (e.g., `def5678`)

### Step 11: View Commit Details (Optional)
```bash
git show def5678
```

This shows what changes the commit contains.

### Step 12: Cherry-Pick the Commit
```bash
git cherry-pick def5678
```

**Replace `def5678` with the actual commit hash you found**

**Expected output (success):**
```
[master 1a2b3c4] Update info.txt
 Date: Mon Dec 2 10:00:00 2025
 1 file changed, 5 insertions(+), 2 deletions(-)
```

**If there are conflicts:**
```
error: could not apply def5678... Update info.txt
hint: after resolving the conflicts, mark the corrected paths
hint: with 'git add <paths>' or 'git rm <paths>'
hint: and commit the result with 'git cherry-pick --continue'
```

### Step 13: Handle Conflicts (If Any)
```bash
# View conflicted files
git status

# Edit files to resolve conflicts
vi info.txt

# After resolving, add the file
git add info.txt

# Continue cherry-pick
git cherry-pick --continue
```

### Step 14: Verify Cherry-Pick
```bash
git log --oneline -n 3
```

**Expected output:**
```
1a2b3c4 (HEAD -> master) Update info.txt
xyz9876 Previous commit on master
...
```

### Step 15: Check Changes
```bash
git show HEAD
```

This displays the cherry-picked commit details.

### Step 16: Verify Working Directory
```bash
git status
```

**Expected output:**
```
On branch master
Your branch is ahead of 'origin/master' by 1 commit.
  (use "git push" to publish your local commits)

nothing to commit, working tree clean
```

### Step 17: Push Changes to Remote
```bash
git push origin master
```

**Expected output:**
```
Counting objects: 5, done.
Delta compression using up to 2 threads.
Compressing objects: 100% (3/3), done.
Writing objects: 100% (5/5), 456 bytes | 456.00 KiB/s, done.
Total 5 (delta 2), reused 0 (delta 0)
To /opt/news.git
   xyz9876..1a2b3c4  master -> master
```

### Step 18: Verify Push
```bash
git log origin/master --oneline -n 3
```

**Expected output:**
```
1a2b3c4 (HEAD -> master, origin/master) Update info.txt
xyz9876 Previous commit
...
```

---

## Complete Command Summary

### Quick Method:
```bash
# SSH into storage server
ssh natasha@ststor01

# Navigate to repository
cd /usr/src/kodekloudrepos/news

# Check current branch
git status

# Switch to master (if not already)
git checkout master

# Update master
git pull origin master

# Find commit hash
git log feature --oneline --grep="Update info.txt"

# Cherry-pick the commit (replace HASH with actual hash)
git cherry-pick HASH

# Push changes
git push origin master

# Verify
git log --oneline -n 3
```

### Detailed Method with Verification:
```bash
# SSH and navigate
ssh natasha@ststor01
cd /usr/src/kodekloudrepos/news

# Verify branches exist
git branch -a

# Ensure on master
git checkout master
git pull origin master

# View feature branch commits
git log feature --oneline

# Find specific commit
git log feature --grep="Update info.txt"

# View commit details
git show <commit-hash>

# Cherry-pick
git cherry-pick <commit-hash>

# Verify cherry-pick
git log --oneline -n 3
git show HEAD

# Check status
git status

# Push
git push origin master

# Verify push
git log origin/master --oneline -n 3
```

---

## Understanding Git Cherry-Pick

### How Cherry-Pick Works:

1. **Identify target commit** on source branch
2. **Calculate changes** (diff) in that commit
3. **Apply same changes** to current branch
4. **Create new commit** with same message and changes
5. **New commit hash** (different from original)

### Cherry-Pick Syntax:
```bash
# Basic cherry-pick
git cherry-pick <commit-hash>

# Cherry-pick multiple commits
git cherry-pick <hash1> <hash2> <hash3>

# Cherry-pick range (exclusive start)
git cherry-pick <start-hash>..<end-hash>

# Cherry-pick range (inclusive start)
git cherry-pick <start-hash>^..<end-hash>

# Cherry-pick without committing
git cherry-pick <hash> --no-commit

# Cherry-pick with custom message
git cherry-pick <hash> -e
```

---

## Finding the Right Commit

### Method 1: Search by Message
```bash
git log feature --grep="Update info.txt"
```

### Method 2: View All Feature Commits
```bash
git log feature --oneline
```

### Method 3: View Commits Not in Master
```bash
git log master..feature --oneline
```

### Method 4: Search with Author
```bash
git log feature --author="developer" --oneline
```

### Method 5: Search by Date
```bash
git log feature --since="2025-12-01" --oneline
```

### Method 6: Interactive Search
```bash
git log feature --oneline --all
```

---

## Verification Steps

### Verify 1: Commit Exists on Master
```bash
git log master --grep="Update info.txt"
```

**Expected:** Should show the cherry-picked commit

### Verify 2: File Changes Applied
```bash
# Check if info.txt has expected changes
cat info.txt

# Or view in log
git show HEAD:info.txt
```

### Verify 3: Commit on Both Branches
```bash
# View master
git log master --oneline -n 5

# View feature
git log feature --oneline -n 5
```

**Note:** Different commit hashes but same message

### Verify 4: Remote Updated
```bash
git fetch origin
git log origin/master --oneline -n 3
```

### Verify 5: Compare Branches
```bash
git log master..feature --oneline
```

Shows commits on feature not on master (should not include cherry-picked commit)

---

## Troubleshooting

### Issue 1: Merge Conflicts During Cherry-Pick

**Problem:**
```
error: could not apply abc1234... Update info.txt
hint: after resolving the conflicts, mark the corrected paths
```

**Solution:**
```bash
# View conflicted files
git status

# Files with conflicts shown as "both modified"
# Edit conflicted file
vi info.txt

# Look for conflict markers:
# <<<<<<< HEAD
# ... current branch content ...
# =======
# ... cherry-picked content ...
# >>>>>>> abc1234 (Update info.txt)

# Resolve conflicts, then:
git add info.txt
git cherry-pick --continue
```

### Issue 2: Wrong Commit Hash

**Problem:** Can't find commit or cherry-pick fails

**Solution:**
```bash
# Verify commit exists
git log feature --oneline

# Search for it
git log --all --grep="Update info.txt"

# Use correct hash
git cherry-pick <correct-hash>
```

### Issue 3: Already Cherry-Picked

**Problem:**
```
The previous cherry-pick is now empty, possibly due to conflict resolution.
```

**Solution:**
```bash
# Skip empty commit
git cherry-pick --skip

# Or abort and retry
git cherry-pick --abort
```

### Issue 4: Not on Master Branch

**Problem:** Cherry-picking to wrong branch

**Solution:**
```bash
# Check current branch
git branch

# Abort cherry-pick
git cherry-pick --abort

# Switch to master
git checkout master

# Retry
git cherry-pick <hash>
```

### Issue 5: Push Rejected

**Problem:**
```
! [rejected]        master -> master (fetch first)
```

**Solution:**
```bash
# Pull first
git pull origin master

# Then push
git push origin master
```

### Issue 6: Can't Find Repository

**Problem:** Directory structure different than expected

**Solution:**
```bash
# List repositories
ls /usr/src/kodekloudrepos/

# Check for news.git or news
cd /usr/src/kodekloudrepos/
find . -name "*news*" -type d

# Navigate to correct directory
cd <found-directory>
```

---

## Cherry-Pick Advanced Options

### Cherry-Pick Multiple Commits:
```bash
# Pick commits one by one
git cherry-pick abc1234 def5678 ghi9012

# Pick range (exclusive start)
git cherry-pick abc1234..def5678

# Pick range (inclusive start)
git cherry-pick abc1234^..def5678
```

### Cherry-Pick Without Committing:
```bash
# Stage changes but don't commit
git cherry-pick abc1234 --no-commit

# Review changes
git diff --cached

# Commit when ready
git commit -m "Custom message"
```

### Cherry-Pick with Edit:
```bash
# Edit commit message
git cherry-pick abc1234 -e
```

### Cherry-Pick and Sign:
```bash
# Sign the cherry-picked commit
git cherry-pick abc1234 -s
```

### Cherry-Pick Specific Files Only:
```bash
# Stage changes without committing
git cherry-pick abc1234 --no-commit

# Unstage unwanted files
git reset HEAD unwanted-file.txt

# Commit selected changes
git commit -m "Partial cherry-pick"
```

---

## Common Cherry-Pick Scenarios

### Scenario 1: Single Commit (This Task)
```bash
# Find commit
git log feature --grep="Update info.txt"

# Cherry-pick it
git cherry-pick <hash>

# Push
git push origin master
```

### Scenario 2: Bug Fix to Multiple Branches
```bash
# Fix committed to develop
git checkout develop
git commit -m "Fix critical bug"

# Apply to master
git checkout master
git cherry-pick <fix-hash>

# Apply to release branch
git checkout release-v1.0
git cherry-pick <fix-hash>
```

### Scenario 3: Feature Preview
```bash
# Developer working on feature branch
# Manager wants to see one feature

# Cherry-pick to demo branch
git checkout demo
git cherry-pick <feature-commit>
```

### Scenario 4: Hotfix Distribution
```bash
# Hotfix on master
git checkout master
git commit -m "Security fix"

# Apply to all release branches
git checkout release-v2.0
git cherry-pick <hotfix-hash>

git checkout release-v1.9
git cherry-pick <hotfix-hash>
```

### Scenario 5: Selective Migration
```bash
# Migrating from old to new architecture
# Only want certain commits

git checkout new-arch
git cherry-pick <commit1>
git cherry-pick <commit2>
git cherry-pick <commit3>
```

---

## Cherry-Pick vs Merge

### When to Use Cherry-Pick:

✅ **Selective changes** - Only want specific commits
✅ **Work in progress** - Feature branch not ready for full merge
✅ **Bug fixes** - Apply fix without merging all changes
✅ **Cross-branch** - Different development lines
✅ **Hotfixes** - Critical fixes needed on multiple branches

### When to Use Merge:

✅ **Complete feature** - All work ready for integration
✅ **Full branch** - Want entire branch history
✅ **Team collaboration** - Preserve all contributions
✅ **Release integration** - Bring everything together
✅ **Standard workflow** - Normal development flow

---

## Best Practices

### 1. Always Verify Before Cherry-Picking
```bash
# View the commit
git show <commit-hash>

# Check what will be applied
git diff <commit-hash>~1 <commit-hash>
```

### 2. Keep Track of Cherry-Picked Commits
```bash
# Use -x flag to record original commit
git cherry-pick -x <hash>

# Adds: (cherry picked from commit <hash>)
```

### 3. Test After Cherry-Picking
```bash
# After cherry-pick
# Run tests
# Verify functionality
# Then push
```

### 4. Document Cherry-Picks
```bash
# Use descriptive messages
git cherry-pick <hash> -e

# Add: "Cherry-picked from feature branch for..."
```

### 5. Avoid Cherry-Picking Too Much
- If cherry-picking many commits, consider merge instead
- Too many cherry-picks make history confusing
- Indicates branches might need restructuring

---

## Understanding Commit Hashes

### Full Hash vs Short Hash:
```bash
# Full hash (40 characters)
abc123def456789012345678901234567890abcd

# Short hash (7+ characters)
abc123d
```

**Either works for cherry-pick, Git will figure it out**

### Finding Hashes:
```bash
# Short format
git log --oneline

# Full format
git log

# With branches
git log --all --oneline --graph
```

---

## Key Git Commands Reference

| Command | Description |
|---------|-------------|
| `git cherry-pick <hash>` | Apply specific commit to current branch |
| `git cherry-pick <hash1> <hash2>` | Apply multiple commits |
| `git cherry-pick --continue` | Continue after resolving conflicts |
| `git cherry-pick --abort` | Cancel cherry-pick operation |
| `git cherry-pick --skip` | Skip current commit |
| `git cherry-pick -x <hash>` | Record original commit in message |
| `git cherry-pick -e <hash>` | Edit commit message |
| `git cherry-pick --no-commit` | Apply changes without committing |
| `git log <branch> --grep="text"` | Search commits by message |
| `git log <branch1>..<branch2>` | Commits in branch2 not in branch1 |

---

## Visualizing Cherry-Pick

### Before Cherry-Pick:
```
master:     A --- B --- C
                         ↑
                      master

feature:    A --- B --- D --- E --- F
                              ↑
                        "Update info.txt"
```

### After Cherry-Pick:
```
master:     A --- B --- C --- E'
                              ↑
                          master

feature:    A --- B --- D --- E --- F
                              ↑
                        "Update info.txt"
```

### Key Points:
- E and E' have **same changes**
- E and E' have **different hashes**
- E and E' have **same commit message**
- E remains on feature branch
- E' is new commit on master

---

## Key Takeaways

- **Cherry-pick** selectively applies specific commits to current branch
- **Useful for** work-in-progress, bug fixes, and selective integration
- **Creates new commit** with same changes but different hash
- **Preserves original** commit on source branch
- **Safe operation** - can be aborted if issues arise
- **Flexible** - pick one commit or many
- **Professional practice** - common in real-world workflows
- **Remember to push** changes to share with team

---

## Real-World Applications

### 1. Production Hotfix
```bash
# Critical bug fixed in develop
git checkout develop
git commit -m "Fix payment processing bug"

# Apply immediately to production
git checkout master
git cherry-pick <fix-hash>
git push origin master
```

### 2. Feature Preview
```bash
# Show specific feature to stakeholders
git checkout demo-branch
git cherry-pick <feature-commit>
```

### 3. Backporting Fixes
```bash
# Fix in latest version
git checkout v3.0
git commit -m "Security patch"

# Backport to older versions
git checkout v2.0
git cherry-pick <patch-hash>
```

### 4. Selective Code Review
```bash
# Review and integrate one commit at a time
git cherry-pick <reviewed-commit>
```

### 5. Cross-Project Integration
```bash
# Share fixes between related projects
git cherry-pick <commit-from-other-repo>
```

---

## Completion Checklist

- [ ] SSH into Storage Server (ststor01)
- [ ] Navigated to `/usr/src/kodekloudrepos/news`
- [ ] Verified repository status
- [ ] Listed all branches (master and feature)
- [ ] Checked out master branch
- [ ] Updated master branch (`git pull`)
- [ ] Found commit with "Update info.txt" message on feature branch
- [ ] Noted the commit hash
- [ ] Cherry-picked the commit to master
- [ ] Resolved any conflicts (if occurred)
- [ ] Verified cherry-pick success
- [ ] Checked commit appears on master
- [ ] Pushed changes to origin/master
- [ ] Verified remote updated

---

## Completion Details

- **Completion Date:** December 3, 2025
- **Day:** 28 / 100
- **Challenge:** KodeKloud 100 Days Cloud DevOps
- **Topic:** Git Cherry-Pick - Selective Commit Merging
- **Server:** Storage Server (ststor01)
- **Repository:** `/opt/news.git` → `/usr/src/kodekloudrepos/news`
- **Branches:** master, feature
- **Action:** Cherry-pick commit "Update info.txt"
- **Source Branch:** feature
- **Target Branch:** master
- **Command Used:** `git cherry-pick <hash>`
- **Final Step:** `git push origin master`
- **Key Skill:** Selective commit integration across branches
- **Status:** ✅ Successfully Completed

---

## Summary

This task demonstrated **selective commit integration** using `git cherry-pick`:

✅ **Identified target commit** on feature branch
✅ **Found specific commit** by message "Update info.txt"
✅ **Applied commit** to master branch
✅ **Created new commit** with same changes
✅ **Pushed changes** to remote repository

**Key Insight:** Cherry-pick is essential when you need specific changes from one branch without merging everything. It:
- Allows selective integration
- Keeps branches independent
- Copies commits between branches
- Creates new commit with same changes
- Preserves original commit in source branch

Unlike merge which brings all commits, cherry-pick targets individual commits. This is perfect for work-in-progress scenarios where only certain changes are ready for integration.

**Remember:** `git cherry-pick` = Selective Merge = Surgical Precision! 🍒
