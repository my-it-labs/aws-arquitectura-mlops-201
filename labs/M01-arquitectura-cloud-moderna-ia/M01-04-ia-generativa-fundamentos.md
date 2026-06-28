# M01-04 — IA generativa — fundamentos

[← Página anterior](M01-03-tipos-carga-ia.md) · [Siguiente página →](M01-05-mlops-pipeline-conceptual.md)

## Modelos fundacionales

Un **modelo fundacional** (foundation model) es un modelo de gran tamaño preentrenado con vastos corpus (texto, código, multimodal) que puede **adaptarse** a muchas tareas mediante prompting, RAG o fine-tuning.

Características relevantes para decisiones IT:

| Aspecto | Implicación |
|---------|-------------|
| Tamaño (parámetros) | Más capacidad pero más coste de inferencia |
| Ventana de contexto | Cuánto texto «ve» en una petición |
| Modalidades | Solo texto vs imagen/audio |
| Licencia / uso comercial | Contrato con proveedor (Bedrock, Azure OpenAI, etc.) |
| Región | Dónde se procesan datos y prompts (soberanía) |

En enterprise **casi nunca** autoalojarás un fundacional completo al inicio: consumirás **API gestionada** del proveedor cloud.

## Tokens y contexto

Los modelos generativos procesan texto en **tokens** (fragmentos de palabras). Facturación, límites y latencia suelen expresarse en tokens.

- **Prompt:** lo que envías (instrucción + contexto RAG + historial).
- **Completion:** lo que genera el modelo.
- **Ventana de contexto:** máximo combinado; si el corpus RAG es enorme, hay que **recortar** o resumir.

> [!TIP]
> Para presupuestos: estima usuarios × consultas/día × tokens medios. Un chat interno sin límites puede generar **sorpresas en factura** en semanas.

## Embeddings

Un **embedding** es una representación numérica (vector) de un texto que captura similitud semántica. Son la base de:

- Búsqueda semántica («documentos parecidos a esta pregunta»).
- RAG (recuperar chunks relevantes antes de generar).
- Clustering y clasificación de documentos.

Flujo RAG simplificado:

```
Pregunta → embedding pregunta → búsqueda en índice vectorial → top-K chunks
    → prompt enriquecido → modelo generativo → respuesta
```

La calidad del RAG depende tanto del **modelo generativo** como del **modelo de embeddings** y del **chunking** de documentos.

## Bases de datos vectoriales e índices

Los vectores se almacenan en:

- Servicios de búsqueda con soporte vectorial (OpenSearch, AI Search…).
- Bases vectoriales especializadas (integradas en plataformas cloud).
- Knowledge Bases managed (p. ej. Bedrock KB) que abstraen parte del pipeline.

| Criterio | Pregunta |
|----------|----------|
| Escala | ¿Millones de chunks o miles? |
| Híbrido | ¿Búsqueda keyword + vector? |
| Latencia | ¿Milisegundos para recuperación? |
| Gobierno | ¿Aislamiento por tenant o proyecto? |

## Prompt engineering (nivel decisión)

No se trata de «escribir bonito», sino de **políticas y plantillas**:

- Instrucciones de tono y límites («no inventes cifras operativas»).
- Formato de salida (JSON, lista, resumen ejecutivo).
- Citas obligatorias a fuentes (en RAG).
- System prompts auditables y versionados.

En organizaciones reguladas, los prompts efectivos de producción suelen ser **plantillas aprobadas**, no texto libre del usuario final.

## Guardrails y post-procesado

Capas que limitan riesgo:

- **Filtros de contenido** (PII, toxicidad, temas prohibidos).
- **Grounding:** exigir que la respuesta cite fuentes del corpus.
- **Human review** en flujos sensibles.
- **Logging** de prompts y respuestas (con política de retención RGPD).

## Riesgos y limitaciones específicas de GenAI

| Riesgo | Descripción | Mitigación arquitectónica |
|--------|-------------|---------------------------|
| Alucinación | Respuesta plausible pero falsa | RAG, grounding, disclaimers, revisión |
| Fuga de datos | Prompt incluye secretos o PII | DLP, clasificación, anonimización |
| Prompt injection | Usuario manipula instrucciones | Separación de roles, sanitización, allowlists |
| Dependencia de proveedor | Modelo o API cambia | Abstracción, evaluación continua, multi-modelo |
| Sesgo | Reflejo del corpus | Curación documental, tests de equidad |
| Coste | Uso descontrolado | Cuotas, presupuestos, autenticación |

> [!WARNING]
> **IA generativa ≠ verdad.** En contextos operativos (horarios, seguridad, regulación), la salida debe tratarse como **borrador** hasta validación por fuentes autorizadas o personas responsables.

## IA tradicional vs generativa — cuándo cada una

| Situación | Enfoque habitual |
|-----------|------------------|
| Predicción numérica con histórico limpio | ML clásico (regresión, series temporales) |
| Clasificación con etiquetas | ML supervisado |
| Exploración de documentación natural | RAG + fundacional |
| Interacción conversacional acotada | GenAI con guardrails |
| Automatización multi-sistema | Agente (madurez alta) |

Muchos proyectos maduros combinan **ML clásico para la señal** y **GenAI para explicación o interfaz**.

## Resumen

- Modelos fundacionales se consumen por API; tokens y contexto definen coste y límites.
- Embeddings e índices vectoriales son el corazón de RAG enterprise.
- Gobierno de prompts, guardrails y logging son parte de la arquitectura, no un extra.
- GenAI amplía capacidades pero introduce riesgos que ML clásico no tenía en la misima forma.
