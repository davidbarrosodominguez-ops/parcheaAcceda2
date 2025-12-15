# Spec de Red: Reglas para la Revisión Manual

Este documento define las reglas de negocio relacionadas con la configuración de red del servidor. Cualquier condición que cumpla estas reglas se marcará en el informe con el formato `REVISAR_..._REVISAR`.

## Reglas de Auditoría de Red

| Área | Condición Crítica | Lógica de Activación en el Rol | Ejemplo de Mensaje de Revisión |
| :--- | :--- | :--- | :--- |
| **Rutas** | Existe una ruta por defecto inesperada o no hay ruta por defecto. | La tarea en `06_red.yml` analiza la salida de `ip route`. | `REVISAR_No se ha encontrado una ruta por defecto para el sistema._REVISAR` |
| **Bonding** | Una interfaz de red `bond` no tiene todos sus esclavos (`slaves`) activos. | La tarea en `06_red.yml` inspecciona el estado de los interfaces `bond`. | `REVISAR_La interfaz bond0 no tiene todos sus esclavos activos._REVISAR` |
