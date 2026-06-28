# M02-05 — Observabilidad, coste y seguridad

[← Página anterior](M02-04-bedrock-genai-aws.md) · [Siguiente página →](M02-06-sagemaker-vs-kubernetes-mlflow.md)

## Observabilidad en plataformas cloud IA

Observabilidad = **métricas + logs + trazas** con contexto suficiente para diagnosticar fallos y degradación de modelos.

### Amazon CloudWatch

Servicio central de **métricas y logs** en AWS.

| Uso | Ejemplos |
|-----|----------|
| Métricas infra | CPU, memoria, latencia ALB |
| Logs | `/aws/lambda/...`, endpoints SageMaker, Bedrock |
| Alarmas | SNS cuando latencia > umbral |
| Dashboards | Vista operativa para NOC |

**Coste oculto:** retención de logs larga sin filtrar — volumen GenAI (prompt/response logging) puede ser enorme.

### AWS X-Ray

**Trazas distribuidas** — seguir una petición a través de Lambda, API Gateway, DynamoDB, etc.

Útil para latencia en cadenas de inferencia multi-servicio.

### OpenTelemetry (OTel)

Estándar **abierto** para instrumentación (métricas, trazas, logs).

Patrón moderno: apps emiten OTel → collector → CloudWatch, Prometheus, Grafana, Datadog…

**Decisión arquitectónica:** OTel evita lock-in en instrumentación aunque el backend sea AWS.

## Cost Explorer y gestión de coste

**AWS Cost Explorer** y **Budgets** — visibilidad y alertas de gasto.

Prácticas para plataformas IA:

| Práctica | Objetivo |
|----------|----------|
| **Tagging** obligatorio (`proyecto`, `entorno`, `owner`) | Imputar coste por iniciativa |
| Presupuestos y alertas | Detectar desviaciones |
| Informes por servicio | Separar S3, SageMaker, Bedrock |
| Rightsizing | Instancias sobredimensionadas |
| Apagado programado | Dev/test fuera de horario |

**Cost drivers IA en AWS:**

1. Endpoints SageMaker GPU persistentes.
2. Tokens Bedrock sin control de uso.
3. OpenSearch / KB infra 24/7.
4. NAT Gateway y egress cross-AZ.
5. Logs verbose sin sampling.

## Seguridad y compliance (visión AWS)

### IAM (Identity and Access Management)

- Usuarios humanos → **roles** federados (SSO), no claves permanentes.
- Servicios → roles con **least privilege**.
- Políticas separadas: `data-scientist`, `ml-deployer`, `read-only-audit`.

### VPC y networking

- Subnets privadas para endpoints de inferencia.
- **VPC endpoints** para S3, Bedrock — tráfico sin internet pública.
- Security groups — mínimo puerto necesario.

### Cifrado

- **KMS** — claves customer-managed para buckets y modelos sensibles.
- TLS en tránsito obligatorio.

### Auditoría

- **CloudTrail** — quién llamó a qué API.
- Logs de acceso S3.
- Integración con SIEM corporativo.

### Compliance compartida

AWS certifica infraestructura (ISO, SOC…); **tú** certificas configuración, datos y procesos. En sector público pueden aplicarse marcos adicionales (ENS en España, etc.) — validar con legal.

## Well-Architected — lentes útiles

El **AWS Well-Architected Framework** ofrece pilares aplicables a IA:

| Pilar | Pregunta en IA |
|-------|----------------|
| Excelencia operacional | ¿Runbooks para deriva de modelo? |
| Seguridad | ¿IAM y cifrado en todo el flujo? |
| Fiabilidad | ¿Multi-AZ? ¿Fallback si Bedrock falla? |
| Eficiencia de rendimiento | ¿GPU justificada? |
| Optimización de costes | ¿Endpoints apagados fuera de uso? |
| Sostenibilidad | ¿Región y sizing eficientes? |

## Integración con operación enterprise

En transporte público, conecta observabilidad cloud con:

- Centro de monitorización existente (alertas críticas).
- Gestión de incidencias (ITSM).
- Separación **OT/IT** donde regulación lo exija — no mezclar telemetria de seguridad ferroviaria con POC GenAI sin fronteras claras.

## Resumen

- CloudWatch + X-Ray + OTel cubren salud técnica; Model Monitor / métricas de negocio cubren salud del modelo.
- Cost Explorer y tags son imprescindibles — IA sin presupuesto es riesgo financiero.
- IAM, VPC, KMS y CloudTrail son la base de seguridad — Bedrock no los sustituye.
