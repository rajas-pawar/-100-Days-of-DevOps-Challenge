# Day 40: Configuring Apache in Docker Container
**100 Days Cloud DevOps Challenge – KodeKloud**

---

## Objective

Complete pending work on a running Docker container by installing and configuring Apache web server with custom port configuration.

**Requirements:**
1. Work on Application Server 1 (App Server 1)
2. Container: `kkloud` (running)
3. Install: `apache2` using apt
4. Configure: Apache to listen on port `5002` (not default 80)
5. Bind: Listen on all interfaces (not specific IP)
6. Service: Apache must be running
7. State: Keep container running

---

## Understanding Container Configuration

**Container Service Configuration** involves modifying running containers to install and configure services. This is different from building images - we're making live changes to an existing container.

### What is Container Configuration?

Container configuration is the process of:
- Installing packages in running containers
- Modifying configuration files
- Starting services
- Ensuring persistence of running services

### Configuration Flow:

```
Running Container (kkloud)
         ↓
    Access Container
         ↓
    Install apache2
         ↓
    Configure Port 5002
         ↓
    Start Apache Service
         ↓
    Verify Service Running
         ↓
    Keep Container Running
```

### Why Configure Running Containers?

- **Quick Development** - Fast iteration and testing
- **Debugging** - Fix issues in live containers
- **Learning** - Understand service configuration
- **Temporary Changes** - Test before building into image
- **Emergency Fixes** - Hotfix production issues

### Container vs Host Configuration:

| Aspect | Host Configuration | Container Configuration |
|--------|-------------------|------------------------|
| **Persistence** | Permanent on disk | Lost if container deleted |
| **Isolation** | System-wide | Container-specific |
| **Impact** | Affects entire system | Only affects container |
| **Service Management** | systemctl/service | Manual start or supervisor |
| **Network** | Host networking | Container networking |

---

## Understanding Apache Configuration

### Apache Web Server:

Apache HTTP Server (apache2) is one of the most popular web servers in the world. It serves web content over HTTP/HTTPS.

### Default Apache Configuration:

```
Default Port: 80 (HTTP)
Default SSL Port: 443 (HTTPS)
Config Location: /etc/apache2/
Main Config: /etc/apache2/apache2.conf
Ports Config: /etc/apache2/ports.conf
Sites: /etc/apache2/sites-available/
```

### Port Configuration:

**Why Change Default Port?**
- Avoid port conflicts
- Security through obscurity
- Run multiple web servers
- Comply with network policies
- Testing and development

**Port Binding Options:**

```bash
# Listen on all interfaces, port 5002
Listen 5002

# Listen on specific IP only
Listen 192.168.1.10:5002

# Listen on localhost only
Listen 127.0.0.1:5002

# Listen on all interfaces (default)
Listen *:5002
```

**Task Requirement:** `Listen 5002` (all interfaces)

---

## Understanding the Scenario

### Team Workflow:

```
1. Team member started container configuration
2. Container 'kkloud' is running
3. Work incomplete - apache2 not installed/configured
4. Team member on PTO (Personal Time Off)
5. Need to complete:
   - Install apache2
   - Configure port 5002
   - Start Apache
   - Keep running
```

### Current State:

```
App Server 1
│
├── Container: kkloud (running)
│   ├── Status: Running but incomplete
│   ├── Apache: Not installed
│   └── Port 5002: Not configured
│
└── Task: Complete apache2 setup
```

### Target State:

```
App Server 1
│
├── Container: kkloud (running)
│   ├── Apache2: Installed ✅
│   ├── Port: 5002 configured ✅
│   ├── Service: Running ✅
│   └── Listening: 0.0.0.0:5002 ✅
```

---

## Infrastructure Overview

### Application Servers:
| Server | User | Password | IP |
|--------|------|----------|-----|
| **stapp01** | **tony** | **Ir0nM@n** | **172.16.238.10** |
| stapp02 | steve | Am3ric@ | 172.16.238.11 |
| stapp03 | banner | BigGr33n | 172.16.238.12 |

### Task Details:
| Item | Value |
|------|-------|
| Server | Application Server 1 (stapp01) |
| User | tony |
| Container | `kkloud` (running) |
| Package | apache2 |
| Package Manager | apt |
| Port | 5002 |
| Listen Address | All interfaces (0.0.0.0) |
| Service State | Running |
| Container State | Running |

---

## Understanding the Task

### What We're Configuring:

```
Application Server 1 (stapp01)
│
├── Running Container: kkloud
│   │
│   ├── Install apache2
│   │   └── apt install apache2
│   │
│   ├── Configure Port
│   │   └── /etc/apache2/ports.conf
│   │   └── Listen 5002
│   │
│   ├── Update VirtualHost (if needed)
│   │   └── /etc/apache2/sites-available/000-default.conf
│   │   └── <VirtualHost *:5002>
│   │
│   └── Start Apache
│       └── apache2 service running
│
└── Verification
    ├── Container running
    └── Apache listening on 5002
```

### Configuration Steps Flow:

```
1. SSH to App Server 1
2. Switch to root
3. Verify kkloud container running
4. Access container shell
5. Update apt repositories
6. Install apache2
7. Configure ports.conf → 5002
8. Configure VirtualHost → *:5002
9. Start apache2 service
10. Verify service running
11. Test port 5002 listening
12. Keep container running (don't exit incorrectly)
```

---

## Step-by-Step Implementation

### Step 1: SSH into Application Server 1
```bash
ssh tony@stapp01
```

**Expected output:**
```
The authenticity of host 'stapp01 (172.16.238.10)' can't be established.
ED25519 key fingerprint is SHA256:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'stapp01' (ED25519) to the list of known hosts.
tony@stapp01's password:
```

**Enter password:** `Ir0nM@n`

```
[tony@stapp01 ~]$
```

### Step 2: Switch to Root User
```bash
sudo su -
```

**Expected output:**
```
We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

[sudo] password for tony:
```

**Enter password:** `Ir0nM@n`

```
[root@stapp01 ~]#
```

### Step 3: Verify Container is Running
```bash
docker ps
```

**Expected output:**
```
CONTAINER ID   IMAGE     COMMAND       CREATED          STATUS          PORTS     NAMES
a1b2c3d4e5f6   ubuntu    "/bin/bash"   15 minutes ago   Up 15 minutes             kkloud
```

**Verify:**
- ✅ Container `kkloud` is present
- ✅ STATUS shows "Up" (running)
- ✅ Note the CONTAINER ID

**Alternative check:**
```bash
docker ps --filter "name=kkloud"
```

### Step 4: Check Container Details
```bash
docker inspect kkloud
```

**Expected output (partial):**
```json
[
    {
        "Id": "a1b2c3d4e5f6...",
        "Name": "/kkloud",
        "State": {
            "Status": "running",
            "Running": true
        },
        "Config": {
            "Image": "ubuntu",
            "Cmd": ["/bin/bash"]
        }
    }
]
```

### Step 5: Access Container Shell
```bash
docker exec -it kkloud /bin/bash
```

**Expected output:**
```
root@a1b2c3d4e5f6:/#
```

**You're now inside the container!**

**Verify you're in the container:**
```bash
hostname
```

**Output:** Container ID (like `a1b2c3d4e5f6`)

### Step 6: Update Package Repository
```bash
apt update
```

**Expected output:**
```
Get:1 http://archive.ubuntu.com/ubuntu jammy InRelease [270 kB]
Get:2 http://security.ubuntu.com/ubuntu jammy-security InRelease [129 kB]
Get:3 http://archive.ubuntu.com/ubuntu jammy-updates InRelease [128 kB]
...
Fetched 25.4 MB in 8s (3,234 kB/s)
Reading package lists... Done
Building dependency tree... Done
All packages are up to date.
```

**This updates the apt package index**

### Step 7: Install Apache2
```bash
apt install apache2 -y
```

**Expected output:**
```
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  apache2-bin apache2-data apache2-utils libapr1 libaprutil1 ...
Suggested packages:
  apache2-doc apache2-suexec-pristine | apache2-suexec-custom ...
The following NEW packages will be installed:
  apache2 apache2-bin apache2-data apache2-utils libapr1 ...
0 upgraded, 23 newly installed, 0 to remove and 0 not upgraded.
Need to get 6,234 kB of archives.
After this operation, 25.6 MB of additional disk space will be used.
...
Setting up apache2 (2.4.52-1ubuntu4) ...
Enabling module mpm_event.
Enabling module authz_core.
Enabling module authz_host.
...
```

**Installation complete!**

### Step 8: Verify Apache Installed
```bash
apache2 -v
```

**Expected output:**
```
Server version: Apache/2.4.52 (Ubuntu)
Server built:   2024-10-08T15:23:17
```

**Check Apache binary location:**
```bash
which apache2
```

**Output:** `/usr/sbin/apache2`

### Step 9: Check Apache Configuration Files
```bash
ls -la /etc/apache2/
```

**Expected output:**
```
total 84
drwxr-xr-x  8 root root  4096 Dec 15 10:30 .
drwxr-xr-x 90 root root  4096 Dec 15 10:30 ..
-rw-r--r--  1 root root  7224 Oct  8 15:23 apache2.conf
drwxr-xr-x  2 root root  4096 Dec 15 10:30 conf-available
drwxr-xr-x  2 root root  4096 Dec 15 10:30 conf-enabled
-rw-r--r--  1 root root  1782 Oct  8 15:23 envvars
-rw-r--r--  1 root root 31063 Oct  8 15:23 magic
drwxr-xr-x  2 root root 12288 Dec 15 10:30 mods-available
drwxr-xr-x  2 root root  4096 Dec 15 10:30 mods-enabled
-rw-r--r--  1 root root   320 Oct  8 15:23 ports.conf
drwxr-xr-x  2 root root  4096 Dec 15 10:30 sites-available
drwxr-xr-x  2 root root  4096 Dec 15 10:30 sites-enabled
```

### Step 10: View Current Ports Configuration
```bash
cat /etc/apache2/ports.conf
```

**Expected output:**
```
# If you just change the port or add more ports here, you will likely also
# have to change the VirtualHost statement in
# /etc/apache2/sites-enabled/000-default.conf

Listen 80

<IfModule ssl_module>
        Listen 443
</IfModule>

<IfModule mod_gnutls.c>
        Listen 443
</IfModule>
```

**Currently listening on port 80 - we need to change to 5002**

### Step 11: Configure Apache to Listen on Port 5002
```bash
sed -i 's/Listen 80/Listen 5002/g' /etc/apache2/ports.conf
```

**Verify the change:**
```bash
cat /etc/apache2/ports.conf
```

**Expected output:**
```
# If you just change the port or add more ports here, you will likely also
# have to change the VirtualHost statement in
# /etc/apache2/sites-enabled/000-default.conf

Listen 5002

<IfModule ssl_module>
        Listen 443
</IfModule>

<IfModule mod_gnutls.c>
        Listen 443
</IfModule>
```

**✅ Port changed to 5002**

**Alternative method (manual edit):**
```bash
# Using vi or nano
vi /etc/apache2/ports.conf
# Change: Listen 80 → Listen 5002
# Save and exit
```

### Step 12: Check Default VirtualHost Configuration
```bash
cat /etc/apache2/sites-available/000-default.conf
```

**Expected output:**
```
<VirtualHost *:80>
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/html

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

**Notice `<VirtualHost *:80>` - needs to match port 5002**

### Step 13: Update VirtualHost to Port 5002
```bash
sed -i 's/<VirtualHost \*:80>/<VirtualHost *:5002>/g' /etc/apache2/sites-available/000-default.conf
```

**Verify the change:**
```bash
cat /etc/apache2/sites-available/000-default.conf
```

**Expected output:**
```
<VirtualHost *:5002>
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/html

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

**✅ VirtualHost updated to port 5002**

### Step 14: Test Apache Configuration
```bash
apache2ctl configtest
```

**Expected output:**
```
AH00558: apache2: Could not reliably determine the server's fully qualified domain name, using 172.17.0.2. Set the 'ServerName' directive globally to suppress this message
Syntax OK
```

**"Syntax OK" means configuration is valid!**

**The warning about ServerName is normal and can be ignored**

### Step 15: Start Apache Service

**Note:** In containers, systemctl often doesn't work. We start Apache directly.

```bash
service apache2 start
```

**Expected output:**
```
 * Starting Apache httpd web server apache2
 * 
```

**Alternative method:**
```bash
apache2ctl start
```

**Or directly:**
```bash
/usr/sbin/apache2ctl -D FOREGROUND &
```

### Step 16: Check Apache Service Status
```bash
service apache2 status
```

**Expected output:**
```
 * apache2 is running
```

**Alternative check:**
```bash
ps aux | grep apache2
```

**Expected output:**
```
root         123  0.0  0.5  12345  12345 ?        Ss   10:35   0:00 /usr/sbin/apache2 -k start
www-data     124  0.0  0.3  12345  12345 ?        S    10:35   0:00 /usr/sbin/apache2 -k start
www-data     125  0.0  0.3  12345  12345 ?        S    10:35   0:00 /usr/sbin/apache2 -k start
```

**✅ Apache processes running**

### Step 17: Verify Apache Listening on Port 5002
```bash
netstat -tuln | grep 5002
```

**Expected output:**
```
tcp        0      0 0.0.0.0:5002            0.0.0.0:*               LISTEN
```

**This confirms Apache is listening on port 5002 on all interfaces (0.0.0.0)**

**If netstat is not available, install it:**
```bash
apt install net-tools -y
netstat -tuln | grep 5002
```

**Alternative check with ss:**
```bash
ss -tuln | grep 5002
```

**Expected output:**
```
tcp   LISTEN 0      511          0.0.0.0:5002       0.0.0.0:*
```

### Step 18: Test Apache from Inside Container
```bash
curl http://localhost:5002
```

**Expected output (partial HTML):**
```
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" ...>
<html xmlns="http://www.w3.org/1999/xhtml">
  <head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
    <title>Apache2 Ubuntu Default Page: It works</title>
    ...
  </head>
  <body>
    <div class="main_page">
      <div class="page_header floating_element">
        <img src="/icons/ubuntu-logo.png" alt="Ubuntu Logo" />
        <span class="floating_element">
          Apache2 Default Page
        </span>
      </div>
      ...
```

**✅ Apache is serving web pages!**

**Alternative test:**
```bash
curl -I http://localhost:5002
```

**Expected output:**
```
HTTP/1.1 200 OK
Date: Sun, 15 Dec 2025 10:35:42 GMT
Server: Apache/2.4.52 (Ubuntu)
Last-Modified: Sun, 15 Dec 2025 10:30:15 GMT
Content-Type: text/html
```

### Step 19: Check Apache Error Log (Optional)
```bash
tail -f /var/log/apache2/error.log
```

**Should show Apache started successfully**

**Press Ctrl+C to exit**

### Step 20: Keep Container Running - Exit Properly

**⚠️ IMPORTANT:** We need to keep the container running AND Apache service running.

**Problem:** When you exit the container shell, Apache might stop if container's main process exits.

**Solution: Exit without stopping the container**

```bash
# Press Ctrl+P then Ctrl+Q to detach
# OR if that doesn't work, just exit normally
exit
```

**You're back on the host:**
```
[root@stapp01 ~]#
```

### Step 21: Verify Container Still Running
```bash
docker ps | grep kkloud
```

**Expected output:**
```
CONTAINER ID   IMAGE     COMMAND       CREATED          STATUS          PORTS     NAMES
a1b2c3d4e5f6   ubuntu    "/bin/bash"   30 minutes ago   Up 30 minutes             kkloud
```

**✅ Container still running**

### Step 22: Verify Apache Still Running in Container
```bash
docker exec kkloud service apache2 status
```

**Expected output:**
```
 * apache2 is running
```

**Alternative check:**
```bash
docker exec kkloud ps aux | grep apache2
```

**Expected output:**
```
root         123  0.0  0.5  12345  12345 ?        Ss   10:35   0:00 /usr/sbin/apache2 -k start
www-data     124  0.0  0.3  12345  12345 ?        S    10:35   0:00 /usr/sbin/apache2 -k start
```

### Step 23: Test Apache from Host
```bash
docker exec kkloud curl -I http://localhost:5002
```

**Expected output:**
```
HTTP/1.1 200 OK
Date: Sun, 15 Dec 2025 10:40:00 GMT
Server: Apache/2.4.52 (Ubuntu)
Content-Type: text/html
```

**✅ Apache responding on port 5002!**

### Step 24: Check Port Listening from Host
```bash
docker exec kkloud netstat -tuln | grep 5002
```

**Expected output:**
```
tcp        0      0 0.0.0.0:5002            0.0.0.0:*               LISTEN
```

**Perfect! ✅ All requirements met:**
- ✅ apache2 installed
- ✅ Configured to listen on port 5002
- ✅ Listening on all interfaces (0.0.0.0)
- ✅ Apache service running
- ✅ Container running

### Step 25: Final Verification (Optional)
```bash
# Check container is up
docker ps --filter "name=kkloud" --format "{{.Status}}"

# Check Apache version in container
docker exec kkloud apache2 -v

# Check listening ports in container
docker exec kkloud ss -tuln | grep LISTEN

# Test web page
docker exec kkloud curl -s http://localhost:5002 | head -20
```

---

## Complete Command Summary

### Quick Configuration Flow:
```bash
# SSH and access
ssh tony@stapp01
sudo su -

# Access container
docker exec -it kkloud /bin/bash

# Install Apache
apt update
apt install apache2 -y

# Configure port 5002
sed -i 's/Listen 80/Listen 5002/g' /etc/apache2/ports.conf
sed -i 's/<VirtualHost \*:80>/<VirtualHost *:5002>/g' /etc/apache2/sites-available/000-default.conf

# Start Apache
service apache2 start

# Verify
service apache2 status
netstat -tuln | grep 5002
curl http://localhost:5002

# Exit container
exit

# Verify from host
docker ps | grep kkloud
docker exec kkloud service apache2 status
```

### Complete Command Sequence:
```bash
# 1. SSH to server
ssh tony@stapp01
sudo su -

# 2. Verify container
docker ps

# 3. Access container
docker exec -it kkloud /bin/bash

# 4. Inside container: Update and install
apt update
apt install apache2 -y

# 5. Configure ports
sed -i 's/Listen 80/Listen 5002/g' /etc/apache2/ports.conf
sed -i 's/<VirtualHost \*:80>/<VirtualHost *:5002>/g' /etc/apache2/sites-available/000-default.conf

# 6. Test configuration
apache2ctl configtest

# 7. Start Apache
service apache2 start

# 8. Verify running
service apache2 status
netstat -tuln | grep 5002
curl -I http://localhost:5002

# 9. Exit container properly
exit

# 10. Verify from host
docker ps | grep kkloud
docker exec kkloud service apache2 status
docker exec kkloud netstat -tuln | grep 5002
```

---

## Apache in Containers - Deep Dive

### Why Apache Service Management is Different in Containers:

**Host System:**
```bash
# systemctl works
systemctl start apache2
systemctl status apache2
systemctl enable apache2  # Auto-start on boot
```

**Container:**
```bash
# systemctl often doesn't work
# Use service command or direct binary
service apache2 start
apache2ctl start
/usr/sbin/apache2ctl -D FOREGROUND &
```

### Container Process Management:

```
Container Lifecycle:
├── Main Process (PID 1)
│   └── Usually /bin/bash
├── Child Processes
│   └── Apache processes
└── When PID 1 exits → Container stops
```

### Keeping Apache Running in Container:

**Method 1: Keep Shell Open (Background Apache)**
```bash
docker exec -d kkloud service apache2 start
# -d runs in detached mode
```

**Method 2: Apache as Foreground Process**
```bash
docker exec -d kkloud apache2ctl -D FOREGROUND
```

**Method 3: Proper Container Design**
```dockerfile
# In Dockerfile
CMD ["apache2ctl", "-D", "FOREGROUND"]
# Apache runs as main process
```

---

## Understanding Apache Ports Configuration

### Files Involved:

**1. ports.conf - Port Binding**
```
Location: /etc/apache2/ports.conf
Purpose: Define which ports Apache listens on
```

**2. VirtualHost - Site Configuration**
```
Location: /etc/apache2/sites-available/000-default.conf
Purpose: Define how to handle requests on those ports
```

### Why Both Need to Change:

```
ports.conf          → Apache binds to port 5002
                      (Opens the port)
                      
VirtualHost *:5002  → Apache knows what to do with
                      requests on port 5002
                      (Handles the requests)
```

### Listen Directive Options:

**All Interfaces (Our Task):**
```apache
Listen 5002
# Binds to 0.0.0.0:5002 (all interfaces)
```

**Specific IP:**
```apache
Listen 172.17.0.2:5002
# Only this IP
```

**Localhost Only:**
```apache
Listen 127.0.0.1:5002
# Only accessible from inside container
```

**Multiple Ports:**
```apache
Listen 80
Listen 5002
Listen 8080
# Apache listens on all three
```

---

## Troubleshooting

### Issue 1: Container Not Found

**Problem:**
```
Error: No such container: kkloud
```

**Solution:**
```bash
# List all containers
docker ps -a

# Check exact name
docker ps -a --format "{{.Names}}"

# If stopped, start it
docker start kkloud

# If doesn't exist, check task requirements
```

### Issue 2: apt update Fails

**Problem:**
```
E: Could not open lock file /var/lib/apt/lists/lock
```

**Solution:**
```bash
# Remove lock files
rm /var/lib/apt/lists/lock
rm /var/cache/apt/archives/lock

# Try again
apt update
```

### Issue 3: Apache Won't Start

**Problem:**
```
 * Starting Apache httpd web server apache2                              [fail]
```

**Check logs:**
```bash
tail -50 /var/log/apache2/error.log
```

**Common causes:**
```bash
# Port already in use
netstat -tuln | grep 5002
# Kill conflicting process if found

# Configuration error
apache2ctl configtest

# Missing directories
mkdir -p /var/run/apache2
mkdir -p /var/lock/apache2
```

### Issue 4: Port Already in Use

**Problem:**
```
(98)Address already in use: AH00072: make_sock: could not bind to address 0.0.0.0:5002
```

**Solution:**
```bash
# Find what's using port 5002
netstat -tuln | grep 5002
lsof -i :5002

# Kill the process
kill <PID>

# Start Apache again
service apache2 start
```

### Issue 5: Apache Stops When Exiting Container

**Problem:**
Container exits when you type `exit`, stopping Apache.

**Solution:**
```bash
# Method 1: Detach without exiting
# Press Ctrl+P, then Ctrl+Q

# Method 2: Start Apache in background
docker exec -d kkloud service apache2 start

# Method 3: Restart if stopped
docker start kkloud
docker exec kkloud service apache2 start
```

### Issue 6: netstat Command Not Found

**Problem:**
```
bash: netstat: command not found
```

**Solution:**
```bash
# Install net-tools
apt install net-tools -y

# Then use netstat
netstat -tuln | grep 5002

# Alternative: use ss (usually pre-installed)
ss -tuln | grep 5002
```

### Issue 7: curl Command Not Found

**Problem:**
```
bash: curl: command not found
```

**Solution:**
```bash
# Install curl
apt install curl -y

# Test Apache
curl http://localhost:5002
```

### Issue 8: Permission Denied

**Problem:**
```
Permission denied
```

**Solution:**
```bash
# Ensure you're root in container
whoami
# Should show: root

# If not root
docker exec -u root -it kkloud /bin/bash
```

### Issue 9: Configuration Test Fails

**Problem:**
```
apache2ctl configtest
AH00526: Syntax error on line 2 of /etc/apache2/ports.conf:
```

**Solution:**
```bash
# Check exact syntax
cat /etc/apache2/ports.conf

# Ensure it's exactly:
Listen 5002

# Not:
Listen: 5002  # Wrong (colon)
listen 5002   # Wrong (lowercase)
Listen5002    # Wrong (no space)
```

---

## Best Practices

### 1. Always Update Package Lists First
```bash
# Before installing anything
apt update

# Then install
apt install apache2 -y
```

### 2. Use sed for Reliable Configuration Changes
```bash
# Better than manual editing
sed -i 's/Listen 80/Listen 5002/g' /etc/apache2/ports.conf

# Verify the change
cat /etc/apache2/ports.conf | grep Listen
```

### 3. Test Configuration Before Starting Service
```bash
# Check for syntax errors
apache2ctl configtest

# Only start if "Syntax OK"
if apache2ctl configtest; then
    service apache2 start
fi
```

### 4. Verify Service Status
```bash
# Check service
service apache2 status

# Check processes
ps aux | grep apache2

# Check port listening
netstat -tuln | grep 5002

# Test HTTP response
curl -I http://localhost:5002
```

### 5. Keep Container Running
```bash
# Don't just exit - ensure Apache keeps running
service apache2 start

# Exit properly
exit

# Verify from host
docker exec kkloud service apache2 status
```

### 6. Document Port Changes
```bash
# Add comment in configuration
echo "# Changed to port 5002 for task requirements" >> /etc/apache2/ports.conf
```

### 7. Use Detached Mode for Services
```bash
# Start services in background
docker exec -d kkloud service apache2 start

# Container stays up, service runs
```

---

## Real-World Scenarios

### Scenario 1: Multi-Port Apache Configuration
```bash
# Configure Apache on multiple ports
cat >> /etc/apache2/ports.conf <<EOF
Listen 5002
Listen 5003
Listen 5004
EOF

# Create VirtualHost for each port
cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/port-5002.conf
cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/port-5003.conf

# Edit each VirtualHost
sed -i 's/*:80/*:5002/g' /etc/apache2/sites-available/port-5002.conf
sed -i 's/*:80/*:5003/g' /etc/apache2/sites-available/port-5003.conf

# Enable sites
a2ensite port-5002.conf
a2ensite port-5003.conf

# Restart Apache
service apache2 restart
```

### Scenario 2: Apache with Custom Content
```bash
# Install Apache
apt install apache2 -y

# Create custom page
cat > /var/www/html/index.html <<EOF
<!DOCTYPE html>
<html>
<head>
    <title>Nautilus Project</title>
</head>
<body>
    <h1>Welcome to Nautilus DevOps!</h1>
    <p>Apache running on port 5002</p>
</body>
</html>
EOF

# Configure port
sed -i 's/Listen 80/Listen 5002/g' /etc/apache2/ports.conf
sed -i 's/*:80/*:5002/g' /etc/apache2/sites-available/000-default.conf

# Start Apache
service apache2 start

# Test
curl http://localhost:5002
```

### Scenario 3: Apache with SSL on Custom Port
```bash
# Install Apache
apt install apache2 -y

# Enable SSL module
a2enmod ssl

# Configure HTTPS port
cat >> /etc/apache2/ports.conf <<EOF
Listen 5443
EOF

# Copy default SSL site
cp /etc/apache2/sites-available/default-ssl.conf /etc/apache2/sites-available/ssl-5443.conf

# Update port
sed -i 's/*:443/*:5443/g' /etc/apache2/sites-available/ssl-5443.conf

# Enable site
a2ensite ssl-5443.conf

# Restart
service apache2 restart
```

### Scenario 4: Apache Behind Reverse Proxy
```bash
# Configure Apache on non-standard port
sed -i 's/Listen 80/Listen 8080/g' /etc/apache2/ports.conf

# Enable proxy modules
a2enmod proxy
a2enmod proxy_http

# Start Apache
service apache2 start

# Nginx or another proxy forwards to 8080
```

### Scenario 5: Container as Web Server Platform
```bash
# Install Apache + PHP
apt update
apt install apache2 php libapache2-mod-php -y

# Configure port
sed -i 's/Listen 80/Listen 5002/g' /etc/apache2/ports.conf
sed -i 's/*:80/*:5002/g' /etc/apache2/sites-available/000-default.conf

# Create PHP test page
cat > /var/www/html/info.php <<EOF
<?php
phpinfo();
?>
EOF

# Start Apache
service apache2 start

# Test
curl http://localhost:5002/info.php
```

---

## Key Commands Reference

| Command | Description |
|---------|-------------|
| `docker exec -it CONTAINER /bin/bash` | Access container shell |
| `apt update` | Update package lists |
| `apt install apache2 -y` | Install Apache web server |
| `apache2 -v` | Check Apache version |
| `apache2ctl configtest` | Test configuration syntax |
| `service apache2 start` | Start Apache service |
| `service apache2 status` | Check Apache status |
| `service apache2 restart` | Restart Apache |
| `netstat -tuln | grep PORT` | Check listening ports |
| `ss -tuln | grep PORT` | Alternative port check |
| `curl http://localhost:PORT` | Test web server |
| `ps aux | grep apache2` | Check Apache processes |
| `cat /etc/apache2/ports.conf` | View ports configuration |
| `sed -i 's/old/new/g' FILE` | Replace text in file |
| `docker exec CONTAINER COMMAND` | Run command in container |
| `docker ps` | List running containers |

---

## Apache Configuration Files Reference

### Main Configuration:
```
/etc/apache2/apache2.conf          # Main config file
/etc/apache2/ports.conf            # Port bindings
/etc/apache2/envvars               # Environment variables
```

### Sites Configuration:
```
/etc/apache2/sites-available/      # Available site configs
/etc/apache2/sites-enabled/        # Enabled site configs
/etc/apache2/sites-available/000-default.conf  # Default site
```

### Modules:
```
/etc/apache2/mods-available/       # Available modules
/etc/apache2/mods-enabled/         # Enabled modules
```

### Logs:
```
/var/log/apache2/access.log        # Access log
/var/log/apache2/error.log         # Error log
```

### Web Content:
```
/var/www/html/                     # Default document root
/var/www/html/index.html           # Default page
```

---

## Completion Checklist

- [ ] SSH into Application Server 1 (stapp01) as tony
- [ ] Switched to root user
- [ ] Verified kkloud container is running
- [ ] Accessed container shell with docker exec
- [ ] Updated apt package repository
- [ ] Installed apache2 package
- [ ] Verified Apache installation
- [ ] Checked Apache configuration files
- [ ] Modified /etc/apache2/ports.conf to Listen 5002
- [ ] Updated VirtualHost in 000-default.conf to *:5002
- [ ] Tested Apache configuration (configtest)
- [ ] Started Apache service
- [ ] Verified Apache service running
- [ ] Confirmed Apache listening on port 5002
- [ ] Tested Apache responds on port 5002 (curl)
- [ ] Exited container properly
- [ ] Verified container still running from host
- [ ] Verified Apache still running in container
- [ ] All requirements met ✅

---

## Completion Details

- **Completion Date:** December 15, 2025
- **Day:** 40 / 100
- **Challenge:** KodeKloud 100 Days Cloud DevOps
- **Topic:** Configuring Apache in Docker Container
- **Server:** Application Server 1 (stapp01)
- **User:** tony
- **Container:** `kkloud` (running)
- **Package:** apache2
- **Port:** 5002
- **Listen Address:** 0.0.0.0 (all interfaces)
- **Service State:** Running ✅
- **Container State:** Running ✅
- **Key Skill:** Container service configuration and port management
- **Status:** ✅ Successfully Completed

---

## Summary

This task demonstrated **configuring services in running Docker containers**:

✅ **Installed apache2** - Web server package in container
✅ **Configured custom port** - Changed from 80 to 5002
✅ **Updated configuration files** - ports.conf and VirtualHost
✅ **Started service** - Apache running in container
✅ **Verified listening** - Port 5002 accessible
✅ **Kept container running** - Proper exit strategy

**Key Insight:** Containers can run full services like web servers, but **service management is different** from host systems:
- No systemctl in most containers
- Use `service` command or direct binaries
- Need to keep container's main process alive
- Services don't auto-start on container restart (unless configured)

**The Configuration Process:**
```
Container Access → Install Package → Configure Port → Start Service → Verify
```

**Important Configuration Changes:**
```
1. ports.conf: Listen 5002
   → Makes Apache bind to port 5002
   
2. VirtualHost *:5002
   → Makes Apache handle requests on that port
   
Both must match!
```

**Port Binding Explained:**
```
Listen 5002 = Listen 0.0.0.0:5002
↓
Accessible from:
- localhost
- 127.0.0.1
- Container IP
- Any IP mapped to container
```

**Container Service Challenges:**
- Services don't persist after container restart (unless using supervisord/systemd)
- Need to ensure service starts with container
- Better approach: Build proper image with service auto-start

**Production Approach (Better Way):**
```dockerfile
FROM ubuntu:latest
RUN apt-get update && \
    apt-get install -y apache2 && \
    sed -i 's/Listen 80/Listen 5002/g' /etc/apache2/ports.conf && \
    sed -i 's/*:80/*:5002/g' /etc/apache2/sites-available/000-default.conf
EXPOSE 5002
CMD ["apache2ctl", "-D", "FOREGROUND"]
```

**When to Configure Running Containers:**
- ✅ Development and testing
- ✅ Quick fixes and hotpatches
- ✅ Learning and exploration
- ✅ Temporary requirements
- ❌ Production deployments (use Dockerfiles)

**Remember:** `docker exec -it container bash` → Make changes → Test → Exit properly! 🐳

**The Task Handoff:** Successfully completed pending work! Apache now serving on port 5002 inside the kkloud container on App Server 1! 🚀
