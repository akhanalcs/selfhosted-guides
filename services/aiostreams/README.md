# AIOStreams

If you're self hosting AIOStreams, it's recommended to also host StremThru because that way you can have fast StremThru Torz links on your own server.
For setting up StremThru, see [StremThru.md](StremThru.md).

1. Add aiostreams to the list in main `.env` file
   ```env
   COMPOSE_PROFILES="required,...,aiostreams"
   ```
2. Open `aiostreams/.env` file and set the following. Detailed descriptions of each variable can be found in the `.env` file itself.
    - `ADDON_NAME`: `My Name's AIOStreams`
    - `ADDON_ID`: `aiostreams.ashkhanal.com`
    - `BASE_URL`: `https://aiostreams.mydomain.com`
    - `SECRET_KEY`: `<64-character hex string>`
    - `ADDON_PASSWORD`: `<my password for aiostreams>`
    - `AIOSTREAMS_AUTH`: `user1:pass1,user2:pass2` (These are the users that can access Proxied streams)
    - `BUILTIN_STREMTHRU_URL`: `http://stremthru:8080`
    - `STREAM_URL_MAPPINGS`: `'{"http://stremthru:8080":"https://stremthru.mydomain.com"}'`
    - `STREMTHRU_STORE_URL`: `http://stremthru:8080/stremio/store`
    - `STREMTHRU_TORZ_URL`: `http://stremthru:8080/stremio/torz`
3. Start AIOStreams
   ```bash
   $ docker compose up -d
   ```
4. Go to https://aiostreams.mydomain.com/
5. While configuring addons
    - For Comet, put `https://comet.stremio.ru` as the URL since it has a big library with very generous rate limits.
    - For MediaFusion, put `https://mediafusionfortheweebs.midnightignite.me` as the URL.
    - For StremThru Torz, leave URL as empty so it uses the self hosted StremThru instance that we configured in the `.env` file earlier.
6. While setting up proxy in AIOStreams
    - Go to Proxy page
    - Enable it
    - From the "Proxy Service" dropdown, select `Builtin Proxy`
    - Enter credentials that you set in `AIOSTREAMS_AUTH` earlier. For eg: `user1:pass1`
    - Proxied Services: `Real-Debrid`
7. Go to "Save & Install" page
    - Enter password to create this config.
    - It'll ask for the password you set in `ADDON_PASSWORD` earlier. Enter it and click "Save".
    - After it saves successfully, click on "Install" to install it to Stremio.
