# Day 16: Configuring Nginx Load Balancer for High Availability
**100 Days Cloud DevOps Challenge – KodeKloud**

---

## Objective

The Nautilus production support team observed increased traffic on one of their websites, causing performance degradation. To achieve high availability, the application was migrated to a multi-server backend architecture. The final step was configuring the Load Balancer (LBR) to distribute traffic across all application servers.

This task covers:
* Installing Nginx on the LBR server
* Configuring Nginx as a load balancer
* Ensuring Apache is running on all app servers
* Directing all frontend traffic to backend servers via Nginx

---

## Infrastructure Overview

### Load Balancer (LBR):
* **Hostname:** `stlb01`
* **Service:** `nginx`

### Backend Application Servers (Apache):

| Server  | IP            | Port |
|---------|---------------|------|
| stapp01 | 172.16.238.10 | 6300 |
| stapp02 | 172.16.238.11 | 6300 |
| stapp03 | 172.16.238.12 | 6300 |

---

## Step-by-Step Implementation

### 1. Install and Start Nginx on LBR
```bash
yum install -y nginx
systemctl enable nginx
systemctl start nginx
```

### 2. Verify Apache on All App Servers

Each app server must be running Apache on port 6300.

**stapp01:**
```bash
ssh tony@stapp01
sudo ss -tulpn | grep httpd
```

**stapp02:**
```bash
ssh steve@stapp02
sudo ss -tulpn | grep httpd
```

**stapp03:**
```bash
ssh banner@stapp03
sudo ss -tulpn | grep httpd
```

**Expected output:**
```
tcp LISTEN 0 511 0.0.0.0:6300 0.0.0.0:* users:(("httpd"...))
```

### 3. Configure Nginx Load Balancing

Edit the main Nginx configuration file:
```bash
vi /etc/nginx/nginx.conf
```

Replace the default server block and insert the following configuration:

#### Final Load Balancer Configuration (`/etc/nginx/nginx.conf`)
```nginx
# For more information on configuration, see:
# * http://nginx.org/en/docs/

user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;

include /usr/share/nginx/modules/*.conf;

events {
    worker_connections 1024;
}

http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    tcp_nopush      on;
    tcp_nodelay     on;
    keepalive_timeout 65;

    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    upstream backend_servers {
        server stapp01:6300;
        server stapp02:6300;
        server stapp03:6300;
    }

    server {
        listen 80;
        server_name _;

        location / {
            proxy_pass http://backend_servers;
        }
    }
}
```

### 4. Validate and Restart Nginx
```bash
nginx -t
systemctl restart nginx
```

**Expected output:**
```
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

### 5. Test Load Balancer
```bash
curl http://localhost
```

**Expected response (will vary depending on backend HTML):**
```
Welcome to xFusionCorp Industries!
```

Also validate using the **StaticApp** button in KodeKloud.

---

## Troubleshooting

### 1. 502 Bad Gateway

Check Apache on backends:
```bash
curl stapp01:6300
```

### 2. Nginx not running

Check logs:
```bash
journalctl -u nginx -n 50
```

### 3. Wrong port

Verify with:
```bash
ss -tulpn | grep httpd
```

---

## Key Takeaways

* Nginx acts as a reverse proxy and distributes traffic across multiple servers
* Apache runs independently on backend servers
* Only `/etc/nginx/nginx.conf` should be modified
* Default algorithm: Round Robin load balancing
* Ensures scalability and high availability under increasing load

---

## Completion Details

- **Completion Date:** November 21, 2025
- **Day:** 16 / 100
- **Challenge:** KodeKloud 100 Days Cloud DevOps
- **Topic:** High Availability Load Balancer Configuration