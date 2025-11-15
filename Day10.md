# Day 10: Creating Automated Website Backup Script

## 📋 Objective
Create an automated bash script `beta_backup.sh` on App Server 2 to backup the static website, save locally, and copy to Nautilus Backup Server without password prompts for the xFusionCorp Industries production team.

## 🎯 Resolution
Successfully created a bash script that creates a zip archive of the website directory, stores it locally in /backup/, and automatically copies it to the backup server using SSH key-based authentication.

## 📝 Task Steps for Resolution

1. **Setup SSH passwordless authentication** (steve to clint)
2. **Install zip package on App Server 2**
3. **Create required directories**
4. **Create the backup script**
5. **Make script executable**
6. **Test the script**
7. **Verify backup on both servers**

## 💻 Commands

### Phase 1: Setup SSH Passwordless Authentication

```bash
# Step 1: SSH to App Server 2
ssh steve@stapp02
# Password: Am3ric@

# Step 2: Generate SSH key (if not exists)
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa -N ""

# Step 3: Copy public key to Backup Server
ssh-copy-id clint@stbkp01
# Enter clint's password: H@wk3y3

# Step 4: Test passwordless SSH
ssh clint@stbkp01 "echo 'SSH connection successful'"
# Should connect without password prompt

# Exit back to App Server 2
exit
```

### Phase 2: Install Required Packages

```bash
# Switch to root (for package installation only)
sudo su -

# Install zip package
yum install -y zip
# or for Debian/Ubuntu
apt-get install -y zip

# Verify installation
which zip
zip --version

# Exit root
exit
```

### Phase 3: Create Required Directories

```bash
# Create /scripts directory (as root)
sudo mkdir -p /scripts

# Create /backup directory (as root)
sudo mkdir -p /backup

# Set permissions so steve can write
sudo chown steve:steve /scripts
sudo chown steve:steve /backup

# Verify permissions
ls -ld /scripts
ls -ld /backup

# Ensure backup directory exists on backup server
ssh clint@stbkp01 "sudo mkdir -p /backup && sudo chown clint:clint /backup"
```

### Phase 4: Create the Backup Script

```bash
# Create the script
vi /scripts/beta_backup.sh

# Or use cat to create directly:
cat > /scripts/beta_backup.sh << 'EOF'
#!/bin/bash

##############################################
# Website Backup Script for xFusionCorp Beta
# Description: Creates zip archive and copies to backup server
# Author: Production Support Team
# Date: 2025-11-15
##############################################

# Variables
SOURCE_DIR="/var/www/html/beta"
BACKUP_DIR="/backup"
ARCHIVE_NAME="xfusioncorp_beta.zip"
BACKUP_SERVER="clint@stbkp01"
REMOTE_BACKUP_DIR="/backup"

# Create backup directory if it doesn't exist
if [ ! -d "$BACKUP_DIR" ]; then
    mkdir -p "$BACKUP_DIR"
fi

# Remove old archive if exists
if [ -f "$BACKUP_DIR/$ARCHIVE_NAME" ]; then
    rm -f "$BACKUP_DIR/$ARCHIVE_NAME"
fi

# Create zip archive
echo "Creating backup archive..."
cd /var/www/html
zip -r "$BACKUP_DIR/$ARCHIVE_NAME" beta/

# Check if archive was created successfully
if [ $? -eq 0 ]; then
    echo "Backup archive created successfully: $BACKUP_DIR/$ARCHIVE_NAME"
    
    # Get archive size
    ARCHIVE_SIZE=$(du -h "$BACKUP_DIR/$ARCHIVE_NAME" | cut -f1)
    echo "Archive size: $ARCHIVE_SIZE"
else
    echo "Error: Failed to create backup archive"
    exit 1
fi

# Copy archive to backup server
echo "Copying archive to backup server..."
scp "$BACKUP_DIR/$ARCHIVE_NAME" "$BACKUP_SERVER:$REMOTE_BACKUP_DIR/"

# Check if copy was successful
if [ $? -eq 0 ]; then
    echo "Backup copied successfully to $BACKUP_SERVER:$REMOTE_BACKUP_DIR/"
    echo "Backup completed at: $(date)"
else
    echo "Error: Failed to copy backup to remote server"
    exit 1
fi

# Display completion message
echo "========================================="
echo "Backup Process Completed Successfully!"
echo "Local backup: $BACKUP_DIR/$ARCHIVE_NAME"
echo "Remote backup: $BACKUP_SERVER:$REMOTE_BACKUP_DIR/$ARCHIVE_NAME"
echo "========================================="

exit 0
EOF

# Alternative: More detailed script with logging
cat > /scripts/beta_backup.sh << 'EOF'
#!/bin/bash

##############################################
# Website Backup Script for xFusionCorp Beta
# Description: Creates zip archive and copies to backup server
# Author: Production Support Team
# Date: 2025-11-15
##############################################

# Variables
SOURCE_DIR="/var/www/html/beta"
BACKUP_DIR="/backup"
ARCHIVE_NAME="xfusioncorp_beta.zip"
BACKUP_SERVER="clint@stbkp01"
REMOTE_BACKUP_DIR="/backup"
TIMESTAMP=$(date +"%Y-%m-%d %H:%M:%S")

# Function to print messages
print_message() {
    echo "[$TIMESTAMP] $1"
}

# Check if source directory exists
if [ ! -d "$SOURCE_DIR" ]; then
    print_message "ERROR: Source directory $SOURCE_DIR does not exist"
    exit 1
fi

# Create backup directory if it doesn't exist
if [ ! -d "$BACKUP_DIR" ]; then
    mkdir -p "$BACKUP_DIR"
    print_message "Created backup directory: $BACKUP_DIR"
fi

# Remove old archive if exists
if [ -f "$BACKUP_DIR/$ARCHIVE_NAME" ]; then
    rm -f "$BACKUP_DIR/$ARCHIVE_NAME"
    print_message "Removed old archive"
fi

# Create zip archive
print_message "Creating backup archive from $SOURCE_DIR..."
cd /var/www/html
zip -r "$BACKUP_DIR/$ARCHIVE_NAME" beta/ > /dev/null 2>&1

# Check if archive was created successfully
if [ $? -eq 0 ] && [ -f "$BACKUP_DIR/$ARCHIVE_NAME" ]; then
    ARCHIVE_SIZE=$(du -h "$BACKUP_DIR/$ARCHIVE_NAME" | cut -f1)
    print_message "SUCCESS: Backup archive created ($ARCHIVE_SIZE)"
else
    print_message "ERROR: Failed to create backup archive"
    exit 1
fi

# Copy archive to backup server
print_message "Copying archive to backup server $BACKUP_SERVER..."
scp -q "$BACKUP_DIR/$ARCHIVE_NAME" "$BACKUP_SERVER:$REMOTE_BACKUP_DIR/"

# Check if copy was successful
if [ $? -eq 0 ]; then
    print_message "SUCCESS: Backup copied to remote server"
    
    # Verify file exists on remote server
    ssh -q "$BACKUP_SERVER" "test -f $REMOTE_BACKUP_DIR/$ARCHIVE_NAME"
    if [ $? -eq 0 ]; then
        REMOTE_SIZE=$(ssh -q "$BACKUP_SERVER" "du -h $REMOTE_BACKUP_DIR/$ARCHIVE_NAME" | cut -f1)
        print_message "Remote file verified ($REMOTE_SIZE)"
    fi
else
    print_message "ERROR: Failed to copy backup to remote server"
    exit 1
fi

# Display completion summary
echo "========================================="
print_message "Backup Process Completed Successfully!"
echo "Local backup:  $BACKUP_DIR/$ARCHIVE_NAME"
echo "Remote backup: $BACKUP_SERVER:$REMOTE_BACKUP_DIR/$ARCHIVE_NAME"
echo "========================================="

exit 0
EOF
```

### Phase 5: Make Script Executable

```bash
# Set executable permissions
chmod +x /scripts/beta_backup.sh

# Verify permissions
ls -l /scripts/beta_backup.sh
# Should show: -rwxr-xr-x
```

### Phase 6: Test the Script

```bash
# Run the script as steve user
/scripts/beta_backup.sh

# Expected output:
# Creating backup archive...
# Backup archive created successfully: /backup/xfusioncorp_beta.zip
# Archive size: 2.3M
# Copying archive to backup server...
# Backup copied successfully to clint@stbkp01:/backup/
# Backup completed at: Thu Nov 15 10:30:45 UTC 2025
# =========================================
# Backup Process Completed Successfully!
# Local backup: /backup/xfusioncorp_beta.zip
# Remote backup: clint@stbkp01:/backup/xfusioncorp_beta.zip
# =========================================
```

## ✅ Verification

```bash
# Verify script exists and is executable
ls -l /scripts/beta_backup.sh
# Output: -rwxr-xr-x 1 steve steve 1234 Nov 15 10:25 /scripts/beta_backup.sh

# Verify local backup was created
ls -lh /backup/xfusioncorp_beta.zip
# Should show file with size

# Test zip file integrity
unzip -t /backup/xfusioncorp_beta.zip
# Should show: No errors detected

# List contents of zip file
unzip -l /backup/xfusioncorp_beta.zip

# Verify backup on remote server
ssh clint@stbkp01 "ls -lh /backup/xfusioncorp_beta.zip"
# Should show file exists with size

# Compare file sizes (should match)
ls -l /backup/xfusioncorp_beta.zip
ssh clint@stbkp01 "ls -l /backup/xfusioncorp_beta.zip"

# Verify SSH works without password
ssh clint@stbkp01 "echo 'Connection successful'"
# Should not prompt for password

# Run script multiple times to test idempotency
/scripts/beta_backup.sh
/scripts/beta_backup.sh
# Should work each time without errors

# Check if steve can run the script
whoami  # Should show: steve
/scripts/beta_backup.sh
# Should execute successfully
```

## 🔑 Key Points

- **Script location:** `/scripts/beta_backup.sh` on App Server 2
- **Source directory:** `/var/www/html/beta`
- **Local backup:** `/backup/xfusioncorp_beta.zip`
- **Remote backup:** `clint@stbkp01:/backup/xfusioncorp_beta.zip`
- **No sudo in script** - All operations as steve user
- **Passwordless SSH** - Uses SSH keys for authentication
- **Servers involved:**
  - App Server 2 (steve@stapp02) - Source
  - Backup Server (clint@stbkp01) - Destination
- **Package requirement:** zip (install before running)
- **Script components:**
  - Creates zip archive
  - Stores locally in /backup
  - Copies to remote backup server via SCP
  - Error checking at each step
  - Status messages
- **Best practices implemented:**
  - Variables for easy configuration
  - Error checking after each operation
  - Informative output messages
  - Exit codes (0=success, 1=failure)
  - Removes old archive before creating new one
  - Uses -r flag for recursive zip
  - Quiet mode for scp (-q flag)
- **Automation ready:**
  - Can be added to cron for scheduled backups
  - No user interaction required
  - Suitable for CI/CD pipelines
- **Security considerations:**
  - SSH keys instead of passwords
  - No credentials hardcoded in script
  - User-level permissions (no sudo needed)
- **Idempotent:** Can run multiple times safely
- **Common issues prevented:**
  - Checks if source directory exists
  - Creates backup directory if needed
  - Removes old archive to avoid conflicts
  - Verifies archive creation before copying
  - Error handling for SCP failure

## ⚠️ Important Notes

- Install zip package BEFORE running script: `yum install -y zip`
- Setup SSH keys BEFORE first script execution
- Ensure /backup directory exists on both servers
- Script must be run as steve user (App Server 2 user)
- Do not use sudo inside the script
- Test SSH passwordless access before running script
- Backup directory (/backup) is temporary storage
- Weekly cleanup of /backup directory planned
- Remote backup provides redundancy
- Script should be owned by steve:steve
- Permissions should be 755 (rwxr-xr-x)
- Verify archive integrity after creation
- Monitor disk space on both backup locations
- Consider adding to cron for automation
- Log rotation may be needed for large-scale deployments

---

**Date Completed**: November 15, 2025  
**Challenge**: KodeKloud 100 Days Cloud DevOps  
**Day**: 10/100