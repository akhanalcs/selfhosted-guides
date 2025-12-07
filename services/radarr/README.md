# Radarr

## Setup Instructions
1. Add radarr to the list in main `.env` file
   ```env
   COMPOSE_PROFILES="required,...,radarr"
   ```
2. Copy the config files to the VM
3. Start Radarr
   ```bash
   $ docker compose up -d
   ```
4. Go to https://radarr.mydomain.com
    - Set username and password
5. Go to Settings > Media Management > Show Advanced
   - Movie Naming
     - `Rename Movies`: Checked
   - File Management
     - `Unmonitor Deleted Movies`: Checked
     - `Proper & Repacks`: `Do Not Prefer` (We'll be using Repack/Proper Custom Format for this)
   - Root Folders
     - Click `Add Root Folder`
     - Select `/mnt/media/movies`
     - Click `OK`
   - Click `Save Changes`
6. Go to Settings > Profiles
   - Remove them as we'll be adding new ones from TRaSH guides using Recyclaar later
7. Add download clients
   - Go to Settings > Download Clients > Add Download Client
     - Choose `qBittorrent`
       - Name: `Decypharr qBittorrent`
       - Host: `decypharr` (this is the service name defined in docker-compose for Decypharr)
       - Port: `8282`
       - Test the connection and click `Save`
8. Tell Radarr to notify Plex after download is complete
   - Go to Settings > Connect > +Add Connection
     - Choose `Plex Media Server`
       - Name: `Plex`
       - Host: `plex` (this is the service name defined in docker-compose for Plex)
       - Port: `32400`
       - Authenticate with Plex.tv: Click `Start OAuth` button and login to your Plex account to authorize Radarr
       - Test the connection and click `Save`
9. Radarr Setup Done!
