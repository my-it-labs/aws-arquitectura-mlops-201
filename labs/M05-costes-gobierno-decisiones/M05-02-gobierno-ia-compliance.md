# M05-02 — Gobierno de IA y compliance

[← Página anterior](M05-01-cost-management-plataformas-ia.md) · [Siguiente página →](M05-03-evaluar-propuestas-proveedores.md)

## Model governance

**Model governance** define **quién puede hacer qué** con modelos y datos en cada fase del ciclo de vida.

| Fase | Control típico |
|------|----------------|
| Experimentación | Solo datos anonimizados / sandbox |
| Staging | Métricas mínimas + revisión peer |
| Producción | Comité aprobación + rollback plan |
| Retirada | Deprecate endpoint, archive artefacto |

Artefactos gobernados:

- Pesos modelo / contenedor inferencia.
- Dataset snapshot ID.
- Prompt templates / KB version.
- Evaluación (golden set results).

## Model Registry como fuente de verdad

Tanto SageMaker Registry como MLflow Registry deben registrar:

- Versión semántica.
- Autor y fecha.
- Métricas offline.
- Aprobador y ticket change.

**Sin registry** no hay rollback forense tras incidente.

## Auditoría y trazabilidad

Eventos auditables:

- Despliegue modelo v1.3 → prod.
- Cambio índice RAG (500 docs nuevos).
- Elevación permisos IAM Bedrock.
- Acceso dataset restringido.

Integrar CloudTrail / Azure Activity Log con SIEM corporativo.

## Explicabilidad

Regulaciones o políticas internas pueden exigir **explicar** decisiones automatizadas (p. ej. scoring que afecta personal).

| ML clásico | GenAI |
|------------|-------|
| SHAP, feature importance | Citas RAG, chain-of-thought interno (no mostrar al usuario) |
| Reglas + modelo híbrido | Limitar a tareas no críticas sin humano |

En transporte público, tratar explicabilidad según **impacto en personas**.

## RGPD y datos personales

Principios aplicables:

- **Minimización** — no indexar en RAG lo que no hace falta.
- **Base legal** — legitimación tratamiento.
- **DPIA** — evaluación impacto para IA nueva.
- **Derechos** — acceso/borrado complica logs con PII.

GenAI con empleados: política uso aceptable, prohibición pegar datos clientes en chat.

## AI Act y marco regulatorio (EU)

Evolución normativa europea clasifica sistemas IA por riesgo.

**Acción arquitectónica:** clasificar casos uso (asistente interno docs vs decisión automática seguridad) y documentar **nivel de riesgo** antes de escalar.

Consultar siempre legal — tecnología sola no cumple norma.

## Riesgos habituales de gobierno

| Riesgo | Señal |
|--------|-------|
| GenAI sin política | Chat libre con docs confidenciales |
| Shadow POC | Tarjeta corporativa AWS personal |
| Model drift ignorado | Métricas offline OK, negocio queja |
| Vendor lock-in sin exit | Contrato 3 años API propietaria |
| Kubernetes sobredimensionado | Cluster «por si acaso» |

## Comité ML / IA — agenda mínima

Reunión trimestral (ejemplo):

1. Inventario modelos/KB en prod.
2. Incidentes y near-misses.
3. Coste vs presupuesto.
4. Nuevas solicitudes negocio — priorización.
5. Actualización normativa.

## Resumen

- Gobierno = registry + aprobaciones + auditoría + clasificación riesgo.
- RGPD y AI Act afectan **qué datos** entran en RAG y **qué decisiones** automatizas.
- Comité ML evita POCs paralelos sin estándar.
