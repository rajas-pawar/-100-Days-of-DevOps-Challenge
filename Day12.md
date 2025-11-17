# Day 12: Troubleshooting Apache Web Server Connectivity Issues

## 📋 Objective
Investigate and resolve Apache web server connectivity issues on App Server 1 in Stratos Datacenter where the service is not reachable on port 3002, using systematic troubleshooting methodology.

## 🎯 Resolution
Successfully identified and resolved multiple issues preventing Apache from being accessible on port 3002: sendmail occupying the port, and iptables firewall blocking external connections.

## 📝 Task Steps for Resolution

1. **Test connectivity from Jump Host**
2. **SSH to App Server 1**
3. **Check Apache service status**
4. **Identify port conflict (sendmail on port 3002)**
5. **Stop conflicting service**
6. **Start Apache service**
7. **Identify firewall blocking issue**
8. **Add iptables rule for port 3002**
9. **Verify connectivity from Jump Host**

## 💻 Commands

### Phase 1: Initial Testing from Jump Host

```bash
# On Jump Host (thor user)

# Test connectivity to port 3002
curl http://stapp01:3002
# Error: Failed to connect to stapp01 port 3002: No route to host

# Test with telnet
telnet stapp01 3002
# Error: No route to host
```

### Phase 2: Access App Server 1 and Check Service

```bash
# SSH to App Server 1
ssh tony@stapp01
# Password: Ir0nM@n

# Switch to root
sudo su -

# Check Apache service status
systemctl status httpd
# Output: Active: failed (Result: exit-code)
# Error: (98)Address already in use: AH00072: make_sock: could not bind to address 0.0.0.0:3002
```

### Phase 3: Identify Port Conflict

```bash
# Check what's using port 3002
netstat -tlnp | grep 3002
# Output: tcp 0 0 127.0.0.1:3002 0.0.0.0:* LISTEN 503/sendmail: accep

# Sendmail is blocking port 3002!
```

### Phase 4: Stop Sendmail and Start Apache

```bash
# Stop sendmail service
systemctl stop sendmail

# Disable sendmail from auto-start
systemctl disable sendmail

# Verify port is now free
netstat -tlnp | grep 3002
# Should return nothing

# Start Apache
systemctl start httpd

# Enable Apache for auto-start
systemctl enable httpd

# Verify Apache is running
systemctl status httpd
# Output: Active: active (running)

# Verify Apache is listening on port 3002
netstat -tlnp | grep 3002
# Output: tcp 0 0 0.0.0.0:3002 0.0.0.0:* LISTEN 1085/httpd
```

### Phase 5: Test Locally (Works)

```bash
# Test from server itself
curl http://localhost:3002
# Returns HTML - Works!

# Test with server hostname
curl http://stapp01:3002
# Returns HTML - Works locally!
```

### Phase 6: Test from Jump Host (Still Fails - Firewall Issue!)

```bash
# Exit to Jump Host
exit
exit

# Test from Jump Host
curl http://stapp01:3002
# Error: Failed to connect to stapp01 port 3002: No route to host

# "No route to host" = Firewall blocking!
```

### Phase 7: Check and Configure Firewall

```bash
# SSH back to App Server 1
ssh tony@stapp01
sudo su -

# Check if firewalld exists
systemctl status firewalld
# Output: Unit firewalld.service could not be found

# Check iptables instead
iptables -L -n

# Output shows:
# Chain INPUT (policy ACCEPT)
# ACCEPT tcp -- 0.0.0.0/0 0.0.0.0/0 state NEW tcp dpt:22
# REJECT all -- 0.0.0.0/0 0.0.0.0/0 reject-with icmp-host-prohibited
# 
# Only SSH (22) is allowed! Everything else is REJECTED!
```

### Phase 8: Add iptables Rule for Port 3002

```bash
# Add rule to allow port 3002 BEFORE the REJECT rule
iptables -I INPUT -p tcp --dport 3002 -j ACCEPT

# Verify rule was added
iptables -L INPUT -n --line-numbers
# Should show port 3002 rule before REJECT rule

# Save iptables rules (persistence)
service iptables save
# or
iptables-save > /etc/sysconfig/iptables
```

### Phase 9: Final Verification

```bash
# Exit to Jump Host
exit
exit

# Test from Jump Host
curl http://stapp01:3002
# Should return HTML content!

# Test with telnet
telnet stapp01 3002
# Should connect successfully
```

## ✅ Verification

```bash
# On App Server 1:

# Apache is running
systemctl status httpd
# Output: Active: active (running)

# Listening on correct port
netstat -tlnp | grep 3002
# Output: tcp 0 0 0.0.0.0:3002 0.0.0.0:* LISTEN <PID>/httpd

# Sendmail is stopped
systemctl status sendmail
# Output: Active: inactive (dead)

# Firewall allows port 3002
iptables -L INPUT -n | grep 3002
# Shows: ACCEPT tcp -- 0.0.0.0/0 0.0.0.0/0 tcp dpt:3002

# From Jump Host:

# Curl works
curl http://stapp01:3002
# Returns HTML content

# Telnet connects
telnet stapp01 3002
# Connected to stapp01
```

## 🔑 Key Points

- **Two separate issues identified:**
  1. Port conflict - sendmail using port 3002
  2. Firewall blocking - iptables rejecting all except SSH
- **Systematic troubleshooting methodology:**
  - Test from source (Jump Host)
  - Check service status
  - Identify port conflicts
  - Test locally after fix
  - Identify remaining issues (firewall)
  - Fix firewall rules
  - Verify end-to-end
- **"No route to host" error** - Indicates firewall blocking
- **"Address already in use" error** - Indicates port conflict
- **iptables rule order matters** - ACCEPT rules must come before REJECT rules
- **Use -I (insert) not -A (append)** - Inserts at top, before REJECT rule
- **Common mistakes:**
  - Fixing only one issue (service OR firewall, not both)
  - Using -A instead of -I for iptables (rule goes after REJECT)
  - Not making iptables rules persistent
  - Not testing from original source (Jump Host)
- **Tools used:**
  - `systemctl` - Service management
  - `netstat` - Port and process checking
  - `iptables` - Firewall management (when firewalld not available)
  - `curl` - HTTP testing
  - `telnet` - Connectivity testing

## ⚠️ Important Notes

- Multiple issues can exist simultaneously (service + firewall)
- Local testing success ≠ external access working
- "No route to host" strongly indicates firewall blocking
- iptables rules must be inserted (-I) before REJECT rules
- Always test from the actual client (Jump Host) after fixes
- Save iptables rules for persistence across reboots
- Document all issues found and fixes applied
- Enable services for auto-start (systemctl enable)
- Port conflicts are common - always check what's listening
- Systematic approach prevents missing issues

---

**Date Completed**: November 17, 2025  
**Challenge**: KodeKloud Cloud DevOps  
**Day**: 12/100