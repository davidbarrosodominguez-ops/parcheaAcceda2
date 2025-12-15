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
