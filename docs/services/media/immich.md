# Immich

Self-hosted Google Photos replacement with automatic photo backup, albums, sharing, facial recognition, and smart search.

## Setup

- **Host:** CT 104 (immich LXC) on `pve`
- **IP:** 10.0.0.15
- **Port:** 2283
- **GPU:** NVIDIA RTX A3000 via LXC device passthrough (shared with CT 101 and CT 103)
- **Compose:** `/docker/immich/`

## Container Config

- **Type:** Privileged LXC, Ubuntu 24.04
- **CPU:** 4 cores
- **RAM:** 8 GB
- **Disk:** 64 GB (`flash` ZFS, NVMe)
- **Features:** `nesting=1` (Docker-in-LXC)

## Architecture

| Container | Image | Role |
|-----------|-------|------|
| `immich_server` | `ghcr.io/immich-app/immich-server:release` | Main API + web UI |
| `immich_machine_learning` | `ghcr.io/immich-app/immich-machine-learning:release` | Facial recognition + CLIP embeddings (CPU) |
| `immich_postgres` | `tensorchord/pgvecto-rs:pg14-v0.2.0` | Database |
| `immich_redis` | `redis:6.2-alpine` | Cache |
| `whisper-stt` | local build | GPU speech-to-text for the [Byte](../automation/discord-bot.md) and [Jarvis](../automation/jarvis-hud.md) voice agents (unrelated to Immich; it runs here because this CT already has the GPU) |

## Machine Learning: CPU for now

The ML container runs the **CPU** image. That was forced on the old GTX 980 Ti, since Maxwell cards aren't supported by the CUDA 12 builds that `release-cuda` ships. The current RTX A3000 (Ampere) is fully supported, so moving back to GPU inference means:

1. Switch the ML image to `…-machine-learning:release-cuda`
2. Add `runtime: nvidia` and `NVIDIA_VISIBLE_DEVICES=all` to that service
3. Keep `MACHINE_LEARNING_DEVICE=cuda` in `.env` (already set)

The library is small enough that CPU inference is fine day to day. GPU mainly speeds up a full re-index.

## Data Storage

All photos and the database live on pve-guide's `tank_new` RAIDZ1 array over NFS. Only the application containers run on `pve`.

| Mount | Source | Destination |
|-------|--------|-------------|
| Uploads + DB | `10.0.0.1:/tank_new/subvol-200-disk-0/immich` (NFS) | `/data/immich` |

- `/data/immich/uploads`: photo library
- `/data/immich/db`: Postgres data directory

!!! warning "Empty library? Check the mount first"
    If the NFS mount is missing at startup (for example, pve-guide booted after `pve`), Immich starts against an empty directory and looks like the whole library is gone. Check with `mountpoint /data/immich` before touching the app or the database. Remount, then restart the stack.

## Key Config (.env)

```env
DB_PASSWORD=<set-via-secrets>
DB_USERNAME=postgres
DB_DATABASE_NAME=immich
IMMICH_VERSION=release
DB_HOSTNAME=database
REDIS_HOSTNAME=redis
UPLOAD_LOCATION=/data/immich/uploads
DB_DATA_LOCATION=/data/immich/db
MACHINE_LEARNING_DEVICE=cuda
```
