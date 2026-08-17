# Gestión de Distribuidores

**Versión:** 0.1.0  
**Última actualización:** 2026-08-14

---

# Objetivo

Definir las reglas de negocio para el registro y administración de los distribuidores autorizados para comercializar EmailPro.

---

# Alcance

Incluye:

- Registro de distribuidores por parte del Super Admin.
- Asociación de una cuenta de usuario al distribuidor.
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
- Cada distribuidor debe estar asociado a una única cuenta de usuario.
- La cuenta de usuario es creada durante el registro del distribuidor.
- La cuenta asociada al distribuidor debe utilizar el rol Distribuidor.
- Una cuenta de usuario no puede representar a más de un distribuidor.
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
Crear cuenta de usuario
    ↓
Asignar rol Distribuidor
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

La información fiscal pertenece al perfil fiscal asociado al distribuidor y no a su cuenta de usuario.

Los procesos relacionados con clientes, facturación, pagos y múltiples usuarios por distribuidor deberán documentarse de forma independiente cuando sean requeridos por el negocio.