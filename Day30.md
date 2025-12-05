# Day 30: Git Reset - Cleaning Commit History
**100 Days Cloud DevOps Challenge – KodeKloud**

---

## Objective

The Nautilus application development team has a test repository with experimental commits that need to be removed. They want to reset the repository to a specific commit, cleaning up the commit history and work tree.

**Requirements:**
1. Work with repository `/usr/src/kodekloudrepos/games` on Storage server
2. Reset git commit history to keep only two commits:
   - Initial commit
   - add data.txt file
3. Remove all commits after "add data.txt file"
4. Point HEAD and branch back to the "add data.txt file" commit
5. Push changes to remote repository

---

## Understanding Git Reset

**Git reset** moves the current branch pointer (HEAD) to a specific commit, effectively rewriting history. It's a powerful tool for cleaning up commits, but must be used carefully, especially with shared repositories.

### Reset vs Revert

| Command | What It Does | History | Use Case |
|---------|-------------|---------|----------|
| `git reset` | Moves HEAD, rewrites history | Deletes commits | Local cleanup, undo mistakes |
| `git revert` | Creates new commit undoing changes | Preserves all commits | Shared repos, safe undo |

### Git Reset Modes:

| Mode | HEAD | Index (Staging) | Working Directory | Use Case |
|------|------|----------------|-------------------|----------|
| `--soft` | ✅ Moves | ❌ Unchanged | ❌ Unchanged | Redo commits with same changes |
| `--mixed` (default) | ✅ Moves | ✅ Resets | ❌ Unchanged | Unstage changes, keep files |
| `--hard` | ✅ Moves | ✅ Resets | ✅ Resets | Complete cleanup, discard everything |

---

## Infrastructure Overview

### Storage Server:
| Server   | User    | Password | IP            |
|----------|---------|----------|---------------|
| ststor01 | natasha | Bl@kW    | 172.16.238.15 |

### Repository Details:
| Item | Value |
|------|-------|
| Repository Path | `/usr/src/kodekloudrepos/games` |
| Current State | Multiple test commits |
| Target State | Only 2 commits (initial + add data.txt file) |
| Target Commit Message | "add data.txt file" |
| Action | Hard reset to target commit |
| Final Step | Force push to remote |

---

## Understanding the Scenario

### Current State (Before Reset):
```
A (initial) --- B (add data.txt) --- C (test1) --- D (test2) --- E (test3)
                                                                   ↑
                                                                 HEAD
                                                                master
```

### Target State (After Reset):
```
A (initial) --- B (add data.txt)
                                ↑
                              HEAD
                            master

Commits C, D, E are deleted from history
```

**⚠️ Warning:** This permanently removes commits C, D, and E!

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

### Step 4: View Current Commit History
```bash
git log --oneline
```

**Expected output (example):**
```
e5f6g7h (HEAD -> master, origin/master) test commit 3
d4e5f6g test commit 2
c3d4e5f test commit 1
b2c3d4e add data.txt file
a1b2c3d initial commit
```

**Alternative - More detailed:**
```bash
git log
```

### Step 5: Count Current Commits
```bash
git log --oneline | wc -l
```

**Expected:** More than 2 commits (e.g., 5 or more)

### Step 6: Identify Target Commit
```bash
git log --oneline --grep="add data.txt file"
```

**Expected output:**
```
b2c3d4e add data.txt file
```

**Note the commit hash:** `b2c3d4e` (yours will be different)

**Alternative method:**
```bash
git log --oneline | grep -i "add data.txt"
```

### Step 7: Verify Initial Commit Exists
```bash
git log --oneline | tail -1
```

**Expected:** Shows the initial commit

**Or view all commits:**
```bash
git log --reverse --oneline
```

**Expected to see:**
```
a1b2c3d initial commit
b2c3d4e add data.txt file
...other commits...
```

### Step 8: View Target Commit Details
```bash
git show b2c3d4e
```

**Replace `b2c3d4e` with your actual commit hash**

**Expected:** Shows the commit that added data.txt file

### Step 9: Check Current Files
```bash
ls -la
```

**Note the current files before reset**

### Step 10: Reset to Target Commit (Hard Reset)
```bash
git reset --hard b2c3d4e
```

**Replace `b2c3d4e` with your target commit hash**

**Expected output:**
```
HEAD is now at b2c3d4e add data.txt file
```

**⚠️ Warning:** This permanently deletes all commits after `b2c3d4e`!

### Step 11: Verify Reset Success
```bash
git log --oneline
```

**Expected output:**
```
b2c3d4e (HEAD -> master) add data.txt file
a1b2c3d initial commit
```

**Should show only 2 commits now!**

### Step 12: Count Commits After Reset
```bash
git log --oneline | wc -l
```

**Expected output:** `2`

### Step 13: Check Working Directory
```bash
ls -la
```

**Verify:** Files match the state at "add data.txt file" commit

```bash
cat data.txt
```

**Expected:** File exists and contains expected content

### Step 14: Check Repository Status
```bash
git status
```

**Expected output:**
```
On branch master
Your branch is behind 'origin/master' by X commits, and can be fast-forwarded.
  (use "git pull" to update your local branch)

nothing to commit, working tree clean
```

**Wait, it says "behind"?** This is because remote still has the old commits!

### Step 15: View Difference with Remote
```bash
git log origin/master --oneline
```

**Shows:** Remote still has all the old commits

```bash
git log --oneline --graph --all
```

**Visualizes:** Local vs remote divergence

### Step 16: Force Push to Remote
```bash
git push origin master --force
```

**⚠️ Warning:** `--force` overwrites remote history!

**Expected output:**
```
Total 0 (delta 0), reused 0 (delta 0)
To /opt/games.git
 + e5f6g7h...b2c3d4e master -> master (forced update)
```

**Alternative (safer) force push:**
```bash
git push origin master --force-with-lease
```

This fails if someone else pushed changes, preventing accidental overwrites.

### Step 17: Verify Remote Updated
```bash
git log origin/master --oneline
```

**Expected output:**
```
b2c3d4e (HEAD -> master, origin/master) add data.txt file
a1b2c3d initial commit
```

**Should show only 2 commits!**

### Step 18: Final Status Check
```bash
git status
```

**Expected output:**
```
On branch master
Your branch is up to date with 'origin/master'.

nothing to commit, working tree clean
```

### Step 19: Verify Commit Count
```bash
git log --oneline | wc -l
```

**Expected:** `2`

### Step 20: Verify Both Commits Present
```bash
git log --oneline
```

**Must show exactly:**
1. ✅ initial commit
2. ✅ add data.txt file

---

## Complete Command Summary

### Quick Method:
```bash
# SSH into storage server
ssh natasha@ststor01

# Navigate to repository
cd /usr/src/kodekloudrepos/games

# View current commits
git log --oneline

# Find target commit
git log --oneline --grep="add data.txt file"

# Note the commit hash (e.g., b2c3d4e)

# Hard reset to target commit
git reset --hard b2c3d4e

# Verify only 2 commits remain
git log --oneline

# Force push to remote
git push origin master --force

# Verify success
git log --oneline | wc -l  # Should output: 2
```

### Detailed Method with Verification:
```bash
# SSH and navigate
ssh natasha@ststor01
cd /usr/src/kodekloudrepos/games

# Check current state
git status
git log --oneline

# Count commits
git log --oneline | wc -l

# Find target commit
git log --oneline | grep -i "add data.txt"

# View target commit details
git show <commit-hash>

# Perform hard reset
git reset --hard <commit-hash>

# Verify reset
git log --oneline
git log --oneline | wc -l

# Check files
ls -la
cat data.txt

# Check status
git status

# Force push
git push origin master --force

# Verify remote
git log origin/master --oneline
git status
```

---

## Understanding Git Reset Modes

### --soft (Keeps Everything Staged)
```bash
git reset --soft <commit>
```

**Effect:**
- ✅ Moves HEAD to commit
- ❌ Staging area unchanged
- ❌ Working directory unchanged

**Result:** All changes since commit are staged and ready to recommit

**Use case:** Redo commit message or combine commits

### --mixed (Default - Unstages Changes)
```bash
git reset --mixed <commit>
# or simply
git reset <commit>
```

**Effect:**
- ✅ Moves HEAD to commit
- ✅ Resets staging area
- ❌ Working directory unchanged

**Result:** Changes exist as unstaged modifications

**Use case:** Unstage files, reorganize commits

### --hard (Deletes Everything)
```bash
git reset --hard <commit>
```

**Effect:**
- ✅ Moves HEAD to commit
- ✅ Resets staging area
- ✅ Resets working directory

**Result:** Complete reset - all changes after commit are GONE

**Use case:** This task - complete cleanup, discard all changes

---

## Visual Comparison of Reset Modes

### Starting Point:
```
Commit History: A --- B --- C --- D
                                   ↑
                                 HEAD
                                master

Staged: file1.txt (modified)
Working Dir: file2.txt (modified), file3.txt (new)
```

### After: git reset --soft B
```
Commit History: A --- B
                       ↑
                     HEAD
                    master

Staged: all changes from C, D, file1.txt (all staged)
Working Dir: file2.txt, file3.txt (still there)
```

### After: git reset --mixed B (default)
```
Commit History: A --- B
                       ↑
                     HEAD
                    master

Staged: (empty)
Working Dir: changes from C, D, file1.txt, file2.txt, file3.txt (all unstaged)
```

### After: git reset --hard B (this task)
```
Commit History: A --- B
                       ↑
                     HEAD
                    master

Staged: (empty)
Working Dir: matches commit B exactly (everything else DELETED)
```

---

## Finding the Target Commit

### Method 1: Search by Message
```bash
git log --oneline --grep="add data.txt file"
```

### Method 2: View All Commits
```bash
git log --oneline
```

Look for "add data.txt file" manually

### Method 3: Reverse Order (Oldest First)
```bash
git log --reverse --oneline
```

**Shows:**
```
a1b2c3d initial commit
b2c3d4e add data.txt file  ← This is our target!
c3d4e5f test commit 1
...
```

### Method 4: Search Case-Insensitive
```bash
git log --oneline | grep -i "data.txt"
```

### Method 5: Show Commit Details
```bash
git log --all --grep="add data.txt" --pretty=fuller
```

---

## Force Push Explained

### Why Force Push is Needed:

**After local reset:**
```
Local:  A --- B
                ↑
              HEAD

Remote: A --- B --- C --- D --- E
                                ↑
                            origin/master
```

**Local is "behind" remote, but we want to make remote match local**

### Force Push Options:

**1. --force (Dangerous)**
```bash
git push origin master --force
```

- Overwrites remote no matter what
- Can lose others' work
- Use only if you're sure

**2. --force-with-lease (Safer)**
```bash
git push origin master --force-with-lease
```

- Checks if remote changed since your last fetch
- Fails if someone else pushed
- Safer option, prevents accidental overwrites

**3. --force-if-includes (Safest)**
```bash
git push origin master --force-if-includes
```

- Like --force-with-lease but stricter
- Requires Git 2.30+

### Force Push Result:
```
After force push:
Remote: A --- B
                ↑
          origin/master
          
Commits C, D, E are deleted from remote!
```

---

## Verification Steps

### Verify 1: Commit Count
```bash
git log --oneline | wc -l
```

**Expected:** `2`

### Verify 2: Specific Commits Exist
```bash
git log --oneline
```

**Must show:**
```
<hash2> add data.txt file
<hash1> initial commit
```

### Verify 3: No Extra Commits
```bash
git log --oneline | grep -i "test"
```

**Expected:** No results (all test commits removed)

### Verify 4: Remote Matches Local
```bash
git log origin/master --oneline
```

**Should match local exactly**

### Verify 5: Working Directory Clean
```bash
git status
```

**Expected:**
```
On branch master
Your branch is up to date with 'origin/master'.

nothing to commit, working tree clean
```

### Verify 6: File Content
```bash
cat data.txt
```

**Expected:** File exists and has content from "add data.txt file" commit

### Verify 7: Graph View
```bash
git log --oneline --graph --all
```

**Should show simple linear history with only 2 commits**

---

## Troubleshooting

### Issue 1: Can't Find Target Commit

**Problem:** No commit with message "add data.txt file"

**Solution:**
```bash
# Search variations
git log --oneline | grep -i "data"
git log --all --grep="data"

# View all commits
git log --oneline

# Look for commit that adds data.txt
git log --all --oneline -- data.txt
```

### Issue 2: Force Push Rejected

**Problem:**
```
! [rejected]        master -> master (non-fast-forward)
error: failed to push some refs
```

**Solution:**
```bash
# Ensure you used --force
git push origin master --force

# If still fails, check remote permissions
# Verify you're on correct branch
git branch

# Fetch and retry
git fetch origin
git push origin master --force
```

### Issue 3: Wrong Commit Reset

**Problem:** Reset to wrong commit by mistake

**Solution:**
```bash
# Find the previous HEAD position
git reflog

# Output shows:
# b2c3d4e HEAD@{0}: reset: moving to b2c3d4e
# e5f6g7h HEAD@{1}: commit: test commit 3  ← Previous HEAD

# Reset back to previous HEAD
git reset --hard HEAD@{1}

# Or use the commit hash
git reset --hard e5f6g7h

# Then find correct commit and reset again
```

### Issue 4: Lost Commits Accidentally

**Problem:** Reset too far, lost important commits

**Solution:**
```bash
# View reflog (history of HEAD movements)
git reflog

# Find the commit you want
# e5f6g7h HEAD@{5}: commit: important work

# Restore to that commit
git reset --hard e5f6g7h

# Or cherry-pick specific commits
git cherry-pick <commit-hash>
```

### Issue 5: Can't Count Commits Correctly

**Problem:** `wc -l` not working or giving wrong count

**Solution:**
```bash
# Alternative ways to count
git rev-list --count HEAD

# Or manually
git log --oneline

# Count visually - should see exactly:
# <hash> add data.txt file
# <hash> initial commit
```

### Issue 6: Remote Still Shows Old Commits

**Problem:** After force push, remote still has old commits

**Solution:**
```bash
# Fetch to update remote tracking
git fetch origin

# View remote
git log origin/master --oneline

# If still wrong, verify push succeeded
git push origin master --force

# Check you pushed to correct remote
git remote -v
```

---

## Git Reflog - Your Safety Net

### What is Reflog?

**Reflog** records every movement of HEAD. It's your "undo" for Git operations.

```bash
git reflog
```

**Output:**
```
b2c3d4e HEAD@{0}: reset: moving to b2c3d4e
e5f6g7h HEAD@{1}: commit: test commit 3
d4e5f6g HEAD@{2}: commit: test commit 2
c3d4e5f HEAD@{3}: commit: test commit 1
b2c3d4e HEAD@{4}: commit: add data.txt file
a1b2c3d HEAD@{5}: commit (initial): initial commit
```

### Using Reflog to Recover:

**Scenario:** Accidentally reset too far

```bash
# View reflog
git reflog

# Find the commit you want (e.g., HEAD@{1})
git reset --hard HEAD@{1}

# Or use commit hash
git reset --hard e5f6g7h
```

**Note:** Reflog only keeps history for ~90 days by default

---

## When to Use Git Reset

### ✅ Good Use Cases:

1. **Local cleanup before pushing**
   ```bash
   # Made messy commits locally, clean up before push
   git reset --soft HEAD~3
   git commit -m "Clean commit message"
   ```

2. **Remove sensitive data (before push)**
   ```bash
   git reset --hard HEAD~1
   # Remove commit with passwords/secrets
   ```

3. **Test repository cleanup (this task)**
   ```bash
   git reset --hard <commit>
   # Remove experimental commits
   ```

4. **Unstage files**
   ```bash
   git reset HEAD file.txt
   # Unstage file while keeping changes
   ```

5. **Redo last commit**
   ```bash
   git reset --soft HEAD~1
   # Recommit with changes
   ```

### ❌ Dangerous Use Cases:

1. **Shared/public branches**
   ```bash
   # DON'T reset commits that others have pulled
   # Use git revert instead
   ```

2. **After others have pulled**
   ```bash
   # Causes divergent history
   # Their clones become incompatible
   ```

3. **Production branches without communication**
   ```bash
   # Coordinate with team first
   # Document the reset
   ```

---

## Reset vs Revert vs Checkout

### Git Reset (This Task):
```bash
git reset --hard <commit>
```

**Effect:** Deletes commits, rewrites history
**Use:** Local cleanup, test repos
**Safety:** ⚠️ Dangerous on shared branches

### Git Revert (Day 27):
```bash
git revert <commit>
```

**Effect:** Creates new commit undoing changes
**Use:** Shared repos, production
**Safety:** ✅ Safe, preserves history

### Git Checkout:
```bash
git checkout <commit>
```

**Effect:** View old commit (detached HEAD)
**Use:** Inspect history, read-only
**Safety:** ✅ Safe, no changes

---

## Force Push Best Practices

### 1. Communicate with Team
```
Before force push:
- Notify team in chat/email
- Explain what's being reset
- Give time for others to push pending work
```

### 2. Use --force-with-lease
```bash
# Safer than --force
git push origin master --force-with-lease
```

### 3. Verify Before Force Push
```bash
# Check what will be pushed
git log origin/master..HEAD

# Verify remote
git log origin/master --oneline
```

### 4. Create Backup Tag
```bash
# Before force push, tag current state
git tag backup-before-reset

# If needed, restore later
git reset --hard backup-before-reset
```

### 5. Document the Action
```
- Update team wiki/docs
- Note what was reset and why
- Record date and person who did it
```

---

## Alternative Approaches

### Approach 1: Interactive Rebase (More Control)
```bash
git rebase -i HEAD~5

# In editor, delete lines for unwanted commits
# Save and exit
# Then force push
```

### Approach 2: Create New Branch
```bash
# Keep old branch, create new clean one
git checkout -b master-clean b2c3d4e
git push origin master-clean
```

### Approach 3: Revert Instead of Reset
```bash
# If already public, revert is safer
git revert HEAD~3..HEAD
git push origin master
```

---

## Key Git Commands Reference

| Command | Description |
|---------|-------------|
| `git reset --soft <commit>` | Move HEAD, keep staging and files |
| `git reset --mixed <commit>` | Move HEAD, unstage, keep files |
| `git reset --hard <commit>` | Move HEAD, discard everything |
| `git push --force` | Overwrite remote (dangerous) |
| `git push --force-with-lease` | Safer force push |
| `git reflog` | View HEAD movement history |
| `git log --oneline` | View commit history |
| `git log --grep="text"` | Search commits by message |
| `git log --oneline \| wc -l` | Count commits |
| `git reset --hard HEAD~1` | Reset to previous commit |

---

## Commit References

### Relative References:
```bash
HEAD        # Current commit
HEAD~1      # 1 commit before HEAD (parent)
HEAD~2      # 2 commits before HEAD (grandparent)
HEAD~3      # 3 commits before HEAD
HEAD^       # First parent (same as HEAD~1)
HEAD^^      # Grandparent (same as HEAD~2)
```

### Using Relative References:
```bash
# Reset to 3 commits ago
git reset --hard HEAD~3

# Reset to previous commit
git reset --hard HEAD~1

# Reset to parent commit
git reset --hard HEAD^
```

### By Commit Hash:
```bash
# Full hash
git reset --hard abc123def456789...

# Short hash (7+ characters)
git reset --hard abc123d
```

---

## Real-World Scenarios

### Scenario 1: Cleanup Test Repository (This Task)
```bash
# Remove experimental commits
git reset --hard <production-ready-commit>
git push origin master --force
```

### Scenario 2: Undo Last Commit (Keep Changes)
```bash
# Made commit too early
git reset --soft HEAD~1
# Make more changes
git commit -m "Complete feature"
```

### Scenario 3: Discard All Local Changes
```bash
# Start fresh from remote
git fetch origin
git reset --hard origin/master
```

### Scenario 4: Unstage Accidentally Staged Files
```bash
# Staged wrong files
git reset HEAD secret-file.txt
# Now file is unstaged but still modified
```

### Scenario 5: Remove Sensitive Data
```bash
# Committed password by mistake (not pushed yet!)
git reset --hard HEAD~1
# Remove the commit with sensitive data
```

---

## Key Takeaways

- **Git reset rewrites history** - moves HEAD and branch pointer
- **--hard is destructive** - permanently deletes commits and changes
- **Force push required** after reset to update remote
- **Use with caution** on shared branches - can break others' clones
- **Reflog is safety net** - can recover "lost" commits
- **Communicate with team** before force pushing
- **Test repos are safer** for practicing reset
- **For shared/production** repos, prefer `git revert`

---

## Completion Checklist

- [ ] SSH into Storage Server
- [ ] Navigated to `/usr/src/kodekloudrepos/games`
- [ ] Verified repository status
- [ ] Viewed current commit history
- [ ] Counted commits (more than 2 initially)
- [ ] Found "add data.txt file" commit
- [ ] Noted commit hash
- [ ] Verified "initial commit" exists
- [ ] Performed hard reset to target commit
- [ ] Verified only 2 commits remain locally
- [ ] Confirmed commits are: "initial commit" and "add data.txt file"
- [ ] Force pushed to remote
- [ ] Verified remote updated
- [ ] Confirmed commit count: exactly 2
- [ ] Verified working tree clean

---

## Completion Details

- **Completion Date:** December 5, 2025
- **Day:** 30 / 100
- **Challenge:** KodeKloud 100 Days Cloud DevOps
- **Topic:** Git Reset - Cleaning Commit History
- **Server:** Storage Server (ststor01)
- **Repository:** `/usr/src/kodekloudrepos/games`
- **Initial State:** Multiple test commits
- **Final State:** Only 2 commits (initial + add data.txt file)
- **Target Commit:** "add data.txt file"
- **Reset Mode:** `--hard` (complete cleanup)
- **Command Used:** `git reset --hard <commit-hash>`
- **Force Push:** `git push origin master --force`
- **Key Skill:** History manipulation and cleanup
- **Status:** ✅ Successfully Completed

---

## Summary

This task demonstrated **Git history cleanup** using `git reset --hard`:

✅ **Identified target commit** ("add data.txt file")
✅ **Hard reset** to remove all subsequent commits
✅ **Cleaned commit history** to only 2 commits
✅ **Force pushed** to update remote repository
✅ **Verified** both local and remote have correct history

**Key Insight:** Git reset is a powerful but dangerous tool. It:
- Permanently deletes commits from history
- Requires force push to update remote
- Can break others' clones if used on shared branches
- Is perfect for cleaning test repositories
- Should be used carefully with communication

Unlike `git revert` which preserves history, `git reset` rewrites it. This makes it ideal for test repositories and local cleanup, but dangerous for production branches where `revert` is preferred.

**Remember:** `git reset --hard` = History Eraser = Use with Caution! ⚠️

**The DevOps Lesson:** Just like cleaning up test commits, in production we must carefully clean up test infrastructure and resources. Always have a rollback plan! 🧹
