<!-- Overview -->
<details open>
  <summary>
    <span style="font-size:1.5em;font-weight:600;">Overview</span>
  </summary>
  <div align="justify" style="margin-top:8px;">
    <p>
      At this point, the server hardware is assembled and the operating system is configured.
      Now, the applications have to be downloaded to turn this remote terminal into a fully functional homelab.
    </p>
    <p>
      The installation of applications on this device centers around Docker, specifically the use of Docker Compose,
      to create containers that will run each application independently.
    </p>
  </div>
</details>
<!-- Pre-Requisites -->
<details open>
  <summary>
    <span style="font-size:1.5em;font-weight:600;">Pre-Requisites</span>
  </summary>
  <div align="justify" style="margin-top:8px;">
    <!-- User Group Creation -->
    <details open>
      <summary>
        <span style="font-size:1.17em;font-weight:600;">User Group Creation</span>
      </summary>
      <p>
        Before any applications are installed, make sure to go to the "Users" section in Open Media Vault
        and add the following user:
      </p>
      <div style="overflow-x:auto;">
        <table style="border-collapse:collapse;width:100%;">
          <thead>
            <tr style="background-color:#f5f5f5;" align="center">
              <th style="border:1px solid #ddd;padding:8px;text-align:center;">Name</th>
              <th style="border:1px solid #ddd;padding:8px;text-align:center;">Shell</th>
              <th style="border:1px solid #ddd;padding:8px;text-align:center;">Groups</th>
            </tr>
          </thead>
          <tbody>
            <tr align="center">
              <td style="border:1px solid #ddd;padding:8px;text-align:center;">appuser</td>
              <td style="border:1px solid #ddd;padding:8px;text-align:center;">/usr/sbin/nologin</td>
              <td style="border:1px solid #ddd;padding:8px;text-align:center;">users</td>
            </tr>
          </tbody>
        </table>
      </div>
      <p>
        After creating the user, apply changes and SSH into the ROCK 5C.
        Then use:
      </p>
      <pre><code>id appuser</code></pre>
    </details>
    <!-- Folder Creation -->
    <details open>
      <summary>
        <span style="font-size:1.17em;font-weight:600;">Folder Creation</span>
      </summary>
      <p>
        Create a storage location on the ZFS pool for application data.
      </p>
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
          <tbody>
            <tr align="center">
              <td style="border:1px solid #ddd;padding:8px;text-align:center;">data</td>
              <td style="border:1px solid #ddd;padding:8px;text-align:center;">(PoolName)</td>
              <td style="border:1px solid #ddd;padding:8px;text-align:center;">/data</td>
              <td style="border:1px solid #ddd;padding:8px;text-align:center;">Don't Change</td>
              <td style="border:1px solid #ddd;padding:8px;text-align:center;">App Data Folder</td>
            </tr>
          </tbody>
        </table>
      </div>
    </details>
  </div>
</details>
