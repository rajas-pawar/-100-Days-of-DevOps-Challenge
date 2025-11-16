# Day 11: Installing and Configuring Apache Tomcat for Java Application Deployment

## 📋 Objective
Install Apache Tomcat server on App Server 2, configure it to run on port 6100, and deploy the ROOT.war Java application from Jump Host for the Nautilus beta application in Stratos Datacenter.

## 🎯 Resolution
Successfully installed Tomcat on App Server 2, configured it to listen on port 6100, deployed the ROOT.war application, and verified the application is accessible at the base URL.

## 📝 Task Steps for Resolution

1. **Install Java (Tomcat prerequisite)**
2. **Install Tomcat on App Server 2**
3. **Configure Tomcat to run on port 6100**
4. **Transfer ROOT.war from Jump Host to App Server 2**
5. **Deploy WAR file to Tomcat**
6. **Start Tomcat service**
7. **Verify deployment and accessibility**

## 💻 Commands

### Phase 1: Install Java

```bash
# Step 1: SSH to App Server 2
ssh steve@stapp02
# Password: Am3ric@

# Step 2: Switch to root
sudo su -

# Step 3: Check if Java is installed
java -version

# Step 4: Install Java if not present
# For RHEL/CentOS 7
yum install -y java-1.8.0-openjdk java-1.8.0-openjdk-devel

# For RHEL/CentOS 8+
yum install -y java-11-openjdk java-11-openjdk-devel

# For Debian/Ubuntu
apt-get update
apt-get install -y openjdk-11-jdk

# Step 5: Verify Java installation
java -version
javac -version

# Step 6: Set JAVA_HOME (if not set)
echo 'export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk' >> /etc/profile
source /etc/profile
echo $JAVA_HOME
```

### Phase 2: Install Tomcat

```bash
# Method 1: Using Package Manager (Recommended for this task)
# Install Tomcat
yum install -y tomcat

# Verify installation
rpm -qa | grep tomcat

# Method 2: Manual Installation (Alternative)
# Download Tomcat
cd /tmp
wget https://archive.apache.org/dist/tomcat/tomcat-9/v9.0.65/bin/apache-tomcat-9.0.65.tar.gz

# Extract
tar -xzf apache-tomcat-9.0.65.tar.gz

# Move to /opt
mv apache-tomcat-9.0.65 /opt/tomcat

# Create tomcat user
useradd -r -m -U -d /opt/tomcat -s /bin/false tomcat

# Set ownership
chown -R tomcat:tomcat /opt/tomcat
```

### Phase 3: Configure Tomcat Port

```bash
# For package installation
# Edit server.xml
vi /etc/tomcat/server.xml

# Find this line (around line 69):
# <Connector port="8080" protocol="HTTP/1.1"

# Change to:
# <Connector port="6100" protocol="HTTP/1.1"

# Using sed to change port automatically
sed -i 's/port="8080"/port="6100"/g' /etc/tomcat/server.xml

# Verify the change
grep 'Connector port="6100"' /etc/tomcat/server.xml

# For manual installation
vi /opt/tomcat/conf/server.xml
# Make same port change
```

### Phase 4: Transfer ROOT.war

```bash
# Exit from App Server 2
exit
exit  # Exit root, exit steve

# Now on Jump Host (thor user)
# Verify WAR file exists
ls -l /tmp/ROOT.war

# Copy WAR file to App Server 2
scp /tmp/ROOT.war steve@stapp02:/tmp/

# SSH back to App Server 2
ssh steve@stapp02
sudo su -
```

### Phase 5: Deploy WAR File

```bash
# For package installation
# Remove default ROOT application
rm -rf /var/lib/tomcat/webapps/ROOT

# Copy WAR file to webapps
cp /tmp/ROOT.war /var/lib/tomcat/webapps/

# Set proper ownership
chown tomcat:tomcat /var/lib/tomcat/webapps/ROOT.war

# For manual installation
rm -rf /opt/tomcat/webapps/ROOT
cp /tmp/ROOT.war /opt/tomcat/webapps/
chown tomcat:tomcat /opt/tomcat/webapps/ROOT.war
```

### Phase 6: Start Tomcat

```bash
# For package installation
# Start Tomcat service
systemctl start tomcat

# Enable for auto-start
systemctl enable tomcat

# Check status
systemctl status tomcat

# For manual installation
# Create systemd service file
vi /etc/systemd/system/tomcat.service

# Add content:
[Unit]
Description=Apache Tomcat Web Application Container
After=network.target

[Service]
Type=forking
User=tomcat
Group=tomcat

Environment="JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk"
Environment="CATALINA_PID=/opt/tomcat/temp/tomcat.pid"
Environment="CATALINA_HOME=/opt/tomcat"
Environment="CATALINA_BASE=/opt/tomcat"

ExecStart=/opt/tomcat/bin/startup.sh
ExecStop=/opt/tomcat/bin/shutdown.sh

RestartSec=10
Restart=always

[Install]
WantedBy=multi-user.target

# Reload systemd and start
systemctl daemon-reload
systemctl start tomcat
systemctl enable tomcat
```

### Phase 7: Configure Firewall (if needed)

```bash
# Check if firewalld is running
systemctl status firewalld

# If running, allow port 6100
firewall-cmd --permanent --add-port=6100/tcp
firewall-cmd --reload

# Verify
firewall-cmd --list-ports
```

## ✅ Verification

```bash
# Check if Tomcat is running
systemctl status tomcat
# Should show: active (running)

# Check if Tomcat is listening on port 6100
netstat -tlnp | grep 6100
# or
ss -tlnp | grep 6100
# Should show: LISTEN on port 6100

# Check Tomcat process
ps aux | grep tomcat
# Should show tomcat process

# Check if WAR was deployed (creates ROOT directory)
ls -la /var/lib/tomcat/webapps/
# Should show ROOT directory

# Check Tomcat logs
tail -50 /var/log/tomcat/catalina.out
# or
journalctl -u tomcat -f

# Test locally on App Server 2
curl http://localhost:6100
# Should return HTML content

# Test with server hostname
curl http://stapp02:6100
# Should return HTML content

# Test with IP
curl http://172.16.238.11:6100
# Should return HTML content

# From Jump Host, test accessibility
ssh steve@stapp02
curl http://stapp02:6100
# Should work

# Or from Jump Host directly
curl http://stapp02:6100
# Should work if network allows
```

## 🔑 Key Points

- **Tomcat** - Java servlet container and web server
- **Port Configuration** - Changed from default 8080 to 6100
- **WAR File** - Web Application Archive (Java web app package)
- **ROOT.war** - Deploys to root context (accessible at base URL)
- **Key Directories (Package Installation):**
  - Config: `/etc/tomcat/`
  - Webapps: `/var/lib/tomcat/webapps/`
  - Logs: `/var/log/tomcat/`
  - Service: `systemctl tomcat`
- **Key Directories (Manual Installation):**
  - Config: `/opt/tomcat/conf/`
  - Webapps: `/opt/tomcat/webapps/`
  - Logs: `/opt/tomcat/logs/`
  - Binaries: `/opt/tomcat/bin/`
- **Key Files:**
  - `server.xml` - Main configuration (port settings)
  - `web.xml` - Web application configuration
  - `catalina.out` - Main log file
  - `localhost.log` - Access logs
- **Deployment Process:**
  1. Place WAR in webapps directory
  2. Tomcat auto-extracts on startup
  3. Creates directory with WAR name (minus .war)
  4. ROOT.war → ROOT directory → accessible at /
- **Java Requirements:**
  - Tomcat 9.x requires Java 8+
  - Tomcat 10.x requires Java 11+
  - OpenJDK or Oracle JDK works
- **Port 6100:**
  - Non-standard port (default is 8080)
  - Configured in server.xml
  - Must be open in firewall
  - Choose unprivileged port (>1024) for non-root
- **Service Management:**
  - `systemctl start tomcat` - Start service
  - `systemctl stop tomcat` - Stop service
  - `systemctl restart tomcat` - Restart service
  - `systemctl status tomcat` - Check status
  - `systemctl enable tomcat` - Auto-start on boot
- **Common Issues:**
  - Port already in use → Change port or kill process
  - Permission denied → Check ownership of webapps
  - WAR not deploying → Check logs for errors
  - Java not found → Install/configure JAVA_HOME
  - Connection refused → Check service status and port

## ⚠️ Important Notes

- Java must be installed before Tomcat
- Remove default ROOT app before deploying custom one
- WAR file name determines URL path (ROOT.war → /)
- Tomcat auto-deploys WAR files on startup
- Check logs if deployment fails
- Port 6100 must not be in use by another service
- Firewall must allow port 6100
- Enable Tomcat service for auto-start on reboot
- Always verify with curl before declaring success
- Tomcat takes time to deploy WAR (wait 10-30 seconds)
- Check catalina.out for deployment status
- ROOT directory created when ROOT.war deploys successfully
- Default Tomcat runs as tomcat user (not root)
- Ownership of WAR files matters for security
- Consider Tomcat manager app for production deployments

---

**Date Completed**: November 16, 2025  
**Challenge**: KodeKloud 100 Days Cloud DevOps  
**Day**: 11/100