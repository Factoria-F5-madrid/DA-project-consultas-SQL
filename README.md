<img width="11520" height="3456" alt="Banner_notebooks" src="https://github.com/user-attachments/assets/66a3d9a6-798f-4397-9e60-b8c529ad2b0a" />

# Flujo de datos de SQL a Python

## 📝 Descripción del proyecto

Durante este proyecto se trabajará con la base de datos **Olist**, un marketplace brasileño real, generando tres dataframes exportados desde SQL, para posteriormente seleccionar uno de ellos y aplicar una limpieza preliminar en SQL.

Luego, el dataframe limpio será procesado y documentado en un notebook de Google Colab, donde se realizará un procesamiento y limpieza final de los datos.

El proyecto está dividido en dos partes:

1. **Extracción desde SQL** — Generación de tres dataframes mediante joins entre tablas específicas.
**Limpieza preliminar en SQL** — Normalización, filtrado, estandarización de columnas clave de **UNO** de los dataframes.
2. **Procesamiento en Python** — Procesamiento y limpieza en python de **UNO** de los dataframes.

Se evaluará la claridad del proceso, la justificación de las decisiones de limpieza y la organización del notebook.

---

## 💾 Instalación de la base de datos

Descarga el volcado y cárgalo en tu MySQL local:

📥 **[olist.sql.gz](https://drive.google.com/drive/folders/1apSXjn6eQ5o9RdutbD4skSjvH6ytvR06?usp=sharing)** · 46 MB comprimido, 145 MB al descomprimir

```bash
# Linux / macOS / Git Bash
gunzip -c olist.sql.gz | mysql -u root -p

# Windows (PowerShell), tras descomprimir con 7-Zip:
mysql -u root -p < olist.sql
```

Tarda unos minutos: son **1.550.922 filas**. Al terminar tendrás la base `olist` con 9 tablas.

```sql
USE olist;
SHOW TABLES;
SELECT COUNT(*) FROM orders;   -- 99441
```

> [!NOTE]
> Son datos **reales** de 99.441 pedidos realizados en Brasil entre 2016 y 2018.
> Fuente: [Brazilian E-Commerce Public Dataset by Olist](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce) · licencia CC BY-NC-SA 4.0.

### Modelo de datos

```
customers ──< orders ──< order_items >── products >── categoria_traduccion
                 │            │
                 │            └────────>── sellers
                 ├──< order_payments
                 └──< order_reviews

geolocation   se une a customers y sellers por zip_code_prefix
```

---

## 🎯 Objetivos concretos

- Comprender la estructura de la base de datos Olist y cómo conectar múltiples tablas.
- Exportar tres dataframes construidos mediante joins SQL.
- Aplicar reglas de limpieza y estandarización directamente en SQL.
- Crear un notebook ordenado y reproducible que documente los pasos ejecutados.
- Exportar un dataset limpio (CSV o Parquet) para futuras etapas del proyecto.

---

## 📊 Dataframes a generar desde SQL

Cada estudiante debe generar **tres** dataframes obligatorios, exportados desde SQL con las tablas indicadas. Posteriormente elegirá uno de ellos para limpiar en SQL y luego en Python.

### 📁 Dataframe 1: Actividad de clientes

**Tablas involucradas:**
- customers
- orders
- order_payments
- order_reviews
- geolocation

**Objetivo:** obtener un dataset que describa la actividad de compra, pago y satisfacción de los clientes, junto con su ubicación geográfica.

### 📁 Dataframe 2: Catálogo de productos

**Tablas:**
- products
- categoria_traduccion
- order_items
- sellers

**Objetivo:** generar una vista completa del catálogo, asignando productos a sus categorías traducidas, precios, costes de envío y vendedor.

### 📁 Dataframe 3: Vendedores y popularidad

**Tablas:**
- sellers
- order_items
- products

**Objetivo:** analizar el catálogo por vendedor y la frecuencia con que se vende cada producto, identificando qué vendedores concentran la actividad.

---

## 📌 Elección del dataframe para limpieza completa

Cada estudiante debe seleccionar uno de los tres dataframes y:

- Aplicar limpieza SQL obligatoria.
- Exportar el dataframe limpio.
- Importarlo en Google Colab para un procesamiento adicional.
- Hacer uso de las recomendaciones siguientes para la limpieza en SQL del dataframe.

### 👓 Dataframe 1: Actividad de clientes

**Limpieza preliminar SQL:**
- Filtrar por `order_status = 'delivered'` si el análisis es de pedidos completados. Hay **8 estados** distintos: `delivered`, `shipped`, `canceled`, `unavailable`, `invoiced`, `processing`, `created` y `approved`.
- Decidir qué hacer con los pedidos sin `order_delivered_customer_date`: son **2.965**, el 3,0%. No son errores, son pedidos que nunca se entregaron. Documenta la decisión.
- Asegurar `payment_value > 0`. Hay **9** registros a cero, y **3** con `payment_type = 'not_defined'`.
- Estandarizar textos: ciudades y estados con `LOWER(TRIM())`.
- Verificar consistencia temporal: `order_purchase_timestamp < order_delivered_customer_date`.
- Joins bien definidos con claves primarias y foráneas, evitando duplicación.

> [!WARNING]
> **La trampa de `geolocation`.** Tiene **1.000.163 filas para solo 19.015 prefijos postales**: una media de 52,6 filas por prefijo, y el peor caso (`24220`) llega a 1.146. Si haces `JOIN` directo, tu dataset se multiplica y todos los agregados salen mal. Agrega primero —media de latitud y longitud por prefijo, o un `DISTINCT`— y une después.

> [!WARNING]
> **La trampa de `customer_id`.** Hay **99.441 `customer_id` pero solo 96.096 `customer_unique_id`**: Olist genera un `customer_id` nuevo en cada pedido. Si cuentas clientes con `customer_id`, te salen de más. Para clientes reales, usa `customer_unique_id`.

**Sugerencias adicionales:**
- Columna derivada `delivery_days`: `DATEDIFF(order_delivered_customer_date, order_purchase_timestamp)`.
- Columna derivada `is_late`: entregado después de `order_estimated_delivery_date`.

### 👓 Dataframe 2: Catálogo de productos

**Limpieza preliminar SQL:**
- Normalizar cadenas de categoría con `LOWER(TRIM())`.
- Decidir qué hacer con los **610 productos sin `product_category_name`**.
- Eliminar registros con `product_weight_g <= 0` o con dimensiones nulas.
- Traducir las categorías uniendo con `categoria_traduccion`.
- Verificar integridad: que todo `order_items.product_id` tenga producto existente.

> [!WARNING]
> **La trampa de la traducción.** Hay **73 categorías distintas y solo 71 tienen traducción**. Faltan `pc_gamer` y `portateis_cozinha_e_preparadores_de_alimentos`. Con `INNER JOIN` pierdes esos productos sin enterarte; con `LEFT JOIN` los conservas con la traducción a `NULL`. Elige y justifica.

**Sugerencias adicionales:**
- Agrupaciones útiles: productos por categoría, precio medio por categoría, vendedores distintos por producto.
- Columna derivada `is_heavy` (≥ 5 kg) o `freight_ratio` (`freight_value / price`).

### 👓 Dataframe 3: Vendedores y popularidad

**Limpieza preliminar SQL:**
- Estandarizar `seller_city` y `seller_state` con `LOWER(TRIM())`. Cuidado: la misma ciudad aparece escrita de varias formas.
- Asegurar que `seller_id` y `product_id` existan en sus tablas.
- Filtrar los productos nunca vendidos, o conservarlos y justificarlo.
- Evitar duplicación: `order_items` tiene una fila **por unidad**, no por pedido. Un pedido de 3 unidades genera 3 filas.

**Sugerencias adicionales:**
- Tabla agregada con: productos distintos por vendedor, ventas por producto, ingreso total por vendedor.
- Columna derivada `ticket_medio` por vendedor.

---

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
