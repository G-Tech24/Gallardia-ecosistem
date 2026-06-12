# 11 — Investigación open source: catálogo evaluado

**Objetivo:** encontrar ecosistemas open source que reemplacen, mejoren o aceleren el blueprint v1 ([docs 01–10](.)), evaluados con criterio de CTO: tiempo de desarrollo eliminado, costo, capacidades, comunidad. Contexto de negocio: empresa panameña de jardinería/paisajismo, poda, limpieza de terrenos, pintura, remodelaciones menores, mantenimiento y limpieza de edificios, con clientes privados y gubernamentales, cuadrillas a ~USD 35/día.

**Metodología y honestidad:** los puntos marcados ✅ fueron verificados con fuentes en junio 2026 (URLs al pie de cada sección); el resto proviene de conocimiento técnico actualizado a enero 2026 — re-verificar estrellas/licencias antes de adoptar (los proyectos OSS cambian de licencia con frecuencia: HashiCorp, dbt y otros ya lo hicieron).

---

## Fase 1 — Mapeo por módulos y veredicto rápido

| Módulo | Plan v1 (a medida/SaaS) | Ganador open source | Veredicto |
|---|---|---|---|
| ERP (cotizar, vender, comprar, inventario, proyectos, contabilidad) | Esquema Postgres a medida | **ERPNext / Frappe** (alt.: Odoo CE + OCA) | **Reemplazar el 70% del modelo de datos a medida** |
| CRM + embudo | Tablas propias + Chatwoot | El CRM del ERP elegido (Twenty/EspoCRM solo si no hay ERP) | Absorbe el ERP |
| Órdenes de trabajo / cuadrillas / móvil | No existía (¡hueco del v1!) | **Atlas CMMS** ✅ o módulos ERPNext | Incorporar |
| Atención omnicanal | Chatwoot | **Chatwoot** (confirmado) + Typebot para flujos | Mantener |
| WhatsApp | Cloud API oficial | Cloud API oficial (Evolution API/Baileys = riesgo de baneo y legal — rechazado) | Mantener |
| Publicación social | Metricool (SaaS) | **Postiz** ✅ (AGPL-3.0, 12+ redes incl. TikTok/IG/YouTube self-hosted) | Reemplazar SaaS |
| Video/reels | Creatomate (SaaS) | **Remotion** (ojo licencia: gratis ≤3 empleados) / **Revideo** (MIT) + FFmpeg + WhisperX | Reemplazar SaaS |
| Imagen | Cloudinary (SaaS) | sharp + Real-ESRGAN/Upscayl + rembg (+ ComfyUI opcional con GPU) | Reemplazar SaaS |
| TTS español | ElevenLabs (SaaS) | Piper / Kokoro / F5-TTS (calidad ya aceptable; ElevenLabs solo si se exige premium) | Híbrido |
| Orquestación | n8n | **n8n** (licencia fair-code Sustainable Use — legal para uso interno propio; no es OSI) / alt. MIT: Activepieces; alt. potente: Windmill | Mantener con conciencia de licencia |
| Agentes IA | Prompts + workers propios | **Claude Agent SDK** o **LangGraph** (checkpointing, humano-en-el-loop) | Incorporar en capa crítica |
| Gateway LLM / control de costos | A medida | **LiteLLM** (MIT): enrutamiento Haiku/Sonnet/Opus, presupuestos por agente, fallbacks | Incorporar |
| Observabilidad LLM | Logs propios | **Langfuse** (núcleo MIT, self-hosted): trazas, costos por agente, evaluaciones | Incorporar |
| RAG / conocimiento | pgvector a medida | pgvector basta a esta escala; LlamaIndex/Docling para ingestión | Mantener simple |
| OCR documental (facturas, pliegos) | LLM multimodal | **Docling** (IBM, MIT) + PaddleOCR; LLM multimodal como árbitro | Incorporar |
| Gestión documental | No existía | **Paperless-ngx** (contratos, fianzas, idoneidades, archivo fiscal 5 años) | Incorporar |
| Firma electrónica | No existía | **DocuSeal** (AGPL) — aceptación de cotizaciones/contratos (firma simple; la calificada sigue siendo del Registro Público) | Incorporar |
| BI / dashboards | Metabase | **Metabase** OSS (AGPL; las preguntas en lenguaje natural las da nuestro Director General vía SQL, no la versión paga) | Mantener |
| Reportes de campo con fotos GPS | No existía | **KoboToolbox / ODK** (estándar humanitario/empresarial, brutalmente probado) | Incorporar — joya |
| Rutas de cuadrillas | No existía | **VROOM + OSRM/OpenRouteService** | Incorporar — joya |
| Medición de terrenos | No existía | **Leaflet + Turf.js** sobre imagen satelital | Incorporar — joya |
| Clima para estimación | No existía | **Open-Meteo** (API abierta, histórico + pronóstico Panamá) | Incorporar |
| Scraping banco de precios | No existía | **Crawlee / Scrapy / crawl4ai + Playwright** | Incorporar |
| Identidad/SSO | Google OAuth | Google OAuth basta; Authentik si crece; Keycloak = overkill PyME | Mantener simple |
| Secretos | Variables cifradas | **Infisical** (o OpenBao si se quiere Vault sin licencia BSL) | Incorporar en nivel pro |
| Despliegue | Docker Compose manual | **Coolify** o **Dokploy** (PaaS self-hosted: deploy, SSL, backups con UI) | Incorporar |
| Monitoreo | Logs n8n | **Uptime Kuma** + **Netdata/Beszel**; SigNoz si se quiere APM completo | Incorporar |
| Backups | Snapshots | **restic** (media/config) + **pgBackRest/barman** (Postgres) — automatizados y probados | Incorporar |

---

## Fase 2/4 — Análisis de los componentes columna vertebral

### ERPNext + Frappe Framework — la decisión más importante de toda la investigación

- **Qué es:** ERP completo 100% libre (GPLv3, sin módulos "Enterprise" retenidos): CRM, cotizaciones/ventas, compras, inventario, **proyectos con costeo y partes de horas**, **mantenimiento**, activos, RR. HH./planilla, contabilidad de doble partida multi-libro, suscripciones (contratos recurrentes), portal de cliente, REST API nativa + webhooks, y el framework low-code Frappe para campos/doctypes propios (ej.: "Orden de trabajo de jardinería" con fotos y firma).
- **Qué reemplaza del v1:** las tablas `quotes, price_list, invoices_*, ledger_entries, chart_of_accounts, appointments` y gran parte de `leads/deals` — es decir, **el 60–70% del modelo de datos a medida y sus pantallas**, que era lo más caro de construir y mantener.
- **Ventajas:** un solo sistema para cotizar→ordenar→comprar→ejecutar→facturar→cobrar→contabilizar; el costeo por proyecto (rentabilidad real por trabajo) viene de fábrica; comunidad global activa (≈30k★ GitHub) con releases anuales mayores; Frappe Cloud existe como plan B gestionado.
- **Riesgos/complejidad:** corre sobre **MariaDB, no Postgres** → la analítica (Metabase) se conecta directo a MariaDB o se replica a Postgres; curva de aprendizaje 2–4 semanas; la localización contable de Panamá no viene hecha (plan de cuentas se importa; la facturación electrónica se resuelve por API del PAC, ver abajo).
- **Veredicto: incorporar como columna vertebral.** Ahorro estimado: **12–18 meses-desarrollador** frente a construir cotizador+contabilidad+inventario+proyectos a medida.

### Odoo Community + OCA — el contendiente

- Odoo CE (LGPL) es excelente, pero los módulos que esta empresa más necesita — **Field Service, contabilidad completa, planificación, hojas de horas avanzadas — son solo Enterprise** ✅ (confirmado para Odoo 18/19). La OCA (Odoo Community Association) mantiene [`OCA/field-service`](https://github.com/OCA/field-service) y módulos de proyecto que llenan huecos, con fricción de integración y upgrades anuales laboriosos.
- **Cuándo elegir Odoo:** si se contrata un partner local/regional (el ecosistema de consultores Odoo en LatAm es el más grande) o si se acepta pagar Enterprise (~USD 25–35/usuario/mes, que mata parte del ahorro).
- **Veredicto:** segunda opción válida; **ERPNext gana para esta empresa** por costo total y API.

### Atlas CMMS — operación de cuadrillas ✅

- AGPLv3, web + **app móvil**, self-hosted: órdenes de trabajo con fotos, checklists, **firma del cliente al completar**, mantenimiento preventivo programado (ideal para contratos mensuales de jardinería/limpieza), inventario de partes, reportes. Activo (releases recientes).
- Reemplaza: desarrollo móvil a medida para cuadrillas (~4–6 meses-dev).
- Riesgo: solapa con los módulos de mantenimiento de ERPNext → **regla de decisión:** empezar con ERPNext solo; adoptar Atlas CMMS únicamente si la experiencia móvil de cuadrilla de ERPNext resulta insuficiente en el piloto (criterio: ¿el capataz puede cerrar una orden con 5 fotos y firma en <2 minutos desde un teléfono de gama baja en el sol?).

### Facturación electrónica Panamá

- No existe localización FE panameña madura open source para ningún ERP. **The Factory HKA (PAC autorizado, Res. 201-9719) ofrece API de integración para ERPs** ✅ — el flujo correcto: ERPNext genera la factura interna → API HKA la autoriza/emite (XML legal + CAFE PDF) → respuesta se archiva en ERPNext y Paperless-ngx. Esto convierte la limitación legal del v1 en una integración de ~2 semanas.

### Capa de IA (sustituye al "pegamento" a medida del v1)

| Pieza | Elección | Por qué |
|---|---|---|
| Agentes con estado (Director, Cotizador asistido) | **Claude Agent SDK** o **LangGraph** | Checkpointing, interrupciones humano-en-el-loop (la "cola de aprobaciones" del v1 viene casi gratis), trazabilidad |
| Tareas mecánicas (clasificar, extraer, etiquetar) | Llamadas directas vía **LiteLLM** | Enrutamiento barato Haiku→Sonnet, presupuesto por agente, fallback de proveedor |
| Trazas y costos | **Langfuse** self-hosted | Cada acción de agente con costo, latencia y evaluación — el `audit_log` del v1 a nivel LLM, sin construirlo |
| Conexión agentes↔ERP | **MCP** (servidores Postgres/HTTP existentes + uno propio fino sobre la REST API de Frappe) | El ecosistema MCP elimina semanas de integraciones |
| OCR/documentos | **Docling** + PaddleOCR, LLM como árbitro de casos dudosos | Facturas de proveedores y pliegos de licitación |

### Lo que se evaluó y se descartó (con razón)

- **Evolution API / Baileys (WhatsApp no oficial):** viola términos de Meta → baneo del número = perder el canal de ventas; además expone datos de clientes fuera del marco contratado (Ley 81). Rechazado aunque sea popular.
- **Dify / Flowise / Langflow:** excelentes para prototipos de chatbots; para agentes de producción con auditoría y compuertas, quedan cortos y algunas licencias (Dify) traen cláusulas adicionales. No entran al núcleo.
- **Temporal:** durabilidad de workflows de clase mundial, pero operarlo excede a una PyME; n8n + colas Postgres cubren el caso. Reconsiderar en nivel extremo.
- **Kubernetes/microservicios:** Coolify/Dokploy + Docker en 1–2 VPS es el punto óptimo; k8s agrega costo operativo sin retorno a esta escala.
- **Mautic:** marketing automation por email pesado; el motor real de esta empresa es WhatsApp + redes. Listmonk (newsletters) basta si algún día hay lista de correo.
- **SuiteCRM/Krayin:** generaciones detrás de Twenty/EspoCRM, y el CRM del ERP los vuelve innecesarios.

---

## Fase 3 — Joyas ocultas (lo que usaría una empresa con presupuesto de cientos de miles)

1. **Datos abiertos de PanamaCompra en estándar OCDS** ✅ — la Dirección General de Contrataciones Públicas publica TODOS los procesos (salvo convenio marco) en formato estándar internacional, **actualizado semanalmente, con API** (panamacompraencifras.gob.pa) y datasets en datosabiertos.gob.pa. Esto habilita el **Agente de Licitaciones** (ver doc 12 §10X): vigilancia diaria de oportunidades de jardinería/mantenimiento/limpieza/pintura del Estado, scoring de fit, y armado asistido del expediente. Nadie en el sector lo está haciendo sistemáticamente — ventaja competitiva real.
2. **KoboToolbox / ODK** — formularios móviles offline-first con foto georreferenciada y GPS, usados por la ONU en terreno hostil. Para reportes de avance con evidencia fotográfica fechada y geolocalizada — exactamente lo que exige cobrarle a una institución pública sin discusiones.
3. **VROOM + OSRM** — el motor de optimización de rutas de vehículos que usan logísticas enormes, gratis: planifica las rutas diarias de N cuadrillas con ventanas horarias. Una visita más por cuadrilla/día = ~20% más capacidad sin contratar.
4. **Leaflet + Turf.js** — dibujar el polígono del terreno del cliente sobre imagen satelital y obtener m² al instante → **cotizar jardinería/limpieza de terrenos sin visita previa** en el 70% de los casos. De días a minutos.
5. **Open-Meteo** — histórico y pronóstico climático abierto para Panamá: márgenes de estimación dinámicos por temporada lluviosa (mayo–noviembre) y reprogramación proactiva de pintura exterior.
6. **Docling (IBM)** — convierte PDFs infernales (pliegos de licitación, facturas escaneadas) en estructura limpia para LLMs; estado del arte open source en documento→datos.
7. **statsforecast/Prophet (Nixtla/Meta)** — pronóstico de series de tiempo serio sin equipo de ciencia de datos: estacionalidad de demanda por servicio y proyección de caja.
8. **Coolify/Dokploy** — el "Heroku privado": deploy con git push, SSL automático, backups con UI. Reduce el costo de operar 12 contenedores a algo que una persona maneja.

---

## Fuentes principales verificadas (junio 2026)

- PanamaCompra OCDS/datos abiertos: [panamacompraencifras — API](https://ocdsv2dev.panamacompraencifras.gob.pa/api) · [compendio PanamaCompra — datos abiertos](https://compendio.panamacompra.gob.pa/datos-abiertos-en-compras-publicas/) · [Open Contracting Partnership — Panamá](https://data.open-contracting.org/en/publication/120) · [datosabiertos.gob.pa](https://www.datosabiertos.gob.pa/dataset/?tags=Compras)
- Odoo Field Service solo Enterprise / OCA: [doc oficial Odoo 18 Field Service](https://www.odoo.com/documentation/18.0/applications/services/field_service.html) · [OCA/field-service](https://github.com/OCA/field-service) · [comparativa CE/Enterprise proyectos](https://www.cybrosys.com/blog/what-are-the-differences-between-odoo-18-community-enterprise-in-project)
- Salario mínimo Panamá 2026 (D.E. N.°13 de 31-dic-2025, vigente 16-ene-2026; construcción $3.51/h R1; agro $1.64–2.10/h): [EY tax alert](https://www.ey.com/es_ce/technical/tax/tax-alerts/panama-salario-minimo-2026) · [La Prensa](https://www.prensa.com/economia/salario-minimo-2026-en-panama-estas-son-las-nuevas-tarifas-por-actividad-y-region/)
- Postiz AGPL-3.0, TikTok/IG/YouTube self-hosted: [github.com/gitroomhq/postiz-app](https://github.com/gitroomhq/postiz-app)
- The Factory HKA PAC con API para ERPs: [thefactoryhka.com.pa](https://thefactoryhka.com.pa/facturacion-electronica-cumple-con-la-dgi/)
- Atlas CMMS AGPLv3 activo (web+móvil, firmas, OT): [github.com/Grashjs/cmms](https://github.com/Grashjs/cmms) · [atlas-cmms.com](https://atlas-cmms.com/)
- Banco de precios scrapeable: [cochezycia.com](https://www.cochezycia.com/) · [novey.com.pa](https://www.novey.com.pa/) · [puntocochez.com](https://puntocochez.com/ferreteria)
