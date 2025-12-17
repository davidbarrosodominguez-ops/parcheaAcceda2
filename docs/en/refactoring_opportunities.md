# Refactoring Opportunities

This document details areas for improvement identified in the current code to facilitate maintenance, readability, and robustness.

## 1. Eliminating Duplication
**Issue**: There is almost total duplication between the `sgadprevio` role and the `playbooks/rhel_SGADprevioAcceda2.yml` playbook.
**Recommended Action**: Delete or archive `playbooks/rhel_SGADprevioAcceda2.yml`. Use `sgadprevio.yml` exclusively, invoking the role.

## 2. Path and Variable Management
**Issue**: There are multiple paths and filenames defined directly in the task files and the main playbook, which makes it difficult to adapt the role to different environments.

**Specific Examples:**
- **Main Playbook (`sgadprevio.yml`):** The paths for temporary files and the final report are hardcoded (e.g., `/home/reexus/Acceda2/SGADprevio/rhel_previo_{{...}}.html`).
- **Security Task (`08_seguridad.yml`):** The path to find the application's configuration is fixed to a specific Wildfly version: `/opt/wildfly-10.1.0.Final/standalone/configuration/`.
- **Enrichment Tasks (`11_enriquecidos.yml`):** The paths to the enrichment files (`paraenriquecerParchea.lst`) and the logo (`LOGO_GOB_MTDFP_AEAD.png`) are absolute and depend on the `reexus` user.

**Recommended Action**:
- **Centralize variables:** Move all these paths to the role's variables file (`roles/sgadprevio/vars/all_vars.yml`).
- **Create variables for:**
  - `report_base_path`: Base directory for reports.
  - `app_config_path`: Path to the application's configuration directory.
  - `enrichment_file_path`: Path to the enrichment file.
  - `logo_file_path`: Path to the logo file.
- This would allow a user to easily override these paths from the inventory or an extra variables file, making the role much more reusable.

## 3. Ansible Modules vs Shell Usage
**Issue**: Extensive use of `shell` and `command` modules with text processing (`grep`, `awk`, etc.) to obtain information that native Ansible modules or facts already provide in a more reliable and structured way.

**Specific Examples:**
- **Get service status (in `07_servicios.yml`):** The `serviciosrunning` and `serviciosFallidos` tasks execute `systemctl` and parse its output. This could be replaced by the `service_facts` module, which returns a structured list of all services and their state, eliminating the need for text processing.
- **List updates (in `05_updates.yml`):** The `Lis` task runs `yum list updates`. The `yum` module itself can be used with the `list=updates` parameter to get this information programmatically and more safely.
- **Check disk space (in `10_discos.yml`):** The `uso_disco` task runs `df -h`. Ansible already gathers this information automatically during `gather_facts` and stores it in the `ansible_mounts` variable. Using this fact is more efficient than running a new command.
- **Certificate checking (in `08_seguridad.yml`):** The `find` command with `openssl` to check certificates could be replaced by the `community.crypto.x509_certificate_info` module, which retrieves the information in a structured manner.

**Recommended Action**:
- Replace `shell` commands with native Ansible modules (`service_facts`, `yum`, `package_facts`, etc.) whenever possible.
- Leverage Ansible facts (`ansible_mounts`, `ansible_default_ipv4`, etc.) instead of re-running commands that obtain the same information.
- **Benefit**: This will make the role more idempotent, readable, faster (by reducing command executions), and less prone to breaking if the text format of a Linux command changes between versions.

## 4. Error Handling
**Issue**: The `ignore_errors: True` parameter is overused (over 40 instances) to prevent the playbook from failing. This practice can mask real problems (e.g., a command does not exist, a service fails for an unexpected reason) and makes debugging difficult.

**Specific Examples:**
- **File Checking (in `08_seguridad.yml`):** The "Fichero config" task runs an `ls` and, if it fails, prints an `echo` with HTML. It would be more robust to use the `stat` module to verify if the path exists. If it doesn't, a variable can be registered and used in the Jinja2 template to display the error, instead of generating HTML within the task itself.
- **Connectivity Tests (in `07_servicios.yml`):** The tasks that use `curl` to check connectivity simply ignore failures. It is preferable to use the `uri` module, which allows for more granular control over the result (e.g., `failed_when: my_result.status_code != 200`) and does not depend on the `curl` command being installed.
- **Data Collection (in `09_hardware.yml`):** The calls to `dmidecode` ignore errors. If `dmidecode` were not installed, the variables would be empty without any notification of the root cause. A better practice would be to check if the command exists first, or use a `block/rescue` to catch the error and register a clear message that can be displayed in the report.

**Recommended Action**:
- Limit the use of `ignore_errors: True` only to situations where a failure is fully expected and not relevant.
- Use `failed_when` to define explicit failure conditions based on the return code or output of a command.
- Use `block/rescue/always` blocks to manage errors in a controlled way, allowing cleanup tasks to run or specific error messages to be registered.
- Use modules like `stat` to check preconditions (e.g., if a file exists) before running a command that depends on it.

## 6. HTML Generation
**Issue**: In several tasks, HTML code is generated directly from the `shell` using `echo`, especially for displaying error messages. This mixes data collection logic with presentation logic.

**Specific Example (from `08_seguridad.yml`):**
```yaml
- name: Fichero config
  shell: ls ... || echo -e '<p style=color:red;>...NO CONFIGURATION...</p>'
  register: etcconfigini
  ignore_errors: True
```
In this case, the Ansible task is responsible for generating an HTML snippet. If you wanted to change the error style (e.g., use a CSS class instead of `style=color:red`), you would have to modify the Ansible task code, not the template.

**Recommended Action**:
- **Separate Logic and Presentation**: Ansible tasks should only collect data and register variables (e.g., `etcconfigini_found: false`).
- **Move Presentation Logic to the Template**: The `cabecera.html.j2` template should be solely responsible for generating HTML. It should contain the logic to display the error based on the variables registered by the tasks.
  ```jinja
  {% if not etcconfigini_found %}
    <p class="error-message">⚠️ NO CONFIGURATION for Fichero config.</p>
  {% endif %}
  ```
- **Benefit**: This follows the principle of separation of concerns, making both the Ansible role and the HTML template much easier to maintain and modify independently.

## 7. Implement Insecure Port Check
**Problem**: The specification `specs/REGLAS_SEGURIDAD.md` defines a rule to detect insecure open ports (e.g., 21/ftp, 23/telnet). However, this check is not currently implemented in any task, which creates an inconsistency between the documentation and the actual audit.

**Recommended Action**:
- Add a new task in the `roles/sgadprevio/tasks/08_seguridad.yml` file.
- This task should use a command like `ss -tlpn` to list listening TCP ports.
- The port list should be compared against a blacklist of insecure ports (initially: 21, 23, 80, 110, 143).
- If a match is found, it should register a result that the template can use to display a `REVISAR_..._REVISAR` message in the final report.
- **Benefit**: Increases the security audit's coverage and aligns the role's implementation with its specifications.

## 8. Refactor Error Generation to the Template
**Problem**: Following the principle of point 6, many tasks that check states and can fail generate the `REVISAR_` message directly in the `shell`. This couples the task's logic to the error's presentation.

**Refactoring Example**:
The `cabecera.html.j2` template is already set up to handle this logic. For example, it checks if a variable has content and, if not, displays an error.

*   **Case Study**: Task `configuracion de dominio` in `08_seguridad.yml`.

*   **🔴 BEFORE (Current Code)**:
    ```yaml
    - name: configuracion de dominio
      shell: realm list |grep ':' || echo -e '<p style=color:red;>⚠️ ⚠️ REVISAR_⚠️ ⚠️ SIN CONFIGURACIÓN REALMD _REVISAR</p>'
      register: realmlist
    ```

*   **🟢 AFTER (Proposed Solution)**:
    1.  **Simplify the Task**: The task only gathers the data and doesn't worry about the error.
        ```yaml
        - name: configuracion de dominio
          shell: realm list | grep ':'
          register: realmlist
          ignore_errors: True
          changed_when: false
        ```
    2.  **Move Logic to the Template (`.j2`)**: The template checks the variable and displays the error if necessary.
        ```jinja
        <details>
            <summary>DOMAIN DATA</summary>
            {% if realmlist.stdout | trim %}
                <p>{{ realmlist.stdout_lines | join('<br>') }}</p>
            {% else %}
                <p style="color:red;">REVISAR_⚠️ ⚠️ NO DOMAIN CONFIGURATION (REALMD) FOUND_REVISAR</p>
            {% endif %}
        </details>
        ```

**Recommended Action**: Apply this pattern to all candidate tasks to centralize error presentation logic in the template and clean up the Ansible tasks.

### List of Candidate Tasks for Refactoring

*   **02_subscription.yml**:
    *   `Version sugerida por redhat`
    *   `subscripcion`
*   **03_repos.yml**:
    *   `Repositorios disponibles`
    *   `release en satellite`
*   **05_updates.yml**:
    *   `paquetes obsoletos o sin reclamo`
*   **06_red.yml**:
    *   `rutas ip r`
    *   `rutas route`
    *   `rutas all`
    *   `rutas nmcli`
    *   `ficheros del red`
    *   `master slave`
    *   `ficheros de red ifcfg`
    *   `Ficheros de bond`
    *   `Ficheros ifcfg`
    *   `revisa bootproto ifcfg-bond1`
*   **07_servicios.yml**:
    *   `conectividad COMMVAULT-NG`
    *   `conectividad AAP`
    *   `synchronized`
    *   `ntpd`
    *   `chronyd`
    *   `Configuracion commvault`
    *   `status de RHC`
*   **08_seguridad.yml**:
    *   `Fichero config`
    *   `configuracion de dominio`
*   **10_discos.yml**:
    *   `MONTAJE DISCOS`:
        *   **Status**: In progress.
        *   **Description**: The `MONTAJE DISCOS` task has been duplicated. The `(new)` version uses `ansible_facts`, and the `(old)` version keeps using `shell`.
        *   **Pending action**: After validating the report with the new implementation, the `MONTAJE DISCOS (old)` task and its corresponding section in the template must be deleted.
*   **11_enriquecidos.yml**:
    *   `procesos de aplicacion`
    *   `log de aplicacion`
