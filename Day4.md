# Day 4: Granting Executable Permissions to Script

## 📋 Objective
Grant executable permissions to the `/tmp/xfusioncorp.sh` backup script on App Server 1 in Stratos Datacenter, ensuring all users can execute it.

## 🎯 Resolution
Successfully applied executable permissions to the backup script using chmod command, allowing all users (owner, group, others) to execute the script.

## 📝 Task Steps for Resolution

1. **Connect to App Server 1**
2. **Switch to root user**
3. **Verify current permissions**
4. **Grant executable permissions to all users**
5. **Verify permissions are applied**
6. **Test script execution** (optional)

## 💻 Commands

```bash
# Step 1: SSH into App Server 1
ssh tony@stapp01

# Step 2: Switch to root user
sudo su -

# Step 3: Check current permissions
ls -l /tmp/xfusioncorp.sh
# Output example: -rw-r--r-- 1 root root 256 Nov 8 10:00 /tmp/xfusioncorp.sh

# Step 4: Grant executable permissions to all users
chmod +x /tmp/xfusioncorp.sh

# Alternative methods:
chmod a+x /tmp/xfusioncorp.sh        # Explicit: all users (+x)
chmod 755 /tmp/xfusioncorp.sh        # Numeric: rwxr-xr-x
chmod u+x,g+x,o+x /tmp/xfusioncorp.sh # Explicit: each category
```

## ✅ Verification

```bash
# Verify permissions are applied
ls -l /tmp/xfusioncorp.sh
# Expected output: -rwxr-xr-x 1 root root 256 Nov 8 10:00 /tmp/xfusioncorp.sh
#                   ^^^ ^^^ ^^^
#                   Owner Group Others - all have execute (x)

# Check permissions in numeric format
stat -c '%a %n' /tmp/xfusioncorp.sh
# Expected output: 755 /tmp/xfusioncorp.sh

# Detailed file information
stat /tmp/xfusioncorp.sh

# Test execution as current user
/tmp/xfusioncorp.sh
# or
bash /tmp/xfusioncorp.sh

# Test with another user (optional)
su - tony
/tmp/xfusioncorp.sh
```

## 🔑 Key Points

- **chmod** - Command to change file permissions
- **+x** - Adds execute permission
- **a+x** - Adds execute permission for all (owner, group, others)
- Three permission categories:
  - **Owner (u)** - The user who owns the file
  - **Group (g)** - Users in the file's group
  - **Others (o)** - Everyone else
- Three permission types:
  - **Read (r)** - View file contents (4)
  - **Write (w)** - Modify file contents (2)
  - **Execute (x)** - Run file as program (1)
- **Symbolic notation**: `rwxr-xr-x` (letters)
- **Numeric notation**: `755` (numbers)
  - First digit: Owner permissions
  - Second digit: Group permissions
  - Third digit: Others permissions
- Calculate numeric: `r(4) + w(2) + x(1) = 7`
  - `755` = `rwxr-xr-x` (owner: rwx, group: r-x, others: r-x)
  - `777` = `rwxrwxrwx` (all permissions to everyone - use carefully!)
  - `644` = `rw-r--r--` (common for files)
  - `755` = `rwxr-xr-x` (common for scripts/executables)
- Scripts need execute permission to run directly: `./script.sh`
- Without execute permission, use: `bash script.sh` or `sh script.sh`
- **Best practice**: Give minimum necessary permissions
- Scripts in `/tmp` are temporary and typically cleared on reboot

---

**Date Completed**: November 9, 2025  
**Challenge**: KodeKloud 100 Days Cloud DevOps  
**Day**: 4/100