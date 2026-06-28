# M03-02 — On-premise, cloud e híbrido

[← Página anterior](M03-01-mapeo-servicios-aws-azure.md) · [Siguiente página →](M03-03-estrategia-aws-vs-azure.md)

## Por qué sigue existiendo on-premise

La nube resuelve elasticidad y servicios managed, pero **no elimina**:

- Requisitos de **soberanía** o datos que no pueden salir del CPD.
- Inversión sunk en CPD reciente.
- Latencia extrema con sistemas OT/industriales.
- Políticas que exigen ciertos datos en territorio controlado.

La pregunta arquitectónica moderna es **qué cargas** van a cloud, **cuáles** permanecen y **cómo** se conectan — no «todo o nada».

## CapEx vs OpEx

| | On-premise (CapEx) | Cloud (OpEx) |
|---|-------------------|--------------|
| Pago | Hardware/licencias upfront | Uso mensual |
| Escalado | Lento (compra) | Rápido (API) |
| Riesgo financiero | Inmovilizado | Variable — puede dispararse sin gobierno |
| Previsibilidad presupuesto pública | Ciclo licitación | Factura cloud + transfer pricing interno |

En administración pública, OpEx cloud **facilita pilotos** pero exige **governance de coste** para no superar créditos aprobados.

## Elasticidad y time-to-market

Cloud brilla en:

- Picos analíticos (eventos, obras, huelgas, incidencias masivas).
- Pilotos IA de duración acotada.
- Entornos dev/test efímeros.

On-premise brilla en:

- Carga ** estable** predecible durante años con hardware amortizado.
- Sistemas con requisitos físicos de red aislada.

## Alta disponibilidad y DR

Cloud ofrece regiones, AZs y servicios DR managed — **si los diseñas** (no son automáticos).

On-premise requiere **segundo CPD**, replicación y pruebas de failover — costoso pero controlado.

Patrón híbrido DR: datos críticos replicados a cloud como **warm standby** — complejidad en networking y consistencia.

## Kubernetes: on-prem vs EKS vs AKS

| Aspecto | K8s on-prem | EKS / AKS |
|---------|-------------|-----------|
| Control plane | Tuyo | Managed (parcial) |
| Upgrades | Tu responsabilidad | AWS/Microsoft parchean control plane |
| GPU | Hardware local | Instancias cloud bajo demanda |
| Coste oculto | Personal platform | Personal + factura cloud |
| Portabilidad manifests | Alta | Alta entre K8s |

**Decisión:** K8s on-prem solo si hay **mandato** o skills; si no, managed reduce toil.

## IA on-prem vs IA cloud

| Factor | On-prem GPU | Cloud GenAI / GPU |
|--------|-------------|-------------------|
| Modelos fundacionales grandes | Impracticable sin inversión masiva | API managed (Bedrock, Azure OpenAI) |
| Entrenamiento custom ocasional | GPU local si ya existe | SageMaker / Azure ML jobs |
| Inferencia estable baja latencia | Edge/on-prem puede ganar | Endpoints regionales |
| Soberanía prompts/datos | Control total si modelo local pequeño | Contrato + región + VPC |
| Energía / CPD | Coste energético visible | Incluido en OpEx cloud |

> [!NOTE]
> Modelos fundacionales **privativos de gran escala** raramente se autoalojan en CPD típico enterprise — el mercado empuja a **API + RAG** sobre documentos propios.

## Patrones híbridos habituales

| Patrón | Descripción |
|--------|-------------|
| Datos on-prem + analítica cloud | Extracts anonimizados o agregados vía enlace dedicado |
| RAG híbrido | Documentos sensibles indexados on-prem; inferencia fundacional en cloud con VPC |
| Active Directory híbrido | Identidad corporativa sync — favorece Azure para apps M365 |
| Edge + cloud | Preproceso en vehículo/garaje; agregación central |
| Burst training | Entrenamiento puntual en cloud; artefacto vuelve on-prem |

## Conectividad dedicada

- **AWS Direct Connect** — enlace dedicado o partner a VPC.
- **Azure ExpressRoute** — equivalente Azure.

Reduce latencia, evita internet público, **no elimina** requisitos de cifrado y DLP.

## Trade-offs resumen

| Eje | On-prem / híbrido | Cloud nativo |
|-----|-------------------|--------------|
| Control | Alto | Compartido con proveedor |
| Agilidad | Baja | Alta |
| Coste inicial | Alto CapEx | Bajo inicio, OpEx variable |
| Skills | CPD + redes | Cloud + FinOps |
| IA GenAI cutting-edge | Difícil | Acceso rápido vía API |

## Resumen

- Híbrido es la norma en enterprise maduro — no excepción.
- GPU local vs cloud depende de **frecuencia**, **tamaño de modelo** y **legal**.
- Direct Connect / ExpressRoute son piezas de arquitectura, no detalle de red menor.
