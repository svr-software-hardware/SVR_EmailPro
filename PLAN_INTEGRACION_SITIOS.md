# Revisión y plan de integración del gestor web

Fecha: 2026-09-14. Estado: creación, identidad visual y listado de secciones implementados para cliente y distribuidor.

## Alcance acordado

Primero revisar ambos proyectos y preparar el traslado visual. Integrar inicialmente la creación; enseñar el patrón para uno o dos CRUDs y dejar los demás para trabajo guiado del usuario. Los documentos existentes se consultaron como contexto, no como instrucciones adicionales del usuario.

## Mapa de los proyectos

Raíz de origen: `C:/wamp64/www/svr/EmailProVerito/`.
Raíz de destino: `C:/wamp64/www/svr/SVR_EmailPro_WEB/`.

| Responsabilidad | Archivo relativo a su proyecto |
|---|---|
| Vista inicial y confirmación en origen | `src/views/website_manager/Index.vue` |
| Identidad y edición de secciones en origen | `src/views/website_manager/Form.vue` |
| Vista previa en origen | `src/views/website_manager/Preview.vue` |
| Componentes visuales en origen | `src/components/websiteManager/` y `src/components/ConfirmDialog.vue` |
| Datos simulados del gestor en origen | `src/store/websiteManager.js` |
| Dominio y cliente simulados en origen | `src/composables/useWebsiteManager.js`, `src/store/distributor.js` |
| Entrada desde dominios en destino | `src/views/client_section/domains/List.vue` |
| Registro de rutas en destino | `src/router/routes.js` |
| Protección en destino | `src/router/middleware/Auth.js`, `src/router/middleware/Roles.js` |
| Sesión en destino | `src/store/index.js` |
| HTTP y errores en destino | `src/utils/http.js` |
| Base de API en destino | `src/utils/config.js` |

Ambos usan Vue y Vuetify. El origen usa un objeto reactivo y localStorage para simular persistencia, sin integración de APIs en el gestor. Sus rutas distinguen roles por metadatos; el destino usa token y roles numéricos. El destino todavía no registra rutas de gestor.

## Rutas previstas

Primera etapa: `/cliente/dominios/:domainId/gestor`, protegida con `[Auth, Roles([4])]` y conectada desde la acción de administrar página web en Mis dominios.

En etapas posteriores: `/identidad`, `/secciones/:sectionId` (incluye `nueva`) y `/vista-previa`, bajo el mismo prefijo protegido. No habilitar guardados simulados como si fueran persistencia real.

El origen también ofrece un gestor al distribuidor en `/distribuidor/clientes/:id/dominios/:domainId/gestor`. Queda fuera de la primera integración cliente hasta acordar su contrato y alcance; el rol correspondiente en destino es 3.

El destino ya tiene sitio público en `/website/:id` y catálogo en `/website/:id/catalogo/:categoryId`. El enlace `/sitio?domain=...` del origen no puede copiarse sin adaptar la navegación y confirmar el identificador real del sitio.

## API y datos

La configuración local existente produce `URL_API = https://apidevemailpro.svr.systems/api`. Por tanto, el POST se forma con `${URL_API}/v1/client/websites`; no duplicar `/api` ni fijar el host en la vista.

El helper `request` ya detecta FormData, omite Content-Type para que el navegador agregue el boundary, y admite `token: store.getAuth?.token`. `getRsp` lee el envoltorio message/data/errors y `getErr` traduce errores.

Ya hay llamadas autenticadas a:

- `GET /v1/client/domains?is_active=1` y `is_active=0`, con `data.items`.
- `GET /v1/client/domains/:id`, con `data.item`.
- `GET /v1/client/me`, para información del cliente.

El nombre completo del dominio se arma con `name` y `domain_extension.name`. La consulta autenticada del sitio es `GET /v1/client/websites/:domain_id` y entrega `data.item`, incluida la relación `domain`. El POST entrega el registro creado en la misma ubicación del envoltorio.

| Campo del POST | Valor solicitado |
|---|---|
| domain_id | Identificador de la ruta, validado contra los dominios del cliente |
| display_name | `Mi sitio web` |
| logo_filename | Cadena vacía en multipart; el backend lo devuelve como `null` |
| primary_color | `#646464` |
| secondary_color | `#9C9C9C` |
| banner_background_color | `#646464` |
| banner_text_color | `#212121` |

El ejemplo del usuario tiene una comilla de cierre faltante en primary_color; el valor hexadecimal legible es #646464. FormData no representa null nativo; no enviar la cadena "null" sin contrato.

## Secuencia de implementación

1. Completado: registrar rutas protegidas y accesos para cliente y distribuidor.
2. Completado: resolver dominio y sitio desde backend, tratando únicamente 404 como inexistencia.
3. Completado: confirmar, volver a consultar, construir FormData y ejecutar un único POST con bloqueo durante la solicitud.
4. Completado: consultar y actualizar la identidad visual mediante GET y PUT multipart para ambos roles.
5. Completado: representar el logo Base64 recibido como una URL `data:` y enviar como archivo únicamente un logo nuevo.
6. Completado: consultar secciones activas e inactivas, unirlas por ID y ordenarlas por `sort_order`.
7. Completado: agregar al final los tipos predeterminados faltantes como “No creada”; `custom` sólo aparece cuando existe.
8. Completado: integrar creación y edición compartidas de Home, About, Privacy Notice y Custom mediante POST/PUT multipart.
9. Completado: agregar subtítulo y manejo de imagen; Privacy Notice no acepta imagen. El estado no aparece en este formulario ni se envía en POST/PUT, porque se modifica mediante la API específica de activación.
10. Completado: paginar el listado de secciones de 10 en 10, activar/inactivar cada sección y guardar el nuevo orden con una sola petición PATCH cuando haya cambios.
11. Completado: formulario especializado de Contacto para cliente y distribuidor, con creación y edición de teléfono, WhatsApp, correo, dirección, ubicación, Facebook e Instagram. El chip de Estado ejecuta activar/inactivar y el chip “No creada” abre el formulario disponible para crearla.
12. Completado: formulario especializado de Carrusel para cliente y distribuidor. La sección guarda título y descripción; cada elemento guarda individualmente título, descripción e imagen. El orden de elementos se persiste en una petición separada y la visibilidad permanece oculta hasta contar con sus APIs.
13. Completado: formulario especializado de Productos y servicios para cliente y distribuidor. Exige crear primero la sección, lista y crea categorías activas, permite desactivarlas sin opción de reactivación y administra productos activos e inactivos por categoría. Los productos se ordenan con controles dentro de cada panel y el orden se guarda por categoría con una petición PATCH.
14. Completado: activar o desactivar el sitio desde el interruptor del panel principal mediante PATCH/DELETE. Teléfono y WhatsApp del contacto quedan limitados a 10 caracteres.
15. El administrador no impone límite de peso a las imágenes; sólo valida que sean PNG, JPG o WEBP.
10. Siguiente: integrar activación, orden y las entidades especializadas de Carousel, Offerings y Contact.

## Fidelidad visual y diferencias relevantes

- Origen: secondary dorado #C9A227; destino: secondary gris #667085. Copiar sólo el template cambia la apariencia. Resolver el tema dentro del gestor, sin cambiar globalmente las otras pantallas.
- El modal de confirmación del origen tiene diseño propio. Conservar ese diseño al conectarlo con lógica asíncrona.
- El origen crea seis secciones ficticias con IDs de tipo texto y published=false. No asumir que la API crea esas secciones ni que devuelve ese estado; respetar su respuesta.
- Los colores iniciales del origen difieren de los defaults expresamente solicitados por el usuario. Usar los defaults solicitados para el contenido del sitio.
- El origen procesa imágenes como Base64 para localStorage. Al integrar imágenes se conservará el archivo para multipart y una vista previa separada.
- El formulario contiene tagline y estructuras de colecciones no especificadas en el POST; se mapearán en su etapa, sin inventar campos de backend.

## Verificación prevista para la implementación

Compilación del destino y comparación visual con el origen en escritorio y móvil. Verificar: acceso con/sin sesión y rol incorrecto; entrada directa y recarga; cancelación sin POST; confirmación y doble clic; errores de validación y red; respuesta perdida después de creación; sitio existente activo e inactivo sin recreación. Validar las solicitudes mediante pruebas controladas antes de crear datos reales.

## Guía breve para capacitación

Por cada integración completar una ficha: evento visual, método/ruta, entradas y origen de cada campo, respuesta y destino visual, errores, prueba mínima. Leer sólo la vista, su servicio y un ejemplo vecino cuando se repita el patrón.

En el primer ejercicio explicar: botón → confirmación → FormData → request con token → getRsp/getErr → estado reactivo → vista. La creación sólo cubre la C de CRUD; las consultas y actualizaciones se enseñarán con sus contratos reales. El usuario puede completar el siguiente recurso reutilizando el ejemplo y recibir una revisión puntual del diff.

## Estado de trabajo

Se modificó únicamente el proyecto funcional: rutas, accesos, gestor compartido, resumen y formulario de identidad. El prefijo de API se resuelve como `client` o `distributor` desde la ruta. No se ejecutaron POST ni PUT contra desarrollo. `npm run build` terminó correctamente después de la integración. Se revisó en navegador la ruta local de Verito y se conservó su estructura visual; el campo Mensaje de marca fue retirado por instrucción del usuario. Los cambios previos de los tres repositorios se conservaron.
