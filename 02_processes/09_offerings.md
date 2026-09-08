# Gestión de Productos y Servicios

**Versión:** 0.2.0  
**Última actualización:** 2026-09-08

---

# Objetivo

Definir las reglas de negocio para organizar y administrar los productos y servicios mostrados en las páginas web de EmailPro.

---

# Alcance

Incluye:

- Creación y administración de categorías por parte del cliente.
- Creación y administración unificada de productos y servicios.
- Asociación de productos o servicios a categorías.
- Captura de nombre, descripción, imagen y precio opcional.
- Configuración del orden de categorías y productos o servicios.
- Activación y desactivación de categorías y productos o servicios.
- Administración por parte del distribuidor responsable del cliente.
- Consulta pública del contenido mediante una API.

No incluye:

- Inventarios.
- Existencias.
- Variantes.
- Códigos de producto.
- Carrito de compras.
- Pedidos.
- Cobros o pagos.
- Envíos.
- Impuestos.
- Descuentos.
- Promociones.
- Diferenciación estructural entre productos y servicios.
- Almacenamiento de imágenes en Base64.

---

# Reglas de Negocio

- Las categorías pertenecen a una única página web.
- Una página web puede tener múltiples categorías.
- El cliente puede crear, modificar, ordenar, activar y desactivar las categorías de su página web.
- El distribuidor responsable del cliente puede crear, modificar, ordenar, activar y desactivar las categorías de la página web.
- Un distribuidor únicamente puede administrar categorías y productos o servicios pertenecientes a páginas de sus propios clientes.
- Las categorías nunca se eliminan físicamente; únicamente pueden desactivarse.
- Cada producto o servicio pertenece a una única categoría.
- Una categoría puede contener múltiples productos o servicios.
- Los productos y servicios se administran mediante una sola entidad.
- El sistema no requiere clasificar un elemento como producto o como servicio.
- El cliente puede crear, modificar, ordenar, activar y desactivar sus productos o servicios.
- El distribuidor responsable del cliente puede crear, modificar, ordenar, activar y desactivar los productos o servicios de la página web.
- Los productos y servicios nunca se eliminan físicamente; únicamente pueden desactivarse.

---

# Categorías

Cada categoría puede contener:

- Nombre.
- Descripción opcional.
- Orden de presentación.

El nombre de una categoría debe ser único dentro de la misma página web.

Una categoría desactivada no se muestra públicamente junto con sus productos o servicios.

---

# Productos y Servicios

Cada producto o servicio contiene:

- Nombre.
- Descripción.
- Imagen.
- Precio opcional.
- Orden de presentación.

## Reglas

- La imagen se conserva mediante su nombre interno y no como contenido Base64.
- El precio puede permanecer vacío cuando el negocio no desea publicar un importe.
- Cuando el precio está vacío, la página no muestra un importe numérico.
- El precio representa únicamente información pública y no habilita ventas en línea.
- Un producto o servicio desactivado no se muestra públicamente.
- El nombre de un producto o servicio debe ser único dentro de su categoría.

---

# Consulta Pública

- La API entrega únicamente categorías activas.
- La API entrega únicamente productos o servicios activos pertenecientes a categorías activas.
- Las categorías se entregan según el orden definido por el cliente.
- Los productos o servicios se entregan según el orden definido dentro de su categoría.
- Las imágenes se entregan mediante una referencia de archivo o URL generada por la implementación.

---

# Flujo Principal

```text
Cliente
    ↓
Seleccionar su página web
    ↓
Crear o seleccionar categoría
    ↓
Registrar producto o servicio
    ↓
Capturar nombre, descripción, imagen y precio opcional
    ↓
Definir orden de presentación
    ↓
Contenido disponible mediante la API
```

---

# Entidades

- Websites
- Website Categories
- Website Offerings

---

# Observaciones

Los productos y servicios se unifican porque actualmente comparten los mismos atributos y comportamiento.

Si en el futuro requieren reglas distintas, el proceso deberá evolucionar antes de modificar el modelo de datos.

El contenido se conserva en entidades relacionadas y la API construye la respuesta necesaria para el frontend. No se almacena como un documento JSON completo.

Los procesos relacionados con inventarios, pedidos, ventas en línea y otras funcionalidades comerciales deberán documentarse de forma independiente cuando sean requeridos por el negocio.
