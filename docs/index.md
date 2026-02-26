# Seminario: RAG y Copilot Studio

Bienvenido/a 👋  
Aquí encontrarás los **apuntes y prácticas** del seminario, organizados por sesiones para que puedas **navegar por módulos**, buscar conceptos y reutilizar ejemplos como “recetario” de implementación.

[Empezar por la Sesión 1.1](sesion1_1_arquitectura_rag_entorno_empresarial.md)
[Ver toda la estructura](#estructura-del-seminario)

---

## ¿Qué vas a aprender?

- Diseñar una **arquitectura RAG** en entorno empresarial (ingesta → indexación → recuperación → respuesta).
- Entender *por qué* funciona (y *por qué falla*) la **recuperación semántica**: chunking, embeddings, búsqueda híbrida, metadatos.
- Implementar y operar un agente en **Copilot Studio** con conocimiento, citaciones y evaluación básica.
- Identificar cuándo **RAG no es suficiente** y conviene un enfoque híbrido: **herramientas (tools)** y **datos estructurados**.
- Medir y mejorar: **evaluación**, **seguridad/gobierno**, **citaciones y trazabilidad**.

---

!!! tip "Cómo usar estos apuntes"
    - Usa el **menú lateral** para moverte por sesiones y bloques.
    - Usa el **buscador** (arriba) para localizar rápidamente términos: *chunking, embeddings, citaciones, prompt injection…*
    - Si estás siguiendo el seminario en orden, avanza de **1.1 → 4.4**.  
      Si quieres ir “a lo práctico”, salta directamente a **1.4**, **2.4** y **3.4**.

---

## Estructura del seminario

<div class="grid cards" markdown>

-   ## Sesión 1 — Fundamentos operativos de RAG  
    Arquitectura, indexación, embeddings y primera práctica en Copilot Studio.

    [Abrir Sesión 1 (recomendado)](sesion1_1_arquitectura_rag_entorno_empresarial.md)
    [Ir a la práctica 1.4](sesion1_4_practica_copilot_studio.md)

    **Bloques**
    - [1.1 Arquitectura RAG en entorno empresarial](sesion1_1_arquitectura_rag_entorno_empresarial.md)
    - [1.2 Proceso de indexación documental](sesion1_2_proceso_indexacion_documental.md)
    - [1.3 Embeddings y recuperación semántica](sesion1_3_embeddings_y_recuperacion_semantica.md)
    - [1.4 Aplicación práctica en Copilot Studio](sesion1_4_practica_copilot_studio.md)

-   ## Sesión 2 — Orígenes de conocimiento y calidad de recuperación  
    Qué fuentes soporta Copilot Studio, cómo se “tratan” internamente y cómo mejorar la calidad de recuperación.

    [Abrir Sesión 2](sesion2_1_origenes_soportados_y_tratamiento.md)
    [Ir a la práctica 2.4](sesion2_4_practica_aplicada.md)

    **Bloques**
    - [2.1 Orígenes soportados y su tratamiento](sesion2_1_origenes_soportados_y_tratamiento.md)
    - [2.2 Diferencias operativas clave](sesion2_2_diferencias_operativas_clave.md)
    - [2.3 Factores de calidad de la recuperación](sesion2_3_factores_calidad_recuperacion.md)
    - [2.4 Práctica aplicada](sesion2_4_practica_aplicada.md)

-   ## Sesión 3 — Arquitectura híbrida: RAG + herramientas + datos  
    Cómo decidir cuándo usar RAG, cuándo usar herramientas (acciones) y cómo encaja el dato estructurado.

    [Abrir Sesión 3](sesion3_1_cuando_rag_no_es_suficiente.md)
    [Ir al ejercicio 3.4](sesion3_4_ejercicio_practico_hibrido.md)

    **Bloques**
    - [3.1 Cuándo RAG no es suficiente](sesion3_1_cuando_rag_no_es_suficiente.md)
    - [3.2 Uso de herramientas en Copilot Studio](sesion3_2_uso_herramientas_copilot_studio.md)
    - [3.3 Datos estructurados y “agentes de datos”](sesion3_3_datos_estructurados_y_agente_datos.md)
    - [3.4 Ejercicio práctico híbrido](sesion3_4_ejercicio_practico_hibrido.md)

-   ## Sesión 4 — Evaluación, seguridad y gobierno  
    Medición repetible, evaluación en Copilot Studio, controles de seguridad y trazabilidad.

    [Abrir Sesión 4](sesion4_1_evaluacion_calidad.md)
    [Ir a citaciones y trazabilidad (4.4)](sesion4_4_citaciones_y_trazabilidad.md)

    **Bloques**
    - [4.1 Evaluación de calidad en sistemas RAG](sesion4_1_evaluacion_calidad.md)
    - [4.2 Evaluación en Copilot Studio](sesion4_2_evaluacion_en_copilot_studio.md)
    - [4.3 Seguridad y gobierno](sesion4_3_seguridad_y_gobierno.md)
    - [4.4 Citaciones y trazabilidad](sesion4_4_citaciones_y_trazabilidad.md)

</div>

---

## Ruta rápida recomendada (si tienes poco tiempo)

1. **Empieza por el “mapa”**: [1.1 Arquitectura RAG](sesion1_1_arquitectura_rag_entorno_empresarial.md)  
2. **Asegura la base de calidad**: [1.2 Indexación](sesion1_2_proceso_indexacion_documental.md) + [1.3 Embeddings](sesion1_3_embeddings_y_recuperacion_semantica.md)  
3. **Pasa a manos a la obra**: [1.4 Práctica Copilot Studio](sesion1_4_practica_copilot_studio.md)  
4. **Aprende “por qué falla” y cómo operarlo**: [2.3 Factores de calidad](sesion2_3_factores_calidad_recuperacion.md) + [2.4 Práctica](sesion2_4_practica_aplicada.md)  
5. **Hazlo robusto**: [3.1 RAG no basta](sesion3_1_cuando_rag_no_es_suficiente.md) + [3.2 Tools](sesion3_2_uso_herramientas_copilot_studio.md)  
6. **Cierra con evaluación y gobierno**: [4.1 Evaluación](sesion4_1_evaluacion_calidad.md) + [4.3 Seguridad](sesion4_3_seguridad_y_gobierno.md) + [4.4 Trazabilidad](sesion4_4_citaciones_y_trazabilidad.md)

---

!!! note "Sugerencia para añadir una imagen de portada (opcional)"
    Si quieres un “banner” en la home, guarda una imagen en `docs/assets/images/banner.png` y añade esta línea debajo del título:

    `![Banner del seminario](assets/images/banner.png)`
