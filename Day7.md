# Day 7: Configuring SSH Passwordless Authentication

## 📋 Objective
Set up password-less SSH authentication from user `thor` on jump host to all app servers in Stratos Datacenter through their respective sudo users (tony, steve, banner) for automated script execution.

## 🎯 Resolution
Successfully generated SSH key pair for thor user and distributed public key to all app servers, enabling passwordless authentication for automated operations.

## 📝 Task Steps for Resolution

1. **Login to jump host as thor user**
2. **Generate SSH key pair** (if not already exists)
3. **Copy public key to App Server 1** (user: tony)
4. **Copy public key to App Server 2** (user: steve)
5. **Copy public key to App Server 3** (user: banner)
6. **Verify passwordless SSH access to all servers**

## 💻 Commands

### Step 1: Login to Jump Host as thor
```bash
# If currently logged in as another user
su - thor
# or login directly as thor
ssh thor@jump_host
```

### Step 2: Generate SSH Key Pair (if not exists)
```bash
# Check if SSH keys already exist
ls -la ~/.ssh/

# Generate new SSH key pair (if needed)
ssh-keygen -t rsa -b 4096
# Press Enter for all prompts to accept defaults
# Or specify custom path and no passphrase:
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa -N ""

# Alternative: Use Ed25519 (more modern, secure)
ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519 -N ""

# Verify key generation
ls -la ~/.ssh/
# Should see: id_rsa and id_rsa.pub (or id_ed25519 and id_ed25519.pub)
```

### Step 3: Copy Public Key to App Server 1 (tony)
```bash
# Method 1: Using ssh-copy-id (Recommended)
ssh-copy-id tony@stapp01
# Enter tony's password when prompted

# Method 2: Manual copy
cat ~/.ssh/id_rsa.pub | ssh tony@stapp01 "mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"

# Method 3: Using SCP
scp ~/.ssh/id_rsa.pub tony@stapp01:/tmp/
ssh tony@stapp01
mkdir -p ~/.ssh
chmod 700 ~/.ssh
cat /tmp/id_rsa.pub >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
rm /tmp/id_rsa.pub
exit
```

### Step 4: Copy Public Key to App Server 2 (steve)
```bash
# Using ssh-copy-id
ssh-copy-id steve@stapp02
# Enter steve's password when prompted

# Alternative manual method
cat ~/.ssh/id_rsa.pub | ssh steve@stapp02 "mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"
```

### Step 5: Copy Public Key to App Server 3 (banner)
```bash
# Using ssh-copy-id
ssh-copy-id banner@stapp03
# Enter banner's password when prompted

# Alternative manual method
cat ~/.ssh/id_rsa.pub | ssh banner@stapp03 "mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"
```

### Step 6: Verify Passwordless SSH Access
```bash
# Test connection to App Server 1 (should NOT ask for password)
ssh tony@stapp01
hostname
exit

# Test connection to App Server 2
ssh steve@stapp02
hostname
exit

# Test connection to App Server 3
ssh banner@stapp03
hostname
exit

# One-liner test for all servers
for server in tony@stapp01 steve@stapp02 banner@stapp03; do
    echo "Testing $server..."
    ssh -o BatchMode=yes -o ConnectTimeout=5 $server "echo 'Connection successful!'" && echo "✓ $server works" || echo "✗ $server failed"
done
```

## ✅ Verification

```bash
# Verify SSH keys exist on jump host
ls -la ~/.ssh/
# Should see: id_rsa (private) and id_rsa.pub (public)

# Check private key permissions (should be 600)
ls -l ~/.ssh/id_rsa
# Output: -rw------- (600)

# Check public key content
cat ~/.ssh/id_rsa.pub
# Should show: ssh-rsa AAAAB3NzaC... thor@jump_host

# Test passwordless login to each server
ssh tony@stapp01 "echo 'App Server 1 accessible without password'"
ssh steve@stapp02 "echo 'App Server 2 accessible without password'"
ssh banner@stapp03 "echo 'App Server 3 accessible without password'"

# Verify authorized_keys on remote servers (login to each)
ssh tony@stapp01
cat ~/.ssh/authorized_keys
# Should contain thor's public key

# Check authorized_keys permissions
ls -l ~/.ssh/authorized_keys
# Output: -rw------- (600)

# Check .ssh directory permissions
ls -ld ~/.ssh
# Output: drwx------ (700)
```

## 🔑 Key Points

- **SSH keys** - More secure than passwords for authentication
- **Two types of keys:**
  - **Private key** (`id_rsa`) - Keep secret, never share! (like a password)
  - **Public key** (`id_rsa.pub`) - Safe to share, add to authorized_keys
- **Key algorithms:**
  - **RSA** - Traditional, widely supported (2048-4096 bits recommended)
  - **Ed25519** - Modern, faster, more secure (fixed 256 bits)
  - **ECDSA** - Elliptic curve, good but Ed25519 preferred
- **ssh-keygen options:**
  - `-t` - Type of key (rsa, ed25519, ecdsa)
  - `-b` - Bits (for RSA: 2048, 4096)
  - `-f` - Output file path
  - `-N` - Passphrase (empty "" for no passphrase)
  - `-C` - Comment (usually email or description)
- **ssh-copy-id** - Easiest way to copy public key to remote servers
- **Manual method** - Useful when ssh-copy-id not available
- **File permissions are critical:**
  - `~/.ssh/` directory: **700** (rwx------)
  - `~/.ssh/id_rsa` (private key): **600** (rw-------)
  - `~/.ssh/id_rsa.pub` (public key): **644** (rw-r--r--)
  - `~/.ssh/authorized_keys`: **600** (rw-------)
- **authorized_keys file** - Contains public keys allowed to authenticate
- **Passwordless = No passphrase** - Key has no password protection
- **Security considerations:**
  - Private key should NEVER leave the source machine
  - Protect private key with proper permissions (600)
  - Consider using passphrase for added security
  - Use ssh-agent to avoid typing passphrase repeatedly
- **Common locations:**
  - Keys: `~/.ssh/id_rsa` and `~/.ssh/id_rsa.pub`
  - Authorized keys: `~/.ssh/authorized_keys`
  - Known hosts: `~/.ssh/known_hosts`
  - Config: `~/.ssh/config`
- **Troubleshooting:**
  - Check permissions on .ssh directory and files
  - Verify public key is in authorized_keys
  - Check SSH server config (PermitRootLogin, PubkeyAuthentication)
  - Use `ssh -v` for verbose debugging
  - Check `/var/log/auth.log` or `/var/log/secure` for errors
- **Best practices:**
  - Generate different keys for different purposes
  - Use strong key sizes (4096 for RSA)
  - Rotate keys periodically
  - Remove old/unused keys from authorized_keys
  - Use SSH agent forwarding carefully (security risk)
  - Document which keys have access where

## ⚠️ Important Notes

- Never share or expose private keys (id_rsa)
- Public keys (id_rsa.pub) are safe to share
- Incorrect permissions will cause SSH to refuse the key
- ssh-copy-id automatically sets correct permissions
- Manual copying requires setting permissions manually
- Test connections after setup to ensure it works
- This enables automation but also increases security responsibility
- Jump host becomes a critical security point - protect it well!
- Consider using SSH certificates for enterprise environments
- Regular audit of authorized_keys is important for security

---

**Date Completed**: November 12, 2025  
**Challenge**: KodeKloud 100 Days Cloud DevOps  
**Day**: 7/100