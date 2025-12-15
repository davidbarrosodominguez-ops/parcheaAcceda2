# Spec de Seguridad: Reglas para la Revisión Manual

Este documento define las reglas de negocio que el rol de Ansible utiliza para marcar puntos de atención de seguridad en el informe. Cualquier condición que cumpla estas reglas se marcará en el informe con el formato `REVISAR_..._REVISAR`.

## Reglas de Auditoría de Seguridad

| Área | Condición de Seguridad | Lógica de Activación en el Rol | Ejemplo de Mensaje de Revisión |
| :--- | :--- | :--- | :--- |
| **Firewall** | El servicio `firewalld` o `iptables` no está activo. | La tarea en `08_seguridad.yml` comprueba el estado de estos servicios. | `REVISAR_Firewall inactivo. El servidor puede estar expuesto._REVISAR` |
| **Puertos** | Existen puertos abiertos comúnmente inseguros (ej: 21/ftp, 23/telnet). | La tarea en `06_red.yml` o `08_seguridad.yml` cruza los puertos abiertos con una lista negra. | `REVISAR_Puerto inseguro abierto: 23/tcp (telnet)._REVISAR` |
| **Acceso SSH** | Se permite el acceso directo del usuario `root` por SSH. | La tarea en `08_seguridad.yml` inspecciona la configuración de `sshd_config`. | `REVISAR_Permitido el login de root por SSH, mala práctica de seguridad._REVISAR` |
| **Permisos** | El fichero `/etc/shadow` tiene permisos inseguros. | La tarea en `08_seguridad.yml` comprueba los permisos de este fichero. | `REVISAR_Permisos inseguros detectados en /etc/shadow._REVISAR` |