# Sesión 4.2 — Evaluación en Copilot Studio

> **Módulo**: SESIÓN 4 — *Evaluación, seguridad y gobierno de un sistema RAG*  
> **Bloque**: 4.2 Evaluación en Copilot Studio  
>> **Propósito**: que el alumnado sepa **crear test sets**, elegir **métodos de evaluación**, ejecutar evaluaciones, interpretar resultados y proponer mejoras iterativas.

---

## 0) Objetivos de aprendizaje

Al finalizar este bloque, el participante será capaz de:

1. Crear un **test set** en Copilot Studio usando distintos métodos:
   - generación rápida,
   - generación a partir de knowledge/topics,
   - importación CSV/TXT,
   - test chat,
   - temas (analytics).
2. Seleccionar métodos de evaluación adecuados a cada tipo de pregunta:
   - General quality
   - Compare meaning
   - Keyword match
   - Text similarity
   - Exact match
   - Capability use (si aplica).
3. Ejecutar una evaluación y **leer el resumen** + “drill‑down” por test case.
4. Exportar resultados y construir un plan de **mejora iterativa**.

---

## 2) Preparación (antes del hands‑on)

### 2.1 Requisitos

- Acceso a Copilot Studio con permisos de edición del agente.
- Un agente con:
  - al menos una fuente de conocimiento (SharePoint o Documents/Dataverse),
  - Generative Answers habilitado (o nodo por topic).
- Un “mini‑corpus” controlado (p. ej. 1–3 documentos o 1 URL de SharePoint).

### 2.2 Advertencia importante (seguridad en evaluación)

Copilot Studio permite ejecutar evaluaciones automáticas usando una **cuenta de prueba** (test account / user profile).  
En evaluaciones automáticas:

- se usan las credenciales de esa cuenta para acceder a knowledge sources y tools,
- los resultados y preguntas generadas pueden incluir información a la que esa cuenta tiene acceso,
- y esa información puede ser visible para makers con acceso al test set.

> En entornos corporativos, esto obliga a elegir cuentas de prueba “controladas” y limitar el acceso a los test sets.

---

## 3) Crear un test set en Copilot Studio (opciones)

Un test set:
- agrupa hasta **100 test cases**,  
- y se ejecuta “en batch” contra el agente.

### 3.1 Métodos típicos para crear un test set

En Copilot Studio, puedes crear un test set de varias formas:

1) **Quick question set**
   - genera 10 preguntas rápidas basadas en la descripción/instrucciones/capacidades del agente.
   - útil para arrancar y para “smoke tests”.

2) **Full question set**
   - genera preguntas usando conocimiento o topics del agente.
   - útil para ampliar cobertura a partir del corpus real.

3) **Use your test chat conversation**
   - convertir las preguntas usadas en el panel de test (test chat) en test cases reutilizables.
   - útil para “capturar” pruebas manuales.

4) **Import test cases from a file**
   - subir CSV/TXT con preguntas y expected responses/método.

5) **Manual (escribir a mano)**
   - útil para casos límite, edge cases y compliance.

6) **Production data based on themes**
   - generar test cases a partir de “themes” del analytics (preguntas reales agrupadas).

> Recomendación didáctica: combina 1 (rápido) + 4 (import) + 6 (temas) para pasar de prototipo a operación.

---

### 3.2 Pasos guiados: crear un test set desde la UI

1) Ir a la página **Evaluation** del agente.  
2) Seleccionar **New evaluation**.  
3) Elegir método (por ejemplo “Quick question set” o “Import”).  
4) Dar nombre al test set.  
5) Seleccionar **test methods**.  
6) Seleccionar **User profile** (cuenta para ejecutar la evaluación) o continuar sin auth.  
7) Guardar o ejecutar (Evaluate).

---

## 4) Importación desde CSV/TXT (formato mínimo)

Si tu organización ya tiene FAQs, tickets, y tests existentes, la importación es el camino rápido.

### 4.1 Cabeceras obligatorias (en este orden)

En la primera fila del CSV/TXT, usa:

- `Question`
- `Expected response`
- `Testing method`

### 4.2 Métodos válidos (columna “Testing method”)

- General quality
- Compare meaning
- Similarity (Text similarity)
- Exact match
- Keyword match

### 4.3 ¿Expected response es obligatorio?

- Para importar: puede ser opcional.
- Para ejecutar métodos tipo “match/similarity/compare meaning”: necesitas expected response.

> Práctica recomendada: incluso si usas General quality, incluir expected response en una parte del dataset mejora diagnóstico.

---

## 5) Elegir métodos de evaluación (qué mide cada uno)

### 5.1 Método “General quality” (el más RAG‑friendly)

Usa un modelo para puntuar calidad con criterios como:

- **Relevance**: responde a la pregunta.
- **Groundedness**: se basa en contexto recuperado (sin inventar).
- **Completeness**: cubre lo necesario.
- **Abstention**: si el agente intentó responder.

**Cuándo usarlo**
- preguntas abiertas,
- respuestas con múltiples formas correctas,
- evaluación de groundedness/abstención.

---

### 5.2 Compare meaning

- compara *el significado* con expected response.
- útil cuando hay variaciones de redacción válidas.

**Cuándo usarlo**
- definiciones y procedimientos que pueden reformularse.

---

### 5.3 Keyword match

- pasa/falla si aparecen “palabras clave esperadas”.
- útil para garantizar términos críticos.

**Cuándo usarlo**
- compliance: “debe mencionar X”
- IT: “debe incluir Y”

---

### 5.4 Text similarity

- puntúa similitud con expected response (coseno).
- útil como “termómetro” de cercanía textual.

---

### 5.5 Exact match

- pasa/falla si es exactamente igual.
- útil para:
  - números exactos,
  - códigos,
  - respuestas cortas fijas.

---

### 5.6 Capability use (muy útil en agentes híbridos)

- pasa/falla según si el agente usó:
  - un tool esperado,
  - o un topic esperado.

**Cuándo usarlo**
- cuando quieres asegurar:
  - “siempre consulta el sistema de tickets”
  - o “siempre llama a la acción X”.

---

## 6) Ejecutar evaluación y leer resultados (hands‑on)

### 6.1 Ejecutar

- Al final de crear o editar un test set, selecciona **Evaluate**.
- Puedes re‑ejecutar desde “Recent results”.
- Puedes exportar resultados a CSV.

> Nota operativa: los resultados no son “para siempre”; planifica exportación periódica si necesitas auditoría o histórico.

---

### 6.2 Interpretación de resultados: patrón diagnóstico

Cuando un test case falla, intenta clasificarlo así:

**Tipo A — Fallo de retrieval (la evidencia no aparece)**
- citaciones irrelevantes,
- respuesta se inventa o responde fuera de scope,
- groundedness bajo.

**Tipo B — Fallo de respuesta (la evidencia estaba, pero…)**
- la respuesta omite la excepción,
- confunde matices,
- responde con formato inadecuado,
- no abstiene aunque debería.

**Tipo C — Fallo de seguridad/política**
- el agente no accede por permisos,
- DLP bloquea un conector,
- el agente no puede usar cierto canal,
- moderación bloquea.

---

### 6.3 Ejemplo de “drill‑down”

Para un test case fallido:
1) ver la respuesta,
2) ver citaciones,
3) verificar si la respuesta coincide con expected/rúbrica,
4) apuntar “acción correctiva” (ver sección 7).

---

## 7) Mejora iterativa: del resultado al backlog

Copilot Studio (y cualquier RAG) mejora por **iteración**.

### 7.1 Lista de palancas (orden recomendado)

1) **Alcance del conocimiento**
   - ¿está entrando contenido fuera de dominio?
2) **Calidad del corpus**
   - duplicados, versiones antiguas, PDFs mal extraídos.
3) **Chunking / estructura**
   - demasiado grandes → ruido
   - demasiado pequeños → incompletitud
4) **Metadatos/filtros**
   - país, versión, status, clasificación.
5) **Instrucciones (prompt modification)**
   - abstención responsable,
   - formato de respuesta,
   - no inventar.
6) **Moderación**
   - equilibrar seguridad vs cobertura.

---

### 7.2 Plantilla de backlog (copiar/pegar)

| Test case | Tipo fallo | Causa probable | Cambio propuesto | Dueño | Prioridad |
|---|---|---|---|---|---|
| T12 | Retrieval | duplicados | dedupe + versionado | Data/KB | Alta |
| T03 | Respuesta | omite excepción | ajustar instrucciones | Maker | Media |
| T21 | Seguridad | permisos | auth Entra + fuente | Admin | Alta |

---

## 8) Mini‑lab (10 min): import + evaluación rápida

### Paso 1 — Crea un CSV con 10 preguntas
- 4 directas
- 2 paráfrasis
- 2 exact match
- 1 ambigua
- 1 fuera de scope

### Paso 2 — Importa el CSV como test set
- Método: General quality para casi todas
- Exact match para las 2 exactas

### Paso 3 — Ejecuta
- Mira el score
- Identifica 2 fallos
- Propón 2 acciones

---

## 9) Material de apoyo (oficial)

```text
Copilot Studio — Create or modify a test set:
https://learn.microsoft.com/en-us/microsoft-copilot-studio/analytics-agent-evaluation-create

Copilot Studio — Run tests and view results:
https://learn.microsoft.com/en-us/microsoft-copilot-studio/analytics-agent-evaluation-results

Copilot Studio — Choose evaluation methods:
https://learn.microsoft.com/en-us/microsoft-copilot-studio/analytics-agent-evaluation-overview

(Avanzado) Copilot Studio Kit — Configure tests:
https://learn.microsoft.com/en-us/microsoft-copilot-studio/guidance/kit-configure-tests
```
