# Immich Compose

The official Immich compose pulled with `release-cuda` for the ML container so the GTX 980 Ti is used for facial recognition + CLIP embeddings.

Reference: https://github.com/immich-app/immich/blob/main/docker/docker-compose.yml

The only deviations from upstream:

1. `immich_machine_learning` uses `image: ghcr.io/immich-app/immich-machine-learning:release-cuda` and `runtime: nvidia` with `NVIDIA_VISIBLE_DEVICES=all`.
2. `MACHINE_LEARNING_DEVICE=cuda` set in `.env` so the app actually targets the GPU.
3. `UPLOAD_LOCATION` and `DB_DATA_LOCATION` point at an NFS mount from the bulk array on pve-guide (so the photos and DB live on RAIDZ1, not on the LXC's flash pool).

See `.env.example` for the variables that need filling in.
