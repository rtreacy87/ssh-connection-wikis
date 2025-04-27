# Preparing Windows for SSH Access to WSL

## What we'll do in this guide

In this guide, we'll get your Windows computer ready for SSH connections by:
1. Installing WSL2 (Windows Subsystem for Linux)
2. Setting up Ubuntu within WSL
3. Making sure everything is updated
4. Checking that your system is ready for the next steps

## What is WSL?

WSL (Windows Subsystem for Linux) is a feature in Windows that lets you run a Linux environment directly on Windows, without needing a separate computer or virtual machine. It's like having Linux inside Windows!

WSL2 is the newer version that performs better and has more features than the original WSL.

## Step 1: Install WSL2 and Ubuntu

### The easy way (Windows 10 updated version or Windows 11)

1. Open PowerShell as Administrator:
   - Click the Start menu
   - Type "PowerShell"
   - Right-click on "Windows PowerShell" and select "Run as administrator"
   - Click "Yes" if prompted by User Account Control

2. In the PowerShell window, type this command and press Enter:
   ```powershell
   wsl --install -d Ubuntu
   ```

   **What this command does:**
   - `wsl --install` is the main command to install WSL
   - `-d Ubuntu` specifies that you want to install the Ubuntu distribution
   - This command automatically enables necessary Windows features, downloads Ubuntu, and sets up WSL2

   **Why this is necessary:**
   This single command handles all the steps needed to set up WSL2 with Ubuntu on newer Windows versions. It's much simpler than the manual process required on older Windows versions.

3. Restart your computer when prompted.

4. After restarting, a new window will open automatically to finish the Ubuntu setup.

### If the easy way doesn't work

If you're using an older version of Windows 10 or the command above doesn't work, follow these steps:

1. Open PowerShell as Administrator (as described above)

2. Enable the WSL feature:
   ```powershell
   dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
   ```

   **What this command does:**
   - `dism.exe` (Deployment Image Servicing and Management) is a Windows tool that manages Windows features
   - `/online` tells it to work on the currently running Windows system
   - `/enable-feature` indicates we want to turn on a Windows feature
   - `/featurename:Microsoft-Windows-Subsystem-Linux` specifies which feature to enable (the WSL feature)
   - `/all` installs all necessary components for the feature
   - `/norestart` prevents automatic restart after installation

   **Why this is necessary:**
   This command enables the basic WSL functionality in Windows. It's the first step in the installation process because it adds the core components needed to run Linux on Windows. Without this step, your computer wouldn't know how to run Linux binaries.

3. Enable Virtual Machine Platform:
   ```powershell
   dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
   ```

   **What this command does:**
   - Similar to the previous command, but enables a different feature
   - `/featurename:VirtualMachinePlatform` enables the Windows virtualization platform
   - This provides the virtualization capabilities needed by WSL2

   **Why this is necessary:**
   WSL2 runs Linux in a lightweight virtual machine for better performance and compatibility. This command enables the virtualization technology that WSL2 depends on. Without this step, you could only use WSL1, which has more limitations and lower performance.

4. Restart your computer

5. Download and install the WSL2 Linux kernel update package:
   - Go to: https://aka.ms/wsl2kernel
   - Download and run the installer

6. Set WSL2 as the default:
   ```powershell
   wsl --set-default-version 2
   ```

   **What this command does:**
   - Tells Windows to use WSL2 by default for any new Linux distributions you install
   - Changes a system-wide setting that affects all future WSL installations

   **Why this is necessary:**
   This command ensures that when you install Ubuntu (or any other Linux distribution), it will automatically use WSL2 instead of WSL1. WSL2 provides better performance, improved file system compatibility, and full system call compatibility, making it the recommended version for most users.

7. Install Ubuntu from the Microsoft Store:
   - Open the Microsoft Store app
   - Search for "Ubuntu"
   - Select "Ubuntu" (not Ubuntu 18.04 or other versions)
   - Click "Get" or "Install"
   - Click "Launch" when installation is complete

## Step 2: Set up your Ubuntu username and password

After installing Ubuntu, you'll need to create a user account:

1. When Ubuntu opens for the first time, you'll be asked to create a username and password
2. Type a username you want to use (it can be anything you like)
3. Press Enter
4. Type a password (the characters won't show as you type - this is normal for security)
5. Press Enter
6. Type the same password again to confirm
7. Press Enter

**Important**: Remember this username and password! You'll need them later to connect from your Mac.

## Step 3: Make sure everything is up to date

Now let's update Ubuntu to make sure you have the latest software:

1. In the Ubuntu window, type this command and press Enter:
   ```bash
   sudo apt update
   ```
   - "sudo" runs commands with administrator privileges
   - "apt" is Ubuntu's package manager (like an app store)
   - "update" tells it to refresh its list of available software

   **What this command does:**
   - Connects to Ubuntu's software repositories (online software libraries)
   - Downloads the latest list of available packages and their versions
   - Doesn't actually install or upgrade anything yet, just updates the catalog

   **Why this is necessary:**
   Before installing or upgrading software, Ubuntu needs to know what's available and what versions exist. This command ensures you have the most current information about available software, which helps prevent errors and security issues.

2. When prompted, enter your password (the one you just created)

3. Next, upgrade all the software by typing:
   ```bash
   sudo apt upgrade -y
   ```
   - The "-y" automatically answers "yes" to installation questions

   **What this command does:**
   - Compares your installed software with the updated catalog from the previous command
   - Identifies packages that have newer versions available
   - Downloads and installs those newer versions
   - The `-y` flag automatically confirms any prompts, so you don't have to type "yes" repeatedly

   **Why this is necessary:**
   This command actually performs the upgrades to your software. Keeping your system updated is crucial for security and stability. Updated packages include bug fixes, security patches, and sometimes new features.

4. Wait for the upgrade to complete (this might take a few minutes)

## Step 4: Check that WSL2 is working correctly

Let's make sure WSL2 is properly installed:

1. Open PowerShell as Administrator again

2. Type this command and press Enter:
   ```powershell
   wsl --list --verbose
   ```

   **What this command does:**
   - `wsl` is the main command-line tool for managing WSL
   - `--list` (or `-l` for short) shows all installed Linux distributions
   - `--verbose` (or `-v` for short) shows detailed information including the WSL version being used

   **Why this is necessary:**
   This command helps you verify that Ubuntu is properly installed and running under WSL2. It's important to check this because some features we'll use later only work with WSL2, not WSL1.

3. You should see output that includes "Ubuntu" with version "2" like this:
   ```
   NAME      STATE           VERSION
   * Ubuntu    Running         2
   ```

If you see version "1" instead of "2", you can convert it to WSL2 with:
```powershell
wsl --set-version Ubuntu 2
```

**What this command does:**
- Converts an existing WSL1 Ubuntu installation to WSL2
- This process might take several minutes as it creates a virtual machine file for Ubuntu

**Why this is necessary:**
This command upgrades your Ubuntu installation from WSL1 to WSL2 if needed. WSL2 provides better performance and compatibility, which is essential for the SSH server we'll set up later.

## Step 5: Find your Ubuntu IP address

We'll need to know the IP address of your Ubuntu system for later steps:

1. In your Ubuntu window, type this command and press Enter:
   ```bash
   ip addr show eth0 | grep 'inet '
   ```

   **What this command does:**
   - `ip addr show eth0` displays information about the network interface named "eth0" (the main network adapter in WSL)
   - `grep 'inet '` filters the output to only show lines containing "inet " (IPv4 addresses)
   - The pipe symbol `|` connects these two commands, sending the output of the first command to the second

   **Why this is necessary:**
   This command helps you find the IP address that WSL assigned to your Ubuntu system. We need this IP address for the port forwarding setup in a later guide, which will allow your Mac to connect to the SSH server running in Ubuntu.

2. You'll see output that looks something like:
   ```
   inet 172.29.235.131/20 brd 172.29.239.255 scope global eth0
   ```

3. The number after "inet" (like 172.29.235.131) is your Ubuntu IP address. Write this down or remember it - you'll need it in a later guide.

## What we've accomplished

Great job! You've now:
- Installed WSL2 on your Windows computer
- Set up Ubuntu Linux
- Updated all the software
- Verified that everything is working correctly
- Found your Ubuntu's IP address

## What's next?

In the next guide, we'll set up the SSH server in your Ubuntu system. This will allow your Mac to connect to it securely.

Move on to the next guide: "Configuring the SSH Server in WSL Ubuntu"
