# Gluetun
Setting up Gluetun for StremThru Proxy to resolve Torrentio links.

## Run Wireguard Server at home
Since WARP is no longer a viable solution for accessing Torrentio, I decided to setup a WireGuard server at home & connecting to it using gluetun.

This is necessary because Torrentio blocks StremThru running on the Oracle Cloud VM from accessing the original RD link.

1. Get GL.iNet Brume 2 or setup PiVPN
2. On your main home router, forward UDP port 51820 to the GL.iNet router's IP address and give Brume 2 a static IP.
3. Follow instructions [here](https://docs.gl-inet.com/router/en/3/tutorials/wireguard_server/#initialize-wireguard-server) to setup WireGuard server on the router
4. Create a WireGuard client conf for the Oracle Cloud VM
    - Go to **VPN > WireGuard Server > Profiles**
    - Click on **+Add**.
    - Name the profile (e.g., `vps-client`).
    - Click the **Configuration File** tab.
    - Make sure Endpoint shows your home public IP and port 51820.
    - Click on **Download** to get the client configuration file: `vps-client.conf`.

## Setup Gluetun
1. Open `gluetun/.env` and set the "VPN Type"
   ```env
   # VPN Type
   VPN_SERVICE_PROVIDER=custom
   VPN_TYPE=wireguard
   ```
   Those are the only 2 variables that need to be there.
2. Copy the `vps-client.conf` file downloaded earlier to `apps/gluetun/` directory.
3. Change the subnet mask for the `Address` field in the `[Interface]` section of the `vps-client.conf` file from `Address = X.X.X.X/24` to `Address = X.X.X.X/32`
4. Set the WireGuard conf file path in `apps/gluetun/compose.yml`
   ```yaml
   volumes:
     - ./vps-client.conf:/gluetun/wireguard/wg0.conf
   ```
5. Verify that gluetun is connected to home by checking the IP address
    - Start
      ```bash
      $ docker compose up -d gluetun gost
      ```
    - Check public IP inside gluetun
      ```bash
      $ docker exec -it gluetun sh -c "wget -qO- http://ipinfo.io/ip; echo"
      <home-public-ip>
      ```

## Pitfall
1. Currently, gluetun doesn't support hostname on WireGuard endpoint, so if my home IP changes, the WireGuard connection will break.
2. A feature request has been opened on their GitHub repo to support hostnames for WireGuard endpoints.
   - https://github.com/qdm12/gluetun/issues/788
3. Developer has committed a fix for it but not yet released.
4. Follow the issue to know when it's released.
