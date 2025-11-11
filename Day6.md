# Day 6: Setting Up Cron Jobs for Automation

## 📋 Objective
Install cronie package, start crond service, and create a test cron job on all Nautilus app servers in Stratos Datacenter to prepare for automated day-to-day task scheduling.

## 🎯 Resolution
Successfully installed cronie package, enabled and started crond service, and configured a cron job to run every 5 minutes on all three app servers for testing automation functionality.

## 📝 Task Steps for Resolution

1. **Connect to each App Server** (App Server 1, 2, and 3)
2. **Switch to root user**
3. **Install cronie package**
4. **Start and enable crond service**
5. **Add cron job for root user**
6. **Verify cron job is scheduled**
7. **Test cron job execution** (optional)

## 💻 Commands

### For App Server 1
```bash
# Step 1: SSH into App Server 1
ssh tony@stapp01

# Step 2: Switch to root user
sudo su -

# Step 3: Install cronie package
yum install -y cronie
# For Debian/Ubuntu
apt-get install -y cron

# Step 4: Start and enable crond service
systemctl start crond
systemctl enable crond

# For Debian/Ubuntu
systemctl start cron
systemctl enable cron

# Step 5: Add cron job for root user
crontab -e
# Add this line:
# */5 * * * * echo hello > /tmp/cron_text

# Alternative: Add directly without editor
echo "*/5 * * * * echo hello > /tmp/cron_text" | crontab -

# Or append to existing crontab
(crontab -l 2>/dev/null; echo "*/5 * * * * echo hello > /tmp/cron_text") | crontab -
```

### For App Server 2
```bash
# SSH into App Server 2
ssh steve@stapp02

# Follow the same steps as App Server 1
sudo su -
yum install -y cronie
systemctl start crond
systemctl enable crond
echo "*/5 * * * * echo hello > /tmp/cron_text" | crontab -
```

### For App Server 3
```bash
# SSH into App Server 3
ssh banner@stapp03

# Follow the same steps
sudo su -
yum install -y cronie
systemctl start crond
systemctl enable crond
echo "*/5 * * * * echo hello > /tmp/cron_text" | crontab -
```

## ✅ Verification

```bash
# Verify cronie package is installed
rpm -qa | grep cronie
# or
yum list installed | grep cronie

# Check crond service status
systemctl status crond
# Should show: active (running)

# Verify service is enabled at boot
systemctl is-enabled crond
# Output: enabled

# List current user's cron jobs
crontab -l
# Output: */5 * * * * echo hello > /tmp/cron_text

# Check cron log for job execution
tail -f /var/log/cron
# Should show cron job executions every 5 minutes

# Wait 5 minutes and check the output file
cat /tmp/cron_text
# Output: hello

# Check file timestamp to see last update
ls -l /tmp/cron_text
# Should update every 5 minutes

# Monitor in real-time
watch -n 60 "ls -l /tmp/cron_text && cat /tmp/cron_text"
```

## 🔑 Key Points

- **cronie** - Modern cron implementation for RHEL/CentOS
- **crond** - The cron daemon service (cron on Debian/Ubuntu)
- **Cron syntax:** `minute hour day month weekday command`
- **Crontab management:**
  - `crontab -e` - Edit current user's crontab
  - `crontab -l` - List current user's crontab
  - `crontab -r` - Remove current user's crontab
  - `crontab -u username -e` - Edit another user's crontab (as root)
- **Special characters:**
  - `*` - Any value (every)
  - `,` - List of values (1,3,5)
  - `-` - Range of values (1-5)
  - `/` - Step values (*/5 = every 5)
  - `*/5` in minute field = Every 5 minutes
- **Cron job format breakdown:**
  ```
  */5 * * * * echo hello > /tmp/cron_text
  │   │ │ │ │ │
  │   │ │ │ │ └─── Command to execute
  │   │ │ │ └───── Day of week (0-7, 0 and 7 = Sunday)
  │   │ │ └─────── Month (1-12)
  │   │ └───────── Day of month (1-31)
  │   └─────────── Hour (0-23)
  └───────────── Minute (0-59), */5 = every 5 minutes
  ```
- **Common cron schedules:**
  - `* * * * *` - Every minute
  - `*/5 * * * *` - Every 5 minutes
  - `0 * * * *` - Every hour (at minute 0)
  - `0 0 * * *` - Daily at midnight
  - `0 2 * * *` - Daily at 2 AM
  - `0 0 * * 0` - Weekly on Sunday
  - `0 0 1 * *` - Monthly on the 1st
  - `@reboot` - Run at system startup
  - `@daily` - Run once a day (0 0 * * *)
  - `@hourly` - Run once an hour (0 * * * *)
- **Environment in cron:**
  - Limited PATH (specify full paths or set PATH in crontab)
  - No interactive shell environment
  - Output is emailed to user (if mail is configured)
- **Redirect output:**
  - `> /tmp/file` - Redirect stdout (overwrites)
  - `>> /tmp/file` - Redirect stdout (appends)
  - `2>&1` - Redirect stderr to stdout
  - `>/dev/null 2>&1` - Discard all output
- **Best practices:**
  - Use absolute paths in cron jobs
  - Test commands manually before scheduling
  - Redirect output to log files
  - Use comments in crontab for documentation
  - Be careful with `crontab -r` (removes all jobs!)
- **Logs location:**
  - RHEL/CentOS: `/var/log/cron`
  - Debian/Ubuntu: `/var/log/syslog`
- **System-wide cron directories:**
  - `/etc/cron.d/` - System cron jobs
  - `/etc/cron.daily/` - Daily scripts
  - `/etc/cron.hourly/` - Hourly scripts
  - `/etc/cron.weekly/` - Weekly scripts
  - `/etc/cron.monthly/` - Monthly scripts

## ⚠️ Important Notes

- Always test cron commands manually before scheduling
- Cron runs with minimal environment - use full paths
- Output redirection (`>`) overwrites file each time
- Use `>>` to append instead of overwrite if needed
- Service must be running and enabled for cron jobs to execute
- User-specific crontabs are stored in `/var/spool/cron/`
- Root's crontab affects system-wide operations
- Be cautious with disk space when logging output
- Time is in server's timezone
- Cron doesn't source `.bashrc` or `.bash_profile`

---

**Date Completed**: November 11, 2025  
**Challenge**: KodeKloud 100 Days Cloud DevOps  
**Day**: 6/100