# Immich Compose

The official Immich compose, running on an LXC that has the RTX A3000 passed through.

Reference: https://github.com/immich-app/immich/blob/main/docker/docker-compose.yml

Deviations from upstream:

1. The ML container currently runs the plain `release` (CPU) image. It was pinned there when the GPU was a GTX 980 Ti, which CUDA 12 no longer supports. To use the A3000, switch to `immich-machine-learning:release-cuda` and add `runtime: nvidia` + `NVIDIA_VISIBLE_DEVICES=all`.
2. `MACHINE_LEARNING_DEVICE=cuda` is already set in `.env`, ready for that switch.
3. `UPLOAD_LOCATION` and `DB_DATA_LOCATION` point at an NFS mount from the bulk array on pve-guide (so the photos and DB live on RAIDZ1, not on the LXC's flash pool).

See `.env.example` for the variables that need filling in.
