# AIOMetadata

1. Get TMDB API key following directions here: https://guides.viren070.me/stremio/faq#how-to-obtain-a-tmdb-api-key
2. Get TVDB API key from https://www.thetvdb.com/api-information
3. For AI-Powered Search, get Google Gemini API key from here: https://aistudio.google.com/app/u/6/api-keys
4. Get MDBList API key from https://mdblist.com/preferences/
5. Go to https://cinebye.elfhosted.com/
   - Login with your Stremio account
   - Click `Download Backup`
   - Turn ON these options:
     - Remove Cinemeta Search
     - Remove Cinemeta Catalogs
     - Remove Cinemeta Metadata
   - Click `Sync to Stremio`
6. Remove `AIOLists` addon if present
7. Open `aiometadata/.env` file and set the following
   - `TMDB_API`:
   - `TVDB_API_KEY`:
   - `CACHE_WARMUP_MODE`: `comprehensive` (was `essential` by default)
   - `CACHE_WARMUP_UUIDS`: Comma separated list of UUIDs to prefetch metadata for. I populated this after I got my configuration saved which generated the UUID. I can do this for others later. (was commented out by default)
   - `ADMIN_KEY`: 14 character password generated using Bitwarden (was commented out by default)
8. Open `aiostreams/.env` file and set the following. `-1` means to disable caching.
   - `STREAM_CACHE_TTL`: `-1` (was `-1` by default)
   - `CATALOG_CACHE_TTL`: `-1` (was `300` by default)
   - `META_CACHE_TTL`: `-1` (was `300` by default)
9. Add aiometadata to the list in main `.env` file
   ```env
   COMPOSE_PROFILES="required,...,aiometadata"
   ```
10. Start AIOMetadata
    ```bash
    $ docker compose up -d
    ```
11. Go to https://aiometadata.mydomain.com/configure/
12. Configure `Presets` settings
    - `Movies & Shows Only`: Click `Apply this preset`
13. Configure `General` settings
    - Accept the defaults
14. Configure `Integrations` settings
    - It already had prepopulated my TMDB and TVDB keys from the `.env` file
    - Enter Google Gemini API key
    - Enter MDBList API key
    - Click `Test All Keys` to verify
15. Configure `Meta Providers` settings
    - Accept the defaults
16. Configure `Art Providers` settings
    - Accept the defaults
17. Configure `Filters` settings
    - Digital Release Filter
      - `Hide Unreleased Movies`: ON
18. Configure `Search` settings
    - AI-Powered Search: ON
19. Configure `Catalogs` settings
    - Click `Manage Streaming Providers` to add streaming services catalog like Netflix, Hulu, Disney+, etc.
    - Click `Manage MDBList Integration` to get custom lists as catalogs
      - Default settings
        - Sort By: `IMDb Rating`
        - Order: `Descending`
      - Import Lists from Any User
        - Enter `garycrawfordgc`
        - Click `Load User Lists`
20. Click `Configuration` tab
    - Click `Save Configuration`
    - Enter a password to create this config
    - Save both password and UUID to Bitwarden
    - Copy the manifest link and install it in Stremio
21. Go back to Cinebye and click `Logged in` to refresh the addons
    - You should see `AIOMetadata` addon installed now.
      - If you need to reinstall it after you add or remove catalog in the `AIOMetadata` `Catalogs` page, just click that `Re-install addon` button.
    - Now do the ordering of addons
      - Drag `AIOMetadata` just below `Cinemeta`
    - Click `Sync to Stremio` to apply the changes
