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
        <td style="border:1px solid #ddd;padding:8px;text-align:center;">ROCK 5C Home Server</td>
        <td style="border:1px solid #ddd;padding:8px;text-align:center;">Lucas Costello</td>
        <td style="border:1px solid #ddd;padding:8px;text-align:center;">2025-10-7</td>
      </tr>
    </tbody>
  </table>
</div>

<!-- Operating System -->
<details open>
  <summary><h2 id="operating-system" style="display:inline;">Operating System</h2></summary>
  <div align="justify" style="margin-top:8px;">
    <p>
      Although the ROCK 5C is a fantastic SBC, its capability to run a server is far off from an actual home lab setup.
      Due to the limited resources available, choosing an OS that ambiently uses less resources is important. Most servers
      run off of a Linux distribution, and this one will too.
    </p>
    <p>
      In order to run <a href="https://www.openmediavault.org/">Open Media Vault</a> on the ROCK 5C, we need to download
      the CLI version of the Radxa OS. The download for this version of the operating system can be found
      <a href="https://docs.radxa.com/en/rock5/rock5c/download">here</a>, and the installation guide can be found below.
    </p>
    <details open>
      <summary><h3 id="installing-the-os">Installing the OS:</h3></summary>
      <ul>
        <li>After downloading the image for the OS, go to the download location and extract the compressed file.</li>
        <li>Download <a href="https://etcher.balena.io/">balenaEtcher</a>.</li>
        <li>Use the “Flash from file” option and select <code>rock-5c_bookworm_cli_b1.output.img</code> obtained after extraction.</li>
        <li>Select the microSD card as the target.</li>
        <li>Hit “Flash!” and wait for the process to be completed.</li>
        <li>Once finished, install the microSD card into its slot on the bottom of the ROCK 5C.</li>
      </ul>
    </details> 
    <details open>
      <summary><h3 id="first-boot-remote-connections">First Boot &amp; Remote Connections:</h3></summary>
      <p><strong>Important:</strong> <em>Radxa enables SSH automatically on first boot if no monitor is detected.</em></p>
      <ul>
        <li>Using an ethernet cable, connect the RJ45 port on the ROCK 5C to a LAN port on the router. Then, give the device power by plugging the 12V barrel jack connector into the plug on the Penta SATA HAT.</li>
        <li>After 3–5 minutes, access your router’s admin panel and look for the device. Take note of the IP. If you can’t view the IP using your admin panel, follow <a href="#finding-device-ip-using-zenmap">this guide</a>.</li>
        <li>Using SSH, connect to the ROCK 5C with:
          <pre><code>ssh radxa@[Device IP]</code></pre>
        </li>
        <li>The default password for the device is <code>radxa</code>.</li>
        <li>Once connected, use the <code>rsetup</code> command and go to the “System Maintenance” tab. Select “System Update” and follow the prompts until complete.</li>
        <li>Run <code>rsetup</code> again, go to “User Settings,” and change your hostname and password used for SSH.</li>
      </ul>
    </details>
    <details open>
      <summary><h3 id="optional-penta-sata-hat-top-board">(Optional) Penta SATA HAT Top Board:</h3></summary>
      <ul>
        <li>If using the Penta SATA HAT Top Board, download and install the <code>rockpi-penta</code> package provided by Radxa:
          <pre><code>wget https://github.com/radxa/rockpi-penta/releases/download/v0.2.2/rockpi-penta-0.2.2.deb
  sudo apt install -y ./rockpi-penta-0.2.2.deb</code></pre>
        </li>
        <li>After installation, a config file <code>rockpi-penta.conf</code> can be found in the <code>/etc/</code> folder.</li>
        <li>Edit the config with vim or nano to tweak settings. If needed, rotate the OLED text 180° here.</li>
        <li>After changing settings, restart the service:
          <pre><code>sudo systemctl restart rockpi-penta.service</code></pre>
        </li>
      </ul>
    </details> 
  </div>
</details>

<!-- Open Media Vault -->
<details open>
  <summary><h2 id="open-media-vault" style="display:inline;">Open Media Vault</h2></summary>
  <div align="justify" style="margin-top:8px;">
    <details open>
      <summary><h3 id="pre-requisites">Pre-Requisites:</h3></summary>
      <ul>
        <li>Even though the installed OS is the lite version, there are still desktop-related libraries that can interfere with OMV installation.</li>
        <li>As of October 2025, there is a faulty uninstall script that requires specific targeting. Disable the script and remove the package:
          <pre><code>sudo mv /var/lib/dpkg/info/radxa-sddm-theme.postrm /var/lib/dpkg/info/radxa-sddm-theme.postrm.bak
  sudo dpkg --remove --force-remove-reinstreq radxa-sddm-theme</code></pre>
        </li>
        <li>Purge the remaining desktop-related libraries:
          <pre><code>sudo apt purge -y desktop-base libqt5waylandclient5 libqt5waylandcompositor5 libwayland-client0 libwayland-cursor0 libwayland-egl1 libwayland-server0 qtwayland5 gsettings-desktop-schemas gir1.2-freedesktop radxa-desktop-branding radxa-sddm-theme</code></pre>
        </li>
        <li>Repair and clean dependencies:
          <pre><code>sudo apt --fix-broken install -y
  sudo apt autoremove -y</code></pre>
        </li>
        <li>Verify no desktop components remain (no output = good to proceed):
          <pre><code>dpkg-query -W -f='${binary:Package}\n' | grep -E 'xserver|desktop|wayland|gnome|kde|xfce|lxqt|lightdm|sddm'</code></pre>
        </li>
      </ul>
    </details>
    <details open>
      <summary><h3 id="installation">Installation:</h3></summary>
      <ul>
        <li>While connected via SSH, run:
          <pre><code>wget -O - https://github.com/OpenMediaVault-Plugin-Developers/installScript/raw/master/preinstall | sudo bash</code></pre>
        </li>
        <li>Once completed, restart the device with <code>sudo reboot</code>.</li>
        <li>Reconnect and begin installing OMV (10–30 minutes):
          <pre><code>wget -O - https://github.com/OpenMediaVault-Plugin-Developers/installScript/raw/master/install | sudo bash</code></pre>
        </li>
        <li>After install, the system restarts. After 3–5 minutes, open in your browser:
          <pre><code>http://[Device IP]</code></pre>
        </li>
        <li>If successful, you’ll see the OMV login page. Use default credentials <code>admin/openmediavault</code>.</li>
        <li><em>Recommended:</em> Configure a static IP for the ROCK 5C (in OMV or your router’s admin panel).</li>
      </ul>
    </details>
    <details open>
      <summary><h3 id="raid-setup-using-zfs">Raid Setup Using ZFS:</h3></summary>
      <ul>
        <li>Before installing plugins, go to <em>Storage &gt; Disks</em> and confirm all 4 hard drives are listed (plus the microSD). Drives should appear as <code>sda</code>/<code>sdb</code>/<code>sdc</code>/<code>sdd</code>.</li>
        <li>Install the following plugins under <em>System &gt; Plugins</em>:
          <ul>
            <li><code>openmediavault-cterm</code></li>
            <li><code>openmediavault-compose</code></li>
            <li><code>openmediavault-zfs</code></li>
          </ul>
        </li>
        <li>Before applying, go to <em>System &gt; omv-extras</em> and enable the “Docker repo”.</li>
        <li>Apply changes and wait for completion.</li>
        <li>Go to <em>Storage &gt; ZFS &gt; Pools</em> and select “Add Pool”.</li>
        <li>Select the name, <a href="#pool-type">Pool Type</a>, storage devices, and other preferences.</li>
        <li>For this setup, select RAID Z1 to allow for recoverability while maximizing disk space.</li>
      </ul>
    </details>
    <details open>
      <summary><h4 id="pool-type">Pool Type:</h4></summary>
      <div style="overflow-x:auto;">
        <table style="border-collapse:collapse;width:100%;">
          <thead>
            <tr style="background-color:#f5f5f5;"  align="center">
              <th style="border:1px solid #ddd;padding:8px;text-align:center;">Type Name</th>
              <th style="border:1px solid #ddd;padding:8px;text-align:center;">Min. Drives Required</th>
              <th style="border:1px solid #ddd;padding:8px;text-align:center;">Parity Drives</th>
              <th style="border:1px solid #ddd;padding:8px;text-align:center;">Tolerated Drive Failures</th>
              <th style="border:1px solid #ddd;padding:8px;text-align:center;">Space Utilization</th>
              <th style="border:1px solid #ddd;padding:8px;text-align:center;">Recommended For</th>
            </tr>
          </thead>
          <tbody  align="center">
            <tr>
              <td style="border:1px solid #ddd;padding:8px;text-align:center;">Basic</td>
              <td style="border:1px solid #ddd;padding:8px;text-align:center;">1</td>
              <td style="border:1px solid #ddd;padding:8px;text-align:center;">-</td>
              <td style="border:1px solid #ddd;padding:8px;text-align:center;">0</td>
              <td style="border:1px solid #ddd;padding:8px;text-align:center;">100%</td>
              <td style="border:1px solid #ddd;padding:8px;text-align:center;">Testing Setup, Non-Critical Storage</td>
            </tr>
            <tr>
              <td style="border:1px solid #ddd;padding:8px;text-align:center;">Mirror</td>
              <td style="border:1px solid #ddd;padding:8px;text-align:center;">2</td>
              <td style="border:1px solid #ddd;padding:8px;text-align:center;">-</td>
              <td style="border:1px solid #ddd;padding:8px;text-align:center;">1</td>
              <td style="border:1px solid #ddd;padding:8px;text-align:center;">50%</td>
              <td style="border:1px solid #ddd;padding:8px;text-align:center;">Small, High-Reliability Setups</td>
            </tr>
            <tr>
              <td style="border:1px solid #ddd;padding:8px;text-align:center;">RAID-Z1</td>
              <td style="border:1px solid #ddd;padding:8px;text-align:center;">3</td>
              <td style="border:1px solid #ddd;padding:8px;text-align:center;">1</td>
              <td style="border:1px solid #ddd;padding:8px;text-align:center;">1</td>
              <td style="border:1px solid #ddd;padding:8px;text-align:center;">60–80%</td>
              <td style="border:1px solid #ddd;padding:8px;text-align:center;">Balanced Performance &amp; Capacity</td>
            </tr>
            <tr>
              <td style="border:1px solid #ddd;padding:8px;text-align:center;">RAID-Z2</td>
              <td style="border:1px solid #ddd;padding:8px;text-align:center;">4</td>
              <td style="border:1px solid #ddd;padding:8px;text-align:center;">2</td>
              <td style="border:1px solid #ddd;padding:8px;text-align:center;">2</td>
              <td style="border:1px solid #ddd;padding:8px;text-align:center;">50–75%</td>
              <td style="border:1px solid #ddd;padding:8px;text-align:center;">Priority on Data Integrity &amp; Uptime</td>
            </tr>
            <tr>
              <td style="border:1px solid #ddd;padding:8px;text-align:center;">RAID-Z3</td>
              <td style="border:1px solid #ddd;padding:8px;text-align:center;">5</td>
              <td style="border:1px solid #ddd;padding:8px;text-align:center;">3</td>
              <td style="border:1px solid #ddd;padding:8px;text-align:center;">3</td>
              <td style="border:1px solid #ddd;padding:8px;text-align:center;">40–70%</td>
              <td style="border:1px solid #ddd;padding:8px;text-align:center;">Critical Long-Term Data Storage</td>
            </tr>
          </tbody>
        </table>
      </div>
    </details>
  </div>
</details>

<!-- Extra Guides -->
<a id="finding-device-ip-using-zenmap"></a>
<details open>
  <summary><h2 id="extra-guides" style="display:inline;">Extra Guides</h2></summary>
  <div align="justify" style="margin-top:8px;">
    <h3>Finding Device IP Using Zenmap:</h3>
    <ul>
      <li>If the IP of the ROCK 5C can’t be determined from the router’s admin panel, <a href="https://nmap.org/">Zenmap</a> can be used from a PC to scan for devices on the network.</li>
      <li>Open a command prompt and run <code>ipconfig</code> to view your current IPv4 address. For this example, assume <code>192.168.10.XXX</code>.</li>
      <li>In Zenmap, set the target to <code>192.168.10.0/24</code> to scan the full /24 (192.168.10.0–192.168.10.255).</li>
      <li>In the command field, use (replace “Target” as needed):
        <pre><code>nmap -F -n -Pn Target</code></pre>
      </li>
      <li>Start the scan. When complete, look for the device that has SSH open on port 22—this should be the ROCK 5C. The scan will reveal both the IP and MAC address.</li>
    </ul>
  </div>
</details>



