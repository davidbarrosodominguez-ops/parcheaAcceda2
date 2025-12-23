# Roadmap: Implementation of Advanced Security Checks

## Objective
This document serves as a work plan to incrementally incorporate a series of advanced security checks into the `sgadprevio` role. These improvements are based on industry standards like the **CIS Benchmarks**.

## Strategic Analysis: Manual Implementation vs. SCAP?
Before implementing specific controls, it's crucial to decide on the strategic approach.

### Option 1: Manual Implementation (The approach of this task list)
This consists of translating security guides (like CIS) into Ansible tasks.
- **Pros**: Full control over the logic and the final report.
- **Cons**: High manual effort, difficult to maintain, and it's not a "certified" scan.

### Option 2: Orchestrate OpenSCAP Scans (Recommended Approach)
This consists of using Ansible to install and run the `oscap` scanner on the servers, using standard security profiles (e.g., CIS, STIG).
- **Pros**: Quick to implement, uses certified content, generates standard reports recognized by auditors.
- **Recommendation**: **This is the most robust and professional path.** It provides a solid, auditable, and standards-aligned technical foundation.

## Regulatory Context: The NIS2 Directive
It's important to understand that NIS2 is a **legal framework**, not a technical checklist. It requires organizations to demonstrate that they are actively managing their cybersecurity risks.

- **This project is the evidence**: All the hardening work (whether manual or via SCAP) serves as **documentary proof** to show auditors that the organization is taking the "appropriate technical measures" required by the directive.

---

## Proposed Methodology for Manual Implementation
If **Option 1** is chosen, the process for each group of checks is as follows:

1.  **Create a Dedicated Task File**: A new file, for example `roles/sgadprevio/tasks/13_security_hardening.yml`, will be created to house all these new tasks.
2.  **Tag the Tasks**: All tasks in this new file will be tagged with `security_hardening` to allow for selective execution (`ansible-playbook sgadprevio.yml --tags security_hardening`).
3.  **Register Results (Do Not Modify)**: The tasks should not modify the system. Their sole purpose is to **check** the state and **register** the result in a variable (using `register:` and `check_mode: yes` where applicable).
4.  **Update the Report**: The `12_report.yml` task and the associated Jinja2 template (`cabecera.html.j2`) will be modified to include a new section in the HTML report with the results of these checks.
5.  **Update `main.yml`**: The import of the new `13_security_hardening.yml` file will be added to `roles/sgadprevio/tasks/main.yml`, ideally in a conditional or tagged manner.

---

## Implementation Task List (Example of Manual Approach)

### Module 1: Filesystem Hardening

- [ ] **Verify permissions of credential backup files**
    - **Task**: Check that `/etc/passwd-` and `/etc/group-` have permissions of `600` or more restrictive.
    - **Ansible Modules**: `ansible.builtin.file`.

- [ ] **Verify secure mount options for temporary partitions**
    - **Task**: Ensure that `/tmp`, `/var/tmp`, and `/dev/shm` are mounted with `nodev`, `nosuid`, and `noexec`.
    - **Ansible Modules**: Inspect `ansible_facts.mounts`.

- [ ] **Detect world-writable files**
    - **Task**: Find and list files with `o+w` permissions in system directories.
    - **Ansible Modules**: `ansible.builtin.find`.

### Module 2: Secure SSH Configuration

- [ ] **Audit `sshd_config` settings**
    - **Task**: Check that `PermitRootLogin` is `no`, `Protocol` is `2`, and `MaxAuthTries` is low.
    - **Ansible Modules**: `ansible.builtin.command` with `sshd -T` to get the final service configuration.

### Module 3: Account and Privilege Management

- [ ] **Detect accounts without passwords**
    - **Task**: Review `/etc/shadow` to find users with empty or insecure password fields.
    - **Ansible Modules**: `community.general.shadow` or `ansible.builtin.command` with `awk`.

- [ ] **Audit `sudo` policy**
    - **Task**: Verify that `/etc/sudoers` and files in `/etc/sudoers.d/` do not contain the `NOPASSWD` directive.
    - **Ansible Modules**: `ansible.builtin.find` and `ansible.builtin.slurp`.

### Module 4: Kernel Hardening (sysctl)

- [ ] **Verify secure network parameters**
    - **Task**: Ensure that `net.ipv4.tcp_syncookies`, `net.ipv4.conf.all.accept_redirects`, etc., have secure values.
    - **Ansible Modules**: `ansible.posix.sysctl`.
