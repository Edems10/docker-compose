# Media Server Setup with Docker Compose for plex with Nvidia GPU for transcoding

This repository contains a Docker Compose setup for a complete media server environment. The setup includes a VPN service, torrent client, media server applications, and other supporting services.

## Services

### 1. VPN (NordLynx)
- **Image**: ghcr.io/bubuntux/nordlynx
- **Purpose**: Provides a secure VPN connection using NordLynx (WireGuard).
- **Environment Variables**:
  - `TOKEN`: Your NordVPN API token.
  - `PRIVATE_KEY`: WireGuard private key for NordLynx.
  - `CONNECT`: The location to connect to (e.g., Czech_Republic).
  - `NETWORK`: Local network range (e.g., 192.168.1.0/24).

### 2. qBittorrent
- **Image**: ghcr.io/linuxserver/qbittorrent
- **Purpose**: A torrent client that routes all traffic through the VPN.
- **Environment Variables**:
  - `PUID=1000`
  - `PGID=1000`
- **Volumes**:
  - `/docker_config/qbittorrent:/config`: Configuration files.
  - `/pool0/Heh/Series:/downloads`: Download location for torrents.

### 3. Plex
- **Image**: lscr.io/linuxserver/plex:latest
- **Purpose**: A media server that streams your media content.
- **Environment Variables**:
  - `PUID=1000`
  - `PGID=1000`
  - `TZ=Etc/UTC`
  - `VERSION=docker`
  - `PLEX_CLAIM`: Optional claim token for server linking.
  - `NVIDIA_VISIBLE_DEVICES=all`: Enables GPU acceleration.
- **Volumes**:
  - `/docker_config/plex:/config`: Configuration files.
  - `/pool0/Heh/Series:/tv`: TV series library.
  - `/pool0/Heh/Movies:/movies`: Movies library.

### 4. Radarr
- **Image**: lscr.io/linuxserver/radarr:latest
- **Purpose**: A movie collection manager.
- **Environment Variables**:
  - `PUID=1000`
  - `PGID=1000`
  - `TZ=Etc/UTC`
- **Volumes**:
  - `/docker_config/radarr:/config`: Configuration files.
  - `/pool0/Heh/Movies:/movies`: Movies library.
  - `/pool0/Heh/Torrents_mov:/downloads`: Download location for movie torrents.

### 5. Sonarr
- **Image**: lscr.io/linuxserver/sonarr:latest
- **Purpose**: A TV series collection manager.
- **Environment Variables**:
  - `PUID=1000`
  - `PGID=1000`
  - `TZ=Etc/UTC`
- **Volumes**:
  - `/docker_config/sonarr:/config`: Configuration files.
  - `/pool0/Heh/Series:/tv`: TV series library.
  - `/pool0/Heh/Torrents_ser:/downloads`: Download location for TV torrents.

### 6. Overseerr
- **Image**: lscr.io/linuxserver/overseerr:latest
- **Purpose**: A request management and media discovery tool.
- **Environment Variables**:
  - `PUID=1000`
  - `PGID=1000`
  - `TZ=Etc/UTC`
- **Volumes**:
  - `/docker_config/overseerr:/config`: Configuration files.

### 7. Jackett
- **Image**: lscr.io/linuxserver/jackett:latest
- **Purpose**: A proxy server for torrent indexers.
- **Environment Variables**:
  - `PUID=1000`
  - `PGID=1000`
  - `TZ=Etc/UTC`
  - `AUTO_UPDATE=true` (optional): Enables automatic updates.
- **Volumes**:
  - `/docker_config/jackett:/config`: Configuration files.
  - `/pool0/Heh/blackhole:/downloads`: Download location for torrents.

## Prerequisites
- Docker and Docker Compose installed on your system.
- Valid NordVPN account with API access.
