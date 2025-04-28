# SSH Connection Quick Reference Guide

This quick reference guide provides commands, configurations, and troubleshooting tips for your Mac-to-Windows WSL SSH connection.

## Command Cheat Sheet

### SSH Commands

| Command | Description |
|---------|-------------|
| `ssh windows-wsl` | Connect to your WSL Ubuntu system |
| `ssh -p 2222 windows-wsl` | Connect using a custom port |
| `ssh -X windows-wsl` | Connect with X11 forwarding |
| `ssh -L 8080:localhost:80 windows-wsl` | Create a local port forward |
| `ssh -R 8000:localhost:8000 windows-wsl` | Create a remote port forward |
| `ssh -v windows-wsl` | Connect with verbose output for troubleshooting |
| `ssh windows-wsl "command"` | Run a command on the remote system |

### File Transfer Commands

| Command | Description |
|---------|-------------|
| `scp file.txt windows-wsl:~/` | Copy a file to your Ubuntu system |
| `scp windows-wsl:~/file.txt ./` | Copy a file from your Ubuntu system |
| `scp -r folder/ windows-wsl:~/` | Copy a directory to your Ubuntu system |
| `rsync -avz folder/ windows-wsl:~/folder/` | Sync a directory to your Ubuntu system |
| `sftp windows-wsl` | Start an interactive SFTP session |

### WSL Commands (Windows PowerShell)

| Command | Description |
|---------|-------------|
| `wsl --list --verbose` | List installed WSL distributions |
| `wsl --shutdown` | Shut down all WSL instances |
| `wsl --update` | Update WSL |
| `wsl hostname -I` | Get WSL IP address |
| `wsl -d Ubuntu -u root service ssh status` | Check SSH service status |

### Port Forwarding Commands (Windows PowerShell)

| Command | Description |
|---------|-------------|
| `netsh interface portproxy show all` | Show all port forwarding rules |
| `netsh interface portproxy add v4tov4 listenaddress=0.0.0.0 listenport=22 connectaddress=172.x.x.x connectport=22` | Add port forwarding rule |
| `netsh interface portproxy reset` | Remove all port forwarding rules |

### Ubuntu Service Commands

| Command | Description |
|---------|-------------|
| `sudo service ssh status` | Check SSH service status |
| `sudo service ssh start` | Start SSH service |
| `sudo service ssh restart` | Restart SSH service |
| `sudo systemctl enable ssh` | Enable SSH service at startup |

## Configuration File Templates

### SSH Config (~/.ssh/config on Mac)

```
# Default settings for all hosts
Host *
    ServerAliveInterval 60
    ServerAliveCountMax 3
    ForwardAgent no

# Local WSL connection
Host windows-wsl
    HostName 192.168.1.5
    User your-ubuntu-username
    Port 22
    IdentityFile ~/.ssh/id_ed25519
    ForwardX11 yes
    ControlMaster auto
    ControlPath ~/.ssh/control:%h:%p:%r
    ControlPersist 1h

# Remote WSL connection
Host windows-wsl-remote
    HostName your-dynamic-dns-hostname.com
    User your-ubuntu-username
    Port 22
    IdentityFile ~/.ssh/id_ed25519
    ServerAliveInterval 60
    ServerAliveCountMax 3
```

### SSH Server Config (/etc/ssh/sshd_config on Ubuntu)

```
# Basic SSH server settings
Port 22
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
X11Forwarding yes
X11DisplayOffset 10

# Security settings
ClientAliveInterval 300
ClientAliveCountMax 2
LoginGraceTime 30s
MaxAuthTries 3

# Limit users who can connect
AllowUsers your-ubuntu-username
```

### Port Forwarding Script (update-wsl-ssh.ps1 on Windows)

```powershell
# Remove existing port forwarding
netsh interface portproxy reset

# Get WSL IP address
$wslIP = (wsl hostname -I).Trim()

# Set up port forwarding
netsh interface portproxy add v4tov4 listenaddress=0.0.0.0 listenport=22 connectaddress=$wslIP connectport=22
```

## Troubleshooting Flowchart

```
Connection issue
│
├── "Connection refused"
│   ├── Is SSH service running?
│   │   ├── Yes → Check port forwarding
│   │   └── No → Start SSH service: sudo service ssh start
│   │
│   ├── Is port forwarding set up?
│   │   ├── Yes → Check Windows Firewall
│   │   └── No → Set up port forwarding
│   │
│   └── Is Windows Firewall allowing connections?
│       ├── Yes → Check if WSL IP changed
│       └── No → Add firewall rule
│
├── "Permission denied"
│   ├── Using correct username?
│   │   ├── Yes → Check SSH key permissions
│   │   └── No → Use correct username
│   │
│   ├── SSH key permissions correct?
│   │   ├── Yes → Check authorized_keys
│   │   └── No → Fix permissions: chmod 600 ~/.ssh/id_ed25519
│   │
│   └── Public key in authorized_keys?
│       ├── Yes → Try password authentication temporarily
│       └── No → Add key: ssh-copy-id username@ip-address
│
├── "Host key verification failed"
│   └── Remove old host key: ssh-keygen -R ip-address
│
└── Slow connection
    ├── Enable compression: Compression yes
    ├── Set up multiplexing
    └── Check internet connection
```

## Security Checklist

### SSH Security
- [ ] Disable password authentication
- [ ] Use strong SSH keys (ED25519 or RSA 4096-bit)
- [ ] Set proper permissions on SSH keys and config files
- [ ] Install and configure fail2ban
- [ ] Set SSH timeout settings
- [ ] Limit SSH access to specific users
- [ ] Keep systems updated
- [ ] Regularly check logs for unauthorized access attempts
- [ ] Use a non-standard SSH port (optional)
- [ ] Back up SSH configuration

### Windows Host Security
- [ ] Use a strong Windows password
- [ ] Enable Windows Hello biometric authentication
- [ ] Configure account lockout policies
- [ ] Keep Windows updated
- [ ] Enable Windows Security (antivirus)
- [ ] Configure Windows Firewall
- [ ] Enable controlled folder access (ransomware protection)
- [ ] Secure Remote Desktop (if used)
- [ ] Enable Network Level Authentication
- [ ] Limit users with remote access privileges

## Useful Resources

### Official Documentation

- [Windows Subsystem for Linux Documentation](https://docs.microsoft.com/en-us/windows/wsl/)
- [OpenSSH Documentation](https://www.openssh.com/manual.html)
- [Ubuntu Documentation](https://help.ubuntu.com/)

### Tools

- [XQuartz](https://www.xquartz.org/) - X11 server for macOS
- [Visual Studio Code Remote - SSH](https://code.visualstudio.com/docs/remote/ssh) - Remote development over SSH
- [No-IP](https://www.noip.com/) - Dynamic DNS service
- [fail2ban](https://www.fail2ban.org/) - Intrusion prevention tool

### Further Reading

- [SSH Mastery](https://www.tiltedwindmillpress.com/product/ssh-mastery-2nd-edition/) by Michael W. Lucas
- [Linux Command Line and Shell Scripting Bible](https://www.wiley.com/en-us/Linux+Command+Line+and+Shell+Scripting+Bible%2C+4th+Edition-p-9781119700913) by Richard Blum and Christine Bresnahan
- [Pro Git](https://git-scm.com/book/en/v2) by Scott Chacon and Ben Straub
