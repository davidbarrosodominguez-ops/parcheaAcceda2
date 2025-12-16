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

