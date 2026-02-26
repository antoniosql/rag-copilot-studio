# Sesión 1.4 — Aplicación práctica en Copilot Studio

> **Bloque**: Sesión 1 — *Fundamentos operativos de RAG y su aplicación inicial en Copilot Studio* 
> **Objetivo**: activar *Generative Answers*, incorporar un primer *knowledge source*, entender citaciones y hacer una evaluación rápida del comportamiento.


---

## 2) Activación de Generative Answers

### 2.1 ¿Qué es “Generative Answers” en Copilot Studio?

A alto nivel, **Generative Answers** permite que el agente:
- busque información en **knowledge sources** configuradas,
- genere una respuesta **fundamentada** (grounded) en ese contenido,
- incluya **citaciones** para trazabilidad.

En Copilot Studio, estas respuestas generativas pueden usarse como:
- **fuente principal** de información, o
- **fallback** cuando no hay un topic que resuelva la intención del usuario.

---

### 2.2 Pasos (guía de clase)

> Nota: los nombres exactos de menús pueden variar ligeramente por versión/idioma, pero la estructura general es consistente.

1) Abre Copilot Studio y entra en tu agente (o crea uno nuevo).  
2) Ve a la configuración/overview del agente y **habilita Generative Answers** (si está disponible como opción por agente).  
3) Guarda los cambios.

**Consejo operativo**
- Si la característica no aparece, suele deberse a:
  - política del tenant,
  - licencia/capacidad,
  - limitaciones del entorno.

---

## 3) Incorporación de un primer knowledge source

En Copilot Studio, puedes incorporar conocimiento en dos niveles:

1) **A nivel de agente** (Knowledge / Knowledge sources del agente)  
2) **A nivel de topic** mediante un **Generative answers node** (más controlado y específico)

> Regla práctica: *si quieres control por caso de uso, usa el nodo en el topic.*  
> Si quieres “cobertura general” como fallback, usa el nivel de agente.

---

### 3.1 Opción A (rápida): subir un archivo como fuente de conocimiento

#### Pasos
1) En el agente, abre la sección **Knowledge** (o equivalente).
2) Selecciona **Add knowledge source** → **Files / Documents**.
3) Sube tu PDF/DOCX.
4) Espera a que el estado pase a “Ready” (listo para uso).

#### Consideraciones importantes (para explicar en clase)

- Los archivos subidos se almacenan de forma segura en **Dataverse** y están sujetos al almacenamiento disponible del entorno.
- Los archivos pueden tardar un tiempo en estar listos (“In progress” → “Ready”).
- Subir un archivo con el mismo nombre **no lo sobreescribe**: crea una copia adicional.
- **Advertencia de seguridad**: el contenido del archivo subido puede quedar disponible para cualquier usuario que chatee con el agente (no hereda permisos del archivo original).  
  → Esto es crítico para entorno corporativo: *solo subir material apto para acceso general del agente.*

---

### 3.2 Opción B (más “enterprise”): SharePoint / OneDrive

#### Cuándo usarlo
- Cuando necesitas que el agente respete **permisos por usuario** de SharePoint/OneDrive.
- Cuando el conocimiento ya vive en SharePoint y no quieres duplicarlo.

#### Pasos (alto nivel)
1) Añade SharePoint como knowledge source (URL/s) según el asistente de Copilot Studio.
2) Configura **autenticación de usuario** (Microsoft Entra ID) si procede.
3) Verifica que el usuario del chat solo ve contenido al que tiene acceso.

#### Nota práctica
- Algunas integraciones requieren que **Dataverse search** esté habilitado en el entorno para poder usar determinadas fuentes conectadas.

---

### 3.3 Añadir un “Generative answers node” en un topic (control fino)

**Objetivo didáctico**: mostrar que un nodo en topic permite:
- escoger fuentes específicas para ese momento,
- capturar la respuesta en una variable,
- controlar si se envía el mensaje directamente.

#### Pasos recomendados (para demo guiada)
1) Ve a **Topics** y abre/crea un topic de prueba: “Preguntas sobre políticas”.
2) En el canvas, añade un nodo **Generative answers** (Advanced → Generative answers).
3) En propiedades del nodo:
   - Input: usa el texto del usuario (p. ej. `Activity.Text`).
   - Selecciona el/los knowledge sources adecuados para este nodo.
4) Opcional (útil para explicar citaciones):
   - Desmarca “Send a message” para capturar respuesta y presentarla luego.
   - Guarda en una variable global o de topic.

> Nota para instructor: si personalizas el envío del mensaje, asegúrate de explicar que el canal de publicación puede tratar citaciones de forma distinta (por ejemplo, Teams puede requerir renderizado explícito cuando personalizas).

---

## 4) Análisis de citaciones

### 4.1 Qué es una citación (para el usuario final)

Una citación debería responder a:
- **¿de qué documento/página sale esta afirmación?**
- **¿puedo abrirlo y verificarlo?**

En un sistema RAG, citación ≈ evidencia del retrieval.

---

### 4.2 Qué mirar en una citación (checklist rápido)

- ¿Apunta a un documento correcto (título/URL)?
- ¿Apunta a una sección/página razonable?
- ¿Hay 1–3 citaciones relevantes o 10 irrelevantes?
- ¿La respuesta contiene afirmaciones sin citación (posible alucinación)?

---

### 4.3 Síntomas comunes y qué significan

**Síntoma A**: citación irrelevante  
- Recuperación mala (chunking/embeddings/keyword).
- Scope demasiado amplio.
- Falta de filtros por metadatos.

**Síntoma B**: citaciones duplicadas  
- Overlap alto o documentos duplicados.
- Retrievers devolviendo chunks casi idénticos.

**Síntoma C**: no hay citaciones  
- Se respondió con “conocimiento general” (si está permitido).
- No se recuperó nada (vacío) y el sistema generó igual (mala práctica).
- Fuente sin `ContentLocation` o no mapeada.

---

### 4.4 Limitación importante para arquitectura híbrida

En algunos escenarios, las citaciones devueltas por una fuente de conocimiento no se pueden reutilizar como entradas de otras herramientas/acciones (por ejemplo, no “pasar la citación” a un flujo como input).  
Esto condiciona diseños donde querías automatizar un “open citation → ejecutar acción”.

---

## 5) Evaluación preliminar de comportamiento (mini test set)

### 5.1 Objetivo

Sin entrar aún en evaluación formal (sesión 4), haremos un chequeo rápido:

- ¿Recupera lo correcto?
- ¿Cita correctamente?
- ¿Abstiene cuando no hay evidencia?

---

### 5.2 Construye un test set mínimo (10 min)

Crea 8–10 preguntas:

1. 4 preguntas fáciles (directamente respondidas en el documento).
2. 2 preguntas con sinónimos (“vacaciones” vs “permiso anual”).
3. 2 preguntas con términos exactos/códigos (si el documento tiene).
4. 1 pregunta fuera de scope (no debería responder con seguridad).
5. 1 pregunta ambigua (debería pedir aclaración).

Ejemplo (RRHH vacaciones):
- “¿Cuántos días de vacaciones corresponden en España?”
- “¿Qué ocurre si me incorporo a mitad de año?”
- “¿Se pueden trasladar días al año siguiente?”
- “¿Cuál es el procedimiento para solicitar vacaciones?”
- “¿Qué dice exactamente el apartado ‘Devengo’?”
- “¿Cuál es el código del procedimiento?”
- “¿Qué política aplica a Portugal?” (si solo hay España, debería abstener/preguntar)
- “¿Cuántos días me tocan?” (ambigua: depende de contrato/antigüedad)

---

### 5.3 Registro rápido (plantilla)

| Pregunta | ¿Respuesta correcta? | ¿Citaciones relevantes? | ¿Pide aclaración/abstiene? | Comentarios |
|---|---:|---:|---:|---|
| P1 |  |  |  |  |
| P2 |  |  |  |  |
| … |  |  |  |  |

---

### 5.4 Qué ajustar si falla (sin entrar en ingeniería profunda aún)

- Si falla “exactitud literal”: considerar activar **búsqueda híbrida** (o asegurar que la fuente soporte keyword + vector).
- Si falla por “mezcla de temas”: revisar **chunking** (tamaño, estructura, overlap).
- Si falla por “documentos equivocados”: ajustar **alcance** y filtros (metadatos).
- Si responde fuera de scope: reforzar instrucciones del agente (abstención responsable) y recortar fuentes.

---

## 6) Cierre del bloque (2–3 min)

### 6.1 Conclusión práctica

Lo que acabas de ver es el “ciclo” mínimo de RAG en Copilot Studio:
- habilitar capacidad,
- conectar conocimiento,
- probar con preguntas reales,
- observar citaciones,
- iterar sobre chunking/alcance/metadata.

### 6.2 Puente a la siguiente sesión

En la siguiente sesión profundizaremos en:
- diferencias entre orígenes de conocimiento,
- calidad de recuperación,
- implicaciones de seguridad según el tipo de fuente.

---

## 7) Referencias oficiales (lectura opcional)

```text
Nodo de Generative Answers (cómo añadirlo y configurar fuentes):
https://learn.microsoft.com/en-us/microsoft-copilot-studio/nlu-boost-node

Uso de archivos subidos como conocimiento (consideraciones y límites):
https://learn.microsoft.com/en-us/microsoft-copilot-studio/nlu-documents

Uso de SharePoint/OneDrive como conocimiento (notas de configuración):
https://learn.microsoft.com/en-us/microsoft-copilot-studio/nlu-generative-answers-sharepoint-onedrive

Guía RAG en Copilot Studio (proceso general):
https://learn.microsoft.com/en-us/microsoft-copilot-studio/guidance/retrieval-augmented-generation
```

