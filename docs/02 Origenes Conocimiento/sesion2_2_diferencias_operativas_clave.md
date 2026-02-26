# SESIÓN 2.2 — Diferencias operativas clave 

> **Sesión 2 (total: 5 horas)** — *Orígenes de conocimiento y calidad de recuperación en Copilot Studio*  
> **Bloque 2.2**: Diferencias operativas clave  
> **Objetivo del bloque**: que el alumnado distinga claramente **qué significa cada “tipo” de conocimiento** en Copilot Studio, cómo cambia el comportamiento del agente y cuáles son las **implicaciones de seguridad y exposición**.

---

## 1) Objetivos de aprendizaje del bloque

Al finalizar este bloque, el participante podrá:

1. Diferenciar con precisión:
   - **Knowledge source persistente**
   - **Documento incorporado al agente**
   - **Archivo subido por el usuario durante la conversación**
2. Identificar para cada tipo:
   - cómo se indexa,
   - cómo se actualiza,
   - si respeta permisos,
   - qué riesgos de exposición existen,
   - cómo se audita (citaciones / transcripciones / trazabilidad).
3. Elegir el tipo correcto según 3 restricciones típicas: **seguridad**, **frescura** y **control de cambios**.

---

## 2) Definiciones (sin ambigüedad)

### 2.1 Knowledge source persistente

**Definición**  
Un **knowledge source persistente** es una fuente configurada en el agente (o en un nodo de generative answers) que el agente puede consultar en múltiples conversaciones, de forma continua.

Ejemplos típicos:
- SharePoint por URL
- Public website por dominio
- Dataverse (tablas)
- Azure AI Search
- Conectores para unstructured data

**Dónde vive**
- A nivel de agente (Knowledge page) o
- a nivel de topic (nodo Generative Answers).  

**Prioridad**  
Las fuentes configuradas en un nodo de generative answers **tienen prioridad** y las del agente actúan como **fallback**.  
Fuentes:
```text
https://learn.microsoft.com/en-us/microsoft-copilot-studio/knowledge-copilot-studio
https://learn.microsoft.com/en-us/microsoft-copilot-studio/nlu-documents
https://learn.microsoft.com/en-us/microsoft-copilot-studio/nlu-generative-answers-sharepoint-onedrive
```

---

### 2.2 Documento incorporado al agente (archivo subido como conocimiento)

**Definición**  
Es un archivo que el *maker* sube como knowledge source (a nivel de agente o topic), y que pasa a formar parte del “conocimiento” del agente.

**Dónde vive**
- Se almacena en **Dataverse**, dentro del entorno del agente.  
Fuentes:
```text
https://learn.microsoft.com/en-us/microsoft-copilot-studio/knowledge-add-file-upload
https://learn.microsoft.com/en-us/microsoft-copilot-studio/nlu-documents
```

**Propiedad clave (y riesgo)**
- El contenido subido queda disponible para **cualquier usuario** que chatee con el agente, **independientemente** de permisos del archivo original.  
Fuente (advertencia explícita):
```text
https://learn.microsoft.com/en-us/microsoft-copilot-studio/nlu-documents
```

---

### 2.3 Archivo subido por el usuario durante la conversación

**Definición**  
El usuario final adjunta un archivo en el chat (paperclip), y el agente puede:
- **analizarlo en la conversación**, y/o
- **pasarlo** a acciones, flujos, conectores o tools.

**Activación**
- Se habilita en Settings → Generative AI → File uploads.  
Fuente:
```text
https://learn.microsoft.com/en-us/microsoft-copilot-studio/image-input-analysis
```

**Acceso en el runtime**
- El archivo suele estar disponible como parte de `System.Activity.Attachments` y puede solicitarse con un **Question node** identificando tipo **File**.  
Fuente:
```text
https://learn.microsoft.com/en-us/microsoft-copilot-studio/guidance/pass-files-to-connectors
```

> Importante: esto no convierte automáticamente el archivo del usuario en “knowledge source persistente”. Es una **entrada** para esa conversación (o para un flujo), no una base de conocimiento corporativa.

---

## 3) Diferencias operativas clave (comparativa)

### 3.1 Tabla comparativa “de proyecto”

| Dimensión | Knowledge source persistente (conector/URL/AIS) | Documento incorporado (archivo subido como knowledge) | Archivo del usuario en conversación |
|---|---|---|---|
| Persistencia | Sí | Sí | No (por defecto) |
| ¿Quién lo aporta? | Maker/admin | Maker/admin | Usuario final |
| Indexación | Depende del origen (GraphSearch / Dataverse index / Azure AI Search) | Dataverse (chunking + embeddings) | Análisis ad hoc + paso a acciones |
| Actualización | Depende del origen (puede ser dinámica) | **Estática**: re‑subir para reflejar cambios | “Snapshot” del momento |
| Permisos por usuario | A menudo **sí** (Entra ID / Dataverse perms) | **No**: expone a todos los usuarios del agente | Depende del flujo; el usuario comparte el archivo explícitamente |
| Riesgo principal | Scope y ruido | **Exposición** de contenido sensible | Exfiltración / tratamiento de datos personales |
| Citaciones | Sí (si el origen las soporta) | Sí | Depende (no siempre hay citación, puede ser resumen) |
| Auditoría / transcripción | Variable | Variable | Variable |

---

## 4) Profundización por tipo

## 4.1 Knowledge source persistente: agente vs nodo y “cómo se busca”

### A) A nivel de agente (fallback general)
- Útil para “cobertura base” o FAQ general.
- Riesgo: scope demasiado amplio (recupera de donde no toca).

### B) A nivel de nodo (búsqueda intencionada)
- Útil para un caso de uso concreto (“políticas RRHH España”).
- Mejora calidad: menos fuentes → mejor precision/groundedness.
- La documentación recomienda especificar fuentes concretas en nodos para “best results”.  
Fuente:
```text
https://learn.microsoft.com/en-us/microsoft-copilot-studio/nlu-boost-node
```

**Consejo didáctico**  
Para entrenar criterio en el alumnado:
- “Si el conocimiento es multi‑dominio, evita agent‑level mega‑fuentes y usa nodos por intención”.

---

## 4.2 Documento incorporado al agente: lo que NO suele contarte nadie

### Cómo se comporta en la práctica

1) **Copia estática**
- Si el documento original cambia, el agente no lo sabe, salvo que:
  - lo vuelvas a subir, o
  - uses OneDrive/SharePoint unstructured con sync (ver 2.1).  
Fuente comparativa OneDrive vs upload:
```text
https://learn.microsoft.com/en-us/microsoft-copilot-studio/knowledge-add-unstructured-data
```

2) **No sobreescribe**
- Subir el mismo nombre crea una copia nueva (duplicados).  
Fuente:
```text
https://learn.microsoft.com/en-us/microsoft-copilot-studio/nlu-documents
```

3) **Forma parte de la solución**
- Exportar/importar la solución del agente incluye los archivos.  
Fuente:
```text
https://learn.microsoft.com/en-us/microsoft-copilot-studio/nlu-documents
```

4) **Exposición a todos**
- Advertencia oficial: el contenido es accesible a cualquiera que chatee con el agente, sin respetar permisos del archivo original.  
Fuente:
```text
https://learn.microsoft.com/en-us/microsoft-copilot-studio/nlu-documents
```

### ¿Cuándo sí usar “archivo subido como knowledge”?

- Prototipo rápido con documentación “safe”.
- Demos o formación.
- Conocimiento que es **público interno** (misma visibilidad para todo el mundo).
- “Snapshot” aprobado (normativa congelada para auditoría).

### ¿Cuándo evitarlo (casos típicos)?

- Documentos por departamento con permisos.
- Información sensible (legal, finanzas, nóminas).
- Documentos que cambian cada semana (procedimientos vivos).

---

## 4.3 Archivo del usuario en conversación: habilitarlo y conectarlo a un flujo

### 4.3.1 Habilitar file uploads

En Settings → Generative AI:
- activar **File uploads**
- ajustar content moderation

Fuente:
```text
https://learn.microsoft.com/en-us/microsoft-copilot-studio/image-input-analysis
```

**Limitaciones relevantes (y sorpresa habitual)**
La documentación de “allow file input” indica límites específicos para archivos “analizados en conversación”:
- tipos soportados (CSV/PDF/TXT/imagenes…)
- límites de tamaño y páginas (p. ej. PDFs < 40 páginas, TXT/CSV pequeños)
- y nota: si el agente se publica en canal SharePoint, los usuarios **no pueden subir archivos**.  
Fuente:
```text
https://learn.microsoft.com/en-us/microsoft-copilot-studio/image-input-analysis
```

> Ojo: estos límites son distintos de los límites de “knowledge files” (512 MB) y pueden confundir al equipo.

---

### 4.3.2 Pedir un archivo al usuario y capturarlo

**Patrón 1 — Question node**
- En un topic, añade un Question node.
- En Identify → selecciona **File**.
- Activa “Include file metadata”.  
Fuente:
```text
https://learn.microsoft.com/en-us/microsoft-copilot-studio/guidance/pass-files-to-connectors
```

**Patrón 2 — Leer attachment si ya venía adjunto**
- Usa `First(System.Activity.Attachments)` para detectar archivos ya adjuntos (por ejemplo en Teams).  
Fuente:
```text
https://learn.microsoft.com/en-us/microsoft-copilot-studio/guidance/pass-files-to-connectors
```

---

### 4.3.3 Pasar el archivo a un flujo / conector / tool

La guía oficial da fórmulas Power Fx para pasar contenido:
- `{ contentBytes: Topic.userReceipt.Content, name: Topic.userReceipt.Name }`
- o directamente desde `First(System.Activity.Attachments)`.

Fuente:
```text
https://learn.microsoft.com/en-us/microsoft-copilot-studio/guidance/pass-files-to-connectors
```

**Implicación práctica**
Esto habilita casos como:
- “Sube el PDF y te extraigo fechas clave”
- “Adjunta la factura y creo un ticket con el adjunto”
- “Adjunta el contrato y clasifico cláusulas”

---

## 5) Implicaciones de seguridad y exposición (bloque crítico)

### 5.1 Riesgos por tipo de conocimiento

#### A) Archivo subido como knowledge (maker)
Riesgo principal:
- **exposición involuntaria** (todo usuario que chatea con el agente puede “ver” el contenido recuperado).  
Fuente: advertencia explícita en *nlu-documents*.  

Mitigación:
- No subir contenido sensible.
- Segmentar agentes por audiencia (p. ej., HR‑Agent vs Everyone‑Agent).
- Preferir SharePoint/OneDrive con permisos.

#### B) SharePoint/OneDrive con autenticación del usuario
Riesgo principal:
- configuración de autenticación incorrecta,
- scope demasiado amplio (sitio raíz “/sites” sin delimitar),
- dependencia de límites del producto (páginas clásicas, SPFx, etc.).  
Fuente:
```text
https://learn.microsoft.com/en-us/microsoft-copilot-studio/requirements-quotas
https://learn.microsoft.com/en-us/microsoft-copilot-studio/nlu-generative-answers-sharepoint-onedrive
```

Mitigación:
- delimitar URLs específicas,
- validar con usuarios de roles distintos,
- usar nodos por intención para limitar fuentes.

#### C) Archivo del usuario (runtime)
Riesgo principal:
- tratamiento de PII/contratos/secretos industriales,
- pasar el archivo a sistemas downstream sin control (Power Automate, conectores),
- retención/logging en herramientas externas.

Mitigación:
- establecer reglas de uso (banner/aviso),
- DLP + policies de entorno,
- moderación,
- “data minimization”: extraer solo lo necesario.

---

### 5.2 Auditoría: citaciones y transcripciones

- SharePoint: respuestas usando SharePoint como knowledge source no aparecen en conversation transcripts.  
Fuente:
```text
https://learn.microsoft.com/en-us/microsoft-copilot-studio/nlu-generative-answers-sharepoint-onedrive
```

**Pregunta para el equipo de gobierno**
- Si no hay transcripción, ¿cómo auditamos decisiones?  
Opciones típicas:
- logging alternativo, feedback, o “respuestas con citación + click tracking”.

---

## 6) Árbol de decisión (para elegir rápido)

```mermaid
flowchart TD
  A[¿El contenido es sensible / con permisos por usuario?] -->|Sí| B[Usa SharePoint/OneDrive con Entra ID o Dataverse perms]
  A -->|No| C[¿Necesitas que el contenido cambie a menudo?]
  C -->|Sí| D[Usa SharePoint/OneDrive (sync) o Azure AI Search con pipeline]
  C -->|No| E[¿Quieres snapshot incluido en la solución?]
  E -->|Sí| F[Sube archivos como knowledge (Dataverse)]
  E -->|No| G[Usa web corporativa o mezcla de fuentes]
  B --> H[Limita scope con nodos por intención]
  D --> H
  F --> I[No subir contenido sensible]
  G --> H
```

---

## 7) Mini‑ejercicio (10 min): elige el “tipo” correcto

Para cada caso, responde:
- ¿qué tipo de conocimiento usarías?
- ¿por qué?
- ¿qué riesgo te preocupa más?
- ¿cómo lo mitigas?

### Caso 1 — Manual interno (todos los empleados)
- 1 PDF estable, sin cambios frecuentes.

### Caso 2 — Políticas RRHH (solo managers)
- 20 documentos, con versiones y permisos.

### Caso 3 — Analizador de CVs (el candidato sube su CV)
- archivo del usuario, hay PII, y se necesita extraer datos.

---

## 8) Lecturas oficiales recomendadas

```text
Knowledge sources summary:
https://learn.microsoft.com/en-us/microsoft-copilot-studio/knowledge-copilot-studio

Upload files as knowledge source:
https://learn.microsoft.com/en-us/microsoft-copilot-studio/knowledge-add-file-upload
https://learn.microsoft.com/en-us/microsoft-copilot-studio/nlu-documents

Allow file input from users:
https://learn.microsoft.com/en-us/microsoft-copilot-studio/image-input-analysis

Pass files to flows/connectors/tools:
https://learn.microsoft.com/en-us/microsoft-copilot-studio/guidance/pass-files-to-connectors

SharePoint generative answers behavior:
https://learn.microsoft.com/en-us/microsoft-copilot-studio/nlu-generative-answers-sharepoint-onedrive

Limits (SharePoint constraints, quotas):
https://learn.microsoft.com/en-us/microsoft-copilot-studio/requirements-quotas
```
