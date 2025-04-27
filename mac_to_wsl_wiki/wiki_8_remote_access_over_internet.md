# Accessing Your WSL Environment Remotely

## What we'll do in this guide

In this guide, we'll set up remote access to your WSL environment over the internet. This will allow you to connect to your Windows/Ubuntu system from anywhere, not just from your local network. We'll cover:

1. Understanding the challenges of remote access
2. Setting up Dynamic DNS to handle changing IP addresses
3. Configuring your router for port forwarding
4. Exploring VPN alternatives for more secure access
5. Testing and securing your remote connection

## Understanding remote access challenges

When you want to access your computer from outside your home network, you face several challenges:

1. **Dynamic IP addresses**: Most home internet connections don't have a fixed IP address. Your internet provider might change your IP address periodically.

2. **Router/firewall**: Your home router creates a private network and blocks incoming connections by default.

3. **Security risks**: Opening your computer to the internet increases security risks.

We'll address each of these challenges in this guide.

## Step 1: Find your public IP address

First, let's find your current public IP address (the address of your home network on the internet):

1. On your Windows computer, open a web browser and go to:
   ```
   https://whatismyip.com
   ```

2. The website will show your public IP address. Write this down - it's what you'll connect to from the internet.

3. Note that this address might change periodically unless you have a static IP from your internet provider.

## Step 2: Set up Dynamic DNS

Dynamic DNS (DDNS) solves the problem of changing IP addresses by giving you a permanent domain name that automatically updates when your IP changes.

### 2.1: Choose a Dynamic DNS provider

Several free and paid services offer Dynamic DNS. Here are some popular options:
- No-IP (https://www.noip.com) - Free tier available
- DuckDNS (https://www.duckdns.org) - Free
- Dynu (https://www.dynu.com) - Free tier available

We'll use No-IP in this example, but the process is similar for other providers.

### 2.2: Create a Dynamic DNS account and hostname

1. Go to https://www.noip.com and sign up for a free account

2. After signing in, go to "Dynamic DNS" and click "Create Hostname"

3. Choose a hostname (e.g., "mywindowswsl")

4. Select a domain from the free options (e.g., "hopto.org")

5. Keep the IP address as the default (your current IP)

6. Click "Create Hostname"

You now have a domain name (e.g., "mywindowswsl.hopto.org") that points to your current public IP address.

### 2.3: Install the Dynamic DNS update client

To automatically update your IP address when it changes:

1. On your Windows computer, download the No-IP Dynamic Update Client:
   - Go to https://www.noip.com/download
   - Download the Windows version

2. Install the client:
   - Run the downloaded installer
   - Follow the installation wizard
   - Enter your No-IP account credentials when prompted
   - Select your hostname to keep updated
   - The client will run in the background and update your IP address when it changes

## Step 3: Configure your router for port forwarding

Now we need to tell your router to forward incoming SSH connections to your Windows computer:

### 3.1: Find your router's admin page

1. Find your router's IP address:
   - Open Command Prompt on Windows
   - Type `ipconfig` and press Enter
   - Look for "Default Gateway" - that's your router's IP address (usually 192.168.0.1 or 192.168.1.1)

2. Access your router's admin page:
   - Open a web browser
   - Enter your router's IP address in the address bar
   - Log in with your router's admin credentials
   - If you don't know the credentials, check the router's manual or look for a sticker on the router

### 3.2: Set up port forwarding

The exact steps vary depending on your router model, but generally:

1. Look for a section called "Port Forwarding," "Virtual Server," or "NAT"

2. Create a new port forwarding rule with these settings:
   - **Name/Description**: SSH
   - **External/WAN Port**: 22 (or your custom SSH port if you changed it)
   - **Internal/LAN Port**: 22 (or your custom SSH port)
   - **Protocol**: TCP
   - **Internal IP Address**: Your Windows computer's local IP address (from `ipconfig`, usually 192.168.x.x)
   - **Enabled/Status**: On/Enabled

3. Save the settings

Your router will now forward incoming connections on port 22 to your Windows computer, which will then forward them to your Ubuntu system thanks to the port forwarding we set up earlier.

## Step 4: Test your remote connection

Let's test your remote connection from outside your home network:

1. Disconnect your Mac from your home Wi-Fi and connect to a different network (like a mobile hotspot or public Wi-Fi)

2. Open Terminal on your Mac

3. Connect using your Dynamic DNS hostname:
   ```bash
   ssh your-ubuntu-username@mywindowswsl.hopto.org
   ```

   Replace:
   - `your-ubuntu-username` with your Ubuntu username
   - `mywindowswsl.hopto.org` with your actual Dynamic DNS hostname

4. If everything is set up correctly, you should connect to your Ubuntu system!

5. If you're using a custom SSH port, specify it with the -p option:
   ```bash
   ssh -p 2222 your-ubuntu-username@mywindowswsl.hopto.org
   ```

### 4.1: Add your remote connection to SSH config

To make remote connections easier, add your Dynamic DNS hostname to your SSH config:

1. On your Mac, edit your SSH config:
   ```bash
   nano ~/.ssh/config
   ```

2. Add a new section for remote access:
   ```
   Host windows-wsl-remote
       HostName mywindowswsl.hopto.org
       User your-ubuntu-username
       Port 22
       IdentityFile ~/.ssh/id_ed25519
       ServerAliveInterval 60
       ServerAliveCountMax 3
   ```

   Replace:
   - `mywindowswsl.hopto.org` with your Dynamic DNS hostname
   - `your-ubuntu-username` with your Ubuntu username
   - `22` with your custom port if you changed it

3. Save the file (Ctrl+O, Enter, Ctrl+X)

4. Now you can connect simply with:
   ```bash
   ssh windows-wsl-remote
   ```

## Step 5: VPN alternatives for more secure access

Opening SSH directly to the internet has security risks. A more secure alternative is to use a VPN (Virtual Private Network):

### 5.1: Set up a VPN server on Windows

Windows 10 Pro and Windows 11 Pro include a built-in VPN server:

1. Open Settings > Network & Internet > VPN

2. Click "Add a VPN connection" (on the right side)

3. Select "Windows (built-in)" as the VPN provider

4. Enter a connection name (e.g., "Home VPN")

5. Enter your Dynamic DNS hostname as the server name

6. Choose a VPN type (L2TP/IPsec with pre-shared key is relatively easy to set up)

7. Enter a pre-shared key (make it strong!)

8. Enter a username and password for VPN access

9. Click Save

10. Configure your router to forward the VPN port (typically port 1723 for PPTP or port 500 and 4500 for L2TP/IPsec)

### 5.2: Connect to your VPN from your Mac

1. On your Mac, go to System Preferences > Network

2. Click the "+" button to add a new connection

3. Choose "VPN" as the interface

4. Choose the VPN type that matches your server

5. Enter a service name (e.g., "Home VPN")

6. Click Create

7. Enter your Dynamic DNS hostname as the server address

8. Enter the account name (username) you created

9. Click Authentication Settings and enter your password and shared secret

10. Click OK, then Apply

11. Connect to the VPN

12. Once connected, you can SSH to your Windows computer using its local IP address

## Step 6: Additional security considerations for remote access

When exposing your system to the internet, security becomes even more important:

### 6.1: Use a non-standard SSH port

If you haven't already, consider changing your SSH port from the default 22 to a non-standard port (as described in the security hardening guide).

### 6.2: Limit SSH access by IP address

If you know the IP addresses you'll be connecting from, you can restrict SSH access to only those addresses:

1. Connect to your Ubuntu system:
   ```bash
   ssh windows-wsl
   ```

2. Install the `ufw` firewall if it's not already installed:
   ```bash
   sudo apt install ufw -y
   ```

3. Allow SSH only from specific IP addresses:
   ```bash
   sudo ufw allow from your-remote-ip to any port 22
   ```

   Replace:
   - `your-remote-ip` with the IP address you'll be connecting from
   - `22` with your custom SSH port if you changed it

4. Enable the firewall:
   ```bash
   sudo ufw enable
   ```

   Be careful with this step! If you make a mistake, you might lock yourself out. Make sure you have direct access to your Windows computer as a backup.

### 6.3: Monitor login attempts

Regularly check for unauthorized login attempts:

1. Connect to your Ubuntu system

2. Check the authentication log:
   ```bash
   sudo cat /var/log/auth.log | grep "Failed password"
   ```

3. Check fail2ban status:
   ```bash
   sudo fail2ban-client status sshd
   ```

## Troubleshooting

If you can't connect remotely:

1. **Check your Dynamic DNS**: Make sure your hostname is pointing to your current public IP:
   - Go to https://www.whatismyip.com to see your current public IP
   - Use `nslookup mywindowswsl.hopto.org` to check what IP your hostname resolves to

2. **Check your router's port forwarding**: Make sure the port forwarding rule is enabled and pointing to the correct internal IP address.

3. **Check Windows Firewall**: Make sure the Windows Firewall is allowing incoming connections on your SSH port.

4. **Test locally first**: Make sure you can still connect to your SSH server from within your home network.

5. **Try connecting with verbose output**:
   ```bash
   ssh -v your-ubuntu-username@mywindowswsl.hopto.org
   ```

6. **Check if your ISP is blocking port 22**: Some internet providers block common server ports. Try using a non-standard port.

## What we've accomplished

Great job! You've now:
- Set up Dynamic DNS to handle changing IP addresses
- Configured your router to forward SSH connections to your Windows computer
- Added your remote connection to your SSH config for easy access
- Explored VPN alternatives for more secure remote access
- Learned about additional security considerations for internet-facing services

You can now securely access your WSL environment from anywhere with an internet connection!

## What's next?

In the next guide, we'll cover troubleshooting and maintenance to keep your SSH setup running smoothly.

Move on to the next guide: "Troubleshooting and Maintaining Your SSH Setup"
