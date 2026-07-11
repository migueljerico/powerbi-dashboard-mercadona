# 📊 Power BI Dashboard - Mercadona

![Power BI](https://img.shields.io/badge/Power%20BI-F2C811?style=for-the-badge&logo=powerbi&logoColor=black) ![DAX](https://img.shields.io/badge/DAX-0078D4?style=for-the-badge) ![Power Query](https://img.shields.io/badge/Power%20Query-M_Language-8A2BE2?style=for-the-badge) ![Estado](https://img.shields.io/badge/Estado-Publicado-green?style=for-the-badge) ![Licencia](https://img.shields.io/badge/Licencia-MIT-blue?style=for-the-badge)

*Análisis inteligente del catálogo de productos de Mercadona mediante Business Intelligence para optimizar decisiones comerciales y de pricing.*

## 🔗 Acceso / Demo
El proyecto se distribuye como archivo `.pbix` (Power BI Desktop) y plantilla `.pbit`. Requiere **Power BI Desktop** (versión más reciente) para su visualización y análisis interactivo.

## 📋 Descripción
Este proyecto consiste en el desarrollo de un cuadro de mando (dashboard) profesional utilizando **Microsoft Power BI**, diseñado para analizar de manera visual y estructurada el catálogo de productos de Mercadona. El objetivo principal es transformar datos brutos del archivo `products_macro.csv` en información accionable para la dirección, permitiendo monitorizar la distribución de precios, el peso de las categorías de productos y el impacto de la estrategia de "Productos Destacados" o promociones.

El dashboard resuelve la necesidad de analizar KPIs críticos del catálogo (precio medio, penetración de promociones, concentración por categorías) eliminando la dependencia de hojas de cálculo estáticas y facilitando una visión dinámica, interactiva y accesible del negocio.

## ✨ Funcionalidades

| Funcionalidad | Descripción |
| :--- | :--- |
| **Análisis de Precios** | Visualización del precio medio del catálogo y precio medio con descuento mediante medidas DAX. |
| **Segmentación por Categorías** | Top 10 categorías más caras por precio medio y distribución de volumen (Top 5) en gráfico de anillos. |
| **KPIs de Promociones** | Indicador **% Productos Destacados** calculado vía DAX sobre la columna `En_Promocion`. |
| **Filtros Interactivos (Slicers)** | Panel lateral para filtrar por periodos, categorías y estado promocional. |
| **Cross-filtering Nativo** | Interacción entre gráficos: al seleccionar una categoría, el resto de visuales se filtran automáticamente. |
| **Renderizado de Imágenes** | Tabla de detalle con imágenes dinámicas de productos vía URL (`main_image_url` categorizada como *Image URL*). |
| **Tooltips Enriquecidos** | Información contextual al pasar cursor: precio, descuento, categoría, subtítulo. |
| **Accesibilidad Visual** | Paleta corporativa de alto contraste, etiquetas de datos con categoría + % (no solo color). |

## ⚙️ Instalación

1. **Instalar Power BI Desktop** (versión más reciente):
   ```bash
   # Windows (via winget)
   winget install Microsoft.PowerBIDesktop
   
   # O descargar desde:
   # https://powerbi.microsoft.com/desktop/
   ```

2. **Clonar el repositorio**:
   ```bash
   git clone https://github.com/migueljerico/powerbi-dashboard-mercadona.git
   cd powerbi-dashboard-mercadona
   ```

3. **Abrir el archivo principal**:
   ```bash
   # Opción A: Archivo completo con datos
   start Mercadona_Analysis.pbix
   
   # Opción B: Plantilla sin datos (requiere configurar ruta CSV)
   start Mercadona_Analysis.pbit
   ```

4. **Si usa la plantilla (.pbit)**: Al abrir, Power BI solicitará la ruta del archivo `products_macro.csv`. Indique la ubicación local (ej. `C:\Datos\Mercadona\products_macro.csv`).

## 🚀 Uso

### Navegación del Dashboard
- **Página 1 - Visión General**: KPIs superiores (Total Productos, Precio Medio, Nº Categorías), gráfico barras Top 10 categorías, gráfico anillos distribución volumen.
- **Página 2 - Detalle y Análisis**: Tabla interactiva con imágenes, filtros avanzados, gráficos de tendencias.

### Interacciones Clave
```text
# Filtrar por categoría
Click en barra del gráfico "Top 10 Categorías" → Filtra KPIs y gráfico de anillos

# Ver detalle de producto
Tabla Página 2 → Scroll horizontal → Imagen renderizada + datos completos

# Análisis promocional
Slicer "En_Promocion" = "Sí" → Actualiza % Productos Destacados y Precio Medio
```

## 📁 Estructura del proyecto
```text
.
├── Mercadona_Analysis.pbix          # Reporte completo con datos embebidos
├── Mercadona_Analysis.pbit          # Plantilla sin datos (para nuevas fuentes)
├── docs/
│   └── Analisis_Catalogo_Mercadona_PowerBI_Miguel_Jerico.md  # Documentación técnica extendida
├── MANUAL_TECNICO.md                # Manual técnico de arquitectura y despliegue
└── README.md                        # Este archivo
```

## 🛠️ Tecnologías

| Herramienta | Versión/Detalle | Uso en el proyecto |
| :--- | :--- | :--- |
| **Power BI Desktop** | Latest (2024+) | Motor de visualización, modelado y publicación |
| **Power Query (M)** | M Language | ETL: limpieza, tipado, columna condicional `En_Promocion`, eliminación columnas |
| **DAX** | Advanced | Medidas: `Precio Medio del Catálogo`, `Descuento Medio`, `% Productos Destacados` |
| **Modelado** | Star Schema (implícito) | Tabla única `products` optimizada; dimensión temporal no requerida (catálogo estático) |
| **Fuente de datos** | `products_macro.csv` | Dataset original (formato anglosajón, decimales con punto) |
| **Multimedia** | Image URL Category | Renderizado dinámico de `main_image_url` en tablas y tooltips |
| **Git/GitHub** | Git 2.x | Control de versiones y distribución |

## 📚 Contexto formativo
Desarrollado como proyecto práctico de **Business Intelligence** aplicando:
- **ETL robusto en Power Query**: Manejo de formatos regionales (anglosajón → español), creación de columnas de negocio en lenguaje M.
- **Optimización de modelo semántico**: Reducción de dimensionalidad (eliminación `secondary_image_url`), categorización de recursos multimedia.
- **DAX aplicado a retail**: Métricas de pricing, penetración promocional, concentración de categorías.
- **Diseño UI/UX orientado a datos**: Paleta corporativa accesible, etiquetas de datos redundantes (color + texto), navegación en dos capas (ejecutiva → operativa).

<p align="center">Creado por @migueljerico y documentado por OpenRouter (nvidia/nemotron-3-ultra-550b-a55b:free) · 2026</p>