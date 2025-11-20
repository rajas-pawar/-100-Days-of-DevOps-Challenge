# Day 15: Deploying Nginx with SSL on App Server 3
**100 Days Cloud DevOps Challenge – KodeKloud**

---

## Objective

Prepare App Server 3 (stapp03) for deployment by completing the following:

1. Install and configure Nginx
2. Move the provided SSL certificate and key to a secure location
3. Configure Nginx to serve HTTPS using those certificates
4. Create a basic `index.html` with the content: `Welcome!`
5. Validate HTTPS access from the jump host

---

## Environment Details

| Server     | User   | Password | IP            |
|------------|--------|----------|---------------|
| stapp03    | banner | B4nn3r!  | 172.16.238.12 |
| Jump Host  | thor   |          | 172.16.238.5  |

**SSL certificate files provided:**
- `/tmp/nautilus.crt`
- `/tmp/nautilus.key`

---

## Step-by-Step Implementation

### 1. SSH into App Server 3
```bash
ssh banner@stapp03
sudo su -
```

### 2. Install Nginx
```bash
yum install -y nginx
systemctl enable nginx
systemctl start nginx
systemctl status nginx
```

### 3. Create SSL Directory & Move Certificates
```bash
mkdir -p /etc/nginx/ssl
mv /tmp/nautilus.crt /etc/nginx/ssl/
mv /tmp/nautilus.key /etc/nginx/ssl/
chmod 600 /etc/nginx/ssl/nautilus.key
```

### 4. Configure Nginx for SSL

Create an SSL configuration file:
```bash
vi /etc/nginx/conf.d/ssl.conf
```

Add the following configuration:
```nginx
server {
    listen 443 ssl;
    server_name stapp03;

    ssl_certificate /etc/nginx/ssl/nautilus.crt;
    ssl_certificate_key /etc/nginx/ssl/nautilus.key;

    root /usr/share/nginx/html;
    index index.html;
}
```

Test and reload configuration:
```bash
nginx -t
systemctl restart nginx
```

### 5. Create index.html
```bash
echo "Welcome!" > /usr/share/nginx/html/index.html
```

Verify file:
```bash
cat /usr/share/nginx/html/index.html
```

### 6. Validate HTTPS Locally on Server
```bash
curl -Ik https://localhost/
```

**Expected output:**
```
HTTP/1.1 200 OK
```

### 7. Validate From Jump Host
```bash
curl -Ik https://stapp03/
```

---

## Full Commands (One-Block Copy-Paste)
```bash
yum install -y nginx
systemctl enable nginx
systemctl start nginx

mkdir -p /etc/nginx/ssl
mv /tmp/nautilus.crt /etc/nginx/ssl/
mv /tmp/nautilus.key /etc/nginx/ssl/
chmod 600 /etc/nginx/ssl/nautilus.key

echo 'server {
    listen 443 ssl;
    server_name stapp03;

    ssl_certificate /etc/nginx/ssl/nautilus.crt;
    ssl_certificate_key /etc/nginx/ssl/nautilus.key;

    root /usr/share/nginx/html;
    index index.html;
}' > /etc/nginx/conf.d/ssl.conf

nginx -t
systemctl restart nginx

echo "Welcome!" > /usr/share/nginx/html/index.html

curl -Ik https://localhost/
```

---

## Troubleshooting Notes

### Check if Nginx is listening on port 443
```bash
ss -tulpn | grep nginx
```

### Ensure firewall is not blocking HTTPS (if firewalld exists)
```bash
firewall-cmd --add-service=https --permanent
firewall-cmd --reload
```

### View Nginx logs
```bash
journalctl -u nginx -f
```

---

## Key Takeaways

- Certificate and key must be stored securely under `/etc/nginx/ssl/`
- Private key requires restrictive permissions (600)
- Nginx SSL configuration can be placed in `/etc/nginx/conf.d/ssl.conf`
- Using `nginx -t` before restarting helps prevent downtime
- Always validate from both server and jump host

---

## Completion Details

- **Completion Date:** November 20, 2025
- **Day:** 15 / 100
- **Challenge:** KodeKloud 100 Days Cloud DevOps
- **Topic:** SSL-enabled Nginx Deployment on Linux