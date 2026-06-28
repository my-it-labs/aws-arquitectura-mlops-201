# M03-03 — Estrategia AWS vs Azure

[← Página anterior](M03-02-on-prem-cloud-hibrido.md) · [Siguiente página →](M03-04-casos-eleccion-contexto.md)

## Dos filosofías de plataforma

### AWS — «Amplio catálogo, componibilidad»

AWS prioriza **máximo catálogo de servicios** y composición fina. Muchas decisiones arquitectónicas recaen en el cliente (servicio X vs Y).

**Fortalezas estratégicas:**

- Madurez global y regiones extensas.
- Liderazgo en infra cloud «genérica» y K8s (EKS).
- Ecosistema partners e ISVs enorme.
- Referencia en data lake S3 + analítica + SageMaker + Bedrock.

**Debilidades estratégicas:**

- Curva de aprendizaje — demasiadas opciones.
- Gobierno multi-cuenta necesario pronto (Control Tower).
- Cost management exigente (FinOps no opcional).
- Experiencia identidad «Microsoft» menos nativa.

### Azure — «Enterprise Microsoft-first»

Azure enfatiza **integración con stack Microsoft** (Entra ID, M365, Dynamics, Windows, GitHub en grupo MS).

**Fortalezas estratégicas:**

- SSO y gobierno identidad para orgs AD-centric.
- Contratos enterprise Microsoft existentes (EA).
- Azure OpenAI acoplado a políticas Microsoft.
- Percepción favorable en sector público EU en algunos marcos.

**Debilidades estratégicas:**

- Algunos servicios percibidos menos maduros que AWS equivalente (varía por área).
- Riesgo de **vendor stack** Microsoft completo.
- Complejidad de precios y SKUs.

## Comparativa estratégica resumida

| Dimensión | AWS | Azure |
|-----------|-----|-------|
| Amplitud servicios | ★★★★★ | ★★★★ |
| Integración AD/M365 | ★★★ | ★★★★★ |
| ML/GenAI managed | SageMaker + Bedrock fuerte | Azure ML + Azure OpenAI fuerte |
| FinOps / coste transparente | Difícil — requiere disciplina | Difícil — requiere disciplina |
| Portabilidad K8s | EKS referencia | AKS muy sólido |
| Sector público EU | Fuerte presencia | Fuerte presencia |

(Las estrellas son orientativas — validar en tu contrato y región.)

## Multicloud — cuándo sí y cuándo no

**Multicloud real** (misma carga en dos nubes) **duplica** operación: dos IAM, dos redes, dos FinOps.

**Multicloud sensato:**

- **Best-of-breed puntual** (p. ej. Azure OpenAI + data lake AWS) con integración explícita.
- **DR** en segunda nube (costoso).
- **Adquisiciones** — consolidación gradual.

**Multicloud anti-patrón:**

- «Evitar lock-in» sin portabilidad real de datos ni equipos duplicados.
- Cada equipo elige nube distinta sin platform governance.

## Lock-in — perspectiva realista

Todo diseño tiene lock-in **en alguna capa**:

| Capa | Lock-in |
|------|---------|
| Fundacional API (Bedrock vs Azure OpenAI) | Alto en prompts/tools |
| Data lake format (Parquet open) | Bajo |
| Orquestación K8s | Medio-bajo |
| SageMaker Pipelines / Azure ML jobs | Medio-alto |
| Identidad Entra | Alto en org MS |

**Mitigación:** abstracción en apps, formatos abiertos, OTel, contratos exit, no negar dependencia.

## FinOps como disciplina común

Independiente de nube:

- Tagging, chargeback/showback.
- Presupuestos por proyecto.
- Revisión mensual de top 10 servicios.
- Política de apagado dev/test.
- Unit economics IA (coste por predicción / por consulta GenAI).

## Resumen

- AWS gana amplitud y composibilidad; Azure gana encaje Microsoft e identidad.
- Multicloud debe estar **justificado** por caso de negocio, no por miedo abstracto.
- Lock-in se gestiona — no se evita mágicamente cambiando logo en el slide.
