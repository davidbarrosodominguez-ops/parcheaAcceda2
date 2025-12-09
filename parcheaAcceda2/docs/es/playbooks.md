# Documentación de Playbooks

## `sgadprevio.yml`
Este es el punto de entrada recomendado para ejecutar la auditoría utilizando la estructura de roles.

**Función:**
- Define los hosts objetivo (`hosts: all`).
- Habilita la elevación de privilegios (`become: yes`).
- Define variables globales de ficheros (`fichero`, `fich1`, etc.).
- Llama al rol `sgadprevio`.

**Ejemplo de uso:**
```bash
ansible-playbook sgadprevio.yml -i inventario
```


