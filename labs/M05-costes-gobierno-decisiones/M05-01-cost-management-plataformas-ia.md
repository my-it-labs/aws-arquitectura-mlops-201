# M05-01 — Cost management en plataformas IA

[← Página anterior](README.md) · [Siguiente página →](M05-02-gobierno-ia-compliance.md)

## Por qué la IA dispara costes cloud

Los proyectos IA combinan:

- Compute **caro** (GPU).
- Almacenamiento **creciente** (lakes, embeddings).
- Servicios **siempre encendidos** (endpoints).
- Consumo **impredecible** (tokens GenAI).
- Observabilidad **verbose** (logs de prompts).

Sin FinOps, el POC «barato» se convierte en línea OpEx permanente.

## GPU y entrenamiento

| Driver | Control |
|--------|---------|
| Instancias GPU olvidadas | Apagado automático, tags, alarmas |
| Entrenamiento sin checkpointing | Re-run caro ante fallo |
| Dataset no optimizado (CSV gigante) | Parquet particionado en S3 |
| Overkill modelo (billions params) | Modelo más pequeño que cumpla SLA |

**Regla:** entrenamiento = **job finito**; no dejes GPU idle post-job.

## Inferencia

| Patrón | Coste típico | Optimización |
|--------|--------------|--------------|
| Endpoint GPU 24/7 | Alto fijo | Serverless inference, scale-to-zero, batch nocturno |
| Lambda + modelo ligero | Bajo volumen | Consolidar si tráfico constante |
| Bedrock tokens | Variable | Cache respuestas, modelos más baratos para tareas simples |
| Multi-AZ redundante | x2+ | Justificar SLA antes de duplicar |

Calcula **coste por predicción** o **por consulta GenAI** — unit economics.

## Storage

- S3 sin lifecycle → TB acumulados años.
- Embeddings duplicados por cada POC RAG.
- Checkpoints de entrenamiento sin TTL.

**Políticas:** mover a Glacier, borrar checkpoints > N días, deduplicar índices vectoriales.

## Transferencias (egress)

Salir de AWS hacia internet o cross-region **cuesta**:

- Diseña regiones donde están datos y usuarios.
- VPC endpoints para evitar NAT innecesario.
- No entrenar en región A con datos en región B sin motivo.

## Logs y observabilidad

Logging cada prompt/response completo en prod:

- Útil para debug limitado.
- Caro en CloudWatch.
- Riesgo RGPD.

**Alternativas:** sampling 1%, hash + metadata, retención 30 días.

## Costes ocultos (no en factura AWS)

| Oculto | Ejemplo |
|--------|---------|
| Personas | 2 FTE platform K8s |
| Licencias | Herramientas MLOps SaaS |
| Integración | Meses conectar legacy |
| Re-trabajo | POC sin path a prod |
| Incidents | Modelo malo → decisiones operativas erróneas |

TCO = cloud bill + **operación + riesgo**.

## FinOps para IA — prácticas

1. **Tagging obligatorio** antes de crear recurso.
2. **Presupuestos** por iniciativa con alertas 80/100%.
3. **Review mensual** top servicios + anomalías Bedrock.
4. **Sandbox accounts** con límites; prod separado.
5. **Apagar entornos** dev viernes noche.
6. **Showback** a dirección de cada unidad consumidora.

## Presupuesto público / enterprise

En organismos con techo anual:

- Negociar **commitment** o créditos POC por escrito.
- Piloto con **tope tokens** y usuarios allowlist.
- Informe trimestral coste vs valor (KPI negocio).

## Resumen

- GPU idle y endpoints 24/7 son los clásicos en ML; tokens sin límite en GenAI.
- Egress y logs sorprenden a escala — diseña desde día 1.
- FinOps es requisito de arquitectura, no departamento aparte opcional.
