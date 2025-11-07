# Day 2: Creating User Account with Expiry Date

## 📋 Objective
Create a temporary user account named `javed` on App Server 2 in Stratos Datacenter with an expiry date of `2023-12-07` for the Nautilus project.

## 🎯 Resolution
Successfully created user `javed` with account expiration set to December 7, 2023, implementing temporary access management for project requirements.

## 📝 Task Steps for Resolution

1. **Connect to App Server 2**
2. **Switch to root user**
3. **Create user with expiry date**
4. **Set password for the user** (if required)
5. **Verify user creation and expiry date**

## 💻 Commands

```bash
# Step 1: SSH into App Server 2
ssh steve@stapp02

# Step 2: Switch to root user
sudo su -

# Step 3: Create user javed with expiry date
useradd javed -e 2023-12-07

# Alternative: Create user first, then set expiry
useradd javed
chage -E 2023-12-07 javed

# Step 4: Set password (if needed)
passwd javed
```

## ✅ Verification

```bash
# Verify user exists
id javed
# Output: uid=1002(1002) gid=1002(javed) groups=1002(javed)

# Check user in /etc/passwd
grep javed /etc/passwd
# Output: javed:x:1002:1002::/home/javed:/bin/bash

# Verify expiry date using chage
chage -l javed
# Look for: Account expires : Dec 07, 2023

# Alternative verification
getent shadow javed | cut -d: -f8
# Output: 19698 (days since epoch for 2023-12-07)

# Check expiry with date calculation
date -d @$(( $(getent shadow javed | cut -d: -f8) * 86400 ))
```

## 🔑 Key Points

- **User expiry** automatically disables account after specified date
- Expiry date format: `YYYY-MM-DD` (ISO format)
- `-e` flag with `useradd` sets expiry during creation
- `chage -E` command sets/modifies expiry for existing users
- Expired accounts cannot login but data remains intact
- `chage -l username` shows detailed account aging information
- Expiry date stored in `/etc/shadow` file (8th field)
- Use `-1` or empty date to remove expiry: `chage -E -1 username`
- Different from password expiry (password aging vs account expiry)
- Temporary accounts are common for contractors, vendors, and project-based access

---

**Date Completed**: November 7, 2025  
**Challenge**: KodeKloud 100 Days Cloud DevOps  
**Day**: 2/100