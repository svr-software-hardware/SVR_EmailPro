# Gestión de Páginas Web

**Versión:** 0.2.1  
**Última actualización:** 2026-09-08

---

# Objetivo

Definir las reglas de negocio para la activación y administración de la página web asociada a cada dominio de EmailPro.

---

# Alcance

Incluye:

- Activación de una página web por parte del cliente o de su distribuidor.
- Asociación de una página web a un dominio.
- Activación y desactivación posterior de la página.
- Administración de la identidad visual general.
- Administración del banner principal.
- Administración de la sección Nosotros.
- Administración de un carrusel de imágenes.
- Administración del aviso de privacidad.
- Administración de la información de contacto.
- Administración de secciones personalizadas.
- Configuración de la visibilidad y el orden de las secciones.
- Modificación de la página por parte del distribuidor responsable del cliente.
- Consulta pública de la información mediante una API.

No incluye:

- Categorías, productos o servicios.
- Inventarios.
- Ventas en línea.
- Carrito de compras.
- Pagos desde la página web.
- Historial de versiones o publicaciones.
- Plantillas visuales múltiples.
- Almacenamiento de imágenes en Base64.

---

# Reglas de Negocio

- Cada página web pertenece a un único dominio.
- Cada dominio puede tener como máximo una página web.
- La página web no se crea automáticamente al registrar el dominio.
- El cliente propietario del dominio y su distribuidor pueden activar inicialmente la página web.
- El cliente puede modificar la información de su página web.
- El distribuidor responsable del cliente puede modificar la información de la página web.
- Un distribuidor únicamente puede modificar páginas pertenecientes a sus propios clientes.
- El cliente y su distribuidor pueden desactivar y volver a activar la página web.
- La desactivación de la página web no elimina su contenido.
- Las páginas web nunca se eliminan físicamente; únicamente pueden desactivarse.
- La activación o desactivación de la página web no modifica el estado del dominio ni de sus cuentas de correo.

---

# Identidad Visual

- El cliente y su distribuidor pueden definir el nombre mostrado en la página.
- El cliente y su distribuidor pueden cargar un logotipo.
- El cliente y su distribuidor pueden definir un color primario y un color secundario.
- El cliente y su distribuidor pueden definir el color de fondo y el color del texto del banner.
- Todos los colores se almacenan en formato hexadecimal `#RRGGBB`.
- Los colores configurados aplican de forma general a la página.
- En el alcance actual, no se almacenan colores individuales para cada sección o elemento.
- Las imágenes se almacenan como archivos y la base de datos conserva únicamente sus nombres internos.
- La API no entrega imágenes codificadas en Base64.

---

# Secciones

La página puede contener las siguientes secciones:

- Inicio y banner principal.
- Nosotros.
- Carrusel de imágenes.
- Productos y servicios.
- Contacto.
- Aviso de privacidad.
- Secciones personalizadas.

## Reglas de Secciones

- El cliente y su distribuidor pueden mostrar u ocultar las secciones configurables.
- El cliente y su distribuidor pueden modificar el orden en que se presentan las secciones.
- Cada sección conserva su título, subtítulo y descripción únicamente cuando correspondan.
- Los tipos de sección disponibles son `home`, `about`, `carousel`, `offerings`, `contact`, `privacy_notice` y `custom`.
- Una página puede contener como máximo una sección de cada tipo predeterminado.
- Una página puede contener múltiples secciones de tipo `custom`.
- La API debe impedir la duplicación de los tipos predeterminados; esta regla no se representa mediante una restricción única porque el tipo `custom` puede repetirse.
- La sección de productos y servicios obtiene su contenido mediante el proceso de categorías y productos o servicios.
- La información de contacto se administra mediante una entidad propia y no como contenido genérico de una sección.

## Secciones Personalizadas

- Las secciones personalizadas se administran mediante la misma entidad utilizada para las secciones predeterminadas.
- Se identifican con el tipo `custom` y una página puede contener múltiples registros de este tipo.
- El cliente y su distribuidor pueden crear secciones adicionales para contenido que no corresponda a una sección especializada.
- Cada sección personalizada contiene título, descripción e imagen opcional.
- Cada sección personalizada puede ordenarse, activarse y desactivarse.
- Pueden utilizarse para contenido como eventos, promociones, misión, visión, valores u otra información libre.
- No deben utilizarse para sustituir secciones con estructura propia, como contacto, carrusel o productos y servicios.
- Las secciones personalizadas nunca se eliminan físicamente; únicamente pueden desactivarse.
- Las imágenes se conservan mediante su nombre interno y no mediante contenido Base64.

---

# Banner Principal

- El cliente y su distribuidor pueden definir el título, subtítulo y descripción del banner.
- El cliente y su distribuidor pueden cargar una imagen para el banner.
- El banner utiliza los colores generales configurados para su fondo y texto.
- La imagen se conserva mediante su nombre interno y no mediante contenido Base64.

---

# Carrusel de Imágenes

- Una página web puede tener múltiples elementos en su carrusel.
- Cada elemento pertenece a una única página web.
- Cada elemento contiene una imagen.
- Cada elemento puede contener título y descripción.
- El cliente y su distribuidor pueden modificar el orden de los elementos.
- El cliente y su distribuidor pueden activar o desactivar elementos sin eliminarlos físicamente.

---

# Información de Contacto

- Cada página web puede tener como máximo un registro de contacto.
- El contacto puede incluir teléfono, WhatsApp, correo electrónico, dirección y enlace de ubicación.
- El contacto puede incluir enlaces de Facebook e Instagram.
- Los enlaces para llamar, enviar correo o abrir WhatsApp se generan a partir de la información registrada y no se almacenan duplicados.
- El cliente y su distribuidor pueden actualizar la información de contacto.

---

# Información Derivada

La página web no almacena información que ya pertenece a otras entidades de EmailPro.

No deberán duplicarse:

- Nombre y extensión del dominio.
- Fecha de expiración del dominio.
- Días restantes de vigencia.
- Cuentas de correo asociadas.
- Información fiscal del cliente.

La API obtiene estos valores desde sus entidades originales cuando sean necesarios.

---

# Flujo Principal

```text
Cliente o distribuidor
    ↓
Seleccionar uno de sus dominios
    ↓
Activar página web
    ↓
Configurar identidad visual y banner
    ↓
Registrar contenido institucional
    ↓
Registrar imágenes del carrusel
    ↓
Registrar información de contacto
    ↓
Configurar visibilidad y orden de secciones
    ↓
Página disponible mediante la API
```

---

# Entidades

- Domains
- Websites
- Website Sections
- Website Carousel Items
- Website Contacts

---

# Observaciones

La base de datos constituye la fuente de la información editable. La API reúne los registros relacionados y entrega al frontend la estructura necesaria para representar la página.

La estructura de respuesta de la API no constituye el modelo de almacenamiento y puede evolucionar sin duplicar la información del dominio o del cliente.

Los archivos de imagen deberán gestionarse mediante el mecanismo de almacenamiento definido por la implementación. Los YAML conservarán únicamente los nombres internos requeridos por el negocio.

Los procesos relacionados con productos y servicios, plantillas, versiones y comercio electrónico deberán documentarse de forma independiente cuando sean requeridos por el negocio.
