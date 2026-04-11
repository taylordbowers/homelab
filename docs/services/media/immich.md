# Immich

Self-hosted Google Photos replacement. Automatic photo backup, albums, sharing, and AI-powered facial recognition and smart search.

## Setup

- **Host:** CT 104 (immich LXC) on pve2
- **IP:** 192.168.1.179
- **Port:** 2283
- **GPU:** NVIDIA GTX 980 Ti via LXC device passthrough (shared with CT 101 and CT 103)

> **Migrated 2026-04-10:** Previously ran inside VM 219 alongside Jellyfin. Split into its own dedicated LXC container. GPU access is now via LXC device passthrough rather than PCIe passthrough, allowing the GPU to be shared with Jellyfin and Nextcloud simultaneously.

## Container Config

- **Type:** Privileged LXC, Ubuntu 24.04
- **CPU:** 4 cores
- **RAM:** 8 GB
- **Disk:** 64 GB (flash ZFS)
- **Features:** `nesting=1` (Docker-in-LXC)
- **Autostart:** `systemd: immich.service`

## Architecture

| Container | Image | Role |
|-----------|-------|------|
| `immich_server` | `ghcr.io/immich-app/immich-server:release` | Main API + web UI |
| `immich_machine_learning` | `ghcr.io/immich-app/immich-machine-learning:release-cuda` | Facial recognition + CLIP embeddings (GPU) |
| `immich_postgres` | `tensorchord/pgvecto-rs:pg14` | Database |
| `immich_redis` | `redis:6.2-alpine` | Cache |

## GPU Acceleration

The machine learning container uses the `release-cuda` image for GPU-accelerated inference on the GTX 980 Ti via `runtime: nvidia`. This enables:

- **Facial recognition** — clusters faces into people automatically
- **Smart search** — CLIP embeddings for natural language photo search

See [GPU Passthrough](../../infrastructure/gpu-passthrough.md) for full LXC config details.

## Data Storage

All photo and database data lives on pve-guide's `tank_new` ZFS array — it was never moved during the migration.

| Mount | Source | Destination |
|-------|--------|-------------|
| Uploads + DB | `192.168.1.118:/tank_new/subvol-200-disk-0/immich` (NFS) | `/data/immich` |

- `/data/immich/uploads` — photo library
- `/data/immich/db` — postgres data directory

## Key Config (.env)

```env
DB_PASSWORD=immich_db_pass
DB_USERNAME=postgres
DB_DATABASE_NAME=immich
IMMICH_VERSION=release
DB_HOSTNAME=database
REDIS_HOSTNAME=redis
UPLOAD_LOCATION=/data/immich/uploads
DB_DATA_LOCATION=/data/immich/db
MACHINE_LEARNING_DEVICE=cuda
```
