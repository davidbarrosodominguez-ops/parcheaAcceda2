# Spec Post-Parcheado: Reglas para la Revisión Manual

Este documento define las reglas de negocio que el rol de Ansible utiliza para marcar puntos de atención en el informe **posterior** al parcheado. Cualquier condición que cumpla estas reglas se marcará en el informe con el formato `REVISAR_..._REVISAR`.

## Reglas de Auditoría Post-Parcheado

| Área | Condición de Validación | Lógica de Activación en el Rol | Ejemplo de Mensaje de Revisión |
| :--- | :--- | :--- | :--- |
| **Kernel** | El kernel en ejecución no es el esperado tras el parcheo. | La tarea en `13_postchecks.yml` compara `uname -r` con la versión de kernel que se debería haber instalado. | `REVISAR_El kernel en ejecución no es el nuevo. El parcheo pudo fallar._REVISAR` |
| **Reinicio** | El sistema no se ha reiniciado cuando era necesario. | La tarea en `13_postchecks.yml` comprueba el `uptime`. Si es elevado y se esperaba un reinicio, genera una alerta. | `REVISAR_El sistema no se ha reiniciado tras una actualización de kernel._REVISAR` |
| **Servicios**| Un servicio que estaba activo antes no ha levantado. | La tarea en `13_postchecks.yml` compara el estado actual de los servicios con el estado previo. | `REVISAR_El servicio httpd no se ha iniciado correctamente tras el parcheo._REVISAR` |