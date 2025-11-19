# Day 14: Troubleshooting Apache Failure and Securing Port 5001 with iptables

## 📋 Objective
Restore Apache service on App Server 1 (stapp01) where it was failing due to a port conflict, ensure Apache listens on port 5001, and secure the port using iptables so only the Load Balancer can access it.

## 🎯 Resolution
Successfully identified that the Apache service was down because **sendmail** was occupying port **5001**.  
Stopped and disabled sendmail, restarted Apache, verified service health, and applied iptables rules allowing only the Load Balancer to reach port 5001.

## 📝 Task Steps for Resolution

### 1. SSH into App Server 1
```bash
ssh tony@stapp01
```

### 2. Check Apache Service Status
```bash
sudo systemctl status httpd
```

### 3. Identify Port Conflict
```bash
sudo netstat -tulpn | grep :5001
```
Output showed:
```
770/sendmail: accept
```

### 4. Stop and Disable sendmail
```bash
sudo systemctl stop sendmail
sudo systemctl disable sendmail
```

### 5. Restart Apache Service
```bash
sudo systemctl restart httpd
sudo systemctl status httpd
```

### 6. Verify Apache Listening on Port 5001
```bash
sudo ss -tulpn | grep httpd
```

## 🔐 Securing Apache with iptables

### 1. Allow Only Load Balancer Access
```bash
sudo iptables -A INPUT -p tcp -s 172.16.238.3 --dport 5001 -j ACCEPT
```

### 2. Block All Other Access
```bash
sudo iptables -A INPUT -p tcp --dport 5001 -j DROP
```

### 3. Save Rules
```bash
sudo iptables-save | sudo tee /etc/sysconfig/iptables
```

### 4. Restart iptables
```bash
sudo systemctl restart iptables
```

## 🔍 Verification

### Apache Health
```bash
curl -I http://localhost:5001
```

### Port Status
```bash
sudo ss -tulpn | grep :5001
```

### Firewall Test
✔ From Load Balancer → Allowed  
❌ From any other server → Blocked

## 🔑 Key Points

- Apache failed due to **port conflict** with sendmail.
- Port conflicts are diagnosed using **netstat** or **ss**.
- Stopping and disabling the conflicting service resolved the issue.
- iptables used to restrict access to port 5001.
- Always save iptables rules to persist after reboot.
- Apache config syntax can be tested using:
```bash
httpd -t
```

---

**Date Completed**: November 19, 2025  
**Challenge**: KodeKloud 100 Days Cloud DevOps  
**Day**: 14/100
