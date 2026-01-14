# Day 70: Jenkins User Management and Matrix Authorization Strategy

## 📋 Task Overview

**Scenario:** The Nautilus team is integrating Jenkins into their CI/CD pipelines. After setting up a new Jenkins server, they're now configuring user access for the development team. You need to create a new user, configure authorization strategy, and set up granular permissions.

**Given Requirements:**
- **Jenkins UI Access:** Click Jenkins button on top bar
- **Login Credentials:**
  - Username: `admin`
  - Password: `Adm!n321`
- **New User to Create:**
  - Username: `siva`
  - Password: `Rc5C9EyvbU`
  - Full Name: `Siva`
- **Authorization Configuration:**
  - Use Project-based Matrix Authorization Strategy
  - Grant `siva` user overall read permission
  - Remove all permissions for Anonymous users
  - Ensure `admin` retains overall Administer permissions
  - Grant `siva` read permission on existing job only

**Your Mission:**
1. Log in to Jenkins
2. Install Matrix Authorization Strategy Plugin (if needed)
3. Create user `siva`
4. Configure Matrix Authorization Strategy
5. Set global permissions
6. Configure job-specific permissions
7. Verify access control

---

## 🎯 Learning Objectives

By the end of this task, you will understand:
- **Jenkins Security Realms:** User authentication methods
- **Authorization Strategies:** Different permission models
- **Matrix-based Authorization:** Granular permission control
- **User Management:** Creating and managing Jenkins users
- **Global vs Project Permissions:** Permission inheritance
- **Security Best Practices:** Least privilege principle

---

## 📖 Understanding Jenkins Security

### Jenkins Security Architecture

```
┌─────────────────────────────────────────────┐
│         Jenkins Security Layers              │
├─────────────────────────────────────────────┤
│                                             │
│  1. Security Realm (Authentication)         │
│     - Who can log in?                       │
│     - Jenkins' own user database            │
│     - LDAP, Active Directory, OAuth         │
│                                             │
│  2. Authorization Strategy (Permissions)    │
│     - What can users do?                    │
│     - Global permissions                    │
│     - Project-level permissions             │
│                                             │
│  3. Credential Management                   │
│     - Storing secrets securely              │
│     - API tokens, SSH keys, passwords       │
│                                             │
└─────────────────────────────────────────────┘
```

---

### Security Realm vs Authorization Strategy

**Security Realm (Authentication):**
- Manages user identities
- Handles login process
- Validates credentials

**Common Security Realms:**
```
1. Jenkins' own user database (default)
   - Users stored in Jenkins
   - Simple username/password

2. LDAP
   - Corporate directory integration
   - Centralized user management

3. Active Directory
   - Microsoft AD integration
   - Windows authentication

4. OAuth / SAML
   - Single Sign-On (SSO)
   - Google, GitHub, Okta
```

**Authorization Strategy (Permissions):**
- Controls what authenticated users can do
- Manages access to resources
- Enforces security policies

---

### Authorization Strategies in Jenkins

**1. Anyone can do anything**
```
⚠️ NOT RECOMMENDED - No security
All users (including anonymous) have full access
```

**2. Logged-in users can do anything**
```
⚠️ Limited security
Any authenticated user has full access
Good for: Trusted internal teams only
```

**3. Legacy mode**
```
⚠️ Deprecated
Admin has full access, others are read-only
```

**4. Matrix-based security**
```
✅ RECOMMENDED
Granular permission control
Global-level permissions only
```

**5. Project-based Matrix Authorization Strategy**
```
✅ BEST FOR MOST USE CASES
Global + project-level permissions
Fine-grained access control
Per-job permission configuration
```

---

### Matrix Authorization Permissions

**Permission Categories:**

**Overall (Global):**
- Administer - Full Jenkins administration
- Read - View Jenkins dashboard
- RunScripts - Execute Groovy scripts
- UploadPlugins - Install plugins
- ConfigureUpdateCenter - Manage update sites

**Credentials:**
- Create - Create credentials
- Delete - Delete credentials
- ManageDomains - Manage credential domains
- Update - Update credentials
- View - View credentials

**Agent:**
- Build - Run builds on agents
- Configure - Configure agents
- Connect - Connect agents
- Create - Create new agents
- Delete - Delete agents
- Disconnect - Disconnect agents

**Job:**
- Build - Trigger builds
- Cancel - Cancel builds
- Configure - Modify job configuration
- Create - Create new jobs
- Delete - Delete jobs
- Discover - See job exists (without read)
- Move - Move/rename jobs
- Read - View job configuration
- Workspace - Access job workspace

**Run:**
- Delete - Delete build records
- Replay - Replay pipeline builds
- Update - Modify build information

**View:**
- Configure - Configure views
- Create - Create views
- Delete - Delete views
- Read - View views

**SCM:**
- Tag - Create SCM tags

---

## 📖 Matrix Authorization Strategy Plugin

### What is it?

**Matrix Authorization Strategy Plugin** provides fine-grained permission control:
- Global permissions for all jobs
- Per-project permissions for specific jobs
- User and group-based permissions
- Permission inheritance

### Installation Check

**The plugin may already be installed if:**
- You selected "Install suggested plugins" during setup
- It was installed as a dependency

**If not installed, you'll need to install it first.**

---

## 🛠️ Task Implementation

### Step 1: Access Jenkins and Log In

**Click the "Jenkins" button on the top bar.**

**Or navigate to:**
```
http://<jenkins-server-ip>:8080
```

**Log in with admin credentials:**
- **Username:** `admin`
- **Password:** `Adm!n321`

---

### Step 2: Check if Matrix Authorization Plugin is Installed

**Method 1: Via Configure Global Security (Quick Check)**

1. Go to **Manage Jenkins** → **Configure Global Security** (or **Security**)
2. Scroll to **Authorization** section
3. Look for **"Project-based Matrix Authorization Strategy"** option

**If you see it:** Plugin is already installed ✅

**If you don't see it:** Proceed to Step 3 to install it

---

**Method 2: Via Plugin Manager**

1. Go to **Manage Jenkins** → **Plugins** → **Installed plugins**
2. Search for: `Matrix Authorization`
3. Look for: **Matrix Authorization Strategy Plugin**

**If found:** Already installed ✅

**If not found:** Install it in Step 3

---

### Step 3: Install Matrix Authorization Strategy Plugin (If Needed)

**If the plugin is not installed:**

1. Go to **Manage Jenkins** → **Plugins**
2. Click **Available plugins** tab
3. In the search/filter box, type: `Matrix Authorization`
4. Find **"Matrix Authorization Strategy Plugin"**
5. Check the checkbox next to it
6. Click **"Install"** (or **"Install without restart"**)

**Monitor installation progress:**
```
┌──────────────────────────────────────────────┐
│     Installing Plugins/Upgrades              │
├──────────────────────────────────────────────┤
│                                              │
│  ✓ Matrix Authorization Strategy Plugin      │
│    Success                                   │
│                                              │
│  [ ] Restart Jenkins when installation is   │
│      complete and no jobs are running        │
│                                              │
└──────────────────────────────────────────────┘
```

**If restart is required:**
- Check the box: **"Restart Jenkins when installation is complete and no jobs are running"**
- Wait for Jenkins to restart (30-60 seconds)
- Login page will reappear
- Log back in with `admin` / `Adm!n321`

---

### Step 4: Create User `siva`

**Navigate to User Management:**

1. From Jenkins Dashboard, click **Manage Jenkins**
2. Click **Manage Users** (or **Users** in newer versions)

**You'll see the user list:**
```
┌──────────────────────────────────────────────┐
│         People                                │
├──────────────────────────────────────────────┤
│                                              │
│  [Create User]                               │
│                                              │
│  User ID        | Name          | Buttons   │
│  ---------------|---------------|------------ │
│  admin          | admin         | [⚙] [🗑] │
│  theadmin       | Siva          | [⚙] [🗑] │
│                                              │
└──────────────────────────────────────────────┘
```

**Note:** You may see a user `theadmin` with full name "Siva" from Day 68. This is a different user. You're creating a NEW user with username `siva`.

---

**Click "Create User" button.**

**Fill in the user creation form:**
```
┌──────────────────────────────────────────────┐
│         Create User                           │
├──────────────────────────────────────────────┤
│                                              │
│  Username:     [siva___________________]    │
│                                              │
│  Password:     [Rc5C9EyvbU_____________]    │
│                                              │
│  Confirm password: [Rc5C9EyvbU_________]    │
│                                              │
│  Full name:    [Siva___________________]    │
│                                              │
│  E-mail address: [____________________]     │
│                 (optional)                   │
│                                              │
│              [Create User]                   │
│                                              │
└──────────────────────────────────────────────┘
```

**Enter the following:**
- **Username:** `siva`
- **Password:** `Rc5C9EyvbU`
- **Confirm password:** `Rc5C9EyvbU`
- **Full name:** `Siva`
- **E-mail address:** (leave empty or enter `siva@jenkins.com`)

**Click "Create User"**

**Expected result:**
- User `siva` appears in the user list
- You now have multiple users: admin, theadmin (from Day 68), and siva (newly created)
- You're redirected back to the People page

---

### Step 5: Enable Matrix-Based Security

**Navigate to Security Configuration:**

1. Go to **Manage Jenkins**
2. Click **Configure Global Security** (or **Security**)

**You'll see the Security Configuration page.**

---

**In the "Authorization" section:**

**Current setting might be:**
- "Logged-in users can do anything" (default after setup)
- Or another strategy

**Change to Matrix-Based Security:**

1. Select the radio button: **"Project-based Matrix Authorization Strategy"**

**Important:** Choose **"Project-based"** not just "Matrix-based security"
- Matrix-based security = Global permissions only
- Project-based Matrix Authorization = Global + per-job permissions ✅

---

### Step 6: Configure Global Permissions

**After selecting "Project-based Matrix Authorization Strategy", you'll see a permission matrix:**

```
┌──────────────────────────────────────────────────────────────────┐
│  User/group to add: [_______________] [Add]                      │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│           Overall | Credentials | Agent | Job | Run | View | ... │
│  User/Group  A R U | C D M U V | B C C C... | B Ca Co Cr... |    │
│  ----------------------------------------------------------------│
│  admin       ✓ ✓ ✓ | ✓ ✓ ✓ ✓ ✓ | ✓ ✓ ✓ ✓... | ✓ ✓  ✓  ✓ ... |    │
│  Anonymous                                                        │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘

Legend:
A = Administer, R = Read, U = Upload Plugins, etc.
```

---

**Step 6a: Ensure `admin` has Administer permissions**

**The `admin` user should already have all permissions.**

**Verify:**
- Find the row with `admin` user
- Check that **"Overall → Administer"** (first checkbox) is checked ✓
- This gives admin full control

**If not checked, click to enable it.**

---

**Step 6b: Add `siva` user to the matrix**

1. **In the "User/group to add" field, type:** `siva`
2. **Click "Add"** (or press Enter)

**The `siva` user row will appear in the matrix.**

**Initially, `siva` will have NO permissions (all unchecked).**

---

**Step 6c: Grant `siva` Overall Read permission**

1. **Find the `siva` row in the matrix**
2. **In the "Overall" section, find the "Read" column**
   - Usually the second column in "Overall"
   - Hover over checkboxes to see tooltips
3. **Check the "Overall → Read" checkbox for `siva` ✓**

**This allows `siva` to:**
- View the Jenkins dashboard
- See the list of jobs
- Access Jenkins UI

**Do NOT check other Overall permissions for siva:**
- ❌ Overall → Administer
- ❌ Overall → RunScripts
- ❌ Overall → UploadPlugins
- etc.

**Only check: Overall → Read ✓**

---

**Step 6d: Remove Anonymous permissions**

**If you see an "Anonymous" row in the matrix:**

**Method 1: Remove all checkboxes**
1. Find the "Anonymous" row
2. Uncheck ALL permissions for Anonymous
3. Every checkbox should be empty

**Method 2: Remove Anonymous entirely (if available)**
1. Look for a [Delete] or [✖] button next to "Anonymous"
2. Click it to remove the entire row

**Result:** Anonymous users will have NO access to Jenkins

---

**Step 6e: Verify the permission matrix**

**Your matrix should now look like:**

```
┌──────────────────────────────────────────────┐
│  Authorization                                │
│  ○ Logged-in users can do anything           │
│  ● Project-based Matrix Authorization Strategy│
├──────────────────────────────────────────────┤
│                                              │
│  User/group to add: [___________] [Add]     │
│                                              │
│           | Overall  | Credentials | Agent...│
│  User     | Adm Read | Cre Del ... | Bui...  │
│  ---------+----------+-------------+--------- │
│  admin    |  ✓   ✓   |  ✓   ✓  ... |  ✓  ... │
│  javed    |      ✓   |             |         │
│                                              │
│  (No Anonymous row or all unchecked)         │
│                                              │
└──────────────────────────────────────────────┘
```

**Summary:**
- ✅ admin: Overall → Administer (and all other permissions)
- ✅ siva: Overall → Read ONLY
- ✅ Anonymous: No permissions (removed or all unchecked)

---

**Click "Save" at the bottom of the page.**

**⚠️ Important:** After clicking Save, you might be logged out if you removed your own permissions. Since we kept admin's Administer permission, you should remain logged in.

---

### Step 7: Configure Job-Specific Permissions for `siva`

**Now we need to grant `siva` read permission on the existing job.**

**Find the existing job:**

1. **Go to Jenkins Dashboard**
2. **Look for existing jobs in the list**
   - You might see a job from Day 69 (like `test-git-plugin`)
   - Or another job created during setup

**For this example, let's assume the job is named:** `test-git-plugin`

---

**Configure job permissions:**

1. **Click on the job name** (e.g., `test-git-plugin`)
2. **Click "Configure"** in the left sidebar
3. **Scroll down to find the "Enable project-based security" section**

**You should see:**
```
┌──────────────────────────────────────────────┐
│  [✓] Enable project-based security           │
├──────────────────────────────────────────────┤
│                                              │
│  User/group to add: [___________] [Add]     │
│                                              │
│           | Job                              │
│  User     | Build Cancel Configure Create... │
│  ---------+---------------------------------- │
│  admin    |  ✓     ✓       ✓        ✓  ...  │
│                                              │
└──────────────────────────────────────────────┘
```

---

**If "Enable project-based security" is NOT visible:**
- You're likely not using "Project-based Matrix Authorization Strategy"
- Go back to Step 5 and ensure you selected the correct authorization strategy
- Make sure it says "Project-based" not just "Matrix-based"

---

**If "Enable project-based security" checkbox is NOT checked:**
1. **Check the box: ✓ Enable project-based security**
2. The permission matrix will appear

---

**Add `siva` to the job permissions:**

1. **In the "User/group to add" field, type:** `siva`
2. **Click "Add"**
3. **The `siva` row appears in the job permission matrix**

---

**Grant `siva` Read permission on the job:**

1. **Find the `siva` row**
2. **In the "Job" section, find the "Read" column**
3. **Check "Job → Read" for `siva` ✓**

**Do NOT check other Job permissions:**
- ❌ Job → Build
- ❌ Job → Cancel
- ❌ Job → Configure
- ❌ Job → Create
- ❌ Job → Delete
- etc.

**Only check: Job → Read ✓**

---

**Verify job permission matrix:**

```
┌──────────────────────────────────────────────┐
│  ✓ Enable project-based security             │
│                                              │
│  User/group to add: [___________] [Add]     │
│                                              │
│           | Job                              │
│  User     | Build Cancel Configure Read...   │
│  ---------+---------------------------------- │
│  admin    |  ✓     ✓       ✓        ✓  ...  │
│  javed    |                          ✓       │
│                                              │
└──────────────────────────────────────────────┘
```

**Summary:**
- ✅ admin: All job permissions
- ✅ siva: Job → Read ONLY

---

**Click "Save" at the bottom of the page.**

**Repeat for all existing jobs if there are multiple jobs.**

---

### Step 8: Verify Access Control

**Test as `siva` user:**

**Logout from admin:**
1. Click on **admin** (top right corner)
2. Click **Log Out**

---

**Login as `siva`:**
1. **Username:** `siva`
2. **Password:** `Rc5C9EyvbU`
3. Click **Sign in**

---

**Verify `siva` permissions:**

**What `siva` SHOULD be able to do:**
- ✅ View Jenkins dashboard
- ✅ See the list of jobs
- ✅ Click on job and view configuration (read-only)
- ✅ View build history
- ✅ View console output of builds

**What `siva` should NOT be able to do:**
- ❌ Access "Manage Jenkins"
- ❌ Create new jobs
- ❌ Build jobs (no "Build Now" button)
- ❌ Configure jobs (no "Configure" option or it's read-only)
- ❌ Delete jobs
- ❌ Install plugins
- ❌ Manage users

---

**Test the limitations:**

**1. Try to access Manage Jenkins:**
- "Manage Jenkins" should NOT appear in the sidebar
- Or clicking it shows "Access Denied" / "403 Forbidden"

**2. Try to create a new job:**
- "New Item" should NOT appear in the sidebar
- Or clicking it shows "Access Denied"

**3. View existing job:**
- Click on the job name (should work ✓)
- You should see the job page
- Build history should be visible
- "Configure" might be visible but read-only (depending on Jenkins version)
- "Build Now" should NOT appear (no build permission)

**4. Try to build:**
- No "Build Now" button/link
- If you manually navigate to `/job/<jobname>/build`, you'll get "Access Denied"

---

**Expected "Access Denied" message:**
```
┌──────────────────────────────────────────────┐
│         Access Denied                         │
├──────────────────────────────────────────────┤
│                                              │
│  siva is missing the Overall/Administer      │
│  permission                                  │
│                                              │
│  [Go back to top page]                       │
│                                              │
└──────────────────────────────────────────────┘
```

---

**Logout as `siva` and login back as `admin`:**
1. Click **siva** → **Log Out**
2. Login with `admin` / `Adm!n321`
3. Verify you still have full administrative access

---

### Step 9: Verify Configuration via CLI (Optional)

**From Jenkins server:**

```bash
# SSH to Jenkins server
ssh root@jenkins

# Check users
ls -la /var/lib/jenkins/users/

# Should see directories for admin, theadmin (from Day 68), and siva
```

**Expected output:**
```
drwxr-xr-x 2 jenkins jenkins   80 Jan 14 10:30 admin_1234567890/
drwxr-xr-x 2 jenkins jenkins   80 Jan 14 10:25 theadmin_9876543210/
drwxr-xr-x 2 jenkins jenkins   80 Jan 14 11:45 siva_5555555555/
-rw-r--r-- 1 jenkins jenkins  456 Jan 14 11:45 users.xml
```

---

**Check global security configuration:**

```bash
cat /var/lib/jenkins/config.xml | grep -A 20 "authorizationStrategy"
```

**Expected output:**
```xml
<authorizationStrategy class="hudson.security.ProjectMatrixAuthorizationStrategy">
  <permission>hudson.model.Hudson.Administer:admin</permission>
  <permission>hudson.model.Hudson.Read:siva</permission>
</authorizationStrategy>
```

---

**Check job configuration:**

```bash
cat /var/lib/jenkins/jobs/test-git-plugin/config.xml | grep -A 10 "properties"
```

**Expected to see project-based security enabled with siva having read permission.**

---

## 📊 Complete Configuration Verification Script

**Run this script from Jenkins server to verify all settings:**

```bash
#!/bin/bash

echo "🔐 Day 70: Jenkins Security Configuration Verification"
echo "======================================================"

# Colors
GREEN='\033[0;32m'
RED='\033[0;31m'
YELLOW='\033[1;33m'
NC='\033[0m'

# Check if Jenkins is running
echo -e "\n${YELLOW}1️⃣ Checking Jenkins Service...${NC}"
if systemctl is-active --quiet jenkins; then
    echo -e "${GREEN}✅ Jenkins service is running${NC}"
else
    echo -e "${RED}❌ Jenkins service is not running${NC}"
    exit 1
fi

# Check users
echo -e "\n${YELLOW}2️⃣ Checking Jenkins Users...${NC}"
if [ -d "/var/lib/jenkins/users" ]; then
    echo -e "${GREEN}✅ Users directory exists${NC}"
    
    # Check for admin
    if ls /var/lib/jenkins/users/ | grep -q "admin"; then
        echo -e "${GREEN}  ✓ admin user exists${NC}"
    else
        echo -e "${RED}  ✗ admin user not found${NC}"
    fi
    
    # Check for siva
    if ls /var/lib/jenkins/users/ | grep -q "siva"; then
        echo -e "${GREEN}  ✓ siva user exists${NC}"
    else
        echo -e "${RED}  ✗ siva user not found${NC}"
    fi
    
    echo -e "\n  All users:"
    ls -1 /var/lib/jenkins/users/ | grep -v "users.xml" | while read user; do
        echo -e "    - ${user}"
    done
fi

# Check authorization strategy
echo -e "\n${YELLOW}3️⃣ Checking Authorization Strategy...${NC}"
if grep -q "ProjectMatrixAuthorizationStrategy" /var/lib/jenkins/config.xml; then
    echo -e "${GREEN}✅ Project-based Matrix Authorization Strategy enabled${NC}"
elif grep -q "GlobalMatrixAuthorizationStrategy" /var/lib/jenkins/config.xml; then
    echo -e "${YELLOW}⚠️  Matrix Authorization Strategy enabled (not project-based)${NC}"
else
    echo -e "${RED}❌ Matrix Authorization Strategy not found${NC}"
fi

# Check admin permissions
echo -e "\n${YELLOW}4️⃣ Checking Admin Permissions...${NC}"
if grep -q "Hudson.Administer:admin" /var/lib/jenkins/config.xml; then
    echo -e "${GREEN}✅ admin has Administer permission${NC}"
else
    echo -e "${RED}❌ admin Administer permission not found${NC}"
fi

# Check siva permissions
echo -e "\n${YELLOW}5️⃣ Checking Siva Permissions...${NC}"
if grep -q "Hudson.Read:siva" /var/lib/jenkins/config.xml; then
    echo -e "${GREEN}✅ siva has Read permission${NC}"
else
    echo -e "${RED}❌ siva Read permission not found${NC}"
fi

# Check for Anonymous permissions (should be none)
echo -e "\n${YELLOW}6️⃣ Checking Anonymous Permissions...${NC}"
if grep -q "anonymous" /var/lib/jenkins/config.xml; then
    echo -e "${RED}❌ Anonymous permissions found (should be removed)${NC}"
else
    echo -e "${GREEN}✅ No Anonymous permissions found${NC}"
fi

# Check Matrix Authorization Plugin
echo -e "\n${YELLOW}7️⃣ Checking Matrix Authorization Plugin...${NC}"
if [ -d "/var/lib/jenkins/plugins/matrix-auth" ]; then
    echo -e "${GREEN}✅ Matrix Authorization Strategy Plugin installed${NC}"
    ls -lah /var/lib/jenkins/plugins/matrix-auth*.jpi 2>/dev/null | head -1
else
    echo -e "${RED}❌ Matrix Authorization Strategy Plugin not found${NC}"
fi

# Check jobs with project-based security
echo -e "\n${YELLOW}8️⃣ Checking Job-Level Security...${NC}"
if [ -d "/var/lib/jenkins/jobs" ]; then
    JOB_COUNT=$(ls -1 /var/lib/jenkins/jobs/ | wc -l)
    echo -e "  Found ${JOB_COUNT} job(s)"
    
    for job in /var/lib/jenkins/jobs/*/config.xml; do
        job_name=$(basename $(dirname "$job"))
        echo -e "\n  Job: ${job_name}"
        
        if grep -q "hudson.security.AuthorizationMatrixProperty" "$job"; then
            echo -e "${GREEN}    ✓ Project-based security enabled${NC}"
            
            if grep -q "hudson.model.Item.Read:siva" "$job"; then
                echo -e "${GREEN}    ✓ siva has Read permission${NC}"
            else
                echo -e "${YELLOW}    ⚠️  siva Read permission not found${NC}"
            fi
        else
            echo -e "${YELLOW}    ⚠️  Project-based security not enabled${NC}"
        fi
    done
fi

# Final Summary
echo -e "\n${YELLOW}======================================================"
echo "📊 Configuration Summary"
echo "======================================================${NC}"

echo -e "\n${GREEN}✅ Jenkins Service: Running${NC}"
echo -e "${GREEN}✅ Users: admin, siva created${NC}"
echo -e "${GREEN}✅ Authorization: Project-based Matrix Strategy${NC}"
echo -e "${GREEN}✅ Global Permissions: Configured${NC}"
echo -e "${GREEN}✅ Job Permissions: Configured${NC}"

echo -e "\n${YELLOW}📋 Next Steps:${NC}"
echo -e "1. Test login as siva user"
echo -e "2. Verify siva can view but not modify jobs"
echo -e "3. Verify anonymous access is blocked"
echo -e "4. Document configuration with screenshots"

echo -e "\n${GREEN}✅ Security configuration verification completed!${NC}"
```

**Save and run:**
```bash
chmod +x verify-security.sh
./verify-security.sh
```

---

## 🐛 Common Issues and Solutions

### Issue 1: Matrix Authorization Plugin Not Found

**Symptoms:**
- "Project-based Matrix Authorization Strategy" option not in Configure Global Security
- Only see basic authorization options

**Solution:**

```
1. Manage Jenkins → Plugins → Available plugins
2. Search for "Matrix Authorization Strategy Plugin"
3. Install the plugin
4. Check "Restart Jenkins when installation is complete..."
5. Wait for restart and login again
6. Go back to Configure Global Security
7. Option should now appear
```

---

### Issue 2: Locked Out After Changing Security Settings

**Symptoms:**
- Accidentally removed admin permissions
- Can't log in or access Manage Jenkins
- "Access Denied" for admin user

**Solution A: Disable Security Temporarily**

```bash
# SSH to Jenkins server
ssh root@jenkins

# Edit config.xml
vi /var/lib/jenkins/config.xml

# Find and set useSecurity to false:
<useSecurity>false</useSecurity>

# Save and restart Jenkins
systemctl restart jenkins

# Access Jenkins (no login required)
# Fix permissions
# Re-enable security
# Set useSecurity back to true
# Restart again
```

---

**Solution B: Grant Admin Permissions via CLI**

```bash
# Edit config.xml directly
vi /var/lib/jenkins/config.xml

# Find authorizationStrategy section
# Add admin permission:
<permission>hudson.model.Hudson.Administer:admin</permission>

# Save and restart
systemctl restart jenkins
```

---

### Issue 3: User `siva` Can't See Jobs

**Symptoms:**
- siva logs in successfully
- Dashboard is empty
- No jobs visible

**Diagnosis:**
- siva has Overall Read but no job-level permissions
- Or project-based security not enabled on jobs

**Solution:**

```
1. Login as admin
2. Go to each job → Configure
3. Enable project-based security
4. Add siva with Read permission
5. Save
6. Login as siva again
7. Jobs should now be visible
```

---

### Issue 4: "Enable project-based security" Option Not Visible in Job

**Symptoms:**
- In job configuration, can't find project-based security checkbox
- No permission matrix in job settings

**Diagnosis:**
- Using "Matrix-based security" instead of "Project-based Matrix Authorization Strategy"
- Wrong authorization strategy selected

**Solution:**

```
1. Manage Jenkins → Configure Global Security
2. Verify authorization strategy selected is:
   "Project-based Matrix Authorization Strategy"
   NOT just "Matrix-based security"
3. Save
4. Go back to job configuration
5. Option should now appear
```

---

### Issue 5: Anonymous Users Can Still Access Jenkins

**Symptoms:**
- Can access Jenkins without logging in
- Jobs visible in incognito/private browser window

**Diagnosis:**
- Anonymous still has permissions in matrix
- Security not properly saved

**Solution:**

```
1. Manage Jenkins → Configure Global Security
2. Find "Anonymous" in the permission matrix
3. Either:
   A. Uncheck ALL permissions for Anonymous
   B. Remove Anonymous row entirely (if delete button available)
4. Verify no checkmarks for Anonymous
5. Click Save
6. Test in incognito window - should redirect to login
```

---

### Issue 6: siva Can Build Jobs (Should Only Read)

**Symptoms:**
- siva sees "Build Now" button
- siva can trigger builds
- Too many permissions granted

**Diagnosis:**
- Build permission accidentally granted
- Global permissions too broad

**Solution:**

```
1. Login as admin
2. Check GLOBAL permissions:
   Manage Jenkins → Configure Global Security
   Verify siva only has: Overall → Read
   
3. Check JOB permissions:
   Job → Configure → Enable project-based security
   Verify siva only has: Job → Read
   
4. Remove any extra permissions
5. Save
6. Test as siva - "Build Now" should disappear
```

---

## 📖 Key Takeaways

### Concepts Mastered Today:

1. ✅ **Jenkins Security Architecture**
   - Security Realm (authentication)
   - Authorization Strategy (permissions)
   - Separation of concerns

2. ✅ **User Management**
   - Creating users in Jenkins database
   - User credentials management
   - User directory structure

3. ✅ **Matrix-Based Authorization**
   - Global permissions
   - Project-level permissions
   - Permission inheritance
   - Granular access control

4. ✅ **Permission Types**
   - Overall (global) permissions
   - Job-specific permissions
   - Credential permissions
   - Agent permissions

5. ✅ **Security Best Practices**
   - Least privilege principle
   - Removing anonymous access
   - Role-based access control
   - Testing permission changes

---

## 🎯 Real-World Lessons

### Security Configuration Patterns

**Development Environment:**
```yaml
Current Setup (Day 70):
- Jenkins own user database
- Manual user creation
- Project-based Matrix Authorization
- Per-user permissions

Good for:
- Small teams (< 20 users)
- Learning environments
- Proof of concepts
```

**Production Environment:**
```yaml
Recommended Setup:
- LDAP/Active Directory integration
- Group-based permissions
- Role-Based Access Control (RBAC)
- Audit logging
- SSO integration

Example LDAP:
---
Security Realm: LDAP
Server: ldap://ldap.company.com
Root DN: dc=company,dc=com
User search base: ou=users
Group search base: ou=groups

Authorization: Project-based Matrix
Groups:
- jenkins-admins → Overall Administer
- jenkins-developers → Overall Read, Job Build/Read
- jenkins-viewers → Overall Read, Job Read
```

---

### Enterprise Permission Structure

**1. Role Definitions:**

```
Admin Role:
- Overall: Administer, Read, RunScripts
- Credentials: All
- Agent: All
- Job: All
- View: All

Developer Role:
- Overall: Read
- Job: Build, Cancel, Read, Workspace
- View: Read
- Run: Delete (own builds)

Viewer Role:
- Overall: Read
- Job: Read
- View: Read
```

---

**2. Group-Based Management:**

```
Instead of:
- user1 → permissions
- user2 → permissions
- user3 → permissions

Use:
- dev-team group → developer permissions
  - user1 (member of dev-team)
  - user2 (member of dev-team)
  
- ops-team group → admin permissions
  - user3 (member of ops-team)
```

---

**3. Job Organization:**

```
Folder Structure:
- Production/
  - app1-prod
  - app2-prod
  Permissions: ops-team (build), dev-team (read)
  
- Staging/
  - app1-staging
  - app2-staging
  Permissions: dev-team (build, configure)
  
- Development/
  - app1-dev
  - app2-dev
  Permissions: dev-team (all), interns (read)
```

---

### Permission Matrix Best Practices

**1. Start Restrictive, Add as Needed:**
```
✅ Grant minimum permissions initially
✅ Add permissions based on requests
❌ Don't grant all permissions upfront
❌ Don't use "Logged-in users can do anything"
```

**2. Document Permission Rationale:**
```
User: siva
Reason: Junior developer on Project X
Permissions:
- Overall Read (access Jenkins)
- Project X: Build, Read (work on assigned project)
- Other projects: Read only (learn from others)
```

**3. Regular Audits:**
```
Monthly:
- Review user list
- Remove inactive users
- Verify permissions still appropriate

Quarterly:
- Audit permission changes
- Review access logs
- Update documentation
```

**4. Separation of Duties:**
```
✅ Separate roles:
- Admin: Manages Jenkins, plugins, security
- Release Manager: Deploys to production
- Developer: Builds and tests
- Viewer: Monitors only

❌ Don't give everyone admin access
```

---

## 📚 Additional Resources

**Official Documentation:**
- [Jenkins Security](https://www.jenkins.io/doc/book/security/)
- [Matrix Authorization Strategy Plugin](https://plugins.jenkins.io/matrix-auth/)
- [Managing Security](https://www.jenkins.io/doc/book/managing/security/)
- [Standard Security Setup](https://www.jenkins.io/doc/book/security/standard/)

**Tutorials:**
- [Securing Jenkins](https://www.jenkins.io/doc/book/security/securing/)
- [Authorization Strategies](https://www.jenkins.io/doc/book/system-administration/security/)
- [User Management](https://www.jenkins.io/doc/book/managing/users/)

**Best Practices:**
- [Jenkins Security Best Practices](https://www.jenkins.io/doc/book/security/best-practices/)
- [Access Control](https://www.jenkins.io/doc/book/security/access-control/)

**Next Steps:**
- **Day 71:** LDAP Integration for Jenkins
- **Day 72:** Jenkins Credentials Management
- **Day 73:** Jenkins Audit Trail and Logging
- **Day 74:** Jenkins Role-Based Access with Folders

---

## ✅ Task Completion Checklist

**Plugin Installation:**
- [ ] Checked if Matrix Authorization Strategy Plugin installed
- [ ] Installed plugin if needed
- [ ] Restarted Jenkins (if required)
- [ ] Verified plugin appears in installed list

**User Creation:**
- [ ] Navigated to Manage Users
- [ ] Created user `siva`
  - Username: `siva`
  - Password: `Rc5C9EyvbU`
  - Full Name: `Siva`
- [ ] Verified user appears in user list

**Global Security Configuration:**
- [ ] Navigated to Configure Global Security
- [ ] Selected "Project-based Matrix Authorization Strategy"
- [ ] Verified `admin` has Overall → Administer permission
- [ ] Added `siva` to permission matrix
- [ ] Granted `siva` Overall → Read permission (only)
- [ ] Removed ALL Anonymous permissions
- [ ] Saved configuration
- [ ] Verified still logged in as admin

**Job-Level Security:**
- [ ] Identified existing job(s)
- [ ] Opened job configuration
- [ ] Enabled "Enable project-based security"
- [ ] Added `siva` to job permission matrix
- [ ] Granted `siva` Job → Read permission (only)
- [ ] Saved job configuration

**Verification:**
- [ ] Logged out as admin
- [ ] Logged in as `siva` / `Rc5C9EyvbU`
- [ ] Verified siva can see dashboard
- [ ] Verified siva can see job(s)
- [ ] Verified siva CANNOT access Manage Jenkins
- [ ] Verified siva CANNOT build jobs
- [ ] Verified siva CANNOT create new jobs
- [ ] Logged out as siva
- [ ] Logged back in as admin
- [ ] Verified admin still has full access
- [ ] Tested anonymous access (should be blocked)

**Documentation:**
- [ ] Took screenshots of permission matrix
- [ ] Documented user credentials
- [ ] Captured any issues encountered
- [ ] Noted plugin versions used

---

**🎉 Congratulations!** You've successfully configured Jenkins user management and implemented Matrix-based authorization with granular permissions! You've learned how to secure Jenkins, create users, and implement the principle of least privilege.

**Day 70 Status:** ✅ Complete

**Next:** Day 71 - Advanced Jenkins Security with LDAP Integration 🚀
