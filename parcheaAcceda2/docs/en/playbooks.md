# Playbooks Documentation

## `sgadprevio.yml`
This is the recommended entry point for executing the audit using the role structure.

**Function:**
- Defines target hosts (`hosts: all`).
- Enables privilege escalation (`become: yes`).
- Defines global file variables (`fichero`, `fich1`, etc.).
- Calls the `sgadprevio` role.



**Usage Example:**
```bash
ansible-playbook sgadprevio.yml -i inventory
```

## `playbooks/rhel_SGADprevioAcceda2.yml`
This playbook contains a monolithic definition of the audit.

**Analysis:**
- It appears to contain a "flattened" copy of all tasks existing in the `sgadprevio` role.
- **Important Note**: Keeping this file synchronized with the role is error-prone. Its deprecation in favor of using the role is recommended.

**Risks:**
- Code duplication.
- Divergence in logic if the role is modified but this playbook is not.
