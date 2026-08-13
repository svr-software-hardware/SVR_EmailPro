# Gestión de Usuarios Internos

**Versión:** 0.1.0  
**Última actualización:** 2026-08-10

---

# Objetivo

Definir las reglas de negocio para la administración del personal interno de SVR responsable de operar la plataforma EmailPro.

---

# Alcance

Incluye:

- Super Admin.
- Operador.
- Alta de usuarios.
- Edición de usuarios.
- Activación y desactivación de cuentas.
- Confirmación inicial de cuenta.

No incluye:

- Distribuidores.
- Clientes.
- Autenticación.
- Recuperación de contraseña.
- Permisos por módulo.

---

# Reglas de Negocio

- El primer Super Admin es creado durante la instalación inicial del sistema.
- Solo un Super Admin puede crear Super Admin y Operadores.
- Toda cuenta nueva inicia sin contraseña.
- El usuario establece su contraseña al confirmar su cuenta mediante correo electrónico.
- Una cuenta debe estar confirmada para poder iniciar sesión.
- Las cuentas nunca se eliminan físicamente.
- Un usuario no puede desactivar su propia cuenta.
- Los permisos específicos del Operador se definen en un proceso independiente.

---

# Flujo Principal

```text
Super Admin
    ↓
Registrar usuario
    ↓
Enviar correo de confirmación
    ↓
Usuario confirma cuenta
    ↓
Define contraseña
    ↓
Cuenta activa
```

---

# Entidades

- Roles
- Users

---

# Observaciones

Este proceso define únicamente la administración del personal interno de SVR.

Las reglas relacionadas con autenticación, permisos, distribuidores y clientes serán documentadas en procesos independientes.