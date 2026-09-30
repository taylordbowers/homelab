# GPU Passthrough (LXC)

The NVIDIA RTX A3000 Laptop GPU on `pve` is shared by three LXC containers at once. They get the device nodes directly, not a full-GPU VM passthrough, so any number of containers can use the card together.

## GPU Details

- **Card:** NVIDIA RTX A3000 Laptop GPU (Ampere, GA104)
- **VRAM:** 6 GB
- **Host driver:** NVIDIA production branch, installed on the host. Containers bind-mount the host's userspace libraries so the versions always match
- **Node:** pve (10.0.0.3)
- **Encoders:** NVENC h264 / hevc / **av1** (the retired GTX 980 Ti had no AV1 encode)

## Containers with GPU Access

| CT | Name | GPU Use |
|----|------|---------|
| CT 101 | nextcloud | Available (not actively used) |
| CT 103 | jellyfin | NVENC/NVDEC transcoding |
| CT 104 | immich | GPU speech-to-text service (Whisper) for the voice agents. Immich's ML container currently runs on CPU (see [Immich](../services/media/immich.md)) |

## LXC Config (`/etc/pve/lxc/NNN.conf`)

Proxmox VE 8 supports `devN:` entries, which replace most of the hand-written `lxc.cgroup2` / `lxc.mount.entry` lines needed before:

```
# NVIDIA device nodes (Proxmox 8 device passthrough)
dev0: path=/dev/nvidia0
dev1: path=/dev/nvidiactl
dev2: path=/dev/nvidia-modeset
dev3: path=/dev/nvidia-uvm
dev4: path=/dev/nvidia-uvm-tools
dev5: path=/dev/nvidia-caps/nvidia-cap1
dev6: path=/dev/nvidia-caps/nvidia-cap2

# DRI nodes: the laptop also has an Intel iGPU, so the NVIDIA card is
# card0/renderD129 on the host. Remap it to the names apps expect.
lxc.cgroup2.devices.allow: c 226:* rwm
lxc.mount.entry: /dev/dri/card0 dev/dri/card1 none bind,optional,create=file
lxc.mount.entry: /dev/dri/renderD129 dev/dri/renderD128 none bind,optional,create=file

# Bind-mount the host's NVIDIA userspace libs (read-only) so they always
# match the host kernel module; no driver install inside the CT
lxc.mount.entry: /usr/lib/x86_64-linux-gnu/nvidia/current usr/lib/x86_64-linux-gnu/nvidia/current none bind,ro,optional,create=dir

# Docker-in-LXC only (jellyfin/immich): let Docker manage AppArmor
lxc.apparmor.profile: unconfined
lxc.cap.drop:
```

> **Why `lxc.cap.drop:` (empty)?** `/usr/share/lxc/config/common.conf` drops `mac_admin` by default, which stops Docker from loading AppArmor profiles even in privileged containers. An empty value overrides the drop list.

!!! tip "Check which card is which"
    On a machine with both an iGPU and a dGPU, look at `ls -l /dev/dri/by-path/` on the host before writing the remap lines. The NVIDIA card isn't always `card0`.

## Boot ordering

The driver creates `/dev/nvidia-uvm-tools` and `/dev/nvidia-caps/*` **lazily**, on first CUDA use. On a cold boot, Proxmox's onboot `startall` got ahead of them and all three GPU containers failed with:

```
Device /dev/nvidia-uvm-tools does not exist
```

The fix is a oneshot service that creates every node up front, plus a drop-in that orders `pve-guests` after it:

```ini
# /etc/systemd/system/nvidia-device-nodes.service
[Unit]
Description=Force creation of NVIDIA uvm/caps device nodes before PVE guests start
After=nvidia-persistenced.service
Wants=nvidia-persistenced.service

[Service]
Type=oneshot
RemainAfterExit=yes
ExecStart=/usr/bin/nvidia-modprobe -c 0
ExecStart=/usr/bin/nvidia-modprobe -m
ExecStart=/usr/bin/nvidia-modprobe -c 0 -u
ExecStart=-/usr/bin/nvidia-modprobe -f /proc/driver/nvidia/capabilities/mig/config -f /proc/driver/nvidia/capabilities/mig/monitor
ExecStart=-/usr/bin/nvidia-smi

[Install]
WantedBy=multi-user.target
```

```ini
# /etc/systemd/system/pve-guests.service.d/nvidia-wait.conf
[Unit]
Wants=nvidia-device-nodes.service
After=nvidia-device-nodes.service
```

It was validated on a real cold boot: all three containers auto-start with no manual steps.

## Inside Each Container

### 1. Configure ldconfig for the bind-mounted libs

```bash
echo '/usr/lib/x86_64-linux-gnu/nvidia/current' > /etc/ld.so.conf.d/nvidia.conf
ldconfig
```

### 2. Install nvidia-container-toolkit (Docker containers only)

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

Use `runtime: nvidia` (not `deploy.resources.reservations.devices`, because CDI isn't configured):

```yaml
your-service:
  runtime: nvidia
  environment:
    - NVIDIA_VISIBLE_DEVICES=all
    - NVIDIA_DRIVER_CAPABILITIES=compute,video,utility
```

## Verification

From the `pve` host:

```bash
nvidia-smi
# Shows GPU processes from all containers
```

From inside a container:

```bash
docker run --rm --runtime=nvidia -e NVIDIA_VISIBLE_DEVICES=all nvidia/cuda:12.2.0-base-ubuntu22.04 nvidia-smi
```

## History

Before July 2026 a **GTX 980 Ti** (Maxwell) on the old `pve2` node did this job with the same pattern (hand-written `lxc.mount.entry` lines). Moving to the A3000 added AV1 encode and a much newer CUDA compute capability. The Immich ML container had been forced onto CPU because Maxwell dropped out of CUDA 12 support.
