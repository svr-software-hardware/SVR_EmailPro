# Gestión de Pagos

**Versión:** 0.1.0  
**Última actualización:** 2026-08-25

---

# Objetivo

Definir las reglas de negocio para el pago de la vigencia de los dominios y el cálculo de los importes correspondientes a SVR y a los distribuidores de EmailPro.

---

# Alcance

Incluye:

- Definición del costo por cuenta de correo que SVR asigna a cada distribuidor.
- Definición del precio por cuenta de correo que el distribuidor asigna a cada dominio.
- Selección del periodo de pago por parte del cliente.
- Cálculo del importe según la capacidad del dominio.
- Procesamiento del pago mediante el proveedor de pagos integrado.
- Registro de pagos exitosos.
- Conservación de los valores utilizados para calcular cada pago.
- Actualización de la fecha de expiración del dominio.
- Consulta de pagos por parte del cliente y del distribuidor.
- Cálculo del importe correspondiente a SVR y de la diferencia correspondiente al distribuidor.

No incluye:

- Intentos de pago fallidos.
- Definición de transacciones.
- Facturación.
- Tratamiento de impuestos.
- Descuentos.
- Promociones.
- Liquidaciones o depósitos a distribuidores.
- Almacenamiento de tarjetas.
- Suscripciones o renovaciones automáticas.
- Consulta o administración de pagos por parte de SVR.

---

# Configuración de Precios

- SVR define un costo por cuenta de correo para cada distribuidor.
- El costo asignado al distribuidor se utiliza para calcular la parte correspondiente a SVR.
- El distribuidor define un precio por cuenta de correo para cada dominio.
- Un mismo cliente puede tener precios diferentes en sus distintos dominios.
- El precio definido para un dominio debe ser, como mínimo, un peso mayor que el costo vigente asignado al distribuidor.
- El cliente puede consultar el precio de sus dominios, pero no puede modificarlo.
- SVR puede modificar el costo asignado al distribuidor.
- El distribuidor puede modificar el precio asignado a un dominio.
- Los cambios de costo o precio aplican únicamente a pagos futuros.
- Los pagos existentes conservan los valores utilizados cuando fueron realizados.
- Si un cambio en el costo de SVR provoca que el precio de un dominio deje una diferencia menor a un peso, el dominio no puede recibir un nuevo pago hasta que el distribuidor actualice su precio.
- El sistema no modifica automáticamente el precio definido por el distribuidor.

---

# Periodos de Pago

Los periodos disponibles son:

- Mensual: 1 mes.
- Trimestral: 3 meses.
- Semestral: 6 meses.
- Anual: 12 meses.

El periodo debe seleccionarse del catálogo de periodos de pago activos.

En el alcance actual, el importe se obtiene mediante una multiplicación directa y no contempla descuentos por periodo.

---

# Reglas de Cálculo

- El cobro se calcula utilizando la capacidad del dominio y no la cantidad de cuentas de correo activas.
- La capacidad utilizada corresponde al límite de cuentas de correo vigente en el dominio al momento del pago.
- El pago conserva una copia de la capacidad, el precio, el costo y la cantidad de meses utilizados en el cálculo.
- El importe total pagado por el cliente se calcula de la siguiente manera:

```text
importe_cliente = precio_unitario_cliente × capacidad_dominio × meses
```

- El importe correspondiente a SVR se calcula de la siguiente manera:

```text
importe_svr = costo_unitario_distribuidor × capacidad_dominio × meses
```

- La diferencia correspondiente al distribuidor se calcula de la siguiente manera:

```text
importe_distribuidor = importe_cliente - importe_svr
```

- El importe correspondiente al distribuidor representa una cantidad pendiente de liquidación y no confirma que haya sido depositada.

---

# Disponibilidad del Pago

- Solo el cliente puede iniciar el pago de uno de sus dominios.
- El pago únicamente puede iniciarse cuando falten cinco días o menos para la fecha de expiración del dominio.
- Un dominio vencido puede recibir un pago.
- Un dominio desactivado no puede recibir pagos.
- El dominio debe conservar un precio válido respecto al costo vigente del distribuidor.

La disponibilidad se determina de la siguiente manera:

```text
fecha_actual >= expires_at - 5 días
```

---

# Actualización de la Expiración

- La fecha de expiración se actualiza únicamente después de confirmar un pago exitoso.
- Si el dominio todavía no ha vencido, el periodo pagado se agrega a su fecha de expiración vigente.
- Si el dominio ya venció, el periodo pagado se agrega a la fecha del pago.

La fecha base se determina de la siguiente manera:

```text
fecha_base = mayor entre fecha_actual y expires_at
nueva_expiración = fecha_base + meses_pagados
```

- El pago conserva la fecha de expiración anterior y la nueva fecha de expiración.

---

# Reglas de Registro

- La entidad Payments contiene únicamente pagos exitosos.
- Un intento fallido no genera un registro de pago.
- Los intentos y resultados técnicos del proveedor se documentarán posteriormente mediante el proceso de transacciones.
- Cada pago pertenece a un único dominio.
- Cada pago conserva el periodo seleccionado.
- Cada pago conserva la cantidad de cuentas cobradas.
- Cada pago conserva el precio unitario aplicado al cliente.
- Cada pago conserva el costo unitario aplicado al distribuidor.
- Cada pago conserva la cantidad de meses pagados.
- Cada pago conserva el importe total pagado por el cliente.
- Cada pago conserva el importe correspondiente a SVR.
- Cada pago conserva el importe correspondiente al distribuidor.
- Cada pago conserva el identificador externo de la transacción exitosa proporcionado por el proveedor de pagos.
- Cada pago conserva la fecha de expiración anterior y la nueva fecha de expiración.
- La información utilizada para calcular un pago no se modifica aunque posteriormente cambien el costo, el precio o la capacidad del dominio.
- Los pagos constituyen registros históricos y no se eliminan físicamente.

---

# Permisos de Consulta

- El cliente puede consultar los pagos pertenecientes a sus dominios.
- El distribuidor puede consultar los pagos de los dominios pertenecientes a sus clientes.
- El distribuidor no puede iniciar pagos en nombre de un cliente.
- En el alcance actual, SVR no puede consultar ni administrar pagos.

---

# Flujo Principal

```text
Cliente
    ↓
Seleccionar dominio disponible para pago
    ↓
Seleccionar periodo
    ↓
Calcular importes con los valores vigentes
    ↓
Proporcionar información de pago
    ↓
Procesar operación mediante el proveedor de pagos
    ↓
Confirmar pago exitoso
    ↓
Registrar evidencia histórica del pago
    ↓
Actualizar fecha de expiración del dominio
```

---

# Entidades

- Distributors
- Clients
- Domains
- Payment Terms
- Payments

---

# Observaciones

El pago pertenece al dominio porque cada dominio puede tener una capacidad, un precio y una vigencia diferentes.

Los importes se conservan como valores finales de la operación. El tratamiento de impuestos se definirá posteriormente junto con el proceso de facturación.

La diferencia calculada para el distribuidor deberá utilizarse posteriormente en un proceso independiente de liquidaciones.

La integración técnica con el proveedor de pagos, la autenticación reforzada y el manejo de respuestas técnicas pertenecen a la librería y a la implementación de la API, no a este proceso.

Los procesos relacionados con transacciones, facturación, liquidaciones, tarjetas, suscripciones y acceso futuro de SVR deberán documentarse de forma independiente cuando sean requeridos por el negocio.
