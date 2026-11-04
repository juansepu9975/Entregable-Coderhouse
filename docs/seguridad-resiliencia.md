# Seguridad y resiliencia

Este documento describe cómo el sistema protege los datos, cómo responde a fallos y dónde están los puntos de aprobación humana.

## 1. Minimización de datos

- El flujo solo lee los campos necesarios para generar el borrador: Idea Semilla, Tono y el contenido de la base de conocimiento.
- A OpenAI se envía únicamente el tema, el tono y los fragmentos de la base de conocimiento. No se envían datos de identificación de pacientes.
- El correo del destinatario (**Email destinatario**) es un dato personal. Se carga solo cuando es necesario para el envío y, en la demostración, se usan direcciones de prueba.
- El dashboard público muestra solo los campos de control (Idea Semilla, fecha de modificación, registros de error y KPIs). Los campos de correo y el borrador no deben estar visibles en la vista compartida.
- Las credenciales (claves de OpenAI, Notion, Gmail y Airtable) están guardadas en las conexiones de Make y no aparecen en el repositorio ni en los archivos exportados.

## 2. Puntos de aprobación humana (HITL)

1. **Aviso de revisión:** cuando el borrador está listo, el revisor recibe un correo. La pieza queda en **En revisión** y no se envía nada al destinatario.
2. **Checkbox Aprobado:** la distribución solo toma piezas con **Aprobado** marcado. Sin esa marca, el contenido no sale.
3. **Filtro de estado:** la distribución excluye las piezas en **Publicado**, para que el mismo contenido no se envíe dos veces.

## 3. Rutas de error

| Situación | Respuesta del sistema |
|---|---|
| Falta Idea Semilla, Tono o Email revisor | El router envía la pieza a la ruta de fallback. Se crea un registro "Datos incompletos" en Registro de errores y no se llama a OpenAI. |
| La API de OpenAI falla | El módulo de error crea un registro "Fallo en Clasificar prioridad" vinculado a la pieza y termina la ejecución sin modificar la pieza. |
| Falla el envío de correo o la consulta a Notion | Hoy no hay una ruta de error específica. Es una limitación conocida, detallada en la sección 5. |

## 4. Prevención de bucles

- El disparador solo toma piezas con Estado **Generando** (filtro "Solo Generando").
- Al terminar, la pieza pasa a **En revisión**, por lo que el propio flujo no vuelve a procesarla.
- Los campos de fecha de modificación se comparan como fecha y hora, no como texto.

## 5. Limitaciones y mejoras pendientes

- **Rutas de error incompletas:** falta una ruta para fallos de Gmail y de Notion. Se propone agregar un registro de error en cada uno.
- **Directiva de error:** el cierre de las rutas de error usa Commit. Para una versión productiva, conviene usar Resume o Break en el módulo de OpenAI, para reintentar automáticamente.
- **Reprocesamiento:** si una pieza queda en Generando después de una prueba, el disparador vuelve a tomarla en corridas siguientes. Durante las pruebas esto generó registros duplicados. La regla operativa es no dejar piezas de prueba en Generando.
- **Aprobación no bloqueante:** el sistema no espera la aprobación dentro de la misma ejecución. La aprobación se hace en Airtable y la distribución es una segunda ejecución.

## 6. Hallazgos de las pruebas

- Una pieza sin tono generó un borrador con un tono inventado por el modelo. Esto motivó la validación previa, que ahora bloquea esos casos.
- Un error de configuración dejó un destinatario vacío en el aviso, lo que habría fallado el envío. Se corrigió mapeando el campo de la tabla.
- Una credencial de Notion apareció en una captura durante el desarrollo. Se recomienda regenerarla antes de publicar el repositorio.

## 7. Recomendaciones

- Rotar las claves de API al finalizar el proyecto.
- Revisar cada mes el borrador por muestreo, para detectar desvíos de calidad.
- Agregar rutas de error para Gmail y Notion, y usar Resume o Break en el módulo de IA.
