# Gestión de Distribuidores

**Versión:** 0.4.0  
**Última actualización:** 2026-09-25

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
- Definición y actualización del costo mensual por cuenta de correo asignado al distribuidor.
- Captura y actualización de información fiscal.
- Captura y actualización de la CLABE utilizada para recibir depósitos.
- Administración del uso CFDI predeterminado.
- Creación y actualización de la organización de facturación del distribuidor.
- Carga directa del Certificado de Sello Digital mediante el proveedor de facturación.

No incluye:

- Clientes.
- Autenticación.
- Recuperación de contraseña.
- Emisión de facturas relacionadas con pagos.
- Cálculo o pago de comisiones.
- Administración de múltiples usuarios por distribuidor.

---

# Reglas de Negocio

- Solo un Super Admin puede registrar distribuidores.
- Durante el registro, el Super Admin captura el nombre comercial del distribuidor y los datos de la persona responsable de su cuenta de acceso.
- Los datos de la cuenta de acceso incluyen nombre, apellido paterno, apellido materno cuando corresponda y correo electrónico.
- Durante el registro, el Super Admin define el costo mensual por cuenta de correo asignado al distribuidor.
- El costo por cuenta debe ser un importe mayor que cero.
- El Super Admin puede actualizar el costo asignado al distribuidor.
- Los cambios en el costo aplican únicamente a pagos futuros y no modifican pagos existentes.
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
- Al guardar un perfil fiscal por primera vez, el sistema crea una organización para el distribuidor mediante el proveedor de facturación.
- El identificador externo de la organización se conserva en el distribuidor.
- Si el perfil fiscal se modifica, sus datos se sincronizan con la organización existente y no se crea otra organización.
- Si el perfil fiscal ya existía pero el distribuidor todavía no tiene una organización, el siguiente guardado o actualización debe intentar crearla.
- Un error del proveedor no elimina ni revierte el perfil fiscal guardado; el distribuidor permanece sin habilitación para recibir pagos hasta completar su organización.
- El distribuidor carga su archivo `.cer`, archivo `.key` y contraseña mediante un formulario protegido.
- EmailPro transmite los archivos y la contraseña a la librería de facturación, pero no almacena ninguno de esos datos.
- La organización debe completar los requisitos reportados por el proveedor para poder emitir CFDI en producción.
- La disponibilidad para facturar se valida mediante el estado vigente de la organización en el proveedor y no mediante un indicador local permanente.
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
Definir costo mensual por cuenta de correo
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
Crear y configurar organización de facturación
    ↓
Cargar Certificado de Sello Digital
    ↓
Organización lista para emitir CFDI
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

El costo por cuenta representa el importe mensual que corresponde a SVR por cada espacio de correo contratado mediante los dominios del distribuidor.

La información fiscal pertenece al perfil fiscal asociado al distribuidor y no a su cuenta de usuario. El identificador de la organización de facturación pertenece al distribuidor porque únicamente los distribuidores actúan como emisores adicionales dentro de EmailPro.

La organización puede requerir pasos adicionales definidos por el proveedor, como datos fiscales completos, suscripción activa y Carta Manifiesto. La API debe utilizar el estado de preparación reportado por la librería antes de habilitar pagos.

Los procesos relacionados con facturación y múltiples usuarios por distribuidor deberán documentarse de forma independiente cuando sean requeridos por el negocio.
