# M05-03 — Evaluar propuestas de proveedores

[← Página anterior](M05-02-gobierno-ia-compliance.md) · [Siguiente página →](../../README.md)

## Tu rol en la evaluación

Como arquitecto o responsable IT **no firmas** cada línea técnica — pero sí detectas **huecos**, **sobredimensionamiento** y **riesgos** antes de comprometer presupuesto.

## Preguntas obligatorias al proveedor cloud/integrador

### Datos y soberanía

1. ¿En qué **región** se procesan datos, prompts y logs?
2. ¿Se usan mis datos para **entrenar** modelos del proveedor?
3. ¿Cómo se **cifran** reposo y tránsito?
4. ¿Compatible con clasificación **confidencial/restringida** interna?

### Identidad y acceso

5. ¿Integración **SSO** (Entra, SAML, OIDC)?
6. ¿IAM least privilege documentado por rol?
7. ¿Acceso break-glass auditado?

### Arquitectura y operación

8. Diagrama con **todas las capas** (ingesta → gobierno) — no solo «IA».
9. ¿Quién opera **day-2** (monitorización, parches, coste)?
10. ¿SLA latencia/disponibilidad inferencia?
11. Plan **rollback** modelo/KB.

### Coste

12. Modelo precio: tokens, GPU hora, storage, egress.
13. Estimación **TCO 36 meses** con supuestos tráfico.
14. ¿Qué pasa si tráfico x10?

### Salida y lock-in

15. ¿Formatos abiertos exportables (Parquet, ONNX, MLflow)?
16. Plan **exit** a 90 días — pasos concretos.
17. Dependencias propietarias listadas.

### GenAI específico

18. ¿RAG o fine-tuning? ¿Por qué?
19. Guardrails y evaluación **groundedness**?
20. Política retención logs prompts.

## Validar arquitecturas — checklist visual

Al revisar diagrama, marca ✅ / ❌:

| Elemento | Presente |
|----------|----------|
| Fuente dato identificada | |
| Clasificación sensibilidad | |
| Catálogo / linaje | |
| Separación dev/prod cuentas | |
| Auth en API inferencia | |
| Observabilidad + alertas | |
| Cost tagging | |
| DR / backup | |
| Human review donde crítico | |

Diagrama «solo cajas mágicas IA» sin IAM ni red → **red flag**.

## Métricas a exigir antes de prod

| Tipo | Ejemplos |
|------|----------|
| ML clásico | Precision/recall, drift, latency p99 |
| GenAI | Token cost/user, citation rate, thumbs down rate |
| Negocio | Tiempo ahorrado, reducción llamadas, NO solo «wow demo» |
| Operación | MTTR, error budget |

Exige **golden set** evaluación antes de go-live GenAI interno.

## Anti-patrones en propuestas comerciales

| Propuesta | Realidad |
|-----------|----------|
| «Plataforma IA lista en 2 semanas» | Solo UI demo |
| «Kubernetes porque es futuro» | Sin SRE |
| «GPT para todo» | Sin caso uso medido |
| «100% automatizado» | Sin humano en loop |
| «Sin lock-in» | APIs vendor-specific |
| «On-prem GPU barato» | Sin coste energía/ops |

## Plantilla de decisión (simplificada)

| Criterio | Peso | Opción A | Opción B |
|----------|------|----------|----------|
| Encaje identidad | 20% | | |
| TCO 3 años | 20% | | |
| Time-to-market | 15% | | |
| Riesgo compliance | 20% | | |
| Operabilidad interna | 15% | | |
| Exit / portabilidad | 10% | | |

Puntuación cualitativa 1-5 — documenta **assumptions**.

## Cierre integrador — visión de plataforma

Un diseño maduro de **plataforma IA moderna** (referencia genérica) incluye:

```
[ Fuentes operacionales + documentales ]
           ↓
[ Ingesta + lago gobernado + catálogo ]
           ↓
[ Pipelines ML batch ] ──► [ Registry ]
           ↓                      ↓
[ APIs inferencia ]    [ GenAI RAG acotado ]
           ↓                      ↓
[ Observabilidad · IAM · FinOps · Comité ML ]
```

Adapta cajas a contratos y datos reales — la plantilla sirve para **completar huecos** en propuestas externas.

## Resumen del curso

Has recorrido:

1. **Capas** arquitectura cloud/IA moderna.
2. **Servicios AWS** y cuándo usarlos.
3. **Comparativa Azure** y híbrido.
4. **Patrones** enterprise.
5. **Coste, gobierno y evaluación** — donde se gana o pierde un proyecto real.

La competencia clave no es desplegar un endpoint — es **preguntar bien**, **leer diagramas** y **decidir con trazabilidad**.
