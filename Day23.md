# Day 23: Forking Git Repository in Gitea
**100 Days Cloud DevOps Challenge – KodeKloud**

---

## Objective

A new developer named Jon has joined the Nautilus project teams and needs to begin working on a project. To start, he must fork an existing Git repository on the Gitea server.

**Requirements:**
- Access Gitea UI through the provided button
- Login to Gitea server using Jon's credentials
- Locate the Git repository `sarah/story-blog`
- Fork the repository under the `jon` user account
- Take screenshots for documentation

---

## What is Forking?

**Forking** is creating a personal copy of someone else's repository. This allows you to:
- Freely experiment with changes without affecting the original project
- Make modifications independently
- Propose changes back to the original repository (via Pull Requests)
- Start your own version of a project

### Fork vs Clone

| Feature | Fork | Clone |
|---------|------|-------|
| Location | Creates copy on server | Downloads to local machine |
| Visibility | Public/visible to others | Local only |
| Connection | Maintains link to original | Independent copy |
| Use Case | Contributing to projects | Local development |
| Where | Git hosting platform (GitHub/Gitea) | Your computer |

---

## Infrastructure Overview

### Gitea Server Details:
| Component | Details |
|-----------|---------|
| Platform | Gitea UI (Git hosting platform) |
| Access | Via "Gitea UI" button on top bar |
| User | jon |
| Password | Jon_pass123 |

### Repository Details:
| Item | Value |
|------|-------|
| Original Repository | `sarah/story-blog` |
| Owner | sarah |
| Repository Name | story-blog |
| Fork Destination | jon's account |

---

## Step-by-Step Implementation

### Step 1: Access Gitea UI

1. **Click on the "Gitea UI" button** located on the top bar of your KodeKloud interface

**Screenshot Required:** Take a screenshot showing the Gitea UI button

### Step 2: Login to Gitea

1. You will be redirected to the Gitea login page
2. **Enter credentials:**
   - **Username:** `jon`
   - **Password:** `Jon_pass123`
3. Click the **"Sign In"** button

**Screenshots Required:**
- The login page (before entering credentials)
- After successful login (dashboard view)

### Step 3: Locate the Repository

Navigate to the repository using one of these methods:

**Method 1: Direct Search**
1. Use the search bar at the top
2. Type: `story-blog`
3. Look for the repository owned by `sarah`
4. Click on `sarah/story-blog`

**Method 2: Explore Repositories**
1. Click on "Explore" in the top navigation
2. Click "Repositories"
3. Search for `sarah/story-blog`
4. Click on the repository

**Method 3: Browse Users**
1. Click "Explore" → "Users"
2. Find and click on user `sarah`
3. Navigate to the `story-blog` repository
4. Click on the repository name

**Screenshots Required:**
- Repository search/explore page
- The `sarah/story-blog` repository page

### Step 4: Fork the Repository

1. On the `sarah/story-blog` repository page, locate the **"Fork"** button (usually in the top-right area)
2. Click the **"Fork"** button
3. You'll see fork configuration options:
   - **Owner:** Should show `jon` (your account)
   - **Repository Name:** Will default to `story-blog`
   - **Description:** (Optional) Keep the same or modify
4. Click **"Fork Repository"** to confirm

**Screenshots Required:**
- The original repository page showing the Fork button
- The fork configuration dialog
- The successful fork completion

### Step 5: Verify the Fork

1. After forking, you'll be redirected to your forked repository
2. **Verify:**
   - URL shows: `jon/story-blog` (not `sarah/story-blog`)
   - You see text like: "Forked from sarah/story-blog"
   - Repository is under your account

**Screenshot Required:**
- Your forked repository (`jon/story-blog`) showing the "forked from" indicator

---

## Complete Step Summary
```
1. Click "Gitea UI" button
   ↓
2. Login as jon/Jon_pass123
   ↓
3. Search/Find sarah/story-blog
   ↓
4. Click on repository
   ↓
5. Click "Fork" button
   ↓
6. Confirm fork (owner: jon)
   ↓
7. Verify jon/story-blog exists
```

---

## Visual Guide

### Expected UI Elements:

**Login Page:**
- Username field
- Password field
- Sign In button

**Dashboard:**
- Navigation bar
- Your repositories section
- Activity feed

**Repository Page (sarah/story-blog):**
- Repository name and owner
- Fork button (top-right)
- Star and Watch buttons
- Files and commits tabs

**Fork Dialog:**
- Owner dropdown (select jon)
- Repository name field
- Description field (optional)
- Fork Repository button

**Forked Repository (jon/story-blog):**
- Your username in URL
- "Forked from sarah/story-blog" text
- Full repository access

---

## Verification Steps

### Verify 1: Check Your Repositories

1. Click on your profile icon/username
2. Go to "Your Repositories"
3. **Confirm** `story-blog` appears in your list

### Verify 2: Check Fork Relationship

On your `jon/story-blog` page:
- Look for "forked from sarah/story-blog" indicator
- This confirms the fork relationship

### Verify 3: Test Repository Access

- Browse files in your forked repository
- You should see all files from original
- Check if you have edit permissions

### Verify 4: Compare with Original

- Open both repositories in separate tabs
- Original: `sarah/story-blog`
- Your fork: `jon/story-blog`
- Content should be identical

---

## Screenshot Checklist

### Required Screenshots:

- [ ] **Screenshot 1:** Gitea UI button on top bar
- [ ] **Screenshot 2:** Login page
- [ ] **Screenshot 3:** Dashboard after login
- [ ] **Screenshot 4:** Finding sarah/story-blog
- [ ] **Screenshot 5:** sarah/story-blog repository page with Fork button
- [ ] **Screenshot 6:** Fork configuration dialog
- [ ] **Screenshot 7:** jon/story-blog forked repository
- [ ] **Screenshot 8:** "Forked from sarah/story-blog" indicator
- [ ] **Screenshot 9:** Your repositories list showing story-blog

### Screenshot Tips:

- Use full-screen captures
- Ensure URLs are visible
- Highlight important buttons/text
- Use clear, descriptive filenames
- Organize in sequential order

---

## Troubleshooting

### Issue 1: Cannot Find Repository

**Problem:** Cannot locate `sarah/story-blog`

**Solutions:**
```
1. Check spelling: sarah/story-blog (lowercase)
2. Use Explore → Repositories
3. Search for "story-blog" in search box
4. Verify repository is public
5. Check if logged in correctly
```

### Issue 2: Login Failed

**Problem:** Cannot login with credentials

**Solutions:**
```
1. Verify username: jon (lowercase, no spaces)
2. Verify password: Jon_pass123 (case-sensitive)
3. Clear browser cache/cookies
4. Try incognito/private browsing
5. Refresh the page and try again
```

### Issue 3: Fork Button Not Visible

**Problem:** Cannot see Fork button

**Solutions:**
```
1. Ensure you're on repository page (not profile page)
2. Check if already viewing your own fork
3. Verify you're logged in as jon
4. Scroll up - button might be above fold
5. Try different browser
```

### Issue 4: Fork Already Exists

**Problem:** Error: "Fork already exists"

**Solutions:**
```
1. Check your repositories - may already be forked
2. Go to jon/story-blog if it exists
3. Delete existing fork and re-fork
4. Or rename existing fork
```

### Issue 5: Permission Denied

**Problem:** Cannot fork repository

**Solutions:**
```
1. Verify repository is public
2. Check login status
3. Confirm logged in as jon (not sarah)
4. Repository settings may restrict forking
```

---

## Understanding Fork Workflow

### What Happens When You Fork:

1. **Server Creates Copy:**
   - Complete repository duplicated
   - All branches copied
   - Full commit history preserved
   - All tags included

2. **You Become Owner:**
   - Full administrative access
   - Can modify freely
   - Changes don't affect original

3. **Link Maintained:**
   - Fork relationship tracked
   - Can sync with original
   - Can propose changes via Pull Requests

### Repository Structure:
```
sarah/story-blog (Original/Upstream)
        ↓
     [FORK]
        ↓
jon/story-blog (Your Fork/Origin)
        ↓
     [CLONE]
        ↓
Local Machine (Optional - for development)
```

---

## After Forking: Next Steps

### Option 1: Work via Web UI

1. Navigate to `jon/story-blog`
2. Click on file to edit
3. Make changes directly in browser
4. Commit changes

### Option 2: Clone Locally
```bash
# Clone your fork
git clone http://gitea-url/jon/story-blog.git
cd story-blog

# Add upstream remote
git remote add upstream http://gitea-url/sarah/story-blog.git

# Verify remotes
git remote -v
```

### Option 3: Create Pull Request
```bash
# After making changes in your fork
1. Go to jon/story-blog on Gitea UI
2. Click "New Pull Request"
3. Select source: jon/story-blog
4. Select target: sarah/story-blog
5. Add description
6. Submit Pull Request
```

---

## Key Concepts

### Repository Ownership:

**Original (sarah/story-blog):**
- Owner: sarah
- Your access: Read only
- Cannot push directly

**Fork (jon/story-blog):**
- Owner: jon
- Your access: Full control
- Can push, modify, delete

### Remote Names:

- **origin:** Your fork (jon/story-blog)
- **upstream:** Original repository (sarah/story-blog)

### Collaboration Flow:
```
1. Fork (Web UI) - Create your copy
2. Clone (CLI) - Download locally
3. Branch - Create feature branch
4. Commit - Make changes
5. Push - Upload to your fork
6. Pull Request - Propose to original
```

---

## Screen Recording (Optional)

### Using Loom.com:

**Setup:**
1. Visit [loom.com](https://www.loom.com)
2. Sign up for free account
3. Install browser extension

**Record:**
1. Click Loom extension icon
2. Select "Screen + Camera" or "Screen Only"
3. Choose "Full Screen" or "Browser Tab"
4. Click "Start Recording"

**Capture Process:**
1. Show Gitea UI access
2. Demonstrate login
3. Navigate to repository
4. Show fork process
5. Verify forked repository

**Share:**
1. Click "Stop Recording"
2. Wait for processing
3. Copy share link
4. Submit with task

### Alternative Tools:

- **OBS Studio** (Free, open-source, all platforms)
- **ShareX** (Windows, free)
- **Kazam** (Linux, free)
- **QuickTime** (Mac, built-in)
- **Windows Game Bar** (Windows 10+, Win+G)

---

## Best Practices

### 1. Keep Fork Updated
```bash
# Fetch upstream changes
git fetch upstream

# Merge into your main branch
git checkout main
git merge upstream/main

# Push to your fork
git push origin main
```

### 2. Use Feature Branches
```bash
# Don't work directly on main
git checkout -b feature/new-post
# Make changes
git commit -m "Add new blog post"
git push origin feature/new-post
```

### 3. Clear Communication

- Use descriptive branch names
- Write clear commit messages
- Document changes in Pull Requests

### 4. Regular Syncing
```bash
# Regularly sync with upstream
git fetch upstream
git rebase upstream/main
```

---

## Common Git Commands After Fork

### View Remotes:
```bash
git remote -v
```

**Expected output:**
```
origin    http://gitea/jon/story-blog.git (fetch)
origin    http://gitea/jon/story-blog.git (push)
upstream  http://gitea/sarah/story-blog.git (fetch)
upstream  http://gitea/sarah/story-blog.git (push)
```

### Sync with Upstream:
```bash
git fetch upstream
git checkout main
git merge upstream/main
```

### Create Feature Branch:
```bash
git checkout -b my-feature
```

### Push Changes:
```bash
git push origin my-feature
```

---

## Key Takeaways

- **Forking is web-based** - done through UI, not CLI
- **Creates independent copy** on the server
- **Maintains link** to original repository
- **Full control** over your fork
- **Screenshots required** for task verification
- **Fork enables collaboration** without direct repository access
- **Standard workflow** in open-source development
- **One-click process** in modern Git platforms

---

## Real-World Applications

### 1. Open Source Contribution
- Fork project
- Make improvements
- Submit Pull Request
- Contribute to community

### 2. Learning & Experimentation
- Fork interesting projects
- Study the code
- Experiment safely
- Learn best practices

### 3. Starting Your Project
- Fork template/starter
- Customize for your needs
- Build your application
- Deploy independently

### 4. Team Collaboration
- Fork team repository
- Develop features independently
- Submit for review
- Merge after approval

### 5. Bug Fixes
- Fork project
- Fix the bug
- Test thoroughly
- Submit fix via PR

---

## Gitea Features Overview

### Repository Management:
- Create repositories
- Fork repositories
- Star/Watch repositories
- Manage collaborators

### Code Collaboration:
- Pull Requests
- Code review
- Issue tracking
- Project boards

### Version Control:
- Branch management
- Tag releases
- Compare branches
- Merge strategies

### Access Control:
- User permissions
- Organization management
- Team access
- Repository visibility

---

## Comparison: Gitea vs GitHub vs GitLab

| Feature | Gitea | GitHub | GitLab |
|---------|-------|--------|--------|
| Hosting | Self-hosted | Cloud-based | Both options |
| Fork Feature | ✅ Yes | ✅ Yes | ✅ Yes |
| Pull Requests | ✅ Yes | ✅ Yes | ✅ Merge Requests |
| Issue Tracker | ✅ Yes | ✅ Yes | ✅ Yes |
| CI/CD | ✅ Actions | ✅ Actions | ✅ Built-in |
| Cost | Free | Free + Paid | Free + Paid |
| Resource Usage | Lightweight | N/A | Heavier |
| UI Complexity | Simple | Feature-rich | Feature-rich |

**Note:** Fork workflow is similar across all platforms!

---

## Task Validation

### What KodeKloud Checks:

1. ✅ Repository `jon/story-blog` exists
2. ✅ Fork relationship to `sarah/story-blog` established
3. ✅ Repository accessible under jon's account
4. ✅ All content copied correctly
5. ✅ Fork indicator visible on repository page

### Manual Verification:

1. Can you access `jon/story-blog`?
2. Does it show "forked from sarah/story-blog"?
3. Can you see all files from original?
4. Do you have admin access to your fork?
5. Can you edit files in your fork?

---

## Additional Tips

### Browser Tips:
- Use Chrome or Firefox for best compatibility
- Clear cache if UI doesn't load properly
- Zoom to 100% for screenshots
- Use full-screen mode for cleaner captures

### Screenshot Organization:
```
screenshots/
├── 01-gitea-ui-button.png
├── 02-login-page.png
├── 03-dashboard.png
├── 04-find-repository.png
├── 05-repository-page.png
├── 06-fork-button.png
├── 07-fork-dialog.png
├── 08-forked-repository.png
└── 09-verification.png
```

### Documentation:
- Number screenshots sequentially
- Add brief captions
- Highlight key UI elements
- Show complete URL in screenshots
- Include timestamp if possible

---

## Completion Checklist

- [x] Accessed Gitea UI via top bar button
- [x] Successfully logged in as `jon`
- [x] Located repository `sarah/story-blog`
- [x] Clicked Fork button
- [x] Confirmed fork configuration (owner: jon)
- [x] Successfully created fork
- [x] Verified `jon/story-blog` exists
- [x] Confirmed "forked from" indicator visible
- [x] Repository appears in jon's repository list
- [x] Took all required screenshots
- [x] Organized screenshots sequentially
- [x] Optional: Created screen recording

---

## Completion Details

- **Completion Date:** November 28, 2025
- **Day:** 23 / 100
- **Challenge:** KodeKloud 100 Days Cloud DevOps
- **Topic:** Forking Git Repository in Gitea Web UI
- **Platform:** Gitea
- **User Account:** jon
- **Original Repository:** sarah/story-blog
- **Forked Repository:** jon/story-blog
- **Task Type:** Web UI Operation
- **Documentation:** Screenshots required
- **Key Skill:** Git collaboration workflow
- **Status:** ✅ Successfully Completed

---

## Summary

This task introduced the fundamental concept of **forking repositories** in Git hosting platforms. You learned:

✅ How to access and navigate Gitea UI
✅ The difference between fork and clone
✅ How to create a fork through web interface
✅ Verifying fork relationships
✅ Taking proper documentation screenshots

Forking is a cornerstone of open-source collaboration and team-based development. What would require complex Git commands is simplified to a single click in modern platforms like Gitea, GitHub, and GitLab.

**Key Insight:** Fork creates your personal copy on the server, giving you full control while maintaining a link to the original project. This enables safe experimentation and collaborative development!

**Remember:** Fork → Modify → Pull Request = The Open Source Way! 🚀