# Diccionario de Datos del Informe HTML

Este documento detalla el origen de las variables utilizadas en la plantilla del informe HTML (`roles/sgadprevio/templates/cabecera.html.j2`). Sirve como referencia para entender de dónde proviene cada dato y facilitar el mantenimiento del informe.

Las variables se originan principalmente de tres fuentes:
1.  **Ansible Facts**: Datos recopilados automáticamente por Ansible sobre el sistema (`ansible_*`).
2.  **Comandos del Sistema**: Variables registradas (`register`) tras ejecutar un comando en el host.
3.  **Variables Definidas**: Variables creadas explícitamente con `set_fact`.

---

## Tabla de Variables

| Variable | Fichero Origen | Tarea / Comando que la genera | Descripción |
| :--- | :--- | :--- | :--- |
| `logo.content` | `11_enriquecidos.yml` | `slurp` del fichero `/home/reexus/LOGO_GOB_MTDFP_AEAD.png` | Contenido en Base64 del logotipo para el encabezado. |
| `hayupdates` | `02_subscription.yml` | `set_fact` | Mensaje que indica si hay o no actualizaciones disponibles. |
| `ansible_date_time.date` | Ansible Fact | - | Fecha de ejecución del playbook. |
| `ansible_hostname` | Ansible Fact | - | Nombre del host sobre el que se ejecuta el informe. |
| `ansible_distribution_version` | Ansible Fact | - | Versión de la distribución del sistema operativo (e.g., 8.6). |
| `versionfutura.stdout` | `02_subscription.yml` | `subscription-manager release --show` | Release de RHEL al que el sistema está fijado para futuras actualizaciones. |
| `ansible_kernel` | Ansible Fact | - | Versión del kernel de Linux en ejecución. |
| `suggested.stdout` | `02_subscription.yml` | `insights-client --show-details` | Versión de RHEL sugerida por Red Hat Insights. |
| `diasultimoreinicio.stdout_lines` | `09_hardware.yml` | `who -b` y procesado | Días transcurridos desde el último reinicio del sistema. |
| `kernelLast.stdout` | `09_hardware.yml` | `rpm -q --last kernel` | Fecha de la última instalación/actualización del paquete del kernel. |
| `subscription.stdout_lines` | `02_subscription.yml` | `subscription-manager status` | Bloque de texto con el estado detallado de la suscripción de Red Hat. |
| `exclu.stdout` | `03_repos.yml` | `cat /etc/dnf/dnf.conf` o `yum.conf` | Muestra la línea `exclude` para ver paquetes excluidos de actualizaciones. |
| `subscriptionrelease.stdout` | `03_repos.yml` | `subscription-manager release --show` | Release de RHEL configurado en `subscription-manager`. |
| `repos.stdout_lines` | `03_repos.yml` | `dnf repolist` o `yum repolist` | Lista de repositorios de software habilitados. |
| `ulti.stdout_lines` | `04_historial.yml` | `rpm -qa --last` | Lista de los últimos paquetes instalados a través de RPM. |
| `histo.stdout` | `04_historial.yml` | `dnf history` o `yum history` | Historial de transacciones de `yum`/`dnf`. |
| `reclamo.stdout_lines` | `05_updates.yml` | `package-cleanup --unclaimed` | Paquetes que no son requeridos por ningún otro paquete. |
| `rpminstalados.stdout_lines` | `05_updates.yml` | `yum list-sec` | Parches de seguridad ya instalados. |
| `rpmsegactualizables.stdout_lines`| `05_updates.yml` | `yum list-sec-minimal --sec-severity=Important,Critical` | Actualizaciones de seguridad críticas e importantes que están pendientes. |
| `Lis.stdout_lines` | `05_updates.yml` | `yum list updates` | Lista completa de todos los paquetes con actualizaciones pendientes. |
| `rhcstatus.stdout_lines` | `07_servicios.yml` | `rhc status` | Estado de la conexión con Red Hat Cloud (RHC). |
| `insights_status` / `insights_version` | `07_servicios.yml` | `insights-client --status` | Estado y versión del cliente de Red Hat Insights. |
| `conectaap.stdout` | `07_servicios.yml` | `curl aap.correos.es` | Resultado de la prueba de conectividad con `aap.correos.es`. |
| `conectcommvault.stdout` | `07_servicios.yml` | `curl commvault-ng.correos.es` | Resultado de la prueba de conectividad con `commvault-ng.correos.es`. |
| `sincrontpd.stdout` / `sincrochronie.stdout` | `07_servicios.yml` | `systemctl status ntpd/chronyd` | Estado de los servicios de sincronización horaria. |
| `synchronized.stdout_lines` | `07_servicios.yml` | `timedatectl status` | Estado de la sincronización del reloj del sistema. |
| `ansible_selinux.status` | Ansible Fact | - | Estado de SELinux (e.g., enforcing, permissive, disabled). |
| `ansible_virtualization_role` | Ansible Fact | - | Rol de virtualización (e.g., host, guest). |
| `bios.stdout_lines` | `09_hardware.yml` | `dmidecode -t bios` | Información de la BIOS. |
| `system.stdout_lines` | `09_hardware.yml` | `dmidecode -t system` | Información del sistema (fabricante, modelo). |
| `processor.stdout_lines` | `09_hardware.yml` | `dmidecode -t processor` | Información de la CPU. |
| `memory.stdout_lines` | `09_hardware.yml` | `dmidecode -t memory` | Información de los módulos de memoria RAM. |
| `iproute.stdout_lines` | `06_red.yml` | `ip route` | Tabla de enrutamiento del sistema. |
| `route.stdout_lines` | `06_red.yml` | `route -n` | Tabla de enrutamiento (formato antiguo). |
| `iptableslist.stdout` | `06_red.yml` | `iptables -L -n` | Listado de reglas del firewall `iptables`. |
| `serviciosrunning.stdout` | `07_servicios.yml` | `systemctl --type=service --state=running` | Lista de todos los servicios que están actualmente en ejecución. |
| `serviciosFallidos.stdout` | `07_servicios.yml` | `systemctl --failed --type=service` | Lista de servicios que han fallado al iniciarse o ejecutarse. |
| `procesosapp.stdout` | `11_enriquecidos.yml` | `ps -ef` | Procesos en ejecución de la aplicación identificada. |
| `etchosts.stdout_lines` | `08_seguridad.yml` | `cat /etc/hosts` | Contenido del fichero `/etc/hosts`. |
| `etcfstab.stdout_lines` | `08_seguridad.yml` | `cat /etc/fstab` | Contenido del fichero `/etc/fstab`. |
| `etcconfigini.stdout_lines` | `08_seguridad.yml` | `ls` en `/opt/wildfly-10.1.0.Final/...` | Contenido del fichero de configuración de la aplicación (ruta hardcodeada a Wildfly). |
| `realmlist.stdout_lines` | `08_seguridad.yml` | `realm list` | Dominios a los que está unido el sistema. |
| `fichsssd.stdout_lines` | `08_seguridad.yml` | `cat /etc/sssd/sssd.conf` | Contenido del fichero de configuración de SSSD. |
| `sudoers.stdout_lines` | `08_seguridad.yml` | `cat /etc/sudoers` | Contenido del fichero de `sudoers`. |
| `homeusers.stdout_lines`| `08_seguridad.yml` | `awk` sobre `/etc/passwd` | Listado de todos los usuarios del sistema. |
| `blkidstatus.stdout_lines` | `10_discos.yml` | `blkid -s UUID -s TYPE` | Identificadores únicos (UUID) y tipos de los dispositivos de bloque. |
| `uso_disco.stdout_lines` | `10_discos.yml` | `df -h` | Uso del espacio en disco para todos los sistemas de ficheros. |
| `opt_info`, `tmp_info`, etc. | `10_discos.yml` | `set_fact` sobre `ansible_mounts` | Información detallada de puntos de montaje específicos (`/opt`, `/tmp`...). |
| `fsmount.stdout_lines` | `10_discos.yml` | `mount` | Sistemas de ficheros actualmente montados. |
| `optusers.stdout_lines`| `11_enriquecidos.yml` | `ls -l /opt` | Listado de ficheros y propietarios en `/opt`. |
| `contacto.stdout` | `11_enriquecidos.yml` | `grep` sobre `paraenriquecerParchea.lst` | Datos del contacto asociado al servidor. |
| `aplicacion.stdout` | `11_enriquecidos.yml` | `grep` sobre `paraenriquecerParchea.lst` | Nombre de la aplicación asociada al servidor. |
| `fechas_parcheado.stdout_lines`| `11_enriquecidos.yml` | `cat fechasParcheado.lst` | Contenido del fichero de fechas de parcheo. |
| `resumen.stdout` | `11_enriquecidos.yml` | Contenido del fichero `resumen.txt` | Resumen de advertencias y errores encontrados durante la ejecución. |

---