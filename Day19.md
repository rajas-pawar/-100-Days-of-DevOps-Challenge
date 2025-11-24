# Day 19: Hosting Multiple Static Websites on Apache with Custom Ports
**100 Days Cloud DevOps Challenge – KodeKloud**

---

## Objective

xFusionCorp Industries is planning to host two static websites on their infrastructure in Stratos Datacenter. The development is still in progress, but we need to prepare the servers.

**Requirements:**
- Install `httpd` package and dependencies on App Server 1
- Configure Apache to serve on port `3003`
- Deploy two website backups from jump host: `ecommerce` and `apps`
- `ecommerce` should be accessible at `http://localhost:3003/ecommerce/`
- `apps` should be accessible at `http://localhost:3003/apps/`
- Verify access using curl commands

---

## Infrastructure Overview

### Application Server:
| Server  | User | Password | IP            |
|---------|------|----------|---------------|
| stapp01 | tony | Ir0nM@n  | 172.16.238.10 |

### Jump Host:
| Server    | User | Website Backups Location |
|-----------|------|--------------------------|
| jump_host | thor | `/home/thor/ecommerce`   |
|           |      | `/home/thor/apps`        |

---

## Step-by-Step Implementation

### Part 1: Copy Website Backups from Jump Host to App Server

#### Step 1.1: Copy Websites to App Server 1

From the **jump host**, copy both website directories:
```bash
scp -r /home/thor/ecommerce tony@stapp01:/tmp/
scp -r /home/thor/apps tony@stapp01:/tmp/
```

Enter password: `Ir0nM@n` when prompted (twice, once for each copy).

**Alternative: Copy both in one command**
```bash
scp -r /home/thor/ecommerce /home/thor/apps tony@stapp01:/tmp/
```

---

### Part 2: Configure Apache on App Server 1

#### Step 2.1: SSH into App Server 1
```bash
ssh tony@stapp01
```

Password: `Ir0nM@n`

#### Step 2.2: Switch to Root User
```bash
sudo su -
```

#### Step 2.3: Install Apache (httpd)
```bash
yum install -y httpd
```

#### Step 2.4: Configure Apache to Listen on Port 3003
```bash
sed -i 's/Listen 80/Listen 3003/g' /etc/httpd/conf/httpd.conf
```

**Verify the change:**
```bash
grep "Listen" /etc/httpd/conf/httpd.conf
```

**Expected output:**
```
Listen 3003
```

#### Step 2.5: Create Website Directories
```bash
mkdir -p /var/www/html/ecommerce
mkdir -p /var/www/html/apps
```

#### Step 2.6: Move Website Content
```bash
cp -r /tmp/ecommerce/* /var/www/html/ecommerce/
cp -r /tmp/apps/* /var/www/html/apps/
```

**Alternative: Use mv if you don't need to keep originals**
```bash
mv /tmp/ecommerce/* /var/www/html/ecommerce/
mv /tmp/apps/* /var/www/html/apps/
```

#### Step 2.7: Set Proper Permissions
```bash
chown -R apache:apache /var/www/html/ecommerce
chown -R apache:apache /var/www/html/apps
chmod -R 755 /var/www/html/ecommerce
chmod -R 755 /var/www/html/apps
```

#### Step 2.8: Verify Directory Structure
```bash
ls -la /var/www/html/
```

**Expected output:**
```
drwxr-xr-x. 2 apache apache   apps
drwxr-xr-x. 2 apache apache   ecommerce
```
```bash
ls -la /var/www/html/ecommerce/
ls -la /var/www/html/apps/
```

#### Step 2.9: Start and Enable Apache
```bash
systemctl start httpd
systemctl enable httpd
systemctl status httpd
```

**Expected status:** `active (running)`

#### Step 2.10: Verify Apache is Listening on Port 3003
```bash
ss -tulpn | grep httpd
```

**Expected output:**
```
tcp   LISTEN 0   511   *:3003   *:*   users:(("httpd",pid=...))
```

---

### Part 3: Verification

#### Step 3.1: Test Ecommerce Website
```bash
curl http://localhost:3003/ecommerce/
```

**Expected:** HTML content of the ecommerce website

#### Step 3.2: Test Apps Website
```bash
curl http://localhost:3003/apps/
```

**Expected:** HTML content of the apps website

#### Step 3.3: Check HTTP Status Codes
```bash
curl -I http://localhost:3003/ecommerce/
curl -I http://localhost:3003/apps/
```

**Expected output:**
```
HTTP/1.1 200 OK
Date: ...
Server: Apache/2.4.x
Content-Type: text/html
```

#### Step 3.4: Verify from Jump Host (Optional)

From the **jump host**:
```bash
curl http://stapp01:3003/ecommerce/
curl http://stapp01:3003/apps/
```

---

## Complete Command Summary

### On Jump Host:
```bash
# Copy website backups to app server
scp -r /home/thor/ecommerce /home/thor/apps tony@stapp01:/tmp/
```

### On App Server 1 (stapp01):
```bash
# SSH and switch to root
ssh tony@stapp01
sudo su -

# Install Apache
yum install -y httpd

# Configure port
sed -i 's/Listen 80/Listen 3003/g' /etc/httpd/conf/httpd.conf

# Create directories
mkdir -p /var/www/html/ecommerce
mkdir -p /var/www/html/apps

# Copy website content
cp -r /tmp/ecommerce/* /var/www/html/ecommerce/
cp -r /tmp/apps/* /var/www/html/apps/

# Set permissions
chown -R apache:apache /var/www/html/ecommerce
chown -R apache:apache /var/www/html/apps
chmod -R 755 /var/www/html/ecommerce
chmod -R 755 /var/www/html/apps

# Start Apache
systemctl start httpd
systemctl enable httpd

# Verify
ss -tulpn | grep httpd
curl http://localhost:3003/ecommerce/
curl http://localhost:3003/apps/
```

---

## One-Line Execution Block
```bash
# On Jump Host
scp -r /home/thor/ecommerce /home/thor/apps tony@stapp01:/tmp/

# On App Server 1 (as root)
sudo su -
yum install -y httpd && \
sed -i 's/Listen 80/Listen 3003/g' /etc/httpd/conf/httpd.conf && \
mkdir -p /var/www/html/{ecommerce,apps} && \
cp -r /tmp/ecommerce/* /var/www/html/ecommerce/ && \
cp -r /tmp/apps/* /var/www/html/apps/ && \
chown -R apache:apache /var/www/html/{ecommerce,apps} && \
chmod -R 755 /var/www/html/{ecommerce,apps} && \
systemctl start httpd && \
systemctl enable httpd && \
curl http://localhost:3003/ecommerce/ && \
curl http://localhost:3003/apps/
```

---

## Troubleshooting

### Issue 1: SCP Permission Denied

**Problem:** Cannot copy files to `/home/tony/` or other directories

**Solution:** Copy to `/tmp/` which has universal write permissions:
```bash
scp -r /home/thor/ecommerce tony@stapp01:/tmp/
```

### Issue 2: Apache Fails to Start

**Check configuration syntax:**
```bash
httpd -t
```

**Check logs:**
```bash
journalctl -u httpd -n 50
tail -f /var/log/httpd/error_log
```

**Common causes:**
- Port already in use
- Configuration syntax error
- SELinux blocking

### Issue 3: 404 Not Found

**Check if files exist:**
```bash
ls -la /var/www/html/ecommerce/
ls -la /var/www/html/apps/
```

**Check for index.html:**
```bash
ls -la /var/www/html/ecommerce/index.html
ls -la /var/www/html/apps/index.html
```

**If index.html is missing, check what files are present:**
```bash
find /var/www/html/ecommerce/ -type f
find /var/www/html/apps/ -type f
```

### Issue 4: Permission Denied (403 Forbidden)

**Check file permissions:**
```bash
ls -la /var/www/html/ecommerce/
ls -la /var/www/html/apps/
```

**Set correct permissions:**
```bash
chown -R apache:apache /var/www/html/
chmod -R 755 /var/www/html/
```

**Check SELinux context:**
```bash
ls -Z /var/www/html/ecommerce/
ls -Z /var/www/html/apps/
```

**Restore SELinux context:**
```bash
restorecon -Rv /var/www/html/
```

### Issue 5: Port 3003 Not Listening

**Check Apache configuration:**
```bash
grep "Listen" /etc/httpd/conf/httpd.conf
```

**Should show:**
```
Listen 3003
```

**If not, fix it:**
```bash
sed -i 's/Listen 80/Listen 3003/g' /etc/httpd/conf/httpd.conf
systemctl restart httpd
```

### Issue 6: SELinux Blocking Custom Port

**Check SELinux status:**
```bash
getenforce
```

**If Enforcing, add port to SELinux policy:**
```bash
semanage port -a -t http_port_t -p tcp 3003
```

**Or temporarily disable SELinux (not recommended for production):**
```bash
setenforce 0
```

### Issue 7: Firewall Blocking Port

**Check firewall status:**
```bash
systemctl status firewalld
```

**Add port to firewall:**
```bash
firewall-cmd --add-port=3003/tcp --permanent
firewall-cmd --reload
```

---

## Directory Structure Verification

**Expected structure:**
```
/var/www/html/
├── ecommerce/
│   ├── index.html
│   └── [other website files]
└── apps/
    ├── index.html
    └── [other website files]
```

**Check structure:**
```bash
tree /var/www/html/
```

Or:
```bash
find /var/www/html/ -type f | head -20
```

---

## Testing Different Scenarios

### Test 1: Check if Index Files Exist
```bash
test -f /var/www/html/ecommerce/index.html && echo "Ecommerce index exists" || echo "Ecommerce index missing"
test -f /var/www/html/apps/index.html && echo "Apps index exists" || echo "Apps index missing"
```

### Test 2: Test with Full Path
```bash
curl http://localhost:3003/ecommerce/index.html
curl http://localhost:3003/apps/index.html
```

### Test 3: Check Response Headers
```bash
curl -v http://localhost:3003/ecommerce/
curl -v http://localhost:3003/apps/
```

### Test 4: Test from Different User
```bash
su - tony
curl http://localhost:3003/ecommerce/
curl http://localhost:3003/apps/
```

---

## Apache Configuration Concepts

### DocumentRoot vs Subdirectories

**Default DocumentRoot:**
```
/var/www/html/
```

**URL Mapping:**
- `http://localhost:3003/` → `/var/www/html/`
- `http://localhost:3003/ecommerce/` → `/var/www/html/ecommerce/`
- `http://localhost:3003/apps/` → `/var/www/html/apps/`

### Directory Index Files

Apache looks for these files by default (in order):

1. `index.html`
2. `index.htm`
3. `index.php`

### Custom Port Configuration

**In `/etc/httpd/conf/httpd.conf`:**
```apache
Listen 3003
```

This makes Apache listen on port 3003 instead of the default port 80.

---

## Key Apache Commands Reference

| Command | Description |
|---------|-------------|
| `systemctl start httpd` | Start Apache service |
| `systemctl stop httpd` | Stop Apache service |
| `systemctl restart httpd` | Restart Apache service |
| `systemctl reload httpd` | Reload configuration without restart |
| `systemctl status httpd` | Check Apache status |
| `systemctl enable httpd` | Enable Apache at boot |
| `httpd -t` | Test configuration syntax |
| `httpd -V` | Show Apache version and compile settings |
| `httpd -M` | List loaded modules |
| `apachectl configtest` | Test configuration (alternative) |

---

## Key File Locations

| Path | Description |
|------|-------------|
| `/etc/httpd/conf/httpd.conf` | Main Apache configuration |
| `/etc/httpd/conf.d/` | Additional configuration files |
| `/var/www/html/` | Default document root |
| `/var/log/httpd/access_log` | Access log |
| `/var/log/httpd/error_log` | Error log |
| `/etc/httpd/conf/magic` | MIME type definitions |

---

## Key Takeaways

- Apache can host multiple websites under different subdirectories
- Custom ports are configured using the `Listen` directive
- DocumentRoot maps URL paths to filesystem paths
- Proper permissions are critical: files should be owned by `apache:apache`
- Directory permissions should be `755`, file permissions `644`
- SELinux and firewall may need configuration for custom ports
- Always verify with `curl` after configuration
- Use `httpd -t` to validate configuration before restarting
- The trailing slash in URLs matters: `/ecommerce/` vs `/ecommerce`

---

## Completion Checklist

- [x] Apache (httpd) installed on App Server 1
- [x] Apache configured to listen on port 3003
- [x] Website backups copied from jump host to app server
- [x] `ecommerce` website deployed to `/var/www/html/ecommerce/`
- [x] `apps` website deployed to `/var/www/html/apps/`
- [x] Proper file permissions set (apache:apache, 755)
- [x] Apache service started and enabled
- [x] Port 3003 verified as listening
- [x] `curl http://localhost:3003/ecommerce/` returns website content
- [x] `curl http://localhost:3003/apps/` returns website content
- [x] Both websites accessible and functional

---

## Completion Details

- **Completion Date:** November 24, 2025
- **Day:** 19 / 100
- **Challenge:** KodeKloud 100 Days Cloud DevOps
- **Topic:** Hosting Multiple Static Websites on Apache with Custom Ports
- **Status:** ✅ Successfully Completed

---

