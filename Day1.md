# Day 1: Creating User with Non-Interactive Shell

## 📋 Objective
Create a user named `jim` with a non-interactive shell on App Server 3 for the backup agent tool at xFusionCorp Industries.

## 🎯 Resolution
Successfully created user `jim` with `/sbin/nologin` shell to prevent interactive login while allowing the backup agent to function properly.

## 📝 Task Steps for Resolution

1. **Connect to App Server 3**
2. **Switch to root user** (if not already root)
3. **Create user with non-interactive shell**
4. **Verify user creation**
5. **Confirm shell assignment**

## 💻 Commands

```bash
# Step 1: SSH into App Server 3
ssh banner@stapp03

# Step 2: Switch to root user
sudo su -

# Step 3: Create user jim with non-interactive shell
useradd jim -s /sbin/nologin

# Alternative method (if you want to set password)
useradd jim -s /sbin/nologin
passwd jim
```

## ✅ Verification

```bash
# Verify user exists in /etc/passwd
grep jim /etc/passwd
# Output: jim:x:1001:1001::/home/jim:/sbin/nologin

# Check user details
id jim
# Output: uid=1001(jim) gid=1001(jim) groups=1001(jim)

# Verify shell
getent passwd jim | cut -d: -f7
# Output: /sbin/nologin

# Try to login as jim (should fail)
su - jim
# Output: This account is currently not available.
```

## 🔑 Key Points

- **Non-interactive shell** (`/sbin/nologin`) prevents user from getting shell access
- User can still be used by processes/services but cannot login interactively
- Common shells:
  - `/bin/bash` - Interactive shell (default)
  - `/sbin/nologin` - Non-interactive, blocks login
  - `/bin/false` - Non-interactive, returns false
- `useradd` vs `adduser`: `useradd` is low-level, `adduser` is interactive wrapper
- Always verify user creation after execution
- Useful for service accounts, backup agents, and system processes

---

**Date Completed**: November 6, 2025  
**Challenge**: KodeKloud 100 Days Cloud DevOps  
**Day**: 1/100