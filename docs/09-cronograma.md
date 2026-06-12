# 09 — Cronograma de implementación (16 semanas, nivel Profesional)

Principio: **cada fase entrega algo que produce dinero o ahorra tiempo por sí solo.** Si el proyecto se detuviera en cualquier hito, lo construido seguiría siendo útil. El MVP (nivel 1) es exactamente este plan ejecutado hasta la semana 8.

```mermaid
gantt
    dateFormat  YYYY-MM-DD
    title Plan de implementación (inicio de referencia: semana 0)
    section F0 Fundaciones
    Descubrimiento y configuración (precios, tono, FAQs, accesos)  :f0a, 2026-07-01, 7d
    Infra base: VPS, Postgres, n8n, R2, backups, repositorio       :f0b, 2026-07-01, 10d
    section F1 Ventas (primero: paga el resto)
    WhatsApp Cloud API + Chatwoot + esquema de datos                :f1a, 2026-07-11, 7d
    Conserje + Calificador + base de conocimiento                   :f1b, 2026-07-18, 10d
    Cotizador + motor de precios + PDF + aprobaciones               :f1c, 2026-07-25, 7d
    Seguimiento + Agendador + Yappy                                 :f1d, 2026-08-01, 7d
    🏁 HITO 1: ventas 24/7 operando                                 :milestone, 2026-08-08, 0d
    section F2 Marketing
    Messenger + IG DM al conserje                                   :f2a, 2026-08-08, 5d
    Pipeline multimedia (clasificar, mejorar, variantes)            :f2b, 2026-08-08, 12d
    Publicador 6 redes (Metricool/APIs) + calendario editorial      :f2c, 2026-08-15, 10d
    Investigador + Redactor + Guardián                              :f2d, 2026-08-20, 10d
    🏁 HITO 2: 1 contenido diario automático                        :milestone, 2026-08-30, 0d
    section F3 Finanzas
    Capturador (API facturación + OCR recibidas)                    :f3a, 2026-08-30, 10d
    Clasificador + ledger + estados financieros                     :f3b, 2026-09-09, 10d
    Centinela de alertas + borradores tributarios                   :f3c, 2026-09-16, 7d
    🏁 HITO 3: finanzas al día, diario                              :milestone, 2026-09-23, 0d
    section F4 Dirección
    Analistas (métricas + comercial) y atribución completa          :f4a, 2026-09-23, 7d
    Director General + briefings + chat ejecutivo                   :f4b, 2026-09-30, 10d
    Dashboards Metabase + roles                                     :f4c, 2026-09-30, 7d
    section F5 Endurecimiento
    Seguridad, pruebas de restauración, runbooks, ajuste de costos  :f5, 2026-10-10, 10d
    🏁 HITO 4: sistema completo en producción                       :milestone, 2026-10-20, 0d
```

## Detalle por fase y criterios de aceptación

### Fase 0 — Fundaciones (semanas 1–2)
Insumos del dueño (esto es lo único que no puede hacer la máquina): lista de servicios con precios y costos, política de descuentos, tono de marca y ejemplos de comunicación, FAQs reales, accesos (Meta Business, cuentas de redes, software de facturación, Google Workspace), umbrales de aprobación.
**Aceptación:** infra desplegada con backups probados; configuración versionada en Git; cuentas API creadas y verificadas (la verificación de negocio de Meta puede tomar días — se inicia el día 1).

### Fase 1 — Ventas (semanas 2–6) → HITO 1
**Aceptación:** lead de prueba por WhatsApp recibe respuesta < 60 s; conversación completa hasta cotización PDF correcta (ITBMS y márgenes verificados contra hoja de cálculo del contador); aprobación por WhatsApp funciona; seguimiento D+1 dispara; visita técnica agendada con cobro Yappy. **Desde aquí el sistema ya genera retorno.**

### Fase 2 — Marketing (semanas 6–10) → HITO 2
**Aceptación:** 14 días de calendario lleno; flujo foto→publicación en < 30 min con aprobación; pieza educativa con fuentes verificables publicada; las 6 redes reciben contenido; métricas capturadas a 24/72 h; cero publicaciones sin pasar por el Guardián.

### Fase 3 — Finanzas (semanas 10–13) → HITO 3
**Aceptación:** 100% de facturas emitidas del mes en la BD; OCR de recibidas > 90% sin intervención; estado de resultados del mes cuadra con el del contador (±1%); proyección de caja 13 semanas generada; alerta de moroso dispara correctamente; borrador ITBMS coincide con el cálculo del contador.

### Fase 4 — Dirección (semanas 13–15)
**Aceptación:** briefing diario 7:00 con datos correctos; el DG responde las 6 preguntas estratégicas del requerimiento con cifras trazables a la BD; dashboards con roles funcionando.

### Fase 5 — Endurecimiento (semanas 15–16) → HITO 4
**Aceptación:** prueba de restauración de backup exitosa; rotación de secretos documentada; runbook de incidentes; revisión de costos reales vs. presupuesto; reducción de compuertas de aprobación según confianza ganada (el dueño decide cuáles abrir).

## Régimen permanente (post-semana 16)

- **Tiempo humano de operación:** 20–40 min/día del dueño (aprobaciones, briefing, excepciones) + 1–2 h/semana del contador (cola de revisión, presentaciones DGI).
- **Mantenimiento técnico:** 4–8 h/mes (actualizaciones, cambios de APIs de plataformas, ajuste de prompts según métricas de calidad).
- **Ciclo de mejora mensual:** el Director propone; el dueño aprueba; los prompts y políticas se versionan en Git.
