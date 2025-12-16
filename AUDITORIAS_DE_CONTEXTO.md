# Resumen de Comprobaciones de Contexto

Este documento resume las comprobaciones de coherencia y análisis que se pueden solicitar para auditar el estado del proyecto.

### 1. Auditoría de Coherencia de Documentación
Revisa que los directorios de documentación en español (`docs/es`) e inglés (`docs/en`) tengan la misma estructura de ficheros.
*   **Acción:** Compara los nombres de los archivos, detecta inconsistencias y ficheros faltantes.
*   **Utilidad:** Asegurar que la documentación no diverge entre idiomas.

### 2. Auditoría de Coherencia de Especificaciones
Analiza el contenido de todos los ficheros de reglas (`specs/REGLAS_*.md`) para ver si son coherentes entre sí y con el código.
*   **Acción:** Lee todas las especificaciones y las compara con la lógica de las tareas de Ansible.
*   **Utilidad:** Detectar discrepancias entre lo que la documentación dice que el código debe hacer y lo que realmente hace.

### 3. Análisis de Patrones de Código (Separación Lógica/Presentación)
Busca tareas que no siguen la arquitectura deseada de "separar la lógica de la presentación".
*   **Acción:** Busca tareas que generan código HTML y mensajes `REVISAR_` directamente desde `shell`, en lugar de delegar en la plantilla.
*   **Utilidad:** Identificar código que necesita ser refactorizado para mejorar la mantenibilidad y seguir las convenciones del proyecto.

### 4. Revisión de Completitud de la Documentación
Verifica que un fichero de documentación (como `refactoring_opportunities.md`) esté completo y refleje todos los hallazgos de un análisis de código.
*   **Acción:** Realiza una búsqueda de un patrón de código específico y compara los resultados con una lista existente en un documento.
*   **Utilidad:** Asegurar que la documentación sobre el estado del código (ej. deuda técnica) es exhaustiva y no omite casos.
