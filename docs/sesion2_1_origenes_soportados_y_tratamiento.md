# SESIÓN 2.1 — Orígenes soportados y su tratamiento en Copilot Studio

> **Sesión 2 (total: 5 horas)** — *Orígenes de conocimiento y calidad de recuperación en Copilot Studio*  
> **Bloque 2.1**: Orígenes soportados y su tratamiento  
> **Objetivo del bloque**: que el alumnado entienda **qué orígenes soporta Copilot Studio**, **cómo los “trata” internamente** (indexación, recuperación, citaciones, permisos) y **qué implicaciones operativas** trae cada uno.

---

## 0) Objetivos de aprendizaje del bloque

Al finalizar este bloque, el participante podrá:

1. Enumerar los **orígenes de conocimiento** típicos en Copilot Studio (SharePoint/OneDrive, web corporativa, Dataverse, Azure AI Search) y sus **límites principales**.
2. Explicar las **dos grandes familias** de “tratamiento” de conocimiento:
   - **Conectar y recuperar en tiempo de consulta** (p. ej., SharePoint por URL/Graph Search).
   - **Ingerir y persistir en Dataverse** (p. ej., archivos subidos, OneDrive/SharePoint seleccionados como unstructured data).
3. Anticipar **cómo se generan citaciones** según el origen (y cuándo se rompen).
4. Identificar **qué orígenes respetan permisos por usuario** y cuáles **exponen contenido** a todos los usuarios del agente.


---

## 2) Marco general: cómo “funciona” el conocimiento en Copilot Studio

### 2.1 Knowledge sources + Generative answers: la pareja inseparable

En Copilot Studio, los **knowledge sources** se consumen normalmente a través de **generative answers**: el sistema busca contenido relevante y lo sintetiza en una respuesta “grounded”.  
La incorporación de conocimiento puede hacerse:
- **a nivel de agente** (página Knowledge), o
- **a nivel de topic** (nodo de generative answers).  

Por defecto, Copilot Studio crea un topic de sistema (“Conversational boosting”) con un nodo de generative answers que usa las fuentes del agente como fallback.  
Fuente: *Knowledge sources summary*.  
```text
https://learn.microsoft.com/en-us/microsoft-copilot-studio/knowledge-copilot-studio
```

**Regla de prioridad** (muy importante para diseño):
- Fuentes en **nodo** > fuentes en **agente** (agente = fallback).  
```text
https://learn.microsoft.com/en-us/microsoft-copilot-studio/nlu-documents
https://learn.microsoft.com/en-us/microsoft-copilot-studio/nlu-generative-answers-sharepoint-onedrive
```

---

### 2.2 Orquestación: “classic” vs “generative” (impacta límites y comportamiento)

Copilot Studio tiene **modos de orquestación** (classic/generative) que afectan:
- cuántas fuentes pueden consultarse,
- qué tipos de fuente están disponibles,
- cómo se priorizan/filtran fuentes cuando hay muchas.

En **generative orchestration**:
- si hay más de **25** knowledge sources, el sistema **filtra** internamente qué fuentes consultar usando descripciones.  
- los **archivos subidos** al agente no cuentan dentro de ese límite de 25.  
Fuente: *Knowledge sources summary*.
```text
https://learn.microsoft.com/en-us/microsoft-copilot-studio/knowledge-copilot-studio
```

**Implicación didáctica**: en entornos con muchas fuentes, la **descripción** de cada fuente (y su “scope”) afecta directamente al recall/precision del sistema.

---

## 3) SharePoint y OneDrive

### 3.1 Dos formas de “meter” SharePoint/OneDrive en el sistema

#### A) SharePoint por URL (GraphSearch / consulta en tiempo real)

Un nodo de generative answers con SharePoint suele funcionar así:
- se configura una **URL** base (ej.: `contoso.sharepoint.com/sites/policies`)
- cuando el usuario pregunta, el agente busca en esa URL **y en sus subpaths** (ej.: `.../sites` incluye `.../sites/policies`).  
Fuente: *Use SharePoint content for generative answers*.  
```text
https://learn.microsoft.com/en-us/microsoft-copilot-studio/nlu-generative-answers-sharepoint-onedrive
```

**Permisos**  
Las llamadas se realizan **en nombre del usuario** que conversa con el agente (Microsoft Entra ID), de modo que el agente solo debería “ver” lo que el usuario puede ver, si la configuración está bien.  
Fuente: *Use SharePoint content for generative answers*.

**Transcripciones**  
Un detalle clave: las respuestas que usan SharePoint como knowledge source **no se incluyen en conversation transcripts** (importante para auditoría).  
Fuente: *Use SharePoint content for generative answers*.

**Limitaciones prácticas (muy relevantes)**
- Solo **modern SharePoint pages**; páginas con SPFx no soportadas; ASPX clásico no usado.  
- Sin licencia de Microsoft 365 Copilot en el tenant, por limitaciones de memoria, generative answers puede quedar limitado a archivos pequeños (p. ej., umbral 7 MB indicado en límites).  
- Preguntas que referencian nombres de archivo explícitos (“¿qué dice file-name.pdf?”) **no se responden**.  
Fuente: *Quotas and limits — SharePoint limits*.  
```text
https://learn.microsoft.com/en-us/microsoft-copilot-studio/requirements-quotas
```

---

#### B) OneDrive/SharePoint “unstructured data” (ingestión a Dataverse)

Otra vía es seleccionar archivos o carpetas desde OneDrive/SharePoint mediante un selector. El sistema:
- **trae el contenido a Dataverse**,  
- lo procesa y lo indexa (chunks + embeddings),  
- y luego se recupera desde ese índice.  

Fuentes:
```text
https://learn.microsoft.com/en-us/microsoft-copilot-studio/knowledge-unstructured-data
https://learn.microsoft.com/en-us/microsoft-copilot-studio/knowledge-add-unstructured-data
```

**Diferencias importantes frente a “subir archivos” manualmente**
- Con OneDrive puedes añadir **carpetas**, no solo archivos sueltos.
- El contenido puede estar **sincronizado** (si cambia el archivo, se actualiza).
- El acceso se valida con credenciales del usuario (permission check) antes de responder.  
Fuente: *Add unstructured data as a knowledge source*.

**Ojo**: aunque el contenido se almacena en Dataverse para indexación, el sistema puede seguir validando permisos de acceso antes de usarlo, según el tipo de origen y configuración (esto es clave para compliance).

---

### 3.2 Consejos de diseño para SharePoint/OneDrive (empresa)

1) Si necesitas **frescura** (documentos cambian) y **permisos por usuario**:
- prioriza SharePoint/OneDrive conectados (URL/Graph o unstructured data) sobre “archivo subido”.

2) Si necesitas **snapshot controlado** (una versión concreta) y no te importa permission trimming:
- puede valer “archivo subido como conocimiento” (ver 2.2).

3) Si el contenido es grande:
- revisa límites; para SharePoint, los límites dependen de licencia/tenant graph grounding.  
Fuente: *Quotas and limits — SharePoint limits*.

---

## 4) Web corporativa

### 4.1 “Public website knowledge source” (búsqueda acotada)

Copilot Studio puede usar un **sitio web público** como knowledge source, buscando con Bing **pero limitando** los resultados a dominios/URLs proporcionados.  
Fuente: *Knowledge sources summary*.
```text
https://learn.microsoft.com/en-us/microsoft-copilot-studio/knowledge-copilot-studio
```

**Buenas prácticas**
- Usa un dominio que sea claramente “fuente de verdad” (portal corporativo, base de conocimiento pública).
- Evita incluir dominios demasiado amplios si la información es heterogénea.
- Ten presente que el sistema depende de lo que Bing indexa (latencia de indexación externa).

**Riesgo importante (para explicar en clase)**
Aunque hay mitigaciones, es posible que respuestas incluyan contenido de **webs distintas** a la seleccionada.  
Fuente: *FAQ for generative answers*.  
```text
https://learn.microsoft.com/en-us/microsoft-copilot-studio/faqs-generative-answers
```

---

### 4.2 “Use information from the web / Web Search” (búsqueda amplia, opcional)

En modo generativo, existe un ajuste para permitir que el agente busque “en la web” más allá de tus fuentes.  
Esto puede:
- ayudar con preguntas “de actualidad”,
- pero romper el **scope** del agente y reducir “groundedness” corporativa.

Fuente: *Knowledge sources summary* (sección Web Search).  
```text
https://learn.microsoft.com/en-us/microsoft-copilot-studio/knowledge-copilot-studio
```

**Recomendación didáctica**: en agentes empresariales internos, por defecto tiende a ser más seguro:
- mantener Web Search **apagado**,
- y usar fuentes corporativas controladas.

---

## 5) Dataverse como repositorio documental

### 5.1 Dataverse para documentos “subidos” (knowledge files)

Cuando subes un documento como knowledge source:
- se almacena “de forma segura” en Dataverse,
- el número de archivos depende del storage del entorno,
- el tamaño por archivo llega a 512 MB (según límites del producto).  
Fuentes:
```text
https://learn.microsoft.com/en-us/microsoft-copilot-studio/knowledge-add-file-upload
https://learn.microsoft.com/en-us/microsoft-copilot-studio/nlu-documents
https://learn.microsoft.com/en-us/microsoft-copilot-studio/requirements-quotas
```

**Tratamiento interno (simplificado)**
- ingestión → chunking → embeddings → índice semántico → retrieval por chunks.  
Fuente: *Unstructured data as a knowledge source*.

---

### 5.2 Dataverse para datos en tablas (conocimiento “semi‑estructurado”)

También puedes añadir **tablas Dataverse** como knowledge source para que el agente responda usando registros/tablas.

Puntos operativos clave:
- requiere Dataverse search,
- máximo típico: **hasta 15 tablas** por knowledge source,
- puedes mejorar calidad con **synonyms** y **glossary terms** (especialmente para columnas con códigos/números).  
Fuente: *Add Dataverse tables as a knowledge source*.  
```text
https://learn.microsoft.com/en-us/microsoft-copilot-studio/knowledge-add-dataverse
```

> Nota: aunque Dataverse tables es “dato estructurado”, aquí se trata como conocimiento para grounding. La explotación avanzada de datos estructurados (queries, herramientas) se amplía en la Sesión 3.

---

## 6) Visión general: integración con Azure AI Search

### 6.1 ¿Por qué Azure AI Search?

Azure AI Search se usa cuando necesitas:
- control fino de indexación (chunking, embeddings, fields),
- escalabilidad y rendimiento de búsqueda,
- búsqueda híbrida y reranking (semantic ranker),
- integración con otros pipelines (enriquecimiento, OCR, etc.).

Copilot Studio permite añadir **Azure AI Search como knowledge source**.  
Fuente:
```text
https://learn.microsoft.com/en-us/microsoft-copilot-studio/knowledge-azure-ai-search
```

### 6.2 Requisitos y restricciones (para que no falle la demo)

Según documentación oficial:
- Copilot Studio **no soporta** índices de Azure AI Search configurados para **virtual networks**.  
- Solo se puede añadir **un** vector index por conexión.
- Soporta **integrated vectorization** (usar el mismo modelo de embeddings en indexación y en runtime).
- Soporta **semantic ranker** (si lo configuras en Azure AI Search).
Fuente: *Add Azure AI Search as a knowledge source*.

### 6.3 Citaciones con Azure AI Search (clave para calidad y auditoría)

Para devolver citaciones:
- el índice debe incluir un campo URL real al documento,
- si existe `metadata_storage_path`, Copilot Studio lo interpreta como citación; si no, usa cualquier campo con URL completa.  
Fuente: *Add Azure AI Search as a knowledge source*.

**Checklist de citaciones en Azure AI Search**
- [ ] Campo URL en índice (por documento o chunk)
- [ ] URL accesible por el usuario (si apunta a un recurso restringido, el usuario no podrá abrir la citación)
- [ ] Modelo de embeddings coherente con el de runtime (ideal: integrated vectorization)

---

## 7) Matriz comparativa rápida (para el instructor)

| Origen                           | ¿Se ingiere a Dataverse? | Frescura               | Permisos por usuario          | Citaciones         | Riesgo típico                                                |
| -------------------------------- | ------------------------ | ---------------------- | ----------------------------- | ------------------ | ------------------------------------------------------------ |
| SharePoint por URL (GraphSearch) | No necesariamente        | Alta                   | Sí (Entra ID)                 | Sí                 | límites de páginas/archivos; preguntas por nombre de archivo |
| OneDrive/SharePoint unstructured | Sí (para indexación)     | Media‑Alta (sync)      | Validación de acceso          | Sí                 | consumo de storage; sensibilidad labels bloquean indexación  |
| Archivos subidos como knowledge  | Sí                       | Baja (estático)        | No (exposición amplia)        | Sí                 | fuga de contenido a usuarios no autorizados                  |
| Web corporativa (dominio)        | No                       | Depende de Bing        | No (es público)               | Sí                 | contenido fuera de dominio, ruido, SEO                       |
| Dataverse tablas                 | No (dato local)          | Alta                   | Sí (según permisos Dataverse) | Variable           | semántica pobre en columnas con códigos si no hay synonyms   |
| Azure AI Search                  | No (externo)             | Alta (pipeline propio) | Depende (tu auth)             | Sí (si mapeas URL) | sin URL/citation mapping, citaciones vacías                  |

---

## 8) Mini‑actividad (10 min): elige el origen correcto

En grupos de 2–3, decidid:

### Caso A — Política de RRHH (solo empleados, con permisos por país)
- ¿SharePoint? ¿OneDrive? ¿archivo subido?  
- ¿Qué harías con versiones?

### Caso B — Base de conocimiento pública (clientes externos)
- ¿Web corporativa? ¿Archivo subido?  
- ¿Qué riesgos tienes con “Use information from the web”?

### Caso C — Manual técnico (muy largo, cientos de PDFs) + necesidad de búsqueda híbrida
- ¿Azure AI Search? ¿Qué necesitas para citaciones?

**Entrega (1 minuto por grupo)**: elección + 2 razones + 1 riesgo + 1 mitigación.

---

## 9) Cierre: “lo que hay que recordar”

1) La fuente de conocimiento no es neutra: cambia **seguridad, frescura, calidad y trazabilidad**.
2) “Subir archivos” es rápido pero puede ser el peor caso en seguridad.
3) SharePoint/OneDrive suelen ser la vía “enterprise” cuando hay permisos por usuario.
4) Azure AI Search entra cuando necesitas control/escala/híbrido.
5) Cuantas más fuentes, más importante es **describir bien** el scope de cada una.

---

## 10) Lecturas oficiales recomendadas

```text
Knowledge sources summary:
https://learn.microsoft.com/en-us/microsoft-copilot-studio/knowledge-copilot-studio

SharePoint/OneDrive as knowledge:
https://learn.microsoft.com/en-us/microsoft-copilot-studio/nlu-generative-answers-sharepoint-onedrive

Upload files as knowledge source:
https://learn.microsoft.com/en-us/microsoft-copilot-studio/knowledge-add-file-upload
https://learn.microsoft.com/en-us/microsoft-copilot-studio/nlu-documents

Unstructured data & OneDrive connector behavior:
https://learn.microsoft.com/en-us/microsoft-copilot-studio/knowledge-unstructured-data
https://learn.microsoft.com/en-us/microsoft-copilot-studio/knowledge-add-unstructured-data

Dataverse tables as knowledge:
https://learn.microsoft.com/en-us/microsoft-copilot-studio/knowledge-add-dataverse

Azure AI Search as knowledge:
https://learn.microsoft.com/en-us/microsoft-copilot-studio/knowledge-azure-ai-search

Limits & SharePoint constraints:
https://learn.microsoft.com/en-us/microsoft-copilot-studio/requirements-quotas

FAQ generative answers (riesgos web):
https://learn.microsoft.com/en-us/microsoft-copilot-studio/faqs-generative-answers
```
