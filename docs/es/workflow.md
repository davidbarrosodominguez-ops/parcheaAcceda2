# Flujo de Trabajo del Playbook `parcheaAcceda2`

Este documento describe el flujo de ejecución del playbook de auditoría, desde el inicio hasta la generación y notificación del informe.

## Proceso General
El proceso se ejecuta de forma centralizada y delega ciertas acciones a un nodo específico para la consolidación de los informes.

## Diagrama de Flujo
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

## Pasos Detallados
1.  **Ejecución**: El usuario lanza el playbook `sgadprevio.yml` desde el nodo de control de Ansible.
2.  **Recopilación**: El rol `sgadprevio` ejecuta una serie de tareas de recopilación de datos en los hosts RHEL destino. Estas tareas cubren desde la subscripción y repositorios hasta la configuración de red, seguridad y hardware.
3.  **Delegación y Ensamblaje**: Los datos se procesan y se envían al nodo delegado (`adgesasateinfc2`), donde la tarea `12_report.yml` se encarga de renderizar la plantilla `cabecera.html.j2` para generar el informe HTML final y un resumen en formato CSV.
4.  **Notificación**: Se utiliza `sendmail` en el nodo delegado para enviar un correo electrónico con el informe adjunto a los destinatarios configurados.
5.  **Limpieza**: La tarea final, `99_postchecks.yml`, elimina los ficheros temporales utilizados durante el proceso en los hosts destino.
