# Immich

Self-hosted Google Photos replacement. Automatic photo backup, albums, sharing, and AI-powered facial recognition and smart search.

## Setup

- **Host:** jellyfin VM (ID 219) on pve2
- **Port:** 2283
- **GPU:** NVIDIA GTX 980 Ti — used for CUDA-accelerated ML inference
- **Photo library:** NFS-mounted from media LXC (`/data/immich`, 4.5TB)

## Architecture

Immich runs as a multi-container stack:

| Container | Image | Role |
|---|---|---|
| `immich_server` | `immich-app/immich-server:release` | Main API + web UI |
| `immich_machine_learning` | `immich-app/immich-machine-learning:release-cuda` | Facial recognition + CLIP embeddings |
| `immich_postgres` | `tensorchord/pgvecto-rs:pg14` | Database |
| `immich_redis` | `redis:6.2-alpine` | Cache |

## GPU Acceleration

The machine learning container uses the `release-cuda` image to run inference on the GTX 980 Ti. This enables:

- **Facial recognition** — clusters faces into people automatically
- **Smart search** — CLIP embeddings for natural language photo search (e.g. "dog at the beach")

The NVIDIA Container Toolkit (`nvidia-container-toolkit`) is installed in the jellyfin VM, and Docker is configured with `nvidia` as the default runtime.

### Key config

`docker-compose.yml` — ML service:
```yaml
immich-machine-learning:
  image: ghcr.io/immich-app/immich-machine-learning:${IMMICH_VERSION:-release}-cuda
  deploy:
    resources:
      reservations:
        devices:
          - driver: nvidia
            count: 1
            capabilities: [gpu]
```

`.env`:
```env
MACHINE_LEARNING_DEVICE=cuda
```

## Docker Compose

```yaml
name: immich

services:
  immich-server:
    container_name: immich_server
    image: ghcr.io/immich-app/immich-server:${IMMICH_VERSION:-release}
    volumes:
      - ${UPLOAD_LOCATION}:/usr/src/app/upload
      - /mnt/photos:/mnt/photos
      - /etc/localtime:/etc/localtime:ro
    env_file:
      - .env
    ports:
      - 2283:2283
    depends_on:
      - redis
      - database
    restart: always
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: 1
              capabilities: [gpu]

  immich-machine-learning:
    container_name: immich_machine_learning
    image: ghcr.io/immich-app/immich-machine-learning:${IMMICH_VERSION:-release}-cuda
    volumes:
      - model-cache:/cache
      - ${UPLOAD_LOCATION}:/usr/src/app/upload:ro
    env_file:
      - .env
    restart: always
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: 1
              capabilities: [gpu]

  redis:
    container_name: immich_redis
    image: docker.io/redis:6.2-alpine
    healthcheck:
      test: redis-cli ping || exit 1
    restart: always

  database:
    container_name: immich_postgres
    image: docker.io/tensorchord/pgvecto-rs:pg14-v0.2.0
    environment:
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      POSTGRES_USER: ${DB_USERNAME}
      POSTGRES_DB: ${DB_DATABASE_NAME}
    volumes:
      - ${DB_DATA_LOCATION}:/var/lib/postgresql/data
    restart: always

volumes:
  model-cache:
```

## Notes

- Immich was originally run in an LXC (CT 220) but was migrated to the jellyfin VM to gain GPU access for ML inference — LXCs cannot do PCIe passthrough
- Photo data lives on the media LXC's `tank_new` pool and is NFS-mounted into the VM
