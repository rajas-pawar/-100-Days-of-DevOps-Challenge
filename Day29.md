# Day 29: Git Pull Requests & Code Review Workflow
**100 Days Cloud DevOps Challenge – KodeKloud**

---

## Objective

Max wants to push new changes to a repository, but direct pushes to the master branch should be prevented. The master branch should only contain reviewed and approved code. This task demonstrates the proper workflow using Pull Requests (PRs) and code reviews.

**Requirements:**
1. SSH into storage server as user `max` (password: `Max_pass123`)
2. Verify Max's story about "The Fox and Grapes" in branch `story/fox-and-grapes`
3. Confirm the repository contents and commit history
4. Create a Pull Request to merge `story/fox-and-grapes` into `master`
5. Assign `tom` as a reviewer through Gitea UI
6. Login as `tom` and review/approve the PR
7. Merge the approved PR into master branch

---

## Understanding Pull Requests (PRs)

**Pull Request** (also called Merge Request in GitLab) is a method of submitting contributions to a project. It's a request to merge code from one branch into another after review.

### Why Use Pull Requests?

- **Code Review:** Team reviews changes before merging
- **Quality Control:** Catch bugs and issues early
- **Knowledge Sharing:** Team learns from each other's code
- **Documentation:** PR serves as discussion record
- **Branch Protection:** Prevents direct pushes to important branches
- **Approval Workflow:** Ensures changes meet standards

### Pull Request Workflow:

```
1. Developer creates feature branch
2. Developer commits changes to feature branch
3. Developer pushes branch to remote
4. Developer creates Pull Request
5. Reviewer(s) assigned to PR
6. Code review and discussion
7. Changes approved
8. PR merged into target branch
```

---

## Infrastructure Overview

### Storage Server Users:
| User | Password | Role |
|------|----------|------|
| max | Max_pass123 | Developer (Author) |
| tom | Tom_pass123 | Reviewer (Approver) |

### Gitea UI Access:
| Component | Details |
|-----------|---------|
| Access | Via "Gitea UI" button on top bar |
| Max Login | Username: `max`, Password: `Max_pass123` |
| Tom Login | Username: `tom`, Password: `Tom_pass123` |

### Repository Details:
| Item | Value |
|------|-------|
| Location | Max's home directory (`/home/max`) |
| Story | The Fox and Grapes 🦊🍇 |
| Source Branch | `story/fox-and-grapes` |
| Target Branch | `master` |
| PR Title | "Added fox-and-grapes story" |
| Reviewer | tom |

---

## Understanding the Scenario

### Current Branch Structure:
```
master:              A --- B --- C (Sarah's story)
                                  ↑
                               master

story/fox-and-grapes: A --- B --- C --- D --- E (Max's story)
                                              ↑
                                    story/fox-and-grapes
```

### After PR Merge:
```
master:              A --- B --- C --- D --- E
                                              ↑
                                           master
                                    (includes Max's story)
```

---

## Step-by-Step Implementation

### Phase 1: Verify Repository as Max

#### Step 1: SSH into Storage Server
```bash
ssh max@ststor01
```

**Enter password:** `Max_pass123`

#### Step 2: Navigate to Home Directory
```bash
cd ~
# or
cd /home/max
```

#### Step 3: List Directories
```bash
ls -la
```

**Look for the cloned repository directory**

#### Step 4: Navigate to Repository
```bash
# Find the repository name
ls -la

# Navigate to it (example: story-blog)
cd story-blog
```

**Note:** Repository name may vary. Common names:
- `story-blog`
- `blog`
- `stories`

#### Step 5: Verify Git Repository
```bash
# Check if it's a git repository
git status

# View current branch
git branch
```

**Expected output:**
```
On branch story/fox-and-grapes
Your branch is up to date with 'origin/story/fox-and-grapes'.

nothing to commit, working tree clean
```

#### Step 6: List All Branches
```bash
git branch -a
```

**Expected output:**
```
  master
* story/fox-and-grapes
  remotes/origin/HEAD -> origin/master
  remotes/origin/master
  remotes/origin/story/fox-and-grapes
```

#### Step 7: Check Repository Contents
```bash
ls -la
```

**Expected:** Story files, README, etc.

```bash
# View story files
cat *.txt
# or
cat *.md
```

#### Step 8: View Commit History
```bash
git log
```

**Expected output:**
```
commit abc123... (HEAD -> story/fox-and-grapes, origin/story/fox-and-grapes)
Author: Max <max@example.com>
Date:   Mon Dec 2 10:00:00 2025

    Added fox-and-grapes story

commit def456...
Author: Sarah <sarah@example.com>
Date:   Sun Dec 1 15:00:00 2025

    Initial commit with Sarah's story
```

#### Step 9: View Commit History (Oneline)
```bash
git log --oneline --graph --all
```

**Expected output:**
```
* abc123 (HEAD -> story/fox-and-grapes, origin/story/fox-and-grapes) Added fox-and-grapes story
* def456 (origin/master, master) Sarah's story
* 789xyz Initial commit
```

#### Step 10: Verify Sarah's Story
```bash
# Switch to master to see Sarah's content
git checkout master

# View files
ls -la
cat *.txt  # or *.md
```

#### Step 11: Switch Back to Feature Branch
```bash
git checkout story/fox-and-grapes
```

#### Step 12: Verify Max's Story
```bash
# View Max's fox-and-grapes story
cat fox-and-grapes.txt
# or
cat *.txt | grep -i "fox\|grapes"
```

#### Step 13: Confirm Branch is Pushed
```bash
git branch -vv
```

**Expected output:**
```
  master                 def456 [origin/master] Sarah's story
* story/fox-and-grapes   abc123 [origin/story/fox-and-grapes] Added fox-and-grapes story
```

---

### Phase 2: Create Pull Request as Max

#### Step 14: Access Gitea UI
1. Click on **"Gitea UI"** button on the top bar
2. Browser opens with Gitea interface

#### Step 15: Login to Gitea as Max
**Login credentials:**
- **Username:** `max`
- **Password:** `Max_pass123`

**Steps:**
1. Click **"Sign In"** (if not already logged in)
2. Enter username: `max`
3. Enter password: `Max_pass123`
4. Click **"Sign In"** button

#### Step 16: Navigate to Repository
1. You should see the repository on your dashboard
2. Click on the repository name (e.g., `story-blog`)

#### Step 17: Create Pull Request
1. You may see a banner: **"story/fox-and-grapes had recent pushes"**
2. Click **"Compare & pull request"** (if available)

**OR manually:**

1. Click on **"Pull Requests"** tab
2. Click **"New Pull Request"** button

#### Step 18: Configure Pull Request
**Pull Request Settings:**

1. **Base branch (destination):** `master`
2. **Compare branch (source):** `story/fox-and-grapes`

**Verify the branches:**
```
base: master  ←  compare: story/fox-and-grapes
```

3. **Title:** `Added fox-and-grapes story`

4. **Description (optional but recommended):**
```
Added the classic Aesop's fable "The Fox and Grapes" 🦊🍇

This story teaches the lesson about sour grapes and rationalization.
```

#### Step 19: Review Changes
1. Scroll down to see **"Files Changed"** section
2. Review the changes being proposed
3. Verify files added/modified are correct

**Expected changes:**
- New file: `fox-and-grapes.txt` (or similar)
- Lines added (in green)

#### Step 20: Create Pull Request
1. Click **"Create Pull Request"** button
2. PR is now created and visible

**Expected result:**
```
Pull Request #1
Added fox-and-grapes story
max wants to merge 1 commit into master from story/fox-and-grapes
```

---

### Phase 3: Assign Reviewer

#### Step 21: Add Tom as Reviewer
**On the PR page:**

1. Look for **"Reviewers"** section on the right sidebar
2. Click on **"Reviewers"** (or gear icon ⚙️ next to it)
3. Search for **"tom"** in the dropdown
4. Click on **"tom"** to add him as a reviewer

**Expected result:**
```
Reviewers
👤 tom
```

#### Step 22: Verify Reviewer Assignment
**Confirm:**
- Tom appears in "Reviewers" section
- Tom will receive notification about review request

#### Step 23: Screenshot for Documentation
**Take screenshot showing:**
- PR title: "Added fox-and-grapes story"
- Source branch: `story/fox-and-grapes`
- Target branch: `master`
- Reviewer: tom
- PR status: Open

---

### Phase 4: Review and Approve as Tom

#### Step 24: Logout as Max
1. Click on **user avatar/profile** (top right)
2. Click **"Sign Out"**
3. Confirm logout

#### Step 25: Login as Tom
**Login credentials:**
- **Username:** `tom`
- **Password:** `Tom_pass123`

**Steps:**
1. Click **"Sign In"**
2. Enter username: `tom`
3. Enter password: `Tom_pass123`
4. Click **"Sign In"** button

#### Step 26: Navigate to Pull Request
**Method 1 - Dashboard:**
1. On dashboard, look for notification about PR review request
2. Click on the PR notification

**Method 2 - Direct:**
1. Click on the repository name
2. Click on **"Pull Requests"** tab
3. Click on PR #1: **"Added fox-and-grapes story"**

#### Step 27: Review Changes
1. On the PR page, review the **Description**
2. Click on **"Files Changed"** tab
3. Review the code/content changes

**Review checklist:**
- ✅ Story content is appropriate
- ✅ No syntax errors
- ✅ Formatting is correct
- ✅ File naming follows conventions
- ✅ Commit message is clear

#### Step 28: Add Review Comments (Optional)
**If you want to comment:**

1. Click on a line in the changed file
2. Add a comment (e.g., "Great story! 👍")
3. Click **"Add single comment"**

**OR add general comment:**
1. Click on **"Conversation"** tab
2. Scroll to comment box at bottom
3. Add comment (e.g., "LGTM! Nice addition to our collection.")

#### Step 29: Approve Pull Request
1. Click **"Files Changed"** tab
2. Click **"Review Changes"** button (top right)
3. Select **"Approve"** radio button
4. Add comment (optional): "Approved! Great story."
5. Click **"Submit Review"** button

**Expected result:**
```
✅ tom approved these changes
```

#### Step 30: Merge Pull Request
1. Go back to **"Conversation"** tab
2. Scroll down to the bottom
3. You should see **"Merge Pull Request"** button (now enabled)
4. Click **"Merge Pull Request"** button

**Merge options:**
- **Create a merge commit** (default, recommended)
- **Squash and merge** (combines commits)
- **Rebase and merge** (linear history)

5. Confirm merge message (optional):
```
Merge pull request #1 from story/fox-and-grapes

Added fox-and-grapes story

Reviewed by: tom
```

6. Click **"Confirm Merge"** button

**Expected result:**
```
✅ Pull request successfully merged and closed
```

#### Step 31: Verify Merge Success
**On the PR page:**
- Status changes to **"Merged"** (purple badge)
- Shows: "merged 1 commit into master from story/fox-and-grapes"
- Branch `story/fox-and-grapes` can be deleted (optional)

#### Step 32: Optional - Delete Source Branch
1. Click **"Delete branch"** button (if you want to clean up)
2. This removes `story/fox-and-grapes` branch from remote

**Note:** This is optional and depends on your workflow

---

### Phase 5: Verify Merge in Terminal

#### Step 33: Return to SSH Terminal as Max
```bash
# Switch back to your SSH session
# If disconnected, reconnect:
ssh max@ststor01
cd ~/story-blog  # or your repo name
```

#### Step 34: Update Local Repository
```bash
# Fetch latest changes
git fetch origin

# View branches
git branch -a
```

#### Step 35: Switch to Master Branch
```bash
git checkout master
```

**Expected output:**
```
Switched to branch 'master'
Your branch is behind 'origin/master' by 1 commit, and can be fast-forwarded.
  (use "git pull" to update your local branch)
```

#### Step 36: Pull Latest Master
```bash
git pull origin master
```

**Expected output:**
```
Updating def456..abc123
Fast-forward
 fox-and-grapes.txt | 15 +++++++++++++++
 1 file changed, 15 insertions(+)
 create mode 100644 fox-and-grapes.txt
```

#### Step 37: Verify Merge
```bash
# List files
ls -la

# View commit log
git log --oneline --graph
```

**Expected output:**
```
* abc123 (HEAD -> master, origin/master) Merge pull request #1 from story/fox-and-grapes
* def456 Added fox-and-grapes story
* 789xyz Sarah's story
```

#### Step 38: View Merged Story
```bash
cat fox-and-grapes.txt
```

**Expected:** Max's "The Fox and Grapes" story content

---

## Complete Command Summary

### Terminal Commands (as Max):
```bash
# SSH and navigate
ssh max@ststor01
cd ~/story-blog  # or your repo name

# Verify repository
git status
git branch -a

# View commit history
git log --oneline --graph --all

# Check Sarah's story
git checkout master
cat *.txt

# Check Max's story
git checkout story/fox-and-grapes
cat fox-and-grapes.txt

# After PR merged - update local repo
git checkout master
git pull origin master

# Verify merge
git log --oneline --graph
ls -la
cat fox-and-grapes.txt
```

### Gitea UI Workflow:

**As Max (Author):**
1. Login to Gitea UI
2. Navigate to repository
3. Create Pull Request
   - Base: `master`
   - Compare: `story/fox-and-grapes`
   - Title: "Added fox-and-grapes story"
4. Add reviewer: `tom`
5. Logout

**As Tom (Reviewer):**
1. Login to Gitea UI
2. Navigate to Pull Request
3. Review changes in "Files Changed"
4. Click "Review Changes"
5. Select "Approve"
6. Submit review
7. Click "Merge Pull Request"
8. Confirm merge

---

## Understanding Pull Request Components

### PR Status States:
| Status | Icon | Meaning |
|--------|------|---------|
| Open | 🟢 Green | Awaiting review/merge |
| Merged | 🟣 Purple | Successfully merged |
| Closed | 🔴 Red | Closed without merging |
| Draft | ⚫ Gray | Work in progress |

### PR Review Actions:
| Action | Symbol | Meaning |
|--------|--------|---------|
| Approve | ✅ | Changes look good |
| Request Changes | ❌ | Needs modifications |
| Comment | 💬 | General feedback |

### Merge Strategies:

**1. Create Merge Commit (Default)**
```
master:     A --- B --- C ------ M
                         \      /
feature:                  D --- E
```
- Preserves full history
- Shows branch structure
- Recommended for most cases

**2. Squash and Merge**
```
master:     A --- B --- C --- DE'
```
- Combines all commits into one
- Clean linear history
- Good for cleaning up messy branches

**3. Rebase and Merge**
```
master:     A --- B --- C --- D' --- E'
```
- Linear history, no merge commit
- Replays commits on top of target
- Good for maintaining clean history

---

## Pull Request Best Practices

### 1. Write Clear PR Titles
```
✅ Good: "Added fox-and-grapes story"
✅ Good: "Fix: Correct typo in README"
✅ Good: "Feature: Add user authentication"

❌ Bad: "Update"
❌ Bad: "Changes"
❌ Bad: "Fix stuff"
```

### 2. Provide Detailed Descriptions
```markdown
## What
Added the classic Aesop's fable "The Fox and Grapes"

## Why
Expanding our collection of moral stories

## Changes
- New file: fox-and-grapes.txt
- Added story content and moral

## Testing
- Verified formatting
- Checked for typos
```

### 3. Keep PRs Small and Focused
- ✅ One feature/fix per PR
- ✅ Easy to review (< 400 lines changed)
- ✅ Single purpose
- ❌ Avoid mixing multiple unrelated changes

### 4. Request Appropriate Reviewers
- Choose team members familiar with the code
- Request 1-2 reviewers (not too many)
- Tag specific people for specific sections

### 5. Respond to Review Comments
- Address all comments
- Ask for clarification if needed
- Mark resolved comments
- Update PR based on feedback

### 6. Test Before Creating PR
```bash
# Run tests locally
npm test
# or
pytest
# or
./run-tests.sh

# Ensure code compiles
# Check linting
# Verify functionality
```

### 7. Link Related Issues
```markdown
Fixes #123
Relates to #456
Part of epic #789
```

---

## Code Review Best Practices

### As a Reviewer:

**1. Review Promptly**
- Respond within 24 hours
- Don't leave PRs hanging
- Block time for reviews

**2. Be Constructive**
```
✅ Good: "Consider using a constant here for better maintainability"
❌ Bad: "This is wrong"

✅ Good: "Great approach! One suggestion: we could optimize this loop"
❌ Bad: "This looks terrible"
```

**3. Focus on Important Issues**
- **Blocking:** Security issues, bugs, breaking changes
- **Non-blocking:** Style preferences, minor optimizations
- Use labels: "nit:", "optional:", "blocking:"

**4. Approve When Ready**
```
✅ All functionality works
✅ Code follows conventions
✅ Tests are included
✅ Documentation updated
✅ No security vulnerabilities
```

### As an Author:

**1. Accept Feedback Graciously**
- Don't take comments personally
- Ask questions if unclear
- Thank reviewers

**2. Make Requested Changes**
- Address comments promptly
- Push updates to same branch
- Reply when changes made

**3. Explain Your Decisions**
- If disagreeing with suggestion, explain why
- Provide context for approach chosen
- Be open to alternatives

---

## Branch Protection Rules

### Why Protect Master Branch?

- **Prevent Direct Pushes:** Force code review
- **Require Approvals:** Ensure quality gate
- **Run CI/CD Checks:** Automated testing
- **Enforce Standards:** Maintain code quality
- **Audit Trail:** Track all changes

### Common Protection Rules:

| Rule | Purpose |
|------|---------|
| Require PR | No direct commits to master |
| Require Reviews | Min 1-2 approvals needed |
| Require Status Checks | CI must pass |
| No Force Push | Prevent history rewriting |
| Admin Bypass | Only admins can override |

### Setting Branch Protection (Gitea):
```
Repository Settings → Branches → Protected Branches
- Branch Name: master
- Enable Branch Protection
- Require Pull Request: Yes
- Required Approvals: 1
- Dismiss Stale Approvals: Yes
```

---

## Troubleshooting

### Issue 1: Can't Create Pull Request

**Problem:** "Create Pull Request" button not visible

**Solution:**
```bash
# Ensure branch is pushed to remote
git push origin story/fox-and-grapes

# Refresh Gitea UI
# Navigate to repository → Pull Requests → New Pull Request
```

### Issue 2: Can't Find Reviewer

**Problem:** Tom doesn't appear in reviewers list

**Solution:**
- Verify Tom has access to repository
- Check Tom is a collaborator
- Try typing full username
- Refresh page

### Issue 3: Merge Button Disabled

**Problem:** Can't click "Merge Pull Request"

**Solution:**
- Wait for required approvals
- Resolve merge conflicts if any
- Ensure CI checks pass (if configured)
- Verify you have merge permissions

### Issue 4: Merge Conflicts

**Problem:**
```
⚠️ This pull request has conflicts that must be resolved
```

**Solution:**
```bash
# On your local machine
git checkout story/fox-and-grapes
git pull origin master

# Resolve conflicts in files
vi fox-and-grapes.txt

# Add and commit resolved files
git add fox-and-grapes.txt
git commit -m "Resolved merge conflicts"

# Push updated branch
git push origin story/fox-and-grapes

# PR will automatically update
```

### Issue 5: Wrong Branch Selected

**Problem:** Created PR from wrong branch

**Solution:**
- Close incorrect PR
- Create new PR with correct branches
- OR edit PR (if option available)

### Issue 6: Lost Changes After Merge

**Problem:** Local repository doesn't have merged changes

**Solution:**
```bash
# Update your local repository
git checkout master
git pull origin master

# Verify changes
git log --oneline
ls -la
```

---

## Git Workflow Comparison

### Feature Branch Workflow (This Task):
```
1. Create feature branch
2. Commit changes to feature
3. Push feature branch
4. Create Pull Request
5. Code Review
6. Merge after approval
```

**Pros:**
- Isolated development
- Code review before merge
- Safe for production branches

### Gitflow Workflow:
```
master (production)
  ↓
develop (integration)
  ↓
feature branches
```

**Pros:**
- Structured for releases
- Multiple environments
- Clear branch purposes

### Trunk-Based Development:
```
Everyone commits to main/master frequently
Short-lived feature branches (< 1 day)
Feature flags for incomplete features
```

**Pros:**
- Continuous integration
- Fast feedback
- Simple structure

---

## Real-World Applications

### 1. Open Source Contributions
```
1. Fork repository
2. Create feature branch
3. Make changes
4. Push to your fork
5. Create PR to original repo
6. Wait for maintainer review
7. Address feedback
8. PR merged
```

### 2. Enterprise Development
```
1. Create feature branch from develop
2. Develop and test locally
3. Push and create PR
4. Request reviews from team leads
5. Run automated CI/CD tests
6. Get 2+ approvals
7. Merge to develop
8. Later merge to master for production
```

### 3. Hotfix Workflow
```
1. Create hotfix branch from master
2. Fix critical bug
3. Create emergency PR
4. Fast-track review
5. Merge to master (production)
6. Also merge to develop
```

### 4. Release Management
```
1. Create release branch from develop
2. Final testing and bug fixes
3. Create PR to master
4. Review and approve
5. Merge to master
6. Tag release (v1.0.0)
7. Merge back to develop
```

---

## Key Git Commands for PR Workflow

| Command | Purpose |
|---------|---------|
| `git checkout -b feature/new` | Create feature branch |
| `git add .` | Stage changes |
| `git commit -m "message"` | Commit changes |
| `git push origin feature/new` | Push feature branch |
| `git fetch origin` | Get remote updates |
| `git pull origin master` | Update local master |
| `git merge master` | Merge master into feature |
| `git log --graph --oneline` | Visual commit history |
| `git branch -d feature/new` | Delete local branch |
| `git push origin --delete feature/new` | Delete remote branch |

---

## Completion Checklist

**Part 1: Verify Repository**
- [ ] SSH into storage server as `max`
- [ ] Navigated to cloned repository
- [ ] Verified repository contents
- [ ] Ran `git log` to check commit history
- [ ] Confirmed Sarah's story exists
- [ ] Confirmed Max's story in `story/fox-and-grapes` branch
- [ ] Verified branch is pushed to remote

**Part 2: Create Pull Request**
- [ ] Accessed Gitea UI
- [ ] Logged in as `max`
- [ ] Navigated to repository
- [ ] Created new Pull Request
- [ ] Set base branch: `master`
- [ ] Set compare branch: `story/fox-and-grapes`
- [ ] Set title: "Added fox-and-grapes story"
- [ ] Reviewed changes in PR
- [ ] Added `tom` as reviewer
- [ ] Took screenshots for documentation

**Part 3: Review and Merge**
- [ ] Logged out as `max`
- [ ] Logged in as `tom`
- [ ] Navigated to Pull Request
- [ ] Reviewed files changed
- [ ] Added review comments (optional)
- [ ] Approved the Pull Request
- [ ] Merged Pull Request into master
- [ ] Verified merge success message
- [ ] Took screenshots of merged PR

**Part 4: Verify Merge**
- [ ] Returned to SSH terminal
- [ ] Checked out master branch
- [ ] Pulled latest changes
- [ ] Verified Max's story now in master
- [ ] Confirmed commit history includes merge

---

## Completion Details

- **Completion Date:** December 4, 2025
- **Day:** 29 / 100
- **Challenge:** KodeKloud 100 Days Cloud DevOps
- **Topic:** Git Pull Requests & Code Review Workflow
- **Users:** max (author), tom (reviewer)
- **Repository:** story-blog (in Max's home directory)
- **Story:** The Fox and Grapes 🦊🍇
- **Source Branch:** `story/fox-and-grapes`
- **Target Branch:** `master`
- **PR Title:** "Added fox-and-grapes story"
- **Review Process:** Tom reviewed and approved
- **Merge Status:** ✅ Successfully merged
- **Key Skills:** Pull Requests, Code Review, Branch Protection, Collaboration
- **Status:** ✅ Successfully Completed

---

## Summary

This task demonstrated **professional Git collaboration workflow** using Pull Requests:

✅ **Verified repository** and commit history as Max
✅ **Created Pull Request** from feature branch to master
✅ **Assigned reviewer** (tom) through Gitea UI
✅ **Performed code review** as tom
✅ **Approved and merged** PR into master branch
✅ **Verified merge** in local repository

**Key Insight:** Pull Requests are the cornerstone of collaborative development. They:
- Enable code review before merging
- Protect important branches (master/main)
- Provide discussion platform for changes
- Create audit trail of who approved what
- Ensure code quality through peer review
- Facilitate knowledge sharing across team

Unlike direct commits, PRs enforce a review process that catches bugs, improves code quality, and spreads knowledge across the team. This is the industry standard for professional software development.

**Remember:** `Pull Request` = Collaboration + Quality + Documentation! 🎯

**The Story:** The Fox and Grapes teaches us that it's easy to despise what you cannot have. In DevOps, we learn that proper process (even if it seems slower) is better than rushing changes that might break production! 🦊🍇