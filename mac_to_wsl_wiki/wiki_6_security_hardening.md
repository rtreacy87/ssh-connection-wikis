# Securing Your SSH Connection

## What we'll do in this guide

In this guide, we'll improve the security of your SSH connection by:

1. Disabling password authentication
2. Setting up fail2ban to protect against brute force attacks
3. Configuring SSH timeout settings
4. Implementing other security best practices
5. Securing the Windows host machine running WSL

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

## Step 6: Secure your Windows host machine

Since your WSL environment runs on a Windows host, it's equally important to secure the Windows machine itself. If an attacker gains access to your Windows system, they could potentially access your WSL environment regardless of your SSH security settings.

### 6.1: Strengthen Windows login security

1. Use a strong password for your Windows user account:
   - At least 12 characters long
   - Mix of uppercase and lowercase letters, numbers, and special characters
   - Avoid common words or easily guessable information

2. Enable Windows Hello for biometric authentication (if your hardware supports it):
   - Open Windows Settings > Accounts > Sign-in options
   - Set up fingerprint, facial recognition, or PIN as additional authentication methods
   - This provides an extra layer of security beyond just passwords

3. Enable account lockout policies:
   - Open Local Security Policy (search for "secpol.msc" in the Start menu)
   - Navigate to Account Policies > Account Lockout Policy
   - Set "Account lockout threshold" to 5 attempts
   - Set "Account lockout duration" to 30 minutes
   - Set "Reset account lockout counter after" to 30 minutes

### 6.2: Keep Windows updated

Regularly update Windows to protect against known vulnerabilities:

1. Enable automatic updates:
   - Open Windows Settings > Update & Security > Windows Update
   - Click "Advanced options"
   - Enable "Receive updates for other Microsoft products"
   - Set active hours so updates don't interrupt your work

2. Regularly check for updates manually:
   - Open Windows Settings > Update & Security > Windows Update
   - Click "Check for updates"
   - Install any available updates

### 6.3: Use Windows security features

1. Ensure Windows Security (formerly Windows Defender) is active:
   - Open Windows Security from the Start menu
   - Check that all protection areas show green checkmarks
   - Run periodic full scans of your system

2. Enable controlled folder access to protect against ransomware:
   - Open Windows Security > Virus & threat protection
   - Under Ransomware protection, click "Manage ransomware protection"
   - Turn on "Controlled folder access"

3. Enable Windows Firewall:
   - Open Windows Security > Firewall & network protection
   - Ensure firewall is enabled for all network types (Domain, Private, Public)
   - Only allow the specific ports needed for SSH (as configured earlier)

### 6.4: Secure remote access to Windows

If you're using Remote Desktop or other remote access tools to connect to your Windows machine:

1. Use a non-standard port for Remote Desktop (if enabled):
   - Open Registry Editor (regedit)
   - Navigate to HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp
   - Modify the "PortNumber" value (default is 3389)
   - Update your firewall rules accordingly

2. Enable Network Level Authentication (NLA):
   - Open System Properties (right-click on This PC > Properties)
   - Click "Remote settings"
   - Check "Allow remote connections to this computer"
   - Ensure "Allow connections only from computers running Remote Desktop with Network Level Authentication" is checked

3. Limit Remote Desktop users:
   - In the Remote settings dialog, click "Select Users"
   - Only add users who absolutely need remote access

## What we've accomplished

Great job! You've now:
- Disabled password authentication for better security
- Set up fail2ban to protect against brute force attacks
- Configured SSH timeout settings
- Implemented additional security measures for your SSH connection
- Secured your Windows host machine that runs WSL

Your entire setup is now much more secure against common threats, protecting both your WSL environment and the Windows host it runs on.

## What's next?

In the next guide, we'll explore advanced SSH features that can make your remote work more productive.

Move on to the next guide: "Advanced SSH Features and Configuration"
