# Day 9: Troubleshooting and Fixing MariaDB Service - Missing Data Directory

## 📋 Objective
Investigate and resolve the critical issue with Nautilus application in Stratos DC where MariaDB service is down on the database server, preventing application connectivity. Root cause: Missing MariaDB data directory.

## 🎯 Resolution
Successfully identified that MariaDB data directory (/var/lib/mysql) was missing, initialized the database using mysql_install_db, and started the service to restore application database connectivity.

## 📝 Task Steps for Resolution

1. **Connect to the database server**
2. **Check MariaDB service status**
3. **Identify why service is down**
4. **Check MariaDB data directory**
5. **Initialize MariaDB database** (missing data directory)
6. **Set proper permissions**
7. **Start MariaDB service**
8. **Enable service for auto-start**
9. **Verify service is running**
10. **Test database connectivity**

## 💻 Commands

```bash
# Step 1: SSH into database server
ssh peter@stdb01
# Password: Sp!dy

# Step 2: Switch to root user
sudo su -

# Step 3: Check MariaDB service status
systemctl status mariadb
# Output shows: failed or inactive

# Step 4: Check if MariaDB is installed
rpm -qa | grep mariadb
yum list installed | grep mariadb

# Step 5: Try to start (will fail if data directory missing)
systemctl start mariadb
# Likely fails with error

# Step 6: Check MariaDB logs
journalctl -u mariadb -n 50
tail -50 /var/log/mariadb/mariadb.log

# Step 7: Check if data directory exists (KEY FINDING!)
ls -la /var/lib/mysql
# Output: ls: cannot access '/var/lib/mysql': No such file or directory
# THIS IS THE PROBLEM!

# Step 8: Check disk space (to ensure we have room)
df -h
# Verify /var has sufficient space

# Step 9: Check MariaDB configuration
cat /etc/my.cnf
# Shows includedir /etc/my.cnf.d

# Step 10: Install MariaDB server if needed
yum install -y mariadb-server

# Step 11: Create the data directory
mkdir -p /var/lib/mysql

# Step 12: Set proper ownership
chown -R mysql:mysql /var/lib/mysql
chmod 755 /var/lib/mysql

# Step 13: Initialize the MariaDB database (CRITICAL STEP!)
mysql_install_db --user=mysql --datadir=/var/lib/mysql

# Alternative for newer MariaDB versions:
# mysqld --initialize-insecure --user=mysql --datadir=/var/lib/mysql

# Step 14: Verify data directory now has files
ls -la /var/lib/mysql/
# Should now show: mysql/, performance_schema/, ib* files

# Step 15: Start MariaDB service
systemctl start mariadb

# Step 16: Enable service for auto-start on boot
systemctl enable mariadb

# Step 17: Verify service is running
systemctl status mariadb
# Should show: active (running)

# Check MariaDB process
ps aux | grep mysql

# Check if listening on port 3306
netstat -tlnp | grep 3306
# or
ss -tlnp | grep 3306
```

## ✅ Verification

```bash
# Verify service is active and running
systemctl status mariadb
# Output should show: 
# Active: active (running)

# Check if service is enabled
systemctl is-enabled mariadb
# Output: enabled

# Verify data directory exists with proper structure
ls -la /var/lib/mysql/
# Should show:
# drwxr-xr-x mysql mysql .
# drwxr-xr-x mysql mysql mysql/
# drwxr-xr-x mysql mysql performance_schema/
# -rw-rw---- mysql mysql ibdata1
# -rw-rw---- mysql mysql ib_logfile*

# Check directory permissions
stat /var/lib/mysql/
# Owner: mysql, Group: mysql, Mode: 0755

# Verify MariaDB is listening on port 3306
ss -tlnp | grep 3306
# Should show: LISTEN on *:3306

# Check MariaDB process
ps aux | grep mysql
# Should show mysqld process running as mysql user

# Test connection locally
mysql -u root
# Should connect successfully

# Inside MySQL, verify databases exist
SHOW DATABASES;
# Should show:
# information_schema
# mysql
# performance_schema

# Test basic query
SELECT VERSION();
EXIT;

# Alternative quick test
mysql -u root -e "SELECT 1;"
# Should return: 1

# Test from application server
# SSH to app server:
ssh tony@stapp01

# Test port connectivity
telnet stdb01 3306
# Should connect and show MariaDB handshake

# Better test:
nc -zv stdb01 3306
# Output: Connection to stdb01 3306 port [tcp/mysql] succeeded!

# Test actual MySQL connection from app server
mysql -h stdb01 -u root -p
# Should connect (if remote access configured)
```

## 🔑 Key Points

- **Root Cause** - Missing `/var/lib/mysql` data directory
- **Why this happens:**
  - Fresh MariaDB installation without initialization
  - Data directory accidentally deleted
  - Incorrect uninstall/reinstall procedure
  - Container/VM provisioning without data persistence
  - Filesystem issues or corruption
- **Critical command:** `mysql_install_db` - Initializes the database
- **What mysql_install_db creates:**
  - System tables (mysql database)
  - Performance schema database
  - User accounts (root)
  - Grant tables
  - Time zone tables
  - System stored procedures
- **Data directory structure:**
  ```
  /var/lib/mysql/
  ├── mysql/              # System database
  ├── performance_schema/ # Performance monitoring
  ├── ibdata1            # InnoDB tablespace
  ├── ib_logfile0        # InnoDB logs
  ├── ib_logfile1
  └── mysql.sock         # Unix socket
  ```
- **Permissions are critical:**
  - Directory: 755 (drwxr-xr-x)
  - Owner: mysql:mysql
  - Files: 660 (rw-rw----)
- **Alternative initialization methods:**
  - `mysql_install_db` - Traditional method (all versions)
  - `mysqld --initialize` - Newer method (creates random root password)
  - `mysqld --initialize-insecure` - Newer method (no root password)
- **Post-initialization steps:**
  - Start the service
  - Enable auto-start
  - Run mysql_secure_installation
  - Configure remote access (if needed)
  - Create application databases/users
- **Common errors without initialization:**
  - "Can't find file: './mysql/user.frm'"
  - "Fatal error: Can't open privilege tables"
  - "Table 'mysql.user' doesn't exist"
  - Service fails to start immediately
- **Difference from other DB issues:**
  - Service stopped: Just restart
  - Data directory missing: Must initialize first
  - Corrupted data: Restore from backup
  - Config error: Fix configuration
- **Best practices:**
  - Always initialize after fresh install
  - Use configuration management for consistency
  - Document initialization procedure
  - Back up data directory regularly
  - Use persistent volumes in containers
  - Monitor data directory health
- **Security considerations:**
  - Run mysql_secure_installation after initialization
  - Set root password
  - Remove anonymous users
  - Disable remote root login
  - Remove test database

## ⚠️ Important Notes

- Never start MariaDB without initializing data directory first
- `mysql_install_db` must be run as mysql user or with --user=mysql
- Always check if data directory exists before attempting start
- Initialization creates default root user with no password
- Must run mysql_secure_installation in production
- Data directory location defined in /etc/my.cnf (datadir parameter)
- Initialization is one-time operation - don't run on existing database!
- Running mysql_install_db on existing data can corrupt it
- Always backup before any database maintenance
- Document the root cause for post-mortem analysis
- This issue is common in containerized environments without persistent storage
- Different from disk space issues - logs would show different errors
- systemctl status might not show clear error about missing data directory
- Always check both logs AND data directory existence
- Enable service to prevent same issue after reboot

## 🐛 Troubleshooting Steps Used

```bash
# 1. Check service status
systemctl status mariadb
# Showed: inactive (dead) or failed

# 2. Check logs
journalctl -u mariadb -n 50
# May show generic errors

# 3. Check disk space
df -h
# Confirmed space available (21% used)

# 4. Check configuration
cat /etc/my.cnf
# Configuration exists and valid

# 5. Check data directory (EUREKA moment!)
ls -la /var/lib/mysql
# ERROR: No such file or directory
# This revealed the root cause!

# 6. Solution path
mkdir -p /var/lib/mysql
chown -R mysql:mysql /var/lib/mysql
mysql_install_db --user=mysql --datadir=/var/lib/mysql
systemctl start mariadb
systemctl enable mariadb
```

## 📊 Real-World Scenario

**Common in:**
- Docker containers without persistent volumes
- Fresh server provisioning
- Automated deployments without proper initialization
- VM cloning without data directory
- Kubernetes pods without PersistentVolumeClaims
- Configuration management tools missing initialization step

**Impact:**
- Complete database unavailability
- Application cannot connect
- No data persistence possible
- Service won't start at all

**Prevention:**
- Include database initialization in deployment scripts
- Use persistent storage in containerized environments
- Configuration management should handle initialization
- Document initialization in runbooks
- Test deployments in staging first
- Monitor data directory existence

---

**Date Completed**: November 14, 2025  
**Challenge**: KodeKloud 100 Days Cloud DevOps  
**Day**: 9/100