# 📊 Dashboard de Power BI - Catálogo de Productos Mercadona

[![Power BI](https://img.shields.io/badge/Power_BI-F2C811?style=for-the-badge&logo=powerbi&logoColor=black)](https://powerbi.microsoft.com/)
[![DAX](https://img.shields.io/badge/DAX-0078D4?style=for-the-badge&logo=microsoft&logoColor=white)](https://learn.microsoft.com/dax/)
[![Power Query](https://img.shields.io/badge/Power_Query-M_Language-8A2BE2?style=for-the-badge)](https://learn.microsoft.com/power-query/)
[![Estado](https://img.shields.io/badge/Estado-Publicado-brightgreen?style=for-the-badge)](https://github.com/migueljerico/powerbi-dashboard-mercadona)
[![Licencia](https://img.shields.io/badge/Licencia-GPL--3.0-blue?style=for-the-badge)](LICENSE)

*Cuadro de mando interactivo y solución de Business Intelligence para el análisis comercial, estrategia de precios y penetración promocional en el catálogo de Mercadona.*

---

## 🔗 Acceso / Demo

El cuadro de mando está disponible en este repositorio para ser ejecutado localmente utilizando **Microsoft Power BI Desktop**:

* **Archivo de informe (`.pbix`):** Incluye el modelo de datos precargado en el motor en memoria VertiPaq, las métricas DAX compiladas y las pantallas interactivas de visualización.
* **Plantilla de informe (`.pbit`):** Versión liviana parametrizada que permite reconfigurar la ruta del archivo de datos fuente `products_macro.csv` para refrescar los datos.

Para interactuar con la solución, clona el repositorio o descarga el archivo `.pbix` y ábrelo directamente mediante **Power BI Desktop**.

---

## 📋 Descripción

Este proyecto ofrece una solución integral de **Business Intelligence (BI)** concebida para auditar y analizar el catálogo comercial de Mercadona a partir del dataset procesado `products_macro.csv`. Su propósito es estructurar datos no procesados y convertirlos en indicadores clave e *insights* directamente aplicables al negocio.

El desarrollo resuelve desafíos habituales en la ingesta y preparación de datos dentro del sector *retail*: estandarización de formatos anglosajones (conversión de puntos decimales a comas para entornos de habla hispana), eliminación de columnas redundantes de imagen secundaria para optimizar la memoria columnar del motor VertiPaq, y parametrización de reglas de negocio para distinguir productos promocionados de artículos estándar.

Diseñado para **analistas de datos, responsables de pricing y profesionales de category management**, el cuadro de mando evalúa la distribución de precios por categoría, mide el porcentaje de penetración de "Productos Destacados" en el canal digital y analiza la concentración de la oferta comercial.

---

## ✨ Funcionalidades

| Funcionalidad | Descripción |
| :--- | :--- |
| **Pipeline ETL en Power Query (M)** | Ingesta en UTF-8, reemplazo del separador decimal anglosajón (`.` por `,`) y tipado estricto manual de columnas numéricas. |
| **KPIs de Pricing en DAX** | Cálculos dinámicos de *Precio Medio del Catálogo*, *Descuento Medio* y métricas de recuento de surtido. |
| **Penetración Promocional** | Columna condicional M (`En_Promocion`) y medida DAX (`% Productos Destacados`) para evaluar la presencia promocional. |
| **Ranking Top 10 Categorías** | Gráfico de barras horizontales interactivo con el Top 10 de categorías según su precio medio. |
| **Distribución de Volumen Comercial** | Gráfico de anillos con el Top 5 de categorías por presencia en el catálogo y etiquetas descriptivas accesibles. |
| **Galería Multimedia Renderizada** | Clasificación de URLs (`main_image_url`) como categoría de datos de imagen para renderizar fotos de producto dinámicamente. |
| **Diseño Multipágina Accesible** | Interfaz en 2 pantallas (Visión General y Detalle Granular) con paleta corporativa de alto contraste (`#253494`, `#0072B2`, `#01665E`). |
| **Filtros Cruzados (Slicers)** | Segmentación lateral interactiva por categoría y estado promocional ("Sí" / "No") con comportamiento dinámico. |

---

## ⚙️ Instalación

Sigue estos pasos para clonar el repositorio e interactuar con la solución en tu equipo:

1. **Requisito previo:**
   Asegúrate de contar con **Power BI Desktop** instalado (disponible gratis para Windows en Microsoft Store o en la web oficial de Power BI).

2. **Clonar el repositorio:**
   Abre una terminal y ejecuta:
   ```bash
   git clone https://github.com/migueljerico/powerbi-dashboard-mercadona.git
   ```

3. **Navegar al directorio del proyecto:**
   ```bash
   cd powerbi-dashboard-mercadona
   ```

4. **Ejecutar el proyecto:**
   * Abre el archivo ejecutable `.pbix` con Power BI Desktop para interactuar con los datos embebidos en el modelo en memoria VertiPaq.
   * Opcionalmente, abre el archivo `.pbit` e introduce la ruta absoluta local del archivo `products_macro.csv` en el cuadro de diálogo de parámetros.

---

## 🚀 Uso

### 1. Transformación ETL en Power Query (Lenguaje M)
Durante la fase de preparación de datos, se aplicó una lógica condicional para clasificar la presencia de precios promocionales a partir del atributo `discount_price`:

```powerquery
// Creación de la columna condicional En_Promocion en el editor de Power Query
Table.AddColumn(#"Tipo cambiado", "En_Promocion", each if [discount_price] <> null then "Sí" else "No")
```

### 2. Capa de Métricas DAX (Lógica de Negocio)
El modelo semántico incluye fórmulas DAX agrupadas en la tabla dedicada `_Measures`:

```dax
-- Precio medio general del catálogo de productos
Precio Medio del Catálogo = AVERAGE(products[price])

-- Precio promedio de los artículos en promoción
Descuento Medio = AVERAGE(products_macro[discount_price])

-- Porcentaje dinámico de penetración promocional
% Productos Destacados = 
DIVIDE(
    CALCULATE(COUNTROWS(products), products[En_Promocion] = "Sí"), 
    COUNTROWS(products)
)
```

---

## 📁 Estructura del proyecto

```text
powerbi-dashboard-mercadona/
├── README.md
├── MANUAL_TECNICO.md
├── LICENSE
└── docs/
    └── Analisis_Catalogo_Mercadona_PowerBI_Miguel_Jerico.md
```

---

## 🛠️ Tecnologías

| Herramienta | Versión / Detalle | Uso en el proyecto |
| :--- | :--- | :--- |
| **Microsoft Power BI** | Desktop | Entorno de desarrollo, modelado semántico, capas visuales y UI/UX. |
| **Power Query (M)** | Lenguaje M | Capa ETL: Limpieza, sustitución de decimales (`.` por `,`) y lógica condicional. |
| **DAX** | Data Analysis Expressions | Métricas agregadas, indicadores de pricing y cálculos dinámicos de penetración. |
| **VertiPaq Engine** | In-Memory Columnar DB | Almacenamiento optimizado de datos en memoria para alto rendimiento analítico. |
| **CSV / UTF-8** | `products_macro.csv` | Fuente primario de datos con el catálogo completo de Mercadona. |
| **Markdown** | Documentación | Documentación del proyecto, especificaciones de arquitectura y manual técnico. |

---

## 📚 Contexto formativo o motivación del proyecto

Este proyecto fue desarrollado por **Miguel Jericó** como una solución práctica de Business Intelligence orientada al análisis de datos comerciales en el sector del comercio minorista (*retail*).

La iniciativa abarca un ciclo de vida completo de análisis de datos dentro del ecosistema Microsoft Power BI: desde la ingestión y transformación de un origen de datos en texto plano (`products_macro.csv`), la estructuración de una arquitectura por capas (Origen → ETL → Modelo Semántico VertiPaq → Lógica DAX → Capa de Presentación UI/UX), hasta la elaboración de una documentación técnica orientada a estándares profesionales.

---

<p align="center">Creado por @migueljerico y documentado por Google Gemini (gemini-3.6-flash) · 2026</p>