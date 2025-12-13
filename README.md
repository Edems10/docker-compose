# Media Server with Docker Compose & NVIDIA Transcoding

This repository contains a production-ready Docker Compose setup for a secure, automated media server. It uses **Gluetun** for a VPN kill-switch with port-forwarding support, **NVIDIA GPU** for Plex transcoding, and the **"Arr" stack** for automated media management.

## Prerequisites

Before you begin, ensure your system meets these requirements:

### 1. Docker & Docker Compose
You must have Docker installed.
```bash
# Install Docker Official (Ubuntu)
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```

### 2. NVIDIA GPU Drivers (Optional)
**Required only if you want hardware transcoding in Plex.**
If you do not have an NVIDIA GPU, you can skip this, but you must edit the `compose.yml` (see "No GPU Configuration" below).

1.  Install NVIDIA Drivers: `sudo apt install nvidia-driver-535`
2.  Install **NVIDIA Container Toolkit** (Crucial for Docker to see the GPU):
    ```bash
    sudo apt-get install -y nvidia-container-toolkit
    sudo nvidia-ctk runtime configure --runtime=docker
    sudo systemctl restart docker
    ```

### 3. VPN Credentials
You need a VPN provider that supports **Port Forwarding** for high-speed torrenting.
*   **Recommended:** Private Internet Access (PIA) or ProtonVPN.
*   *Note:* NordVPN is supported but generally does not allow port forwarding, which may slow down seeding.

***

## Architecture & Services

The services are grouped by function to keep the system modular and secure.

### Connectivity & Security
*   **Gluetun (VPN)**: The heart of the network stack. It creates a secure WireGuard tunnel to your VPN provider. All download traffic goes through this container. It acts as a **Kill Switch**—if the VPN drops, internet access is cut instantly.
*   **Port-Updater**: A sidecar automation bot. It continuously checks your VPN's forwarded port and automatically updates qBittorrent settings. No manual intervention required.

### Downloaders
*   **qBittorrent**: A lightweight torrent client. It has no direct internet access; it is routed entirely through `Gluetun`. It saves files to the unified `/data/torrents` folder.

### Media Server
*   **Plex**: The streaming server. It is configured to use your **NVIDIA GPU** to transcode video streams, offloading the work from your CPU. It streams media from `/data/media`.

### The "Arr" Automation Stack
*   **Radarr**: Movies manager.
*   **Sonarr**: TV Series manager.
*   **Readarr**: Audiobooks manager.
*   **Prowlarr**: Indexer manager. It connects to torrent sites and syncs them to Radarr/Sonarr.
*   **Overseerr**: Allows users to request movies/TV shows without accessing the backend apps.

***

## Directory Structure (The "Hardlink" Setup)
This setup uses a **Unified Volume** strategy (`/data`). This allows file moves to be instant (atomic) and saves hard drive space by using hardlinks instead of copying files.

**Host Path** -> **Container Path**
*   `/pool0/Heh/data/torrents` -> `/data/torrents`
*   `/pool0/Heh/data/media` -> `/data/media`

***

## Installation Guide

### Step 1: Automatic Folder Setup
Run the included helper script to create the optimized directory structure and configuration file.

1.  Save the code below as `setup.sh`:
    ```bash
    #!/bin/bash
    BASE_PATH="/pool0/Heh"
    CONFIG_PATH="/docker_config"

    echo "Creating folder structure..."
    sudo mkdir -p "$BASE_PATH/data/torrents"
    sudo mkdir -p "$BASE_PATH/data/media/movies"
    sudo mkdir -p "$BASE_PATH/data/media/tv"
    sudo mkdir -p "$BASE_PATH/data/media/audiobooks"
    
    echo "Generating .env file..."
    cat <<EOF > .env
    # System
    PUID=1000
    PGID=1000
    TZ=Europe/Prague
    ROOT_DATA_DIR=$BASE_PATH/data
    CONFIG_DIR=$CONFIG_PATH

    # VPN (Private Internet Access)
    VPN_USER=replace_me
    VPN_PASSWORD=replace_me
    VPN_REGION=de-frankfurt.privacy.network

    # Plex
    PLEX_CLAIM=optional_claim_token
    EOF
    ```
2.  Run it: `bash setup.sh`

### Step 2: Configure Secrets
Open the newly created `.env` file and fill in your details:
*   `VPN_USER` / `VPN_PASSWORD`: Your PIA/VPN credentials.
*   `PLEX_CLAIM`: (Optional) Token from [plex.tv/claim](https://plex.tv/claim) for easy login.

### Step 3: Start the Stack
```bash
docker compose up -d
```

***

## Configuration & Customization

### Option: I don't have an NVIDIA GPU
If you are running this on a standard CPU without a graphics card, you must disable the GPU runtime to prevent errors.

Open `compose.yml`, find the **Plex** service, and **remove/comment out** these lines:

```yaml
  plex:
    # ...
    # DELETE THESE LINES IF NO GPU:
    # environment:
    #   - NVIDIA_VISIBLE_DEVICES=all
    #   - NVIDIA_DRIVER_CAPABILITIES=all
    # runtime: nvidia
```

### Accessing the Services
Once running, access your services at your server's IP (e.g., `192.168.1.50`):

| Service | Port | URL |
| :--- | :--- | :--- |
| **Homepage** | 3000 | `http://IP:3000` |
| **Plex** | 32400 | `http://IP:32400/web` |
| **qBittorrent** | 8080 | `http://IP:8080` |
| **Overseerr** | 5055 | `http://IP:5055` |
| **Sonarr** | 8989 | `http://IP:8989` |
| **Radarr** | 7878 | `http://IP:7878` |

### Legal Disclaimer & Usage Note
>
> This project is intended for managing **personally owned media libraries** (backups of physical discs you own) and for sharing **open-source software** (such as Linux ISOs) via the BitTorrent protocol.
>
> *   **Personal Backups:** The media management tools ("Arr" stack) are designed to organize valid personal backups of content you have legally purchased.
> *   **Linux ISOs:** The torrent client is optimized for seeding and downloading large open-source distributions (e.g., Ubuntu, Debian, Arch) which rely on peer-to-peer sharing for bandwidth efficiency.
> *   **Respect Copyright:** The authors of this repository do not condone or support the downloading or distribution of copyrighted material without permission. Users are responsible for ensuring their usage complies with all applicable local laws and regulations.

