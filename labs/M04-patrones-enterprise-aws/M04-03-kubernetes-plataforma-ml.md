# M04-03 — Kubernetes como plataforma ML

[← Página anterior](M04-02-patrones-ia-serving.md) · [Siguiente página →](M04-04-enterprise-seguridad-modelos.md)

## Por qué Kubernetes aparece en ML

Kubernetes (**EKS** en AWS) unifica despliegue de:

- Microservicios que consumen modelos.
- Servidores de inferencia (TorchServe, Triton, KServe).
- Notebooks/jupyterhub para equipos (con cautela).
- Pipelines CI/CD GitOps.

Tiene sentido cuando ya es **plataforma estándar** de la organización — no como addon solo para ML.

## GPU scheduling

GPUs son recurso **escaso y caro**:

- **Node pools** dedicados GPU con taints/tolerations.
- **Fractional GPU** / MIG en NVIDIA (particionar GPU).
- Autoscaling cluster: scale-to-zero difícil — latencia cold start nodes.

| Anti-patrón | Consecuencia |
|-------------|--------------|
| Un pod GPU por microservicio mínimo | GPU subutilizada |
| Sin quotas namespace | Un equipo consume todo |

## Autoscaling

- **HPA** (Horizontal Pod Autoscaler) — CPU/mem/custom metrics (cola requests).
- **KEDA** — escala por longitud cola SQS/Kafka.
- **Cluster Autoscaler** — añade nodos EC2.

Para inferencia ML, métrica custom **requests/sec** o **latencia** suele ser mejor que CPU.

## Inferencia distribuida

Modelos grandes pueden requerir **multi-GPU** o sharding — complejidad alta (vLLM, TensorRT-LLM en K8s).

Evaluar si **API managed** (Bedrock) es más barato que operar esto in-house.

## Serving de modelos en K8s

Patrones:

- **KServe** — CRDs para InferenceService, canary, explainability hooks.
- **Seldon** — despliegue multi-modelo.
- **Custom Deployment + HPA** — simple pero manual.

Integración con MLflow Registry: pull artefacto versionado → imagen → deploy.

## Multi-tenant

Varios equipos comparten cluster:

| Mecanismo | Propósito |
|-----------|-----------|
| Namespaces | Aislamiento lógico |
| Resource quotas | Límites CPU/GPU/mem |
| Network policies | Tráfico este-oeste |
| IAM roles for SA (IRSA) | Credenciales AWS por pod |

Sin multi-tenant discipline → un namespace comprometido afecta otros.

## K8s vs managed ML — recordatorio

| Usa EKS para ML si | Usa SageMaker si |
|--------------------|------------------|
| Platform team K8s > 3-5 FTE equivalente | Equipo ML pequeño |
| Portabilidad multi-cloud | Time-to-market |
| Custom serving no soportado managed | Pipelines managed suficientes |

## Resumen

- K8s aporta flexibilidad GPU y serving custom a costa operativa.
- GPU scheduling y multi-tenant mal hechos destruyen ROI del cloud.
- Pregunta siempre: «¿Bedrock/SageMaker endpoint resuelve esto sin cluster?»
