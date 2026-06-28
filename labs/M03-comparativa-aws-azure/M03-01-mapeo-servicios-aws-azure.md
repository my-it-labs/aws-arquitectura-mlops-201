# M03-01 — Mapeo de servicios AWS ↔ Azure

[← Página anterior](README.md) · [Siguiente página →](M03-02-on-prem-cloud-hibrido.md)

## Cómo usar esta tabla

Los servicios **no son idénticos** — la tabla indica **equivalencia funcional** para conversaciones arquitectónicas y licitaciones. Siempre valida: regiones, SLAs, precios, compliance y features concretas.

## Compute

| Función | AWS | Azure | Comentarios |
|---------|-----|-------|-------------|
| Máquinas virtuales | **EC2** | **Virtual Machines** | Familias CPU/GPU en ambos; modelos de reserva similares |
| Contenedores (K8s) | **EKS** | **AKS** | AKS integra fuerte con Entra ID; EKS ecosistema AWS nativo |
| Contenedores (simple) | **ECS**, Fargate | **Container Apps**, ACI | Abstracción distinta |
| Serverless funciones | **Lambda** | **Azure Functions** | Triggers, timeout y networking difieren |
| Batch / HPC | **AWS Batch**, ParallelCluster | **Azure Batch** | Jobs GPU episódicos |

## Red e identidad

| Función | AWS | Azure | Comentarios |
|---------|-----|-------|-------------|
| Red virtual | **VPC** | **Virtual Network** | Subnets, peering, hybrid |
| DNS | **Route 53** | **Azure DNS** | |
| CDN | **CloudFront** | **Azure Front Door**, CDN | |
| Identidad workforce | **IAM** + IAM Identity Center | **Microsoft Entra ID** | **Azure ventaja** en orgs AD/M365 |
| Identidad workloads | IAM roles | **Managed identities** | Patrones distintos — diseño cuidadoso en híbrido |

## Almacenamiento y datos

| Función | AWS | Azure | Comentarios |
|---------|-----|-------|-------------|
| Objeto | **S3** | **Blob Storage** | Lago analítico en ambos |
| File SMB / NFS | **EFS**, FSx | **Azure Files**, NetApp | |
| Data lake analytics | S3 + Glue + Athena | **ADLS Gen2** + Synapse serverless | Modelos de permisos distintos |
| ETL / integración | **Glue**, AppFlow | **Azure Data Factory** | Orquestación híbrida fuerte en ADF |
| Warehouse | **Redshift** | **Synapse Analytics** | |
| SQL managed | **RDS**, Aurora | **Azure SQL**, Cosmos DB | |
| Streaming | **Kinesis** | **Event Hubs** | |

## IA y ML

| Función | AWS | Azure | Comentarios |
|---------|-----|-------|-------------|
| Plataforma ML | **SageMaker** | **Azure Machine Learning** | Pipelines, registry, endpoints en ambos |
| GenAI fundacionales | **Bedrock** | **Azure OpenAI Service** | Contratos y modelos difieren |
| Búsqueda + vectores | **OpenSearch** | **Azure AI Search** | RAG patterns |
| Speech / Vision APIs | Rekognition, Transcribe… | Azure Cognitive Services | Catálogo cambiante |
| ML open en K8s | EKS + add-ons | AKS + MLflow | Depende de platform team |

## Observabilidad y gestión

| Función | AWS | Azure |
|---------|-----|-------|
| Métricas / logs | **CloudWatch** | **Azure Monitor**, Log Analytics |
| Trazas | **X-Ray** | **Application Insights** |
| Coste | **Cost Explorer** | **Cost Management** |
| Governance multi-sub | **Organizations**, Control Tower | **Management Groups**, Policy |
| Landing zone | **Control Tower** | **Azure Landing Zones** |

## Open source transversal

Independiente de nube:

- **Kubernetes**, **MLflow**, **Kubeflow**, **Airflow** (MWAA en AWS, varios en Azure).
- **LangChain** / **LlamaIndex** — orquestación RAG (código, no servicio managed).
- **OpenTelemetry** — instrumentación portable.

## Diferencias que no aparecen en la tabla

1. **Facturación y descuentos** — Enterprise Agreements Microsoft vs EDP AWS.
2. **Regiones** — soberanía (EU, España) varía por servicio.
3. **Skills internos** — certificaciones y experiencia previa del equipo.
4. **Contratos marco** — sector público suele condicionar proveedor.

## Resumen

- Existe par funcional para casi cada bloque de plataforma IA moderna.
- La diferencia decisiva suele ser **Entra ID vs IAM** y **Synapse/ADF vs Glue/Redshift**, no el nombre del contenedor.
