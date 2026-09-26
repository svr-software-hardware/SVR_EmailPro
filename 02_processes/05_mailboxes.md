# Gestión de Cuentas de Correo

**Versión:** 0.3.0  
**Última actualización:** 2026-09-25

---

# Objetivo

Definir las reglas de negocio para el registro y administración de las cuentas de correo asociadas a los dominios de EmailPro.

---

# Alcance

Incluye:

- Registro de cuentas de correo por parte del distribuidor.
- Modificación de la parte local de una cuenta de correo.
- Activación y desactivación de cuentas de correo por parte del distribuidor.
- Consulta de cuentas de correo por parte del distribuidor y del cliente.
- Control de la cantidad máxima de cuentas activas permitidas para cada dominio.
- Generación de tareas para que SVR aplique las altas, modificaciones, activaciones y desactivaciones en IONOS.
- Presentación de operaciones pendientes sobre las cuentas de correo.

No incluye:

- Registro o administración de dominios.
- Contraseñas de las cuentas de correo.
- Acceso a los buzones de correo.
- Configuración de servidores de correo.
- Aprovisionamiento de infraestructura.
- Pagos.
- Facturación.
- Precios.
- Consulta o administración de cuentas de correo por parte de SVR.

---

# Reglas de Negocio

- Cada cuenta de correo pertenece a un único dominio.
- Un dominio puede tener múltiples cuentas de correo.
- El distribuidor puede solicitar el registro y la modificación de cuentas de correo únicamente en los dominios de los clientes que le pertenecen.
- El cliente no puede registrar ni modificar cuentas de correo.
- Solo el distribuidor puede solicitar la modificación de la parte local de una cuenta de correo.
- Solo el distribuidor puede solicitar la activación o desactivación de una cuenta de correo.
- Una solicitud del distribuidor no modifica inmediatamente el estado real de la cuenta.
- La cuenta se crea o actualiza en EmailPro únicamente cuando SVR confirma que realizó la operación en IONOS.
- Mientras exista una modificación pendiente, la cuenta conserva visualmente su valor actual y muestra que tiene una operación en proceso.
- Las cuentas de correo nunca se eliminan físicamente; únicamente pueden desactivarse.
- Toda cuenta de correo se identifica mediante una parte local y el dominio al que pertenece.
- La parte local corresponde al texto anterior al símbolo `@`.
- La dirección completa se obtiene al combinar la parte local con el dominio y no se almacena de forma duplicada.
- La combinación de la parte local y el dominio debe ser única en EmailPro.
- Una cuenta desactivada conserva su dirección y no permite registrar otra cuenta con la misma combinación de parte local y dominio.
- Solo las cuentas activas consumen lugares del límite definido para el dominio.
- La cantidad de cuentas activas no puede superar el límite definido para el dominio.
- Una cuenta desactivada puede reactivarse únicamente cuando exista capacidad disponible dentro del límite del dominio.
- El cliente puede consultar las cuentas de correo de sus dominios.
- El distribuidor puede consultar las cuentas de correo de los dominios pertenecientes a sus clientes.
- En el alcance actual, SVR no puede consultar ni administrar las cuentas de correo.
- Los precios y cobros relacionados con las cuentas de correo se definirán en un proceso independiente.
- Las tareas requeridas para aplicar cambios en IONOS se definen en el proceso de operaciones de cuentas de correo.

---

# Flujo Principal

```text
Distribuidor
    ↓
Seleccionar dominio permitido
    ↓
Capturar parte local
    ↓
Validar unicidad dentro del dominio
    ↓
Validar capacidad disponible
    ↓
Generar tarea de creación
    ↓
SVR crea la cuenta en IONOS
    ↓
SVR marca la tarea como completada
    ↓
Registrar cuenta de correo en EmailPro
```

---

# Flujo de Activación y Desactivación

```text
Distribuidor
    ↓
Seleccionar cuenta de correo
    ↓
Solicitar activación o desactivación
    ↓
SVR aplica el cambio en IONOS
    ↓
SVR marca la tarea como completada
    ↓
Actualizar disponibilidad dentro del límite del dominio
```

---

# Entidades

- Domains
- Mailboxes
- Mailbox Tasks

---

# Observaciones

Para la dirección `ventas@empresa.com.mx`, la parte local es `ventas` y el dominio es `empresa.com.mx`.

La dirección completa se presenta al usuario mediante la información de la cuenta y su dominio asociado.

Este proceso define únicamente el registro y administración lógica de las cuentas de correo. Las credenciales, el acceso al buzón y el aprovisionamiento técnico deberán documentarse en procesos independientes cuando sean requeridos por el negocio.

Los procesos relacionados con pagos, facturación, precios y acceso futuro de SVR deberán documentarse de forma independiente.
