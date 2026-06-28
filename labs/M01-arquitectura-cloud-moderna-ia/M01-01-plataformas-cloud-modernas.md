# M01-01 — Plataformas cloud modernas

[← Página anterior](README.md) · [Siguiente página →](M01-02-capas-arquitectura-ia.md)

## De la infraestructura fija a la plataforma elástica

Durante décadas, la mayoría de organizaciones IT construyeron software sobre **infraestructura propia o externalizada pero estática**: servidores con capacidad fija, ciclos de compra de hardware, despliegues manuales y equipos silo (desarrollo, operaciones, redes, base de datos). Ese modelo funcionaba cuando la demanda era predecible y los releases eran pocos al año.

Las **plataformas cloud modernas** invierten parte de esa lógica:

| Aspecto | Modelo tradicional | Modelo cloud moderno |
|---------|-------------------|----------------------|
| Capacidad | Provisionada por adelantado (CapEx) | Elasticidad bajo demanda (OpEx) |
| Despliegue | Ventanas largas, riesgo alto | Releases frecuentes, automatizados |
| Escalado | Comprar más hardware | Añadir instancias o servicios gestionados |
| Responsabilidad | Todo «bajo tu techo» | Modelo de responsabilidad compartida con el proveedor |
| Time-to-market | Lento (aprovisionamiento) | Más rápido (APIs y servicios listos) |

> [!NOTE]
> **Responsabilidad compartida:** el proveedor cloud opera la infraestructura global; **tú** sigues siendo responsable de la configuración, los datos, las identidades y el uso seguro de los servicios. En sector público y transporte, esto implica contratos, auditoría y clasificación de datos.

En un operador de transporte metropolitano, la elasticidad importa en picos de demanda de información (incidencias, eventos, horas punta), en procesos analíticos estacionales y en pilotos de IA que no justifican comprar GPU para todo el año.

## Qué problemas resuelve la nube moderna

1. **Escalabilidad sin ciclo hardware:** subir capacidad en horas, no en meses de licitación.
2. **Servicios gestionados:** bases de datos, colas, ML o GenAI sin montar y parchear todo el stack.
3. **Separación por APIs:** integrar sistemas legacy con componentes nuevos sin acoplar despliegues.
4. **Observabilidad y automatización** como ciudadanos de primera clase.
5. **Experimentación controlada:** entornos aislados para probar IA o analítica con coste acotado.

La nube **no elimina** la complejidad: la **desplaza** hacia decisiones de arquitectura, gobierno, coste y operación de servicios.

## DevOps, Platform Engineering y automatización

### DevOps

**DevOps** es la convergencia entre desarrollo y operaciones mediante cultura, procesos y **automatización** (CI/CD, infraestructura como código, pruebas automatizadas). Objetivo: entregar software de forma más frecuente y fiable.

En la práctica enterprise suele manifestarse como:

- Pipelines que construyen, prueban y despliegan artefactos.
- Entornos dev / pre / prod claramente separados.
- Rollback y trazabilidad de cambios.

### Platform Engineering

**Platform Engineering** va un paso más allá: un equipo (o plataforma interna) ofrece a los equipos de producto **«golden paths»** — formas estandarizadas de desplegar, observar y consumir datos o modelos — para que no cada proyecto reinvente Kubernetes, logging o seguridad.

| DevOps | Platform Engineering |
|--------|---------------------|
| Enfoque en el pipeline de una aplicación | Enfoque en la **plataforma** que muchas aplicaciones comparten |
| «¿Cómo despliego este servicio?» | «¿Qué capacidades comunes ofrecemos a todos los equipos?» |
| Herramientas: Jenkins, GitLab CI, etc. | Catálogo interno, templates, abstracciones (IDP) |

Para perfiles que **deciden** arquitecturas, la pregunta relevante es: *¿nuestro integrador nos vende un producto aislado o encaja en una plataforma que ya tenemos o queremos construir?*

### Automatización de pipelines de datos e IA

Los pipelines de **datos** mueven información desde fuentes operacionales hacia zonas de analítica o entrenamiento. Los pipelines de **IA** añaden entrenamiento, validación, registro y despliegue de modelos.

**DataOps** aplica ideas DevOps al dato: versionado, pruebas de calidad, linaje, despliegues repetibles de transformaciones. Sin DataOps, los proyectos ML suelen fracasar **antes** del algoritmo — por datos incorrectos, obsoletos o sin gobierno.

## Riesgos habituales al modernizar

| Riesgo | Descripción | Señal de alerta |
|--------|-------------|-----------------|
| Lift-and-shift ingenuo | Migrar VMs sin rediseñar | Mismo coste o mayor, poca elasticidad |
| Shadow IT cloud | Equipos contratan SaaS sin gobierno | Facturas dispersas, datos fuera de política |
| Sobrecarga de servicios | Cada proyecto elige stack distinto | Fragmentación, skills imposibles de mantener |
| IA sin fundamento de datos | GenAI antes de catalogar fuentes | POC impresionante, producción inviable |
| Subestimar operación | «Es managed» = «no hay que operar» | Configuración, IAM, costes y DR siguen siendo tuyos |

> [!WARNING]
> **En entornos críticos** (explotación de red de transporte, seguridad, datos personales de empleados o clientes), la velocidad cloud debe ir pareja con **clasificación de datos**, continuidad de negocio y trazabilidad. La nube acelera; no sustituye el análisis de riesgo.

## Relación con AWS como ecosistema de referencia

Este curso toma **AWS** como lente principal porque ofrece **amplitud de servicios** y patrones maduros en datos, ML y GenAI. Las ideas de capas y gobierno, sin embargo, son **transferibles** a Azure u otras nubes: cambian nombres y matices operativos, no la necesidad de diseño.

En los siguientes capítulos verás **cómo se organiza una arquitectura IA concreta** sobre esas ideas de plataforma moderna.

## Resumen

- Cloud moderna = elasticidad, servicios gestionados, APIs y automatización — con gobierno explícito.
- DevOps acorta el camino del código a producción; Platform Engineering estandariza capacidades comunes.
- DataOps y pipelines de IA requieren calidad y linaje de datos; la IA generativa no los sustituye.
- Modernizar mal (lift-and-shift, fragmentación, IA prematura) genera coste y riesgo operativo.
