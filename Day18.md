# Day 18: Deploying WordPress Infrastructure with Apache, PHP, and MariaDB
**100 Days Cloud DevOps Challenge – KodeKloud**

---

## Objective

xFusionCorp Industries is planning to host a WordPress website on their infrastructure in Stratos Datacenter. The infrastructure configuration is already done with a shared directory `/var/www/html` mounted on all app hosts.

**Requirements:**
- Install httpd, php and its dependencies on all app hosts
- Apache should serve on port `3004` within the apps
- Install/Configure MariaDB server on DB Server
- Create database `kodekloud_db2` and user `kodekloud_roy` with password `Rc5C9EyvbU`
- Grant all privileges to the user on the database
- Verify website accessibility through LBR link showing database connection

---

## Infrastructure Overview

### Application Servers:
| Server  | User   | Password | IP            |
|---------|--------|----------|---------------|
| stapp01 | tony   | Ir0nM@n  | 172.16.238.10 |
| stapp02 | steve  | Am3ric@  | 172.16.238.11 |
| stapp03 | banner | BigGr33n | 172.16.238.12 |

### Database Server:
| Server | User  | Password |
|--------|-------|----------|
| stdb01 | peter | Sp!dy    |

### Shared Storage:
- **Storage Path:** `/var/www/html` (mounted on all app servers)

---

## Step-by-Step Implementation

### Part 1: Configure Application Servers (All 3 Servers)

Execute these steps on **stapp01**, **stapp02**, and **stapp03**.

#### Step 1.1: SSH into App Server 1
```bash
ssh tony@stapp01
sudo su -
```

#### Step 1.2: Install Apache, PHP and Dependencies
```bash
yum install -y httpd php php-mysqlnd php-fpm php-json
```

#### Step 1.3: Configure Apache to Listen on Port 3004
```bash
sed -i 's/Listen 80/Listen 3004/g' /etc/httpd/conf/httpd.conf
```

**Verify the change:**
```bash
grep "Listen" /etc/httpd/conf/httpd.conf
```

**Expected output:**
```
Listen 3004
```

#### Step 1.4: Start and Enable Apache
```bash
systemctl start httpd
systemctl enable httpd
systemctl status httpd
```

#### Step 1.5: Verify Apache is Listening on Port 3004
```bash
ss -tulpn | grep httpd
```

**Expected output:**
```
tcp   LISTEN 0   511   *:3004   *:*   users:(("httpd",pid=...))
```

#### Step 1.6: Repeat for App Server 2
```bash
ssh steve@stapp02
sudo su -
yum install -y httpd php php-mysqlnd php-fpm php-json
sed -i 's/Listen 80/Listen 3004/g' /etc/httpd/conf/httpd.conf
systemctl start httpd
systemctl enable httpd
ss -tulpn | grep httpd
```

#### Step 1.7: Repeat for App Server 3
```bash
ssh banner@stapp03
sudo su -
yum install -y httpd php php-mysqlnd php-fpm php-json
sed -i 's/Listen 80/Listen 3004/g' /etc/httpd/conf/httpd.conf
systemctl start httpd
systemctl enable httpd
ss -tulpn | grep httpd
```

---

### Part 2: Configure Database Server

#### Step 2.1: SSH into Database Server
```bash
ssh peter@stdb01
sudo su -
```

#### Step 2.2: Install MariaDB Server
```bash
yum install -y mariadb-server
```

#### Step 2.3: Start and Enable MariaDB
```bash
systemctl start mariadb
systemctl enable mariadb
systemctl status mariadb
```

#### Step 2.4: Access MariaDB Shell
```bash
mysql
```

#### Step 2.5: Create Database
```sql
CREATE DATABASE kodekloud_db2;
```

**Output:**
```
Query OK, 1 row affected (0.00 sec)
```

#### Step 2.6: Create Database User
```sql
CREATE USER 'kodekloud_roy'@'%' IDENTIFIED BY 'Rc5C9EyvbU';
```

**Note:** The `'%'` allows connection from any host.

**Output:**
```
Query OK, 0 rows affected (0.00 sec)
```

#### Step 2.7: Grant All Privileges
```sql
GRANT ALL PRIVILEGES ON kodekloud_db2.* TO 'kodekloud_roy'@'%';
```

**Output:**
```
Query OK, 0 rows affected (0.00 sec)
```

#### Step 2.8: Flush Privileges
```sql
FLUSH PRIVILEGES;
```

#### Step 2.9: Verify Database and User
```sql
SHOW DATABASES;
```
```sql
SELECT User, Host FROM mysql.user WHERE User = 'kodekloud_roy';
```

**Expected output:**
```
+---------------+------+
| User          | Host |
+---------------+------+
| kodekloud_roy | %    |
+---------------+------+
```

#### Step 2.10: Exit MySQL
```sql
EXIT;
```

---

### Part 3: Create Database Connection Test File

#### Step 3.1: SSH to App Server 1
```bash
ssh tony@stapp01
sudo su -
```

#### Step 3.2: Create PHP Test File

**Method 1: Using cat command (Recommended)**
```bash
cat > /var/www/html/index.php <<'EOF'
<?php
$servername = "stdb01";
$username = "kodekloud_roy";
$password = "Rc5C9EyvbU";
$dbname = "kodekloud_db2";

$conn = new mysqli($servername, $username, $password, $dbname);

if ($conn->connect_error) {
    die("Connection failed: " . $conn->connect_error);
}
echo "App is able to connect to the database using user kodekloud_roy";

$conn->close();
?>
EOF
```

**Important Note:** Use `'EOF'` with single quotes to prevent bash from interpreting PHP variables.

**Method 2: Using vi editor**
```bash
vi /var/www/html/index.php
```

1. Press `i` to enter insert mode
2. Paste the following code:
```php
<?php
$servername = "stdb01";
$username = "kodekloud_roy";
$password = "Rc5C9EyvbU";
$dbname = "kodekloud_db2";

$conn = new mysqli($servername, $username, $password, $dbname);

if ($conn->connect_error) {
    die("Connection failed: " . $conn->connect_error);
}
echo "App is able to connect to the database using user kodekloud_roy";

$conn->close();
?>
```

3. Press `Esc` to exit insert mode
4. Type `:wq` and press `Enter` to save and quit

#### Step 3.3: Set Proper Permissions
```bash
chown apache:apache /var/www/html/index.php
chmod 644 /var/www/html/index.php
```

#### Step 3.4: Verify File Contents
```bash
cat /var/www/html/index.php
```

---

### Part 4: Verification

#### Step 4.1: Test from App Server
```bash
curl http://localhost:3004
```

**Expected output:**
```
App is able to connect to the database using user kodekloud_roy
```

#### Step 4.2: Test from Jump Host
```bash
curl http://stapp01:3004
curl http://stapp02:3004
curl http://stapp03:3004
```

#### Step 4.3: Test via Load Balancer

Click the **App** button on the KodeKloud top bar.

**Expected:** You should see the message:
```
App is able to connect to the database using user kodekloud_roy
```

---

## Complete Command Summary

### On All App Servers (stapp01, stapp02, stapp03):
```bash
sudo su -
yum install -y httpd php php-mysqlnd php-fpm php-json
sed -i 's/Listen 80/Listen 3004/g' /etc/httpd/conf/httpd.conf
systemctl start httpd
systemctl enable httpd
ss -tulpn | grep httpd
```

### On Database Server (stdb01):
```bash
sudo su -
yum install -y mariadb-server
systemctl start mariadb
systemctl enable mariadb
mysql
```

**SQL Commands:**
```sql
CREATE DATABASE kodekloud_db2;
CREATE USER 'kodekloud_roy'@'%' IDENTIFIED BY 'Rc5C9EyvbU';
GRANT ALL PRIVILEGES ON kodekloud_db2.* TO 'kodekloud_roy'@'%';
FLUSH PRIVILEGES;
SHOW DATABASES;
SELECT User, Host FROM mysql.user WHERE User = 'kodekloud_roy';
EXIT;
```

### Create Test File (on stapp01):
```bash
sudo su -

cat > /var/www/html/index.php <<'EOF'
<?php
$servername = "stdb01";
$username = "kodekloud_roy";
$password = "Rc5C9EyvbU";
$dbname = "kodekloud_db2";

$conn = new mysqli($servername, $username, $password, $dbname);

if ($conn->connect_error) {
    die("Connection failed: " . $conn->connect_error);
}
echo "App is able to connect to the database using user kodekloud_roy";

$conn->close();
?>
EOF

chown apache:apache /var/www/html/index.php
chmod 644 /var/www/html/index.php

# Verify
cat /var/www/html/index.php

# Test
curl http://localhost:3004
```

---

## Troubleshooting

### Issue 1: Bash Syntax Error When Creating PHP File

**Problem:**
```bash
-bash: syntax error near unexpected token `('
```

**Cause:** Trying to run PHP code directly in bash shell

**Solution:** Use heredoc with single quotes:
```bash
cat > /var/www/html/index.php <<'EOF'
<?php code here ?>
EOF
```

The `'EOF'` (with quotes) prevents bash from interpreting PHP variables.

### Issue 2: Apache Not Starting

**Check logs:**
```bash
journalctl -u httpd -n 50
```

**Check if port is already in use:**
```bash
ss -tulpn | grep 3004
```

### Issue 3: PHP Code Showing Instead of Executing

**Check PHP module is loaded:**
```bash
httpd -M | grep php
```

**Restart Apache:**
```bash
systemctl restart httpd
```

### Issue 4: Database Connection Failed

**Test database connectivity:**
```bash
mysql -h stdb01 -u kodekloud_roy -p'Rc5C9EyvbU' kodekloud_db2
```

**Check MariaDB is listening:**
```bash
ss -tulpn | grep mysql
```

**Verify user privileges in MySQL:**
```sql
SHOW GRANTS FOR 'kodekloud_roy'@'%';
```

### Issue 5: Permission Denied on /var/www/html

**Check file permissions:**
```bash
ls -la /var/www/html/index.php
```

**Set correct ownership:**
```bash
chown apache:apache /var/www/html/index.php
chmod 644 /var/www/html/index.php
```

**Check SELinux context:**
```bash
ls -Z /var/www/html/index.php
```

**Restore SELinux context if needed:**
```bash
restorecon -Rv /var/www/html
```

### Issue 6: 502 Bad Gateway

**Possible causes:**
- Apache not running on app servers
- Wrong port configuration
- PHP-FPM issues

**Check Apache status:**
```bash
systemctl status httpd
```

**Verify port configuration:**
```bash
grep Listen /etc/httpd/conf/httpd.conf
```

---

## Key Commands Reference

### Apache Commands

| Command | Description |
|---------|-------------|
| `systemctl start httpd` | Start Apache service |
| `systemctl stop httpd` | Stop Apache service |
| `systemctl restart httpd` | Restart Apache service |
| `systemctl status httpd` | Check Apache status |
| `systemctl enable httpd` | Enable Apache at boot |
| `httpd -t` | Test Apache configuration |
| `httpd -M` | List loaded Apache modules |

### MariaDB/MySQL Commands

| Command | Description |
|---------|-------------|
| `systemctl start mariadb` | Start MariaDB service |
| `systemctl status mariadb` | Check MariaDB status |
| `mysql` | Access MySQL shell |
| `mysql -u user -p` | Login with username |
| `SHOW DATABASES;` | List all databases |
| `USE database;` | Switch to database |
| `SHOW TABLES;` | List tables in database |
| `EXIT;` | Exit MySQL shell |

### File Operations

| Command | Description |
|---------|-------------|
| `cat > file <<'EOF'` | Create file with heredoc |
| `vi file` | Edit file with vi |
| `cat file` | Display file contents |
| `chown user:group file` | Change file ownership |
| `chmod 644 file` | Change file permissions |
| `ls -la file` | List file details |

---

## Security Best Practices

1. **Database User Host Restriction:**
```sql
   -- Instead of '%', use specific IP range
   CREATE USER 'kodekloud_roy'@'172.16.238.%' IDENTIFIED BY 'Rc5C9EyvbU';
```

2. **Strong Passwords:**
   - Use complex passwords for production environments
   - Avoid hardcoding passwords in PHP files
   - Use environment variables or configuration files

3. **File Permissions:**
   - Keep `/var/www/html` files owned by `apache:apache`
   - Use `644` for files and `755` for directories
   - Never use `777` permissions

4. **Firewall Configuration:**
```bash
   firewall-cmd --add-port=3004/tcp --permanent
   firewall-cmd --reload
```

5. **Secure MariaDB Installation:**
```bash
   mysql_secure_installation
```

---

## Key Takeaways

- Apache can be configured to listen on custom ports via `Listen` directive in `httpd.conf`
- PHP requires `php-mysqlnd` extension for MySQL/MariaDB connectivity
- MariaDB user must have proper privileges and host access (`%` for any host)
- Shared storage (`/var/www/html`) enables consistent content across app servers
- Use heredoc with single quotes (`<<'EOF'`) to prevent bash variable interpolation
- Always verify each component individually before testing end-to-end
- File ownership and permissions are critical for web server functionality
- `mysqli` provides object-oriented database connectivity in PHP

---

## Completion Checklist

- [x] Apache installed on all 3 app servers
- [x] Apache configured to listen on port 3004
- [x] PHP and required extensions installed
- [x] MariaDB installed and started on database server
- [x] Database `kodekloud_db2` created
- [x] User `kodekloud_roy` created with password `Rc5C9EyvbU`
- [x] Full privileges granted to user on database
- [x] PHP test file created in `/var/www/html/index.php`
- [x] Proper file permissions set
- [x] Database connection verified locally
- [x] Connection verified via Load Balancer
- [x] Success message displayed: "App is able to connect to the database using user kodekloud_roy"

---

## Completion Details

- **Completion Date:** November 23, 2025
- **Day:** 18 / 100
- **Challenge:** KodeKloud 100 Days Cloud DevOps
- **Topic:** WordPress Infrastructure Setup with LAMP Stack
- **Status:** ✅ Successfully Completed

---
