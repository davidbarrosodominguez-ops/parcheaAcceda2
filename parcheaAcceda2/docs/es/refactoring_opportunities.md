# Oportunidades de Refactorización

Este documento detalla áreas de mejora identificadas en el código actual para facilitar su mantenimiento, legibilidad y robustez.



## 2. Gestión de Rutas y Variables
**Problema**: Hay muchas rutas absolutas "hardcoded" (e.g., `/home/reexus/...`, `/opt/...`).
**Acción Recomendada**:
- Mover todas las rutas a variables en `defaults/main.yml` o `vars/main.yml`.
- Usar rutas relativas o variables de inventario donde sea posible.

## 3. Uso de Módulos de Ansible vs Shell
**Problema**: Uso extensivo de los módulos `shell` y `command` con pipes complejos (`| grep | awk ...`) para tareas que Ansible puede manejar nativamente.
**Acción Recomendada**:
- Usar módulo `yum`/`package` para listar actualizaciones.
- Usar módulo `service`/`systemd_service` para comprobar servicios.
- Usar módulo `mount` para comprobar montajes.
- Esto hace el código más idempotente y menos frágil a cambios en la salida de comandos de Linux.

## 4. Gestión de Errores
**Problema**: Uso frecuente de `ignore_errors: True` para gestionar fallos en comandos `shell`. Esto puede enmascarar problemas reales.
**Acción Recomendada**: Implementar comprobaciones más robustas (`failed_when`, `changed_when`) y bloques `block/rescue` si es necesario.

## 5. Delegación
**Problema**: La delegación al host `adgesasateinfc2` está escrita directamente en las tareas (`delegate_to: adgesasateinfc2`).
**Acción Recomendada**: Definir el host de gestión como una variable (e.g., `delegate_to: "{{ reporting_host }}"`) para permitir cambiarlo fácilmente desde el inventario.

## 6. Generación de HTML
**Problema**: Parte del HTML se genera concatenando strings en comandos `shell` y `echo`.
**Acción Recomendada**: Mover toda la lógica de presentación a plantillas Jinja2 (`templates/`) para separar datos de presentación.
