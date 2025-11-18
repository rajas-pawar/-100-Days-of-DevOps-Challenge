# Day 13 - Securing Apache Port with iptables 🔒

## 📋 Title
Implementing iptables Firewall Rules to Secure Apache Port 8088 on Nautilus Infrastructure

## 🎯 Objective
Configure firewall rules on app servers to restrict Apache port 8088 access only to the Load Balancer, blocking all other incoming traffic.

## ✅ Resolution
Successfully installed iptables on all app hosts, configured persistent firewall rules allowing port 8088 access exclusively from LBR host while blocking all other traffic.

## 📝 Task
Our security team identified that Apache's port 8088 is open to all hosts without firewall protection in the Stratos DC. We need to:
1. Install iptables and dependencies on each app host
2. Block incoming port 8088 for everyone except LBR host
3. Ensure rules persist after system reboot

## 🔧 Steps for Resolution

### Step 1: Identify Infrastructure Details
```bash
# From jump host, check the hosts file to find LBR IP
sudo cat /etc/hosts

# Look for stlb01 (Load Balancer) - typically 172.16.238.14
# App servers: stapp01, stapp02, stapp03
```

**Nautilus Environment Mapping:**
| Server | User | Password | IP |
|--------|------|----------|-----|
| stapp01 | tony | Ir0nM@n | 172.16.238.10 |
| stapp02 | steve | Am3ric@ | 172.16.238.11 |
| stapp03 | robot | ROb0t@ | 172.16.238.12 |
| stlb01 (LBR) | loki | Mischi3f | 172.16.238.14 |

### Step 2: Install iptables on Each App Host

**Connect to first app server:**
```bash
# From jump host
ssh tony@stapp01
# Password: Ir0nM@n

# Install iptables
sudo yum install -y iptables-services
```

**Repeat for other app servers:**
```bash
# App Server 2
ssh steve@stapp02
# Password: Am3ric@
sudo yum install -y iptables-services

# App Server 3
ssh robot@stapp03
# Password: ROb0t@
sudo yum install -y iptables-services
```

### Step 3: Configure Firewall Rules
```bash
# Allow traffic from LBR host on port 8088
sudo iptables -I INPUT -p tcp -s <LBR_HOST_IP> --dport 8088 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT

# Block all other traffic on port 8088
sudo iptables -A INPUT -p tcp --dport 8088 -j DROP

# Verify rules
sudo iptables -L -n -v --line-numbers
```

### Step 4: Make Rules Persistent

**RHEL/CentOS:**
```bash
# Save rules
sudo service iptables save

# Enable on boot
sudo systemctl enable iptables
sudo systemctl status iptables
```

**Ubuntu/Debian:**
```bash
# Save rules
sudo netfilter-persistent save
sudo systemctl enable netfilter-persistent
```

### Step 5: Verify Configuration
```bash
# Check saved rules
sudo cat /etc/sysconfig/iptables  # RHEL/CentOS
sudo cat /etc/iptables/rules.v4   # Ubuntu/Debian

# Test from LBR host (should succeed)
curl -I http://<APP_HOST_IP>:8088

# Test from another host (should fail)
telnet <APP_HOST_IP> 8088
```

## 📌 Commands Summary

```bash
# Installation
sudo yum install -y iptables-services

# Configuration
sudo iptables -I INPUT -p tcp -s <LBR_IP> --dport 8088 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 8088 -j DROP

# Persistence
sudo service iptables save
sudo systemctl enable iptables

# Verification
sudo iptables -L -n -v
```

## ✅ Verification Steps

1. **Check Rules Active:**
   ```bash
   sudo iptables -L INPUT -n -v | grep 8088
   ```

2. **Test Connectivity:**
   - From LBR: `curl http://<app-host>:8088` ✅ Should work
   - From other host: `curl http://<app-host>:8088` ❌ Should timeout

3. **Verify Persistence:**
   ```bash
   sudo reboot
   # After reboot
   sudo iptables -L -n -v | grep 8088
   ```

## 🔑 Key Points

- **Rule Order Matters**: ACCEPT rules must come before DROP rules
- **Use `-I` (Insert)** for ACCEPT rules to place them at the top
- **Use `-A` (Append)** for DROP rules to place them at the bottom
- **Connection Tracking**: Using `-m conntrack --ctstate NEW,ESTABLISHED` ensures proper connection handling
- **Persistence**: Different methods for RHEL/CentOS vs Ubuntu/Debian
- **Testing**: Always test from both allowed and blocked sources
- **Backup**: Save current rules before making changes: `sudo iptables-save > backup.rules`

## 🎓 Learning Outcomes

- Understanding iptables INPUT chain and rule ordering
- Implementing source-based IP filtering
- Making firewall rules persistent across reboots
- Securing web services in multi-tier architecture

---

**Environment**: Nautilus Infrastructure, Stratos DC  
**Security Level**: Port-level access control  
**Day**: 13 of DevOps Journey 🚀