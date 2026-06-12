# 05 — Área 3: Administración y Contabilidad

Objetivo: que cada dólar que entra y sale quede registrado, clasificado y convertido en información el mismo día — y que los problemas de caja se detecten semanas antes de que duelan.

> Marco legal y fiscal panameño aplicable: ver [doc 06](06-contexto-panama.md). Regla de oro del sistema: **prepara, calcula y alerta; no presenta declaraciones ni emite facturas fiscales por su cuenta.**

---

## 1. Ingesta de datos financieros

| Fuente | Mecanismo | Frecuencia |
|--------|-----------|------------|
| **Facturas emitidas** | API/exportación del software de facturación electrónica (PAC) existente — p. ej. Alegra, The Factory HKA, Webpos, o el Facturador Gratuito DGI vía exportes | Tiempo real o diaria |
| **Facturas recibidas** | Buzón de correo dedicado (compras@…) + foto por WhatsApp → OCR con LLM multimodal: RUC/DV del proveedor, fecha, ítems, ITBMS, total | Al recibirse |
| **Cobros** | Confirmaciones de Yappy/ACH (notificaciones por correo parseadas) + registro manual asistido ("cobré 500 de la factura 0231" por WhatsApp) | Al ocurrir |
| **Banco** | Importación de estado de cuenta CSV/Excel (los bancos panameños no ofrecen API a PyMEs) — semanal, 10 minutos humanos | Semanal |
| **Planilla/CSS** | Registro de la planilla mensual y cargas sociales como asiento recurrente | Mensual |

El **Capturador** valida cada documento (RUC con dígito verificador, cuadre ITBMS = 7% de la base gravada, duplicados) y lo deja en `invoices_issued` / `invoices_received`.

## 2. Clasificación contable

El **Clasificador** asigna cada movimiento a un plan de cuentas PyME pre-configurado (adaptable al del contador):

```
1xxx Activos        (caja, bancos, cuentas por cobrar, equipo, ITBMS crédito)
2xxx Pasivos        (proveedores, ITBMS débito por pagar, CSS por pagar, préstamos)
3xxx Patrimonio
4xxx Ingresos       (por línea de servicio — clave para margen por servicio)
5xxx Costos directos (materiales, mano de obra directa, subcontratos)
6xxx Gastos operativos (planilla admin, alquiler, combustible, marketing, software)
```

- Clasificación por reglas (proveedor conocido → cuenta) + LLM para casos nuevos, con confianza; bajo umbral → cola de revisión semanal (10–15 min del contador).
- Cada corrección humana se convierte en regla: el sistema converge a >95% automático en ~8 semanas.
- Doble partida derivada automáticamente en `ledger_entries` (factura emitida → CxC + ingreso + ITBMS débito; cobro → caja − CxC; etc.).

## 3. Estados e indicadores generados

Actualizados a diario, con cortes mensual/trimestral/anual:

1. **Estado de resultados:** ingresos por línea → **utilidad bruta** (− costos directos) → **utilidad operativa** (− gastos op.) → **utilidad neta** (− ISR estimado). Márgenes en % en cada nivel, comparados contra mes anterior y mismo mes del año previo.
2. **Flujo de caja:** real (entradas/salidas) + **proyección a 13 semanas**: cobros esperados por vencimiento de CxC × probabilidad histórica de pago del cliente, pagos comprometidos (proveedores, planilla, CSS, ITBMS), estacionalidad. Es el reporte más valioso del área.
3. **Balance general:** activos, pasivos y patrimonio desde el ledger.
4. **Reportes tributarios (borradores):** ITBMS mensual (débito − crédito = a pagar, listo para el formulario 430 que presenta el contador), resumen anual para la declaración de renta, base del impuesto municipal.
5. **Indicadores:** márgenes bruto/operativo/neto, DSO (días promedio de cobro), aging de cartera (0–30/31–60/61–90/+90), punto de equilibrio mensual, runway de caja, ingreso por empleado, **margen por servicio y por cliente** (alimenta al Director General).
6. **Proyecciones:** ingresos próximos 90 días = pipeline ponderado de ventas + recurrencias; escenarios optimista/base/pesimista.

## 4. Alertas del Centinela

| Alerta | Condición (configurable) | Acción |
|--------|--------------------------|--------|
| Factura por vencer | CxC vence en 3 días | Recordatorio amable al cliente (plantilla utility) + aviso interno |
| Cliente moroso | CxC vencida > 7 días | Escala: recordatorio → segundo aviso → tarea humana de cobranza; congela nuevas cotizaciones a ese cliente (configurable) |
| Caída de ingresos | Ingresos del mes < 80% del promedio móvil 3 meses al día N | Alerta al dueño + análisis del Director (¿leads? ¿conversión? ¿ticket?) |
| Estrés de caja | Proyección a 4 semanas < umbral (ej. USD 5,000) | Alerta crítica + lista de cobros acelerables y pagos diferibles |
| Gasto atípico | Gasto > 2σ de su categoría | Notificación con detalle |
| Obligaciones fiscales | Calendario DGI/CSS/municipio (ITBMS día 15, planilla, renta 31-mar) | Recordatorio al dueño y contador con el borrador adjunto |

## 5. Dashboards ejecutivos

Metabase (self-hosted, gratis) sobre la BD central, con SSO y roles:

- **Panel CEO:** caja hoy, proyección 13 semanas, ventas del mes vs. meta, margen neto, top 5 clientes por rentabilidad, alertas activas.
- **Panel Ventas:** embudo, conversión por canal, pipeline ponderado, actividad del conserje.
- **Panel Marketing:** ingreso atribuido por pilar/campaña, alcance/interacción, mejores piezas.
- **Panel Finanzas (contador):** estados completos, aging, borradores tributarios, cola de revisión.

Los mismos datos son consultables en lenguaje natural vía el Director General por WhatsApp: "¿cómo va el flujo de caja?" responde con cifras del dashboard, no con estimaciones del modelo.
