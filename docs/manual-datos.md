# Manual operativo de datos

Este manual describe las tablas de Airtable, sus relaciones, los estados del flujo y los formatos de datos que intercambian Make, Airtable, OpenAI y Gmail.

## 1. Tablas y relaciones

La base **Entrega 8 - Pipeline Contenido** tiene dos tablas vinculadas.

- **Piezas**: una fila por pieza de comunicación. Es la memoria del sistema.
- **Registro de errores**: una fila por cada error o bloqueo registrado. Cada error apunta a una pieza.

Relación: **Piezas (1) → Registro de errores (N)**. Un registro de error siempre pertenece a una pieza, mediante el campo vinculado **Pieza**. Si la pieza no tiene Idea Semilla, el registro aparece como "registro sin nombre".

## 2. Tabla Piezas

| Campo | Tipo | Descripción |
|---|---|---|
| Idea Semilla | Texto (principal) | Tema de la comunicación. Obligatorio para generar. |
| Estado | Selección única | `Generando`, `En revisión`, `Publicado`. Es el disparador del flujo. |
| Tono | Selección única | Tono del texto, por ejemplo `Formal` o `Cálido y claro`. Obligatorio. |
| Borrador generado | Texto largo | Texto producido por la IA. |
| Aprobado | Casilla | Aprobación humana. Solo las piezas aprobadas se distribuyen. |
| Email revisor | Email | Persona que recibe el aviso de revisión. |
| Email destinatario | Email | Destinatario del contenido aprobado. Dato personal: cargarlo solo cuando sea necesario. |
| Última modificación | Fecha y hora (automática) | Campo que usa Make para detectar cambios. |
| Última ejecución | Fecha | Reservado para registrar la última ejecución. |
| Registro de errores | Vínculo | Enlaza con la tabla Registro de errores. |
| KPI Salida | Fórmula | `IF({Borrador generado}, 1, 0)`. Vale 1 si la pieza generó borrador. |
| KPI Aprobación | Fórmula | `IF(AND({Estado}="Publicado", {Borrador generado}), 1, 0)`. Vale 1 si la pieza fue publicada con borrador. |
| KPI Error | Fórmula | `IF({Registro de errores}, 1, 0)`. Vale 1 si la pieza tiene al menos un error. |

### Estados del flujo

- **Generando**: la pieza espera que Make genere el borrador.
- **En revisión**: el borrador está listo y espera la aprobación humana.
- **Publicado**: la pieza fue aprobada y enviada al destinatario.

## 3. Tabla Registro de errores

| Campo | Tipo | Descripción |
|---|---|---|
| Error | Texto | Nombre corto del problema, por ejemplo `Datos incompletos` o `Fallo en Clasificar prioridad`. |
| Detalle | Texto largo | Descripción del problema y acción esperada. |
| Módulo | Texto | Módulo de Make que registró el error, por ejemplo `Validación datos (28)`. |
| Pieza | Vínculo a Piezas | Pieza afectada. |
| Resuelto | Casilla | Se marca cuando una persona resuelve el caso. |
| Fecha | Fecha y hora (automática) | Momento en que se creó el registro. |

## 4. Estados y reglas de negocio

1. Una pieza entra al flujo solo si su Estado es **Generando** (filtro "Solo Generando").
2. Si falta **Idea Semilla**, **Tono** o **Email revisor**, el flujo registra un error y no llama a la IA.
3. Si la IA falla, el flujo registra un error y termina sin modificar la pieza.
4. Si el borrador se genera, la pieza pasa a **En revisión** y se envía el aviso al revisor.
5. La distribución solo envía piezas con **Aprobado** marcado y Estado distinto de **Publicado**.
6. Al distribuir, la pieza pasa a **Publicado** y no vuelve a enviarse.

## 5. Formatos de transferencia (JSON)

### 5.1 Make → Airtable (lectura de la pieza)

Salida del módulo *Watch records* (ejemplo simplificado):

```json
{
  "ID": "recXXXXXXXXXXXXXX",
  "Idea Semilla": "Recordatorio: confirmá tu turno con 24 h de anticipación",
  "Estado": "Generando",
  "Tono": "Formal",
  "Email revisor": "revisor@ejemplo.cl",
  "Aprobado": false,
  "Última modificación": "2026-11-04T14:06:03.000Z"
}
```

### 5.2 Make → OpenAI (solicitud)

Modelo `gpt-4o-mini`, límite de 300 tokens, con dos mensajes: uno de sistema (rol, reglas de estilo y firma "Clínica Pulmahue") y uno de usuario con las variables de la pieza y la base de conocimiento.

```json
{
  "model": "gpt-4o-mini",
  "max_tokens": 300,
  "messages": [
    { "role": "system", "content": "Eres redactor de contenidos para una clínica. ... Firmá siempre como Clínica Pulmahue." },
    { "role": "user", "content": "Tema: {{Idea Semilla}} Tono: {{Tono}} Base de conocimiento: {{Título}} {{Contenido}}" }
  ]
}
```

### 5.3 OpenAI → Make (respuesta)

El texto se toma de `choices[0].message.content` y se guarda en **Borrador generado**.

```json
{
  "choices": [
    { "index": 0, "message": { "role": "assistant", "content": "Estimado paciente, ..." } }
  ]
}
```

### 5.4 Make → Gmail (aviso y envío)

- **Aviso al revisor:** destinatario `{{Email revisor}}`, asunto "Pieza lista para revisar: {{Idea Semilla}}", texto fijo que pide revisar el borrador y marcar Aprobado.
- **Envío final:** destinatario `{{Email destinatario}}`, asunto "Contenido aprobado: {{Idea Semilla}}", cuerpo `{{Borrador generado}}`.

## 6. Ejemplo de registro de error

| Error | Detalle | Módulo | Pieza |
|---|---|---|---|
| Datos incompletos | Falta Idea Semilla o Tono. No se llamó a OpenAI | Validación datos (28) | Prueba validación: sin tono |
