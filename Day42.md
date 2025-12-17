# Day 42: Docker Custom Network with Subnet Configuration
**100 Days Cloud DevOps Challenge – KodeKloud**

---

## Objective

Create a custom Docker network with specific network configuration for application deployment requirements.

**Requirements:**
1. Work on Application Server 1 (App Server 1)
2. Network Name: `beta`
3. Driver: `bridge`
4. Subnet: `10.10.1.0/24`
5. IP Range: `10.10.1.0/24`

---

## Understanding Docker Networks

**Docker Networks** provide isolated communication channels for containers. They control how containers communicate with each other and the outside world.

### What is Docker Networking?

Docker networking allows containers to:
- Communicate with each other
- Access external networks
- Isolate applications
- Control traffic flow
- Assign custom IP addresses

### Docker Network Architecture:

```
Host Machine
│
├── Docker Network: bridge (default)
│   ├── Container 1: 172.17.0.2
│   ├── Container 2: 172.17.0.3
│   └── Container 3: 172.17.0.4
│
├── Docker Network: beta (custom)
│   ├── Subnet: 10.10.1.0/24
│   ├── IP Range: 10.10.1.0/24
│   └── Available IPs: 10.10.1.1 - 10.10.1.254
│
└── Docker Network: host
    └── Shares host networking
```

### Why Custom Docker Networks?

- **IP Management** - Control IP address allocation
- **Network Isolation** - Separate application environments
- **Custom Subnets** - Match organizational IP schemes
- **Security** - Isolate sensitive workloads
- **Multi-tenancy** - Separate customer environments
- **Service Discovery** - Built-in DNS resolution

---

## Understanding Docker Network Drivers

### Available Network Drivers:

| Driver | Description | Use Case |
|--------|-------------|----------|
| **bridge** | Default driver, isolated network | Single host, container-to-container |
| **host** | Removes network isolation | High performance, no isolation |
| **none** | Disables networking | Maximum isolation |
| **overlay** | Multi-host networking | Docker Swarm, multi-host |
| **macvlan** | Assigns MAC addresses | Legacy apps, VLAN integration |

### Bridge Driver (Our Task):

```
Bridge Network
│
├── Virtual Network Interface
├── Internal DHCP
├── NAT to host
├── Container isolation
└── Inter-container communication
```

**Features:**
- Isolated network segment
- Containers get private IPs
- NAT for external access
- DNS resolution between containers
- Port mapping to host

**Bridge vs Default Bridge:**

| Aspect | Default Bridge | Custom Bridge |
|--------|---------------|---------------|
| **Name** | `bridge` | User-defined (e.g., `beta`) |
| **DNS** | Manual linking | Automatic DNS |
| **Isolation** | Shared | Dedicated |
| **Configuration** | Limited | Full control |
| **Best For** | Quick testing | Production use |

---

## Understanding Subnet and IP Range

### Subnet Configuration:

**Subnet: 10.10.1.0/24**
```
10.10.1.0/24 means:
├── Network Address: 10.10.1.0
├── Subnet Mask: 255.255.255.0
├── CIDR Notation: /24 (24 bits for network)
├── Usable IPs: 10.10.1.1 - 10.10.1.254
├── Broadcast: 10.10.1.255
└── Total Hosts: 254 usable addresses
```

**CIDR Breakdown:**
```
/24 = 255.255.255.0
24 bits for network, 8 bits for hosts
2^8 = 256 addresses (254 usable)
```

### IP Range vs Subnet:

```
Subnet: 10.10.1.0/24
└── Defines entire network scope (10.10.1.0 - 10.10.1.255)

IP Range: 10.10.1.0/24
└── Defines which IPs Docker can assign to containers
```

**In our task:** Both are the same, meaning Docker can use the entire subnet for container IPs.

### IP Address Allocation:

```
10.10.1.0/24 with IP Range 10.10.1.0/24:

10.10.1.0       → Network address (reserved)
10.10.1.1       → Gateway (Docker bridge)
10.10.1.2       → First container
10.10.1.3       → Second container
...
10.10.1.254     → Last usable IP
10.10.1.255     → Broadcast (reserved)
```

---

## Understanding the Scenario

### DevOps Team Requirement:

```
1. Multiple docker environments needed
2. Different applications require network isolation
3. Custom IP addressing for better management
4. Preparation for future container deployments
5. Network named 'beta' for beta testing environment
```

### Network Planning:

```
Application Server 1 (stapp01)
│
├── Default Docker Network: bridge
│   └── 172.17.0.0/16 (Docker default)
│
├── Custom Network: beta (New)
│   ├── Driver: bridge
│   ├── Subnet: 10.10.1.0/24
│   ├── IP Range: 10.10.1.0/24
│   └── Purpose: Beta environment containers
│
└── Future Networks:
    ├── alpha (development)
    ├── gamma (staging)
    └── prod (production)
```

### Why These Settings?

**Network Name: beta**
- Identifies beta/testing environment
- Easy to reference in commands
- Follows naming conventions

**Driver: bridge**
- Standard container networking
- Good isolation
- Container-to-container communication
- NAT for external access

**Subnet: 10.10.1.0/24**
- Private IP range (RFC 1918)
- 254 available IPs
- Matches organizational standards
- Easy to remember and manage

**IP Range: 10.10.1.0/24**
- Docker can use all available IPs
- Maximum flexibility
- No IP restrictions

---

## Infrastructure Overview

### Application Servers:
| Server | User | Password | IP |
|--------|------|----------|-----|
| **stapp01** | **tony** | **Ir0nM@n** | **172.16.238.10** |
| stapp02 | steve | Am3ric@ | 172.16.238.11 |
| stapp03 | banner | BigGr33n | 172.16.238.12 |

### Task Details:
| Item | Value |
|------|-------|
| Server | Application Server 1 (stapp01) |
| User | tony |
| Network Name | `beta` |
| Driver | `bridge` |
| Subnet | `10.10.1.0/24` |
| IP Range | `10.10.1.0/24` |
| Gateway | 10.10.1.1 (auto-assigned) |

---

## Understanding the Task

### What We're Creating:

```
Application Server 1 (stapp01)
│
└── Docker Network: beta
    ├── Type: Custom bridge network
    ├── Subnet: 10.10.1.0/24
    ├── Gateway: 10.10.1.1
    ├── IP Range: 10.10.1.0/24
    ├── Usable IPs: 10.10.1.2 - 10.10.1.254
    └── Containers: Will get IPs from this range
```

### Network Creation Flow:

```
1. SSH to App Server 1
2. Switch to root
3. Check existing networks
4. Create network with specifications:
   - Name: beta
   - Driver: bridge
   - Subnet: 10.10.1.0/24
   - IP Range: 10.10.1.0/24
5. Verify network created
6. Inspect network configuration
7. Test network (optional)
```

---

## Step-by-Step Implementation

### Step 1: SSH into Application Server 1
```bash
ssh tony@stapp01
```

**Expected output:**
```
The authenticity of host 'stapp01 (172.16.238.10)' can't be established.
ED25519 key fingerprint is SHA256:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'stapp01' (ED25519) to the list of known hosts.
tony@stapp01's password:
```

**Enter password:** `Ir0nM@n`

```
[tony@stapp01 ~]$
```

### Step 2: Switch to Root User
```bash
sudo su -
```

**Expected output:**
```
We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

[sudo] password for tony:
```

**Enter password:** `Ir0nM@n`

```
[root@stapp01 ~]#
```

### Step 3: List Existing Docker Networks
```bash
docker network ls
```

**Expected output:**
```
NETWORK ID     NAME      DRIVER    SCOPE
a1b2c3d4e5f6   bridge    bridge    local
f6e5d4c3b2a1   host      host      local
b2a1f6e5d4c3   none      null      local
```

**Default networks:**
- `bridge` - Default bridge network
- `host` - Host network mode
- `none` - No networking

### Step 4: Check if 'beta' Network Already Exists
```bash
docker network ls | grep beta
```

**Should return empty** (no output means beta doesn't exist yet)

### Step 5: Create Docker Network with Specifications
```bash
docker network create \
  --driver bridge \
  --subnet 10.10.1.0/24 \
  --ip-range 10.10.1.0/24 \
  beta
```

**Expected output:**
```
c7d8e9f0a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6e7f8a9b0c1d2e3f4e5f6a7b8
```

**This is the network ID (yours will differ)**

**Command breakdown:**
- `docker network create` - Create new network
- `--driver bridge` - Use bridge driver
- `--subnet 10.10.1.0/24` - Define subnet
- `--ip-range 10.10.1.0/24` - Define IP allocation range
- `beta` - Network name

**Alternative single-line command:**
```bash
docker network create --driver bridge --subnet 10.10.1.0/24 --ip-range 10.10.1.0/24 beta
```

### Step 6: Verify Network Created
```bash
docker network ls
```

**Expected output:**
```
NETWORK ID     NAME      DRIVER    SCOPE
a1b2c3d4e5f6   bridge    bridge    local
c7d8e9f0a1b2   beta      bridge    local
f6e5d4c3b2a1   host      host      local
b2a1f6e5d4c3   none      null      local
```

**✅ Network 'beta' is now listed!**

**Filter for beta network:**
```bash
docker network ls | grep beta
```

**Expected output:**
```
c7d8e9f0a1b2   beta      bridge    local
```

### Step 7: Inspect Network Configuration
```bash
docker network inspect beta
```

**Expected output:**
```json
[
    {
        "Name": "beta",
        "Id": "c7d8e9f0a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6e7f8a9b0c1d2e3f4e5f6a7b8",
        "Created": "2025-12-17T10:30:45.123456789Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "10.10.1.0/24",
                    "IPRange": "10.10.1.0/24",
                    "Gateway": "10.10.1.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {},
        "Options": {},
        "Labels": {}
    }
]
```

**Verify the key fields:**
- ✅ "Name": "beta"
- ✅ "Driver": "bridge"
- ✅ "Subnet": "10.10.1.0/24"
- ✅ "IPRange": "10.10.1.0/24"
- ✅ "Gateway": "10.10.1.1" (auto-assigned)

### Step 8: Check Specific Network Details
```bash
docker network inspect beta --format '{{.Name}}'
```

**Expected output:**
```
beta
```

**Check driver:**
```bash
docker network inspect beta --format '{{.Driver}}'
```

**Expected output:**
```
bridge
```

**Check subnet:**
```bash
docker network inspect beta --format '{{range .IPAM.Config}}{{.Subnet}}{{end}}'
```

**Expected output:**
```
10.10.1.0/24
```

**Check IP range:**
```bash
docker network inspect beta --format '{{range .IPAM.Config}}{{.IPRange}}{{end}}'
```

**Expected output:**
```
10.10.1.0/24
```

**Check gateway:**
```bash
docker network inspect beta --format '{{range .IPAM.Config}}{{.Gateway}}{{end}}'
```

**Expected output:**
```
10.10.1.1
```

### Step 9: View Network Configuration Summary
```bash
docker network inspect beta | grep -E "Name|Driver|Subnet|IPRange|Gateway"
```

**Expected output:**
```
        "Name": "beta",
        "Driver": "bridge",
                    "Subnet": "10.10.1.0/24",
                    "IPRange": "10.10.1.0/24",
                    "Gateway": "10.10.1.1"
```

**Perfect! All specifications match requirements ✅**

### Step 10: Test Network with Container (Optional)
```bash
docker run -d --name test_container --network beta nginx:alpine
```

**Expected output:**
```
d4e5f6a7b8c9d0e1f2a3b4c5d6e7f8a9b0c1d2e3f4e5f6a7b8c9d0e1f2a3b4c5
```

**Check container IP:**
```bash
docker inspect test_container --format '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}'
```

**Expected output:**
```
10.10.1.2
```

**✅ Container got IP from our 10.10.1.0/24 range!**

**Check container network details:**
```bash
docker inspect test_container | grep -A 20 "Networks"
```

**Expected output (partial):**
```json
"Networks": {
    "beta": {
        "IPAMConfig": null,
        "Links": null,
        "Aliases": [
            "d4e5f6a7b8c9"
        ],
        "NetworkID": "c7d8e9f0a1b2...",
        "EndpointID": "a1b2c3d4e5f6...",
        "Gateway": "10.10.1.1",
        "IPAddress": "10.10.1.2",
        "IPPrefixLen": 24,
        "IPv6Gateway": "",
        "GlobalIPv6Address": "",
        "GlobalIPv6PrefixLen": 0,
        "MacAddress": "02:42:0a:0a:01:02",
        "DriverOpts": null
    }
}
```

**Clean up test container:**
```bash
docker rm -f test_container
```

### Step 11: Verify Network Still Exists
```bash
docker network ls | grep beta
```

**Expected output:**
```
c7d8e9f0a1b2   beta      bridge    local
```

**✅ Network persists after removing containers**

### Step 12: Check Network Interface on Host (Optional)
```bash
ip addr show | grep -A 5 "br-"
```

**Expected output (partial):**
```
br-c7d8e9f0a1b2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:d8:a3:f1:2c brd ff:ff:ff:ff:ff:ff
    inet 10.10.1.1/24 brd 10.10.1.255 scope global br-c7d8e9f0a1b2
       valid_lft forever preferred_lft forever
```

**Shows bridge interface with gateway IP 10.10.1.1**

---

## Complete Command Summary

### Quick Network Creation:
```bash
# SSH to server
ssh tony@stapp01

# Switch to root
sudo su -

# Create network
docker network create --driver bridge --subnet 10.10.1.0/24 --ip-range 10.10.1.0/24 beta

# Verify
docker network ls | grep beta
docker network inspect beta
```

### Detailed Workflow:
```bash
# SSH and authenticate
ssh tony@stapp01
sudo su -

# Check existing networks
docker network ls

# Create beta network with specifications
docker network create \
  --driver bridge \
  --subnet 10.10.1.0/24 \
  --ip-range 10.10.1.0/24 \
  beta

# Verify creation
docker network ls
docker network ls | grep beta

# Inspect configuration
docker network inspect beta

# Verify specific settings
docker network inspect beta --format '{{.Driver}}'
docker network inspect beta --format '{{range .IPAM.Config}}{{.Subnet}}{{end}}'
docker network inspect beta --format '{{range .IPAM.Config}}{{.IPRange}}{{end}}'

# Optional: Test with container
docker run -d --name test --network beta nginx:alpine
docker inspect test --format '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}'
docker rm -f test
```

---

## Understanding Docker Network Commands

### Network Management Commands:

**1. List Networks:**
```bash
docker network ls
docker network ls --filter driver=bridge
docker network ls --filter name=beta
```

**2. Create Network:**
```bash
# Basic
docker network create my_network

# With driver
docker network create --driver bridge my_network

# With subnet
docker network create --subnet 192.168.1.0/24 my_network

# With IP range
docker network create --subnet 192.168.1.0/24 --ip-range 192.168.1.0/25 my_network

# With gateway
docker network create --subnet 192.168.1.0/24 --gateway 192.168.1.254 my_network

# Complete example
docker network create \
  --driver bridge \
  --subnet 10.10.1.0/24 \
  --ip-range 10.10.1.0/24 \
  --gateway 10.10.1.1 \
  beta
```

**3. Inspect Network:**
```bash
docker network inspect beta
docker network inspect beta --format '{{json .IPAM.Config}}'
docker network inspect beta --format '{{.Driver}}'
```

**4. Remove Network:**
```bash
docker network rm beta
docker network rm network1 network2 network3  # Multiple
docker network prune  # Remove unused networks
```

**5. Connect/Disconnect Containers:**
```bash
# Connect container to network
docker network connect beta container_name

# Disconnect
docker network disconnect beta container_name

# Connect with specific IP
docker network connect --ip 10.10.1.50 beta container_name
```

---

## Advanced Network Configuration

### Creating Networks with Additional Options:

**1. Network with Custom Gateway:**
```bash
docker network create \
  --driver bridge \
  --subnet 10.10.1.0/24 \
  --gateway 10.10.1.254 \
  beta
```

**2. Network with Multiple Subnets:**
```bash
docker network create \
  --driver bridge \
  --subnet 10.10.1.0/24 \
  --subnet 10.10.2.0/24 \
  beta
```

**3. Network with Restricted IP Range:**
```bash
docker network create \
  --driver bridge \
  --subnet 10.10.1.0/24 \
  --ip-range 10.10.1.128/25 \
  beta
# Containers only get IPs from 10.10.1.128 - 10.10.1.254
```

**4. Network with Labels:**
```bash
docker network create \
  --driver bridge \
  --subnet 10.10.1.0/24 \
  --label environment=beta \
  --label team=devops \
  beta
```

**5. Internal Network (No External Access):**
```bash
docker network create \
  --driver bridge \
  --subnet 10.10.1.0/24 \
  --internal \
  beta
```

**6. Network with IPv6:**
```bash
docker network create \
  --driver bridge \
  --subnet 10.10.1.0/24 \
  --ipv6 \
  --subnet fd00::/64 \
  beta
```

---

## Understanding Network Isolation

### Network Segmentation:

```
Host Machine
│
├── Network: frontend (10.10.1.0/24)
│   ├── Web Server: 10.10.1.2
│   └── Load Balancer: 10.10.1.3
│
├── Network: backend (10.10.2.0/24)
│   ├── App Server: 10.10.2.2
│   └── Cache: 10.10.2.3
│
└── Network: database (10.10.3.0/24)
    ├── Primary DB: 10.10.3.2
    └── Replica DB: 10.10.3.3
```

**Containers in different networks cannot communicate by default!**

### Multi-Network Containers:

```bash
# Create networks
docker network create --subnet 10.10.1.0/24 frontend
docker network create --subnet 10.10.2.0/24 backend

# Container connected to both networks
docker run -d --name app \
  --network frontend \
  nginx

docker network connect backend app

# Now 'app' can communicate with both networks
```

---

## Troubleshooting

### Issue 1: Subnet Overlap

**Problem:**
```
Error response from daemon: Pool overlaps with other one on this address space
```

**Solution:**
```bash
# Check existing networks
docker network ls
docker network inspect bridge

# Find conflicting subnet
docker network inspect $(docker network ls -q)

# Use different subnet
docker network create --subnet 10.10.2.0/24 --ip-range 10.10.2.0/24 beta
```

### Issue 2: Network Already Exists

**Problem:**
```
Error response from daemon: network with name beta already exists
```

**Solution:**
```bash
# Check if network exists
docker network ls | grep beta

# Option 1: Use existing network
docker network inspect beta

# Option 2: Remove and recreate
docker network rm beta
docker network create --driver bridge --subnet 10.10.1.0/24 --ip-range 10.10.1.0/24 beta
```

### Issue 3: Cannot Remove Network

**Problem:**
```
Error response from daemon: network beta has active endpoints
```

**Solution:**
```bash
# Find containers using the network
docker network inspect beta --format '{{range .Containers}}{{.Name}} {{end}}'

# Disconnect containers
docker network disconnect beta container_name

# Or remove containers
docker rm -f container_name

# Then remove network
docker network rm beta
```

### Issue 4: Invalid Subnet Format

**Problem:**
```
Error response from daemon: invalid CIDR address: 10.10.1.0
```

**Solution:**
```bash
# ❌ Wrong format
docker network create --subnet 10.10.1.0 beta

# ✅ Correct format (include /24)
docker network create --subnet 10.10.1.0/24 beta
```

### Issue 5: IP Range Outside Subnet

**Problem:**
```
Error response from daemon: ip-range must be within subnet
```

**Solution:**
```bash
# ❌ IP range outside subnet
docker network create --subnet 10.10.1.0/24 --ip-range 10.10.2.0/24 beta

# ✅ IP range within subnet
docker network create --subnet 10.10.1.0/24 --ip-range 10.10.1.0/24 beta

# ✅ Partial range also valid
docker network create --subnet 10.10.1.0/24 --ip-range 10.10.1.128/25 beta
```

### Issue 6: Permission Denied

**Problem:**
```
Got permission denied while trying to connect to the Docker daemon socket
```

**Solution:**
```bash
# Use sudo
sudo docker network create --subnet 10.10.1.0/24 beta

# Or switch to root
sudo su -
docker network create --subnet 10.10.1.0/24 --ip-range 10.10.1.0/24 beta
```

### Issue 7: Gateway Conflict

**Problem:**
Gateway IP conflicts with existing network

**Solution:**
```bash
# Check all network gateways
docker network inspect $(docker network ls -q) | grep Gateway

# Specify different gateway
docker network create \
  --subnet 10.10.1.0/24 \
  --gateway 10.10.1.254 \
  beta
```

---

## Best Practices

### 1. Use Descriptive Network Names
```bash
# ✅ Good names
docker network create frontend
docker network create api-backend
docker network create prod-database

# ❌ Vague names
docker network create net1
docker network create test
```

### 2. Document Network Purpose
```bash
# Add labels to networks
docker network create \
  --label purpose="beta testing environment" \
  --label team="devops" \
  --label environment="staging" \
  --subnet 10.10.1.0/24 \
  beta
```

### 3. Use Private IP Ranges
```bash
# ✅ RFC 1918 private ranges
10.0.0.0/8
172.16.0.0/12
192.168.0.0/16

# Example
docker network create --subnet 10.10.1.0/24 beta
docker network create --subnet 172.20.0.0/16 frontend
docker network create --subnet 192.168.100.0/24 backend
```

### 4. Plan IP Ranges
```bash
# Reserve ranges for different environments
# Development: 10.10.0.0/16
# Staging: 10.20.0.0/16
# Production: 10.30.0.0/16

docker network create --subnet 10.10.1.0/24 dev-beta
docker network create --subnet 10.20.1.0/24 staging-beta
docker network create --subnet 10.30.1.0/24 prod-beta
```

### 5. Use Appropriate Subnet Sizes
```bash
# Small environment (up to 14 hosts)
docker network create --subnet 10.10.1.0/28 beta  # /28 = 16 IPs

# Medium environment (up to 62 hosts)
docker network create --subnet 10.10.1.0/26 beta  # /26 = 64 IPs

# Large environment (up to 254 hosts)
docker network create --subnet 10.10.1.0/24 beta  # /24 = 256 IPs

# Very large environment (up to 1022 hosts)
docker network create --subnet 10.10.0.0/22 beta  # /22 = 1024 IPs
```

### 6. Restrict IP Range When Needed
```bash
# Reserve IPs for static assignment
docker network create \
  --subnet 10.10.1.0/24 \
  --ip-range 10.10.1.128/25 \
  beta
# Docker uses 10.10.1.128-254
# Manual use: 10.10.1.2-127
```

### 7. Clean Up Unused Networks
```bash
# List networks
docker network ls

# Remove unused networks
docker network prune

# Remove specific network
docker network rm old_network
```

---

## Real-World Scenarios

### Scenario 1: Multi-Tier Application
```bash
# Create networks for each tier
docker network create --subnet 10.10.1.0/24 frontend
docker network create --subnet 10.10.2.0/24 backend
docker network create --subnet 10.10.3.0/24 database

# Deploy containers
docker run -d --name web --network frontend nginx
docker run -d --name api --network backend node:alpine
docker run -d --name db --network database postgres

# Connect API to both frontend and backend
docker network connect frontend api
```

### Scenario 2: Microservices Architecture
```bash
# Create service networks
docker network create --subnet 10.10.10.0/24 auth-service
docker network create --subnet 10.10.20.0/24 payment-service
docker network create --subnet 10.10.30.0/24 notification-service
docker network create --subnet 10.10.40.0/24 api-gateway

# API gateway connected to all services
docker run -d --name gateway \
  --network api-gateway \
  api-gateway:latest

docker network connect auth-service gateway
docker network connect payment-service gateway
docker network connect notification-service gateway
```

### Scenario 3: Development Environment
```bash
# Create dev network with descriptive labels
docker network create \
  --driver bridge \
  --subnet 172.20.0.0/16 \
  --ip-range 172.20.1.0/24 \
  --label environment=development \
  --label team=engineering \
  dev-network

# Run development stack
docker run -d --name dev-db --network dev-network postgres
docker run -d --name dev-redis --network dev-network redis
docker run -d --name dev-app --network dev-network myapp:dev
```

### Scenario 4: Testing Isolated Environments
```bash
# Create separate networks for each test suite
docker network create --subnet 10.100.1.0/24 test-suite-1
docker network create --subnet 10.100.2.0/24 test-suite-2
docker network create --subnet 10.100.3.0/24 test-suite-3

# Run tests in isolation
docker run --rm --network test-suite-1 test-runner:latest npm test
docker run --rm --network test-suite-2 test-runner:latest pytest
docker run --rm --network test-suite-3 test-runner:latest go test
```

### Scenario 5: Security Zones
```bash
# DMZ network for public-facing services
docker network create --subnet 10.1.0.0/24 dmz

# Internal network for backend services
docker network create --subnet 10.2.0.0/24 --internal internal

# Secure network for sensitive data
docker network create --subnet 10.3.0.0/24 --internal secure

# Deploy with appropriate network access
docker run -d --name web --network dmz nginx
docker run -d --name app --network internal app:latest
docker run -d --name secrets --network secure vault:latest

# Web server can access app
docker network connect internal web
```

---

## Key Commands Reference

| Command | Description |
|---------|-------------|
| `docker network ls` | List all networks |
| `docker network create NAME` | Create network with defaults |
| `docker network create --driver DRIVER NAME` | Create with specific driver |
| `docker network create --subnet CIDR NAME` | Create with subnet |
| `docker network create --ip-range CIDR NAME` | Create with IP range |
| `docker network inspect NAME` | View network details |
| `docker network rm NAME` | Remove network |
| `docker network prune` | Remove unused networks |
| `docker network connect NET CONTAINER` | Connect container to network |
| `docker network disconnect NET CONTAINER` | Disconnect container |

---

## Network Configuration Reference

### Common Subnet Sizes:

| CIDR | Subnet Mask | Usable IPs | Use Case |
|------|-------------|------------|----------|
| /30 | 255.255.255.252 | 2 | Point-to-point |
| /29 | 255.255.255.248 | 6 | Very small |
| /28 | 255.255.255.240 | 14 | Small environment |
| /27 | 255.255.255.224 | 30 | Small team |
| /26 | 255.255.255.192 | 62 | Medium team |
| /25 | 255.255.255.128 | 126 | Large team |
| /24 | 255.255.255.0 | 254 | **Standard (our task)** |
| /23 | 255.255.254.0 | 510 | Large deployment |
| /22 | 255.255.252.0 | 1022 | Very large |
| /16 | 255.255.0.0 | 65534 | Enterprise |

### Private IP Ranges (RFC 1918):

```
10.0.0.0 - 10.255.255.255     (10/8 prefix)
172.16.0.0 - 172.31.255.255   (172.16/12 prefix)
192.168.0.0 - 192.168.255.255 (192.168/16 prefix)
```

---

## Completion Checklist

- [ ] SSH into Application Server 1 (stapp01) as tony
- [ ] Switched to root user
- [ ] Listed existing Docker networks
- [ ] Verified 'beta' network doesn't exist
- [ ] Created network with name 'beta'
- [ ] Specified driver as 'bridge'
- [ ] Set subnet to 10.10.1.0/24
- [ ] Set IP range to 10.10.1.0/24
- [ ] Verified network created successfully
- [ ] Inspected network configuration
- [ ] Confirmed driver is bridge
- [ ] Confirmed subnet is 10.10.1.0/24
- [ ] Confirmed IP range is 10.10.1.0/24
- [ ] Verified gateway auto-assigned (10.10.1.1)
- [ ] All requirements met ✅

---

## Completion Details

- **Completion Date:** December 17, 2025
- **Day:** 42 / 100
- **Challenge:** KodeKloud 100 Days Cloud DevOps
- **Topic:** Docker Custom Network with Subnet Configuration
- **Server:** Application Server 1 (stapp01)
- **User:** tony
- **Network Name:** `beta`
- **Driver:** bridge
- **Subnet:** 10.10.1.0/24
- **IP Range:** 10.10.1.0/24
- **Gateway:** 10.10.1.1 (auto-assigned)
- **Usable IPs:** 10.10.1.2 - 10.10.1.254 (254 addresses)
- **Key Skill:** Docker network creation with custom IP addressing
- **Status:** ✅ Successfully Completed

---

## Summary

This task demonstrated **creating custom Docker networks with specific subnet configuration**:

✅ **Created custom network** - Named 'beta' for beta environment
✅ **Used bridge driver** - Standard container networking
✅ **Configured subnet** - 10.10.1.0/24 (254 usable IPs)
✅ **Set IP range** - 10.10.1.0/24 (full subnet available)
✅ **Automatic gateway** - 10.10.1.1 assigned by Docker

**Key Insight:** Custom Docker networks provide **better control over container networking** compared to the default bridge:
- **DNS Resolution** - Containers can reach each other by name
- **Network Isolation** - Separate environments don't interfere
- **IP Management** - Control exact IP addressing scheme
- **Security** - Isolate different applications/tiers
- **Flexibility** - Multiple networks per container

**The Network Creation Process:**
```
Define Requirements → Create Network → Verify Configuration → Deploy Containers
```

**Why Custom Networks Matter:**
```
Default Bridge Network:
- Shared by all containers
- No automatic DNS
- Limited control
- Not production-ready

Custom Bridge Network (beta):
- Dedicated network segment
- Automatic DNS resolution
- Full IP control
- Production-ready
```

**Network Configuration Breakdown:**
```
Network Name: beta
├── Driver: bridge (standard container networking)
├── Subnet: 10.10.1.0/24
│   ├── Network: 10.10.1.0 (reserved)
│   ├── Gateway: 10.10.1.1 (Docker bridge)
│   ├── Container IPs: 10.10.1.2 - 10.10.1.254
│   └── Broadcast: 10.10.1.255 (reserved)
└── IP Range: 10.10.1.0/24 (Docker can use all IPs)
```

**Container Connectivity:**
```bash
# Containers on 'beta' network can:
✅ Communicate with each other by name
✅ Reach external networks via NAT
✅ Get automatic IP assignment from range
✅ Use built-in DNS resolution

# Containers on 'beta' network cannot:
❌ Directly communicate with default bridge
❌ Reach containers on other custom networks
❌ Access host networking directly (without port mapping)
```

**Best Use Cases:**
- **Environment Isolation** - Dev, staging, prod networks
- **Application Tiers** - Frontend, backend, database separation
- **Microservices** - Service-specific networks
- **Security Zones** - DMZ, internal, secure networks
- **Multi-tenancy** - Customer-specific isolation

**Important Considerations:**
- Subnet must not overlap with existing networks
- IP range must be within subnet
- Gateway is auto-assigned (first IP in subnet)
- Network persists after containers are removed
- Can't change subnet/IP range after creation (must recreate)

**Production Tip:**
Plan your network addressing scheme before creating networks:
```
10.10.0.0/16 - Development
10.20.0.0/16 - Staging
10.30.0.0/16 - Production

Each environment subdivided:
10.10.1.0/24 - Frontend
10.10.2.0/24 - Backend
10.10.3.0/24 - Database
```

**Remember:** `docker network create --driver bridge --subnet 10.10.1.0/24 --ip-range 10.10.1.0/24 beta` = Full Control Over Container Networking! 🐳

**Next Steps:** Deploy containers to the beta network and test container-to-container communication using DNS names! 🚀
