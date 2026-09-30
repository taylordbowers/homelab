# Jellyfin

Personal media server that streams movies, TV, and music to any device.

## Setup

- **Host:** CT 103 (jellyfin LXC) on `pve`
- **IP:** 10.0.0.14
- **Port:** 8096
- **External access:** behind Nginx Proxy Manager with TLS
- **GPU:** NVIDIA RTX A3000 via LXC device passthrough (shared with CT 101 and CT 104)

## Container Config

- **Type:** Privileged LXC, Ubuntu 24.04
- **CPU:** 4 cores
- **RAM:** 8 GB
- **Disk:** 64 GB (`flash` ZFS, NVMe)
- **Features:** `nesting=1` (Docker-in-LXC)
- **Compose:** `/docker/jellyfin/compose.yaml`

## Services in This Stack

| Service | Port | Description |
|---------|------|-------------|
| Jellyfin | 8096 | Media server |
| Jellyseerr | 5055 | Movie/TV requests, connected to Radarr and Sonarr on VM 119. Now runs the `seerr/seerr` image (the merged Overseerr/Jellyseerr project) |
| Jellystat | 3000 | Usage statistics dashboard |
| Jellystat-DB | — | Postgres 15.2 backend for Jellystat |

## Docker Compose (key sections)

```yaml
jellyfin:
  image: lscr.io/linuxserver/jellyfin:latest
  runtime: nvidia
  environment:
    - PUID=1000
    - PGID=993
    - TZ=America/Chicago
    - NVIDIA_VISIBLE_DEVICES=all
    - NVIDIA_DRIVER_CAPABILITIES=compute,video,utility
    - JELLYFIN_PublishedServerUrl=http://10.0.0.14
  volumes:
    - /docker/jellyfin/config:/config
    - /data:/data
  ports:
    - 8096:8096
    - 7359:7359/udp   # client auto-discovery
    - 1900:1900/udp   # DLNA
  restart: unless-stopped

jellyseerr:
  image: seerr/seerr:latest
  environment:
    - TZ=America/Chicago
  volumes:
    - /docker/jellyfin/jellyseerr:/app/config
  ports:
    - 5055:5055
  restart: unless-stopped

jellystat:
  image: cyfershepard/jellystat:latest
  environment:
    - POSTGRES_USER=postgres
    - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
    - POSTGRES_IP=jellystat-db
    - POSTGRES_PORT=5432
    - JWT_SECRET=${JWT_SECRET}
    - TZ=America/Chicago
  ports:
    - 3000:3000
  depends_on:
    - jellystat-db
  restart: unless-stopped
```

The full file is in [`compose/jellyfin/`](https://github.com/taylordbowers/homelab/tree/main/compose/jellyfin).

## Hardware Transcoding

The RTX A3000 is shared through LXC device passthrough. `nvidia-container-toolkit` runs inside the container, and Docker uses the `nvidia` runtime. NVENC encode tests pass:

- `h264_nvenc` ✓
- `hevc_nvenc` ✓
- `av1_nvenc` ✓ (new with the Ampere card; the old GTX 980 Ti couldn't encode AV1)

See [GPU Passthrough](../../infrastructure/gpu-passthrough.md) for the full LXC config.

## Data Mounts

| Mount | Source | Destination |
|-------|--------|-------------|
| Media files | `//10.0.0.20/data` (SMB from CT 200 on pve-guide) | `/data` |

The SMB share lives on the other node, so if CT 200 comes up late after a power event the mount can be missing at boot. An `ensure-data-mount.timer` inside the CT re-checks every couple of minutes and remounts when needed.

## Media Library

| Library | Path |
|---------|------|
| Movies | `/data/Movies` (~200 titles) |
| TV Shows | `/data/Shows` (~25 series) |
| Music | `/data/music` |

Physical discs are ripped and remuxed into the library with a small queue script (`remux-queue.sh`) in the stack directory.
