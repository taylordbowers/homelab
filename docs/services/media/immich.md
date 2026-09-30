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
| `immich_machine_learning` | `ghcr.io/immich-app/immich-machine-learning:release-cuda` | Facial recognition + CLIP embeddings (GPU) |
| `immich_postgres` | `tensorchord/pgvecto-rs:pg14-v0.2.0` | Database |
| `immich_redis` | `redis:6.2-alpine` | Cache |
| `whisper-stt` | local build | GPU speech-to-text for the [Byte](../automation/discord-bot.md) and [Jarvis](../automation/jarvis-hud.md) voice agents (unrelated to Immich; it runs here because this CT already has the GPU) |

## GPU Acceleration

The ML container uses the `release-cuda` image with `runtime: nvidia`, so facial recognition and CLIP smart-search embeddings run on the RTX A3000 through ONNX Runtime's CUDA provider:

```yaml
immich-machine-learning:
  image: ghcr.io/immich-app/immich-machine-learning:${IMMICH_VERSION:-release}-cuda
  runtime: nvidia
  environment:
    - NVIDIA_VISIBLE_DEVICES=all
    - NVIDIA_DRIVER_CAPABILITIES=compute,utility
```

Models load on demand and are unloaded after a few idle minutes, so the GPU memory is only held while jobs run. It shares the card with Jellyfin transcodes and the Whisper service without trouble on 6GB.

!!! note "History"
    From April to September 2026 the ML container ran on **CPU**. The old GTX 980 Ti (Maxwell) isn't supported by the CUDA 12 builds that `release-cuda` ships. After the move to the Ampere A3000 it went back to GPU.

!!! tip "Harmless log noise"
    Inside an LXC, ONNX Runtime logs `pthread_setaffinity_np failed … Invalid argument` at model load. It's trying to pin threads to CPUs the container's cpuset doesn't expose. Inference is unaffected.

See [GPU Passthrough](../../infrastructure/gpu-passthrough.md) for full LXC config details.

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
