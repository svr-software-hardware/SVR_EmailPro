# Operaciones de Cuentas de Correo

**Versión:** 0.1.0  
**Última actualización:** 2026-09-25

---

# Objetivo

Definir el flujo mediante el cual los cambios solicitados por un distribuidor son aplicados manualmente por SVR en IONOS y confirmados posteriormente en EmailPro.

---

# Alcance

Incluye:

- Solicitudes de creación de cuentas de correo.
- Solicitudes de modificación de la parte local.
- Solicitudes de activación y desactivación.
- Consulta administrativa de tareas pendientes y procesadas.
- Confirmación de tareas realizadas por usuarios de SVR.
- Cancelación de tareas pendientes.
- Conservación del valor actual y del valor solicitado.
- Auditoría del usuario que solicitó, actualizó, completó o canceló una tarea.

No incluye:

- Integración automática con IONOS.
- Contraseñas o credenciales de las cuentas de correo.
- Acceso al contenido de los buzones.
- Historial técnico de comunicaciones con IONOS.
- Ejecución de tareas por parte del cliente.

---

# Principio de Estado Real

- La entidad `mailboxes` representa únicamente las cuentas y valores que SVR ya confirmó en IONOS.
- Una solicitud pendiente no modifica inmediatamente la cuenta de correo.
- El valor solicitado se conserva en `mailbox_tasks` hasta que SVR complete la operación.
- Al completar la tarea, EmailPro aplica el cambio correspondiente sobre `mailboxes`.
- El distribuidor puede ver que existe una operación en proceso.
- Durante una modificación pendiente, la interfaz conserva el valor actual de la cuenta y señala que hay un cambio solicitado.
- El cliente únicamente consulta los valores ya confirmados en IONOS, pero puede ver el indicador de operación en proceso sobre una cuenta existente.
- Una creación pendiente se muestra al distribuidor solicitante, pero no se presenta al cliente como cuenta existente hasta que SVR la completa.

---

# Tipos de Operación

Los tipos permitidos son:

- `create`: crear una cuenta en IONOS y posteriormente registrarla en `mailboxes`.
- `update`: cambiar la parte local de una cuenta existente.
- `activate`: activar una cuenta existente.
- `deactivate`: desactivar una cuenta existente.

---

# Estados de la Tarea

Los estados permitidos son:

- `pending`: la operación todavía debe realizarse en IONOS.
- `completed`: SVR confirmó que realizó la operación y EmailPro aplicó el cambio.
- `cancelled`: la operación fue cancelada antes de ser realizada.

Toda tarea se crea con estado `pending`.

Una tarea completada o cancelada no puede volver a estado pendiente.

---

# Reglas de Creación

- Solo el distribuidor responsable del cliente puede solicitar la creación de una cuenta.
- Una solicitud de creación todavía no genera un registro en `mailboxes`.
- La tarea conserva el dominio y la parte local solicitada.
- `mailbox_id` permanece vacío hasta que SVR completa la creación.
- Al completar la tarea, el sistema vuelve a validar la unicidad y la capacidad del dominio.
- Si las validaciones son correctas, se crea la cuenta, se asocia su identificador a la tarea y se marca como completada dentro de la misma operación.
- Una tarea de creación pendiente puede cancelarse sin crear una cuenta de correo.

---

# Reglas de Modificación

- Solo el distribuidor responsable puede solicitar el cambio de la parte local.
- La tarea conserva la parte local vigente en `previous_local_part` y la solicitada en `requested_local_part`.
- Mientras la tarea permanezca pendiente, `mailboxes.local_part` conserva el valor vigente.
- La interfaz muestra el valor vigente e indica que existe una modificación en proceso.
- Al completar la tarea, el sistema vuelve a validar la unicidad del valor solicitado y actualiza `mailboxes.local_part`.
- Si el distribuidor solicita regresar al valor vigente antes de que SVR realice la operación, la tarea pendiente se cancela y no se genera una nueva.

---

# Reglas de Activación y Desactivación

- Una solicitud de activación o desactivación no cambia inmediatamente `mailboxes.is_active`.
- La tarea conserva el estado vigente en `previous_is_active` y el solicitado en `requested_is_active`.
- Al completar la tarea, EmailPro actualiza `mailboxes.is_active` con el estado solicitado.
- Si existe una desactivación pendiente y el distribuidor vuelve a solicitar la activación antes de que SVR la realice, la desactivación se cancela y no se genera una activación.
- Si existe una activación pendiente y el distribuidor vuelve a solicitar la desactivación antes de que SVR la realice, la activación se cancela y no se genera una desactivación.
- Una tarea cancelada no aparece en el listado administrativo predeterminado de pendientes.

---

# Consolidación de Solicitudes Pendientes

- Cada cuenta puede tener como máximo una tarea pendiente.
- No se crean tareas consecutivas que se contradigan mientras la primera continúe pendiente.
- Si la nueva solicitud devuelve la cuenta a su estado o valor vigente, la tarea pendiente se cancela.
- Si una modificación pendiente cambia a un tercer valor antes de ser realizada, se actualiza el valor solicitado en la misma tarea.
- El valor anterior continúa representando el estado real confirmado en IONOS.
- Los campos de auditoría identifican quién creó y quién realizó la última actualización de la tarea.

---

# Capacidad del Dominio

- Las creaciones y activaciones pendientes reservan un lugar dentro del límite del dominio.
- Una solicitud de creación o activación se rechaza cuando la suma de cuentas activas y lugares reservados supera el límite.
- Una desactivación pendiente no libera capacidad hasta que SVR completa la tarea.
- Cancelar una creación o activación pendiente libera el lugar reservado.
- Las modificaciones de la parte local no alteran la capacidad utilizada.

---

# Listado Administrativo

- Solo los usuarios autorizados de SVR pueden consultar la cola administrativa.
- El listado muestra únicamente tareas pendientes por defecto.
- El administrador puede consultar también tareas completadas o canceladas mediante filtros.
- Cada registro presenta la operación solicitada, dominio, cuenta actual, valor solicitado, distribuidor, cliente, fecha de solicitud y usuario solicitante.
- Para las tareas completadas se presenta además la fecha y el usuario que realizó la operación.
- Para las tareas canceladas se presenta la fecha y el usuario que realizó la cancelación.
- Las tareas pendientes deben ordenarse de la más antigua a la más reciente.

---

# Confirmación por SVR

- SVR debe realizar primero la operación correspondiente en IONOS.
- Después de realizarla, el usuario de SVR marca la tarea como completada.
- La confirmación aplica el alta o cambio correspondiente sobre `mailboxes`.
- La actualización de la cuenta y de la tarea debe ser una sola operación consistente.
- El usuario que confirma se conserva en `completed_by_id` y la fecha en `completed_at`.
- Una tarea ya completada o cancelada no puede completarse nuevamente.

---

# Cancelación

- Una tarea únicamente puede cancelarse mientras se encuentre pendiente.
- El distribuidor puede cancelar una tarea pendiente perteneciente a uno de sus clientes.
- SVR puede cancelar una tarea pendiente cuando determine que no debe ejecutarse.
- La cancelación no modifica la cuenta existente.
- El usuario que cancela se conserva en `cancelled_by_id` y la fecha en `cancelled_at`.
- Las tareas canceladas se conservan para identificar qué solicitud fue descartada, pero no representan una operación realizada en IONOS.

---

# Auditoría

- `created_by_id` identifica al distribuidor que originó la solicitud.
- `updated_by_id` identifica al usuario que realizó el último cambio sobre la tarea.
- `completed_by_id` identifica al usuario de SVR que confirmó la ejecución en IONOS.
- `cancelled_by_id` identifica al usuario que canceló la solicitud.
- `created_at` representa la fecha de solicitud.
- `updated_at` representa la última modificación de la tarea.

---

# Flujo Principal

```text
Distribuidor solicita operación
    ↓
Validar dominio, cuenta, unicidad y capacidad
    ↓
Crear o consolidar tarea pendiente
    ↓
Mostrar operación en la cola de SVR
    ↓
SVR realiza la operación en IONOS
    ↓
SVR marca la tarea como completada
    ↓
Aplicar el cambio en Mailboxes
```

---

# Entidades

- Domains
- Mailboxes
- Mailbox Tasks
- Users

---

# Observaciones

`mailbox_tasks` no constituye un historial técnico de IONOS. Representa la cola operativa necesaria para solicitar, ejecutar, cancelar y auditar cambios manuales.

La separación entre `mailboxes` y `mailbox_tasks` evita mostrar como realizado un cambio que todavía no existe en IONOS.
