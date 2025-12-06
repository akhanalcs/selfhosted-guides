# Plex

Reference: https://github.com/plexinc/pms-docker/blob/master/README.md

1. Add plex to the list in main `.env` file
   ```env
   COMPOSE_PROFILES="required,...,plex"
   ```
2. Open `plex/.env` file and set the following:
    - `PLEX_CLAIM`: Get it from https://www.plex.tv/claim
    - `PLEX_TRAEFIK_ENABLE`: `true`
3. Check out `plex/compose.yml` and make sure it looks like this
   ```yml
   services:
     plex:
       image: plexinc/pms-docker:latest
       container_name: plex
       restart: unless-stopped
       # Network mode is bridge here (Default) - Keeps it secure behind Traefik
       ports:
         - 32400:32400/tcp
         - 8324:8324/tcp
         - 32469:32469/tcp
         - 1900:1900/udp
         - 32410:32410/udp
         - 32412:32412/udp
         - 32413:32413/udp
         - 32414:32414/udp
       expose:
         - 32400
       environment:
         - TZ=${TZ:-UTC}
         - PLEX_UID=${PUID}
         - PLEX_GID=${PGID}
         - ADVERTISE_IP=https://${PLEX_HOSTNAME}
       env_file:
         - .env
       volumes:
         - ${DOCKER_DATA_DIR}/plex/config:/config
         - ${DOCKER_DATA_DIR}/plex/transcodes:/transcode
         - /dev/shm:/dev/shm # RAM Transcoding support (Performance Boost)
         - /mnt:/mnt:rslave # Lets Plex see the Decypharr mount
       labels:
         - "traefik.enable=${PLEX_TRAEFIK_ENABLE:-true}"
         - "traefik.http.routers.plex.rule=Host(`${PLEX_HOSTNAME?}`)"
         - "traefik.http.routers.plex.entrypoints=websecure"
         - "traefik.http.routers.plex.tls.certresolver=letsencrypt"
         - "traefik.http.services.plex.loadbalancer.server.port=32400"
       healthcheck:
         test: curl --connect-timeout 15 --silent --show-error --fail http://localhost:32400/identity
         interval: 1m
         timeout: 15s
         retries: 3
         start_period: 1m
       depends_on:
         decypharr:
           condition: service_healthy
       profiles:
         - all
         - plex 
   ```
4. Plan a good folder structure for your media
   ```
   mnt
   ├── symlinks <-- (Writable) Decypharr drops raw symlinks here
   │   ├── sonarr
   │   └── radarr
   ├── remote <-- (Read-Only) The Rclone Mounts (RealDebrid/Torbox) live here
   │   ├── realdebrid
   │   │   └── torrents <--- RD content shows up here
   │   └── torbox
   │       └── torrents <--- Torbox content shows up here
   └── media
       ├── movies <-- (Writable) Radarr brings movies from symlinks/radarr to here. Plex also points here.
       ├── tv
       └── other
   ```
5. Create the above folders in the server
   ```bash
   $ sudo mkdir -p /mnt/media/movies
   $ sudo mkdir -p /mnt/media/tv
   $ sudo mkdir -p /mnt/media/other
   $ sudo mkdir -p /mnt/symlinks/radarr
   $ sudo mkdir -p /mnt/symlinks/sonarr
   ```
6. Give your user ownership
   ```bash
   $ sudo chown -R $(id -u):$(id -g) -R /mnt/media
   $ sudo chown -R $(id -u):$(id -g) -R /mnt/symlinks
   ```
7. Copy the config files to the VM
8. Start Plex
   ```bash
   $ docker compose up -d plex
   ```
9. Go to https://plex.mydomain.com and login
10. Name your server: `my-plex-server`

    Also checkmark to allow remote access.
11. Set media libraries using the folders created earlier

Plex Setup Done!