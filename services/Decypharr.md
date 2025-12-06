# Decypharr

1. Add decypharr to the list in main `.env` file
   ```env
   COMPOSE_PROFILES="required,...,decypharr"
   ```
2. Open `decypharr/config.json` file and set the following:
    - For RealDebrid integration
        - `api_key`: <rd-api-key-here>
        - `torrents_refresh_interval`: `60s` (was `15s` by default)
    - Copy the whole real debrid section again for TorBox integration
        - `name`: `torbox`
        - `api_key`: <tb-api-key-here>
        - `folder`: `/mnt/remote/torbox/__all__/`
        - `torrents_refresh_interval`: `300s`
        - Change the `torrents_refresh_interval` to `60s`
    - For `qbittorrent` settings
        - `refresh_interval`: `60` (was `5` by default)
    - For `rclone` settings
        - Set rclone.uid and rclone.gid to the value of PUID and PGID from the main .env respectively
3. Create mount folders in the server
   ```bash
   $ sudo mkdir -p /mnt/remote
   $ sudo mkdir -p /mnt/symlinks
   ```
4. Give your user ownership
    ```bash
    $ sudo chown -R $(id -u):$(id -g) -R /mnt/remote
    # Verify
    $ ls -ld /mnt/remote
    drwxr-xr-x 2 ubuntu ubuntu 4096 Nov 30 01:33 /mnt/remote
    $ sudo chown -R $(id -u):$(id -g) -R /mnt/symlinks
    ```
5. Start Decypharr
   ```bash
   $ docker compose up -d decypharr
   ```
6. Go to https://decypharr.mydomain.com/
