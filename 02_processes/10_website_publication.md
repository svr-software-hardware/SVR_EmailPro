# Publicación de Páginas Web

**Versión:** 0.1.1  
**Última actualización:** 2026-09-08

---

# Objetivo

Definir las reglas para publicar una página web de EmailPro y entregar al frontend el contenido general necesario para representarla.

---

# Alcance

Incluye:

- Consulta pública de una página web.
- Validación de la disponibilidad de la página y de su dominio.
- Entrega de la identidad visual general.
- Entrega de la navegación principal.
- Entrega ordenada de las secciones activas.
- Entrega de los elementos activos del carrusel.
- Entrega de la información de contacto.
- Entrega de URLs para consultar imágenes.
- Contrato JSON de contenido general utilizado por el frontend.

No incluye:

- Resolución definitiva de páginas mediante subdominios.
- Infraestructura de hospedaje o publicación.
- Envoltorio estándar de respuestas de la API de SVR.
- Categorías.
- Productos o servicios.
- Administración de la página web.
- Almacenamiento de documentos JSON en la base de datos.
- Almacenamiento o entrega de imágenes en Base64.

---

# Reglas de Disponibilidad

- La consulta pública no requiere que el visitante inicie sesión.
- La página únicamente puede publicarse cuando el dominio asociado se encuentra activo y vigente.
- La página debe encontrarse activa.
- La fecha de expiración del dominio se considera vigente durante el día indicado en `expires_at`.
- Si no existe una página asociada al dominio solicitado, la API responde con `404 Not Found`.
- Si la página se encuentra desactivada, la API responde con `404 Not Found`.
- Si el dominio se encuentra desactivado, la API responde con `404 Not Found`.
- Si la fecha actual es posterior a `expires_at`, la API responde con `404 Not Found`.
- El frontend determina si representa el error como página no encontrada o página en construcción.
- El código `422 Unprocessable Content` se reserva para solicitudes con datos de entrada inválidos y no se utiliza para indicar que una página pública no está disponible.

---

# Resolución Temporal de la Página

- Durante la etapa de pruebas se utilizará un único subdominio configurado en el entorno de implementación.
- La página utilizada para las pruebas también se determinará mediante la configuración de ese entorno.
- El subdominio temporal no se almacena en la base de datos.
- El modelo actual no incorpora un campo de subdominio en la entidad `websites`.
- La resolución definitiva de cada página a partir de su subdominio deberá documentarse cuando se defina la infraestructura de publicación.
- La definición futura del subdominio no deberá modificar la estructura del contenido general entregado al frontend.

---

# Contenido de la Respuesta

La respuesta contiene tres elementos raíz:

- `website`: identidad y colores generales de la página.
- `navbar`: enlaces de navegación generados a partir de las secciones predeterminadas.
- `sections`: secciones activas en el orden en que deben mostrarse.

No se entregan:

- Llaves foráneas.
- Campos de auditoría.
- Nombres internos de archivos.
- Estados internos de activación.
- Información fiscal.
- Capacidad o cuentas de correo del dominio.
- Fecha de expiración del dominio.
- Categorías, productos o servicios.

---

# Identidad Visual

El objeto `website` contiene:

- `display_name`.
- `logo_url`.
- `colors.primary`.
- `colors.secondary`.
- `colors.banner_background`.
- `colors.banner_text`.

Los colores se entregan en formato hexadecimal `#RRGGBB`.

`logo_url` contiene la URL utilizable por el frontend y nunca el nombre interno del archivo. Cuando no existe un logotipo, su valor es `null`.

---

# Barra de Navegación

- `navbar` es un arreglo generado por la API.
- La API genera sus elementos a partir de las secciones activas cuyo tipo sea diferente de `custom`.
- El orden de sus elementos corresponde al orden configurado para las secciones.
- El texto visible de cada elemento se obtiene del campo `title` de la sección.
- El destino se construye agregando `#` antes del tipo de sección.
- Por ejemplo, una sección con tipo `about` genera el destino `#about`.
- El frontend asigna el valor de `type` como identificador HTML de cada sección predeterminada.
- Las secciones de tipo `custom` no se incluyen en `navbar`.
- Las secciones de tipo `custom` no utilizan su tipo como identificador HTML, debido a que puede existir más de una.
- Si no existen secciones predeterminadas activas, `navbar` se entrega como un arreglo vacío.

Cada elemento contiene:

- `label`: texto mostrado en la barra de navegación.
- `target`: selector de la sección a la que dirige el enlace.

La barra de navegación es información derivada y no se almacena en una entidad independiente.

---

# Secciones

- `sections` es un arreglo.
- La posición de cada elemento en el arreglo representa su orden de presentación.
- La API entrega únicamente secciones activas.
- Todos los tipos de sección conservan una estructura base común.
- Los campos opcionales que no correspondan a una sección se entregan con valor `null`.
- Una página puede entregar múltiples secciones con el tipo `custom`.

Cada sección contiene:

- `type`.
- `title`.
- `subtitle`.
- `description`.
- `image_url`.

Los tipos permitidos son:

- `home`.
- `about`.
- `carousel`.
- `offerings`.
- `contact`.
- `privacy_notice`.
- `custom`.

La sección `offerings` entrega únicamente su contenido general. Sus categorías, productos y servicios se consultan mediante contratos independientes.

---

# Carrusel

- La sección de tipo `carousel` incorpora la propiedad `items`.
- `items` es un arreglo ordenado.
- La API entrega únicamente elementos activos.
- Cada elemento contiene `title`, `description` e `image_url`.
- Cuando el carrusel no tiene elementos activos, `items` se entrega como un arreglo vacío.
- Las demás secciones no incorporan la propiedad `items`.

---

# Contacto

- La sección de tipo `contact` incorpora la propiedad `contact`.
- `contact` contiene únicamente los medios registrados para la página.
- Los valores opcionales no registrados se entregan como `null`.
- La API puede generar URLs utilizables para teléfono, WhatsApp y correo a partir de sus valores originales.
- Estas URLs derivadas no se almacenan en la base de datos.
- Si no existe un registro de contacto, `contact` se entrega con valor `null`.
- Las demás secciones no incorporan la propiedad `contact`.

El objeto de contacto puede contener:

- `phone`.
- `phone_url`.
- `whatsapp`.
- `whatsapp_url`.
- `email`.
- `email_url`.
- `address`.
- `location_url`.
- `facebook_url`.
- `instagram_url`.

---

# Imágenes

- La base de datos conserva únicamente los nombres internos de los archivos.
- La API transforma esos nombres en URLs utilizables por el frontend.
- Los campos `logo_url` e `image_url` nunca contienen imágenes codificadas en Base64.
- Cuando una imagen opcional no existe, su URL se entrega como `null`.

---

# Contrato JSON

El archivo `04_api_contracts/01_public_website_content.json` contiene un ejemplo completo del objeto requerido por el frontend.

El contrato representa únicamente los datos funcionales. La implementación puede incorporarlo dentro del envoltorio estándar de respuestas de SVR sin modificar su estructura interna.

---

# Flujo Principal

```text
Visitante
    ↓
Solicitar página pública
    ↓
Resolver página configurada para el entorno
    ↓
Validar existencia y estado de la página
    ↓
Validar estado y vigencia del dominio
    ↓
Obtener identidad visual
    ↓
Obtener secciones y elementos activos en su orden de presentación
    ↓
Generar barra de navegación
    ↓
Transformar nombres de imágenes en URLs
    ↓
Entregar contenido general al frontend
```

---

# Entidades

- Domains
- Websites
- Website Sections
- Website Carousel Items
- Website Contacts

---

# Pendientes

- Definir la infraestructura donde se publicarán las páginas.
- Definir el dominio base de publicación.
- Definir la asociación definitiva entre subdominios y páginas web.
- Incorporar en frontend y backend la validación definitiva del nombre utilizado para formar dominios o subdominios.
- Definir los contratos públicos de categorías y productos o servicios.

---

# Observaciones

La respuesta se construye a partir de las entidades normalizadas de EmailPro. El JSON es un contrato de lectura para el frontend y no representa una tabla ni un documento almacenado en la base de datos.

La estructura basada en un arreglo de secciones permite respetar el orden configurado e incorporar múltiples secciones personalizadas sin modificar el contrato.
