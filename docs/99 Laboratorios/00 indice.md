# Caso FraSoHome — Laboratorios (Copilot Studio + RAG + Dataverse)

## Introducción del caso

**FraSoHome** es una cadena ficticia B2C de muebles y decoración con operación omnicanal (tiendas físicas, e-commerce y fidelización en CRM). En los últimos meses, la compañía sufre:

- **Fragmentación de la información** (CRM, POS, e-commerce, ERP) y dificultad para “unir” datos.
- **Problemas de calidad y consistencia** (duplicados, claves que no encajan, formatos heterogéneos, nulos y anomalías).
- **Dependencia del equipo de datos** para responder preguntas recurrentes y falta de consistencia en definiciones y procedimientos.

El objetivo del taller es construir un **asistente en Copilot Studio** con enfoque **híbrido**:

- **RAG (documentos)** para preguntas “documentales” (políticas, procedimientos, definiciones de KPI).
- **Tools / acciones (Dataverse)** para preguntas “métricas” (ventas, devoluciones, stock) que requieren cálculos o consulta a datos estructurados.
- **Respuesta con evidencia**: citaciones para lo documental y trazabilidad de la consulta para lo métrico.

> Nota: Estos laboratorios empiezan en **LAB 2** .

## Qué construirás al final

Un agente capaz de:

1. Responder preguntas internas con **citaciones** desde conocimiento controlado.
2. Consultar datos en Dataverse mediante **Tools** y devolver métricas con supuestos claros.
3. Integrar “métrica + definición/política” en respuestas mixtas.
4. Pasar una evaluación básica con test sets y aplicar controles mínimos de seguridad.

## Índice de laboratorios

- [LAB 2 — Base de conocimiento (RAG) en Copilot Studio](LAB2_Base_de_conocimiento_RAG_en_Copilot_Studio.md)  
  Carga de documentos vigentes, activación de citaciones y prueba controlada de conflicto de versiones.

- [LAB 3 — Prompting, citaciones y seguridad (control de versionado + prompt injection)](LAB3_Prompting_Citaciones_y_Control_de_Fallback.md)  
  Instrucciones del agente, prompt modification, formato, abstención y pruebas de seguridad.

- [LAB 4 — Dataverse: crear tablas e importar datos estructurados (CSV)](LAB4_Dataverse_y_Datos_Estructurados.md)  
  Creación de tablas y carga de CSV para ventas, devoluciones y stock.

- [LAB 5 — Arquitectura híbrida: Tools (Dataverse) + RAG (documentos)](LAB5_Arquitectura_Hibrida_Tools_y_RAG.md)  
  Añadir Tools de Dataverse (solo lectura), crear topic métrico y responder preguntas con datos + definiciones.

- [LAB 6 — Evaluación, seguridad y gobierno del asistente](LAB6_Evaluacion_Seguridad_y_Gobierno.md)  
  Ejecutar evaluaciones con test sets, diagnosticar fallos y aplicar acciones correctivas.

## Estructura recomendada (referencia)

Este repo de markdown asume que tienes disponible el pack del caso con estas carpetas:

- `Documentos_Knowledge_Clasificados/` (políticas, KPI, operaciones, catálogo, CRM, gobierno, tests)
- `Datos_Estructurados/` (CSV)
- `TestSets/` (CSV)

Los tienes disponibles aquí : [Recursos](materiales)


# FraSoHome — Índice y clasificación de documentos de conocimiento (Knowledge)

Este documento clasifica los archivos de la carpeta `Documentos_Knowledge`
y sugiere cómo usarlos en los laboratorios y en un escenario de producción.

---

## 📅 Última actualización

2026-02-26

---

## 🎯 Uso

Referencia rápida para:

- Seleccionar fuentes de conocimiento (RAG)
- Evitar mezclar versiones
- No subir documentos de prueba en producción

---

# 1️⃣ Clasificación por dominio

---

## 01_Politicas

- **FS-KB-01_Politica_Devoluciones_v1.3_Vigente.docx**  
  Estado: **VIGENTE**  
  Política oficial de devoluciones (plazos, excepciones, reembolsos).  
  ✔ Usar en respuestas a empleados / atención.

- **FS-KB-02_Politica_Devoluciones_v1.2_Obsoleta.docx**  
  Estado: **OBSOLETA** (solo formación).  
  Versión anterior para practicar control de versiones.  
  ❌ No usar en producción.

---

## 02_KPI_y_Datos

- **FS-KB-03_Diccionario_KPI_Reglas_Calculo_v1.0.docx**  
  Estado: **VIGENTE**  
  Definiciones y fórmulas KPI:
  - Ventas netas
  - Stock disponible
  - Tasa de devolución
  - Etc.

---

## 03_Operaciones

- **FS-KB-04_Manual_Tienda_Caja_y_Pagos_Mixtos_v2.1.docx**  
  Estado: **VIGENTE**  
  Procedimientos de tienda:
  - Cierre de caja
  - Pagos mixtos
  - Incidencias

- **FS-KB-05_Guia_Conciliacion_Pagos_Ecommerce_v1.4.docx**  
  Estado: **VIGENTE**  
  Conciliación e-commerce:
  - Cobros
  - Reembolsos
  - Cupones

- **FS-KB-09_FAQ_Operaciones_y_Atencion_v1.0.docx**  
  Estado: **VIGENTE**  
  FAQ operacional:
  - Respuestas rápidas
  - Pautas

---

## 04_Catalogo

- **FS-KB-06_Taxonomia_Catalogo_y_Reglas_SKU_v1.2.docx**  
  Estado: **VIGENTE**  
  Reglas de SKU, categorías y naming para catálogo.

---

## 05_CRM

- **FS-KB-07_Guia_Fidelizacion_CRM_v1.1.docx**  
  Estado: **VIGENTE**  
  Programa de fidelización:
  - Tiers
  - Beneficios
  - Condiciones

---

## 06_Gobierno_y_Seguridad

- **FS-KB-08_Politica_Datos_y_Permisos_Asistente_v1.0.docx**  
  Estado: **VIGENTE**  
  Política de datos:
  - Qué puede / no puede responder
  - PII
  - Permisos

---

## 07_Test_Security

- **FS-KB-10_Documento_Prueba_Prompt_Injection_NO_PROD.docx**  
  Estado: **NO_PROD** (solo pruebas).  
  Texto simulado de prompt injection para red teaming.  
  ⚠ Debe retirarse tras el LAB 3.

---

# 2️⃣ Recomendación de carga por laboratorio

---

## LAB 2 (Base RAG)

Subir SOLO documentos **VIGENTES**:

- FS-KB-01
- FS-KB-03
- FS-KB-04
- FS-KB-05
- FS-KB-06
- FS-KB-07
- FS-KB-08
- FS-KB-09

Al final, subir **FS-KB-02** para provocar conflicto controlado.

---

## LAB 3 (Prompting y seguridad)

- Subir temporalmente **FS-KB-10** para pruebas de inyección.
- Luego retirarlo.
- Mantener FS-KB-02 para probar regla de versionado.

---

## LAB 4 (Dataverse)

Sin cambios en documentos.  
Se preparan tablas y CSV.

---

## LAB 5 (Híbrido)

- Asegurar FS-KB-03 para definiciones KPI.
- Mantener FS-KB-08 por reglas de permisos al usar datos.

---

## LAB 6 (Evaluación y gobierno)

Preparar "modo producción":

- Retirar FS-KB-02
- Retirar FS-KB-10
- Dejar solo documentos **VIGENTES**

---

# 3️⃣ Regla práctica de versionado

Mantener patrón en nombres de archivo: FS-KB-XX_<Tema>vX.Y<Estado>.docx 

Estados recomendados:

- Vigente
- Obsoleta
- NO_PROD

---

## 📌 Regla operativa

- Cuando se publique una nueva versión:
  - Subir la nueva
  - Retirar la anterior (o moverla a repositorio histórico fuera de Knowledge)

- Si por formación conviven versiones:
  - El agente debe incluir una regla de priorización:
    - Documento **Vigente**
    - Versión mayor
