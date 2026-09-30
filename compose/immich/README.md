# Immich Compose

The official Immich compose, running on an LXC that has the RTX A3000 passed through.

Reference: https://github.com/immich-app/immich/blob/main/docker/docker-compose.yml

Deviations from upstream:

1. `immich_machine_learning` uses `image: ghcr.io/immich-app/immich-machine-learning:${IMMICH_VERSION:-release}-cuda` and `runtime: nvidia` with `NVIDIA_VISIBLE_DEVICES=all`.
2. `immich_server` also gets `runtime: nvidia` (for hardware video transcoding of uploads).
3. `UPLOAD_LOCATION` and `DB_DATA_LOCATION` point at an NFS mount from the bulk array on pve-guide (so the photos and DB live on RAIDZ1, not on the LXC's flash pool).

See `.env.example` for the variables that need filling in.
