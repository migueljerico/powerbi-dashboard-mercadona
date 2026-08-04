# 📊 Dashboard de Power BI - Catálogo de Productos Mercadona

[![Power BI](https://img.shields.io/badge/Power_BI-F2C811?style=for-the-badge&logo=powerbi&logoColor=black)](https://powerbi.microsoft.com/)
[![DAX](https://img.shields.io/badge/DAX-0078D4?style=for-the-badge&logo=microsoft&logoColor=white)](https://learn.microsoft.com/dax/)
[![Power Query](https://img.shields.io/badge/Power_Query-M_Language-8A2BE2?style=for-the-badge)](https://learn.microsoft.com/power-query/)
[![Estado](https://img.shields.io/badge/Estado-Publicado-brightgreen?style=for-the-badge)](https://github.com/migueljerico/powerbi-dashboard-mercadona)
[![Licencia](https://img.shields.io/badge/Licencia-GPL--3.0-blue?style=for-the-badge)](LICENSE)

*Cuadro de mando interactivo y solución de Business Intelligence para el análisis comercial, estrategia de precios y penetración promocional en el catálogo de Mercadona.*

---

## 🔗 Acceso / Demo

El cuadro de mando se encuentra disponible en este repositorio en dos formatos de distribución estándar de Microsoft Power BI:

* **Archivo de informe (`.pbix`):** Incluye la estructura, el modelo de datos VertiPaq optimizado, la capa analítica de medidas DAX y la interfaz visual completa interactiva.
* **Plantilla de informe (`.pbit`):** Versión liviana reutilizable que permite conectarse a una fuente local limpia actualizando la ruta del archivo fuente `products_macro.csv`.

Para interactuar con el cuadro de mando en tu entorno local, descarga el archivo ejecutable desde el repositorio y ábrelo directamente utilizando **Power BI Desktop**.

---

## 📋 Descripción

Este proyecto ofrece una solución integral de **Business Intelligence (BI)** desarrollada para analizar y auditar el catálogo completo de productos de Mercadona a partir del dataset procesado `products_macro.csv`. Su objetivo principal es transformar datos brutos en insights de negocio directamente accionables para la toma de decisiones comerciales.

El sistema resuelve problemas críticos de ingesta y estandarización de datos, tales como el tratamiento de datos de origen anglosajón (conversión de puntos decimales a comas para entornos regionales de habla hispana), la eliminación de atributos multimedia redundantes para optimizar la memoria del motor VertiPaq, y la creación de reglas de negocio para distinguir productos promocionados de artículos estándar.

Diseñado específicamente para **analistas de datos, responsables de pricing, equipos de marketing y profesionales del sector retail**, este cuadro de mando permite evaluar la distribución de precios por categoría, medir la penetración de "Productos Destacados" en la plataforma web y analizar la concentración de la oferta comercial.

---

## ✨ Funcionalidades

| Funcionalidad | Descripción |
| :--- | :--- |
| **Pipeline ETL en Power Query (M)** | Limpieza automatizada de tipos de datos, reemplazo de separadores decimales (`.` por `,`) e ingesta limpia de codificación UTF-8. |
| **KPIs de Pricing en DAX** | Cálculo dinámico de indicadores clave como *Precio Medio del Catálogo*, *Descuento Medio* y el recuento total de surtido. |
| **Métrica de Penetración Promocional** | Columna condicional M `En_Promocion` y medida DAX `% Productos Destacados` para medir el volumen de oferta en promoción. |
| **Ranking Top 10 Categorías** | Gráfico de barras horizontales interactivo que identifica las 10 categorías con el precio medio más elevado. |
| **Distribución de Volumen Comercial** | Gráfico de anillos con el Top 5 de categorías por presencia en el catálogo y etiquetas detalladas adaptadas para accesibilidad visual. |
| **Galería Multimedia Renderizada** | Clasificación de URLs (`main_image_url`) como recurso de imagen dinámico dentro de las tablas de detalle de producto. |
| **Diseño Multipágina Accesible** | Estructura en 2 pantallas (Visión General y Detalle Granular) con una paleta corporativa de alto contraste (`#253494`, `#0072B2`, `#01665E`). |
| **Segmentación Dinámica (Slicers)** | Filtros laterales para exploración cruzada por categorías de producto y estado de promoción ("Sí" / "No"). |

---

## ⚙️ Instalación

Sigue estos pasos para clonar el repositorio e interactuar con el proyecto en tu máquina local:

1. **Requisito previo:**
   Asegúrate de tener instalado **Power BI Desktop** (disponible de forma gratuita para Windows en la Microsoft Store o la web oficial de Power BI).

2. **Clonar el repositorio:**
   Abre tu terminal o consola Git y ejecuta:
   ```bash
   git clone https://github.com/migueljerico/powerbi-dashboard-mercadona.git
   ```

3. **Navegar al directorio del proyecto:**
   ```bash
   cd powerbi-dashboard-mercadona
   ```

4. **Abrir el cuadro de mando:**
   * Haz doble clic sobre el archivo de informe `.pbix` para cargarlo con los datos ya embebidos en el modelo en memoria VertiPaq.
   * Opcionalmente, abre la plantilla `.pbit` e introduce la ruta local de tu archivo de origen `products_macro.csv` en el prompt de Power Query.

---

## 🚀 Uso

### 1. Transformación M en Power Query (Capa ETL)
Durante la fase de ingesta, se implementó una columna lógica de negocio en lenguaje M dentro de Power Query para segmentar la presencia de productos destacados:

```powerquery
// Creación de la columna condicional En_Promocion en el editor de Power Query
Table.AddColumn(#"Tipo cambiado", "En_Promocion", each if [discount_price] <> null then "Sí" else "No")
```

### 2. Medidas Analíticas en DAX (Capa de Negocio)
El modelo semántico incluye fórmulas DAX optimizadas para alimentar los visuales y las tarjetas de KPI:

```dax
-- Cálculo del precio medio global del catálogo
Precio Medio del Catálogo = AVERAGE(products[price])

-- Precio medio de productos con precio promocional
Descuento Medio = AVERAGE(products_macro[discount_price])

-- Porcentaje dinámico de penetración de promociones
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
| **Microsoft Power BI** | Desktop | Entorno principal de desarrollo, modelado semántico y diseño UI/UX. |
| **Power Query (M)** | Engine M | Capa ETL: Limpieza, tipado manual, estandarización regional y columnas condicionales. |
| **DAX** | Data Analysis Expressions | Creación de medidas agregadas, cálculos dinámicos de KPIs y porcentajes de penetración. |
| **VertiPaq Engine** | In-Memory Columnar DB | Motor de datos columnar para el almacenamiento optimizado y rápido rendimiento de consultas. |
| **CSV / UTF-8** | `products_macro.csv` | Origen de datos primario del catálogo de productos de Mercadona. |
| **Markdown** | Documentación técnica | Documentación del proyecto, guía técnica y especificaciones de arquitectura. |

---

## 📚 Contexto formativo o motivación del proyecto

Este proyecto fue desarrollado por **Miguel Jericó** como una solución práctica orientada al análisis de Business Intelligence aplicado al sector del comercio minorista (*retail*). 

La iniciativa demuestra la implementación de un ciclo de vida de datos completo dentro del ecosistema Microsoft Power BI: desde la ingestión y tratamiento de un conjunto de datos sin estructurar (`products_macro.csv`), pasando por la construcción de una arquitectura sólida de 4 capas (Origen → ETL → Modelo Semántico VertiPaq → Capa de Presentación DAX/Visuales), hasta la creación de un cuadro de mando con altos estándares de UI/UX y accesibilidad visual.

---

<p align="center">Creado por @migueljerico y documentado por Google Gemini (gemini-3.6-flash) · 2026</p>