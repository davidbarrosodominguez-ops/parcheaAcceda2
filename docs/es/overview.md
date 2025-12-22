# Visión General del Proyecto `parcheaAcceda2`

## Introducción
Este proyecto implementa una auditoría automática previa al parcheado ("Health Check") para servidores Red Hat Enterprise Linux (RHEL). Su objetivo es recopilar el estado actual del sistema, verificar prerequisitos de actualización y generar informes detallados para los administradores.

## Modelo Arquitectónico
Este proyecto encaja perfectamente en el modelo de **Infraestructura como Código (IaC)**, y también se puede clasificar dentro de los siguientes patrones:

*   **Gestión de la Configuración:** Utiliza Ansible para definir y mantener el estado deseado de los sistemas.
*   **Automatización de TI:** Automatiza tareas repetitivas de auditoría y reporte que tradicionalmente haría un administrador de sistemas.
*   **Cumplimiento como Código (Compliance as Code):** Las reglas de auditoría se implementan en código (playbooks de Ansible) para verificar automáticamente el cumplimiento de las políticas de seguridad y configuración.

Es un ejemplo clásico de cómo usar la automatización para garantizar que la infraestructura cumple con un estándar definido.

## Estructura del Directorio
El proyecto sigue una estructura estándar de Ansible, aunque con algunas particularidades:

- **`roles/`**: Contiene la lógica principal encapsulada en roles.
  - **`sgadprevio/`**: El rol principal que realiza todas las comprobaciones.
- **`sgadprevio.yml`**: El playbook principal en la raíz del proyecto que llama al rol `sgadprevio`.
- **`docs/`**: Documentación del proyecto (este directorio).

## Flujo de Trabajo
1. El playbook se ejecuta contra un inventario de hosts.
2. Se realizan comprobaciones secuenciales (suscripción, repositorios, actualizaciones, red, servicios, seguridad, hardware, discos).
3. Se genera un informe en formato HTML y CSV en el nodo delegado (`adgesasateinfc2`).
4. Se envía un correo electrónico con los resultados.
5. Se limpian los ficheros temporales.

## Dependencias Clave
- **Nodo Delegado**: `adgesasateinfc2` se utiliza para centralizar la generación de informes y el envío de correos.
- **Ficheros Externos**:
  - `/home/reexus/enriquece/paraenriquecerParchea.lst`: Lista para enriquecer datos (contacto, aplicación).
  - `/home/reexus/LOGO_GOB_MTDFP_AEAD.png`: Logo para el informe HTML.
- **Comandos del Sistema**:
  - `subscription-manager`
  - `insights-client`
  - `yum` / `dnf`
  - `sendmail`
