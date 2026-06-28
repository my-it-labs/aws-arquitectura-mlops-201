# M02-04 — Bedrock y GenAI en AWS

[← Página anterior](M02-03-ecosistema-sagemaker.md) · [Siguiente página →](M02-05-observabilidad-coste-seguridad.md)

## Amazon Bedrock

**Amazon Bedrock** es el servicio AWS para consumir **modelos fundacionales** de distintos proveedores (Anthropic, Meta, Mistral, Amazon Titan, etc.) vía **API unificada**, con integración en IAM, VPC y facturación AWS.

Posicionamiento arquitectónico: **capa de inferencia GenAI managed** — no sustituye el lago de datos ni el gobierno documental.

## Modelos disponibles (concepto)

Los modelos se eligen por:

| Criterio | Pregunta |
|----------|----------|
| Capacidad | ¿Razonamiento, código, multilingüe? |
| Contexto | ¿Cuántos tokens caben? |
| Coste | Precio input/output por 1K tokens |
| Región | ¿Disponible donde deben residir datos? |
| Política | ¿Aprobado por compliance interno? |

Cambiar de modelo debe ser **decisión versionada** — no un toggle sin re-evaluación.

## Invocación básica

Flujo típico:

1. Cliente (app, Lambda, API Gateway) autenticado con IAM.
2. Llamada `InvokeModel` o API Converse con prompt.
3. Respuesta + metadatos de uso (tokens).

**Sin RAG:** el modelo responde con conocimiento de entrenamiento + prompt — riesgo de alucinación en datos internos.

## Guardrails for Amazon Bedrock

Capa de **políticas** sobre prompts y respuestas:

- Filtros de contenido (violencia, PII según configuración).
- Deny topics — temas prohibidos.
- Word filters — términos bloqueados.
- Grounding checks — cuando se integra con KB.

**Arquitectura:** Guardrails se asocian a aplicaciones o flujos — deben ser **mismo entorno** que auditoría (logs).

## Knowledge Bases (RAG managed)

Bedrock **Knowledge Bases** abstraen parte del pipeline RAG:

1. Conectas fuentes (S3, web crawler, Confluence… según conectores).
2. Bedrock chunking + embeddings + índice (a menudo OpenSearch Serverless).
3. **RetrieveAndGenerate** — recupera contexto y genera respuesta.

| Ventaja | Consideración |
|---------|---------------|
| Menos código custom | Coste de OpenSearch Serverless + indexación |
| Integración IAM AWS | Lock-in en patrón AWS |
| Time-to-market POC | Corpus debe estar **curado** |

> [!NOTE]
> Knowledge Base **no elimina** gobierno documental: documentos obsoletos o clasificados incorrectamente producen respuestas incorrectas o filtraciones.

## Agents for Amazon Bedrock

Orquestan **acción + razonamiento**: llamar APIs (Action Groups), consultar KB, encadenar pasos.

Útiles para asistentes que consultan sistemas internos **con allowlist estricta**.

Riesgos: superficie de acción amplia — exige revisión de seguridad por cada API expuesta.

## RAG en AWS — piezas alternativas

No todo RAG pasa por Knowledge Bases:

| Enfoque | Componentes |
|---------|-------------|
| KB managed | S3 + Bedrock KB + fundacional |
| DIY | S3 + Lambda/Glue embeddings + OpenSearch + Bedrock |
| Híbrido | Azure AI Search on-prem index + Bedrock (raro, multi-cloud) |

La decisión suele ser **build vs managed** en indexación y operación de vectores.

## Seguridad y compliance

- **IAM:** quién puede invocar qué modelo.
- **VPC:** endpoints privados sin salida a internet pública.
- **Cifrado:** KMS en reposo y tránsito.
- **Logging:** CloudWatch / CloudTrail — atención a **retención de prompts** (PII).
- **Contrato AWS:** términos de uso de modelos (entrenamiento con tus datos — normalmente **no** en enterprise, verificar).

## Coste Bedrock

Drivers:

- Tokens input/output por modelo.
- Indexación y almacenamiento KB.
- OpenSearch Serverless si aplica.
- Invocaciones de Agent con múltiples pasos.

**Control:** presupuestos AWS, cuotas por aplicación, autenticación obligatoria.

## Resumen

- Bedrock = API managed de fundacionales + Guardrails + KB + Agents.
- RAG enterprise = corpus gobernado + embeddings + políticas — Bedrock acelera, no sustituye datos.
- Agents amplían capacidad con riesgo operativo y de seguridad — madurez previa requerida.
