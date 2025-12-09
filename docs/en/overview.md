# `parcheaAcceda2` Project Overview

## Introduction
This project implements an automated pre-patching audit ("Health Check") for Red Hat Enterprise Linux (RHEL) servers. Its goal is to collect the current system status, verify update prerequisites, and generate detailed reports for administrators.

## Directory Structure
The project follows a standard Ansible structure with some specificities:

- **`roles/`**: Contains the main logic encapsulated in roles.
  - **`sgadprevio/`**: The main role that performs all checks.
- **`playbooks/`**: Contains playbooks that orchestrate execution.
  - **`rhel_SGADprevioAcceda2.yml`**: A monolithic version of the audit logic.
- **`sgadprevio.yml`**: A wrapper playbook that calls the `sgadprevio` role.
- **`docs/`**: Project documentation (this directory).

## Workflow
1. The playbook is executed against an inventory of hosts.
2. Sequential checks are performed (subscription, repositories, updates, network, services, security, hardware, disks).
3. An HTML and CSV report is generated on the delegated node (`adgesasateinfc2`).
4. An email with the results is sent.
5. Temporary files are cleaned up.

## Key Dependencies
- **Delegated Node**: `adgesasateinfc2` is used to centralize report generation and email sending.
- **External Files**:
  - `/home/reexus/enriquece/paraenriquecerParchea.lst`: List for enriching data (contact, application).
  - `/home/reexus/LOGO_GOB_MTDFP_AEAD.png`: Logo for the HTML report.
- **System Commands**:
  - `subscription-manager`
  - `insights-client`
  - `yum` / `dnf`
  - `sendmail`
