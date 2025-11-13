# Day 8: Installing Ansible on Jump Host

## 📋 Objective
Install Ansible version 4.9.0 on Jump Host using pip3, ensuring the Ansible binary is globally accessible for all users on the system for testing automation and configuration management.

## 🎯 Resolution
Successfully installed Ansible 4.9.0 globally using pip3, making the ansible command available system-wide for all users to execute automation tasks.

## 📝 Task Steps for Resolution

1. **Login to Jump Host**
2. **Switch to root user** (for global installation)
3. **Check if pip3 is installed**
4. **Install pip3** (if needed)
5. **Install Ansible 4.9.0 using pip3**
6. **Verify Ansible installation**
7. **Test Ansible command as different users**

## 💻 Commands

```bash
# Step 1: Login to Jump Host
# (Usually already on jump host)

# Step 2: Switch to root user
sudo su -

# Step 3: Check if pip3 is installed
pip3 --version
# or
python3 -m pip --version

# Step 4: Install pip3 if not present
# For RHEL/CentOS
yum install -y python3-pip
# For Debian/Ubuntu
apt-get update
apt-get install -y python3-pip

# Step 5: Install Ansible 4.9.0 globally using pip3
pip3 install ansible==4.9.0

# Alternative: Specify system-wide installation explicitly
python3 -m pip install ansible==4.9.0

# Step 6: Verify installation location
which ansible
# Expected output: /usr/local/bin/ansible

# Check Ansible version
ansible --version

# Verify all Ansible binaries are available
which ansible-playbook
which ansible-galaxy
which ansible-vault

# Step 7: Check PATH includes /usr/local/bin
echo $PATH
# Should include: /usr/local/bin

# Test as different user
su - thor
ansible --version
exit
```

## ✅ Verification

```bash
# Verify Ansible version
ansible --version
# Should show: ansible [core 2.11.x]
# and package location

# Check installed version specifically
pip3 show ansible
# Look for: Version: 4.9.0

# Verify binary location
which ansible
# Output: /usr/local/bin/ansible

# Check all Ansible commands are available
ansible --version
ansible-playbook --version
ansible-galaxy --version
ansible-vault --version
ansible-inventory --version

# Test as root
ansible --version

# Test as regular user (e.g., thor)
su - thor
ansible --version
exit

# Verify PATH for all users
echo $PATH | grep /usr/local/bin
# Should be present

# Check if binary has proper permissions
ls -l /usr/local/bin/ansible
# Should be executable by all: -rwxr-xr-x

# List all Ansible-related packages
pip3 list | grep ansible
```

## 🔑 Key Points

- **Ansible** - Agentless configuration management and automation tool
- **Version 4.9.0** - Specific version required (Ansible package version)
- **pip3** - Python package installer for Python 3
- **Global installation** - Using pip3 installs to /usr/local/bin by default
- **System-wide access** - All users can run Ansible commands
- **Ansible components installed:**
  - `ansible` - Main command for ad-hoc tasks
  - `ansible-playbook` - Run playbooks
  - `ansible-galaxy` - Manage roles and collections
  - `ansible-vault` - Encrypt sensitive data
  - `ansible-inventory` - View inventory
  - `ansible-config` - View configuration
- **Ansible versioning:**
  - Ansible 4.9.0 (package version) includes ansible-core 2.11.x
  - Two version numbers: package version and core version
- **Why pip3 over package managers:**
  - Latest versions available
  - Specific version control
  - Consistent across distributions
  - Direct from source (PyPI)
- **Installation locations:**
  - Binaries: `/usr/local/bin/`
  - Libraries: `/usr/local/lib/python3.x/site-packages/`
- **PATH considerations:**
  - `/usr/local/bin` typically in default PATH
  - If not, users need to add it to PATH
  - Global installation ensures all users have access
- **Dependencies:**
  - Python 3.6+ required
  - pip3 automatically installs dependencies
  - Includes: PyYAML, Jinja2, cryptography, etc.
- **Why Ansible for this team:**
  - Simple setup (no agents on managed nodes)
  - Minimal prerequisites (just SSH and Python)
  - Easy to learn (YAML-based playbooks)
  - Powerful automation capabilities
  - Agentless architecture
- **Jump host as Ansible controller:**
  - Central control point
  - Already has SSH access to all servers
  - Secure and auditable
  - Standard DevOps pattern

## ⚠️ Important Notes

- Installing with pip3 as root makes it available globally
- Do NOT use `--user` flag for global installation
- Ensure /usr/local/bin is in all users' PATH
- Specific version (4.9.0) must be specified with `==`
- Ansible requires Python 3 on controller (Python 2.7+ on managed nodes)
- pip3 may be called `pip` on some systems if Python 3 is default
- Use `pip3 install --upgrade pip` if pip3 is outdated
- Ansible is agentless - no installation needed on managed nodes
- Only SSH and Python required on target servers
- Regular updates recommended for security patches
- Test connectivity to managed nodes after installation
- Consider using virtual environments for development (but not this case)
- Global installation means easier for all team members

---

**Date Completed**: November 13, 2025  
**Challenge**: KodeKloud 100 Days Cloud DevOps  
**Day**: 8/100