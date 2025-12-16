# HTML Report Data Dictionary

This document details the origin of the variables used in the HTML report template (`roles/sgadprevio/templates/cabecera.html.j2`). It serves as a reference to understand where each piece of data comes from and to facilitate report maintenance.

The variables originate mainly from three sources:
1.  **Ansible Facts**: Data automatically collected by Ansible about the system (`ansible_*`).
2.  **System Commands**: Variables registered (`register`) after executing a command on the host.
3.  **Defined Variables**: Variables explicitly created with `set_fact`.

---

## Variables Table

| Variable | Origin File | Generating Task / Command | Description |
| :--- | :--- | :--- | :--- |
| `logo.content` | `11_enriquecidos.yml` | `slurp` of file `/home/reexus/LOGO_GOB_MTDFP_AEAD.png` | Base64 content of the logo for the header. |
| `hayupdates` | `02_subscription.yml` | `set_fact` | Message indicating whether updates are available or not. |
| `ansible_date_time.date`| Ansible Fact | - | Date the playbook was executed. |
| `ansible_hostname` | Ansible Fact | - | Name of the host on which the report is run. |
| `ansible_distribution_version` | Ansible Fact | - | Operating system distribution version (e.g., 8.6). |
| `versionfutura.stdout` | `02_subscription.yml` | `subscription-manager release --show` | RHEL release to which the system is pinned for future updates. |
| `ansible_kernel` | Ansible Fact | - | Version of the running Linux kernel. |
| `suggested.stdout` | `02_subscription.yml` | `insights-client --show-details` | RHEL version suggested by Red Hat Insights. |
| `diasultimoreinicio.stdout_lines` | `09_hardware.yml` | `who -b` and processing | Days elapsed since the last system reboot. |
| `kernelLast.stdout` | `09_hardware.yml` | `rpm -q --last kernel` | Date of the last kernel package installation/update. |
| `subscription.stdout_lines` | `02_subscription.yml` | `subscription-manager status` | Text block with the detailed status of the Red Hat subscription. |
| `exclu.stdout` | `03_repos.yml` | `cat /etc/dnf/dnf.conf` or `yum.conf` | Shows the `exclude` line to see packages excluded from updates. |
| `subscriptionrelease.stdout`| `03_repos.yml` | `subscription-manager release --show` | RHEL release configured in `subscription-manager`. |
| `repos.stdout_lines` | `03_repos.yml` | `dnf repolist` or `yum repolist` | List of enabled software repositories. |
| `ulti.stdout_lines` | `04_historial.yml` | `rpm -qa --last` | List of the last packages installed via RPM. |
| `histo.stdout` | `04_historial.yml` | `dnf history` or `yum history` | Transaction history of `yum`/`dnf`. |
| `reclamo.stdout_lines` | `05_updates.yml` | `package-cleanup --unclaimed` | Packages that are not required by any other package. |
| `rpminstalados.stdout_lines`| `05_updates.yml` | `yum list-sec` | Security patches that are already installed. |
| `rpmsegactualizables.stdout_lines`| `05_updates.yml` | `yum list-sec-minimal --sec-severity=Important,Critical` | Critical and Important security updates that are pending. |
| `Lis.stdout_lines` | `05_updates.yml` | `yum list updates` | Full list of all packages with pending updates. |
| `rhcstatus.stdout_lines` | `07_servicios.yml` | `rhc status` | Connection status with Red Hat Cloud (RHC). |
| `insights_status` / `insights_version` | `07_servicios.yml` | `insights-client --status` | Status and version of the Red Hat Insights client. |
| `conectaap.stdout` | `07_servicios.yml` | `curl aap.correos.es` | Result of the connectivity test with `aap.correos.es`. |
| `conectcommvault.stdout`| `07_servicios.yml` | `curl commvault-ng.correos.es` | Result of the connectivity test with `commvault-ng.correos.es`. |
| `sincrontpd.stdout` / `sincrochronie.stdout` | `07_servicios.yml` | `systemctl status ntpd/chronyd` | Status of the time synchronization services. |
| `synchronized.stdout_lines`| `07_servicios.yml` | `timedatectl status` | Status of the system clock synchronization. |
| `ansible_selinux.status`| Ansible Fact | - | SELinux status (e.g., enforcing, permissive, disabled). |
| `ansible_virtualization_role` | Ansible Fact | - | Virtualization role (e.g., host, guest). |
| `bios.stdout_lines` | `09_hardware.yml` | `dmidecode -t bios` | BIOS information. |
| `system.stdout_lines` | `09_hardware.yml` | `dmidecode -t system` | System information (manufacturer, model). |
| `processor.stdout_lines`| `09_hardware.yml` | `dmidecode -t processor` | CPU information. |
| `memory.stdout_lines` | `09_hardware.yml` | `dmidecode -t memory` | Information about RAM modules. |
| `iproute.stdout_lines` | `06_red.yml` | `ip route` | The system's routing table. |
| `route.stdout_lines` | `06_red.yml` | `route -n` | Routing table (legacy format). |
| `iptableslist.stdout` | `06_red.yml` | `iptables -L -n` | List of `iptables` firewall rules. |
| `serviciosrunning.stdout`| `07_servicios.yml` | `systemctl --type=service --state=running` | List of all services that are currently running. |
| `serviciosFallidos.stdout`| `07_servicios.yml` | `systemctl --failed --type=service` | List of services that have failed to start or run. |
| `procesosapp.stdout` | `11_enriquecidos.yml` | `ps -ef` | Running processes of the identified application. |
| `etchosts.stdout_lines` | `08_seguridad.yml` | `cat /etc/hosts` | Content of the `/etc/hosts` file. |
| `etcfstab.stdout_lines` | `08_seguridad.yml` | `cat /etc/fstab` | Content of the `/etc/fstab` file. |
| `etcconfigini.stdout_lines` | `08_seguridad.yml` | `ls` on `/opt/wildfly-10.1.0.Final/...` | Content of the application's configuration file (path is hardcoded to Wildfly). |
| `realmlist.stdout_lines` | `08_seguridad.yml` | `realm list` | Domains the system is joined to. |
| `fichsssd.stdout_lines` | `08_seguridad.yml` | `cat /etc/sssd/sssd.conf` | Content of the SSSD configuration file. |
| `sudoers.stdout_lines` | `08_seguridad.yml` | `cat /etc/sudoers` | Content of the `sudoers` file. |
| `homeusers.stdout_lines`| `08_seguridad.yml` | `awk` on `/etc/passwd` | List of all system users. |
| `blkidstatus.stdout_lines`| `10_discos.yml` | `blkid -s UUID -s TYPE` | Unique identifiers (UUID) and types of block devices. |
| `uso_disco.stdout_lines` | `10_discos.yml` | `df -h` | Disk space usage for all file systems. |
| `opt_info`, `tmp_info`, etc. | `10_discos.yml` | `set_fact` on `ansible_mounts` | Detailed information for specific mount points (`/opt`, `/tmp`...). |
| `fsmount.stdout_lines` | `10_discos.yml` | `mount` | Currently mounted file systems. |
| `optusers.stdout_lines`| `11_enriquecidos.yml` | `ls -l /opt` | List of files and owners in `/opt`. |
| `contacto.stdout` | `11_enriquecidos.yml` | `grep` on `paraenriquecerParchea.lst` | Contact data associated with the server. |
| `aplicacion.stdout` | `11_enriquecidos.yml` | `grep` on `paraenriquecerParchea.lst` | Name of the application associated with the server. |
| `fechas_parcheado.stdout_lines`| `11_enriquecidos.yml` | `cat fechasParcheado.lst` | Content of the patching dates file. |
| `resumen.stdout` | `11_enriquecidos.yml` | Content of the `resumen.txt` file | Summary of warnings and errors found during execution. |

---