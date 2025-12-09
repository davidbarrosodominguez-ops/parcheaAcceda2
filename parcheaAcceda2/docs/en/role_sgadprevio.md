# `sgadprevio` Role Documentation

## Purpose
The `sgadprevio` role is the core of the audit. It executes a series of sequential tasks to verify system health and detect pending updates.

## Task Breakdown

The role divides its logic into multiple task files (`tasks/`), imported from `main.yml`:

1.  **`01_prechecks.yml`**:
    *   Creates temporary and destination files for reports on the delegated node.

2.  **`02_subscription.yml`**:
    *   Verifies `subscription-manager` status.
    *   Calculates the future version (minor release).
    *   Checks if updates are available (`yum list updates`).
    *   Queries `insights-client` for suggested versions.

3.  **`03_repos.yml`**:
    *   Lists enabled repositories.
    *   Verifies release in Satellite.

4.  **`04_historial.yml`**:
    *   Collects latest installed packages (`rpm -qa --last`).
    *   Checks `yum` history.

5.  **`05_updates.yml`**:
    *   Detailed list of updates.
    *   Identifies specific security patches.
    *   Detects obsolete packages.

6.  **`06_red.yml`**:
    *   Collects routes (`ip route`, `route -n`).
    *   Verifies interface configuration (`ifcfg-*`), bonding, and VLANs.
    *   Lists `iptables` rules.

7.  **`07_servicios.yml`**:
    *   Lists running and failed services (`systemctl`).
    *   Verifies `/etc/hosts`.
    *   Checks connectivity with key ports (Commvault, AAP).
    *   Verifies time synchronization (NTP/Chrony).

8.  **`08_seguridad.yml`**:
    *   Reviews PKI certificates (`/etc/pki`).
    *   Lists system users.
    *   Checks `sudoers` and `sssd` configuration.

9.  **`09_hardware.yml`**:
    *   Collects hardware info using `dmidecode` (BIOS, system, CPU, memory).
    *   `insights-client` status.

10. **`10_discos.yml`**:
    *   Verifies disk space (`df -h`).
    *   Mount points (`mount`).
    *   Block identification (`blkid`, `lsblk`).
    *   Analyzes usage in `/opt`.

11. **`11_enriquecidos.yml`**:
    *   Cross-references info with an external list to get contact and application data.
    *   Verifies processes and logs for specific applications (Java, Wildfly, Oracle, etc.).

12. **`12_report.yml`**:
    *   Renders the HTML template (`cabecera.html.j2`).
    *   Generates the CSV with the summary.
    *   Sends the final email using `sendmail`.

13. **`13_postchecks.yml`**:
    *   Cleans up temporary files.
    *   Moves reports to the final storage location.

## Key Variables
*   `fichero`: Main path for the HTML report.
*   `fich1`, `fich2`, `fich3`: Temporary files for command output.
*   `destinatorio_email`: (Currently hardcoded in the mail task) Recipient address.

## Templates
*   **`templates/cabecera.html.j2`**: Base Jinja2 template for the HTML report.
