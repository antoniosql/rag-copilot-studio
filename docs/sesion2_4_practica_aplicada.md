# SESIÓN 2.4 — Práctica aplicada

> **Sesión 2 (total: 5 horas)** — *Orígenes de conocimiento y calidad de recuperación en Copilot Studio*  
> **Bloque 2.4**: Práctica aplicada  
> **Objetivo**: configurar **múltiples fuentes** en un agente, comparar respuestas y citaciones, e identificar limitaciones reales del sistema (producto + diseño).

---

## 0) Resultados esperados (qué debe lograr el alumno)

Al terminar, cada alumno o pareja tendrá:

- Un agente con **al menos 2 fuentes** configuradas (ideal 3–4):
  - 1 documento subido (Dataverse file)
  - 1 SharePoint/OneDrive (si disponible)
  - 1 web corporativa (opcional)
  - Azure AI Search (opcional avanzado)
- Un topic “Comparador” con **nodos de generative answers** por fuente.
- Un mini‑informe (tabla) comparando **exactitud, citaciones, latencia y límites**.

---

## 1) Preparación (5 min)

### Material mínimo
- Un PDF/DOCX corto (3–10 páginas) para subir como conocimiento.
- (Opcional) URL de un sitio SharePoint con páginas modernas / docs.
- (Opcional) Dominio de web corporativa pública (o una web de documentación).
- (Opcional) Un índice en Azure AI Search con campo URL de citación.

### Requisitos de entorno
- Permisos para crear/editar agentes.
- Si vas a usar archivos subidos como conocimiento o Dataverse tables:
  - **Dataverse search** habilitado (requisito para varias capacidades).  
Fuentes:
```text
https://learn.microsoft.com/en-us/microsoft-copilot-studio/knowledge-add-file-upload
https://learn.microsoft.com/en-us/microsoft-copilot-studio/knowledge-add-dataverse
```


---

## 3) Paso a paso: configurar múltiples fuentes (15 min)

> Consejo: si un origen no está disponible (por licencias o accesos), no te detengas: sigue con 2 fuentes y usa el apartado “Variantes”.

### 3.1 Fuente A — Subir un documento como knowledge source (Dataverse file)

1) Entra en tu agente → **Knowledge** (o “Add knowledge” desde Overview).
2) Selecciona **Upload files** (archivo local) y sube el documento.
3) Añade **nombre** y **descripción** (importante para orquestación y scope).
4) Espera a estado **Ready**.

Notas oficiales útiles:
- El archivo se almacena en Dataverse y el límite depende del storage del entorno.  
- Subir el mismo nombre crea copias; no sobreescribe.  
Fuentes:
```text
https://learn.microsoft.com/en-us/microsoft-copilot-studio/knowledge-add-file-upload
https://learn.microsoft.com/en-us/microsoft-copilot-studio/nlu-documents
```

⚠️ **Seguridad**  
El contenido del archivo subido como conocimiento puede quedar accesible a **todos** los usuarios del agente.  
Fuente (advertencia):
```text
https://learn.microsoft.com/en-us/microsoft-copilot-studio/nlu-documents
```

---

### 3.2 Fuente B — SharePoint por URL (si disponible)

1) Add knowledge → SharePoint
2) Introduce una URL lo más específica posible (ej.: `.../sites/policies/hr`)
3) Asegura que el agente está configurado para autenticar con Microsoft (Entra ID).
4) Prueba con una pregunta simple para validar citaciones.

Notas oficiales:
- SharePoint por URL busca también subpaths.
- Respuestas usando SharePoint como knowledge source **no** entran en conversation transcripts.  
Fuente:
```text
https://learn.microsoft.com/en-us/microsoft-copilot-studio/nlu-generative-answers-sharepoint-onedrive
```

💡 **Límites que debes “intentar romper” en la práctica**
- Preguntar por nombre de archivo explícito no suele funcionar (“¿Qué dice file-name.pdf?”).  
- Páginas clásicas ASPX no se usan; SPFx no soportado.  
Fuente:
```text
https://learn.microsoft.com/en-us/microsoft-copilot-studio/requirements-quotas
```

---

### 3.3 Fuente C — Web corporativa (opcional)

1) Add knowledge → Public website
2) Añade el dominio o URLs permitidas (ideal: un único dominio).
3) Incluye una descripción que explique qué hay en ese sitio.

Riesgo a observar:
- Aunque la búsqueda se diseña para el sitio seleccionado, puede colarse contenido de otras webs.  
Fuente:
```text
https://learn.microsoft.com/en-us/microsoft-copilot-studio/faqs-generative-answers
```

---

### 3.4 Fuente D — Azure AI Search (opcional avanzado)

1) Add knowledge → Featured → Azure AI Search
2) Crea conexión (Access Key / Entra ID, etc.)
3) Selecciona el índice vectorial (solo uno por conexión)
4) Verifica que tu índice incluye un campo URL para citaciones (`metadata_storage_path` u otro URL completo)

Fuente:
```text
https://learn.microsoft.com/en-us/microsoft-copilot-studio/knowledge-azure-ai-search
```

---

## 4) Construir el topic “Comparador” (15 min)

### 4.1 Idea

Crear un topic con **3 nodos de generative answers**, cada uno configurado con:
- **Search only selected sources**
- una sola fuente distinta (A, B, C o D)
- guardar el resultado en variables:
  - `Answer_A`, `Answer_B`, etc.

Luego mostrar un mensaje comparativo al usuario (o al tester).

> Recordatorio: fuentes del nodo tienen prioridad sobre fuentes del agente.  
Fuente:
```text
https://learn.microsoft.com/en-us/microsoft-copilot-studio/nlu-documents
```

---

### 4.2 Pasos (alto nivel)

1) Topics → New topic → “Comparador de fuentes”
2) Trigger phrases:
   - “comparar fuentes”
   - “test conocimiento”
3) Añade un Question node para capturar la pregunta del usuario:
   - “¿Qué quieres preguntar?”
   - guarda en `Topic.userQuestion`

4) Añade nodo Generative answers (Fuente A)
   - Input: `Topic.userQuestion`
   - Data sources: selecciona solo “Documento subido (A)”
   - (Opcional) desactiva “Send a message” para capturar salida en variable

5) Repite para Fuente B y C (y/o D)

6) Message node final:
   - muestra 3 respuestas separadas con un prefijo:
     - “Respuesta desde Documento subido: …”
     - “Respuesta desde SharePoint: …”
     - “Respuesta desde Web: …”

**Tip**  
Si publicas en Teams y personalizas la respuesta, puede requerir renderizado explícito de citaciones según limitación documentada para custom output.  
Fuente:
```text
https://learn.microsoft.com/en-us/microsoft-copilot-studio/nlu-boost-node
```

---

## 5) Test set y registro comparativo (20 min)

### 5.1 Construye un test set mínimo (8 preguntas)

Incluye:
- 3 preguntas respondibles por el documento subido
- 3 preguntas respondibles por SharePoint (o por web)
- 1 pregunta de “nombre de archivo” (para ver limitación SharePoint)
- 1 pregunta fuera de scope (para observar abstención o error)

Ejemplos genéricos:
1) “¿Cuál es el procedimiento para X?”
2) “¿Qué excepciones hay para Y?”
3) “¿Qué dice el apartado Z?”
4) “¿Qué política aplica en España?”
5) “¿Qué dice file-name.pdf sobre mitigación?” (limitación)
6) “¿Cuándo se actualizó esta política?” (frescura/versionado)
7) “¿Qué es el acrónimo ABC?” (glosario)
8) “¿Cuál es la capital de…?” (fuera de scope, debería abstener o no usar fuentes)

---

### 5.2 Plantilla de registro (copiar/pegar)

| Pregunta | Fuente A (Doc subido) | Fuente B (SP/OD) | Fuente C (Web) | Exactitud | Citaciones | Latencia | Observaciones |
|---|---|---|---|---:|---:|---:|---|
| P1 |  |  |  |  |  |  |  |
| P2 |  |  |  |  |  |  |  |
| P3 |  |  |  |  |  |  |  |
| P4 |  |  |  |  |  |  |  |
| P5 |  |  |  |  |  |  |  |
| P6 |  |  |  |  |  |  |  |
| P7 |  |  |  |  |  |  |  |
| P8 |  |  |  |  |  |  |  |

---

## 6) Identificación de limitaciones (5 min)

En la tabla, marca con ⚠️ cuando observes alguno:

- ⚠️ Citación irrelevante
- ⚠️ Respuesta mezcla versiones
- ⚠️ SharePoint no responde a pregunta con nombre de archivo
- ⚠️ Respuesta trae “conocimiento general” (si está habilitado)
- ⚠️ Web trae contenido no corporativo (posible “leak” de dominio)
- ⚠️ Falta de citaciones (o citaciones rotas por URLs sin acceso)

**Guía oficial para límites**
```text
https://learn.microsoft.com/en-us/microsoft-copilot-studio/requirements-quotas
https://learn.microsoft.com/en-us/microsoft-copilot-studio/faqs-generative-answers
```

---

## 7) Variantes si te falta algún origen

- Sin SharePoint: compara “Documento subido” vs “Web corporativa”.
- Sin web: compara “Documento subido” vs “SharePoint”.
- Sin Azure AI Search: omite Fuente D y usa 2–3 fuentes.

---

## 8) Cierre: 2 mejoras accionables (1 min por equipo)

Cada equipo entrega:
1) **Una limitación** observada (con ejemplo).
2) **Dos mejoras** que implementarían, elegidas de:
   - delimitar scope con nodos por intención,
   - retirar duplicados o versiones antiguas,
   - mejorar estructura del documento,
   - añadir descripción más precisa a la fuente,
   - (si Dataverse) añadir synonyms/glossary,
   - (si Azure AI Search) mapear URLs de citación.

---

## 9) Lecturas oficiales útiles para el lab

```text
Knowledge sources summary:
https://learn.microsoft.com/en-us/microsoft-copilot-studio/knowledge-copilot-studio

SharePoint as knowledge (URL/subpaths, transcripts):
https://learn.microsoft.com/en-us/microsoft-copilot-studio/nlu-generative-answers-sharepoint-onedrive

Upload files as knowledge + límites:
https://learn.microsoft.com/en-us/microsoft-copilot-studio/knowledge-add-file-upload
https://learn.microsoft.com/en-us/microsoft-copilot-studio/nlu-documents

Azure AI Search as knowledge (citations mapping):
https://learn.microsoft.com/en-us/microsoft-copilot-studio/knowledge-azure-ai-search

Limits (SharePoint constraints):
https://learn.microsoft.com/en-us/microsoft-copilot-studio/requirements-quotas
```
