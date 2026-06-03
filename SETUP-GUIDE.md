# 📋 Complete Lab Setup Guide

## Requirements
- Computer with 8GB+ RAM
- 50GB free disk space
- VirtualBox installed
- Internet connection for downloads

## Step 1: Create Virtual Machines

### Kali Linux VM
1. Download Kali Linux ISO from official website
2. Create VM in VirtualBox:
   - RAM: 4GB
   - Disk: 30GB
   - Network: NAT Network

### Ubuntu VM
1. Download Ubuntu 22.04 LTS ISO
2. Create VM:
   - RAM: 2GB
   - Disk: 20GB
   - Network: NAT Network (same as Kali)

## Step 2: Install Splunk on Kali

1. Download Splunk Enterprise from splunk.com
2. Copy to Kali VM
3. Install with: `sudo dpkg -i splunk.deb`
4. Start Splunk: `sudo /opt/splunk/bin/splunk start`
5. Access Splunk web: `http://[kali-ip]:8000`

## Step 3: Configure Ubuntu Target

1. Install SSH: `sudo apt install openssh-server`
2. Create test user: `sudo useradd -m testuser`
3. Set password: `sudo passwd testuser` (use 'password123')

## Step 4: Forward Logs to Splunk

1. Install Splunk Universal Forwarder on Ubuntu
2. Configure: `add forward-server [kali-ip]:9997`
3. Monitor auth.log: `add monitor /var/log/auth.log`

## Step 5: Simulate Attack

On Kali, run:
```bash
hydra -l testuser -P /usr/share/wordlists/rockyou.txt [ubuntu-ip] ssh
