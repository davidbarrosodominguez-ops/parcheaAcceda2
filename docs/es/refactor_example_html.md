# Ejemplo de Refactorización: Generación de HTML

Este documento ilustra cómo refactorizar la generación de HTML/Contenido que actualmente se realiza mediante comandos de shell, moviéndola a plantillas de Ansible (Jinja2).

## Caso Práctico: Generación de Correo Electrónico

### 🔴 Antes (Código Actual)
En `roles/sgadprevio/tasks/12_report.yml`, el contenido del correo se genera concatenando múltiples comandos `echo` dentro de un bloque `shell`.

```yaml
    - name: Genera mail
      shell: |
        (
        echo "Subject: Mantenimiento PREVIO {{ aplicacion.stdout }} {{ ansible_hostname }} ..."
        echo "Content-Type: text/html; charset=utf-8"
        echo "..."
        echo "<div style='text-align:center'>"
        echo "<br><h1><u>MANTENIMIENTO PREVIO ... </u></h1>"
        echo "<pre><br>{{ subscription.stdout_lines|join('<br>') }}</pre>"
        # ... más comandos echo ...
        ) | sendmail  -t "david.bdominguez@externos.correo.gob.es"
```

**Problemas:**
*   Difícil de leer y mantener.
*   Propenso a errores de sintaxis (comillas, escapes).
*   Mezcla lógica de presentación con lógica de ejecución.

### 🟢 Después (Solución Recomendada)

La solución consiste en separar el contenido en un fichero de plantilla (`.j2`) y usar el módulo `template`.

#### 1. Crear el Fichero de Plantilla
Crear `roles/sgadprevio/templates/email_body.j2`. Aquí podemos escribir HTML/Texto normal y usar variables de Ansible directamente.

```html
Subject: Mantenimiento PREVIO {{ aplicacion.stdout }} {{ ansible_hostname }} PARCHEADO DE {{ ansible_distribution_version }} OK {{ ansible_date_time.date }}
CC: david.bdominguez@externos.correo.gob.es
Content-Type: multipart/mixed; boundary=boundary42; charset=utf-8

--boundary42
Content-Type: text/html; charset=utf-8

{{ cabecera_content }}

<div style='text-align:center'>
    <br><h1><u>MANTENIMIENTO PREVIO PARCHEADO DE {{ ansible_distribution_version }} DE {{ ansible_hostname }}:<br></u></h1>
    <pre><br>
    {% for line in subscription.stdout_lines %}
    {{ line }}<br>
    {% endfor %}
    </pre>
    <p><br>{{ ansible_date_time.date }}<p><br>
    
    Datos de Contacto:
    {% for contacto_item in contacto.stdout.split(';') %}
    <br>{{ contacto_item }}<br>
    {% endfor %}
    
    <!-- ... resto del contenido ... -->
</div>
</body>
</html>
--boundary42
Content-Type: application/octet-stream; name=...
...
--boundary42--
```

#### 2. Actualizar la Tarea de Ansible
Usar el módulo `template` para generar el fichero completo y luego enviarlo.

```yaml
    - name: Renderizar plantilla de correo
      template:
        src: email_body.j2
        dest: /tmp/email_ready_to_send.txt
      delegate_to: adgesasateinfc2

    - name: Enviar correo
      shell: cat /tmp/email_ready_to_send.txt | sendmail -t "david.bdominguez@externos.correo.gob.es"
      delegate_to: adgesasateinfc2
```

**Beneficios:**
*   **Claridad**: El HTML se lee como HTML, no como strings de shell.
*   **Potencia**: Uso de bucles (`{% for %}`) y condicionales (`{% if %}`) de Jinja2 nativos, en lugar de `sed`/`awk` complejos.
*   **Mantenibilidad**: Es mucho más fácil cambiar el diseño del correo sin romper el script.
