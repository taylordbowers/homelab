# Jellyfin

Personal media server. Streams movies, TV shows, and music to any device.

## Setup

- **Host:** jellyfin VM (ID 219) on pve2
- **Image:** `lscr.io/linuxserver/jellyfin:latest`
- **Port:** 8096
- **URL:** `stream.taylorsfunlab.com`
- **GPU:** NVIDIA GTX 980 Ti via PCIe passthrough for hardware transcoding

## Docker Compose

```yaml
jellyfin:
  image: lscr.io/linuxserver/jellyfin:latest
  container_name: jellyfin
  environment:
    - PUID=1000
    - PGID=1000
    - TZ=America/Chicago
  volumes:
    - /docker/jellyfin/config:/config
    - /mnt/media/Movies:/data/movies
    - /mnt/media/Shows:/data/tvshows
  ports:
    - 8096:8096
  devices:
    - /dev/dri:/dev/dri  # GPU passthrough
  restart: unless-stopped
```

## Hardware Transcoding

The jellyfin VM has the GTX 980 Ti passed through directly via Proxmox PCIe passthrough (`hostpci0: 0000:02:00.0`). Jellyfin is configured to use NVENC for hardware-accelerated transcoding, significantly reducing CPU load when streaming to devices that can't direct play.

## Media Library

Media files are stored on the `tank_new` RAIDZ1 array on pve-guide and mounted into this VM via NFS from the media LXC.

| Library | Path |
|---|---|
| Movies | `/data/Movies` (150+ titles) |
| TV Shows | `/data/Shows` (20+ series) |

## Companion Services

- **Jellyseerr** (port 5055) — request management for movies and TV
- **Jellystat** — usage statistics and analytics
