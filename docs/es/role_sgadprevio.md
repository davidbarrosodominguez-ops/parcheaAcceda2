# Documentación del Rol `sgadprevio`

## Propósito
El rol `sgadprevio` es el núcleo de la auditoría. Ejecuta una serie de tareas secuenciales para verificar la salud del sistema y detectar actualizaciones pendientes.

## Desglose de Tareas

El rol divide su lógica en múltiples ficheros de tareas (`tasks/`), importados desde `main.yml`:

1.  **`01_prechecks.yml`**:
    *   Crea ficheros temporales y de destino para los informes en el nodo delegado.

2.  **`02_subscription.yml`**:
    *   Verifica el estado de `subscription-manager`.
    *   Calcula la versión futura (minor release).
    *   Verifica si hay actualizaciones disponibles (`yum list updates`).
    *   Consulta `insights-client` para versiones sugeridas.

3.  **`03_repos.yml`**:
    *   Lista los repositorios habilitados.
    *   Verifica el release en Satellite.

4.  **`04_historial.yml`**:
    *   Recopila los últimos paquetes instalados (`rpm -qa --last`).
    *   Revisa el historial de `yum`.

5.  **`05_updates.yml`**:
    *   Lista detallada de actualizaciones.
    *   Identifica parches de seguridad específicos.
    *   Detecta paquetes obsoletos.

6.  **`06_red.yml`**:
    *   Recopila rutas (`ip route`, `route -n`).
    *   Verifica configuración de interfaces (`ifcfg-*`), bonding y VLANs.
    *   Lista reglas de `iptables`.

7.  **`07_servicios.yml`**:
    *   Lista servicios en ejecución y fallidos (`systemctl`).
    *   Verifica `/etc/hosts`.
    *   Comprueba conectividad con puertos clave (Commvault, AAP).
    *   Verifica sincronización horaria (NTP/Chrony).

8.  **`08_seguridad.yml`**:
    *   Revisa certificados PKI (`/etc/pki`).
    *   Lista usuarios del sistema.
    *   Revisa configuración de `sudoers` y `sssd`.

9.  **`09_hardware.yml`**:
    *   Recopila información de hardware usando `dmidecode` (BIOS, sistema, CPU, memoria).
    *   Estado de `insights-client`.

10. **`10_discos.yml`**:
    *   Verifica espacio en disco (`df -h`).
    *   Puntos de montaje (`mount`).
    *   Identificación de bloques (`blkid`, `lsblk`).
    *   Analiza uso en `/opt`.

11. **`11_enriquecidos.yml`**:
    *   Cruza información con una lista externa para obtener datos de contacto y aplicación.
    *   Verifica procesos y logs de aplicaciones específicas (Java, Wildfly, Oracle, etc.).

12. **`12_report.yml`**:
    *   Renderiza la plantilla HTML (`cabecera.html.j2`).
    *   Genera el CSV con el resumen.
    *   Envía el correo electrónico final usando `sendmail`.

13. **`99_postchecks.yml`**:
    *   Limpia ficheros temporales.
    *   Mueve los informes al almacén definitivo.

## Variables Importantes
*   `fichero`: Ruta principal del informe HTML.
*   `fich1`, `fich2`, `fich3`: Ficheros temporales para salida de comandos.
*   `destinatorio_email`: (Hardcoded actualmente en la tarea de mail) Dirección de envío.

## Plantillas
*   **`templates/cabecera.html.j2`**: Plantilla Jinja2 base para el informe HTML.
