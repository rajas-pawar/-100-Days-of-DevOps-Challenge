# Day 69: Installing Jenkins Plugins (Git and GitLab)

## 📋 Task Overview

**Scenario:** The Nautilus DevOps team has recently set up a Jenkins server and wants to install essential plugins that will be used in most CI/CD jobs. Your task is to install the Git and GitLab plugins through the Jenkins web interface.

**Given Requirements:**
- **Jenkins UI Access:** Click Jenkins button on top bar
- **Login Credentials:**
  - Username: `admin`
  - Password: `Adm!n321`
- **Plugins to Install:**
  - Git plugin
  - GitLab plugin
- **Post-Installation:** Restart Jenkins if required

**Your Mission:**
1. Access Jenkins web UI
2. Log in with provided credentials
3. Navigate to plugin manager
4. Install Git plugin
5. Install GitLab plugin
6. Restart Jenkins service (if required)
7. Verify plugin installation

---

## 🎯 Learning Objectives

By the end of this task, you will understand:
- **Jenkins Plugin System:** Architecture and management
- **Plugin Installation:** Multiple methods (UI, CLI, manual)
- **Plugin Dependencies:** How plugins depend on each other
- **Jenkins Restart:** Safe restart procedures
- **Plugin Verification:** Confirming successful installation
- **CI/CD Integration:** Git and GitLab plugin capabilities

---

## 📖 Understanding Jenkins Plugins

### What are Jenkins Plugins?

**Jenkins plugins** extend the core functionality of Jenkins by adding features for:
- Source code management (Git, SVN, GitLab)
- Build tools (Maven, Gradle, npm)
- Testing frameworks (JUnit, Selenium)
- Deployment (Docker, Kubernetes, AWS)
- Notifications (Slack, Email, Microsoft Teams)
- Security (LDAP, OAuth, SAML)

**Plugin Architecture:**
```
┌─────────────────────────────────────────┐
│         Jenkins Core                     │
│  - Basic functionality                   │
│  - Job execution                         │
│  - User management                       │
└──────────────┬──────────────────────────┘
               │
               │ Plugin API
               │
    ┌──────────┴──────────┬──────────────┐
    ↓                     ↓              ↓
┌─────────┐         ┌──────────┐    ┌─────────┐
│   Git   │         │  GitLab  │    │ Docker  │
│ Plugin  │         │  Plugin  │    │ Plugin  │
└─────────┘         └──────────┘    └─────────┘
```

---

### Git Plugin

**Purpose:** Integrates Git version control with Jenkins

**Key Features:**
- Clone repositories from Git servers
- Checkout specific branches, tags, or commits
- Trigger builds on Git commits
- Support for multiple Git repositories
- Submodule support
- Credential management for Git authentication

**Common Use Cases:**
```groovy
// Pipeline example using Git plugin
pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/user/repo.git'
            }
        }
    }
}
```

---

### GitLab Plugin

**Purpose:** Integrates GitLab with Jenkins for advanced CI/CD workflows

**Key Features:**
- GitLab webhook integration
- Trigger builds on GitLab events (push, merge request)
- Report build status back to GitLab
- GitLab merge request builder
- Pipeline status updates
- GitLab authentication

**Integration Flow:**
```
GitLab Repository
       ↓
    Git Push / Merge Request
       ↓
   Webhook Trigger
       ↓
  Jenkins Build
       ↓
 Build Status Update
       ↓
   GitLab UI
```

**Benefits:**
- ✅ Automated CI/CD workflows
- ✅ Merge request validation
- ✅ Build status visibility in GitLab
- ✅ Seamless integration with GitLab features

---

## 📖 Jenkins Plugin Manager

### Plugin Installation Methods

**1. Web UI (Recommended for manual installation):**
- User-friendly interface
- Automatic dependency resolution
- Visual plugin information
- Restart management

**2. Jenkins CLI:**
```bash
java -jar jenkins-cli.jar -s http://localhost:8080/ install-plugin git gitlab
```

**3. Manual Installation:**
```bash
# Download .hpi file and place in plugins directory
cp plugin.hpi /var/lib/jenkins/plugins/
systemctl restart jenkins
```

**4. Configuration as Code (JCasC):**
```yaml
jenkins:
  plugins:
    required:
      - git:latest
      - gitlab:latest
```

---

### Plugin Update Center

**What is it?**
- Central repository for Jenkins plugins
- Hosts plugin metadata and downloads
- Provides version information
- Manages plugin dependencies

**Update Sites:**
- **Default:** `https://updates.jenkins.io/update-center.json`
- **Experimental:** `https://updates.jenkins.io/experimental/update-center.json`
- **Custom:** Organizations can host their own

---

## 🛠️ Task Implementation

### Step 1: Access Jenkins Web UI

**Click the "Jenkins" button on the top bar of your lab environment.**

**Or navigate to:**
```
http://<jenkins-server-ip>:8080
```

**You should see the Jenkins login page:**
```
┌──────────────────────────────────────────────┐
│              Jenkins                          │
├──────────────────────────────────────────────┤
│                                              │
│              [Jenkins Logo]                  │
│                                              │
│  Username: [___________________]            │
│                                              │
│  Password: [___________________]            │
│                                              │
│  [ ] Remember me                            │
│                                              │
│              [Sign in]                       │
│                                              │
└──────────────────────────────────────────────┘
```

---

### Step 2: Log in to Jenkins

**Enter the credentials:**
- **Username:** `admin`
- **Password:** `Adm!n321`

**Click "Sign in"**

**Expected result:**
- You'll be redirected to the Jenkins Dashboard
- You'll see "Welcome to Jenkins!" at the top
- Navigation menu on the left side

---

### Step 3: Navigate to Plugin Manager

**From the Jenkins Dashboard:**

**Method 1: Via Main Menu**
1. Click **"Manage Jenkins"** on the left sidebar
2. Click **"Manage Plugins"** (or **"Plugins"** in newer versions)

**Method 2: Direct URL**
```
http://<jenkins-server-ip>:8080/pluginManager/
```

**You'll see the Plugin Manager page with tabs:**
```
┌──────────────────────────────────────────────┐
│         Manage Plugins                        │
├──────────────────────────────────────────────┤
│  [Updates] [Available] [Installed] [Advanced]│
├──────────────────────────────────────────────┤
│                                              │
│  Search: [________________] 🔍              │
│                                              │
└──────────────────────────────────────────────┘
```

---

### Step 4: Navigate to Available Plugins Tab

**Click on the "Available" tab** (or "Available plugins" in newer versions)

**This shows all plugins that can be installed.**

**Expected view:**
```
┌──────────────────────────────────────────────┐
│         Available Plugins                     │
├──────────────────────────────────────────────┤
│                                              │
│  Filter: [________________] 🔍              │
│                                              │
│  [✓] Select All                             │
│                                              │
│  [ ] Plugin Name 1 (v1.2.3)                 │
│      Description...                          │
│                                              │
│  [ ] Plugin Name 2 (v2.3.4)                 │
│      Description...                          │
│                                              │
└──────────────────────────────────────────────┘
```

---

### Step 5: Search and Select Git Plugin

**In the search/filter box, type:** `Git`

**Find the "Git plugin" in the list:**
```
┌──────────────────────────────────────────────┐
│  Search: [Git____________] 🔍               │
├──────────────────────────────────────────────┤
│  [ ] Git plugin (v5.x.x)                    │
│      This plugin integrates Git with         │
│      Jenkins. It allows you to use Git       │
│      as a SCM tool.                          │
│                                              │
│  [ ] Git Parameter (v0.x.x)                 │
│      Adds ability to choose branches...      │
│                                              │
│  [ ] Git Client Plugin (v4.x.x)             │
│      Utility plugin for Git support...       │
│                                              │
└──────────────────────────────────────────────┘
```

**Check the checkbox next to "Git plugin"**

**Note:** You may see "Git plugin" already installed if it was part of the suggested plugins during initial setup. If already installed, you'll see it in the "Installed" tab instead.

---

### Step 6: Search and Select GitLab Plugin

**Clear the search box and type:** `GitLab`

**Find the "GitLab plugin" in the list:**
```
┌──────────────────────────────────────────────┐
│  Search: [GitLab_________] 🔍               │
├──────────────────────────────────────────────┤
│  [ ] GitLab Plugin (v1.x.x)                 │
│      This plugin allows GitLab to trigger    │
│      builds in Jenkins and send build        │
│      status back to GitLab                   │
│                                              │
│  [ ] GitLab API Plugin (v5.x.x)             │
│      This plugin provides GitLab API...      │
│                                              │
│  [ ] GitLab Authentication (v1.x.x)         │
│      This plugin provides authentication...  │
│                                              │
└──────────────────────────────────────────────┘
```

**Check the checkbox next to "GitLab Plugin"**

**Important:** The GitLab plugin may have dependencies. Jenkins will automatically select required dependencies.

---

### Step 7: Review Selected Plugins

**Scroll down to see all selected plugins.**

**You should see:**
- ✅ Git plugin (if not already installed)
- ✅ GitLab Plugin
- ✅ Any dependencies automatically selected

**Common dependencies for GitLab plugin:**
- GitLab API Plugin
- Credentials Plugin
- Plain Credentials Plugin
- Git Client Plugin

---

### Step 8: Install Plugins

**Scroll to the bottom of the page.**

**You'll see installation options:**
```
┌──────────────────────────────────────────────┐
│                                              │
│  [Download now and install after restart]   │
│                                              │
│  [Install without restart]                  │
│                                              │
└──────────────────────────────────────────────┘
```

**Click "Install without restart"** (recommended)

**Alternative:** Click "Download now and install after restart" if you prefer to restart manually later.

---

### Step 9: Monitor Plugin Installation

**You'll be redirected to the installation progress page:**

```
┌──────────────────────────────────────────────┐
│         Installing Plugins/Upgrades           │
├──────────────────────────────────────────────┤
│                                              │
│  ✓ Git plugin                                │
│    Success                                   │
│                                              │
│  ⟳ GitLab API Plugin                        │
│    Installing... (dependency)                │
│                                              │
│  ⟳ GitLab Plugin                            │
│    Pending...                                │
│                                              │
│  [ ] Restart Jenkins when installation is   │
│      complete and no jobs are running        │
│                                              │
└──────────────────────────────────────────────┘
```

**Watch the progress:**
- Plugins will install sequentially
- Dependencies install first
- Each plugin shows status: Pending → Installing → Success

---

### Step 10: Check for Restart Requirement

**After all plugins finish installing, look for restart notification:**

```
┌──────────────────────────────────────────────┐
│                                              │
│  ✓ Success                                   │
│                                              │
│  This plugin requires a restart of Jenkins. │
│                                              │
│  [ ] Restart Jenkins when installation is   │
│      complete and no jobs are running        │
│                                              │
└──────────────────────────────────────────────┘
```

**If restart is required:**
- Check the box: **"Restart Jenkins when installation is complete and no jobs are running"**
- Jenkins will automatically restart

**If restart checkbox appears, check it and Jenkins will restart automatically.**

---

### Step 11: Wait for Jenkins to Restart

**After restart is initiated, you'll see:**

```
┌──────────────────────────────────────────────┐
│                                              │
│         Jenkins is restarting...             │
│                                              │
│         Please wait...                       │
│                                              │
└──────────────────────────────────────────────┘
```

**Wait for the login page to reappear** (typically 30-60 seconds)

**Page will automatically reload and show login screen.**

---

### Step 12: Log Back In After Restart

**Once Jenkins restarts, you'll see the login page again.**

**Log in with the same credentials:**
- **Username:** `admin`
- **Password:** `Adm!n321`

**Click "Sign in"**

---

### Step 13: Verify Plugin Installation

**Method 1: Via Plugin Manager**

1. Go to **Manage Jenkins** → **Manage Plugins**
2. Click on the **"Installed"** tab
3. Search for "Git" in the filter box
4. Verify you see:
   - ✅ **Git plugin** with version number
5. Search for "GitLab" in the filter box
6. Verify you see:
   - ✅ **GitLab Plugin** with version number

**Expected view in Installed tab:**
```
┌──────────────────────────────────────────────┐
│         Installed Plugins                     │
├──────────────────────────────────────────────┤
│  Filter: [Git____________] 🔍               │
├──────────────────────────────────────────────┤
│                                              │
│  ✓ Git plugin (v5.2.1)                      │
│    Enabled                                   │
│    This plugin integrates Git with Jenkins   │
│    [Uninstall] [Disable]                    │
│                                              │
│  ✓ Git Client Plugin (v4.6.0)               │
│    Enabled                                   │
│    Utility plugin for Git support            │
│    [Uninstall] [Disable]                    │
│                                              │
└──────────────────────────────────────────────┘
```

---

**Method 2: Via System Information**

1. Go to **Manage Jenkins** → **System Information**
2. Scroll down to "Plugin Dependencies" or look for plugin versions
3. Search for "git" and "gitlab" in the page (Ctrl+F)

---

**Method 3: Via Jenkins CLI (From Jenkins Server)**

```bash
# SSH to Jenkins server
ssh root@jenkins

# Check installed plugins
java -jar /var/cache/jenkins/war/WEB-INF/jenkins-cli.jar -s http://localhost:8080/ -auth admin:Adm!n321 list-plugins | grep -E "git|gitlab"
```

**Expected output:**
```
git                     Git                     5.2.1
gitlab-plugin           GitLab                  1.7.15
gitlab-api              GitLab API              5.3.0
```

---

**Method 4: Check Plugin Directory (From Jenkins Server)**

```bash
# SSH to Jenkins server
ssh root@jenkins

# List Git-related plugins
ls -lah /var/lib/jenkins/plugins/ | grep -E "git|gitlab"
```

**Expected output:**
```
drwxr-xr-x 3 jenkins jenkins 4.0K Jan 13 10:30 git
-rw-r--r-- 1 jenkins jenkins  137 Jan 13 10:30 git.jpi
drwxr-xr-x 3 jenkins jenkins 4.0K Jan 13 10:31 gitlab-plugin
-rw-r--r-- 1 jenkins jenkins  156 Jan 13 10:31 gitlab-plugin.jpi
```

---

### Step 14: Test Git Plugin Functionality

**Create a test job to verify Git plugin works:**

1. **From Jenkins Dashboard, click "New Item"**
2. **Enter item name:** `test-git-plugin`
3. **Select "Freestyle project"**
4. **Click "OK"**

5. **In job configuration, scroll to "Source Code Management"**
6. **Select "Git"** (this option appears because Git plugin is installed)
7. **In "Repository URL", enter a public repo:**
   ```
   https://github.com/jenkinsci/git-plugin.git
   ```
8. **Leave branch as:** `*/master`
9. **Click "Save"**

10. **Click "Build Now"**

**If Git plugin is working correctly:**
- Build will start
- Git will clone the repository
- Build will succeed (or fail if repo has issues, but Git checkout should work)

**Check build console output:**
```
Started by user admin
Cloning repository https://github.com/jenkinsci/git-plugin.git
 > git init /var/lib/jenkins/workspace/test-git-plugin
 > git fetch --tags --progress https://github.com/jenkinsci/git-plugin.git
 > git checkout -f <commit-hash>
Finished: SUCCESS
```

---

### Step 15: Test GitLab Plugin Configuration

**Verify GitLab plugin is available in system configuration:**

1. **Go to Manage Jenkins → Configure System**
2. **Scroll to "GitLab" section**
3. **You should see GitLab configuration options:**
   - Connection name
   - GitLab host URL
   - API token
   - Test connection button

**Expected view:**
```
┌──────────────────────────────────────────────┐
│         GitLab                                │
├──────────────────────────────────────────────┤
│                                              │
│  GitLab connections                          │
│                                              │
│  Connection name: [_______________]         │
│  GitLab host URL: [https://gitlab.com]     │
│  Credentials:     [- none -        ▼]      │
│  API Token:       [Add ▼]                   │
│                                              │
│  [Test Connection]                          │
│                                              │
│  [Add GitLab Server]                        │
│                                              │
└──────────────────────────────────────────────┘
```

**If you see these options, GitLab plugin is successfully installed!**

---

## 📊 Complete Installation Verification Script

**For comprehensive verification, run this from Jenkins server:**

```bash
#!/bin/bash

echo "🔍 Day 69: Jenkins Plugin Verification"
echo "========================================"

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

# Check Git plugin
echo -e "\n${YELLOW}2️⃣ Checking Git Plugin...${NC}"
if [ -d "/var/lib/jenkins/plugins/git" ]; then
    echo -e "${GREEN}✅ Git plugin directory exists${NC}"
    ls -lah /var/lib/jenkins/plugins/git*.jpi 2>/dev/null
else
    echo -e "${RED}❌ Git plugin not found${NC}"
fi

# Check GitLab plugin
echo -e "\n${YELLOW}3️⃣ Checking GitLab Plugin...${NC}"
if [ -d "/var/lib/jenkins/plugins/gitlab-plugin" ]; then
    echo -e "${GREEN}✅ GitLab plugin directory exists${NC}"
    ls -lah /var/lib/jenkins/plugins/gitlab*.jpi 2>/dev/null
else
    echo -e "${RED}❌ GitLab plugin not found${NC}"
fi

# Check GitLab API plugin (dependency)
echo -e "\n${YELLOW}4️⃣ Checking GitLab API Plugin (dependency)...${NC}"
if [ -d "/var/lib/jenkins/plugins/gitlab-api" ]; then
    echo -e "${GREEN}✅ GitLab API plugin directory exists${NC}"
else
    echo -e "${YELLOW}⚠️  GitLab API plugin not found (may be bundled)${NC}"
fi

# List all Git-related plugins
echo -e "\n${YELLOW}5️⃣ All Git-related Plugins:${NC}"
ls -d /var/lib/jenkins/plugins/git* 2>/dev/null | while read plugin; do
    plugin_name=$(basename "$plugin")
    echo -e "${GREEN}  ✓ $plugin_name${NC}"
done

# Check Jenkins logs for plugin loading
echo -e "\n${YELLOW}6️⃣ Checking Jenkins Logs for Plugin Loading...${NC}"
if journalctl -u jenkins --since "10 minutes ago" | grep -q "git.*plugin"; then
    echo -e "${GREEN}✅ Git plugin loaded in recent logs${NC}"
fi

if journalctl -u jenkins --since "10 minutes ago" | grep -q "gitlab.*plugin"; then
    echo -e "${GREEN}✅ GitLab plugin loaded in recent logs${NC}"
fi

# Test Jenkins API for plugins
echo -e "\n${YELLOW}7️⃣ Testing Jenkins API for Installed Plugins...${NC}"
JENKINS_URL="http://localhost:8080"

# Get plugin list via API
PLUGINS=$(curl -s -u admin:Adm!n321 "${JENKINS_URL}/pluginManager/api/json?depth=1" | grep -o '"shortName":"[^"]*"' | cut -d'"' -f4)

if echo "$PLUGINS" | grep -q "^git$"; then
    echo -e "${GREEN}✅ Git plugin found via API${NC}"
else
    echo -e "${RED}❌ Git plugin not found via API${NC}"
fi

if echo "$PLUGINS" | grep -q "gitlab-plugin"; then
    echo -e "${GREEN}✅ GitLab plugin found via API${NC}"
else
    echo -e "${RED}❌ GitLab plugin not found via API${NC}"
fi

# Final summary
echo -e "\n${YELLOW}========================================"
echo "📊 Verification Summary"
echo "========================================${NC}"

echo -e "\n${GREEN}✅ Jenkins Service: Running${NC}"
echo -e "${GREEN}✅ Git Plugin: Installed${NC}"
echo -e "${GREEN}✅ GitLab Plugin: Installed${NC}"

echo -e "\n${YELLOW}📋 Next Steps:${NC}"
echo -e "1. Access Jenkins UI and verify plugins in Manage Plugins"
echo -e "2. Create a test job using Git SCM"
echo -e "3. Configure GitLab connection in Manage Jenkins → Configure System"

echo -e "\n${GREEN}✅ Plugin verification completed!${NC}"
```

**Save and run:**
```bash
chmod +x verify-plugins.sh
./verify-plugins.sh
```

---

## 🐛 Common Issues and Solutions

### Issue 1: Plugin Not Found in Available Tab

**Symptoms:**
- Can't find Git or GitLab plugin in Available plugins

**Diagnosis:**
- Plugin may already be installed
- Update center not loaded
- Network connectivity issues

**Solutions:**

**A. Check if Already Installed**
```
1. Go to Manage Jenkins → Manage Plugins
2. Click "Installed" tab
3. Search for "Git" or "GitLab"
```

**B. Update Plugin List**
```
1. Go to Manage Jenkins → Manage Plugins
2. Click "Advanced" tab
3. Scroll to bottom
4. Click "Check now" button
5. Wait for update center to refresh (30-60 seconds)
6. Go back to "Available" tab and search again
```

**C. Check Update Center URL**
```
1. Manage Jenkins → Manage Plugins → Advanced
2. Verify Update Site URL:
   https://updates.jenkins.io/update-center.json
3. Click "Submit"
4. Click "Check now"
```

---

### Issue 2: Plugin Installation Fails

**Symptoms:**
```
Failed to install: Connection timeout
Failed to install: 403 Forbidden
```

**Diagnosis:**
- Network connectivity
- Proxy configuration
- Firewall blocking
- Disk space

**Solutions:**

**A. Check Internet Connectivity**
```bash
# From Jenkins server
curl -I https://updates.jenkins.io
ping -c 3 updates.jenkins.io
```

**B. Check Disk Space**
```bash
df -h /var/lib/jenkins
# Ensure sufficient space
```

**C. Configure Proxy (if needed)**
```
1. Manage Jenkins → Manage Plugins → Advanced
2. Scroll to "HTTP Proxy Configuration"
3. Enter proxy details:
   - Server: proxy.example.com
   - Port: 8080
4. Click "Submit"
5. Click "Check now"
6. Try installing again
```

**D. Manual Plugin Installation**
```bash
# Download plugin manually
cd /var/lib/jenkins/plugins/
wget https://updates.jenkins.io/download/plugins/git/latest/git.hpi
wget https://updates.jenkins.io/download/plugins/gitlab-plugin/latest/gitlab-plugin.hpi

# Set ownership
chown jenkins:jenkins *.hpi

# Restart Jenkins
systemctl restart jenkins
```

---

### Issue 3: Jenkins Won't Restart After Plugin Installation

**Symptoms:**
- Clicked restart but Jenkins not restarting
- Page hangs on "restarting" message

**Diagnosis:**
```bash
# Check Jenkins service status
systemctl status jenkins

# Check Jenkins logs
journalctl -u jenkins -n 50
```

**Solutions:**

**A. Manual Restart via CLI**
```bash
# SSH to Jenkins server
ssh root@jenkins

# Restart Jenkins service
systemctl restart jenkins

# Wait 30-60 seconds
sleep 60

# Check status
systemctl status jenkins
```

**B. Safe Restart via Web UI**
```
1. Navigate to: http://<jenkins-ip>:8080/restart
2. Click "Yes" to confirm restart
3. Wait for Jenkins to come back up
```

**C. Check for Running Jobs**
```
1. If jobs are running, Jenkins won't restart
2. Go to Dashboard
3. Check if any jobs are building
4. Wait for jobs to complete or cancel them
5. Try restart again
```

---

### Issue 4: Plugin Dependencies Not Installed

**Symptoms:**
- Plugin shows as installed but doesn't work
- Missing functionality
- Error in Jenkins logs about missing dependencies

**Diagnosis:**
```bash
# Check Jenkins logs for dependency errors
journalctl -u jenkins | grep -i "dependency\|missing"
```

**Solutions:**

**A. Install Dependencies Manually**
```
1. Manage Jenkins → Manage Plugins → Installed
2. Find the plugin (e.g., GitLab Plugin)
3. Look at dependencies listed
4. Go to Available tab
5. Search and install each dependency
6. Restart Jenkins
```

**B. Reinstall Plugin**
```
1. Uninstall the plugin
2. Restart Jenkins
3. Reinstall plugin (will auto-install dependencies)
4. Restart Jenkins again
```

---

### Issue 5: After Restart, Login Page Doesn't Appear

**Symptoms:**
- Jenkins restarting for too long (5+ minutes)
- "Service Unavailable" or timeout error

**Diagnosis:**
```bash
# Check service status
systemctl status jenkins

# Check if listening on port
netstat -tulpn | grep 8080

# Check logs for errors
journalctl -u jenkins -f
```

**Solutions:**

**A. Check Service Status**
```bash
systemctl status jenkins

# If failed, restart manually
systemctl restart jenkins
```

**B. Check Java Memory**
```bash
# Check memory usage
free -m

# If low memory, adjust Jenkins memory
vi /etc/sysconfig/jenkins
# Add: JENKINS_JAVA_OPTIONS="-Xms512m -Xmx1024m"

systemctl restart jenkins
```

**C. Check Plugin Conflicts**
```bash
# Move plugins temporarily
cd /var/lib/jenkins/plugins/
mkdir ../plugins-backup
mv gitlab-plugin* ../plugins-backup/

# Restart Jenkins
systemctl restart jenkins

# If starts, plugin has issue
# Check plugin version compatibility
```

---

## 📖 Key Takeaways

### Concepts Mastered Today:

1. ✅ **Jenkins Plugin System**
   - Plugin architecture and extension points
   - Plugin Manager interface
   - Available vs Installed plugins
   - Update center functionality

2. ✅ **Plugin Installation Process**
   - UI-based installation
   - Dependency management
   - Installation progress monitoring
   - Restart procedures

3. ✅ **Git Plugin**
   - Git SCM integration
   - Repository cloning
   - Branch/tag support
   - Credential management

4. ✅ **GitLab Plugin**
   - GitLab webhook integration
   - Merge request triggers
   - Build status reporting
   - GitLab API integration

5. ✅ **Jenkins Administration**
   - Safe restart procedures
   - Plugin verification methods
   - Troubleshooting techniques
   - System configuration

---

## 🎯 Real-World Lessons

### Plugin Management Best Practices

**Development Environment:**
```yaml
Current Setup:
- Manual plugin installation via UI
- Install as needed
- Restart immediately

Works for:
- Learning and testing
- Small teams
- Low-frequency changes
```

**Production Environment:**
```yaml
Recommended Setup:
- Configuration as Code (JCasC)
- Plugin list in version control
- Automated installation
- Scheduled maintenance windows

Example JCasC:
---
jenkins:
  plugins:
    required:
      - git:5.2.1
      - gitlab-plugin:1.7.15
      - workflow-aggregator:latest
      - docker-plugin:1.5.0
```

---

### Enterprise Plugin Strategy

**1. Plugin Approval Process:**
```
Request → Security Review → Testing → Approval → Installation
```

**2. Version Pinning:**
```yaml
# Pin critical plugins to specific versions
plugins:
  git: 5.2.1  # Don't auto-update
  gitlab-plugin: 1.7.15  # Tested version
```

**3. Update Schedule:**
```
Monthly: Security updates
Quarterly: Feature updates
Annually: Major version updates
```

**4. Testing Pipeline:**
```
Dev Jenkins → Test Jenkins → Staging Jenkins → Prod Jenkins
   ↓              ↓              ↓              ↓
Install        Test          Validate       Deploy
```

---

### Essential Jenkins Plugins

**Source Control:**
- Git Plugin - Git integration
- GitHub Plugin - GitHub features
- GitLab Plugin - GitLab integration
- Bitbucket Plugin - Bitbucket integration

**Build Tools:**
- Maven Integration - Maven builds
- Gradle Plugin - Gradle builds
- NodeJS Plugin - Node.js builds
- Docker Pipeline - Docker integration

**CI/CD:**
- Pipeline - Pipeline as Code
- Blue Ocean - Modern UI
- Kubernetes - K8s integration
- AWS Steps - AWS operations

**Notifications:**
- Slack Notification - Slack integration
- Email Extension - Advanced email
- Microsoft Teams - Teams integration

**Security:**
- Credentials Plugin - Credential management
- LDAP Plugin - LDAP authentication
- Role-based Authorization - RBAC

---

## 📚 Additional Resources

**Official Documentation:**
- [Jenkins Plugins Index](https://plugins.jenkins.io/)
- [Git Plugin Documentation](https://plugins.jenkins.io/git/)
- [GitLab Plugin Documentation](https://plugins.jenkins.io/gitlab-plugin/)
- [Plugin Development Guide](https://www.jenkins.io/doc/developer/plugin-development/)

**Tutorials:**
- [Managing Jenkins Plugins](https://www.jenkins.io/doc/book/managing/plugins/)
- [Git Plugin Tutorial](https://www.jenkins.io/doc/pipeline/steps/git/)
- [GitLab Integration Guide](https://docs.gitlab.com/ee/integration/jenkins.html)

**Community:**
- [Jenkins Plugin Site](https://plugins.jenkins.io/)
- [Plugin Development Mailing List](https://www.jenkins.io/mailing-lists/)
- [Jenkins Jira](https://issues.jenkins.io/)

**Next Steps:**
- **Day 70:** Creating Jenkins Pipeline with Git
- **Day 71:** GitLab CI/CD Integration with Jenkins
- **Day 72:** Jenkins Shared Libraries
- **Day 73:** Jenkins Multi-Branch Pipelines

---

## ✅ Task Completion Checklist

**Pre-Installation:**
- [ ] Accessed Jenkins web UI
- [ ] Logged in with admin credentials
- [ ] Navigated to Plugin Manager

**Plugin Installation:**
- [ ] Opened "Available" plugins tab
- [ ] Searched for Git plugin
- [ ] Selected Git plugin (or verified already installed)
- [ ] Searched for GitLab plugin
- [ ] Selected GitLab plugin
- [ ] Reviewed selected plugins and dependencies
- [ ] Clicked "Install without restart"
- [ ] Monitored installation progress

**Post-Installation:**
- [ ] Checked restart requirement
- [ ] Opted to restart Jenkins (if required)
- [ ] Waited for Jenkins to restart
- [ ] Jenkins login page reappeared
- [ ] Logged back in

**Verification:**
- [ ] Verified Git plugin in Installed tab
- [ ] Verified GitLab plugin in Installed tab
- [ ] Checked plugin versions
- [ ] Git option available in job SCM configuration
- [ ] GitLab section visible in Configure System
- [ ] Created test job with Git SCM (optional)
- [ ] Confirmed plugins working correctly

**Documentation:**
- [ ] Took screenshots of plugin installation
- [ ] Documented any issues encountered
- [ ] Noted plugin versions installed

---

**🎉 Congratulations!** You've successfully installed Jenkins plugins (Git and GitLab) and learned how to manage the Jenkins plugin system. Your Jenkins server is now ready for Git-based CI/CD workflows and GitLab integration!

**Day 69 Status:** ✅ Complete

**Next:** Day 70 - Creating Your First Jenkins Pipeline with Git 🚀
