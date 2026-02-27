# Homelab Setup for Android Remote Access

## Overview
Setup guide for accessing your Linux homelab server from Android devices for development and system management.

---

## Architecture Approaches

### Approach 1: Browser-Based Development Environment
Access full IDE through mobile browser - no app installation needed.

### Approach 2: Mobile SSH Client + Optional GUI
Terminal access with graphical desktop option when needed.

### Approach 3: Containerized Web Services
Deploy web-based tools in containers for specific workflows.

---

## Solution 1: Code-Server (Web-Based VS Code)

### Server Setup

**Install Code-Server:**
```bash
# Create installation directory
mkdir -p ~/apps/code-server
cd ~/apps/code-server

# Download latest release
export VERSION=4.20.0
wget https://github.com/coder/code-server/releases/download/v${VERSION}/code-server-${VERSION}-linux-amd64.tar.gz

# Extract and setup
tar -xzf code-server-${VERSION}-linux-amd64.tar.gz
ln -s code-server-${VERSION}-linux-amd64 current
```

**Create systemd service:**
```bash
sudo tee /etc/systemd/system/code-server.service << 'EOF'
[Unit]
Description=Code Server IDE
After=network.target

[Service]
Type=simple
User=yourusername
WorkingDirectory=/home/yourusername
ExecStart=/home/yourusername/apps/code-server/current/bin/code-server --bind-addr 0.0.0.0:8080
Restart=always

[Install]
WantedBy=multi-user.target
EOF

# Enable and start
sudo systemctl daemon-reload
sudo systemctl enable code-server
sudo systemctl start code-server
```

**Configuration file:**
```bash
mkdir -p ~/.config/code-server
nano ~/.config/code-server/config.yaml
```

**Config content:**
```yaml
bind-addr: 0.0.0.0:8080
auth: password
password: ChangeThisToSecurePassword123!
cert: false
```

### Secure Access with Reverse Proxy

**Install Nginx:**
```bash
sudo apt update && sudo apt install -y nginx certbot python3-certbot-nginx
```

**Configure reverse proxy:**
```bash
sudo nano /etc/nginx/sites-available/code-server
```

**Nginx configuration:**
```nginx
server {
    listen 80;
    server_name your-domain.duckdns.org;
    
    location / {
        proxy_pass http://127.0.0.1:8080/;
        proxy_set_header Host $host;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection upgrade;
        proxy_set_header Accept-Encoding gzip;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

**Enable site:**
```bash
sudo ln -s /etc/nginx/sites-available/code-server /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

**Add SSL certificate:**
```bash
sudo certbot --nginx -d your-domain.duckdns.org
```

---

## Solution 2: JupyterLab (For Python/Data Science)

### Installation

```bash
# Install Python and pip
sudo apt install -y python3 python3-pip python3-venv

# Create virtual environment
mkdir -p ~/jupyter
python3 -m venv ~/jupyter/venv
source ~/jupyter/venv/bin/activate

# Install JupyterLab
pip install jupyterlab notebook ipywidgets
```

### Configuration

```bash
# Generate config
jupyter lab --generate-config

# Set password
jupyter lab password

# Edit config
nano ~/.jupyter/jupyter_lab_config.py
```

**Key settings:**
```python
c.ServerApp.ip = '0.0.0.0'
c.ServerApp.port = 8888
c.ServerApp.open_browser = False
c.ServerApp.allow_remote_access = True
```

### Create systemd service

```bash
sudo nano /etc/systemd/system/jupyterlab.service
```

**Service file:**
```ini
[Unit]
Description=Jupyter Lab Server
After=network.target

[Service]
Type=simple
User=yourusername
WorkingDirectory=/home/yourusername
ExecStart=/home/yourusername/jupyter/venv/bin/jupyter lab --config=/home/yourusername/.jupyter/jupyter_lab_config.py
Restart=always

[Install]
WantedBy=multi-user.target
```

**Start service:**
```bash
sudo systemctl daemon-reload
sudo systemctl enable jupyterlab
sudo systemctl start jupyterlab
```

---

## Solution 3: SSH Access from Android

### Recommended Android SSH Clients

1. **Termux** - Full terminal emulator with package manager
2. **ConnectBot** - Simple SSH client
3. **JuiceSSH** - Feature-rich SSH client with plugins

### Server SSH Hardening

```bash
# Edit SSH config
sudo nano /etc/ssh/sshd_config
```

**Recommended settings:**
```bash
Port 2222  # Change from default 22
PermitRootLogin no
PubkeyAuthentication yes
PasswordAuthentication no
MaxAuthTries 3
```

**Restart SSH:**
```bash
sudo systemctl restart sshd
```

### Generate SSH Keys on Android (Using Termux)

```bash
# Install Termux from F-Droid or Google Play
# Inside Termux app:

pkg update && pkg upgrade
pkg install openssh

# Generate key pair
ssh-keygen -t ed25519 -C "android-device"

# Copy public key to server
ssh-copy-id -p 2222 username@server-ip

# Or manually:
cat ~/.ssh/id_ed25519.pub
# Copy output and add to server's ~/.ssh/authorized_keys
```

### Mobile-Friendly tmux Configuration

```bash
# Install tmux
sudo apt install -y tmux

# Create config
nano ~/.tmux.conf
```

**tmux.conf for mobile:**
```bash
# Increase scrollback buffer
set -g history-limit 10000

# Easy prefix key for mobile
set -g prefix C-a
unbind C-b
bind C-a send-prefix

# Mouse support
set -g mouse on

# Status bar
set -g status-position bottom
set -g status-style 'bg=blue fg=white'

# Split commands
bind | split-window -h
bind - split-window -v
```

---

## Solution 4: Tailscale VPN (Access from Anywhere)

### Installation on Server

```bash
# Install Tailscale
curl -fsSL https://pkgs.tailscale.com/stable/ubuntu/jammy.noarch.gpg | sudo tee /usr/share/keyrings/tailscale-archive-keyring.gpg >/dev/null
curl -fsSL https://pkgs.tailscale.com/stable/ubuntu/jammy.list | sudo tee /etc/apt/sources.list.d/tailscale.list

sudo apt update
sudo apt install -y tailscale

# Start and authenticate
sudo tailscale up

# Enable subnet routes (optional)
sudo tailscale up --advertise-routes=192.168.1.0/24
```

### Installation on Android

1. Download **Tailscale** app from Play Store
2. Sign in with same account
3. Connect to network
4. Access server using Tailscale IP (e.g., 100.x.x.x)

**Benefits:**
- No port forwarding needed
- Encrypted mesh network
- Stable IPs across networks
- Works on mobile data

---

## Solution 5: Docker-Based Web Applications

### Portainer (Container Management)

```bash
docker volume create portainer_data

docker run -d \
  --name=portainer \
  --restart=always \
  -p 9000:9000 \
  -p 9443:9443 \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data \
  portainer/portainer-ce:latest
```

Access at: `https://server-ip:9443`

### File Browser (Web File Manager)

```bash
docker run -d \
  --name=filebrowser \
  --restart=always \
  -p 8082:80 \
  -v /home/yourusername:/srv \
  -v /path/to/filebrowser.db:/database.db \
  filebrowser/filebrowser:latest
```

### Nginx File Server (Simple Sharing)

```bash
docker run -d \
  --name=file-server \
  --restart=always \
  -p 8083:80 \
  -v /home/yourusername/shared:/usr/share/nginx/html:ro \
  nginx:alpine
```

---

## Dynamic DNS Setup

### DuckDNS Configuration

```bash
# Create update script
mkdir -p ~/scripts
nano ~/scripts/duckdns-update.sh
```

**Script content:**
```bash
#!/bin/bash
echo url="https://www.duckdns.org/update?domains=yourdomain&token=your-token&ip=" | curl -k -o ~/duckdns.log -K -
```

**Make executable and add cron:**
```bash
chmod +x ~/scripts/duckdns-update.sh

# Add to crontab
crontab -e

# Add line:
*/5 * * * * ~/scripts/duckdns-update.sh
```

---

## Android Apps Recommendations

### Development Tools
- **Termux** - Terminal emulator + Linux environment
- **Code Editor** - Lightweight code editor (offline)
- **Spck Editor** - Mobile-optimized web development

### Remote Access
- **JuiceSSH** - SSH/Mosh client with snippets
- **RealVNC Viewer** - VNC client for GUI access
- **Microsoft Remote Desktop** - If using xRDP

### File Management
- **Solid Explorer** - SFTP/FTP support
- **Total Commander** - With SFTP plugin
- **X-plore** - Built-in network features

### Browsers
- **Brave** - Privacy-focused with mobile desktop mode
- **Kiwi Browser** - Chrome extensions support
- **Firefox** - Good developer tools

---

## Network Configuration

### Router Port Forwarding

**For direct access (if not using Tailscale):**

| Service | Internal Port | External Port |
|---------|--------------|---------------|
| SSH | 2222 | 2222 |
| Code-Server (HTTP) | 80 | 80 |
| Code-Server (HTTPS) | 443 | 443 |
| JupyterLab | 8888 | 8888 |

### Firewall Rules

```bash
# Install UFW
sudo apt install -y ufw

# Default policies
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow services
sudo ufw allow 2222/tcp comment 'SSH'
sudo ufw allow 80/tcp comment 'HTTP'
sudo ufw allow 443/tcp comment 'HTTPS'
sudo ufw allow 8080/tcp comment 'Code-Server'

# Enable firewall
sudo ufw enable
sudo ufw status
```

---

## Security Best Practices

### Fail2Ban Configuration

```bash
# Install fail2ban
sudo apt install -y fail2ban

# Create local config
sudo nano /etc/fail2ban/jail.local
```

**Basic configuration:**
```ini
[DEFAULT]
bantime = 1h
findtime = 10m
maxretry = 5

[sshd]
enabled = true
port = 2222
```

**Start service:**
```bash
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
```

### Automatic Updates

```bash
# Install unattended-upgrades
sudo apt install -y unattended-upgrades

# Configure
sudo dpkg-reconfigure -plow unattended-upgrades
```

---

## Monitoring from Android

### Glances (System Monitor)

```bash
# Install Glances
sudo apt install -y glances

# Run as web server
glances -w --port 61208

# Or as systemd service
sudo nano /etc/systemd/system/glances.service
```

**Service file:**
```ini
[Unit]
Description=Glances System Monitor
After=network.target

[Service]
ExecStart=/usr/bin/glances -w --port 61208
Restart=always

[Install]
WantedBy=multi-user.target
```

Access at: `http://server-ip:61208`

---

## Quick Start Checklist

- [ ] Install Code-Server or JupyterLab
- [ ] Setup SSH with key authentication
- [ ] Configure firewall (UFW)
- [ ] Install Tailscale for secure access
- [ ] Setup dynamic DNS (DuckDNS)
- [ ] Configure reverse proxy with SSL
- [ ] Install fail2ban
- [ ] Setup automatic updates
- [ ] Test connection from Android device
- [ ] Bookmark web interfaces
- [ ] Configure tmux for terminal sessions

---

## Troubleshooting

### Can't Connect from Android

```bash
# Check service status
sudo systemctl status code-server
sudo systemctl status nginx

# Check firewall
sudo ufw status
sudo netstat -tulpn | grep LISTEN

# Check logs
journalctl -u code-server -f
tail -f /var/log/nginx/error.log
```

### SSL Certificate Issues

```bash
# Renew certificates
sudo certbot renew --dry-run
sudo certbot renew

# Check certificate status
sudo certbot certificates
```

### Performance Issues on Mobile

- Enable compression in Nginx
- Reduce Code-Server extensions
- Use lighter themes
- Disable unnecessary features
- Consider using SSH + vim/nano for quick edits

---

## Backup Configuration

```bash
# Backup script
nano ~/scripts/backup-configs.sh
```

**Script:**
```bash
#!/bin/bash
BACKUP_DIR=~/backups/$(date +%Y%m%d)
mkdir -p $BACKUP_DIR

# Backup configurations
cp ~/.config/code-server/config.yaml $BACKUP_DIR/
cp ~/.jupyter/jupyter_lab_config.py $BACKUP_DIR/
cp ~/.tmux.conf $BACKUP_DIR/
cp -r ~/.ssh $BACKUP_DIR/

# Compress
tar -czf ~/backups/config-$(date +%Y%m%d).tar.gz -C ~/backups $(date +%Y%m%d)
rm -rf $BACKUP_DIR

echo "Backup completed: config-$(date +%Y%m%d).tar.gz"
```

---

## Resources

- Code-Server Docs: https://coder.com/docs/code-server
- JupyterLab Docs: https://jupyterlab.readthedocs.io
- Tailscale Setup: https://tailscale.com/kb/
- DuckDNS: https://www.duckdns.org
- Termux Wiki: https://wiki.termux.com

---

**Happy mobile coding! 📱💻**

---

## Exposing Code-Server to Internet

### Security Warning ⚠️

Exposing services to the internet requires proper security measures:
- Use strong passwords or OAuth authentication
- Enable HTTPS/SSL certificates
- Implement rate limiting
- Use fail2ban for brute force protection
- Consider VPN (Tailscale) as safer alternative

---

## Method 1: Port Forwarding + Dynamic DNS (Basic)

### Step 1: Setup Dynamic DNS

**Using DuckDNS (Free):**

```bash
# Register at duckdns.org and get your token
# Create update script
mkdir -p ~/scripts
nano ~/scripts/duckdns-update.sh
```

**Script content:**
```bash
#!/bin/bash
DOMAIN="your-domain"
TOKEN="your-token-from-duckdns"

echo url="https://www.duckdns.org/update?domains=${DOMAIN}&token=${TOKEN}&ip=" | curl -k -o ~/duckdns.log -K -

# Log the update
date >> ~/duckdns.log
```

**Make executable and automate:**
```bash
chmod +x ~/scripts/duckdns-update.sh

# Test it
~/scripts/duckdns-update.sh

# Add to crontab (update every 5 minutes)
crontab -e

# Add this line:
*/5 * * * * ~/scripts/duckdns-update.sh
```

### Step 2: Configure Router Port Forwarding

**Router configuration (varies by model):**
1. Log into router (usually 192.168.1.1 or 192.168.0.1)
2. Find "Port Forwarding" or "Virtual Server" section
3. Add rule:
   - **External Port**: 443 (HTTPS)
   - **Internal IP**: Your server's local IP (e.g., 192.168.1.100)
   - **Internal Port**: 443
   - **Protocol**: TCP

**Optional - also forward HTTP:**
   - **External Port**: 80
   - **Internal Port**: 80
   - **Protocol**: TCP

### Step 3: Configure Nginx with SSL

**Get SSL certificate:**
```bash
# Install certbot
sudo apt update
sudo apt install -y certbot python3-certbot-nginx

# Get certificate (router must be forwarding port 80/443)
sudo certbot --nginx -d your-domain.duckdns.org
```

**Nginx configuration:**
```bash
sudo nano /etc/nginx/sites-available/code-server
```

**Complete configuration:**
```nginx
# HTTP - Redirect to HTTPS
server {
    listen 80;
    listen [::]:80;
    server_name your-domain.duckdns.org;
    
    # Let's Encrypt verification
    location /.well-known/acme-challenge/ {
        root /var/www/html;
    }
    
    # Redirect all other traffic to HTTPS
    location / {
        return 301 https://$server_name$request_uri;
    }
}

# HTTPS
server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name your-domain.duckdns.org;

    # SSL Configuration
    ssl_certificate /etc/letsencrypt/live/your-domain.duckdns.org/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/your-domain.duckdns.org/privkey.pem;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;

    # Security Headers
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;

    # Code-Server Proxy
    location / {
        proxy_pass http://127.0.0.1:8080/;
        proxy_set_header Host $host;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection upgrade;
        proxy_set_header Accept-Encoding gzip;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # WebSocket support
        proxy_http_version 1.1;
        proxy_read_timeout 86400;
    }

    # Rate limiting
    limit_req_zone $binary_remote_addr zone=codeserver:10m rate=10r/s;
    limit_req zone=codeserver burst=20 nodelay;
}
```

**Enable site:**
```bash
sudo ln -s /etc/nginx/sites-available/code-server /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

### Step 4: Setup Certificate Auto-Renewal

```bash
# Test renewal
sudo certbot renew --dry-run

# Certbot automatically adds renewal to cron/systemd
# Verify timer is active
sudo systemctl status certbot.timer
```

---

## Method 2: Cloudflare Tunnel (No Port Forwarding Needed)

**Best for:**
- No need to open router ports
- DDoS protection
- Free SSL
- Works behind CGNAT

### Installation

```bash
# Download cloudflared
wget https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb
sudo dpkg -i cloudflared-linux-amd64.deb

# Login to Cloudflare
cloudflared tunnel login
```

### Create Tunnel

```bash
# Create new tunnel
cloudflared tunnel create code-server

# Note the tunnel ID and credentials file location
# Example output: Created tunnel code-server with id: abc123...
```

### Configure Tunnel

```bash
# Create config directory
mkdir -p ~/.cloudflared

# Create config file
nano ~/.cloudflared/config.yml
```

**Config content:**
```yaml
url: http://localhost:8080
tunnel: abc123-your-tunnel-id
credentials-file: /home/yourusername/.cloudflared/abc123-your-tunnel-id.json

ingress:
  - hostname: code.yourdomain.com
    service: http://localhost:8080
  - service: http_status:404
```

### Setup DNS

```bash
# Route DNS to tunnel
cloudflared tunnel route dns code-server code.yourdomain.com
```

### Run as Service

```bash
# Install as system service
sudo cloudflared service install

# Enable and start
sudo systemctl enable cloudflared
sudo systemctl start cloudflared

# Check status
sudo systemctl status cloudflared
```

**Access your Code-Server:**
```
https://code.yourdomain.com
```

---

## Method 3: Tailscale + Funnel (Hybrid Approach)

**Best for:**
- Secure VPN by default
- Optionally share specific services publicly
- No port forwarding
- Works anywhere

### Setup Tailscale

```bash
# Install Tailscale (if not already installed)
curl -fsSL https://tailscale.com/install.sh | sh

# Start Tailscale
sudo tailscale up
```

### Enable HTTPS

```bash
# Enable HTTPS on Tailscale machine
sudo tailscale cert $(tailscale status --json | jq -r '.Self.DNSName')
```

### Configure Code-Server for HTTPS

```bash
# Update code-server config
nano ~/.config/code-server/config.yaml
```

**Config:**
```yaml
bind-addr: 0.0.0.0:8080
auth: password
password: YourSecurePassword123!
cert: /var/lib/tailscale/certs/your-machine.tailnet.ts.net.crt
cert-key: /var/lib/tailscale/certs/your-machine.tailnet.ts.net.key
```

### Enable Funnel (Public Access)

```bash
# Expose port 8080 publicly via Funnel
sudo tailscale funnel 8080
```

**Access:**
- Private (Tailscale only): `https://your-machine.tailnet.ts.net:8080`
- Public (Funnel): `https://your-machine.tailnet.ts.net` (port 443 automatically)

---

## Method 4: Ngrok (Quick Testing)

**Best for:**
- Quick testing
- Temporary access
- Development/demos

### Installation

```bash
# Download ngrok
wget https://bin.equinox.io/c/bNyj1mQVY4c/ngrok-v3-stable-linux-amd64.tgz
tar -xzf ngrok-v3-stable-linux-amd64.tgz
sudo mv ngrok /usr/local/bin/

# Signup at ngrok.com and get auth token
ngrok config add-authtoken YOUR_AUTH_TOKEN
```

### Start Tunnel

```bash
# Expose code-server
ngrok http 8080

# Or with custom subdomain (paid plans)
ngrok http --domain=your-subdomain.ngrok.io 8080
```

### Run as Background Service

```bash
# Create systemd service
sudo nano /etc/systemd/system/ngrok.service
```

**Service file:**
```ini
[Unit]
Description=Ngrok Tunnel
After=network.target

[Service]
Type=simple
User=yourusername
ExecStart=/usr/local/bin/ngrok http 8080
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

**Start service:**
```bash
sudo systemctl daemon-reload
sudo systemctl enable ngrok
sudo systemctl start ngrok

# Get public URL
curl http://localhost:4040/api/tunnels | jq -r '.tunnels[0].public_url'
```

---

## Security Hardening Checklist

### 1. Strong Authentication

**Update Code-Server password:**
```bash
nano ~/.config/code-server/config.yaml
```

**Use strong password (20+ characters):**
```yaml
auth: password
password: ""
```

**Or use hashed password:**
```bash
# Generate hashed password
echo -n "YourPassword" | sha256sum

# Use in config
auth: password
hashed-password: ""
```

### 2. Enable Fail2Ban

```bash
# Install fail2ban
sudo apt install -y fail2ban

# Create nginx filter
sudo nano /etc/fail2ban/filter.d/nginx-code-server.conf
```

**Filter content:**
```ini
[Definition]
failregex = ^<HOST> -.*POST /login HTTP/1\.." (401|403)
ignoreregex =
```

**Configure jail:**
```bash
sudo nano /etc/fail2ban/jail.local
```

**Jail configuration:**
```ini
[nginx-code-server]
enabled = true
port = http,https
filter = nginx-code-server
logpath = /var/log/nginx/access.log
maxretry = 3
bantime = 3600
findtime = 600
```

**Restart fail2ban:**
```bash
sudo systemctl restart fail2ban
sudo fail2ban-client status nginx-code-server
```

### 3. Rate Limiting in Nginx

Already included in Nginx config above, but here's the key part:

```nginx
# Add to http block in /etc/nginx/nginx.conf
limit_req_zone $binary_remote_addr zone=codeserver:10m rate=10r/s;

# Add to location block
limit_req zone=codeserver burst=20 nodelay;
```

### 4. Firewall Configuration

```bash
# Allow only necessary ports
sudo ufw default deny incoming
sudo ufw default allow outgoing

sudo ufw allow 22/tcp    # SSH
sudo ufw allow 80/tcp    # HTTP
sudo ufw allow 443/tcp   # HTTPS

# If using non-standard SSH port
sudo ufw allow 2222/tcp

sudo ufw enable
sudo ufw status
```

### 5. Disable Password Authentication (Use OAuth)

**For GitHub OAuth:**

```bash
# Stop code-server
sudo systemctl stop code-server

# Update config
nano ~/.config/code-server/config.yaml
```

**OAuth config:**
```yaml
bind-addr: 0.0.0.0:8080
auth: none  # Will be handled by Nginx

# Let Nginx handle auth with oauth2-proxy
```

**Install oauth2-proxy:**
```bash
wget https://github.com/oauth2-proxy/oauth2-proxy/releases/download/v7.5.1/oauth2-proxy-v7.5.1.linux-amd64.tar.gz
tar -xzf oauth2-proxy-v7.5.1.linux-amd64.tar.gz
sudo mv oauth2-proxy-v7.5.1.linux-amd64/oauth2-proxy /usr/local/bin/
```

**Configure oauth2-proxy:**
```bash
nano ~/oauth2-proxy.cfg
```

**Config:**
```ini
provider = "github"
client_id = "your-github-oauth-app-id"
client_secret = "your-github-oauth-app-secret"
cookie_secret = "generate-with-openssl-rand-base64-32"
email_domains = ["*"]
github_org = "your-github-org"
github_team = "your-team"

http_address = "127.0.0.1:4180"
upstream = "http://127.0.0.1:8080"
```

---

## Monitoring Access

### Setup Access Logs

```bash
# Create log directory
sudo mkdir -p /var/log/code-server

# Update code-server service to log
sudo nano /etc/systemd/system/code-server.service
```

**Add logging:**
```ini
[Service]
StandardOutput=append:/var/log/code-server/output.log
StandardError=append:/var/log/code-server/error.log
```

### Monitor with Grafana/Prometheus

```bash
# Install node_exporter
wget https://github.com/prometheus/node_exporter/releases/download/v1.7.0/node_exporter-1.7.0.linux-amd64.tar.gz
tar -xzf node_exporter-1.7.0.linux-amd64.tar.gz
sudo mv node_exporter-1.7.0.linux-amd64/node_exporter /usr/local/bin/

# Create systemd service
sudo nano /etc/systemd/system/node_exporter.service
```

**Service:**
```ini
[Unit]
Description=Node Exporter
After=network.target

[Service]
Type=simple
ExecStart=/usr/local/bin/node_exporter
Restart=always

[Install]
WantedBy=multi-user.target
```

---

## Comparison Table

| Method | Difficulty | Security | Cost | Speed | Notes |
|--------|-----------|----------|------|-------|-------|
| Port Forwarding + DynDNS | Medium | Medium | Free | Fast | Requires router access |
| Cloudflare Tunnel | Easy | High | Free | Fast | DDoS protection included |
| Tailscale + Funnel | Easy | Very High | Free | Fast | Best security/convenience |
| Ngrok | Very Easy | Medium | Free/Paid | Medium | Good for testing |

---

## Recommended Setup (Most Secure)

**Best practice combination:**

1. **Primary access**: Tailscale (VPN for your devices)
2. **Public sharing**: Cloudflare Tunnel (when needed)
3. **Backup**: Port forwarding with fail2ban

```bash
# Install both
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up

# AND

wget https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb
sudo dpkg -i cloudflared-linux-amd64.deb
```

**Use Tailscale for personal access, Cloudflare for sharing with others.**

---

## Quick Test Script

```bash
# Test external access
nano ~/test-access.sh
```

**Script:**
```bash
#!/bin/bash

echo "Testing Code-Server Access..."

# Test local
echo "1. Local access:"
curl -I http://localhost:8080

# Test via domain
echo "2. Domain access:"
curl -I https://your-domain.duckdns.org

# Check SSL
echo "3. SSL certificate:"
echo | openssl s_client -connect your-domain.duckdns.org:443 2>/dev/null | openssl x509 -noout -dates

# Check firewall
echo "4. Firewall status:"
sudo ufw status

# Check fail2ban
echo "5. Fail2ban status:"
sudo fail2ban-client status nginx-code-server

echo "Done!"
```

```bash
chmod +x ~/test-access.sh
~/test-access.sh
```

---

## Troubleshooting

### Can't Access from Outside

```bash
# 1. Check if code-server is running
sudo systemctl status code-server

# 2. Check if nginx is running
sudo systemctl status nginx

# 3. Check firewall
sudo ufw status

# 4. Check if port is open (from external site: canyouseeme.org)
# Or use nmap from another computer
nmap -p 443 your-public-ip

# 5. Check nginx logs
sudo tail -f /var/log/nginx/error.log

# 6. Test local connection first
curl http://localhost:8080

# 7. Check DuckDNS is updating
cat ~/duckdns.log
```

### SSL Certificate Errors

```bash
# Check certificate
sudo certbot certificates

# Force renewal
sudo certbot renew --force-renewal

# Check Nginx SSL config
sudo nginx -t
```

### Cloudflare Tunnel Not Working

```bash
# Check tunnel status
sudo systemctl status cloudflared

# View logs
journalctl -u cloudflared -f

# Test tunnel connection
cloudflared tunnel info code-server

# Recreate tunnel if needed
cloudflared tunnel delete code-server
cloudflared tunnel create code-server-new
```

---

**Now you can code from anywhere! 🌍💻**


consider to to use code-server, tailscale and connect from a browser 



# ...existing code...

---

## Kubernetes Cluster Setup (K3s + ArgoCD + Istio + Prometheus)

### Prerequisites

```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install required tools
sudo apt install -y curl wget git apt-transport-https

# Check available resources (Istio needs at least 4GB RAM)
free -h
nproc
df -h
```

---

### Step 1: Install K3s

```bash
# Install without Traefik (conflicts with Istio)
curl -sfL https://get.k3s.io | sh -s - --disable traefik

# Verify k3s is running
sudo systemctl status k3s
kubectl get nodes
```

**Setup kubeconfig for your user:**
```bash
mkdir -p ~/.kube
sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
sudo chown $USER:$USER ~/.kube/config

# Add to shell profile
echo 'export KUBECONFIG=~/.kube/config' >> ~/.bashrc
source ~/.bashrc

# Verify
kubectl get nodes
kubectl get pods -A
```

---

### Step 2: Install kubectl + Helm

```bash
# kubectl (standalone for convenience)
sudo snap install kubectl --classic

# Helm
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# Verify
kubectl version --client
helm version
```

---

### Step 3: Install ArgoCD (GitOps)

```bash
# Create namespace
kubectl create namespace argocd

# Install ArgoCD
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Wait for pods to be ready
kubectl wait --for=condition=Ready pods --all -n argocd --timeout=300s

# Verify
kubectl get pods -n argocd
```

**Install ArgoCD CLI:**
```bash
# Download latest CLI
curl -sSL -o argocd https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
sudo install -m 555 argocd /usr/local/bin/argocd
rm argocd

# Verify
argocd version --client
```

**Get initial admin password:**
```bash
# Retrieve auto-generated password
argocd admin initial-password -n argocd
```

**Access ArgoCD UI:**
```bash
# Port-forward ArgoCD server
kubectl port-forward svc/argocd-server -n argocd 8088:443 --address=0.0.0.0 &

# Login via CLI
argocd login <tailscale-ip>:8088 \
  --username admin \
  --password <initial-password> \
  --insecure

# Change default password
argocd account update-password
```

**Connect your Git repository:**
```bash
# Add your GitHub repo to ArgoCD
argocd repo add https://github.com/<your-username>/k8s-homelab \
  --username <your-username> \
  --password <github-token>
```

**Create your first ArgoCD Application:**
```bash
argocd app create homelab \
  --repo https://github.com/<your-username>/k8s-homelab \
  --path ./apps \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace default \
  --sync-policy automated \
  --auto-prune \
  --self-heal
```

> **GitOps workflow:** Push YAML manifests to GitHub → ArgoCD detects changes → auto-syncs to cluster → drift is auto-corrected via self-heal.

---

### Step 4: Install Istio

```bash
# Download istioctl
curl -L https://istio.io/downloadIstio | sh -

# Add to PATH (replace version number accordingly)
export ISTIO_VERSION=$(ls -d istio-* | head -1)
export PATH=$PWD/$ISTIO_VERSION/bin:$PATH
echo "export PATH=~/$ISTIO_VERSION/bin:\$PATH" >> ~/.bashrc

# Pre-check cluster compatibility
istioctl x precheck

# Install with demo profile (all components for learning)
istioctl install --set profile=demo -y

# Verify
kubectl get pods -n istio-system
kubectl get svc -n istio-system
```

**Enable sidecar injection on default namespace:**
```bash
kubectl label namespace default istio-injection=enabled

# Verify
kubectl get namespace default --show-labels
```

**Install Istio addons (Kiali + Jaeger):**
```bash
kubectl apply -f ~/$ISTIO_VERSION/samples/addons/kiali.yaml
kubectl apply -f ~/$ISTIO_VERSION/samples/addons/jaeger.yaml

# Wait for pods
kubectl get pods -n istio-system --watch
```

---

### Step 5: Install Prometheus + Grafana Stack

```bash
# Add helm repo
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

# Create namespace
kubectl create namespace monitoring

# Install kube-prometheus-stack
helm install monitoring prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --set grafana.adminPassword=admin123 \
  --set prometheus.prometheusSpec.retention=7d

# Wait for all pods to be Running
kubectl get pods -n monitoring --watch
```

---

### Step 6: Access Dashboards via Tailscale

Run all port-forwards so dashboards are reachable via Tailscale IP from any device including Android browser:

```bash
# ArgoCD UI
kubectl port-forward svc/argocd-server -n argocd 8088:443 --address=0.0.0.0 &

# Grafana
kubectl port-forward -n monitoring svc/monitoring-grafana 3000:80 --address=0.0.0.0 &

# Prometheus UI
kubectl port-forward -n monitoring svc/monitoring-kube-prometheus-prometheus 9090:9090 --address=0.0.0.0 &

# Kiali - Istio mesh visualization
kubectl port-forward -n istio-system svc/kiali 20001:20001 --address=0.0.0.0 &
```

| Dashboard  | URL                              | Credentials          |
|------------|----------------------------------|----------------------|
| Code-Server| `http://<tailscale-ip>:8080`     | Your config password |
| ArgoCD     | `https://<tailscale-ip>:8088`    | admin / your-password|
| Grafana    | `http://<tailscale-ip>:3000`     | admin / admin123     |
| Prometheus | `http://<tailscale-ip>:9090`     | None                 |
| Kiali      | `http://<tailscale-ip>:20001`    | None                 |

---

### Step 7: Verify Full Stack

```bash
# Check all namespaces and pods
kubectl get pods -A

# Expected namespaces:
# - kube-system    → k3s system pods
# - argocd         → ArgoCD GitOps controllers
# - istio-system   → Istio + Kiali + Jaeger
# - monitoring     → Prometheus + Grafana

# Check node resource usage
kubectl top nodes
kubectl top pods -A
```

---

### Step 8: Deploy Sample App to Test Stack

```bash
# Deploy Istio bookinfo sample app
kubectl apply -f ~/$ISTIO_VERSION/samples/bookinfo/platform/kube/bookinfo.yaml

# Verify sidecars injected (should show 2/2 READY)
kubectl get pods

# Apply Istio gateway
kubectl apply -f ~/$ISTIO_VERSION/samples/bookinfo/networking/bookinfo-gateway.yaml

# Get ingress IP
kubectl get svc istio-ingressgateway -n istio-system

# Register app in ArgoCD for GitOps management
argocd app create bookinfo \
  --repo https://github.com/<your-username>/k8s-homelab \
  --path ./apps/bookinfo \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace default \
  --sync-policy automated \
  --self-heal
```

---

### Troubleshooting

```bash
# k3s service logs
journalctl -u k3s -f

# Pod not starting
kubectl describe pod <pod-name> -n <namespace>
kubectl logs <pod-name> -n <namespace>

# Istio sidecar not injecting
kubectl get namespace default --show-labels
# Must show: istio-injection=enabled

# ArgoCD app out of sync
argocd app sync homelab
argocd app get homelab

# ArgoCD logs
kubectl logs -n argocd -l app.kubernetes.io/name=argocd-server -f
```

---

### Learning Path

| Week | Focus | Practice |
|------|-------|----------|
| 1-2 | K3s + kubectl | Deploy workloads, understand cluster components |
| 3-4 | ArgoCD GitOps | Push manifests to Git, observe auto-sync, test drift correction |
| 5-6 | Istio basics | VirtualService, DestinationRule, canary deployments |
| 7-8 | Prometheus + Grafana | Custom dashboards, alerting rules |
| 9+  | Combine all | GitOps-managed Istio configs, Prometheus Istio metrics in Grafana |