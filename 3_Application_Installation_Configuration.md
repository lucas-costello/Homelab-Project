<!doctype html>
<html>
<head>
<meta charset="utf-8">
<title>Homelab Installation Guide</title>
</head>
<body>
<h2 id="overview">Overview</h2>
<p>At this point, the server hardware is assembled and the operating system is configured. Now, the applications have to be downloaded to turn this remote terminal into a fully functional homelab. The installation of applications on this device centers around Docker, specifically the use of Docker Compose, to create containers that will run each application independently. Below is the list of all applications currently running on the ROCK 5C server along with instructions on how to install them.</p>
<h3 id="ui--system-management">UI &amp; System Management:</h3>
<ul>
<li><a href="#heimdall">Heimdall</a> (Application Dashboard)</li>
<li><a href="#portainer">Portainer</a> (Container Management)</li>
<li><a href="#watchtower">Watchtower</a> (Container Update Automation)</li>
</ul>
<h3 id="connectivity">Connectivity:</h3>
<ul>
<li><a href="#cloudflared">Cloudflared</a> (Cloudflare Tunnels | Reverse Proxy)</li>
<li><a href="#gluetun">Gluetun</a> (VPN Client)</li>
</ul>
<h3 id="files--media">Files &amp; Media:</h3>
<ul>
<li><a href="#nextcloud">Nextcloud</a> (Cloud Storage)</li>
<li><a href="#plex-media-server">Plex Media Server</a> (Personal Media Streaming)</li>
</ul>
<h3 id="p2p-downloading-setup">P2P Downloading Setup:</h3>
<ul>
<li><a href="#flaresolverr">Flaresolverr</a> (Bypass Cloudflare)</li>
<li><a href="#prowlarr">Prowlarr</a> (Indexer Management)</li>
<li><a href="#qbittorrent">qBittorrent</a> (P2P Downloader Client)</li>
<li><a href="#radarr">Radarr</a> (Movie Collection Manager)</li>
<li><a href="#sonarr">Sonarr</a> (TV-Series Collection Manager)</li>
</ul>
<h2 id="pre-requisites">Pre-Requisites:</h2>
<h3 id="user-group-creation">User Group Creation:</h3>
<p>Before any applications are installed, make sure to go to the 'Users' section in Open Media Vault and add the following user:</p>
<table>
<thead>
<tr class="header">
<th style="text-align: center;">Name</th>
<th style="text-align: center;">Shell</th>
<th style="text-align: center;">Groups</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: center;">appuser</td>
<td style="text-align: center;">/usr/sbin/nologin</td>
<td style="text-align: center;">users</td>
</tr>
</tbody>
</table>
<p>After creating the user, apply changes and then SSH into the ROCK 5C. Then, use the following command and keep note of the users 'UID' and 'GID':</p>
<div class="sourceCode" id="cb1"><pre class="sourceCode bash"><code class="sourceCode bash"><span id="cb1-1"><a href="#cb1-1" aria-hidden="true" tabindex="-1"></a><span class="fu">id</span> appuser</span></code></pre></div>
<h3 id="folder-creation">Folder Creation:</h3>
<p>Another thing to do before installing any applications is to create a storage location on the ZFS pool for any data relating to applications. I just created a data folder but you can name it as you see fit.</p>
<table>
<thead>
<tr class="header">
<th style="text-align: center;">Name</th>
<th style="text-align: center;">File System</th>
<th style="text-align: center;">Relative Path</th>
<th style="text-align: center;">Permissions</th>
<th style="text-align: center;">Tags</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: center;">data</td>
<td style="text-align: center;">(PoolName)</td>
<td style="text-align: center;">/data</td>
<td style="text-align: center;">Don't Change</td>
<td style="text-align: center;">App Data Folder</td>
</tr>
</tbody>
</table>
<p>Also, a good practice is to create a shared folder for everything. For the applications listed, if a shared folder is required then it will be listed in the instructions.</p>
<h3 id="installing-applications-using-docker-containers">Installing Applications Using Docker Containers:</h3>
<p>By using Docker Containers, the process of running multi-container Docker applications has never been easier. Compose relies on using YAML files to configure an application's services for each container. For the most part, each application will be within its own docker container. To get to the location to add new files, follow the pathway in Open Media Vault: 'Services &gt; Compose &gt; Files'. After getting there, click the plus button labeled 'Add' that's in the top left of the window. Here, you can input the container name, description, and YAML configuration file for the application. The last thing that has to be done prior to installing any applications is to create shared folders for Docker with the relative paths:</p>
<div class="sourceCode" id="cb2"><pre class="sourceCode bash"><code class="sourceCode bash"><span id="cb2-1"><a href="#cb2-1" aria-hidden="true" tabindex="-1"></a><span class="ex">/data/docker</span></span>
<span id="cb2-2"><a href="#cb2-2" aria-hidden="true" tabindex="-1"></a><span class="ex">/data/docker/appdata</span></span>
<span id="cb2-3"><a href="#cb2-3" aria-hidden="true" tabindex="-1"></a><span class="ex">/data/docker/compose</span></span></code></pre></div>
<p>After creating those shared folders in Open Media Vault, head over to 'Services &gt; Compose &gt; Settings' and change the shared folder location for compose files and data to the ones that were just created. With that, applications are ready to be installed.</p>
<h3 id="useful-information">Useful Information:</h3>
<table>
<thead>
<tr class="header">
<th style="text-align: center;">Topic</th>
<th style="text-align: center;">Description</th>
<th style="text-align: center;">Help</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: center;">User &amp; Group ID</td>
<td style="text-align: center;">Avoids permission issues when application has access to volumes.</td>
<td style="text-align: center;">use command <br>'id user_name' to get UID &amp; GID</td>
</tr>
<tr class="even">
<td style="text-align: center;">Timezone</td>
<td style="text-align: center;">Specify a timezone for the application.</td>
<td style="text-align: center;">View timezone options <a href="https://en.wikipedia.org/wiki/List_of_tz_database_time_zones#List">here</a></td>
</tr>
<tr class="odd">
<td style="text-align: center;">Volumes</td>
<td style="text-align: center;">Give the container the absolute path of the storage location and reference it to a shorter path for the application to use.<br></td>
<td style="text-align: center;">[Absolute Path]:[Relative Path]</td>
</tr>
<tr class="even">
<td style="text-align: center;">Ports</td>
<td style="text-align: center;">Set ports to be used inside and outside of the container. Internal ports are only used within the container. External ports are set to be accessible using the hosts IP outside of the container.</td>
<td style="text-align: center;">[External]:[Internal]</td>
</tr>
<tr class="odd">
<td style="text-align: center;">Network Modes</td>
<td style="text-align: center;">[Bridge] - Connects containers to a virtual bridge, allowing them to communicate.<br>[Host] - Containers directly share the hosts network stack including IP address and ports. <br>[None] - Completely isolates a container from access to the network.</td>
<td style="text-align: center;">Docker Wiki - Networking (<a href="https://docs.docker.com/engine/network/">Link</a>)</td>
</tr>
</tbody>
</table>
<h2 id="installation--configuration-guides">Installation &amp; Configuration Guides:</h2>
<h3 id="heimdall">Heimdall:</h3>
<h4 id="overview-1">Overview:</h4>
<p><a href="https://github.com/linuxserver/Heimdall">Heimdall</a> is a locally hosted application dashboard that is most commonly used as a central point of access to the web interfaces of other applications being hosted on the same server. By default, Heimdall is operated using ports 80 &amp; 443. Due to Open Media Vault using the same ones, the external ports for HTTP &amp; HTTPS were set to 8888 &amp; 8444 respectively.</p>
<h4 id="shared-folders">Shared Folders:</h4>
<div class="sourceCode" id="cb3"><pre class="sourceCode bash"><code class="sourceCode bash"><span id="cb3-1"><a href="#cb3-1" aria-hidden="true" tabindex="-1"></a><span class="ex">/data/heimdall</span></span>
<span id="cb3-2"><a href="#cb3-2" aria-hidden="true" tabindex="-1"></a><span class="ex">/data/heimdall/config</span></span></code></pre></div>
<h4 id="compose-file">Compose File:</h4>
<div class="sourceCode" id="cb4"><pre class="sourceCode yaml"><code class="sourceCode yaml"><span id="cb4-1"><a href="#cb4-1" aria-hidden="true" tabindex="-1"></a><span class="pp">---</span></span>
<span id="cb4-2"><a href="#cb4-2" aria-hidden="true" tabindex="-1"></a><span class="fu">services</span><span class="kw">:</span></span>
<span id="cb4-3"><a href="#cb4-3" aria-hidden="true" tabindex="-1"></a><span class="at">  </span><span class="fu">heimdall</span><span class="kw">:</span></span>
<span id="cb4-4"><a href="#cb4-4" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">image</span><span class="kw">:</span><span class="at"> lscr.io/linuxserver/heimdall:latest</span></span>
<span id="cb4-5"><a href="#cb4-5" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">container_name</span><span class="kw">:</span><span class="at"> heimdall</span></span>
<span id="cb4-6"><a href="#cb4-6" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">environment</span><span class="kw">:</span></span>
<span id="cb4-7"><a href="#cb4-7" aria-hidden="true" tabindex="-1"></a><span class="at">      </span><span class="kw">-</span><span class="at"> PUID=UserID</span></span>
<span id="cb4-8"><a href="#cb4-8" aria-hidden="true" tabindex="-1"></a><span class="at">      </span><span class="kw">-</span><span class="at"> PGID=GroupID</span></span>
<span id="cb4-9"><a href="#cb4-9" aria-hidden="true" tabindex="-1"></a><span class="at">      </span><span class="kw">-</span><span class="at"> TZ=Country/City</span></span>
<span id="cb4-10"><a href="#cb4-10" aria-hidden="true" tabindex="-1"></a><span class="at">      </span><span class="kw">-</span><span class="at"> ALLOW_INTERNAL_REQUESTS=true</span><span class="co"> #optional</span></span>
<span id="cb4-11"><a href="#cb4-11" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">volumes</span><span class="kw">:</span></span>
<span id="cb4-12"><a href="#cb4-12" aria-hidden="true" tabindex="-1"></a><span class="at">      </span><span class="kw">-</span><span class="at"> /path/to/heimdall/config:/config</span></span>
<span id="cb4-13"><a href="#cb4-13" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">ports</span><span class="kw">:</span></span>
<span id="cb4-14"><a href="#cb4-14" aria-hidden="true" tabindex="-1"></a><span class="at">      </span><span class="kw">-</span><span class="at"> 8888:80</span></span>
<span id="cb4-15"><a href="#cb4-15" aria-hidden="true" tabindex="-1"></a><span class="at">      </span><span class="kw">-</span><span class="at"> 8444:443</span></span>
<span id="cb4-16"><a href="#cb4-16" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">restart</span><span class="kw">:</span><span class="at"> always</span></span></code></pre></div>
<h3 id="portainer">Portainer:</h3>
<h4 id="overview-2">Overview:</h4>
<p><a href="https://github.com/portainer/portainer">Portainer</a> is an application that lets us manage our containers, images, volumes, networks, etc. all from one platform. This service will be useful to us as we can allocate resource limits to containers if needed.</p>
<h4 id="compose-file-1">Compose File:</h4>
<div class="sourceCode" id="cb5"><pre class="sourceCode yaml"><code class="sourceCode yaml"><span id="cb5-1"><a href="#cb5-1" aria-hidden="true" tabindex="-1"></a><span class="pp">---</span></span>
<span id="cb5-2"><a href="#cb5-2" aria-hidden="true" tabindex="-1"></a><span class="fu">services</span><span class="kw">:</span></span>
<span id="cb5-3"><a href="#cb5-3" aria-hidden="true" tabindex="-1"></a><span class="at">  </span><span class="fu">portainer-ce</span><span class="kw">:</span></span>
<span id="cb5-4"><a href="#cb5-4" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">image</span><span class="kw">:</span><span class="at"> portainer/portainer-ce:latest</span></span>
<span id="cb5-5"><a href="#cb5-5" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">container_name</span><span class="kw">:</span><span class="at"> portainer</span></span>
<span id="cb5-6"><a href="#cb5-6" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">restart</span><span class="kw">:</span><span class="at"> always</span></span>
<span id="cb5-7"><a href="#cb5-7" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">ports</span><span class="kw">:</span></span>
<span id="cb5-8"><a href="#cb5-8" aria-hidden="true" tabindex="-1"></a><span class="co">      # - &quot;8000:8000&quot;</span></span>
<span id="cb5-9"><a href="#cb5-9" aria-hidden="true" tabindex="-1"></a><span class="at">      </span><span class="kw">-</span><span class="at"> </span><span class="st">&quot;9000:9000&quot;</span><span class="at"> </span></span>
<span id="cb5-10"><a href="#cb5-10" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">volumes</span><span class="kw">:</span></span>
<span id="cb5-11"><a href="#cb5-11" aria-hidden="true" tabindex="-1"></a><span class="at">      </span><span class="kw">-</span><span class="at"> </span><span class="st">&quot;/var/run/docker.sock:/var/run/docker.sock&quot;</span></span>
<span id="cb5-12"><a href="#cb5-12" aria-hidden="true" tabindex="-1"></a><span class="at">      </span><span class="kw">-</span><span class="at"> </span><span class="st">&quot;CHANGE_TO_COMPOSE_DATA_PATH/portainer/portainer_data:/data&quot;</span></span>
<span id="cb5-13"><a href="#cb5-13" aria-hidden="true" tabindex="-1"></a><span class="fu">    command</span><span class="kw">: </span><span class="ch">&gt;</span></span>
<span id="cb5-14"><a href="#cb5-14" aria-hidden="true" tabindex="-1"></a>      -H unix:///var/run/docker.sock</span>
<span id="cb5-15"><a href="#cb5-15" aria-hidden="true" tabindex="-1"></a>      --http-enabled</span></code></pre></div>
<h3 id="watchtower">Watchtower:</h3>
<h4 id="overview-3">Overview:</h4>
<p><a href="https://github.com/containrrr/watchtower">Watchtower</a> can be ran in its own container for the purpose of updating and restarting other containers. This service is a must as it automates the tedious task of checking containers for updates and having to manually restart them.</p>
<h4 id="configuration">Configuration:</h4>
<h5 id="scheduling">Scheduling:</h5>
<p>The schedule for checking if updates are available can be set using the following format. Currently, it is setup so that it scans for updates everyday at 4 AM.</p>
<table>
<thead>
<tr class="header">
<th style="text-align: center;">Second</th>
<th style="text-align: center;">Minute</th>
<th style="text-align: center;">Hour</th>
<th style="text-align: center;">Day</th>
<th style="text-align: center;">Month</th>
<th style="text-align: center;">Weekday</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: center;">0</td>
<td style="text-align: center;">0</td>
<td style="text-align: center;">4</td>
<td style="text-align: center;">*</td>
<td style="text-align: center;">*</td>
<td style="text-align: center;">*</td>
</tr>
</tbody>
</table>
<h4 id="compose-file-2">Compose File:</h4>
<div class="sourceCode" id="cb6"><pre class="sourceCode yaml"><code class="sourceCode yaml"><span id="cb6-1"><a href="#cb6-1" aria-hidden="true" tabindex="-1"></a><span class="pp">---</span></span>
<span id="cb6-2"><a href="#cb6-2" aria-hidden="true" tabindex="-1"></a><span class="fu">services</span><span class="kw">:</span></span>
<span id="cb6-3"><a href="#cb6-3" aria-hidden="true" tabindex="-1"></a><span class="at">  </span><span class="fu">watchtower</span><span class="kw">:</span></span>
<span id="cb6-4"><a href="#cb6-4" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">image</span><span class="kw">:</span><span class="at"> containrrr/watchtower</span></span>
<span id="cb6-5"><a href="#cb6-5" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">container_name</span><span class="kw">:</span><span class="at"> watchtower</span></span>
<span id="cb6-6"><a href="#cb6-6" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">volumes</span><span class="kw">:</span></span>
<span id="cb6-7"><a href="#cb6-7" aria-hidden="true" tabindex="-1"></a><span class="at">      </span><span class="kw">-</span><span class="at"> /var/run/docker.sock:/var/run/docker.sock</span></span>
<span id="cb6-8"><a href="#cb6-8" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">environment</span><span class="kw">:</span></span>
<span id="cb6-9"><a href="#cb6-9" aria-hidden="true" tabindex="-1"></a><span class="at">      </span><span class="kw">-</span><span class="at"> WATCHTOWER_SCHEDULE=0 0 4 * * *</span><span class="co">     # check nightly (4 AM)</span></span>
<span id="cb6-10"><a href="#cb6-10" aria-hidden="true" tabindex="-1"></a><span class="at">      </span><span class="kw">-</span><span class="at"> WATCHTOWER_DISABLE_CONTAINERS=nextcloud-aio-mastercontainer</span></span>
<span id="cb6-11"><a href="#cb6-11" aria-hidden="true" tabindex="-1"></a><span class="at">      </span><span class="kw">-</span><span class="at"> WATCHTOWER_CLEANUP=true</span><span class="co">     # Remove old images after updating</span></span>
<span id="cb6-12"><a href="#cb6-12" aria-hidden="true" tabindex="-1"></a><span class="at">      </span><span class="kw">-</span><span class="at"> WATCHTOWER_INCLUDE_RESTARTING=true</span><span class="co"> # Restart Containers Post Update</span></span>
<span id="cb6-13"><a href="#cb6-13" aria-hidden="true" tabindex="-1"></a><span class="at">      </span><span class="kw">-</span><span class="at"> WATCHTOWER_INCLUDE_STOPPED=true</span><span class="co">    # Allows management of created and exited containers</span></span>
<span id="cb6-14"><a href="#cb6-14" aria-hidden="true" tabindex="-1"></a><span class="at">      </span><span class="kw">-</span><span class="at"> WATCHTOWER_REVIVE_STOPPED=true</span><span class="co">     # Start any stopped containers that have had their image updated (needs WATCHTOWER_INCLUDE_STOPPED=true)</span></span>
<span id="cb6-15"><a href="#cb6-15" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">ports</span><span class="kw">:</span></span>
<span id="cb6-16"><a href="#cb6-16" aria-hidden="true" tabindex="-1"></a><span class="at">      </span><span class="kw">-</span><span class="at"> 7272:8080</span></span>
<span id="cb6-17"><a href="#cb6-17" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">restart</span><span class="kw">:</span><span class="at"> always</span></span></code></pre></div>
<h3 id="cloudflared">Cloudflared:</h3>
<h4 id="overview-4">Overview:</h4>
<p><a href="https://github.com/jonas-merkle/container-cloudflare-tunnel">Cloudflared</a> acts as the string of this servers applications, connecting everything together. By setting up a domain to point to your public IP, you can then run the local IP and ports of your applications through tunnels to make them both more secure and remotely accessible. Using a reverse proxy to increase security and accessibility is pretty mainstream, but those aren't the only perks gained from doing this. By using Cloudflare Tunnels, the problem of being unable to utilize port forwarding due to CGNAT is directly bypassed. We can receive traffic from external sources through the tunnels!</p>
<h4 id="configuration-1">Configuration:</h4>
<p>In order to utilize the tunnels, you need a domain name. There are plenty of suppliers but for simplicities sake, choose Cloudflare. As far as guides go, there is a great video on YouTube by <a href="https://www.youtube.com/watch?v=ey4u7OUAF3c&amp;t=218s">NetworkChuck</a> that covers every step required to configure your tunnels. As can be seen in the compose file, the value of 'TUNNEL_TOKEN' is '${TUNNEL_TOKEN}'. That just means that it's referencing a variable from the global environment. To configure your token, login to Open Media Vault and go to 'Services &gt; Compose &gt; Files' and select the 'paper with the star' icon. It should be in the left-middle of the window right next to the scissors icon. Once selected, enter the following line:</p>
<div class="sourceCode" id="cb7"><pre class="sourceCode yaml"><code class="sourceCode yaml"><span id="cb7-1"><a href="#cb7-1" aria-hidden="true" tabindex="-1"></a><span class="at">TUNNEL_TOKEN=YourTunnelToken</span></span></code></pre></div>
<h4 id="compose-file-3">Compose File:</h4>
<div class="sourceCode" id="cb8"><pre class="sourceCode yaml"><code class="sourceCode yaml"><span id="cb8-1"><a href="#cb8-1" aria-hidden="true" tabindex="-1"></a><span class="pp">---</span></span>
<span id="cb8-2"><a href="#cb8-2" aria-hidden="true" tabindex="-1"></a><span class="fu">services</span><span class="kw">:</span><span class="at"> </span></span>
<span id="cb8-3"><a href="#cb8-3" aria-hidden="true" tabindex="-1"></a><span class="at">  </span><span class="fu">cloudflared</span><span class="kw">:</span><span class="at"> </span></span>
<span id="cb8-4"><a href="#cb8-4" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">image</span><span class="kw">:</span><span class="at"> cloudflare/cloudflared </span></span>
<span id="cb8-5"><a href="#cb8-5" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">container_name</span><span class="kw">:</span><span class="at"> cloudflare-tunnel</span></span>
<span id="cb8-6"><a href="#cb8-6" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">network_mode</span><span class="kw">:</span><span class="at"> host</span></span>
<span id="cb8-7"><a href="#cb8-7" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">restart</span><span class="kw">:</span><span class="at"> always</span></span>
<span id="cb8-8"><a href="#cb8-8" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">environment</span><span class="kw">:</span></span>
<span id="cb8-9"><a href="#cb8-9" aria-hidden="true" tabindex="-1"></a><span class="at">      </span><span class="kw">-</span><span class="at"> TUNNEL_TOKEN=${TUNNEL_TOKEN}</span></span>
<span id="cb8-10"><a href="#cb8-10" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">command</span><span class="kw">:</span><span class="at"> tunnel --no-autoupdate run --token ${TUNNEL_TOKEN}</span></span>
<span id="cb8-11"><a href="#cb8-11" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">healthcheck</span><span class="kw">:</span></span>
<span id="cb8-12"><a href="#cb8-12" aria-hidden="true" tabindex="-1"></a><span class="at">      </span><span class="fu">test</span><span class="kw">:</span><span class="at"> </span><span class="kw">[</span><span class="st">&quot;CMD&quot;</span><span class="kw">,</span><span class="at"> </span><span class="st">&quot;cloudflared&quot;</span><span class="kw">,</span><span class="at"> </span><span class="st">&quot;--version&quot;</span><span class="kw">]</span></span>
<span id="cb8-13"><a href="#cb8-13" aria-hidden="true" tabindex="-1"></a><span class="at">      </span><span class="fu">interval</span><span class="kw">:</span><span class="at"> 60s</span></span>
<span id="cb8-14"><a href="#cb8-14" aria-hidden="true" tabindex="-1"></a><span class="at">      </span><span class="fu">timeout</span><span class="kw">:</span><span class="at"> 10s</span></span>
<span id="cb8-15"><a href="#cb8-15" aria-hidden="true" tabindex="-1"></a><span class="at">      </span><span class="fu">retries</span><span class="kw">:</span><span class="at"> </span><span class="dv">3</span></span>
<span id="cb8-16"><a href="#cb8-16" aria-hidden="true" tabindex="-1"></a><span class="at">      </span><span class="fu">start_period</span><span class="kw">:</span><span class="at"> 10s</span></span></code></pre></div>
<h3 id="gluetun">Gluetun:</h3>
<h4 id="overview-5">Overview:</h4>
<p><a href="https://github.com/qdm12/gluetun">Gluetun</a> is a lightweight VPN client that can be used to connect to multiple VPN service providers. I personally use <a href="https://protonvpn.com/?srsltid=AfmBOoqpGGOJ91RAD8M9didvkwL-sjlc_QwECYhwM5zxT3WiE0SrUDvy">ProtonVPN</a> and with them I can choose to create a WireGuard configuration. This gives me the private key and addresses needed to successfully setup my VPN. For this application there is no compose file, as it is ran in the same container as <a href="#qbittorrent">qBittorrent</a>.</p>
<h3 id="nextcloud">Nextcloud:</h3>
<h4 id="overview-6">Overview:</h4>
<p><a href="https://github.com/nextcloud/all-in-one">Nextcloud All-in-One</a> is the application ran on this server to self-host a cloud storage platform that can be accessed from any location. There are other services included in this installation including:</p>
<ul>
<li>ClamAV (Antivirus)</li>
<li>Collabora (Nextcloud Office)</li>
<li>Fulltextsearch</li>
<li>Imaginary (File Previews)</li>
<li>Nextcloud Talk (VOIP Service)</li>
<li>OnlyOffice</li>
<li>Docker Socket Proxy</li>
<li>Whiteboard Due to the lack of RAM on the ROCK 5C (8gb), everything besides 'Imaginary' is deselected. Nextcloud is only being used in this setup for its cloud storage functionality.</li>
</ul>
<h4 id="configuration-2">Configuration:</h4>
<p>As a product of using Cloudflare Tunnels as a sort of reverse proxy, the two ports (80, 8443) were commented out of their section. Also, because of the reverse proxy Apache has to be used. I didn't change anything for Apache and used the default configuration. Take note of the Apache IP and port because that's what will be used to setup the tunnel using Cloudflare. If not using the ROCK 5C, navigate to the /dev/ folder and see if 'dri' is available. If it is, keep 'NEXTCLOUD_ENABLE_DRI_DEVICE' uncommented. Otherwise, comment it out. If using a reverse proxy, keep 'SKIP_DOMAIN_VALIDATION' uncommented.</p>
<h4 id="shared-folders-1">Shared Folders:</h4>
<div class="sourceCode" id="cb9"><pre class="sourceCode bash"><code class="sourceCode bash"><span id="cb9-1"><a href="#cb9-1" aria-hidden="true" tabindex="-1"></a><span class="ex">/data/nextcloud</span></span>
<span id="cb9-2"><a href="#cb9-2" aria-hidden="true" tabindex="-1"></a><span class="ex">/data/nextcloud/datadir</span></span></code></pre></div>
<h4 id="compose-file-4">Compose File:</h4>
<div class="sourceCode" id="cb10"><pre class="sourceCode yaml"><code class="sourceCode yaml"><span id="cb10-1"><a href="#cb10-1" aria-hidden="true" tabindex="-1"></a><span class="fu">services</span><span class="kw">:</span></span>
<span id="cb10-2"><a href="#cb10-2" aria-hidden="true" tabindex="-1"></a><span class="at">  </span><span class="fu">nextcloud-aio-mastercontainer</span><span class="kw">:</span></span>
<span id="cb10-3"><a href="#cb10-3" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">image</span><span class="kw">:</span><span class="at"> ghcr.io/nextcloud-releases/all-in-one:latest </span></span>
<span id="cb10-4"><a href="#cb10-4" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">init</span><span class="kw">:</span><span class="at"> </span><span class="ch">true</span><span class="at"> </span></span>
<span id="cb10-5"><a href="#cb10-5" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">restart</span><span class="kw">:</span><span class="at"> always </span></span>
<span id="cb10-6"><a href="#cb10-6" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">container_name</span><span class="kw">:</span><span class="at"> nextcloud-aio-mastercontainer</span><span class="co"> # This line is not allowed to be changed as otherwise AIO will not work correctly</span></span>
<span id="cb10-7"><a href="#cb10-7" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">volumes</span><span class="kw">:</span></span>
<span id="cb10-8"><a href="#cb10-8" aria-hidden="true" tabindex="-1"></a><span class="at">      </span><span class="kw">-</span><span class="at"> nextcloud_aio_mastercontainer:/mnt/docker-aio-config</span><span class="co"> # This line is not allowed to be changed as otherwise the built-in backup solution will not work</span></span>
<span id="cb10-9"><a href="#cb10-9" aria-hidden="true" tabindex="-1"></a><span class="at">      </span><span class="kw">-</span><span class="at"> /var/run/docker.sock:/var/run/docker.sock:ro </span></span>
<span id="cb10-10"><a href="#cb10-10" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">network_mode</span><span class="kw">:</span><span class="at"> bridge </span></span>
<span id="cb10-11"><a href="#cb10-11" aria-hidden="true" tabindex="-1"></a><span class="co">    # networks: [&quot;nextcloud-aio&quot;]</span></span>
<span id="cb10-12"><a href="#cb10-12" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">ports</span><span class="kw">:</span></span>
<span id="cb10-13"><a href="#cb10-13" aria-hidden="true" tabindex="-1"></a><span class="co">      # 80:80 </span></span>
<span id="cb10-14"><a href="#cb10-14" aria-hidden="true" tabindex="-1"></a><span class="at">      </span><span class="kw">-</span><span class="at"> 8080:8080 </span></span>
<span id="cb10-15"><a href="#cb10-15" aria-hidden="true" tabindex="-1"></a><span class="co">      # 8443:8443 </span></span>
<span id="cb10-16"><a href="#cb10-16" aria-hidden="true" tabindex="-1"></a><span class="co">    # security_opt: [&quot;label:disable&quot;] </span></span>
<span id="cb10-17"><a href="#cb10-17" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">environment</span><span class="kw">:</span><span class="co"> # Is needed when using any of the options below</span></span>
<span id="cb10-18"><a href="#cb10-18" aria-hidden="true" tabindex="-1"></a><span class="co">      # AIO_DISABLE_BACKUP_SECTION: false </span></span>
<span id="cb10-19"><a href="#cb10-19" aria-hidden="true" tabindex="-1"></a><span class="at">      </span><span class="fu">APACHE_PORT</span><span class="kw">:</span><span class="at"> </span><span class="dv">11000</span><span class="at"> </span></span>
<span id="cb10-20"><a href="#cb10-20" aria-hidden="true" tabindex="-1"></a><span class="at">      </span><span class="fu">APACHE_IP_BINDING</span><span class="kw">:</span><span class="at"> </span><span class="fl">127.0.0.1</span><span class="at"> </span></span>
<span id="cb10-21"><a href="#cb10-21" aria-hidden="true" tabindex="-1"></a><span class="co">      # APACHE_ADDITIONAL_NETWORK: frontend_net # (Optional) </span></span>
<span id="cb10-22"><a href="#cb10-22" aria-hidden="true" tabindex="-1"></a><span class="co">      # BORG_RETENTION_POLICY: --keep-within=7d --keep-weekly=4 --keep-monthly=6 </span></span>
<span id="cb10-23"><a href="#cb10-23" aria-hidden="true" tabindex="-1"></a><span class="co">      # COLLABORA_SECCOMP_DISABLED: false </span></span>
<span id="cb10-24"><a href="#cb10-24" aria-hidden="true" tabindex="-1"></a><span class="co">      # FULLTEXTSEARCH_JAVA_OPTIONS: &quot;-Xms1024M -Xmx1024M&quot; </span></span>
<span id="cb10-25"><a href="#cb10-25" aria-hidden="true" tabindex="-1"></a><span class="at">      </span><span class="fu">NEXTCLOUD_DATADIR</span><span class="kw">:</span><span class="at"> /homelab/data/nextcloud/datadir</span><span class="co"> # Allows to set the host directory for Nextcloud&#39;s datadir. ⚠️⚠️⚠️ Warning: do not set or adjust this value after the initial Nextcloud installation is done!</span></span>
<span id="cb10-26"><a href="#cb10-26" aria-hidden="true" tabindex="-1"></a><span class="co">      # NEXTCLOUD_MOUNT: /mnt/</span></span>
<span id="cb10-27"><a href="#cb10-27" aria-hidden="true" tabindex="-1"></a><span class="co">      # NEXTCLOUD_UPLOAD_LIMIT: 16G # Can be adjusted if you need more.</span></span>
<span id="cb10-28"><a href="#cb10-28" aria-hidden="true" tabindex="-1"></a><span class="co">      # NEXTCLOUD_MAX_TIME: 3600 # Can be adjusted if you need more. </span></span>
<span id="cb10-29"><a href="#cb10-29" aria-hidden="true" tabindex="-1"></a><span class="co">      # NEXTCLOUD_MEMORY_LIMIT: 512M # Can be adjusted if you need more.</span></span>
<span id="cb10-30"><a href="#cb10-30" aria-hidden="true" tabindex="-1"></a><span class="co">      # NEXTCLOUD_TRUSTED_CACERTS_DIR: /path/to/my/cacerts </span></span>
<span id="cb10-31"><a href="#cb10-31" aria-hidden="true" tabindex="-1"></a><span class="co">      # NEXTCLOUD_STARTUP_APPS: deck twofactor_totp tasks calendar contacts notes</span></span>
<span id="cb10-32"><a href="#cb10-32" aria-hidden="true" tabindex="-1"></a><span class="co">      # NEXTCLOUD_ADDITIONAL_APKS: imagemagick </span></span>
<span id="cb10-33"><a href="#cb10-33" aria-hidden="true" tabindex="-1"></a><span class="co">      # NEXTCLOUD_ADDITIONAL_PHP_EXTENSIONS: imagick </span></span>
<span id="cb10-34"><a href="#cb10-34" aria-hidden="true" tabindex="-1"></a><span class="at">      </span><span class="fu">NEXTCLOUD_ENABLE_DRI_DEVICE</span><span class="kw">:</span><span class="at"> </span><span class="ch">true</span><span class="co"> # This allows to enable the /dev/dri device for containers that profit from it. ⚠️⚠️⚠️ Warning: this only works if the &#39;/dev/dri&#39; device is present on the host!</span></span>
<span id="cb10-35"><a href="#cb10-35" aria-hidden="true" tabindex="-1"></a><span class="co">      # NEXTCLOUD_ENABLE_NVIDIA_GPU: true </span></span>
<span id="cb10-36"><a href="#cb10-36" aria-hidden="true" tabindex="-1"></a><span class="co">      # NEXTCLOUD_KEEP_DISABLED_APPS: false </span></span>
<span id="cb10-37"><a href="#cb10-37" aria-hidden="true" tabindex="-1"></a><span class="at">      </span><span class="fu">SKIP_DOMAIN_VALIDATION</span><span class="kw">:</span><span class="at"> </span><span class="ch">True</span><span class="at"> </span></span>
<span id="cb10-38"><a href="#cb10-38" aria-hidden="true" tabindex="-1"></a><span class="co">      # TALK_PORT: 3478 </span></span>
<span id="cb10-39"><a href="#cb10-39" aria-hidden="true" tabindex="-1"></a><span class="co">      # WATCHTOWER_DOCKER_SOCKET_PATH: /var/run/docker.sock </span></span>
<span id="cb10-40"><a href="#cb10-40" aria-hidden="true" tabindex="-1"></a></span>
<span id="cb10-41"><a href="#cb10-41" aria-hidden="true" tabindex="-1"></a><span class="fu">volumes</span><span class="kw">:</span></span>
<span id="cb10-42"><a href="#cb10-42" aria-hidden="true" tabindex="-1"></a><span class="at">  </span><span class="fu">nextcloud_aio_mastercontainer</span><span class="kw">:</span></span>
<span id="cb10-43"><a href="#cb10-43" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">name</span><span class="kw">:</span><span class="at"> nextcloud_aio_mastercontainer</span><span class="co"> # This line is not allowed to be changed as otherwise the built-in backup solution will not work</span></span>
<span id="cb10-44"><a href="#cb10-44" aria-hidden="true" tabindex="-1"></a><span class="co">  # caddy_certs:</span></span>
<span id="cb10-45"><a href="#cb10-45" aria-hidden="true" tabindex="-1"></a><span class="co">  # caddy_config:</span></span>
<span id="cb10-46"><a href="#cb10-46" aria-hidden="true" tabindex="-1"></a><span class="co">  # caddy_data:</span></span>
<span id="cb10-47"><a href="#cb10-47" aria-hidden="true" tabindex="-1"></a><span class="co">  # caddy_sites:</span></span></code></pre></div>
<h3 id="plex-media-server">Plex Media Server:</h3>
<h4 id="overview-7">Overview:</h4>
<p><a href="https://github.com/plexinc/pms-docker">Plex Media Server (PMS)</a> is an application that organizes video, music, and photos from personal media libraries and allows them to be streamed to other devices logged in under the same user. Most commonly, Plex is used to create a locally hosted streaming platform for downloaded movies and tv-shows that can be accessed anywhere.</p>
<h4 id="configuration-3">Configuration:</h4>
<p>In order to setup PMS, you're going to need the UID and GID for 'appuser' and the correct timezone. In addition to that, two folders have to be created to store the movies and tv-shows we want to be able to stream. After creating those folders, change the movies and tv pathways to accurately reflect the shared folder locations. The only port required for this application is port 32400. By running the host network mode for this application, no ports have to be defined in the compose file.</p>
<h4 id="shared-folders-2">Shared Folders:</h4>
<div class="sourceCode" id="cb11"><pre class="sourceCode bash"><code class="sourceCode bash"><span id="cb11-1"><a href="#cb11-1" aria-hidden="true" tabindex="-1"></a><span class="ex">/data/plex/tvseries</span></span>
<span id="cb11-2"><a href="#cb11-2" aria-hidden="true" tabindex="-1"></a><span class="ex">/data/plex/movies</span></span></code></pre></div>
<h4 id="compose-file-5">Compose File:</h4>
<div class="sourceCode" id="cb12"><pre class="sourceCode yaml"><code class="sourceCode yaml"><span id="cb12-1"><a href="#cb12-1" aria-hidden="true" tabindex="-1"></a><span class="pp">---</span></span>
<span id="cb12-2"><a href="#cb12-2" aria-hidden="true" tabindex="-1"></a><span class="fu">services</span><span class="kw">:</span></span>
<span id="cb12-3"><a href="#cb12-3" aria-hidden="true" tabindex="-1"></a><span class="at">  </span><span class="fu">plex</span><span class="kw">:</span></span>
<span id="cb12-4"><a href="#cb12-4" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">image</span><span class="kw">:</span><span class="at"> lscr.io/linuxserver/plex:latest</span></span>
<span id="cb12-5"><a href="#cb12-5" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">container_name</span><span class="kw">:</span><span class="at"> plex</span></span>
<span id="cb12-6"><a href="#cb12-6" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">network_mode</span><span class="kw">:</span><span class="at"> host</span></span>
<span id="cb12-7"><a href="#cb12-7" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">environment</span><span class="kw">:</span></span>
<span id="cb12-8"><a href="#cb12-8" aria-hidden="true" tabindex="-1"></a><span class="at">      </span><span class="kw">-</span><span class="at"> PUID=UserID</span><span class="co">  # Change this to your user PUID</span></span>
<span id="cb12-9"><a href="#cb12-9" aria-hidden="true" tabindex="-1"></a><span class="at">      </span><span class="kw">-</span><span class="at"> PGID=GroupID</span><span class="co">   # Change this to your user PGID</span></span>
<span id="cb12-10"><a href="#cb12-10" aria-hidden="true" tabindex="-1"></a><span class="at">      </span><span class="kw">-</span><span class="at"> TZ=Country/City</span><span class="co"> # Change timezone if different</span></span>
<span id="cb12-11"><a href="#cb12-11" aria-hidden="true" tabindex="-1"></a><span class="at">      </span><span class="kw">-</span><span class="at"> VERSION=docker</span></span>
<span id="cb12-12"><a href="#cb12-12" aria-hidden="true" tabindex="-1"></a><span class="at">      </span><span class="kw">-</span><span class="at"> PLEX_CLAIM=</span><span class="co"> #optional</span></span>
<span id="cb12-13"><a href="#cb12-13" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">volumes</span><span class="kw">:</span></span>
<span id="cb12-14"><a href="#cb12-14" aria-hidden="true" tabindex="-1"></a><span class="at">      </span><span class="kw">-</span><span class="at"> CHANGE_TO_COMPOSE_DATA_PATH/plex/config:/config</span><span class="co"> # Change this path</span></span>
<span id="cb12-15"><a href="#cb12-15" aria-hidden="true" tabindex="-1"></a><span class="at">      </span><span class="kw">-</span><span class="at"> /path/to/plex/tvseries:/tv</span><span class="co"> # Change this path</span></span>
<span id="cb12-16"><a href="#cb12-16" aria-hidden="true" tabindex="-1"></a><span class="at">      </span><span class="kw">-</span><span class="at"> /path/to/plex/movies:/movies</span><span class="co"> # Change this path</span></span>
<span id="cb12-17"><a href="#cb12-17" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">restart</span><span class="kw">:</span><span class="at"> unless-stopped</span></span></code></pre></div>
<h3 id="qbittorrent">qBittorrent:</h3>
<h4 id="overview-8">Overview:</h4>
<p><a href="https://github.com/linuxserver/docker-qbittorrent">qBittorrent</a> is a BitTorrent client that uses the P2P protocol to download and share files. It is frequently referred to as the open-source alternative to µTorrent. This type of data sharing protocol breaks large files into small pieces and relies on users to 'seed' the files after downloading. This makes it so that instead of downloading a file from a single server, users essentially download pieces of a file directly to and from each other.</p>
<h4 id="configuration-4">Configuration:</h4>
<p>The compose file for this application is different than the others, as there are two services running in this one container. In this container, we are running Gluetun and qBittorrent together.</p>
<table>
<thead>
<tr class="header">
<th style="text-align: center;">VPN Configuration Options &amp; Other Info</th>
<th style="text-align: center;">&lt;</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: center;">Supported VPN Providers<br>(OpenVPN)<br></td>
<td style="text-align: center;">AirVPN, Cyberghost, ExpressVPN, FastestVPN, Giganews, HideMyAss, IPVanish, IVPN, Mullvad, NordVPN, Perfect Privacy, Privado, Private Internet Access, PrivateVPN, ProtonVPN, PureVPN, SlickVPN, Surfshark, TorGuard, VPNSecure.me, VPNUnlimited, Vyprvpn, WeVPN, Windscribe</td>
</tr>
<tr class="even">
<td style="text-align: center;">Supported VPN Providers<br>(WireGuard)</td>
<td style="text-align: center;">AirVPN, FastestVPN, Ivpn, Mullvad, NordVPN, Perfect Privacy, ProtonVPN, Surfshark, Windscribe</td>
</tr>
<tr class="odd">
<td style="text-align: center;">Ports</td>
<td style="text-align: center;">Make sure the 'qB WebUI' port for Gluetun is the same as the 'WEBUI_PORT' for qBittorrent</td>
</tr>
<tr class="even">
<td style="text-align: center;">WIREGUARD_PRIVATE_KEY</td>
<td style="text-align: center;">Obtained from setting up a WireGuard Configuration through VPN provider</td>
</tr>
<tr class="odd">
<td style="text-align: center;">WIREGUARD_ADDRESSES</td>
<td style="text-align: center;">Obtained from setting up a WireGuard Configuration through VPN provider</td>
</tr>
<tr class="even">
<td style="text-align: center;">SERVER_CITIES</td>
<td style="text-align: center;">Can change to SERVER_COUNTRIES or SERVER_HOSTNAMES<br>Full list depends on VPN provider</td>
</tr>
<tr class="odd">
<td style="text-align: center;">LocalIP</td>
<td style="text-align: center;">LocalIP is a modified version of the ROCK 5C device IP. <br>Ex) If my IP was 192.168.10.XXX then the LocalIP would be 192.168.10.0/24</td>
</tr>
</tbody>
</table>
<h4 id="shared-folders-3">Shared Folders:</h4>
<div class="sourceCode" id="cb13"><pre class="sourceCode bash"><code class="sourceCode bash"><span id="cb13-1"><a href="#cb13-1" aria-hidden="true" tabindex="-1"></a><span class="ex">/data/qbittorrent</span></span>
<span id="cb13-2"><a href="#cb13-2" aria-hidden="true" tabindex="-1"></a><span class="ex">/data/qbittorrent/appdata</span></span>
<span id="cb13-3"><a href="#cb13-3" aria-hidden="true" tabindex="-1"></a><span class="ex">/data/qbittorrent/downloads</span></span>
<span id="cb13-4"><a href="#cb13-4" aria-hidden="true" tabindex="-1"></a></span>
<span id="cb13-5"><a href="#cb13-5" aria-hidden="true" tabindex="-1"></a><span class="ex">/data/gluetun</span></span></code></pre></div>
<h4 id="compose-file-6">Compose File:</h4>
<div class="sourceCode" id="cb14"><pre class="sourceCode yaml"><code class="sourceCode yaml"><span id="cb14-1"><a href="#cb14-1" aria-hidden="true" tabindex="-1"></a><span class="fu">networks</span><span class="kw">:</span></span>
<span id="cb14-2"><a href="#cb14-2" aria-hidden="true" tabindex="-1"></a><span class="at">  </span><span class="fu">NetworkName</span><span class="kw">:</span></span>
<span id="cb14-3"><a href="#cb14-3" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">external</span><span class="kw">:</span><span class="at"> </span><span class="ch">true</span></span>
<span id="cb14-4"><a href="#cb14-4" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">name</span><span class="kw">:</span><span class="at"> NetworkName</span></span>
<span id="cb14-5"><a href="#cb14-5" aria-hidden="true" tabindex="-1"></a><span class="at">    </span></span>
<span id="cb14-6"><a href="#cb14-6" aria-hidden="true" tabindex="-1"></a><span class="fu">services</span><span class="kw">:</span></span>
<span id="cb14-7"><a href="#cb14-7" aria-hidden="true" tabindex="-1"></a><span class="at">  </span><span class="fu">gluetun</span><span class="kw">:</span></span>
<span id="cb14-8"><a href="#cb14-8" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">image</span><span class="kw">:</span><span class="at"> qmcgaw/gluetun</span></span>
<span id="cb14-9"><a href="#cb14-9" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">container_name</span><span class="kw">:</span><span class="at"> gluetun</span></span>
<span id="cb14-10"><a href="#cb14-10" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">cap_add</span><span class="kw">:</span></span>
<span id="cb14-11"><a href="#cb14-11" aria-hidden="true" tabindex="-1"></a><span class="at">      </span><span class="kw">-</span><span class="at"> NET_ADMIN</span></span>
<span id="cb14-12"><a href="#cb14-12" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">environment</span><span class="kw">:</span></span>
<span id="cb14-13"><a href="#cb14-13" aria-hidden="true" tabindex="-1"></a><span class="co">      # --- choose ONE stack (WireGuard OR OpenVPN) and fill in your provider details ---</span></span>
<span id="cb14-14"><a href="#cb14-14" aria-hidden="true" tabindex="-1"></a></span>
<span id="cb14-15"><a href="#cb14-15" aria-hidden="true" tabindex="-1"></a><span class="co">      # Example (WireGuard) – fast &amp; preferred:</span></span>
<span id="cb14-16"><a href="#cb14-16" aria-hidden="true" tabindex="-1"></a><span class="at">      </span><span class="kw">-</span><span class="at"> VPN_SERVICE_PROVIDER=protonvpn</span><span class="co">        # mullvad | protonvpn | pia | nordvpn | ...</span></span>
<span id="cb14-17"><a href="#cb14-17" aria-hidden="true" tabindex="-1"></a><span class="at">      </span><span class="kw">-</span><span class="at"> VPN_TYPE=wireguard</span></span>
<span id="cb14-18"><a href="#cb14-18" aria-hidden="true" tabindex="-1"></a><span class="at">      </span><span class="kw">-</span><span class="at"> WIREGUARD_PRIVATE_KEY=PrivateKey</span></span>
<span id="cb14-19"><a href="#cb14-19" aria-hidden="true" tabindex="-1"></a><span class="at">      </span><span class="kw">-</span><span class="at"> WIREGUARD_ADDRESSES=WgAddress</span><span class="co">    # provider gives you this or it&#39;s in the wg config</span></span>
<span id="cb14-20"><a href="#cb14-20" aria-hidden="true" tabindex="-1"></a><span class="at">      </span><span class="kw">-</span><span class="at"> SERVER_CITIES=CityName</span><span class="co">        # or SERVER_CITIES=..., SERVER_HOSTNAMES=...</span></span>
<span id="cb14-21"><a href="#cb14-21" aria-hidden="true" tabindex="-1"></a></span>
<span id="cb14-22"><a href="#cb14-22" aria-hidden="true" tabindex="-1"></a><span class="co">      # Example (OpenVPN) – if you use OVPN creds instead:</span></span>
<span id="cb14-23"><a href="#cb14-23" aria-hidden="true" tabindex="-1"></a><span class="co">      # - VPN_SERVICE_PROVIDER=protonvpn</span></span>
<span id="cb14-24"><a href="#cb14-24" aria-hidden="true" tabindex="-1"></a><span class="co">      # - VPN_TYPE=openvpn</span></span>
<span id="cb14-25"><a href="#cb14-25" aria-hidden="true" tabindex="-1"></a><span class="co">      # - OPENVPN_USER=yourusername</span></span>
<span id="cb14-26"><a href="#cb14-26" aria-hidden="true" tabindex="-1"></a><span class="co">      # - OPENVPN_PASSWORD=yourpassword</span></span>
<span id="cb14-27"><a href="#cb14-27" aria-hidden="true" tabindex="-1"></a><span class="co">      # - SERVER_COUNTRIES=United States</span></span>
<span id="cb14-28"><a href="#cb14-28" aria-hidden="true" tabindex="-1"></a></span>
<span id="cb14-29"><a href="#cb14-29" aria-hidden="true" tabindex="-1"></a><span class="co">      # General:</span></span>
<span id="cb14-30"><a href="#cb14-30" aria-hidden="true" tabindex="-1"></a><span class="at">      </span><span class="kw">-</span><span class="at"> TZ=Country/City</span></span>
<span id="cb14-31"><a href="#cb14-31" aria-hidden="true" tabindex="-1"></a><span class="at">      </span><span class="kw">-</span><span class="at"> FIREWALL_OUTBOUND_SUBNETS=LocalIP</span><span class="co">  # so you can reach WebUI from your LAN</span></span>
<span id="cb14-32"><a href="#cb14-32" aria-hidden="true" tabindex="-1"></a><span class="at">      </span><span class="kw">-</span><span class="at"> FIREWALL_INPUT_PORTS=8585</span></span>
<span id="cb14-33"><a href="#cb14-33" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">volumes</span><span class="kw">:</span></span>
<span id="cb14-34"><a href="#cb14-34" aria-hidden="true" tabindex="-1"></a><span class="at">      </span><span class="kw">-</span><span class="at"> /path/to/gluetun:/gluetun</span></span>
<span id="cb14-35"><a href="#cb14-35" aria-hidden="true" tabindex="-1"></a><span class="co">    # Expose qBittorrent&#39;s ports HERE (since qB routes through gluetun)</span></span>
<span id="cb14-36"><a href="#cb14-36" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">networks</span><span class="kw">:</span><span class="at"> </span><span class="kw">[</span><span class="at">NetworkName</span><span class="kw">]</span></span>
<span id="cb14-37"><a href="#cb14-37" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">ports</span><span class="kw">:</span></span>
<span id="cb14-38"><a href="#cb14-38" aria-hidden="true" tabindex="-1"></a><span class="at">      </span><span class="kw">-</span><span class="at"> </span><span class="st">&quot;8585:8585&quot;</span><span class="co">       # qBittorrent WebUI</span></span>
<span id="cb14-39"><a href="#cb14-39" aria-hidden="true" tabindex="-1"></a><span class="at">      </span><span class="kw">-</span><span class="at"> </span><span class="st">&quot;6881:6881&quot;</span><span class="co">       # TCP</span></span>
<span id="cb14-40"><a href="#cb14-40" aria-hidden="true" tabindex="-1"></a><span class="at">      </span><span class="kw">-</span><span class="at"> </span><span class="st">&quot;6881:6881/udp&quot;</span><span class="co">   # UDP</span></span>
<span id="cb14-41"><a href="#cb14-41" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">restart</span><span class="kw">:</span><span class="at"> unless-stopped</span></span>
<span id="cb14-42"><a href="#cb14-42" aria-hidden="true" tabindex="-1"></a></span>
<span id="cb14-43"><a href="#cb14-43" aria-hidden="true" tabindex="-1"></a><span class="at">  </span><span class="fu">qbittorrent</span><span class="kw">:</span></span>
<span id="cb14-44"><a href="#cb14-44" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">image</span><span class="kw">:</span><span class="at"> lscr.io/linuxserver/qbittorrent:latest</span></span>
<span id="cb14-45"><a href="#cb14-45" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">container_name</span><span class="kw">:</span><span class="at"> qbittorrent</span></span>
<span id="cb14-46"><a href="#cb14-46" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">network_mode</span><span class="kw">:</span><span class="at"> </span><span class="st">&quot;service:gluetun&quot;</span><span class="co">   # route all qB traffic through gluetun</span></span>
<span id="cb14-47"><a href="#cb14-47" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">depends_on</span><span class="kw">:</span></span>
<span id="cb14-48"><a href="#cb14-48" aria-hidden="true" tabindex="-1"></a><span class="at">      </span><span class="kw">-</span><span class="at"> gluetun</span></span>
<span id="cb14-49"><a href="#cb14-49" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">environment</span><span class="kw">:</span></span>
<span id="cb14-50"><a href="#cb14-50" aria-hidden="true" tabindex="-1"></a><span class="at">      </span><span class="kw">-</span><span class="at"> PUID=UserID</span></span>
<span id="cb14-51"><a href="#cb14-51" aria-hidden="true" tabindex="-1"></a><span class="at">      </span><span class="kw">-</span><span class="at"> PGID=GroupID</span></span>
<span id="cb14-52"><a href="#cb14-52" aria-hidden="true" tabindex="-1"></a><span class="at">      </span><span class="kw">-</span><span class="at"> TZ=Country/City</span></span>
<span id="cb14-53"><a href="#cb14-53" aria-hidden="true" tabindex="-1"></a><span class="at">      </span><span class="kw">-</span><span class="at"> WEBUI_PORT=8585</span><span class="co">              # make qB listen on 8585 to match the published port</span></span>
<span id="cb14-54"><a href="#cb14-54" aria-hidden="true" tabindex="-1"></a><span class="co">      # no TORRENTING_PORT env — set the listening port to 6881 inside qB UI (or via config)</span></span>
<span id="cb14-55"><a href="#cb14-55" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">volumes</span><span class="kw">:</span></span>
<span id="cb14-56"><a href="#cb14-56" aria-hidden="true" tabindex="-1"></a><span class="at">      </span><span class="kw">-</span><span class="at"> /path/to/qbittorrent/appdata:/config</span></span>
<span id="cb14-57"><a href="#cb14-57" aria-hidden="true" tabindex="-1"></a><span class="at">      </span><span class="kw">-</span><span class="at"> /path/to/qbittorrent/downloads:/downloads</span></span>
<span id="cb14-58"><a href="#cb14-58" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">restart</span><span class="kw">:</span><span class="at"> unless-stopped</span></span></code></pre></div>
<h3 id="flaresolverr">Flaresolverr:</h3>
<h4 id="overview-9">Overview:</h4>
<p><a href="https://github.com/FlareSolverr/FlareSolverr">Flaresolverr</a> is a proxy server that is used to bypass Cloudflare and DDoS-GUARD protection on websites. We will be running this so that we can access indexers like x1337x with other applications like Prowlarr, Radarr, and Sonarr.</p>
<h4 id="configuration-5">Configuration:</h4>
<p>Besides creating the two shared folders, there's only one other thing that has to be done to finish setting up this application. In order to allow Flaresolverr to interact with Prowlarr, a network bridge has to be created. In Open Media Vault, go to 'Services &gt; Compose &gt; Networks' and add a new network. Only input the name and make sure that the driver is set to 'bridge'.</p>
<h4 id="shared-folders-4">Shared Folders:</h4>
<div class="sourceCode" id="cb15"><pre class="sourceCode bash"><code class="sourceCode bash"><span id="cb15-1"><a href="#cb15-1" aria-hidden="true" tabindex="-1"></a><span class="ex">/data/flaresolverr</span></span>
<span id="cb15-2"><a href="#cb15-2" aria-hidden="true" tabindex="-1"></a><span class="ex">/data/flaresolverr/config</span></span></code></pre></div>
<h4 id="compose-file-7">Compose File:</h4>
<div class="sourceCode" id="cb16"><pre class="sourceCode yaml"><code class="sourceCode yaml"><span id="cb16-1"><a href="#cb16-1" aria-hidden="true" tabindex="-1"></a><span class="pp">---</span></span>
<span id="cb16-2"><a href="#cb16-2" aria-hidden="true" tabindex="-1"></a><span class="fu">networks</span><span class="kw">:</span></span>
<span id="cb16-3"><a href="#cb16-3" aria-hidden="true" tabindex="-1"></a><span class="at">  </span><span class="fu">NetworkName</span><span class="kw">:</span></span>
<span id="cb16-4"><a href="#cb16-4" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">external</span><span class="kw">:</span><span class="at"> </span><span class="ch">true</span></span>
<span id="cb16-5"><a href="#cb16-5" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">name</span><span class="kw">:</span><span class="at"> NetworkName</span></span>
<span id="cb16-6"><a href="#cb16-6" aria-hidden="true" tabindex="-1"></a><span class="at">    </span></span>
<span id="cb16-7"><a href="#cb16-7" aria-hidden="true" tabindex="-1"></a><span class="fu">services</span><span class="kw">:</span></span>
<span id="cb16-8"><a href="#cb16-8" aria-hidden="true" tabindex="-1"></a><span class="at">  </span><span class="fu">flaresolverr</span><span class="kw">:</span></span>
<span id="cb16-9"><a href="#cb16-9" aria-hidden="true" tabindex="-1"></a><span class="co">    # DockerHub mirror flaresolverr/flaresolverr:latest</span></span>
<span id="cb16-10"><a href="#cb16-10" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">image</span><span class="kw">:</span><span class="at"> ghcr.io/flaresolverr/flaresolverr:latest</span></span>
<span id="cb16-11"><a href="#cb16-11" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">container_name</span><span class="kw">:</span><span class="at"> flaresolverr</span></span>
<span id="cb16-12"><a href="#cb16-12" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">environment</span><span class="kw">:</span></span>
<span id="cb16-13"><a href="#cb16-13" aria-hidden="true" tabindex="-1"></a><span class="at">      </span><span class="kw">-</span><span class="at"> LOG_LEVEL=${LOG_LEVEL:-info}</span></span>
<span id="cb16-14"><a href="#cb16-14" aria-hidden="true" tabindex="-1"></a><span class="at">      </span><span class="kw">-</span><span class="at"> LOG_FILE=${LOG_FILE:-none}</span></span>
<span id="cb16-15"><a href="#cb16-15" aria-hidden="true" tabindex="-1"></a><span class="at">      </span><span class="kw">-</span><span class="at"> LOG_HTML=${LOG_HTML:-false}</span></span>
<span id="cb16-16"><a href="#cb16-16" aria-hidden="true" tabindex="-1"></a><span class="at">      </span><span class="kw">-</span><span class="at"> CAPTCHA_SOLVER=${CAPTCHA_SOLVER:-none}</span></span>
<span id="cb16-17"><a href="#cb16-17" aria-hidden="true" tabindex="-1"></a><span class="at">      </span><span class="kw">-</span><span class="at"> TZ=Country/City</span></span>
<span id="cb16-18"><a href="#cb16-18" aria-hidden="true" tabindex="-1"></a><span class="at">      </span><span class="kw">-</span><span class="at"> TEST_URL=https://www.google.com</span></span>
<span id="cb16-19"><a href="#cb16-19" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">networks</span><span class="kw">:</span><span class="at"> </span><span class="kw">[</span><span class="at">NetworkName</span><span class="kw">]</span></span>
<span id="cb16-20"><a href="#cb16-20" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">ports</span><span class="kw">:</span></span>
<span id="cb16-21"><a href="#cb16-21" aria-hidden="true" tabindex="-1"></a><span class="at">      </span><span class="kw">-</span><span class="at"> </span><span class="st">&quot;${PORT:-8191}:8191&quot;</span></span>
<span id="cb16-22"><a href="#cb16-22" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">volumes</span><span class="kw">:</span></span>
<span id="cb16-23"><a href="#cb16-23" aria-hidden="true" tabindex="-1"></a><span class="at">      </span><span class="kw">-</span><span class="at"> /path/to/flaresolverr/config:/config</span></span>
<span id="cb16-24"><a href="#cb16-24" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">restart</span><span class="kw">:</span><span class="at"> unless-stopped</span></span></code></pre></div>
<h3 id="prowlarr">Prowlarr:</h3>
<h4 id="overview-10">Overview:</h4>
<p><a href="https://github.com/linuxserver/docker-prowlarr">Prowlarr</a> is a manager for indexers that integrates seamlessly with Sonarr, Radarr, and other -arr applications. By using this service, we can effectively have complete management over all indexers for other applications, bypassing that 'per app' setup.</p>
<h4 id="configuration-6">Configuration:</h4>
<p>To setup this application, you will need the UID and GID for 'appuser'. After getting Prowlarr up and running, login to the service. After logging in, make sure to do the following:</p>
<ul>
<li>Go to 'Settings &gt; Indexers' and add Flaresolverr. (Host: http://flaresolverr:8191)</li>
<li>Add the tags 'movies', 'tv', and 'other' to the indexer proxy.</li>
<li>Then go to 'Indexers' and hit 'Add Indexer'. Add whatever indexers you want. At the time of making this I use the following:
<ul>
<li>1337x</li>
<li>Knaben</li>
<li>The Pirate Bay</li>
<li>Uindex</li>
<li>YTS</li>
</ul></li>
<li>Then, go to 'Settings &gt; Download Clients' and add qBittorent. For 'Host' use the IP of the ROCK 5C and set the port to be the same as what is used by qBittorent. Type in the username and password used to login to qB and then add two mapped categories, one for movies and one for tv-shows.</li>
<li>Before continuing the setup for Prowlarr, make sure Radarr and Sonarr are installed.</li>
</ul>
<h4 id="shared-folders-5">Shared Folders:</h4>
<div class="sourceCode" id="cb17"><pre class="sourceCode bash"><code class="sourceCode bash"><span id="cb17-1"><a href="#cb17-1" aria-hidden="true" tabindex="-1"></a><span class="ex">/data/prowlarr</span></span>
<span id="cb17-2"><a href="#cb17-2" aria-hidden="true" tabindex="-1"></a><span class="ex">/data/prowlarr/config</span></span></code></pre></div>
<h4 id="compose-file-8">Compose File:</h4>
<div class="sourceCode" id="cb18"><pre class="sourceCode yaml"><code class="sourceCode yaml"><span id="cb18-1"><a href="#cb18-1" aria-hidden="true" tabindex="-1"></a><span class="pp">---</span></span>
<span id="cb18-2"><a href="#cb18-2" aria-hidden="true" tabindex="-1"></a><span class="fu">networks</span><span class="kw">:</span></span>
<span id="cb18-3"><a href="#cb18-3" aria-hidden="true" tabindex="-1"></a><span class="at">  </span><span class="fu">NetworkName</span><span class="kw">:</span></span>
<span id="cb18-4"><a href="#cb18-4" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">external</span><span class="kw">:</span><span class="at"> </span><span class="ch">true</span></span>
<span id="cb18-5"><a href="#cb18-5" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">name</span><span class="kw">:</span><span class="at"> NetworkName</span></span>
<span id="cb18-6"><a href="#cb18-6" aria-hidden="true" tabindex="-1"></a><span class="at">    </span></span>
<span id="cb18-7"><a href="#cb18-7" aria-hidden="true" tabindex="-1"></a><span class="fu">services</span><span class="kw">:</span></span>
<span id="cb18-8"><a href="#cb18-8" aria-hidden="true" tabindex="-1"></a><span class="at">  </span><span class="fu">prowlarr</span><span class="kw">:</span></span>
<span id="cb18-9"><a href="#cb18-9" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">image</span><span class="kw">:</span><span class="at"> lscr.io/linuxserver/prowlarr:latest</span></span>
<span id="cb18-10"><a href="#cb18-10" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">container_name</span><span class="kw">:</span><span class="at"> prowlarr</span></span>
<span id="cb18-11"><a href="#cb18-11" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">environment</span><span class="kw">:</span></span>
<span id="cb18-12"><a href="#cb18-12" aria-hidden="true" tabindex="-1"></a><span class="at">      </span><span class="kw">-</span><span class="at"> PUID=UserID</span></span>
<span id="cb18-13"><a href="#cb18-13" aria-hidden="true" tabindex="-1"></a><span class="at">      </span><span class="kw">-</span><span class="at"> PGID=GroupID</span></span>
<span id="cb18-14"><a href="#cb18-14" aria-hidden="true" tabindex="-1"></a><span class="at">      </span><span class="kw">-</span><span class="at"> TZ=Country/City</span></span>
<span id="cb18-15"><a href="#cb18-15" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">volumes</span><span class="kw">:</span></span>
<span id="cb18-16"><a href="#cb18-16" aria-hidden="true" tabindex="-1"></a><span class="at">      </span><span class="kw">-</span><span class="at"> /path/to/prowlarr/config:/config</span></span>
<span id="cb18-17"><a href="#cb18-17" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">networks</span><span class="kw">:</span><span class="at"> </span><span class="kw">[</span><span class="at">NetworkName</span><span class="kw">]</span></span>
<span id="cb18-18"><a href="#cb18-18" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">ports</span><span class="kw">:</span><span class="at"> </span><span class="kw">[</span><span class="st">&quot;9696:9696&quot;</span><span class="kw">]</span></span>
<span id="cb18-19"><a href="#cb18-19" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">restart</span><span class="kw">:</span><span class="at"> unless-stopped</span></span></code></pre></div>
<h3 id="radarr">Radarr:</h3>
<h4 id="overview-11">Overview:</h4>
<p><a href="https://github.com/Radarr/Radarr">Radarr</a> is a movie collection manager for Usenet and BitTorrent users. It can monitor the web for new movies and can interface directly with clients and indexers to obtain and manage them on a file level. A useful feature of this application is that it can 'upgrade' movies to a higher quality as long as you set its quality profile. This is the application used with this server to manage the Plex movie collection.</p>
<h4 id="configuration-7">Configuration:</h4>
<p>To setup Radarr, a few things have to be done. First, you have to grab the UID and GID from 'appuser' for the compose file. Then, create the required shared folders for the application. After that, fill in your network name in order to ensure that is has access to the qBittorrent client.</p>
<h4 id="shared-folders-6">Shared Folders:</h4>
<div class="sourceCode" id="cb19"><pre class="sourceCode bash"><code class="sourceCode bash"><span id="cb19-1"><a href="#cb19-1" aria-hidden="true" tabindex="-1"></a><span class="ex">/data/radarr</span></span>
<span id="cb19-2"><a href="#cb19-2" aria-hidden="true" tabindex="-1"></a><span class="ex">/data/radarr/config</span></span></code></pre></div>
<h4 id="compose-file-9">Compose File:</h4>
<div class="sourceCode" id="cb20"><pre class="sourceCode yaml"><code class="sourceCode yaml"><span id="cb20-1"><a href="#cb20-1" aria-hidden="true" tabindex="-1"></a><span class="pp">---</span></span>
<span id="cb20-2"><a href="#cb20-2" aria-hidden="true" tabindex="-1"></a><span class="fu">networks</span><span class="kw">:</span></span>
<span id="cb20-3"><a href="#cb20-3" aria-hidden="true" tabindex="-1"></a><span class="at">  </span><span class="fu">NetworkName</span><span class="kw">:</span></span>
<span id="cb20-4"><a href="#cb20-4" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">external</span><span class="kw">:</span><span class="at"> </span><span class="ch">true</span></span>
<span id="cb20-5"><a href="#cb20-5" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">name</span><span class="kw">:</span><span class="at"> NetworkName</span></span>
<span id="cb20-6"><a href="#cb20-6" aria-hidden="true" tabindex="-1"></a><span class="at">    </span></span>
<span id="cb20-7"><a href="#cb20-7" aria-hidden="true" tabindex="-1"></a><span class="fu">services</span><span class="kw">:</span></span>
<span id="cb20-8"><a href="#cb20-8" aria-hidden="true" tabindex="-1"></a><span class="at">  </span><span class="fu">radarr</span><span class="kw">:</span></span>
<span id="cb20-9"><a href="#cb20-9" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">image</span><span class="kw">:</span><span class="at"> lscr.io/linuxserver/radarr:latest</span></span>
<span id="cb20-10"><a href="#cb20-10" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">container_name</span><span class="kw">:</span><span class="at"> radarr</span></span>
<span id="cb20-11"><a href="#cb20-11" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">environment</span><span class="kw">:</span></span>
<span id="cb20-12"><a href="#cb20-12" aria-hidden="true" tabindex="-1"></a><span class="at">      </span><span class="kw">-</span><span class="at"> PUID=UserID</span></span>
<span id="cb20-13"><a href="#cb20-13" aria-hidden="true" tabindex="-1"></a><span class="at">      </span><span class="kw">-</span><span class="at"> PGID=GroupID</span></span>
<span id="cb20-14"><a href="#cb20-14" aria-hidden="true" tabindex="-1"></a><span class="at">      </span><span class="kw">-</span><span class="at"> TZ=Country/City</span></span>
<span id="cb20-15"><a href="#cb20-15" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">volumes</span><span class="kw">:</span></span>
<span id="cb20-16"><a href="#cb20-16" aria-hidden="true" tabindex="-1"></a><span class="at">      </span><span class="kw">-</span><span class="at"> /path/to/radarr/config:/config</span></span>
<span id="cb20-17"><a href="#cb20-17" aria-hidden="true" tabindex="-1"></a><span class="at">      </span><span class="kw">-</span><span class="at"> /path/to/plex/movies:/movies</span><span class="co"> #optional</span></span>
<span id="cb20-18"><a href="#cb20-18" aria-hidden="true" tabindex="-1"></a><span class="at">      </span><span class="kw">-</span><span class="at"> /path/to/qbittorrent/downloads:/downloads</span><span class="co"> #optional</span></span>
<span id="cb20-19"><a href="#cb20-19" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">networks</span><span class="kw">:</span><span class="at"> </span><span class="kw">[</span><span class="at">NetworkName</span><span class="kw">]</span></span>
<span id="cb20-20"><a href="#cb20-20" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">ports</span><span class="kw">:</span></span>
<span id="cb20-21"><a href="#cb20-21" aria-hidden="true" tabindex="-1"></a><span class="at">      </span><span class="kw">-</span><span class="at"> </span><span class="st">&quot;7878:7878&quot;</span></span>
<span id="cb20-22"><a href="#cb20-22" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">restart</span><span class="kw">:</span><span class="at"> unless-stopped</span></span></code></pre></div>
<h3 id="sonarr">Sonarr:</h3>
<h4 id="overview-12">Overview:</h4>
<p><a href="https://github.com/Sonarr/Sonarr">Sonarr</a>, like Radarr, is another manager that is used in this setup. It also monitors multiple RSS feeds to find new episodes of managed series. In terms of functionality, it's the same as Radarr except for TV shows instead of movies.</p>
<h4 id="configuration-8">Configuration:</h4>
<p>See configuration guide for <a href="#radarr">Radarr</a>.</p>
<h4 id="shared-folders-7">Shared Folders:</h4>
<div class="sourceCode" id="cb21"><pre class="sourceCode bash"><code class="sourceCode bash"><span id="cb21-1"><a href="#cb21-1" aria-hidden="true" tabindex="-1"></a><span class="ex">/data/sonarr</span></span>
<span id="cb21-2"><a href="#cb21-2" aria-hidden="true" tabindex="-1"></a><span class="ex">/data/sonarr/config</span></span></code></pre></div>
<h4 id="compose-file-10">Compose File:</h4>
<div class="sourceCode" id="cb22"><pre class="sourceCode yaml"><code class="sourceCode yaml"><span id="cb22-1"><a href="#cb22-1" aria-hidden="true" tabindex="-1"></a><span class="pp">---</span></span>
<span id="cb22-2"><a href="#cb22-2" aria-hidden="true" tabindex="-1"></a><span class="fu">networks</span><span class="kw">:</span></span>
<span id="cb22-3"><a href="#cb22-3" aria-hidden="true" tabindex="-1"></a><span class="at">  </span><span class="fu">NetworkName</span><span class="kw">:</span></span>
<span id="cb22-4"><a href="#cb22-4" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">external</span><span class="kw">:</span><span class="at"> </span><span class="ch">true</span></span>
<span id="cb22-5"><a href="#cb22-5" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">name</span><span class="kw">:</span><span class="at"> NetworkName</span></span>
<span id="cb22-6"><a href="#cb22-6" aria-hidden="true" tabindex="-1"></a><span class="at">    </span></span>
<span id="cb22-7"><a href="#cb22-7" aria-hidden="true" tabindex="-1"></a><span class="fu">services</span><span class="kw">:</span></span>
<span id="cb22-8"><a href="#cb22-8" aria-hidden="true" tabindex="-1"></a><span class="at">  </span><span class="fu">sonarr</span><span class="kw">:</span></span>
<span id="cb22-9"><a href="#cb22-9" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">image</span><span class="kw">:</span><span class="at"> lscr.io/linuxserver/sonarr:latest</span></span>
<span id="cb22-10"><a href="#cb22-10" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">container_name</span><span class="kw">:</span><span class="at"> sonarr</span></span>
<span id="cb22-11"><a href="#cb22-11" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">environment</span><span class="kw">:</span></span>
<span id="cb22-12"><a href="#cb22-12" aria-hidden="true" tabindex="-1"></a><span class="at">      </span><span class="kw">-</span><span class="at"> PUID=UserID</span></span>
<span id="cb22-13"><a href="#cb22-13" aria-hidden="true" tabindex="-1"></a><span class="at">      </span><span class="kw">-</span><span class="at"> PGID=GroupID</span></span>
<span id="cb22-14"><a href="#cb22-14" aria-hidden="true" tabindex="-1"></a><span class="at">      </span><span class="kw">-</span><span class="at"> TZ=Country/City</span></span>
<span id="cb22-15"><a href="#cb22-15" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">volumes</span><span class="kw">:</span></span>
<span id="cb22-16"><a href="#cb22-16" aria-hidden="true" tabindex="-1"></a><span class="at">      </span><span class="kw">-</span><span class="at"> /path/to/sonarr/config:/config</span></span>
<span id="cb22-17"><a href="#cb22-17" aria-hidden="true" tabindex="-1"></a><span class="at">      </span><span class="kw">-</span><span class="at"> /path/to/plex/tvseries:/tv</span><span class="co"> #optional</span></span>
<span id="cb22-18"><a href="#cb22-18" aria-hidden="true" tabindex="-1"></a><span class="at">      </span><span class="kw">-</span><span class="at"> /path/to/qbittorrent/downloads:/downloads</span><span class="co"> #optional</span></span>
<span id="cb22-19"><a href="#cb22-19" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">networks</span><span class="kw">:</span><span class="at"> </span><span class="kw">[</span><span class="at">NetworkName</span><span class="kw">]</span></span>
<span id="cb22-20"><a href="#cb22-20" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">ports</span><span class="kw">:</span></span>
<span id="cb22-21"><a href="#cb22-21" aria-hidden="true" tabindex="-1"></a><span class="at">      </span><span class="kw">-</span><span class="at"> </span><span class="st">&quot;8989:8989&quot;</span></span>
<span id="cb22-22"><a href="#cb22-22" aria-hidden="true" tabindex="-1"></a><span class="at">    </span><span class="fu">restart</span><span class="kw">:</span><span class="at"> unless-stopped</span></span></code></pre></div>
<h2 id="utilized-ports-external">Utilized Ports (External):</h2>
<table>
<thead>
<tr class="header">
<th style="text-align: center;">Port Layout</th>
<th style="text-align: center;">&lt;</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: center;"><strong>Service</strong></td>
<td style="text-align: center;"><strong>Port</strong></td>
</tr>
<tr class="even">
<td style="text-align: center;">Flaresolverr</td>
<td style="text-align: center;">8191</td>
</tr>
<tr class="odd">
<td style="text-align: center;">Gluetun - Torrent</td>
<td style="text-align: center;">6881</td>
</tr>
<tr class="even">
<td style="text-align: center;">Gluetun - qB</td>
<td style="text-align: center;">8585</td>
</tr>
<tr class="odd">
<td style="text-align: center;">Heimdall - https</td>
<td style="text-align: center;">8444</td>
</tr>
<tr class="even">
<td style="text-align: center;">Heimdall - http</td>
<td style="text-align: center;">8888</td>
</tr>
<tr class="odd">
<td style="text-align: center;">Nextcloud - AIO</td>
<td style="text-align: center;">8080</td>
</tr>
<tr class="even">
<td style="text-align: center;">Nextcloud - Apache</td>
<td style="text-align: center;">11000</td>
</tr>
<tr class="odd">
<td style="text-align: center;">Portainer</td>
<td style="text-align: center;">9000</td>
</tr>
<tr class="even">
<td style="text-align: center;">Prowlarr</td>
<td style="text-align: center;">9696</td>
</tr>
<tr class="odd">
<td style="text-align: center;">Radarr</td>
<td style="text-align: center;">7878</td>
</tr>
<tr class="even">
<td style="text-align: center;">Sonarr</td>
<td style="text-align: center;">8989</td>
</tr>
<tr class="odd">
<td style="text-align: center;">Uptime Kuma</td>
<td style="text-align: center;">3001</td>
</tr>
<tr class="even">
<td style="text-align: center;">Watchtower</td>
<td style="text-align: center;">7272</td>
</tr>
</tbody>
</table>

</body>
</html>
