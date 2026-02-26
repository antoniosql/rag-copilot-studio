# Sesión 3.1 — Cuándo RAG no es suficiente 

> **Módulo**: SESIÓN 3 — *Arquitectura híbrida: RAG, herramientas y breve introducción a datos estructurados*  
> **Bloque**: 3.1 Cuándo RAG no es suficiente  
> **Propósito**: que el alumnado sea capaz de **detectar** cuándo la respuesta no debe basarse solo en recuperación documental (RAG) y cuándo conviene un enfoque con **herramientas** (acciones) y/o **datos estructurados**.

---

## 0) Resultados de aprendizaje

Al finalizar este bloque, la persona participante podrá:

1. Identificar señales de que un caso de uso requiere **datos dinámicos** o **transaccionales** y, por tanto, RAG es insuficiente.
2. Distinguir entre preguntas de **conocimiento** (documental) y preguntas de **estado** (sistemas vivos).
3. Reconocer cuándo una consulta es **altamente estructurada** y conviene un acceso a datos (SQL/Dataverse/API) en lugar de RAG.
4. Diseñar una decisión rápida “RAG vs Tool vs Data” con criterios de precisión, latencia, seguridad y coste.

---

## 2) Recordatorio: qué hace bien RAG (y qué no)

### 2.1 RAG es excelente cuando…
- La información está en **documentos** relativamente estables (políticas, manuales, procedimientos).
- La respuesta necesita **síntesis** (explicar, resumir, comparar) a partir de contenido disponible.
- Queremos **citaciones** y trazabilidad de la fuente.

### 2.2 RAG se degrada cuando…
- La información cambia continuamente (tiempo real).
- La respuesta requiere **exactitud de campo** (valores exactos: estado, saldo, inventario).
- Se necesita **personalización por usuario** basada en identidad/rol/datos transaccionales.
- La consulta exige operaciones sobre datos (filtrar, ordenar, agrupar, “top N”, etc.) con garantías.

> **Regla práctica**:  
> Si la pregunta implica *“¿qué pasa ahora mismo?”*, *“¿cuál es mi…?”* o *“hazme…”* → sospecha que necesitas herramienta/datos.

---

## 3) Situación 1 — Información dinámica o en tiempo real

### 3.1 Qué significa “dinámica”
Datos que cambian en intervalos cortos:
- disponibilidad de sistemas, incidentes activos,
- precios, stock,
- estado de un envío,
- KPI del día, métricas en tiempo real,
- calendarios y agenda,
- estado meteorológico (ejemplo didáctico).

### 3.2 Por qué RAG falla aquí
- RAG depende de un índice: aunque lo reindexaras, nunca será tan inmediato como el sistema fuente.
- La “verdad” está en el **sistema**, no en un documento.

### 3.3 Patrón recomendado
- **Tool‑first**: llamar una herramienta/API que devuelva el dato actual y luego generar una respuesta explicativa.

#### Ejemplo
Usuario: “¿Está el servicio X caído ahora?”  
- **Tool**: consulta a monitorización / status page interna  
- **LLM**: “Ahora mismo hay un incidente P2 abierto desde las 09:12; impacto…; próximos pasos…”

---

## 4) Situación 2 — Datos transaccionales (operaciones)

### 4.1 Qué es “transaccional”
Acciones que crean o modifican estado:
- crear un ticket,
- aprobar una solicitud,
- registrar una compra,
- actualizar un pedido,
- cambiar una dirección,
- bloquear una tarjeta (ejemplo conceptual).

### 4.2 Por qué RAG no basta
RAG **no ejecuta** acciones; solo recupera texto y redacta.
Si intentas simular la transacción con texto:
- el usuario cree que “se hizo” cuando no se hizo (riesgo operativo),
- no hay auditoría ni confirmación,
- no hay idempotencia ni control de errores.

### 4.3 Patrón recomendado
- **RAG + Tool**:
  - RAG para explicar el proceso/política (“qué hay que hacer”),
  - Tool para **hacerlo** (crear registro, iniciar flujo),
  - LLM para confirmar y contextualizar (“creado ticket INC‑…”, “siguiente paso…”).

#### Ejemplo
Usuario: “Crea una solicitud de vacaciones del 10 al 14 de marzo.”  
- **Tool**: crear solicitud en HRIS / Dataverse  
- **LLM**: resume, confirma, advierte políticas (y cita) + devuelve ID.

---

## 5) Situación 3 — Consultas altamente estructuradas

### 5.1 Señales típicas
- el usuario pide “lista”, “top”, “filtra por”, “agrupa por”
- consultas con campos:
  - “Dame los clientes de Barcelona con facturación > 50k”
  - “Incidencias abiertas por prioridad y equipo”
- requiere exactitud en columnas, no “aproximación semántica”.

### 5.2 Por qué RAG falla o es caro
- RAG recupera texto, no es un motor de consulta relacional.
- Meter miles de filas en el contexto es:
  - caro (tokens),
  - lento,
  - arriesgado (exposición de datos),
  - y difícil de verificar.

### 5.3 Patrón recomendado
- **Data‑first** con una herramienta que haga la consulta:
  - Dataverse “Search rows/List rows” (con límites),
  - SQL / API,
  - servicio analítico.

Y después:
- el LLM explica resultados y propone acciones.

---

## 6) Situación 4 — Información personalizada por usuario

### 6.1 Qué incluye
- “¿Cuántos días de vacaciones me quedan?”
- “¿Cuál es mi número de empleado?”
- “¿Qué pedidos tengo pendientes?”
- “¿Cuál es el estado de mi ticket?”

### 6.2 Por qué RAG no basta
- Un documento puede describir la regla (“cómo se calcula”), pero el valor depende del **usuario** y del **momento**.
- Requiere identidad, permisos, y acceso a datos por usuario.

### 6.3 Patrón recomendado
- **RAG + Tool + reglas de autorización**
  - Tool obtiene datos del usuario autenticado (claims/Entra ID),
  - RAG aporta explicación (política y procedimiento) con citaciones,
  - respuesta final combina ambos.

> **Nota de gobierno**: en enterprise, “personalización” implica casi siempre **control de acceso** (row‑level security) y auditoría.

---

## 7) Antipatrones: lo que sale mal si fuerzas RAG

### 7.1 “RAG como sistema de verdad”
- “El bot dice que…” se convierte en fuente de verdad.
- Si el índice está desactualizado → decisiones incorrectas.

### 7.2 “Copiar y pegar datos” (dump de tablas)
- costos altos y respuestas poco fiables,
- exposición de información no necesaria,
- problemas de privacidad.

### 7.3 “Simular una acción”
- el bot “promete” que hizo algo sin confirmación real.
- genera tickets falsos o expectativas incorrectas.

### 7.4 “Hacer RAG con datos que deberían ser estructurados”
- p. ej. facturas o CRM como PDF indexado: posible, pero suele ser peor que consultar campos.

---

## 8) Matriz de decisión: RAG vs Tool vs Data agent

### 8.1 Criterios prácticos (para decidir rápido)

| Criterio | RAG (documental) | Tool (acción / API) | Datos estructurados (consulta) |
|---|---|---|---|
| “Verdad vigente” | Media (depende de indexación) | Alta (fuente viva) | Alta (fuente viva) |
| Personalización por usuario | Limitada | Alta (si hay auth) | Alta (si hay RLS) |
| Consultas “top/filter/group” | Débil | Media | Alta |
| Citaciones | Alta | Baja (no hay doc) | Baja/Media (depende de trazabilidad) |
| Riesgo de alucinación | Medio | Bajo (si el output es correcto) | Bajo |
| Coste por consulta | Medio | Medio/alto (depende) | Medio |
| Time‑to‑value | Alto | Medio | Medio |

### 8.2 Regla de oro (operativa)
- Si la pregunta busca **norma/procedimiento** → RAG.
- Si busca **estado actual** → Tool / Data.
- Si busca **operar** → Tool.
- Si busca **consulta precisa** sobre registros → Data.

---

## 9) Mini‑actividad (5 min): clasificar preguntas

Clasifica cada pregunta como:
- **RAG**
- **Tool**
- **Datos estructurados**
- **Híbrido** (RAG + Tool/Data)

1. “¿Cuál es la política de gastos de viaje?”  
2. “¿Cuánto llevo gastado este mes en dietas?”  
3. “Crea una incidencia por fallo de VPN.”  
4. “¿Qué significa severidad P1?”  
5. “Dame los 10 tickets abiertos más antiguos del equipo X.”  
6. “¿Puedo trasladar vacaciones al año siguiente?”  
7. “¿Cuál es el estado de mi ticket INC‑1234?”  
8. “¿Qué proveedores están bloqueados y por qué?”

### Respuestas esperadas (para el instructor)
- 1 RAG
- 2 Datos/Tool (depende del sistema)
- 3 Tool
- 4 RAG
- 5 Datos estructurados (consulta)
- 6 RAG
- 7 Tool/Datos (consulta por ID)
- 8 Híbrido (datos + explicación/política)

---

## 10) Cierre del bloque

Mensaje clave:
> Un buen asistente empresarial no elige “solo RAG” o “solo herramientas”.  
> Diseña una **arquitectura híbrida** que combine recuperación documental para contexto y herramientas/datos para exactitud y acciones.

---

## Referencias oficiales (lectura opcional)

```text
Copilot Studio — Guía de RAG (patrones, grounding, comportamiento general):
https://learn.microsoft.com/en-us/microsoft-copilot-studio/guidance/retrieval-augmented-generation

Copilot Studio — Guía general (incluye integración y herramientas):
https://learn.microsoft.com/en-us/microsoft-copilot-studio/guidance/
```
