# Restreamer

Self-hosted live-streaming relay. Accepts RTMP ingest, transcodes, and republishes as HLS for browser playback.

## Setup

- **Host:** portainer LXC (CT 121) on pve-guide, `10.0.0.11`
- **Image:** `datarhei/restreamer:latest`
- **Ports:**
    - **1935** — RTMP ingest (e.g. OBS)
    - **8080** — Web UI / HLS output

## Docker Compose

```yaml
restreamer:
  image: datarhei/restreamer:latest
  container_name: restreamer
  restart: unless-stopped
  ports:
    - "1935:1935"   # RTMP in
    - "8080:8080"   # web UI / HLS out
  volumes:
    - restreamer-data:/core/data
    - restreamer-config:/core/config

volumes:
  restreamer-data:
  restreamer-config:
```

## Use case

Useful for restreaming an RTMP source to multiple destinations (or to embed in a dashboard) without exposing the source server directly. Also doubles as a low-latency local stream viewer for cameras or recording rigs that can push RTMP.
