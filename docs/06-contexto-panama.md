# 06 — Contexto Panamá: legal, fiscal y operativo

Todo lo que condiciona el diseño por operar en Panamá. **Nota:** esto es análisis de diseño de sistemas, no asesoría legal/fiscal; las cifras y obligaciones deben validarse con el contador/abogado de la empresa, y el sistema está diseñado precisamente para que esa validación humana sea rápida.

---

## 1. Facturación

- Panamá opera el **Sistema de Facturación Electrónica de Panamá (SFEP)** administrado por la DGI. La emisión válida se hace mediante un **PAC** (Proveedor de Autorización Calificado — p. ej. The Factory HKA, Webpos, entre otros autorizados) o mediante el **Facturador Gratuito de la DGI** para contribuyentes que califican. Los equipos fiscales tradicionales están en proceso de reemplazo por FE.
- **Implicación de arquitectura:** el sistema **no genera facturas fiscales por su cuenta**. Se integra con el software de facturación existente (lectura vía API/exportación para contabilidad; emisión, si se automatiza, solo a través del API oficial del PAC/software autorizado). Las cotizaciones y proformas sí las genera libremente porque no son documentos fiscales.
- Datos obligatorios que el sistema valida en cada factura: RUC y DV, CUFE (código único de factura electrónica), desglose de ITBMS, fecha de autorización.

## 2. Impuestos que el sistema modela

| Impuesto | Regla general | Cómo lo maneja el sistema |
|----------|---------------|---------------------------|
| **ITBMS** | 7% general (10% alojamiento y bebidas alcohólicas, 15% tabaco). Declaración mensual (form. 430) dentro de los 15 días del mes siguiente. Exentos contribuyentes con ingresos brutos < USD 36,000/año | El Cotizador lo calcula por tipo de servicio; Finanzas lleva débito/crédito fiscal y prepara el borrador mensual; recordatorio el día 10 |
| **ISR persona jurídica** | 25% sobre renta neta gravable. CAIR (cálculo alterno) aplica a ingresos gravables > USD 1.5M. Declaración jurada anual (31 de marzo, prorrogable) | Provisión mensual estimada en el estado de resultados (utilidad neta realista); paquete anual de soporte para el contador |
| **Dividendos / complementario** | 10% dividendos de fuente panameña (5% en ciertos casos); impuesto complementario 4% si no se distribuyen | Alerta informativa al cierre fiscal; decisión del contador |
| **Impuesto municipal** | Mensual, según actividad y municipio (tablas distritales) | Asiento recurrente + recordatorio |
| **Aviso de Operación** | 2% anual sobre el capital de la empresa (mín. USD 100, máx. USD 60,000); registro en PanamáEmprende | Recordatorio anual |
| **Tasa Única** | USD 300/año para sociedades anónimas y SRL | Recordatorio anual (multas por atraso) |
| **CSS / planilla** | Cuota patronal ~12.25% + obrera 9.75%; décimo tercer mes en 3 partidas (abr/ago/dic); SIPE para planillas | Asientos recurrentes + calendario de partidas del décimo |

**Moneda:** Panamá usa USD (balboa a la par) — sin riesgo cambiario ni módulo multi-moneda en MVP; el esquema deja `currency` por si se expande a la región.

## 3. Protección de datos — Ley 81 de 2019

Vigente desde 2021, reglamentada por el Decreto Ejecutivo 285 de 2021, fiscalizada por **ANTAI**. Es la norma que más toca a un sistema que conversa con clientes y guarda historiales:

- **Consentimiento y finalidad:** los datos de leads se usan para atención comercial (base: relación precontractual iniciada por el titular). Para marketing posterior (newsletter de WhatsApp, remarketing) se requiere **opt-in explícito registrado** (`consentimiento_at` + texto aceptado).
- **Derechos ARCO + portabilidad:** el sistema implementa flujo de solicitud: acceso (exportar lo que tenemos del titular), rectificación, cancelación (borrado con excepción de lo exigido por ley fiscal), oposición.
- **Medidas de seguridad:** cifrado en tránsito y reposo, acceso por roles, registro de accesos — ya parte de la arquitectura ([doc 02 §6](02-arquitectura.md)).
- **Transferencia internacional:** los datos viven en proveedores cloud fuera de Panamá (AWS/Cloudflare/Anthropic) — la política de privacidad debe declararlo y el contrato con proveedores debe garantizar nivel adecuado de protección (DPAs estándar de cada proveedor).
- **Sanciones:** multas administrativas (hasta ~USD 10,000) más el daño reputacional.
- **Entregable del proyecto:** política de privacidad publicada en la web + aviso corto en el primer mensaje del bot ("tus datos se usan para atenderte; ver política: <link>").

Complementos: Ley 51 de 2008 (documentos y firmas electrónicas — las cotizaciones aceptadas por WhatsApp tienen valor probatorio razonable; contratos grandes mejor con firma), Ley 45 de 2007 (protección al consumidor/ACODECO — publicidad veraz, precios claros con ITBMS, respetar cotizaciones vigentes).

## 4. Pagos y cobros en Panamá

| Medio | Integración | Uso en el sistema |
|-------|-------------|-------------------|
| **Yappy Comercial** (Banco General) | API / Botón de Pago oficial | Cobro de visita técnica USD 25 y abonos: el agente envía link de pago; confirmación por webhook/correo concilia automático |
| **ACH / transferencia** | Sin API; confirmación por comprobante | El cliente envía captura → OCR la valida contra CxC |
| **Tarjetas** | Pasarelas que operan en Panamá: Tilopay, PayPal, ePago/Banesco | Nivel profesional: link de pago en la cotización |
| **Efectivo/cheque** | Registro manual asistido por WhatsApp | "Cobré 500 de la factura 0231" → asiento |

Stripe **no** opera para cuentas panameñas — descartado; Tilopay/Yappy cubren el caso local mejor y más barato.

## 5. Operación local — checklist administrativo que el sistema vigila

- Aviso de Operación vigente (PanamáEmprende) ✔ recordatorio anual
- RUC activo y NIT de e-Tax para el contador ✔ datos en config
- Declaración ITBMS mensual (si aplica) ✔ borrador + recordatorio día 10
- Planilla CSS (SIPE) mensual ✔ recordatorio
- Décimo tercer mes (15-abr / 15-ago / 15-dic) ✔ provisión mensual automática en el flujo de caja proyectado
- Renta anual (31-mar) ✔ paquete anual para el contador desde enero
- Municipio mensual ✔ asiento recurrente
- Tasa Única (S.A./SRL) ✔ recordatorio
- Renovaciones de idoneidades/permisos del rubro ✔ tabla de vencimientos configurable

## 6. Integraciones viables en Panamá — resumen de realidad

| Categoría | Viable hoy | No viable / workaround |
|-----------|-----------|------------------------|
| Redes sociales | Todas las APIs de Meta, TikTok, LinkedIn, Google funcionan igual que en cualquier país | — |
| WhatsApp Business API | Sí, número panameño +507 sin problema (vía Meta/BSP como 360dialog/Twilio) | — |
| Facturación | API del PAC / software (Alegra, HKA, Webpos) | DGI e-Tax sin API → contador presenta |
| Bancos | Yappy Comercial (API) | Cuentas bancarias sin API → importación CSV semanal |
| Pagos online | Yappy, Tilopay, PayPal | Stripe no disponible |
| Contabilidad SaaS | Alegra (localizado Panamá), QuickBooks, Odoo | — |
| IA (Claude API, etc.) | Sin restricción | — |
