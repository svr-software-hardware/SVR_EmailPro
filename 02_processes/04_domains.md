# Gestión de Dominios

**Versión:** 0.4.0  
**Última actualización:** 2026-09-25

---

# Objetivo

Definir las reglas de negocio para el registro y administración de los dominios asociados a los clientes de EmailPro.

---

# Alcance

Incluye:

- Registro de dominios por parte del distribuidor responsable del cliente.
- Asociación de múltiples dominios a un cliente.
- Selección de la extensión del dominio.
- Inicialización automática de la fecha de expiración del dominio.
- Definición de la cantidad máxima de cuentas de correo activas permitidas para el dominio.
- Definición y actualización del precio mensual por cuenta de correo cobrado al cliente.
- Activación y desactivación de dominios por parte del distribuidor.
- Consulta de dominios por parte del cliente.
- Consulta de dominios por parte de SVR.
- Modificación administrativa del nombre y la vigencia del dominio por parte de SVR.
- Presentación del estado de vencimiento en los listados de SVR, distribuidores y clientes.

No incluye:

- Registro o administración de cuentas de correo.
- Clientes.
- Pagos.
- Facturación.
- Comisiones.
- Historial de renovaciones o expiraciones.
- Integraciones con proveedores de dominios.
- Modificación de la extensión, capacidad, precio o estado del dominio por parte de SVR.

---

# Reglas de Negocio

- Solo un distribuidor puede registrar dominios.
- El distribuidor únicamente puede registrar y administrar dominios para los clientes que le pertenecen.
- Un dominio puede registrarse inmediatamente después de crear al cliente, aunque su cuenta de usuario todavía no haya sido confirmada.
- Cada dominio pertenece a un único cliente.
- Un cliente puede tener múltiples dominios.
- El dominio se compone de un nombre y una extensión.
- La extensión debe seleccionarse del catálogo de extensiones de dominio activas.
- La combinación del nombre y la extensión debe ser única en EmailPro.
- Al crear un dominio, el sistema establece su fecha de expiración con la fecha de creación.
- La fecha de expiración no puede ser capturada ni modificada manualmente por el distribuidor o el cliente.
- La fecha de expiración se actualiza cuando se registra un pago exitoso conforme al proceso de pagos o cuando SVR realiza un ajuste administrativo.
- El sistema conserva únicamente la fecha de expiración vigente y no mantiene un historial de expiraciones.
- El distribuidor define la cantidad máxima de cuentas de correo activas permitidas para el dominio.
- La cantidad máxima de cuentas de correo debe ser un número entero mayor que cero.
- Un dominio puede utilizar una cantidad de cuentas de correo menor que el límite definido.
- Si se modifica el límite, el nuevo valor no puede ser menor que la cantidad de cuentas de correo que se encuentren activas en ese momento.
- El distribuidor define el precio mensual por cuenta de correo para cada dominio.
- El precio de un dominio debe ser igual o mayor que el costo vigente asignado al distribuidor.
- El distribuidor puede vender al cliente sin margen de ganancia, pero nunca por debajo de su costo vigente.
- El distribuidor puede actualizar el precio del dominio.
- Los cambios en el precio aplican únicamente a pagos futuros y no modifican pagos existentes.
- El cliente puede consultar el precio de sus dominios, pero no puede modificarlo.
- Si el costo de SVR aumenta y el precio del dominio queda por debajo de ese costo, el dominio no puede recibir nuevos pagos hasta que el distribuidor actualice su precio.
- Después de registrar el dominio, el distribuidor no puede modificar su nombre ni su extensión.
- El distribuidor puede modificar la capacidad de cuentas de correo y el precio asignado al dominio.
- Solo el distribuidor puede activar o desactivar un dominio.
- Los dominios nunca se eliminan físicamente; únicamente pueden desactivarse.
- El cliente puede consultar sus dominios, pero no puede modificarlos, activarlos ni desactivarlos.
- SVR puede modificar únicamente el nombre y la fecha de expiración de un dominio.
- SVR no puede modificar la extensión, capacidad, precio o estado activo del dominio.
- El vencimiento del dominio permitirá detener posteriormente el servicio de sus cuentas de correo.
- Los precios y cobros relacionados con la cantidad de cuentas de correo se definirán en un proceso independiente.

---

# Estado de Vencimiento

El estado de vencimiento se calcula a partir de `expires_at` y de la fecha actual.

Los valores entregados por la API son:

- `expired`: la fecha de expiración es anterior a la fecha actual.
- `expiring`: la fecha de expiración se encuentra entre la fecha actual y los próximos siete días, inclusive.
- `no expired`: la fecha de expiración es posterior al periodo de siete días.

El estado es información derivada y no se almacena en la entidad `domains`.

Los listados de dominios del cliente, del distribuidor y de SVR deben entregar el mismo estado utilizando estas reglas.

El frontend puede representar el estado mediante texto, color o un indicador visual. La presentación visual no modifica los valores entregados por la API.

---

# Listado Administrativo

- El listado administrativo es una consulta exclusiva de SVR.
- Los dominios se ordenan por fecha de expiración ascendente.
- Los dominios más vencidos se presentan primero y los de vencimiento más lejano se presentan al final.
- Para cada registro se presenta el nombre completo del dominio, formado por su nombre y extensión.
- Para cada registro se presenta la fecha de expiración.
- Para cada registro se presenta el estado de vencimiento calculado.
- Para cada registro se presenta el costo por cuenta de correo vigente asignado por SVR al distribuidor responsable.
- Para cada registro se presenta el nombre del distribuidor responsable.
- El costo administrativo se obtiene de `distributors.mailbox_unit_cost`.
- El listado administrativo no presenta `domains.mailbox_unit_price`, correspondiente al precio que el distribuidor cobra a su cliente.
- Desde el listado administrativo, SVR puede corregir el nombre del dominio y su fecha de expiración.
- La extensión del dominio no forma parte de la edición administrativa.

---

# Flujo Principal

```text
Distribuidor
    ↓
Seleccionar uno de sus clientes
    ↓
Registrar nombre y extensión del dominio
    ↓
Inicializar fecha de expiración con la fecha de creación
    ↓
Definir límite de cuentas de correo
    ↓
Definir precio mensual por cuenta de correo
    ↓
Dominio disponible para administrar cuentas de correo
```

---

# Entidades

- Clients
- Domain Extensions
- Domains

---

# Observaciones

El nombre del dominio representa la parte que antecede a su extensión. Por ejemplo, para `empresa.com.mx`, el nombre es `empresa` y la extensión es `com.mx`.

El límite de cuentas de correo representa la capacidad activa permitida para el dominio. Las reglas para registrar y administrar dichas cuentas se definen en un proceso independiente.

El precio por cuenta pertenece al dominio porque un mismo cliente puede tener precios diferentes en cada uno de sus dominios.

La fecha de expiración representa la vigencia del servicio del dominio. Normalmente se actualiza mediante el proceso de pagos, pero SVR puede corregirla administrativamente cuando sea necesario.

Los procesos relacionados con suscripciones, facturación y acceso futuro de SVR deberán documentarse de forma independiente cuando sean requeridos por el negocio.
