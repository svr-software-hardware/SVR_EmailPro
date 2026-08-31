# Facturación de Pagos

**Versión:** 0.1.0  
**Última actualización:** 2026-08-31

---

# Objetivo

Definir las reglas de negocio para generar y consultar la factura correspondiente a un pago exitoso de EmailPro.

---

# Alcance

Incluye:

- Intento automático de facturación después de registrar un pago exitoso.
- Generación de la factura mediante el proveedor de facturación integrado.
- Utilización de la información fiscal vigente del cliente.
- Conservación del identificador externo de la factura en el pago.
- Reintento manual de facturación por parte del cliente o del distribuidor.
- Consulta de la factura por parte del cliente y del distribuidor.
- Obtención de los documentos de la factura mediante el proveedor de facturación.

No incluye:

- Historial de intentos de facturación.
- Almacenamiento de mensajes de error.
- Cancelación de facturas.
- Sustitución de facturas.
- Refacturación.
- Almacenamiento local de archivos PDF o XML.
- Definición técnica de impuestos.
- Implementación del proveedor de facturación.
- Consulta o administración de facturas por parte de SVR.

---

# Reglas de Negocio

- Únicamente un pago exitoso puede facturarse.
- Cada pago puede tener como máximo una factura asociada.
- Un pago se considera pendiente de facturar cuando su identificador externo de factura es nulo.
- Un pago se considera facturado cuando conserva un identificador externo de factura.
- Después de registrar un pago exitoso y actualizar la vigencia del dominio, el sistema intenta generar la factura automáticamente.
- El registro del pago y la actualización de la vigencia deben completarse antes de iniciar la facturación.
- Un error de facturación no revierte el pago ni modifica la fecha de expiración actualizada del dominio.
- Si el intento automático falla, el pago permanece pendiente de facturar.
- Mientras el pago permanezca pendiente, el cliente puede volver a intentar la facturación.
- Mientras el pago permanezca pendiente, el distribuidor responsable del cliente puede volver a intentar la facturación.
- Un pago que ya conserva un identificador externo de factura no puede facturarse nuevamente.
- El identificador externo se registra únicamente después de que el proveedor confirma la creación de la factura.
- Una vez registrado, el identificador externo de factura no puede modificarse ni reemplazarse.
- Los errores se muestran durante el intento, pero no se almacenan en la base de datos.
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
- La factura utiliza la cantidad de cuentas de correo cobrada en el pago.
- La factura utiliza el precio unitario aplicado al cliente en el pago.
- La factura utiliza el periodo y la cantidad de meses conservados en el pago.
- Los importes no se recalculan utilizando el precio, el costo o la capacidad actuales del dominio.
- Los cambios posteriores en la configuración del dominio no modifican la factura ni los valores históricos del pago.

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
```

Si el proveedor rechaza la operación o la información fiscal no es válida, el pago conserva su identificador externo de factura nulo.

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

El identificador de factura se conserva directamente en Payments porque la versión actual requiere únicamente conocer si un pago está pendiente o facturado y recuperar sus documentos externos.

No se crea una entidad local de facturas ni una entidad de intentos de facturación porque actualmente no existe una necesidad funcional que justifique su almacenamiento.

La integración técnica, la formación de solicitudes y el manejo de respuestas del proveedor pertenecen a la librería y a la implementación de la API.

Los procesos relacionados con cancelaciones, sustituciones, refacturación, historial de intentos y acceso futuro de SVR deberán documentarse de forma independiente cuando sean requeridos por el negocio.
