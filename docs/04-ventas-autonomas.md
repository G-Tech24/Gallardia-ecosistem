# 04 — Área 2: Ventas Autónomas

Objetivo: que ningún lead se enfríe jamás. Un vendedor profesional digital que responde en segundos, califica, cotiza, agenda, da seguimiento y reporta — 24/7, en todos los canales, con historial completo.

---

## 1. Canales y captura

| Canal | Integración | Captura |
|-------|-------------|---------|
| WhatsApp Business | Cloud API (número dedicado de empresa) | Mensajes entrantes; links wa.me con texto pre-llenado para atribución de campaña |
| Facebook Messenger | Messenger Platform (webhooks de la página) | DMs y respuestas a anuncios |
| Instagram DM | Instagram Messaging API | DMs, respuestas a historias |
| Formularios web / landing | Webhook directo a n8n | Campos + UTM |
| Meta Lead Ads | Leads API | Formularios nativos de anuncios |

Todo converge en **una sola bandeja omnicanal** (Chatwoot self-hosted en MVP/Pro; alternativa SaaS: Kommo o respond.io). Cada contacto se deduplica por teléfono/email y conserva su hilo histórico completo — el requisito de "registrar conversaciones y mantener historial" se cumple a nivel de plataforma, no de chat individual.

**Cumplimiento:** primer contacto registra `consentimiento_at` (el cliente inició la conversación = base legítima para responder; para marketing posterior se pide opt-in explícito). Ver Ley 81 en [doc 06](06-contexto-panama.md).

---

## 2. El Conserje: conversación de venta profesional

Prompt de rol del Conserje (resumen de diseño):
- Personalidad configurada: cordial, profesional, español de Panamá, sin tecnicismos innecesarios, respuestas cortas tipo chat (no párrafos de robot).
- **Solo afirma lo que está en la base de conocimiento** (servicios, cobertura, tiempos, garantías, FAQs) y **solo cotiza con la tabla de precios vigente**. Si no sabe: "déjame confirmarlo y te escribo en unos minutos" + tarea a humano.
- Objetivo de cada conversación: avanzar una etapa del embudo (descubrir necesidad → calificar → cotizar → agendar → cerrar), nunca solo "responder".
- Escalamiento a humano: cliente molesto, negociación fuera de política, solicitud legal/compleja, o petición explícita de hablar con una persona. El traspaso incluye resumen de la conversación.

## 3. Calificación e intención

El **Calificador** corre sobre cada conversación (modelo barato, en segundo plano) y mantiene en el CRM:

- **Score 0–100** con framework BANT adaptado: Necesidad clara (+30), presupuesto mencionado o implícito (+25), urgencia (+25), autoridad de decisión (+20).
- **Etapa:** nuevo → conversando → calificado → cotizado → negociación → agendado → ganado/perdido (con motivo).
- **Señales de intención de compra** (dispara prioridad alta + notificación): pregunta por precio total, por fechas de inicio, por formas de pago, pide la cotización "formal".
- **Upsell/cross-sell:** reglas configurables por servicio ("quien pide A suele necesitar B") + detección en conversación; el Conserje ofrece de forma natural, máximo 1 oferta adicional por conversación.

## 4. Motor de cotizaciones y precios

La tabla `price_list` (versionada, editable por el dueño vía panel o WhatsApp: "actualiza el precio de X a 450") define por servicio:

```yaml
servicio: "Instalación de <servicio ejemplo>"
precio_base: 1200.00        # USD
costo_estimado: 700.00      # para margen
itbms: 7%                   # según tipo de servicio (ver doc 06)
modalidades:
  - venta_completa: 100% contra entrega de acta
  - abono_inicial: 50% inicio / 50% entrega
  - pago_parcial: 3 pagos (40/30/30)
  - financiamiento: enlace a financiera aliada (el sistema no financia)
visita_tecnica: 25.00       # acreditable al contratar (configurable)
descuentos:
  auto_max: 5%              # el agente puede aplicar solo
  con_aprobacion: 6-15%     # requiere ✅ del dueño por WhatsApp
  prohibido: ">15%"
```

El **Cotizador**:
1. Construye la cotización con ítems, cantidades, modalidad elegida.
2. Calcula automáticamente: subtotal, **ITBMS (7% general)**, total, costo estimado, **margen y utilidad estimada** (visible solo internamente, jamás en el PDF del cliente).
3. Genera PDF con marca (plantilla HTML→PDF), número correlativo, vigencia (ej. 15 días) y condiciones.
4. Si total > umbral configurable (ej. USD 3,000) o descuento > 5% → cola de aprobación; si no, envía directo.
5. Registra en `quotes` y arma el seguimiento.

**Importante (Panamá):** la cotización no es factura. La factura fiscal la emite el software autorizado/PAC cuando la venta se concreta ([doc 06](06-contexto-panama.md)).

## 5. Seguimiento y agendamiento

- **Secuencia post-cotización:** D+1 ("¿pudiste revisarla? ¿alguna duda?"), D+3 (valor agregado: foto de trabajo similar terminado), D+7 (recordatorio de vigencia), D+14 (última, con opción de archivar). Fuera de la ventana de 24 h de WhatsApp se usan **plantillas aprobadas** (categoría utility cuando aplica — más barata y mejor entregada que marketing).
- **Anti-spam:** máximo de toques sin respuesta configurable; un "no me interesa" detiene todo y marca el motivo.
- **Recordatorios de cobro de abonos** según la modalidad pactada (enlaza con Finanzas).
- **Agendador:** integrado a Google Calendar; propone 2–3 huecos reales, agenda reunión o **visita técnica (USD 25)** — genera el cobro de la visita (link de pago Yappy/transferencia) y la confirma; recordatorio 24 h y 2 h antes; reagenda en lenguaje natural.

## 6. Reportería comercial

El **Analista comercial** genera:

- **Diario (7:00 a. m., WhatsApp al dueño):** leads nuevos ayer, conversaciones activas, cotizaciones enviadas, requieren tu acción (aprobaciones, escalamientos), citas de hoy.
- **Semanal (lunes):** embudo completo, tasa de conversión por etapa y por canal, cotizaciones por vencer, pipeline ponderado (Σ valor × probabilidad por etapa).
- **Mensual:** tasa de cierre, **tiempo promedio de venta** (lead→ganado), **valor promedio de cliente**, CAC por canal (cruzando con gasto de marketing), ranking de servicios por margen real, motivos de pérdida agregados.

Todas las métricas salen de la BD central — son las mismas que consume el Director General y los dashboards.
