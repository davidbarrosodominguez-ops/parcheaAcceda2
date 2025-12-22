# `parcheaAcceda2` Project Overview

## Introduction
This project implements an automated pre-patching audit ("Health Check") for Red Hat Enterprise Linux (RHEL) servers. Its goal is to collect the current system status, verify update prerequisites, and generate detailed reports for administrators.

## Architectural Model
This project fits perfectly into the **Infrastructure as Code (IaC)** model, and can also be classified within the following patterns:

*   **Configuration Management:** It uses Ansible to define and maintain the desired state of the systems.
*   **IT Automation:** It automates repetitive audit and reporting tasks that a system administrator would traditionally perform.
*   **Compliance as Code:** The audit rules are implemented in code (Ansible playbooks) to automatically verify compliance with security and configuration policies.

It is a classic example of using automation to ensure that the infrastructure complies with a defined standard.

## Directory Structure
The project follows a standard Ansible structure with some specificities:

- **`roles/`**: Contains the main logic encapsulated in roles.
  - **`sgadprevio/`**: The main role that performs all checks.
- **`sgadprevio.yml`**: The main playbook in the project root that calls the `sgadprevio` role.
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
