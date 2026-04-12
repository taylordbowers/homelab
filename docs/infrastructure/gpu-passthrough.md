# GPU Passthrough (LXC)

The NVIDIA GTX 980 Ti on pve2 is shared across three LXC containers simultaneously via device bind-mounts.

## GPU Details

- **Card:** NVIDIA GeForce GTX 980 Ti (GM200)
- **VRAM:** 6 GB
- **Host driver:** 535.261.03
- **CUDA:** 12.2
- **Node:** pve2 (10.0.0.2)

## Containers with GPU Access

| CT | Name | GPU Use |
|----|------|---------|
| CT 101 | nextcloud | Available (not actively used yet) |
| CT 103 | jellyfin | NVENC transcoding (h264, hevc, av1) |
| CT 104 | immich | CUDA ML inference (facial recognition, CLIP) |

## LXC Config (`/etc/pve/lxc/NNN.conf`)

Add the following to each container's config:

```
# NVIDIA device cgroup access
lxc.cgroup2.devices.allow: c 195:* rwm
lxc.cgroup2.devices.allow: c 226:* rwm
lxc.cgroup2.devices.allow: c 235:* rwm
lxc.cgroup2.devices.allow: c 241:* rwm

# Device bind-mounts
lxc.mount.entry: /dev/nvidia0 dev/nvidia0 none bind,optional,create=file
lxc.mount.entry: /dev/nvidiactl dev/nvidiactl none bind,optional,create=file
lxc.mount.entry: /dev/nvidia-modeset dev/nvidia-modeset none bind,optional,create=file
lxc.mount.entry: /dev/nvidia-uvm dev/nvidia-uvm none bind,optional,create=file
lxc.mount.entry: /dev/nvidia-uvm-tools dev/nvidia-uvm-tools none bind,optional,create=file
lxc.mount.entry: /dev/dri/card1 dev/dri/card1 none bind,optional,create=file
lxc.mount.entry: /dev/dri/renderD128 dev/dri/renderD128 none bind,optional,create=file
lxc.mount.entry: /dev/nvidia-caps/nvidia-cap1 dev/nvidia-caps/nvidia-cap1 none bind,optional,create=file
lxc.mount.entry: /dev/nvidia-caps/nvidia-cap2 dev/nvidia-caps/nvidia-cap2 none bind,optional,create=file

# Bind-mount host NVIDIA userspace libs to avoid version mismatch
# Ubuntu 24.04 repo has 535.288 but host kernel module is 535.261.03
lxc.mount.entry: /usr/lib/x86_64-linux-gnu/nvidia/current usr/lib/x86_64-linux-gnu/nvidia/current none bind,optional,create=dir

# Required for Docker + AppArmor to work inside the LXC
lxc.apparmor.profile: unconfined
lxc.cap.drop =
```

> **Why `lxc.cap.drop =`?** `/usr/share/lxc/config/common.conf` drops `mac_admin` by default, which prevents Docker from loading AppArmor profiles even in privileged containers. Setting `cap.drop` to empty overrides the drop list.

## Inside Each Container

### 1. Configure ldconfig for NVIDIA libs

```bash
echo '/usr/lib/x86_64-linux-gnu/nvidia/current' > /etc/ld.so.conf.d/nvidia.conf
ldconfig
```

### 2. Install nvidia-container-toolkit

```bash
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg
curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
  sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
  tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
apt update && apt install -y nvidia-container-toolkit
nvidia-ctk runtime configure --runtime=docker
systemctl restart docker
```

### 3. Docker Compose

Use `runtime: nvidia` (not `deploy.resources.reservations.devices` — CDI is not configured):

```yaml
your-service:
  runtime: nvidia
  environment:
    - NVIDIA_VISIBLE_DEVICES=all
    - NVIDIA_DRIVER_CAPABILITIES=compute,video,utility
```

## Verification

From the pve2 host:

```bash
nvidia-smi
# Should show processes from containers using the GPU
```

From inside a container:

```bash
docker run --rm --runtime=nvidia -e NVIDIA_VISIBLE_DEVICES=all nvidia/cuda:12.2.0-base-ubuntu22.04 nvidia-smi
```
