# Practical Workflows: Using Your Mac-to-WSL SSH Connection

## What we'll do in this guide

In this guide, we'll explore practical ways to use your Mac-to-WSL SSH connection for real-world tasks:

1. Development workflows across platforms
2. File synchronization and management
3. Remote command execution and automation
4. Integration with development tools and IDEs
5. Real-world examples and case studies

## Development workflows across platforms

Your SSH connection enables powerful cross-platform development workflows:

### Web development with Linux tools

You can develop web applications using Linux tools while working on your Mac:

1. Connect to your Ubuntu system:
   ```bash
   ssh windows-wsl
   ```

2. Install web development tools:
   ```bash
   sudo apt update
   sudo apt install nodejs npm -y
   ```

3. Create a simple web project:
   ```bash
   mkdir ~/web-project
   cd ~/web-project
   npm init -y
   npm install express
   ```

4. Create a simple Express server:
   ```bash
   echo 'const express = require("express");
   const app = express();
   const port = 3000;

   app.get("/", (req, res) => {
     res.send("Hello from WSL!");
   });

   app.listen(port, () => {
     console.log(`Server running at http://localhost:${port}`);
   });' > index.js
   ```

5. Start the server:
   ```bash
   node index.js
   ```

6. On your Mac, create an SSH tunnel to access the web server:
   ```bash
   ssh -L 3000:localhost:3000 windows-wsl
   ```

7. Open a web browser on your Mac and go to:
   ```
   http://localhost:3000
   ```

   You should see "Hello from WSL!"

### Using Linux-specific tools

Some development tools work better on Linux. Now you can use them from your Mac:

1. Connect to your Ubuntu system:
   ```bash
   ssh windows-wsl
   ```

2. Install Linux-specific tools:
   ```bash
   sudo apt install build-essential gcc g++ make -y
   ```

3. Create a simple C program:
   ```bash
   echo '#include <stdio.h>
   int main() {
       printf("Hello from C on Linux!\n");
       return 0;
   }' > hello.c
   ```

4. Compile and run it:
   ```bash
   gcc hello.c -o hello
   ./hello
   ```

### Cross-platform testing

You can test your applications on both macOS and Linux:

1. Develop on your Mac
2. Push to a Git repository
3. Pull on your Ubuntu system
4. Test in the Linux environment
5. Fix any platform-specific issues

## File synchronization and management

Your SSH connection makes it easy to manage files between your Mac and Ubuntu:

### Using SCP for file transfers

SCP (Secure Copy) lets you copy files securely:

1. Copy a file from Mac to Ubuntu:
   ```bash
   scp ~/Documents/file.txt windows-wsl:~/
   ```

2. Copy a file from Ubuntu to Mac:
   ```bash
   scp windows-wsl:~/file.txt ~/Documents/
   ```

3. Copy an entire directory:
   ```bash
   scp -r ~/Projects/my-project windows-wsl:~/Projects/
   ```

### Using SFTP for interactive file management

SFTP provides an interactive file transfer session:

1. Start an SFTP session:
   ```bash
   sftp windows-wsl
   ```

2. Navigate directories:
   ```
   cd Documents
   lcd ~/Downloads  # local change directory
   ```

3. Upload and download files:
   ```
   put file.txt     # upload from Mac to Ubuntu
   get report.pdf   # download from Ubuntu to Mac
   ```

4. List files:
   ```
   ls               # list remote files
   lls              # list local files
   ```

5. Exit SFTP:
   ```
   exit
   ```

### Using rsync for efficient synchronization

Rsync is perfect for keeping directories in sync:

1. Install rsync on your Mac (if not already installed):
   ```bash
   brew install rsync
   ```

2. Install rsync on Ubuntu:
   ```bash
   sudo apt install rsync -y
   ```

3. Sync a directory from Mac to Ubuntu:
   ```bash
   rsync -avz ~/Projects/my-project/ windows-wsl:~/Projects/my-project/
   ```

4. Sync a directory from Ubuntu to Mac:
   ```bash
   rsync -avz windows-wsl:~/Projects/my-project/ ~/Projects/my-project/
   ```

5. Set up automatic synchronization with a script:
   ```bash
   echo '#!/bin/bash
   rsync -avz ~/Projects/my-project/ windows-wsl:~/Projects/my-project/
   ' > ~/sync-project.sh
   chmod +x ~/sync-project.sh
   ```

   Run this script whenever you want to sync your files.

## Remote command execution and automation

Your SSH connection allows you to run commands on your Ubuntu system from your Mac:

### Running individual commands remotely

You can run a single command without starting a full SSH session:

```bash
ssh windows-wsl "ls -la"
```

This runs `ls -la` on your Ubuntu system and shows the output on your Mac.

### Creating remote command aliases

Add aliases to your Mac's shell configuration for frequently used remote commands:

1. Edit your shell configuration:
   ```bash
   nano ~/.zshrc  # For macOS Catalina and newer
   # OR
   nano ~/.bash_profile  # For older macOS versions
   ```

2. Add aliases for remote commands:
   ```bash
   # Run Ubuntu update
   alias update-ubuntu="ssh windows-wsl 'sudo apt update && sudo apt upgrade -y'"
   
   # Check Ubuntu disk space
   alias ubuntu-disk="ssh windows-wsl 'df -h'"
   
   # Start a development server
   alias start-server="ssh windows-wsl 'cd ~/web-project && node index.js'"
   ```

3. Save the file (Ctrl+O, Enter, Ctrl+X)

4. Apply the changes:
   ```bash
   source ~/.zshrc  # For macOS Catalina and newer
   # OR
   source ~/.bash_profile  # For older macOS versions
   ```

5. Now you can run these commands directly from your Mac:
   ```bash
   update-ubuntu
   ubuntu-disk
   start-server
   ```

### Setting up scheduled tasks

You can set up scheduled tasks on your Mac that run commands on your Ubuntu system:

1. Create a script for your task:
   ```bash
   echo '#!/bin/bash
   ssh windows-wsl "cd ~/web-project && git pull && npm install"
   ' > ~/update-project.sh
   chmod +x ~/update-project.sh
   ```

2. Schedule it with cron:
   ```bash
   crontab -e
   ```

3. Add a line to run the script daily at 9 AM:
   ```
   0 9 * * * ~/update-project.sh
   ```

4. Save and exit the editor

## Integration with development tools and IDEs

Many development tools and IDEs support remote development over SSH:

### Visual Studio Code Remote Development

VS Code has excellent support for remote development over SSH:

1. Install the "Remote - SSH" extension in VS Code

2. Click the green "Remote Explorer" icon in the sidebar

3. Click "+" to add a new SSH target

4. Enter `your-ubuntu-username@windows-ip-address` or use your SSH config name `windows-wsl`

5. Select a configuration file to update (usually the first option)

6. Connect to your SSH target by clicking on it in the Remote Explorer

7. When prompted, select "Linux" as the platform

8. VS Code will connect to your Ubuntu system and you can open folders, edit files, and use the integrated terminal

### Using Git with SSH

You can use Git on your Mac to work with repositories on your Ubuntu system:

1. Clone a repository from your Ubuntu system to your Mac:
   ```bash
   git clone windows-wsl:~/Projects/my-repo
   ```

2. Push changes back to your Ubuntu system:
   ```bash
   cd my-repo
   # Make some changes
   git add .
   git commit -m "Made changes on Mac"
   git push
   ```

### Database management

You can connect to databases running on your Ubuntu system:

1. Install a database on Ubuntu:
   ```bash
   sudo apt install postgresql -y
   sudo service postgresql start
   ```

2. Create an SSH tunnel for database access:
   ```bash
   ssh -L 5432:localhost:5432 windows-wsl
   ```

3. On your Mac, use your favorite database tool (like pgAdmin, DBeaver, or TablePlus) to connect to:
   - Host: localhost
   - Port: 5432
   - Username: your PostgreSQL username
   - Password: your PostgreSQL password

## Real-world examples and case studies

Let's look at some real-world examples of how you might use your Mac-to-WSL SSH connection:

### Example 1: Full-stack web development

**Scenario**: You're developing a web application with a React frontend and a Node.js backend. You prefer macOS for frontend work but need Linux for the backend.

**Workflow**:

1. Set up your project structure:
   ```bash
   # On Ubuntu
   mkdir -p ~/Projects/fullstack-app/{frontend,backend}
   cd ~/Projects/fullstack-app/backend
   npm init -y
   npm install express mongoose
   
   # Create a simple backend
   echo 'const express = require("express");
   const app = express();
   const port = 3001;
   
   app.use(express.json());
   
   app.get("/api/data", (req, res) => {
     res.json({ message: "Hello from the backend!" });
   });
   
   app.listen(port, () => {
     console.log(`Backend running on http://localhost:${port}`);
   });' > index.js
   ```

2. Set up the frontend on your Mac:
   ```bash
   # On Mac
   mkdir -p ~/Projects/fullstack-app/frontend
   cd ~/Projects/fullstack-app/frontend
   npx create-react-app .
   ```

3. Start the backend on Ubuntu:
   ```bash
   # On Ubuntu
   cd ~/Projects/fullstack-app/backend
   node index.js
   ```

4. Create an SSH tunnel to access the backend:
   ```bash
   # On Mac (in a new terminal)
   ssh -L 3001:localhost:3001 windows-wsl
   ```

5. Configure the frontend to use the backend:
   ```bash
   # On Mac
   cd ~/Projects/fullstack-app/frontend
   # Edit src/App.js to fetch from the backend
   ```

6. Start the frontend:
   ```bash
   # On Mac
   npm start
   ```

7. Develop the frontend on your Mac and the backend on Ubuntu, with changes immediately visible.

### Example 2: Data science workflow

**Scenario**: You're doing data analysis with Python. You want to use Linux-specific libraries but visualize results on your Mac.

**Workflow**:

1. Set up your data science environment on Ubuntu:
   ```bash
   # On Ubuntu
   sudo apt install python3-pip python3-venv -y
   mkdir -p ~/Projects/data-analysis
   cd ~/Projects/data-analysis
   python3 -m venv env
   source env/bin/activate
   pip install numpy pandas matplotlib jupyter
   ```

2. Create a Jupyter notebook:
   ```bash
   # On Ubuntu
   echo '{
    "cells": [
     {
      "cell_type": "code",
      "execution_count": null,
      "metadata": {},
      "source": [
       "import numpy as np\\n",
       "import pandas as pd\\n",
       "import matplotlib.pyplot as plt\\n",
       "\\n",
       "# Generate some data\\n",
       "data = np.random.randn(1000)\\n",
       "\\n",
       "# Create a histogram\\n",
       "plt.hist(data, bins=30)\\n",
       "plt.title(\\"Random Data\\")\\n",
       "plt.show()"
      ]
     }
    ],
    "metadata": {},
    "nbformat": 4,
    "nbformat_minor": 2
   }' > analysis.ipynb
   ```

3. Start Jupyter notebook server:
   ```bash
   # On Ubuntu
   jupyter notebook --no-browser --port=8888
   ```

4. Create an SSH tunnel to access Jupyter:
   ```bash
   # On Mac (in a new terminal)
   ssh -L 8888:localhost:8888 windows-wsl
   ```

5. On your Mac, open a web browser and go to:
   ```
   http://localhost:8888
   ```

6. You'll see the Jupyter interface. Open your notebook and run the code.

7. The visualization will appear in your browser on your Mac, even though the code is running on Ubuntu.

### Example 3: Continuous integration testing

**Scenario**: You're developing an application that needs to be tested on both macOS and Linux.

**Workflow**:

1. Set up a Git repository on your Mac

2. Create a test script that runs on both platforms:
   ```bash
   # On Mac
   echo '#!/bin/bash
   echo "Running tests on $(uname -s)"
   # Add your test commands here
   ' > run_tests.sh
   chmod +x run_tests.sh
   ```

3. Create a script to run tests on both platforms:
   ```bash
   # On Mac
   echo '#!/bin/bash
   echo "Running tests on Mac..."
   ./run_tests.sh
   
   echo "Running tests on Ubuntu..."
   ssh windows-wsl "cd $(pwd) && ./run_tests.sh"
   ' > test_all.sh
   chmod +x test_all.sh
   ```

4. Set up your project on both systems:
   ```bash
   # On Mac
   rsync -avz ~/Projects/my-project/ windows-wsl:~/Projects/my-project/
   ```

5. Run the tests on both platforms:
   ```bash
   # On Mac
   cd ~/Projects/my-project
   ./test_all.sh
   ```

6. The script will run your tests on both macOS and Ubuntu, helping you catch platform-specific issues early.

## What we've accomplished

Great job! You've now learned how to:
- Use your SSH connection for cross-platform development
- Synchronize and manage files between your Mac and Ubuntu
- Execute commands remotely and automate tasks
- Integrate with development tools and IDEs
- Apply these skills to real-world scenarios

Your Mac-to-WSL SSH connection opens up a world of possibilities for development, testing, and automation across different operating systems.

## What's next?

Congratulations on completing this series! You now have a powerful setup that combines the strengths of macOS, Windows, and Linux.

For further learning, consider exploring:
- More advanced SSH features and configurations
- Containerization with Docker in WSL
- Setting up development environments for specific programming languages
- Automating more of your workflow with scripts

Happy coding across platforms!
