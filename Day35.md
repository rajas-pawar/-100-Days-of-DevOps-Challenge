# Day 35: Docker Installation & Setup
**100 Days Cloud DevOps Challenge – KodeKloud**

---

## Objective

The Nautilus DevOps team is beginning their containerization journey. The first step is to install Docker and Docker Compose on App Server 3 and ensure the Docker service is running properly.

**Requirements:**
1. Install `docker-ce` (Docker Community Edition) on App Server 3
2. Install `docker-compose` package
3. Start the Docker service
4. Verify Docker installation and service status

---

## Understanding Docker

**Docker** is a platform for developing, shipping, and running applications in containers. Containers package application code with dependencies, libraries, and configuration files, ensuring consistency across different environments.

### Why Docker?

- **Consistency:** "Works on my machine" → "Works everywhere"
- **Isolation:** Applications run in isolated environments
- **Portability:** Run anywhere - laptop, server, cloud
- **Efficiency:** Lightweight compared to VMs
- **Speed:** Start containers in seconds
- **Scalability:** Easy to scale up/down
- **DevOps:** Perfect for CI/CD pipelines

### Docker vs Virtual Machines:

| Aspect | Docker Containers | Virtual Machines |
|--------|------------------|------------------|
| Size | MBs | GBs |
| Startup | Seconds | Minutes |
| Performance | Near native | Overhead |
| Isolation | Process level | Hardware level |
| OS | Shares host kernel | Full OS per VM |
| Resource Usage | Lightweight | Heavy |

### Docker Components:

```
┌─────────────────────────────────────┐
│     Docker Architecture             │
├─────────────────────────────────────┤
│  Docker Client (docker CLI)         │
│           ↓                          │
│  Docker Daemon (dockerd)            │
│           ↓                          │
│  ┌──────────────────────────┐      │
│  │  Images  │  Containers    │      │
│  │  Volumes │  Networks      │      │
│  └──────────────────────────┘      │
│           ↓                          │
│  Container Runtime (containerd)     │
│           ↓                          │
│  Host Operating System              │
└─────────────────────────────────────┘
```

---

## Infrastructure Overview

### App Server 3:
| Server | User   | Password    | IP            |
|--------|--------|-------------|---------------|
| stapp03| banner | BigGr33n    | 172.16.238.12 |

### Installation Details:
| Component | Package | Purpose |
|-----------|---------|---------|
| Docker CE | docker-ce | Container runtime |
| Docker Compose | docker-compose | Multi-container orchestration |
| Docker CLI | docker-ce-cli | Command-line interface |
| containerd | containerd.io | Container runtime |

---

## Understanding Docker Compose

**Docker Compose** is a tool for defining and running multi-container Docker applications using YAML files.

### Why Docker Compose?

- **Multi-container apps:** Define all services in one file
- **Simplified management:** Start/stop entire stack with one command
- **Environment consistency:** Same setup across dev/test/prod
- **Networking:** Automatic network setup between containers
- **Volume management:** Persistent data handling

### Docker Compose Use Cases:

- **Web apps:** Nginx + App + Database
- **Microservices:** Multiple interconnected services
- **Development:** Local dev environment matching production
- **Testing:** Spin up test environments quickly

---

## Step-by-Step Implementation

### Step 1: SSH into App Server 3
```bash
ssh banner@stapp03
```

**Enter password:** `BigGr33n`

### Step 2: Switch to Root User
```bash
sudo su -
```

**Or perform each command with sudo:**
```bash
sudo <command>
```

### Step 3: Update System Packages
```bash
yum update -y
```

**This ensures all packages are current**

**Expected output:**
```
Loaded plugins: fastestmirror, ovl
Loading mirror speeds from cached hostfile
...
Complete!
```

### Step 4: Install Required Dependencies
```bash
yum install -y yum-utils device-mapper-persistent-data lvm2
```

**These packages are needed for Docker repository and storage drivers**

**Expected output:**
```
Loaded plugins: fastestmirror, ovl
...
Installed:
  yum-utils.noarch 0:1.1.31-54.el7_8
  device-mapper-persistent-data.x86_64 0:0.8.5-3.el7_9.2
  lvm2.x86_64 7:2.02.187-6.el7_9.5
Complete!
```

### Step 5: Add Docker Repository
```bash
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

**This adds the official Docker CE repository**

**Expected output:**
```
Loaded plugins: fastestmirror, ovl
adding repo from: https://download.docker.com/linux/centos/docker-ce.repo
grabbing file https://download.docker.com/linux/centos/docker-ce.repo to /etc/yum.repos.d/docker-ce.repo
repo saved to /etc/yum.repos.d/docker-ce.repo
```

### Step 6: Verify Repository Added
```bash
yum repolist | grep docker
```

**Expected output:**
```
docker-ce-stable/x86_64       Docker CE Stable - x86_64                   239
```

### Step 7: Install Docker CE
```bash
yum install -y docker-ce docker-ce-cli containerd.io
```

**This installs Docker Community Edition and its components**

**Expected output:**
```
Loaded plugins: fastestmirror, ovl
Loading mirror speeds from cached hostfile
Resolving Dependencies
--> Running transaction check
---> Package containerd.io.x86_64 0:1.6.26-3.1.el7 will be installed
---> Package docker-ce.x86_64 3:24.0.7-1.el7 will be installed
---> Package docker-ce-cli.x86_64 1:24.0.7-1.el7 will be installed
...
Installed:
  docker-ce.x86_64 3:24.0.7-1.el7
  docker-ce-cli.x86_64 1:24.0.7-1.el7
  containerd.io.x86_64 0:1.6.26-3.1.el7

Complete!
```

### Step 8: Verify Docker Installation
```bash
docker --version
```

**Expected output:**
```
Docker version 24.0.7, build afdd53b
```

### Step 9: Check Docker Service Status (Before Starting)
```bash
systemctl status docker
```

**Expected output:**
```
● docker.service - Docker Application Container Engine
   Loaded: loaded (/usr/lib/systemd/system/docker.service; disabled; vendor preset: disabled)
   Active: inactive (dead)
     Docs: https://docs.docker.com
```

**Status:** inactive (dead) - service not yet started

### Step 10: Start Docker Service
```bash
systemctl start docker
```

**No output means success**

### Step 11: Verify Docker Service Running
```bash
systemctl status docker
```

**Expected output:**
```
● docker.service - Docker Application Container Engine
   Loaded: loaded (/usr/lib/systemd/system/docker.service; disabled; vendor preset: disabled)
   Active: active (running) since Tue 2025-12-10 10:00:00 UTC; 5s ago
     Docs: https://docs.docker.com
 Main PID: 12345 (dockerd)
   Memory: 38.5M
   CGroup: /docker/12345
           └─12345 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock

Dec 10 10:00:00 stapp03 systemd[1]: Starting Docker Application Container Engine...
Dec 10 10:00:00 stapp03 dockerd[12345]: time="2025-12-10T10:00:00Z" level=info msg="Starting up"
Dec 10 10:00:00 stapp03 systemd[1]: Started Docker Application Container Engine.
```

**Key indicators:**
- ✅ Active: **active (running)**
- ✅ Main PID shown
- ✅ "Started Docker Application Container Engine"

### Step 12: Enable Docker to Start on Boot
```bash
systemctl enable docker
```

**Expected output:**
```
Created symlink from /etc/systemd/system/multi-user.target.wants/docker.service to /usr/lib/systemd/system/docker.service.
```

**This ensures Docker starts automatically after system reboot**

### Step 13: Verify Docker is Enabled
```bash
systemctl is-enabled docker
```

**Expected output:**
```
enabled
```

### Step 14: Test Docker with Hello World
```bash
docker run hello-world
```

**Expected output:**
```
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
c1ec31eb5944: Pull complete
Digest: sha256:4bd78111b6914a99dbc560e6a20eab57ff6655aea4a80c50b0c5491968cbc2e6
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.
...
```

### Step 15: Install Docker Compose
```bash
yum install -y docker-compose-plugin
```

**Or install standalone Docker Compose (older method):**
```bash
curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```

**Expected output:**
```
Loaded plugins: fastestmirror, ovl
...
Installed:
  docker-compose-plugin.x86_64 0:2.23.3-1.el7

Complete!
```

### Step 16: Verify Docker Compose Installation
```bash
docker compose version
```

**Expected output:**
```
Docker Compose version v2.23.3
```

**Or for standalone:**
```bash
docker-compose --version
```

**Expected output:**
```
docker-compose version 1.29.2, build 5becea4c
```

### Step 17: Check Docker Info
```bash
docker info
```

**Expected output (abbreviated):**
```
Client:
 Version:    24.0.7
 Context:    default
 Debug Mode: false

Server:
 Containers: 1
  Running: 0
  Paused: 0
  Stopped: 1
 Images: 1
 Server Version: 24.0.7
 Storage Driver: overlay2
 Logging Driver: json-file
 Cgroup Driver: cgroupfs
 Cgroup Version: 1
 Plugins:
  Volume: local
  Network: bridge host ipvlan macvlan null overlay
 Kernel Version: 3.10.0-1160.el7.x86_64
 Operating System: CentOS Linux 7 (Core)
 OSType: linux
 Architecture: x86_64
 CPUs: 2
 Total Memory: 3.7GiB
 Docker Root Dir: /var/lib/docker
...
```

### Step 18: List Docker Images
```bash
docker images
```

**Expected output:**
```
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
hello-world   latest    d2c94e258dcb   7 months ago   13.3kB
```

### Step 19: List Docker Containers
```bash
docker ps -a
```

**Expected output:**
```
CONTAINER ID   IMAGE         COMMAND    CREATED          STATUS                      PORTS     NAMES
abc123def456   hello-world   "/hello"   2 minutes ago    Exited (0) 2 minutes ago              wonderful_pike
```

### Step 20: Verify Docker Group
```bash
cat /etc/group | grep docker
```

**Expected output:**
```
docker:x:993:
```

**Optional - Add user to docker group (avoid sudo for docker commands):**
```bash
usermod -aG docker banner
```

**Then logout and login again for changes to take effect**

---

## Complete Command Summary

### Quick Installation:
```bash
# SSH and switch to root
ssh banner@stapp03
sudo su -

# Update system
yum update -y

# Install dependencies
yum install -y yum-utils device-mapper-persistent-data lvm2

# Add Docker repository
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

# Install Docker CE
yum install -y docker-ce docker-ce-cli containerd.io

# Start Docker service
systemctl start docker

# Enable Docker on boot
systemctl enable docker

# Verify Docker
docker --version
systemctl status docker
docker run hello-world

# Install Docker Compose
yum install -y docker-compose-plugin

# Verify Docker Compose
docker compose version

# Check Docker info
docker info
```

### Verification Commands:
```bash
# Check versions
docker --version
docker compose version

# Check service status
systemctl status docker
systemctl is-enabled docker

# Test Docker
docker run hello-world
docker ps -a
docker images

# View Docker info
docker info
docker version
```

---

## Understanding Docker Installation Components

### 1. docker-ce (Docker Community Edition)
**The main Docker engine**
- Container runtime
- Image management
- Volume management
- Network management

### 2. docker-ce-cli
**Docker command-line interface**
- `docker run`, `docker build`, etc.
- Client that communicates with Docker daemon
- Can be installed separately from daemon

### 3. containerd.io
**Container runtime**
- Low-level container runtime
- Manages container lifecycle
- Used by Docker daemon
- Industry standard (CNCF project)

### 4. docker-compose-plugin
**Multi-container orchestration**
- Define services in YAML
- Manage multiple containers
- Networking between containers
- Volume management

---

## Docker Service Management

### Service Commands:

**Start Docker:**
```bash
systemctl start docker
```

**Stop Docker:**
```bash
systemctl stop docker
```

**Restart Docker:**
```bash
systemctl restart docker
```

**Enable on boot:**
```bash
systemctl enable docker
```

**Disable on boot:**
```bash
systemctl disable docker
```

**Check status:**
```bash
systemctl status docker
```

**View logs:**
```bash
journalctl -u docker
```

**Follow logs:**
```bash
journalctl -u docker -f
```

---

## Basic Docker Commands

### Image Management:

**Pull image:**
```bash
docker pull nginx
```

**List images:**
```bash
docker images
```

**Remove image:**
```bash
docker rmi nginx
```

**Search images:**
```bash
docker search ubuntu
```

### Container Management:

**Run container:**
```bash
docker run -d --name webserver -p 80:80 nginx
```

**List running containers:**
```bash
docker ps
```

**List all containers:**
```bash
docker ps -a
```

**Stop container:**
```bash
docker stop webserver
```

**Start container:**
```bash
docker start webserver
```

**Remove container:**
```bash
docker rm webserver
```

**View logs:**
```bash
docker logs webserver
```

**Execute command in container:**
```bash
docker exec -it webserver bash
```

### System Management:

**View Docker info:**
```bash
docker info
```

**View version:**
```bash
docker version
```

**View disk usage:**
```bash
docker system df
```

**Clean up:**
```bash
docker system prune
```

---

## Docker Compose Basics

### Sample docker-compose.yml:

```yaml
version: '3.8'

services:
  web:
    image: nginx:latest
    ports:
      - "80:80"
    volumes:
      - ./html:/usr/share/nginx/html
    
  database:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: password123
      MYSQL_DATABASE: myapp
    volumes:
      - db-data:/var/lib/mysql

volumes:
  db-data:
```

### Docker Compose Commands:

**Start services:**
```bash
docker compose up -d
```

**Stop services:**
```bash
docker compose down
```

**View logs:**
```bash
docker compose logs
```

**List services:**
```bash
docker compose ps
```

**Rebuild services:**
```bash
docker compose up -d --build
```

---

## Troubleshooting

### Issue 1: Docker Service Won't Start

**Problem:**
```
Job for docker.service failed...
```

**Solution:**
```bash
# Check logs
journalctl -u docker

# Common issues:
# 1. Port conflict
netstat -tlnp | grep 2375

# 2. Storage driver issue
dockerd --debug

# 3. Restart containerd
systemctl restart containerd
systemctl start docker
```

### Issue 2: Permission Denied

**Problem:**
```
Got permission denied while trying to connect to the Docker daemon socket
```

**Solution:**
```bash
# Add user to docker group
sudo usermod -aG docker $USER

# Logout and login again
exit
ssh banner@stapp03

# Or use sudo
sudo docker ps
```

### Issue 3: Cannot Connect to Docker Daemon

**Problem:**
```
Cannot connect to the Docker daemon. Is the docker daemon running?
```

**Solution:**
```bash
# Check if docker is running
systemctl status docker

# Start docker if not running
systemctl start docker

# Check socket
ls -la /var/run/docker.sock
```

### Issue 4: Docker Compose Not Found

**Problem:**
```
bash: docker-compose: command not found
```

**Solution:**
```bash
# Install docker-compose-plugin
yum install -y docker-compose-plugin

# Or download standalone
curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose

# Verify
docker compose version
```

### Issue 5: Storage Issues

**Problem:**
```
no space left on device
```

**Solution:**
```bash
# Check disk usage
df -h
docker system df

# Clean up
docker system prune -a

# Remove unused volumes
docker volume prune

# Remove unused images
docker image prune -a
```

### Issue 6: Network Issues in Container

**Problem:** Container can't reach internet

**Solution:**
```bash
# Check Docker network
docker network ls

# Inspect network
docker network inspect bridge

# Check DNS
docker run busybox nslookup google.com

# Restart Docker with DNS config
vi /etc/docker/daemon.json
# Add:
# {
#   "dns": ["8.8.8.8", "8.8.4.4"]
# }

systemctl restart docker
```

---

## Docker Storage and Networking

### Storage Drivers:

**View current driver:**
```bash
docker info | grep "Storage Driver"
```

**Common drivers:**
- **overlay2** - Recommended for most distributions
- **devicemapper** - Older CentOS/RHEL
- **btrfs** - BTRFS filesystem
- **zfs** - ZFS filesystem

### Docker Networks:

**Default networks:**
```bash
docker network ls
```

**Output:**
```
NETWORK ID     NAME      DRIVER    SCOPE
abc123def456   bridge    bridge    local
def456ghi789   host      host      local
ghi789jkl012   none      null      local
```

**Network types:**
- **bridge** - Default, isolated network
- **host** - Use host network directly
- **none** - No networking
- **overlay** - Multi-host networking

---

## Docker Best Practices

### 1. Keep Docker Updated
```bash
# Check for updates
yum check-update docker-ce

# Update Docker
yum update docker-ce docker-ce-cli containerd.io
```

### 2. Use Docker Volumes for Persistence
```bash
# Create volume
docker volume create mydata

# Use volume
docker run -v mydata:/data nginx
```

### 3. Limit Container Resources
```bash
# Limit memory and CPU
docker run -m 512m --cpus="1.0" nginx
```

### 4. Use .dockerignore
```
# .dockerignore
.git
node_modules
*.log
.env
```

### 5. Clean Up Regularly
```bash
# Remove stopped containers
docker container prune

# Remove unused images
docker image prune

# Remove everything unused
docker system prune -a
```

### 6. Use Official Images
```bash
# Official images from Docker Hub
docker pull nginx
docker pull mysql
docker pull node
```

### 7. Tag Images Properly
```bash
# Build with tag
docker build -t myapp:v1.0 .

# Tag existing image
docker tag myapp:v1.0 myapp:latest
```

---

## Docker Architecture Deep Dive

### Docker Components Flow:

```
┌─────────────────────────────────────────────────┐
│                  Docker Client                   │
│               (docker command)                   │
└──────────────────┬──────────────────────────────┘
                   │ REST API
                   ↓
┌─────────────────────────────────────────────────┐
│              Docker Daemon (dockerd)             │
│  ┌──────────────────────────────────────────┐  │
│  │         Image Management                  │  │
│  │    (pull, build, push, tag, remove)      │  │
│  └──────────────────────────────────────────┘  │
│  ┌──────────────────────────────────────────┐  │
│  │      Container Management                 │  │
│  │   (create, start, stop, remove, exec)    │  │
│  └──────────────────────────────────────────┘  │
│  ┌──────────────────────────────────────────┐  │
│  │   Volume & Network Management             │  │
│  └──────────────────────────────────────────┘  │
└──────────────────┬──────────────────────────────┘
                   ↓
┌─────────────────────────────────────────────────┐
│              containerd                          │
│      (container lifecycle management)            │
└──────────────────┬──────────────────────────────┘
                   ↓
┌─────────────────────────────────────────────────┐
│              runc                                │
│      (OCI runtime - runs containers)             │
└──────────────────┬──────────────────────────────┘
                   ↓
┌─────────────────────────────────────────────────┐
│         Linux Kernel (cgroups, namespaces)       │
└─────────────────────────────────────────────────┘
```

---

## Real-World Docker Use Cases

### 1. Web Application Deployment
```bash
# Nginx + App + Database
docker compose up -d
```

### 2. Development Environment
```bash
# Consistent dev environment
docker run -v $(pwd):/app -p 3000:3000 node:18
```

### 3. Microservices
```bash
# Multiple interconnected services
docker compose with multiple services
```

### 4. CI/CD Pipeline
```bash
# Build, test, deploy in containers
docker build -t app:test .
docker run app:test npm test
```

### 5. Database Testing
```bash
# Spin up test database
docker run -d -p 3306:3306 -e MYSQL_ROOT_PASSWORD=pass mysql:8.0
```

---

## Key Docker Commands Reference

| Command | Description |
|---------|-------------|
| `docker --version` | Show Docker version |
| `docker run <image>` | Run container from image |
| `docker ps` | List running containers |
| `docker ps -a` | List all containers |
| `docker images` | List images |
| `docker pull <image>` | Pull image from registry |
| `docker build -t <tag> .` | Build image from Dockerfile |
| `docker stop <container>` | Stop container |
| `docker start <container>` | Start container |
| `docker rm <container>` | Remove container |
| `docker rmi <image>` | Remove image |
| `docker logs <container>` | View container logs |
| `docker exec -it <container> bash` | Enter container shell |
| `docker compose up` | Start compose services |
| `docker system prune` | Clean up unused resources |

---

## Key Takeaways

- **Docker** enables containerization for consistent application deployment
- **Installation** requires docker-ce, docker-ce-cli, and containerd.io
- **Service management** through systemctl (start, enable, status)
- **Docker Compose** manages multi-container applications
- **Testing** with hello-world verifies installation
- **Lightweight** compared to VMs, starts in seconds
- **Portable** - same container runs anywhere
- **Foundation** for modern DevOps and microservices

---

## Completion Checklist

- [ ] SSH into App Server 3 as banner
- [ ] Switched to root/sudo privileges
- [ ] Updated system packages
- [ ] Installed required dependencies
- [ ] Added Docker CE repository
- [ ] Installed docker-ce package
- [ ] Installed docker-ce-cli package
- [ ] Installed containerd.io package
- [ ] Started Docker service
- [ ] Enabled Docker service on boot
- [ ] Verified Docker version
- [ ] Verified Docker service status (active/running)
- [ ] Tested Docker with hello-world
- [ ] Installed docker-compose-plugin
- [ ] Verified Docker Compose version
- [ ] Checked Docker info output
- [ ] Listed Docker images
- [ ] Listed Docker containers

---

## Completion Details

- **Completion Date:** December 10, 2025
- **Day:** 35 / 100
- **Challenge:** KodeKloud 100 Days Cloud DevOps
- **Topic:** Docker Installation & Setup
- **Server:** App Server 3 (stapp03)
- **User:** banner (password: BigGr33n)
- **Packages Installed:**
  - docker-ce (Docker Community Edition)
  - docker-ce-cli (Docker CLI)
  - containerd.io (Container runtime)
  - docker-compose-plugin (Docker Compose)
- **Service Status:** active (running), enabled
- **Testing:** hello-world container successfully run
- **Key Skill:** Container platform installation and configuration
- **Status:** ✅ Successfully Completed

---

## Summary

This task marked the **transition from Git to Containerization**:

✅ **Installed Docker CE** - Container runtime platform
✅ **Installed Docker Compose** - Multi-container orchestration
✅ **Started Docker service** - Daemon running and enabled
✅ **Verified installation** - hello-world test successful
✅ **Ready for containerization** - Platform ready for apps

**Key Insight:** Docker is the foundation of modern application deployment. It:
- Packages applications with dependencies
- Ensures consistency across environments
- Simplifies deployment and scaling
- Enables microservices architecture
- Integrates with CI/CD pipelines
- Is essential for cloud-native development

Unlike traditional deployments where environment differences cause issues, Docker containers guarantee that applications run the same way everywhere - from developer laptop to production server.

**Remember:** `Docker` = Containerization = DevOps Foundation! 🐳

**The DevOps Journey Continues:** From version control (Git) to containerization (Docker), we're building the complete DevOps toolkit!

**Next Steps:** Create Dockerfiles, build images, run containers, docker-compose applications! 🚀
