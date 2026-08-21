# Gestión de Dominios

**Versión:** 0.1.0  
**Última actualización:** 2026-08-21

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
- Activación y desactivación de dominios por parte del distribuidor.
- Consulta de dominios por parte del cliente.

No incluye:

- Registro o administración de cuentas de correo.
- Clientes.
- Pagos.
- Facturación.
- Precios.
- Comisiones.
- Historial de renovaciones o expiraciones.
- Integraciones con proveedores de dominios.
- Consulta o administración de dominios por parte de SVR.

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
- La fecha de expiración no puede capturarse ni modificarse manualmente.
- La fecha de expiración se actualiza únicamente cuando se registra un pago exitoso conforme al proceso de pagos.
- El sistema conserva únicamente la fecha de expiración vigente y no mantiene un historial de expiraciones.
- El distribuidor define la cantidad máxima de cuentas de correo activas permitidas para el dominio.
- La cantidad máxima de cuentas de correo debe ser un número entero mayor que cero.
- Un dominio puede utilizar una cantidad de cuentas de correo menor que el límite definido.
- Si se modifica el límite, el nuevo valor no puede ser menor que la cantidad de cuentas de correo que se encuentren activas en ese momento.
- Solo el distribuidor puede modificar, activar o desactivar un dominio.
- Los dominios nunca se eliminan físicamente; únicamente pueden desactivarse.
- El cliente puede consultar sus dominios, pero no puede modificarlos, activarlos ni desactivarlos.
- En el alcance actual, SVR no puede consultar ni administrar los dominios.
- El vencimiento del dominio permitirá detener posteriormente el servicio de sus cuentas de correo.
- Los precios y cobros relacionados con la cantidad de cuentas de correo se definirán en un proceso independiente.

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

La fecha de expiración representa la vigencia del servicio del dominio. Su cálculo posterior dependerá del proceso de pagos y no forma parte de la administración manual del dominio.

Los procesos relacionados con cuentas de correo, renovaciones, pagos, facturación y acceso futuro de SVR deberán documentarse de forma independiente cuando sean requeridos por el negocio.
