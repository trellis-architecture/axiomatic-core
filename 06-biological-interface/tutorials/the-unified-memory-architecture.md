# The Unified Memory Architecture

This module establishes the cloud server as the Single Source of Truth for the Trellis architecture. We will provision the Node.js application environment, structure the dual-layer memory directories, and install the Knowledge Base Management System (TiddlyWiki).

*Prerequisite: Ensure you are logged into the **Cloud Server** via SSH, and your Trellis environment variables have been actively sourced in your terminal.*

## 1. System Preparation & Core Dependencies

To align the server with the latest security patches and provide the required runtime environments, we update the package indexes and bulk-install the core dependencies. 

```bash
# Update package lists and upgrade existing software
sudo apt update && sudo apt upgrade -y

# Install the Node.js runtime, package manager, and archive utility
sudo apt install nodejs npm unzip -y

```

With the Node.js environment active, use the Node Package Manager to download and install the TiddlyWiki command-line interface globally across the server.

```bash
# Install TiddlyWiki globally
sudo npm install -g tiddlywiki

```

## 2. Initializing the Memory Architecture

The architecture relies on the physical separation of data. By initializing the Knowledge Base via our mapped environment variables, we ensure public data and private logs remain fundamentally isolated.

First, generate the base TiddlyWiki server structure. This populates the application directory with the required system files.

```bash
# Initialize the server files in the defined application directory
tiddlywiki $APP_WIKI --init server

```

Next, create the specific routing directories for the public and private memory graphs. TiddlyWiki requires these directories to exist before it attempts to write to them.

```bash
# Create the public data repository inside the wiki application directory
mkdir -p $DATA_PUB

# Create the private data repository securely outside the application root
mkdir -p $DATA_PRIV

```

## 3. Configure Essential Plugins

In the Node.js architecture, plugins are managed structurally and must be declared in the `tiddlywiki.info` configuration file before the server boots.

First, download and install the third-party `flibbles/relink` plugin. This plugin automatically updates all interconnected links across your graph if a tiddler is ever renamed, preventing orphaned data and broken relationships.

```bash
# Create the publisher directory inside your wiki's plugin folder
mkdir -p $APP_WIKI/plugins/flibbles

# Download the Relink repository as a ZIP archive to a temporary location
wget [https://github.com/flibbles/tw5-relink/archive/refs/heads/master.zip](https://github.com/flibbles/tw5-relink/archive/refs/heads/master.zip) -O /tmp/relink.zip

# Extract the archive
unzip /tmp/relink.zip -d /tmp/

# Copy the core plugin folder into your wiki (using a wildcard for versioning)
cp -r /tmp/tw5-relink-*/plugins/relink $APP_WIKI/plugins/flibbles/relink

# Clean up the temporary files
rm -rf /tmp/relink.zip /tmp/tw5-relink-*

```

Next, tell the application to load Relink alongside the official plugins (which handle Markdown parsing, KaTeX mathematical rendering, and the CodeMirror text editor).

Open the configuration file:

```bash
$EDITOR$APP_WIKI/tiddlywiki.info

```

Locate the `"plugins"` array and update it to match this exact list.

```json
    "plugins": [
        "tiddlywiki/tiddlyweb",
        "tiddlywiki/filesystem",
        "tiddlywiki/markdown",
        "tiddlywiki/katex",
        "tiddlywiki/highlight",
        "tiddlywiki/codemirror",
        "tiddlywiki/codemirror-closebrackets",
        "tiddlywiki/codemirror-closetag",
        "tiddlywiki/codemirror-fullscreen",
        "tiddlywiki/codemirror-keymap-vim",
        "tiddlywiki/codemirror-mode-css",
        "tiddlywiki/codemirror-mode-htmlmixed",
        "tiddlywiki/codemirror-mode-javascript",
        "tiddlywiki/codemirror-mode-markdown",
        "tiddlywiki/codemirror-mode-xml",
        "tiddlywiki/codemirror-search-replace",
        "flibbles/relink"
    ],

```

*Critical: Ensure every item ends with a comma EXCEPT the final item (`"flibbles/relink"`). An errant trailing comma will cause a fatal JSON parsing error, preventing the server from booting.*

## 4. The Persistent Background Service (systemd)

If we launch TiddlyWiki directly in the terminal, the process will die the moment you disconnect your SSH session. To ensure the Soliton's memory interface is always online, we configure it as a native `systemd` background service.

Because `systemd` configuration files cannot dynamically read user-level environment variables, we use a shell command to evaluate our variables and inject them directly into a new service file.

Run the following command block entirely:

```bash
# Generate the systemd service file with environment variables injected
sudo bash -c "cat << EOF > /etc/systemd/system/${SOLITON}-wiki.service
[Unit]
Description=Trellis Knowledge Base - Private Read/Write Interface
After=network.target

[Service]
Type=simple
User=$REMOTE_USER
WorkingDirectory=$APP_WIKI
ExecStart=/usr/bin/env tiddlywiki $APP_WIKI --listen port=8080 host=127.0.0.1
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF"

```

With the service file created, instruct Debian to reload its daemon manager, enable the service to start automatically on system boot, and launch it immediately.

```bash
# Reload the systemd manager to register the new file
sudo systemctl daemon-reload

# Enable the service to start automatically upon server reboots
sudo systemctl enable ${SOLITON}-wiki.service

# Start the background service immediately
sudo systemctl start ${SOLITON}-wiki.service

```

To verify that the service is running successfully and broadcasting on port 8080, check its status:

```bash
sudo systemctl status ${SOLITON}-wiki.service

```

*(Press `q` to exit the status view).*

## 5. The Local Client Integration

*Prerequisite: Ensure you have returned to your **Local Machine's** terminal, that the local environment variables are sourced, and that the `sshfs` package is installed (`sudo apt install sshfs -y`).*

To interface with the cloud server, we need to securely mount the remote markdown files to our local filesystem and establish an SSH tunnel for the web UI. While standard SSH configurations (`~/.ssh/config`) can handle port forwarding, they cannot execute filesystem mounts or query remote variables dynamically. 

To achieve this, we will write a session orchestrator script (`$SOLITON_OS`). This script queries the cloud server for its architectural ground truth, mounts the data layers locally, and opens a secure tunnel to the background Knowledge Base we created in Section 4.

Create a local binaries directory and open the new script file:

```bash
# Create the local bin directory if it does not exist
mkdir -p ~/.local/bin

# Open the orchestrator script using your preferred editor
$EDITOR ~/.local/bin/$SOLITON_OS

```

Paste the following script. *Note: Because this is an executable script, we write the literal bash variables (e.g., `$SERVER_IP`) rather than having the text editor expand them.*

```bash
#!/bin/bash

echo "Connecting..."

# 1. Ensure local mount points exist
mkdir -p "$LOCAL_MNT/data-public"
mkdir -p "$LOCAL_MNT/data-private"

# 2. Query the cloud server for the architectural Ground Truth paths
echo "Retrieving remote directory mappings..."
REMOTE_PUB=$(ssh -i "$SSH_KEY" "$REMOTE_USER@$SERVER_IP" "source ~/.trellis_profile && echo \$DATA_PUB")
REMOTE_PRIV=$(ssh -i "$SSH_KEY" "$REMOTE_USER@$SERVER_IP" "source ~/.trellis_profile && echo \$DATA_PRIV")

# 3. Mount the data layers via SSHFS (verifying they aren't already mounted)
if ! mountpoint -q "$LOCAL_MNT/data-public"; then
    echo "Mounting public data layer..."
    sshfs -o IdentityFile="$SSH_KEY" "$REMOTE_USER@$SERVER_IP:$REMOTE_PUB" "$LOCAL_MNT/data-public"
fi

if ! mountpoint -q "$LOCAL_MNT/data-private"; then
    echo "Mounting private data layer..."
    sshfs -o IdentityFile="$SSH_KEY" "$REMOTE_USER@$SERVER_IP:$REMOTE_PRIV" "$LOCAL_MNT/data-private"
fi

# 4. Open interactive shell with secure UI Port Forwarding
echo "Establishing secure UI tunnel on localhost:8080..."
ssh -i "$SSH_KEY" -L 8080:127.0.0.1:8080 "${REMOTE_USER}@${SERVER_IP}"

```

Save and exit the text editor.

Finally, instruct your local Debian environment to treat this file as an executable application, and reload your terminal so the new command registers:

```bash
# Make the script executable
chmod +x ~/.local/bin/$SOLITON_OS

# Reload the bash profile to register the new command path
source ~/.bashrc

```

### 6. Entering the Environment

The architecture is now fully integrated. From your local machine, run your newly minted command:

```bash
$SOLITON_OS

```

Once the connection is established, leave the terminal window open. You can now open a local web browser and navigate to `http://127.0.0.1:8080`. You are securely interfacing with the cloud-hosted Knowledge Base.

Simultaneously, you can open your local `$LOCAL_MNT` folder and use your local text editors to modify the raw Markdown files, with all changes saving directly to the cloud. When you type `exit` in the terminal to close the shell, the SSHFS connection will safely persist until you unmount it or reboot your local machine.
