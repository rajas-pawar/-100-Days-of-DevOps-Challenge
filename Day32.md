# Day 32: Git Rebase - Maintaining Linear History
**100 Days Cloud DevOps Challenge – KodeKloud**

---

## Objective

The Nautilus development team has a feature branch with work in progress. The master branch has received new commits, and they need to update the feature branch with these changes without creating merge commits, maintaining a clean linear history.

**Requirements:**
1. Work with repository `/opt/news.git` cloned at `/usr/src/kodekloudrepos` on storage server
2. Rebase feature branch with master branch
3. Do NOT lose any data from feature branch
4. Do NOT create merge commits
5. Maintain linear commit history
6. Push changes after rebasing

---

## Understanding Git Rebase

**Git rebase** rewrites commit history by moving a series of commits to a new base commit. It replays your commits on top of another branch, creating a linear history without merge commits.

### Rebase vs Merge

| Aspect | Rebase | Merge |
|--------|--------|-------|
| History | Linear, clean | Shows branch structure |
| Commits | Rewrites (new hashes) | Preserves original |
| Graph | Straight line | Shows branches |
| Merge Commit | No | Yes |
| Use Case | Clean history | Preserve context |

### Why Use Rebase?

- **Clean History:** Linear commit sequence, easier to follow
- **No Merge Commits:** Avoids cluttering history
- **Easier Bisect:** Simpler to find bugs with `git bisect`
- **Professional:** Industry standard for feature branches
- **Readable:** `git log` shows straight timeline
- **Upstream Sync:** Keep feature branch updated with main development

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
| Clone Location | `/usr/src/kodekloudrepos/news` |
| Branches | `master` and `feature` |
| Current State | Master has new commits, feature is behind |
| Action | Rebase feature onto master |
| Requirement | No merge commits, no data loss |
| Final Step | Push rebased feature branch |

---

## Understanding the Scenario

### Current State (Before Rebase):
```
master:  A --- B --- C --- D --- E
                           ↑
                         master
                     (new commits)

feature: A --- B --- C --- F --- G --- H
                           ↑
                        feature
                   (work in progress)
```

**Diverged:** Feature branch split from master at commit C, then both branches evolved separately.

### After Rebase:
```
master:  A --- B --- C --- D --- E
                                 ↑
                               master

feature: A --- B --- C --- D --- E --- F' --- G' --- H'
                                              ↑
                                           feature
                                     (rebased commits)
```

**Note:** F', G', H' are new commits with same changes as F, G, H but different hashes

---

## Step-by-Step Implementation

### Step 1: SSH into Storage Server
```bash
ssh natasha@ststor01
```

**Enter password:** `Bl@kW`

### Step 2: Navigate to Repository
```bash
cd /usr/src/kodekloudrepos/news
```

### Step 3: Verify Repository Status
```bash
git status
```

**Expected output:**
```
On branch feature
Your branch is up to date with 'origin/feature'.

nothing to commit, working tree clean
```

### Step 4: List All Branches
```bash
git branch -a
```

**Expected output:**
```
  master
* feature
  remotes/origin/HEAD -> origin/master
  remotes/origin/master
  remotes/origin/feature
```

### Step 5: View Current Branch Commit History
```bash
git log --oneline
```

**Shows commits on feature branch**

### Step 6: Switch to Master Branch (Update)
```bash
git checkout master
```

**Expected output:**
```
Switched to branch 'master'
Your branch is up to date with 'origin/master'.
```

### Step 7: Pull Latest Master
```bash
git pull origin master
```

**Expected output:**
```
Already up to date.
```

**Or if there are updates:**
```
Updating abc1234..def5678
Fast-forward
 file.txt | 10 ++++++++++
 1 file changed, 10 insertions(+)
```

### Step 8: View Master Commit History
```bash
git log --oneline -n 5
```

**Note the commits on master**

### Step 9: Switch Back to Feature Branch
```bash
git checkout feature
```

**Expected output:**
```
Switched to branch 'feature'
Your branch is up to date with 'origin/feature'.
```

### Step 10: View Divergence
```bash
git log --oneline --graph --all
```

**Shows visual representation:**
```
* abc1234 (origin/feature, feature) Feature commit 3
* def5678 Feature commit 2
* ghi9012 Feature commit 1
| * jkl3456 (HEAD -> master, origin/master) Master update 2
| * mno7890 Master update 1
|/
* pqr1234 Common ancestor commit
```

### Step 11: View Commits Unique to Feature
```bash
git log master..feature --oneline
```

**Shows commits only on feature branch**

### Step 12: View Commits Unique to Master
```bash
git log feature..master --oneline
```

**Shows commits only on master branch**

### Step 13: Check Working Directory Clean
```bash
git status
```

**Expected:**
```
On branch feature
nothing to commit, working tree clean
```

**⚠️ Important:** Working directory must be clean before rebase

### Step 14: Perform Rebase
```bash
git rebase master
```

**Expected output (success):**
```
First, rewinding head to replay your work on top of it...
Applying: Feature commit 1
Applying: Feature commit 2
Applying: Feature commit 3
```

**This replays feature commits on top of master**

**If conflicts occur (see troubleshooting section):**
```
Auto-merging file.txt
CONFLICT (content): Merge conflict in file.txt
error: could not apply abc1234... Feature commit 1
Resolve all conflicts manually, mark them as resolved with
"git add/rm <conflicted_files>", then run "git rebase --continue".
You can instead skip this commit: run "git rebase --skip".
To abort and get back to the state before "git rebase", run "git rebase --abort".
Could not apply abc1234... Feature commit 1
```

### Step 15: Verify Rebase Success
```bash
git status
```

**Expected output:**
```
On branch feature
Your branch and 'origin/feature' have diverged,
and have X and Y different commits each, respectively.
  (use "git pull" to merge the remote branch into yours)

nothing to commit, working tree clean
```

### Step 16: View New Commit History
```bash
git log --oneline --graph
```

**Expected:** Linear history with feature commits on top of master commits

```
* new-hash3 (HEAD -> feature) Feature commit 3
* new-hash2 Feature commit 2
* new-hash1 Feature commit 1
* jkl3456 (origin/master, master) Master update 2
* mno7890 Master update 1
* pqr1234 Common ancestor commit
```

### Step 17: Verify No Merge Commits
```bash
git log --oneline | grep -i "merge"
```

**Expected:** No merge commits (empty output)

### Step 18: Compare with Master
```bash
git log master..HEAD --oneline
```

**Shows feature commits now based on latest master**

### Step 19: Force Push Feature Branch
```bash
git push origin feature --force-with-lease
```

**⚠️ Important:** Rebase rewrites history, so force push is required

**Expected output:**
```
Counting objects: 10, done.
Delta compression using up to 2 threads.
Compressing objects: 100% (8/8), done.
Writing objects: 100% (10/10), 1.23 KiB | 1.23 MiB/s, done.
Total 10 (delta 5), reused 0 (delta 0)
To /opt/news.git
 + abc1234...new-hash3 feature -> feature (forced update)
```

**Why force push?**
- Rebase creates new commit hashes
- Remote still has old commits
- Must overwrite remote history

### Step 20: Verify Remote Updated
```bash
git log origin/feature --oneline -n 5
```

**Expected:** Shows new rebased commits

### Step 21: Final Status Check
```bash
git status
```

**Expected output:**
```
On branch feature
Your branch is up to date with 'origin/feature'.

nothing to commit, working tree clean
```

### Step 22: Verify Linear History
```bash
git log --oneline --graph --all
```

**Expected:** Clean linear history, no branch divergence

---

## Complete Command Summary

### Quick Method:
```bash
# SSH into storage server
ssh natasha@ststor01

# Navigate to repository
cd /usr/src/kodekloudrepos/news

# Switch to master and update
git checkout master
git pull origin master

# Switch to feature branch
git checkout feature

# Rebase feature onto master
git rebase master

# Push rebased feature
git push origin feature --force-with-lease

# Verify
git log --oneline --graph --all
```

### Detailed Method with Verification:
```bash
# SSH and navigate
ssh natasha@ststor01
cd /usr/src/kodekloudrepos/news

# Check current branch
git status
git branch -a

# Update master branch
git checkout master
git pull origin master
git log --oneline -n 5

# Switch to feature
git checkout feature

# View current state
git log --oneline --graph --all
git log master..feature --oneline

# Ensure clean working directory
git status

# Perform rebase
git rebase master

# If conflicts, resolve them (see troubleshooting)

# Verify rebase
git status
git log --oneline --graph

# No merge commits check
git log --oneline | grep -i "merge"

# Force push (rebase changed history)
git push origin feature --force-with-lease

# Verify push
git status
git log origin/feature --oneline -n 5

# Final verification
git log --oneline --graph --all
```

---

## Understanding Git Rebase Mechanics

### How Rebase Works:

**Step-by-step process:**

1. **Find common ancestor** of current and target branch
2. **Save commits** from current branch since ancestor
3. **Reset branch** to target branch head
4. **Replay saved commits** one by one on new base
5. **Create new commits** with same changes but new hashes

### Visual Process:

**Before:**
```
      F --- G --- H  (feature)
     /
A --- B --- C --- D --- E  (master)
```

**During rebase (git rebase master):**
```
Step 1: Find common ancestor (C)
Step 2: Save F, G, H
Step 3: Move to E
Step 4: Apply F → F'
Step 5: Apply G → G'
Step 6: Apply H → H'
```

**After:**
```
A --- B --- C --- D --- E --- F' --- G' --- H'  (feature)
                         ↑
                      (master)
```

### Why Commit Hashes Change:

Each commit hash is calculated from:
- Commit content
- Parent commit hash
- Author info
- Timestamp

When parent changes (rebase), hash changes!

---

## Handling Rebase Conflicts

### When Conflicts Occur:

**Conflict message:**
```
Auto-merging file.txt
CONFLICT (content): Merge conflict in file.txt
error: could not apply abc1234... Feature commit 1
```

### Resolving Conflicts:

**Step 1: View conflicted files**
```bash
git status
```

**Output:**
```
rebase in progress; onto def5678
You are currently rebasing branch 'feature' on 'def5678'.
  (fix conflicts and then run "git rebase --continue")
  (use "git rebase --skip" to skip this patch)
  (use "git rebase --abort" to check out the original branch)

Unmerged paths:
  (use "git restore --staged <file>..." to unstage)
  (use "git add <file>..." to mark resolution)
        both modified:   file.txt
```

**Step 2: Edit conflicted files**
```bash
vi file.txt
```

**Look for conflict markers:**
```
<<<<<<< HEAD (master's version)
Content from master branch
=======
Content from feature branch
>>>>>>> abc1234 (Feature commit 1)
```

**Step 3: Resolve conflict**
- Keep needed changes
- Remove conflict markers
- Combine changes if necessary

**Step 4: Stage resolved files**
```bash
git add file.txt
```

**Step 5: Continue rebase**
```bash
git rebase --continue
```

**Step 6: Repeat if more conflicts**
Rebase applies commits one by one, so you may need to resolve conflicts multiple times.

### Rebase Control Commands:

**Continue after resolving:**
```bash
git rebase --continue
```

**Skip current commit:**
```bash
git rebase --skip
```

**Abort and return to original state:**
```bash
git rebase --abort
```

---

## Interactive Rebase

### Basic Interactive Rebase:
```bash
git rebase -i master
```

**Opens editor with:**
```
pick abc1234 Feature commit 1
pick def5678 Feature commit 2
pick ghi9012 Feature commit 3

# Commands:
# p, pick = use commit
# r, reword = use commit, but edit message
# e, edit = use commit, but stop for amending
# s, squash = use commit, but meld into previous
# f, fixup = like squash, but discard message
# d, drop = remove commit
```

### Interactive Rebase Actions:

**Reorder commits:** Change line order
**Squash commits:** Combine multiple commits
**Edit commits:** Modify commit content
**Reword messages:** Change commit messages
**Drop commits:** Remove commits

### Example: Squash Last 3 Commits
```bash
git rebase -i HEAD~3
```

**Change to:**
```
pick abc1234 Feature commit 1
squash def5678 Feature commit 2
squash ghi9012 Feature commit 3
```

**Result:** Single commit combining all three

---

## Rebase Best Practices

### 1. Never Rebase Public/Shared Branches
```
❌ DON'T: git checkout master && git rebase feature
✅ DO: git checkout feature && git rebase master
```

**Why?** Master is shared, rewriting its history breaks others' repos

### 2. Always Rebase Before Pushing
```bash
# Before creating PR
git checkout feature
git rebase master
git push origin feature
```

### 3. Use --force-with-lease, Not --force
```bash
# Safer
git push origin feature --force-with-lease

# Dangerous
git push origin feature --force
```

`--force-with-lease` fails if someone else pushed, preventing accidental overwrites

### 4. Communicate with Team
```
Before rebasing shared feature branches:
- Notify team members
- Ensure no one is working on it
- Coordinate timing
```

### 5. Keep Commits Atomic
```bash
# Before rebase, ensure each commit:
- Has single logical change
- Builds successfully
- Passes tests
- Has good commit message
```

### 6. Test After Rebase
```bash
# After rebase
git rebase master

# Run tests
npm test
# or
pytest

# Ensure nothing broke
```

### 7. Use Rebase for Feature Branches Only
```
✅ Good: Rebase feature/bugfix branches
❌ Avoid: Rebase master/main/production branches
```

---

## Rebase vs Merge - When to Use What

### Use Rebase When:

✅ **Feature branches** - Keep clean history
✅ **Local branches** - Not yet pushed
✅ **Sync with upstream** - Update feature with master
✅ **Before PR** - Clean up commits
✅ **Personal branches** - You're the only developer

### Use Merge When:

✅ **Main branches** - master, develop, release
✅ **Shared work** - Multiple developers on branch
✅ **Want to preserve** branch history
✅ **Integration points** - Merging completed features
✅ **Release branches** - Document when features merged

### Hybrid Approach (Common):

```bash
# On feature branch
git rebase master  # Clean up

# On master branch
git merge feature --no-ff  # Preserve integration point
```

**Result:** Clean feature history + documented integration

---

## Troubleshooting

### Issue 1: Conflicts During Rebase

**Problem:**
```
CONFLICT (content): Merge conflict in file.txt
```

**Solution:**
```bash
# View conflicts
git status

# Edit conflicted files
vi file.txt

# Resolve conflicts, remove markers
# Stage resolved files
git add file.txt

# Continue rebase
git rebase --continue

# Repeat until done
```

### Issue 2: Lost in Middle of Rebase

**Problem:** Not sure what state you're in

**Solution:**
```bash
# Check status
git status

# If in rebase
cat .git/rebase-merge/head-name

# Abort and start over
git rebase --abort

# Try again with clean slate
git rebase master
```

### Issue 3: Force Push Rejected

**Problem:**
```
! [rejected]        feature -> feature (non-fast-forward)
```

**Solution:**
```bash
# Use force-with-lease
git push origin feature --force-with-lease

# If someone else pushed, fetch first
git fetch origin
git log origin/feature

# Decide: rebase again or force push
```

### Issue 4: Accidentally Rebased Master

**Problem:** Rebased master branch (bad!)

**Solution:**
```bash
# Abort if still in progress
git rebase --abort

# If already completed, use reflog
git reflog

# Find previous master position
# HEAD@{n}: checkout: moving from master to feature

# Reset master
git reset --hard HEAD@{n}

# Verify
git log --oneline
```

### Issue 5: Rebase Completed But Wrong

**Problem:** Rebase finished but result is incorrect

**Solution:**
```bash
# Use reflog to find pre-rebase state
git reflog

# Find: HEAD@{n}: checkout: moving from master to feature
# This is before rebase

# Reset to pre-rebase
git reset --hard HEAD@{n}

# Try rebase again or use different approach
```

### Issue 6: Detached HEAD After Rebase

**Problem:**
```
You are in 'detached HEAD' state.
```

**Solution:**
```bash
# Create branch from current position
git checkout -b recovered-feature

# Or move existing branch here
git branch -f feature HEAD
git checkout feature
```

---

## Advanced Rebase Operations

### Rebase onto Different Branch:
```bash
# Rebase feature onto develop instead of master
git rebase --onto develop master feature
```

### Rebase Specific Range:
```bash
# Rebase only last 3 commits
git rebase -i HEAD~3
```

### Rebase and Autosquash:
```bash
# Mark commits for squashing
git commit --fixup=abc1234
git commit --squash=def5678

# Auto-arrange during rebase
git rebase -i --autosquash master
```

### Rebase with Strategy:
```bash
# Use specific merge strategy
git rebase -X theirs master

# Options:
# -X ours: Prefer current branch
# -X theirs: Prefer target branch
```

### Preserve Merge Commits:
```bash
# Rebase but keep merge commits
git rebase -p master
# or
git rebase --preserve-merges master
```

---

## Visualizing Rebase

### Before Rebase:
```bash
git log --oneline --graph --all
```

**Output:**
```
* abc1234 (feature) Feature commit 3
* def5678 Feature commit 2
* ghi9012 Feature commit 1
| * jkl3456 (master) Master commit 2
| * mno7890 Master commit 1
|/
* pqr1234 Common ancestor
```

### After Rebase:
```bash
git log --oneline --graph --all
```

**Output:**
```
* abc'234 (feature) Feature commit 3
* def'678 Feature commit 2
* ghi'012 Feature commit 1
* jkl3456 (master) Master commit 2
* mno7890 Master commit 1
* pqr1234 Common ancestor
```

**Notice:** Linear history, feature commits have new hashes

---

## Real-World Rebase Workflows

### Workflow 1: Daily Feature Development (This Task)
```bash
# Morning: Update with master
git checkout master
git pull origin master

git checkout feature
git rebase master

# Work on feature
# ... make changes ...
git add .
git commit -m "Progress"

# End of day: push
git push origin feature --force-with-lease
```

### Workflow 2: Pre-Pull Request Cleanup
```bash
# Clean up commits before PR
git rebase -i master

# Squash WIP commits
# Fix commit messages
# Reorder logically

# Push cleaned history
git push origin feature --force-with-lease

# Create PR
```

### Workflow 3: Sync Long-Running Feature
```bash
# Feature branch running for weeks
# Master has many new commits

# Weekly sync
git checkout master
git pull origin master

git checkout feature
git rebase master
# Resolve any conflicts

git push origin feature --force-with-lease
```

### Workflow 4: Collaborative Feature with Rebase
```bash
# Pull teammate's changes
git checkout feature
git pull origin feature

# Rebase on master
git rebase master

# Push with care
git push origin feature --force-with-lease

# Notify team: "Rebased feature on master"
```

---

## Key Git Commands Reference

| Command | Description |
|---------|-------------|
| `git rebase master` | Rebase current branch onto master |
| `git rebase -i master` | Interactive rebase |
| `git rebase --continue` | Continue after resolving conflicts |
| `git rebase --abort` | Cancel rebase, return to original |
| `git rebase --skip` | Skip current commit |
| `git push --force-with-lease` | Safer force push |
| `git log --graph --oneline` | Visualize commit history |
| `git reflog` | View HEAD movement history |
| `git rebase --onto <new> <old>` | Rebase onto different base |

---

## Commit History Comparison

### With Merge (Messy):
```
*   m4 Merge branch 'master' into feature
|\  
| * m3 Master commit 3
| * m2 Master commit 2
* | f3 Feature commit 3
* | f2 Feature commit 2
|/  
* m1 Common ancestor
```

### With Rebase (Clean):
```
* f3' Feature commit 3
* f2' Feature commit 2
* m3 Master commit 3
* m2 Master commit 2
* m1 Common ancestor
```

**Rebase advantage:** Linear, easier to read

---

## Key Takeaways

- **Rebase** replays commits on new base, creating linear history
- **No merge commits** - cleaner history
- **Rewrites history** - commit hashes change
- **Force push required** after rebase (use --force-with-lease)
- **Never rebase** public/shared branches (master, main)
- **Great for** feature branches and local cleanup
- **Conflicts possible** - resolve during rebase process
- **Can abort** anytime with `git rebase --abort`
- **Test after rebase** - ensure nothing broke

---

## Completion Checklist

- [ ] SSH into Storage Server
- [ ] Navigated to `/usr/src/kodekloudrepos/news`
- [ ] Verified repository status
- [ ] Checked available branches
- [ ] Updated master branch (`git pull`)
- [ ] Switched to feature branch
- [ ] Viewed branch divergence
- [ ] Ensured clean working directory
- [ ] Performed rebase (`git rebase master`)
- [ ] Resolved conflicts (if any)
- [ ] Verified linear history
- [ ] Checked no merge commits exist
- [ ] Force pushed feature branch
- [ ] Verified remote updated
- [ ] Confirmed clean linear history

---

## Completion Details

- **Completion Date:** December 7, 2025
- **Day:** 32 / 100
- **Challenge:** KodeKloud 100 Days Cloud DevOps
- **Topic:** Git Rebase - Maintaining Linear History
- **Server:** Storage Server (ststor01)
- **Repository:** `/opt/news.git` → `/usr/src/kodekloudrepos/news`
- **Branches:** master (base), feature (rebased)
- **Initial State:** Feature branch diverged from master
- **Action:** Rebase feature onto master
- **Result:** Linear history, no merge commits
- **Command Used:** `git rebase master`
- **Push:** `git push origin feature --force-with-lease`
- **Key Skill:** Branch rebasing and history management
- **Status:** ✅ Successfully Completed

---

## Summary

This task demonstrated **Git rebase for clean linear history**:

✅ **Updated master branch** with latest commits
✅ **Rebased feature branch** onto master
✅ **Maintained linear history** without merge commits
✅ **No data loss** - all feature commits preserved
✅ **Force pushed** rebased feature branch

**Key Insight:** Git rebase is essential for maintaining clean project history. It:
- Creates linear commit sequence
- Eliminates unnecessary merge commits
- Makes history easier to understand
- Simplifies debugging with `git bisect`
- Is standard practice for feature branch development

Unlike merge which preserves branch structure, rebase rewrites history to make it appear as if feature work happened after master's latest commit. This is perfect for feature branches but should never be used on shared branches.

**Remember:** `git rebase` = History Rewriter = Clean Linear Timeline! 📏

**The Golden Rule:** Never rebase commits that have been pushed to public/shared branches!

**The DevOps Lesson:** Just like keeping git history clean, in DevOps we maintain clean infrastructure, organized documentation, and streamlined processes. Clean foundations enable scalable growth! 🎯
