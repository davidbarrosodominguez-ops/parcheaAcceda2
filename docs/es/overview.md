# Visión General del Proyecto `parcheaAcceda2`

## Introducción
Este proyecto implementa una auditoría automática previa al parcheado ("Health Check") para servidores Red Hat Enterprise Linux (RHEL). Su objetivo es recopilar el estado actual del sistema, verificar prerequisitos de actualización y generar informes detallados para los administradores.

## Estructura del Directorio
El proyecto sigue una estructura estándar de Ansible, aunque con algunas particularidades:

- **`roles/`**: Contiene la lógica principal encapsulada en roles.
  - **`sgadprevio/`**: El rol principal que realiza todas las comprobaciones.
- **`playbooks/`**: Contiene playbooks que orquestan la ejecución.

- **`sgadprevio.yml`**: Un playbook "wrapper" o envoltorio que llama al rol `sgadprevio`.
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
