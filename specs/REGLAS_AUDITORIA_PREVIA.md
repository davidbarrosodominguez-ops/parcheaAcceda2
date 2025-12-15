# Spec de Auditoría: Reglas para la Revisión Manual

Este documento define las reglas de negocio que el rol de Ansible utiliza para marcar puntos de atención en el informe previo al parcheado. Cualquier condición que cumpla estas reglas se marcará en el informe con el formato `REVISAR_..._REVISAR`.

## Reglas de Auditoría Pre-Parcheado

| Área | Condición Crítica | Lógica de Activación en el Rol | Ejemplo de Mensaje de Revisión |
| :--- | :--- | :--- | :--- |
| **Disco** | Un sistema de ficheros supera el 80% de ocupación. | La tarea en `10_discos.yml` comprueba la salida del comando `df -h`. | `REVISAR_Espacio en disco crítico: /dev/sda1 al 85%_REVISAR` |
| **Kernel** | La actualización del kernel requiere un reinicio. | La tarea en `05_updates.yml` compara la versión del kernel en ejecución con la que se va a instalar. | `REVISAR_Actualización de kernel pendiente, se requerirá reinicio_REVISAR` |
| **Servicios**| Se detecta un servicio crítico (BBDD, Web, etc.). | La tarea en `07_servicios.yml` busca procesos con nombres conocidos (`httpd`, `oracle`, etc.). | `REVISAR_Detectado servicio crítico: httpd. Requiere parada manual._REVISAR` |