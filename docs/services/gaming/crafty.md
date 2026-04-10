# Crafty Controller

Web-based Minecraft server management platform. Supports Java and Bedrock editions.

## Setup

- **Host:** portainer LXC (CT 121) on pve-guide, `192.168.1.122`
- **Web UI:** port 8443 (HTTPS) / 8123 (HTTP)
- **URL:** `crafty.taylorsfunlab.com`
- **Image:** `registry.gitlab.com/crafty-controller/crafty-4:latest`
- **Bedrock port:** 19132 (UDP)
- **Java ports:** 25500–25600 (TCP)

## Docker Compose

```yaml
crafty_container:
  image: registry.gitlab.com/crafty-controller/crafty-4:latest
  container_name: crafty_container
  restart: always
  ports:
    - 8000:8000
    - 8123:8123
    - 8443:8443
    - 19132:19132/udp
    - 25500-25600:25500-25600
  volumes:
    - /data/servers/servers:/crafty/servers
    - /data/servers/config:/crafty/app/config
    - /data/servers/import:/crafty/import
    - /data/servers/backups:/crafty/backups
    - /data/servers/logs:/crafty/logs
  environment:
    - TZ=America/Chicago
```

## Server Data

Server files, configs, and backups are stored on the `tank_new` RAIDZ1 array via the media LXC at `/data/servers/`.
