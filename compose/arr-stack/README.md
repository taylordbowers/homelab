# *arr Stack Compose

Standard *arr layout with all download clients (qBittorrent, NZBGet, Prowlarr) routed through Gluetun. Indexer-only services (Sonarr, Radarr, Lidarr) sit on a separate Docker bridge so they keep working if the VPN drops.

Key shape:

```yaml
networks:
  servarrnetwork:
    name: servarrnetwork
    ipam:
      config:
        - subnet: 172.39.0.0/24

services:
  gluetun:
    image: qmcgaw/gluetun
    cap_add:
      - NET_ADMIN
    devices:
      - /dev/net/tun:/dev/net/tun
    environment:
      - VPN_SERVICE_PROVIDER=<your-provider>
      - VPN_TYPE=wireguard
      - WIREGUARD_PRIVATE_KEY=${WIREGUARD_PRIVATE_KEY}
      - FIREWALL_VPN_INPUT_PORTS=${FORWARDED_PORT}
    ports:
      - "8080:8080"   # qBittorrent WebUI
      - "6881:6881"   # qBittorrent torrent port
      - "6789:6789"   # NZBGet
      - "9696:9696"   # Prowlarr
    healthcheck:
      test: [ "CMD", "ping", "-c", "1", "1.1.1.1" ]
      interval: 30s
      retries: 3

  qbittorrent:
    image: lscr.io/linuxserver/qbittorrent:latest
    network_mode: service:gluetun
    depends_on:
      gluetun:
        condition: service_healthy

  prowlarr:
    image: lscr.io/linuxserver/prowlarr:latest
    network_mode: service:gluetun
    depends_on:
      gluetun:
        condition: service_healthy

  nzbget:
    image: lscr.io/linuxserver/nzbget:latest
    network_mode: service:gluetun
    depends_on:
      gluetun:
        condition: service_healthy

  sonarr:
    image: lscr.io/linuxserver/sonarr:latest
    networks:
      servarrnetwork:
        ipv4_address: 172.39.0.3
    ports:
      - "8989:8989"

  radarr:
    image: lscr.io/linuxserver/radarr:latest
    networks:
      servarrnetwork:
        ipv4_address: 172.39.0.4
    ports:
      - "7878:7878"

  bazarr:
    image: lscr.io/linuxserver/bazarr:latest
    ports:
      - "6767:6767"

  deunhealth:
    image: qmcgaw/deunhealth
    network_mode: none
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
```

The `deunhealth` container watches qBittorrent's healthcheck and auto-restarts it if Gluetun cycles and qB stalls.

See `.env.example` for the variables that need filling in.
