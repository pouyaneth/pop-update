Pipe Network POP Node Upgrade Guide: v0.3.0 → v0.3.1
This guide helps you upgrade your existing Pipe Network POP node from version 0.3.0 to 0.3.1, adapting the process for different installation directories and configurations.
⚠️ Important Notes

Configuration Change: v0.3.1 uses a config file instead of command-line arguments
Identity Registration: You may need to re-register your node identity
Backup Recommended: Always backup your current setup before upgrading

Prerequisites

Existing Pipe Network POP node running v0.3.0 or earlier
Root or sudo access to your VPS
Your original invite code

Step 1: Locate Your Current Installation
First, find where your POP node is currently installed:
bash# Find your pop executable
find / -name "pop" -type f 2>/dev/null

# Check running processes
ps aux | grep pop

# Check systemd services
sudo systemctl list-units | grep -i pop
Common installation paths:

/opt/popcache/
/root/pipenetwork/
/home/user/pipenetwork/

For this guide, we'll use /root/pipenetwork/ as an example. Replace with your actual path.
Step 2: Stop the Service and Remove Old Files
bash# Navigate to your installation directory
cd /root/pipenetwork/

# Stop your service (replace 'pipe-pop' with your actual service name)
sudo systemctl stop pipe-pop

# Remove old files
rm -rf pop pop-v0.3.0-linux-*.tar.gz

# Clear old logs
rm -f logs/*
Step 3: Download and Extract v0.3.1

Download the new version:

Visit the Pipe Network download page
Enter your invite code and download pop-v0.3.1-linux-x64.tar.gz
Upload to your installation directory using SFTP


Extract the new version:

bashcd /root/pipenetwork/
sudo tar -xzf pop-v0.3.1-linux-*.tar.gz
chmod +x pop
Note: You may see a harmless warning about LIBARCHIVE.xattr.com.apple.provenance - this can be ignored.
Step 4: Create Configuration File
v0.3.1 uses a JSON configuration file instead of command-line arguments.
bash# Create default config
./pop create-config

# Edit the configuration
nano config.json
Sample configuration:
json{
  "pop_name": "your-node-name",
  "pop_location": "Your City, Country",
  "invite_code": "YOUR_INVITE_CODE_HERE",
  "server": {
    "host": "0.0.0.0",
    "port": 443,
    "http_port": 80,
    "workers": 40
  },
  "cache_config": {
    "memory_cache_size_mb": 512,
    "disk_cache_path": "./download_cache",
    "disk_cache_size_gb": 250,
    "default_ttl_seconds": 86400,
    "respect_origin_headers": true,
    "max_cacheable_size_mb": 1024
  },
  "api_endpoints": {
    "base_url": "https://dataplane2.pipenetwork.com"
  },
  "identity_config": {
    "node_name": "your-unique-node-name",
    "name": "Your Name",
    "email": "your@email.com",
    "website": "https://your-website.com",
    "twitter": "your-twitter",
    "discord": "your-discord",
    "telegram": "your-telegram",
    "solana_pubkey": "YOUR_SOLANA_PUBKEY_IF_YOU_HAVE_ONE"
  }
}
Key configuration notes:

Use your existing cache directory path
Adjust disk_cache_size_gb to match your previous --max-disk setting
All string values must be in double quotes
Use a unique node_name to avoid conflicts

Step 5: Update Service Configuration
v0.3.1 uses a simplified service configuration without command-line arguments.
bash# Edit your systemd service file
sudo nano /etc/systemd/system/pipe-pop.service
Updated service file:
ini[Unit]
Description=Pipe POP Node Service
After=network.target
Wants=network-online.target

[Service]
User=root
Group=root
WorkingDirectory=/root/pipenetwork
ExecStart=/root/pipenetwork/pop
Restart=always
RestartSec=5
LimitNOFILE=65536
LimitNPROC=4096
StandardOutput=journal
StandardError=journal
SyslogIdentifier=dcdn-node

[Install]
WantedBy=multi-user.target
Key changes:

Remove all command-line arguments from ExecStart
Update WorkingDirectory to your installation path
Update ExecStart path to your pop executable

Step 6: Start the Updated Service
bash# Create logs directory
mkdir -p logs

# Reload systemd and start service
sudo systemctl daemon-reload
sudo systemctl enable pipe-pop
sudo systemctl start pipe-pop
Step 7: Verify Installation
Check service status:
bashsudo systemctl status pipe-pop
Monitor logs:
bashsudo journalctl -u pipe-pop -f
Look for these success indicators:

✅ 🔄 Registering PoP node with: Name='...', Location='...', Invite Code='...'
✅ 💾 Disk cache size limit set to XXX GB
✅ 🔌 Successfully bound to port 443
✅ Successfully bound to HTTP port 80
✅ HTTP server task was spawned successfully

Step 8: Get Your Node ID and Check Dashboard
bash# Check your node state
cat .pop_state.json | grep pop_id
Visit your dashboard: https://dashboard.testnet.pipe.network/node/YOUR_NODE_ID
Troubleshooting
Identity Registration Issues
If you see warnings like Failed to find identity by name, your node name might be taken or invalid:

Change node name:

bashnano config.json
# Change "node_name" to something unique
sudo systemctl restart pipe-pop

Force re-registration (if needed):

bashsudo systemctl stop pipe-pop
mv .pop_state.json .pop_state.json.backup
sudo systemctl start pipe-pop
Service Won't Start
Check for configuration errors:
bash# Test config validity
./pop --help

# Check detailed logs
sudo journalctl -u pipe-pop -n 50
Dashboard Shows No Data
This is normal for new/updated nodes:

Identity info: Takes 5-30 minutes to sync
Performance metrics: Appears when network starts routing traffic (hours to days)

Common Migration Scenarios
From /opt/popcache/ Installation
Replace all paths in commands:

/root/pipenetwork/ → /opt/popcache/
Update service file WorkingDirectory and ExecStart paths accordingly

Different Service Names
Replace pipe-pop with your actual service name:

popcache
pipenetwork
pop-node

Custom Cache Directories
Update disk_cache_path in config.json to match your existing cache location.
Cleanup
After successful upgrade:
bash# Remove installation archive
rm -f pop-v0.3.1-linux-*.tar.gz

# Keep backup of old state file for reference
# rm -f .pop_state.json.backup  # Only if you're sure everything works
What's New in v0.3.1

Config-based setup: Cleaner than command-line arguments
Enhanced identity management: Better dashboard integration
Improved error handling: More detailed logs and diagnostics
Optimized performance: Better resource management

Support

Pipe Network Discord: Join for community support
Dashboard: https://dashboard.testnet.pipe.network/
Issues: Report problems to Pipe Network team


Note: This guide is based on community experience upgrading from v0.3.0 to v0.3.1. Always refer to official Pipe Network documentation for the latest information.
