# M04-05 — Casos de uso de agentes (contexto TMB)

[← Página anterior](M04-04-enterprise-seguridad-modelos.md) · [Siguiente página →](../M05-costes-gobierno-decisiones/README.md)

> **Documento para validación.** Describe casos de uso de **agentes de IA** aplicados al contexto de **Transportes Metropolitanos de Barcelona (TMB)**.  
> Revisa con tu equipo qué encaja con la realidad operativa, qué datos existen hoy y qué sería roadmap.

Relacionado con [M04-02 — Patrones de IA y serving](M04-02-patrones-ia-serving.md) (RAG, agentes, human-in-the-loop) y [M02-04 — Bedrock y GenAI](M02-04-bedrock-genai-aws.md).

---

## Qué es un agente (en una frase)

Un **agente** es un asistente basado en un modelo de lenguaje que **razona en pasos** y puede **usar herramientas** (consultar APIs, buscar documentos, ejecutar acciones **permitidas**) para ayudar a una persona a decidir o actuar más rápido.

```text
Operador / analista  →  Agente (LLM + políticas + auditoría)
                              │
                    ┌─────────┼─────────┐
                    ▼         ▼         ▼
              Documentación   Datos vivos   Maestros /
              procedimientos  (streams)     histórico / SAP
```

No confundir:

| Enfoque | Qué hace | Ejemplo TMB |
|---------|----------|-------------|
| **BI (Power BI)** | Dashboards, KPIs | Ocupación por línea, costes |
| **ML clásico** | Predicción numérica | Afluencia, scoring de avería |
| **RAG** | Responde con documentos indexados | «¿Qué dice el procedimiento X?» |
| **Agente** | Cruza fuentes y **orquesta** consultas/acciones acotadas | Copiloto CCO en incidencia |

---

## Mapa de datos en explotación de metro

En una operadora como TMB conviven **contextos de dato distintos**. Un agente útil sabe **de dónde puede leer** y **qué no puede tocar**.

| Tipo | Ejemplos | Latencia típica | Uso en agente |
|------|----------|-----------------|---------------|
| **Datos vivos (streaming)** | Posición de tren, alarmas CCO, ocupación, sensores energía/ventilación | Segundos | Situación actual, resumen en lenguaje natural |
| **Eventos / incidencias** | Retraso, avería convoy, corte parcial de línea, restricción de velocidad | Minutos | Contexto del incidente, similitud con histórico |
| **Patrones históricos** | Recurrencia de incidencias por tramo, estacionalidad, correlaciones | Batch / analítica | «Qué pasó otras veces en situaciones parecidas» |
| **Topología / circuitos** | Líneas, tramos, bloques, correspondencias, restricciones de circulación | Referencia (casi estática) | Explicar **dónde** ocurre y qué activos afecta |
| **Activos de explotación** | Material rodante, agujas, escaleras, ventilación, bombas | Maestro + telemetría | Mantenimiento, criticidad, historial de OT |
| **Datos corporativos (p. ej. SAP)** | Activos, órdenes de trabajo, repuestos, compras | Batch / transaccional | Priorización mantenimiento, trazabilidad activo |
| **Documentación** | Procedimientos CCO, planes de contingencia, manuales técnicos | Actualización lenta | RAG — respuestas con **cita** al documento vigente |
| **Datos de demanda** | Validaciones, afluencia, eventos ciudad (conciertos, partidos) | Horas / días | Impacto en servicio, briefing operativo |

> **Validación:** marca qué filas tenéis hoy con pipeline automatizado, cuáles en Excel/SharePoint, cuáles solo en SAP o en apps internas.

---

## Escenario arquitectónico (orientativo)

Muchas organizaciones de transporte combinan **ecosistema Microsoft** (documentos, BI) con **cargas en nube** (lago objeto, ML). Un agente enterprise no sustituye esos sistemas: **se conecta** con permisos estrictos.

```text
                    ┌─────────────────────────────────┐
                    │  Agente (copiloto, con límites) │
                    └───────────────┬─────────────────┘
                                    │
         ┌──────────────────────────┼──────────────────────────┐
         ▼                          ▼                          ▼
   Base de conocimiento         Tools (APIs)              Modelos ML
   procedimientos               allowlist /                (batch /
   SharePoint, S3…              solo lectura               scoring)
         │                          │                          │
         ▼                          ▼                          ▼
   Contingencia,              Incidencias,                 Patrones
   manuales                   streams tiempo real          históricos
                              SAP activos (RO)
```

Referencia AWS del curso: [Agents for Amazon Bedrock](../M02-servicios-aws-plataformas/M02-04-bedrock-genai-aws.md), Knowledge Bases, SageMaker para ML batch.

---

## Caso 1 — Copiloto de incidencias (sala de control / CCO)

### Problema

Durante una incidencia el operador recibe **alarmas, radios, sistemas y documentación** a la vez. Hay que decidir rápido siguiendo **procedimientos** que pueden ser extensos.

### Datos que cruza

- **Tiempo real:** estado de líneas, trenes retenidos, alarmas.
- **Eventos:** incidencia abierta, severidad, tramo afectado.
- **Histórico:** incidencias similares (misma línea, mismo tipo de avería).
- **Documentos:** procedimiento de contingencia vigente.

### Qué haría el agente (propuesta)

1. Resume la situación en lenguaje claro (castellano / catalán según política).
2. Señala pasos relevantes del procedimiento **con referencia al documento**.
3. Lista incidencias parecidas y acciones que funcionaron en el pasado.
4. **No** acciona señalización ni mandos OT; el operador valida cada paso.

### Patrón

RAG + agente con **tools de solo lectura** (consulta incidencias, topología, KB documental).

> **Validación:** ¿Qué sistemas usa hoy el CCO? ¿Dónde están los procedimientos oficiales? ¿Hay API de incidencias consultable?

---

## Caso 2 — Asistente de mantenimiento (activos + SAP + telemetría)

### Problema

Miles de **activos** (convoys, agujas, escaleras mecánicas…): historial en **SAP PM**, sensores en otro silo, priorización a mano o con modelos aparte.

### Datos que cruza

- **SAP:** orden de trabajo, historial averías, repuestos, criticidad del activo.
- **Telemetría / batch:** vibración, temperatura, consumo energético.
- **Modelos ML:** scoring nocturno de riesgo (si existe pipeline hacia lago o almacén objeto).

### Qué haría el agente (propuesta)

- Explicar **por qué** un activo subió de prioridad («score del modelo + 3 OT similares en 6 meses + manual sección 4.2»).
- Proponer **borrador** de orden de trabajo para revisión humana.
- Responder preguntas en lenguaje natural («¿qué convoys comparten componente X?») vía consultas **preaprobadas**.

### Patrón

**ML predice**; el **agente explica y orquesta** consultas — no sustituye al modelo ni a SAP.

> **Validación:** ¿SAP es maestro de activos? ¿Hay telemetría integrada con mantenimiento predictivo?

---

## Caso 3 — Explotación por circuito / tramo

### Problema

Un **tramo** o **circuito de circulación** concentra incidencias; hace falta visión **espacial** (dónde, qué activos, qué historial).

### Datos que cruza

- Topología: estaciones, bloques, restricciones temporales (obras).
- Incidencias geolocalizadas por tramo (90 días, 1 año).
- Eventos externos que afectan demanda (eventos masivos en Barcelona).

### Qué haría el agente (propuesta)

- Briefing narrativo antes de un turno o simulacro.
- «Mapa de riesgo» cualitativo del tramo (activos críticos + recurrencia).
- Cruce patrón horario («viernes tarde + evento en zona Z») con datos de afluencia.

### Patrón

Consultas analíticas + RAG; acciones limitadas a informes internos.

> **Validación:** ¿Tenéis GIS / topología unificada? ¿Cómo se nombra internamente «circuito» vs «tramo» vs «bloque»?

---

## Caso 4 — Analista / dirección (BI + lago + SAP)

### Problema

Preguntas ad hoc que requieren cruzar **Power BI**, ficheros en almacén objeto y extracts de **SAP** — lento sin un catálogo de datos claro.

### Datos que cruza

- Agregados en **Power BI** (SaaS).
- Datasets curados en almacén (p. ej. S3 u OneLake en roadmap).
- SAP (costes, compras, indicadores operativos **sin datos personales** sin control).

### Qué haría el agente (propuesta)

- Responder con **consultas preaprobadas** (no SQL libre generado sin revisión).
- Generar narrativa para informe con **enlace al dashboard** correspondiente.
- Rol **solo lectura**; filtros RGPD obligatorios.

### Patrón

Agente como capa de **lenguaje natural** sobre APIs acotadas — evolución natural hacia plataforma unificada (p. ej. Microsoft Fabric).

> **Validación:** ¿De dónde se alimenta hoy Power BI? ¿Hay catálogo de datasets?

---

## Caso 5 — Comunicación al pasajero (interno, con supervisión)

### Problema

Coordinar mensajes **consistentes** en app, web y redes cuando hay incidencia.

### Datos que cruza

- Retrasos y estaciones afectadas (tiempo real).
- Alternativas de ruta (planner multimodal, APIs de servicio).

### Qué haría el agente (propuesta)

- Borrador multicanal **consistente** (misma causa, mismo ETA).
- Variantes CA / ES / EN.
- **Publicación solo tras aprobación humana** — nunca autónoma en producción.

### Patrón

Generación + tools lectura; **human-in-the-loop** obligatorio.

> **Validación:** ¿Quién aprueba hoy los mensajes públicos? ¿Hay plantillas oficiales?

---

## Matriz: ¿agente, ML o BI?

| Necesidad | Enfoque recomendado | Ejemplo TMB |
|-----------|---------------------|-------------|
| KPIs y reporting estable | **Power BI** | Costes, indicadores por línea |
| Predicción numérica repetible | **ML clásico** (SageMaker u otro) | Afluencia, probabilidad de avería |
| Buscar en procedimientos | **RAG** | Contingencia, manuales |
| Cruzar varias fuentes en una conversación | **Agente** | Copiloto CCO, mantenimiento |
| Mando de señales / seguridad ferroviaria | **Sistemas OT certificados** | **No agente GenAI autónomo** |

---

## Líneas rojas (gobierno y compliance)

1. **Seguridad ferroviaria:** el agente es **informativo**; no sustituye interlocking, ATP ni procedimientos certificados OT.
2. **RGPD:** datos de empleados o usuarios — minimización, filtros, auditoría de prompts (ver [M05-02](../M05-costes-gobierno-decisiones/M05-02-gobierno-ia-compliance.md)).
3. **Una sola verdad:** si BI y lago discrepan, el agente **amplifica** el error — hace falta gobierno del dato antes de escalar GenAI.
4. **Documentación obsoleta:** procedimientos en SharePoint sin revisión → respuestas peligrosas; gobierno documental primero.
5. **Coste en pico de crisis:** muchas invocaciones durante incidencia masiva — cuotas, modelos más baratos para tareas simples ([M05-01](../M05-costes-gobierno-decisiones/M05-01-cost-management-plataformas-ia.md)).

---

## Checklist de validación (para el equipo TMB)

Marca **Sí / No / Parcial** en sesión o async:

| # | Pregunta |
|---|----------|
| 1 | ¿Existen APIs o feeds de **incidencias** consultables (no solo pantalla)? |
| 2 | ¿Hay **streams** de explotación accesibles para analítica (posición, alarmas)? |
| 3 | ¿**SAP** es el maestro de activos y OT de mantenimiento? |
| 4 | ¿Los **procedimientos CCO** están digitalizados y versionados? |
| 5 | ¿Hay **modelos ML** en producción (afluencia, mantenimiento)? |
| 6 | ¿**Power BI** es la vía principal de reporting ejecutivo? |
| 7 | ¿Qué casos 1–5 priorizaríais en un piloto de 6 meses? |
| 8 | ¿Qué acciones nunca delegaríais a un agente? |

---

## Resumen

- En explotación de metro hay **datos vivos**, **eventos**, **histórico**, **SAP** y **documentación** — un agente aporta valor **cruzando** fuentes con permisos, no sustituyendo el CCO ni SAP.
- El patrón enterprise es **copiloto + allowlist + human-in-the-loop**, alineado con [M04-02](M04-02-patrones-ia-serving.md).
- Este capítulo es **borrador para validar** con TMB: corregid datos, sistemas y prioridades; el material del curso se ajustará según vuestro feedback.
