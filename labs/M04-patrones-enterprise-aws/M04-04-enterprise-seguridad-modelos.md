# M04-04 — Enterprise, landing zones y seguridad de modelos

[← Página anterior](M04-03-kubernetes-plataforma-ml.md) · [Siguiente página →](../M05-costes-gobierno-decisiones/README.md)

## Landing Zone

**Landing zone** es una **cuenta/subscription base** preconfigurada con:

- Estructura OU / Management Groups.
- IAM roles baseline.
- Logging centralizado (CloudTrail → SIEM).
- Red hub-spoke o similar.
- Guardrails (SCPs en AWS, Azure Policy).

Objetivo: que cada proyecto **no reinvente** seguridad desde cero.

### AWS Control Tower

Automatiza landing zone multi-cuenta:

| Cuenta típica | Uso |
|---------------|-----|
| Management | Facturación, org |
| Log archive | Logs inmutables |
| Audit | Security tooling |
| Workload dev/test/prod | Cargas ML separadas |

**IA:** entrenamiento en dev; prod endpoint en cuenta prod con IAM distinto.

### Azure Landing Zones

Equivalente con management groups, policy sets, hub VNet.

## Multi-account / multi-subscription

**Por qué separar:**

- Blast radius — error en POC no tumba prod.
- Facturación por proyecto (showback).
- Límites servicio (quotas) independientes.
- Compliance — datos sensibles en cuenta restringida.

## IAM empresarial

Principios:

- **Least privilege** — permiso mínimo por rol.
- **Separation of duties** — quien entrena ≠ quien aprueba prod.
- **Federación SSO** — sin IAM users long-lived.
- **Roles por workload** — IRSA en EKS, execution roles SageMaker.

Para Bedrock: políticas que limiten **modelos** y **regiones** invocables.

## Zero trust (aplicado a IA)

No confiar en red perimetral alone:

- Autenticación en cada API ML/GenAI.
- mTLS service-to-service donde aplique.
- Validación continua (device/posture) en acceso corporativo.
- Micro-segmentación — endpoint ML no expuesto a internet 0.0.0.0/0.

## Seguridad de modelos

| Amenaza | Mitigación |
|---------|------------|
| Prompt injection | Input sanitization, system prompt hardened, output filters |
| Model inversion / extraction | Rate limits, no exponer logits |
| Data poisoning (training) | Validación datasets, provenance |
| Supply chain (model artifact) | Scan images, sign artifacts |
| Leakage PII en output | DLP, Guardrails, anonimización |

## Riesgos de datos en GenAI

- Usuario pega datos personales en chat → log con PII.
- RAG indexa documentos **over-classified**.
- Retención logs incompatible RGPD.

**Controles:** clasificación automática pre-index, redacción, TTL logs, opt-in datasets.

## Auditoría

Registros mínimos:

- Quién invocó qué modelo (CloudTrail).
- Versión modelo / KB / prompt template.
- Input/output hash o sample según política (no siempre texto completo).
- Aprobaciones comité ML.

## Anti-patrones enterprise

| Anti-patrón | Por qué duele |
|--------------|---------------|
| Una cuenta AWS para todo | Blast radius, coste opaco |
| Endpoint ML público sin auth | Abuso y coste |
| GenAI sin registro uso | Imposible investigar incidentes |
| Copiar landing zone vendor sin adaptar | Checkbox compliance sin efecto |

## Resumen

- Landing zones y multi-cuenta son la base de gobierno cloud serio.
- IAM + zero trust aplican igual a APIs ML y Bedrock.
- Seguridad de modelos es dominio nuevo — extiende AppSec + DataSec.
