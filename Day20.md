# Day 20: Deploying PHP-FPM 8.3 with Nginx on Custom Port
**100 Days Cloud DevOps Challenge – KodeKloud**

---

## Objective

The Nautilus application development team is planning to launch a new PHP-based application on Nautilus infrastructure in Stratos DC. We need to set up the infrastructure as per their requirements.

**Requirements:**
- Install `nginx` on App Server 1 and configure it to use port `8096`
- Set document root to `/var/www/html`
- Install `php-fpm` version `8.3` on App Server 1
- Configure PHP-FPM to use unix socket `/var/run/php-fpm/default.sock`
- Configure PHP-FPM and Nginx to work together
- Verify setup using `curl http://stapp01:8096/index.php` from jump host

**Note:** Two files (`index.php` and `info.php`) are already present in `/var/www/html` - do not modify them.

---

## Infrastructure Overview

### Application Server:
| Server  | User | Password | IP            |
|---------|------|----------|---------------|
| stapp01 | tony | Ir0nM@n  | 172.16.238.10 |

### Configuration Details:
- **Nginx Port:** 8096
- **Document Root:** `/var/www/html`
- **PHP-FPM Version:** 8.3
- **PHP-FPM Socket:** `/var/run/php-fpm/default.sock`
- **OS:** CentOS Stream 9

---

## Step-by-Step Implementation

### Part 1: Verify System and Install Nginx

#### Step 1.1: SSH into App Server 1
```bash
ssh tony@stapp01
```

Password: `Ir0nM@n`

#### Step 1.2: Switch to Root User
```bash
sudo su -
```

#### Step 1.3: Check OS Version
```bash
cat /etc/os-release
```

**Expected output:**
```
NAME="CentOS Stream"
VERSION="9"
ID="centos"
VERSION_ID="9"
PLATFORM_ID="platform:el9"
```

#### Step 1.4: Install Nginx
```bash
dnf install -y nginx
```

#### Step 1.5: Configure Nginx to Listen on Port 8096
```bash
sed -i 's/listen       80;/listen       8096;/g' /etc/nginx/nginx.conf
sed -i 's/listen       \[::\]:80;/listen       [::]:8096;/g' /etc/nginx/nginx.conf
```

#### Step 1.6: Configure Document Root
```bash
sed -i 's|root         /usr/share/nginx/html;|root         /var/www/html;|g' /etc/nginx/nginx.conf
```

#### Step 1.7: Verify Nginx Configuration
```bash
nginx -t
```

**Expected output:**
```
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

---

### Part 2: Install PHP-FPM 8.3

#### Step 2.1: Install EPEL Repository
```bash
dnf install -y epel-release
```

#### Step 2.2: Install Remi Repository for CentOS Stream 9
```bash
dnf install -y https://rpms.remirepo.net/enterprise/remi-release-9.rpm
```

#### Step 2.3: Reset PHP Module
```bash
dnf module reset php -y
```

#### Step 2.4: Install PHP 8.3 Module
```bash
dnf module install php:remi-8.3 -y
```

#### Step 2.5: Install PHP-FPM and Extensions
```bash
dnf install -y php php-fpm php-cli php-mysqlnd php-opcache php-xml php-mbstring
```

#### Step 2.6: Verify PHP Installation
```bash
php -v
```

**Expected output:**
```
PHP 8.3.28 (cli) (built: Nov 18 2025 22:17:16) (NTS gcc x86_64)
Copyright (c) The PHP Group
Zend Engine v4.3.28, Copyright (c) Zend Technologies
    with Zend OPcache v8.3.28, Copyright (c), by Zend Technologies
```
```bash
php-fpm -v
```

**Expected output:**
```
PHP 8.3.28 (fpm-fcgi) (built: Nov 18 2025 22:17:16)
Copyright (c) The PHP Group
Zend Engine v4.3.28, Copyright (c) Zend Technologies
    with Zend OPcache v8.3.28, Copyright (c), by Zend Technologies
```

---

### Part 3: Configure PHP-FPM

#### Step 3.1: Create PHP-FPM Socket Directory
```bash
mkdir -p /var/run/php-fpm
```

#### Step 3.2: Configure PHP-FPM to Use Custom Socket
```bash
sed -i 's|listen = /run/php-fpm/www.sock|listen = /var/run/php-fpm/default.sock|g' /etc/php-fpm.d/www.conf
```

#### Step 3.3: Change PHP-FPM User and Group to Nginx
```bash
sed -i 's/^user = apache/user = nginx/g' /etc/php-fpm.d/www.conf
sed -i 's/^group = apache/group = nginx/g' /etc/php-fpm.d/www.conf
```

#### Step 3.4: Set Socket Permissions
```bash
sed -i 's/;listen.owner = nginx/listen.owner = nginx/g' /etc/php-fpm.d/www.conf
sed -i 's/;listen.group = nginx/listen.group = nginx/g' /etc/php-fpm.d/www.conf
sed -i 's/;listen.mode = 0660/listen.mode = 0660/g' /etc/php-fpm.d/www.conf
```

#### Step 3.5: Verify PHP-FPM Configuration
```bash
grep "^listen = " /etc/php-fpm.d/www.conf
grep "^user = " /etc/php-fpm.d/www.conf
grep "^group = " /etc/php-fpm.d/www.conf
```

**Expected output:**
```
listen = /var/run/php-fpm/default.sock
user = nginx
group = nginx
```

#### Step 3.6: Test PHP-FPM Configuration
```bash
php-fpm -t
```

**Expected output:**
```
[25-Nov-2025 05:00:00] NOTICE: configuration file /etc/php-fpm.conf test is successful
```

---

### Part 4: Configure Nginx to Work with PHP-FPM

#### Step 4.1: Create PHP Configuration for Nginx
```bash
cat > /etc/nginx/default.d/php.conf <<'EOF'
location ~ \.php$ {
    fastcgi_pass unix:/var/run/php-fpm/default.sock;
    fastcgi_index index.php;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    include fastcgi_params;
}

location / {
    index index.php index.html index.htm;
}
EOF
```

#### Step 4.2: Verify PHP Configuration File
```bash
cat /etc/nginx/default.d/php.conf
```

#### Step 4.3: Test Nginx Configuration
```bash
nginx -t
```

---

### Part 5: Set Permissions and Verify Files

#### Step 5.1: Set Proper Ownership on Document Root
```bash
chown -R nginx:nginx /var/www/html
chmod -R 755 /var/www/html
```

#### Step 5.2: Verify PHP Files Exist
```bash
ls -la /var/www/html/
```

**Expected output:**
```
total 8
drwxr-xr-x. 2 nginx nginx   40 Nov 25 05:00 .
drwxr-xr-x. 3 root  root    18 Nov 25 05:00 ..
-rw-r--r--. 1 nginx nginx  xxx Nov 25 05:00 index.php
-rw-r--r--. 1 nginx nginx  xxx Nov 25 05:00 info.php
```

---

### Part 6: Start Services

#### Step 6.1: Start and Enable PHP-FPM
```bash
systemctl start php-fpm
systemctl enable php-fpm
systemctl status php-fpm
```

**Expected status:** `active (running)`

#### Step 6.2: Start and Enable Nginx
```bash
systemctl start nginx
systemctl enable nginx
systemctl status nginx
```

**Expected status:** `active (running)`

---

### Part 7: Verification

#### Step 7.1: Verify Socket is Created
```bash
ls -la /var/run/php-fpm/
```

**Expected output:**
```
srw-rw----. 1 nginx nginx 0 Nov 25 05:00 default.sock
```

#### Step 7.2: Verify Nginx is Listening on Port 8096
```bash
ss -tulpn | grep nginx
```

**Expected output:**
```
tcp   LISTEN 0   511   *:8096   *:*   users:(("nginx",pid=...))
```

#### Step 7.3: Test PHP Processing Locally
```bash
curl http://localhost:8096/index.php
```

**Expected:** PHP output from index.php
```bash
curl http://localhost:8096/info.php
```

**Expected:** PHP output from info.php

#### Step 7.4: Check HTTP Headers
```bash
curl -I http://localhost:8096/index.php
```

**Expected output:**
```
HTTP/1.1 200 OK
Server: nginx/1.x.x
Content-Type: text/html; charset=UTF-8
```

#### Step 7.5: Test from Jump Host

Exit from the app server:
```bash
exit
exit
```

From jump host, test the connection:
```bash
curl http://stapp01:8096/index.php
curl http://stapp01:8096/info.php
```

**Expected:** Both should return PHP output successfully.

---

## Complete Command Summary

### All Commands in One Block:
```bash
# SSH and become root
ssh tony@stapp01
sudo su -

# Part 1: Install and Configure Nginx
dnf install -y nginx
sed -i 's/listen       80;/listen       8096;/g' /etc/nginx/nginx.conf
sed -i 's/listen       \[::\]:80;/listen       [::]:8096;/g' /etc/nginx/nginx.conf
sed -i 's|root         /usr/share/nginx/html;|root         /var/www/html;|g' /etc/nginx/nginx.conf
nginx -t

# Part 2: Install PHP-FPM 8.3
dnf install -y epel-release
dnf install -y https://rpms.remirepo.net/enterprise/remi-release-9.rpm
dnf module reset php -y
dnf module install php:remi-8.3 -y
dnf install -y php php-fpm php-cli php-mysqlnd php-opcache php-xml php-mbstring
php -v
php-fpm -v

# Part 3: Configure PHP-FPM
mkdir -p /var/run/php-fpm
sed -i 's|listen = /run/php-fpm/www.sock|listen = /var/run/php-fpm/default.sock|g' /etc/php-fpm.d/www.conf
sed -i 's/^user = apache/user = nginx/g' /etc/php-fpm.d/www.conf
sed -i 's/^group = apache/group = nginx/g' /etc/php-fpm.d/www.conf
sed -i 's/;listen.owner = nginx/listen.owner = nginx/g' /etc/php-fpm.d/www.conf
sed -i 's/;listen.group = nginx/listen.group = nginx/g' /etc/php-fpm.d/www.conf
sed -i 's/;listen.mode = 0660/listen.mode = 0660/g' /etc/php-fpm.d/www.conf
php-fpm -t

# Part 4: Configure Nginx for PHP
cat > /etc/nginx/default.d/php.conf <<'EOF'
location ~ \.php$ {
    fastcgi_pass unix:/var/run/php-fpm/default.sock;
    fastcgi_index index.php;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    include fastcgi_params;
}

location / {
    index index.php index.html index.htm;
}
EOF

# Part 5: Set Permissions
chown -R nginx:nginx /var/www/html
chmod -R 755 /var/www/html

# Part 6: Start Services
nginx -t
systemctl start php-fpm
systemctl enable php-fpm
systemctl start nginx
systemctl enable nginx

# Part 7: Verify
ls -la /var/run/php-fpm/
ss -tulpn | grep nginx
curl http://localhost:8096/index.php
curl http://localhost:8096/info.php
```

---

## Troubleshooting

### Issue 1: Remi Repository Installation Failed

**Problem:**
```
Error: nothing provides epel-release = 8
```

**Cause:** Wrong OS version detected or EPEL not installed

**Solution:**
```bash
# Check OS version first
cat /etc/os-release

# For CentOS Stream 9
dnf install -y epel-release
dnf install -y https://rpms.remirepo.net/enterprise/remi-release-9.rpm

# For CentOS Stream 8
dnf install -y epel-release
dnf install -y https://rpms.remirepo.net/enterprise/remi-release-8.rpm

# For CentOS 7
yum install -y epel-release
yum install -y https://rpms.remirepo.net/enterprise/remi-release-7.rpm
```

### Issue 2: PHP Module Not Found

**Problem:**
```
Error: Problems in request:
missing groups or modules: php:remi-8.3
```

**Solution:**
```bash
# List available PHP modules
dnf module list php

# If remi-8.3 not available, check repositories
dnf repolist

# Clean cache and try again
dnf clean all
dnf makecache
dnf module reset php -y
dnf module install php:remi-8.3 -y
```

### Issue 3: 502 Bad Gateway

**Symptoms:** Nginx returns 502 error when accessing PHP files

**Causes and Solutions:**
```bash
# Check PHP-FPM is running
systemctl status php-fpm

# If not running, start it
systemctl start php-fpm

# Check socket exists
ls -la /var/run/php-fpm/default.sock

# If socket doesn't exist, check PHP-FPM logs
tail -f /var/log/php-fpm/error.log

# Verify socket path in both configs
grep "listen = " /etc/php-fpm.d/www.conf
grep "fastcgi_pass" /etc/nginx/default.d/php.conf

# Check socket permissions
chmod 660 /var/run/php-fpm/default.sock
chown nginx:nginx /var/run/php-fpm/default.sock
```

### Issue 4: PHP Files Download Instead of Execute

**Cause:** Nginx not configured to pass PHP files to PHP-FPM

**Solution:**
```bash
# Verify PHP config exists
cat /etc/nginx/default.d/php.conf

# If missing, recreate it
cat > /etc/nginx/default.d/php.conf <<'EOF'
location ~ \.php$ {
    fastcgi_pass unix:/var/run/php-fpm/default.sock;
    fastcgi_index index.php;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    include fastcgi_params;
}

location / {
    index index.php index.html index.htm;
}
EOF

# Test and reload Nginx
nginx -t
systemctl reload nginx
```

### Issue 5: Permission Denied

**Check file ownership:**
```bash
ls -la /var/www/html/
```

**Fix permissions:**
```bash
chown -R nginx:nginx /var/www/html
chmod -R 755 /var/www/html
chmod 644 /var/www/html/*.php
```

**Check SELinux context:**
```bash
ls -Z /var/www/html/
```

**Restore SELinux context:**
```bash
restorecon -Rv /var/www/html/
```

### Issue 6: SELinux Blocking

**Check SELinux status:**
```bash
getenforce
```

**Check for denials:**
```bash
ausearch -m avc -ts recent | grep nginx
```

**Allow Nginx to connect to PHP-FPM socket:**
```bash
setsebool -P httpd_can_network_connect 1
```

**Check SELinux context on socket:**
```bash
ls -Z /var/run/php-fpm/default.sock
```

### Issue 7: Service Won't Start

**Check service status:**
```bash
systemctl status php-fpm
systemctl status nginx
```

**View detailed logs:**
```bash
journalctl -u php-fpm -n 50
journalctl -u nginx -n 50
```

**Check error logs:**
```bash
tail -50 /var/log/php-fpm/error.log
tail -50 /var/log/nginx/error.log
```

---

## Configuration Files Reference

### Nginx Main Configuration (`/etc/nginx/nginx.conf`)

Key sections modified:
```nginx
http {
    # ... other settings ...

    server {
        listen       8096;
        listen       [::]:8096;
        server_name  _;
        root         /var/www/html;

        include /etc/nginx/default.d/*.conf;

        # ... error pages ...
    }
}
```

### PHP Configuration for Nginx (`/etc/nginx/default.d/php.conf`)
```nginx
location ~ \.php$ {
    fastcgi_pass unix:/var/run/php-fpm/default.sock;
    fastcgi_index index.php;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    include fastcgi_params;
}

location / {
    index index.php index.html index.htm;
}
```

### PHP-FPM Pool Configuration (`/etc/php-fpm.d/www.conf`)

Key settings:
```ini
[www]
user = nginx
group = nginx

listen = /var/run/php-fpm/default.sock

listen.owner = nginx
listen.group = nginx
listen.mode = 0660

pm = dynamic
pm.max_children = 50
pm.start_servers = 5
pm.min_spare_servers = 5
pm.max_spare_servers = 35
```

---

## Key Commands Reference

### System Information

| Command | Description |
|---------|-------------|
| `cat /etc/os-release` | Check OS version |
| `uname -r` | Check kernel version |
| `hostnamectl` | Display system information |

### Nginx Commands

| Command | Description |
|---------|-------------|
| `nginx -t` | Test configuration syntax |
| `nginx -T` | Test and dump configuration |
| `nginx -s reload` | Reload configuration |
| `systemctl start nginx` | Start Nginx |
| `systemctl stop nginx` | Stop Nginx |
| `systemctl restart nginx` | Restart Nginx |
| `systemctl status nginx` | Check Nginx status |
| `systemctl enable nginx` | Enable at boot |

### PHP-FPM Commands

| Command | Description |
|---------|-------------|
| `php -v` | Check PHP CLI version |
| `php-fpm -v` | Check PHP-FPM version |
| `php-fpm -t` | Test PHP-FPM configuration |
| `php -m` | List loaded PHP modules |
| `systemctl start php-fpm` | Start PHP-FPM |
| `systemctl stop php-fpm` | Stop PHP-FPM |
| `systemctl restart php-fpm` | Restart PHP-FPM |
| `systemctl status php-fpm` | Check PHP-FPM status |
| `systemctl enable php-fpm` | Enable at boot |

### DNF/Package Management

| Command | Description |
|---------|-------------|
| `dnf module list php` | List PHP modules |
| `dnf module reset php` | Reset PHP module |
| `dnf module install php:remi-8.3` | Install PHP 8.3 module |
| `dnf search php` | Search PHP packages |
| `rpm -qa \| grep php` | List installed PHP packages |

---

## Communication Flow
```
Client Request → Port 8096
         ↓
    Nginx (listening on 8096)
         ↓
    Checks file extension (.php)
         ↓
    Routes to FastCGI
         ↓
    Unix Socket (/var/run/php-fpm/default.sock)
         ↓
    PHP-FPM Worker Process
         ↓
    Executes PHP Code
         ↓
    Returns Output to Nginx
         ↓
    Nginx Sends Response to Client
```

---

## Key Takeaways

- **OS-Specific Repository:** CentOS Stream 9 requires `remi-release-9.rpm`
- **Unix Sockets:** Faster than TCP sockets for local communication between Nginx and PHP-FPM
- **Socket Permissions:** Critical - socket must be readable by nginx user (owner: nginx, group: nginx, mode: 0660)
- **Configuration Matching:** Socket path must match exactly in both Nginx and PHP-FPM configs
- **User/Group Alignment:** PHP-FPM must run as `nginx` user for proper file access
- **Module System:** CentOS/RHEL 8+ uses DNF modules for PHP version management
- **Testing First:** Always test configurations (`nginx -t`, `php-fpm -t`) before starting services
- **Service Dependencies:** PHP-FPM must be running before Nginx can process PHP files
- **Document Root Permissions:** Files must be owned by nginx:nginx with proper permissions (755 for directories, 644 for files)

---

## Security Best Practices

1. **Socket Permissions:**
```bash
   # Socket should only be accessible by nginx
   listen.mode = 0660
   listen.owner = nginx
   listen.group = nginx
```

2. **File Permissions:**
```bash
   # Web files should not be writable by web server
   chown nginx:nginx /var/www/html
   chmod 755 /var/www/html
   chmod 644 /var/www/html/*.php
```

3. **PHP Security Settings:**
```ini
   # In /etc/php.ini
   expose_php = Off
   display_errors = Off
   log_errors = On
```

4. **Nginx Security:**
```nginx
   # Hide Nginx version
   server_tokens off;
```

---

## Performance Optimization

### PHP-FPM Pool Settings

For better performance, tune these in `/etc/php-fpm.d/www.conf`:
```ini
pm = dynamic
pm.max_children = 50          # Adjust based on available RAM
pm.start_servers = 5
pm.min_spare_servers = 5
pm.max_spare_servers = 35
pm.max_requests = 500         # Restart worker after N requests
```

### OPcache Settings

Enable OPcache for better PHP performance in `/etc/php.d/10-opcache.ini`:
```ini
opcache.enable=1
opcache.memory_consumption=128
opcache.interned_strings_buffer=8
opcache.max_accelerated_files=4000
opcache.revalidate_freq=60
```

---

## Completion Checklist

- [x] Nginx installed on App Server 1
- [x] Nginx configured to listen on port 8096
- [x] Document root set to `/var/www/html`
- [x] EPEL repository installed
- [x] Remi repository for CentOS Stream 9 installed
- [x] PHP 8.3 module enabled
- [x] PHP-FPM 8.3.28 installed and verified
- [x] Socket directory created at `/var/run/php-fpm/`
- [x] PHP-FPM configured to use socket `/var/run/php-fpm/default.sock`
- [x] PHP-FPM user/group set to nginx
- [x] Socket permissions configured (owner: nginx, group: nginx, mode: 0660)
- [x] Nginx configured to pass PHP requests to PHP-FPM via socket
- [x] Proper file permissions set on `/var/www/html`
- [x] Both services started and enabled
- [x] Socket file created and accessible
- [x] Port 8096 listening verified
- [x] `curl http://localhost:8096/index.php` works locally
- [x] `curl http://stapp01:8096/index.php` works from jump host
- [x] Both index.php and info.php accessible and executing properly

---

## Completion Details

- **Completion Date:** November 25, 2025
- **Day:** 20 / 100
- **Challenge:** KodeKloud 100 Days Cloud DevOps
- **Topic:** PHP-FPM 8.3 with Nginx on Custom Port using Unix Socket
- **OS:** CentOS Stream 9
- **PHP Version:** 8.3.28
- **Nginx Port:** 8096
- **Status:** ✅ Successfully Completed

---

