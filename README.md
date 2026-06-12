# Gallardia — Ecosistema de Agentes Empresariales Autónomos

**Empresa digital 24/7 para una empresa operando en Panamá**: Marketing, Ventas y Administración Financiera operadas por agentes de IA especializados, supervisados por un Agente Director General, con la mínima intervención humana viable (y compuertas humanas exactamente donde la ley, el dinero y la reputación lo exigen).

> Este repositorio contiene el diseño completo del sistema: análisis estratégico, arquitectura, agentes, flujos, herramientas, costos, cronograma, riesgos y hoja de ruta. Es el plano de construcción ("blueprint") sobre el cual se implementa el código.

---

## Índice de documentación

| # | Documento | Contenido |
|---|-----------|-----------|
| 01 | [Análisis estratégico](docs/01-analisis-estrategico.md) | El problema real, cuellos de botella, riesgos legales, limitaciones técnicas duras, qué es automatizable hoy y qué no |
| 02 | [Arquitectura del ecosistema](docs/02-arquitectura.md) | Diagrama general, roster de agentes, bases de datos, infraestructura, seguridad, control de acceso, respaldos, escalabilidad |
| 03 | [Área Marketing Autónomo](docs/03-marketing-autonomo.md) | Agentes de contenido, pipeline multimedia, publicación multicanal, investigación con fuentes, optimización por métricas |
| 04 | [Área Ventas Autónomas](docs/04-ventas-autonomas.md) | Conserje omnicanal, calificación de leads, motor de cotizaciones y precios, seguimiento, agendamiento, reportería comercial |
| 05 | [Área Administración y Contabilidad](docs/05-finanzas-contabilidad.md) | Ingesta de facturas, clasificación contable, estados financieros, alertas, dashboards ejecutivos |
| 06 | [Contexto Panamá](docs/06-contexto-panama.md) | Facturación electrónica (DGI/PAC), ITBMS, ISR, municipio, CSS, Ley 81 de protección de datos, pagos locales (Yappy/ACH) |
| 07 | [Herramientas, APIs y presupuesto](docs/07-herramientas-apis-costos.md) | Stack completo en 3 niveles: MVP económico, Profesional, Empresarial — con costos estimados |
| 08 | [Flujos de trabajo detallados](docs/08-flujos-de-trabajo.md) | Flujos paso a paso: día típico del sistema, lead → cierre, foto → publicación, factura → estado financiero |
| 09 | [Cronograma de implementación](docs/09-cronograma.md) | Plan por fases (16 semanas), hitos, criterios de aceptación |
| 10 | [Riesgos, limitaciones y roadmap](docs/10-riesgos-limitaciones-roadmap.md) | Matriz de riesgos, limitaciones honestas, versión ideal vs. versión construible hoy, mejoras futuras |

---

## Resumen ejecutivo (TL;DR)

**Qué se construye:** un ecosistema de ~15 agentes especializados organizados en 4 capas (Dirección, Marketing, Ventas, Finanzas) sobre una columna vertebral común: una base de datos central (PostgreSQL), un orquestador de eventos (n8n + Claude API / Claude Agent SDK), un bus de aprobaciones humanas vía WhatsApp, y dashboards ejecutivos (Metabase).

**Qué hace solo:**
- Produce y publica mínimo 1 contenido diario en Facebook, Instagram, TikTok, LinkedIn, Google Business Profile y YouTube Shorts; si no hay material tuyo, investiga fuentes oficiales y crea contenido educativo citado.
- Procesa tus fotos/videos de trabajos: clasifica, mejora, edita, genera reels, shorts, historias, carruseles y miniaturas.
- Atiende leads 24/7 por WhatsApp, Messenger, Instagram DM y formularios web; califica, cotiza con tu lista de precios (incluida visita técnica USD 25), agenda, da seguimiento y reporta.
- Lee facturas emitidas y recibidas, clasifica movimientos, construye estado de resultados, flujo de caja y balance, y alerta sobre morosos y problemas de caja.
- Un Agente Director General consolida todo, prioriza, asigna trabajo y responde preguntas estratégicas ("¿cuál fue el cliente más rentable?", "¿qué hago esta semana?").

**Qué NO hace solo (por diseño):** presentar impuestos ante la DGI, aprobar descuentos fuera de política, firmar cotizaciones por encima de un umbral, publicar contenido sensible sin tu visto bueno inicial. Estas compuertas son deliberadas — ver [doc 01](docs/01-analisis-estrategico.md) y [doc 10](docs/10-riesgos-limitaciones-roadmap.md).

**Cuánto cuesta (operación mensual estimada):**

| Nivel | Setup (única vez) | Operación mensual | Para quién |
|-------|------------------|-------------------|------------|
| 1. MVP económico | USD 3,000–8,000 (o sudor propio) | USD 180–400 | Validar el modelo en 6–8 semanas |
| 2. Sistema profesional | USD 15,000–35,000 | USD 700–1,600 | Operación seria, el recomendado |
| 3. Empresarial de alto nivel | USD 60,000–150,000 | USD 3,000–8,000 | Multi-marca, multi-país, equipo |

Detalle completo en [doc 07](docs/07-herramientas-apis-costos.md).

**Tiempo de implementación realista:** 16 semanas para el sistema profesional completo; el MVP genera valor desde la semana 4 (ventas por WhatsApp) y la semana 6 (publicación diaria). Cronograma en [doc 09](docs/09-cronograma.md).

---

## Principios de diseño

1. **Una sola fuente de verdad.** Todo (leads, conversaciones, contenidos, métricas, facturas) vive en una base de datos central. Los agentes leen y escriben ahí, no en silos.
2. **Eventos, no cron jobs sueltos.** Cada cosa que pasa (llega un mensaje, se publica un post, entra una factura) emite un evento; los agentes reaccionan a eventos. Esto hace el sistema auditable y extensible.
3. **Human-in-the-loop quirúrgico.** La intervención humana no se elimina: se concentra en 4 compuertas de alto valor (aprobación de contenido sensible, descuentos fuera de política, cotizaciones grandes, cierres fiscales). Todo lo demás fluye solo.
4. **Cada acción deja rastro.** Log inmutable de qué agente hizo qué, cuándo, con qué datos y qué costo. Sin esto no hay confianza ni mejora.
5. **Los agentes proponen, las políticas disponen.** Los precios, descuentos, tonos de marca y reglas fiscales viven en tablas de configuración versionadas — no en la "cabeza" del modelo. El LLM razona; la política decide.
6. **Degradación elegante.** Si una API de terceros cae (Meta, TikTok), el sistema encola, reintenta y te avisa — no se cae todo.

---

## Estado del proyecto

- [x] Fase 0 — Diseño y blueprint (este repositorio)
- [ ] Fase 1 — Fundaciones: base de datos, orquestador, WhatsApp + CRM
- [ ] Fase 2 — Marketing: pipeline de contenido y publicación multicanal
- [ ] Fase 3 — Ventas: calificación, cotizador, seguimiento
- [ ] Fase 4 — Finanzas: ingesta de facturas, estados financieros, alertas
- [ ] Fase 5 — Agente Director General + dashboards ejecutivos
