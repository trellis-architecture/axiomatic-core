# Local Machine Configuration (The Editing Environment)

This tutorial details how to establish a local, Node.js-based TiddlyWiki environment on a **Debian 13.x** machine. 

While TiddlyWiki is famous for its single-file HTML version, this setup utilizes the Node.js (client-server) architecture. This is a critical distinction: the Node.js version separates the user interface from the data, allowing individual tiddlers to be saved as discrete Markdown (`.md`) files directly to your local filesystem. This makes your knowledge base version-controllable, highly resilient, and easily readable by automated scripts or AI agents.

## 1. System Updates and Dependencies

Before installing the wiki, prepare your fresh Debian 13.x environment by updating the system packages and installing the necessary utilities. We will use `vim` as our command-line text editor for this tutorial.

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install curl vim nodejs npm -y
```

## 2. Install TiddlyWiki Globally

With Node.js and the Node Package Manager (`npm`) installed, download and install the TiddlyWiki command-line interface globally across your system.

```bash
sudo npm install -g tiddlywiki
```

## 3. Initialize the Workspace

Create and initialize your local wiki directory. We use the `$USER` variable so these commands work universally for any logged-in user. In this example, our wiki folder is named `memex.local`.

```bash
tiddlywiki /home/$USER/memex.local --init server
```

Immediately after initialization, create the `tiddlers` subdirectory. This prevents permission errors when the server attempts to save system states upon its first launch.

```bash
mkdir -p /home/$USER/memex.local/tiddlers
```

## 4. Configure Essential Plugins (KaTeX, CodeMirror, Relink)

In the standalone HTML version of TiddlyWiki, plugins are installed via drag-and-drop. In the Node.js client-server architecture, plugins are managed structurally. They must be declared in your `tiddlywiki.info` JSON file before the server boots.

The architecture relies on two types of plugins: **Official Plugins** (bundled with your global NPM installation) and **Third-Party Plugins** (which must be manually downloaded to a local directory).

### Step 4A: Install Third-Party Plugins (Relink)
To prevent your knowledge graph from fracturing when a tiddler is renamed, we use the `flibbles/relink` plugin. Because this is a third-party plugin, we must create a local `plugins` directory and download it from the creator's GitHub repository. 

To avoid potential Git authentication issues with public repositories, we will download the source directly as a ZIP archive.

Execute the following sequence in your terminal:

```bash
# Ensure the unzip utility is installed
sudo apt install unzip -y

# Create the publisher directory inside your wiki
mkdir -p /home/$USER/memex.local/plugins/flibbles

# Download the Relink repository as a ZIP archive
wget [https://github.com/flibbles/tw5-relink/archive/refs/heads/master.zip](https://github.com/flibbles/tw5-relink/archive/refs/heads/master.zip) -O /tmp/relink.zip

# Extract the archive into the temporary folder
unzip /tmp/relink.zip -d /tmp/

# Copy the actual plugin folder into your wiki (using a wildcard for the version name)
cp -r /tmp/tw5-relink-*/plugins/relink /home/$USER/memex.local/plugins/flibbles/relink

# Clean up the temporary files
rm -rf /tmp/relink.zip /tmp/tw5-relink-*

```

### Step 4B: Declare Plugins in Configuration

Now we must tell the Node.js server to load Relink, alongside the official physics, coding, and Markdown plugins.

Open your configuration file using `vim`:

```bash
vim /home/$USER/memex.local/tiddlywiki.info

```

Locate the `"plugins"` array. You will add the entire suite here. TiddlyWiki will automatically source the official plugins (like `tiddlywiki/katex`) from your global NPM installation, and it will source `flibbles/relink` from the local folder we just created.

**Target Modification Structure:**
Modify the `"plugins"` array to match this exact list.

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

*CRITICAL: Ensure the new final item (`"tiddlywiki/markdown"`) does NOT have a trailing comma, but the item directly above it now does. Incorrect comma placement will cause a JSON parser failure.*

## 5. Launch and Access the Server

With the directory configured, deploy the local node process:

```bash
tiddlywiki /home/$USER/memex.local --listen
```

Leave your terminal open. Open a web browser on your local machine and navigate to:
http://127.0.0.1:8080

You are now in your editing environment. Any tiddlers you create here (ensure you set their type to "Markdown" in the editor dropdown) will automatically save as individual .md files in your /home/$USER/memex.local/tiddlers directory.

## 6. Creating a Native Launch Command (Quality of Life)

Typing out the full node command every time you want to start your server is inefficient. Furthermore, modern versions of Node.js (v24+) will output harmless but noisy `DeprecationWarning` messages regarding TiddlyWiki's internal use of `url.parse()`. 

We can solve both issues by wrapping the launch parameters into a native system command using a shell script.

First, create a local binaries directory. This is the standard Debian folder for user-specific custom commands:

```bash
mkdir -p ~/.local/bin

```

Next, create the script file and open it using `vim`:

```bash
vim ~/.local/bin/memex

```

Press `i` to enter Insert mode, and paste the following script. This script mutes the Node.js warnings, explicitly targets your directory, and binds it to port 8080.

```bash
#!/bin/bash

echo "Initiating The Trellis (Local Memex) on port 8080..."

# Launch TiddlyWiki with absolute paths and muted Node.js deprecation warnings
NODE_OPTIONS="--no-warnings" tiddlywiki /home/$USER/memex.local --listen port=8080 host=127.0.0.1

```

Save and exit `vim` (Press `Esc`, type `:wq`, and press `Enter`).

Now, tell Debian that this file is an executable application, and reload your terminal environment so it registers the new command:

```bash
chmod +x ~/.local/bin/memex
source ~/.bashrc

```

From now on, no matter where you are in the filesystem, you can simply type `memex` into the terminal to launch your local environment cleanly.

## 7. [Optional] Directory Management & Multiple Instances

Because the Node.js version of TiddlyWiki stores data as flat files, it is highly portable.

**Moving or Renaming a Wiki:**
TiddlyWiki does not strictly care what its parent folder is named. If you want to rename `memex.local` to `knowledge_base`, simply move it using the standard `mv` command:

```bash
mv /home/$USER/memex.local /home/$USER/knowledge_base

```

*Note: If you do this, remember to update the absolute path in your `~/.local/bin/memex` launch script.*

**Running Multiple Instances:**
If you want to create a second, separate wiki (for example, a `memex.public` instance), you must ensure they do not attempt to broadcast on the same network port.

Initialize the new wiki normally:

```bash
tiddlywiki /home/$USER/memex.public --init server
mkdir -p /home/$USER/memex.public/tiddlers

```

When launching the second instance, append the `port=` argument to assign it to an open port (e.g., 8081), keeping port 8080 free for your primary local instance:

```bash
tiddlywiki /home/$USER/memex.public --listen port=8081 host=127.0.0.1

```

## 8. Troubleshooting the Local Environment

When operating a Node.js filesystem server, you may occasionally encounter operational errors. Here are the three most common pitfalls and how to resolve them.

### Error 1: Port Collisions (`EADDRINUSE`)

* **Symptom:** Terminal outputs `Error: listen EADDRINUSE: address already in use 127.0.0.1:8080`.
* **Cause:** Port 8080 is already occupied. This usually means a previous TiddlyWiki process crashed but failed to release the port, or it is still running in another terminal window.
* **Solution:** Find the lingering process and terminate it.
```bash
# Find the Process ID (PID) using the port
lsof -i :8080

# Terminate the process (replace <PID> with the number from the previous command)
kill -9 <PID>

```

### Error 2: The Sudo Trap (`EACCES`)

* **Symptom:** The terminal throws a sync error when you try to save a tiddler: `Error: EACCES: permission denied, open...`
* **Cause:** If you ever accidentally launch the server using `sudo tiddlywiki...`, the operating system assigns ownership of the generated state files to the `root` user. When you later launch the server normally, your standard user account hits a permission wall trying to overwrite those root-owned files.
* **Solution:** Stop the server (`Ctrl+C`), and recursively hand ownership of the entire directory back to your user account.
```bash
sudo chown -R $USER:$USER /home/$USER/memex.local

```
