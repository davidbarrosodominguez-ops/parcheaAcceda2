# Spec Post-Parcheado: Reglas para la Revisión Manual

Este documento define las reglas de negocio que el rol de Ansible utiliza para marcar puntos de atención en el informe **posterior** al parcheado. Cualquier condición que cumpla estas reglas se marcará en el informe con el formato `REVISAR_..._REVISAR`.

## Reglas de Validación de Limpieza

| Área | Condición de Validación | Lógica de Activación en el Rol | Ejemplo de Mensaje de Revisión |
| :--- | :--- | :--- | :--- |
| **Copia** | El informe final no se copió correctamente al almacén. | La tarea en `99_postchecks.yml` verifica si el fichero del informe existe en el directorio de destino después de la operación de copia. | `REVISAR_La copia del informe final al almacén ha fallado._REVISAR` |
| **Limpieza** | Los ficheros temporales no se eliminaron correctamente. | La tarea en `99_postchecks.yml` verifica que los ficheros temporales del informe ya no existen después de la operación de borrado. | `REVISAR_La limpieza de los ficheros temporales ha fallado._REVISAR` |

---

## Reglas de Auditoría Post-Parcheado (Histórico - Deprecado)

> [!NOTE]
> Las siguientes reglas están en desuso y no se implementan en la versión actual del rol. Se mantienen únicamente como registro histórico.

| Área | Condición de Validación | Lógica de Activación en el Rol | Ejemplo de Mensaje de Revisión |
| :--- | :--- | :--- | :--- |
| **Kernel** | El kernel en ejecución no es el esperado tras el parcheo. | La tarea en `99_postchecks.yml` compara `uname -r` con la versión de kernel que se debería haber instalado. | `REVISAR_El kernel en ejecución no es el nuevo. El parcheo pudo fallar._REVISAR` |
| **Reinicio** | El sistema no se ha reiniciado cuando era necesario. | La tarea en `99_postchecks.yml` comprueba el `uptime`. Si es elevado y se esperaba un reinicio, genera una alerta. | `REVISAR_El sistema no se ha reiniciado tras una actualización de kernel._REVISAR` |
| **Servicios**| Un servicio que estaba activo antes no ha levantado. | La tarea en `99_postchecks.yml` compara el estado actual de los servicios con el estado previo. | `REVISAR_El servicio httpd no se ha iniciado correctamente tras el parcheo._REVISAR` |