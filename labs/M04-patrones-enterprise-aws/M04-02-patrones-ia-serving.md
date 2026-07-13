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

### Gestor de agentes (orquestador)

Un **gestor de agentes** no es solo el modelo: es la **capa que orquesta** la tarea. Recibe la petición, monta contexto, elige herramientas, ejecuta pasos, lee resultados y repite hasta completar o pedir intervención humana.

```
Usuario (prompt)
    → Gestor / orquestador
        → Modelo (razona: qué hacer ahora)
        → Tool call (consultar API, leer doc, ejecutar acción permitida)
        → Observación (resultado)
        → Modelo (¿sigo, corrijo, termino?)
    → Respuesta o efecto en sistemas externos
```

| Pieza del gestor | Función |
|------------------|---------|
| **Modelo fundacional** | Razonamiento y planificación |
| **Tool registry** | Lista cerrada de capacidades (solo lo permitido) |
| **Contexto** | Corpus, reglas, estado de sesión, metadata |
| **Políticas** | Guardrails, permisos, modos de operación |
| **Human-in-the-loop** | Aprobación antes de acciones sensibles |
| **Subagentes / workers** | Delegar subtareas acotadas (exploración, ejecución) |

En **Amazon Bedrock**, el rol de gestor lo cumple **Agents for Bedrock** más la aplicación que lo invoca. En otros entornos puede ser código propio (Lambda, Step Functions) o productos de terceros.

### Ejemplo didáctico: IDE con agente (p. ej. Cursor)

Un entorno de desarrollo con agente integrado ilustra el patrón sin ser un sistema operacional crítico:

| Pieza | En un IDE con agente | Equivalente enterprise |
|-------|----------------------|------------------------|
| Corpus / contexto | Código, docs y reglas del proyecto | Procedimientos, KB, políticas |
| Tools | Leer/editar archivos, buscar, terminal, APIs MCP | Action Groups, consultas SAP/incidencias (RO) |
| Autonomía | Modos lectura vs edición/ejecución | Copiloto CCO vs publicación autónoma |
| Supervisión | Usuario aprueba comandos y valida cambios | Operador valida antes de actuar en OT |

**Comportamiento típico:**

1. **Interpretar** la intención (explicar, buscar, modificar, desplegar).
2. **Montar contexto** (workspace, reglas, archivos relevantes).
3. **Elegir herramientas** de la allowlist (no inventa capacidades nuevas).
4. **Iterar** con observaciones reales (errores de build, ficheros no encontrados).
5. **Parar** al terminar, ante límites o si falta una decisión humana.

**Qué no es:** un agente 24/7 desatendido, infalible ni con acceso ilimitado. El valor arquitectónico está en **orquestación + herramientas acotadas + supervisión** — el mismo esquema que un copiloto en sala de control o mantenimiento (ver [M04-05](M04-05-casos-uso-agentes-tmb.md)), aplicado aquí a desarrollo de software.

| Cursor / IDE agente | Agente operaciones (transporte) |
|---------------------|----------------------------------|
| Repo + reglas | Procedimientos + APIs allowlist |
| Editar código con OK del usuario | Publicar mensaje al pasajero solo con aprobación |
| Subagente de exploración | Worker que consulta un sistema concreto |

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
- RAG es el patrón GenAI enterprise por defecto; agentes son escalación con gobierno — ver casos TMB en [M04-05](M04-05-casos-uso-agentes-tmb.md).
- Un **gestor de agentes** orquesta modelo + tools + políticas + supervisión humana; Bedrock Agents y copilotos operativos comparten el mismo patrón.
- Multi-model routing controla coste y compliance sin multiplicar apps.
