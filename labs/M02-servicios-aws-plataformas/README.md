# M02 — Servicios AWS y Diseño de Plataformas

[← Página anterior](../M01-arquitectura-cloud-moderna-ia/M01-05-mlops-pipeline-conceptual.md) · [Siguiente página →](M02-01-compute-cargas-ia.md)

## Qué aprenderás

- Qué servicios AWS cubren **compute**, **datos**, **ML**, **GenAI** y **operación**.
- Cuándo elegir cada familia y qué **complejidad operativa** implica.
- Cómo encajan SageMaker y Bedrock en una plataforma.
- Criterios para comparar **SageMaker managed** frente a **Kubernetes + MLflow**.

## Índice del módulo

| Cap. | Título | Contenido |
|------|--------|-----------|
| 1 | [Compute para cargas IA](M02-01-compute-cargas-ia.md) | EC2, ECS, EKS, Lambda, Batch, GPU |
| 2 | [Datos y almacenamiento](M02-02-datos-almacenamiento-analitica.md) | S3, EFS, FSx, Glue, Athena, Redshift |
| 3 | [Ecosistema SageMaker](M02-03-ecosistema-sagemaker.md) | Pipelines, Registry, Feature Store, endpoints |
| 4 | [Bedrock y GenAI en AWS](M02-04-bedrock-genai-aws.md) | Modelos, Guardrails, Agents, Knowledge Bases |
| 5 | [Observabilidad, coste y seguridad](M02-05-observabilidad-coste-seguridad.md) | CloudWatch, X-Ray, OTel, Cost Explorer, IAM |
| 6 | [SageMaker vs Kubernetes + MLflow](M02-06-sagemaker-vs-kubernetes-mlflow.md) | Matriz de decisión arquitectónica |

→ Empieza por **[Capítulo 1 — Compute](M02-01-compute-cargas-ia.md)**.

## Idea clave del módulo

AWS ofrece **muchas** opciones a propósito: tu trabajo como arquitecto es **acotar** el menú al patrón de carga y a la capacidad operativa real del equipo.
