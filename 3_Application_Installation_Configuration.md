## Overview
At this point, the server hardware is assembled and the operating system is configured. Now, the applications have to be downloaded to turn this remote terminal into a fully functional homelab. 
The installation of applications on this device centers around Docker, specifically the use of Docker Compose, to create containers that will run each application independently. Below is the list of all applications currently running on the ROCK 5C server along with instructions on how to install them. 
### UI & System Management:
- [[#Heimdall]] (Application Dashboard)
- [[#Portainer]] (Container Management)
- [[#Watchtower]] (Container Update Automation)
### Connectivity:
- [[#Cloudflared]] (Cloudflare Tunnels | Reverse Proxy)
- [[#Gluetun]] (VPN Client)
### Files & Media:
- [[#Nextcloud]] (Cloud Storage)
- [[#Plex Media Server]] (Personal Media Streaming)
### P2P Downloading Setup:
- [[#Flaresolverr]] (Bypass Cloudflare)
- [[#Prowlarr]] (Indexer Management)
- [[#qBittorrent]] (P2P Downloader Client)
- [[#Radarr]] (Movie Collection Manager)
- [[#Sonarr]] (TV-Series Collection Manager)

## Pre-Requisites:
### User Group Creation:
Before any applications are installed, make sure to go to the 'Users' section in Open Media Vault and add the following user:

|  Name   |       Shell       | Groups |
| :-----: | :---------------: | :----: |
| appuser | /usr/sbin/nologin | users  |
After creating the user, apply changes and then SSH into the ROCK 5C. Then, use the following command and keep note of the users 'UID' and 'GID':
```bash
id appuser
```
### Folder Creation:
Another thing to do before installing any applications is to create a storage location on the ZFS pool for any data relating to applications. I just created a data folder but you can name it as you see fit. 

| Name | File System | Relative Path | Permissions  |      Tags       |
| :--: | :---------: | :-----------: | :----------: | :-------------: |
| data | (PoolName)  |     /data     | Don't Change | App Data Folder |
Also, a good practice is to create a shared folder for everything. For the applications listed, if a shared folder is required then it will be listed in the instructions. 
### Installing Applications Using Docker Containers:
By using Docker Containers, the process of running multi-container Docker applications has never been easier. Compose relies on using YAML files to configure an application's services for each container. For the most part, each application will be within its own docker container. 
To get to the location to add new files, follow the pathway in Open Media Vault: 'Services > Compose > Files'. After getting there, click the plus button labeled 'Add' that's in the top left of the window. Here, you can input the container name, description, and YAML configuration file for the application. 
The last thing that has to be done prior to installing any applications is to create shared folders for Docker with the relative paths:
```bash
/data/docker
/data/docker/appdata
/data/docker/compose
```
After creating those shared folders in Open Media Vault, head over to 'Services > Compose > Settings' and change the shared folder location for compose files and data to the ones that were just created. 
With that, applications are ready to be installed.
### Useful Information:

|      Topic      |                                                                                                                         Description                                                                                                                         |                                              Help                                               |
| :-------------: | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------: | :---------------------------------------------------------------------------------------------: |
| User & Group ID |                                                                                              Avoids permission issues when application has access to volumes.                                                                                               |                         use command <br>'id user_name' to get UID & GID                         |
|    Timezone     |                                                                                                           Specify a timezone for the application.                                                                                                           | View timezone options [here](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones#List) |
|     Volumes     |                                                               Give the container the absolute path of the storage location and reference it to a shorter path for the application to use.<br>                                                               |                                \[Absolute Path]:\[Relative Path]                                |
|      Ports      |                              Set ports to be used inside and outside of the container. Internal ports are only used within the container. External ports are set to be accessible using the hosts IP outside of the container.                              |                                     \[External]:\[Internal]                                     |
|  Network Modes  | \[Bridge] - Connects containers to a virtual bridge, allowing them to communicate.<br>\[Host] - Containers directly share the hosts network stack including IP address and ports. <br>\[None] - Completely isolates a container from access to the network. |           Docker Wiki - Networking ([Link](https://docs.docker.com/engine/network/))            |

## Installation & Configuration Guides:
### Heimdall:
#### Overview:
[Heimdall](https://github.com/linuxserver/Heimdall) is a locally hosted application dashboard that is most commonly used as a central point of access to the web interfaces of other applications being hosted on the same server. 
By default, Heimdall is operated using ports 80 & 443. Due to Open Media Vault using the same ones, the external ports for HTTP & HTTPS were set to 8888 & 8444 respectively. 
#### Shared Folders:
```bash
/data/heimdall
/data/heimdall/config
```
#### Compose File:
```yaml
---
services:
  heimdall:
    image: lscr.io/linuxserver/heimdall:latest
    container_name: heimdall
    environment:
      - PUID=UserID
      - PGID=GroupID
      - TZ=Country/City
      - ALLOW_INTERNAL_REQUESTS=true #optional
    volumes:
      - /path/to/heimdall/config:/config
    ports:
      - 8888:80
      - 8444:443
    restart: always
```

### Portainer:
#### Overview:
[Portainer](https://github.com/portainer/portainer) is an application that lets us manage our containers, images, volumes, networks, etc. all from one platform. This service will be useful to us as we can allocate resource limits to containers if needed. 
#### Compose File:
```yaml
---
services:
  portainer-ce:
    image: portainer/portainer-ce:latest
    container_name: portainer
    restart: always
    ports:
      # - "8000:8000"
      - "9000:9000" 
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock"
      - "CHANGE_TO_COMPOSE_DATA_PATH/portainer/portainer_data:/data"
    command: >
      -H unix:///var/run/docker.sock
      --http-enabled
```

### Watchtower:
#### Overview:
[Watchtower](https://github.com/containrrr/watchtower) can be ran in its own container for the purpose of updating and restarting other containers. This service is a must as it automates the tedious task of checking containers for updates and having to manually restart them. 
#### Configuration:
##### Scheduling:
The schedule for checking if updates are available can be set using the following format. Currently, it is setup so that it scans for updates everyday at 4 AM. 

| Second | Minute | Hour | Day | Month | Weekday |
| :----: | :----: | :--: | :-: | :---: | :-----: |
|   0    |   0    |  4   | \*  |  \*   |   \*    |
#### Compose File:
```yaml
---
services:
  watchtower:
    image: containrrr/watchtower
    container_name: watchtower
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    environment:
      - WATCHTOWER_SCHEDULE=0 0 4 * * *     # check nightly (4 AM)
      - WATCHTOWER_DISABLE_CONTAINERS=nextcloud-aio-mastercontainer
      - WATCHTOWER_CLEANUP=true     # Remove old images after updating
      - WATCHTOWER_INCLUDE_RESTARTING=true # Restart Containers Post Update
      - WATCHTOWER_INCLUDE_STOPPED=true    # Allows management of created and exited containers
      - WATCHTOWER_REVIVE_STOPPED=true     # Start any stopped containers that have had their image updated (needs WATCHTOWER_INCLUDE_STOPPED=true)
    ports:
      - 7272:8080
    restart: always
```
### Cloudflared:
#### Overview:
[Cloudflared](https://github.com/jonas-merkle/container-cloudflare-tunnel) acts as the string of this servers applications, connecting everything together. By setting up a domain to point to your public IP, you can then run the local IP and ports of your applications through tunnels to make them both more secure and remotely accessible. 
Using a reverse proxy to increase security and accessibility is pretty mainstream, but those aren't the only perks gained from doing this. By using Cloudflare Tunnels, the problem of being unable to utilize port forwarding due to CGNAT is directly bypassed. We can receive traffic from external sources through the tunnels!
#### Configuration:
In order to utilize the tunnels, you need a domain name. There are plenty of suppliers but for simplicities sake, choose Cloudflare. As far as guides go, there is a great video on YouTube by [NetworkChuck](https://www.youtube.com/watch?v=ey4u7OUAF3c&t=218s) that covers every step required to configure your tunnels. 
As can be seen in the compose file, the value of 'TUNNEL_TOKEN' is '${TUNNEL_TOKEN}'. That just means that it's referencing a variable from the global environment. To configure your token, login to Open Media Vault and go to 'Services > Compose > Files' and select the 'paper with the star' icon. It should be in the left-middle of the window right next to the scissors icon. Once selected, enter the following line:
```yaml
TUNNEL_TOKEN=YourTunnelToken
```
#### Compose File:
```yaml
---
services: 
  cloudflared: 
    image: cloudflare/cloudflared 
    container_name: cloudflare-tunnel
    network_mode: host
    restart: always
    environment:
      - TUNNEL_TOKEN=${TUNNEL_TOKEN}
    command: tunnel --no-autoupdate run --token ${TUNNEL_TOKEN}
    healthcheck:
      test: ["CMD", "cloudflared", "--version"]
      interval: 60s
      timeout: 10s
      retries: 3
      start_period: 10s
```
### Gluetun:
#### Overview:
[Gluetun](https://github.com/qdm12/gluetun) is a lightweight VPN client that can be used to connect to multiple VPN service providers. I personally use [ProtonVPN](https://protonvpn.com/?srsltid=AfmBOoqpGGOJ91RAD8M9didvkwL-sjlc_QwECYhwM5zxT3WiE0SrUDvy) and with them I can choose to create a WireGuard configuration. This gives me the private key and addresses needed to successfully setup my VPN. For this application there is no compose file, as it is ran in the same container as [[#qBittorrent]].
### Nextcloud:
#### Overview:
[Nextcloud All-in-One](https://github.com/nextcloud/all-in-one) is the application ran on this server to self-host a cloud storage platform that can be accessed from any location. There are other services included in this installation including:
- ClamAV (Antivirus)
- Collabora (Nextcloud Office)
- Fulltextsearch
- Imaginary (File Previews)
- Nextcloud Talk (VOIP Service)
- OnlyOffice
- Docker Socket Proxy
- Whiteboard
Due to the lack of RAM on the ROCK 5C (8gb), everything besides 'Imaginary' is deselected. Nextcloud is only being used in this setup for its cloud storage functionality. 
#### Configuration:
As a product of using Cloudflare Tunnels as a sort of reverse proxy, the two ports (80, 8443) were commented out of their section. Also, because of the reverse proxy Apache has to be used. I didn't change anything for Apache and used the default configuration. Take note of the Apache IP and port because that's what will be used to setup the tunnel using Cloudflare. 
If not using the ROCK 5C, navigate to the /dev/ folder and see if 'dri' is available. If it is, keep 'NEXTCLOUD_ENABLE_DRI_DEVICE' uncommented. Otherwise, comment it out. If using a reverse proxy, keep 'SKIP_DOMAIN_VALIDATION' uncommented. 
#### Shared Folders:
```bash
/data/nextcloud
/data/nextcloud/datadir
```
#### Compose File:
```yaml
services:
  nextcloud-aio-mastercontainer:
    image: ghcr.io/nextcloud-releases/all-in-one:latest 
    init: true 
    restart: always 
    container_name: nextcloud-aio-mastercontainer # This line is not allowed to be changed as otherwise AIO will not work correctly
    volumes:
      - nextcloud_aio_mastercontainer:/mnt/docker-aio-config # This line is not allowed to be changed as otherwise the built-in backup solution will not work
      - /var/run/docker.sock:/var/run/docker.sock:ro 
    network_mode: bridge 
    # networks: ["nextcloud-aio"]
    ports:
      # 80:80 
      - 8080:8080 
      # 8443:8443 
    # security_opt: ["label:disable"] 
    environment: # Is needed when using any of the options below
      # AIO_DISABLE_BACKUP_SECTION: false 
      APACHE_PORT: 11000 
      APACHE_IP_BINDING: 127.0.0.1 
      # APACHE_ADDITIONAL_NETWORK: frontend_net # (Optional) 
      # BORG_RETENTION_POLICY: --keep-within=7d --keep-weekly=4 --keep-monthly=6 
      # COLLABORA_SECCOMP_DISABLED: false 
      # FULLTEXTSEARCH_JAVA_OPTIONS: "-Xms1024M -Xmx1024M" 
      NEXTCLOUD_DATADIR: /homelab/data/nextcloud/datadir # Allows to set the host directory for Nextcloud's datadir. ⚠️⚠️⚠️ Warning: do not set or adjust this value after the initial Nextcloud installation is done!
      # NEXTCLOUD_MOUNT: /mnt/
      # NEXTCLOUD_UPLOAD_LIMIT: 16G # Can be adjusted if you need more.
      # NEXTCLOUD_MAX_TIME: 3600 # Can be adjusted if you need more. 
      # NEXTCLOUD_MEMORY_LIMIT: 512M # Can be adjusted if you need more.
      # NEXTCLOUD_TRUSTED_CACERTS_DIR: /path/to/my/cacerts 
      # NEXTCLOUD_STARTUP_APPS: deck twofactor_totp tasks calendar contacts notes
      # NEXTCLOUD_ADDITIONAL_APKS: imagemagick 
      # NEXTCLOUD_ADDITIONAL_PHP_EXTENSIONS: imagick 
      NEXTCLOUD_ENABLE_DRI_DEVICE: true # This allows to enable the /dev/dri device for containers that profit from it. ⚠️⚠️⚠️ Warning: this only works if the '/dev/dri' device is present on the host!
      # NEXTCLOUD_ENABLE_NVIDIA_GPU: true 
      # NEXTCLOUD_KEEP_DISABLED_APPS: false 
      SKIP_DOMAIN_VALIDATION: True 
      # TALK_PORT: 3478 
      # WATCHTOWER_DOCKER_SOCKET_PATH: /var/run/docker.sock 

volumes:
  nextcloud_aio_mastercontainer:
    name: nextcloud_aio_mastercontainer # This line is not allowed to be changed as otherwise the built-in backup solution will not work
  # caddy_certs:
  # caddy_config:
  # caddy_data:
  # caddy_sites:
```
### Plex Media Server:
#### Overview:
[Plex Media Server (PMS)](https://github.com/plexinc/pms-docker) is an application that organizes video, music, and photos from personal media libraries and allows them to be streamed to other devices logged in under the same user. Most commonly, Plex is used to create a locally hosted streaming platform for downloaded movies and tv-shows that can be accessed anywhere. 
#### Configuration:
In order to setup PMS, you're going to need the UID and GID for 'appuser' and the correct timezone. In addition to that, two folders have to be created to store the movies and tv-shows we want to be able to stream. After creating those folders, change the movies and tv pathways to accurately reflect the shared folder locations. 
The only port required for this application is port 32400. By running the host network mode for this application, no ports have to be defined in the compose file. 
#### Shared Folders:
```bash
/data/plex/tvseries
/data/plex/movies
```
#### Compose File:
```yaml
---
services:
  plex:
    image: lscr.io/linuxserver/plex:latest
    container_name: plex
    network_mode: host
    environment:
      - PUID=UserID  # Change this to your user PUID
      - PGID=GroupID   # Change this to your user PGID
      - TZ=Country/City # Change timezone if different
      - VERSION=docker
      - PLEX_CLAIM= #optional
    volumes:
      - CHANGE_TO_COMPOSE_DATA_PATH/plex/config:/config # Change this path
      - /path/to/plex/tvseries:/tv # Change this path
      - /path/to/plex/movies:/movies # Change this path
    restart: unless-stopped
```
### qBittorrent:
#### Overview:
[qBittorrent](https://github.com/linuxserver/docker-qbittorrent) is a BitTorrent client that uses the P2P protocol to download and share files. It is frequently referred to as the open-source alternative to µTorrent. This type of data sharing protocol breaks large files into small pieces and relies on users to 'seed' the files after downloading. This makes it so that instead of downloading a file from a single server, users essentially download pieces of a file directly to and from each other. 
#### Configuration:
The compose file for this application is different than the others, as there are two services running in this one container. In this container, we are running Gluetun and qBittorrent together. 

|  VPN Configuration Options & Other Info  |                                                                                                                                      <                                                                                                                                      |
| :--------------------------------------: | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------: |
| Supported VPN Providers<br>(OpenVPN)<br> | AirVPN, Cyberghost, ExpressVPN, FastestVPN, Giganews, HideMyAss, IPVanish, IVPN, Mullvad, NordVPN, Perfect Privacy, Privado, Private Internet Access, PrivateVPN, ProtonVPN, PureVPN, SlickVPN, Surfshark, TorGuard, VPNSecure.me, VPNUnlimited, Vyprvpn, WeVPN, Windscribe |
|  Supported VPN Providers<br>(WireGuard)  |                                                                                        AirVPN, FastestVPN, Ivpn, Mullvad, NordVPN, Perfect Privacy, ProtonVPN, Surfshark, Windscribe                                                                                        |
|                  Ports                   |                                                                                          Make sure the 'qB WebUI' port for Gluetun is the same as the 'WEBUI_PORT' for qBittorrent                                                                                          |
|          WIREGUARD_PRIVATE_KEY           |                                                                                                   Obtained from setting up a WireGuard Configuration through VPN provider                                                                                                   |
|           WIREGUARD_ADDRESSES            |                                                                                                   Obtained from setting up a WireGuard Configuration through VPN provider                                                                                                   |
|              SERVER_CITIES               |                                                                                           Can change to SERVER_COUNTRIES or SERVER_HOSTNAMES<br>Full list depends on VPN provider                                                                                           |
|                 LocalIP                  |                                                                    LocalIP is a modified version of the ROCK 5C device IP. <br>Ex) If my IP was 192.168.10.XXX then the LocalIP would be 192.168.10.0/24                                                                    |

#### Shared Folders:
```bash
/data/qbittorrent
/data/qbittorrent/appdata
/data/qbittorrent/downloads

/data/gluetun
```
#### Compose File:
```yaml
networks:
  NetworkName:
    external: true
    name: NetworkName
    
services:
  gluetun:
    image: qmcgaw/gluetun
    container_name: gluetun
    cap_add:
      - NET_ADMIN
    environment:
      # --- choose ONE stack (WireGuard OR OpenVPN) and fill in your provider details ---

      # Example (WireGuard) – fast & preferred:
      - VPN_SERVICE_PROVIDER=protonvpn        # mullvad | protonvpn | pia | nordvpn | ...
      - VPN_TYPE=wireguard
      - WIREGUARD_PRIVATE_KEY=PrivateKey
      - WIREGUARD_ADDRESSES=WgAddress    # provider gives you this or it's in the wg config
      - SERVER_CITIES=CityName        # or SERVER_CITIES=..., SERVER_HOSTNAMES=...

      # Example (OpenVPN) – if you use OVPN creds instead:
      # - VPN_SERVICE_PROVIDER=protonvpn
      # - VPN_TYPE=openvpn
      # - OPENVPN_USER=yourusername
      # - OPENVPN_PASSWORD=yourpassword
      # - SERVER_COUNTRIES=United States

      # General:
      - TZ=Country/City
      - FIREWALL_OUTBOUND_SUBNETS=LocalIP  # so you can reach WebUI from your LAN
      - FIREWALL_INPUT_PORTS=8585
    volumes:
      - /path/to/gluetun:/gluetun
    # Expose qBittorrent's ports HERE (since qB routes through gluetun)
    networks: [NetworkName]
    ports:
      - "8585:8585"       # qBittorrent WebUI
      - "6881:6881"       # TCP
      - "6881:6881/udp"   # UDP
    restart: unless-stopped

  qbittorrent:
    image: lscr.io/linuxserver/qbittorrent:latest
    container_name: qbittorrent
    network_mode: "service:gluetun"   # route all qB traffic through gluetun
    depends_on:
      - gluetun
    environment:
      - PUID=UserID
      - PGID=GroupID
      - TZ=Country/City
      - WEBUI_PORT=8585              # make qB listen on 8585 to match the published port
      # no TORRENTING_PORT env — set the listening port to 6881 inside qB UI (or via config)
    volumes:
      - /path/to/qbittorrent/appdata:/config
      - /path/to/qbittorrent/downloads:/downloads
    restart: unless-stopped
```
### Flaresolverr:
#### Overview:
[Flaresolverr](https://github.com/FlareSolverr/FlareSolverr) is a proxy server that is used to bypass Cloudflare and DDoS-GUARD protection on websites. We will be running this so that we can access indexers like x1337x with other applications like Prowlarr, Radarr, and Sonarr. 
#### Configuration:
Besides creating the two shared folders, there's only one other thing that has to be done to finish setting up this application. In order to allow Flaresolverr to interact with Prowlarr, a network bridge has to be created. 
In Open Media Vault, go to 'Services > Compose > Networks' and add a new network. Only input the name and make sure that the driver is set to 'bridge'. 
#### Shared Folders:
```bash
/data/flaresolverr
/data/flaresolverr/config
```
#### Compose File:
```yaml
---
networks:
  NetworkName:
    external: true
    name: NetworkName
    
services:
  flaresolverr:
    # DockerHub mirror flaresolverr/flaresolverr:latest
    image: ghcr.io/flaresolverr/flaresolverr:latest
    container_name: flaresolverr
    environment:
      - LOG_LEVEL=${LOG_LEVEL:-info}
      - LOG_FILE=${LOG_FILE:-none}
      - LOG_HTML=${LOG_HTML:-false}
      - CAPTCHA_SOLVER=${CAPTCHA_SOLVER:-none}
      - TZ=Country/City
      - TEST_URL=https://www.google.com
    networks: [NetworkName]
    ports:
      - "${PORT:-8191}:8191"
    volumes:
      - /path/to/flaresolverr/config:/config
    restart: unless-stopped
```
### Prowlarr: 
#### Overview:
[Prowlarr](https://github.com/linuxserver/docker-prowlarr) is a manager for indexers that integrates seamlessly with Sonarr, Radarr, and other -arr applications. By using this service, we can effectively have complete management over all indexers for other applications, bypassing that 'per app' setup. 
#### Configuration:
To setup this application, you will need the UID and GID for 'appuser'. After getting Prowlarr up and running, login to the service. After logging in, make sure to do the following:
- Go to 'Settings > Indexers' and add Flaresolverr. (Host: http://flaresolverr:8191)
- Add the tags 'movies', 'tv', and 'other' to the indexer proxy. 
- Then go to 'Indexers' and hit 'Add Indexer'. Add whatever indexers you want. At the time of making this I use the following:
	- 1337x
	- Knaben
	- The Pirate Bay
	- Uindex
	- YTS
- Then, go to 'Settings > Download Clients' and add qBittorent. For 'Host' use the IP of the ROCK 5C and set the port to be the same as what is used by qBittorent. Type in the username and password used to login to qB and then add two mapped categories, one for movies and one for tv-shows.
- Before continuing the setup for Prowlarr, make sure Radarr and Sonarr are installed. 
#### Shared Folders:
```bash
/data/prowlarr
/data/prowlarr/config
```
#### Compose File:
```yaml
---
networks:
  NetworkName:
    external: true
    name: NetworkName
    
services:
  prowlarr:
    image: lscr.io/linuxserver/prowlarr:latest
    container_name: prowlarr
    environment:
      - PUID=UserID
      - PGID=GroupID
      - TZ=Country/City
    volumes:
      - /path/to/prowlarr/config:/config
    networks: [NetworkName]
    ports: ["9696:9696"]
    restart: unless-stopped
```
### Radarr:
#### Overview:
[Radarr](https://github.com/Radarr/Radarr) is a movie collection manager for Usenet and BitTorrent users. It can monitor the web for new movies and can interface directly with clients and indexers to obtain and manage them on a file level. A useful feature of this application is that it can 'upgrade' movies to a higher quality as long as you set its quality profile. This is the application used with this server to manage the Plex movie collection. 
#### Configuration:
To setup Radarr, a few things have to be done. First, you have to grab the UID and GID from 'appuser' for the compose file. Then, create the required shared folders for the application. After that, fill in your network name in order to ensure that is has access to the qBittorrent client. 
#### Shared Folders:
```bash
/data/radarr
/data/radarr/config
```
#### Compose File:
```yaml
---
networks:
  NetworkName:
    external: true
    name: NetworkName
    
services:
  radarr:
    image: lscr.io/linuxserver/radarr:latest
    container_name: radarr
    environment:
      - PUID=UserID
      - PGID=GroupID
      - TZ=Country/City
    volumes:
      - /path/to/radarr/config:/config
      - /path/to/plex/movies:/movies #optional
      - /path/to/qbittorrent/downloads:/downloads #optional
    networks: [NetworkName]
    ports:
      - "7878:7878"
    restart: unless-stopped
```
### Sonarr:
#### Overview:
[Sonarr](https://github.com/Sonarr/Sonarr), like Radarr, is another manager that is used in this setup. It also monitors multiple RSS feeds to find new episodes of managed series. In terms of functionality, it's the same as Radarr except for TV shows instead of movies.  
#### Configuration:
See configuration guide for [[#Radarr]].
#### Shared Folders:
```bash
/data/sonarr
/data/sonarr/config
```
#### Compose File:
```yaml
---
networks:
  NetworkName:
    external: true
    name: NetworkName
    
services:
  sonarr:
    image: lscr.io/linuxserver/sonarr:latest
    container_name: sonarr
    environment:
      - PUID=UserID
      - PGID=GroupID
      - TZ=Country/City
    volumes:
      - /path/to/sonarr/config:/config
      - /path/to/plex/tvseries:/tv #optional
      - /path/to/qbittorrent/downloads:/downloads #optional
    networks: [NetworkName]
    ports:
      - "8989:8989"
    restart: unless-stopped
```


## Utilized Ports (External):

|    Port Layout     |    <     |
| :----------------: | :------: |
|    **Service**     | **Port** |
|    Flaresolverr    |   8191   |
| Gluetun - Torrent  |   6881   |
|    Gluetun - qB    |   8585   |
|  Heimdall - https  |   8444   |
|  Heimdall - http   |   8888   |
|  Nextcloud - AIO   |   8080   |
| Nextcloud - Apache |  11000   |
|     Portainer      |   9000   |
|      Prowlarr      |   9696   |
|       Radarr       |   7878   |
|       Sonarr       |   8989   |
|    Uptime Kuma     |   3001   |
|     Watchtower     |   7272   |

