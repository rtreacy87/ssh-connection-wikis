# Design Document: SSH Connection from Mac to Windows with WSL Ubuntu

## 1. Overview

This document outlines the design and implementation steps for establishing a secure SSH connection from a Mac laptop to a Windows machine running WSL (Windows Subsystem for Linux) with Ubuntu installed. This setup will enable remote access to the Linux environment running on Windows, allowing for file transfers, remote command execution, and development workflows.

## 2. System Architecture

### 2.1 Components

1. **Client System**: 
   - Mac laptop running macOS
   - SSH client (built into macOS)
   - Terminal application

2. **Server System**:
   - Windows machine with WSL2 installed
   - Ubuntu distribution running on WSL
   - SSH server (OpenSSH) running on Ubuntu
   - Port forwarding from Windows to WSL

3. **Network Components**:
   - Local network connectivity between Mac and Windows
   - Optional: Internet connectivity for remote access
   - Optional: Dynamic DNS for remote access with changing IP addresses

### 2.2 Architecture Diagram

```
+----------------+                   +----------------------------------+
|                |                   |                                  |
|  Mac Laptop    |                   |  Windows Machine                 |
|                |                   |                                  |
|  +----------+  |    SSH (Port 22)  |  +---------+     +----------+   |
|  |          |  |                   |  |         |     |          |   |
|  |  SSH     +------------------------> Windows  +-----> WSL      |   |
|  |  Client  |  |                   |  |  Port   |     | Ubuntu   |   |
|  |          |  |                   |  |  Forward|     | OpenSSH  |   |
|  +----------+  |                   |  +---------+     +----------+   |
|                |                   |                                  |
+----------------+                   +----------------------------------+
```

## 3. Implementation Plan

### 3.1 Windows WSL Setup

1. **Install WSL2 and Ubuntu**:
   - Ensure WSL2 is installed on the Windows machine
   - Install Ubuntu distribution from Microsoft Store or command line
   - Update Ubuntu packages

2. **Configure Ubuntu in WSL**:
   - Update package repositories
   - Install OpenSSH server
   - Configure SSH server for secure access
   - Generate SSH host keys
   - Set up user accounts and permissions

3. **Configure Windows Firewall**:
   - Create inbound rule for SSH port
   - Configure Windows Defender to allow SSH traffic
   - Set up port forwarding from Windows to WSL

### 3.2 Mac Client Setup

1. **SSH Client Configuration**:
   - Configure SSH client settings in `~/.ssh/config`
   - Generate and manage SSH keys
   - Set up key-based authentication

2. **Connection Testing**:
   - Verify connectivity from Mac to Windows
   - Test SSH authentication
   - Troubleshoot connection issues

### 3.3 Security Considerations

1. **Authentication**:
   - Use key-based authentication instead of passwords
   - Configure strong SSH keys (ED25519 or RSA 4096-bit)
   - Implement proper key management

2. **Firewall and Network Security**:
   - Restrict SSH access to specific IP addresses if possible
   - Configure fail2ban to prevent brute force attacks
   - Use non-standard SSH port (optional)

3. **SSH Hardening**:
   - Disable root login
   - Limit user access
   - Configure SSH timeout settings

## 4. Detailed Implementation Steps

### 4.1 Windows WSL Setup

#### 4.1.1 Install and Configure WSL2 with Ubuntu

```powershell
# Install WSL with Ubuntu
wsl --install -d Ubuntu

# Verify installation
wsl --list --verbose
```

#### 4.1.2 Configure Ubuntu SSH Server

```bash
# Update package repositories
sudo apt update
sudo apt upgrade -y

# Install OpenSSH server
sudo apt install openssh-server -y

# Edit SSH configuration
sudo nano /etc/ssh/sshd_config
```

Key SSH configuration settings:
```
# Allow key authentication
PubkeyAuthentication yes

# Disable password authentication (after setting up keys)
PasswordAuthentication no

# Disable root login
PermitRootLogin no

# Set SSH port (default 22)
Port 22
```

```bash
# Start SSH service
sudo service ssh start

# Enable SSH service to start automatically
sudo systemctl enable ssh
```

#### 4.1.3 Configure Windows Port Forwarding

```powershell
# Get WSL IP address
wsl hostname -I

# Set up port forwarding (replace 172.x.x.x with actual WSL IP)
netsh interface portproxy add v4tov4 listenaddress=0.0.0.0 listenport=22 connectaddress=172.x.x.x connectport=22

# Verify port forwarding
netsh interface portproxy show all

# Add firewall rule
New-NetFirewallRule -Name "SSH" -DisplayName "SSH" -Protocol TCP -LocalPort 22 -Action Allow -Direction Inbound
```

#### 4.1.4 Make Port Forwarding Persistent

Create a PowerShell script to run at startup:

```powershell
# Create a script to update port forwarding
$scriptContent = @'
# Remove existing port forwarding
netsh interface portproxy reset

# Get WSL IP address
$wslIP = (wsl hostname -I).Trim()

# Set up port forwarding
netsh interface portproxy add v4tov4 listenaddress=0.0.0.0 listenport=22 connectaddress=$wslIP connectport=22
'@

# Save script to file
$scriptContent | Out-File -FilePath "$env:USERPROFILE\update-wsl-ssh.ps1" -Encoding ASCII

# Create a scheduled task to run at startup
$action = New-ScheduledTaskAction -Execute "PowerShell.exe" -Argument "-ExecutionPolicy Bypass -File `"$env:USERPROFILE\update-wsl-ssh.ps1`""
$trigger = New-ScheduledTaskTrigger -AtStartup
$principal = New-ScheduledTaskPrincipal -UserId "$env:USERDOMAIN\$env:USERNAME" -LogonType S4U -RunLevel Highest
Register-ScheduledTask -TaskName "WSL SSH Port Forwarding" -Action $action -Trigger $trigger -Principal $principal
```

### 4.2 Mac Client Setup

#### 4.2.1 Generate SSH Keys

```bash
# Generate SSH key (ED25519 recommended)
ssh-keygen -t ed25519 -C "your_email@example.com"

# Alternative: RSA 4096-bit key (for older systems)
# ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

#### 4.2.2 Copy SSH Key to WSL Ubuntu

```bash
# Copy SSH key to WSL (replace with Windows IP address)
ssh-copy-id username@windows-ip-address

# Alternative manual method
cat ~/.ssh/id_ed25519.pub | ssh username@windows-ip-address "mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"
```

#### 4.2.3 Configure SSH Client

Create or edit `~/.ssh/config`:

```
Host windows-wsl
    HostName windows-ip-address
    User username
    Port 22
    IdentityFile ~/.ssh/id_ed25519
    ServerAliveInterval 60
    ServerAliveCountMax 3
```

#### 4.2.4 Test Connection

```bash
# Test SSH connection
ssh windows-wsl

# Test file transfer
scp ~/test-file.txt windows-wsl:~/
```

### 4.3 Additional Security Measures

#### 4.3.1 Install and Configure Fail2ban on WSL

```bash
# Install fail2ban
sudo apt install fail2ban -y

# Configure fail2ban
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
sudo nano /etc/fail2ban/jail.local
```

Edit the SSH section in jail.local:
```
[sshd]
enabled = true
port = 22
filter = sshd
logpath = /var/log/auth.log
maxretry = 5
bantime = 3600
```

```bash
# Start fail2ban
sudo service fail2ban start
sudo systemctl enable fail2ban
```

#### 4.3.2 Configure SSH Timeout Settings

Edit `/etc/ssh/sshd_config`:
```
# Client alive settings
ClientAliveInterval 300
ClientAliveCountMax 2

# Login grace time
LoginGraceTime 30s

# Max auth tries
MaxAuthTries 3
```

## 5. Troubleshooting

### 5.1 Common Issues and Solutions

1. **Connection Refused**:
   - Verify SSH service is running: `sudo service ssh status`
   - Check Windows firewall settings
   - Verify port forwarding: `netsh interface portproxy show all`

2. **Authentication Failures**:
   - Check SSH key permissions: `chmod 600 ~/.ssh/id_ed25519`
   - Verify authorized_keys file: `cat ~/.ssh/authorized_keys`
   - Check SSH server logs: `sudo cat /var/log/auth.log`

3. **WSL IP Address Changes**:
   - Verify the scheduled task is running to update port forwarding
   - Manually update port forwarding after IP changes

4. **Performance Issues**:
   - Consider using compression: Add `Compression yes` to SSH config
   - Use multiplexing for multiple connections

## 6. Maintenance and Updates

### 6.1 Regular Maintenance Tasks

1. **Update Systems**:
   - Keep Ubuntu updated: `sudo apt update && sudo apt upgrade -y`
   - Update WSL: `wsl --update`

2. **Monitor Logs**:
   - Check SSH logs: `sudo cat /var/log/auth.log`
   - Monitor failed login attempts: `sudo fail2ban-client status sshd`

3. **Backup Configuration**:
   - Backup SSH keys and configurations
   - Document port forwarding settings

## 7. Advanced Configuration (Optional)

### 7.1 Remote Access Over Internet

1. **Dynamic DNS Setup**:
   - Register with a dynamic DNS provider
   - Configure dynamic DNS client on Windows
   - Update SSH client configuration with domain name

2. **VPN Alternative**:
   - Consider using a VPN for secure remote access
   - Configure split tunneling if needed

### 7.2 X11 Forwarding

Enable X11 forwarding for GUI applications:

1. **On WSL Ubuntu**:
   Edit `/etc/ssh/sshd_config`:
   ```
   X11Forwarding yes
   X11DisplayOffset 10
   ```

2. **On Mac**:
   - Install XQuartz X11 server
   - Add to SSH config: `ForwardX11 yes`
   - Connect with: `ssh -X windows-wsl`

## 8. Conclusion

This design document provides a comprehensive approach to setting up a secure SSH connection from a Mac laptop to a Windows machine running WSL Ubuntu. By following these implementation steps, you can establish a robust remote access solution that enables efficient development workflows across different operating systems.

The solution leverages the strengths of each platform: macOS's Unix-based environment for client operations, Windows as the host system, and Ubuntu Linux for development tools and environments. The SSH connection provides a secure channel for remote access, file transfers, and command execution.
