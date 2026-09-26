# Facturación de Pagos

**Versión:** 1.0.0  
**Última actualización:** 2026-09-25

---

# Objetivo

Definir las facturas y recibos que se generan después de un pago exitoso de EmailPro.

Cada pago origina documentación fiscal en dos direcciones:

- SVR documenta el importe pagado por el cliente.
- El distribuidor factura a SVR el importe que le corresponde por ese pago.

---

# Alcance

Incluye:

- Factura individual de SVR al cliente cuando este la solicita.
- Recibo electrónico cuando el cliente no solicita factura individual.
- Factura global diaria de SVR a público en general.
- Factura automática del distribuidor hacia SVR por cada pago.
- Conservación independiente de cada CFDI.
- Asociación de una factura con uno o múltiples pagos.
- Estados de facturación y último error de cada factura o recibo.
- Reintentos sin revertir el pago ni la vigencia.
- Consulta de documentos mediante URLs firmadas y temporales.

No incluye:

- Historial detallado de intentos de facturación.
- Cancelación, sustitución o refacturación de CFDI.
- Almacenamiento local de archivos PDF o XML.
- Definición técnica de impuestos o catálogos fiscales.
- Transferencias o liquidaciones bancarias a distribuidores.
- Generación del archivo bancario para BBVA.

---

# Principios

- Únicamente un pago exitoso puede iniciar estos procesos.
- Un error de facturación no revierte el pago ni la vigencia otorgada al dominio.
- Los importes se obtienen de los valores históricos conservados en el pago.
- Los importes no se recalculan utilizando precios, costos o capacidades actuales.
- Los CFDI, recibos y errores se conservan separados del registro principal del pago.
- Los archivos PDF y XML permanecen en el proveedor de facturación.
- EmailPro no almacena Certificados de Sello Digital, llaves privadas ni sus contraseñas.

---

# Tipos de Factura

La entidad `invoices` utiliza los siguientes tipos:

- `svr_to_customer`: factura individual emitida por SVR al cliente.
- `svr_global`: factura global diaria emitida por SVR a público en general.
- `distributor_to_svr`: factura emitida por el distribuidor a SVR.

Una factura individual se relaciona con un pago.

Una factura del distribuidor se relaciona con un pago.

Una factura global puede relacionarse con múltiples pagos mediante `invoice_payments`.

---

# Estados de Factura

Los estados permitidos son:

- `pending`: todavía no se ha confirmado la creación del CFDI.
- `completed`: el proveedor confirmó el CFDI.
- `failed`: el último intento no pudo completarse.

Una factura completada conserva su identificador externo, UUID y fecha de emisión.

Una factura completada no puede generarse nuevamente.

Únicamente se conserva el último error; no se crea un historial técnico de intentos.

---

# Selección del Cliente

- El pago conserva la selección `requires_invoice` realizada antes del cobro.
- La selección no puede modificarse después de registrar el pago exitoso.
- Cuando `requires_invoice` es `true`, SVR genera una factura individual al cliente.
- Cuando `requires_invoice` es `false`, SVR genera un recibo electrónico para incorporar la operación a una factura global diaria.
- En ambos casos, el importe documentado por SVR es `payments.gross_amount`.

---

# Factura Individual de SVR al Cliente

- La factura utiliza el perfil fiscal vigente del cliente asociado al dominio.
- El perfil fiscal debe existir y contener todos sus datos obligatorios.
- La factura utiliza el régimen fiscal y el uso CFDI vigentes al momento del intento.
- Si la información fiscal está incompleta o no es válida, la factura permanece fallida y puede reintentarse.
- Un reintento utiliza la información fiscal vigente en ese momento.
- La factura conserva el importe total histórico pagado por el cliente.
- La factura utiliza la cantidad de cuentas, precio unitario, periodo y meses conservados en el pago.
- El importe histórico ya incluye IVA y no se incrementa nuevamente.
- Cada pago puede asociarse como máximo a una factura de tipo `svr_to_customer`.

---

# Recibos de Público en General

- Cada pago con `requires_invoice` igual a `false` genera un registro en `payment_receipts`.
- Cada pago puede tener como máximo un recibo.
- El recibo utiliza `payments.gross_amount`.
- El recibo conserva el identificador asignado por el proveedor.
- Un recibo pendiente o fallido puede reintentarse sin volver a cobrar al cliente.
- Un recibo abierto todavía no representa el CFDI global definitivo.
- Después de incorporarse a una factura global, el recibo cambia a estado `invoiced_globally`.

Los estados locales del recibo son:

- `pending`.
- `open`.
- `invoiced_globally`.
- `failed`.

---

# Factura Global Diaria

- SVR genera una factura global diaria con los recibos abiertos correspondientes al periodo.
- La factura se emite a público en general.
- La fecha del periodo se conserva en la factura.
- Al confirmar la factura global, cada pago incluido se relaciona con ella mediante `invoice_payments`.
- Los recibos incluidos cambian a `invoiced_globally`.
- La suma de `gross_amount` de los pagos relacionados debe corresponder al importe de la factura global.
- Un fallo en la factura global conserva abiertos los recibos para permitir un reintento.
- La generación debe ser idempotente y no puede incluir dos veces el mismo pago.
- Los pagos de clientes que solicitaron factura individual no se incorporan a la factura global.

---

# Factura del Distribuidor hacia SVR

- Después de completar la documentación fiscal de SVR correspondiente al pago, el sistema genera automáticamente una factura del distribuidor hacia SVR.
- No se requiere confirmación manual del distribuidor.
- Se genera una factura independiente por cada pago.
- El emisor es la organización de facturación del distribuidor responsable del dominio.
- El receptor es SVR.
- El importe es exactamente `payments.distributor_amount`.
- Cada pago puede asociarse como máximo a una factura de tipo `distributor_to_svr`.
- La organización del distribuidor debe estar lista para emitir CFDI en producción.
- Si la factura del distribuidor falla, el pago y la documentación emitida por SVR permanecen válidos.
- La factura fallida puede reintentarse sin volver a cobrar al cliente ni duplicar la factura de SVR.
- Una factura del distribuidor completada será requisito para el proceso posterior de liquidación.

Para una factura individual solicitada por el cliente, la factura del distribuidor puede generarse después de completar `svr_to_customer`.

Para una operación de público en general, la factura del distribuidor se genera después de que el pago quede incluido en la factura `svr_global` correspondiente.

---

# Organización del Distribuidor

- El identificador externo de la organización se obtiene de `distributors.invoice_organization_id`.
- La API debe validar mediante la librería que la organización continúa lista para facturar.
- EmailPro no almacena los archivos `.cer`, `.key` ni su contraseña.
- La librería utiliza la organización para emitir el CFDI en nombre del distribuidor.
- Los datos fiscales de la organización deben corresponder al perfil fiscal vigente del distribuidor.

---

# Asociación entre Facturas y Pagos

- La relación se conserva en `invoice_payments`.
- Una factura puede incluir uno o múltiples pagos.
- Un pago puede asociarse a más de una factura porque documenta operaciones en sentidos distintos: la emitida por SVR al cliente y la emitida por el distribuidor a SVR.
- La combinación de factura y pago debe ser única.
- Un pago no puede asociarse a dos facturas completadas del mismo tipo.
- La relación no modifica los importes históricos del pago.

---

# Consulta de Documentos

- El cliente puede consultar la factura individual de sus pagos cuando la solicitó.
- El cliente no consulta la factura emitida por el distribuidor hacia SVR.
- El distribuidor puede consultar las facturas individuales correspondientes a los pagos de sus clientes.
- El distribuidor puede consultar las facturas que su organización emitió hacia SVR.
- SVR puede consultar las facturas relacionadas con los pagos y distribuidores.
- Los documentos se obtienen del proveedor mediante el identificador externo de la factura.
- La solicitud autenticada genera una URL firmada y temporal para PDF o XML.
- La URL puede abrirse en una pestaña nueva sin exponer el token de autenticación.
- El documento se entrega en modo de visualización directa mediante su tipo de contenido correspondiente.
- La URL deja de ser válida al concluir el periodo configurado.

---

# Flujo con Factura Individual

```text
Pago exitoso con requires_invoice = true
    ↓
Actualizar vigencia del dominio
    ↓
Crear factura svr_to_customer
    ↓
Proveedor confirma el CFDI
    ↓
Crear factura distributor_to_svr por distributor_amount
    ↓
Organización del distribuidor emite el CFDI a SVR
    ↓
Facturación del pago completada
```

---

# Flujo con Público en General

```text
Pago exitoso con requires_invoice = false
    ↓
Actualizar vigencia del dominio
    ↓
Crear recibo electrónico por gross_amount
    ↓
Conservar recibo abierto
    ↓
Cierre diario
    ↓
Crear factura svr_global con recibos abiertos
    ↓
Relacionar factura global con sus pagos
    ↓
Crear una factura distributor_to_svr por cada pago incluido
    ↓
Organizaciones de distribuidores emiten sus CFDI a SVR
```

---

# Recuperación de Errores

- Cada factura y recibo conserva su propio estado y último error.
- Un reintento utiliza el mismo registro local.
- Antes de reintentar, la implementación debe consultar el proveedor cuando exista la posibilidad de que la respuesta anterior se haya perdido.
- Un reintento no puede crear un CFDI duplicado para el mismo tipo y pago.
- Los errores de la factura del distribuidor no bloquean el acceso del cliente al servicio ya pagado.
- Los errores pendientes sí pueden impedir la liquidación posterior al distribuidor.

---

# Entidades

- Distributors
- Clients
- Domains
- Fiscal Profiles
- Payments
- Payment Receipts
- Invoices
- Invoice Payments

---

# Observaciones

Las claves de producto o servicio, unidad, impuestos, forma de pago y método de pago pertenecen a la configuración de la integración fiscal.

La factura global diaria permite consolidar operaciones de público en general sin perder la relación entre cada pago y su recibo.

El proceso bancario que agrupe o liquide cantidades a distribuidores será independiente. La existencia de una factura `distributor_to_svr` no confirma que el importe ya haya sido transferido.
