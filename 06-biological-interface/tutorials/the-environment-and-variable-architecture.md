# The Environment and Variable Architecture

To prevent configuration drift between your machines, we enforce a **"Need-to-Know" Split**:
*   **The Cloud Profile:** Holds the architectural ground truth (directories, web roots, domains).
*   **The Local Profile:** Holds only the networking keys required to connect to the cloud.

## 1. The Variable Dictionaries

The architecture relies on the following variables, split by their respective environments.

### The Cloud Variables (Architectural Ground Truth)
| Variable | Purpose | Example Value |
| :--- | :--- | :--- |
| `$DOMAIN` | The target URL for web traffic | `systems.anyazelie.com` |
| `$BASE_DIR` | The OS-level directory for web servers | `/var/www` |
| `$WEB_ROOT` | The parent directory for this specific domain | `$BASE_DIR/$DOMAIN` |
| `$APP_TRELLIS` | The backend scripts and processes | `$WEB_ROOT/trellis` |
| `$APP_WIKI` | The Node.js TiddlyWiki application directory | `$WEB_ROOT/wiki` |
| `$DATA_PUB` | The visible Markdown memory (read-only to the public) | `$APP_WIKI/data-public` |
| `$DATA_PRIV` | The hidden Markdown memory (private data and logs) | `$WEB_ROOT/data-private` |
| `$SOLITON` | The designation of the autopoietic agent | `zara` |
| `$EDITOR` | The system's preferred command-line text editor | `vim` |

*Architectural Note: `$DATA_PRIV` is intentionally placed outside of `$APP_WIKI`. If the web application is ever compromised, the private data layer remains physically isolated from the application directory.*

### The Local Variables (Client Connection)
| Variable | Purpose | Example Value |
| :--- | :--- | :--- |
| `$SOLITON` | The designation of the autopoietic agent | `zara` |
| `$EDITOR` | The system's preferred command-line text editor | `vim` |
| `$SERVER_IP` | The IPv4 address of the cloud server | `216.58.194.174` |
| `$REMOTE_USER` | The sudo-enabled user on the cloud server | `debian` |
| `$SSH_KEY` | The absolute path to the local SSH private key | `~/.ssh/id_ed25519` |
| `$LOCAL_MNT` | The local directory where remote files will appear | `~/mnt/$SOLITON` |
| `$SOLITON_OS` | The master command to connect to the Trellis | `zos` |

## 2. Establishing the Cloud Profile

*Prerequisite: SSH into your **Cloud Server**.*

Write the architectural variables to a dedicated environment file, replacing the placeholder text with your actual system parameters.

```bash
# Write the variable block to a hidden file in the remote user's home directory
cat << 'EOF' > ~/.trellis_profile
export DOMAIN="DOMAIN_NAME"
export BASE_DIR="/var/www"
export WEB_ROOT="$BASE_DIR/$DOMAIN"
export APP_TRELLIS="$WEB_ROOT/trellis"
export APP_WIKI="$WEB_ROOT/wiki"
export DATA_PUB="$APP_WIKI/data-public"
export DATA_PRIV="$WEB_ROOT/data-private"
export SOLITON="AGENT_NAME"
export EDITOR="TEXT_EDITOR"
EOF

```

## 3. Establishing the Local Profile

*Prerequisite: Open a terminal on your **Local Machine**.*

Write the networking variables to a local environment file, replacing the placeholder text with your actual connection credentials.

```bash
# Write the variable block to a hidden file in the local user's home directory
cat << 'EOF' > ~/.trellis_profile
export SOLITON="AGENT_NAME"
export EDITOR="TEXT_EDITOR"
export SERVER_IP="IPV4_ADDRESS"
export REMOTE_USER="REMOTE_USERNAME"
export SSH_KEY="$HOME/.ssh/PRIVATE_KEY_FILE"
export LOCAL_MNT="$HOME/mnt/$SOLITON"
export SOLITON_OS="MASTER_COMMAND"
EOF

```

## 4. Activation and Verification

To ensure these profiles load automatically upon login, execute the following commands on **both** your Cloud Server and your Local Machine:

```bash
# Append the source command to the end of the bashrc file
echo "source ~/.trellis_profile" >> ~/.bashrc

# Reload the current terminal environment to activate the variables
source ~/.bashrc

```

To confirm the variables are actively held in the system's memory, echo a test variable to the terminal.

**On the Cloud Server:**

```bash
# Verify architectural expansion
echo $DATA_PRIV

```

**On the Local Machine:**

```bash
# Verify networking expansion
echo $LOCAL_MNT

```

If the terminals output the resolved paths, your partitioned environment architecture is correctly established.
