# StremThru

Setting up StremThru proxy to work with AIOStreams and Torrentio on Oracle Cloud VM.

1. Open `stremthru/.env`
2. Add users that you want to allow access to StremThru proxy:
   ```env
   STREMTHRU_PROXY_AUTH=alex:AlexPass1,tom:TomSecretPass
   ```
3. Add debrid services' API keys:
   ```env
   STREMTHRU_STORE_AUTH=*:realdebrid:<rd-api-key-here>,*:torbox:<tb-api-key-here>
    ```
4. Set URL of the tunnel to use gluetun container as HTTP proxy. This is only required if you use Torrentio because StremThru needs to access the original RD links through non-datacenter IPs. See section on how to setup Gluetun [here](Gluetun.md).
   ```env
   STREMTHRU_HTTP_PROXY=http://gluetun:8080
   ```
   Personally, I stopped using Torrentio because of this hack, and Comet, StremThru, MediaFusion are plenty good.
5. Configure what to use the tunnel for

   StremThru won't use the tunnel for the actual playback link; only to get access to the streaming link through Torrentio.
   ```env
   STREMTHRU_TUNNEL=*:false,torrentio.strem.fun:true
   ```
6. Configure RealDebrid api access to route through the tunnel, not the playback links.
   ```env
   STREMTHRU_STORE_TUNNEL=realdebrid:api
   ```
7. Start StremThru container
   ```bash
   $ docker compose up -d stremthru
   ```
   If you made any changes after already running it, you need to recreate the container for the changes to take effect
   ```bash
   $ docker compose restart stremthru # Don't do this as it only reboots the existing container. It does not re-read the .env file or apply configuration changes. It just turns it off and on again.
   # Instead do this:
   # up -d is usually enough if the config changed, but --force-recreate guarantees it
   $ docker compose up -d --force-recreate stremthru
   ```
8. Verify that StremThru can use the gluetun proxy
   ```bash
   $ docker exec -e http_proxy=http://gluetun:8080 -it stremthru sh -c "wget -Y on -qO- http://ipinfo.io/ip; echo"
   <home-public-ip-address>
   ```
9. Verify that StremThru can use the direct connection
   ```bash
   $ docker exec -it stremthru sh -c "wget -qO- http://ipinfo.io/ip; echo"
   <oracle-server-public-ip-address>
   ```
10. Setup AIOStreams to use StremThru proxy. (UPDATE: I moved on to using In-Built proxy in AIOStreams which I found to be more reliable. This requires me to self host AIOStreams. Setup steps on it is shown below.)
    - Go to your aiostreams configuration page and login
    - Find the "Proxy" page
    - Enable it
    - From the "Proxy Service" dropdown, select `StremThru`
    - Set URL to https://stremthru.mydomain.com
    - Enter the username and password you set in `STREMTHRU_PROXY_AUTH` earlier.
        - It was `alex:AlexPass1` in my case.
    - Go to "Save & Install" page and click "Save"
        - If the save is successful, it means the connection to StremThru proxy is working.
11. Test playback
    - Open Stremio app
    - Select a random movie or show
    - If you see a detective icon on the links, it means StremThru proxy is working.
    - Select a Torrentio link and see if it works.
    - It plays successfully! 🎉
12. Verify in RealDebrid
    - Go to https://real-debrid.com/downloads
    - You should see your home public IP address for the file you just played.
    - For non-Torrentio links, you'll see Cloudflare IP addresses but that's not an issue as [explained by Viren](https://github.com/Viren070/AIOStreams/issues/508#issuecomment-3573278605):

      > The IP you see in the RD dashboard is just the IP used to generate the download link.
      > RD only uses that IP to select nearest CDN. It won't cause any issues regarding account ban.
      > The IP they look at is whoever is actually making the requests to the generated CDN links which is always going to be either you from Stremio (or other client) or your proxy (or the http proxy's IP if its configured to be used for CDN links.)
