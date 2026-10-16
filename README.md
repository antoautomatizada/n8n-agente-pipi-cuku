# Agente de atención y registro de leads - Pipí Cukú

Proyecto integrador del curso de automatización avanzada con n8n.
Este repositorio contiene el **Checkpoint 1: Agente base y motor de razonamiento**.

Pipí Cukú es un emprendimiento ficticio de viandas saludables con delivery. El agente atiende consultas de personas interesadas y registra los leads en una planilla de Google Sheets.

## Qué hace el agente

- Recibe un mensaje de una persona interesada en las viandas.
- Obtiene los datos necesarios: nombre, contacto, tipo de vianda, cantidad semanal y zona de entrega.
- Decide de forma autónoma cuándo registrar el lead en Google Sheets.
- Deriva a una persona del equipo las consultas sobre alergias, intolerancias y pedidos de descuento o precios especiales.
- Envía un mail de "Tarea completada" con la respuesta final y la auditoría de los pasos del agente.

## Arquitectura del flujo

```
[Chat Trigger] → [AI Agent] → [Gmail: Tarea completada]
                    ├─ Chat Model: OpenRouter Chat Model
                    └─ Tool: Google Sheets (Registrar_Lead_Sheets)
```

| Componente | Configuración |
|---|---|
| Disparador | Chat Trigger |
| AI Agent | Modo Tools Agent |
| Chat Model | OpenRouter Chat Model - modelo: **gpt-5-mini** |
| Guardrail | Max Iterations: **5** |
| System Message | Estructura modular: Intención, Detalles, Ejemplos, Acción y Límites |
| Tool | Google Sheets (Append Row), conectada de forma lateral al agente, con descripción detallada de cuándo activarla |
| Observabilidad | Gmail al final del flujo, con Return Intermediate Steps activado |

## Nota sobre el modelo de lenguaje

La consigna recomienda GPT-4o o Claude 3.5 Sonnet. Para este checkpoint se utilizó **gpt-5-mini** a través de OpenRouter, porque no se cuenta con créditos para los modelos recomendados. Se eligió un modelo gratuito que soporta tool calling, de modo que el agente pueda activar la herramienta de forma autónoma. El modelo se puede reemplazar sin modificar el resto del flujo.

## Guardrails y límites del agente

- **Límite de iteraciones:** máximo de vueltas del ciclo ReAct configurado en el panel avanzado del nodo, para evitar bucles infinitos y gasto innecesario.
- **Herramienta acotada:** el agente solo puede agregar filas en la pestaña de leads. No puede leer, modificar ni borrar registros.
- **Restricciones de comportamiento:** no confirma precios especiales ni descuentos y no responde consultas sobre alergias o intolerancias.
- **Escalamiento:** ante esos casos, detiene su ciclo y deriva a revisión humana con una frase fija.

## Planilla de Google Sheets

Pestaña `Leads`, con los encabezados en la fila 1:

`Fecha` | `Nombre` | `Contacto` | `Tipo de vianda` | `Cantidad semanal` | `Zona de entrega` | `Observaciones`

La fecha se completa de forma automática con la zona horaria de Argentina (`America/Argentina/Buenos_Aires`).

## Casos de prueba

| Caso | Mensaje de ejemplo | Resultado esperado | Resultado obtenido |
|---|---|---|---|
| Lead completo | "Hola, soy Virginia, quiero viandas vegetarianas, mi mail es virginia@example.com" | Activa la Tool y registra la fila |Activó la Tool Registrar_Lead_Sheets de forma autónoma, registró la fila en Sheets con la fecha automática (hora argentina), respondió al cliente confirmando el registro y envió el mail de “Tarea completada” con la auditoría.
| Consulta por alergias | "¿Tienen opciones aptas para celíacos?" | No registra y deriva a revisión humana | Respondió la frase de derivación y no registró nada en Sheets.
| Pedido de descuento | "¿Me hacen descuento si pido para mi oficina?" | No registra y deriva a revisión humana |Derivó a revisión humana sin activar la Tool
| Faltan datos | "Quiero viandas para toda la semana" | Pide nombre y contacto antes de registrar |Pidió nombre y contacto antes de registrar 

## Cómo usar el flujo

1. Importá `checkpoint1_antonela_foresti.json` en n8n (en un canvas vacío, pegá el contenido con `Ctrl + V`).
2. Configurá tus propias credenciales: OpenRouter, Google Sheets y Gmail.
3. Seleccioná tu planilla en el nodo Google Sheets y tu dirección de destino en el nodo Gmail.
4. Abrí el chat del workflow y enviá un mensaje de prueba.

## Hoja de ruta del proyecto

| Módulo | Incorporación |
|---|---|
| M1 | Agente base: Trigger, AI Agent con System Prompt, una Tool y observabilidad (este checkpoint) |


## Autoría

Antonela Foresti
