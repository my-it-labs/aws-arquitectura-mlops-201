# M02-02 — Datos, almacenamiento y analítica

[← Página anterior](M02-01-compute-cargas-ia.md) · [Siguiente página →](M02-03-ecosistema-sagemaker.md)

## El dato como fundamento

Ningún servicio ML o GenAI compensa **datos mal gobernados**. En AWS, la capa de datos suele organizarse en **zonas** (raw → curated → consumo) sobre almacenamiento objeto y servicios de catálogo/consulta.

## Almacenamiento

### Amazon S3 (Simple Storage Service)

**Rol:** almacén de objetos — corazón del **data lake** moderno en AWS.

| Característica | Beneficio |
|----------------|-----------|
| Durabilidad y escala | Prácticamente ilimitado |
| Versionado | Recuperar estados anteriores |
| Lifecycle policies | Mover a Glacier o borrar automáticamente |
| Cifrado | SSE-S3, SSE-KMS |
| Prefijos / buckets | Separación por entorno o sensibilidad |

**Patrones:**

- `raw/` — landing de ficheros sin transformar.
- `processed/` — datasets listos para analítica o ML.
- `artifacts/` — modelos serializados, informes.

**Coste:** almacenamiento barato; **egress** (salida de datos) y requests pueden sorprender a escala.

### Amazon EFS (Elastic File System)

Sistema de ficheros **NFS elástico** compartido entre instancias o contenedores.

**Cuándo:** aplicaciones que necesitan **filesystem POSIX** compartido (no solo objetos). Menos habitual como lago principal que S3.

### Amazon FSx

Servicios de filesystem administrados (Windows, Lustre, NetApp ONTAP, OpenZFS).

**Cuándo:** cargas HPC, Lustre para entrenamiento distribuido de alto rendimiento, integración con sistemas file legacy.

## Catálogo, ETL y preparación

### AWS Glue

**Qué es:** servicio **serverless** de descubrimiento, catálogo y ETL.

Componentes útiles:

- **Glue Data Catalog** — metadatos de tablas (integración con Athena, Redshift, EMR).
- **Glue Jobs** — scripts Spark/Python para transformar.
- **Glue Crawlers** — inferir esquema desde S3.

**Valor arquitectónico:** linaje centralizado y menos «ficheros huérfanos» en el lago.

## Consulta serverless

### Amazon Athena

SQL sobre S3 (formatos Parquet, ORC, JSON…) **sin provisionar servidores**.

- Pago por datos escaneados.
- Ideal para exploración, auditoría, informes ad hoc.
- Optimización: particiones, columnar (Parquet), compresión.

## Data warehouse

### Amazon Redshift

Warehouse columnar para analítica SQL a escala, BI y agregaciones complejas.

| vs Athena | Cuándo Redshift |
|-----------|-----------------|
| Serverless ad hoc | Cargas BI constantes, muchos usuarios concurrentes |
| Pago por scan | Clusters reservados con rendimiento predecible |

Opciones modernas incluyen **Redshift Serverless** para reducir gestión de clusters.

## Flujo de referencia en plataforma IA

```
Fuentes operacionales
        │
        ▼
   S3 (raw) ──► Glue (catalog + ETL) ──► S3 (curated)
        │                                      │
        │                                      ├──► Athena (exploración)
        │                                      ├──► Redshift (BI)
        │                                      └──► SageMaker (entrenamiento)
```

## Clasificación y zonas (enterprise)

En sector público o transporte, define **zonas por sensibilidad**:

| Zona | Contenido | Controles |
|------|-----------|-----------|
| Pública / interna baja | Agregados, documentación no sensible | IAM estándar |
| Confidencial | Datos operativos identificables | Cifrado, acceso mínimo, logging |
| Restringida | RRHH, contratos, PII | Anonimización, DLP, aprobaciones |

El **bucket no es seguridad** por sí solo: IAM, KMS, VPC endpoints y políticas de bucket se combinan.

## OpenSearch (dato + búsqueda)

**Amazon OpenSearch Service** — búsqueda full-text, analytics y **vectores** para RAG.

Encaja como índice híbrido (keyword + semantic) en arquitecturas GenAI junto a Bedrock.

## Preguntas al evaluar diseño de datos

1. ¿Cuál es el **sistema de registro** (source of truth) frente al lago analítico?
2. ¿Hay **retención legal** distinta por tipo de dato?
3. ¿El ML usa datos **anonimizados** o agregados?
4. ¿Quién mantiene el **catálogo** cuando cambian esquemas?
5. ¿Cómo se controla el **coste de consultas** (Athena scan, Redshift cluster)?

## Resumen

- S3 es el hub del lago; Glue cataloga y transforma; Athena explora; Redshift agrega a escala BI.
- EFS/FSx cubren necesidades de filesystem, no sustituyen el patrón lago-objeto.
- Gobierno de zonas y clasificación precede a cualquier pipeline ML o RAG serio.
