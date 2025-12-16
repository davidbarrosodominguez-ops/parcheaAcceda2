# Oportunidades de Refactorización

Este documento detalla áreas de mejora identificadas en el código actual para facilitar su mantenimiento, legibilidad y robustez.

## 1. Eliminación de Duplicación
**Problema**: Existe una duplicación casi total entre el rol `sgadprevio` y el playbook `playbooks/rhel_SGADprevioAcceda2.yml`.
**Acción Recomendada**: Eliminar o archivar `playbooks/rhel_SGADprevioAcceda2.yml`. Usar exclusivamente `sgadprevio.yml`, que invoca al rol.

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

## 5. Delegación
**Problema**: El nombre del host de delegación, `adgesasateinfc2`, está hardcodeado en más de 20 tareas a lo largo de múltiples ficheros, incluyendo `01_prechecks.yml`, `11_enriquecidos.yml`, `12_report.yml` y `13_postchecks.yml`.

**Acción Recomendada**:
- Crear una nueva variable en `roles/sgadprevio/vars/all_vars.yml`, por ejemplo: `reporting_host: adgesasateinfc2`.
- Reemplazar todas las instancias de `delegate_to: adgesasateinfc2` por `delegate_to: "{{ reporting_host }}"`.
- **Beneficio**: Esto permitiría cambiar el host de delegación de forma centralizada en un solo lugar, o incluso desde el inventario de Ansible, lo que incrementa drásticamente la flexibilidad y reutilización del rol en otros entornos.

## 6. Generación de HTML
**Problema**: En varias tareas se genera código HTML directamente desde el `shell` usando `echo`, especialmente para mostrar mensajes de error. Esto mezcla la lógica de recopilación de datos con la lógica de presentación.

**Ejemplo Específico (de `08_seguridad.yml`):**
```yaml
- name: Fichero config
  shell: ls ... || echo -e '<p style=color:red;>...SIN CONFIGURACIÓN...</p>'
  register: etcconfigini
  ignore_errors: True
```
En este caso, la tarea de Ansible es responsable de generar un fragmento de HTML. Si se quisiera cambiar el estilo del error (e.g., usar una clase CSS en vez de `style=color:red`), habría que modificar el código de la tarea de Ansible, no la plantilla.

**Acción Recomendada**:
- **Separar Lógica y Presentación**: Las tareas de Ansible solo deben recopilar datos y registrar variables (e.g., `etcconfigini_found: false`).
- **Mover la Lógica de Presentación a la Plantilla**: La plantilla `cabecera.html.j2` debe ser la única responsable de generar HTML. Debería contener la lógica para mostrar el error basándose en las variables que las tareas han registrado.
  ```jinja
  {% if not etcconfigini_found %}
    <p class="error-message">⚠️ SIN CONFIGURACIÓN de Fichero config.</p>
  {% endif %}
  ```
- **Beneficio**: Esto sigue el principio de separación de conceptos, haciendo que tanto el rol de Ansible como la plantilla HTML sean mucho más fáciles de mantener y modificar de forma independiente.
