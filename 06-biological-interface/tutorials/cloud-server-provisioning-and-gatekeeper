# Cloud Server Provisioning & Web Gatekeeper (Nginx)

This document details the provisioning and configuration of the production cloud environment for Zara.

**Functional Overview & Architecture Strategy:**
In this phase, we establish Zara's "cloud brain" and public face. We use a custom Debian 13.x RAW image on DreamCompute (OpenStack) to ensure a clean, untampered operating system. Crucially, we split the server into two distinct volumes: a boot volume for the OS, and a data volume for Zara's knowledge base. This isolates her memory from system updates and makes backups or future migrations highly resilient.

At the application layer, Nginx acts as the "Gatekeeper." It handles secure web traffic (HTTPS) and serves only the compiled, static HTML version of the TiddlyWiki to the public. Meanwhile, the raw markdown data (Zara's active memory) is stored outside of the public web root, ensuring that internal AI processing and raw data manipulation remain strictly secure and invisible to web visitors.

## 1. Infrastructure Provisioning (DreamCompute)

The server infrastructure was built using the following specifications via the DreamCompute dashboard/CLI:

* **OS Image:** Custom Debian 13.x ISO, converted to RAW format and uploaded via SSH.
* **Boot Volume (`zara.boot.01`):** 15 GiB (Attached to the Debian RAW image).
* **Data Volume (`zara.data.01`):** 35 GiB (Dedicated storage for the web root and Zara's data).
* **Compute Instance (`zara.core.01`):** `gp1.subsonic1` flavor (1GB RAM) — optimized and cost-effective for API-driven AI workflows.
* **DNS Routing:** An `A Record` was created with the domain registrar, pointing `systems.anyazelie.com` to the instance's public IP address.

## 2. Formatting and Mounting the Data Volume

Once logged into `zara.core.01` via SSH, the attached 35 GiB data volume must be formatted and mounted to house our web directories. *(Assuming the volume is attached as `/dev/vdb`)*

```bash
# Format the volume to ext4
sudo mkfs.ext4 /dev/vdb

# Create the standard web directory and mount the volume
sudo mkdir -p /var/www
sudo mount /dev/vdb /var/www

```

*(Note: To ensure the mount persists after reboots, the UUID of `/dev/vdb` was added to `/etc/fstab`.)*

## 3. Directory Architecture

We separate the public-facing HTML from the private Markdown data to ensure absolute security.

```bash
# Create the PUBLIC root for Nginx (where the compiled index.html will live)
sudo mkdir -p /var/www/[systems.anyazelie.com/public/memex](https://systems.anyazelie.com/public/memex)

# Create the PRIVATE data directory (where the AI and Node.js will interact with raw Markdown)
sudo mkdir -p /var/www/[systems.anyazelie.com/memex_data](https://systems.anyazelie.com/memex_data)

```

Assign appropriate permissions so your user and the web server can read/write as needed:

```bash
sudo chown -R $USER:$USER /var/www/systems.anyazelie.com

```

## 4. Install Nginx & SSL Utilities

Update the fresh system and install the Nginx web server along with Certbot for automated HTTPS encryption.

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install nginx certbot python3-certbot-nginx -y

```

## 5. Create a Placeholder & Verify DNS

Before configuring SSL, verify that DNS has propagated by creating a simple placeholder index file in the public root.

```bash
echo "<h1>Zara Memex Initializing...</h1>" > /var/www/[systems.anyazelie.com/public/memex/index.html](https://systems.anyazelie.com/public/memex/index.html)

```

From your local machine, ping the server to confirm routing:

```bash
ping systems.anyazelie.com

```

## 6. Configure HTTPS (Certbot)

With Nginx running and the domain resolving correctly, use Certbot to automatically fetch Let's Encrypt SSL certificates and configure Nginx to force HTTPS connections.

```bash
sudo certbot --nginx -d systems.anyazelie.com

```

Navigate to `https://systems.anyazelie.com/memex` in a web browser. You should see the secure padlock icon and your "Zara Memex Initializing..." placeholder text. The gatekeeper is now online and ready to receive the compiled wiki.
