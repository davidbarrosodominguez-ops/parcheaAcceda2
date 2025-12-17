# Guía de Contexto Técnico del Proyecto "parcheaAcceda2"

Este documento proporciona el contexto técnico, las convenciones de código y los objetivos de refactorización para este proyecto.

## 1. Objetivo del Proyecto

El proyecto `parcheaAcceda2` es una herramienta de automatización con **Ansible** diseñada para realizar **auditorías previas al parcheado de servidores RHEL**. El rol recolecta datos del sistema y genera un informe para ser revisado por el equipo responsable.

La carpeta `specs/` documenta las reglas de negocio que determinan qué condiciones generan una alerta `REVISAR_...` en el informe.

## 2. Estructura del Proyecto

- `sgadprevio.yml`: Playbook principal que invoca al rol.
- `roles/sgadprevio/`: Contiene toda la lógica de Ansible.
    - `tasks/`: Lógica de ejecución. `main.yml` incluye secuencialmente otros ficheros de tareas.
    - `vars/`: Variables del rol. `all_vars.yml` centraliza la mayoría de las variables.
- `specs/`: Documentación de las reglas de negocio para la revisión de informes.
- `README.md`: Documentación principal orientada a humanos.
- `GEMINI.md`: (Este fichero) Guía de contexto técnico y objetivos.

## 3. Flujo de Trabajo y Convenciones de Código

Al modificar o añadir funcionalidades, es mandatorio seguir estas reglas:

### Tareas de Ansible
- **Secuencia**: La lógica de ejecución está en `roles/sgadprevio/tasks/main.yml`, que incluye ficheros de tareas numerados. Para añadir una nueva etapa de recolección, crea un nuevo fichero YAML numerado y añádelo en la secuencia correcta en `main.yml`.
- **Idempotencia**: Todas las tareas deben ser idempotentes. Utiliza los módulos de Ansible de forma que no realicen cambios si no es necesario (p.ej. `command` con `creates`/`removes`, `lineinfile`, etc.).
- **Nomenclatura**: Los nombres de las tareas deben ser descriptivos y en inglés o español, siguiendo el estilo existente.

### Variables
- **Definición**: Las variables deben definirse en `roles/sgadprevio/vars/all_vars.yml`.
- **Nomenclatura**: Utiliza `lowercase_with_underscores` (ej: `subscription_release`, `hay_updates`).
- **Registro de Datos**: Utiliza `register` para capturar la salida de los comandos y guárdala en una variable con un nombre descriptivo. Inicializa estas variables con un valor por defecto (string vacío o `false`) en `vars/all_vars.yml` para evitar errores de "variable not defined".

### Generación de Informes
- **Ubicación**: La lógica de generación de informes (HTML, CSV) y el envío de email residen íntegramente en `roles/sgadprevio/tasks/12_report.yml`.
- **Método**: Se utiliza una serie de tareas con el módulo `shell` que construyen los ficheros y el email manualmente a base de comandos `echo`, `sed` y `cat`. Este enfoque no utiliza el sistema de plantillas de Ansible.

El informe generado debe recopilar, como mínimo, la siguiente información (según la Guía Operativa):
1. Versión de sistema operativo antes de la actualización.
2. Versión de kernel actual antes de la actualización.
3. Versión recomendada por RedHat.
4. Repositorios activos.
5. Paquetes disponibles pendientes de actualizar.
6. Últimos paquetes instalados y la fecha de instalación.
7. Histórico de actualizaciones.
8. Parches de seguridad instalados (RHBA, RHSA o RHEA).
9. Paquetes de seguridad que pueden instalarse.
10. Paquetes de seguridad que pueden ser instalados en la actualización.
11. Rutas, servicios activos y fallidos.
12. Fechas de parcheado previstas.
13. Contenido de ficheros hosts, fstab.
14. Ocupación, identificación y montaje de los discos.
15. Usuarios de sistema, usuarios sudo.
16. Datos de contacto de los responsables del sistema y aplicación.
17. Estado del servicio de sincronización horaria.


## 4. Arquitectura: Separación de Lógica y Presentación

Es un principio fundamental de este proyecto separar la lógica de la presentación:

- **Tareas de Ansible (*.yml)**: Su única responsabilidad es ejecutar comandos, recopilar datos y registrar los resultados en variables (usando `register` o `set_fact`). Las tareas no deben generar HTML ni mensajes de cara al usuario final.

- **Plantillas (*.j2)**: Su única responsabilidad es la presentación. Leen las variables registradas por las tareas y las usan para generar el informe HTML final, incluyendo los mensajes de error o advertencia (`REVISAR_...`).

**Regla práctica**: Si modificas una tarea en un fichero `.yml` que afecta a la información del informe, debes revisar y, si es necesario, actualizar el fichero `.j2` correspondiente para asegurar que la representación gráfica sea coherente.

## 5. Objetivo de Refactorización (Rol de Gemini)

Según la última instrucción, el rol principal de Gemini en este proyecto es asistir en la **refactorización y mejora** del código. Los objetivos son:

-   Revisar las tareas (`tasks/`) para simplificarlas y hacerlas más mantenibles.
-   Reintroducir el uso de plantillas Jinja2 (`.j2`) para la generación de informes, eliminando la lógica de `shell` compleja donde sea posible.
-   Asegurar que el código resultante sea idempotente, declarativo y siga las mejores prácticas de Ansible.

### Flujo de Trabajo para Propuestas

Todas las propuestas de refactorización que genere Gemini se documentarán para su revisión en los siguientes ficheros:
- `docs/es/refactoring_opportunities.md` (en español)
- `docs/en/refactoring_opportunities.md` (en inglés)
