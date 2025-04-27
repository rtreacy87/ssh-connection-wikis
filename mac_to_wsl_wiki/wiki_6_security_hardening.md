# Securing Your SSH Connection

## What we'll do in this guide

In this guide, we'll improve the security of your SSH connection by:

1. Disabling password authentication
2. Setting up fail2ban to protect against brute force attacks
3. Configuring SSH timeout settings
4. Implementing other security best practices

## Why security matters

Now that you have a working SSH connection, it's important to make it secure. An unsecured SSH server can be a target for attackers who might try to:

- Guess your password through brute force attempts
- Access your files without permission
- Use your computer for malicious purposes

The steps in this guide will help protect your system from these threats.

## Step 1: Disable password authentication

Now that you're using SSH keys, we can disable password authentication entirely. This means that only someone with your private key can connect, even if they know your password.

1. Connect to your Ubuntu system:
   ```bash
   ssh windows-wsl
   ```

2. Open the SSH server configuration file:
   ```bash
   sudo nano /etc/ssh/sshd_config
   ```

3. Find the line with `PasswordAuthentication` and change it to:
   ```
   PasswordAuthentication no
   ```

   If the line is commented out (starts with #), remove the # symbol.

4. Save the file:
   - Press Ctrl+O to save
   - Press Enter to confirm
   - Press Ctrl+X to exit nano

5. Restart the SSH service to apply the changes:
   ```bash
   sudo service ssh restart
   ```

6. **Important**: Don't log out yet! Open a new Terminal window on your Mac and verify that you can still connect:
   ```bash
   ssh windows-wsl
   ```

   If you can connect, your key-based authentication is working correctly.

## Step 2: Install and configure fail2ban

Fail2ban is a security tool that monitors login attempts and blocks IP addresses that show malicious signs, like too many failed login attempts.

1. Install fail2ban:
   ```bash
   sudo apt update
   sudo apt install fail2ban -y
   ```

2. Create a copy of the configuration file:
   ```bash
   sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
   ```

3. Edit the configuration:
   ```bash
   sudo nano /etc/fail2ban/jail.local
   ```

4. Find the `[sshd]` section and make sure it looks like this:
   ```
   [sshd]
   enabled = true
   port = 22
   filter = sshd
   logpath = /var/log/auth.log
   maxretry = 5
   bantime = 3600
   ```

   This configuration:
   - Enables fail2ban for SSH
   - Allows 5 retry attempts before banning
   - Bans the IP address for 1 hour (3600 seconds) after too many failed attempts

5. Save the file (Ctrl+O, Enter, Ctrl+X)

6. Start fail2ban and enable it to start automatically:
   ```bash
   sudo service fail2ban start
   sudo systemctl enable fail2ban
   ```

7. Check that fail2ban is running:
   ```bash
   sudo service fail2ban status
   ```

   You should see "Active: active (running)" in the output.

## Step 3: Configure SSH timeout settings

Setting timeouts helps close idle connections, which reduces the risk of someone accessing an unattended session.

1. Edit the SSH server configuration:
   ```bash
   sudo nano /etc/ssh/sshd_config
   ```

2. Add or modify these lines:
   ```
   # Close idle sessions after 5 minutes
   ClientAliveInterval 300
   ClientAliveCountMax 2

   # Give users 30 seconds to log in
   LoginGraceTime 30s

   # Limit authentication attempts per connection
   MaxAuthTries 3
   ```

3. Save the file (Ctrl+O, Enter, Ctrl+X)

4. Restart the SSH service:
   ```bash
   sudo service ssh restart
   ```

## Step 4: Additional security measures

Here are some more security improvements you can implement:

### 4.1: Restrict SSH access to specific users

1. Edit the SSH configuration:
   ```bash
   sudo nano /etc/ssh/sshd_config
   ```

2. Add this line (replace "yourusername" with your Ubuntu username):
   ```
   AllowUsers yourusername
   ```

   This restricts SSH access to only the specified users.

3. Save and restart SSH:
   ```bash
   sudo service ssh restart
   ```

### 4.2: Secure SSH key permissions on your Mac

Make sure your SSH keys have the correct permissions:

1. On your Mac, open Terminal and run:
   ```bash
   chmod 700 ~/.ssh
   chmod 600 ~/.ssh/id_ed25519
   chmod 600 ~/.ssh/config
   ```

   This ensures that only you can read your private key and SSH configuration.

### 4.3: Use a non-standard SSH port (optional)

Using a non-standard port can reduce automated attacks, but it's not a substitute for other security measures.

1. On your Ubuntu system, edit the SSH configuration:
   ```bash
   sudo nano /etc/ssh/sshd_config
   ```

2. Change the Port line to a number between 1024 and 65535 (e.g., 2222):
   ```
   Port 2222
   ```

3. Save and restart SSH:
   ```bash
   sudo service ssh restart
   ```

4. Update your Windows port forwarding (in PowerShell as Administrator):
   ```powershell
   # Remove the old rule
   netsh interface portproxy delete v4tov4 listenport=22 listenaddress=0.0.0.0

   # Add the new rule (replace 172.x.x.x with your Ubuntu IP)
   netsh interface portproxy add v4tov4 listenaddress=0.0.0.0 listenport=2222 connectaddress=172.x.x.x connectport=2222
   ```

5. Update your Windows Firewall rule:
   ```powershell
   # Remove the old rule
   Remove-NetFirewallRule -Name "SSH"

   # Add the new rule
   New-NetFirewallRule -Name "SSH" -DisplayName "SSH" -Protocol TCP -LocalPort 2222 -Action Allow -Direction Inbound
   ```

6. Update your Mac's SSH config:
   ```bash
   nano ~/.ssh/config
   ```

   Change the Port line to match:
   ```
   Port 2222
   ```

7. Update your port forwarding script:
   ```powershell
   $scriptContent = @'
   # Remove existing port forwarding
   netsh interface portproxy reset

   # Get WSL IP address
   $wslIP = (wsl hostname -I).Trim()

   # Set up port forwarding
   netsh interface portproxy add v4tov4 listenaddress=0.0.0.0 listenport=2222 connectaddress=$wslIP connectport=2222
   '@

   $scriptContent | Out-File -FilePath "$env:USERPROFILE\update-wsl-ssh.ps1" -Encoding ASCII
   ```

## Step 5: Test your security settings

Let's make sure everything still works with the new security settings:

1. Try connecting from your Mac:
   ```bash
   ssh windows-wsl
   ```

2. You should connect successfully using your SSH key.

3. If you changed the port, you might need to specify it the first time:
   ```bash
   ssh -p 2222 windows-wsl
   ```

   After updating your SSH config, you can just use `ssh windows-wsl` again.

## Troubleshooting

If you can't connect after making these changes:

1. Check the SSH server logs for error messages:
   ```bash
   sudo cat /var/log/auth.log | grep sshd
   ```

2. Make sure the SSH service is running:
   ```bash
   sudo service ssh status
   ```

3. Verify your port forwarding is set up correctly:
   ```powershell
   netsh interface portproxy show all
   ```

4. If you're completely locked out, you can access your Ubuntu system directly from Windows and revert your changes:
   - Open Ubuntu from the Start menu
   - Edit the SSH configuration: `sudo nano /etc/ssh/sshd_config`
   - Change the settings back and restart SSH: `sudo service ssh restart`

## What we've accomplished

Great job! You've now:
- Disabled password authentication for better security
- Set up fail2ban to protect against brute force attacks
- Configured SSH timeout settings
- Implemented additional security measures

Your SSH connection is now much more secure against common threats.

## What's next?

In the next guide, we'll explore advanced SSH features that can make your remote work more productive.

Move on to the next guide: "Advanced SSH Features and Configuration"
