# Guía: primera integración del gestor web

## El recorrido completo

```text
Botón de Mis dominios
  → ruta protegida con domainId
  → GET dominio + GET sitio
  → 404: mostrar Crear página web
  → confirmación del usuario
  → repetir GET para evitar duplicados
  → construir FormData
  → POST con token
  → volver a consultar el registro
  → actualizar la pantalla
```

## Qué debe observar Roberto

La vista no escribe URLs completas. Usa `URL_API`, que ya contiene el host y `/api`. Esto permite cambiar de ambiente sin editar componentes.

El `domain_id` sale de la ruta, pero también se consulta el dominio autenticado. La autorización final pertenece al backend; ocultar o mostrar botones nunca reemplaza esa validación.

`request` detecta automáticamente `FormData`. No se agrega manualmente `Content-Type`, porque el navegador debe generar el boundary de multipart.

Sólo un 404 del GET significa “puedes crear”. Un 401, 403, 422, 500 o error de conexión se muestra como error. Convertir cualquier error en “no existe” podría provocar intentos duplicados.

Antes del POST se repite el GET. Esto cubre una pestaña que quedó abierta mientras otra creó el sitio. La restricción única de backend por `domain_id` sigue siendo la protección definitiva frente a concurrencia.

Durante el POST, `isCreating` bloquea nuevos clics. Después del POST se consulta otra vez el sitio para obtener la representación canónica, incluido `is_active`.

## Plantilla mental para el siguiente CRUD

Para cada operación responder estas preguntas antes de escribir código:

1. ¿Qué acción visible la inicia?
2. ¿Cuál es el método y la ruta?
3. ¿De dónde sale cada campo?
4. ¿Qué contiene `data.item` o `data.items`?
5. ¿Qué estados debe mostrar la vista: cargando, vacío, éxito y error?
6. ¿La operación se puede repetir con seguridad?
7. ¿Qué prueba confirma que no dañamos otro flujo?

Para activar y desactivar el sitio faltará conocer el método, la ruta, el cuerpo y su respuesta. Esa integración sustituirá el switch deshabilitado que ya refleja `website.is_active`.
