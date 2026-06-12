# 07 — Herramientas, APIs y presupuesto en 3 niveles

Tres configuraciones del mismo diseño. La arquitectura lógica ([doc 02](02-arquitectura.md)) no cambia entre niveles — cambia cuánto es self-hosted vs. gestionado, cuánta redundancia hay y cuánto músculo de IA se compra. **Se puede empezar en Nivel 1 y subir sin re-arquitecturar.**

Precios en USD, aproximados a mediados de 2026; los de APIs de plataformas (Meta, tokens LLM) varían — tratarlos como órdenes de magnitud, no contrato.

---

## Nivel 1 — MVP económico

Para validar el modelo en 6–8 semanas con volumen bajo-medio (≤500 leads/mes, 1–2 posts/día).

| Componente | Herramienta | Costo/mes |
|-----------|-------------|-----------|
| Servidor (todo en Docker) | Hetzner/DigitalOcean VPS 4 vCPU/8 GB | 25–50 |
| Orquestador | n8n self-hosted | 0 |
| BD + vectores | PostgreSQL + pgvector (mismo VPS) | 0 |
| Bandeja omnicanal/CRM | Chatwoot self-hosted (WA, Messenger, IG DM) | 0 |
| WhatsApp Business API | Meta Cloud API directo (conversaciones de servicio gratis; plantillas utility/marketing por mensaje) | 15–60 |
| LLM (agentes) | Claude API — Haiku clasifica, Sonnet conversa/redacta; prompt caching | 50–150 |
| Publicación 6 redes + métricas | Metricool (plan Advanced) **o** Postiz self-hosted (0) | 0–55 |
| Edición de imagen | Pipeline propio (sharp/ffmpeg/Real-ESRGAN en el VPS) | 0 |
| Video (reels/shorts) | Remotion self-hosted o Creatomate plan base | 0–49 |
| Almacenamiento media | Cloudflare R2 (100 GB aprox.) | 2–5 |
| Dashboards | Metabase self-hosted | 0 |
| Cotizaciones PDF | HTML→PDF (gotenberg) self-hosted | 0 |
| Agenda | Google Calendar API + Workspace existente | 0–7 |
| Backups | Snapshots del VPS + R2 | 5–10 |
| **Total operación** | | **≈ 100–390/mes** |

**Setup:** USD 3,000–8,000 si lo construye un desarrollador freelance competente en ~6–8 semanas (o tiempo propio). 

**Sacrificios honestos del MVP:** TikTok semi-manual o vía Metricool; mejora de imagen buena pero no premium; un solo servidor (RTO ~4 h); aprobación humana recomendada en casi todo el contenido los primeros 2 meses; conciliación bancaria semanal manual-asistida.

---

## Nivel 2 — Sistema profesional ⭐ recomendado

Operación seria: 1,500–5,000 leads/mes, 2–3 contenidos/día, finanzas completas, alta disponibilidad razonable.

| Componente | Herramienta | Costo/mes |
|-----------|-------------|-----------|
| Cómputo | 2 VPS (app + workers) o k3s | 80–150 |
| BD gestionada | Supabase Pro / Neon / RDS (PITR, réplicas) | 25–100 |
| Orquestador | n8n self-hosted (HA) o n8n Cloud Pro | 0–60 |
| CRM/bandeja | Chatwoot gestionado o Kommo/respond.io | 50–150 |
| WhatsApp API | Cloud API o BSP (360dialog) — mayor volumen | 80–250 |
| LLM | Claude API con escalonamiento + caching + batch; Director en modelo top | 250–700 |
| Publicación/métricas | Metricool Advanced + APIs nativas Meta/GBP/YouTube donde convenga | 55–100 |
| Mejora imagen premium | Adobe Firefly Services API o Cloudinary AI o Topaz | 50–150 |
| Video | Creatomate/Shotstack + ElevenLabs TTS (voz ES-LA) | 80–200 |
| Avatares/explicativos (opcional) | HeyGen | 0–90 |
| Almacenamiento + CDN | R2 + CDN, ~500 GB | 10–25 |
| Dashboards | Metabase self-hosted + alertas | 0–20 |
| Observabilidad | Better Stack / Grafana Cloud free→pro | 0–30 |
| Pasarela de pago | Yappy Comercial + Tilopay (comisiones por transacción, no fijas) | variable |
| Contabilidad conectada | Alegra (localización Panamá, API) o QuickBooks | 20–60 |
| Backups multi-región | BD PITR + R2 replicado | 10–20 |
| **Total operación** | | **≈ 710–2,100/mes** (típico ~1,200) |

**Setup:** USD 15,000–35,000 (equipo de 1–2 desarrolladores + el dueño, 14–16 semanas — ver [doc 09](09-cronograma.md)).

**Qué gana sobre el MVP:** TikTok automatizado, video y foto de calidad publicitaria, conciliación más automática (Yappy webhook + Alegra), redundancia (RTO < 1 h), Director General completo con briefing semanal, atribución contenido→venta de punta a punta.

---

## Nivel 3 — Empresarial de alto nivel

Multi-marca/multi-país, equipo humano usando el sistema, SLAs.

| Componente | Herramienta | Costo/mes |
|-----------|-------------|-----------|
| Cloud gestionado | AWS/GCP: ECS/GKE, colas SQS/PubSub, autoscaling, dev/staging/prod | 600–1,500 |
| Plataforma de agentes | **Claude Agent SDK** (sub-agentes, MCP, memoria) + LangGraph para flujos críticos; evaluaciones automáticas de calidad de agente (LLM-as-judge + golden sets) | en LLM |
| LLM | Claude API alto volumen, modelos top para Dirección/Guardián | 1,000–3,000 |
| Datos | Postgres gestionado + warehouse (BigQuery/ClickHouse) + dbt; ML propio de scoring de leads y predicción de caja | 200–500 |
| CRM | HubSpot Pro / Salesforce si hay equipo comercial | 400–1,500 |
| Marketing | APIs nativas completas + Marketing API para pauta con reglas automáticas; CDP ligero | 100–300 |
| Media | Pipeline Firefly Services + render farm Remotion; biblioteca de marca DAM | 200–500 |
| Seguridad | SSO/SAML, Vault, WAF, pentest anual, prácticas tipo SOC 2 | 100–300 |
| Observabilidad | Datadog/Grafana, on-call, presupuestos de tokens por agente con corte | 100–300 |
| **Total operación** | | **≈ 2,700–8,000/mes** |

**Setup:** USD 60,000–150,000 (equipo de 3–4, 5–6 meses).

---

## Catálogo de APIs utilizadas (todos los niveles)

| API | Para qué | Requisito clave |
|-----|----------|-----------------|
| Meta Graph API (Pages, Instagram) | Publicar FB/IG, métricas, comentarios | App Business verificada; permisos `pages_manage_posts`, `instagram_content_publish` |
| Messenger Platform + Instagram Messaging | DMs de ventas | Webhooks + revisión de app |
| WhatsApp Business Platform (Cloud API) | Conversaciones de venta, aprobaciones internas, notificaciones | Número dedicado; plantillas aprobadas; verificación de negocio |
| TikTok Content Posting API | Publicar videos | Auditoría de app (o usar Metricool/Postiz) |
| LinkedIn Community Management API | Posts de página | Aprobación de partner (o Metricool) |
| Google Business Profile API | Updates locales, fotos, reseñas | Verificación del perfil |
| YouTube Data API v3 | Subir Shorts, miniaturas | Verificación OAuth de la app; cuota 10k unidades/día |
| Claude API (Anthropic) | Cerebro de todos los agentes; visión para OCR y clasificación de fotos | Escalonamiento de modelos + caching |
| Metricool API / Postiz | Capa unificada de publicación + analítica | — |
| Creatomate/Shotstack/Remotion | Render de reels/shorts con plantilla | — |
| ElevenLabs / Azure TTS | Voz en español para explicativos | — |
| API del PAC / Alegra | Leer facturas emitidas; emisión fiscal autorizada | Credenciales del software existente |
| Yappy Comercial API | Links de cobro y confirmación automática | Cuenta Banco General comercial |
| Google Calendar API | Agendamiento de reuniones/visitas | Workspace |

---

## Reglas de control de costo (todas las configuraciones)

1. Presupuesto mensual de tokens **por agente** con alerta al 80% y corte suave al 100% (el agente degrada a plantillas).
2. Prompt caching obligatorio en prompts de sistema largos (conserje, redactor) — ahorra 60–90% del costo de input.
3. Batch API para trabajos nocturnos (clasificación de métricas, contabilidad) — 50% de descuento.
4. El Director revisa el costo del propio sistema como una línea más del estado de resultados: **el ecosistema se audita a sí mismo como centro de costo.**
