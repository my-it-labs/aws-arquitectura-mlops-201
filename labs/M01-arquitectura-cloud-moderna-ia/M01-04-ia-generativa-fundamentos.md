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

## Corpus

El **corpus** es el **conjunto de documentos** (o textos) que alimentan un sistema GenAI — sobre todo en **RAG**. No es «toda la base de datos de la organización», sino la **colección acotada y gobernada** que el sistema puede consultar.

| Ejemplo sector transporte | Tipo de contenido |
|---------------------------|-------------------|
| Procedimientos de sala de control | PDFs, Word en repositorio corporativo |
| Manuales de mantenimiento | Documentación técnica de activos |
| Políticas internas | RGPD, uso de IA, comunicación |
| FAQs operativas | Textos curados por expertos |

Propiedades que importan en decisiones de arquitectura:

| Propiedad | Por qué importa |
|-----------|-----------------|
| **Calidad** | Documentos obsoletos o contradictorios → respuestas incorrectas |
| **Gobierno** | Quién añade/borra, versiones, clasificación (interno / confidencial) |
| **Vigencia** | Un procedimiento antiguo indexado puede ser peligroso |
| **Alcance** | Streams, SAP en tiempo real o tablas operativas **no son corpus** por sí solos — el corpus suele ser **texto documental** indexado para búsqueda semántica |

En cloud, el corpus suele residir en **almacén objeto** (p. ej. S3) antes de indexarse. Bedrock Knowledge Bases y pipelines DIY parten de ahí.

## Chunking

El **chunking** consiste en **partir cada documento en trozos** («chunks») antes de generar **embeddings** y cargarlos en el índice vectorial.

### Por qué hace falta

Los modelos y los índices tienen **límite de contexto**. La recuperación funciona mejor con **fragmentos focalizados** que con un manual entero de decenas de páginas.

```
Manual de contingencia (50 páginas)
        │
        ▼ chunking
   [chunk 1]  [chunk 2]  [chunk 3]  …  [chunk N]
        │
        ▼ embeddings
   vectores en índice → búsqueda por similitud
```

Ante una pregunta del usuario, el sistema **recupera los chunks más relevantes** y los incluye en el prompt del modelo fundacional.

### Parámetros de diseño

| Parámetro | Efecto |
|-----------|--------|
| **Tamaño del chunk** (p. ej. 500–1000 tokens) | Muy pequeño → pierde contexto; muy grande → ruido y respuestas vagas |
| **Solapamiento (overlap)** (p. ej. 10–20 %) | Evita cortar procedimientos o listas de pasos entre dos chunks |
| **Criterio de corte** | Párrafo, sección o página — en manuales técnicos suele ir mejor **por sección** que por caracteres fijos |
| **Metadata** (documento, versión, línea, fecha) | Permite filtrar: «solo procedimientos vigentes 2025» |

### Errores habituales

| Error | Consecuencia |
|-------|--------------|
| Corpus sin limpiar | Borradores y duplicados indexados |
| Chunks demasiado grandes | Contexto irrelevante en el prompt |
| Chunks demasiado pequeños | Falta el «si… entonces…» completo del procedimiento |
| Sin metadata | Mezcla versiones o ámbitos distintos (líneas, departamentos) |
| Confundir corpus con datos vivos | RAG no sustituye APIs de tiempo real ni maestros transaccionales |

### Relación corpus → chunking → RAG

| Concepto | Rol |
|----------|-----|
| **Corpus** | *Qué* documentos puede consultar el sistema |
| **Chunking** | *Cómo* se trocean para encontrar la parte correcta |
| **Embeddings + índice** | Representación buscable de cada chunk |
| **RAG** | Recuperar chunks + generar respuesta con el fundacional |


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
- **Corpus** gobernado y **chunking** bien diseñado determinan la calidad del RAG tanto como el modelo.
- Gobierno de prompts, guardrails y logging son parte de la arquitectura, no un extra.
- GenAI amplía capacidades pero introduce riesgos que ML clásico no tenía en la misima forma.
