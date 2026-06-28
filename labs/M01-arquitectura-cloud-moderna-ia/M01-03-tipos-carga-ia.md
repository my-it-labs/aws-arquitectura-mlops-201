# M01-03 — Tipos de carga IA

[← Página anterior](M01-02-capas-arquitectura-ia.md) · [Siguiente página →](M01-04-ia-generativa-fundamentos.md)

## Por qué importa el «tipo de carga»

Cuando un proveedor propone «IA», la arquitectura, el coste y el gobierno cambian radicalmente según **cómo** se consume el modelo. Clasificar la carga te permite:

- Detectar sobredimensionamiento (p. ej. cluster GPU para un informe mensual).
- Exigir SLAs realistas (latencia, disponibilidad).
- Alinear compliance (datos personales, explicabilidad).

## Batch ML (entrenamiento e inferencia por lotes)

**Entrenamiento batch:** se procesa un volumen grande de datos de una vez; el job termina y libera recursos.

**Inferencia batch:** se aplica un modelo a millones de filas (p. ej. cada noche); resultado en tabla o fichero.

| Ventaja | Limitación |
|---------|------------|
| Coste predecible (ventanas acotadas) | No sirve para interacción inmediata |
| Escala con jobs paralelos | Datos «de ayer» — lag aceptable |
| Encaja con OpEx elastic (GPU bajo demanda) | Requiere orquestación de pipelines |

**Casos típicos:** predicción de demanda por línea, mantenimiento predictivo con revisión diaria, scoring de riesgo operativo.

## Inferencia en tiempo real (online)

El cliente (app, API gateway, sistema SCADA analítico) envía una petición y espera respuesta en **segundos o menos**.

Requisitos arquitectónicos:

- Endpoint o función siempre disponible (o autoscaling agresivo).
- Modelo optimizado (tamaño, cuantización) si la latencia es crítica.
- Observabilidad de latencia p95/p99 y tasa de error.

**Coste:** instancias reservadas o endpoints managed 24/7 — revisar si el negocio justifica continuidad.

## RAG (Retrieval-Augmented Generation)

Patrón híbrido muy extendido en enterprise:

1. **Recuperar** fragmentos relevantes de una base documental (normativa, manuales, FAQs).
2. **Inyectar** ese contexto en un modelo generativo.
3. **Generar** respuesta fundamentada en esos fragmentos (no solo en memoria del modelo).

Componentes:

- Almacén documental (objeto o SharePoint-like).
- Pipeline de chunking y embeddings.
- Base vectorial o índice de búsqueda híbrida.
- Modelo fundacional (API managed).
- Capa de políticas (qué documentos entran, qué usuarios preguntan).

| Ventaja | Riesgo |
|---------|--------|
| Respuestas ancladas a corpus interno | Calidad depende del corpus y del chunking |
| Menos fine-tuning inicial | Documentos obsoletos → respuestas incorrectas |
| Encaja en asistentes internos | Fuga si el corpus incluye datos sensibles mal filtrados |

**Ejemplo orientativo:** asistente para consultar procedimientos de explotación o mantenimiento — **no** sustituye sistemas de misión crítica sin validación humana.

## Fine-tuning

Ajustar un modelo preentrenado con **datos propios** para especializar comportamiento o vocabulario.

- Útil cuando el dominio es muy específico y RAG no basta.
- Requiere dataset curado, evaluación, gobierno de versiones.
- Coste de entrenamiento + **mantenimiento** del modelo custom.

En sector público suele ser **segunda fase**, no el primer entregable — salvo casos muy acotados.

## Agentes

Un **agente** orquesta varios pasos: consultar APIs, buscar documentos, ejecutar acciones, razonar en cadena.

| Potencial | Riesgo |
|-----------|--------|
| Automatizar flujos multi-sistema | Efectos en cadena si falla un paso |
| Experiencias «asistente» ricas | Superficie de ataque (prompt injection, acciones no autorizadas) |

> [!WARNING]
> En operación crítica (seguridad, control de tráfico, sistemas de pago), los agentes autónomos exigen **límites estrictos** (allowlist de acciones, human-in-the-loop, entornos aislados). Evalúalos como **software de misión secundaria** hasta madurez de gobierno.

## Sistemas híbridos

Combinaciones frecuentes en enterprise:

| Patrón | Descripción |
|--------|-------------|
| Datos on-prem + inferencia cloud | Dato sensible local; modelo en región contratada |
| Batch + online | Entrenamiento batch; API online con modelo versionado |
| ML clásico + GenAI | Scoring tradicional + capa de lenguaje para explicar resultados |
| Edge + cloud | Preproceso en vehículo o garaje; agregación central |

La decisión híbrida suele venir de **legal/compliance**, no solo de tecnología.

## Matriz de decisión rápida

| Necesidad | Patrón probable |
|-----------|-----------------|
| Informe diario / semanal | Batch inference |
| Alerta en segundos | Online inference |
| Preguntas sobre documentación interna | RAG |
| Lenguaje muy específico del dominio | Fine-tuning (tras RAG) |
| Automatizar proceso multi-paso | Agente (con gobierno fuerte) |
| Datos no pueden salir del CPD | Híbrido o on-prem selectivo |

## Resumen

- El tipo de carga determina compute, coste, SLA y gobierno.
- Batch optimiza coste; online optimiza latencia; RAG es el patrón GenAI enterprise más habitual al inicio.
- Agentes y fine-tuning añaden capacidad y **complejidad** — no son el primer paso por defecto.
