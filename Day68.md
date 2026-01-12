# Day 68: Jenkins Installation and Initial Configuration on Linux

## 📋 Task Overview

**Scenario:** The DevOps team at xFusionCorp Industries is initiating the setup of CI/CD pipelines and has decided to utilize Jenkins as their automation server. You need to install Jenkins on a dedicated server and configure the initial admin user.

**Given Requirements:**
- **Server:** jenkins (accessible from jump host)
- **Installation Method:** yum utility only
- **Service:** Start Jenkins service
- **Admin User Configuration:**
  - Username: `theadmin`
  - Password: `Adm!n321`
  - Full Name: `Siva`
  - Email: `siva@jenkins.stratos.xfusioncorp.com`

**Connection Details:**
- Jump Host → Jenkins Server
- User: `root`
- Password: `S3curePass`

**Your Mission:**
1. Connect to Jenkins server from jump host
2. Install Jenkins using yum
3. Start and enable Jenkins service
4. Access Jenkins UI
5. Complete initial setup wizard
6. Create admin user with specified credentials

---

## 🎯 Learning Objectives

By the end of this task, you will understand:
- **Jenkins Architecture:** Master server, plugins, jobs
- **Jenkins Installation:** RPM-based installation on CentOS/RHEL
- **System Requirements:** Java, ports, dependencies
- **Initial Configuration:** Setup wizard, admin user creation
- **Jenkins Service Management:** systemctl commands
- **CI/CD Fundamentals:** Continuous Integration and Deployment concepts

---

## 📖 Understanding Jenkins

### What is Jenkins?

**Jenkins** is an open-source automation server used for Continuous Integration and Continuous Deployment (CI/CD).

**Key Features:**
- **Automation:** Build, test, deploy applications automatically
- **Plugins:** 1800+ plugins for various tools and platforms
- **Distributed Builds:** Master-agent architecture for scalability
- **Pipeline as Code:** Jenkinsfile for version-controlled pipelines
- **Easy Configuration:** Web-based UI for job management

---

### Jenkins Architecture

```
┌─────────────────────────────────────────────────┐
│              Jenkins Master                      │
│  - Web UI (Port 8080)                           │
│  - Job scheduling                               │
│  - Plugin management                            │
│  - Build coordination                           │
└────────────┬────────────────────────────────────┘
             │
             │ Communicates with
             │
    ┌────────┴────────┬──────────────┐
    ↓                 ↓              ↓
┌──────────┐    ┌──────────┐   ┌──────────┐
│ Agent 1  │    │ Agent 2  │   │ Agent 3  │
│ (Linux)  │    │ (Windows)│   │ (Docker) │
└──────────┘    └──────────┘   └──────────┘
     ↓               ↓              ↓
   Build          Build          Build
  Execution      Execution      Execution
```

---

### CI/CD Pipeline

**Continuous Integration (CI):**
```
Developer → Git Push → Jenkins → Build → Test → Report
```

**Continuous Deployment (CD):**
```
CI Success → Deploy to Staging → Test → Deploy to Production
```

**Benefits:**
- ✅ Early bug detection
- ✅ Automated testing
- ✅ Faster releases
- ✅ Consistent deployments
- ✅ Reduced manual work

---

## 📖 Jenkins System Requirements

### Minimum Requirements

| Component | Requirement |
|-----------|-------------|
| **RAM** | 256 MB (1GB+ recommended) |
| **Disk Space** | 1 GB (10GB+ for builds) |
| **Java** | Java 11 or Java 17 |
| **OS** | Linux, Windows, macOS |
| **Ports** | 8080 (UI), 50000 (agents) |

### Software Dependencies

1. **Java Runtime Environment (JRE)**
   - Jenkins requires Java to run
   - OpenJDK 11 or 17 recommended

2. **Web Browser**
   - For accessing Jenkins UI
   - Chrome, Firefox, Safari, Edge

3. **Network Access**
   - Port 8080 for web interface
   - Internet access for plugin installation

---

## 🛠️ Task Implementation

### Step 1: Connect to Jenkins Server

**From jump host, SSH to Jenkins server:**
```bash
ssh root@jenkins
```

**Enter password when prompted:**
```
Password: S3curePass
```

**Expected output:**
```
Warning: Permanently added 'jenkins' (ECDSA) to the list of known hosts.
Last login: Sun Jan 12 10:00:00 2026 from jump_host
[root@jenkins ~]#
```

**Verify you're on Jenkins server:**
```bash
hostname
```

**Expected output:**
```
jenkins
```

---

### Step 2: Check System Information

**Check OS version:**
```bash
cat /etc/os-release
```

**Expected output:**
```
NAME="CentOS Linux"
VERSION="7 (Core)"
ID="centos"
ID_LIKE="rhel fedora"
VERSION_ID="7"
```

**Check Java version (if installed):**
```bash
java -version
```

**If Java not installed, you'll see:**
```
bash: java: command not found
```

**Check available disk space:**
```bash
df -h
```

**Check available memory:**
```bash
free -m
```

---

### Step 3: Install Java (Jenkins Dependency)

**Jenkins requires Java 17 or 21. Install OpenJDK 17:**
```bash
yum install java-17-openjdk java-17-openjdk-devel -y
```

**Expected output:**
```
Loaded plugins: fastestmirror, ovl
Loading mirror speeds from cached hostfile
Resolving Dependencies
--> Running transaction check
---> Package java-17-openjdk.x86_64 will be installed
...
Installed:
  java-17-openjdk.x86_64 1:17.0.x-x.el9
  java-17-openjdk-devel.x86_64 1:17.0.x-x.el9

Complete!
```

**Verify Java installation:**
```bash
java -version
```

**Expected output:**
```
openjdk version "17.0.x" 2023-xx-xx LTS
OpenJDK Runtime Environment (Red_Hat-17.0.x-x) (build 17.0.x+x-LTS)
OpenJDK 64-Bit Server VM (Red_Hat-17.0.x-x) (build 17.0.x+x-LTS, mixed mode, sharing)
```

**Check JAVA_HOME:**
```bash
echo $JAVA_HOME
```

**If empty, set it:**
```bash
export JAVA_HOME=/usr/lib/jvm/java-17-openjdk
export PATH=$JAVA_HOME/bin:$PATH
```

---

### Step 4: Add Jenkins Repository

**Jenkins is not in default CentOS repositories. Add Jenkins repo:**

```bash
curl -L https://pkg.jenkins.io/redhat-stable/jenkins.repo -o /etc/yum.repos.d/jenkins.repo
```

**Alternative (if wget is available):**
```bash
wget -O /etc/yum.repos.d/jenkins.repo \
    https://pkg.jenkins.io/redhat-stable/jenkins.repo
```

**Expected output (curl):**
```
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100    85  100    85    0     0    xxx      0 --:--:-- --:--:-- --:--:--   xxx
```

**Import Jenkins GPG key:**
```bash
rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
```

**No output means success.**

**Verify repository added:**
```bash
cat /etc/yum.repos.d/jenkins.repo
```

**Expected output:**
```
[jenkins]
name=Jenkins-stable
baseurl=http://pkg.jenkins.io/redhat-stable
gpgcheck=1
```

---

### Step 5: Install Jenkins

**Install Jenkins using yum:**
```bash
yum install jenkins -y
```

**Expected output:**
```
Loaded plugins: fastestmirror, ovl
Loading mirror speeds from cached hostfile
jenkins                                                  | 2.9 kB  00:00:00
jenkins/primary_db                                       |  37 kB  00:00:00
Resolving Dependencies
--> Running transaction check
---> Package jenkins.noarch 0:2.xxx.x-1.1 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================
 Package         Arch           Version              Repository          Size
================================================================================
Installing:
 jenkins         noarch         2.xxx.x-1.1          jenkins             xx M

Transaction Summary
================================================================================
Install  1 Package

Total download size: xx M
Installed size: xx M
Downloading packages:
jenkins-2.xxx.x-1.1.noarch.rpm                           |  xx MB  00:00:10
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : jenkins-2.xxx.x-1.1.noarch                                  1/1
  Verifying  : jenkins-2.xxx.x-1.1.noarch                                  1/1

Installed:
  jenkins.noarch 0:2.xxx.x-1.1

Complete!
```

**Verify Jenkins installation:**
```bash
rpm -qa | grep jenkins
```

**Expected output:**
```
jenkins-2.xxx.x-1.1.noarch
```

**Check Jenkins version:**
```bash
jenkins --version
```

**Or check package info:**
```bash
rpm -qi jenkins
```

---

### Step 6: Configure Jenkins (Before Starting)

**Check Jenkins home directory:**
```bash
ls -la /var/lib/jenkins/
```

**Check Jenkins configuration file:**
```bash
cat /etc/sysconfig/jenkins
```

**Key configuration parameters:**
```bash
# Jenkins port (default 8080)
JENKINS_PORT="8080"

# Jenkins user
JENKINS_USER="jenkins"

# Java options
JENKINS_JAVA_OPTIONS="-Djava.awt.headless=true"
```

**Note:** Default configuration is usually fine for initial setup.

---

### Step 7: Start Jenkins Service

**Enable Jenkins to start on boot:**
```bash
systemctl enable jenkins
```

**Expected output:**
```
Created symlink from /etc/systemd/system/multi-user.target.wants/jenkins.service to /usr/lib/systemd/system/jenkins.service.
```

**Start Jenkins service:**
```bash
systemctl start jenkins
```

**No output means success.**

**Check Jenkins service status:**
```bash
systemctl status jenkins
```

**Expected output:**
```
● jenkins.service - Jenkins Continuous Integration Server
   Loaded: loaded (/usr/lib/systemd/system/jenkins.service; enabled; vendor preset: disabled)
   Active: active (running) since Sun 2026-01-12 10:10:00 UTC; 30s ago
 Main PID: 12345 (java)
   CGroup: /docker/xxxx/system.slice/jenkins.service
           └─12345 /usr/bin/java -Djava.awt.headless=true -jar /usr/share/java/jenkins.war --webroot=%C/jenkins/war --httpPort=8080

Jan 12 10:10:00 jenkins systemd[1]: Starting Jenkins Continuous Integration Server...
Jan 12 10:10:00 jenkins jenkins[12345]: Running from: /usr/share/java/jenkins.war
Jan 12 10:10:00 jenkins systemd[1]: Started Jenkins Continuous Integration Server.
Jan 12 10:10:10 jenkins jenkins[12345]: Jenkins initial setup is required. An admin user has been created...
Jan 12 10:10:10 jenkins jenkins[12345]: Please use the following password to proceed to installation:
Jan 12 10:10:10 jenkins jenkins[12345]: a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6
```

**Key indicators:**
- ✅ `Active: active (running)`
- ✅ `Main PID: 12345` (process running)
- ✅ Initial admin password displayed in logs

---

### Step 8: Handle Timeout Issues (If Occurs)

**If you see timeout error:**
```
Job for jenkins.service failed because a timeout was exceeded.
See "systemctl status jenkins.service" and "journalctl -xe" for details.
```

**Troubleshooting steps:**

**1. Check Java memory:**
```bash
free -m
```

**2. Increase systemd timeout:**
```bash
mkdir -p /etc/systemd/system/jenkins.service.d/
cat > /etc/systemd/system/jenkins.service.d/timeout.conf <<EOF
[Service]
TimeoutStartSec=300
EOF
```

**3. Reload systemd and restart:**
```bash
systemctl daemon-reload
systemctl restart jenkins
```

**4. Check logs:**
```bash
journalctl -u jenkins -f
```

**5. Alternative: Adjust Java memory:**
```bash
# Edit Jenkins config
vi /etc/sysconfig/jenkins

# Find JENKINS_JAVA_OPTIONS and modify:
JENKINS_JAVA_OPTIONS="-Djava.awt.headless=true -Xms256m -Xmx512m"

# Restart Jenkins
systemctl restart jenkins
```

---

### Step 9: Verify Jenkins is Running

**Check if Jenkins is listening on port 8080:**
```bash
netstat -tulpn | grep 8080
```

**Expected output:**
```
tcp6       0      0 :::8080                 :::*                    LISTEN      12345/java
```

**Alternative using ss:**
```bash
ss -tlnp | grep 8080
```

**Check Jenkins process:**
```bash
ps aux | grep jenkins
```

**Expected output:**
```
jenkins  12345  1.5 15.2 2345678 123456 ?     Ssl  10:10   0:30 /usr/bin/java -Djava.awt.headless=true -jar /usr/share/java/jenkins.war
```

**Test local connection:**
```bash
curl -I http://localhost:8080
```

**Expected output:**
```
HTTP/1.1 403 Forbidden
X-Content-Type-Options: nosniff
Connection: close
```

**403 is expected (initial setup required)**

---

### Step 10: Get Initial Admin Password

**Jenkins generates a random admin password on first start:**

```bash
cat /var/lib/jenkins/secrets/initialAdminPassword
```

**Expected output (random string):**
```
95c1963370014404a154d974ea585cf3
```

**Copy this password - you'll need it for the web UI setup.**

**Alternative: Check logs:**
```bash
journalctl -u jenkins | grep "password"
```

---

### Step 11: Access Jenkins Web UI

**From the browser, click the "Jenkins" button on the top bar, or navigate to:**
```
http://<jenkins-server-ip>:8080
```

**You'll see the "Unlock Jenkins" page.**

---

### Step 12: Complete Initial Setup Wizard

**Step 1: Unlock Jenkins**
```
┌──────────────────────────────────────────────┐
│         Unlock Jenkins                        │
├──────────────────────────────────────────────┤
│                                              │
│  To ensure Jenkins is securely set up by    │
│  the administrator, a password has been      │
│  written to the log and this file on the    │
│  server:                                     │
│                                              │
│  /var/lib/jenkins/secrets/initialAdminPassword│
│                                              │
│  Administrator password:                     │
│  [________________________________]          │
│                                              │
│              [Continue]                      │
└──────────────────────────────────────────────┘
```

**Action:**
1. Paste the initial admin password from Step 10
2. Click **Continue**

---

**Step 2: Customize Jenkins - Install Plugins**
```
┌──────────────────────────────────────────────┐
│         Customize Jenkins                     │
├──────────────────────────────────────────────┤
│                                              │
│  ┌────────────────────────────────────┐    │
│  │  Install suggested plugins          │    │
│  │  (Recommended for most users)       │    │
│  └────────────────────────────────────┘    │
│                                              │
│  ┌────────────────────────────────────┐    │
│  │  Select plugins to install          │    │
│  │  (Choose specific plugins)          │    │
│  └────────────────────────────────────┘    │
│                                              │
└──────────────────────────────────────────────┘
```

**Action:**
1. Click **Install suggested plugins** (recommended)
2. Wait for plugins to install (2-5 minutes)

**Plugin installation progress:**
```
Getting Started
Installing plugins...

✓ Git plugin
✓ GitHub plugin
✓ Pipeline plugin
✓ Credentials plugin
✓ Docker plugin
✓ SSH Slaves plugin
...
```

---

**Step 3: Create First Admin User**
```
┌──────────────────────────────────────────────┐
│         Create First Admin User               │
├──────────────────────────────────────────────┤
│                                              │
│  Username: [theadmin__________________]     │
│                                              │
│  Password: [Adm!n321__________________]     │
│                                              │
│  Confirm password: [Adm!n321__________]     │
│                                              │
│  Full name: [Siva_____________________]     │
│                                              │
│  E-mail address:                            │
│  [siva@jenkins.stratos.xfusioncorp.com__]   │
│                                              │
│       [Save and Continue]                    │
│                                              │
│       Skip and continue as admin             │
└──────────────────────────────────────────────┘
```

**Action:**
Enter the following details:
- **Username:** `theadmin`
- **Password:** `Adm!n321`
- **Confirm password:** `Adm!n321`
- **Full name:** `Siva`
- **E-mail address:** `siva@jenkins.stratos.xfusioncorp.com`

Click **Save and Continue**

---

**Step 4: Instance Configuration**
```
┌──────────────────────────────────────────────┐
│         Instance Configuration                │
├──────────────────────────────────────────────┤
│                                              │
│  Jenkins URL:                                │
│  [https://8080-port-e6pzduociibi32ej.labs.kodekloud.com/]     │
│                                              │
│  This URL is used to configure webhooks      │
│  and notifications                           │
│                                              │
│       [Save and Finish]                      │
│                                              │
│       Not now                                │
└──────────────────────────────────────────────┘
```

**Action:**
1. Keep the default URL or modify if needed
2. Click **Save and Finish**

---

**Step 5: Jenkins is Ready!**
```
┌──────────────────────────────────────────────┐
│         Jenkins is ready!                     │
├──────────────────────────────────────────────┤
│                                              │
│         ✓ Jenkins is ready!                  │
│                                              │
│         [Start using Jenkins]                │
│                                              │
└──────────────────────────────────────────────┘
```

**Action:**
Click **Start using Jenkins**

---

### Step 13: Verify Admin User Login

**You should now see the Jenkins Dashboard.**

**Verify admin user:**
1. Click on **theadmin** (top right corner)
2. Click **Configure**
3. Verify details:
   - **User ID:** theadmin
   - **Full Name:** Siva
   - **Email Address:** siva@jenkins.stratos.xfusioncorp.com

**Test logout and login:**
1. Click **theadmin** → **Log Out**
2. Enter credentials:
   - **Username:** theadmin
   - **Password:** Adm!n321
3. Click **Sign in**

**Successful login = Task complete! ✅**

---

### Step 14: Verify Jenkins Installation from CLI

**Back on the Jenkins server terminal:**

**Check Jenkins version:**
```bash
curl -s http://localhost:8080/api/json | grep -o '"version":"[^"]*"'
```

**Check Jenkins user database:**
```bash
ls -la /var/lib/jenkins/users/
```

**Expected output:**
```
drwxr-xr-x 3 jenkins jenkins   60 Jan 12 10:20 .
drwxr-xr-x 8 jenkins jenkins 4096 Jan 12 10:15 ..
drwxr-xr-x 2 jenkins jenkins   80 Jan 12 10:20 theadmin_12345678901234567890/
-rw-r--r-- 1 jenkins jenkins  123 Jan 12 10:20 users.xml
```

**Check admin user config:**
```bash
ls -la /var/lib/jenkins/users/theadmin*/
```

**Expected output:**
```
-rw-r--r-- 1 jenkins jenkins 1234 Jan 12 10:20 config.xml
```

**View user config:**
```bash
cat /var/lib/jenkins/users/theadmin*/config.xml
```

**You should see:**
```xml
<user>
  <fullName>Siva</fullName>
  <emailAddress>siva@jenkins.stratos.xfusioncorp.com</emailAddress>
  ...
</user>
```

---

## 📊 Complete Installation Script

**Save and run this script for automated installation:**

```bash
#!/bin/bash

echo "🚀 Day 68: Jenkins Installation and Configuration"
echo "=================================================="

# Colors
GREEN='\033[0;32m'
RED='\033[0;31m'
YELLOW='\033[1;33m'
NC='\033[0m'

# Step 1: Install Java 17
echo -e "\n${YELLOW}1️⃣ Installing Java 17...${NC}"
yum install java-17-openjdk java-17-openjdk-devel -y

if [ $? -eq 0 ]; then
    echo -e "${GREEN}✅ Java 17 installed${NC}"
    java -version
    
    # Set JAVA_HOME
    export JAVA_HOME=/usr/lib/jvm/java-17-openjdk
    export PATH=$JAVA_HOME/bin:$PATH
    echo -e "${GREEN}✅ JAVA_HOME set to ${JAVA_HOME}${NC}"
else
    echo -e "${RED}❌ Failed to install Java${NC}"
    exit 1
fi

# Step 2: Add Jenkins Repository
echo -e "\n${YELLOW}2️⃣ Adding Jenkins repository...${NC}"
curl -L https://pkg.jenkins.io/redhat-stable/jenkins.repo -o /etc/yum.repos.d/jenkins.repo

rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key

if [ $? -eq 0 ]; then
    echo -e "${GREEN}✅ Jenkins repository added${NC}"
else
    echo -e "${RED}❌ Failed to add Jenkins repository${NC}"
    exit 1
fi

# Step 3: Install Jenkins
echo -e "\n${YELLOW}3️⃣ Installing Jenkins...${NC}"
yum install jenkins -y

if [ $? -eq 0 ]; then
    echo -e "${GREEN}✅ Jenkins installed${NC}"
    rpm -qa | grep jenkins
else
    echo -e "${RED}❌ Failed to install Jenkins${NC}"
    exit 1
fi

# Step 4: Configure systemd timeout (prevent timeout issues)
echo -e "\n${YELLOW}4️⃣ Configuring systemd timeout...${NC}"
mkdir -p /etc/systemd/system/jenkins.service.d/
cat > /etc/systemd/system/jenkins.service.d/timeout.conf <<EOF
[Service]
TimeoutStartSec=300
EOF

systemctl daemon-reload
echo -e "${GREEN}✅ Systemd timeout configured${NC}"

# Step 5: Enable and Start Jenkins
echo -e "\n${YELLOW}5️⃣ Starting Jenkins service...${NC}"
systemctl enable jenkins
systemctl start jenkins

# Wait for Jenkins to start
echo -e "${YELLOW}Waiting for Jenkins to start (this may take 1-2 minutes)...${NC}"
sleep 30

# Check service status
systemctl status jenkins --no-pager

if systemctl is-active --quiet jenkins; then
    echo -e "${GREEN}✅ Jenkins service is running${NC}"
else
    echo -e "${RED}❌ Jenkins service failed to start${NC}"
    echo -e "${YELLOW}Checking logs:${NC}"
    journalctl -u jenkins --no-pager -n 50
    exit 1
fi

# Step 6: Verify Jenkins is listening
echo -e "\n${YELLOW}6️⃣ Verifying Jenkins is listening on port 8080...${NC}"
sleep 10

if netstat -tulpn | grep -q 8080; then
    echo -e "${GREEN}✅ Jenkins is listening on port 8080${NC}"
    netstat -tulpn | grep 8080
else
    echo -e "${RED}❌ Jenkins is not listening on port 8080${NC}"
    exit 1
fi

# Step 7: Get Initial Admin Password
echo -e "\n${YELLOW}7️⃣ Retrieving initial admin password...${NC}"
sleep 5

if [ -f /var/lib/jenkins/secrets/initialAdminPassword ]; then
    INIT_PASSWORD=$(cat /var/lib/jenkins/secrets/initialAdminPassword)
    echo -e "${GREEN}✅ Initial admin password:${NC}"
    echo -e "${YELLOW}╔════════════════════════════════════════╗${NC}"
    echo -e "${YELLOW}║  ${INIT_PASSWORD}  ║${NC}"
    echo -e "${YELLOW}╚════════════════════════════════════════╝${NC}"
else
    echo -e "${YELLOW}⚠️  Initial password file not found yet, waiting...${NC}"
    sleep 20
    if [ -f /var/lib/jenkins/secrets/initialAdminPassword ]; then
        INIT_PASSWORD=$(cat /var/lib/jenkins/secrets/initialAdminPassword)
        echo -e "${GREEN}✅ Initial admin password:${NC}"
        echo -e "${YELLOW}╔════════════════════════════════════════╗${NC}"
        echo -e "${YELLOW}║  ${INIT_PASSWORD}  ║${NC}"
        echo -e "${YELLOW}╚════════════════════════════════════════╝${NC}"
    else
        echo -e "${RED}❌ Could not retrieve initial password${NC}"
    fi
fi

# Step 8: Check Jenkins URL
echo -e "\n${YELLOW}8️⃣ Jenkins URL Information...${NC}"
JENKINS_IP=$(hostname -I | awk '{print $1}')
echo -e "${GREEN}Jenkins Web UI: http://${JENKINS_IP}:8080${NC}"
echo -e "${GREEN}Jenkins Web UI: http://jenkins:8080${NC}"

# Final Summary
echo -e "\n${YELLOW}=================================================="
echo "📊 Installation Summary"
echo "==================================================${NC}"

echo -e "\n${GREEN}✅ Java installed:${NC}"
java -version 2>&1 | head -1

echo -e "\n${GREEN}✅ Jenkins installed:${NC}"
rpm -qi jenkins | grep Version

echo -e "\n${GREEN}✅ Jenkins service:${NC}"
systemctl status jenkins --no-pager | grep Active

echo -e "\n${GREEN}✅ Jenkins port:${NC}"
netstat -tulpn | grep 8080

echo -e "\n${YELLOW}=================================================="
echo "📋 Next Steps"
echo "==================================================${NC}"
echo -e "${YELLOW}1. Click 'Jenkins' button on top bar to access UI${NC}"
echo -e "${YELLOW}2. Enter initial admin password (shown above)${NC}"
echo -e "${YELLOW}3. Install suggested plugins${NC}"
echo -e "${YELLOW}4. Create admin user:${NC}"
echo -e "${GREEN}   - Username: theadmin${NC}"
echo -e "${GREEN}   - Password: Adm!n321${NC}"
echo -e "${GREEN}   - Full Name: Siva${NC}"
echo -e "${GREEN}   - Email: siva@jenkins.stratos.xfusioncorp.com${NC}"
echo -e "${YELLOW}5. Complete instance configuration${NC}"

echo -e "\n${GREEN}✅ Jenkins installation completed successfully!${NC}"
```

**Save and run:**
```bash
chmod +x jenkins-install.sh
./jenkins-install.sh
```

---

## 🧹 Cleanup / Uninstall (Optional)

**To remove Jenkins (if needed for testing):**

```bash
# Stop Jenkins service
systemctl stop jenkins

# Disable Jenkins service
systemctl disable jenkins

# Remove Jenkins package
yum remove jenkins -y

# Remove Jenkins data (careful!)
rm -rf /var/lib/jenkins

# Remove Jenkins repository
rm -f /etc/yum.repos.d/jenkins.repo

# Optional: Remove Java
yum remove java-17-openjdk java-17-openjdk-devel -y

# Verify removal
rpm -qa | grep jenkins
# Should return nothing
```

---

## 🐛 Common Issues and Solutions

### Issue 1: Jenkins Service Fails to Start

**Symptoms:**
```bash
systemctl start jenkins
# Job for jenkins.service failed
```

**Diagnosis:**
```bash
systemctl status jenkins
journalctl -u jenkins -n 50
```

**Common causes and solutions:**

**A. Wrong Java Version or Java Not Installed**
```bash
# Check Java version
java -version

# Jenkins 2.528+ requires Java 17 or 21
# If you have Java 11, upgrade to Java 17:
yum remove java-11-openjdk java-11-openjdk-devel -y
yum install java-17-openjdk java-17-openjdk-devel -y

# Set JAVA_HOME
export JAVA_HOME=/usr/lib/jvm/java-17-openjdk
export PATH=$JAVA_HOME/bin:$PATH

# Restart Jenkins
systemctl restart jenkins
```

**B. Port 8080 Already in Use**
```bash
# Check what's using port 8080
netstat -tulpn | grep 8080

# Kill process or change Jenkins port
vi /etc/sysconfig/jenkins
# Change: JENKINS_PORT="8081"

systemctl restart jenkins
```

**C. Insufficient Memory**
```bash
# Check memory
free -m

# Reduce Jenkins memory if needed
vi /etc/sysconfig/jenkins
# Add: JENKINS_JAVA_OPTIONS="-Xms256m -Xmx512m"
```

**D. Permission Issues**
```bash
# Fix Jenkins home permissions
chown -R jenkins:jenkins /var/lib/jenkins

# Restart
systemctl restart jenkins
```

---

### Issue 2: Cannot Access Jenkins UI

**Symptoms:**
```bash
curl http://localhost:8080
# Connection refused
```

**Diagnosis:**
```bash
# Check service
systemctl status jenkins

# Check port
netstat -tulpn | grep 8080

# Check firewall
firewall-cmd --list-ports
```

**Solutions:**

**A. Jenkins Not Started**
```bash
systemctl start jenkins
systemctl status jenkins
```

**B. Firewall Blocking Port**
```bash
# Open port 8080
firewall-cmd --permanent --add-port=8080/tcp
firewall-cmd --reload

# Verify
firewall-cmd --list-ports
```

**C. SELinux Blocking**
```bash
# Check SELinux status
getenforce

# Temporarily disable (testing only)
setenforce 0

# Permanent (not recommended for production)
vi /etc/selinux/config
# SELINUX=permissive
```

---

### Issue 3: Initial Admin Password Not Found

**Symptoms:**
```bash
cat /var/lib/jenkins/secrets/initialAdminPassword
# No such file or directory
```

**Diagnosis:**
```bash
# Check if Jenkins finished initializing
systemctl status jenkins
journalctl -u jenkins | grep "Jenkins is fully up"
```

**Solution:**
```bash
# Wait for Jenkins to fully initialize (1-2 minutes)
sleep 60

# Check again
cat /var/lib/jenkins/secrets/initialAdminPassword

# Alternative: Check logs
journalctl -u jenkins | grep password
```

---

### Issue 4: Plugin Installation Fails

**Symptoms:**
During setup wizard, plugins fail to install

**Diagnosis:**
- No internet connection
- Proxy configuration needed
- Disk space full

**Solutions:**

**A. Check Internet Connection**
```bash
ping -c 3 google.com
curl -I https://updates.jenkins.io
```

**B. Configure Proxy (if needed)**
```bash
vi /etc/sysconfig/jenkins

# Add proxy settings
JENKINS_JAVA_OPTIONS="-Djava.awt.headless=true -Dhttp.proxyHost=proxy.example.com -Dhttp.proxyPort=8080"

systemctl restart jenkins
```

**C. Check Disk Space**
```bash
df -h /var/lib/jenkins
# Ensure enough space available
```

**D. Skip Plugins and Install Later**
- Click "Skip and continue as admin"
- Install plugins later from **Manage Jenkins → Manage Plugins**

---

### Issue 5: "Offline" Error During Setup

**Symptoms:**
Setup wizard shows "Offline" or "No internet connection"

**Diagnosis:**
```bash
# Test connectivity from Jenkins
curl https://updates.jenkins.io/update-center.json
```

**Solution:**

**Option 1: Wait and Retry**
```bash
# Jenkins may still be initializing
# Wait 2-3 minutes and refresh page
```

**Option 2: Manually Configure Update Center**
```bash
# Access Jenkins CLI
cd /var/lib/jenkins

# Download update center manually
wget https://updates.jenkins.io/update-center.json

# Restart Jenkins
systemctl restart jenkins
```

**Option 3: Change Update Site**
1. In Jenkins UI: **Manage Jenkins → Manage Plugins**
2. Click **Advanced** tab
3. Update Site URL: `https://updates.jenkins.io/update-center.json`
4. Click **Submit** and **Check Now**

---

## 📖 Key Takeaways

### Concepts Mastered Today:

1. ✅ **Jenkins Installation**
   - Added Jenkins repository
   - Installed via yum package manager
   - Configured Java dependency
   - Started Jenkins service

2. ✅ **System Administration**
   - systemctl service management
   - Repository configuration
   - Package installation with yum
   - Service troubleshooting

3. ✅ **Jenkins Architecture**
   - Master server on port 8080
   - Initial setup wizard
   - Plugin system
   - User authentication

4. ✅ **Security Configuration**
   - Initial admin password
   - Custom admin user creation
   - Email configuration
   - Authentication system

5. ✅ **CI/CD Fundamentals**
   - Continuous Integration concepts
   - Automation server purpose
   - Jenkins use cases
   - Pipeline basics

---

## 🎯 Real-World Lessons

### Production Jenkins Setup

**Current Setup (Basic):**
```yaml
Single Jenkins Master:
- Installed on single server
- Default configuration
- Basic authentication
- No agents
```

**Production Setup:**
```yaml
High Availability Setup:
- Jenkins Master (Active)
- Jenkins Master (Standby)
- Multiple Agents (Linux, Windows, Docker)
- Shared Storage for JENKINS_HOME
- Load Balancer
- Backup and Restore procedures

Security:
- LDAP/Active Directory integration
- Role-Based Access Control (RBAC)
- SSL/TLS certificates
- Security plugins (OWASP, Audit)
- Secrets management (HashiCorp Vault)

Scalability:
- Distributed builds
- Docker-based agents
- Kubernetes plugin for dynamic agents
- Resource monitoring

Backup:
- Daily JENKINS_HOME backups
- Configuration as Code (JCasC)
- Job definitions in Git
- Plugin list versioning
```

---

### Best Practices

1. **Regular Backups:**
   ```bash
   # Backup Jenkins home
   tar -czf jenkins-backup-$(date +%Y%m%d).tar.gz /var/lib/jenkins/
   ```

2. **Configuration as Code:**
   ```yaml
   # Use JCasC plugin
   # jenkins.yaml
   jenkins:
     systemMessage: "Welcome to xFusionCorp Jenkins"
     numExecutors: 2
     securityRealm:
       local:
         users:
          - id: "theadmin"
            name: "Siva"
   ```

3. **Update Management:**
   ```bash
   # Regular updates
   # Manage Jenkins → Manage Plugins → Updates
   # Test in staging first
   ```

4. **Resource Monitoring:**
   ```bash
   # Monitor disk space
   df -h /var/lib/jenkins

   # Monitor memory
   free -m

   # Monitor Jenkins logs
   tail -f /var/log/jenkins/jenkins.log
   ```

5. **Security Hardening:**
   - Enable CSRF protection
   - Disable CLI over remoting
   - Use HTTPS (SSL/TLS)
   - Regular security audits
   - Principle of least privilege

---

## 📚 Additional Resources

**Official Documentation:**
- [Jenkins Installation Guide](https://www.jenkins.io/doc/book/installing/)
- [Jenkins User Handbook](https://www.jenkins.io/doc/book/)
- [Jenkins Plugins](https://plugins.jenkins.io/)
- [Jenkins Security](https://www.jenkins.io/doc/book/security/)

**Tutorials:**
- [Jenkins Pipeline Tutorial](https://www.jenkins.io/doc/book/pipeline/)
- [Jenkins with Docker](https://www.jenkins.io/doc/book/installing/docker/)
- [Jenkins Configuration as Code](https://github.com/jenkinsci/configuration-as-code-plugin)

**Community:**
- [Jenkins Community Chat](https://www.jenkins.io/chat/)
- [Jenkins Mailing Lists](https://www.jenkins.io/mailing-lists/)
- [Jenkins JIRA](https://issues.jenkins.io/)

**Next Steps:**
- **Day 69:** Creating First Jenkins Pipeline
- **Day 70:** Jenkins Agents and Distributed Builds
- **Day 71:** Jenkins with Docker Integration
- **Day 72:** Jenkins Pipeline as Code (Jenkinsfile)

---

## ✅ Task Completion Checklist

**Installation:**
- [ ] Connected to Jenkins server (root@jenkins)
- [ ] Installed Java 17 using yum
- [ ] Added Jenkins repository (using curl)
- [ ] Imported Jenkins GPG key
- [ ] Installed Jenkins using yum
- [ ] Verified Jenkins package installed

**Service Configuration:**
- [ ] Enabled Jenkins service
- [ ] Started Jenkins service
- [ ] Verified service is active and running
- [ ] Confirmed Jenkins listening on port 8080
- [ ] Retrieved initial admin password

**Web UI Configuration:**
- [ ] Accessed Jenkins UI via browser
- [ ] Entered initial admin password
- [ ] Installed suggested plugins
- [ ] Created admin user with credentials:
  - Username: `theadmin`
  - Password: `Adm!n321`
  - Full Name: `Siva`
  - Email: `siva@jenkins.stratos.xfusioncorp.com`
- [ ] Completed instance configuration
- [ ] Successfully logged in as theadmin

**Verification:**
- [ ] Jenkins dashboard accessible
- [ ] Admin user profile shows correct information
- [ ] Can logout and login with new credentials
- [ ] Service survives reboot (systemctl enable)

---

**🎉 Congratulations!** You've successfully installed and configured Jenkins on a Linux server! You now have a fully functional CI/CD automation server ready for creating pipelines and automating your software delivery process.

**Day 68 Status:** ✅ Complete

**Next:** Day 69 - Creating Your First Jenkins Pipeline 🚀
