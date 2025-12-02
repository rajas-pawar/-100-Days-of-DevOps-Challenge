# Day 27: Reverting Git Commits
**100 Days Cloud DevOps Challenge – KodeKloud**

---

## Objective

The Nautilus application development team reported an issue with recent commits in their Git repository. The DevOps team needs to revert the repository HEAD to the last commit.

**Requirements:**
1. Revert the latest commit (HEAD) to the previous commit in `/usr/src/kodekloudrepos/apps`
2. The previous commit should have "initial commit" message
3. Use commit message "revert apps" (all lowercase) for the revert commit

---

## Understanding Git Revert

**Git revert** creates a new commit that undoes changes from a previous commit. Unlike reset, it preserves history.

### Revert vs Reset vs Checkout

| Command | What It Does | History | Use Case |
|---------|-------------|---------|----------|
| `git revert` | Creates new commit undoing changes | Preserves | Safe for shared repos |
| `git reset` | Moves HEAD pointer | Rewrites | Local changes only |
| `git checkout` | Switches branches/files | No change | View old commits |

### Why Use Revert?

- **Safe:** Doesn't rewrite history
- **Traceable:** Shows what was undone
- **Collaborative:** Safe for shared repositories
- **Reversible:** Can revert the revert
- **Professional:** Standard practice in teams

---

## Infrastructure Overview

### Storage Server:
| Server   | User    | Password | IP            |
|----------|---------|----------|---------------|
| ststor01 | natasha | Bl@kW    | 172.16.238.15 |

### Repository Details:
| Item | Value |
|------|-------|
| Repository Path | `/usr/src/kodekloudrepos/apps` |
| Action | Revert latest commit (HEAD) |
| Target | Previous commit with "initial commit" message |
| Revert Message | "revert apps" (lowercase) |

---

## Understanding the Scenario

### Current State:
```
A (initial commit) --- B (bad commit) ← HEAD
```

### After Revert:
```
A (initial commit) --- B (bad commit) --- C (revert apps) ← HEAD
                                          ↑
                                    Undoes changes from B
```

**Note:** Commit B still exists in history, but commit C undoes its changes.

---

## Step-by-Step Implementation

### Step 1: SSH into Storage Server
```bash
ssh natasha@ststor01
```

**Enter password:** `Bl@kW`

### Step 2: Navigate to Repository
```bash
cd /usr/src/kodekloudrepos/apps
```

### Step 3: Verify Repository Status
```bash
git status
```

**Expected output:**
```
On branch master
Your branch is up to date with 'origin/master'.

nothing to commit, working tree clean
```

### Step 4: View Commit History
```bash
git log --oneline
```

**Expected output (example):**
```
def5678 (HEAD -> master) Some recent commit
abc1234 initial commit
```

Or with more detail:
```bash
git log
```

**Expected output:**
```
commit def5678... (HEAD -> master)
Author: Developer <dev@example.com>
Date:   Mon Dec 1 10:00:00 2025

    Some recent commit

commit abc1234...
Author: Developer <dev@example.com>
Date:   Mon Dec 1 09:00:00 2025

    initial commit
```

### Step 5: Identify Commits
```bash
# View last 2 commits
git log --oneline -n 2
```

**Identify:**
- **Current HEAD:** Latest commit (to be reverted)
- **Previous commit:** Should contain "initial commit" message

### Step 6: Verify the Previous Commit
```bash
# Show the commit before HEAD
git log --oneline HEAD~1 -n 1
```

**Expected:** Should show "initial commit"

Or:
```bash
# Search for initial commit
git log --oneline --grep="initial commit"
```

### Step 7: Revert the Latest Commit
```bash
git revert HEAD --no-edit
```

**Wait!** We need to use a custom message. Use this instead:
```bash
git revert HEAD -m "revert apps"
```

Or interactively:
```bash
git revert HEAD
```

This will open an editor where you can change the commit message to:
```
revert apps
```

**Recommended approach for this task:**
```bash
git revert HEAD --no-edit
git commit --amend -m "revert apps"
```

**Or in one command:**
```bash
git revert HEAD -e
```

Then in the editor, replace the default message with:
```
revert apps
```

Save and exit (`:wq` in vim, or `Ctrl+X` then `Y` then `Enter` in nano)

**Expected output:**
```
[master 1a2b3c4] revert apps
 1 file changed, X insertions(+), X deletions(-)
```

### Step 8: Verify the Revert
```bash
git log --oneline -n 3
```

**Expected output:**
```
1a2b3c4 (HEAD -> master) revert apps
def5678 Some recent commit
abc1234 initial commit
```

### Step 9: Check Repository Status
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

### Step 10: Verify Commit Message
```bash
git log -1
```

**Expected output:**
```
commit 1a2b3c4... (HEAD -> master)
Author: natasha <natasha@localhost>
Date:   Mon Dec 1 11:00:00 2025

    revert apps
```

### Step 11: View What Was Reverted
```bash
git show HEAD
```

This shows the changes made by the revert commit.

---

## Complete Command Summary

### Method 1: Quick Revert with Custom Message
```bash
# SSH into storage server
ssh natasha@ststor01

# Navigate to repository
cd /usr/src/kodekloudrepos/apps

# Check current status
git status

# View commit history
git log --oneline

# Revert latest commit with custom message
git revert HEAD --no-edit
git commit --amend -m "revert apps"

# Verify
git log --oneline -n 3
git status
```

### Method 2: Interactive Revert
```bash
# SSH and navigate
ssh natasha@ststor01
cd /usr/src/kodekloudrepos/apps

# View commits
git log --oneline -n 3

# Revert with editor
git revert HEAD

# In the editor that opens:
# - Delete default message
# - Type: revert apps
# - Save and exit

# Verify
git log --oneline -n 3
```

### Method 3: Using Environment Variable
```bash
# SSH and navigate
ssh natasha@ststor01
cd /usr/src/kodekloudrepos/apps

# Revert with message
git revert HEAD -m "revert apps"

# Verify
git log --oneline -n 3
```

---

## Understanding Git Revert Options

### Basic Revert:
```bash
# Revert latest commit
git revert HEAD
```

Opens editor for commit message

### Revert with Message:
```bash
# Revert with inline message
git revert HEAD -m "revert apps"
```

### No Edit (Use Default):
```bash
# Use default revert message
git revert HEAD --no-edit
```

### Revert Specific Commit:
```bash
# Revert by commit hash
git revert abc1234

# Revert by reference
git revert HEAD~2
```

### Revert Multiple Commits:
```bash
# Revert range (oldest first)
git revert HEAD~3..HEAD

# Revert multiple specific commits
git revert abc1234 def5678
```

### Revert Without Committing:
```bash
# Stage changes but don't commit
git revert HEAD --no-commit
```

---

## Verification Steps

### Verify 1: Check Commit Count
```bash
git log --oneline | wc -l
```

**Expected:** One more commit than before

### Verify 2: Check Latest Commit Message
```bash
git log -1 --pretty=%B
```

**Expected output:**
```
revert apps
```

### Verify 3: Check Commit Author
```bash
git log -1 --pretty="%an <%ae>"
```

**Expected:** Your username (natasha)

### Verify 4: Verify Changes Were Reversed
```bash
# Compare with commit before the one reverted
git diff HEAD~2 HEAD
```

**Expected:** Should show no differences (back to original state)

### Verify 5: Check Branch Status
```bash
git status
```

**Expected:** Clean working tree, ahead of origin by 1 commit

---

## Troubleshooting

### Issue 1: Merge Conflicts During Revert

**Problem:**
```
error: could not revert abc1234... commit message
hint: after resolving the conflicts, mark the corrected paths
hint: with 'git add <paths>' or 'git rm <paths>'
hint: and commit the result with 'git commit'
```

**Solution:**
```bash
# View conflicted files
git status

# Edit files to resolve conflicts
vi conflicted-file.txt

# After resolving
git add conflicted-file.txt

# Continue revert
git revert --continue

# Use correct commit message
# Type: revert apps
# Save and exit
```

### Issue 2: Wrong Commit Message

**Problem:** Used wrong commit message

**Solution:**
```bash
# Amend the last commit message
git commit --amend -m "revert apps"

# Verify
git log -1 --pretty=%B
```

### Issue 3: Reverted Wrong Commit

**Problem:** Reverted the wrong commit

**Solution:**
```bash
# Undo the revert (revert the revert)
git revert HEAD

# Or reset to before revert (if not pushed)
git reset --hard HEAD~1

# Then revert correct commit
git revert <correct-commit-hash>
```

### Issue 4: Cannot Find Initial Commit

**Problem:** Can't identify which commit is "initial commit"

**Solution:**
```bash
# Search for commits with "initial" in message
git log --oneline --all --grep="initial"

# View all commits
git log --oneline --all

# Show first commit
git log --reverse --oneline | head -1
```

### Issue 5: Detached HEAD State

**Problem:**
```
You are in 'detached HEAD' state...
```

**Solution:**
```bash
# Return to master branch
git checkout master

# Then perform revert
git revert HEAD -m "revert apps"
```

---

## Understanding Revert Mechanics

### What Happens During Revert:

1. **Git identifies changes** in the commit to revert
2. **Creates inverse changes** (opposite of original)
3. **Applies inverse changes** to current working directory
4. **Creates new commit** with these inverse changes
5. **Updates HEAD** to new commit

### Example:

**Original Commit (to revert):**
```diff
+ Added new feature
+ function newFeature() {
+     console.log("New");
+ }
```

**Revert Commit:**
```diff
- Removed new feature
- function newFeature() {
-     console.log("New");
- }
```

Result: Code returns to state before original commit

---

## Revert vs Reset Comparison

### Git Revert (This Task):
```bash
git revert HEAD
```

**Result:**
```
A --- B --- C (revert B)
          ↑
         HEAD
```

**Characteristics:**
- ✅ Safe for shared repositories
- ✅ Preserves history
- ✅ Traceable
- ✅ Can be undone
- ❌ Creates additional commit

### Git Reset (Alternative - NOT for shared repos):
```bash
git reset --hard HEAD~1
```

**Result:**
```
A --- B (deleted)
  ↑
 HEAD
```

**Characteristics:**
- ❌ Dangerous for shared repositories
- ❌ Rewrites history
- ❌ Can't be easily undone
- ✅ Clean history
- ✅ No extra commits

---

## When to Use Revert vs Reset

### Use Revert When:

- ✅ Working with shared/public repository
- ✅ Commits already pushed to remote
- ✅ Need to maintain history
- ✅ Multiple people working on repo
- ✅ Want traceable changes

### Use Reset When:

- ✅ Working locally only
- ✅ Commits not yet pushed
- ✅ Want clean history
- ✅ Solo developer
- ✅ Experimental branches

---

## Common Revert Scenarios

### Scenario 1: Revert Last Commit (This Task)
```bash
git revert HEAD -m "revert apps"
```

### Scenario 2: Revert Specific Old Commit
```bash
# Find commit hash
git log --oneline

# Revert it
git revert abc1234 -m "Revert feature X"
```

### Scenario 3: Revert Multiple Commits
```bash
# Revert last 3 commits (one by one)
git revert HEAD~2..HEAD --no-commit
git commit -m "Revert last 3 commits"
```

### Scenario 4: Revert and Continue Working
```bash
# Revert bad commit
git revert HEAD -m "Revert bug"

# Continue development
git checkout -b fix-branch
# ... make corrections ...
git commit -m "Proper fix"
```

### Scenario 5: Revert Merge Commit
```bash
# Revert merge (specify parent)
git revert -m 1 merge-commit-hash
```

---

## Git Log Options for Verification

### View Commit Graph:
```bash
git log --oneline --graph --all
```

### View Commit with Changes:
```bash
git log -p -1
```

### View Specific Commit:
```bash
git show HEAD
git show abc1234
```

### Search Commits:
```bash
# By message
git log --grep="revert apps"

# By author
git log --author="natasha"

# By date
git log --since="2025-12-01"
```

### Pretty Format:
```bash
git log --pretty=format:"%h - %an, %ar : %s"
```

---

## Best Practices

### 1. Always Verify Before Reverting
```bash
# Check what you're reverting
git log --oneline -n 3

# View the commit
git show HEAD

# Then revert
git revert HEAD
```

### 2. Use Descriptive Revert Messages
```bash
# Good
git revert HEAD -m "revert apps"
git revert HEAD -m "Revert broken login feature"

# Avoid
git revert HEAD -m "revert"
git revert HEAD -m "undo"
```

### 3. Test After Reverting
```bash
# After revert
# Test the application
# Verify functionality restored
# Then push
```

### 4. Document Why
```bash
# Add explanation in commit message
git revert HEAD -m "Revert login feature - causes production errors"
```

### 5. Communicate with Team
```bash
# Notify team of revert
# Explain why commit was reverted
# Coordinate fix
```

---

## Key Git Commands Reference

| Command | Description |
|---------|-------------|
| `git revert HEAD` | Revert latest commit |
| `git revert <hash>` | Revert specific commit |
| `git revert HEAD -m "msg"` | Revert with message |
| `git revert --no-edit` | Use default message |
| `git revert --no-commit` | Stage but don't commit |
| `git revert --continue` | Continue after conflict |
| `git revert --abort` | Cancel revert |
| `git log --oneline` | View commit history |
| `git show HEAD` | Show latest commit details |
| `git commit --amend` | Modify last commit |

---

## Understanding Commit References

### HEAD:
```bash
HEAD        # Current commit
HEAD~1      # Previous commit
HEAD~2      # Two commits back
HEAD^       # Parent commit
```

### By Hash:
```bash
git revert abc1234    # Full or partial hash
```

### By Branch:
```bash
git revert master~2   # Two commits back on master
```

### By Tag:
```bash
git revert v1.0.0     # Commit tagged as v1.0.0
```

---

## Key Takeaways

- **Revert creates new commit** that undoes previous changes
- **Safe for shared repositories** - doesn't rewrite history
- **Preserves all commits** - original commit still visible
- **Custom commit messages** clarify why revert was needed
- **Traceable** - team can see what was undone and why
- **Reversible** - can revert the revert if needed
- **HEAD reference** points to latest commit
- **Professional practice** - used in production environments

---

## Real-World Applications

### 1. Emergency Rollback
```bash
# Production issue detected
git revert HEAD -m "Revert deployment - critical bug"
git push origin master
# Deploy fixed version
```

### 2. Feature Removal
```bash
# Remove unwanted feature
git revert feature-commit -m "Remove analytics feature"
```

### 3. Bug Introduction
```bash
# Bug found in recent commit
git revert buggy-commit -m "Revert change causing login failures"
```

### 4. Configuration Error
```bash
# Wrong config committed
git revert config-commit -m "Revert incorrect database config"
```

### 5. Merge Gone Wrong
```bash
# Bad merge
git revert merge-commit -m 1 "Revert problematic merge"
```

---

## Completion Checklist

- [x] SSH into Storage Server (ststor01)
- [x] Navigated to repository `/usr/src/kodekloudrepos/apps`
- [x] Verified repository status
- [x] Viewed commit history
- [x] Identified latest commit (HEAD)
- [x] Verified previous commit has "initial commit" message
- [x] Reverted latest commit (HEAD)
- [x] Used commit message "revert apps" (lowercase)
- [x] Verified revert commit created
- [x] Confirmed commit message is correct
- [x] Checked working tree is clean
- [x] Verified commit history shows revert

---

## Completion Details

- **Completion Date:** December 2, 2025
- **Day:** 27 / 100
- **Challenge:** KodeKloud 100 Days Cloud DevOps
- **Topic:** Reverting Git Commits Safely
- **Server:** Storage Server (ststor01)
- **Repository:** `/usr/src/kodekloudrepos/apps`
- **Action:** Revert HEAD to previous commit
- **Target Commit:** "initial commit"
- **Revert Message:** "revert apps"
- **Command Used:** `git revert HEAD -m "revert apps"`
- **Key Skill:** Safe commit reversal in shared repositories
- **Status:** ✅ Successfully Completed

---

## Summary

This task demonstrated **safe commit reversal** using `git revert`:

✅ **Identified problematic commit** (HEAD)
✅ **Verified target state** (initial commit)
✅ **Created revert commit** undoing changes
✅ **Used custom message** "revert apps"
✅ **Preserved history** - all commits visible

**Key Insight:** Revert is the professional way to undo commits in shared repositories. It:
- Doesn't rewrite history
- Keeps audit trail
- Safe for collaboration
- Shows what was undone and why

Unlike `git reset` which deletes commits, `git revert` creates a new commit that undoes changes. This makes it safe for repositories where others are working.

**Remember:** `git revert HEAD` = Safe Undo = Production Ready! 🔄