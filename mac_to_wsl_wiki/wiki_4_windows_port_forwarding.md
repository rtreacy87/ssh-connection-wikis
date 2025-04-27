# Windows Port Forwarding for WSL SSH Access

## What we'll do in this guide

In this guide, we'll set up port forwarding on your Windows computer. This is a crucial step because:
1. Your Ubuntu system is running inside WSL
2. Windows needs to know how to direct incoming SSH connections to Ubuntu
3. We need to make this setup survive computer restarts

## What is port forwarding?

Port forwarding is like setting up a mail forwarding service. When someone sends a letter to your temporary address (Windows), the service forwards it to your actual address (Ubuntu in WSL).

In our case:
- Your Mac will connect to your Windows computer's IP address on port 22 (the standard SSH port)
- Windows will forward that connection to your Ubuntu system's IP address

## Step 1: Find your WSL Ubuntu IP address

First, let's make sure we have the correct IP address for your Ubuntu system:

1. Open your Ubuntu terminal (if it's not already open)
   - Click the Start menu
   - Type "Ubuntu"
   - Click on the Ubuntu app

2. Find your IP address:
   ```bash
   ip addr show eth0 | grep 'inet '
   ```

3. You'll see output that looks something like:
   ```
   inet 172.29.235.131/20 brd 172.29.239.255 scope global eth0
   ```

4. The number after "inet" (like 172.29.235.131) is your Ubuntu IP address. Write this down - you'll need it in the next step.

## Step 2: Set up port forwarding

Now we'll tell Windows to forward SSH connections to your Ubuntu system:

1. Open PowerShell as Administrator:
   - Click the Start menu
   - Type "PowerShell"
   - Right-click on "Windows PowerShell" and select "Run as administrator"
   - Click "Yes" if prompted by User Account Control

2. Set up port forwarding with this command (replace 172.x.x.x with your actual Ubuntu IP address from Step 1):
   ```powershell
   netsh interface portproxy add v4tov4 listenaddress=0.0.0.0 listenport=22 connectaddress=172.x.x.x connectport=22
   ```

   This command tells Windows:
   - Listen for connections to any IP address on this computer (0.0.0.0)
   - On port 22 (the standard SSH port)
   - Forward those connections to your Ubuntu's IP address
   - On Ubuntu's port 22 (where the SSH server is listening)

3. Verify that the port forwarding is set up:
   ```powershell
   netsh interface portproxy show all
   ```

   You should see your port forwarding rule in the output.

## Step 3: Add a Windows Firewall rule

Now we need to allow SSH connections through the Windows Firewall:

1. Still in your Administrator PowerShell, run:
   ```powershell
   New-NetFirewallRule -Name "SSH" -DisplayName "SSH" -Protocol TCP -LocalPort 22 -Action Allow -Direction Inbound
   ```

   This command:
   - Creates a new firewall rule named "SSH"
   - Allows incoming TCP connections
   - On port 22
   - From any address

2. Verify the firewall rule was created:
   ```powershell
   Get-NetFirewallRule -Name "SSH" | Format-List -Property *
   ```

   You should see details about your new firewall rule.

## Step 4: Make port forwarding persistent

The port forwarding we set up will be lost when your computer restarts. Let's create a script that will automatically restore it:

1. Still in your Administrator PowerShell, create a script file:
   ```powershell
   $scriptContent = @'
   # Remove existing port forwarding
   netsh interface portproxy reset

   # Get WSL IP address
   $wslIP = (wsl hostname -I).Trim()

   # Set up port forwarding
   netsh interface portproxy add v4tov4 listenaddress=0.0.0.0 listenport=22 connectaddress=$wslIP connectport=22
   '@

   $scriptContent | Out-File -FilePath "$env:USERPROFILE\update-wsl-ssh.ps1" -Encoding ASCII
   ```

   This creates a script that:
   - Clears any existing port forwarding rules
   - Gets your Ubuntu's current IP address (which might change after restarts)
   - Sets up the port forwarding rule with the current IP address

2. Create a scheduled task to run this script at startup:
   ```powershell
   $action = New-ScheduledTaskAction -Execute "PowerShell.exe" -Argument "-ExecutionPolicy Bypass -File `"$env:USERPROFILE\update-wsl-ssh.ps1`""
   $trigger = New-ScheduledTaskTrigger -AtStartup
   $principal = New-ScheduledTaskPrincipal -UserId "$env:USERDOMAIN\$env:USERNAME" -LogonType S4U -RunLevel Highest
   Register-ScheduledTask -TaskName "WSL SSH Port Forwarding" -Action $action -Trigger $trigger -Principal $principal
   ```

   **Detailed explanation of each line:**

   Line 1: `$action = New-ScheduledTaskAction -Execute "PowerShell.exe" -Argument "-ExecutionPolicy Bypass -File `"$env:USERPROFILE\update-wsl-ssh.ps1`""`
   - Creates a variable `$action` that defines what program to run and with what arguments
   - `-Execute "PowerShell.exe"` specifies that PowerShell will be launched
   - `-Argument` defines the parameters to pass to PowerShell
   - `-ExecutionPolicy Bypass` allows the script to run without being blocked by security policies
   - `-File "$env:USERPROFILE\update-wsl-ssh.ps1"` tells PowerShell which script file to run
   - The backticks (`) before the quotes are escape characters to include quotes within quotes
   - `$env:USERPROFILE` is an environment variable that points to your user profile folder (typically C:\Users\YourUsername)

   Line 2: `$trigger = New-ScheduledTaskTrigger -AtStartup`
   - Creates a variable `$trigger` that defines when the task should run
   - `-AtStartup` means the task will run whenever your computer starts up

   Line 3: `$principal = New-ScheduledTaskPrincipal -UserId "$env:USERDOMAIN\$env:USERNAME" -LogonType S4U -RunLevel Highest`
   - Creates a variable `$principal` that defines who the task runs as and what permissions it has
   - `-UserId "$env:USERDOMAIN\$env:USERNAME"` uses your current username and domain
   - `-LogonType S4U` (Service for User) allows the task to run without you being logged in
   - `-RunLevel Highest` gives the task administrator privileges

   Line 4: `Register-ScheduledTask -TaskName "WSL SSH Port Forwarding" -Action $action -Trigger $trigger -Principal $principal`
   - Actually creates the scheduled task in Windows using the variables defined above
   - `-TaskName "WSL SSH Port Forwarding"` gives the task a name you can find in Task Scheduler
   - The other parameters link to the variables we created in the previous lines

   **Where is the script file saved?**

   The script is saved to your user profile folder at:
   ```
   C:\Users\YourUsername\update-wsl-ssh.ps1
   ```

   You can find this location by typing `echo $env:USERPROFILE` in PowerShell, which will show something like `C:\Users\YourUsername`. The script will be in that folder.

   **Executive Summary:**

   This set of commands creates an automated task that runs every time your computer starts up. The task runs a PowerShell script with administrator privileges, even if you're not logged in. The script updates the port forwarding rules to use the current WSL IP address, which might change between restarts. This ensures that SSH connections to your Windows machine will always be correctly forwarded to your Ubuntu system, making your SSH server reliably accessible.

## Step 5: Test the port forwarding

Let's make sure everything is working:

1. Find your Windows computer's IP address:
   ```powershell
   ipconfig
   ```

   Look for the "IPv4 Address" under your main network adapter (usually "Ethernet adapter" or "Wireless LAN adapter"). It will look something like 192.168.1.5.

2. Try connecting to your SSH server from Windows:
   ```powershell
   ssh your-ubuntu-username@localhost
   ```

   Replace "your-ubuntu-username" with the username you created in Ubuntu.

3. If prompted about host authenticity, type "yes" and press Enter.

4. Enter your Ubuntu password when prompted.

5. If you successfully connect, type `exit` to disconnect.

## Troubleshooting

If you can't connect to your SSH server:

1. Check that the SSH service is running in Ubuntu:
   ```bash
   sudo service ssh status
   ```

2. Verify your port forwarding is set up:
   ```powershell
   netsh interface portproxy show all
   ```

3. Check your firewall rule:
   ```powershell
   Get-NetFirewallRule -Name "SSH" | Format-List -Property *
   ```

4. Make sure you're using the correct Ubuntu username and password.

5. Try restarting the SSH service in Ubuntu:
   ```bash
   sudo service ssh restart
   ```

## What we've accomplished

Great job! You've now:
- Set up port forwarding from Windows to your Ubuntu system
- Created a Windows Firewall rule to allow SSH connections
- Made the port forwarding persistent across restarts
- Tested that the connection works from Windows

## What's next?

In the next guide, we'll set up your Mac to connect to your Windows/Ubuntu system using SSH.

Move on to the next guide: "Setting Up Your Mac as an SSH Client"
