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

<div align="justify">
In this documentation, I will cover the step by step process of setting up a home server using the Radxa ROCK 5C. This server will allow for the global use of services such as <a href="https://www.plex.tv/watch-free/">Plex</a> and <a href="https://github.com/nextcloud/all-in-one?tab=readme-ov-file#are-reverse-proxies-supported">Nextcloud</a> (cloud storage). Additionally, this server will make use of services including <a href="https://github.com/qbittorrent/qBittorrent">qBittorrent</a> and media managers <a href="https://radarr.video/">Radarr</a>, <a href="https://sonarr.tv/">Sonarr</a>, and <a href="https://prowlarr.com/">Prowlarr</a>. All of these services will be ran in <a href="https://www.openmediavault.org/">Open Media Vault</a> using <a href="https://www.docker.com/products/docker-desktop/">Docker</a> and managed using <a href="https://github.com/portainer/portainer">Portainer</a>. Additionally, this server will make use of <a href="https://radarr.video/">Cloudflare Tunnels</a>, allowing it to reliably host services over CGNAT. 
</div>

<h2>Component List</h2>
<table style="border-collapse:collapse;width:100%;">
  <thead>
    <tr style="background-color:#f5f5f5;">
      <th style="border:1px solid #ddd;padding:8px;text-align:center;">Qty</th>
      <th style="border:1px solid #ddd;padding:8px;text-align:center;">Item</th>
      <th style="border:1px solid #ddd;padding:8px;text-align:center;">Supplier</th>
      <th style="border:1px solid #ddd;padding:8px;text-align:center;">Unit Cost</th>
    </tr>
  </thead>
  <tbody>
    <tr style="background-color:#ffffff;">
      <td style="border:1px solid #ddd;padding:8px;text-align:center;">1</td>
      <td style="border:1px solid #ddd;padding:8px;">Radxa ROCK 5C 8GB</td>
      <td style="border:1px solid #ddd;padding:8px;"><a href="https://arace.tech/products/radxa-rock-5c?variant=42798016954548">Arace</a></td>
      <td style="border:1px solid #ddd;padding:8px;text-align:center;">$79.90</td>
    </tr>
    <tr style="background-color:#fafafa;">
      <td style="border:1px solid #ddd;padding:8px;text-align:center;">1</td>
      <td style="border:1px solid #ddd;padding:8px;">Radxa Penta SATA HAT</td>
      <td style="border:1px solid #ddd;padding:8px;"><a href="https://arace.tech/products/radxa-penta-sata-hat-up-to-5x-sata-disks-hat-for-raspberry-pi-5?variant=42788282400948">Arace</a></td>
      <td style="border:1px solid #ddd;padding:8px;text-align:center;">$44.99</td>
    </tr>
    <tr style="background-color:#ffffff;">
      <td style="border:1px solid #ddd;padding:8px;text-align:center;">1</td>
      <td style="border:1px solid #ddd;padding:8px;">Radxa Heatsink 6540B w/ Fan</td>
      <td style="border:1px solid #ddd;padding:8px;"><a href="https://arace.tech/products/radxa-heatsink-6540b-for-rock-5c?_pos=5&_sid=cfd82791e&_ss=r">Arace</a></td>
      <td style="border:1px solid #ddd;padding:8px;text-align:center;">$4.90</td>
    </tr>
    <tr style="background-color:#fafafa;">
      <td style="border:1px solid #ddd;padding:8px;text-align:center;">1</td>
      <td style="border:1px solid #ddd;padding:8px;">Raspberry Pi 5 Penta SATA HAT Top Board</td>
      <td style="border:1px solid #ddd;padding:8px;"><a href="https://a.co/d/2CCh0uS">Amazon</a></td>
      <td style="border:1px solid #ddd;padding:8px;text-align:center;">$17.00</td>
    </tr>
    <tr style="background-color:#ffffff;">
      <td style="border:1px solid #ddd;padding:8px;text-align:center;">1</td>
      <td style="border:1px solid #ddd;padding:8px;">PNY 32GB Elite microSD Card</td>
      <td style="border:1px solid #ddd;padding:8px;"><a href="https://a.co/d/7hQ9TFv">Amazon</a></td>
      <td style="border:1px solid #ddd;padding:8px;text-align:center;">$6.99</td>
    </tr>
    <tr style="background-color:#fafafa;">
      <td style="border:1px solid #ddd;padding:8px;text-align:center;">4</td>
      <td style="border:1px solid #ddd;padding:8px;">KingSpec 512GB 2.5&quot; SSD</td>
      <td style="border:1px solid #ddd;padding:8px;"><a href="https://a.co/d/7ZX31wF">Amazon</a></td>
      <td style="border:1px solid #ddd;padding:8px;text-align:center;">$32.59</td>
    </tr>
    <tr style="background-color:#fafafa;">
      <td style="border:1px solid #ddd;padding:8px;text-align:center;">1</td>
      <td style="border:1px solid #ddd;padding:8px;">12V 3A AC/DC Power Adapter</td>
      <td style="border:1px solid #ddd;padding:8px;"><a href="https://a.co/d/dq2EbIb">Amazon</a></td>
      <td style="border:1px solid #ddd;padding:8px;text-align:center;">$9.99</td>
    </tr>
    <tr>
      <th style="border:1px solid #ddd;padding:8px;text-align:center;"colspan="3" align="right"><strong>Total</strong></th>
      <td style="border:1px solid #ddd;padding:8px;text-align:center;"><strong>$294.13</strong></td>
    </tr>
  </tbody>
</table>

<h2>Technical Specifications</h2>
<h3>Radxa ROCK 5C</h3>
<div align="justify">
<table style="border-collapse:collapse;width:100%;">
  <thead style="background-color:#e6f0ff;" align="center">
    <tr>
      <th style="border:1px solid #bbb;padding:10px;text-align:left;font-weight:700;">Category</th>
      <th style="border:1px solid #bbb;padding:10px;text-align:left;font-weight:700;">Details</th>
    </tr>
  </thead>
  <tbody>
    <tr style="background-color:#ffffff;">
      <td style="border:1px solid #ddd;padding:8px;">Processor</td>
      <td style="border:1px solid #ddd;padding:8px;">Quad-core Cortex-A76 up to 2.4&nbsp;GHz; Quad-core Cortex-A55 up to 1.8&nbsp;GHz</td>
    </tr>
    <tr style="background-color:#fafafa;">
      <td style="border:1px solid #ddd;padding:8px;">GPU</td>
      <td style="border:1px solid #ddd;padding:8px;">Arm Mali‑G610MC4</td>
    </tr>
    <tr style="background-color:#ffffff;">
      <td style="border:1px solid #ddd;padding:8px;">RAM</td>
      <td style="border:1px solid #ddd;padding:8px;">LPDDR4x (2, 4, 8, 16, 32 GB)</td>
    </tr>
    <tr style="background-color:#fafafa;">
      <td style="border:1px solid #ddd;padding:8px;">Storage</td>
      <td style="border:1px solid #ddd;padding:8px;">1× eMMC connector; 1× microSD card; No SPI flash</td>
    </tr>
    <tr style="background-color:#ffffff;">
      <td style="border:1px solid #ddd;padding:8px;">Video Output</td>
      <td style="border:1px solid #ddd;padding:8px;">1× HDMI 2.1 (up to 8K); 1× MIPI DSI (up to 2K)</td>
    </tr>
    <tr style="background-color:#fafafa;">
      <td style="border:1px solid #ddd;padding:8px;">Multimedia</td>
      <td style="border:1px solid #ddd;padding:8px;">H.265 &amp; VP9 decoder 8K@60fps; H.264 decoder 8K@30fps; AV1 decoder 4K@60fps; H.264/H.265 encoder 8K@30fps</td>
    </tr>
    <tr style="background-color:#ffffff;">
      <td style="border:1px solid #ddd;padding:8px;">Ethernet</td>
      <td style="border:1px solid #ddd;padding:8px;">Gigabit Ethernet (Non‑PoE); PoE with additional HAT</td>
    </tr>
    <tr style="background-color:#fafafa;">
      <td style="border:1px solid #ddd;padding:8px;">Wireless</td>
      <td style="border:1px solid #ddd;padding:8px;">Wi‑Fi&nbsp;6 and BT&nbsp;5.4 with external antenna connector</td>
    </tr>
    <tr style="background-color:#ffffff;">
      <td style="border:1px solid #ddd;padding:8px;">USB</td>
      <td style="border:1px solid #ddd;padding:8px;">2× USB&nbsp;2; 1× USB&nbsp;3 HOST; 1× USB&nbsp;3 OTG</td>
    </tr>
    <tr style="background-color:#fafafa;">
      <td style="border:1px solid #ddd;padding:8px;">Power</td>
      <td style="border:1px solid #ddd;padding:8px;">5V USB Type‑C</td>
    </tr>
    <tr style="background-color:#ffffff;">
      <td style="border:1px solid #ddd;padding:8px;">Size</td>
      <td style="border:1px solid #ddd;padding:8px;">85&nbsp;mm × 56&nbsp;mm</td>
    </tr>
  </tbody>
</table>
</div>


<h3>Radxa Penta SATA HAT</h3>
<div align="justify">
<table style="border-collapse:collapse;width:100%;">
  <thead style="background-color:#e6f0ff;" align="center">
    <tr style="background-color:#f5f5f5;">
      <th style="border:1px solid #ddd;padding:8px;text-align:left;">Category</th>
      <th style="border:1px solid #ddd;padding:8px;text-align:left;">Details</th>
    </tr>
  </thead>
  <tbody>
    <tr style="background-color:#ffffff;">
      <td style="border:1px solid #ddd;padding:8px;">Storage</td>
      <td style="border:1px solid #ddd;padding:8px;">Supports up to 5× HDD/SSD; Maximum total size of 100&nbsp;TB</td>
    </tr>
    <tr style="background-color:#fafafa;">
      <td style="border:1px solid #ddd;padding:8px;">Connectors</td>
      <td style="border:1px solid #ddd;padding:8px;">4× SATA; 1× eSATA w/ Power; 1× Molex; 1× Expansion interface for Top Board</td>
    </tr>
    <tr style="background-color:#ffffff;">
      <td style="border:1px solid #ddd;padding:8px;">Power</td>
      <td style="border:1px solid #ddd;padding:8px;">Accepts power from SBC; Additional 12V DC jack input</td>
    </tr>
    <tr style="background-color:#fafafa;">
      <td style="border:1px solid #ddd;padding:8px;">Software</td>
      <td style="border:1px solid #ddd;padding:8px;">Supports RAID (0, 1, 5)</td>
    </tr>
  </tbody>
</table>
</div>

<h3>Raspberry Pi 5 Penta SATA HAT Top Board</h3>
<div align="justify">
<table style="border-collapse:collapse;width:100%;">
  <thead style="background-color:#e6f0ff;" align="center">
    <tr style="background-color:#f5f5f5;">
      <th style="border:1px solid #ddd;padding:8px;text-align:left;">Category</th>
      <th style="border:1px solid #ddd;padding:8px;text-align:left;">Details</th>
    </tr>
  </thead>
  <tbody>
    <tr style="background-color:#ffffff;">
      <td style="border:1px solid #ddd;padding:8px;">Display</td>
      <td style="border:1px solid #ddd;padding:8px;">OLED</td>
    </tr>
    <tr style="background-color:#fafafa;">
      <td style="border:1px solid #ddd;padding:8px;">Cooling</td>
      <td style="border:1px solid #ddd;padding:8px;">40×40&nbsp;mm Fan</td>
    </tr>
    <tr style="background-color:#ffffff;">
      <td style="border:1px solid #ddd;padding:8px;">Connector</td>
      <td style="border:1px solid #ddd;padding:8px;">10‑pin connector to interface with the Penta SATA HAT</td>
    </tr>
    <tr style="background-color:#fafafa;">
      <td style="border:1px solid #ddd;padding:8px;">Additional Feature(s)</td>
      <td style="border:1px solid #ddd;padding:8px;">Button to control display</td>
    </tr>
  </tbody>
</table>
</div>

<h3>Storage Drives</h3>
<div align="justify">
<table style="border-collapse:collapse;width:100%;">
  <thead style="background-color:#e6f0ff;" align="center">
    <tr style="background-color:#f5f5f5;">
      <th style="border:1px solid #ddd;padding:8px;text-align:left;">Category</th>
      <th style="border:1px solid #ddd;padding:8px;text-align:left;">Details</th>
    </tr>
  </thead>
  <tbody>
    <tr style="background-color:#ffffff;">
      <td style="border:1px solid #ddd;padding:8px;">MicroSD Card</td>
      <td style="border:1px solid #ddd;padding:8px;">1× PNY 32GB Elite Class; 100&nbsp;MB/s Read &amp; Write</td>
    </tr>
    <tr style="background-color:#fafafa;">
      <td style="border:1px solid #ddd;padding:8px;">Solid State Drives</td>
      <td style="border:1px solid #ddd;padding:8px;">4× KingSpec 512GB 2.5&quot; SSD; 550&nbsp;MB/s Read; 520&nbsp;MB/s Write; Low power consumption</td>
    </tr>
  </tbody>
</table>
</div>

<br/>

<a href="1_Assembly.md"><strong>Next → Assembly.md</strong></a>

