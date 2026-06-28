# M04-01 — Patrones de plataforma y datos

[← Página anterior](README.md) · [Siguiente página →](M04-02-patrones-ia-serving.md)

## Data lake

**Idea:** centralizar datos en **almacenamiento objeto** (S3) en formatos abiertos (Parquet, JSON), schema-on-read.

**Componentes AWS típicos:** S3 + Glue Catalog + Athena/EMR/Redshift Spectrum.

| Beneficio | Riesgo |
|-----------|--------|
| Barato, escalable | «Data swamp» sin catálogo |
| Flexibilidad esquemas | Calidad inconsistente |
| Punto de entrada ML | Permisos mal diseñados |

**Gobierno mínimo:** zonas raw/curated, owners por dataset, lifecycle policies.

## Lakehouse

**Idea:** combinar flexibilidad del lago con **transacciones y gobernanza** tipo warehouse (Delta Lake, Iceberg, Hudi sobre S3).

| vs Data lake puro | Aporta |
|-------------------|--------|
| ACID en tablas | Menos corrupción concurrente |
| Time travel | Auditoría y rollback |
| Unificación BI + ML | Menos copias |

En AWS: EMR, Glue, Redshift con tablas Iceberg/Delta según stack elegido.

## Event-driven architecture

**Idea:** componentes se comunican por **eventos** (Kinesis, SQS/SNS, EventBridge) en lugar de acoplamiento síncrono.

**Encaje transporte:** telemetría vehículo → stream → procesamiento → alerta / modelo.

| Beneficio | Complejidad |
|-----------|-------------|
| Desacoplamiento | Orden, duplicados, idempotencia |
| Escalado independiente | Debug distribuido |
| Resiliencia | Schema evolution en eventos |

## Microservicios

**Idea:** capacidades de negocio en servicios desplegables independientemente (contenedores/Lambda).

**Relación IA:** microservicio de «scoring» o «recomendación» consume endpoint ML; **no** meter modelo dentro de monolito sin versionado.

## API-first

**Idea:** contratos API (OpenAPI) **antes** que implementación; consumo uniforme web/móvil/partners.

**IA:** expone inferencia vía API Gateway + autorización — nunca endpoints ML sin auth en prod.

## Arquitectura desacoplada (strangler)

**Idea:** sustituir legacy gradualmente envolviendo con APIs nuevas — crítico en operadores con sistemas décadas.

```
Legacy mainframe/ERP ──► API façade ──► nuevos servicios cloud / ML
```

## Comparativa rápida

| Patrón | Problema que resuelve | Servicio AWS orientativo |
|--------|----------------------|--------------------------|
| Data lake | Silos de ficheros | S3 + Glue |
| Lakehouse | Gobernanza + lago | S3 + Iceberg + Glue |
| Event-driven | Picos, desacoplar | Kinesis, EventBridge |
| Microservicios | Releases independientes | ECS/EKS, Lambda |
| API-first | Integración ordenada | API Gateway |

## Anti-patrones de plataforma

- Un bucket S3 «para todo» sin prefijos ni IAM fino.
- Copiar datos 5 veces porque cada equipo quiere su silo.
- Stream sin retención ni DLQ (dead letter queue).
- Microservicios nano sin dominio claro — solo latencia de red.

## Resumen

- Lake/lakehouse resuelven **dato**; event-driven resuelve **tiempo real**; microservicios/API resuelven **entrega de software**.
- Toda plataforma IA seria combina al menos **lago + catálogo + APIs**.
