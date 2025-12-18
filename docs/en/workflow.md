# `parcheaAcceda2` Playbook Workflow

This document describes the execution flow of the audit playbook, from initiation to report generation and notification.

## General Process
The process is executed centrally and delegates certain actions to a specific node for report consolidation.

## Flowchart
```mermaid
graph TD
    A[User runs playbook<br>`ansible-playbook sgadprevio.yml`] --> B{Ansible Host};
    B --> C{{Target RHEL Host(s)}};
    subgraph "Task Execution on Target Host"
        direction LR
        C1[01_prechecks] --> C2[02_subscription] --> C3[03_repos] --> C4[04_historial] --> C5[05_updates]
        C5 --> C6[06_red] --> C7[07_servicios] --> C8[08_seguridad] --> C9[09_hardware] --> C10[10_discos]
        C10 --> C11[11_enriquecidos]
    end
    C --> D{Delegate Node<br>`adgesasateinfc2`};
    subgraph "Report Generation on Delegate Node"
        D1[12_report<br>Renders HTML and CSV] --> D2[Send email<br>`sendmail`]
    end
    C11 --> D1
    B --> D
    D2 --> E[Cleanup<br>99_postchecks];
```

## Detailed Steps
1.  **Execution**: The user runs the `sgadprevio.yml` playbook from the Ansible control node.
2.  **Collection**: The `sgadprevio` role executes a series of data collection tasks on the target RHEL hosts. These tasks cover everything from subscription and repositories to network configuration, security, and hardware.
3.  **Delegation and Assembly**: The data is processed and sent to the delegate node (`adgesasateinfc2`), where the `12_report.yml` task is responsible for rendering the `cabecera.html.j2` template to generate the final HTML report and a CSV summary.
4.  **Notification**: `sendmail` is used on the delegate node to send an email with the attached report to the configured recipients.
5.  **Cleanup**: The final task, `99_postchecks.yml`, removes the temporary files used during the process on the target hosts.
