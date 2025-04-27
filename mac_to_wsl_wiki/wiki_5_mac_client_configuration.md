# Setting Up Your Mac as an SSH Client

## What we'll do in this guide

In this guide, we'll configure your Mac to connect to your Windows/Ubuntu system. We'll:

1. Generate SSH keys for secure authentication
2. Set up your Mac's SSH configuration
3. Copy your SSH key to your Ubuntu system
4. Test the connection

## What are SSH keys?

SSH keys are a pair of cryptographic keys that provide a more secure way to log in than using passwords. They consist of:

- A **private key** that stays on your Mac (keep this secret!)
- A **public key** that you copy to your Ubuntu system

Think of the public key as a special lock that only your private key can open. You install this lock on your Ubuntu system, and then your Mac can unlock it without needing a password.

## Step 1: Generate SSH keys on your Mac

First, let's create your SSH key pair:

1. Open Terminal on your Mac:
   - Click the magnifying glass (Spotlight) in the top-right corner
   - Type "Terminal"
   - Click on the Terminal app

2. Generate a new SSH key pair:
   ```bash
   ssh-keygen -t ed25519 -C "your_email@example.com"
   ```

   Replace "your_email@example.com" with your actual email address. This is just a label to help you identify the key.

3. When asked "Enter file in which to save the key," press Enter to accept the default location.

4. When asked for a passphrase:
   - For maximum security, enter a passphrase you'll remember
   - For convenience (but less security), you can leave it empty by pressing Enter
   - If you use a passphrase, you'll need to enter it when you connect

5. You'll see a confirmation that your key has been saved.

## Step 2: Set up your Mac's SSH configuration

Now let's create a configuration file to make connecting easier:

1. Create or edit your SSH config file:
   ```bash
   mkdir -p ~/.ssh
   touch ~/.ssh/config
   chmod 600 ~/.ssh/config
   nano ~/.ssh/config
   ```

2. Add the following configuration (replace the values with your information):
   ```
   Host windows-wsl
       HostName 192.168.1.5
       User your-ubuntu-username
       Port 22
       IdentityFile ~/.ssh/id_ed25519
       ServerAliveInterval 60
       ServerAliveCountMax 3
   ```

   Replace:
   - `192.168.1.5` with your Windows computer's IP address (from the previous guide)
   - `your-ubuntu-username` with your Ubuntu username

3. Save the file:
   - Press Ctrl+O to save
   - Press Enter to confirm
   - Press Ctrl+X to exit nano

This configuration:
- Creates a shortcut named "windows-wsl" for your connection
- Specifies which SSH key to use
- Keeps the connection alive during periods of inactivity

## Step 3: Copy your SSH key to your Ubuntu system

Now we need to copy your public key to your Ubuntu system:

### Option 1: Using ssh-copy-id (easiest)

1. Copy your key (you'll need to enter your Ubuntu password once):
   ```bash
   ssh-copy-id your-ubuntu-username@192.168.1.5
   ```

   Replace:
   - `your-ubuntu-username` with your Ubuntu username
   - `192.168.1.5` with your Windows computer's IP address

### Option 2: Manual method (if ssh-copy-id doesn't work)

1. Display your public key:
   ```bash
   cat ~/.ssh/id_ed25519.pub
   ```

2. Copy the output (it starts with "ssh-ed25519" and ends with your email)

3. Connect to your Ubuntu system with password authentication:
   ```bash
   ssh your-ubuntu-username@192.168.1.5
   ```

4. Enter your Ubuntu password when prompted

5. Create the .ssh directory and authorized_keys file:
   ```bash
   mkdir -p ~/.ssh
   chmod 700 ~/.ssh
   touch ~/.ssh/authorized_keys
   chmod 600 ~/.ssh/authorized_keys
   ```

6. Open the authorized_keys file:
   ```bash
   nano ~/.ssh/authorized_keys
   ```

7. Paste your public key on a new line

8. Save the file (Ctrl+O, Enter, Ctrl+X)

9. Exit the SSH session:
   ```bash
   exit
   ```

## Step 4: Test the connection

Now let's test that everything works:

1. Connect using the shortcut we created:
   ```bash
   ssh windows-wsl
   ```

2. If you set a passphrase on your SSH key, enter it when prompted.

3. You should connect without being asked for your Ubuntu password!

4. Try running a simple command:
   ```bash
   ls -la
   ```

   This should list the files in your Ubuntu home directory.

5. Exit the SSH session:
   ```bash
   exit
   ```

## Step 5: Test file transfer

Let's also test transferring a file:

1. Create a test file on your Mac:
   ```bash
   echo "Hello from Mac" > ~/test-file.txt
   ```

2. Copy the file to your Ubuntu system:
   ```bash
   scp ~/test-file.txt windows-wsl:~/
   ```

3. Connect to your Ubuntu system:
   ```bash
   ssh windows-wsl
   ```

4. Check that the file was transferred:
   ```bash
   cat ~/test-file.txt
   ```

   You should see "Hello from Mac".

5. Exit the SSH session:
   ```bash
   exit
   ```

## Troubleshooting

If you have trouble connecting:

1. Check that you can ping your Windows computer:
   ```bash
   ping 192.168.1.5
   ```

2. Make sure you're using the correct Windows IP address and Ubuntu username.

3. Try connecting with verbose output to see more details:
   ```bash
   ssh -v windows-wsl
   ```

4. Check that the SSH service is running on your Ubuntu system (from the Windows PowerShell):
   ```powershell
   wsl -d Ubuntu -u root service ssh status
   ```

5. Verify your port forwarding is still set up (from the Windows PowerShell):
   ```powershell
   netsh interface portproxy show all
   ```

### Troubleshooting "Host key verification failed" error

If you see an error like this:

```
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
Someone could be eavesdropping on you right now (man-in-the-middle attack)!
It is also possible that a host key has just been changed.
...
Host key verification failed.
```

This happens when the SSH server's fingerprint has changed from what's stored in your `~/.ssh/known_hosts` file. This can occur for several legitimate reasons:

- You reinstalled the operating system on your Windows machine
- You reinstalled or reconfigured the SSH server in WSL
- The WSL instance was reset or recreated
- The IP address is being reused for a different machine

**How to fix it:**

1. Remove the old host key from your Mac's known_hosts file:
   ```bash
   ssh-keygen -R 192.168.1.5
   ```

   Replace `192.168.1.5` with your Windows computer's IP address. If you're using a hostname in your SSH config, also run:
   ```bash
   ssh-keygen -R windows-wsl
   ```

2. If you're using `ssh-copy-id` and still getting the error, you can force it to ignore the known hosts check:
   ```bash
   ssh-copy-id -o StrictHostKeyChecking=no your-ubuntu-username@192.168.1.5
   ```

3. Try connecting again. You'll be asked to confirm the new fingerprint:
   ```
   The authenticity of host '192.168.1.5 (192.168.1.5)' can't be established.
   ED25519 key fingerprint is SHA256:abcdefghijklmnopqrstuvwxyz123456789.
   Are you sure you want to continue connecting (yes/no/[fingerprint])?
   ```

4. **For maximum security**: Before typing "yes", verify the fingerprint is correct by checking it on your Windows/WSL system:
   ```bash
   # Run this in WSL
   sudo ssh-keygen -l -f /etc/ssh/ssh_host_ed25519_key.pub
   ```

   Compare this fingerprint with what's shown in the connection prompt. If they match, type "yes" to continue.

5. If you frequently rebuild your WSL environment and don't want to deal with this warning, you can add this to your SSH config file:
   ```
   Host windows-wsl
       StrictHostKeyChecking no
   ```

   **Note**: This reduces security by disabling host key verification, so only use it in trusted environments.

## What we've accomplished

Great job! You've now:
- Generated SSH keys on your Mac
- Set up your SSH configuration for easy connections
- Copied your public key to your Ubuntu system
- Successfully connected to your Ubuntu system from your Mac
- Transferred a file between your computers

## What's next?

In the next guide, we'll improve the security of your SSH connection by disabling password authentication and setting up additional security measures.

Move on to the next guide: "Securing Your SSH Connection"
