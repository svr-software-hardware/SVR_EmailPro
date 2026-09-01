# Facturación de Pagos

**Versión:** 0.2.0
**Última actualización:** 2026-09-01

---

# Objetivo

Definir las reglas de negocio para generar y consultar la factura correspondiente a un pago exitoso de EmailPro.

---

# Alcance

Incluye:

- Intento automático de facturación después de registrar un pago exitoso.
- Generación de la factura mediante el proveedor de facturación integrado.
- Utilización de la información fiscal vigente del cliente.
- Conservación del estado, identificador externo, UUID y fecha de facturación en el pago.
- Conservación del último error de facturación para diagnóstico y reintento.
- Reintento manual de facturación por parte del cliente o del distribuidor.
- Consulta de la factura por parte del cliente y del distribuidor.
- Obtención de los documentos de la factura mediante el proveedor de facturación.
- Generación de una sola factura de SVR al cliente por el importe total pagado.
- Manejo del precio facturado como importe con IVA incluido.

No incluye:

- Historial de intentos de facturación.
- Cancelación de facturas.
- Sustitución de facturas.
- Refacturación.
- Almacenamiento local de archivos PDF o XML.
- Definición técnica de impuestos.
- Implementación del proveedor de facturación.
- Consulta o administración de facturas por parte de SVR.
- Facturación de SVR hacia el distribuidor.

---

# Reglas de Negocio

- Únicamente un pago exitoso puede facturarse.
- Cada pago puede tener como máximo una factura asociada.
- Un pago conserva un estado de facturación pendiente, completado o fallido.
- Un pago se considera facturado cuando su estado es completado y conserva un identificador externo de factura.
- Después de registrar un pago exitoso y actualizar la vigencia del dominio, el sistema intenta generar la factura automáticamente.
- El registro del pago y la actualización de la vigencia deben completarse antes de iniciar la facturación.
- Un error de facturación no revierte el pago ni modifica la fecha de expiración actualizada del dominio.
- Si el intento automático falla, el pago conserva el estado fallido y el último mensaje de error.
- Mientras la facturación no esté completada, el cliente puede volver a intentar la facturación.
- Mientras la facturación no esté completada, el distribuidor responsable del cliente puede volver a intentar la facturación.
- Un pago que ya conserva un identificador externo de factura no puede facturarse nuevamente.
- El identificador externo se registra únicamente después de que el proveedor confirma la creación de la factura.
- Una vez registrado, el identificador externo de factura no puede modificarse ni reemplazarse.
- Únicamente se conserva el último error de facturación; no se almacena un historial completo de intentos.
- No se conserva un historial de intentos exitosos o fallidos.

---

# Información Fiscal

- La factura utiliza el perfil fiscal vigente del cliente asociado al dominio del pago.
- El perfil fiscal debe existir y contener todos sus datos obligatorios antes de generar la factura.
- La factura utiliza el régimen fiscal vigente del perfil.
- La factura utiliza el uso CFDI predeterminado vigente del perfil.
- Si la información fiscal está incompleta o no es válida, la factura no se genera y el pago permanece pendiente de facturar.
- Un reintento utiliza la información fiscal vigente en el momento en que se realiza.

---

# Información del Pago

- La factura utiliza el importe total histórico conservado en el pago.
- El importe histórico ya incluye IVA y no debe incrementarse al formar la factura.
- La factura utiliza la cantidad de cuentas de correo cobrada en el pago.
- La factura utiliza el precio unitario aplicado al cliente en el pago.
- La factura utiliza el periodo y la cantidad de meses conservados en el pago.
- Los importes no se recalculan utilizando el precio, el costo o la capacidad actuales del dominio.
- Los cambios posteriores en la configuración del dominio no modifican la factura ni los valores históricos del pago.
- Las claves de producto o servicio, unidad, tipo y tasa de impuesto, forma de pago y método de pago son configurables.
- Durante la integración inicial pueden utilizarse claves fiscales provisionales; deberán sustituirse por las claves definitivas sin modificar los pagos históricos.

---

# Consulta de la Factura

- El cliente puede consultar las facturas correspondientes a los pagos de sus dominios.
- El distribuidor puede consultar las facturas correspondientes a los pagos de los dominios pertenecientes a sus clientes.
- En el alcance actual, SVR no puede consultar ni administrar facturas.
- Los documentos de la factura se obtienen mediante el proveedor utilizando el identificador externo conservado en el pago.
- Los archivos PDF y XML no se almacenan en la base de datos ni en el almacenamiento de EmailPro.

---

# Flujo Automático

```text
Pago exitoso registrado
    ↓
Vigencia del dominio actualizada
    ↓
Obtener perfil fiscal vigente del cliente
    ↓
Generar factura mediante el proveedor
    ↓
Proveedor confirma la factura
    ↓
Guardar identificador externo en el pago
    ↓
Guardar UUID y fecha de facturación
```

Si el proveedor rechaza la operación o la información fiscal no es válida, el pago conserva su identificador externo de factura nulo, cambia a estado fallido y guarda el último error.

---

# Flujo de Reintento

```text
Cliente o distribuidor
    ↓
Seleccionar pago pendiente de facturar
    ↓
Obtener perfil fiscal vigente del cliente
    ↓
Generar factura mediante el proveedor
    ↓
Proveedor confirma la factura
    ↓
Guardar identificador externo en el pago
```

---

# Entidades

- Clients
- Domains
- Fiscal Profiles
- Fiscal Regimes
- CFDI Usages
- Payments

---

# Observaciones

La información mínima de la factura se conserva directamente en Payments porque la versión actual requiere conocer su estado, diagnosticar el último fallo y recuperar sus documentos externos.

No se crea una entidad local de facturas ni una entidad de intentos de facturación porque actualmente no existe una necesidad funcional que justifique su almacenamiento.

La integración técnica, la formación de solicitudes y el manejo de respuestas del proveedor pertenecen a la librería y a la implementación de la API.

La factura documentada en este proceso corresponde exclusivamente a SVR hacia el cliente. La facturación de SVR hacia el distribuidor se definirá posteriormente como un proceso independiente.

Los procesos relacionados con cancelaciones, sustituciones, refacturación, historial de intentos y acceso futuro de SVR deberán documentarse de forma independiente cuando sean requeridos por el negocio.
