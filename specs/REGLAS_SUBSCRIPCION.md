# Spec de Subscripción: Reglas para la Revisión Manual

Este documento define las reglas de negocio relacionadas con la subscripción a Red Hat Satellite. Cualquier condición que cumpla estas reglas se marcará en el informe con el formato `REVISAR_..._REVISAR`.

## Reglas de Auditoría de Subscripción

| Área | Condición Crítica | Lógica de Activación en el Rol | Ejemplo de Mensaje de Revisión |
| :--- | :--- | :--- | :--- |
| **Subscripción** | El servidor no está subscrito o el estado de la subscripción no es válido. | La tarea en `02_subscription.yml` comprueba la salida del comando `subscription-manager status`. | `REVISAR_El estado de la subscripción del sistema es 'Unknown' o 'Invalid'._REVISAR` |
| **Release** | El `release` fijado para la subscripción no es el esperado. | La tarea en `02_subscription.yml` comprueba la salida de `subscription-manager release`. | `REVISAR_El release de la subscripción es 8.6 pero se esperaba 8.8._REVISAR` |
