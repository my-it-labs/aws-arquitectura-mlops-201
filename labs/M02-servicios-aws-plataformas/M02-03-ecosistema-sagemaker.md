# M02-03 — Ecosistema SageMaker

[← Página anterior](M02-02-datos-almacenamiento-analitica.md) · [Siguiente página →](M02-04-bedrock-genai-aws.md)

## Qué es Amazon SageMaker

**Amazon SageMaker** es la plataforma **managed** de AWS para todo el ciclo ML: preparar datos, entrenar, registrar, desplegar y monitorizar modelos — sin montar desde cero la infraestructura de cada fase.

Ventaja para decisiones arquitectónicas: **time-to-market** y menos piezas moving parts que K8s DIY.

Limitación: **lock-in** moderado hacia APIs y patrones AWS; coste si endpoints quedan infrautilizados.

## Componentes principales

### SageMaker Studio / Domain

Entorno de trabajo para científicos de datos: notebooks, experimentos, acceso a datos catalogados.

**Decisión:** separar dominios por equipo o proyecto; IAM fino por persona.

### SageMaker Training Jobs

Entrenamiento managed en instancias CPU/GPU que **arrancan y terminan** con el job.

- Pago por segundos de entrenamiento.
- Integración con datos en S3.
- Soporte frameworks (PyTorch, TensorFlow, XGBoost, etc.).

### SageMaker Processing

Jobs para pre/post-procesamiento (validación, transformación batch) sin mezclarlo con entrenamiento.

### SageMaker Pipelines

Orquestación **declarativa** del flujo ML: pasos encadenados con dependencias, re-ejecución, parametrización.

Equivalente conceptual a un CI/CD del modelo.

```
Ingesta S3 → Processing → Training → Evaluación → Register → Deploy (condicional)
```

**Valor:** reproducibilidad y auditoría — «¿qué pipeline produjo el modelo en prod?»

### Model Registry

Repositorio de versiones de modelos con estados (Pending, Approved, Rejected).

- Metadatos y métricas asociadas.
- Gate antes de despliegue automático.

### Feature Store

Almacén de features **online y offline** con mismas definiciones para entrenamiento e inferencia.

Reduce el *training-serving skew* — discrepancia entre lo que ve el modelo en prod y en entrenamiento.

### Endpoints (Hosting)

Despliegue de modelos para **inferencia online** con autoscaling.

| Aspecto | Implicación |
|---------|-------------|
| Instance type | CPU vs GPU — coste dominante |
| Multi-model endpoint | Varios modelos en una instancia |
| Serverless Inference | Escala a cero con cold start |
| Async inference | Colas para inferencia no síncrona |

> [!WARNING]
> Un endpoint GPU **InService** 24/7 sin tráfico es uno de los errores de coste más comunes. Exige **autoscaling**, horarios o serverless según patrón.

### SageMaker Model Monitor

Detecta drift de datos y calidad en endpoints desplegados.

## Arquitectura de referencia (alto nivel)

```
        ┌─────────────────────────────────────────────┐
        │           SageMaker Domain / Studio          │
        └─────────────────────┬───────────────────────┘
                              │
    S3 ◄────► Pipeline ──► Training ──► Registry ──► Endpoint
              ▲                              │
              └── Feature Store ─────────────┘
                              │
                         CloudWatch (métricas)
```

## Cuándo SageMaker es buena opción

- Equipo ML pequeño/medio sin K8s maduro.
- Necesidad de auditoría y pipelines reproducibles rápido.
- Cargas principalmente en AWS y S3.

## Cuándo cuestionar SageMaker

- Requisito fuerte de **portabilidad** multi-cloud.
- Ecosistema ya 100 % Kubernetes con MLflow/Kubeflow operado por platform team grande.
- Modelos ultra-custom que chocan con abstracciones SageMaker.

## Resumen

- SageMaker cubre pipeline ML managed: Pipelines, Registry, Feature Store, Training, Endpoints.
- El valor no es «usar SageMaker» sino **operar ML con trazabilidad** — Registry y Pipelines son las piezas clave de gobierno.
- Endpoints son el principal poste de coste recurrente — dimensiona por tráfico real.
