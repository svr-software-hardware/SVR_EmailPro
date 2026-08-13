# Reglas de YAML

**Versión:** 1.0.0  
**Última actualización:** 2026-08-10

---

# Objetivo

Definir la estructura y las convenciones utilizadas para escribir los archivos YAML que representan oficialmente el modelo de datos de EmailPro.

Todos los archivos YAML deberán seguir estas reglas para garantizar consistencia, legibilidad y facilitar su interpretación tanto por desarrolladores como por herramientas de Inteligencia Artificial.

---

# Filosofía

Los archivos YAML constituyen la especificación oficial de cada entidad del sistema.

Su responsabilidad es describir el modelo de datos de forma clara, consistente e independiente de cualquier lenguaje de programación, framework o motor de base de datos.

Los archivos YAML representan la única fuente de verdad (Single Source of Truth) para la definición estructural de cada entidad.

---

# Principios

Todo archivo YAML deberá cumplir los siguientes principios.

## Responsabilidad Única

Un archivo YAML representa una única entidad del negocio.

---

## Consistencia

Todas las entidades deberán mantener la misma estructura y organización.

---

## Simplicidad

El archivo debe describir únicamente aquello que hace diferente a la entidad.

---

## No Anticipación

No deberán agregarse campos, relaciones o estructuras para funcionalidades futuras.

---

## Derivado del Negocio

Toda entidad debe existir porque previamente existe un proceso que la justifica.

---

## Infraestructura Implícita

La infraestructura común nunca deberá declararse explícitamente dentro del archivo YAML.

---

# Alcance

Los archivos YAML describen exclusivamente el modelo de datos.

No deberán contener:

- Procesos del negocio.
- Casos de uso.
- Interfaces de usuario.
- Código SQL.
- Código Laravel.
- Código Vue.
- Endpoints.
- Reglas de API.
- Lógica de programación.

---

# Estructura General

Todo archivo YAML deberá mantener la siguiente estructura.

```yaml
metadata:

database_schema:
```

Cada bloque posee una responsabilidad específica y no debe mezclar información perteneciente a otro.

---

# metadata

Contiene la información general de la entidad.

Ejemplo:

```yaml
metadata:
  name: users
  type: core_entity
  description: "Entidad principal para la administración de usuarios."
```

## Propiedades

| Propiedad | Descripción |
|------------|-------------|
| `name` | Nombre de la entidad. |
| `type` | Tipo de entidad definido por el proyecto. |
| `description` | Descripción breve orientada al negocio. |

---

# Tipos de Entidad

Actualmente EmailPro utiliza los siguientes tipos.

- `core_entity`
- `general_catalog`

Podrán incorporarse nuevos tipos conforme evolucione el proyecto.

---

# database_schema

Describe la estructura de la entidad.

Ejemplo:

```yaml
database_schema:
  table: users
  table_es: "usuarios"
  timestamps: true

  fields:
```

---

# timestamps

Cuando una entidad define:

```yaml
timestamps: true
```

automáticamente incorpora la infraestructura definida en `database_rules.md`.

Por lo tanto, nunca deberán declararse manualmente los siguientes campos:

```text
created_at
updated_at
created_by_id
updated_by_id
```

---

# Infraestructura Implícita

Toda entidad incorpora automáticamente:

```text
id
is_active
```

Estos campos forman parte del estándar del proyecto.

No deberán declararse dentro del bloque `fields`.

---

# fields

El bloque `fields` contiene únicamente los campos propios de la entidad.

No deberán declararse campos pertenecientes a la infraestructura implícita.

Cada campo podrá utilizar únicamente las propiedades necesarias.

## Propiedades soportadas

| Propiedad | Descripción |
|------------|-------------|
| `type` | Tipo de dato. |
| `default` | Valor por defecto. |
| `not_null` | Permite valores nulos cuando es `false`. |
| `unique` | Restringe valores duplicados. |
| `references` | Llave foránea. |
| `description` | Descripción funcional del campo. |

No deberán declararse propiedades innecesarias.

---

# seed_data

Se utiliza únicamente para entidades de tipo `general_catalog`.

Ejemplo:

```yaml
seed_data:
  - id: 1
    name: "Super Admin"

  - id: 2
    name: "Operador"
```

Las entidades del negocio normalmente no utilizan esta sección.

---

# Convenciones

Todos los archivos YAML deberán seguir las siguientes reglas.

- Utilizar únicamente inglés para nombres de entidades, tablas y campos.
- Utilizar `snake_case`.
- Mantener una indentación consistente.
- Escribir descripciones orientadas al negocio.
- Evitar información duplicada.
- Mantener el mismo orden de bloques.
- Declarar únicamente la información propia de la entidad.
- La infraestructura estándar nunca deberá repetirse.

---

# Evolución

Los archivos YAML evolucionan conforme evoluciona el negocio.

Toda modificación deberá originarse en un proceso previamente documentado.

No deberán agregarse estructuras anticipando funcionalidades futuras.

---

# Flujo de Construcción

Toda entidad deberá seguir el siguiente flujo.

```text
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

---

# Principio Fundamental

Los archivos YAML representan la especificación oficial del modelo de datos de EmailPro.

Toda implementación de base de datos, backend o frontend deberá derivarse de esta especificación y nunca modificarla directamente.