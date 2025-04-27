# Troubleshooting and Maintaining Your SSH Setup

## What we'll do in this guide

In this guide, we'll cover how to troubleshoot common issues and maintain your SSH setup:

1. Diagnosing and fixing common connection problems
2. Handling WSL IP address changes
3. Reading and understanding SSH logs
4. Performing regular maintenance tasks
5. Backing up and restoring your SSH configuration

## Common connection issues and solutions

Let's look at some common SSH connection problems and how to fix them:

### Issue 1: "Connection refused" error

This usually means the SSH server isn't running or can't be reached.

**Solutions:**

1. Check if the SSH service is running in Ubuntu:
   ```bash
   sudo service ssh status
   ```

   If it's not running, start it:
   ```bash
   sudo service ssh start
   ```

2. Verify your port forwarding is set up:
   ```powershell
   netsh interface portproxy show all
   ```

   If it's not set up correctly, recreate it:
   ```powershell
   netsh interface portproxy add v4tov4 listenaddress=0.0.0.0 listenport=22 connectaddress=172.x.x.x connectport=22
   ```

3. Check your Windows Firewall:
   ```powershell
   Get-NetFirewallRule -Name "SSH" | Format-List -Property *
   ```

   If the rule doesn't exist or is disabled, create or enable it:
   ```powershell
   New-NetFirewallRule -Name "SSH" -DisplayName "SSH" -Protocol TCP -LocalPort 22 -Action Allow -Direction Inbound
   ```

### Issue 2: "Permission denied" error

This usually means there's an authentication problem.

**Solutions:**

1. Make sure you're using the correct username:
   ```bash
   ssh -v your-ubuntu-username@windows-ip-address
   ```

2. Check your SSH key permissions on your Mac:
   ```bash
   chmod 700 ~/.ssh
   chmod 600 ~/.ssh/id_ed25519
   ```

3. Verify your public key is in the authorized_keys file:
   ```bash
   ssh windows-wsl cat ~/.ssh/authorized_keys
   ```

   If your key isn't there, add it:
   ```bash
   ssh-copy-id your-ubuntu-username@windows-ip-address
   ```

4. Temporarily enable password authentication to troubleshoot:
   ```bash
   sudo nano /etc/ssh/sshd_config
   ```

   Change `PasswordAuthentication no` to `PasswordAuthentication yes`, then:
   ```bash
   sudo service ssh restart
   ```

### Issue 3: "Host key verification failed" error

This happens when the SSH server's fingerprint has changed.

**Solutions:**

1. Remove the old host key from your Mac:
   ```bash
   ssh-keygen -R windows-ip-address
   ```

2. Try connecting again - you'll be asked to confirm the new fingerprint.

### Issue 4: Slow SSH connections

**Solutions:**

1. Enable compression:
   ```bash
   nano ~/.ssh/config
   ```

   Add `Compression yes` to your host configuration.

2. Set up multiplexing (as described in the advanced configuration guide).

3. Check your internet connection speed.

## Handling WSL IP address changes

WSL's IP address can change when your computer restarts, which can break your port forwarding.

### Solution 1: Use the automatic script

In a previous guide, we created a script to automatically update port forwarding. Make sure it's running at startup:

1. Check if the scheduled task is running:
   ```powershell
   Get-ScheduledTask -TaskName "WSL SSH Port Forwarding" | Format-List -Property *
   ```

2. If it's not running, recreate it:
   ```powershell
   $action = New-ScheduledTaskAction -Execute "PowerShell.exe" -Argument "-ExecutionPolicy Bypass -File `"$env:USERPROFILE\update-wsl-ssh.ps1`""
   $trigger = New-ScheduledTaskTrigger -AtStartup
   $principal = New-ScheduledTaskPrincipal -UserId "$env:USERDOMAIN\$env:USERNAME" -LogonType S4U -RunLevel Highest
   Register-ScheduledTask -TaskName "WSL SSH Port Forwarding" -Action $action -Trigger $trigger -Principal $principal
   ```

### Solution 2: Manually update port forwarding

If the automatic script isn't working, you can manually update port forwarding:

1. Find your WSL IP address:
   ```powershell
   wsl hostname -I
   ```

2. Update port forwarding:
   ```powershell
   netsh interface portproxy reset
   netsh interface portproxy add v4tov4 listenaddress=0.0.0.0 listenport=22 connectaddress=172.x.x.x connectport=22
   ```

   Replace `172.x.x.x` with your actual WSL IP address.

### Solution 3: Use a static IP for WSL

You can configure WSL to use a static IP address:

1. Create or edit the WSL configuration file:
   ```powershell
   notepad "$env:USERPROFILE\.wslconfig"
   ```

2. Add these lines:
   ```
   [wsl2]
   dhcp=false
   ipv6=false
   ```

3. Create a .wsl_ip file in your Ubuntu home directory:
   ```bash
   echo "ip addr add 192.168.50.2/24 broadcast 192.168.50.255 dev eth0" > ~/.wsl_ip
   echo "ip route add default via 192.168.50.1 dev eth0" >> ~/.wsl_ip
   chmod +x ~/.wsl_ip
   ```

4. Edit your .bashrc file to run this script at startup:
   ```bash
   echo "sudo ~/.wsl_ip" >> ~/.bashrc
   ```

5. Update your port forwarding to use this static IP:
   ```powershell
   netsh interface portproxy reset
   netsh interface portproxy add v4tov4 listenaddress=0.0.0.0 listenport=22 connectaddress=192.168.50.2 connectport=22
   ```

6. Restart WSL:
   ```powershell
   wsl --shutdown
   wsl
   ```

## Reading and understanding SSH logs

SSH logs can help you diagnose problems:

### Checking SSH server logs

1. Connect to your Ubuntu system:
   ```bash
   ssh windows-wsl
   ```

2. View the authentication log:
   ```bash
   sudo cat /var/log/auth.log | grep sshd
   ```

   This shows login attempts, successful connections, and errors.

3. For more detailed logging, edit the SSH configuration:
   ```bash
   sudo nano /etc/ssh/sshd_config
   ```

   Add or modify:
   ```
   LogLevel VERBOSE
   ```

   Then restart SSH:
   ```bash
   sudo service ssh restart
   ```

### Checking SSH client logs

When connecting from your Mac, use the verbose option to see detailed logs:

```bash
ssh -v windows-wsl
```

For even more detail, use `-vv` or `-vvv`:

```bash
ssh -vvv windows-wsl
```

### Common log messages and what they mean

- **"Failed password"**: Someone tried to log in with an incorrect password
- **"Connection closed by"**: The connection was terminated
- **"Invalid user"**: Someone tried to log in with a non-existent username
- **"error: maximum authentication attempts exceeded"**: Too many failed login attempts
- **"Accepted publickey for"**: Successful login using an SSH key
- **"Disconnected from"**: Normal disconnection when you log out

## Regular maintenance tasks

To keep your SSH setup running smoothly, perform these maintenance tasks regularly:

### 1. Update your systems

#### Ubuntu:
```bash
sudo apt update
sudo apt upgrade -y
```

#### WSL:
```powershell
wsl --update
```

### 2. Check SSH server configuration

Periodically review your SSH server configuration for security:

```bash
sudo nano /etc/ssh/sshd_config
```

Make sure these settings are secure:
- `PasswordAuthentication no`
- `PermitRootLogin no`
- `X11Forwarding yes` (only if you need it)

### 3. Monitor failed login attempts

Check for unauthorized access attempts:

```bash
sudo cat /var/log/auth.log | grep "Failed password"
```

Check fail2ban status:

```bash
sudo fail2ban-client status sshd
```

### 4. Rotate SSH keys

For maximum security, periodically create new SSH keys:

1. Generate a new key:
   ```bash
   ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519_new -C "your_email@example.com"
   ```

2. Add the new key to your Ubuntu system:
   ```bash
   ssh-copy-id -i ~/.ssh/id_ed25519_new your-ubuntu-username@windows-ip-address
   ```

3. Update your SSH config to use the new key:
   ```bash
   nano ~/.ssh/config
   ```

   Change `IdentityFile ~/.ssh/id_ed25519` to `IdentityFile ~/.ssh/id_ed25519_new`

4. Test the new key:
   ```bash
   ssh windows-wsl
   ```

5. Once confirmed working, remove the old key from authorized_keys:
   ```bash
   ssh windows-wsl
   nano ~/.ssh/authorized_keys
   ```

   Delete the line with your old public key.

### 5. Check port forwarding

Verify that port forwarding is still working:

```powershell
netsh interface portproxy show all
```

If it's not set up correctly, recreate it using the steps in the "Handling WSL IP address changes" section.

## Backing up and restoring your SSH configuration

It's important to back up your SSH configuration so you can restore it if needed.

### Backing up SSH configuration

#### On your Mac:

1. Back up your SSH directory:
   ```bash
   cp -r ~/.ssh ~/ssh_backup_$(date +%Y%m%d)
   ```

2. Back up your SSH config file separately:
   ```bash
   cp ~/.ssh/config ~/ssh_config_backup_$(date +%Y%m%d)
   ```

#### On your Ubuntu system:

1. Back up your SSH server configuration:
   ```bash
   sudo cp /etc/ssh/sshd_config ~/sshd_config_backup_$(date +%Y%m%d)
   ```

2. Back up your authorized_keys file:
   ```bash
   cp ~/.ssh/authorized_keys ~/authorized_keys_backup_$(date +%Y%m%d)
   ```

#### On your Windows system:

1. Back up your port forwarding script:
   ```powershell
   Copy-Item "$env:USERPROFILE\update-wsl-ssh.ps1" "$env:USERPROFILE\update-wsl-ssh_backup_$(Get-Date -Format 'yyyyMMdd').ps1"
   ```

### Restoring SSH configuration

#### On your Mac:

1. Restore your SSH directory:
   ```bash
   cp -r ~/ssh_backup_YYYYMMDD ~/.ssh
   ```

2. Fix permissions:
   ```bash
   chmod 700 ~/.ssh
   chmod 600 ~/.ssh/id_ed25519
   chmod 600 ~/.ssh/config
   ```

#### On your Ubuntu system:

1. Restore your SSH server configuration:
   ```bash
   sudo cp ~/sshd_config_backup_YYYYMMDD /etc/ssh/sshd_config
   sudo service ssh restart
   ```

2. Restore your authorized_keys file:
   ```bash
   cp ~/authorized_keys_backup_YYYYMMDD ~/.ssh/authorized_keys
   chmod 600 ~/.ssh/authorized_keys
   ```

#### On your Windows system:

1. Restore your port forwarding script:
   ```powershell
   Copy-Item "$env:USERPROFILE\update-wsl-ssh_backup_YYYYMMDD.ps1" "$env:USERPROFILE\update-wsl-ssh.ps1"
   ```

2. Run the script to restore port forwarding:
   ```powershell
   PowerShell.exe -ExecutionPolicy Bypass -File "$env:USERPROFILE\update-wsl-ssh.ps1"
   ```

## What we've accomplished

Great job! You've now learned how to:
- Diagnose and fix common SSH connection problems
- Handle WSL IP address changes
- Read and understand SSH logs
- Perform regular maintenance tasks
- Back up and restore your SSH configuration

With these skills, you can keep your SSH connection running smoothly and troubleshoot any issues that arise.

## What's next?

In the final guide, we'll explore practical use cases and workflows for your Mac-to-WSL SSH connection.

Move on to the next guide: "Practical Workflows: Using Your Mac-to-WSL SSH Connection"
