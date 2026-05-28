<div align="center">
  <table style="border-collapse:collapse;width:100%;">
    <thead>
      <tr style="background-color:#f5f5f5;" align="center">
        <th style="border:1px solid #ddd;padding:8px;text-align:center;">Title:</th>
        <th style="border:1px solid #ddd;padding:8px;text-align:center;">Author:</th>
        <th style="border:1px solid #ddd;padding:8px;text-align:center;">last_updated:</th>
      </tr>
    </thead>
    <tbody>
      <tr style="background-color:#ffffff;" align="center">
        <td style="border:1px solid #ddd;padding:8px;text-align:center;">ROCK 5C Home Server Applications</td>
        <td style="border:1px solid #ddd;padding:8px;text-align:center;">Lucas Costello</td>
        <td style="border:1px solid #ddd;padding:8px;text-align:center;">2026-05-28</td>
      </tr>
    </tbody>
  </table>
</div>


<!-- Overview -->
<details open>
  <summary><h2 id="overview" style="display:inline;">Overview</h2></summary>
  <div align="justify" style="margin-top:8px;">
    <p>At this point, the server hardware is assembled and the operating system is configured. Now, the applications have to be downloaded to turn this remote terminal into a fully functional homelab. The installation of applications on this device centers around Docker, specifically the use of Docker Compose, to create containers that will run each application independently. Below is the list of all applications currently running on the ROCK 5C server along with instructions on how to install them.</p>
    <details open>
      <summary><h3 id="ui-system-management">UI &amp; System Management:</h3></summary>
      <ul>
        <li><a href="#heimdall">Heimdall</a> (Application Dashboard)</li>
        <li><a href="#portainer">Portainer</a> (Container Management)</li>
        <li><a href="#watchtower">Watchtower</a> (Container Update Automation)</li>
      </ul>
    </details>
    <details open>
      <summary><h3 id="connectivity">Connectivity:</h3></summary>
      <ul>
        <li><a href="#cloudflared">Cloudflared</a> (Cloudflare Tunnels | Reverse Proxy)</li>
        <li><a href="#gluetun">Gluetun</a> (VPN Client)</li>
      </ul>
    </details>
    <details open>
      <summary><h3 id="files-media">Files &amp; Media:</h3></summary>
      <ul>
        <li><a href="#nextcloud">Nextcloud</a> (Cloud Storage)</li>
        <li><a href="#plex-media-server">Plex Media Server</a> (Personal Media Streaming)</li>
      </ul>
    </details>
    <details open>
      <summary><h3 id="p2p-downloading-setup">P2P Downloading Setup:</h3></summary>
      <ul>
        <li><a href="#flaresolverr">Flaresolverr</a> (Bypass Cloudflare)</li>
        <li><a href="#prowlarr">Prowlarr</a> (Indexer Management)</li>
        <li><a href="#qbittorrent">qBittorrent</a> (P2P Downloader Client)</li>
        <li><a href="#radarr">Radarr</a> (Movie Collection Manager)</li>
        <li><a href="#sonarr">Sonarr</a> (TV-Series Collection Manager)</li>
      </ul>
    </details>
  </div>
</details>

<!-- Pre-Requisites -->
<details open>
  <summary><h2 id="pre-requisites" style="display:inline;">Pre-Requisites</h2></summary>
  <div align="justify" style="margin-top:8px;">
    <details open>
      <summary><h3 id="user-group-creation">User Group Creation:</h3></summary>
      <p>Before any applications are installed, make sure to go to the &#x27;Users&#x27; section in Open Media Vault and add the following user:</p>
      <div style="overflow-x:auto;">
        <table style="border-collapse:collapse;width:100%;">
          <thead>
            <tr style="background-color:#f5f5f5;" align="center">
              <th style="border:1px solid #ddd;padding:8px;text-align:center;">Name</th>
              <th style="border:1px solid #ddd;padding:8px;text-align:center;">Shell</th>
              <th style="border:1px solid #ddd;padding:8px;text-align:center;">Groups</th>
            </tr>
          </thead>
          <tbody align="center">
            <tr>
              <td style="border:1px solid #ddd;padding:8px;text-align:center;">appuser</td>
              <td style="border:1px solid #ddd;padding:8px;text-align:center;">/usr/sbin/nologin</td>
              <td style="border:1px solid #ddd;padding:8px;text-align:center;">users</td>
            </tr>
          </tbody>
        </table>
      </div>
      <p>After creating the user, apply changes and then SSH into the ROCK 5C. Then, use the following command and keep note of the users &#x27;UID&#x27; and &#x27;GID&#x27;:</p>
      <pre><code>id appuser</code></pre>
    </details>
    <details open>
      <summary><h3 id="folder-creation">Folder Creation:</h3></summary>
      <p>Another thing to do before installing any applications is to create a storage location on the ZFS pool for any data relating to applications. I just created a data folder but you can name it as you see fit.</p>
      <div style="overflow-x:auto;">
        <table style="border-collapse:collapse;width:100%;">
          <thead>
            <tr style="background-color:#f5f5f5;" align="center">
              <th style="border:1px solid #ddd;padding:8px;text-align:center;">Name</th>
              <th style="border:1px solid #ddd;padding:8px;text-align:center;">File System</th>
              <th style="border:1px solid #ddd;padding:8px;text-align:center;">Relative Path</th>
              <th style="border:1px solid #ddd;padding:8px;text-align:center;">Permissions</th>
              <th style="border:1px solid #ddd;padding:8px;text-align:center;">Tags</th>
            </tr>
          </thead>
          <tbody align="center">
            <tr>
              <td style="border:1px solid #ddd;padding:8px;text-align:center;">data</td>
              <td style="border:1px solid #ddd;padding:8px;text-align:center;">(PoolName)</td>
              <td style="border:1px solid #ddd;padding:8px;text-align:center;">/data</td>
              <td style="border:1px solid #ddd;padding:8px;text-align:center;">Don&#x27;t Change</td>
              <td style="border:1px solid #ddd;padding:8px;text-align:center;">App Data Folder</td>
            </tr>
          </tbody>
        </table>
      </div>
      <p>Also, a good practice is to create a shared folder for everything. For the applications listed, if a shared folder is required then it will be listed in the instructions.</p>
    </details>
    <details open>
      <summary><h3 id="installing-applications-using-docker-containers">Installing Applications Using Docker Containers:</h3></summary>
      <p>By using Docker Containers, the process of running multi-container Docker applications has never been easier. Compose relies on using YAML files to configure an application&#x27;s services for each container. For the most part, each application will be within its own docker container. To get to the location to add new files, follow the pathway in Open Media Vault: &#x27;Services &gt; Compose &gt; Files&#x27;. After getting there, click the plus button labeled &#x27;Add&#x27; that&#x27;s in the top left of the window. Here, you can input the container name, description, and YAML configuration file for the application. The last thing that has to be done prior to installing any applications is to create shared folders for Docker with the relative paths:</p>
      <pre><code>/data/docker
/data/docker/appdata
/data/docker/compose</code></pre>
      <p>After creating those shared folders in Open Media Vault, head over to &#x27;Services &gt; Compose &gt; Settings&#x27; and change the shared folder location for compose files and data to the ones that were just created. With that, applications are ready to be installed.</p>
    </details>
    <details open>
      <summary><h3 id="useful-information">Useful Information:</h3></summary>
      <div style="overflow-x:auto;">
        <table style="border-collapse:collapse;width:100%;">
          <thead>
            <tr style="background-color:#f5f5f5;" align="center">
              <th style="border:1px solid #ddd;padding:8px;text-align:center;">Topic</th>
              <th style="border:1px solid #ddd;padding:8px;text-align:center;">Description</th>
              <th style="border:1px solid #ddd;padding:8px;text-align:center;">Help</th>
            </tr>
          </thead>
          <tbody align="center">
            <tr>
              <td style="border:1px solid #ddd;padding:8px;text-align:center;">User &amp; Group ID</td>
              <td style="border:1px solid #ddd;padding:8px;text-align:center;">Avoids permission issues when application has access to volumes.</td>
              <td style="border:1px solid #ddd;padding:8px;text-align:center;">use command &lt;br&gt;&#x27;id user_name&#x27; to get UID &amp; GID</td>
            </tr>
            <tr>
              <td style="border:1px solid #ddd;padding:8px;text-align:center;">Timezone</td>
              <td style="border:1px solid #ddd;padding:8px;text-align:center;">Specify a timezone for the application.</td>
              <td style="border:1px solid #ddd;padding:8px;text-align:center;">View timezone options <a href="https://en.wikipedia.org/wiki/List_of_tz_database_time_zones#List">here</a></td>
            </tr>
            <tr>
              <td style="border:1px solid #ddd;padding:8px;text-align:center;">Volumes</td>
              <td style="border:1px solid #ddd;padding:8px;text-align:center;">Give the container the absolute path of the storage location and reference it to a shorter path for the application to use.&lt;br&gt;</td>
              <td style="border:1px solid #ddd;padding:8px;text-align:center;">\[Absolute Path]:\[Relative Path]</td>
            </tr>
            <tr>
              <td style="border:1px solid #ddd;padding:8px;text-align:center;">Ports</td>
              <td style="border:1px solid #ddd;padding:8px;text-align:center;">Set ports to be used inside and outside of the container. Internal ports are only used within the container. External ports are set to be accessible using the hosts IP outside of the container.</td>
              <td style="border:1px solid #ddd;padding:8px;text-align:center;">\[External]:\[Internal]</td>
            </tr>
            <tr>
              <td style="border:1px solid #ddd;padding:8px;text-align:center;">Network Modes</td>
              <td style="border:1px solid #ddd;padding:8px;text-align:center;">\[Bridge] - Connects containers to a virtual bridge, allowing them to communicate.&lt;br&gt;\[Host] - Containers directly share the hosts network stack including IP address and ports. &lt;br&gt;\[None] - Completely isolates a container from access to the network.</td>
              <td style="border:1px solid #ddd;padding:8px;text-align:center;">Docker Wiki - Networking (<a href="https://docs.docker.com/engine/network/">Link</a>)</td>
            </tr>
          </tbody>
        </table>
      </div>
    </details>
  </div>
</details>

<!-- Installation & Configuration Guides -->
<details open>
  <summary><h2 id="installation-configuration-guides" style="display:inline;">Installation &amp; Configuration Guides</h2></summary>
  <div align="justify" style="margin-top:8px;">
    <details open>
      <summary><h3 id="heimdall">Heimdall:</h3></summary>
      <details open>
        <summary><h4 id="overview">Overview:</h4></summary>
        <p><a href="https://github.com/linuxserver/Heimdall">Heimdall</a> is a locally hosted application dashboard that is most commonly used as a central point of access to the web interfaces of other applications being hosted on the same server. By default, Heimdall is operated using ports 80 &amp; 443. Due to Open Media Vault using the same ones, the external ports for HTTP &amp; HTTPS were set to 8888 &amp; 8444 respectively.</p>
      </details>
      <details open>
        <summary><h4 id="shared-folders">Shared Folders:</h4></summary>
        <pre><code>/data/heimdall
/data/heimdall/config</code></pre>
      </details>
      <details open>
        <summary><h4 id="compose-file">Compose File:</h4></summary>
        <pre><code>---
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
    restart: always</code></pre>
      </details>
    </details>
    <details open>
      <summary><h3 id="portainer">Portainer:</h3></summary>
      <details open>
        <summary><h4 id="overview">Overview:</h4></summary>
        <p><a href="https://github.com/portainer/portainer">Portainer</a> is an application that lets us manage our containers, images, volumes, networks, etc. all from one platform. This service will be useful to us as we can allocate resource limits to containers if needed.</p>
      </details>
      <details open>
        <summary><h4 id="compose-file">Compose File:</h4></summary>
        <pre><code>---
services:
  portainer-ce:
    image: portainer/portainer-ce:latest
    container_name: portainer
    restart: always
    ports:
      # - &quot;8000:8000&quot;
      - &quot;9000:9000&quot; 
    volumes:
      - &quot;/var/run/docker.sock:/var/run/docker.sock&quot;
      - &quot;CHANGE_TO_COMPOSE_DATA_PATH/portainer/portainer_data:/data&quot;
    command: &gt;
      -H unix:///var/run/docker.sock
      --http-enabled</code></pre>
      </details>
    </details>
    <details open>
      <summary><h3 id="watchtower">Watchtower:</h3></summary>
      <details open>
        <summary><h4 id="overview">Overview:</h4></summary>
        <p><a href="https://github.com/containrrr/watchtower">Watchtower</a> can be ran in its own container for the purpose of updating and restarting other containers. This service is a must as it automates the tedious task of checking containers for updates and having to manually restart them.</p>
      </details>
      <details open>
        <summary><h4 id="configuration">Configuration:</h4></summary>
        <details open>
          <summary><h5 id="scheduling">Scheduling:</h5></summary>
          <p>The schedule for checking if updates are available can be set using the following format. Currently, it is setup so that it scans for updates everyday at 4 AM.</p>
          <div style="overflow-x:auto;">
            <table style="border-collapse:collapse;width:100%;">
              <thead>
                <tr style="background-color:#f5f5f5;" align="center">
                  <th style="border:1px solid #ddd;padding:8px;text-align:center;">Second</th>
                  <th style="border:1px solid #ddd;padding:8px;text-align:center;">Minute</th>
                  <th style="border:1px solid #ddd;padding:8px;text-align:center;">Hour</th>
                  <th style="border:1px solid #ddd;padding:8px;text-align:center;">Day</th>
                  <th style="border:1px solid #ddd;padding:8px;text-align:center;">Month</th>
                  <th style="border:1px solid #ddd;padding:8px;text-align:center;">Weekday</th>
                </tr>
              </thead>
              <tbody align="center">
                <tr>
                  <td style="border:1px solid #ddd;padding:8px;text-align:center;">0</td>
                  <td style="border:1px solid #ddd;padding:8px;text-align:center;">0</td>
                  <td style="border:1px solid #ddd;padding:8px;text-align:center;">4</td>
                  <td style="border:1px solid #ddd;padding:8px;text-align:center;">\*</td>
                  <td style="border:1px solid #ddd;padding:8px;text-align:center;">\*</td>
                  <td style="border:1px solid #ddd;padding:8px;text-align:center;">\*</td>
                </tr>
              </tbody>
            </table>
          </div>
        </details>
      </details>
      <details open>
        <summary><h4 id="compose-file">Compose File:</h4></summary>
        <pre><code>---
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
    restart: always</code></pre>
      </details>
    </details>
    <details open>
      <summary><h3 id="cloudflared">Cloudflared:</h3></summary>
      <details open>
        <summary><h4 id="overview">Overview:</h4></summary>
        <p><a href="https://github.com/jonas-merkle/container-cloudflare-tunnel">Cloudflared</a> acts as the string of this servers applications, connecting everything together. By setting up a domain to point to your public IP, you can then run the local IP and ports of your applications through tunnels to make them both more secure and remotely accessible. Using a reverse proxy to increase security and accessibility is pretty mainstream, but those aren&#x27;t the only perks gained from doing this. By using Cloudflare Tunnels, the problem of being unable to utilize port forwarding due to CGNAT is directly bypassed. We can receive traffic from external sources through the tunnels!</p>
      </details>
      <details open>
        <summary><h4 id="configuration">Configuration:</h4></summary>
        <p>In order to utilize the tunnels, you need a domain name. There are plenty of suppliers but for simplicities sake, choose Cloudflare. As far as guides go, there is a great video on YouTube by <a href="https://www.youtube.com/watch?v=ey4u7OUAF3c&amp;amp;t=218s">NetworkChuck</a> that covers every step required to configure your tunnels. As can be seen in the compose file, the value of &#x27;TUNNEL_TOKEN&#x27; is &#x27;${TUNNEL_TOKEN}&#x27;. That just means that it&#x27;s referencing a variable from the global environment. To configure your token, login to Open Media Vault and go to &#x27;Services &gt; Compose &gt; Files&#x27; and select the &#x27;paper with the star&#x27; icon. It should be in the left-middle of the window right next to the scissors icon. Once selected, enter the following line:</p>
        <pre><code>TUNNEL_TOKEN=YourTunnelToken</code></pre>
      </details>
      <details open>
        <summary><h4 id="compose-file">Compose File:</h4></summary>
        <pre><code>---
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
      test: [&quot;CMD&quot;, &quot;cloudflared&quot;, &quot;--version&quot;]
      interval: 60s
      timeout: 10s
      retries: 3
      start_period: 10s</code></pre>
      </details>
    </details>
    <details open>
      <summary><h3 id="gluetun">Gluetun:</h3></summary>
      <details open>
        <summary><h4 id="overview">Overview:</h4></summary>
        <p><a href="https://github.com/qdm12/gluetun">Gluetun</a> is a lightweight VPN client that can be used to connect to multiple VPN service providers. I personally use <a href="https://protonvpn.com/?srsltid=AfmBOoqpGGOJ91RAD8M9didvkwL-sjlc_QwECYhwM5zxT3WiE0SrUDvy">ProtonVPN</a> and with them I can choose to create a WireGuard configuration. This gives me the private key and addresses needed to successfully setup my VPN. For this application there is no compose file, as it is ran in the same container as <a href="#qbittorrent">qBittorrent</a>.</p>
      </details>
    </details>
    <details open>
      <summary><h3 id="nextcloud">Nextcloud:</h3></summary>
      <details open>
        <summary><h4 id="overview">Overview:</h4></summary>
        <p><a href="https://github.com/nextcloud/all-in-one">Nextcloud All-in-One</a> is the application ran on this server to self-host a cloud storage platform that can be accessed from any location. There are other services included in this installation including:</p>
        <ul>
          <li>ClamAV (Antivirus)</li>
          <li>Collabora (Nextcloud Office)</li>
          <li>Fulltextsearch</li>
          <li>Imaginary (File Previews)</li>
          <li>Nextcloud Talk (VOIP Service)</li>
          <li>OnlyOffice</li>
          <li>Docker Socket Proxy</li>
          <li>Whiteboard</li>
        </ul>
        <p>Due to the lack of RAM on the ROCK 5C (8gb), everything besides &#x27;Imaginary&#x27; is deselected. Nextcloud is only being used in this setup for its cloud storage functionality.</p>
      </details>
      <details open>
        <summary><h4 id="configuration">Configuration:</h4></summary>
        <p>As a product of using Cloudflare Tunnels as a sort of reverse proxy, the two ports (80, 8443) were commented out of their section. Also, because of the reverse proxy Apache has to be used. I didn&#x27;t change anything for Apache and used the default configuration. Take note of the Apache IP and port because that&#x27;s what will be used to setup the tunnel using Cloudflare. If not using the ROCK 5C, navigate to the /dev/ folder and see if &#x27;dri&#x27; is available. If it is, keep &#x27;NEXTCLOUD_ENABLE_DRI_DEVICE&#x27; uncommented. Otherwise, comment it out. If using a reverse proxy, keep &#x27;SKIP_DOMAIN_VALIDATION&#x27; uncommented.</p>
      </details>
      <details open>
        <summary><h4 id="shared-folders">Shared Folders:</h4></summary>
        <pre><code>/data/nextcloud
/data/nextcloud/datadir</code></pre>
      </details>
      <details open>
        <summary><h4 id="compose-file">Compose File:</h4></summary>
        <pre><code>services:
  nextcloud-aio-mastercontainer:
    image: ghcr.io/nextcloud-releases/all-in-one:latest 
    init: true 
    restart: always 
    container_name: nextcloud-aio-mastercontainer # This line is not allowed to be changed as otherwise AIO will not work correctly
    volumes:
      - nextcloud_aio_mastercontainer:/mnt/docker-aio-config # This line is not allowed to be changed as otherwise the built-in backup solution will not work
      - /var/run/docker.sock:/var/run/docker.sock:ro 
    network_mode: bridge 
    # networks: [&quot;nextcloud-aio&quot;]
    ports:
      # 80:80 
      - 8080:8080 
      # 8443:8443 
    # security_opt: [&quot;label:disable&quot;] 
    environment: # Is needed when using any of the options below
      # AIO_DISABLE_BACKUP_SECTION: false 
      APACHE_PORT: 11000 
      APACHE_IP_BINDING: 127.0.0.1 
      # APACHE_ADDITIONAL_NETWORK: frontend_net # (Optional) 
      # BORG_RETENTION_POLICY: --keep-within=7d --keep-weekly=4 --keep-monthly=6 
      # COLLABORA_SECCOMP_DISABLED: false 
      # FULLTEXTSEARCH_JAVA_OPTIONS: &quot;-Xms1024M -Xmx1024M&quot; 
      NEXTCLOUD_DATADIR: /homelab/data/nextcloud/datadir # Allows to set the host directory for Nextcloud&#x27;s datadir. ⚠️⚠️⚠️ Warning: do not set or adjust this value after the initial Nextcloud installation is done!
      # NEXTCLOUD_MOUNT: /mnt/
      # NEXTCLOUD_UPLOAD_LIMIT: 16G # Can be adjusted if you need more.
      # NEXTCLOUD_MAX_TIME: 3600 # Can be adjusted if you need more. 
      # NEXTCLOUD_MEMORY_LIMIT: 512M # Can be adjusted if you need more.
      # NEXTCLOUD_TRUSTED_CACERTS_DIR: /path/to/my/cacerts 
      # NEXTCLOUD_STARTUP_APPS: deck twofactor_totp tasks calendar contacts notes
      # NEXTCLOUD_ADDITIONAL_APKS: imagemagick 
      # NEXTCLOUD_ADDITIONAL_PHP_EXTENSIONS: imagick 
      NEXTCLOUD_ENABLE_DRI_DEVICE: true # This allows to enable the /dev/dri device for containers that profit from it. ⚠️⚠️⚠️ Warning: this only works if the &#x27;/dev/dri&#x27; device is present on the host!
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
  # caddy_sites:</code></pre>
      </details>
    </details>
    <details open>
      <summary><h3 id="plex-media-server">Plex Media Server:</h3></summary>
      <details open>
        <summary><h4 id="overview">Overview:</h4></summary>
        <p><a href="https://github.com/plexinc/pms-docker">Plex Media Server (PMS)</a> is an application that organizes video, music, and photos from personal media libraries and allows them to be streamed to other devices logged in under the same user. Most commonly, Plex is used to create a locally hosted streaming platform for downloaded movies and tv-shows that can be accessed anywhere.</p>
      </details>
      <details open>
        <summary><h4 id="configuration">Configuration:</h4></summary>
        <p>In order to setup PMS, you&#x27;re going to need the UID and GID for &#x27;appuser&#x27; and the correct timezone. In addition to that, two folders have to be created to store the movies and tv-shows we want to be able to stream. After creating those folders, change the movies and tv pathways to accurately reflect the shared folder locations. The only port required for this application is port 32400. By running the host network mode for this application, no ports have to be defined in the compose file.</p>
      </details>
      <details open>
        <summary><h4 id="shared-folders">Shared Folders:</h4></summary>
        <pre><code>/data/plex/tvseries
/data/plex/movies</code></pre>
      </details>
      <details open>
        <summary><h4 id="compose-file">Compose File:</h4></summary>
        <pre><code>---
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
    restart: unless-stopped</code></pre>
      </details>
    </details>
    <details open>
      <summary><h3 id="qbittorrent">qBittorrent:</h3></summary>
      <details open>
        <summary><h4 id="overview">Overview:</h4></summary>
        <p><a href="https://github.com/linuxserver/docker-qbittorrent">qBittorrent</a> is a BitTorrent client that uses the P2P protocol to download and share files. It is frequently referred to as the open-source alternative to µTorrent. This type of data sharing protocol breaks large files into small pieces and relies on users to &#x27;seed&#x27; the files after downloading. This makes it so that instead of downloading a file from a single server, users essentially download pieces of a file directly to and from each other.</p>
      </details>
      <details open>
        <summary><h4 id="configuration">Configuration:</h4></summary>
        <p>The compose file for this application is different than the others, as there are two services running in this one container. In this container, we are running Gluetun and qBittorrent together.</p>
        <div style="overflow-x:auto;">
          <table style="border-collapse:collapse;width:100%;">
            <thead>
              <tr style="background-color:#f5f5f5;" align="center">
                <th style="border:1px solid #ddd;padding:8px;text-align:center;">VPN Configuration Options &amp; Other Info</th>
                <th style="border:1px solid #ddd;padding:8px;text-align:center;">&lt;</th>
              </tr>
            </thead>
            <tbody align="center">
              <tr>
                <td style="border:1px solid #ddd;padding:8px;text-align:center;">Supported VPN Providers&lt;br&gt;(OpenVPN)&lt;br&gt;</td>
                <td style="border:1px solid #ddd;padding:8px;text-align:center;">AirVPN, Cyberghost, ExpressVPN, FastestVPN, Giganews, HideMyAss, IPVanish, IVPN, Mullvad, NordVPN, Perfect Privacy, Privado, Private Internet Access, PrivateVPN, ProtonVPN, PureVPN, SlickVPN, Surfshark, TorGuard, VPNSecure.me, VPNUnlimited, Vyprvpn, WeVPN, Windscribe</td>
              </tr>
              <tr>
                <td style="border:1px solid #ddd;padding:8px;text-align:center;">Supported VPN Providers&lt;br&gt;(WireGuard)</td>
                <td style="border:1px solid #ddd;padding:8px;text-align:center;">AirVPN, FastestVPN, Ivpn, Mullvad, NordVPN, Perfect Privacy, ProtonVPN, Surfshark, Windscribe</td>
              </tr>
              <tr>
                <td style="border:1px solid #ddd;padding:8px;text-align:center;">Ports</td>
                <td style="border:1px solid #ddd;padding:8px;text-align:center;">Make sure the &#x27;qB WebUI&#x27; port for Gluetun is the same as the &#x27;WEBUI_PORT&#x27; for qBittorrent</td>
              </tr>
              <tr>
                <td style="border:1px solid #ddd;padding:8px;text-align:center;">WIREGUARD_PRIVATE_KEY</td>
                <td style="border:1px solid #ddd;padding:8px;text-align:center;">Obtained from setting up a WireGuard Configuration through VPN provider</td>
              </tr>
              <tr>
                <td style="border:1px solid #ddd;padding:8px;text-align:center;">WIREGUARD_ADDRESSES</td>
                <td style="border:1px solid #ddd;padding:8px;text-align:center;">Obtained from setting up a WireGuard Configuration through VPN provider</td>
              </tr>
              <tr>
                <td style="border:1px solid #ddd;padding:8px;text-align:center;">SERVER_CITIES</td>
                <td style="border:1px solid #ddd;padding:8px;text-align:center;">Can change to SERVER_COUNTRIES or SERVER_HOSTNAMES&lt;br&gt;Full list depends on VPN provider</td>
              </tr>
              <tr>
                <td style="border:1px solid #ddd;padding:8px;text-align:center;">LocalIP</td>
                <td style="border:1px solid #ddd;padding:8px;text-align:center;">LocalIP is a modified version of the ROCK 5C device IP. &lt;br&gt;Ex) If my IP was 192.168.10.XXX then the LocalIP would be 192.168.10.0/24</td>
              </tr>
            </tbody>
          </table>
        </div>
      </details>
      <details open>
        <summary><h4 id="shared-folders">Shared Folders:</h4></summary>
        <pre><code>/data/qbittorrent
/data/qbittorrent/appdata
/data/qbittorrent/downloads

/data/gluetun</code></pre>
      </details>
      <details open>
        <summary><h4 id="compose-file">Compose File:</h4></summary>
        <pre><code>networks:
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

      # Example (WireGuard) – fast &amp; preferred:
      - VPN_SERVICE_PROVIDER=protonvpn        # mullvad | protonvpn | pia | nordvpn | ...
      - VPN_TYPE=wireguard
      - WIREGUARD_PRIVATE_KEY=PrivateKey
      - WIREGUARD_ADDRESSES=WgAddress    # provider gives you this or it&#x27;s in the wg config
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
    # Expose qBittorrent&#x27;s ports HERE (since qB routes through gluetun)
    networks: [NetworkName]
    ports:
      - &quot;8585:8585&quot;       # qBittorrent WebUI
      - &quot;6881:6881&quot;       # TCP
      - &quot;6881:6881/udp&quot;   # UDP
    restart: unless-stopped

  qbittorrent:
    image: lscr.io/linuxserver/qbittorrent:latest
    container_name: qbittorrent
    network_mode: &quot;service:gluetun&quot;   # route all qB traffic through gluetun
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
    restart: unless-stopped</code></pre>
      </details>
    </details>
    <details open>
      <summary><h3 id="flaresolverr">Flaresolverr:</h3></summary>
      <details open>
        <summary><h4 id="overview">Overview:</h4></summary>
        <p><a href="https://github.com/FlareSolverr/FlareSolverr">Flaresolverr</a> is a proxy server that is used to bypass Cloudflare and DDoS-GUARD protection on websites. We will be running this so that we can access indexers like x1337x with other applications like Prowlarr, Radarr, and Sonarr.</p>
      </details>
      <details open>
        <summary><h4 id="configuration">Configuration:</h4></summary>
        <p>Besides creating the two shared folders, there&#x27;s only one other thing that has to be done to finish setting up this application. In order to allow Flaresolverr to interact with Prowlarr, a network bridge has to be created. In Open Media Vault, go to &#x27;Services &gt; Compose &gt; Networks&#x27; and add a new network. Only input the name and make sure that the driver is set to &#x27;bridge&#x27;.</p>
      </details>
      <details open>
        <summary><h4 id="shared-folders">Shared Folders:</h4></summary>
        <pre><code>/data/flaresolverr
/data/flaresolverr/config</code></pre>
      </details>
      <details open>
        <summary><h4 id="compose-file">Compose File:</h4></summary>
        <pre><code>---
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
      - &quot;${PORT:-8191}:8191&quot;
    volumes:
      - /path/to/flaresolverr/config:/config
    restart: unless-stopped</code></pre>
      </details>
    </details>
    <details open>
      <summary><h3 id="prowlarr">Prowlarr:</h3></summary>
      <details open>
        <summary><h4 id="overview">Overview:</h4></summary>
        <p><a href="https://github.com/linuxserver/docker-prowlarr">Prowlarr</a> is a manager for indexers that integrates seamlessly with Sonarr, Radarr, and other -arr applications. By using this service, we can effectively have complete management over all indexers for other applications, bypassing that &#x27;per app&#x27; setup.</p>
      </details>
      <details open>
        <summary><h4 id="configuration">Configuration:</h4></summary>
        <p>To setup this application, you will need the UID and GID for &#x27;appuser&#x27;. After getting Prowlarr up and running, login to the service. After logging in, make sure to do the following:</p>
        <ul>
          <li>Go to &#x27;Settings &gt; Indexers&#x27; and add Flaresolverr. (Host: http://flaresolverr:8191)</li>
          <li>Add the tags &#x27;movies&#x27;, &#x27;tv&#x27;, and &#x27;other&#x27; to the indexer proxy. </li>
          <li>Then go to &#x27;Indexers&#x27; and hit &#x27;Add Indexer&#x27;. Add whatever indexers you want. At the time of making this I use the following:</li>
          <li>1337x</li>
          <li>Knaben</li>
          <li>The Pirate Bay</li>
          <li>Uindex</li>
          <li>YTS</li>
          <li>Then, go to &#x27;Settings &gt; Download Clients&#x27; and add qBittorent. For &#x27;Host&#x27; use the IP of the ROCK 5C and set the port to be the same as what is used by qBittorent. Type in the username and password used to login to qB and then add two mapped categories, one for movies and one for tv-shows.</li>
          <li>Before continuing the setup for Prowlarr, make sure Radarr and Sonarr are installed. </li>
        </ul>
      </details>
      <details open>
        <summary><h4 id="shared-folders">Shared Folders:</h4></summary>
        <pre><code>/data/prowlarr
/data/prowlarr/config</code></pre>
      </details>
      <details open>
        <summary><h4 id="compose-file">Compose File:</h4></summary>
        <pre><code>---
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
    ports: [&quot;9696:9696&quot;]
    restart: unless-stopped</code></pre>
      </details>
    </details>
    <details open>
      <summary><h3 id="radarr">Radarr:</h3></summary>
      <details open>
        <summary><h4 id="overview">Overview:</h4></summary>
        <p><a href="https://github.com/Radarr/Radarr">Radarr</a> is a movie collection manager for Usenet and BitTorrent users. It can monitor the web for new movies and can interface directly with clients and indexers to obtain and manage them on a file level. A useful feature of this application is that it can &#x27;upgrade&#x27; movies to a higher quality as long as you set its quality profile. This is the application used with this server to manage the Plex movie collection.</p>
      </details>
      <details open>
        <summary><h4 id="configuration">Configuration:</h4></summary>
        <p>To setup Radarr, a few things have to be done. First, you have to grab the UID and GID from &#x27;appuser&#x27; for the compose file. Then, create the required shared folders for the application. After that, fill in your network name in order to ensure that is has access to the qBittorrent client.</p>
      </details>
      <details open>
        <summary><h4 id="shared-folders">Shared Folders:</h4></summary>
        <pre><code>/data/radarr
/data/radarr/config</code></pre>
      </details>
      <details open>
        <summary><h4 id="compose-file">Compose File:</h4></summary>
        <pre><code>---
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
      - &quot;7878:7878&quot;
    restart: unless-stopped</code></pre>
      </details>
    </details>
    <details open>
      <summary><h3 id="sonarr">Sonarr:</h3></summary>
      <details open>
        <summary><h4 id="overview">Overview:</h4></summary>
        <p><a href="https://github.com/Sonarr/Sonarr">Sonarr</a>, like Radarr, is another manager that is used in this setup. It also monitors multiple RSS feeds to find new episodes of managed series. In terms of functionality, it&#x27;s the same as Radarr except for TV shows instead of movies.</p>
      </details>
      <details open>
        <summary><h4 id="configuration">Configuration:</h4></summary>
        <p>See configuration guide for <a href="#radarr">Radarr</a>.</p>
      </details>
      <details open>
        <summary><h4 id="shared-folders">Shared Folders:</h4></summary>
        <pre><code>/data/sonarr
/data/sonarr/config</code></pre>
      </details>
      <details open>
        <summary><h4 id="compose-file">Compose File:</h4></summary>
        <pre><code>---
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
      - &quot;8989:8989&quot;
    restart: unless-stopped</code></pre>
      </details>
    </details>
  </div>
</details>

<!-- Utilized Ports (External) -->
<details open>
  <summary><h2 id="utilized-ports-external" style="display:inline;">Utilized Ports (External)</h2></summary>
  <div align="justify" style="margin-top:8px;">
    <div style="overflow-x:auto;">
      <table style="border-collapse:collapse;width:100%;">
        <thead>
          <tr style="background-color:#f5f5f5;" align="center">
            <th style="border:1px solid #ddd;padding:8px;text-align:center;">Port Layout</th>
            <th style="border:1px solid #ddd;padding:8px;text-align:center;">&lt;</th>
          </tr>
        </thead>
        <tbody align="center">
          <tr>
            <td style="border:1px solid #ddd;padding:8px;text-align:center;"><strong>Service</strong></td>
            <td style="border:1px solid #ddd;padding:8px;text-align:center;"><strong>Port</strong></td>
          </tr>
          <tr>
            <td style="border:1px solid #ddd;padding:8px;text-align:center;">Flaresolverr</td>
            <td style="border:1px solid #ddd;padding:8px;text-align:center;">8191</td>
          </tr>
          <tr>
            <td style="border:1px solid #ddd;padding:8px;text-align:center;">Gluetun - Torrent</td>
            <td style="border:1px solid #ddd;padding:8px;text-align:center;">6881</td>
          </tr>
          <tr>
            <td style="border:1px solid #ddd;padding:8px;text-align:center;">Gluetun - qB</td>
            <td style="border:1px solid #ddd;padding:8px;text-align:center;">8585</td>
          </tr>
          <tr>
            <td style="border:1px solid #ddd;padding:8px;text-align:center;">Heimdall - https</td>
            <td style="border:1px solid #ddd;padding:8px;text-align:center;">8444</td>
          </tr>
          <tr>
            <td style="border:1px solid #ddd;padding:8px;text-align:center;">Heimdall - http</td>
            <td style="border:1px solid #ddd;padding:8px;text-align:center;">8888</td>
          </tr>
          <tr>
            <td style="border:1px solid #ddd;padding:8px;text-align:center;">Nextcloud - AIO</td>
            <td style="border:1px solid #ddd;padding:8px;text-align:center;">8080</td>
          </tr>
          <tr>
            <td style="border:1px solid #ddd;padding:8px;text-align:center;">Nextcloud - Apache</td>
            <td style="border:1px solid #ddd;padding:8px;text-align:center;">11000</td>
          </tr>
          <tr>
            <td style="border:1px solid #ddd;padding:8px;text-align:center;">Portainer</td>
            <td style="border:1px solid #ddd;padding:8px;text-align:center;">9000</td>
          </tr>
          <tr>
            <td style="border:1px solid #ddd;padding:8px;text-align:center;">Prowlarr</td>
            <td style="border:1px solid #ddd;padding:8px;text-align:center;">9696</td>
          </tr>
          <tr>
            <td style="border:1px solid #ddd;padding:8px;text-align:center;">Radarr</td>
            <td style="border:1px solid #ddd;padding:8px;text-align:center;">7878</td>
          </tr>
          <tr>
            <td style="border:1px solid #ddd;padding:8px;text-align:center;">Sonarr</td>
            <td style="border:1px solid #ddd;padding:8px;text-align:center;">8989</td>
          </tr>
          <tr>
            <td style="border:1px solid #ddd;padding:8px;text-align:center;">Uptime Kuma</td>
            <td style="border:1px solid #ddd;padding:8px;text-align:center;">3001</td>
          </tr>
          <tr>
            <td style="border:1px solid #ddd;padding:8px;text-align:center;">Watchtower</td>
            <td style="border:1px solid #ddd;padding:8px;text-align:center;">7272</td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</details>
