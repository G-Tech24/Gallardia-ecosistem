# 01 — Análisis estratégico

Antes de proponer una sola herramienta, este documento responde: ¿cuál es el problema real, dónde están los cuellos de botella, qué riesgos legales existen en Panamá, qué limitaciones técnicas son inamovibles hoy, cuánto cuesta operar esto y dónde está el verdadero retorno?

---

## 1. El problema real (no el aparente)

El pedido aparente es "automatizar marketing, ventas y contabilidad". El problema real de una PyME de servicios en Panamá es otro, y tiene cuatro capas:

1. **Capacidad fragmentada.** El dueño/operador es a la vez vendedor, marketero, cobrador y contador. Cada hora invertida en publicar un post es una hora no facturada. El costo de oportunidad es el verdadero gasto.
2. **Inconsistencia.** El marketing se hace "cuando hay tiempo" → el alcance muere entre ráfagas. Las ventas dependen de responder rápido → un lead de WhatsApp respondido a las 6 horas ya compró en otro lado. Estudios del sector (Lead Response Management, InsideSales) muestran que responder en <5 minutos multiplica por ~10–20x la probabilidad de contacto efectivo frente a responder a los 30 minutos.
3. **Ceguera financiera.** La facturación existe (porque la DGI lo obliga), pero no se transforma en información: nadie sabe el margen por servicio, el cliente más rentable ni el flujo de caja proyectado a 60 días.
4. **Memoria inexistente.** Las conversaciones con clientes viven en el teléfono personal. Si el teléfono se pierde o la persona se va, la empresa pierde su activo comercial.

**Conclusión de diseño:** el sistema no es "un bot que publica". Es una **memoria empresarial central** (datos) + **manos que ejecutan** (agentes) + **un cerebro que prioriza** (director). El orden de construcción correcto es datos → ventas → marketing → finanzas → dirección, porque ventas es donde el retorno es inmediato y medible.

---

## 2. Cuellos de botella identificados

| # | Cuello de botella | Síntoma | Cómo lo resuelve el sistema |
|---|------------------|---------|------------------------------|
| CB-1 | Tiempo de respuesta a leads | Leads de IG/WA respondidos horas después | Conserje omnicanal 24/7, primera respuesta < 60 segundos |
| CB-2 | Producción de contenido | Semanas sin publicar; luego 5 posts en un día | Pipeline foto→publicación + generador de contenido educativo de respaldo, cola editorial con 14 días de buffer |
| CB-3 | Cotización manual | Cada cotización toma 30–60 min y varía de formato | Motor de precios configurable + plantilla PDF; cotización en < 3 min |
| CB-4 | Seguimiento post-cotización | "Le envié la cotización y nunca más supe" | Secuencias de seguimiento automático (D+1, D+3, D+7, D+14) con detección de intención |
| CB-5 | Captura contable | Facturas en PDF/papel acumuladas a fin de mes | OCR + clasificación automática al recibirse; cierre diario, no mensual |
| CB-6 | Decisión estratégica | Decisiones por intuición, sin datos | Director General con acceso a todo el dataset, briefing semanal accionable |
| CB-7 | Dependencia de una persona | Todo está "en la cabeza" del dueño | Políticas, precios y tono de marca en configuración versionada; historial completo en CRM |

---

## 3. Riesgos legales y regulatorios (Panamá) — resumen

Detalle completo en [doc 06](06-contexto-panama.md). Lo que condiciona la arquitectura:

| Riesgo | Norma | Implicación de diseño |
|--------|-------|----------------------|
| Tratamiento de datos personales de leads/clientes | **Ley 81 de 2019** (protección de datos), Decreto Ejecutivo 285 de 2021, fiscalizada por ANTAI | Consentimiento registrado, política de privacidad publicada, derechos ARCO atendibles, datos almacenados con cifrado, registro de finalidad. Multas de hasta ~USD 10,000 por infracción. |
| Facturación electrónica | Sistema de Facturación Electrónica de Panamá (SFEP) vía **PAC** autorizado o Facturador Gratuito DGI | El sistema **no emite** facturas por su cuenta: se integra con el software de facturación autorizado existente. Solo lee/concilia. Emisión automática solo a través del API del PAC/software autorizado. |
| Declaraciones fiscales (ITBMS mensual, renta anual, municipio) | Código Fiscal, DGI e-Tax 2.0 | e-Tax 2.0 **no tiene API pública**: la presentación es humana (contador). El sistema prepara los borradores y números; un humano presenta. Compuerta obligatoria. |
| Mensajería comercial no solicitada | Políticas de Meta/WhatsApp + Ley 81 (consentimiento) | Solo mensajes a usuarios que iniciaron contacto u optaron-in; plantillas aprobadas por Meta para mensajes fuera de la ventana de 24 h. Nada de listas frías por WhatsApp: riesgo de baneo del número Y riesgo legal. |
| Publicidad engañosa / promesas de resultado | Ley 45 de 2007 (protección al consumidor, ACODECO) | El generador de contenido tiene guardrails: prohibido inventar cifras, testimonios o garantías; todo dato lleva fuente. |
| Contenido generado por IA con derechos de terceros | Derecho de autor (Ley 64 de 2012) | Solo material propio del cliente, stock licenciado o generado por IA con licencia comercial. Nada de imágenes "encontradas en Google". |

---

## 4. Limitaciones técnicas duras (no negociables hoy)

Estas son restricciones de las plataformas, no del diseño. Cualquiera que prometa lo contrario está vendiendo humo:

1. **WhatsApp**: la API oficial (WhatsApp Business Platform / Cloud API) solo permite iniciar conversaciones con **plantillas pre-aprobadas** y cobra por mensaje fuera de la ventana de servicio de 24 h. Las soluciones "no oficiales" (emulación de WhatsApp Web) violan los términos y provocan baneo del número — descartadas.
2. **Instagram/Facebook**: publicación vía API requiere cuenta Business/Creator vinculada a una página; límite ~50 publicaciones por cuenta cada 24 h (sobra para 1–3 diarias). Las historias y reels son publicables por API; algunos formatos (stickers interactivos, encuestas en historias) **no** lo son.
3. **TikTok**: el Content Posting API requiere aprobación/auditoría de la app; mientras no esté auditada, los posts quedan privados. Alternativa práctica: publicar TikTok mediante una plataforma ya auditada (Metricool, Postiz, Blotato) o dejar TikTok como el único paso semi-manual (notificación con el video listo + caption para publicar en 30 segundos desde el teléfono).
4. **DGI / e-Tax 2.0**: sin API pública. Los reportes tributarios se preparan automáticamente, se presentan manualmente.
5. **Bancos panameños** (Banco General, Banistmo, BAC): sin open banking / API pública para cuentas PyME. La conciliación bancaria se hace por importación de estados de cuenta (CSV/Excel) o lectura de notificaciones de Yappy/ACH por correo. Es un paso semi-automático honesto.
6. **Calidad generativa de video**: la IA hoy edita, corta, subtitula, mejora y ensambla video real de forma excelente; la *generación* de video fotorrealista de trabajos que no existen es detectable y dañaría la credibilidad — además de ser éticamente inaceptable para mostrar "trabajos realizados". El pipeline multimedia trabaja **sobre tu material real**.
7. **Los LLMs alucinan**. Por eso ningún número (precio, impuesto, estadística) sale del modelo: sale de tablas de configuración o de fuentes citadas, y el modelo solo redacta alrededor.

---

## 5. Análisis de costos operativos (drivers)

Los costos del sistema escalan con 4 drivers, en este orden:

1. **Conversaciones de WhatsApp** (~USD 0.01–0.07 por mensaje de plantilla según categoría; las respuestas dentro de la ventana de servicio son gratuitas). 500 leads/mes ≈ USD 15–50.
2. **Tokens de LLM** (Claude API). El conserje de ventas y el generador de contenido son los mayores consumidores. Con prompt caching y modelos escalonados (Haiku para clasificar, Sonnet para conversar y redactar, el modelo mayor solo para el Director y decisiones complejas): USD 30–150/mes en MVP, USD 200–600/mes en profesional.
3. **Procesamiento multimedia** (mejora de imagen, ensamblado de video, almacenamiento). USD 20–100/mes según volumen.
4. **SaaS de soporte** (programador de publicaciones, CRM, hosting). USD 50–300/mes según nivel.

El presupuesto completo por nivel está en [doc 07](07-herramientas-apis-costos.md).

---

## 6. Escalabilidad

- **Vertical (más volumen)**: la arquitectura por eventos escala linealmente; el cuello sería el costo de LLM, mitigable con caching, batching y modelos pequeños para tareas mecánicas.
- **Horizontal (más marcas/sucursales)**: el diseño es multi-tenant desde el esquema de datos (`tenant_id` en todas las tablas) — añadir una segunda marca es configuración, no re-desarrollo.
- **Funcional (más áreas)**: nuevos agentes se suman suscribiéndose al bus de eventos (ej.: agente de RRHH, agente de compras) sin tocar los existentes.

---

## 7. Dónde está el retorno (priorización)

| Iniciativa | Esfuerzo | Retorno | Plazo del retorno |
|-----------|----------|---------|-------------------|
| Conserje de ventas WhatsApp 24/7 | Medio | **Alto y directo** (leads que hoy se pierden) | Semanas |
| Cotizador automático | Bajo | Alto (velocidad de cierre) | Semanas |
| Publicación diaria multicanal | Medio | Medio-alto (compuesto en el tiempo) | Meses |
| Pipeline multimedia | Medio | Medio (calidad + volumen de contenido) | Meses |
| Finanzas automatizadas | Medio | Medio (mejores decisiones, cobranza) | Meses |
| Director General | Bajo (una vez existen los datos) | Alto (estrategia) | Meses |

**Decisión:** el cronograma ([doc 09](09-cronograma.md)) construye primero lo que paga el resto.

---

## 8. Qué es automatizable hoy: el mapa honesto

| Tarea | Automatización real alcanzable |
|-------|-------------------------------|
| Responder y calificar leads | 95% — escala a humano solo casos raros o cliente molesto |
| Cotizar con lista de precios | 90% — compuerta humana sobre umbral configurable |
| Publicar contenido diario | 90% — aprobación humana opcional (recomendada los primeros 60 días), TikTok puede requerir 30 seg manuales |
| Editar fotos/videos de trabajos | 85% — el sistema produce variantes; el ojo humano elige entre opciones al inicio |
| Investigar y crear contenido educativo citado | 85% — revisión humana de afirmaciones sensibles |
| Registrar y clasificar facturas | 90% — excepciones a revisión |
| Estados financieros | 95% — sobre datos ya clasificados |
| Conciliación bancaria | 60% — limitada por falta de APIs bancarias en Panamá |
| Presentar impuestos | 0% por diseño — lo prepara la máquina, lo presenta el contador |
| Cerrar una venta compleja / negociar | 50% — el agente lleva la venta hasta la mesa; el cierre grande es humano |

Este mapa define las compuertas humanas del sistema: **aprobación de contenido (opcional tras 60 días), descuentos fuera de política, cotizaciones sobre umbral, y cierres fiscales (permanente)**.
