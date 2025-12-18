# Proyecto parcheaAcceda2

## Visión General
Este proyecto implementa una auditoría automática previa al parcheado ("Health Check") para servidores Red Hat Enterprise Linux (RHEL). Su objetivo es recopilar el estado actual del sistema, verificar los prerequisitos para la actualización y generar informes detallados para los administradores del sistema.

Utiliza Ansible para automatizar la recopilación de datos y la generación de un informe final en formatos HTML y CSV.

## Flujo de Trabajo
El proceso se ejecuta de forma centralizada y delega ciertas acciones a un nodo específico para la consolidación de los informes.

```mermaid
graph TD
    A[Usuario ejecuta playbook<br>`ansible-playbook sgadprevio.yml`] --> B{Host Ansible};
    B --> C{{Target RHEL Host(s)}};
    subgraph "Ejecución de Tareas en Target Host"
        direction LR
        C1[01_prechecks] --> C2[02_subscription] --> C3[03_repos] --> C4[04_historial] --> C5[05_updates]
        C5 --> C6[06_red] --> C7[07_servicios] --> C8[08_seguridad] --> C9[09_hardware] --> C10[10_discos]
        C10 --> C11[11_enriquecidos]
    end
    C --> D{Nodo Delegado<br>`adgesasateinfc2`};
    subgraph "Generación de Informe en Nodo Delegado"
        D1[12_report<br>Renderiza HTML y CSV] --> D2[Envía email<br>`sendmail`]
    end
    C11 --> D1
    B --> D
    D2 --> E[Limpieza<br>99_postchecks];
```

1.  **Ejecución**: El usuario lanza el playbook `sgadprevio.yml` desde el nodo de control de Ansible.
2.  **Recopilación**: El rol `sgadprevio` ejecuta una serie de tareas de recopilación de datos en los hosts RHEL destino.
3.  **Delegación y Ensamblaje**: Los datos se procesan y se envían al nodo delegado (`adgesasateinfc2`), donde se genera el informe HTML y el resumen CSV.
4.  **Notificación**: Se envía un correo electrónico con el informe adjunto desde el nodo delegado.
5.  **Limpieza**: Se eliminan los ficheros temporales de los hosts.

## Desglose de Tareas del Rol `sgadprevio`
La lógica principal reside en el rol `sgadprevio` y está dividida en las siguientes tareas:

- **`01_prechecks.yml`**: Crea la estructura de directorios y ficheros temporales.
- **`02_subscription.yml`**: Verifica el estado de la suscripción de Red Hat y las versiones de `insights-client`.
- **`03_repos.yml`**: Lista los repositorios de software habilitados.
- **`04_historial.yml`**: Recopila el historial de actualizaciones y paquetes instalados (`yum/rpm`).
- **`05_updates.yml`**: Lista en detalle las actualizaciones pendientes y parches de seguridad.
- **`06_red.yml`**: Analiza la configuración de red (rutas, interfaces, bonding, iptables).
- **`07_servicios.yml`**: Comprueba servicios (`systemctl`), conectividad y sincronización horaria (NTP).
- **`08_seguridad.yml`**: Revisa certificados, usuarios, `sudoers` y `sssd`.
- **`09_hardware.yml`**: Recopila información de hardware vía `dmidecode`.
- **`10_discos.yml`**: Analiza el uso del sistema de ficheros y puntos de montaje.
- **`11_enriquecidos.yml`**: Cruza datos del host con ficheros externos para añadir información de contacto y aplicación.
- **`12_report.yml`**: Genera los informes HTML/CSV y envía el correo. **(Se ejecuta en el nodo delegado)**.
- **`99_postchecks.yml`**: Realiza tareas de limpieza post-ejecución.

## Configuración y Dependencias
- **Usuario de Ejecución**: El playbook se ejecuta con el usuario `reexus` en los hosts remotos.
- **Nodo Delegado**: `adgesasateinfc2` debe ser accesible desde el nodo de control Ansible. Se usa para centralizar la generación de informes y el envío de correos.
- **Ficheros Externos**:
  - `/home/reexus/enriquece/paraenriquecerParchea.lst`: Lista para enriquecer datos.
  - `/home/reexus/LOGO_GOB_MTDFP_AEAD.png`: Logotipo para el informe HTML.
- **Rutas Hardcodeadas**: La tarea que busca ficheros de configuración de aplicación en `08_seguridad.yml` contiene una ruta hardcodeada a una versión específica de Wildfly (`/opt/wildfly-10.1.0.Final/...`). Esto puede requerir modificación si la versión o la ruta de la aplicación cambia.

## Cómo Ejecutar el Playbook
Para generar un informe completo, ejecuta el playbook `sgadprevio.yml` contra tu fichero de inventario.

```bash
ansible-playbook sgadprevio.yml -i your_inventory_file
```

## Salidas Generadas
- **Informe HTML**: Un fichero detallado (`rhel_previo_{{ ansible_hostname }}_{...}.html`) con toda la información recopilada. Se genera a partir de la plantilla `templates/cabecera.html.j2`.
- **Informe CSV**: Un fichero resumen con los puntos clave de la auditoría.

## Principio Arquitectónico: Separación de Lógica y Presentación

Un principio fundamental de este proyecto es separar la lógica de la presentación:

- **Tareas de Ansible (*.yml)**: Su única responsabilidad es ejecutar comandos, recopilar datos y registrar los resultados en variables (usando `register` o `set_fact`). Las tareas no deben generar HTML ni mensajes orientados al usuario.

- **Plantillas (*.j2)**: Su única responsabilidad es la presentación. Leen las variables registradas por las tareas y las utilizan para generar el informe HTML final, incluidos los mensajes de error o advertencia (`REVISAR_...`).

**Regla Práctica**: Si modifica una tarea en un archivo `.yml` que afecta la información del informe, debe revisar y, si es necesario, actualizar el archivo `.j2` correspondiente para garantizar una representación gráfica coherente.
