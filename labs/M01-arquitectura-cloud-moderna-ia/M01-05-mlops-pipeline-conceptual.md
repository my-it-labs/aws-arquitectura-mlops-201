# M01-05 — MLOps — pipeline conceptual

[← Página anterior](M01-04-ia-generativa-fundamentos.md) · [Siguiente página →](../M02-servicios-aws-plataformas/README.md)

## Qué es MLOps

**MLOps** (Machine Learning Operations) es el conjunto de prácticas para **desplegar, monitorizar y gobernar modelos ML** en producción con la misma rigurosidad que el software tradicional — pero teniendo en cuenta que el «artefacto» incluye datos, código y pesos del modelo.

Objetivos:

1. **Reproducibilidad:** mismo dato + mismo código → mismo modelo (dentro de tolerancia).
2. **Trazabilidad:** saber qué versión está en prod y quién la aprobó.
3. **Automatización:** reducir despliegues manuales propensos a error.
4. **Monitorización:** detectar degradación antes de impacto al negocio.

MLOps **no es** un producto único: es un **pipeline** de herramientas y procesos.

## Pipeline simplificado de referencia

```
┌──────────┐    ┌─────────────┐    ┌──────────────┐    ┌─────────────┐
│  Datos   │───▶│ Preparación │───▶│ Entrenamiento│───▶│  Evaluación │
│ versionado│   │  features   │    │  / tuning    │    │  métricas   │
└──────────┘    └─────────────┘    └──────────────┘    └──────┬──────┘
                                                              │
                    ┌─────────────────────────────────────────┘
                    ▼
            ┌──────────────┐    ┌─────────────┐    ┌──────────────┐
            │   Registro   │───▶│  Despliegue │───▶│ Monitorización│
            │   de modelos │    │  (endpoint) │    │  + retrain   │
            └──────────────┘    └─────────────┘    └──────────────┘
```

Cada caja es candidata a servicios managed (p. ej. SageMaker Pipelines, Registry) o a herramientas open source (MLflow, Kubeflow) sobre Kubernetes.

## Etapas en detalle

### 1. Datos versionados

- Snapshots o referencias a datasets aprobados.
- Linaje: de qué sistemas operacionales provienen.
- **No** usar datos de producción sin anonimizar cuando la política lo exija.

### 2. Preparación y features

- Transformaciones documentadas (no notebooks sueltos en producción).
- Feature store opcional pero valioso a escala.

### 3. Entrenamiento y experimentación

- Registro de hiperparámetros y métricas (experiments).
- Separación clara entre **sandbox** de científicos y **pipeline** productivo.

### 4. Evaluación y gates

- Umbrales mínimos de precisión/recall o métricas de negocio.
- Tests de fairness o estabilidad cuando aplique regulación.
- **Gate humano** en sectores críticos antes de prod.

### 5. Registro de modelos (Model Registry)

- Versiones con estado: Staging, Approved, Deprecated.
- Metadatos: autor, fecha, dataset, métricas.

### 6. Despliegue

- Endpoint online, batch job o empaquetado en app.
- Estrategias: blue/green, canary (tráfico parcial al modelo nuevo).

### 7. Monitorización y reentrenamiento

- Drift de datos, decay de métricas, alertas.
- Decisión explícita: ¿reentrenar automáticamente o revisión periódica?

## MLOps y GenAI

En proyectos RAG, el «modelo» incluye también:

- Versión del corpus documental e índice vectorial.
- Versión del fundacional y de embeddings.
- Plantillas de prompt aprobadas.

El pipeline debe versionar **KB + prompts + modelo**, no solo pesos entrenados.

## Roles en la organización

| Rol | Responsabilad típica en MLOps |
|-----|------------------------------|
| Data engineer | Ingesta, calidad, pipelines |
| ML engineer | Entrenamiento, despliegue, métricas |
| Platform / DevOps | Infra, CI/CD, observabilidad |
| Arquitecto | Coherencia, seguridad, integración |
| Negocio / compliance | Aprobación, KPIs, auditoría |

En perfiles **decisores**, no necesitas operar el pipeline — necesitas **exigir** que exista trazabilidad y gates.

## Anti-patrones MLOps

| Anti-patrón | Consecuencia |
|-------------|--------------|
| «El notebook del experto es prod» | Sin reproducibilidad |
| Sin registro de modelos | Imposible rollback |
| Métricas solo offline | Sorpresa en producción |
| Retrain automático sin supervisión | Propaga datos corruptos |
| Ignorar coste de endpoint | Presupuesto agotado |

## Relación con AWS (vista adelantada)

En AWS, piezas habituales del pipeline mapéan a:

- **S3** — datasets y artefactos.
- **Glue / procesamiento** — ETL.
- **SageMaker Pipelines** — orquestación.
- **Model Registry / Feature Store** — gobierno de modelos y features.
- **Endpoints SageMaker o Lambda** — serving.
- **CloudWatch** — métricas e logs.

En el módulo 2 profundizarás en **qué servicio cubre cada función**.

## Resumen

- MLOps convierte ML en un ciclo de vida gobernable: datos → modelo → despliegue → monitor → retrain.
- El registro de modelos y los gates de aprobación son tan importantes como el algoritmo.
- GenAI extiende MLOps a corpus, índices y prompts — no lo simplifica.
