# Guía de Estilo y Desarrollo para el Rol `sgadprevio`

Este documento define el estándar de código y las mejores prácticas para desarrollar y mantener este rol de Ansible.

## Principios Generales
- **Idempotencia:** Todas las tareas deben ser idempotentes. Una tarea ejecutada múltiples veces debe producir el mismo resultado sin errores.
- **Declarativo vs. Imperativo:** Prefiere siempre los módulos nativos de Ansible (declarativos) sobre el uso de `shell` o `command` (imperativos). El uso de `shell` debe ser la excepción, no la norma.
  - *Nota: La generación de informes en `12_report.yml` es una excepción conocida a refactorizar.*

## Ficheros de Tareas (`.yml`)
- **Nomenclatura:** Usa nombres de tarea (`name:`) claros y descriptivos en español.
- **Variables:** No incluyas valores fijos ("hardcode"). Usa siempre variables con el formato `{{ nombre_variable }}`. Las variables deben definirse en `vars/all_vars.yml`.

## Plantillas (`.j2`)
- **Lógica Mínima:** La lógica compleja debe residir en las tareas de Ansible, no en la plantilla. Las plantillas deben ser sencillas.
- **Seguridad:** Nunca incluyas credenciales o información sensible en texto plano en una plantilla.

## Documentación
- **Sincronización de Idiomas:** Toda la documentación generada (ficheros `.md`) debe mantenerse sincronizada en sus versiones en español e inglés. Cualquier cambio en un fichero debe ser replicado en su contraparte.
- **Coherencia Tarea-Plantilla:** Cuando se modifique la tarea de un fichero `.yml` que afecte a la información reportada, se debe verificar que la modificación no afecta a su representación gráfica en el `.j2` correspondiente. Si es necesario, la plantilla `.j2` también debe ser modificada para mantener la coherencia.