# Day 74: Jenkins Automated Database Backup Job

## 📋 Task Overview

**Scenario:** The xFusionCorp Industries team needs to automate database backups to ensure data protection and disaster recovery. You need to create a Jenkins job that automatically takes MySQL database dumps and transfers them to a backup server on a regular schedule.

**Given Requirements:**
- **Jenkins UI Access:** Click Jenkins button on top bar
- **Login Credentials:**
  - Username: `admin`
  - Password: `Adm!n321`
- **Job Configuration:**
  - Job Name: `database-backup`
  - Database Server: Database Server in Stratos Datacenter
  - Database Name: `kodekloud_db01`
  - Database User: `kodekloud_roy`
  - Database Password: `asdfgdsd`
  - Dump Filename: `db_$(date +%F).sql` (e.g., db_2026-01-27.sql)
  - Destination: Backup Server at `/home/clint/db_backups`
  - Schedule: Run every 10 minutes (`*/10 * * * *`)

**Your Mission:**
1. Log in to Jenkins
2. Create new freestyle job named `database-backup`
3. Configure periodic build schedule (every 10 minutes)
4. Set up SSH access to Database Server and Backup Server
5. Configure build steps to:
   - Take mysqldump of kodekloud_db01
   - Name dump with current date format
   - Transfer dump to Backup Server
6. Verify job runs automatically every 10 minutes
7. Confirm backups are created on Backup Server

---

## 🎯 Learning Objectives

By the end of this task, you will understand:
- **Database Backup Automation:** Scheduled MySQL dumps with Jenkins
- **mysqldump Command:** Taking database backups from command line
- **Backup Strategies:** Full backups, incremental, retention policies
- **Scheduled Jobs:** Cron-based automation in Jenkins
- **Disaster Recovery:** Backup verification and restoration
- **Security:** Handling database credentials securely

---

## 📖 Understanding Database Backups

### Why Database Backups Are Critical

**Data Loss Prevention:**
```
Scenarios requiring backups:
- Hardware failure (disk crash, server failure)
- Human error (accidental DELETE, DROP TABLE)
- Software bugs (data corruption)
- Security breaches (ransomware, malicious deletion)
- Natural disasters (fire, flood, power surge)

Without backups: Data is permanently lost ❌
With backups: Data can be restored ✅
```

**Business Continuity:**
```
Downtime costs:
- E-commerce: $5,600/minute average
- Financial services: $7,900/minute average
- Enterprise: $300,000/hour average

Recovery Time Objective (RTO): How quickly can you restore?
Recovery Point Objective (RPO): How much data loss is acceptable?

Example:
- RTO: 1 hour (system must be back in 1 hour)
- RPO: 15 minutes (lose max 15 minutes of data)
- Solution: Backups every 10 minutes with 1-hour restore time
```

**Compliance Requirements:**
```
Regulations requiring backups:
- GDPR (General Data Protection Regulation)
- HIPAA (Healthcare)
- SOX (Sarbanes-Oxley - Financial)
- PCI DSS (Payment Card Industry)

Typical requirements:
- Daily backups minimum
- 30-90 day retention
- Off-site storage
- Tested restore procedures
```

---

### MySQL Backup Methods

**1. Logical Backup (mysqldump) ⭐ Today's Focus**
```
What: SQL statements to recreate database
Tool: mysqldump command
Output: .sql file with CREATE/INSERT statements

Advantages:
✅ Human-readable SQL
✅ Cross-platform (any MySQL version)
✅ Selective backup (specific tables/databases)
✅ Easy to edit or inspect
✅ Works with replication

Disadvantages:
❌ Slower for large databases (>100GB)
❌ Larger file sizes (uncompressed)
❌ Database locked during backup (with --lock-tables)

Use Cases:
- Small to medium databases (<100GB)
- Cross-version migrations
- Schema backup
- Selective table backup
```

**2. Physical Backup (File System Copy)**
```
What: Copy data files directly
Tools: cp, rsync, tar
Output: Binary data files

Advantages:
✅ Fast backup and restore
✅ Smaller file sizes
✅ Minimal overhead

Disadvantages:
❌ Requires database shutdown or special tools
❌ Not portable (version-specific)
❌ Not human-readable

Use Cases:
- Very large databases (>100GB)
- Same MySQL version
- Full server migration
```

**3. Incremental Backup (Binary Logs)**
```
What: Only changes since last backup
Tools: mysqlbinlog
Output: Binary log files

Advantages:
✅ Minimal storage
✅ Point-in-time recovery
✅ Continuous backup

Disadvantages:
❌ Complex restore process
❌ Requires full backup first

Use Cases:
- Point-in-time recovery
- Minimizing backup windows
- Replication setup
```

---

### mysqldump Command Deep Dive

**Basic Syntax:**
```bash
mysqldump [options] database_name > backup.sql
```

**Common Options:**
```bash
# Authentication
-h hostname           # Database server host
-u username           # Database user
-p                    # Prompt for password
-p'password'          # Password (no space after -p)

# Output Options
--result-file=file    # Write to file instead of stdout
--single-transaction  # Consistent backup without locking (InnoDB)
--quick              # Retrieve rows one at a time (less memory)

# What to Backup
--all-databases      # Backup all databases
--databases db1 db2  # Backup specific databases
--tables table1      # Backup specific tables
--ignore-table=db.tbl # Skip specific table

# Additional Data
--routines           # Include stored procedures and functions
--triggers           # Include triggers (default: yes)
--events             # Include scheduled events

# Compression
--compress           # Compress data between client and server

# Locking
--lock-tables        # Lock all tables (MyISAM)
--lock-all-tables    # Lock all tables in all databases
--single-transaction # No locking (InnoDB only, recommended)
```

**Example Commands:**

**Basic database dump:**
```bash
mysqldump -u kodekloud_roy -p'asdfgdsd' kodekloud_db01 > backup.sql
```

**With hostname:**
```bash
mysqldump -h db.example.com -u kodekloud_roy -p'asdfgdsd' kodekloud_db01 > backup.sql
```

**With compression and no locking (InnoDB):**
```bash
mysqldump -u kodekloud_roy -p'asdfgdsd' \
  --single-transaction \
  --quick \
  kodekloud_db01 > backup.sql
```

**With date in filename:**
```bash
mysqldump -u kodekloud_roy -p'asdfgdsd' kodekloud_db01 > db_$(date +%F).sql
# Creates: db_2026-01-27.sql
```

**Multiple databases:**
```bash
mysqldump -u root -p'password' --databases db1 db2 db3 > multi_db.sql
```

**All databases:**
```bash
mysqldump -u root -p'password' --all-databases > all_databases.sql
```

**With gzip compression:**
```bash
mysqldump -u kodekloud_roy -p'asdfgdsd' kodekloud_db01 | gzip > backup.sql.gz
```

---

### Date Formatting with date Command

**The date +%F format:**
```bash
date +%F
# Output: 2026-01-27 (YYYY-MM-DD format)

# This is equivalent to:
date +%Y-%m-%d
# %Y = 4-digit year (2026)
# %m = 2-digit month (01-12)
# %d = 2-digit day (01-31)
```

**Other useful date formats:**
```bash
date +%F          # 2026-01-27 (ISO 8601 date)
date +%Y%m%d      # 20260127 (no separators)
date +%Y-%m-%d_%H-%M-%S  # 2026-01-27_14-30-45 (with time)
date +%s          # 1738075845 (Unix timestamp)
date +%A          # Monday (full weekday name)
date +%B          # January (full month name)

# For backup filenames:
db_$(date +%F).sql                    # db_2026-01-27.sql
backup_$(date +%Y%m%d_%H%M%S).sql    # backup_20260127_143045.sql
mysql_$(date +%Y-%m-%d).tar.gz        # mysql_2026-01-27.tar.gz
```

**Why use date +%F for backups?**
```
✅ ISO 8601 standard (internationally recognized)
✅ Sortable alphabetically (2026-01-27 < 2026-01-28)
✅ Human-readable
✅ No special characters that cause issues
✅ One backup per day (overwrite same-day backups)

Example backup history:
db_2026-01-25.sql
db_2026-01-26.sql
db_2026-01-27.sql  ← Latest
```

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

**Database Server:**
```
Purpose: MySQL/MariaDB database server (source of backup)
Hostname: stdb01.stratos.xfusioncorp.com (or similar)
Short name: stdb01
User: peter (typical for Database Server)
Database Name: kodekloud_db01
Database User: kodekloud_roy
Database Password: asdfgdsd
MySQL Port: 3306 (default)
```

**Backup Server:**
```
Purpose: Backup storage location (destination)
Hostname: stbkp01.stratos.xfusioncorp.com (or similar)
Short name: stbkp01
User: clint (typical for Backup Server)
Destination: /home/clint/db_backups
```

---

### Step 3: Check Required Plugins (Optional)

**For this task, we'll use SSH and shell commands. Plugins are optional.**

**If you want to use Publish Over SSH plugin:**

1. Go to **Manage Jenkins** → **Plugins**
2. Click **Available plugins**
3. Search for: `Publish Over SSH`
4. Check the checkbox
5. Click **Install**
6. Select: **Restart Jenkins when installation is complete and no jobs are running**
7. Wait 30-60 seconds for restart
8. Login again with `admin` / `Adm!n321`

**However, direct SSH/SCP commands work fine for this task.**

---

### Step 4: Configure SSH Access

**SSH keys need to be set up between Jenkins server and target servers.**

**Method A: Using SSH Keys (Recommended)**

```bash
# On Jenkins server (as jenkins user)
sudo -u jenkins ssh-keygen -t rsa -b 4096 -N "" -f /var/lib/jenkins/.ssh/id_rsa

# Copy public key to Database Server
sudo -u jenkins ssh-copy-id peter@stdb01

# Copy public key to Backup Server
sudo -u jenkins ssh-copy-id clint@stbkp01

# Test connections
sudo -u jenkins ssh peter@stdb01 "echo 'DB Server: Connection successful'"
sudo -u jenkins ssh clint@stbkp01 "echo 'Backup Server: Connection successful'"
```

**Method B: Using Password Authentication (Less Secure)**

If SSH keys can't be configured, use `sshpass`:
```bash
# Install sshpass on Jenkins server
sudo yum install sshpass -y  # RHEL/CentOS
# or
sudo apt-get install sshpass -y  # Ubuntu/Debian

# Use in commands:
sshpass -p 'password' ssh user@host "command"
sshpass -p 'password' scp file user@host:/path
```

---

### Step 5: Create New Jenkins Job

**From Jenkins Dashboard:**

1. Click **New Item** (left sidebar)
2. **Enter job name:** `database-backup`
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
*/10 * * * *
```

**Explanation:**
```
*/10  = Every 10 minutes
*     = Every hour
*     = Every day of month
*     = Every month
*     = Every day of week

This runs the job every 10 minutes, 24/7
Runs at: 00:00, 00:10, 00:20, 00:30, 00:40, 00:50, etc.
Total: 144 backups per day
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
│  [*/10 * * * *______________________]        │
│                                              │
│  Would last have run at Monday, January 27,  │
│  2026 2:30 PM; would next run at Monday,     │
│  January 27, 2026 2:40 PM.                   │
│                                              │
└──────────────────────────────────────────────┘
```

**Jenkins will show when the job would last and next run.**

---

### Step 7: Configure Build Steps

Scroll down to the **Build** section.

**Click:** Add build step → **Execute shell**

---

### Complete Shell Script for Database Backup

**Enter the following script:**

```bash
#!/bin/bash

#############################################
# Jenkins Job: database-backup
# Purpose: Automated MySQL database backup
# Schedule: Every 10 minutes
#############################################

# Configuration
DB_SERVER="stdb01.stratos.xfusioncorp.com"
DB_SERVER_USER="peter"
DB_NAME="kodekloud_db01"
DB_USER="kodekloud_roy"
DB_PASSWORD="asdfgdsd"

BACKUP_SERVER="stbkp01.stratos.xfusioncorp.com"
BACKUP_SERVER_USER="clint"
BACKUP_DESTINATION="/home/clint/db_backups"

# Backup filename with date format
BACKUP_DATE=$(date +%F)
BACKUP_FILE="db_${BACKUP_DATE}.sql"
TEMP_BACKUP="/tmp/${BACKUP_FILE}"

echo "=========================================="
echo "MySQL Database Backup Job"
echo "Started at: $(date)"
echo "=========================================="
echo ""
echo "Configuration:"
echo "  Database Server: $DB_SERVER"
echo "  Database Name: $DB_NAME"
echo "  Database User: $DB_USER"
echo "  Backup File: $BACKUP_FILE"
echo "  Destination: $BACKUP_SERVER:$BACKUP_DESTINATION"
echo ""

#############################################
# Step 1: Take Database Dump
#############################################

echo "Step 1: Taking database dump..."
echo "  Connecting to: $DB_SERVER"
echo "  Database: $DB_NAME"

# Execute mysqldump on Database Server via SSH
ssh -o StrictHostKeyChecking=no ${DB_SERVER_USER}@${DB_SERVER} \
  "mysqldump -u ${DB_USER} -p'${DB_PASSWORD}' ${DB_NAME}" > ${TEMP_BACKUP} 2>/dev/null

# Check if dump was successful
if [ $? -eq 0 ] && [ -s ${TEMP_BACKUP} ]; then
    DUMP_SIZE=$(du -h ${TEMP_BACKUP} | cut -f1)
    echo "✓ Database dump created successfully"
    echo "  File: ${TEMP_BACKUP}"
    echo "  Size: ${DUMP_SIZE}"
else
    echo "✗ ERROR: Failed to create database dump"
    echo "  Possible reasons:"
    echo "    - Database server not accessible"
    echo "    - Invalid credentials"
    echo "    - Database does not exist"
    echo "    - MySQL/MariaDB not running"
    rm -f ${TEMP_BACKUP}
    exit 1
fi

# Validate backup file contains SQL data
if grep -q "CREATE TABLE" ${TEMP_BACKUP} 2>/dev/null; then
    echo "✓ Backup file validated (contains SQL statements)"
else
    echo "⚠ WARNING: Backup file may be empty or invalid"
fi

echo ""

#############################################
# Step 2: Transfer Backup to Backup Server
#############################################

echo "Step 2: Transferring backup to Backup Server..."
echo "  Destination: ${BACKUP_SERVER}:${BACKUP_DESTINATION}"

# Create backup directory if it doesn't exist
ssh -o StrictHostKeyChecking=no ${BACKUP_SERVER_USER}@${BACKUP_SERVER} \
  "mkdir -p ${BACKUP_DESTINATION}" 2>/dev/null

if [ $? -eq 0 ]; then
    echo "✓ Backup directory verified/created"
else
    echo "✗ ERROR: Failed to access/create backup directory"
    rm -f ${TEMP_BACKUP}
    exit 1
fi

# Transfer backup file
scp -o StrictHostKeyChecking=no ${TEMP_BACKUP} \
  ${BACKUP_SERVER_USER}@${BACKUP_SERVER}:${BACKUP_DESTINATION}/${BACKUP_FILE}

if [ $? -eq 0 ]; then
    echo "✓ Backup transferred successfully"
    echo "  Remote location: ${BACKUP_DESTINATION}/${BACKUP_FILE}"
else
    echo "✗ ERROR: Failed to transfer backup to Backup Server"
    rm -f ${TEMP_BACKUP}
    exit 1
fi

echo ""

#############################################
# Step 3: Verify Backup on Backup Server
#############################################

echo "Step 3: Verifying backup on Backup Server..."

# Check if file exists and get its size
REMOTE_SIZE=$(ssh -o StrictHostKeyChecking=no ${BACKUP_SERVER_USER}@${BACKUP_SERVER} \
  "du -h ${BACKUP_DESTINATION}/${BACKUP_FILE} 2>/dev/null | cut -f1")

if [ -n "$REMOTE_SIZE" ]; then
    echo "✓ Backup verified on Backup Server"
    echo "  File: ${BACKUP_FILE}"
    echo "  Size: ${REMOTE_SIZE}"
else
    echo "✗ ERROR: Backup file not found on Backup Server"
    rm -f ${TEMP_BACKUP}
    exit 1
fi

# Get total backup count
TOTAL_BACKUPS=$(ssh -o StrictHostKeyChecking=no ${BACKUP_SERVER_USER}@${BACKUP_SERVER} \
  "ls -1 ${BACKUP_DESTINATION}/db_*.sql 2>/dev/null | wc -l")

echo "  Total backups on server: ${TOTAL_BACKUPS}"

echo ""

#############################################
# Step 4: Cleanup
#############################################

echo "Step 4: Cleaning up temporary files..."

rm -f ${TEMP_BACKUP}

if [ $? -eq 0 ]; then
    echo "✓ Temporary files removed"
else
    echo "⚠ WARNING: Failed to remove temporary file: ${TEMP_BACKUP}"
fi

echo ""

#############################################
# Summary
#############################################

echo "=========================================="
echo "Backup Summary:"
echo "  Database: $DB_NAME"
echo "  Backup File: $BACKUP_FILE"
echo "  Remote Location: ${BACKUP_SERVER}:${BACKUP_DESTINATION}/${BACKUP_FILE}"
echo "  Backup Size: ${REMOTE_SIZE}"
echo "  Total Backups: ${TOTAL_BACKUPS}"
echo "  Status: ✓ SUCCESS"
echo "=========================================="
echo "Completed at: $(date)"
echo "=========================================="

exit 0
```

**This script:**
- ✅ Takes mysqldump of kodekloud_db01 database
- ✅ Uses correct database credentials
- ✅ Names backup with date format: db_YYYY-MM-DD.sql
- ✅ Transfers backup to Backup Server
- ✅ Creates destination directory if needed
- ✅ Verifies backup after transfer
- ✅ Provides detailed logging
- ✅ Handles errors gracefully
- ✅ Cleans up temporary files

---

### Alternative Script with Compression

**If backups are large, add gzip compression:**

```bash
#!/bin/bash

# Configuration (same as above)
DB_SERVER="stdb01.stratos.xfusioncorp.com"
DB_SERVER_USER="peter"
DB_NAME="kodekloud_db01"
DB_USER="kodekloud_roy"
DB_PASSWORD="asdfgdsd"

BACKUP_SERVER="stbkp01.stratos.xfusioncorp.com"
BACKUP_SERVER_USER="clint"
BACKUP_DESTINATION="/home/clint/db_backups"

BACKUP_DATE=$(date +%F)
BACKUP_FILE="db_${BACKUP_DATE}.sql.gz"  # Note: .sql.gz extension
TEMP_BACKUP="/tmp/${BACKUP_FILE}"

echo "=========================================="
echo "MySQL Database Backup Job (Compressed)"
echo "Started at: $(date)"
echo "=========================================="

# Step 1: Take compressed database dump
echo ""
echo "Step 1: Taking compressed database dump..."

ssh -o StrictHostKeyChecking=no ${DB_SERVER_USER}@${DB_SERVER} \
  "mysqldump -u ${DB_USER} -p'${DB_PASSWORD}' ${DB_NAME} | gzip" > ${TEMP_BACKUP} 2>/dev/null

if [ $? -eq 0 ] && [ -s ${TEMP_BACKUP} ]; then
    DUMP_SIZE=$(du -h ${TEMP_BACKUP} | cut -f1)
    echo "✓ Compressed database dump created"
    echo "  File: ${TEMP_BACKUP}"
    echo "  Compressed Size: ${DUMP_SIZE}"
else
    echo "✗ ERROR: Failed to create database dump"
    rm -f ${TEMP_BACKUP}
    exit 1
fi

# Step 2: Transfer to Backup Server
echo ""
echo "Step 2: Transferring backup..."

ssh -o StrictHostKeyChecking=no ${BACKUP_SERVER_USER}@${BACKUP_SERVER} \
  "mkdir -p ${BACKUP_DESTINATION}" 2>/dev/null

scp -o StrictHostKeyChecking=no ${TEMP_BACKUP} \
  ${BACKUP_SERVER_USER}@${BACKUP_SERVER}:${BACKUP_DESTINATION}/${BACKUP_FILE}

if [ $? -eq 0 ]; then
    echo "✓ Backup transferred successfully"
else
    echo "✗ ERROR: Transfer failed"
    rm -f ${TEMP_BACKUP}
    exit 1
fi

# Step 3: Verify
echo ""
echo "Step 3: Verifying backup..."

REMOTE_SIZE=$(ssh -o StrictHostKeyChecking=no ${BACKUP_SERVER_USER}@${BACKUP_SERVER} \
  "du -h ${BACKUP_DESTINATION}/${BACKUP_FILE} 2>/dev/null | cut -f1")

if [ -n "$REMOTE_SIZE" ]; then
    echo "✓ Backup verified: ${REMOTE_SIZE}"
else
    echo "✗ ERROR: Verification failed"
    rm -f ${TEMP_BACKUP}
    exit 1
fi

# Cleanup
rm -f ${TEMP_BACKUP}

echo ""
echo "=========================================="
echo "Backup completed: ${BACKUP_FILE}"
echo "Location: ${BACKUP_DESTINATION}/${BACKUP_FILE}"
echo "Size: ${REMOTE_SIZE}"
echo "=========================================="

exit 0
```

**Compression benefits:**
- Reduces file size by 70-90%
- Faster transfer over network
- Saves storage space
- To restore: `gunzip db_2026-01-27.sql.gz && mysql -u user -p database < db_2026-01-27.sql`

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

2. Verify the schedule shows: `*/10 * * * *`

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
Building in workspace /var/lib/jenkins/workspace/database-backup
[database-backup] $ /bin/sh -xe /tmp/jenkins1234567890.sh
==========================================
MySQL Database Backup Job
Started at: Mon Jan 27 14:40:00 UTC 2026
==========================================

Configuration:
  Database Server: stdb01.stratos.xfusioncorp.com
  Database Name: kodekloud_db01
  Database User: kodekloud_roy
  Backup File: db_2026-01-27.sql
  Destination: stbkp01.stratos.xfusioncorp.com:/home/clint/db_backups

Step 1: Taking database dump...
  Connecting to: stdb01.stratos.xfusioncorp.com
  Database: kodekloud_db01
✓ Database dump created successfully
  File: /tmp/db_2026-01-27.sql
  Size: 2.3M
✓ Backup file validated (contains SQL statements)

Step 2: Transferring backup to Backup Server...
  Destination: stbkp01.stratos.xfusioncorp.com:/home/clint/db_backups
✓ Backup directory verified/created
✓ Backup transferred successfully
  Remote location: /home/clint/db_backups/db_2026-01-27.sql

Step 3: Verifying backup on Backup Server...
✓ Backup verified on Backup Server
  File: db_2026-01-27.sql
  Size: 2.3M
  Total backups on server: 1

Step 4: Cleaning up temporary files...
✓ Temporary files removed

==========================================
Backup Summary:
  Database: kodekloud_db01
  Backup File: db_2026-01-27.sql
  Remote Location: stbkp01.stratos.xfusioncorp.com:/home/clint/db_backups/db_2026-01-27.sql
  Backup Size: 2.3M
  Total Backups: 1
  Status: ✓ SUCCESS
==========================================
Completed at: Mon Jan 27 14:40:08 UTC 2026
==========================================
Finished: SUCCESS
```

**Result:** Build should succeed (blue ball) ✅

---

### Step 11: Wait for Automatic Execution

**The job should now run automatically every 10 minutes:**

**Observe build history:**
```
Build History:
#7  ● 3:00 PM - SUCCESS (Scheduled) - 8 seconds
#6  ● 2:50 PM - SUCCESS (Scheduled) - 8 seconds
#5  ● 2:40 PM - SUCCESS (Scheduled) - 9 seconds
#4  ● 2:30 PM - SUCCESS (Scheduled) - 8 seconds
#3  ● 2:20 PM - SUCCESS (Scheduled) - 8 seconds
#2  ● 2:10 PM - SUCCESS (Scheduled) - 9 seconds
#1  ● 2:00 PM - SUCCESS (Manual) - 10 seconds
```

**Each build shows:**
- Build number
- Timestamp (at 10-minute intervals)
- Status (blue ball = success)
- Duration (typically 8-10 seconds)
- Trigger (Scheduled or Manual)

**Wait at least 20-30 minutes to see multiple automatic builds.**

---

### Step 12: Verify Backups on Backup Server

**SSH to Backup Server and verify backups:**

```bash
# Connect to Backup Server
ssh clint@stbkp01

# Navigate to backup directory
cd /home/clint/db_backups

# List backup files
ls -lh

# Expected output:
-rw-r--r-- 1 clint clint 2.3M Jan 27 14:40 db_2026-01-27.sql

# Note: Only ONE file per day (date format)
# Each run overwrites the same-day backup

# View backup file details
du -h db_2026-01-27.sql
file db_2026-01-27.sql

# Output:
2.3M    db_2026-01-27.sql
db_2026-01-27.sql: ASCII text, with very long lines

# View first 20 lines of backup
head -20 db_2026-01-27.sql

# Expected:
-- MySQL dump 10.19  Distrib 10.3.39-MariaDB, for Linux (x86_64)
--
-- Host: localhost    Database: kodekloud_db01
-- ------------------------------------------------------
-- Server version	10.3.39-MariaDB

/*!40101 SET @OLD_CHARACTER_SET_CLIENT=@@CHARACTER_SET_CLIENT */;
/*!40101 SET @OLD_CHARACTER_SET_RESULTS=@@CHARACTER_SET_RESULTS */;
...
CREATE TABLE `users` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(255) DEFAULT NULL,
  ...
```

**Check backup over multiple days:**
```bash
# After running for 3 days:
ls -lh /home/clint/db_backups/

-rw-r--r-- 1 clint clint 2.2M Jan 25 23:50 db_2026-01-25.sql
-rw-r--r-- 1 clint clint 2.3M Jan 26 23:50 db_2026-01-26.sql
-rw-r--r-- 1 clint clint 2.3M Jan 27 15:00 db_2026-01-27.sql

# One backup per day (latest run of the day)
```

---

### Step 13: Test Backup Restoration (Verification)

**To ensure backups are valid, test restoration:**

```bash
# On Database Server (or any server with MySQL client)
ssh peter@stdb01

# Create a test database
mysql -u kodekloud_roy -p'asdfgdsd' -e "CREATE DATABASE test_restore;"

# Restore backup to test database
mysql -u kodekloud_roy -p'asdfgdsd' test_restore < /home/clint/db_backups/db_2026-01-27.sql

# If backup is on Backup Server, copy first:
scp clint@stbkp01:/home/clint/db_backups/db_2026-01-27.sql /tmp/
mysql -u kodekloud_roy -p'asdfgdsd' test_restore < /tmp/db_2026-01-27.sql

# Verify restoration
mysql -u kodekloud_roy -p'asdfgdsd' test_restore -e "SHOW TABLES;"

# Compare with original database
mysql -u kodekloud_roy -p'asdfgdsd' kodekloud_db01 -e "SHOW TABLES;"

# Should show same tables

# Check row counts
mysql -u kodekloud_roy -p'asdfgdsd' test_restore -e "SELECT COUNT(*) FROM users;"
mysql -u kodekloud_roy -p'asdfgdsd' kodekloud_db01 -e "SELECT COUNT(*) FROM users;"

# Should show same counts

# Cleanup test database
mysql -u kodekloud_roy -p'asdfgdsd' -e "DROP DATABASE test_restore;"
```

**If restoration succeeds:** Backups are valid ✅

---

## 📊 Enhanced Script with Retention and Monitoring

**For production use, add backup retention and alerting:**

```bash
#!/bin/bash

#############################################
# Enhanced Database Backup Script
# Features:
# - Backup retention (keep last 30 days)
# - Compression
# - Backup size tracking
# - Error notifications
# - Backup validation
#############################################

# Configuration
DB_SERVER="stdb01.stratos.xfusioncorp.com"
DB_SERVER_USER="peter"
DB_NAME="kodekloud_db01"
DB_USER="kodekloud_roy"
DB_PASSWORD="asdfgdsd"

BACKUP_SERVER="stbkp01.stratos.xfusioncorp.com"
BACKUP_SERVER_USER="clint"
BACKUP_DESTINATION="/home/clint/db_backups"

# Retention policy
RETENTION_DAYS=30
MIN_BACKUP_SIZE_KB=100  # Minimum valid backup size

# Timestamp and filename
BACKUP_DATE=$(date +%F)
BACKUP_TIME=$(date +%H%M%S)
BACKUP_FILE="db_${BACKUP_DATE}.sql.gz"
BACKUP_FILE_WITH_TIME="db_${BACKUP_DATE}_${BACKUP_TIME}.sql.gz"
TEMP_BACKUP="/tmp/${BACKUP_FILE}"

# Email configuration (optional)
ALERT_EMAIL="admin@example.com"
SEND_ALERTS=false  # Set to true to enable email alerts

echo "=========================================="
echo "Enhanced MySQL Database Backup"
echo "Started at: $(date)"
echo "=========================================="

#############################################
# Function: Send alert
#############################################
send_alert() {
    local subject="$1"
    local message="$2"
    
    if [ "$SEND_ALERTS" = true ]; then
        echo "$message" | mail -s "$subject" "$ALERT_EMAIL"
        echo "⚠ Alert sent to: $ALERT_EMAIL"
    fi
}

#############################################
# Function: Check database connectivity
#############################################
check_db_connectivity() {
    echo "Checking database connectivity..."
    
    DB_CHECK=$(ssh -o StrictHostKeyChecking=no ${DB_SERVER_USER}@${DB_SERVER} \
        "mysql -u ${DB_USER} -p'${DB_PASSWORD}' -e 'SELECT 1;' 2>&1")
    
    if [ $? -eq 0 ]; then
        echo "✓ Database server accessible"
        return 0
    else
        echo "✗ ERROR: Cannot connect to database"
        echo "  Error: $DB_CHECK"
        send_alert "Database Backup Failed" "Cannot connect to database server $DB_SERVER"
        return 1
    fi
}

#############################################
# Step 0: Pre-backup checks
#############################################

echo ""
echo "Step 0: Pre-backup checks..."

# Check database connectivity
if ! check_db_connectivity; then
    exit 1
fi

# Check disk space on Backup Server
AVAILABLE_SPACE=$(ssh -o StrictHostKeyChecking=no ${BACKUP_SERVER_USER}@${BACKUP_SERVER} \
    "df -k ${BACKUP_DESTINATION} | tail -1 | awk '{print \$4}'")

if [ $AVAILABLE_SPACE -lt 1048576 ]; then  # Less than 1GB
    echo "⚠ WARNING: Low disk space on Backup Server: $(($AVAILABLE_SPACE / 1024))MB"
    send_alert "Low Disk Space" "Backup server has only $(($AVAILABLE_SPACE / 1024))MB free"
fi

#############################################
# Step 1: Take database dump
#############################################

echo ""
echo "Step 1: Taking compressed database dump..."

START_TIME=$(date +%s)

ssh -o StrictHostKeyChecking=no ${DB_SERVER_USER}@${DB_SERVER} \
    "mysqldump -u ${DB_USER} -p'${DB_PASSWORD}' \
    --single-transaction \
    --quick \
    --routines \
    --triggers \
    ${DB_NAME} | gzip" > ${TEMP_BACKUP} 2>/dev/null

DUMP_EXIT_CODE=$?
END_TIME=$(date +%s)
DURATION=$((END_TIME - START_TIME))

if [ $DUMP_EXIT_CODE -eq 0 ] && [ -s ${TEMP_BACKUP} ]; then
    DUMP_SIZE_KB=$(du -k ${TEMP_BACKUP} | cut -f1)
    DUMP_SIZE_HUMAN=$(du -h ${TEMP_BACKUP} | cut -f1)
    
    if [ $DUMP_SIZE_KB -lt $MIN_BACKUP_SIZE_KB ]; then
        echo "✗ ERROR: Backup file too small (${DUMP_SIZE_HUMAN}), possibly corrupted"
        send_alert "Backup Failed" "Backup file size (${DUMP_SIZE_HUMAN}) below minimum threshold"
        rm -f ${TEMP_BACKUP}
        exit 1
    fi
    
    echo "✓ Database dump created successfully"
    echo "  File: ${TEMP_BACKUP}"
    echo "  Size: ${DUMP_SIZE_HUMAN} (${DUMP_SIZE_KB} KB)"
    echo "  Duration: ${DURATION} seconds"
else
    echo "✗ ERROR: Failed to create database dump"
    send_alert "Backup Failed" "mysqldump failed with exit code: $DUMP_EXIT_CODE"
    rm -f ${TEMP_BACKUP}
    exit 1
fi

#############################################
# Step 2: Transfer to Backup Server
#############################################

echo ""
echo "Step 2: Transferring backup to Backup Server..."

# Create dated subdirectory
DATED_DIR="${BACKUP_DESTINATION}/$(date +%Y-%m)"
ssh -o StrictHostKeyChecking=no ${BACKUP_SERVER_USER}@${BACKUP_SERVER} \
    "mkdir -p ${DATED_DIR}" 2>/dev/null

# Transfer with original filename (overwrites same-day backup)
scp -o StrictHostKeyChecking=no ${TEMP_BACKUP} \
    ${BACKUP_SERVER_USER}@${BACKUP_SERVER}:${BACKUP_DESTINATION}/${BACKUP_FILE}

if [ $? -eq 0 ]; then
    echo "✓ Backup transferred to: ${BACKUP_FILE}"
else
    echo "✗ ERROR: Transfer failed"
    send_alert "Backup Transfer Failed" "Failed to transfer backup to $BACKUP_SERVER"
    rm -f ${TEMP_BACKUP}
    exit 1
fi

# Also save timestamped copy for multiple backups per day
scp -o StrictHostKeyChecking=no ${TEMP_BACKUP} \
    ${BACKUP_SERVER_USER}@${BACKUP_SERVER}:${DATED_DIR}/${BACKUP_FILE_WITH_TIME}

#############################################
# Step 3: Verify backup
#############################################

echo ""
echo "Step 3: Verifying backup..."

REMOTE_SIZE=$(ssh -o StrictHostKeyChecking=no ${BACKUP_SERVER_USER}@${BACKUP_SERVER} \
    "du -h ${BACKUP_DESTINATION}/${BACKUP_FILE} 2>/dev/null | cut -f1")

if [ -n "$REMOTE_SIZE" ]; then
    echo "✓ Backup verified: ${REMOTE_SIZE}"
else
    echo "✗ ERROR: Verification failed"
    send_alert "Backup Verification Failed" "Backup file not found on $BACKUP_SERVER"
    rm -f ${TEMP_BACKUP}
    exit 1
fi

#############################################
# Step 4: Rotate old backups
#############################################

echo ""
echo "Step 4: Rotating old backups..."

# Delete backups older than retention period
DELETED_COUNT=$(ssh -o StrictHostKeyChecking=no ${BACKUP_SERVER_USER}@${BACKUP_SERVER} << EOSSH
    cd ${BACKUP_DESTINATION}
    find . -name "db_*.sql.gz" -mtime +${RETENTION_DAYS} -type f | tee /dev/stderr | wc -l
    find . -name "db_*.sql.gz" -mtime +${RETENTION_DAYS} -type f -delete
EOSSH
)

if [ "$DELETED_COUNT" -gt 0 ]; then
    echo "✓ Deleted $DELETED_COUNT old backup(s) (older than $RETENTION_DAYS days)"
else
    echo "✓ No old backups to delete"
fi

#############################################
# Step 5: Generate statistics
#############################################

echo ""
echo "Step 5: Generating backup statistics..."

TOTAL_BACKUPS=$(ssh -o StrictHostKeyChecking=no ${BACKUP_SERVER_USER}@${BACKUP_SERVER} \
    "find ${BACKUP_DESTINATION} -name 'db_*.sql.gz' -type f | wc -l")

TOTAL_SIZE=$(ssh -o StrictHostKeyChecking=no ${BACKUP_SERVER_USER}@${BACKUP_SERVER} \
    "du -sh ${BACKUP_DESTINATION} 2>/dev/null | cut -f1")

OLDEST_BACKUP=$(ssh -o StrictHostKeyChecking=no ${BACKUP_SERVER_USER}@${BACKUP_SERVER} \
    "find ${BACKUP_DESTINATION} -name 'db_*.sql.gz' -type f -printf '%T+ %p\n' | sort | head -1 | cut -d' ' -f2 | xargs basename")

NEWEST_BACKUP=$(ssh -o StrictHostKeyChecking=no ${BACKUP_SERVER_USER}@${BACKUP_SERVER} \
    "find ${BACKUP_DESTINATION} -name 'db_*.sql.gz' -type f -printf '%T+ %p\n' | sort -r | head -1 | cut -d' ' -f2 | xargs basename")

#############################################
# Cleanup
#############################################

echo ""
echo "Step 6: Cleaning up..."
rm -f ${TEMP_BACKUP}
echo "✓ Temporary files removed"

#############################################
# Summary
#############################################

echo ""
echo "=========================================="
echo "Backup Summary:"
echo "  Database: ${DB_NAME}"
echo "  Date: ${BACKUP_DATE}"
echo "  Time: ${BACKUP_TIME}"
echo "  Backup File: ${BACKUP_FILE}"
echo "  Size: ${REMOTE_SIZE}"
echo "  Duration: ${DURATION} seconds"
echo "  Location: ${BACKUP_SERVER}:${BACKUP_DESTINATION}"
echo ""
echo "Statistics:"
echo "  Total Backups: ${TOTAL_BACKUPS}"
echo "  Total Size: ${TOTAL_SIZE}"
echo "  Oldest Backup: ${OLDEST_BACKUP}"
echo "  Newest Backup: ${NEWEST_BACKUP}"
echo "  Retention: ${RETENTION_DAYS} days"
echo ""
echo "  Status: ✓ SUCCESS"
echo "=========================================="
echo "Completed at: $(date)"
echo "=========================================="

exit 0
```

**Enhanced features:**
- ✅ Pre-backup connectivity check
- ✅ Disk space monitoring
- ✅ Backup size validation
- ✅ Compression with mysqldump options
- ✅ Automatic rotation (30-day retention)
- ✅ Timestamped copies (multiple per day)
- ✅ Organized by month folders
- ✅ Detailed statistics
- ✅ Email alerts (optional)
- ✅ Backup timing metrics

---

## 🧪 Testing and Validation

### Test Scenario 1: Manual Build

**Steps:**
1. Click "Build Now"
2. Wait for build to complete
3. Check console output

**Expected:**
- Build completes in 8-15 seconds
- Console shows dump created, transferred, verified
- Build shows blue ball ✅

---

### Test Scenario 2: Automatic Scheduled Build

**Steps:**
1. Wait 10 minutes after manual build
2. Observe new build appears automatically
3. Check console output

**Expected:**
- Build #2 appears exactly 10 minutes after #1
- Triggered by: "Started by timer"
- Console output similar to manual build
- Build succeeds ✅

---

### Test Scenario 3: Multiple Consecutive Builds

**Steps:**
1. Wait 30 minutes
2. Observe 3 builds have occurred
3. Check timing intervals

**Expected:**
- Builds at exactly 10-minute intervals
- All show blue balls (success)
- Each build takes similar time (8-10 seconds)
- No failures

---

### Test Scenario 4: Backup File Verification

**Steps:**
1. SSH to Backup Server
2. Navigate to /home/clint/db_backups
3. List files and check sizes

**Expected:**
```bash
ls -lh /home/clint/db_backups/

-rw-r--r-- 1 clint clint 2.3M Jan 27 15:30 db_2026-01-27.sql

# File size should be reasonable (not 0 bytes or too small)
# File should be updated every 10 minutes (same filename, updated timestamp)
```

---

### Test Scenario 5: Backup Content Validation

**Steps:**
1. View backup file content
2. Verify SQL statements present
3. Check for database structures

**Expected:**
```bash
head -30 db_2026-01-27.sql

# Should show:
-- MySQL dump ...
-- Host: localhost    Database: kodekloud_db01
...
CREATE TABLE `tablename` (
...
INSERT INTO `tablename` VALUES (...);
```

---

### Test Scenario 6: Restore Test (Critical)

**Steps:**
1. Create test database
2. Restore backup
3. Verify data integrity

**Expected:**
```bash
mysql -u kodekloud_roy -p'asdfgdsd' -e "CREATE DATABASE test_restore;"
mysql -u kodekloud_roy -p'asdfgdsd' test_restore < db_2026-01-27.sql
mysql -u kodekloud_roy -p'asdfgdsd' test_restore -e "SHOW TABLES;"

# Should show all tables from original database
# Data should be intact
# Restoration should complete without errors
```

---

## 🐛 Common Issues and Solutions

### Issue 1: MySQL Access Denied

**Symptoms:**
```
ERROR 1045 (28000): Access denied for user 'kodekloud_roy'@'host'
mysqldump failed
Build failed
```

**Diagnosis:**
- Wrong username or password
- User doesn't have required privileges
- MySQL not accessible from Jenkins server

**Solution:**

```bash
# Test credentials directly on Database Server
ssh peter@stdb01
mysql -u kodekloud_roy -p'asdfgdsd' kodekloud_db01 -e "SELECT 1;"

# If access denied, check user exists
mysql -u root -p
SELECT user, host FROM mysql.user WHERE user='kodekloud_roy';

# Grant necessary privileges
GRANT SELECT, LOCK TABLES, SHOW VIEW ON kodekloud_db01.* TO 'kodekloud_roy'@'localhost';
GRANT SELECT, LOCK TABLES, SHOW VIEW ON kodekloud_db01.* TO 'kodekloud_roy'@'%';
FLUSH PRIVILEGES;

# Test again
mysql -u kodekloud_roy -p'asdfgdsd' kodekloud_db01 -e "SHOW TABLES;"
```

---

### Issue 2: Database Does Not Exist

**Symptoms:**
```
mysqldump: Got error: 1049: Unknown database 'kodekloud_db01'
Backup file empty or very small
```

**Diagnosis:**
- Database name spelled wrong
- Database doesn't exist on server
- Connected to wrong database server

**Solution:**

```bash
# List all databases
ssh peter@stdb01
mysql -u kodekloud_roy -p'asdfgdsd' -e "SHOW DATABASES;"

# Look for correct database name (case-sensitive)
# Common variations:
# - kodekloud_db01
# - kodekloud_db
# - kodekloud_db1
# - kodeklouddb01

# Update script with correct database name
```

---

### Issue 3: Backup File Empty or Too Small

**Symptoms:**
```
✓ Database dump created successfully
  Size: 512B  # Too small!
⚠ WARNING: Backup file may be empty or invalid
```

**Diagnosis:**
- Database is actually empty (no tables)
- mysqldump failed but didn't report error
- Wrong database backed up

**Solution:**

```bash
# Check if database has tables
mysql -u kodekloud_roy -p'asdfgdsd' kodekloud_db01 -e "SHOW TABLES;"

# If empty, database might be new or wrong database
# Check table count
mysql -u kodekloud_roy -p'asdfgdsd' kodekloud_db01 -e "SELECT COUNT(*) FROM information_schema.tables WHERE table_schema='kodekloud_db01';"

# Check if data exists in tables
mysql -u kodekloud_roy -p'asdfgdsd' kodekloud_db01 -e "SELECT table_name, table_rows FROM information_schema.tables WHERE table_schema='kodekloud_db01';"

# If database has data but dump is small, check mysqldump stderr
ssh peter@stdb01
mysqldump -u kodekloud_roy -p'asdfgdsd' kodekloud_db01 > /tmp/test.sql 2>&1
cat /tmp/test.sql
```

---

### Issue 4: Permission Denied on Backup Server

**Symptoms:**
```
scp: /home/clint/db_backups/db_2026-01-27.sql: Permission denied
✗ ERROR: Failed to transfer backup
```

**Diagnosis:**
- User 'clint' doesn't have write permission
- Directory doesn't exist
- Parent directory not writable

**Solution:**

```bash
# On Backup Server
ssh clint@stbkp01

# Check directory permissions
ls -ld /home/clint/db_backups/

# If directory doesn't exist
mkdir -p /home/clint/db_backups

# If permission denied
chmod 755 /home/clint/db_backups

# Ensure ownership
sudo chown -R clint:clint /home/clint/db_backups

# Test write access
touch /home/clint/db_backups/test.txt
rm /home/clint/db_backups/test.txt
```

---

### Issue 5: Build Running Too Long

**Symptoms:**
```
Build #5 started at 3:00 PM
Still running at 3:15 PM (15 minutes)
Previous builds completed in 8-10 seconds
```

**Diagnosis:**
- Very large database (takes long to dump)
- Network issues (slow transfer)
- SSH connection hanging
- mysqldump locked waiting for other queries

**Solution:**

```bash
# Add timeout to commands
timeout 300 ssh ... "mysqldump ..."  # 5-minute timeout

# Check database size
ssh peter@stdb01
mysql -u kodekloud_roy -p'asdfgdsd' kodekloud_db01 -e "
SELECT 
    table_schema AS 'Database',
    ROUND(SUM(data_length + index_length) / 1024 / 1024, 2) AS 'Size (MB)'
FROM information_schema.tables
WHERE table_schema = 'kodekloud_db01'
GROUP BY table_schema;"

# If database is very large (>10GB), consider:
# 1. Using --single-transaction (no locking)
# 2. Dumping during off-peak hours
# 3. Using physical backup methods (xtrabackup)
# 4. Incremental backups

# Add to mysqldump command:
mysqldump --single-transaction --quick ...

# Set build timeout in Jenkins:
# Job configuration → Build Environment
# ☑ Abort the build if it's stuck
# Timeout: 10 minutes
```

---

### Issue 6: Builds Overwriting Same Backup

**Symptoms:**
```
# After 3 builds (30 minutes):
ls /home/clint/db_backups/
-rw-r--r-- 1 clint clint 2.3M Jan 27 15:20 db_2026-01-27.sql

# Only ONE file, not three
# File timestamp shows latest build time
```

**Diagnosis:**
- This is EXPECTED behavior with date +%F format
- Creates one backup per day
- Each build overwrites same-day backup

**Solution:**

**If you want multiple backups per day:**

```bash
# Change filename format to include time
BACKUP_FILE="db_$(date +%Y-%m-%d_%H%M%S).sql"
# Creates: db_2026-01-27_143000.sql, db_2026-01-27_144000.sql, etc.

# Or use timestamp
BACKUP_FILE="db_$(date +%s).sql"
# Creates: db_1738075800.sql, db_1738076400.sql, etc.

# Then you'll have:
db_2026-01-27_143000.sql
db_2026-01-27_144000.sql
db_2026-01-27_145000.sql
...

# Remember to implement cleanup to avoid filling disk:
find /home/clint/db_backups -name "db_*.sql" -mtime +7 -delete
```

**If you want one backup per day (current behavior):**
- This is correct and efficient
- Keeps latest backup of the day
- No cleanup needed (only 1 file per day)
- After 30 days, you'll have 30 files (one per day)

---

## 📖 Key Takeaways

### Concepts Mastered Today:

1. ✅ **Database Backup Automation**
   - Scheduled mysqldump execution
   - Remote database access via SSH
   - Backup file naming conventions
   - Transfer to backup location

2. ✅ **mysqldump Command**
   - Basic syntax and options
   - Authentication and connectivity
   - Output formats (SQL, compressed)
   - Best practices (--single-transaction, --quick)

3. ✅ **Backup Strategies**
   - Full database backups
   - Daily vs hourly vs real-time
   - Retention policies
   - Verification and restoration testing

4. ✅ **Scheduled Automation**
   - Jenkins cron expressions
   - Periodic build triggers
   - Build frequency considerations
   - Monitoring automatic executions

5. ✅ **Disaster Recovery**
   - Backup validation techniques
   - Restoration procedures
   - Testing backup integrity
   - RTO and RPO concepts

---

## 🎯 Real-World Applications

### Use Case 1: Production Database Backups

**Scenario:** E-commerce platform with critical customer data

```yaml
Backup Strategy:
- Full backups: Every 6 hours (00:00, 06:00, 12:00, 18:00)
- Incremental: Binary logs every 15 minutes
- Retention: 30 days full, 90 days incremental
- Storage: On-site + Off-site (S3)

Jenkins Jobs:
- database-backup-full (*/0 0,6,12,18 * * *)
- database-backup-binlog (*/15 * * * *)
- database-backup-verify (0 2 * * *)
- database-backup-cleanup (0 3 * * *)

RTO: 1 hour
RPO: 15 minutes
```

---

### Use Case 2: Multi-Database Backup

**Scenario:** Microservices with 10 separate databases

```bash
# Modified script to backup multiple databases
DATABASES=(
    "users_db"
    "orders_db"
    "inventory_db"
    "payments_db"
    "analytics_db"
    "logs_db"
    "sessions_db"
    "notifications_db"
    "reports_db"
    "audit_db"
)

for DB in "${DATABASES[@]}"; do
    echo "Backing up: $DB"
    mysqldump -u user -p'pass' $DB | gzip > "db_${DB}_$(date +%F).sql.gz"
    # Transfer to backup server
done
```

---

### Use Case 3: Backup with Monitoring

**Pipeline:**
```
Jenkins Job (database-backup)
    ↓
Take mysqldump
    ↓
Compress with gzip
    ↓
Transfer to Backup Server
    ↓
Verify backup integrity
    ↓
Log metrics to monitoring system
    ↓
If failed: Send alert (email, Slack, PagerDuty)
    ↓
If success: Update dashboard
```

---

### Use Case 4: Compliance Backup

**Scenario:** HIPAA-compliant healthcare data

```yaml
Requirements:
- Daily backups mandatory
- 7-year retention required
- Encrypted at rest and in transit
- Audit log of all access
- Tested restore quarterly

Implementation:
- Backup with encryption: mysqldump | gpg -e > backup.sql.gpg
- Store in compliant location (encrypted S3 bucket)
- Access logging enabled
- Quarterly restore drill scheduled
- Compliance report generated monthly
```

---

## 📚 Additional Resources

**Official Documentation:**
- [mysqldump Documentation](https://dev.mysql.com/doc/refman/8.0/en/mysqldump.html)
- [MySQL Backup Methods](https://dev.mysql.com/doc/refman/8.0/en/backup-methods.html)
- [MariaDB Backup](https://mariadb.com/kb/en/backup-and-restore-overview/)

**Backup Tools:**
- [Percona XtraBackup](https://www.percona.com/software/mysql-database/percona-xtrabackup) (Physical backups)
- [mydumper](https://github.com/mydumper/mydumper) (Parallel logical backup)
- [MySQL Enterprise Backup](https://www.mysql.com/products/enterprise/backup.html)

**Best Practices:**
- [MySQL Backup Best Practices](https://dev.mysql.com/doc/refman/8.0/en/backup-strategy.html)
- [Database Backup Strategies](https://www.digitalocean.com/community/tutorials/how-to-backup-mysql-databases-on-an-ubuntu-vps)
- [Disaster Recovery Planning](https://www.google.com/search?q=database+disaster+recovery+best+practices)

**Next Steps:**
- **Day 75:** Jenkins Pipeline with Multiple Stages
- **Day 76:** Jenkins + Docker Integration
- **Day 77:** Jenkins + Ansible for Configuration Management
- **Day 78:** Jenkins + Kubernetes Deployment

---

## ✅ Task Completion Checklist

**Job Creation:**
- [ ] Logged into Jenkins (admin / Adm!n321)
- [ ] Created new job named `database-backup`
- [ ] Selected Freestyle project type
- [ ] Job created successfully

**Build Trigger Configuration:**
- [ ] Configured "Build periodically"
- [ ] Cron expression set to `*/10 * * * *`
- [ ] Jenkins shows "Next build" timing
- [ ] Schedule validated and saved

**SSH Configuration:**
- [ ] SSH access to Database Server configured
- [ ] SSH access to Backup Server configured
- [ ] Test connections successful
- [ ] Keys or passwords configured

**Build Step Configuration:**
- [ ] Added Execute Shell build step
- [ ] Script uses correct database credentials:
  - [ ] Database name: `kodekloud_db01`
  - [ ] Database user: `kodekloud_roy`
  - [ ] Database password: `asdfgdsd`
- [ ] Backup filename format: `db_$(date +%F).sql`
- [ ] Destination: `/home/clint/db_backups` on Backup Server
- [ ] Error handling included
- [ ] Verification steps added

**Testing:**
- [ ] Manual build executed successfully
- [ ] Console output shows:
  - [ ] Database dump created
  - [ ] File size reasonable (not 0 bytes)
  - [ ] Transfer to Backup Server successful
  - [ ] Backup verified on destination
- [ ] Build completed with SUCCESS status

**Automatic Execution:**
- [ ] Waited 10 minutes for first automatic build
- [ ] Build #2 triggered automatically
- [ ] "Started by timer" message in console
- [ ] Build succeeded

**Multiple Automatic Builds:**
- [ ] Observed 3-4 automatic builds
- [ ] All builds at 10-minute intervals (XX:00, XX:10, XX:20, XX:30)
- [ ] All builds succeeded (blue balls)
- [ ] Build history shows consistent timing

**Verification on Backup Server:**
- [ ] SSHed to Backup Server (`ssh clint@stbkp01`)
- [ ] Navigated to `/home/clint/db_backups`
- [ ] Found backup file: `db_2026-01-27.sql` (or current date)
- [ ] File size is reasonable (>100KB, depends on database)
- [ ] File timestamp updated every 10 minutes
- [ ] File contains valid SQL statements

**Backup Validation:**
- [ ] Viewed backup file content (`head -20 db_2026-01-27.sql`)
- [ ] Confirmed SQL dump format (CREATE TABLE, INSERT statements)
- [ ] Backup contains database name: `kodekloud_db01`
- [ ] Test restoration performed (optional but recommended):
  - [ ] Created test database
  - [ ] Restored backup successfully
  - [ ] Verified data integrity
  - [ ] Dropped test database

**Documentation:**
- [ ] Screenshots captured of:
  - [ ] Job configuration page
  - [ ] Build triggers section with `*/10 * * * *`
  - [ ] Build step with shell script
  - [ ] Build history showing multiple automatic builds
  - [ ] Console output of successful build showing all steps
  - [ ] Backup Server directory listing: `ls -lh /home/clint/db_backups/`
  - [ ] Backup file content: `head -20 db_2026-01-27.sql`
- [ ] Or screen recording created (loom.com)

---

**🎉 Congratulations!** You've successfully created an automated database backup system! You've mastered:
- Jenkins scheduled jobs with cron expressions
- MySQL database dumps with mysqldump
- Automated backup transfers
- Backup verification and validation
- Disaster recovery fundamentals

This job now runs every 10 minutes, automatically backing up the kodekloud_db01 database to ensure data protection!

**Day 74 Status:** ✅ Complete

**Next:** Day 75 - Jenkins Multi-Stage Pipeline! 🚀
