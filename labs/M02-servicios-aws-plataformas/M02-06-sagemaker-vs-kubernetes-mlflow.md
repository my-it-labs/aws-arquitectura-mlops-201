# M02-06 — SageMaker vs Kubernetes + MLflow

[← Página anterior](M02-05-observabilidad-coste-seguridad.md) · [Siguiente página →](../M03-comparativa-aws-azure/README.md)

## El dilema arquitectónico

Muchas organizaciones se enfrentan a:

- **Opción A:** Amazon SageMaker (managed ML de AWS).
- **Opción B:** **EKS** (o Kubernetes on-prem) + **MLflow** (tracking, registry) + herramientas serving (KServe, Seldon, custom).

No hay ganador universal — depende de **skills**, **portabilidad**, **madurez platform** y **patrones de carga**.

## Qué aporta cada enfoque

### SageMaker managed

| Fortaleza | Detalle |
|-----------|---------|
| Integración nativa | S3, IAM, Pipelines, endpoints |
| Menor time-to-first-model | Menos montaje inicial |
| Operación ML abstraída | Training jobs, registry managed |
| Cumplimiento AWS | Un vendor en contrato marco |

| Debilidad | Detalle |
|-----------|---------|
| Lock-in AWS | APIs y patrones propietarios |
| Flexibilidad limitada | Casos muy custom pueden frustrar |
| Coste endpoints | Si no se optimiza |

### Kubernetes + MLflow (y ecosistema)

| Fortaleza | Detalle |
|-----------|---------|
| Portabilidad | Multi-cloud, híbrido |
| Ecosistema abierto | MLflow, Kubeflow, Ray, etc. |
| Control fino | GPU scheduling, networking custom |
| Reutilización | Si ya hay **platform team K8s** |

| Debilidad | Detalle |
|-----------|---------|
| Coste operativo | Upgrades, seguridad, observabilidad cluster |
| Time-to-market | Meses para plataforma robusta |
| Fragmentación | Cada equipo monta distinto sin gobierno |

## Matriz de decisión

| Criterio | SageMaker | EKS + MLflow |
|----------|-----------|--------------|
| **Flexibilidad técnica** | Media-alta en AWS | Alta |
| **Lock-in** | Mayor (AWS) | Menor en capa ML |
| **Curva aprendizaje (data scientists)** | Moderada | MLflow fácil; K8s difícil para DS |
| **Curva operativa (platform)** | Menor | Alta |
| **Coste infra directo** | Endpoints + jobs | Cluster 24/7 + personas |
| **Escalabilidad GPU** | Managed autoscaling | Requiere diseño node pools |
| **Auditoría / enterprise** | Registry + Pipelines integrados | Hay que estandarizar MLflow + GitOps |
| **GenAI (Bedrock)** | Complementario en ambos casos | Complementario |

## Escenarios guía

### Inclínate a SageMaker si…

- Primeros proyectos ML en AWS con equipo pequeño.
- Prioridad **time-to-market** y auditoría básica rápida.
- No hay equipo Kubernetes dedicado.
- Cargas ML mayoritariamente en S3/SageMaker.

### Inclínate a EKS + MLflow si…

- Ya existe **cluster productivo** con SRE maduro.
- Requisito explícito multi-cloud / on-prem híbrido.
- Muchos equipos comparten **misma plataforma** container.
- Modelos serving custom (GPU sharing, A/B complejo) no cubiertos bien por SageMaker.

### Enfoque híbrido (frecuente)

- **SageMaker** para entrenamiento managed + Registry como source of truth.
- **EKS** solo para serving de apps microservicio que consumen modelos exportados.
- **Bedrock** para GenAI independiente del stack ML clásico.

Evita **dos plataformas completas** sin integración — duplica coste y confusión.

## MLflow en una frase

**MLflow** rastrea experimentos, empaqueta modelos y ofrece **Model Registry** open source. Corre on-prem, en EC2, en EKS o integrado con SageMaker (como tracking externo).

No es sustituto automático de todo SageMaker — es **capa de gobierno ML** portable.

## Preguntas para un integrador

1. ¿Por qué EKS y no SageMaker para **este** caso concreto?
2. ¿Quién opera el cluster a las 3 AM?
3. ¿Dónde vive el Model Registry autoritativo?
4. ¿Cómo se integra Bedrock/RAG con el mismo IAM y logging?
5. ¿TCO 3 años — licencias, personas, formación?

## Resumen

- SageMaker optimiza velocidad y operación ML dentro de AWS; K8s+MLflow optimiza portabilidad y control con coste operativo alto.
- La decisión correcta es la que tu organización puede **sostener** — no la del diagrama más moderno.
- GenAI (Bedrock) es ortogonal: convive con cualquiera de las dos vías si gobierno e identidad son comunes.
