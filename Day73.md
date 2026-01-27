# Day 73: Jenkins Scheduled Job for Apache Log Collection

## 📋 Task Overview

**Scenario:** The xFusionCorp Industries DevOps team is setting up centralized logging management. While the full solution is being implemented, they need to regularly collect Apache logs from application servers for troubleshooting. You need to create an automated Jenkins job that periodically copies Apache logs to a central storage location.

**Given Requirements:**
- **Jenkins UI Access:** Click Jenkins button on top bar
- **Login Credentials:**
  - Username: `admin`
  - Password: `Adm!n321`
- **Job Configuration:**
  - Job Name: `copy-logs`
  - Schedule: Build every 5 minutes (cron: `*/5 * * * *`)
  - Source: App Server 2 (Apache logs from default location)
  - Logs to copy:
    - `access_log` (Apache access logs)
    - `error_log` (Apache error logs)
  - Destination: Storage Server at `/usr/src/devops`

**Your Mission:**
1. Log in to Jenkins
2. Create new freestyle job named `copy-logs`
3. Configure periodic build schedule (every 5 minutes)
4. Set up SSH access to App Server 2 and Storage Server
5. Configure build steps to copy Apache logs
6. Verify job runs automatically every 5 minutes
7. Confirm logs are copied to destination

---

## 🎯 Learning Objectives

By the end of this task, you will understand:
- **Scheduled Builds:** Configuring periodic job execution with cron
- **Cron Expressions:** Jenkins cron syntax and scheduling
- **Log Management:** Collecting and centralizing application logs
- **SSH Operations:** Remote file copying with scp/rsync
- **Apache Logs:** Understanding Apache log locations and formats
- **Automation:** Building self-healing monitoring systems

---

## 📖 Understanding Jenkins Build Triggers

### Types of Build Triggers

**1. Build Periodically (Time-Based)**
```
Trigger: Cron schedule
Use Case: Regular maintenance, log collection, backups
Example: Run every 5 minutes, hourly, daily
```

**2. Poll SCM (Repository-Based)**
```
Trigger: Check for code changes
Use Case: CI/CD on code commits
Example: Check Git every 2 minutes for changes
```

**3. Build After Other Projects (Dependency-Based)**
```
Trigger: Another job completion
Use Case: Pipeline of jobs
Example: Deploy after build completes
```

**4. GitHub Hook Trigger (Webhook-Based)**
```
Trigger: GitHub webhook
Use Case: Immediate build on push
Example: Build on every commit/PR
```

**5. Manual Trigger**
```
Trigger: "Build Now" button
Use Case: On-demand execution
Example: Manual deployment
```

---

### Cron Expression Syntax in Jenkins

**Format:**
```
MINUTE HOUR DAY_OF_MONTH MONTH DAY_OF_WEEK

Field          | Allowed Values | Special Characters
---------------|----------------|--------------------
MINUTE         | 0-59           | * , - /
HOUR           | 0-23           | * , - /
DAY_OF_MONTH   | 1-31           | * , - / L W
MONTH          | 1-12 or JAN-DEC| * , - /
DAY_OF_WEEK    | 0-7 or SUN-SAT | * , - / L #
               | (0 and 7 = Sun)|
```

**Special Characters:**
- `*` = Every (every minute, every hour, etc.)
- `,` = Multiple values (1,15,30)
- `-` = Range (1-5 = 1,2,3,4,5)
- `/` = Increment (*/5 = every 5th)
- `H` = Hash (distributed timing to avoid spikes)

---

### Common Cron Examples

**Every 5 minutes:**
```
*/5 * * * *
Runs: 00:00, 00:05, 00:10, 00:15, ... (288 times per day)
```

**Every 10 minutes:**
```
*/10 * * * *
Runs: 00:00, 00:10, 00:20, 00:30, ... (144 times per day)
```

**Every hour:**
```
0 * * * *
Runs: 00:00, 01:00, 02:00, ... (24 times per day)
```

**Every day at midnight:**
```
0 0 * * *
Runs: 00:00 daily
```

**Every day at 2:30 AM:**
```
30 2 * * *
Runs: 02:30 daily
```

**Weekdays at 9 AM:**
```
0 9 * * 1-5
Runs: Monday-Friday at 09:00
```

**Every 15 minutes during business hours (9 AM - 5 PM):**
```
*/15 9-17 * * *
Runs: 09:00, 09:15, 09:30, ..., 17:45
```

**First day of every month:**
```
0 0 1 * *
Runs: 00:00 on the 1st of each month
```

---

### Jenkins-Specific: Using H (Hash)

**Why use H?**
```
Problem with */5 * * * *:
- If 100 jobs use this schedule
- All 100 jobs run simultaneously at 00:00, 00:05, 00:10
- Server overload and resource contention

Solution with H/5 * * * *:
- Jenkins distributes job timing
- Jobs spread across 5-minute window
- Job A runs at 00:01, Job B at 00:03, Job C at 00:04, etc.
- Reduces server load spikes
```

**Examples with H:**
```
H/5 * * * *     # Every 5 minutes (distributed timing)
H * * * *       # Once per hour (distributed)
H H * * *       # Once per day (random time each day)
H H * * 0       # Once per week on Sunday (random time)
```

---

## 📖 Understanding Apache Logs

### Apache Log Files

**1. Access Log (access_log or access.log)**
```
Purpose: Records all requests to the web server
Location: 
  - RHEL/CentOS: /var/log/httpd/access_log
  - Debian/Ubuntu: /var/log/apache2/access.log

Format (Combined Log Format):
IP - - [timestamp] "REQUEST" status_code bytes "referer" "user-agent"

Example:
192.168.1.100 - - [27/Jan/2026:10:30:45 +0000] "GET /index.html HTTP/1.1" 200 1234 "https://google.com" "Mozilla/5.0..."

Information captured:
- Client IP address
- Request timestamp
- HTTP method (GET, POST, etc.)
- Requested URL
- HTTP status code (200, 404, 500, etc.)
- Response size in bytes
- Referer (where request came from)
- User agent (browser/client info)
```

**2. Error Log (error_log or error.log)**
```
Purpose: Records server errors, warnings, and diagnostic information
Location:
  - RHEL/CentOS: /var/log/httpd/error_log
  - Debian/Ubuntu: /var/log/apache2/error.log

Format:
[timestamp] [loglevel] [pid] [client IP] error message

Example:
[Mon Jan 27 10:30:45.123456 2026] [error] [pid 1234] [client 192.168.1.100:12345] File does not exist: /var/www/html/missing.php

Log Levels:
- emerg: Emergency - system unusable
- alert: Action must be taken immediately
- crit: Critical conditions
- error: Error conditions
- warn: Warning conditions
- notice: Normal but significant condition
- info: Informational messages
- debug: Debug-level messages
```

---

### Why Collect Apache Logs?

**Troubleshooting:**
- Identify 404 errors (broken links)
- Debug 500 errors (server errors)
- Track down performance issues
- Investigate security incidents

**Analytics:**
- Page view statistics
- Popular content analysis
- Traffic patterns and trends
- User behavior tracking

**Security:**
- Detect attacks (SQL injection, XSS)
- Identify suspicious IPs
- Track failed authentication attempts
- Monitor for DDoS patterns

**Compliance:**
- Audit trail for regulatory requirements
- Track data access
- Maintain historical records
- Generate compliance reports

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

**Expected:** Jenkins Dashboard appears

---

### Step 2: Identify Server Details

**Before configuring the job, identify the servers:**

**App Server 2:**
```
Purpose: Apache web server (source of logs)
Hostname: stapp02.stratos.xfusioncorp.com (or similar)
Short name: stapp02
User: steve (typical for App Server 2)
Apache logs location:
  - RHEL/CentOS: /var/log/httpd/
    - /var/log/httpd/access_log
    - /var/log/httpd/error_log
  - Ubuntu/Debian: /var/log/apache2/
    - /var/log/apache2/access.log
    - /var/log/apache2/error.log
```

**Storage Server:**
```
Purpose: Central log storage (destination)
Hostname: ststor01.stratos.xfusioncorp.com (or similar)
Short name: ststor01
User: natasha (typical for Storage Server)
Destination: /usr/src/devops
```

---

### Step 3: Check Required Plugins

**For this task, we need SSH capability. Check/install plugins:**

**Option A: Publish Over SSH Plugin (Recommended)**

1. Go to **Manage Jenkins** → **Plugins**
2. Click **Available plugins**
3. Search for: `Publish Over SSH`
4. Check the checkbox
5. Click **Install**
6. Select: **Restart Jenkins when installation is complete and no jobs are running**
7. Wait 30-60 seconds for restart
8. Login again with `admin` / `Adm!n321`

**Option B: Use Direct SSH/SCP**

If SSH client is available on Jenkins server, you can use direct `scp` or `rsync` commands without plugins.

---

### Step 4: Configure SSH Access (If Using Plugin)

**If you installed Publish Over SSH plugin:**

1. Go to **Manage Jenkins** → **Configure System**
2. Scroll down to **Publish over SSH** section
3. Click **Add** under "SSH Servers"

**Configure App Server 2:**
```
Name: app-server-2
Hostname: stapp02.stratos.xfusioncorp.com
Username: steve
Remote Directory: /tmp

Authentication:
  - Use password authentication, or
  - Path to key: /var/lib/jenkins/.ssh/id_rsa
  - Or paste key content in "Key" field
```

4. Click **Test Configuration** → Should show "Success"
5. Click **Add** again for Storage Server

**Configure Storage Server:**
```
Name: storage-server
Hostname: ststor01.stratos.xfusioncorp.com
Username: natasha
Remote Directory: /usr/src/devops

Authentication:
  - Use password authentication, or
  - SSH key authentication
```

6. Click **Test Configuration** → Should show "Success"
7. Click **Save**

---

### Step 5: Create New Jenkins Job

**From Jenkins Dashboard:**

1. Click **New Item** (left sidebar)
2. **Enter job name:** `copy-logs`
3. **Select:** Freestyle project
4. Click **OK**

**You'll be redirected to job configuration page.**

---

### Step 6: Configure Build Triggers

**In the job configuration page:**

1. Scroll to **Build Triggers** section
2. Check the box: **☑ Build periodically**

**A "Schedule" text field will appear.**

3. **Enter cron expression:**
```
*/5 * * * *
```

**Explanation:**
```
*/5   = Every 5 minutes
*     = Every hour
*     = Every day of month
*     = Every month
*     = Every day of week

This runs the job every 5 minutes, 24/7
```

**Schedule field should look like:**
```
┌──────────────────────────────────────────────┐
│  Build Triggers                               │
├──────────────────────────────────────────────┤
│                                              │
│  ☑ Build periodically                        │
│                                              │
│  Schedule:                                   │
│  [*/5 * * * *_______________________]        │
│                                              │
│  Would last have run at Monday, January 27,  │
│  2026 2:35 PM; would next run at Monday,     │
│  January 27, 2026 2:40 PM.                   │
│                                              │
└──────────────────────────────────────────────┘
```

**Jenkins will show when the job would last and next run based on your cron expression.**

---

### Step 7: Configure Build Steps

Scroll down to the **Build** section.

**We need to:**
1. Copy logs from App Server 2 to Jenkins server (or directly to Storage Server)
2. Transfer logs to Storage Server at `/usr/src/devops`

---

**Method 1: Using Publish Over SSH Plugin (If Installed)**

This is more complex for copying between two servers. Skip to Method 2 for simpler approach.

---

**Method 2: Using Execute Shell with SCP/RSYNC (Recommended)**

1. Click **Add build step** → **Execute shell**

**Enter the following script:**

```bash
#!/bin/bash

#############################################
# Jenkins Job: copy-logs
# Purpose: Copy Apache logs from App Server 2 to Storage Server
# Schedule: Every 5 minutes
#############################################

# Configuration
APP_SERVER="stapp02.stratos.xfusioncorp.com"
APP_USER="steve"
STORAGE_SERVER="ststor01.stratos.xfusioncorp.com"
STORAGE_USER="natasha"
DESTINATION="/usr/src/devops"

# Apache log locations (try both RHEL and Debian paths)
APACHE_LOGS_RHEL="/var/log/httpd"
APACHE_LOGS_DEBIAN="/var/log/apache2"

# Timestamp for log organization
TIMESTAMP=$(date '+%Y%m%d_%H%M%S')

echo "=========================================="
echo "Apache Log Collection Job"
echo "Started at: $(date)"
echo "=========================================="

# Create temporary directory for logs
TEMP_DIR="/tmp/apache_logs_${TIMESTAMP}"
mkdir -p $TEMP_DIR

echo ""
echo "Step 1: Copying logs from App Server 2..."
echo "  Source: $APP_SERVER"
echo "  User: $APP_USER"

# Try RHEL/CentOS log location first
echo ""
echo "Attempting RHEL/CentOS log location: $APACHE_LOGS_RHEL"
scp -o StrictHostKeyChecking=no ${APP_USER}@${APP_SERVER}:${APACHE_LOGS_RHEL}/access_log ${TEMP_DIR}/access_log 2>/dev/null
if [ $? -eq 0 ]; then
    echo "✓ access_log copied successfully"
    ACCESS_LOG_FOUND=true
else
    echo "⚠ access_log not found in RHEL location, trying Debian..."
    scp -o StrictHostKeyChecking=no ${APP_USER}@${APP_SERVER}:${APACHE_LOGS_DEBIAN}/access.log ${TEMP_DIR}/access_log 2>/dev/null
    if [ $? -eq 0 ]; then
        echo "✓ access.log copied successfully (Debian location)"
        ACCESS_LOG_FOUND=true
    else
        echo "✗ access_log not found in either location"
        ACCESS_LOG_FOUND=false
    fi
fi

# Try to copy error_log
scp -o StrictHostKeyChecking=no ${APP_USER}@${APP_SERVER}:${APACHE_LOGS_RHEL}/error_log ${TEMP_DIR}/error_log 2>/dev/null
if [ $? -eq 0 ]; then
    echo "✓ error_log copied successfully"
    ERROR_LOG_FOUND=true
else
    echo "⚠ error_log not found in RHEL location, trying Debian..."
    scp -o StrictHostKeyChecking=no ${APP_USER}@${APP_SERVER}:${APACHE_LOGS_DEBIAN}/error.log ${TEMP_DIR}/error_log 2>/dev/null
    if [ $? -eq 0 ]; then
        echo "✓ error.log copied successfully (Debian location)"
        ERROR_LOG_FOUND=true
    else
        echo "✗ error_log not found in either location"
        ERROR_LOG_FOUND=false
    fi
fi

# Check if at least one log was found
if [ "$ACCESS_LOG_FOUND" = false ] && [ "$ERROR_LOG_FOUND" = false ]; then
    echo ""
    echo "=========================================="
    echo "ERROR: No Apache logs found on App Server 2"
    echo "=========================================="
    rm -rf $TEMP_DIR
    exit 1
fi

echo ""
echo "Step 2: Transferring logs to Storage Server..."
echo "  Destination: $STORAGE_SERVER:$DESTINATION"
echo "  User: $STORAGE_USER"

# Create destination directory on Storage Server if it doesn't exist
ssh -o StrictHostKeyChecking=no ${STORAGE_USER}@${STORAGE_SERVER} "mkdir -p ${DESTINATION}" 2>/dev/null

# Copy logs to Storage Server with timestamp
if [ "$ACCESS_LOG_FOUND" = true ]; then
    scp -o StrictHostKeyChecking=no ${TEMP_DIR}/access_log ${STORAGE_USER}@${STORAGE_SERVER}:${DESTINATION}/access_log_${TIMESTAMP}
    if [ $? -eq 0 ]; then
        echo "✓ access_log transferred to Storage Server"
    else
        echo "✗ Failed to transfer access_log"
    fi
fi

if [ "$ERROR_LOG_FOUND" = true ]; then
    scp -o StrictHostKeyChecking=no ${TEMP_DIR}/error_log ${STORAGE_USER}@${STORAGE_SERVER}:${DESTINATION}/error_log_${TIMESTAMP}
    if [ $? -eq 0 ]; then
        echo "✓ error_log transferred to Storage Server"
    else
        echo "✗ Failed to transfer error_log"
    fi
fi

# Also copy latest version without timestamp for easy access
if [ "$ACCESS_LOG_FOUND" = true ]; then
    scp -o StrictHostKeyChecking=no ${TEMP_DIR}/access_log ${STORAGE_USER}@${STORAGE_SERVER}:${DESTINATION}/access_log_latest
fi

if [ "$ERROR_LOG_FOUND" = true ]; then
    scp -o StrictHostKeyChecking=no ${TEMP_DIR}/error_log ${STORAGE_USER}@${STORAGE_SERVER}:${DESTINATION}/error_log_latest
fi

# Cleanup temporary directory
echo ""
echo "Step 3: Cleaning up temporary files..."
rm -rf $TEMP_DIR
echo "✓ Temporary files removed"

echo ""
echo "=========================================="
echo "Log Collection Summary:"
echo "  Access Log: $([ "$ACCESS_LOG_FOUND" = true ] && echo "✓ Copied" || echo "✗ Not found")"
echo "  Error Log: $([ "$ERROR_LOG_FOUND" = true ] && echo "✓ Copied" || echo "✗ Not found")"
echo "  Destination: $DESTINATION"
echo "  Timestamp: $TIMESTAMP"
echo "=========================================="
echo "Completed at: $(date)"
echo "=========================================="
```

**This script:**
- ✅ Copies access_log and error_log from App Server 2
- ✅ Handles both RHEL and Debian log locations
- ✅ Creates timestamped copies for historical tracking
- ✅ Maintains "latest" version for quick access
- ✅ Transfers logs to Storage Server
- ✅ Provides detailed logging and error handling
- ✅ Cleans up temporary files

---

**Alternative: Using RSYNC (More efficient for repeated copying)**

```bash
#!/bin/bash

# Configuration
APP_SERVER="stapp02.stratos.xfusioncorp.com"
APP_USER="steve"
STORAGE_SERVER="ststor01.stratos.xfusioncorp.com"
STORAGE_USER="natasha"
DESTINATION="/usr/src/devops"
APACHE_LOGS="/var/log/httpd"

echo "=========================================="
echo "Apache Log Collection (RSYNC Method)"
echo "Started at: $(date)"
echo "=========================================="

# Create destination directory
ssh ${STORAGE_USER}@${STORAGE_SERVER} "mkdir -p ${DESTINATION}"

# Sync access_log
echo "Syncing access_log..."
rsync -avz -e "ssh -o StrictHostKeyChecking=no" \
  ${APP_USER}@${APP_SERVER}:${APACHE_LOGS}/access_log \
  ${STORAGE_USER}@${STORAGE_SERVER}:${DESTINATION}/access_log

# Sync error_log
echo "Syncing error_log..."
rsync -avz -e "ssh -o StrictHostKeyChecking=no" \
  ${APP_USER}@${APP_SERVER}:${APACHE_LOGS}/error_log \
  ${STORAGE_USER}@${STORAGE_SERVER}:${DESTINATION}/error_log

echo "=========================================="
echo "Log sync completed at: $(date)"
echo "=========================================="
```

**RSYNC advantages:**
- Only copies changed portions (incremental)
- Faster for large files
- Automatic compression (-z flag)
- Resume capability

---

### Step 8: Save the Job Configuration

1. Scroll to the bottom of the page
2. Click **Save**

**You'll be redirected to the job page.**

---

### Step 9: Verify Cron Schedule

**On the job page, check:**

1. Look for message near top: 
   ```
   This project is currently configured to build periodically.
   Last build: N/A
   Next build: Will run in X minutes
   ```

2. Verify the schedule is active

---

### Step 10: Test Manual Build First

**Before waiting for automatic execution:**

1. Click **Build Now** (left sidebar)
2. Build #1 will appear in Build History
3. Click on **#1**
4. Click **Console Output**

**Expected output:**
```
Started by user admin
Running as SYSTEM
Building in workspace /var/lib/jenkins/workspace/copy-logs
[copy-logs] $ /bin/sh -xe /tmp/jenkins1234567890.sh
==========================================
Apache Log Collection Job
Started at: Mon Jan 27 14:35:00 UTC 2026
==========================================

Step 1: Copying logs from App Server 2...
  Source: stapp02.stratos.xfusioncorp.com
  User: steve

Attempting RHEL/CentOS log location: /var/log/httpd
✓ access_log copied successfully
✓ error_log copied successfully

Step 2: Transferring logs to Storage Server...
  Destination: ststor01.stratos.xfusioncorp.com:/usr/src/devops
  User: natasha
✓ access_log transferred to Storage Server
✓ error_log transferred to Storage Server

Step 3: Cleaning up temporary files...
✓ Temporary files removed

==========================================
Log Collection Summary:
  Access Log: ✓ Copied
  Error Log: ✓ Copied
  Destination: /usr/src/devops
  Timestamp: 20260127_143500
==========================================
Completed at: Mon Jan 27 14:35:05 UTC 2026
==========================================
Finished: SUCCESS
```

**Result:** Build should succeed (blue ball) ✅

---

### Step 11: Wait for Automatic Execution

**The job should now run automatically every 5 minutes:**

**Observe build history:**
```
Build History:
#4  ● 2:45 PM - SUCCESS (Scheduled)
#3  ● 2:40 PM - SUCCESS (Scheduled)
#2  ● 2:35 PM - SUCCESS (Scheduled)
#1  ● 2:30 PM - SUCCESS (Manual)
```

**Each build shows:**
- Build number
- Timestamp
- Status (blue ball = success)
- Trigger (Scheduled or Manual)

**Wait at least 10-15 minutes to see multiple automatic builds.**

---

### Step 12: Verify Logs on Storage Server

**SSH to Storage Server and verify logs:**

```bash
# Connect to Storage Server
ssh natasha@ststor01

# Check destination directory
cd /usr/src/devops
ls -lh

# Expected output:
-rw-r--r-- 1 natasha natasha 245K Jan 27 14:35 access_log_20260127_143500
-rw-r--r-- 1 natasha natasha  12K Jan 27 14:35 error_log_20260127_143500
-rw-r--r-- 1 natasha natasha 248K Jan 27 14:40 access_log_20260127_144000
-rw-r--r-- 1 natasha natasha  13K Jan 27 14:40 error_log_20260127_144000
-rw-r--r-- 1 natasha natasha 250K Jan 27 14:45 access_log_20260127_144500
-rw-r--r-- 1 natasha natasha  14K Jan 27 14:45 error_log_20260127_144500
-rw-r--r-- 1 natasha natasha 250K Jan 27 14:45 access_log_latest
-rw-r--r-- 1 natasha natasha  14K Jan 27 14:45 error_log_latest

# View latest access log
tail -20 access_log_latest

# View latest error log
tail -20 error_log_latest

# Count total log files
ls -1 | wc -l
```

**Expected:**
- Multiple timestamped log files (one set every 5 minutes)
- `access_log_latest` and `error_log_latest` files (always current)
- Files have reasonable sizes (depends on Apache traffic)
- Timestamps match Jenkins build times

---

### Step 13: Monitor Build Trends

**On the job page:**

1. Look at **Build History** graph
2. Should show regular builds every 5 minutes
3. All builds should be blue (success)

**Click "Trend" or "Build History":**
- Shows all past builds
- Duration of each build
- Success/failure pattern
- Can identify any failures

---

## 📊 Enhanced Script with Compression and Rotation

**For production use, consider adding log rotation and compression:**

```bash
#!/bin/bash

#############################################
# Enhanced Apache Log Collection
# Features:
# - Log rotation (keep last 7 days)
# - Compression (gzip old logs)
# - Disk space check
# - Email notification (if configured)
#############################################

# Configuration
APP_SERVER="stapp02.stratos.xfusioncorp.com"
APP_USER="steve"
STORAGE_SERVER="ststor01.stratos.xfusioncorp.com"
STORAGE_USER="natasha"
DESTINATION="/usr/src/devops"
APACHE_LOGS="/var/log/httpd"

# Retention policy
RETENTION_DAYS=7
MIN_FREE_SPACE_MB=1000

# Timestamp
TIMESTAMP=$(date '+%Y%m%d_%H%M%S')
DATE_FOLDER=$(date '+%Y-%m-%d')

echo "=========================================="
echo "Enhanced Apache Log Collection"
echo "Started at: $(date)"
echo "=========================================="

# Function to check disk space
check_disk_space() {
    local server=$1
    local user=$2
    local path=$3
    
    echo "Checking disk space on $server:$path..."
    FREE_SPACE=$(ssh ${user}@${server} "df -m $path | tail -1 | awk '{print \$4}'")
    
    if [ $FREE_SPACE -lt $MIN_FREE_SPACE_MB ]; then
        echo "WARNING: Low disk space! Only ${FREE_SPACE}MB free"
        return 1
    else
        echo "✓ Sufficient disk space: ${FREE_SPACE}MB free"
        return 0
    fi
}

# Function to rotate old logs
rotate_logs() {
    local server=$1
    local user=$2
    local path=$3
    
    echo "Rotating logs older than $RETENTION_DAYS days..."
    ssh ${user}@${server} << EOSSH
        cd ${path}
        
        # Compress logs older than 1 day
        find . -name "access_log_*" -o -name "error_log_*" -mtime +1 -not -name "*.gz" | while read file; do
            if [ -f "\$file" ]; then
                gzip "\$file"
                echo "  Compressed: \$file"
            fi
        done
        
        # Delete logs older than retention period
        find . -name "*.gz" -mtime +${RETENTION_DAYS} -delete
        echo "  Deleted logs older than ${RETENTION_DAYS} days"
EOSSH
}

# Check disk space first
if ! check_disk_space $STORAGE_SERVER $STORAGE_USER $DESTINATION; then
    echo "ERROR: Insufficient disk space. Aborting."
    exit 1
fi

# Create dated subdirectory structure
DEST_PATH="${DESTINATION}/${DATE_FOLDER}"
ssh ${STORAGE_USER}@${STORAGE_SERVER} "mkdir -p ${DEST_PATH}"

echo ""
echo "Step 1: Copying logs from App Server 2..."

# Copy access_log
scp -o StrictHostKeyChecking=no \
    ${APP_USER}@${APP_SERVER}:${APACHE_LOGS}/access_log \
    ${STORAGE_USER}@${STORAGE_SERVER}:${DEST_PATH}/access_log_${TIMESTAMP}

if [ $? -eq 0 ]; then
    echo "✓ access_log copied"
    ACCESS_SUCCESS=true
else
    echo "✗ Failed to copy access_log"
    ACCESS_SUCCESS=false
fi

# Copy error_log
scp -o StrictHostKeyChecking=no \
    ${APP_USER}@${APP_SERVER}:${APACHE_LOGS}/error_log \
    ${STORAGE_USER}@${STORAGE_SERVER}:${DEST_PATH}/error_log_${TIMESTAMP}

if [ $? -eq 0 ]; then
    echo "✓ error_log copied"
    ERROR_SUCCESS=true
else
    echo "✗ Failed to copy error_log"
    ERROR_SUCCESS=false
fi

# Update symlinks to latest logs
if [ "$ACCESS_SUCCESS" = true ]; then
    ssh ${STORAGE_USER}@${STORAGE_SERVER} \
        "ln -sf ${DEST_PATH}/access_log_${TIMESTAMP} ${DESTINATION}/access_log_latest"
fi

if [ "$ERROR_SUCCESS" = true ]; then
    ssh ${STORAGE_USER}@${STORAGE_SERVER} \
        "ln -sf ${DEST_PATH}/error_log_${TIMESTAMP} ${DESTINATION}/error_log_latest"
fi

echo ""
echo "Step 2: Rotating old logs..."
rotate_logs $STORAGE_SERVER $STORAGE_USER $DESTINATION

echo ""
echo "Step 3: Generating statistics..."

# Get log statistics
ACCESS_SIZE=$(ssh ${STORAGE_USER}@${STORAGE_SERVER} "du -sh ${DEST_PATH}/access_log_${TIMESTAMP} 2>/dev/null | cut -f1" || echo "N/A")
ERROR_SIZE=$(ssh ${STORAGE_USER}@${STORAGE_SERVER} "du -sh ${DEST_PATH}/error_log_${TIMESTAMP} 2>/dev/null | cut -f1" || echo "N/A")
TOTAL_LOGS=$(ssh ${STORAGE_USER}@${STORAGE_SERVER} "find ${DESTINATION} -type f | wc -l")
TOTAL_SIZE=$(ssh ${STORAGE_USER}@${STORAGE_SERVER} "du -sh ${DESTINATION} 2>/dev/null | cut -f1" || echo "N/A")

echo "=========================================="
echo "Log Collection Summary:"
echo "  Date: $DATE_FOLDER"
echo "  Timestamp: $TIMESTAMP"
echo "  Access Log: $([ "$ACCESS_SUCCESS" = true ] && echo "✓ $ACCESS_SIZE" || echo "✗ Failed")"
echo "  Error Log: $([ "$ERROR_SUCCESS" = true ] && echo "✓ $ERROR_SIZE" || echo "✗ Failed")"
echo "  Total logs on storage: $TOTAL_LOGS files ($TOTAL_SIZE)"
echo "  Destination: $DEST_PATH"
echo "=========================================="
echo "Completed at: $(date)"
echo "=========================================="

# Exit with success if at least one log was copied
if [ "$ACCESS_SUCCESS" = true ] || [ "$ERROR_SUCCESS" = true ]; then
    exit 0
else
    exit 1
fi
```

**Enhanced features:**
- ✅ Organizes logs by date folders
- ✅ Checks disk space before copying
- ✅ Compresses logs older than 1 day
- ✅ Deletes logs older than 7 days
- ✅ Creates symlinks to latest logs
- ✅ Generates statistics
- ✅ Better error handling

---

## 🧪 Testing and Validation

### Test Scenario 1: Manual Build

**Steps:**
1. Click "Build Now"
2. Wait for build to complete
3. Check console output for success

**Expected:**
- Build completes in 5-10 seconds
- Console shows both logs copied
- No errors in output
- Build shows blue ball ✅

---

### Test Scenario 2: Automatic Scheduled Build

**Steps:**
1. Wait 5 minutes after manual build
2. Observe new build appears automatically
3. Check console output

**Expected:**
- Build #2 appears exactly 5 minutes after #1
- Triggered by: "Started by timer"
- Console output similar to manual build
- Build succeeds ✅

---

### Test Scenario 3: Multiple Consecutive Builds

**Steps:**
1. Wait 15-20 minutes
2. Observe 3-4 builds have occurred
3. Check all builds succeeded

**Expected:**
- Builds at 5-minute intervals (2:00, 2:05, 2:10, 2:15)
- All show blue balls
- Each build takes similar time
- No failures

---

### Test Scenario 4: Log File Verification

**Steps:**
1. SSH to Storage Server
2. Navigate to `/usr/src/devops`
3. List files and check timestamps

**Expected:**
- Multiple sets of logs (access_log_* and error_log_*)
- Timestamps match Jenkins build times
- File sizes are reasonable (not 0 bytes)
- Latest symlinks point to most recent logs

---

### Test Scenario 5: Apache Log Content

**Steps:**
1. View latest access_log
2. Verify it contains valid Apache log entries
3. Check error_log for Apache errors

**Expected:**
```bash
# Access log should contain entries like:
192.168.1.100 - - [27/Jan/2026:14:35:22 +0000] "GET /index.html HTTP/1.1" 200 1234

# Error log should contain entries like:
[Mon Jan 27 14:35:22.123456 2026] [error] [pid 1234] [client 192.168.1.100:12345] File not found: /var/www/html/missing.php
```

---

## 🐛 Common Issues and Solutions

### Issue 1: SSH Authentication Failed

**Symptoms:**
```
Permission denied (publickey,password)
scp: Connection closed
Build failed
```

**Diagnosis:**
- SSH keys not configured
- Wrong username or password
- SSH not allowed between servers

**Solution:**

```bash
# On Jenkins server, generate SSH key for jenkins user
sudo -u jenkins ssh-keygen -t rsa -b 4096 -N "" -f /var/lib/jenkins/.ssh/id_rsa

# Copy public key to App Server 2
sudo -u jenkins ssh-copy-id steve@stapp02

# Copy public key to Storage Server
sudo -u jenkins ssh-copy-id natasha@ststor01

# Test connections
sudo -u jenkins ssh steve@stapp02 "echo 'Connection successful'"
sudo -u jenkins ssh natasha@ststor01 "echo 'Connection successful'"

# Or use sshpass for password authentication (less secure)
# sshpass -p 'password' scp file user@server:/path
```

---

### Issue 2: Apache Logs Not Found

**Symptoms:**
```
scp: /var/log/httpd/access_log: No such file or directory
✗ access_log not found
Build failed
```

**Diagnosis:**
- Wrong log path
- Apache not installed or running
- Different distribution (Ubuntu vs RHEL)

**Solution:**

```bash
# SSH to App Server 2 and find logs
ssh steve@stapp02

# Check if Apache/httpd is running
sudo systemctl status httpd  # RHEL/CentOS
sudo systemctl status apache2  # Ubuntu/Debian

# Find actual log location
sudo find /var/log -name "*access*log*" -o -name "*error*log*" 2>/dev/null

# Common locations:
# RHEL/CentOS: /var/log/httpd/
# Ubuntu/Debian: /var/log/apache2/
# Custom: Check /etc/httpd/conf/httpd.conf or /etc/apache2/apache2.conf

# Update script with correct path
```

---

### Issue 3: Permission Denied on Destination

**Symptoms:**
```
scp: /usr/src/devops/access_log: Permission denied
✗ Failed to transfer access_log
```

**Diagnosis:**
- User doesn't have write permission to /usr/src/devops
- Directory doesn't exist
- SELinux or AppArmor blocking

**Solution:**

```bash
# On Storage Server
ssh natasha@ststor01

# Create directory if it doesn't exist
sudo mkdir -p /usr/src/devops

# Give ownership to natasha user
sudo chown -R natasha:natasha /usr/src/devops

# Set appropriate permissions
sudo chmod 755 /usr/src/devops

# Verify
ls -ld /usr/src/devops
# Should show: drwxr-xr-x ... natasha natasha ... /usr/src/devops

# If SELinux is enforcing
sudo chcon -t user_home_t /usr/src/devops
# Or disable SELinux temporarily for testing
sudo setenforce 0
```

---

### Issue 4: Cron Job Not Running Automatically

**Symptoms:**
- Manual build works fine
- But no automatic builds appear
- "Next build" time keeps updating but build doesn't trigger

**Diagnosis:**
- Cron expression syntax error
- Jenkins cron service not running
- Build triggers not saved correctly

**Solution:**

```
1. Verify cron expression:
   - Go to job configuration
   - Check "Build periodically" is checked ☑
   - Verify cron expression: */5 * * * *
   - Look for validation message below field
   - Save configuration again

2. Check Jenkins service:
   sudo systemctl status jenkins
   # Should be "active (running)"

3. Review Jenkins logs:
   sudo tail -f /var/log/jenkins/jenkins.log
   # Look for cron trigger messages

4. Test cron expression:
   - Try different expression: */1 * * * * (every minute)
   - Wait 1-2 minutes
   - Should see build trigger
   - Change back to */5 * * * * after test

5. Restart Jenkins if needed:
   sudo systemctl restart jenkins
```

---

### Issue 5: Build Takes Too Long or Times Out

**Symptoms:**
```
Build #5 - Started at 2:35 PM, still running at 2:45 PM
Previous builds completed in 5-10 seconds
Console output frozen
```

**Diagnosis:**
- Network connectivity issues
- Large log files
- SSH connection hanging
- Server overload

**Solution:**

```bash
# Add timeout to scp commands
scp -o ConnectTimeout=30 -o ConnectionAttempts=2 ...

# Add timeout to entire script
timeout 120 scp ...  # 2-minute timeout

# Use compression for large files
scp -C ...  # Enable compression

# Check network connectivity
ping stapp02
ping ststor01

# Check server load
ssh steve@stapp02 "uptime"
ssh natasha@ststor01 "uptime"

# Add timeout to build:
# In job configuration → Build Environment
# ☑ Abort the build if it's stuck
# Timeout strategy: Absolute
# Timeout minutes: 10
```

---

### Issue 6: Disk Space Full on Storage Server

**Symptoms:**
```
scp: write failed: No space left on device
Build failed after several days of operation
Logs accumulating on storage server
```

**Diagnosis:**
- Logs not being rotated or cleaned up
- Disk partition full
- Too many log copies accumulating

**Solution:**

```bash
# Check disk space on Storage Server
ssh natasha@ststor01 "df -h /usr/src/devops"

# See largest files
ssh natasha@ststor01 "du -sh /usr/src/devops/* | sort -hr | head -20"

# Clean up old logs manually
ssh natasha@ststor01 << 'EOF'
    cd /usr/src/devops
    # Delete logs older than 7 days
    find . -name "*log*" -mtime +7 -delete
    
    # Compress logs older than 1 day
    find . -name "*log*" -mtime +1 -not -name "*.gz" -exec gzip {} \;
EOF

# Add log rotation to Jenkins job (see Enhanced Script above)

# Or set up logrotate on Storage Server
sudo cat > /etc/logrotate.d/apache-collected << 'EOF'
/usr/src/devops/*log* {
    daily
    rotate 7
    compress
    delaycompress
    missingok
    notifempty
}
EOF
```

---

## 📖 Key Takeaways

### Concepts Mastered Today:

1. ✅ **Scheduled Builds**
   - Configuring periodic execution with cron
   - Understanding Jenkins cron syntax
   - Build triggers and automation
   - Monitoring scheduled job execution

2. ✅ **Cron Expressions**
   - Five-field format (minute, hour, day, month, weekday)
   - Special characters (*, /, -, ,)
   - Jenkins-specific H (hash) for load distribution
   - Common scheduling patterns

3. ✅ **Log Management**
   - Apache access and error log locations
   - Log collection strategies
   - Central log aggregation
   - Log retention and rotation

4. ✅ **Remote File Operations**
   - SCP for secure file copying
   - RSYNC for efficient synchronization
   - SSH authentication and keys
   - Cross-server file transfers

5. ✅ **Production Considerations**
   - Disk space monitoring
   - Log compression
   - Retention policies
   - Error handling and notifications

---

## 🎯 Real-World Applications

### Use Case 1: Centralized Log Management

**Scenario:** Collect logs from 10 application servers

```yaml
Jenkins Jobs:
- copy-logs-app1 (App Server 1 → Storage)
- copy-logs-app2 (App Server 2 → Storage)
- ...
- copy-logs-app10 (App Server 10 → Storage)

Schedule: */5 * * * * (every 5 minutes)

Storage Structure:
/logs/
  ├── app1/
  │   ├── 2026-01-27/
  │   │   ├── access_log_143000.gz
  │   │   ├── error_log_143000.gz
  │   │   ├── access_log_143500.gz
  │   │   └── error_log_143500.gz
  ├── app2/
  │   └── ...
  └── ...

Analysis:
- ELK Stack (Elasticsearch, Logstash, Kibana)
- Splunk
- Graylog
- Custom scripts
```

---

### Use Case 2: Compliance and Audit Trail

**Scenario:** Maintain 90-day log history for compliance

```bash
Job: collect-compliance-logs
Schedule: 0 0 * * * (daily at midnight)

Script enhancements:
- Collect logs once daily
- Compress immediately
- Encrypt with GPG
- Store for 90 days
- Generate compliance reports
- Alert on anomalies
```

---

### Use Case 3: Real-Time Error Monitoring

**Scenario:** Alert on critical errors in logs

```bash
Job: monitor-apache-errors
Schedule: */1 * * * * (every minute)

Script additions:
- Parse error_log for CRITICAL/ALERT levels
- Count 500-series HTTP errors in access_log
- If threshold exceeded (>10 errors/min):
  - Send email notification
  - Create Jira ticket
  - Trigger PagerDuty alert
  - Scale up application servers
```

---

### Use Case 4: Log Analytics Pipeline

**Pipeline:**
```
Jenkins Job (copy-logs)
    ↓
Storage Server (/usr/src/devops)
    ↓
Logstash (parse and transform)
    ↓
Elasticsearch (index and store)
    ↓
Kibana (visualize and analyze)
    ↓
Alerts and Dashboards
```

---

## 📚 Additional Resources

**Official Documentation:**
- [Jenkins Build Periodically](https://www.jenkins.io/doc/book/pipeline/syntax/#triggers)
- [Jenkins Cron Syntax](https://www.jenkins.io/doc/book/pipeline/syntax/#cron-syntax)
- [Publish Over SSH Plugin](https://plugins.jenkins.io/publish-over-ssh/)

**Apache Logging:**
- [Apache HTTP Server Log Files](https://httpd.apache.org/docs/2.4/logs.html)
- [Apache Log Formats](https://httpd.apache.org/docs/2.4/mod/mod_log_config.html)

**Log Management Tools:**
- [ELK Stack](https://www.elastic.co/elastic-stack)
- [Splunk](https://www.splunk.com/)
- [Graylog](https://www.graylog.org/)
- [Logrotate](https://linux.die.net/man/8/logrotate)

**Best Practices:**
- [Centralized Logging Best Practices](https://www.loggly.com/ultimate-guide/centralizing-logs-with-logstash/)
- [Log Retention Policies](https://www.google.com/search?q=log+retention+best+practices)

**Next Steps:**
- **Day 74:** Jenkins Pipeline as Code (Jenkinsfile)
- **Day 75:** Jenkins + Git Integration
- **Day 76:** Jenkins + Docker Build Pipeline
- **Day 77:** Jenkins Shared Libraries

---

## ✅ Task Completion Checklist

**Job Creation:**
- [ ] Logged into Jenkins (admin / Adm!n321)
- [ ] Created new job named `copy-logs`
- [ ] Selected Freestyle project type
- [ ] Job created successfully

**Build Trigger Configuration:**
- [ ] Configured "Build periodically"
- [ ] Cron expression set to `*/5 * * * *`
- [ ] Jenkins shows "Next build" timing
- [ ] Schedule validated and saved

**Plugin Installation (if needed):**
- [ ] Installed Publish Over SSH plugin (optional)
- [ ] Restarted Jenkins service
- [ ] Plugin verified in installed list

**SSH Configuration:**
- [ ] SSH access to App Server 2 configured
- [ ] SSH access to Storage Server configured
- [ ] Test connections successful
- [ ] Keys or passwords configured

**Build Step Configuration:**
- [ ] Added Execute Shell build step
- [ ] Script copies access_log from App Server 2
- [ ] Script copies error_log from App Server 2
- [ ] Script transfers logs to Storage Server
- [ ] Destination set to `/usr/src/devops`
- [ ] Error handling included
- [ ] Logging and output formatting added

**Testing:**
- [ ] Manual build executed successfully
- [ ] Console output shows both logs copied
- [ ] Logs appear on Storage Server
- [ ] Build completed with SUCCESS status

**Automatic Execution:**
- [ ] Waited 5 minutes for first automatic build
- [ ] Build #2 triggered automatically
- [ ] "Started by timer" message in console
- [ ] Build succeeded

**Multiple Automatic Builds:**
- [ ] Observed 3-4 automatic builds
- [ ] All builds at 5-minute intervals
- [ ] All builds succeeded (blue balls)
- [ ] Build history shows consistent timing

**Verification on Storage Server:**
- [ ] SSHed to Storage Server
- [ ] Navigated to `/usr/src/devops`
- [ ] Found access_log files
- [ ] Found error_log files
- [ ] Timestamps match Jenkins builds
- [ ] File contents are valid Apache logs

**Documentation:**
- [ ] Screenshots captured of:
  - [ ] Job configuration page
  - [ ] Build triggers section with cron expression
  - [ ] Build step with shell script
  - [ ] Build history showing multiple automatic builds
  - [ ] Console output of successful build
  - [ ] Storage server directory listing with logs
- [ ] Or screen recording created (loom.com)

---

**🎉 Congratulations!** You've successfully created an automated Jenkins job for periodic Apache log collection! You've mastered:
- Configuring scheduled builds with cron expressions
- Remote file copying between servers
- Apache log management
- Centralized log collection
- Build automation and monitoring

This job now runs every 5 minutes, automatically collecting Apache logs for analysis and troubleshooting!

**Day 73 Status:** ✅ Complete

**Next:** Day 74 - Jenkins Pipeline as Code with Jenkinsfile! 🚀
