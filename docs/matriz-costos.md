# Matriz de costos y decisión de modelos

Este documento justifica qué modelo de IA se usa para cada tarea del flujo, y cómo se reduce el costo cuando sube el volumen.

> **Nota:** los precios son de referencia y deben verificarse en las páginas oficiales de OpenAI y Anthropic en la fecha de entrega. Los cálculos se hacen con esos precios.

## 1. Tareas que usan IA en el flujo

| Tarea | Módulo | Tipo de trabajo | Modelo elegido |
|---|---|---|---|
| Generar borrador de comunicación | 5. Generar borrador con OpenAI (11) | Redacción breve con instrucciones claras y base de conocimiento corta | gpt-4o-mini |
| Validación de datos | 2. Validar datos completos (28) | Lógica de filtros, sin IA | No usa modelo |
| Búsqueda en base de conocimiento | 3. Buscar base de conocimiento (20) | Consulta a Notion, sin IA | No usa modelo |

## 2. Por qué gpt-4o-mini para esta tarea

- La tarea es corta: un texto de pocas líneas con tono y firma definidos.
- Las instrucciones son explícitas (tono, firma, sin placeholders), y el modelo las sigue bien.
- El costo por pieza es muy bajo, lo que permite escalar sin un gasto relevante.
- Un modelo más grande (por ejemplo, un modelo premium) no aporta una mejora proporcional para un texto de este tamaño.

Claude (Haiku) Se evaluó como alternativa, pero no se probó en esta entrega. Se descartó en esta entrega porque el flujo ya está construido con OpenAI y cambiarlo no aporta mejora medible.

## 3. Estimación de costo por pieza

Supuestos por ejecución (valores de referencia, a confirmar con la pestaña de uso de OpenAI):

- Tokens de entrada (instrucciones, tema, tono y base de conocimiento): ~600.
- Tokens de salida (borrador): máximo 300, por el límite configurado.

Precios de referencia de gpt-4o-mini:

- Entrada: 0,15 USD por millón de tokens.
- Salida: 0,60 USD por millón de tokens.

Cálculo por pieza:

- Entrada: 600 × 0,15 / 1.000.000 = 0,00009 USD.
- Salida: 300 × 0,60 / 1.000.000 = 0,00018 USD.
- **Total por pieza: ~0,00027 USD.**

## 4. Escenarios de volumen

| Volumen mensual | Costo estándar | Costo con Batch (50% de descuento) |
|---|---|---|
| 100 piezas | ~0,03 USD | ~0,01 USD |
| 1.000 piezas | ~0,27 USD | ~0,14 USD |
| 10.000 piezas | ~2,70 USD | ~1,35 USD |

El descuento de Batch aplica cuando el procesamiento puede esperar hasta 24 horas, por ejemplo, la generación nocturna de un lote de piezas. En el flujo actual, cada pieza se procesa al instante, así que Batch no se usa todavía. Se propone para el caso en que la clínica cargue muchas piezas de una vez.

## 5. Medidas de control de costo ya implementadas

- **Límite de salida:** 300 tokens máximos por respuesta.
- **Validación previa:** las piezas con datos incompletos no llaman a la IA, así que no generan costo.
- **Base de conocimiento acotada:** la búsqueda en Notion trae solo los resultados necesarios (límite de 6).
- **Un solo modelo:** no se usan modelos más caros para tareas que no lo requieren.

## 6. Recomendación

Mantener gpt-4o-mini para la generación de borradores. Si el volumen crece, pasar la generación masiva a Batch. Revisar la calidad del borrador por muestreo cada mes, porque si la calidad baja, el costo de corregir texto puede superar el ahorro.
