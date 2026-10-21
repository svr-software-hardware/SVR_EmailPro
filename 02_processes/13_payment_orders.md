# Órdenes de Pago y Conciliación Bancaria

**Versión:** 0.2.0
**Última actualización:** 2026-10-20

---

# Objetivo

Definir el proceso mediante el cual EmailPro agrupa automáticamente los importes que corresponden a los distribuidores, genera archivos de pago masivo para BBVA y concilia los resultados bancarios hasta concluir cada liquidación.

---

# Alcance

Incluye:

- Cálculo de la fecha de disponibilidad para liquidar al distribuidor.
- Selección automática de movimientos elegibles.
- Generación diaria de órdenes de pago.
- Creación de un movimiento independiente por cada pago del cliente.
- Construcción de registros bancarios `PTC`, `PSC` y `CIL`.
- Generación, almacenamiento y descarga del archivo TXT mixto.
- Confirmación de que una orden fue presentada en BBVA.
- Importación y validación del archivo Excel devuelto por BBVA.
- Conciliación total o parcial de una orden.
- Reintento de movimientos rechazados.
- Resolución manual de movimientos ausentes en la respuesta bancaria.
- Consulta histórica de órdenes y movimientos.
- Notificación individual al distribuidor después de confirmar su liquidación.

No incluye:

- Ejecución automática de transferencias mediante una API bancaria.
- Registro o validación de cuentas bancarias de distribuidores.
- Generación de facturas.
- Modificación de pagos realizados por clientes.
- Agrupación de varios pagos en una sola transferencia al distribuidor.
- Generación manual de órdenes de pago.
- Edición manual de los movimientos contenidos en una orden.

---

# Terminología

- **Orden de pago:** agrupación diaria de movimientos que EmailPro prepara para su presentación en BBVA.
- **Movimiento de pago:** transferencia individual al distribuidor correspondiente a un único pago realizado por un cliente.
- **Fecha de disponibilidad:** día a partir del cual un pago puede incorporarse a una orden.
- **Conciliación bancaria:** comparación entre los movimientos de una orden y el archivo de resultados obtenido de BBVA.
- **Liquidado:** movimiento que BBVA o un Super Admin confirmó como pagado al distribuidor.
- **Cuenta de cargo:** cuenta bancaria de SVR desde la cual se realizan las transferencias.

---

# Principios Generales

- El cliente paga a EmailPro y el dinero se recibe en una cuenta de SVR.
- EmailPro factura al cliente conforme al proceso de facturación.
- El distribuidor emite una factura a SVR por `payments.distributor_amount`.
- La liquidación bancaria al distribuidor ocurre después de completar la factura `distributor_to_svr`.
- Cada pago del cliente origina como máximo un importe pendiente de liquidación al distribuidor.
- Cada importe se transfiere de manera independiente.
- Los importes nunca se agrupan por distribuidor, cliente, dominio, fecha u orden.
- Dos pagos del mismo cliente generan dos movimientos bancarios distintos.
- Un movimiento no puede pertenecer simultáneamente a dos órdenes activas.
- La generación de una orden no significa que el distribuidor ya recibió su dinero.
- Una liquidación solo se considera concluida después de conciliar el movimiento como pagado.
- Los importes utilizados son históricos y se obtienen del pago; nunca se recalculan con precios actuales.

---

# Disponibilidad para Liquidación

El cálculo comienza en la fecha en que OpenPay confirma exitosamente el pago del cliente.

Los días hábiles se calculan considerando:

- De lunes a viernes.
- Excluyendo sábados y domingos.
- Excluyendo días inhábiles bancarios de México.

La cantidad de días hábiles se obtiene de la configuración del entorno.

La variable de entorno utilizada es `EMAILPRO_DISTRIBUTOR_PAYMENT_BUSINESS_DAYS` y su valor inicial es `5`.

La configuración debe contener un entero positivo. Si el valor falta o es inválido, el sistema no debe calcular una fecha por defecto distinta ni registrar un pago exitoso incompleto.

El resultado se conserva en `payments.distributor_available_at`.

Al alcanzar `distributor_available_at`, el pago puede incluirse en cualquier orden automática posterior mientras cumpla los demás requisitos.

No existe una fecha máxima de liquidación ni un campo `distributor_due_at`.

El valor se calcula una sola vez al registrar el pago exitoso y no se recalcula por cambios posteriores en la configuración o en el calendario.

Una cuenta bancaria pendiente o rechazada, una factura pendiente o cualquier otra incidencia no modifica `distributor_available_at`.

---

# Calendario Bancario

- El sistema debe disponer de los días inhábiles bancarios de México necesarios para realizar el cálculo.
- El calendario debe distinguir días hábiles de días no laborables.
- Los cambios al calendario aplican a cálculos futuros y no modifican fechas ya conservadas en pagos existentes.
- Si no existe información suficiente para calcular `distributor_available_at`, el pago exitoso no debe quedar registrado de forma incompleta.

---

# Movimientos sin Importe para el Distribuidor

- Cuando `payments.distributor_amount` sea igual a cero, no se crea una transferencia bancaria.
- El componente de liquidación del pago se considera concluido automáticamente.
- El pago no se incorpora a una orden.
- La conclusión sin transferencia debe poder distinguirse de una liquidación bancaria realizada.

---

# Requisitos de Elegibilidad

Un movimiento puede incorporarse a una orden únicamente cuando se cumplan todas estas condiciones:

- El pago del cliente se encuentra confirmado.
- `payments.distributor_amount` es mayor que cero.
- La factura `distributor_to_svr` se encuentra completada.
- `payments.distributor_available_at` ya fue alcanzado.
- El distribuidor tiene una cuenta bancaria verificada.
- El distribuidor no tiene una solicitud bancaria pendiente.
- La cuenta bancaria no se encuentra bloqueada por un rechazo sin resolver.
- SVR completó la configuración bancaria necesaria para `PTC`, `PSC` o `CIL`.
- El movimiento no se encuentra liquidado.
- El movimiento no pertenece a otra orden activa.
- El movimiento no requiere una revisión manual pendiente.

Los movimientos que no cumplan estas condiciones permanecen fuera de la orden sin modificar `distributor_available_at`.

---

# Selección Automática

- La generación se ejecuta todos los días a las 00:00, hora de Ciudad de México.
- La selección se realiza exclusivamente en el servidor.
- El frontend no envía una lista de pagos para construir la orden.
- No existe una opción para generar órdenes manualmente.
- Si no existen movimientos elegibles, no se crea una orden vacía.
- Los movimientos se ordenan por `distributor_available_at` ascendente.
- En caso de igualdad, se ordenan por la fecha de confirmación del pago y posteriormente por su identificador.
- Una orden admite como máximo 5,000 movimientos.
- Si existen más de 5,000 movimientos elegibles, el proceso crea las órdenes adicionales necesarias respetando el mismo orden de prioridad.
- La selección y asignación deben ejecutarse con transacciones y bloqueos que impidan incorporar el mismo movimiento en dos órdenes.

---

# Identificadores

- Cada pago se identifica externamente con el formato `SVR-EPP-<PAYMENT_PUBLIC_ID>`.
- Cada orden se identifica externamente con el formato `SVR-EPO-<PAYMENT_ORDER_PUBLIC_ID>`.
- `EPP` significa EmailPro Pago.
- `EPO` significa EmailPro Orden.
- `PAYMENT_PUBLIC_ID` corresponde al `public_id` del pago.
- `PAYMENT_ORDER_PUBLIC_ID` corresponde al `public_id` de la orden de pago.
- Los valores `public_id` son generados por la librería común del proyecto y contienen exactamente diez caracteres alfanuméricos.
- EmailPro no agrega ceros, no genera consecutivos propios y no expone el identificador interno de base de datos.
- Ejemplos válidos son `SVR-EPP-B3N8R5T1LC` para un pago y `SVR-EPO-A7K9M2Q4XZ` para una orden.
- Cada identificador externo completo mide 18 caracteres y cabe dentro de los campos bancarios de concepto de 30 posiciones.
- El identificador `SVR-EPP-<PAYMENT_PUBLIC_ID>` se conserva en los movimientos bancarios relacionados con el pago y se utiliza para la conciliación dentro de la orden correspondiente.
- El identificador `SVR-EPO-<PAYMENT_ORDER_PUBLIC_ID>` identifica la orden completa y se utiliza en el sistema y en el nombre de su archivo.
- Un reintento conserva el identificador público del pago, pero crea una nueva relación con otra orden.
- Los folios no se reutilizan aunque una orden o un movimiento sean cancelados o rechazados.
- Los folios deben permanecer estables durante todo el proceso.

---

# Estados de la Orden

Los estados permitidos son:

- `generated`: la orden fue creada y su TXT está disponible.
- `submitted`: un Super Admin confirmó que el archivo fue presentado en BBVA.
- `partially_reconciled`: se procesó una respuesta bancaria, pero todavía existen movimientos pendientes de resolución.
- `completed`: todos los movimientos tienen una resolución final dentro de esa orden.
- `cancelled`: la orden fue cancelada antes de ser presentada en BBVA.

La descarga del TXT no cambia por sí misma el estado de la orden.

Una orden puede descargarse nuevamente sin generar otro archivo ni cambiar su contenido.

---

# Estados del Movimiento

Los estados permitidos dentro de una orden son:

- `pending`: el movimiento está incluido y todavía no ha sido conciliado.
- `paid`: BBVA o un Super Admin confirmó la transferencia.
- `failed`: el movimiento no fue pagado y puede volver a ser elegible para otra orden.
- `under_review`: el movimiento no apareció en la respuesta bancaria o requiere resolución manual.
- `cancelled`: la orden fue cancelada antes de su presentación.

Un movimiento `paid` no puede volver a incorporarse a otra orden.

Un movimiento `failed` conserva su relación histórica con la orden anterior y puede generar un nuevo movimiento en una orden posterior.

Un movimiento `under_review` no puede incorporarse a otra orden hasta que SVR lo resuelva.

---

# Creación de la Orden

Al generar una orden, EmailPro conserva para cada movimiento una copia inmutable de:

- Pago relacionado.
- Distribuidor relacionado.
- Cuenta bancaria histórica relacionada.
- Método bancario: `PTC`, `PSC` o `CIL`.
- Cuenta o CLABE beneficiaria.
- Nombre del titular.
- Clave de la institución bancaria.
- Datos del convenio CIE cuando correspondan.
- Cuenta de cargo de SVR.
- Importe exacto.
- Moneda.
- Concepto.
- Referencia bancaria.
- Identificador externo `SVR-EPP-<PAYMENT_PUBLIC_ID>` del pago.
- Fecha de disponibilidad para liquidación.

Cambiar posteriormente la cuenta del distribuidor, la cuenta de cargo o cualquier configuración bancaria no modifica la orden.

La creación de la orden, la copia de datos bancarios, la asignación de movimientos y la generación del contenido deben constituir una operación consistente.

---

# Archivo TXT Mixto

- El archivo utiliza texto de longitud fija sin separadores.
- Cada línea representa un movimiento.
- Cada línea comienza con una clave bancaria de tres caracteres.
- El archivo puede combinar registros `PTC`, `PSC` y `CIL`.
- Cada línea termina con `CRLF`, bytes hexadecimales `0D 0A`.
- El archivo no utiliza encabezado ni pie.
- El archivo no incluye marca BOM.
- La moneda utilizada es `MXP`.
- Los importes utilizan dos decimales, punto decimal, justificación a la derecha y relleno con ceros.
- Los textos se normalizan sin acentos ni caracteres incompatibles con el layout.
- El contenido no incluye los campos opcionales de comprobante fiscal bancario, porque la factura del distribuidor se gestiona previamente mediante el proceso fiscal de EmailPro.
- Antes de confirmar la orden, cada línea debe validarse contra la longitud requerida por su tipo.
- Si un movimiento no puede representarse correctamente, no debe generarse una orden parcialmente inválida.
- La cuenta ordenante se obtiene de la configuración `debit_account` del entorno.
- La variable de entorno utilizada es `EMAILPRO_PAYMENT_ORDER_DEBIT_ACCOUNT`.
- La cuenta configurada debe validarse antes de generar una orden; si falta o es inválida, no se crea la orden ni un TXT parcial.
- La cuenta utilizada se copia en `payment_orders.debit_account` al crear la orden.
- Cambiar la configuración posteriormente solo afecta órdenes nuevas.

El nombre del archivo utiliza un formato equivalente a:

```text
ORDEN_SVR-EPO-<PAYMENT_ORDER_PUBLIC_ID>_YYYYMMDDHHMMSS.txt
```

Por ejemplo:

```text
ORDEN_SVR-EPO-A7K9M2Q4XZ_20261008000000.txt
```

---

# Layout PTC

El registro `PTC` corresponde a un pago a cuenta BBVA y mide 88 caracteres antes de `CRLF`.

| Campo | Longitud | Regla |
|---|---:|---|
| Clave de pago | 3 | Valor fijo `PTC` |
| Cuenta beneficiaria | 18 | Numérica, relleno con ceros a la izquierda |
| Cuenta ordenante | 18 | Numérica, relleno con ceros a la izquierda |
| Moneda | 3 | Valor fijo `MXP` |
| Importe | 16 | Dos decimales, relleno con ceros a la izquierda |
| Motivo de pago | 30 | Alfanumérico, relleno con espacios a la derecha |

---

# Layout PSC

El registro `PSC` corresponde a un pago interbancario mediante SPEI y mide 131 caracteres antes de `CRLF`.

| Campo | Longitud | Regla |
|---|---:|---|
| Clave de pago | 3 | Valor fijo `PSC` |
| CLABE beneficiaria | 18 | Numérica |
| Cuenta ordenante | 18 | Numérica, relleno con ceros a la izquierda |
| Moneda | 3 | Valor fijo `MXP` |
| Importe | 16 | Dos decimales, relleno con ceros a la izquierda |
| Titular beneficiario | 30 | Alfanumérico, normalizado y rellenado con espacios |
| Tipo de cuenta | 2 | Valor fijo `40` para CLABE |
| Clave bancaria | 3 | Clave numérica de la institución |
| Concepto | 30 | Alfanumérico, normalizado y rellenado con espacios |
| Referencia numérica | 7 | Referencia bancaria del movimiento |
| Disponibilidad | 1 | Valor fijo `H` |

La referencia de siete posiciones debe ser numérica y estable para el movimiento.

---

# Layout CIL

El registro `CIL` corresponde a un pago a convenio CIE y mide 124 caracteres antes de `CRLF`.

| Campo | Longitud | Regla |
|---|---:|---|
| Clave de pago | 3 | Valor fijo `CIL` |
| Concepto | 30 | Alfanumérico, normalizado y rellenado con espacios |
| Cuenta o convenio beneficiario | 7 | Numérico, relleno con ceros a la izquierda |
| Cuenta ordenante | 18 | Numérica, relleno con ceros a la izquierda |
| Importe | 16 | Dos decimales, relleno con ceros a la izquierda |
| Motivo de pago | 30 | Alfanumérico, normalizado y rellenado con espacios |
| Referencia CIE | 20 | Alfanumérica, normalizada y rellenada con espacios |

Los datos de convenio y referencia son capturados por el distribuidor y validados por SVR antes de habilitar la cuenta bancaria.

---

# Conceptos y Referencias

- El concepto debe permitir identificar el pago relacionado con el movimiento.
- El formato utilizado es `SVR-EPP-<PAYMENT_PUBLIC_ID>` y debe conservarse completo dentro de las 30 posiciones disponibles.
- El identificador público del pago debe conservarse completo en al menos uno de los campos devueltos por BBVA para permitir la conciliación dentro de la orden correspondiente.
- Los conceptos no dependen de texto capturado libremente por el frontend.
- Las referencias se generan en el servidor.
- Para `PSC`, la referencia numérica debe respetar exactamente siete posiciones.
- Para `CIL`, la referencia puede utilizar hasta veinte posiciones conforme a su configuración bancaria.

---

# Almacenamiento y Descarga

- El TXT se almacena en una ubicación privada.
- `payment_orders.txt_file` conserva el nombre interno del TXT generado.
- El nombre interno admite hasta 50 caracteres.
- Solo un Super Admin puede descargar el TXT.
- Cada descarga debe utilizar autorización y quedar auditada.
- Descargar nuevamente la orden entrega exactamente el mismo contenido.
- La aplicación no regenera el TXT utilizando información vigente después de crear la orden.

---

# Presentación en BBVA

- Un Super Admin descarga el TXT desde EmailPro.
- El Super Admin carga el archivo manualmente en el portal oficial de BBVA.
- BBVA ejecuta las validaciones y autorizaciones requeridas en su portal.
- Después de presentar el archivo, el Super Admin confirma en EmailPro que la orden fue enviada.
- La orden cambia de `generated` a `submitted`.
- La confirmación conserva el usuario y la fecha.
- Una orden `submitted` no puede cancelarse ni editarse.
- La confirmación en EmailPro no marca todavía los movimientos como pagados.

---

# Cancelación de una Orden

- Solo un Super Admin puede cancelar una orden.
- Solo puede cancelarse una orden con estado `generated`.
- La cancelación requiere un motivo obligatorio.
- Todos sus movimientos `pending` cambian a `cancelled` dentro de esa orden.
- Los pagos relacionados vuelven a ser elegibles para una orden posterior, siempre que sigan cumpliendo los requisitos.
- La orden cancelada, su TXT y su auditoría se conservan.
- Una orden descargada puede cancelarse si todavía no fue confirmada como presentada en BBVA.
- Una orden `submitted`, `partially_reconciled` o `completed` no puede cancelarse.

---

# Archivo de Conciliación

- BBVA entrega un archivo Excel con registros de longitud fija.
- Cada fila utilizada se encuentra en una celda de la primera columna.
- El archivo no requiere encabezados para su procesamiento.
- Una fila presente representa un movimiento procesado exitosamente por BBVA.
- La respuesta incluye información que permite relacionar el movimiento con la orden seleccionada y el identificador externo `SVR-EPP-<PAYMENT_PUBLIC_ID>` del pago.
- Los movimientos de la orden que no aparezcan en el archivo no se consideran automáticamente fallidos.
- Un archivo se asocia con una sola orden.
- Solo un Super Admin puede cargar el archivo.
- El archivo original se conserva en almacenamiento privado.
- Después de validar y guardar el archivo, `payment_orders.order_file` conserva su nombre interno.
- El nombre interno admite hasta 50 caracteres.
- Cada orden conserva como máximo un archivo de respuesta bancaria válido.
- Un archivo rechazado durante la validación no se conserva como respuesta aceptada ni se asigna a `order_file`.

---

# Validación del Excel

Antes de aplicar cualquier cambio, EmailPro valida el archivo completo.

La validación debe comprobar:

- Que el archivo sea un Excel permitido.
- Que su estructura corresponda al formato esperado.
- Que contenga al menos un registro procesable.
- Que todos los registros pertenezcan a la misma orden seleccionada.
- Que la orden exista y se encuentre en un estado conciliable.
- Que cada identificador `SVR-EPP-<PAYMENT_PUBLIC_ID>` corresponda a un movimiento de la orden.
- Que no existan dos filas con la misma referencia dentro del archivo.
- Que el tipo bancario coincida.
- Que la cuenta beneficiaria coincida con la conservada en el movimiento.
- Que la cuenta ordenante coincida con la conservada en la orden.
- Que el importe coincida exactamente.
- Que la moneda y los demás valores críticos sean válidos.
- Que el archivo no haya sido modificado después de su obtención cuando exista evidencia para comprobarlo.

Si una sola fila nueva es inválida, no se guarda ningún resultado del archivo.

Los registros válidos que ya fueron conciliados anteriormente no provocan un error ni duplican información; se reportan como ya procesados.

Si el archivo completo ya fue importado, la operación es idempotente y no cambia nuevamente los movimientos.

Un archivo de otra orden, un archivo alterado, un archivo parcialmente válido, un importe diferente o referencias duplicadas se rechazan por completo.

---

# Aplicación de la Conciliación

Después de validar todo el archivo:

- Cada fila presente cambia su movimiento correspondiente a `paid`, salvo que ya estuviera conciliado.
- Se conserva la fecha bancaria disponible en la respuesta.
- Se conservan las referencias y folios bancarios devueltos.
- Se conserva la fecha de importación y el usuario que realizó la carga.
- Cada movimiento de la orden que no aparezca cambia a `under_review`.
- La aplicación de todos los resultados se realiza dentro de una sola transacción.
- La importación no elimina ni reemplaza el archivo TXT original.
- El archivo de respuesta bancaria no reemplaza otro archivo previamente aceptado para la orden.

Si todos los movimientos quedan resueltos, la orden cambia a `completed`.

Si existe por lo menos un movimiento `pending` o `under_review`, la orden cambia a `partially_reconciled`.

---

# Conciliación Parcial

- Una orden puede contener movimientos pagados y movimientos bajo revisión.
- La interfaz muestra el estado general de la orden y el estado individual de cada movimiento.
- Los movimientos pagados permanecen cerrados aunque otros movimientos tengan incidencias.
- Una incidencia no revierte otros movimientos conciliados correctamente.
- La suma pagada, la suma pendiente y la suma fallida deben mostrarse por separado.

---

# Movimientos Ausentes

- Un movimiento ausente en el Excel cambia a `under_review`.
- La ausencia no demuestra por sí sola que BBVA rechazó o dejó de ejecutar la transferencia.
- Un movimiento `under_review` permanece bloqueado para nuevas órdenes.
- SVR investiga el resultado directamente en BBVA.
- Solo un Super Admin puede resolverlo manualmente.
- La resolución manual requiere un motivo obligatorio y evidencia o referencia cuando esté disponible.
- SVR puede resolverlo como `paid` o `failed`.
- Si se resuelve como `paid`, la liquidación concluye.
- Si se resuelve como `failed`, el pago vuelve a ser elegible para una orden posterior.

---

# Movimientos Rechazados o Fallidos

- Un movimiento confirmado como no pagado cambia a `failed`.
- El fallo no modifica el pago del cliente, la vigencia del dominio ni las facturas emitidas.
- El fallo conserva la orden, cuenta, importe, motivo y respuesta bancaria utilizados.
- Un movimiento fallido puede incorporarse a una orden posterior.
- El nuevo intento crea un nuevo movimiento bancario y conserva el intento anterior.
- Antes del reintento se validan nuevamente la cuenta bancaria vigente y todos los requisitos de elegibilidad.
- El mismo pago nunca puede tener dos intentos bancarios activos simultáneamente.

---

# Correcciones Manuales

- Solo un Super Admin puede realizar una resolución manual.
- No se permite editar directamente la cuenta, el importe, el concepto ni las referencias de un movimiento.
- La acción manual se limita a resolver una incidencia como pagada o fallida.
- Toda resolución requiere un motivo obligatorio.
- `updated_at` y `updated_by_id` conservan la fecha y el usuario de la última resolución.
- `status_reason` conserva el motivo de la última resolución.
- Un movimiento `paid` no puede cambiarse manualmente a pendiente para generar otra transferencia.

---

# Conclusión de la Orden

Una orden se considera completada cuando todos sus movimientos se encuentran en alguno de estos estados finales para esa orden:

- `paid`.
- `failed`.
- `cancelled`.

La conclusión de la orden no significa que todos sus movimientos fueron pagados; puede contener movimientos fallidos que serán reintentados en otra orden.

La interfaz debe mostrar claramente la cantidad y el importe de cada resultado.

---

# Notificaciones

- Cada movimiento que cambie a `paid` genera una notificación individual al distribuidor.
- La notificación informa el importe, pago relacionado, fecha de liquidación y referencia disponible.
- Dos movimientos pagados al mismo distribuidor generan dos notificaciones independientes.
- La reimportación de un registro ya pagado no genera otra notificación.
- Una resolución manual como `paid` también genera la notificación correspondiente.
- Las notificaciones fallidas no revierten la conciliación bancaria.

---

# Consulta del Distribuidor

El distribuidor puede consultar para cada pago de sus clientes:

- Importe que le corresponde.
- Fecha de confirmación del pago del cliente.
- Fecha de disponibilidad para liquidación.
- Estado de su factura hacia SVR.
- Estado de liquidación.
- Fecha real de liquidación cuando exista.
- Referencia del movimiento cuando pueda mostrarse.
- Factura asociada conforme a los permisos del proceso de facturación.

El distribuidor no puede consultar:

- Datos bancarios completos contenidos en el TXT.
- Movimientos de otros distribuidores.
- Cuenta de cargo de SVR.
- Archivos TXT o Excel.
- Acciones administrativas internas de conciliación.

---

# Consulta Administrativa

Solo un Super Admin puede:

- Consultar todas las órdenes.
- Consultar movimientos individuales.
- Descargar el TXT.
- Confirmar la presentación en BBVA.
- Cancelar una orden todavía no presentada.
- Importar el Excel de conciliación.
- Consultar los archivos almacenados.
- Resolver movimientos ausentes.
- Consultar auditoría, errores y motivos.

El listado de órdenes debe permitir consultar órdenes generadas, presentadas, parcialmente conciliadas, completadas y canceladas.

El detalle muestra por lo menos:

- Identificador público de la orden.
- Fecha de generación.
- Estado.
- Cantidad total de movimientos.
- Importe total.
- Cantidades e importes pagados, fallidos y pendientes.
- Usuario y fecha de presentación.
- Usuario y fecha de conciliación.
- Archivos asociados.

---

# Conservación de Archivos

- Los TXT generados y Excel importados se conservan en almacenamiento privado.
- Los archivos no se exponen mediante rutas públicas.
- Las descargas requieren autenticación y autorización de Super Admin.
- `payment_orders.txt_file` identifica el TXT exacto generado para la orden.
- `payment_orders.order_file` identifica el Excel de respuesta bancaria validado.
- La base de datos conserva únicamente los nombres internos; las rutas pertenecen a la configuración del almacenamiento privado.
- Cancelar o completar una orden no elimina sus archivos.

---

# Auditoría

- El estado actual de la orden y de cada movimiento se conserva mediante sus catálogos correspondientes.
- `updated_at` y `updated_by_id` identifican cuándo y quién realizó el último cambio de estado.
- `status_reason` conserva el motivo del último cambio cuando corresponda.
- `status_reason` es obligatorio para cancelar una orden y para resolver manualmente un movimiento.
- No se mantienen tablas independientes de acciones para órdenes o movimientos.
- La generación automática queda identificada mediante el mecanismo de usuario de sistema utilizado por EmailPro.

---

# Flujo Principal

```text
OpenPay confirma pago del cliente
    ↓
Calcular distributor_available_at con los días hábiles configurados
    ↓
EmailPro factura al cliente
    ↓
Distribuidor factura a SVR
    ↓
Esperar distributor_available_at y validar cuenta bancaria
    ↓
Proceso diario selecciona movimientos elegibles
    ↓
Generar orden y TXT mixto
    ↓
Super Admin descarga y presenta TXT en BBVA
    ↓
Super Admin confirma presentación
    ↓
BBVA procesa transferencias
    ↓
Super Admin importa Excel de resultados
    ↓
Validar archivo completo
    ↓
Conciliar movimientos presentes
    ↓
Enviar a revisión los movimientos ausentes
    ↓
Notificar cada liquidación confirmada
```

---

# Flujo de Reintento

```text
Movimiento resuelto como failed
    ↓
Cerrar intento dentro de la orden anterior
    ↓
Validar nuevamente requisitos y cuenta vigente
    ↓
Esperar siguiente generación automática
    ↓
Crear nuevo movimiento en otra orden
    ↓
Conservar ambos intentos en el historial
```

---

# Entidades Relacionadas

- Distributors
- Distributor Bank Accounts
- Payments
- Invoices
- Invoice Payments
- Payment Orders
- Payment Order Items
- Bank Holidays
- Users

---

# Observaciones

El proceso bancario es independiente del cobro realizado al cliente. Una falla o demora en la liquidación al distribuidor no revierte el pago, la vigencia del dominio ni los CFDI ya emitidos.

`distributor_available_at` evita intentar una transferencia antes de que SVR reciba normalmente los fondos del procesador. No existe una fecha máxima; una incidencia fiscal o bancaria puede retrasar la inclusión sin modificar la fecha calculada.

El identificador `SVR-EPP-<PAYMENT_PUBLIC_ID>` funciona como referencia principal de conciliación individual dentro de una orden y `SVR-EPO-<PAYMENT_ORDER_PUBLIC_ID>` identifica la orden completa. Las validaciones también deben comparar cuenta, tipo e importe para evitar asociaciones incorrectas.

Los layouts `PSC` y `CIL` fueron reconstruidos con un archivo real y la función histórica proporcionada. La implementación deberá utilizar pruebas de longitud exacta y archivos de muestra antes de operar en producción.
