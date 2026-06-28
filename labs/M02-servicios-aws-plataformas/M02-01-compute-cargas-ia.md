# M02-01 — Compute para cargas IA

[← Página anterior](README.md) · [Siguiente página →](M02-02-datos-almacenamiento-analitica.md)

## Panorama de compute en AWS

La capa de **compute** ejecuta entrenamiento, inferencia, ETL pesado y orquestación. Para IA, la pregunta central es: *¿necesito control total, contenedores, eventos serverless o jobs batch con GPU?*

| Servicio | Modelo | GPU | Operación típica |
|----------|--------|-----|------------------|
| **EC2** | VMs | Sí (instancias P/G) | Alta — SO, parches, scaling |
| **ECS** | Contenedores AWS-native | Con instancias EC2/GPU | Media — clusters, task defs |
| **EKS** | Kubernetes managed | Sí (node groups GPU) | Alta — K8s completo |
| **Lambda** | Funciones serverless | No (limitado) | Baja — por invocación |
| **AWS Batch** | Jobs batch managed | Sí | Media — colas, jobs |

## EC2 (Elastic Compute Cloud)

**Qué es:** máquinas virtuales con control casi total.

**Cuándo inclinar:**

- Necesitas SO específico, drivers GPU custom o software legacy.
- Carga de entrenamiento puntual donde tú gestionas el ciclo de vida de la instancia.
- Lift-and-shift temporal (con plan de optimización).

**Riesgos:**

- Instancias GPU olvidadas encendidas → coste alto.
- Parches, hardening y backup son responsabilidad tuya.

**Decisión:** EC2 es la opción **más flexible** y la **más operativa** si no automatizas apagado y escalado.

## ECS y EKS (contenedores)

### ECS

Orquestador de contenedores integrado en AWS. Menos «estándar industria» que K8s pero **más simple** si todo vive en AWS.

- Encaja en microservicios de inferencia empaquetados en imagen.
- GPU vía instancias EC2 del cluster.

### EKS (Elastic Kubernetes Service)

Kubernetes gestionado — plano de control AWS, nodos tuyos (EC2 o Fargate).

**Cuándo inclinar:**

- Ya existe **platform team** Kubernetes en la organización.
- Multi-cloud o portabilidad de manifests importa.
- Necesitas ecosistema K8s (Helm, operators GPU, KServe, etc.).

**Riesgos:**

- Complejidad de networking (CNI), upgrades, seguridad de cluster.
- Coste «oculto» en personas y herramientas, no solo en la factura AWS.

> [!NOTE]
> **EKS «porque es moderno»** sin equipo que lo opere es un anti-patrón frecuente. SageMaker o Lambda suelen ser más simples para un primer caso ML acotado.

## Lambda (serverless)

**Qué es:** ejecuta código por evento, sin gestionar servidores.

**Encaje en IA:**

- Inferencia **ligera** (modelos pequeños, pre/post-proceso).
- Orquestación de llamadas a APIs (Bedrock, SageMaker endpoint).
- Procesamiento de eventos (validación, transformación).

**Límites:**

- Timeout máximo (minutos), memoria acotada, **sin GPU**.
- Cold start puede afectar latencia estricta.

**Coste:** excelente para bajo volumen; revisar si el tráfico constante hace más barato un endpoint dedicado.

## AWS Batch

**Qué es:** ejecuta jobs batch en colas sobre compute EC2/Fargate.

**Encaje:**

- Entrenamiento nocturno, scoring masivo, pipelines ETL pesados.
- Escala a cero cuando no hay jobs (si apagas compute subyacente).

Combina bien con **SageMaker** (entrenamiento managed) o sustituye jobs custom en EC2 sueltos.

## GPU workloads — criterios de decisión

| Pregunta | Si «sí» → |
|----------|-----------|
| ¿Entrenamiento frecuente con modelos grandes? | GPU managed (SageMaker training) o Batch |
| ¿Inferencia 24/7 con latencia baja? | Endpoints GPU o instancias reservadas |
| ¿Solo pruebas esporádicas? | Instancias bajo demanda + **apagado automático** |
| ¿Modelo ya pequeño optimizado? | CPU o Lambda puede bastar |

## Matriz resumen

| Patrón de carga | Opciones razonables (orden de simplicidad) |
|-----------------|---------------------------------------------|
| API inferencia ligera | Lambda → SageMaker endpoint → ECS/EKS |
| Inferencia GPU continua | SageMaker endpoint → EKS + KServe |
| Entrenamiento episódico | SageMaker Training → Batch → EC2 manual |
| Plataforma ML multi-equipo | SageMaker dominio → EKS + MLflow |

## Resumen

- EC2 = control máximo; Lambda = simplicidad event-driven; EKS = flexibilidad con coste operativo.
- GPU es el principal driver de coste en compute IA — dimensiona por **patrón de uso**, no por pico teórico permanente.
- El servicio «correcto» es el que el equipo puede **operar de forma sostenible**.
