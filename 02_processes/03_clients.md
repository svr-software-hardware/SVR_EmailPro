# Gestión de Clientes

**Versión:** 0.1.0  
**Última actualización:** 2026-08-20

---

# Objetivo

Definir las reglas de negocio para el registro y administración de los clientes de los distribuidores autorizados de EmailPro.

---

# Alcance

Incluye:

- Registro de clientes por parte de un distribuidor.
- Creación de la cuenta de usuario asociada al cliente.
- Asignación automática del rol Cliente.
- Confirmación inicial de la cuenta del cliente.
- Consulta de la información del cliente por parte de su distribuidor.
- Captura y actualización de la información fiscal por parte del cliente.
- Administración del uso CFDI predeterminado.

No incluye:

- Distribuidores.
- Autenticación.
- Recuperación de contraseña.
- Dominios.
- Cuentas de correo electrónico.
- Pagos.
- Facturación.
- Tarjetas.
- Suscripciones.
- Integraciones con proveedores de pago o facturación.
- Administración de múltiples usuarios por cliente.

---

# Reglas de Negocio

- Solo un distribuidor puede registrar clientes.
- Cada cliente pertenece al distribuidor que lo registró.
- Durante el registro, el distribuidor captura el nombre comercial del cliente y los datos de la persona responsable de su cuenta de acceso.
- Los datos de la cuenta de acceso incluyen nombre, apellido paterno, apellido materno cuando corresponda y correo electrónico.
- Cada cliente debe estar asociado a una única cuenta de usuario.
- La cuenta de usuario es creada durante el registro del cliente.
- El sistema asigna automáticamente el rol Cliente a la cuenta creada.
- Toda cuenta nueva de cliente inicia sin contraseña.
- El sistema envía un correo de confirmación a la dirección registrada.
- El cliente establece su contraseña al confirmar su cuenta.
- La cuenta debe estar confirmada para que el cliente pueda acceder a EmailPro.
- Una cuenta de usuario no puede representar a más de un cliente.
- El distribuidor puede consultar la información registrada por el cliente, pero no modificarla.
- La información fiscal puede permanecer sin registrar durante la creación inicial del cliente.
- Cuando exista un perfil fiscal, todos sus datos deberán estar completos.
- Cada cliente puede tener como máximo un perfil fiscal asociado.
- Las modificaciones a la información fiscal actualizan el perfil existente y no generan un nuevo registro.
- El uso CFDI seleccionado más recientemente se conserva como uso predeterminado.
- El uso CFDI predeterminado debe ser compatible con el régimen fiscal seleccionado conforme al catálogo fiscal vigente.
- En el alcance actual, un cliente dispone de una sola cuenta de acceso.

---

# Flujo Principal

```text
Distribuidor
    ↓
Registrar cliente
    ↓
Capturar nombre comercial y datos de la cuenta responsable
    ↓
Crear cuenta de usuario con rol Cliente
    ↓
Enviar correo de confirmación
    ↓
Cliente confirma su cuenta
    ↓
Define contraseña
    ↓
Cliente accede a EmailPro
    ↓
Registrar perfil fiscal
    ↓
Cliente con información completa
```

---

# Entidades

- Roles
- Users
- Distributors
- Clients
- Fiscal Profiles
- Fiscal Regimes
- CFDI Usages

---

# Observaciones

La entidad Users continúa siendo la entidad central utilizada para el acceso al sistema.

Este proceso incorpora el rol Cliente al catálogo de roles, pero no modifica estructuralmente la entidad Users.

El nombre comercial pertenece al cliente. Los nombres, apellidos y correo electrónico de la persona responsable pertenecen a su cuenta de usuario.

La información fiscal pertenece al perfil fiscal asociado al cliente y no a su cuenta de usuario.

Los identificadores de proveedores de pago o facturación no forman parte de este proceso y deberán incorporarse únicamente cuando los procesos correspondientes sean documentados.

Los procesos relacionados con dominios, cuentas de correo electrónico, pagos, facturación y múltiples usuarios por cliente deberán documentarse de forma independiente cuando sean requeridos por el negocio.
