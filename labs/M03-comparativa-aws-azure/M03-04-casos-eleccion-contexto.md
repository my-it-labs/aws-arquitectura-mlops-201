# M03-04 — Casos de elección por contexto

[← Página anterior](M03-03-estrategia-aws-vs-azure.md) · [Siguiente página →](../M04-patrones-enterprise-aws/README.md)

## Cómo usar estos casos

Son **marcos de razonamiento**, no respuestas únicas. En licitación o comité técnico, documenta **supuestos**, **restricciones** y **trade-offs** explícitos.

## Caso 1 — Organización Microsoft-centric

**Perfil:** Entra ID source of truth, M365, Teams, SharePoint documental, contrato EA Microsoft.

**Inclinación habitual:** Azure para identidad, integración documental (AI Search sobre SharePoint), Azure OpenAI para GenAI interno.

**AWS aún puede entrar en:** data lake analítico histórico, partners ISV solo AWS, DR.

**Preguntas clave:**

- ¿El contrato EA cubre Azure OpenAI con datos internos?
- ¿SharePoint es corpus RAG oficial o hay copias en S3?

## Caso 2 — Startup / producto cloud-native

**Perfil:** sin CPD, equipos DevOps, time-to-market crítico, posible inversión internacional.

**Inclinación habitual:** AWS por ecosistema startup, documentación y servicios managed amplios; o «cloud del inversor».

**Riesgo:** coste sin FinOps desde día 1.

## Caso 3 — Sector público / transporte metropolitano

**Perfil:** licitación, transparencia, RGPD, posible ENS, datos operativos sensibles, presupuesto anual.

**Consideraciones:**

- Contrato marco cloud (AWS, Azure, o integrador local).
- **Clasificación de datos** antes de RAG sobre documentación interna.
- Preferencia por regiones EU y logs auditables.
- Identidad workforce puede ser **Entra** aunque data lake esté en AWS — patrón híbrido frecuente.

**No decidir solo IT:** legal y compras en GenAI con documentos laborales o contractuales.

## Caso 4 — IA generativa empresarial (asistente interno)

**Requisitos típicos:** SSO, límite coste, RAG sobre PDFs, sin entrenar con datos en vendor, auditoría.

| Opción | Stack indicativo |
|--------|------------------|
| Microsoft shop | Entra + Azure OpenAI + AI Search |
| AWS shop | IAM Identity Center + Bedrock + KB |
| Híbrido | Entra auth + API gateway + Bedrock backend |

**Comparar contratos:** retención prompt, región procesamiento, modelos permitidos.

## Caso 5 — Plataforma ML clásica (no GenAI)

**Requisitos:** pipelines, registry, batch + online inference, datos en lago.

| Skills | Opción |
|--------|--------|
| Platform K8s maduro | AKS/EKS + MLflow |
| Pocos ops ML | SageMaker o Azure ML managed |

GenAI puede **añadirse después** sin rehacer lago si gobierno común.

## Caso 6 — Multicloud declarado

**Solo viable si:** presupuesto para **dos** operaciones, estándares comunes (K8s, OTel, Parquet), integración identidad federada.

**Alternativa más barata:** una nube primaria + SaaS best-of-breed con APIs.

## Escenario integrado (ilustrativo)

**Contexto tipo operador transporte:**

- AD/Entra corporativo.
- Datos agregados de demanda en lago cloud.
- Manuales mantenimiento en SharePoint.
- Piloto asistente interno acotado a 200 usuarios.

**Razonamiento posible:**

1. Identidad → Entra SSO obligatorio.
2. Corpus → AI Search indexando SharePoint **o** sync a Blob/S3 con gobierno.
3. GenAI → Azure OpenAI **si** contrato cubre; si contrato cloud es AWS, Bedrock + KB con VPC.
4. ML demanda → SageMaker/Azure ML batch según lago elegido.
5. Observabilidad unificada → SIEM corporativo + tags coste.

Resultado puede ser **híbrido pragmático** — no pure AWS ni pure Azure.

## Checklist de comité (10 minutos)

1. ¿Dónde está el dato maestro y su clasificación?
2. ¿Qué identidad usa el usuario final?
3. ¿Qué contrato marco tenemos ya?
4. ¿Qué skills operan day-2?
5. ¿Cuál es presupuesto OpEx IA a 12 meses?
6. ¿Plan de salida si el POC falla?

## Resumen

- No hay «mejor nube» — hay **mejor encaje por contexto**.
- Sector público y Microsoft-centric empujan componentes Azure en identidad y documentos.
- AWS sigue fuerte en lago composable y amplitud ML/infra — híbrido es normal.
