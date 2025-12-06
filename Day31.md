# Day 31: Git Stash - Managing Work in Progress
**100 Days Cloud DevOps Challenge – KodeKloud**

---

## Objective

The Nautilus application development team has stashed some in-progress changes in their git repository. Now they need to restore specific stashed changes to continue their work.

**Requirements:**
1. Work with repository `/usr/src/kodekloudrepos/ecommerce` on Storage server
2. View all stashed changes in the repository
3. Restore the stash with identifier `stash@{1}`
4. Commit the restored changes
5. Push changes to origin

---

## Understanding Git Stash

**Git stash** temporarily saves (or "stashes") uncommitted changes (both staged and unstaged) so you can work on something else. Later, you can restore those changes and continue where you left off.

### Why Use Git Stash?

- **Context Switching:** Switch branches without committing incomplete work
- **Emergency Fixes:** Quickly set aside current work for urgent tasks
- **Clean Working Directory:** Temporarily clear changes without losing them
- **Experiment Safely:** Try different approaches without commits
- **Save Multiple States:** Stack multiple sets of changes

### Stash Workflow:

```
1. Working on feature A
2. Urgent bug needs fixing
3. Stash current changes
4. Switch to bugfix branch
5. Fix and commit bug
6. Switch back to feature A
7. Pop stashed changes
8. Continue working
```

---

## Infrastructure Overview

### Storage Server:
| Server   | User    | Password | IP            |
|----------|---------|----------|---------------|
| ststor01 | natasha | Bl@kW    | 172.16.238.15 |

### Repository Details:
| Item | Value |
|------|-------|
| Repository Path | `/usr/src/kodekloudrepos/ecommerce` |
| Current State | Has stashed changes |
| Target Stash | `stash@{1}` (second stash in stack) |
| Action | Restore stash, commit, and push |
| Final Step | Push changes to origin |

---

## Understanding the Scenario

### Stash Stack Structure:
```
stash@{0}  ← Most recent stash (top of stack)
stash@{1}  ← Second stash (our target)
stash@{2}  ← Third stash
stash@{3}  ← Oldest stash (bottom of stack)
```

### Current State:
```
Working Directory: Clean
Stash Stack: Contains multiple stashes
Target: stash@{1}
```

### After Restoring stash@{1}:
```
Working Directory: Contains changes from stash@{1}
Stash Stack: stash@{1} still exists (unless we use pop)
Next: Commit and push changes
```

---

## Step-by-Step Implementation

### Step 1: SSH into Storage Server
```bash
ssh natasha@ststor01
```

**Enter password:** `Bl@kW`

### Step 2: Navigate to Repository
```bash
cd /usr/src/kodekloudrepos/ecommerce
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

### Step 4: List All Stashed Changes
```bash
git stash list
```

**Expected output (example):**
```
stash@{0}: WIP on master: abc1234 Latest commit message
stash@{1}: WIP on feature: def5678 Working on checkout feature
stash@{2}: On master: xyz9876 Payment gateway changes
```

**Note:** Your output will vary based on actual stashes

### Step 5: View Stash Details
```bash
# View what's in stash@{1}
git stash show stash@{1}
```

**Expected output:**
```
 checkout.js    | 15 ++++++++-------
 payment.js     |  5 +++--
 2 files changed, 11 insertions(+), 11 deletions(-)
```

**More detailed view:**
```bash
git stash show -p stash@{1}
```

**Shows actual code changes (diff format)**

### Step 6: View Stash@{1} Complete Details
```bash
git show stash@{1}
```

**This shows:**
- Commit information
- Author
- Date
- Files changed
- Complete diff

### Step 7: Check Current Branch
```bash
git branch
```

**Expected output:**
```
* master
```

Make sure you're on the correct branch before applying stash.

### Step 8: Verify Working Directory is Clean
```bash
git status
```

**Expected:**
```
nothing to commit, working tree clean
```

**Important:** Apply stash to clean working directory to avoid conflicts

### Step 9: Apply Stash@{1}
```bash
git stash apply stash@{1}
```

**Expected output:**
```
On branch master
Your branch is up to date with 'origin/master'.

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   checkout.js
        modified:   payment.js

no changes added to commit (use "git add" and/or "git commit -a")
```

**Note:** `apply` keeps the stash in the stack. Use `pop` to remove it.

**Alternative - Apply and Remove:**
```bash
git stash pop stash@{1}
```

This applies the stash AND removes it from the stash stack.

### Step 10: Verify Changes Applied
```bash
git status
```

**Expected output:**
```
On branch master
Your branch is up to date with 'origin/master'.

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   checkout.js
        modified:   payment.js

no changes added to commit (use "git add" and/or "git commit -a")
```

### Step 11: View Applied Changes
```bash
git diff
```

**Shows:** Detailed changes that were restored from stash

**View specific file:**
```bash
git diff checkout.js
```

### Step 12: Stage All Changes
```bash
git add .
```

**Or stage specific files:**
```bash
git add checkout.js payment.js
```

### Step 13: Verify Staged Changes
```bash
git status
```

**Expected output:**
```
On branch master
Your branch is up to date with 'origin/master'.

Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        modified:   checkout.js
        modified:   payment.js
```

### Step 14: Commit Changes
```bash
git commit -m "Restored changes from stash@{1} - checkout feature"
```

**Use descriptive commit message indicating what was restored**

**Expected output:**
```
[master 1a2b3c4] Restored changes from stash@{1} - checkout feature
 2 files changed, 11 insertions(+), 11 deletions(-)
```

### Step 15: Verify Commit
```bash
git log --oneline -n 3
```

**Expected output:**
```
1a2b3c4 (HEAD -> master) Restored changes from stash@{1} - checkout feature
abc1234 (origin/master) Latest commit message
...
```

### Step 16: Check Status Before Push
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

### Step 17: Push Changes to Origin
```bash
git push origin master
```

**Expected output:**
```
Counting objects: 5, done.
Delta compression using up to 2 threads.
Compressing objects: 100% (3/3), done.
Writing objects: 100% (5/5), 512 bytes | 512.00 KiB/s, done.
Total 5 (delta 2), reused 0 (delta 0)
To /opt/ecommerce.git
   abc1234..1a2b3c4  master -> master
```

### Step 18: Verify Push Success
```bash
git status
```

**Expected output:**
```
On branch master
Your branch is up to date with 'origin/master'.

nothing to commit, working tree clean
```

### Step 19: Verify Remote Updated
```bash
git log origin/master --oneline -n 3
```

**Expected output:**
```
1a2b3c4 (HEAD -> master, origin/master) Restored changes from stash@{1} - checkout feature
abc1234 Latest commit message
...
```

### Step 20: Optional - Clean Up Stash
```bash
# View stash list
git stash list

# If you want to remove stash@{1} after applying
git stash drop stash@{1}

# Or clear all stashes
git stash clear
```

---

## Complete Command Summary

### Quick Method:
```bash
# SSH into storage server
ssh natasha@ststor01

# Navigate to repository
cd /usr/src/kodekloudrepos/ecommerce

# List stashes
git stash list

# View stash@{1} contents
git stash show stash@{1}

# Apply stash@{1}
git stash apply stash@{1}

# Stage changes
git add .

# Commit changes
git commit -m "Restored changes from stash@{1}"

# Push to origin
git push origin master

# Verify
git status
```

### Detailed Method with Verification:
```bash
# SSH and navigate
ssh natasha@ststor01
cd /usr/src/kodekloudrepos/ecommerce

# Check current status
git status

# List all stashes
git stash list

# View what's in stash@{1}
git stash show stash@{1}

# View detailed diff
git stash show -p stash@{1}

# Ensure working directory is clean
git status

# Apply stash@{1}
git stash apply stash@{1}

# Verify changes
git status
git diff

# Stage all changes
git add .

# Verify staged
git status

# Commit
git commit -m "Restored changes from stash@{1} - checkout feature"

# Verify commit
git log --oneline -n 3

# Push to remote
git push origin master

# Verify push
git status
git log origin/master --oneline -n 3
```

---

## Understanding Git Stash Commands

### Stashing Changes:

**Basic stash:**
```bash
git stash
# or
git stash save
```

**Stash with message:**
```bash
git stash save "Working on checkout feature"
```

**Stash including untracked files:**
```bash
git stash -u
# or
git stash --include-untracked
```

**Stash everything (including ignored files):**
```bash
git stash -a
# or
git stash --all
```

**Stash only staged changes:**
```bash
git stash --staged
```

### Viewing Stashes:

**List all stashes:**
```bash
git stash list
```

**Show stash summary:**
```bash
git stash show stash@{1}
```

**Show stash diff:**
```bash
git stash show -p stash@{1}
```

**Show complete stash details:**
```bash
git show stash@{1}
```

### Applying Stashes:

**Apply most recent stash (keep in stack):**
```bash
git stash apply
```

**Apply specific stash (keep in stack):**
```bash
git stash apply stash@{1}
```

**Apply and remove most recent stash:**
```bash
git stash pop
```

**Apply and remove specific stash:**
```bash
git stash pop stash@{1}
```

### Managing Stashes:

**Drop specific stash:**
```bash
git stash drop stash@{1}
```

**Clear all stashes:**
```bash
git stash clear
```

**Create branch from stash:**
```bash
git stash branch new-branch-name stash@{1}
```

---

## Stash vs Apply vs Pop

### git stash apply (This Task):
```bash
git stash apply stash@{1}
```

**Effect:**
- ✅ Applies stash to working directory
- ❌ Keeps stash in stack
- ✅ Can apply same stash multiple times

**Use case:** Want to apply stash but keep it for later

### git stash pop:
```bash
git stash pop stash@{1}
```

**Effect:**
- ✅ Applies stash to working directory
- ✅ Removes stash from stack
- ❌ Can't reapply (unless you committed)

**Use case:** Done with stash, don't need it anymore

### Comparison:

| Aspect | apply | pop |
|--------|-------|-----|
| Applies changes | ✅ | ✅ |
| Removes from stack | ❌ | ✅ |
| Can reuse stash | ✅ | ❌ |
| On conflict | Keeps stash | Keeps stash |

---

## Stash Stack Indexing

### How Stash Stack Works:

```
git stash  (first time)
→ stash@{0}: First stash

git stash  (second time)
→ stash@{0}: Second stash (new)
→ stash@{1}: First stash (pushed down)

git stash  (third time)
→ stash@{0}: Third stash (new)
→ stash@{1}: Second stash (middle)
→ stash@{2}: First stash (oldest)
```

**Key points:**
- Index 0 is always the most recent
- Each new stash pushes others down
- After pop/drop, indices renumber

### After Dropping Stash:
```
Before: stash@{0}, stash@{1}, stash@{2}

git stash drop stash@{1}

After: stash@{0}, stash@{1} (was stash@{2})
```

---

## Handling Stash Conflicts

### Scenario: Conflicts When Applying Stash

**Problem:**
```
Auto-merging checkout.js
CONFLICT (content): Merge conflict in checkout.js
The stash entry is kept in case you need it again.
```

**Solution:**
```bash
# View conflicted files
git status

# Files with conflicts shown as "both modified"
# Edit conflicted file
vi checkout.js

# Look for conflict markers:
# <<<<<<< Updated upstream
# ... current branch content ...
# =======
# ... stashed content ...
# >>>>>>> Stashed changes

# Resolve conflicts manually
# Remove conflict markers
# Keep desired changes

# Stage resolved file
git add checkout.js

# Verify resolution
git status

# Commit
git commit -m "Resolved stash conflicts"

# Clean up stash (optional)
git stash drop stash@{1}
```

---

## Stash Use Cases

### Use Case 1: Emergency Context Switch (This Task)
```bash
# Working on feature
git stash

# Fix urgent bug
git checkout hotfix-branch
# ... fix bug ...
git commit -m "Fix critical bug"
git push

# Back to feature
git checkout feature-branch
git stash pop
```

### Use Case 2: Pull Latest Changes
```bash
# Have local uncommitted changes
git stash

# Pull latest from remote
git pull origin master

# Reapply your changes
git stash pop
```

### Use Case 3: Experiment with Changes
```bash
# Stash current work
git stash

# Try different approach
# ... make changes ...

# Don't like it?
git checkout .

# Restore original approach
git stash pop
```

### Use Case 4: Apply Changes to Different Branch
```bash
# Accidentally worked on wrong branch
git stash

# Switch to correct branch
git checkout correct-branch

# Apply changes
git stash pop
```

### Use Case 5: Save Multiple Work States
```bash
# Stash partial work
git stash save "Partial checkout implementation"

# Work on something else
git stash save "Payment gateway API draft"

# Later, view and restore specific work
git stash list
git stash apply stash@{1}
```

---

## Best Practices

### 1. Use Descriptive Stash Messages
```bash
# Good
git stash save "WIP: Checkout page validation logic"
git stash save "Bugfix attempt for payment timeout"

# Bad
git stash save "work"
git stash save "stuff"
```

### 2. List Stashes Regularly
```bash
# Don't let stashes accumulate
git stash list

# Clean up old stashes
git stash drop stash@{3}
```

### 3. Apply Stashes to Clean Working Directory
```bash
# Before applying stash
git status  # Should be clean

# If not clean, commit or stash current changes first
git stash
git stash apply stash@{1}
```

### 4. Test After Applying Stash
```bash
# After applying stash
# Run tests
npm test
# or
pytest

# Ensure nothing broke
```

### 5. Don't Rely on Stash Long-Term
```bash
# Stash is for SHORT-TERM storage
# For long-term: use branches

# Instead of stashing for weeks:
git checkout -b feature/new-feature
git commit -m "WIP: Initial work"
```

### 6. Create Branch from Stash for Complex Work
```bash
# If stash is complex
git stash branch feature-from-stash stash@{1}

# This:
# 1. Creates new branch
# 2. Checks it out
# 3. Applies stash
# 4. Drops the stash
```

---

## Verification Steps

### Verify 1: Stash Exists
```bash
git stash list | grep "stash@{1}"
```

**Expected:** Shows stash@{1} entry

### Verify 2: Changes Applied
```bash
git status
```

**Expected:** Shows modified files

### Verify 3: Changes Committed
```bash
git log --oneline -n 1
```

**Expected:** Shows your commit message

### Verify 4: Push Successful
```bash
git status
```

**Expected:**
```
Your branch is up to date with 'origin/master'.
```

### Verify 5: Remote Has Commit
```bash
git log origin/master --oneline -n 1
```

**Expected:** Shows your commit on remote

---

## Troubleshooting

### Issue 1: No Stashes Found

**Problem:**
```
No stash entries found.
```

**Solution:**
```bash
# Verify you're in correct repository
pwd

# Check git repository
git status

# Check all branches
git stash list --all

# Stashes might be on different branch
git checkout <other-branch>
git stash list
```

### Issue 2: Stash@{1} Doesn't Exist

**Problem:**
```
error: stash@{1} does not exist
```

**Solution:**
```bash
# List available stashes
git stash list

# Use available index
# If only stash@{0} exists, use that
git stash apply stash@{0}
```

### Issue 3: Conflicts When Applying Stash

**Problem:**
```
CONFLICT (content): Merge conflict in file.js
```

**Solution:**
```bash
# View conflicts
git status

# Edit conflicted files
vi file.js

# Resolve conflicts
# Stage resolved files
git add file.js

# Continue (no need for git stash continue)
# Just commit
git commit -m "Resolved stash conflicts"
```

### Issue 4: Applied Wrong Stash

**Problem:** Applied stash@{0} instead of stash@{1}

**Solution:**
```bash
# Discard applied changes
git reset --hard HEAD

# Apply correct stash
git stash apply stash@{1}
```

### Issue 5: Stash Apply Failed

**Problem:**
```
error: Your local changes to the following files would be overwritten
```

**Solution:**
```bash
# Working directory not clean
# Commit or stash current changes first
git add .
git commit -m "Current work"

# Then apply stash
git stash apply stash@{1}
```

### Issue 6: Lost Stash After Pop

**Problem:** Popped stash but didn't commit, now can't reapply

**Solution:**
```bash
# Use reflog to find lost stash
git fsck --unreachable | grep commit

# Or check stash reflog
git log --graph --oneline --decorate \
  $(git fsck --no-reflog | \
  awk '/dangling commit/ {print $3}')

# Recover if found
git stash apply <commit-hash>
```

---

## Advanced Stash Operations

### Create Partial Stash (Interactive):
```bash
# Stash specific files interactively
git stash -p

# For each change, choose:
# y - stash this hunk
# n - don't stash this hunk
# q - quit
# a - stash this and remaining hunks
# d - don't stash this and remaining hunks
```

### Stash Specific Files:
```bash
# Stash only specific files
git stash push -m "Stash checkout only" checkout.js payment.js
```

### View Stash as Patch:
```bash
# Generate patch from stash
git stash show -p stash@{1} > stash.patch

# Apply patch elsewhere
git apply stash.patch
```

### Search Stash Content:
```bash
# Search for text in stashes
git stash list | while read stash; do
  echo "$stash"
  git stash show -p $stash | grep "search-term"
done
```

---

## Stash Internal Mechanics

### What Git Stash Creates:

When you stash, Git creates special commits:

```
Working Directory Changes
        ↓
git stash
        ↓
Creates 3 commits:
1. Stash Index (staged changes)
2. Stash Working Tree (unstaged changes)
3. Stash Commit (references both)
```

### Stash References:
```bash
# Stash is stored in refs/stash
cat .git/refs/stash

# Stash reflog
git reflog show stash

# Shows all stash operations
```

---

## Real-World Applications

### 1. Urgent Production Fix
```bash
# Working on feature
git stash save "Feature X in progress"

# Fix production issue
git checkout master
git pull origin master
# ... fix issue ...
git commit -m "Hotfix: Critical bug"
git push origin master

# Back to feature
git checkout feature-x
git stash pop
```

### 2. Code Review Context Switch
```bash
# Working on Task A
git stash save "Task A - validation logic"

# Review colleague's PR
git fetch origin
git checkout pr/123
# ... review code ...

# Back to Task A
git checkout feature-a
git stash pop
```

### 3. Experimental Changes
```bash
# Try risky refactoring
git stash save "Backup before refactoring"

# Make experimental changes
# ... refactor code ...

# Didn't work? Restore
git reset --hard HEAD
git stash pop
```

### 4. Sync with Team
```bash
# Have local changes
git stash

# Pull team's updates
git pull origin master

# Reapply your changes on top
git stash pop

# Resolve any conflicts
# Commit and push
```

---

## Key Git Commands Reference

| Command | Description |
|---------|-------------|
| `git stash` | Stash current changes |
| `git stash save "message"` | Stash with custom message |
| `git stash list` | List all stashes |
| `git stash show stash@{n}` | Show stash summary |
| `git stash show -p stash@{n}` | Show stash diff |
| `git stash apply stash@{n}` | Apply stash (keep in stack) |
| `git stash pop stash@{n}` | Apply and remove stash |
| `git stash drop stash@{n}` | Delete specific stash |
| `git stash clear` | Delete all stashes |
| `git stash branch <name> stash@{n}` | Create branch from stash |

---

## Stash Workflow Patterns

### Pattern 1: Quick Context Switch
```bash
git stash                    # Save current work
git checkout other-branch    # Switch context
# ... do work ...
git checkout -               # Back to previous branch
git stash pop               # Restore work
```

### Pattern 2: Pull and Reapply
```bash
git stash                    # Save local changes
git pull origin master       # Get updates
git stash pop               # Reapply changes
# Resolve conflicts if any
git add .
git commit -m "Merged with latest"
```

### Pattern 3: Multiple Experiments
```bash
git stash save "Approach A"  # Save approach A
# Try approach B
git stash save "Approach B"  # Save approach B
# Try approach C

# Review all approaches
git stash list
git stash apply stash@{1}    # Try approach A
# Test it
git reset --hard HEAD
git stash apply stash@{0}    # Try approach B
```

---

## Key Takeaways

- **Git stash** temporarily saves uncommitted changes
- **Stash stack** maintains multiple saved states
- **stash@{0}** is always the most recent stash
- **apply** keeps stash, **pop** removes it
- **Stash is short-term** - use branches for long-term storage
- **Always verify** what you're applying before applying
- **Commit after applying** to persist changes
- **Clean working directory** before applying stash to avoid conflicts

---

## Completion Checklist

- [ ] SSH into Storage Server
- [ ] Navigated to `/usr/src/kodekloudrepos/ecommerce`
- [ ] Verified repository status (clean working directory)
- [ ] Listed all stashes with `git stash list`
- [ ] Viewed stash@{1} details
- [ ] Verified stash@{1} contents
- [ ] Applied stash@{1} to working directory
- [ ] Verified changes appeared in working directory
- [ ] Staged all changes
- [ ] Committed changes with descriptive message
- [ ] Verified commit created
- [ ] Pushed commit to origin/master
- [ ] Verified push success
- [ ] Confirmed remote updated

---

## Completion Details

- **Completion Date:** December 6, 2025
- **Day:** 31 / 100
- **Challenge:** KodeKloud 100 Days Cloud DevOps
- **Topic:** Git Stash - Managing Work in Progress
- **Server:** Storage Server (ststor01)
- **Repository:** `/usr/src/kodekloudrepos/ecommerce`
- **Initial State:** Clean working directory with stashes
- **Target:** stash@{1}
- **Action:** Apply stash, commit, and push
- **Command Used:** `git stash apply stash@{1}`
- **Commit:** Restored changes from stash
- **Push:** `git push origin master`
- **Key Skill:** Stash management and restoration
- **Status:** ✅ Successfully Completed

---

## Summary

This task demonstrated **Git stash management** for handling work-in-progress:

✅ **Listed all stashes** to see available saved states
✅ **Viewed stash@{1} details** to understand changes
✅ **Applied stash@{1}** to working directory
✅ **Committed restored changes** with proper message
✅ **Pushed to remote** to share work with team

**Key Insight:** Git stash is your temporary workspace manager. It:
- Saves uncommitted changes without committing
- Allows quick context switching
- Maintains multiple saved states in a stack
- Enables experimentation without losing work
- Provides flexibility in development workflow

Unlike commits which are permanent, stashes are temporary storage for work-in-progress. They're perfect for interruptions, context switches, and experimental work that isn't ready for a commit.

**Remember:** `git stash` = Temporary Workspace Saver = Context Switch Hero! 💾

**The DevOps Lesson:** Just like stashing code changes, in DevOps we often need to pause one task to handle urgent issues, then resume where we left off. Good workflow management is essential! 🔄
