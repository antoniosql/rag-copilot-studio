# SESIÓN 2.3 — Factores que influyen en la calidad del sistema 
> **Sesión 2 (total: 5 horas)** — *Orígenes de conocimiento y calidad de recuperación en Copilot Studio*  
> **Bloque 2.3**: Factores que influyen en la calidad del sistema  
> **Objetivo del bloque**: dar al alumnado una “caja de herramientas” práctica para diagnosticar y mejorar la calidad de un sistema RAG en Copilot Studio, centrándose en lo que más impacta: **documentos**, **duplicidad/versiones**, **metadatos**, **scope**.

---

## 1) Objetivos de aprendizaje del bloque

Al finalizar este bloque, el participante podrá:

1. Evaluar la **calidad y estructura** de documentos desde la óptica de recuperación (no de estética).
2. Reconocer patrones de degradación por **duplicidad** y por **mezcla de versiones**.
3. Diseñar y usar metadatos/semántica disponible en Copilot Studio:
   - descripción de knowledge sources (orquestación),
   - synonyms/glossary en Dataverse.
4. Delimitar correctamente el **alcance del conocimiento** (scoping) y evitar “RAG que responde de todo”.
5. Aplicar un método de troubleshooting: **síntoma → causa probable → intervención**.

---

## 2) Qué es “calidad” en un sistema RAG (visión práctica)

En Copilot Studio, la calidad se observa en 3 planos:

1) **Recuperación**: ¿se trae el chunk/documento correcto?
2) **Síntesis**: ¿la respuesta usa lo recuperado sin inventar?
3) **Trazabilidad**: ¿las citaciones ayudan a verificar?

Recordatorio: generative answers se diseñó para buscar en fuentes (web, SharePoint, custom data) y resumir en una respuesta; **no hace un chequeo automático de exactitud**.  
Fuente:
```text
https://learn.microsoft.com/en-us/microsoft-copilot-studio/faqs-generative-answers
```

---

## 3) Calidad y estructura del documento

### 3.1 La regla de oro

> **El retrieval trabaja sobre texto y “trozos” (chunks).**  
> Si el texto está mal extraído o la estructura no existe, el sistema recupera peor.

Copilot Studio (en conocimiento “unstructured”) chunkea, crea embeddings e indexa para recuperar trozos relevantes.  
Fuente:
```text
https://learn.microsoft.com/en-us/microsoft-copilot-studio/knowledge-unstructured-data
```

---

### 3.2 Señales de documento “RAG‑friendly”

**Estructura**
- Títulos claros (H1/H2/H3 o estilos en Word).
- Secciones cortas con subtítulos (“Procedimiento”, “Excepciones”, “Definiciones”).
- Listas numeradas cuando hay pasos.

**Contenido**
- Definiciones explícitas (“Se entiende por…”) antes de excepciones.
- Pocas referencias cruzadas del tipo “ver Anexo 7” sin contexto.
- Evitar dependencias fuertes de tablas “sin encabezado”.

**Formato**
- PDF con texto seleccionable (no escaneado).
- En imágenes dentro de PDF, añadir **alt‑text** si se espera que el agente responda sobre ellas.  
Fuente (support de imágenes anotadas en PDFs):
```text
https://learn.microsoft.com/en-us/microsoft-copilot-studio/knowledge-add-file-upload
```

---

### 3.3 Antipatrones que destruyen la recuperación

1) **PDF escaneado sin OCR**  
→ el texto extraído es pobre o nulo.

2) **Tablas complejas sin encabezado / multi‑columna**  
→ el embedding “pierde” el significado de fila/columna.

3) **Documentos que mezclan temas** (p. ej., RRHH + IT + Legal en un mismo PDF de 80 páginas)  
→ chunks mezclan conceptos, baja precision.

4) **Contenido repetido** (disclaimers, pies de página largos, logos en cada página)  
→ el retriever trae “ruido repetido”.

5) **Jerga interna sin glosario**  
→ queries de usuario no conectan semánticamente con el texto (“baja” vs “IL”, “Solicitud 7B”, etc.).

---

### 3.4 Acciones rápidas para mejorar documentos (sin “ingeniería”)

- Convertir PDF escaneado a docx o PDF con OCR (ideal: fuente original).
- Añadir índices y títulos (al menos H2).
- Reescribir secciones “mega” en subsecciones de 1–2 páginas.
- Mover anexos a documentos separados si no se consultan a menudo.
- Insertar glosario y acrónimos en el propio documento (si no usas Dataverse glossary).

---

## 4) Duplicidad y control de versiones

### 4.1 Por qué esto es el principal problema en empresa

Cuando conviven:
- v1 (2023),
- v2 (2024),
- v3 (2025),

el sistema puede recuperar “la más similar” aunque sea obsoleta.

**Además**: en Copilot Studio, al subir archivos como conocimiento:
- subir el mismo nombre crea copias y no sobreescribe.  
Fuente:
```text
https://learn.microsoft.com/en-us/microsoft-copilot-studio/nlu-documents
```

Resultado típico:
- el agente responde con una mezcla (“en un caso dice 22 días, en otro 23 días”).

---

### 4.2 Síntomas de duplicidad / version drift

- Citaciones apuntan a **dos documentos** con títulos similares.
- Respuestas inconsistentes en pruebas repetidas.
- Cambias la política y el agente sigue citando la versión anterior.

---

### 4.3 Estrategias recomendadas (enterprise)

**Estrategia 1 — “Single source of truth” (SharePoint/OneDrive)**
- Evitar subir PDFs duplicados al agente.
- Apuntar a una ubicación única con control de versiones.
- Si usas OneDrive como unstructured data, los cambios pueden sincronizarse.  
Fuente:
```text
https://learn.microsoft.com/en-us/microsoft-copilot-studio/knowledge-add-unstructured-data
```

**Estrategia 2 — Convención de nombres + descripciones**
- Título: `POL-RRHH-VACACIONES-ES`
- Descripción: “Política vigente España, aprobada 2025-09-01, reemplaza v2.1”
- Las descripciones ayudan a orquestación (especialmente en modo generativo).  
Fuente:
```text
https://learn.microsoft.com/en-us/microsoft-copilot-studio/knowledge-add-file-upload
```

**Estrategia 3 — “Despublicar” versiones antiguas**
- Elimina knowledge sources obsoletos (si son files subidos).
- Para SharePoint/OneDrive, archivar o controlar versiones en el repositorio.

**Estrategia 4 — Azure AI Search con campo de versionado y filtros**
- Si usas tu propio índice, puedes controlar:
  - `version`,
  - `valid_from/valid_to`,
  - `is_current`,
  - `department`.
- Además, mapeas citaciones y controlas ranking con semantic ranker.  
Fuente:
```text
https://learn.microsoft.com/en-us/microsoft-copilot-studio/knowledge-azure-ai-search
```

---

## 5) Uso adecuado de metadatos (y “semántica auxiliar”)

### 5.1 Descripción de knowledge sources (impacta orquestación)

Copilot Studio recomienda describir la fuente “con detalle”, porque la descripción ayuda a la orquestación generativa y a la selección interna cuando hay muchas fuentes.  
Fuente:
```text
https://learn.microsoft.com/en-us/microsoft-copilot-studio/knowledge-add-file-upload
https://learn.microsoft.com/en-us/microsoft-copilot-studio/knowledge-add-dataverse
```

**Plantilla de descripción recomendada**
- Qué contiene
- Para quién aplica (país, negocio)
- Vigencia/versión
- Qué excluye

Ejemplo:
> “Política de vacaciones para empleados en España. Aplica a contratos indefinidos y temporales. Vigente desde 2025‑09‑01. No aplica a Portugal.”

---

### 5.2 Synonyms y Glossary en Dataverse (mejora intent detection)

Para Dataverse tables:
- puedes añadir **synonyms** a columnas,
- y **glossary terms** para acrónimos/jerga interna,
- y esto mejora la interpretación de la consulta y la generación de respuesta.  
Fuente:
```text
https://learn.microsoft.com/en-us/microsoft-copilot-studio/knowledge-add-dataverse
```

**Caso típico**  
Columnas con nombres técnicos o códigos (`cr_123_abc`) requieren sinónimos y descripción para que la IA entienda qué significan (especialmente si contienen números).  
Fuente: *Add Dataverse tables as a knowledge source*.

**Latencia de actualización**
- Puede tardar hasta ~15 min en estar disponible una actualización de glossary.  
Fuente: *Add Dataverse tables as a knowledge source*.

---

## 6) Alcance del conocimiento y delimitación temática (scoping)

### 6.1 El enemigo: “un agente que sabe de todo”

Cuanto más amplio el scope:
- más ruido,
- más riesgo de mezclar documentos,
- más citaciones irrelevantes,
- más difícil gobernanza.

**Solución**: limitar fuentes por intención usando nodos de generative answers en topics.

Recuerda:
- fuentes del nodo tienen prioridad sobre las del agente.  
Fuente:
```text
https://learn.microsoft.com/en-us/microsoft-copilot-studio/nlu-documents
```

---

### 6.2 Límites de producto que afectan scope

- En generative mode: hasta 25 websites/URLs por tipo, con reglas y filtrado interno.  
Fuente:
```text
https://learn.microsoft.com/en-us/microsoft-copilot-studio/knowledge-copilot-studio
```

- SharePoint: limitaciones de páginas, SPFx, tamaños y preguntas por nombre de archivo.  
Fuente:
```text
https://learn.microsoft.com/en-us/microsoft-copilot-studio/requirements-quotas
```

**Conclusión**: scoping no es solo calidad; también es **evitar límites** y “fallos silenciosos” (fuentes que aparecen “Ready” pero no responden).

---

### 6.3 Web search y control de scope

Si activas “Use information from the web”, el agente puede buscar más allá de tus fuentes definidas (web amplio).  
Fuente:
```text
https://learn.microsoft.com/en-us/microsoft-copilot-studio/knowledge-copilot-studio
```

Además, aunque el sistema está diseñado para consultar la web seleccionada, el FAQ advierte que puede incluir contenido de webs distintas.  
Fuente:
```text
https://learn.microsoft.com/en-us/microsoft-copilot-studio/faqs-generative-answers
```

---

## 7) Playbook de diagnóstico: síntoma → causa → intervención

### 7.1 Citaciones irrelevantes
**Causas probables**
- scope demasiado amplio,
- documentos con estructura pobre,
- duplicidad,
- web search habilitado.

**Intervenciones**
- mover la respuesta a un nodo por intención con fuentes específicas,
- limpiar duplicados,
- mejorar títulos/secciones del documento.

---

### 7.2 Respuesta incompleta (falta el matiz/excepción)
**Causas probables**
- chunking automático separó la excepción del contexto,
- el documento tiene excepciones dispersas,
- la pregunta es ambigua.

**Intervenciones**
- reestructurar documento: “Regla general” + “Excepciones” juntas,
- crear un FAQ complementario con excepciones explícitas.

---

### 7.3 Respuestas inconsistentes
**Causas probables**
- versiones coexistentes,
- mezcla de fuentes (web + doc),
- múltiples documentos con frases similares.

**Intervenciones**
- eliminar/archivar versiones antiguas,
- fortalecer naming + descripciones,
- usar repositorio único (SharePoint/OneDrive) o Azure AI Search con filtros.

---

### 7.4 “No responde” o responde con generalidades
**Causas probables**
- fuente no indexa por restricción (sensitivity label, password),
- SharePoint limits (páginas no soportadas),
- el agente no encuentra trozos relevantes y se apoya en conocimiento general (si permitido).

**Intervenciones**
- verificar restricciones de indexación (labels/password),
- revisar limits y formatos soportados,
- desactivar “general knowledge” si quieres abstención.

---

## 8) Checklist de mejora rápida (para el equipo)

**Documentos**
- [ ] Títulos y secciones consistentes
- [ ] PDF con texto seleccionable
- [ ] Tablas con encabezados repetidos
- [ ] Alt‑text para imágenes relevantes

**Versionado**
- [ ] Una fuente “vigente” por tema
- [ ] Convención de nombres + descripciones con vigencia
- [ ] Retirada de versiones antiguas

**Metadatos**
- [ ] Descripción detallada por knowledge source
- [ ] Synonyms/glossary en Dataverse (si aplica)

**Scope**
- [ ] Nodos por intención con fuentes concretas
- [ ] Web search controlado (o desactivado)

---

## 9) Lecturas oficiales recomendadas

```text
Unstructured data: chunking + embeddings + retrieval por chunks:
https://learn.microsoft.com/en-us/microsoft-copilot-studio/knowledge-unstructured-data

Upload files & cómo influyen descripciones:
https://learn.microsoft.com/en-us/microsoft-copilot-studio/knowledge-add-file-upload

Dataverse tables: synonyms + glossary:
https://learn.microsoft.com/en-us/microsoft-copilot-studio/knowledge-add-dataverse

SharePoint limits & restricciones:
https://learn.microsoft.com/en-us/microsoft-copilot-studio/requirements-quotas

FAQ generative answers (riesgos y límites):
https://learn.microsoft.com/en-us/microsoft-copilot-studio/faqs-generative-answers
```
