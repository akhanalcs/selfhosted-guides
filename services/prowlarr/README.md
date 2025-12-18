# Prowlarr
This guide assumes that you're using Viren's Docker Compose template at https://github.com/Viren070/docker-compose-template

But you can adapt it to any other Docker setup.

## Steps
1. Add prowlarr to the list in main `.env` file
   ```env
   COMPOSE_PROFILES="required,...,prowlarr"
   ```
2. Make sure you have a directory to store custom indexers for Prowlarr.
   For this guide, the custom indexers are in `selfhosted-guides/services/prowlarr/definitions/`.
   ```bash
   $ sudo mkdir -p docker-compose-template/apps/prowlarr/definitions
   ```
3. Give ownership of Prowlarr data folder to your user
   ```bash
   $ sudo mkdir -p /opt/docker/data/prowlarr
   $ sudo chown -R $(id -u):$(id -g) -R /opt/docker/data/prowlarr
   ```
4. Configure custom definitions in Prowlarr
    - `prowlarr/definitions/aiostreams.yml`
        - This only applies if you're self-hosting AIOStreams.
        - Open `prowlarr/definitons/aiostreams.yml` file and make sure that the link points to your instance of AIOStreams
          ```yaml
          links:
             - http://aiostreams:3000
          ```
    - `prowlarr/definitions/comet.yml`
        - Change the link to Kuu's instance as it's the most comprehensive instance for Comet
          ```yaml
          links:
            - https://comet.stremio.ru
            - http://comet:8000
          ```
    - `prowlarr/definitions/stremthru.yml`
        - Make sure it only has your instance if you are self-hosting StremThru. I am, so it looks like this for me:
          ```yaml
          links:
            - http://stremthru:8080
          ```
    - Configure other indexers however you like.
    - For more custom indexers, search on GitHub, like https://github.com/dreulavelle/Prowlarr-Indexers/tree/main/Custom
5. Start Prowlarr
   ```bash
   $ docker compose up -d prowlarr
   ```
6. Go to https://prowlarr.yourdomain.com
    - Set username and password
7. On the main page, you'll be asked to add Indexers
    - Click `Add New Indexer` button
    - Search for `aiostreams`, `stremthru`, `torbox` etc. and add them.
8. Connect Prowlarr to Radarr
    - Go to Settings > Apps > Applications > +Add Application
        - Select `Radarr`
        - Name: `Radarr`
        - Prowlarr Server: `http://prowlarr:9696`
        - Radarr Server: `http://radarr:7878`
        - API Key: Get it from Radarr > Settings > General > Security > API Key
        - Test the connection and click `Save`
9. Connect Prowlarr to Sonarr like we did for Radarr above.

