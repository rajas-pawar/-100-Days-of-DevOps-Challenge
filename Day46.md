# Day 46: Docker Compose - Multi-Service Stack (PHP + MariaDB)
**100 Days Cloud DevOps Challenge – KodeKloud**

---

## Objective

Deploy a complete containerized stack using Docker Compose with two services: a PHP Apache web server and a MariaDB database server. Configure port mappings, volume mounts, and database environment variables. Test the deployment before going live.

**Requirements:**
1. Work on Application Server 2 (App Server 2) in Stratos Datacenter
2. Create docker-compose file: `/opt/security/docker-compose.yml` (exact name required)
3. Deploy two services: web and DB
4. **Web Service (php_web):**
   - Container name: `php_web` (exact name required)
   - Image: `php` with any `apache` tag (e.g., `php:apache`)
   - Port mapping: Host `5004` → Container `80`
   - Volume: Host `/var/www/html` → Container `/var/www/html`
5. **DB Service (mysql_web):**
   - Container name: `mysql_web` (exact name required)
   - Image: `mariadb` with any tag (preferably `latest`)
   - Port mapping: Host `3306` → Container `3306`
   - Volume: Host `/var/lib/mysql` → Container `/var/lib/mysql`
   - Environment: `MYSQL_DATABASE=database_web`, custom user (NOT root) with complex password
6. Test with: `curl <server-ip>:5004/` or `curl localhost:5004`
7. **Note:** After finishing, all containers will be destroyed and redeployed using your compose file

---

## Understanding Docker Compose Multi-Service Applications

**Docker Compose** enables you to define and run multi-container applications where services can communicate with each other. It's perfect for web applications that need a database backend.

### What is a Multi-Service Stack?

A **multi-service stack** is an application architecture where different components (web server, database, cache, etc.) run in separate containers but work together as one application.

```
Multi-Service Stack
│
├── Web Service (PHP Apache)
│   ├── Serves web pages
│   ├── Connects to database
│   └── Exposed to users
│
└── DB Service (MariaDB)
    ├── Stores application data
    ├── Only accessible to web service
    └── Persists data in volumes
```

### Why Docker Compose for Multi-Service?

**Traditional Approach (Manual):**
```bash
# Start database
docker run -d --name mysql_apache -e MYSQL_DATABASE=database_apache mariadb:latest

# Start web server (need to link to database)
docker run -d --name php_apache --link mysql_apache:db php:apache

# Complex, error-prone, hard to replicate
```

**Docker Compose Approach:**
```yaml
version: '3'
services:
  web:
    # All web config here
  db:
    # All DB config here
```
```bash
docker-compose up -d
# Simple, reproducible, one command!
```

### Benefits of Docker Compose:

| Benefit | Description |
|---------|-------------|
| **Single File** | All configuration in one YAML file |
| **Service Discovery** | Containers can reach each other by service name |
| **Networking** | Automatic network creation and isolation |
| **Dependency Management** | Control startup order with `depends_on` |
| **Volume Management** | Centralized volume configuration |
| **Environment Variables** | Easy configuration management |
| **One Command** | `docker-compose up` starts entire stack |

---

## Understanding the LAMP Stack

This task deploys a **LAMP-like stack** (Linux, Apache, MySQL/MariaDB, PHP):

```
LAMP Stack in Docker
│
├── Linux (Container Base)
│   └── Debian/Ubuntu in php:apache image
│
├── Apache (Web Server)
│   └── Serves HTTP requests on port 80
│
├── MySQL/MariaDB (Database)
│   └── Stores application data on port 3306
│
└── PHP (Programming Language)
    └── Processes dynamic content
```

### How the Stack Works:

```
User Request
    ↓
http://server:5004
    ↓
Host Port 5004 → Container Port 80
    ↓
Apache Web Server (php_web container)
    ↓
PHP processes request
    ↓
Needs data from database?
    ↓
Connect to mysql_web:3306
    ↓
MariaDB Database (mysql_web container)
    ↓
Returns data
    ↓
PHP generates HTML
    ↓
Apache sends response
    ↓
User sees web page
```

---

## Understanding PHP with Apache

### What is php:apache?

**php:apache** is an official Docker image that combines PHP with Apache web server, perfect for running PHP web applications.

### php:apache Image Tags:

| Tag | Description | Size | Use Case |
|-----|-------------|------|----------|
| **php:apache** | Latest PHP with Apache | ~500MB | **General use (our task)** |
| php:8.3-apache | PHP 8.3 with Apache | ~500MB | Specific PHP version |
| php:8.2-apache | PHP 8.2 with Apache | ~500MB | Stable, production |
| php:7.4-apache | PHP 7.4 with Apache | ~450MB | Legacy applications |

### PHP Apache Directory Structure:

```
php:apache Container
│
├── /var/www/html/          # Web root (our mount point)
│   ├── index.php           # Default PHP file
│   └── index.html          # Default HTML file
│
├── /usr/local/apache2/     # Apache installation
│   ├── conf/               # Apache config
│   └── logs/               # Apache logs
│
└── /usr/local/bin/php      # PHP binary
```

### Testing PHP + Apache:

**Create simple test file:**
```php
<?php
// /var/www/html/index.php
phpinfo();
?>
```

**Access:**
```bash
curl http://localhost:5004
# Shows PHP configuration page
```

---

## Understanding MariaDB

### What is MariaDB?

**MariaDB** is a community-developed, open-source fork of MySQL. It's a drop-in replacement for MySQL with better performance and features.

### MariaDB vs MySQL:

| Feature | MySQL | MariaDB |
|---------|-------|---------|
| **Origin** | Oracle Corporation | Community (MySQL creators) |
| **License** | GPL + Commercial | GPL (fully open) |
| **Performance** | Good | Better optimization |
| **Compatibility** | Standard | MySQL-compatible |
| **Storage Engines** | InnoDB, MyISAM | + Aria, ColumnStore |
| **Our Task** | Either works | ✅ mariadb:latest |

### MariaDB Image Tags:

| Tag | Description | Size | Use Case |
|-----|-------------|------|----------|
| **mariadb:latest** | Latest stable | ~400MB | **Our task** |
| mariadb:11.2 | Specific version | ~400MB | Production pinning |
| mariadb:10.11 | LTS version | ~380MB | Long-term support |
| mariadb:lts | Latest LTS | ~380MB | Stable production |

### MariaDB Environment Variables:

```yaml
environment:
  MYSQL_ROOT_PASSWORD: rootpassword123      # Root user password
  MYSQL_DATABASE: database_apache           # Create database on startup
  MYSQL_USER: apache_user                   # Create custom user
  MYSQL_PASSWORD: SecurePass@123            # User password
```

**Why Custom User (Not Root)?**
- ✅ Security best practice
- ✅ Principle of least privilege
- ✅ Application doesn't need root access
- ✅ Easier to audit and restrict permissions

---

## Understanding Docker Compose File Structure

### Complete docker-compose.yml Anatomy:

```yaml
version: '3'                     # Compose file format version

services:                        # Container definitions
  web:                           # Service name (custom)
    image: php:apache            # Docker image
    container_name: php_apache   # Container name
    ports:                       # Port mappings
      - "5000:80"                # host:container
    volumes:                     # Volume mounts
      - /var/www/html:/var/www/html
    depends_on:                  # Service dependencies
      - db
    networks:                    # Network connections
      - app-network

  db:                            # Database service
    image: mariadb:latest
    container_name: mysql_apache
    ports:
      - "3306:3306"
    volumes:
      - /var/lib/mysql:/var/lib/mysql
    environment:                 # Environment variables
      MYSQL_DATABASE: database_apache
      MYSQL_USER: apache_user
      MYSQL_PASSWORD: SecurePass@123
      MYSQL_ROOT_PASSWORD: RootPass@123
    networks:
      - app-network

networks:                        # Network definitions
  app-network:
    driver: bridge

volumes:                         # Named volume definitions
  mysql-data:
    driver: local
```

### Service Name vs Container Name:

```yaml
services:
  web:                           # Service name (used in docker-compose commands)
    container_name: php_apache   # Container name (used in docker commands)
```

**Usage:**
```bash
# Docker Compose uses service name
docker-compose logs web
docker-compose restart web

# Docker uses container name
docker logs php_apache
docker restart php_apache
```

---

## Understanding Volumes in Docker Compose

### Bind Mounts vs Named Volumes:

**1. Bind Mounts (Our Task):**
```yaml
volumes:
  - /var/www/html:/var/www/html        # Host path : Container path
  - /var/lib/mysql:/var/lib/mysql
```

**Characteristics:**
- Direct mount of host directory
- Full path specified
- Changes on host reflect immediately in container
- Good for development and specific paths

**2. Named Volumes:**
```yaml
volumes:
  - web-data:/var/www/html
  - db-data:/var/lib/mysql

volumes:
  web-data:
  db-data:
```

**Characteristics:**
- Docker manages storage location
- Better for production
- Easier backup and migration
- Survives container removal

### Volume Benefits:

```
Why Use Volumes?

1. Data Persistence
   Container deleted → Data survives ✅

2. Easy Updates
   Edit files on host → Instantly in container ✅

3. Backup Simplicity
   Backup host directory = Backup container data ✅

4. Development Workflow
   Code on host → Runs in container ✅
```

---

## Understanding Port Mapping in Multi-Service

### Port Mapping Strategy:

```yaml
services:
  web:
    ports:
      - "5000:80"          # External access: host 5000 → container 80
  
  db:
    ports:
      - "3306:3306"        # External access: host 3306 → container 3306
```

### External vs Internal Access:

**External (Published Ports):**
```
User → Host Port 5000 → php_apache:80 (web accessible)
Admin → Host Port 3306 → mysql_apache:3306 (DB admin tools)
```

**Internal (Service-to-Service):**
```
php_apache → mysql_apache:3306 (no host port needed!)
```

### Why Port 5000 for Web?

- Port 80 might be used by host system
- Port 5000 is commonly free
- Avoids conflicts with other services
- Easy to remember for testing

---

## Infrastructure Overview

### Application Servers:
| Server | User | Password | IP |
|--------|------|----------|-----|
| stapp01 | tony | Ir0nM@n | 172.16.238.10 |
| **stapp02** | **steve** | **Am3ric@** | **172.16.238.11** |
| stapp03 | banner | BigGr33n | 172.16.238.12 |

### Task Details:
| Item | Value |
|------|-------|
| Server | Application Server 2 (stapp02) |
| User | steve |
| Compose File | `/opt/security/docker-compose.yml` (**exact name**) |
| Web Service | php_web (php:apache, port 5004:80) |
| DB Service | mysql_web (mariadb:latest, port 3306:3306) |
| Database Name | `database_web` |
| Test URL | `curl stapp02:5004` or `curl localhost:5004` |

---

## Step-by-Step Implementation

### Step 1: SSH into Application Server 2
```bash
ssh steve@stapp02
```

**Expected output:**
```
The authenticity of host 'stapp02 (172.16.238.11)' can't be established.
ED25519 key fingerprint is SHA256:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'stapp02' (ED25519) to the list of known hosts.
steve@stapp02's password:
```

**Enter password:** `Am3ric@`

```
[steve@stapp02 ~]$
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

[sudo] password for steve:
```

**Enter password:** `Am3ric@`

```
[root@stapp02 ~]#
```

### Step 3: Create /opt/security Directory
```bash
mkdir -p /opt/security
```

**Verify directory created:**
```bash
ls -la /opt/
```

**Expected output:**
```
total 16
drwxr-xr-x  4 root root 4096 Dec 22 08:00 .
dr-xr-xr-x 18 root root 4096 Dec 22 08:00 ..
drwxr-xr-x  2 root root 4096 Dec 22 08:00 security
```

### Step 4: Navigate to /opt/security
```bash
cd /opt/security
```

**Verify location:**
```bash
pwd
```

**Expected output:**
```
/opt/security
```

### Step 5: Create Host Volume Directories
```bash
# Create web root directory
mkdir -p /var/www/html

# Create database directory
mkdir -p /var/lib/mysql
```

**Verify directories:**
```bash
ls -la /var/www/html
ls -la /var/lib/mysql
```

**Expected output:**
```
drwxr-xr-x 2 root root 4096 Dec 22 08:05 /var/www/html
drwxr-xr-x 2 root root 4096 Dec 22 08:05 /var/lib/mysql
```

### Step 6: Create Test PHP File
```bash
cat > /var/www/html/index.php << 'EOF'
<?php
echo "<h1>Welcome to Nautilus LAMP Stack!</h1>";
echo "<p>PHP Version: " . phpversion() . "</p>";
echo "<p>Server: " . $_SERVER['SERVER_SOFTWARE'] . "</p>";

// Test database connection
$servername = "mysql_web";
$username = "app_user";
$password = "C0mpl3xP@ss!";
$dbname = "database_web";

try {
    $conn = new PDO("mysql:host=$servername;dbname=$dbname", $username, $password);
    $conn->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
    echo "<p style='color: green;'>✅ Database connection successful!</p>";
} catch(PDOException $e) {
    echo "<p style='color: red;'>❌ Database connection failed: " . $e->getMessage() . "</p>";
}
?>
EOF
```

**Note:** Update the credentials in index.php to match your docker-compose.yml environment variables.

**Verify file created:**
```bash
cat /var/www/html/index.php
```

### Step 7: Create docker-compose.yml
```bash
cat > /opt/security/docker-compose.yml << 'EOF'
version: '3.8'

services:
  web:
    image: php:apache
    container_name: php_web
    ports:
      - "5004:80"
    volumes:
      - /var/www/html:/var/www/html
    depends_on:
      - db

  db:
    image: mariadb:latest
    container_name: mysql_web
    ports:
      - "3306:3306"
    volumes:
      - /var/lib/mysql:/var/lib/mysql
    environment:
      MYSQL_DATABASE: database_web
      MYSQL_USER: app_user
      MYSQL_PASSWORD: C0mpl3xP@ss!
      MYSQL_ROOT_PASSWORD: R00tPass!
EOF
```

**Important Notes:**
- **Version 3.8:** While Docker Compose now considers the `version` field obsolete, it's still supported and helps with clarity
- **Custom User:** `app_user` (NOT root) - required by task
- **Complex Passwords:** Must include uppercase, lowercase, numbers, and special characters
- **MYSQL_PASSWORD:** Required for custom user to authenticate - **do not omit this!**

**Expected output:**
```
(No output means success - Ctrl+D to save if using cat, or Ctrl+C then create via editor)
```

**Note:** The warning about `version` being obsolete is informational only and won't affect functionality.

### Step 8: Verify docker-compose.yml Created
```bash
ls -la /opt/security/
```

**Expected output:**
```
total 12
drwxr-xr-x 2 root root 4096 Dec 22 08:10 .
drwxr-xr-x 4 root root 4096 Dec 22 08:00 ..
-rw-r--r-- 1 root root  456 Dec 22 08:10 docker-compose.yml
```

**✅ docker-compose.yml exists at /opt/security/**

### Step 9: View docker-compose.yml Contents
```bash
cat /opt/security/docker-compose.yml
```

**Expected output:**
```yaml
version: '3.8'

services:
  web:
    image: php:apache
    container_name: php_web
    ports:
      - "5004:80"
    volumes:
      - /var/www/html:/var/www/html
    depends_on:
      - db

  db:
    image: mariadb:latest
    container_name: mysql_web
    ports:
      - "3306:3306"
    volumes:
      - /var/lib/mysql:/var/lib/mysql
    environment:
      MYSQL_DATABASE: database_web
      MYSQL_USER: app_user
      MYSQL_PASSWORD: C0mpl3xP@ss!
      MYSQL_ROOT_PASSWORD: R00tPass!
```

**Verify:**
- ✅ version: '3.8' (or '3' - both work, version field is now optional)
- ✅ Two services: web and db
- ✅ web: php:apache, container_name: php_web (exact name)
- ✅ web ports: "5004:80"
- ✅ web volume: /var/www/html:/var/www/html
- ✅ db: mariadb:latest, container_name: mysql_web (exact name)
- ✅ db ports: "3306:3306"
- ✅ db volume: /var/lib/mysql:/var/lib/mysql
- ✅ MYSQL_DATABASE: database_web
- ✅ Custom user: app_user (NOT root)
- ✅ **MYSQL_PASSWORD:** Complex password for custom user (**critical - do not omit!**)
- ✅ MYSQL_ROOT_PASSWORD: Root password

### Step 10: Validate docker-compose.yml Syntax
```bash
docker compose -f /opt/security/docker-compose.yml config
```

**Expected output:**
```yaml
services:
  db:
    container_name: mysql_web
    environment:
      MYSQL_DATABASE: database_web
      MYSQL_PASSWORD: C0mpl3xP@ss!
      MYSQL_ROOT_PASSWORD: R00tPass!
      MYSQL_USER: app_user
    image: mariadb:latest
    ports:
    - published: 3306
      target: 3306
    volumes:
    - /var/lib/mysql:/var/lib/mysql:rw
  web:
    container_name: php_web
    depends_on:
      db:
        condition: service_started
        required: true
    image: php:apache
    ports:
    - published: 5004
      target: 80
    volumes:
    - /var/www/html:/var/www/html:rw
```

**✅ No syntax errors!**

**Note:** You may see a warning: `the attribute 'version' is obsolete` - this is informational only and won't affect deployment.

### Step 11: Check Current Docker Containers
```bash
docker ps -a
```

**Expected output:**
```
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

**Or may show existing containers to be stopped**

### Step 12: Pull Required Images (Optional)
```bash
docker pull php:apache
docker pull mariadb:latest
```

**Expected output:**
```
php:apache: Pulling from library/php
...
Status: Downloaded newer image for php:apache
docker.io/library/php:apache

mariadb:latest: Pulling from library/mariadb
...
Status: Downloaded newer image for mariadb:latest
docker.io/library/mariadb:latest
```

**Note:** `docker-compose up` will pull automatically if not present

### Step 13: Start the Stack with Docker Compose
```bash
cd /opt/security
docker compose up -d
```

**Expected output:**
```
WARN[0000] /opt/security/docker-compose.yml: the attribute `version` is obsolete, it will be ignored, please remove it to avoid potential confusion
[+] Running 24/24
 ✔ db Pulled                                                                30.2s 
   ✔ 20043066d3d5 Pull complete                                             6.4s 
   ✔ 75e5c9f5eeb0 Pull complete                                             7.3s 
   ✔ ee5b64c3e2f5 Pull complete                                             9.0s 
   ✔ ba2d27f4ceca Pull complete                                            10.4s 
   ✔ 4be15142d46e Pull complete                                            11.9s 
   ✔ 9047f410cb7d Pull complete                                            24.7s 
   ✔ be922ac1f9ed Pull complete                                            27.9s 
   ✔ eca7ec75082e Pull complete                                            29.8s 
 ✔ web Pulled                                                               77.1s 
   ✔ 1733a4cd5954 Pull complete                                             6.6s 
   ✔ fbedb3e705cb Pull complete                                             7.3s 
   ✔ debbbfb6890d Pull complete                                            26.8s 
   ✔ f844a1f5e709 Pull complete                                            31.9s 
   ✔ 91c07df83de9 Pull complete                                            36.4s 
   ✔ 84533337b874 Pull complete                                            40.2s 
   ✔ f5cac46fa7e4 Pull complete                                            44.3s 
   ✔ 3036c986058a Pull complete                                            48.3s 
   ✔ 526f66ff2901 Pull complete                                            52.4s 
   ✔ ceb020c3bd99 Pull complete                                            57.4s 
   ✔ 6ca6a8fbedf4 Pull complete                                            61.6s 
   ✔ 6c94fd1a3080 Pull complete                                            65.8s 
   ✔ d42f89d8d9b4 Pull complete                                            71.3s 
   ✔ 4f4fb700ef54 Pull complete                                            76.8s 
[+] Running 3/3
 ✔ Network security_default  Created                                        0.1s 
 ✔ Container mysql_web       Started                                       15.5s 
 ✔ Container php_web         Started                                       10.0s
```

**Command breakdown:**
- `docker compose up` (or `docker-compose up`) - Start services defined in docker-compose.yml
- `-d` - Detached mode (background)
- **WARN about version:** This is informational only - Docker Compose v2 doesn't require version field but still accepts it

**✅ Stack deployed!**

### Step 14: Verify Containers Running
```bash
docker compose ps
```

**Expected output:**
```
    Name                  Command               State                    Ports                  
------------------------------------------------------------------------------------------------
mysql_web    docker-entrypoint.sh mariadbd   Up      0.0.0.0:3306->3306/tcp,:::3306->3306/tcp
php_web      docker-php-entrypoint apac ...   Up      0.0.0.0:5004->80/tcp,:::5004->80/tcp    
```

**Alternative:**
```bash
docker ps
```

**Expected output:**
```
CONTAINER ID   IMAGE            COMMAND                  CREATED          STATUS          PORTS                    NAMES
551fc20b2bc7   php:apache       "docker-php-entrypoi…"   33 seconds ago   Up 24 seconds   0.0.0.0:5004->80/tcp     php_web
c00c2335fc43   mariadb:latest   "docker-entrypoint.s…"   40 seconds ago   Up 24 seconds   0.0.0.0:3306->3306/tcp   mysql_web
```

**Verify:**
- ✅ mysql_web running (exact container name)
- ✅ php_web running (exact container name)
- ✅ Port 5004:80 mapped (web service)
- ✅ Port 3306:3306 mapped (database service)
- ✅ STATUS: Up

### Step 15: Check Container Logs
```bash
# Check database logs
docker compose logs db
```

**Expected output:**
```
mysql_web | 2025-12-22 08:15:00+00:00 [Note] [Entrypoint]: Entrypoint script for MariaDB Server started.
mysql_web | 2025-12-22 08:15:01+00:00 [Note] [Entrypoint]: Initializing database files
mysql_web | 2025-12-22 08:15:05+00:00 [Note] [Entrypoint]: Database files initialized
mysql_web | 2025-12-22 08:15:05+00:00 [Note] [Entrypoint]: Starting temporary server
mysql_web | 2025-12-22 08:15:06+00:00 [Note] [Entrypoint]: Creating database database_web
mysql_web | 2025-12-22 08:15:06+00:00 [Note] [Entrypoint]: Creating user webapp_user
mysql_web | 2025-12-22 08:15:07+00:00 [Note] [Entrypoint]: MariaDB init process done. Ready for start up.
mysql_web | 2025-12-22 08:15:08 0 [Note] mariadbd: ready for connections.
```

**✅ Database initialized and ready!**

```bash
# Check web server logs
docker compose logs web
```

**Expected output:**
```
php_web | AH00558: apache2: Could not reliably determine the server's fully qualified domain name
php_web | [Sun Dec 22 08:15:10.123456 2025] [mpm_prefork:notice] [pid 1] AH00163: Apache/2.4.58 (Debian) PHP/8.3.0 configured -- resuming normal operations
php_web | [Sun Dec 22 08:15:10.123457 2025] [core:notice] [pid 1] AH00094: Command line: 'apache2 -D FOREGROUND'
```

**✅ Apache with PHP running!**

### Step 16: Verify Network Created
```bash
docker network ls | grep security
```

**Expected output:**
```
a1b2c3d4e5f6   security_default   bridge    local
```

**Docker Compose automatically creates a network named after the directory (security_default)**

### Step 17: Test Web Service Accessibility
```bash
curl http://localhost:5004
```

**Expected output (without DB driver):**
```html
<h1>Welcome to Nautilus LAMP Stack!</h1>
<p>PHP Version: 8.5.1</p>
<p>Server: Apache/2.4.65 (Debian)</p>
<p style='color: red;'>❌ Database connection failed: could not find driver</p>
```

**✅ Web service working, but needs MySQL driver!**

### Step 17a: Install PDO MySQL Extension (Required)

**Problem:** The `php:apache` image doesn't include PDO MySQL extension by default.

**Solution:**
```bash
# Install PDO MySQL extension in running container
docker exec php_web docker-php-ext-install pdo pdo_mysql

# Restart web container to apply changes
docker compose restart web
```

**Expected output:**
```
Configuring for:
PHP Api Version:         20240924
Zend Module Api No:      20240924
Zend Extension Api No:   420240924
...
Installing shared extensions:     /usr/local/lib/php/extensions/no-debug-non-zts-20240924/
Installing header files:          /usr/local/include/php/
...
[+] Restarting 1/1
 ✔ Container php_web  Started
```

**Test again:**
```bash
curl http://localhost:5004
```

**Expected output (with DB driver):**
```html
<h1>Welcome to Nautilus LAMP Stack!</h1>
<p>PHP Version: 8.5.1</p>
<p>Server: Apache/2.4.65 (Debian)</p>
<p style='color: green;'>✅ Database connection successful!</p>
```

**✅ Web service working and connected to database!**

**Test with full output:**
```bash
curl -v http://localhost:5004
```

**Expected output:**
```
* Connected to localhost (127.0.0.1) port 5004
> GET / HTTP/1.1
> Host: localhost:5004
> User-Agent: curl/7.68.0
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< Date: Sun, 22 Dec 2025 08:20:00 GMT
< Server: Apache/2.4.58 (Debian) PHP/8.3.0
< Content-Length: 245
< Content-Type: text/html; charset=UTF-8
< 
<h1>Welcome to Nautilus LAMP Stack!</h1>
<p>PHP Version: 8.3.0</p>
<p>Server: Apache/2.4.58 (Debian) PHP/8.3.0</p>
<p style='color: green;'>✅ Database connection successful!</p>
```

### Step 18: Test Database Connection
```bash
# Test MySQL connection from host with custom user
docker exec mysql_web mysql -u app_user -pC0mpl3xP@ss! -e "SHOW DATABASES;"
```

**Expected output:**
```
+--------------------+
| Database           |
+--------------------+
| database_web       |
| information_schema |
+--------------------+
```

**✅ Database accessible with custom user (NOT root)!**

**Verify database created:**
```bash
docker exec mysql_web mysql -u app_user -pC0mpl3xP@ss! database_web -e "SELECT 'Database connection successful!' AS Status;"
```

**Expected output:**
```
+--------------------------------+
| Status                         |
+--------------------------------+
| Database connection successful! |
+--------------------------------+
```

### Step 19: Check Volume Mounts
```bash
# Verify web volume mount
docker exec php_web ls -la /var/www/html/
```

**Expected output:**
```
total 12
drwxr-xr-x 2 root root 4096 Dec 22 08:05 .
drwxr-xr-x 3 root root 4096 Dec 22 08:15 ..
-rw-r--r-- 1 root root  567 Dec 22 08:07 index.php
```

**✅ Host files visible in container!**

```bash
# Verify database volume mount
docker exec mysql_web ls -la /var/lib/mysql/
```

**Expected output:**
```
total 123456
drwxr-xr-x  6 mysql mysql   4096 Dec 22 08:15 .
drwxr-xr-x  1 root  root    4096 Dec 22 08:15 ..
-rw-rw----  1 mysql mysql  12582912 Dec 22 08:15 ibdata1
drwx------  2 mysql mysql   4096 Dec 22 08:15 database_web
...
```

**✅ Database files persisted!**

### Step 20: Test Service-to-Service Communication
```bash
# Web container can reach DB container by service name
docker exec php_web ping -c 2 mysql_web
```

**Expected output:**
```
PING mysql_web (172.18.0.2) 56(84) bytes of data.
64 bytes from security_db_1.security_default (172.18.0.2): icmp_seq=1 ttl=64 time=0.123 ms
64 bytes from security_db_1.security_default (172.18.0.2): icmp_seq=2 ttl=64 time=0.089 ms

--- mysql_web ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1001ms
rtt min/avg/max/mdev = 0.089/0.106/0.123/0.017 ms
```

**✅ Service discovery working!**

### Step 21: Inspect Docker Compose Configuration
```bash
docker-compose config --services
```

**Expected output:**
```
db
web
```

```bash
docker-compose config --volumes
```

**Expected output:**
```
(Empty - we're using bind mounts, not named volumes)
```

### Step 22: View Container Resource Usage
```bash
docker stats --no-stream php_web mysql_web
```

**Expected output:**
```
CONTAINER ID   NAME           CPU %     MEM USAGE / LIMIT     MEM %     NET I/O          BLOCK I/O      PIDS
a1b2c3d4e5f6   php_web        0.25%     45.6MiB / 7.77GiB    0.57%     1.23kB / 2.34kB  12.3MB / 0B    12
b2c3d4e5f6a1   mysql_web      1.52%     387.2MiB / 7.77GiB   4.87%     2.34kB / 3.45kB  234MB / 123MB  45
```

### Step 23: Final Verification Checklist
```bash
# 1. Compose file exists at exact location
ls -la /opt/security/docker-compose.yml

# 2. Both containers running with exact names
docker ps | grep -E "php_web|mysql_web"

# 3. Correct ports mapped
docker port php_web 80
docker port mysql_web 3306

# 4. Web service accessible
curl -s http://localhost:5004 | grep -i "welcome"

# 5. Database accessible with custom user (NOT root)
docker exec mysql_web mysql -u app_user -pC0mpl3xP@ss! -e "SHOW DATABASES;" | grep database_web

# 6. Volumes mounted
docker inspect php_web | grep -A 5 "Mounts"
docker inspect mysql_web | grep -A 5 "Mounts"
```

**All checks should pass:**
- ✅ docker-compose.yml at /opt/security/docker-compose.yml (exact path)
- ✅ php_web container running (exact name)
- ✅ mysql_web container running (exact name)
- ✅ Port 5004:80 mapped (web)
- ✅ Port 3306:3306 mapped (database)
- ✅ Web accessible on port 5004
- ✅ Database "database_web" exists
- ✅ Custom user app_user (NOT root) configured with complex password
- ✅ Volumes correctly mounted

**⚠️ Critical Reminder:**
Ensure your docker-compose.yml includes **all four** environment variables:
1. `MYSQL_DATABASE` - creates the database
2. `MYSQL_USER` - creates custom user (NOT root)
3. **`MYSQL_PASSWORD`** - password for custom user (**required!**)
4. `MYSQL_ROOT_PASSWORD` - root password

---

## Complete Command Summary

### Quick Deployment:
```bash
# SSH and access
ssh steve@stapp02
sudo su -

# Create directories
mkdir -p /opt/security
mkdir -p /var/www/html
mkdir -p /var/lib/mysql

# Create test PHP file
cat > /var/www/html/index.php << 'EOF'
<?php
echo "<h1>Welcome to Nautilus LAMP Stack!</h1>";
echo "<p>PHP Version: " . phpversion() . "</p>";
phpinfo();
?>
EOF

# Create docker-compose.yml at exact location
cd /opt/security
cat > docker-compose.yml << 'EOF'
version: '3'

services:
  web:
    image: php:apache
    container_name: php_web
    ports:
      - "5004:80"
    volumes:
      - /var/www/html:/var/www/html
    depends_on:
      - db

  db:
    image: mariadb:latest
    container_name: mysql_web
    ports:
      - "3306:3306"
    volumes:
      - /var/lib/mysql:/var/lib/mysql
    environment:
      MYSQL_DATABASE: database_web
      MYSQL_USER: app_user
      MYSQL_PASSWORD: C0mpl3xP@ss!
      MYSQL_ROOT_PASSWORD: R00tPass!
EOF

# Deploy stack
docker compose up -d

# Fix: Install PDO MySQL extension (if needed)
docker exec php_web docker-php-ext-install pdo pdo_mysql
docker compose restart web

# Test
curl http://localhost:5004
```

### Detailed Workflow:
```bash
# 1. SSH and authenticate
ssh steve@stapp02
sudo su -

# 2. Create required directories
mkdir -p /opt/security
mkdir -p /var/www/html
mkdir -p /var/lib/mysql

# 3. Create test content
cat > /var/www/html/index.php << 'EOF'
<?php
echo "<h1>Nautilus LAMP Stack</h1>";
echo "<p>PHP: " . phpversion() . "</p>";

$conn = new PDO("mysql:host=mysql_web;dbname=database_web", "app_user", "C0mpl3xP@ss!");
echo "<p>✅ Database connected!</p>";
?>
EOF

# 4. Navigate to compose directory
cd /opt/security

# 5. Create docker-compose.yml
vi docker-compose.yml
# (Paste the compose configuration with exact container names)

# 6. Validate syntax
docker-compose config

# 7. Pull images (optional)
docker-compose pull

# 8. Start services
docker-compose up -d

# 9. Check status
docker-compose ps
docker ps

# 10. View logs
docker-compose logs -f

# 11. Test web service
curl http://localhost:5004

# 12. Test database with custom user (NOT root)
docker exec mysql_web mysql -u app_user -pC0mpl3xP@ss! -e "SHOW DATABASES;"

# 13. Verify volumes
docker inspect php_web | grep -A 10 "Mounts"
docker inspect mysql_web | grep -A 10 "Mounts"
```

---

## Docker Compose Commands Reference

### Basic Operations:

```bash
# Start services (creates and starts containers)
docker-compose up
docker-compose up -d                    # Detached mode (background)
docker-compose up --build               # Rebuild images before starting

# Stop services (stops containers but doesn't remove them)
docker-compose stop

# Start stopped services
docker-compose start

# Restart services
docker-compose restart

# Stop and remove containers, networks
docker-compose down
docker-compose down -v                  # Also remove volumes
docker-compose down --rmi all           # Also remove images

# Pause services
docker-compose pause

# Unpause services
docker-compose unpause
```

### Monitoring and Logs:

```bash
# View service status
docker-compose ps
docker-compose ps -a                    # Include stopped containers

# View logs
docker-compose logs
docker-compose logs -f                  # Follow logs (live)
docker-compose logs web                 # Logs for specific service
docker-compose logs --tail=100 db       # Last 100 lines

# Execute commands in service
docker-compose exec web bash            # Interactive shell in web service
docker-compose exec db mysql -u root -p # MySQL client in DB service

# Run one-off commands
docker-compose run web php -v           # Run PHP version command
docker-compose run db mysql --version   # Run MySQL version command
```

### Configuration and Validation:

```bash
# Validate and view configuration
docker-compose config
docker-compose config --services        # List services
docker-compose config --volumes         # List volumes

# Pull images
docker-compose pull

# Build images (if using build instead of image)
docker-compose build
docker-compose build --no-cache         # Build without cache

# View images
docker-compose images
```

### Scaling and Management:

```bash
# Scale services (create multiple containers)
docker-compose up -d --scale web=3      # Run 3 web containers

# Remove stopped containers
docker-compose rm
docker-compose rm -f                    # Force removal

# View processes
docker-compose top
docker-compose top web                  # Processes in web service
```

---

## Understanding Service Dependencies

### depends_on Explained:

```yaml
services:
  web:
    depends_on:
      - db        # Start db before web
```

**What it does:**
- Controls startup order
- DB starts before web
- Web waits for DB container to be created

**What it doesn't do:**
- Doesn't wait for DB to be ready
- Doesn't check if MySQL is accepting connections

**Better approach with health checks:**
```yaml
services:
  web:
    depends_on:
      db:
        condition: service_healthy
  
  db:
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 10s
      timeout: 5s
      retries: 5
```

---

## Environment Variables Best Practices

### Current Approach (Inline):

```yaml
environment:
  MYSQL_DATABASE: database_apache
  MYSQL_USER: apache_user
  MYSQL_PASSWORD: SecurePass@123
  MYSQL_ROOT_PASSWORD: RootSecure@456
```

**Pros:** Simple, direct
**Cons:** Passwords visible in file

### Better Approach (.env file):

**Create .env file:**
```bash
cat > /opt/data/.env << 'EOF'
MYSQL_DATABASE=database_apache
MYSQL_USER=apache_user
MYSQL_PASSWORD=SecurePass@123
MYSQL_ROOT_PASSWORD=RootSecure@456
EOF
```

**Update docker-compose.yml:**
```yaml
services:
  db:
    env_file:
      - .env
```

**Benefits:**
- Separate config from code
- Easy to change without editing YAML
- Can use different .env for dev/prod
- Add .env to .gitignore

---

## Troubleshooting Guide

### Issue 1: Containers Not Starting

**Problem:**
```bash
docker-compose up -d
# ERROR: Service 'web' failed to build
```

**Diagnosis:**
```bash
docker-compose logs
docker-compose ps -a
```

**Solution:**
```bash
# Check for port conflicts
netstat -tuln | grep -E "5004|3306"

# Stop conflicting services
sudo systemctl stop mysql    # If MySQL running on host

# Remove old containers
docker-compose down
docker-compose up -d
```

### Issue 2: "Address Already in Use"

**Problem:**
```
Error starting userland proxy: listen tcp4 0.0.0.0:3306: bind: address already in use
```

**Solution:**
```bash
# Check what's using the port
sudo lsof -i :3306
sudo netstat -tuln | grep 3306

# Option 1: Stop the service
sudo systemctl stop mysql
sudo systemctl stop mariadb

# Option 2: Change host port in docker-compose.yml
ports:
  - "3307:3306"    # Use different host port
```

### Issue 3: Database Connection Failed

**Problem:**
Web shows: "Database connection failed"

**Diagnosis:**
```bash
# Check if DB is ready
docker-compose logs db | grep "ready for connections"

# Test connection from web container
docker exec php_apache ping mysql_apache

# Verify credentials
docker exec mysql_apache mysql -u apache_user -pSecurePass@123 -e "SELECT 1;"
```

**Solution:**
```bash
# Wait for DB to initialize (takes 30-60 seconds)
docker-compose logs -f db

# Restart web service after DB is ready
docker-compose restart web

# Check environment variables
docker exec mysql_apache printenv | grep MYSQL
```

### Issue 4: Volume Permission Denied

**Problem:**
```
Permission denied: /var/www/html/index.php
```

**Solution:**
```bash
# Fix permissions on host
sudo chown -R www-data:www-data /var/www/html
sudo chmod -R 755 /var/www/html

# Or make world-readable
sudo chmod -R 777 /var/www/html

# Restart container
docker-compose restart web
```

### Issue 5: Can't Access Website

**Problem:**
```bash
curl http://localhost:5004
# curl: (7) Failed to connect to localhost port 5004
```

**Diagnosis:**
```bash
# Check if container is running
docker ps | grep php_web

# Check port mapping
docker port php_web

# Check from inside container
docker exec php_web curl http://localhost:80
```

**Solution:**
```bash
# Check firewall
sudo firewall-cmd --list-ports
sudo firewall-cmd --add-port=5004/tcp --permanent
sudo firewall-cmd --reload

# Check if Apache is running
docker exec php_web apache2ctl status

# Restart services
docker-compose restart web
```

### Issue 6: "could not find driver" - PHP Missing MySQL Extension

**Problem:**
```bash
curl http://localhost:5004
# Shows: Database connection failed: could not find driver
```

**Root Cause:**
The official `php:apache` image doesn't include PDO MySQL extension by default.

**Solution 1: Install Extension in Running Container (Quick Fix)**
```bash
# Install PDO MySQL extension in php_web container
docker exec php_web docker-php-ext-install pdo pdo_mysql

# Restart web container
docker compose restart web

# Test again
curl http://localhost:5004
```

**Solution 2: Create Custom Dockerfile (Production Approach)**

**Create Dockerfile:**
```bash
cat > /opt/security/Dockerfile << 'EOF'
FROM php:apache

# Install PDO MySQL extension
RUN docker-php-ext-install pdo pdo_mysql

# Enable Apache modules if needed
RUN a2enmod rewrite
EOF
```

**Update docker-compose.yml:**
```yaml
version: '3'

services:
  web:
    build: .                    # Build from Dockerfile
    container_name: php_web
    ports:
      - "5004:80"
    volumes:
      - /var/www/html:/var/www/html
    depends_on:
      - db

  db:
    image: mariadb:latest
    container_name: mysql_web
    ports:
      - "3306:3306"
    volumes:
      - /var/lib/mysql:/var/lib/mysql
    environment:
      MYSQL_DATABASE: database_web
      MYSQL_USER: webapp_user
      MYSQL_PASSWORD: W3b@ppP@ss2024!
      MYSQL_ROOT_PASSWORD: R00tS3cur3P@ss!
```

**Rebuild and restart:**
```bash
cd /opt/security
docker compose down
docker compose build
docker compose up -d
```

**Solution 3: Use Pre-configured Image**
```yaml
services:
  web:
    image: php:apache
    container_name: php_web
    command: >
      bash -c "docker-php-ext-install pdo pdo_mysql && apache2-foreground"
    # ... rest of config
```

### Issue 7: Syntax Error in docker-compose.yml

**Problem:**
```
ERROR: yaml.parser.ParserError: while parsing a block mapping
```

**Solution:**
```bash
# Validate YAML syntax
docker-compose config

# Common issues:
# 1. Tabs instead of spaces (use spaces only)
# 2. Incorrect indentation
# 3. Missing colons
# 4. Incorrect quotes

# Fix indentation (2 spaces per level)
version: '3'

services:
  web:              # 2 spaces
    image: ...      # 4 spaces
    ports:          # 4 spaces
      - "5000:80"   # 6 spaces
```

---

## Advanced Configurations

### Add Named Volumes (Instead of Bind Mounts):

```yaml
version: '3'

services:
  web:
    image: php:apache
    container_name: php_apache
    ports:
      - "5000:80"
    volumes:
      - web-data:/var/www/html

  db:
    image: mariadb:latest
    container_name: mysql_apache
    ports:
      - "3306:3306"
    volumes:
      - db-data:/var/lib/mysql
    environment:
      MYSQL_DATABASE: database_apache
      MYSQL_USER: apache_user
      MYSQL_PASSWORD: SecurePass@123
      MYSQL_ROOT_PASSWORD: RootSecure@456

volumes:
  web-data:
  db-data:
```

### Add Custom Network:

```yaml
version: '3'

services:
  web:
    networks:
      - frontend
      - backend
  
  db:
    networks:
      - backend

networks:
  frontend:
  backend:
    internal: true    # No external access
```

### Add Health Checks:

```yaml
services:
  db:
    image: mariadb:latest
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 30s
  
  web:
    image: php:apache
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:80"]
      interval: 30s
      timeout: 3s
      retries: 3
    depends_on:
      db:
        condition: service_healthy
```

### Add Resource Limits:

```yaml
services:
  web:
    image: php:apache
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 512M
        reservations:
          cpus: '0.25'
          memory: 256M
  
  db:
    image: mariadb:latest
    deploy:
      resources:
        limits:
          cpus: '1.0'
          memory: 1G
        reservations:
          cpus: '0.5'
          memory: 512M
```

---

## Real-World Scenarios

### Scenario 1: WordPress Stack

```yaml
version: '3'

services:
  wordpress:
    image: wordpress:latest
    container_name: wordpress
    ports:
      - "8080:80"
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: wordpress_pass
      WORDPRESS_DB_NAME: wordpress_db
    volumes:
      - ./wordpress:/var/www/html
    depends_on:
      - db

  db:
    image: mariadb:latest
    container_name: wordpress_db
    environment:
      MYSQL_DATABASE: wordpress_db
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: wordpress_pass
      MYSQL_ROOT_PASSWORD: root_pass
    volumes:
      - db_data:/var/lib/mysql

volumes:
  db_data:
```

### Scenario 2: Node.js + MongoDB

```yaml
version: '3'

services:
  app:
    image: node:18-alpine
    container_name: node_app
    working_dir: /app
    ports:
      - "3000:3000"
    volumes:
      - ./app:/app
    command: npm start
    depends_on:
      - mongo

  mongo:
    image: mongo:latest
    container_name: mongodb
    ports:
      - "27017:27017"
    environment:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: admin_pass
    volumes:
      - mongo_data:/data/db

volumes:
  mongo_data:
```

### Scenario 3: Full LEMP Stack (Linux, Nginx, MySQL, PHP)

```yaml
version: '3'

services:
  nginx:
    image: nginx:alpine
    container_name: nginx
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
      - ./html:/var/www/html
    depends_on:
      - php

  php:
    image: php:fpm-alpine
    container_name: php-fpm
    volumes:
      - ./html:/var/www/html

  mysql:
    image: mysql:8
    container_name: mysql
    environment:
      MYSQL_ROOT_PASSWORD: root_pass
      MYSQL_DATABASE: app_db
      MYSQL_USER: app_user
      MYSQL_PASSWORD: app_pass
    volumes:
      - mysql_data:/var/lib/mysql

volumes:
  mysql_data:
```

---

## Key Commands Reference

| Command | Description |
|---------|-------------|
| `docker-compose up -d` | Start all services in background |
| `docker-compose down` | Stop and remove containers |
| `docker-compose ps` | List service containers |
| `docker-compose logs -f` | Follow logs for all services |
| `docker-compose restart` | Restart all services |
| `docker-compose exec SERVICE CMD` | Execute command in service |
| `docker-compose config` | Validate and view configuration |
| `docker-compose pull` | Pull service images |
| `docker-compose build` | Build service images |
| `docker-compose stop` | Stop services without removing |
| `docker-compose start` | Start stopped services |

---

## Completion Checklist

- [ ] SSH into Application Server 2 (stapp02) as steve
- [ ] Switched to root user
- [ ] Created /opt/security directory (exact path)
- [ ] Created /var/www/html directory
- [ ] Created /var/lib/mysql directory
- [ ] Created test PHP file in /var/www/html/
- [ ] Created docker-compose.yml at /opt/security/docker-compose.yml (exact path and name)
- [ ] Validated docker-compose.yml syntax
- [ ] Web service configured: php:apache, container php_web (exact name), port 5004:80
- [ ] DB service configured: mariadb:latest, container mysql_web (exact name), port 3306:3306
- [ ] Volumes correctly mapped
- [ ] Environment variables set (MYSQL_DATABASE=database_web, custom user NOT root)
- [ ] Started stack with docker-compose up -d
- [ ] Both containers running with exact names (php_web, mysql_web)
- [ ] Web accessible on port 5004
- [ ] Database created: database_web
- [ ] Database connection tested with custom user (NOT root)
- [ ] Service-to-service communication verified
- [ ] Ready for deployment (stack will be redeployed using compose file)
- [ ] All requirements met ✅

---

## Completion Details

- **Completion Date:** December 22, 2025
- **Day:** 46 / 100
- **Challenge:** KodeKloud 100 Days Cloud DevOps
- **Topic:** Docker Compose - Multi-Service Stack (PHP + MariaDB)
- **Server:** Application Server 2 (stapp02) in Stratos Datacenter
- **User:** steve
- **Compose File:** `/opt/security/docker-compose.yml` (exact path and name)
- **Services:** web (php_web), db (mysql_web)
- **Container Names:** php_web (exact), mysql_web (exact)
- **Images:** php:apache (~500MB), mariadb:latest (~400MB)
- **Ports:** 5004 (web), 3306 (database)
- **Database:** database_web
- **DB User:** webapp_user (custom, NOT root)
- **Test URL:** `curl http://stapp02:5004` or `curl localhost:5004`
- **Key Skill:** Multi-container orchestration with Docker Compose
- **Status:** ✅ Successfully Completed

---

## Summary

This task demonstrated **Docker Compose multi-service orchestration**:

✅ **Created docker-compose.yml** - Single configuration file for entire stack
✅ **Deployed two services** - PHP Apache web server + MariaDB database
✅ **Configured networking** - Automatic service discovery and communication
✅ **Mapped volumes** - Persistent data storage for web and database
✅ **Set environment variables** - Database initialization and credentials
✅ **One-command deployment** - `docker-compose up -d` launched entire stack

**Key Insight:** Docker Compose transforms **multi-container complexity into simple configuration**:

**Without Docker Compose (Manual):**
```bash
docker network create app-net
docker run -d --name mysql_web --network app-net -e MYSQL_DATABASE=... mariadb:latest
docker run -d --name php_web --network app-net --link mysql_web -p 5000:80 php:apache
# Complex, error-prone, hard to reproduce
```

**With Docker Compose:**
```yaml
version: '3'
services:
  web: ...
  db: ...
```
```bash
docker-compose up -d
# Simple, reproducible, version controlled!
```

**The LAMP Stack Architecture:**
```
User Request (port 5004)
    ↓
Apache Web Server (php_web)
    ↓
PHP Processes Request
    ↓
Needs Database?
    ↓
Connect to mysql_web:3306
    ↓
MariaDB Database
    ↓
Return Data
    ↓
Generate Response
    ↓
User Receives Web Page
```

**Service-to-Service Communication:**
```yaml
# Web can reach database by service name!
$conn = new PDO("mysql:host=mysql_web;dbname=database_web", ...);
                              ↑
                    Service name (automatic DNS)
```

**Volume Persistence:**
```
Host: /var/www/html/index.php
         ↓ (mounted to)
Container: /var/www/html/index.php
         ↓
Survives container restarts ✅
Edit on host → Instantly in container ✅
```

**Why Docker Compose for Multi-Service?**

| Feature | Manual Docker | Docker Compose |
|---------|---------------|----------------|
| **Configuration** | Multiple commands | Single YAML file |
| **Networking** | Manual network creation | Automatic |
| **Service Discovery** | Manual linking | Built-in DNS |
| **Volume Management** | Per-container flags | Centralized config |
| **Environment Setup** | Long command lines | Clean YAML syntax |
| **Reproducibility** | Hard to replicate | Version controlled |
| **Team Collaboration** | Share commands | Share YAML file |

**Best Practices Applied:**
- ✅ Used custom user (not root) for database
- ✅ Complex password for security
- ✅ Service dependency with `depends_on`
- ✅ Bind mounts for easy development
- ✅ Port mapping to avoid conflicts
- ✅ Separate services for separation of concerns

**Common Use Cases:**
```
Docker Compose Perfect For:
├── Development environments
├── Testing multi-service apps
├── CI/CD pipelines
├── Microservices locally
├── Database + application combos
└── Quick prototypes
```

**Production Considerations:**
```
For Production, Consider:
├── Named volumes (not bind mounts)
├── Health checks for reliability
├── Resource limits (CPU, memory)
├── Secrets management (not inline passwords)
├── Logging configuration
├── Restart policies (unless-stopped)
└── Container orchestration (Kubernetes for scale)
```

**Remember:** Docker Compose is perfect for **defining, configuring, and running multi-container applications** on a single host. For production at scale, consider Kubernetes or Docker Swarm! 🐳

**Real-World Impact:**
- Consistent development environments across team
- Easy onboarding (one command to start entire stack)
- Simplified testing and CI/CD
- Clear documentation of architecture
- Version-controlled infrastructure

**Next:** Explore Docker networking modes, implement service mesh, add reverse proxy (nginx), and scale services with Docker Swarm or Kubernetes! 🚀
