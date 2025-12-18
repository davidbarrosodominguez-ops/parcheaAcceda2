# Oportunidades de Refactorización

Este documento detalla áreas de mejora identificadas en el código actual para facilitar su mantenimiento, legibilidad y robustez.



## 2. Gestión de Rutas y Variables
**Problema**: Existen múltiples rutas y nombres de fichero definidos directamente en los ficheros de tareas y en el playbook principal, lo que dificulta la adaptación del rol a diferentes entornos.

**Ejemplos Específicos:**
- **Playbook Principal (`sgadprevio.yml`):** Las rutas para los ficheros temporales y el informe final están hardcodeadas (e.g., `/home/reexus/Acceda2/SGADprevio/rhel_previo_{{...}}.html`).
- **Tarea de Seguridad (`08_seguridad.yml`):** La ruta para encontrar la configuración de la aplicación está fijada a una versión específica de Wildfly: `/opt/wildfly-10.1.0.Final/standalone/configuration/`.
- **Tareas de Enriquecimiento (`11_enriquecidos.yml`):** Las rutas a los ficheros de enriquecimiento (`paraenriquecerParchea.lst`) y el logotipo (`LOGO_GOB_MTDFP_AEAD.png`) son absolutas y dependen del usuario `reexus`.

**Acción Recomendada**:
- **Centralizar variables:** Mover todas estas rutas al fichero de variables del rol (`roles/sgadprevio/vars/all_vars.yml`).
- **Crear variables para:**
  - `report_base_path`: Directorio base para los informes.
  - `app_config_path`: Ruta al directorio de configuración de la aplicación.
  - `enrichment_file_path`: Ruta al fichero de enriquecimiento.
  - `logo_file_path`: Ruta al fichero del logotipo.
- Esto permitiría que un usuario sobreescribiera fácilmente estas rutas desde el inventario o un fichero de variables extra, haciendo el rol mucho más reutilizable.

## 3. Uso de Módulos de Ansible vs Shell
**Problema**: Se usan de forma extensiva los módulos `shell` y `command` con procesado de texto (`grep`, `awk`, etc.) para obtener información que los módulos nativos de Ansible o los "facts" ya proveen de una forma más fiable y estructurada.

**Ejemplos Específicos:**
- **Obtener estado de servicios (en `07_servicios.yml`):** Las tareas `serviciosrunning` y `serviciosFallidos` ejecutan `systemctl` y procesan su salida. Esto podría reemplazarse por el módulo `service_facts`, que devuelve un listado estructurado de todos los servicios y su estado, eliminando la necesidad de procesar texto.
- **Listar actualizaciones (en `05_updates.yml`):** La tarea `Lis` ejecuta `yum list updates`. El propio módulo `yum` puede usarse con el parámetro `list=updates` para obtener esta información de forma programática y más segura.
- **Consultar espacio en disco (en `10_discos.yml`):** La tarea `uso_disco` ejecuta `df -h`. Ansible ya recopila esta información de forma automática durante el `gather_facts` y la almacena en la variable `ansible_mounts`. Usar este "fact" es más eficiente que ejecutar un nuevo comando.
- **Comprobación de certificados (en `08_seguridad.yml`):** El comando `find` con `openssl` para revisar certificados podría ser sustituido por el módulo `community.crypto.x509_certificate_info`, que obtiene la información de manera estructurada.

**Acción Recomendada**:
- Reemplazar los comandos `shell` por los módulos nativos de Ansible (`service_facts`, `yum`, `package_facts`, etc.) siempre que sea posible.
- Aprovechar los "Ansible facts" (`ansible_mounts`, `ansible_default_ipv4`, etc.) en lugar de volver a ejecutar comandos que obtienen esa misma información.
- **Beneficio**: Esto hará el rol más idempotente, legible, rápido (al reducir ejecuciones de comandos) y menos propenso a romperse si el formato de texto de un comando de Linux cambia entre versiones.

## 4. Gestión de Errores
**Problema**: Se abusa del parámetro `ignore_errors: True` (más de 40 instancias) para evitar que el playbook falle. Esta práctica puede ocultar problemas reales (ej: un comando no existe, un servicio falla por una razón inesperada) y dificulta el diagnóstico.

**Ejemplos Específicos:**
- **Comprobación de ficheros (en `08_seguridad.yml`):** La tarea "Fichero config" ejecuta un `ls` y si falla, imprime un `echo` con HTML. Sería más robusto usar el módulo `stat` para verificar si la ruta existe. Si no existe, se puede registrar una variable y usar esa variable en la plantilla Jinja2 para mostrar el error, en lugar de generar HTML en la propia tarea.
- **Pruebas de conectividad (en `07_servicios.yml`):** Las tareas que usan `curl` para verificar la conectividad simplemente ignoran los fallos. Es preferible usar el módulo `uri`, que permite un control más granular del resultado (e.g., `failed_when: mi_resultado.status_code != 200`) y no depende de que el comando `curl` esté instalado.
- **Recopilación de datos (en `09_hardware.yml`):** Las llamadas a `dmidecode` ignoran los errores. Si `dmidecode` no estuviera instalado, las variables quedarían vacías sin que se notifique la causa raíz. Una mejor práctica sería comprobar si el comando existe primero, o usar un bloque `block/rescue` para capturar el error y registrar un mensaje claro que se pueda mostrar en el informe.

**Acción Recomendada**:
- Limitar el uso de `ignore_errors: True` solo a situaciones donde el fallo es totalmente esperado y no relevante.
- Usar `failed_when` para definir condiciones de fallo explícitas basadas en el código de retorno o la salida de un comando.
- Usar bloques `block/rescue/always` para gestionar errores de forma controlada, permitiendo ejecutar tareas de limpieza o registrar mensajes de error específicos.
- Utilizar módulos como `stat` para comprobar precondiciones (ej: si un fichero existe) antes de ejecutar un comando que dependa de él.


## 6. Generación de HTML
**Problema**: En varias tareas se genera código HTML directamente desde el `shell` usando `echo`, especialmente para mostrar mensajes de error. Esto mezcla la lógica de recopilación de datos con la lógica de presentación.

**Ejemplo Específico (de `08_seguridad.yml`):**
```yaml
- name: Fichero config
  shell: ls ... || echo -e '<p style=color:red;>...SIN CONFIGURACIÓN...</p>'
  register: etcconfigini
  ignore_errors: True
```
En este caso, la tarea de Ansible es responsable de generar un fragmento de HTML. Si se quisiera cambiar el estilo del error (e.g., usar una clase CSS en vez de `style=color:red`), habría que modificar el código de la tarea de Ansible, no la plantilla.o pasar el aviso a la plantilla.


**Acción Recomendada**:
- **Separar Lógica y Presentación**: Las tareas de Ansible solo deben recopilar datos y registrar variables (e.g., `etcconfigini_found: false`).
- **Mover la Lógica de Presentación a la Plantilla**: La plantilla `cabecera.html.j2` debe ser la única responsable de generar HTML. Debería contener la lógica para mostrar el error basándose en las variables que las tareas han registrado.
  ```jinja
  {% if not etcconfigini_found %}
    <p class="error-message">⚠️ SIN CONFIGURACIÓN de Fichero config.</p>
  {% endif %}
  ```
- **Beneficio**: Esto sigue el principio de separación de conceptos, haciendo que tanto el rol de Ansible como la plantilla HTML sean mucho más fáciles de mantener y modificar de forma independiente.

## 7. Implementar Comprobación de Puertos Inseguros
**Problema**: La especificación `specs/REGLAS_SEGURIDAD.md` define una regla para detectar puertos inseguros abiertos (ej: 21/ftp, 23/telnet). Sin embargo, esta comprobación no está implementada actualmente en ninguna tarea, lo que genera una inconsistencia entre la documentación y la auditoría real.

**Acción Recomendada**:
- Añadir una nueva tarea en el fichero `roles/sgadprevio/tasks/08_seguridad.yml`.
- Esta tarea debe utilizar un comando como `ss -tlpn` para listar los puertos TCP en escucha.
- Se debe comparar la lista de puertos con una lista negra de puertos inseguros (inicialmente: 21, 23, 80, 110, 143).
- Si se encuentra una coincidencia, se debe registrar un resultado que la plantilla pueda utilizar para mostrar un mensaje `REVISAR_..._REVISAR` en el informe final.
- **Beneficio**: Aumenta la cobertura de la auditoría de seguridad y alinea la implementación del rol con sus especificaciones.

## 8. Refactorizar Generación de Errores a la Plantilla
**Problema**: Siguiendo el principio del punto 6, muchas tareas que comprueban estados y pueden fallar, generan el mensaje de `REVISAR_` directamente en la `shell`. Esto acopla la lógica de la tarea a la presentación del error.

**Ejemplo de Refactorización**:
La plantilla `cabecera.html.j2` ya está preparada para manejar esta lógica. Por ejemplo, comprueba si una variable tiene contenido y, si no, muestra un error.

*   **Caso de estudio**: Tarea `configuracion de dominio` en `08_seguridad.yml`.

*   **🔴 ANTES (Código actual)**:
    ```yaml
    - name: configuracion de dominio
      shell: realm list |grep ':' || echo -e '<p style=color:red;>⚠️ ⚠️ REVISAR_⚠️ ⚠️ SIN CONFIGURACIÓN REALMD _REVISAR</p>'
      register: realmlist
    ```

*   **🟢 DESPUÉS (Solución propuesta)**:
    1.  **Simplificar la tarea**: La tarea solo recoge el dato y no se preocupa del error.
        ```yaml
        - name: configuracion de dominio
          shell: realm list | grep ':'
          register: realmlist
          ignore_errors: True
          changed_when: false
        ```
    2.  **Mover lógica a la plantilla (`.j2`)**: La plantilla comprueba la variable y muestra el error si es necesario.
        ```jinja
        <details>
            <summary>DATOS DE DOMINIO</summary>
            {% if realmlist.stdout | trim %}
                <p>{{ realmlist.stdout_lines | join('<br>') }}</p>
            {% else %}
                <p style="color:red;">REVISAR_⚠️ ⚠️ NO HAY CONFIGURACIÓN DE DOMINIO (REALMD)_REVISAR</p>
            {% endif %}
        </details>
        ```

**Acción Recomendada**: Aplicar este patrón a todas las tareas candidatas para centralizar la lógica de presentación de errores en la plantilla y limpiar las tareas de Ansible.

### Lista de Tareas Candidatas para Refactorizar

*   **02_subscription.yml**:
    *   `Version sugerida por redhat`
    *   `subscripcion`
*   **03_repos.yml**:
    *   `Repositorios disponibles`
    *   `release en satellite`
*   **05_updates.yml**:
    *   `paquetes obsoletos o sin reclamo`
*   **06_red.yml**:
    *   `rutas ip r`
    *   `rutas route`
    *   `rutas all`
    *   `rutas nmcli`
    *   `ficheros del red`
    *   `master slave`
    *   `ficheros de red ifcfg`
    *   `Ficheros de bond`
    *   `Ficheros ifcfg`
    *   `revisa bootproto ifcfg-bond1`
*   **07_servicios.yml**:
    *   `conectividad COMMVAULT-NG`
    *   `conectividad AAP`
    *   `synchronized`
    *   `ntpd`
    *   `chronyd`
    *   `Configuracion commvault`
    *   `status de RHC`
    *   `Fichero HOSTS` y `revisa hosts`:
        *   **Estado**: ✅ Completado.
        *   **Descripción**: Se usaban dos tareas con `shell` para leer `/etc/hosts`.
        *   **Solución Aplicada**:
            1.  Se reemplazaron las dos tareas por una sola que usa el módulo `ansible.builtin.slurp` para leer `/etc/hosts` y registrar el resultado en `slurp_etc_hosts`.
            2.  Se movió toda la lógica de presentación y validación a la plantilla `cabecera.html.j2`, que ahora decodifica el contenido, simula `cat -A` y comprueba la existencia de caracteres `\r` para mostrar una advertencia.
        *   **Beneficio**: Lógica de presentación centralizada en la plantilla, eliminación de `shell` y uso de un módulo nativo más eficiente.
*   **08_seguridad.yml**:
    *   `Fichero FSTAB` y `revisa fstab`:
        *   **Estado**: ✅ Completado.
        *   **Descripción**: Se usaban dos tareas con `shell`: una para leer `/etc/fstab` con `cat -A` y otra para buscar errores en esa salida con `grep`.
        *   **Solución Aplicada**:
            1.  Se han reemplazado las dos tareas por una que usa `ansible.builtin.slurp` para leer el fichero de forma eficiente.
            2.  Se ha añadido una segunda tarea con `ansible.builtin.set_fact` para decodificar el contenido y almacenarlo en la variable `etcfstab.stdout`, manteniendo la compatibilidad con la plantilla.
            3.  Se ha movido la lógica de validación a la plantilla `cabecera.html.j2`, que ahora busca caracteres problemáticos (`\t`, `^`) en el contenido y muestra una advertencia o un mensaje de éxito.
        *   **Beneficio**: Se ha eliminado el uso de `shell`, la lógica de presentación se ha centralizado en la plantilla y el proceso es más eficiente al leer el fichero una sola vez.
    *   `Fichero config`
    *   `configuracion de dominio`
*   **10_discos.yml**:
    *   `MONTAJE DISCOS`:
        *   **Estado**: En progreso.
        *   **Descripción**: La tarea `MONTAJE DISCOS` ha sido duplicada. La versión `(new)` utiliza `ansible_facts` y la `(old)` mantiene el `shell`.
        *   **Acción pendiente**: Tras validar el informe con la nueva implementación, se debe eliminar la tarea `MONTAJE DISCOS (old)` y la sección correspondiente en la plantilla.
    *   `uso_disco`:
        *   **Estado**: 📝 Planificado.
        *   **Descripción**: La tarea `Obtener uso del disco (filtrado)` ejecuta `df -h`, lo cual es redundante ya que Ansible provee esta información en la variable `ansible_mounts`. La plantilla, además, contiene una lógica compleja y frágil para interpretar la salida de texto de `df -h`.
        *   **Solución Propuesta**:
            1.  **Eliminar la tarea**: Borrar la tarea `Obtener uso del disco (filtrado)` del fichero `roles/sgadprevio/tasks/10_discos.yml`.
            2.  **Refactorizar la plantilla**: Actualizar la sección "OCUPACIÓN DE DISCOS" en `cabecera.html.j2` para que itere directamente sobre la lista `ansible_mounts`. Esto permite un acceso directo y fiable a los datos (espacio total, disponible, etc.) y simplifica enormemente el código de la plantilla.
                ```jinja
                {# Ejemplo de la nueva lógica en la plantilla #}
                {% for mount in ansible_mounts %}
                  {% if mount.device != 'tmpfs' and mount.device != 'devtmpfs' %}
                  <tr>
                      <td>{{ mount.mount }}</td>
                      <td style="color: {% if mount.size_total > 0 and (mount.size_available / mount.size_total * 100) < 20 %}red{% elif mount.size_total > 0 and (mount.size_available / mount.size_total * 100) < 40 %}orange{% else %}inherit{% endif %}; font-weight: bold;">
                          {{ '%.2f'|format((1 - mount.size_available / mount.size_total) * 100) }}%
                      </td>
                      <td>{{ '%.2f'|format(mount.size_total / 1024/1024/1024) }} GB</td>
                      <td>{{ '%.2f'|format(mount.size_available / 1024/1024/1024) }} GB</td>
                  </tr>
                  {% endif %}
                {% endfor %}
                ```
        *   **Beneficio**: Elimina una tarea redundante, reduce la complejidad de la plantilla, aumenta la fiabilidad de los datos y mejora el rendimiento al no ejecutar un comando innecesario.
*   **11_enriquecidos.yml**:
    *   `procesos de aplicacion`
    *   `log de aplicacion`
