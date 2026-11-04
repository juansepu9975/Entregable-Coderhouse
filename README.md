# Entregable Final: Ecosistema de Automatización IA para Pulmahue

Sistema de automatización de contenido para una clínica (Pulmahue) que genera borradores de comunicaciones a pacientes con IA, los somete a aprobación humana y los distribuye por correo, con registro de errores y un dashboard de control.

## Enlaces de entrega

- **Dashboard de control (Shared View de Airtable):** https://airtable.com/applMITHCnOQM9YdF/shrE93K90jwIXfP0w/tbl5DvfD5RE0S8ee0
- **Video demo (3 min):** https://www.loom.com/share/21ecc162fecd45e1b7c7bd9708675b81
- **Documentación completa:** [docs/](docs/)

## Stack tecnológico

| Categoría | Herramienta |
|---|---|
| Orquestador | Make (2 escenarios) |
| Base de datos | Airtable (tablas Piezas y Registro de errores) |
| Procesamiento IA | OpenAI, modelo gpt-4o-mini |
| Base de conocimiento | Notion |
| Canal de salida | Gmail |

## Arquitectura

El sistema tiene dos escenarios:

1. **Generación:** detecta una pieza en estado *Generando*, valida que tenga Idea Semilla y Tono, busca la base de conocimiento, genera el borrador con IA, lo guarda y avisa al revisor. Si faltan datos o la IA falla, registra el error en Airtable sin continuar.
2. **Distribución:** solo actúa sobre piezas con *Aprobado* marcado y que no estén publicadas. Envía el contenido al destinatario y marca la pieza como *Publicado*.

El punto de aprobación humana (HITL) es el checkbox **Aprobado** en Airtable: sin esa marca, el contenido no sale.

## Carpetas del repositorio

- `docs/`: documentación de la entrega (manual de datos, matriz de costos, seguridad y resiliencia).
- `blueprints/`: exportación JSON de los dos escenarios de Make.
- `datos/`: CSV de las tablas Piezas y Registro de errores.
- `capturas/`: evidencia de ejecuciones, correos y dashboard.
- `diagrama/`: diagrama de arquitectura.

## Pruebas realizadas

Se ejecutaron pruebas con datos completos, con datos incompletos y con un fallo de OpenAI. La evidencia está en capturas/ y datos/.
