# Hoja de Ruta: Implementación de Comprobaciones de Seguridad Avanzadas

## Objetivo
Este documento sirve como plan de trabajo para incorporar de manera incremental una serie de comprobaciones de seguridad avanzadas en el rol `sgadprevio`. Estas mejoras se basan en estándares de la industria como los **CIS Benchmarks**.

## Análisis Estratégico: ¿Implementación Manual o SCAP?
Antes de implementar controles específicos, es crucial decidir el enfoque estratégico.

### Opción 1: Implementación Manual (El enfoque de esta lista de tareas)
Consiste en traducir guías de seguridad (como CIS) en tareas de Ansible.
- **Pros**: Control total sobre la lógica y el informe.
- **Contras**: Gran esfuerzo manual, difícil de mantener y no es un escaneo "certificado".

### Opción 2: Orquestar Escaneos OpenSCAP (Enfoque Recomendado)
Consiste en usar Ansible para instalar y ejecutar el escáner `oscap` en los servidores, utilizando perfiles de seguridad estándar (ej. CIS, STIG).
- **Pros**: Rápido de implementar, usa contenido certificado, genera informes estándar reconocidos por auditores.
- **Recomendación**: **Este es el camino más robusto y profesional.** Proporciona una base técnica sólida, auditable y alineada con los estándares de la industria.

## Contexto Regulatorio: La Directiva NIS2
Es importante entender que NIS2 es un marco **legal**, no una checklist técnica. Exige a las organizaciones demostrar que gestionan activamente sus riesgos de ciberseguridad.

- **Este proyecto es la evidencia**: Todo el trabajo de bastionado (ya sea manual o vía SCAP) sirve como **prueba documental** para demostrar a los auditores que se están tomando las "medidas técnicas adecuadas" que requiere la directiva.

---

## Metodología Propuesta para Implementación Manual
Si se opta por la **Opción 1**, el proceso para cada grupo de comprobaciones es el siguiente:

1.  **Crear Fichero de Tareas Dedicado**: Se creará un nuevo fichero, por ejemplo `roles/sgadprevio/tasks/13_security_hardening.yml`, para alojar todas estas nuevas tareas.
2.  **Etiquetar las Tareas**: Todas las tareas en este nuevo fichero llevarán la etiqueta `security_hardening` para poder ejecutarlas de forma selectiva (`ansible-playbook sgadprevio.yml --tags security_hardening`).
3.  **Registrar Resultados (No Modificar)**: Las tareas no deben modificar el sistema. Su único propósito es **comprobar** el estado y **registrar** el resultado en una variable (usando `register:` y `check_mode: yes` cuando sea aplicable).
4.  **Actualizar el Informe**: Se modificará la tarea `12_report.yml` y la plantilla Jinja2 (`cabecera.html.j2`) para incluir una nueva sección en el informe HTML con los resultados de estas comprobaciones.
5.  **Actualizar `main.yml`**: Se añadirá la importación del nuevo fichero `13_security_hardening.yml` en `roles/sgadprevio/tasks/main.yml`, idealmente de forma condicional o etiquetada.

---

## Lista de Tareas de Implementación (Ejemplo de Enfoque Manual)

### Módulo 1: Bastionado del Sistema de Ficheros

- [ ] **Verificar permisos de ficheros de backup de credenciales**
    - **Tarea**: Comprobar que `/etc/passwd-` y `/etc/group-` tienen permisos `600` o más restrictivos.
    - **Módulos Ansible**: `ansible.builtin.file`.

- [ ] **Verificar opciones de montaje seguras para particiones temporales**
    - **Tarea**: Asegurar que `/tmp`, `/var/tmp` y `/dev/shm` se montan con `nodev`, `nosuid`, y `noexec`.
    - **Módulos Ansible**: Inspeccionar `ansible_facts.mounts`.

- [ ] **Detectar ficheros "world-writable"**
    - **Tarea**: Encontrar y listar ficheros con permisos `o+w` en directorios del sistema.
    - **Módulos Ansible**: `ansible.builtin.find`.

### Módulo 2: Configuración Segura de SSH

- [ ] **Auditar la configuración de `sshd_config`**
    - **Tarea**: Comprobar que `PermitRootLogin` es `no`, `Protocol` es `2`, y `MaxAuthTries` es bajo.
    - **Módulos Ansible**: `ansible.builtin.command` con `sshd -T` para obtener la configuración final del servicio.

### Módulo 3: Gestión de Cuentas y Privilegios

- [ ] **Detectar cuentas sin contraseña**
    - **Tarea**: Revisar `/etc/shadow` para encontrar usuarios con campos de contraseña vacíos o no seguros.
    - **Módulos Ansible**: `community.general.shadow` o `ansible.builtin.command` con `awk`.

- [ ] **Auditar política de `sudo`**
    - **Tarea**: Verificar que `/etc/sudoers` y los ficheros en `/etc/sudoers.d/` no contengan la directiva `NOPASSWD`.
    - **Módulos Ansible**: `ansible.builtin.find` y `ansible.builtin.slurp`.

### Módulo 4: Bastionado del Kernel (sysctl)

- [ ] **Verificar parámetros de red seguros**
    - **Tarea**: Asegurar que `net.ipv4.tcp_syncookies`, `net.ipv4.conf.all.accept_redirects`, etc., tienen valores seguros.
    - **Módulos Ansible**: `ansible.posix.sysctl`.
