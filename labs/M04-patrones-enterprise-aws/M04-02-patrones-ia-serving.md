# M04-02 — Patrones de IA y serving

[← Página anterior](M04-01-patrones-plataforma-datos.md) · [Siguiente página →](M04-03-kubernetes-plataforma-ml.md)

## Batch inference

Modelo aplicado a **grandes volúmenes** en job programado; salida a tabla/fichero.

```
S3 input → Batch transform (SageMaker) / Spark job → S3 output → BI
```

**Cuándo:** informes diarios, scoring masivo sin usuario esperando.

**Coste:** optimizar ventana horaria; apagar recursos tras job.

## Online inference

API síncrona o asíncrona con SLA de latencia.

```
Cliente → API Gateway → Lambda / SageMaker endpoint → respuesta JSON
```

**Patrones avanzados:**

- **Multi-model endpoint** — varios modelos, una instancia.
- **Shadow deployment** — tráfico duplicado a modelo nuevo sin afectar respuesta.
- **A/B** — porcentaje tráfico a versión B.

## RAG (patrón detallado)

```
Usuario → Auth → Orquestador → [Retrieve: índice vectorial] → [Generate: LLM] → respuesta + citas
                      ↑
                 Guardrails / políticas
```

**Variantes:**

- **Naive RAG** — retrieve + prompt simple.
- **Hybrid search** — BM25 + vectores.
- **Graph RAG** — relaciones entre documentos (complejidad alta).

**Calidad:** chunk size, metadata filters (solo docs vigentes), evaluación con golden questions.

## Fine-tuning como patrón

Dataset curado → job entrenamiento → modelo custom → endpoint dedicado.

**Cuándo:** jerga interna extrema, formato salida rígido, RAG insuficiente.

**Coste:** entrenamiento + **re-entrenamiento** ante drift documental.

## Agentes

Bucle: plan → tool call → observación → plan …

Componentes:

- LLM fundacional.
- **Tool registry** (APIs permitidas).
- Memoria conversacional acotada.
- Human approval gates en acciones sensibles.

> [!WARNING]
> Agente con write access a sistemas operacionales (horarios, señales, personal) es **alto riesgo** — exige sandbox, allowlist y auditoría forense.

## Multi-model routing

Enrutar petición al modelo adecuado:

| Router por | Ejemplo |
|------------|---------|
| Tipo tarea | Resumen vs código vs traducción |
| Coste | Modelo barato primero; escalar si confianza baja |
| Compliance | Docs sensibles → modelo on-prem/VPC only |
| Idioma | Modelo multilingüe vs especializado |

Implementación: Lambda/Bedrock con reglas o modelo clasificador ligero.

## Observabilidad específica ML/GenAI

| Métrica | ML clásico | GenAI |
|---------|------------|-------|
| Accuracy/F1 | Sí | A veces difícil |
| Latencia p99 | Sí | Sí |
| Drift features | Sí | N/A |
| Token usage | No | Sí |
| Groundedness | No | Sí (RAG) |
| User thumbs down | Opcional | Muy útil |

## Resumen

- Elige batch vs online según SLA — no mezcles requisitos.
- RAG es el patrón GenAI enterprise por defecto; agentes son escalación con gobierno.
- Multi-model routing controla coste y compliance sin multiplicar apps.
