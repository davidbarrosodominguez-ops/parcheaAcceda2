# parcheaAcceda2 Project

## Overview
This project implements an automatic pre-patching audit ("Health Check") for Red Hat Enterprise Linux (RHEL) servers. Its purpose is to collect the current state of the system, verify update prerequisites, and generate detailed reports for system administrators.

It uses Ansible to automate data collection and the generation of a final report in HTML and CSV formats.

## Workflow
The process runs centrally and delegates certain actions to a specific node for report consolidation.

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
    D2 --> E[Cleanup<br>13_postchecks];
```

1.  **Execution**: The user runs the `sgadprevio.yml` playbook from the Ansible control node.
2.  **Collection**: The `sgadprevio` role executes a series of data collection tasks on the target RHEL hosts.
3.  **Delegation and Assembly**: The data is processed and sent to the delegate node (`adgesasateinfc2`), where the HTML report and CSV summary are generated.
4.  **Notification**: An email with the attached report is sent from the delegate node.
5.  **Cleanup**: Temporary files are removed from the hosts.

## `sgadprevio` Role Task Breakdown
The main logic resides in the `sgadprevio` role and is divided into the following tasks:

- **`01_prechecks.yml`**: Creates the directory structure and temporary files.
- **`02_subscription.yml`**: Checks the status of the Red Hat subscription and `insights-client` versions.
- **`03_repos.yml`**: Lists enabled software repositories.
- **`04_historial.yml`**: Gathers the history of updates and installed packages (`yum/rpm`).
- **`05_updates.yml`**: Provides a detailed list of pending updates and security patches.
- **`06_red.yml`**: Analyzes the network configuration (routes, interfaces, bonding, iptables).
- **`07_servicios.yml`**: Checks services (`systemctl`), connectivity, and time synchronization (NTP).
- **`08_seguridad.yml`**: Reviews certificates, users, `sudoers`, and `sssd`.
- **`09_hardware.yml`**: Collects hardware information via `dmidecode`.
- **`10_discos.yml`**: Analyzes file system usage and mount points.
- **`11_enriquecidos.yml`**: Cross-references host data with external files to add contact and application information.
- **`12_report.yml`**: Generates the HTML/CSV reports and sends the email. **(Runs on the delegate node)**.
- **`13_postchecks.yml`**: Performs post-execution cleanup tasks.

## Architectural Principle: Separation of Logic and Presentation

A fundamental principle of this project is to separate logic from presentation:

- **Ansible Tasks (*.yml)**: Their sole responsibility is to execute commands, gather data, and register the results into variables (using `register` or `set_fact`). Tasks should not generate HTML or user-facing messages.

- **Templates (*.j2)**: Their sole responsibility is presentation. They read the variables registered by the tasks and use them to generate the final HTML report, including error or warning messages (`REVISAR_...`).

**Practical Rule**: If you modify a task in a `.yml` file that affects the information in the report, you must review and, if necessary, update the corresponding `.j2` file to ensure consistent graphical representation.


## Configuration and Dependencies
- **Execution User**: The playbook runs as the `reexus` user on remote hosts.
- **Delegate Node**: `adgesasateinfc2` must be accessible from the Ansible control node. It is used to centralize report generation and email dispatch.
- **External Files**:
  - `/home/reexus/enriquece/paraenriquecerParchea.lst`: A list used to enrich data.
  - `/home/reexus/LOGO_GOB_MTDFP_AEAD.png`: Logo for the HTML report.
- **Hardcoded Paths**: The task that checks for application configuration files in `08_seguridad.yml` contains a hardcoded path to a specific Wildfly version (`/opt/wildfly-10.1.0.Final/...`). This may need to be modified if the application version or path changes.

## How to Run the Playbook
To generate a full report, run the `sgadprevio.yml` playbook against your inventory file.

```bash
ansible-playbook sgadprevio.yml -i your_inventory_file
```

## Generated Outputs
- **HTML Report**: A detailed file (`rhel_previo_{{ ansible_hostname }}_{...}.html`) with all the collected information. It is generated from the `templates/cabecera.html.j2` template.
- **CSV Report**: A summary file with the key points of the audit.
