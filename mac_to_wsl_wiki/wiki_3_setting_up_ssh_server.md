# Configuring the SSH Server in WSL Ubuntu

## What we'll do in this guide

In this guide, we'll set up an SSH server in your Ubuntu system running on WSL. This will allow your Mac to connect to it. We'll:

1. Install the SSH server software
2. Configure the SSH server for security
3. Start the SSH service
4. Test that the server is working

## What is an SSH server?

An SSH server is a program that listens for and accepts secure connections from other computers. Think of it like a secure door to your computer that only opens for people with the right key.

## Step 1: Install the SSH server

First, we need to install the OpenSSH server software on your Ubuntu system:

1. Open your Ubuntu terminal (if it's not already open)
   - Click the Start menu
   - Type "Ubuntu"
   - Click on the Ubuntu app

2. Update your package list (in case you didn't do this in the previous guide):
   ```bash
   sudo apt update
   ```

3. Install the OpenSSH server:
   ```bash
   sudo apt install openssh-server -y
   ```

4. Wait for the installation to complete

## Step 2: Configure the SSH server

Now we'll configure the SSH server to make it secure:

1. Make a backup of the original configuration file (so we can restore it if needed):
   ```bash
   sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.backup
   ```

2. Open the SSH configuration file in a text editor:
   ```bash
   sudo nano /etc/ssh/sshd_config
   ```

   > **Note**: "nano" is a simple text editor. You'll use your keyboard to navigate (not your mouse).

3. Find and modify these settings in the file:
   - Press Ctrl+W to search
   - Type the setting name (like "PasswordAuthentication") and press Enter to find it
   - If the line starts with a # symbol, remove the # to uncomment it
   - Change the value if needed

   Look for and set these options:
   ```
   # Allow SSH key authentication (should already be "yes")
   PubkeyAuthentication yes

   # For now, keep password authentication enabled (we'll disable it later)
   PasswordAuthentication yes

   # Disable root login for security
   PermitRootLogin no

   # Use the default SSH port
   Port 22
   ```

   > **Note**: If you don't find some of these lines, you can add them at the end of the file.

4. Save the file:
   - Press Ctrl+O to save
   - Press Enter to confirm
   - Press Ctrl+X to exit nano

## Step 3: Start the SSH service

Now let's start the SSH server:

1. Start the SSH service:
   ```bash
   sudo service ssh start
   ```

2. Check that the service is running:
   ```bash
   sudo service ssh status
   ```

   You should see output with "Active: active (running)" in it.

3. Set the SSH service to start automatically when Ubuntu starts:
   ```bash
   sudo systemctl enable ssh
   ```

## Step 4: Test the SSH server locally

Before trying to connect from your Mac, let's make sure the SSH server is working within Ubuntu:

1. Connect to your own SSH server:
   ```bash
   ssh localhost
   ```

2. You'll see a message about the authenticity of the host. Type "yes" and press Enter.

3. Enter your Ubuntu password when prompted.

4. If you see a new command prompt, congratulations! Your SSH server is working.

5. Type `exit` and press Enter to disconnect from the SSH session.

## Step 5: Check if the SSH server is listening properly

Let's make sure the SSH server is listening for connections from outside:

1. Check what addresses the SSH server is listening on:
   ```bash
   sudo ss -tunlp | grep ssh
   ```

2. You should see output that includes something like:
   ```
   tcp   LISTEN  0       128    0.0.0.0:22       0.0.0.0:*     users:(("sshd",pid=1234,fd=3))
   ```

   This means the SSH server is listening on all network interfaces (0.0.0.0) on port 22.

## Troubleshooting

If the SSH service isn't starting or you can't connect locally, try these steps:

1. Check for error messages:
   ```bash
   sudo journalctl -u ssh
   ```

2. Make sure the SSH server is installed correctly:
   ```bash
   sudo apt install --reinstall openssh-server
   ```

3. Restart the SSH service:
   ```bash
   sudo service ssh restart
   ```

## What we've accomplished

Great job! You've now:
- Installed the OpenSSH server on your Ubuntu system
- Configured the SSH server for security
- Started the SSH service
- Verified that the server is working locally

## What's next?

In the next guide, we'll set up port forwarding on Windows. This is necessary because your Ubuntu system is running inside WSL, and Windows needs to know how to route SSH connections to it.

Move on to the next guide: "Windows Port Forwarding for WSL SSH Access"
