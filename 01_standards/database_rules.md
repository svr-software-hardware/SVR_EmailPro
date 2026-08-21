# Reglas de Base de Datos

**Versión:** 1.1.0  
**Última actualización:** 2026-08-21

---

# Objetivo

Definir los estándares utilizados para modelar la base de datos de EmailPro.

Estas reglas son obligatorias para todas las entidades del sistema y constituyen la base sobre la cual se diseñan los archivos YAML, las migraciones, los modelos y cualquier implementación relacionada con el almacenamiento de información.

---

# Filosofía

La base de datos representa el negocio de EmailPro.

No representa una implementación específica de Laravel, MySQL, PostgreSQL o cualquier otra tecnología.

Cada tabla, campo y relación debe existir únicamente porque responde a una necesidad real del negocio previamente documentada mediante un proceso.

La simplicidad, la consistencia y la mantenibilidad siempre tienen prioridad sobre la complejidad técnica.

---

# Principios

## Responsabilidad Única

Cada tabla representa una única entidad del negocio.

Una entidad no debe asumir responsabilidades pertenecientes a otro proceso.

---

## Evolución Incremental

El modelo de datos evoluciona conforme evolucionan los procesos del negocio.

Las entidades únicamente podrán modificarse cuando exista una necesidad funcional claramente documentada.

---

## No Anticipación

No se agregan tablas, campos o relaciones pensando en funcionalidades futuras.

Toda estructura debe responder únicamente a una necesidad actual del negocio.

---

## Consistencia

Todas las entidades deben seguir las mismas convenciones de nomenclatura, organización y estructura.

---

## Simplicidad

Siempre se utilizará el modelo más simple capaz de resolver correctamente el proceso del negocio.

---

## Infraestructura Implícita

Toda infraestructura común deberá inferirse automáticamente.

Las entidades únicamente describen la información propia del negocio.

La infraestructura estándar nunca deberá repetirse.

---

# Convenciones Generales

- Toda la estructura se define en inglés.
- Todos los nombres utilizan formato `snake_case`.
- Los nombres deben ser cortos, descriptivos y consistentes.
- Todas las tablas utilizan nombres en plural.
- Todas las llaves primarias utilizan `id`.
- Todas las llaves foráneas terminan con `_id`.
- Los indicadores booleanos comienzan con `is_`.
- Los nombres representan conceptos del negocio y nunca detalles técnicos.

---

# Infraestructura Implícita

Todas las entidades incorporan automáticamente la siguiente infraestructura:

```text
id
is_active
```

Estos campos forman parte del estándar del proyecto y no deben declararse dentro del archivo YAML.

Cuando una entidad define:

```yaml
timestamps: true
```

automáticamente incorpora además:

```text
created_at
updated_at
created_by_id
updated_by_id
```

Estos campos también forman parte del estándar del proyecto y no deben declararse dentro del YAML.

El objetivo es que cada entidad describa únicamente la información propia del negocio.

---

# Tipos de Entidad

Actualmente EmailPro define los siguientes tipos de entidad.

## core_entity

Representa entidades principales del negocio.

Características generales:

- Normalmente utilizan `timestamps`.
- Participan directamente en los procesos del sistema.
- Su información evoluciona constantemente.

Ejemplos:

- users
- partners
- clients
- domains
- mailboxes

---

## general_catalog

Representa catálogos generales utilizados por otras entidades.

Características generales:

- Normalmente no utilizan `timestamps`.
- Se inicializan mediante `seed_data`.
- Contienen información estable.
- Son utilizados como referencia por otras entidades.

Ejemplos:

- roles
- permissions
- countries
- currencies

---

# Restricciones de Unicidad

Las restricciones de unicidad representan reglas reales del negocio que impiden registrar información duplicada.

Cuando un campo debe ser único por sí mismo, la restricción se declara directamente sobre el campo.

Cuando la unicidad depende de la combinación de dos o más campos, deberá declararse una restricción única compuesta.

Una restricción única compuesta no implica que cada campo sea único individualmente. Únicamente impide que se repita la combinación completa de sus valores.

Ejemplos:

- Un correo electrónico de acceso debe ser único por sí mismo.
- El nombre de un dominio puede repetirse con diferentes extensiones, pero la combinación del nombre y la extensión debe ser única.
- Una parte local puede repetirse en diferentes dominios, pero la combinación de la parte local y el dominio debe ser única.

Las restricciones de unicidad deberán:

- Responder a una regla documentada del negocio.
- Incluir únicamente campos pertenecientes a la misma entidad.
- Evitar restricciones anticipadas para procesos no definidos.
- Mantener la integridad de la información sin duplicar datos derivados.

---

# Relaciones

Todas las relaciones deberán cumplir las siguientes reglas:

- Utilizar llaves foráneas.
- Representar únicamente relaciones reales del negocio.
- Mantener la integridad del modelo de datos.
- Evitar relaciones anticipadas para procesos no definidos.
- Utilizar nombres consistentes con la entidad relacionada.

---

# Evolución del Modelo

El modelo de datos evoluciona junto con el negocio.

Ninguna entidad deberá modificarse sin que previamente exista un proceso que justifique el cambio.

No deberán agregarse campos "por si algún día son necesarios".

---

# Flujo de Construcción

Toda entidad deberá seguir el siguiente flujo de definición:

```text
Producto
      ↓
Proceso
      ↓
Entidad
      ↓
Especificación YAML
      ↓
Migración
      ↓
Modelo
      ↓
Backend
      ↓
Frontend
```

Ninguna implementación deberá desarrollarse sin que previamente exista un proceso y una especificación YAML que la definan.

---

# Principio Fundamental

La base de datos constituye una representación del negocio.

Toda tabla, campo o relación deberá responder a una necesidad funcional claramente identificada.

Si una estructura no puede justificarse desde el negocio, no debe formar parte del modelo de datos.
