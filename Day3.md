# Day 3: Disabling Direct SSH Root Login

## 📋 Objective
Disable direct SSH root login on all app servers in Stratos Datacenter following security audit recommendations at xFusionCorp Industries.

## 🎯 Resolution
Successfully disabled direct root SSH access across all app servers by modifying SSH daemon configuration, forcing users to authenticate as regular users before escalating privileges.

## 📝 Task Steps for Resolution

1. **Connect to each App Server** (App Server 1, 2, and 3)
2. **Switch to root user**
3. **Backup SSH configuration file**
4. **Edit SSH daemon configuration**
5. **Disable root login**
6. **Restart SSH service**
7. **Verify configuration**
8. **Test the changes**

## 💻 Commands

### For App Server 1
```bash
# Step 1: SSH into App Server 1
ssh tony@stapp01

# Step 2: Switch to root
sudo su -

# Step 3: Backup SSH config (IMPORTANT!)
cp /etc/ssh/sshd_config /etc/ssh/sshd_config.backup

# Step 4: Edit SSH configuration
vi /etc/ssh/sshd_config
# OR
sed -i 's/^#PermitRootLogin yes/PermitRootLogin no/' /etc/ssh/sshd_config
sed -i 's/^PermitRootLogin yes/PermitRootLogin no/' /etc/ssh/sshd_config

# Step 5: Verify the change
grep "^PermitRootLogin" /etc/ssh/sshd_config
# Output should be: PermitRootLogin no

# Step 6: Test SSH configuration syntax
sshd -t

# Step 7: Restart SSH service
systemctl restart sshd
# OR (for older systems)
service sshd restart

# Step 8: Check SSH service status
systemctl status sshd
```

### For App Server 2
```bash
# SSH into App Server 2
ssh steve@stapp02

# Follow the same steps as App Server 1
sudo su -
cp /etc/ssh/sshd_config /etc/ssh/sshd_config.backup
sed -i 's/^#PermitRootLogin yes/PermitRootLogin no/' /etc/ssh/sshd_config
sed -i 's/^PermitRootLogin yes/PermitRootLogin no/' /etc/ssh/sshd_config
grep "^PermitRootLogin" /etc/ssh/sshd_config
sshd -t
systemctl restart sshd
```

### For App Server 3
```bash
# SSH into App Server 3
ssh banner@stapp03

# Follow the same steps
sudo su -
cp /etc/ssh/sshd_config /etc/ssh/sshd_config.backup
sed -i 's/^#PermitRootLogin yes/PermitRootLogin no/' /etc/ssh/sshd_config
sed -i 's/^PermitRootLogin yes/PermitRootLogin no/' /etc/ssh/sshd_config
grep "^PermitRootLogin" /etc/ssh/sshd_config
sshd -t
systemctl restart sshd
```

## ✅ Verification

```bash
# Check configuration is applied
grep "^PermitRootLogin" /etc/ssh/sshd_config
# Expected: PermitRootLogin no

# Verify SSH service is running
systemctl status sshd
# Should show: active (running)

# Test SSH syntax (no output means success)
sshd -t

# Check all SSH configuration related to root
grep -i "permitroot" /etc/ssh/sshd_config

# From another terminal, try to SSH as root (should fail)
ssh root@stapp01
# Expected: Permission denied or Access denied
```

## 🔑 Key Points

- **PermitRootLogin no** - Disables direct root SSH access
- Always **backup configuration** before making changes
- **Test syntax** with `sshd -t` before restarting service
- **Keep current session open** while testing to avoid lockout
- SSH config location: `/etc/ssh/sshd_config`
- Changes require SSH service restart to take effect
- Users must login with regular accounts, then use `sudo` or `su`
- Other PermitRootLogin options:
  - `yes` - Allow root login (insecure)
  - `no` - Disable root login (recommended)
  - `without-password` - Allow only key-based root login
  - `prohibit-password` - Same as without-password (newer syntax)
  - `forced-commands-only` - Allow root only for specific commands
- This is a **CIS Benchmark** requirement for security hardening
- Protects against brute-force attacks on root account
- Implements principle of least privilege
- Audit trails become more meaningful (individual user accountability)

## ⚠️ Important Notes

- **DO NOT close your active SSH session** until verification is complete
- Always keep one root session open as backup
- If locked out, console access will be needed
- Ensure `sudo` is properly configured for administrative users
- Test from a different terminal before disconnecting

---

**Date Completed**: November 8, 2025  
**Challenge**: KodeKloud 100 Days Cloud DevOps  
**Day**: 3/100