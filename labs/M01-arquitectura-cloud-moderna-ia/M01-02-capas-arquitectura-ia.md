# M01-02 — Capas de la arquitectura IA

[← Página anterior](M01-01-plataformas-cloud-modernas.md) · [Siguiente página →](M01-03-tipos-carga-ia.md)

## Visión por capas

Una arquitectura IA enterprise se entiende mejor como **capas apiladas** que como un único «modelo». Cada capa tiene responsables, SLAs y tecnologías distintas. Cuando evalúas un diagrama de un proveedor, comprueba que **todas las capas están nombradas** — no solo la caja «IA».

```
┌─────────────────────────────────────────┐
│  Consumo (apps, APIs, dashboards, chat)  │
├─────────────────────────────────────────┤
│  Serving (inferencia, endpoints, batch) │
├─────────────────────────────────────────┤
│  Entrenamiento / experimentación (ML)   │
├─────────────────────────────────────────┤
│  Procesamiento (ETL, features, calidad) │
├─────────────────────────────────────────┤
│  Ingesta (batch, streaming, APIs, docs) │
├─────────────────────────────────────────┤
│  Gobierno · Seguridad · Observabilidad  │  ← transversal
└─────────────────────────────────────────┘
```

## Ingesta

**Función:** capturar datos desde sistemas origen hacia la plataforma analítica o de ML.

| Modo | Características | Ejemplos orientativos |
|------|-----------------|------------------------|
| Batch | Ficheros o extracts periódicos | Export nocturno de validaciones, informes CSV |
| Streaming | Eventos en tiempo casi real | Telemetría de flota, contadores de aforo |
| API / CDC | Cambios incrementales desde apps | Estado de vehículos, incidencias |
| Documentos | PDF, HTML, wikis | Procedimientos, normativa interna (base para RAG) |

**Preguntas de decisión:**

- ¿Con qué frecuencia debe estar disponible el dato para el modelo o el informe?
- ¿El dato incluye información personal o sensible (RGPD)?
- ¿Quién es dueño del dato y quién aprueba su uso en IA?

## Procesamiento

**Función:** limpiar, unir, enriquecer y preparar datos o **features** (variables) que alimentan modelos.

Componentes típicos:

- Catálogo de datos y metadatos (linaje).
- Jobs ETL/ELT o procesamiento distribuido.
- Validaciones de calidad (nulos, rangos, duplicados).
- Feature store (repositorio de features versionadas para entrenamiento e inferencia coherentes).

> [!NOTE]
> **Feature store:** evita que entrenamiento e inferencia usen definiciones distintas de la misma variable — una fuente frecuente de modelos que «funcionan en laboratorio» y fallan en producción.

## Entrenamiento

**Función:** aprender o actualizar modelos a partir de datos históricos.

- Suele ser **batch** y **episódico** (semanal, mensual), no 24/7.
- Requiere cómputo puntual (CPU o GPU).
- Debe quedar registrado: datos usados, hiperparámetros, métricas, artefacto del modelo.

En muchos casos enterprise **no entrenarás desde cero**: usarás modelos preentrenados o APIs fundacionales (especialmente en GenAI).

## Serving (inferencia)

**Función:** exponer el resultado del modelo a consumidores.

| Patrón | Latencia | Uso |
|--------|----------|-----|
| Online (API síncrona) | Milisegundos–segundos | Scoring en tiempo real, recomendaciones |
| Batch | Horas | Informes masivos, scoring nocturno |
| Embebido | Local / edge | Casos con conectividad limitada |

**Coste:** los endpoints **siempre encendidos** con GPU son de los mayores drivers de factura en ML cloud.

## Observabilidad

**Función:** saber si el sistema — y el modelo — se comportan como se espera.

- **Infraestructura:** CPU, memoria, errores, latencia de API.
- **Modelo:** drift (cambio en distribución de datos), degradación de métricas de negocio.
- **GenAI:** volumen de tokens, tasa de rechazo por políticas, feedback de usuarios.

Sin observabilidad, un modelo en producción puede degradarse **meses** antes de que el negocio lo detecte.

## Gobierno y seguridad

Capa **transversal** — atraviesa todas las anteriores:

| Área | Qué cubre |
|------|-----------|
| Identidad (IAM) | Quién puede leer datos, entrenar, desplegar |
| Red | Segmentación, acceso privado a endpoints |
| Cumplimiento | RGPD, retención, auditoría, sector público |
| Aprobaciones | Qué modelo pasa a producción y con qué evidencias |
| Seguridad de modelos | Prompt injection, filtrado de salida, datos en prompts |

> [!WARNING]
> **Confundir capas** es un anti-patrón frecuente: tratar un chat GenAI como «solo una app» sin ingesta documental, IAM y logging es asumir riesgos legales y operativos.

## Cómo cambia respecto al software tradicional

| Software tradicional | Plataforma IA moderna |
|---------------------|------------------------|
| Estado en base de datos | Estado + **versiones de modelos** y datasets |
| Release = código | Release = código **+ artefacto ML** + posible prompt/KB |
| Tests unitarios | Tests de código **+ validación de datos y métricas ML** |
| Monitorizar servicio up/down | Monitorizar **calidad de predicción** |

## Ejemplo integrado (transporte público)

Escenario ilustrativo — **no es un diseño aprobado**:

1. **Ingesta:** telemetría de buses vía streaming + export diario de validaciones.
2. **Procesamiento:** agregación por línea y franja; catálogo con clasificación de sensibilidad.
3. **Entrenamiento:** modelo semanal de predicción de demanda (batch en GPU puntual).
4. **Serving:** API interna para planificación + informe batch para dirección.
5. **Observabilidad:** alertas de drift en aforo; dashboard de error de predicción.
6. **Gobierno:** comité que aprueba uso de datos agregados; sin datos personales en el modelo de demanda.

Un segundo hilo **GenAI** podría añadir RAG sobre manuales de mantenimiento — con capa documental, identidad y políticas distintas.

## Resumen

- Toda arquitectura IA seria tiene ingesta, procesamiento, entrenamiento (o adopción de modelos), serving, observabilidad y gobierno.
- Cada capa responde preguntas distintas; un diagrama incompleto suele ocultar coste u operación.
- En enterprise crítico, gobierno y clasificación de datos van **antes** de escalar modelos.
