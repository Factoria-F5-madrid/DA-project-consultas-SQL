# Flujo de datos de SQL a Python

## 📝 Descripción del proyecto

Durante este proyecto, se trabajará con la base de datos Sakila, generando tres dataframes exportados desde SQL, para posteriormente seleccionar uno de ellos y aplicar una limpieza preliminar en SQL.

Luego, el dataframe limpio será procesado y documentado en un notebook de Google Colab, donde se realizará un procesamiento y limpieza final de los datos.

El proyecto está dividido en dos partes:

1. **Extracción desde SQL** — Generación de tres dataframes mediante joins entre tablas específicas.
**Limpieza preliminar en SQL** — Normalización, filtrado, estandarización de columnas clave de **UNO** de los dataframes.
2. **Procesamiento en Python** — Procesamiento y limpieza en python de **UNO** de los dataframes.

Se evaluará la claridad del proceso, la justificación de las decisiones de limpieza y la organización del notebook.

## 🎯 Objetivos concretos

- Comprender la estructura de la base de datos Sakila y cómo conectar múltiples tablas.
- Exportar tres dataframes construidos mediante joins SQL.
- Aplicar reglas de limpieza y estandarización directamente en SQL.
- Crear un notebook ordenado y reproducible que documente los pasos ejecutados.
- Exportar un dataset limpio (CSV o Parquet) para futuras etapas del priyecto.

## 📊 Dataframes a generar desde SQL

Cada estudiante debe generar **tres** dataframes obligatorios indicados a continuación con las tablas a las que tendrá que hacer JOIN, exportados desde SQL con las tablas indicadas. Posteriormente elegirá uno de ellos para limpiar en SQL y luego en Python.

### 📁 Dataframe 1: Actividad de clientes

**Tablas involucradas:**
- customer
- address
- city
- country
- rental
- payment

**Objetivo:** obtener un dataset que describa la actividad y comportamiento de alquileres y pagos de clientes.

### 📁 Dataframe 2: Catálogo de películas

**Tablas:**
- film
- film_category
- category
- language
- inventory

**Objetivo:** generar una vista completa del catálogo, asignando películas a sus categorías, idiomas y disponibilidad física.

### 📁 Dataframe 3: Elenco y popularidad

**Tablas:**
- film
- actor
- film_actor

**Objetivo:** analizar el elenco por película y frecuencia de aparición de actores.

## 📌 Elección del dataframe para limpieza completa

Cada estudiante debe seleccionar uno de los tres dataframes y:

- Aplicar limpieza SQL obligatoria.
- Exportar el dataframe limpio.
- Importarlo en Google Colab para un procesamiento adicional.
- Hacer uso de las recomendaciones siguientes para la limpieza en SQL del dataframe

### 👓 Dataframe 1: Actividad de clientes
**Limpieza preliminar SQL:**
- Eliminar registros con `rental_id` o `payment_id` nulos.
- Asegurar que `amount > 0` en payment.
- Filtrar registros donde `rental.return_date` no sea nula (alquiler completado).
- Estandarizar textos: nombres, apellidos, emails, ciudades → `LOWER()`
- Asegurar consistencia de fechas (`rental_date < return_date`).
- Joins bien definidos con llaves primarias y foráneas, evitando duplicación.

**Sugerencias adicionales:**
- Crear columna derivada: `rental_duration` calculada en días usando `DATEDIFF`.

### 👓 Dataframe 2: Catálogo de películas
**Limpieza preliminar SQL:**
- Normalizar cadenas (títulos, descripciones): `LOWER(TRIM()`).
- Eliminar registros donde `film.length <= 0` o `film.rating` sea nulo.
- Unificar nombres de categoría y lenguaje en minúsculas.
- Verificar integridad del catálogo:
  - `inventory.inventory_id` no nulo
  - `film_category.category_id` asignado correctamente

**Sugerencias adicionales:**
- Agrupaciones útiles:
  - Conteo de películas por categoría
  - Conteo de copias disponibles por film
- Crear columna derivada: `is_long_film` (≥ 120 min).

### 👓 Dataframe 3: Elenco y popularidad
**Limpieza preliminar SQL:**
- Estandarizar nombres de actores (`LOWER(first_name)` y `LOWER(last_name)`).
- Eliminar actores duplicados (verificando combinaciones de nombre + apellido).
- Asegurar que `film.film_id` y `actor.actor_id` existan (joins consistentes).
- Filtrar películas sin actores asociados.

**Sugerencias adicionales:**
- Generar tabla agregada con:
  - número de actores por película
  - número de películas por actor
- Crear columna derivada: `actor_full_name`.


## 🧰 Tecnologías y librerías

- **SQL:** MySQL / Workbench / DBeaver
- **Python:** notebook en Google Colab
- **GitHub:** repositorio del proyecto

## 📦 Condiciones de entrega

- Entrega individual.
- Query SQL de extracción y limpieza del dataframe seleccionado, esto lo harán exportando el script de la consulta SQL final completa.
- Notebook (.ipynb).
- README documentando:
  - pasos SQL
  - criterios de limpieza
  - instrucciones para ejecutar notebook
  - descripción de decisiones tomadas en el proyecto
- Subir todo a un repositorio de GitHub.

## ⏳ Plazo de entrega

**1 semana**

## ✅ Checklist de limpieza (Python)

- [ ] Convertir fechas a datetime.
- [ ] Verificar duplicados y definir criterios para eliminarlos.
- [ ] Analizar valores faltantes y documentar decisiones.
- [ ] Normalizar cadenas (lower, trim).
- [ ] Corregir tipos numéricos.
- [ ] Detectar outliers.
- [ ] Crear columnas derivadas útiles.
- [ ] Generar visualizaciones para validar la distribución de datos.
- [ ] Exportar dataset final (No se sube al repo).

## 🧪 Criterios de evaluación

- Configurar y automatizar su entorno de trabajo.
- Gestionar equipos técnicos
- Evaluar Conjuntos de datos
