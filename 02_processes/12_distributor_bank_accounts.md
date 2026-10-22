# Cuentas Bancarias de Distribuidores

**Versión:** 0.3.0
**Última actualización:** 2026-10-22

---

# Objetivo

Definir el registro, validación, sustitución y conservación histórica de las cuentas bancarias utilizadas para liquidar a los distribuidores de EmailPro.

---

# Alcance

Incluye:

- Registro inicial de la cuenta bancaria por parte del distribuidor.
- Solicitud de cambio de cuenta bancaria.
- Advertencia sobre la pausa de liquidaciones durante una revisión.
- Validación o rechazo manual por parte de SVR.
- Cancelación de una solicitud pendiente por parte del distribuidor.
- Conservación de cuentas bancarias anteriores.
- Configuración bancaria adicional realizada por SVR.
- Compatibilidad con pagos BBVA, SPEI y convenios CIE.
- Pausa y reanudación de movimientos pendientes de liquidación.
- Auditoría de solicitudes, validaciones, rechazos y cancelaciones.
- Notificación al Super Admin cuando exista una cuenta pendiente de validación.
- Consulta administrativa de solicitudes bancarias pendientes y resueltas.

No incluye:

- Generación de órdenes de pago.
- Construcción del archivo TXT de BBVA.
- Importación o conciliación del archivo bancario.
- Ejecución de transferencias dentro de BBVA.
- Eliminación física de cuentas bancarias históricas.

---

# Principios Generales

- Una cuenta bancaria no puede utilizarse para generar órdenes de pago hasta que SVR la valide.
- La primera cuenta registrada y cualquier cuenta posterior deben pasar por el mismo proceso de validación.
- El distribuidor puede solicitar cambios, pero únicamente SVR puede validar o rechazar la información bancaria.
- El registro o cambio de una cuenta bancaria no modifica órdenes de pago previamente generadas.
- Las cuentas anteriores se conservan para auditoría y conciliación.
- Los datos bancarios utilizados por una orden de pago se conservan también como una copia histórica inmutable dentro del movimiento correspondiente.
- Una cuenta verificada no debe confundirse con una liquidación realizada.
- La existencia de una cuenta verificada es solamente uno de los requisitos para que un movimiento pueda incorporarse a una orden de pago.

---

# Métodos Bancarios

Los métodos bancarios contemplados son:

- `PTC`: pago a una cuenta BBVA.
- `PSC`: pago interbancario mediante SPEI.
- `CIL`: pago a un convenio CIE.

El distribuidor elige si desea recibir sus liquidaciones mediante CLABE o mediante un convenio CIE.

Cuando elige CLABE:

- Captura una CLABE válida de 18 dígitos.
- El sistema obtiene el código de la institución financiera de los primeros tres dígitos de la CLABE.
- El sistema relaciona la cuenta con el catálogo de instituciones financieras mediante ese código.
- El sistema obtiene el número de cuenta de once dígitos contenido entre las posiciones 7 y 17 de la CLABE.
- No captura convenio ni referencia CIE.
- Si la institución corresponde a BBVA, con código `012`, el sistema determina automáticamente el método `PTC`.
- Si la CLABE pertenece a otra institución, el sistema determina automáticamente el método `PSC`.

Cuando elige CIE:

- Captura el convenio CIE y su referencia correspondiente.
- No captura CLABE.
- La institución financiera se asigna automáticamente a BBVA, con código `012`.
- El número de cuenta permanece vacío porque el pago se identifica mediante el convenio CIE.
- El sistema determina automáticamente el método `CIL`.

El distribuidor no selecciona directamente `PTC`, `PSC` o `CIL`. El sistema calcula el método a partir de la información bancaria capturada y conserva la relación con el catálogo de métodos bancarios.

---

# Estados de la Cuenta Bancaria

Los estados permitidos son:

- `pending`: la cuenta espera revisión de SVR.
- `verified`: SVR confirmó que la cuenta puede utilizarse para futuras órdenes de pago.
- `rejected`: SVR rechazó la cuenta presentada.
- `cancelled`: el distribuidor canceló la solicitud antes de que SVR la resolviera.
- `replaced`: la cuenta dejó de ser la cuenta vigente después de validarse una cuenta posterior.

Una cuenta rechazada, cancelada o sustituida se conserva y no puede volver directamente al estado `pending`.

Si el distribuidor desea volver a utilizar datos de una cuenta histórica que ya no se encuentra vigente, debe presentar una nueva solicitud para su validación.

---

# Registro Inicial

- La cuenta bancaria puede permanecer sin registrar durante la creación inicial del distribuidor.
- El distribuidor registra la información requerida desde su perfil.
- El distribuidor selecciona CLABE o CIE y captura únicamente los campos correspondientes a su selección.
- CLABE y CIE son alternativas mutuamente excluyentes.
- Una cuenta con CLABE no puede conservar datos CIE y una cuenta CIE no puede conservar una CLABE.
- Una CLABE debe tener una estructura válida y su código institucional debe existir y encontrarse activo en el catálogo de instituciones financieras.
- Para CLABE, la institución financiera y el número de cuenta se deducen en el servidor y no se reciben como valores editables independientes.
- Para CIE, la institución financiera se asigna automáticamente a BBVA y el número de cuenta permanece vacío.
- La primera cuenta se crea con estado `pending`.
- Mientras no exista una cuenta verificada, ningún movimiento puede incorporarse a una orden de pago para ese distribuidor.
- SVR revisa manualmente la información y realiza las acciones externas necesarias, incluida el alta del beneficiario en BBVA cuando corresponda.
- Si la cuenta es correcta, SVR la marca como `verified`.
- Si la cuenta no puede utilizarse, SVR la marca como `rejected` e indica obligatoriamente el motivo.

---

# Solicitud de Cambio

- Solo el distribuidor propietario puede solicitar el cambio de su cuenta bancaria.
- El distribuidor no puede modificar directamente una cuenta histórica.
- Un distribuidor puede tener como máximo una solicitud bancaria pendiente.
- Si ya existe una solicitud pendiente, no puede registrar otra hasta que la anterior sea validada, rechazada o cancelada.
- Antes de confirmar el cambio, la interfaz debe advertir que las liquidaciones pendientes serán pausadas.
- La nueva cuenta se registra con estado `pending`.
- La cuenta anteriormente verificada permanece conservada, pero deja de ser elegible para nuevas órdenes mientras la solicitud se encuentre en revisión.
- Los movimientos que todavía no pertenezcan a una orden de pago permanecen pausados.
- Los pagos nuevos pueden continuar registrándose y facturándose, pero su liquidación permanece pendiente.

La advertencia debe comunicar de forma equivalente lo siguiente:

> Al solicitar el cambio de cuenta bancaria, los pagos pendientes que todavía no formen parte de una orden de pago quedarán pausados hasta que SVR valide la nueva información. Las órdenes ya generadas continuarán utilizando los datos bancarios registrados en ellas.

---

# Validación por SVR

- Solo un Super Admin puede validar una cuenta bancaria.
- La validación es un proceso manual realizado después de revisar la información correspondiente.
- El sistema determina automáticamente el método `PTC`, `PSC` o `CIL` antes de que SVR resuelva la solicitud.
- SVR comprueba que el método calculado corresponda con los datos presentados.
- Para `CIL`, SVR valida el convenio y la referencia CIE capturados por el distribuidor.
- Al validar una nueva cuenta, la cuenta verificada anterior cambia a `replaced`.
- La nueva cuenta cambia de `pending` a `verified`.
- La sustitución de la cuenta anterior y la validación de la nueva deben realizarse en una sola operación consistente.
- Después de la validación, los movimientos pausados vuelven a ser elegibles para una orden de pago, siempre que cumplan todos los demás requisitos del proceso de liquidación.
- La validación no modifica `distributor_available_at` en los pagos existentes.

---

# Listado Administrativo

- La interfaz de administración incluye una sección exclusiva para revisar solicitudes de cuentas bancarias.
- Solo los usuarios Super Admin pueden acceder a esta sección.
- El listado muestra las solicitudes pendientes por defecto.
- El Super Admin puede consultar también solicitudes verificadas, rechazadas y canceladas mediante filtros.
- Las solicitudes pendientes se ordenan de la más antigua a la más reciente.
- Cada registro muestra por lo menos el distribuidor, la cuenta parcialmente oculta, la fecha de solicitud, el usuario solicitante y el estado.
- Cuando corresponde a CLABE, el detalle muestra la institución financiera, la CLABE completa, el número de cuenta deducido y el titular beneficiario.
- Cuando corresponde a CIE, el detalle muestra BBVA, el convenio, la referencia y el titular beneficiario.
- El detalle permite consultar la información completa necesaria para realizar la validación.
- Desde el detalle, el Super Admin puede revisar el método calculado y validar o rechazar la solicitud.
- Para solicitudes resueltas se muestra quién realizó la acción, cuándo la realizó y el motivo cuando corresponda.

---

# Rechazo por SVR

- Solo un Super Admin puede rechazar una cuenta pendiente.
- El rechazo requiere un motivo obligatorio.
- La cuenta cambia de `pending` a `rejected`.
- El distribuidor debe poder consultar el motivo del rechazo.
- Un rechazo no reactiva automáticamente la cuenta anterior.
- Los movimientos pendientes continúan pausados hasta que el distribuidor presente otra cuenta y SVR la valide.
- La cuenta rechazada no puede editarse ni reenviarse como el mismo registro.
- El distribuidor puede presentar una nueva solicitud después del rechazo.

---

# Cancelación por el Distribuidor

- El distribuidor puede cancelar únicamente una solicitud que permanezca en estado `pending`.
- Una solicitud validada o rechazada no puede cancelarse.
- La cuenta pendiente cambia a `cancelled`.
- Si existía una cuenta verificada anterior, vuelve a ser elegible para futuras órdenes de pago.
- Los movimientos pausados vuelven a ser elegibles cuando la cancelación deja nuevamente una cuenta anterior válida, siempre que cumplan los demás requisitos de liquidación.
- Si no existía una cuenta verificada anterior, los movimientos permanecen pausados hasta validar una cuenta nueva.
- La cancelación no modifica órdenes de pago ya generadas.

---

# Pausa de Liquidaciones

Un distribuidor no puede incorporar movimientos nuevos a una orden de pago cuando ocurra cualquiera de estas condiciones:

- No tiene una cuenta bancaria verificada.
- Tiene una solicitud bancaria pendiente.
- Su solicitud más reciente fue rechazada y todavía no cuenta con una nueva validación.
- La información bancaria requiere una corrección o revisión adicional de SVR.

La pausa afecta únicamente a movimientos que todavía no pertenezcan a una orden de pago.

La pausa no afecta:

- El pago realizado previamente por el cliente.
- La vigencia concedida al dominio.
- La factura emitida por EmailPro al cliente.
- La factura emitida por el distribuidor a SVR.
- Las órdenes de pago creadas antes de solicitar el cambio.
- Los movimientos que ya se encuentren dentro de dichas órdenes.

La pausa no reinicia ni modifica `distributor_available_at`.

---

# Inmutabilidad de Órdenes Existentes

- Una orden de pago conserva los datos bancarios que tenía al momento de generarse.
- Una solicitud de cambio posterior no actualiza ni invalida esos datos.
- La orden continúa su proceso de descarga, envío y conciliación con la información conservada originalmente.
- Si una transferencia ya fue enviada a BBVA, el cambio de cuenta no permite cancelarla desde EmailPro.
- Cualquier incidencia sobre una orden existente se resuelve mediante el proceso de conciliación y no mediante la edición de la cuenta bancaria.

---

# Historial

- Las cuentas bancarias nunca se eliminan físicamente.
- Cada cuenta conserva el distribuidor al que pertenece.
- Cada cuenta conserva su estado y sus fechas de solicitud y resolución.
- Las cuentas rechazadas conservan el motivo correspondiente.
- Las cuentas sustituidas permanecen disponibles para auditoría.
- El historial permite identificar qué cuenta se encontraba vigente en cada momento.
- El distribuidor puede consultar su cuenta vigente, sus solicitudes pendientes y el resultado de sus solicitudes anteriores.
- Solo los usuarios autorizados de SVR pueden consultar la información bancaria completa y su historial administrativo.

---

# Seguridad y Presentación

- La cuenta o CLABE debe validarse conforme al método bancario correspondiente.
- La institución financiera y el número de cuenta no se capturan manualmente cuando existe una CLABE; se deducen de su estructura.
- La interfaz debe ocultar parcialmente los números de cuenta en listados y mensajes generales.
- Los datos completos únicamente se muestran a usuarios autorizados y dentro de los flujos que los requieren.
- La información bancaria no se incluye en respuestas públicas.
- La descarga de archivos bancarios y la consulta de cuentas completas pertenecen exclusivamente al Super Admin.
- Los registros de auditoría no deben exponer información bancaria completa cuando no sea indispensable.

---

# Auditoría

- La creación identifica al usuario que presentó la cuenta.
- El estado actual se conserva mediante el catálogo de estados de cuentas bancarias.
- `updated_at` y `updated_by_id` identifican cuándo y quién realizó el último cambio de estado.
- `status_reason` conserva el motivo del último cambio cuando corresponda.
- `status_reason` es obligatorio para un rechazo.
- No se conserva una tabla independiente con el historial de cada transición de estado.
- Las cuentas anteriores continúan conservándose como registros independientes aunque no se mantenga un historial adicional de acciones.

---

# Notificaciones

- Cuando se registra una cuenta inicial o una solicitud de cambio, el sistema notifica a los Super Admin que existe una cuenta pendiente de validación.
- La notificación administrativa identifica al distribuidor y permite acceder a la solicitud correspondiente.
- La notificación al Super Admin no sustituye la presencia de la solicitud en el listado administrativo.
- El distribuidor recibe una notificación cuando SVR valida su cuenta.
- El distribuidor recibe una notificación cuando SVR rechaza su cuenta.
- La notificación de rechazo incluye el motivo registrado.
- La interfaz muestra claramente cuando las liquidaciones se encuentran pausadas por una revisión bancaria.
- La cancelación realizada por el propio distribuidor se confirma en la interfaz.

---

# Flujo de Registro Inicial

```text
Distribuidor registra cuenta bancaria
    ↓
Crear solicitud pending
    ↓
Pausar incorporación a órdenes de pago
    ↓
El sistema calcula el método y SVR revisa la cuenta
    ↓
SVR valida
    ↓
Cuenta verified
    ↓
Movimientos elegibles para órdenes de pago
```

---

# Flujo de Cambio Aprobado

```text
Distribuidor solicita cambio
    ↓
Mostrar advertencia de pausa
    ↓
Crear nueva cuenta pending
    ↓
Pausar movimientos fuera de órdenes existentes
    ↓
SVR revisa y valida
    ↓
Cuenta anterior replaced
    ↓
Cuenta nueva verified
    ↓
Reanudar elegibilidad de movimientos pendientes
```

---

# Flujo de Rechazo

```text
SVR rechaza cuenta pending
    ↓
Registrar motivo obligatorio
    ↓
Cuenta rejected
    ↓
Mantener liquidaciones pausadas
    ↓
Distribuidor presenta otra cuenta
    ↓
SVR valida la nueva cuenta
    ↓
Reanudar elegibilidad
```

---

# Flujo de Cancelación

```text
Distribuidor cancela solicitud pending
    ↓
Cuenta solicitada cancelled
    ↓
¿Existe una cuenta anterior verificada?
    ├─ Sí → Reanudar elegibilidad con la cuenta anterior
    └─ No → Mantener liquidaciones pausadas
```

---

# Entidades Relacionadas

- Distributors
- Distributor Bank Accounts
- Financial Institutions
- Users
- Payments
- Invoices
- Payment Orders
- Payment Order Items

---

# Observaciones

La cuenta bancaria vigente no debe almacenarse únicamente como un valor reemplazable dentro de `distributors`. El historial forma parte de la operación porque permite validar solicitudes, conservar cuentas anteriores y demostrar qué información se utilizó en cada orden.

El proceso de órdenes de pago deberá consultar este proceso para determinar si un movimiento es elegible, pero conservará una copia inmutable de la cuenta y configuración bancaria utilizadas.

El catálogo de instituciones financieras se inicializa con el listado publicado por Banco de México. El código utilizado por EmailPro corresponde a los últimos tres dígitos de la clave de institución publicada y coincide con los primeros tres dígitos de la CLABE.

CLABE y CIE permanecen como alternativas mutuamente excluyentes. Para CLABE, los primeros tres dígitos identifican la institución financiera y las posiciones 7 a 17 identifican el número de cuenta. Para CIE, la institución es siempre BBVA y no existe un número de cuenta derivado.
