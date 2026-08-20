# Gestión de Distribuidores

**Versión:** 0.2.0  
**Última actualización:** 2026-08-20

---

# Objetivo

Definir las reglas de negocio para el registro y administración de los distribuidores autorizados para comercializar EmailPro.

---

# Alcance

Incluye:

- Registro de distribuidores por parte del Super Admin.
- Creación de la cuenta de usuario asociada al distribuidor.
- Confirmación inicial de la cuenta del distribuidor.
- Consulta de la información del distribuidor por parte del Super Admin.
- Captura y actualización de información fiscal.
- Captura y actualización de la CLABE utilizada para recibir depósitos.
- Administración del uso CFDI predeterminado.

No incluye:

- Clientes.
- Autenticación.
- Recuperación de contraseña.
- Facturación.
- Cálculo o pago de comisiones.
- Administración de múltiples usuarios por distribuidor.

---

# Reglas de Negocio

- Solo un Super Admin puede registrar distribuidores.
- Durante el registro, el Super Admin captura el nombre comercial del distribuidor y los datos de la persona responsable de su cuenta de acceso.
- Los datos de la cuenta de acceso incluyen nombre, apellido paterno, apellido materno cuando corresponda y correo electrónico.
- Cada distribuidor debe estar asociado a una única cuenta de usuario.
- La cuenta de usuario es creada durante el registro del distribuidor.
- El sistema asigna automáticamente el rol Distribuidor a la cuenta creada.
- Toda cuenta nueva de distribuidor inicia sin contraseña.
- El sistema envía un correo de confirmación a la dirección registrada.
- El distribuidor establece su contraseña al confirmar su cuenta.
- La cuenta debe estar confirmada para que el distribuidor pueda acceder a EmailPro.
- Una cuenta de usuario no puede representar a más de un distribuidor.
- El Super Admin puede consultar la información registrada por el distribuidor, pero no modificarla.
- La CLABE puede permanecer sin registrar durante la creación inicial del distribuidor.
- La CLABE registrada debe ser una CLABE válida de 18 dígitos.
- La información fiscal puede permanecer sin registrar durante la creación inicial del distribuidor.
- Cuando exista un perfil fiscal, todos sus datos deberán estar completos.
- Cada distribuidor puede tener como máximo un perfil fiscal asociado.
- Las modificaciones a la información fiscal actualizan el perfil existente y no generan un nuevo registro.
- El uso CFDI seleccionado más recientemente se conserva como uso predeterminado.
- El uso CFDI predeterminado debe ser compatible con el régimen fiscal seleccionado conforme al catálogo fiscal vigente.
- La CLABE representa la cuenta bancaria utilizada para realizar depósitos al distribuidor.
- En el alcance actual, un distribuidor dispone de una sola cuenta de acceso.

---

# Flujo Principal

```text
Super Admin
    ↓
Registrar distribuidor
    ↓
Capturar nombre comercial y datos de la cuenta responsable
    ↓
Crear cuenta de usuario con rol Distribuidor
    ↓
Enviar correo de confirmación
    ↓
Distribuidor confirma su cuenta
    ↓
Define contraseña
    ↓
Distribuidor accede a EmailPro
    ↓
Registrar CLABE
    ↓
Registrar perfil fiscal
    ↓
Distribuidor con información completa
```

---

# Entidades

- Roles
- Users
- Distributors
- Fiscal Profiles
- Fiscal Regimes
- CFDI Usages

---

# Observaciones

La entidad Users continúa siendo la entidad central utilizada para el acceso al sistema.

Este proceso incorpora el rol Distribuidor al catálogo de roles, pero no modifica estructuralmente la entidad Users.

El nombre comercial pertenece al distribuidor. Los nombres, apellidos y correo electrónico de la persona responsable pertenecen a su cuenta de usuario.

La información fiscal pertenece al perfil fiscal asociado al distribuidor y no a su cuenta de usuario.

Los procesos relacionados con clientes, facturación, pagos y múltiples usuarios por distribuidor deberán documentarse de forma independiente cuando sean requeridos por el negocio.
