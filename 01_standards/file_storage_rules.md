# Reglas de Almacenamiento de Archivos

**Versión:** 1.0.0  
**Última actualización:** 2026-09-08

---

# Objetivo

Definir las reglas generales para almacenar, organizar, reemplazar y publicar los archivos utilizados por EmailPro.

Estas reglas describen el comportamiento requerido de forma independiente del framework, proveedor o servicio de almacenamiento utilizado por la implementación.

---

# Alcance Actual

Este estándar aplica actualmente a las imágenes públicas de las páginas web de EmailPro:

- Logotipos.
- Imágenes principales de secciones.
- Elementos de carrusel.
- Imágenes de productos o servicios.

No define todavía el almacenamiento de documentos privados ni de archivos pertenecientes a otros procesos.

---

# Principios

- La base de datos almacena únicamente el nombre interno del archivo.
- La base de datos no almacena rutas físicas, URLs, contenido binario ni información en Base64.
- La ubicación del archivo se determina mediante la configuración de almacenamiento y la relación del registro con su página web.
- Las URLs públicas se generan al construir la respuesta de la API.
- Las rutas físicas y la configuración interna del proveedor nunca se entregan al frontend.
- Los archivos públicos de páginas web se mantienen separados de cualquier almacenamiento privado del sistema.

---

# Almacenamiento Público de Páginas Web

- Las imágenes utilizadas por las páginas web se almacenan en un espacio público dedicado.
- La implementación debe configurar un único almacenamiento lógico para estos recursos.
- No se requiere un almacenamiento diferente para cada tabla.
- El almacenamiento puede implementarse mediante el sistema de archivos del servidor o mediante un proveedor de objetos o CDN.
- Cambiar el proveedor de almacenamiento no debe requerir modificar los nombres conservados en la base de datos.

La organización lógica es:

```text
websites/
└── {website_id}/
    ├── logos/
    ├── sections/
    ├── carousel/
    └── offerings/
```

Cada directorio conserva:

- `logos`: logotipo de la entidad `websites`.
- `sections`: imágenes de `website_sections`.
- `carousel`: imágenes de `website_carousel_items`.
- `offerings`: imágenes de `website_offerings`.

El identificador de la página forma parte de la ruta lógica, pero no del nombre almacenado en la base de datos.

---

# Nombres Internos

- El nombre original proporcionado por el usuario no se utiliza como nombre de almacenamiento.
- Cada archivo recibe un nombre aleatorio de 40 caracteres alfanuméricos.
- El nombre conserva la extensión validada en minúsculas.
- El formato es `{40_caracteres}.{extension}`.
- La extensión puede contener como máximo cuatro caracteres.
- El nombre completo ocupa como máximo 45 caracteres.
- Los campos de base de datos que conservan imágenes utilizan `string 45`.
- El nombre aleatorio debe impedir colisiones dentro del almacenamiento.

Ejemplo:

```text
aB3dE5fG7hJ9kL2mN4pQ6rS8tU1vW3xY5zA7bC9d.webp
```

---

# Validación de Imágenes

- La API debe validar el archivo antes de almacenarlo.
- La validación debe comprobar el tipo MIME real y no únicamente la extensión declarada.
- Los formatos permitidos son `jpg`, `jpeg`, `png`, `webp` y `avif`.
- No se permiten archivos SVG dentro del alcance actual porque pueden contener contenido ejecutable.
- No se permiten archivos cuya extensión no corresponda a su contenido real.
- El tamaño máximo permitido por archivo es de 5 MB.
- Un archivo rechazado no debe modificar el nombre almacenado ni eliminar la imagen vigente.

---

# Registro y Reemplazo

Para registrar una imagen:

1. Validar el archivo.
2. Generar su nombre interno.
3. Almacenar el archivo en su ruta lógica.
4. Guardar únicamente el nombre interno en la entidad correspondiente.

Para reemplazar una imagen:

1. Validar la nueva imagen.
2. Almacenar la nueva imagen con un nombre diferente.
3. Actualizar el nombre en la base de datos.
4. Eliminar la imagen anterior únicamente después de confirmar la actualización.

Si la actualización de la base de datos falla, la implementación debe retirar la nueva imagen y conservar la anterior.

La ausencia de una nueva imagen en una solicitud de actualización no significa que la imagen vigente deba eliminarse.

---

# Eliminación

- La eliminación de una imagen debe solicitarse explícitamente.
- Una actualización de otros campos no elimina ni reemplaza la imagen vigente.
- Al retirar explícitamente una imagen, primero se actualiza su campo a `null` y después se elimina el archivo correspondiente.
- Desactivar una página, sección, elemento de carrusel, categoría o producto o servicio no elimina sus archivos.
- La eliminación física de archivos no debe depender de valores de ruta proporcionados por el usuario.

---

# Resolución de URLs

- La API de contenido obtiene el nombre interno desde la entidad correspondiente.
- La API determina la ruta lógica utilizando el identificador de la página y el tipo de recurso.
- El mecanismo de almacenamiento genera la URL pública del archivo.
- La respuesta entrega esa URL mediante `logo_url` o `image_url`.
- Cuando el campo de imagen es `null` o el archivo no existe, la URL se entrega como `null`.
- La falta de una imagen opcional no impide publicar el resto de la página.
- En el modelo actual no se requiere una API que lea y transforme cada imagen a Base64.
- El navegador obtiene directamente el archivo desde su URL pública.

---

# Seguridad

- Las operaciones para cargar, reemplazar o retirar archivos requieren los mismos permisos definidos para administrar la página web.
- La lectura de las imágenes publicadas es pública.
- Los nombres y rutas recibidos del usuario nunca se utilizan directamente para leer o eliminar archivos.
- La implementación debe evitar recorridos de ruta y acceso a directorios diferentes del asignado.
- El almacenamiento público de páginas web no debe contener archivos privados del cliente o del sistema.

---

# Responsabilidades de la Implementación

La implementación debe:

- Configurar el almacenamiento público de páginas web.
- Aplicar las rutas lógicas definidas por este estándar.
- Validar las imágenes antes de almacenarlas.
- Garantizar la consistencia entre el archivo y el nombre conservado en la base de datos.
- Generar URLs según el entorno actual sin persistirlas.
- Permitir que el dominio público o el proveedor de archivos cambien mediante configuración.

Los detalles específicos del framework, como archivos de configuración, enlaces simbólicos, controladores o servicios, pertenecen al repositorio de implementación y no a esta especificación.

---

# Principio Fundamental

La base de datos identifica el archivo; el almacenamiento conserva su contenido y la API resuelve su ubicación pública.
