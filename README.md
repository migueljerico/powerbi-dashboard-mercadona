# 📊 Power BI Dashboard - Mercadona

![Power BI](https://img.shields.io/badge/Power%20BI-F2C811?style=for-the-badge&logo=powerbi&logoColor=black) ![DAX](https://img.shields.io/badge/DAX-0078D4?style=for-the-badge) ![Power Query](https://img.shields.io/badge/Power%20Query-M_Language-8A2BE2?style=for-the-badge) ![CSV](https://img.shields.io/badge/CSV-UTF--8-4CAF50?style=for-the-badge&logo=files&logoColor=white) ![Estado](https://img.shields.io/badge/Estado-Publicado-green?style=for-the-badge) ![Licencia](https://img.shields.io/badge/Licencia-GPL--3.0-blue?style=for-the-badge)

*Análisis inteligente del catálogo de productos de Mercadona mediante Business Intelligence para optimizar decisiones comerciales y de pricing.*

---

## 🔗 Acceso / Demo
El proyecto se distribuye como archivo `.pbix` (Power BI Desktop) y plantilla `.pbit`. Requiere **Power BI Desktop** (versión más reciente) para su visualización y análisis interactivo. Para acceder al código fuente y a los artefactos del proyecto, clone el repositorio desde GitHub.

## 📋 Descripción
Este proyecto consiste en el desarrollo de un cuadro de mando (dashboard) profesional utilizando **Microsoft Power BI**, diseñado para analizar de manera visual y estructurada el catálogo de productos de Mercadona. El objetivo principal es transformar datos brutos del archivo `products_macro.csv` en información accionable para la dirección, permitiendo monitorizar la distribución de precios, el peso de las categorías de productos y el impacto de la estrategia de "Productos Destacados" o promociones.

El dashboard resuelve la necesidad de analizar KPIs críticos del catálogo (precio medio, penetración de promociones, concentración por categorías) eliminando la dependencia de hojas de cálculo estáticas y facilitando una visión dinámica, interactiva y accesible del negocio. La herramienta está orientada a analistas de datos, equipos de marketing, responsables de pricing y, en general, a cualquier perfil interesado en retail intelligence aplicado a una de las cadenas de distribución más relevantes del mercado español.

El flujo de trabajo abarca desde una fase ETL robusta en Power Query (con limpieza regional de decimales punto→coma), pasando por un modelo semántico VertiPaq optimizado, hasta una capa de medidas DAX personalizadas que alimentan visualizaciones interactivas con enfoque en accesibilidad y UX.

## ✨ Funcionalidades

| Funcionalidad | Descripción |
| :--- | :--- |
| **Análisis de Precios** | Visualización del precio medio del catálogo y precio medio con descuento mediante medidas DAX (`AVERAGE(products[price])`). |
| **Segmentación por Categorías** | Top 10 categorías más caras por precio medio en barras horizontales y distribución de volumen (Top 5) en gráfico de anillos. |
| **KPIs de Promociones** | Indicador **% Productos Destacados** calculado vía DAX sobre la columna `En_Promocion` (DIVIDE + CALCULATE + COUNTROWS). |
| **Filtros Interactivos (Slicers)** | Panel lateral para filtrar por periodos, categorías y estado promocional (`En_Promocion` = "Sí"/"No"). |
| **Cross-filtering Nativo** | Interacción entre gráficos: al seleccionar una categoría, el resto de visuales se filtran automáticamente. |
| **Renderizado de Imágenes** | Tabla de detalle con imágenes dinámicas de productos vía URL (`main_image_url` categorizada como *Image URL*). |
| **Tooltips Enriquecidos** | Información contextual al pasar cursor: precio, descuento, categoría, subtítulo. |
| **Accesibilidad Visual** | Paleta corporativa de alto contraste, etiquetas de datos con categoría + % (no solo color). |
| **Modelo Optimizado** | Reducción de dimensionalidad eliminando `secondary_image_url` para mejorar el peso del archivo `.pbix`. |
| **Doble Página Analítica** | Página 1 (Visión General con KPIs) y Página 2 (Detalle y exploración granular). |

## ⚙️ Instalación

1. **Instalar Power BI Desktop** (versión más reciente):
   ```bash
   # Windows (via winget)
   winget install Microsoft.PowerBIDesktop
   
   # macOS: No soportado oficialmente, requiere máquina virtual Windows o Power BI Service web
   
   # Descarga directa desde:
   # https://powerbi.microsoft.com/desktop/
   ```

2. **Clonar el repositorio**:
   ```bash
   git clone https://github.com/migueljerico/powerbi-dashboard-mercadona.git
   cd powerbi-dashboard-mercadona
   ```

3. **Verificar estructura de archivos**:
   ```bash
   ls -la
   # Debe incluir: README.md, MANUAL_TECNICO.md, LICENSE, docs/
   # Y los artefactos Power BI (.pbix / .pbit) junto al CSV
   ```

4. **Abrir el archivo principal**:
   ```bash
   # Opción A: Archivo completo con datos embebidos
   start Ejercicio_3.8_Proyecto_Ciencia_Datos_Mercadona_Miguel_Jeric_.pbix
   
   # Opción B: Plantilla sin datos (requiere configurar ruta CSV)
   start Ejercicio_3.8_Proyecto_Ciencia_Datos_Mercadona_Miguel_Jeric_.pbit
   ```

5. **Si usa la plantilla (.pbit)**: Al abrir, Power BI solicitará la ruta del archivo `products_macro.csv`. Indique la ubicación local del dataset (ej. `C:\Datos\Mercadona\products_macro.csv` o `~/Data/products_macro.csv`).

6. **Actualizar fuente de datos** (si edita la ETL en Power Query):
   ```powerquery
   // Power Query → Editor avanzado → Modificar parámetro:
   Source = Csv.Document(File.Contents("C:/Datos/Mercadona/products_macro.csv"),[Delimiter=",", Encoding=65001])
   ```

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

# Tooltip enriquecido
Hover sobre producto en tabla → Muestra subtitle, categoría, descuento
```

### Ejemplo de medida DAX incluida
```dax
// Medida: % Productos Destacados
% Productos Destacados = 
DIVIDE(
    CALCULATE(COUNTROWS(products), products[En_Promocion] = "Sí"), 
    COUNTROWS(products)
)
```

### Ejemplo de transformación Power Query (M)
```powerquery
// Paso: Creación de columna condicional En_Promocion
= Table.AddColumn(#"Tipo cambiado", "En_Promocion", each if [discount_price] <> null then "Sí" else "No")
```

## 📁 Estructura del proyecto
```text
.
├── README.md                                                                    # Documentación principal
├── MANUAL_TECNICO.md                                                            # Manual técnico detallado
├── LICENSE                                                                       # Licencia GPL-3.0
├── Ejercicio_3.8_Proyecto_Ciencia_Datos_Mercadona_Miguel_Jeric_.pbix            # Reporte completo con datos embebidos
├── Ejercicio_3.8_Proyecto_Ciencia_Datos_Mercadona_Miguel_Jeric_.pbit            # Plantilla sin datos (para nuevas fuentes)
├── products_macro.csv                                                            # Dataset origen (CSV UTF-8)
└── docs/
    └── Analisis_Catalogo_Mercadona_PowerBI_Miguel_Jerico.md                      # Análisis técnico y conclusiones
```

## 🛠️ Tecnologías

| Herramienta | Versión/Detalle | Uso en el proyecto |
| :--- | :--- | :--- |
| **Microsoft Power BI Desktop** | Última versión estable | Plataforma principal de BI para modelado, DAX y visualización. |
| **Power Query (Lenguaje M)** | Integrado en Power BI | Capa ETL: lectura CSV, limpieza regional, tipado de columnas, columna condicional `En_Promocion`. |
| **DAX (Data Analysis Expressions)** | Integrado en Power BI | Medidas calculadas: `Precio Medio del Catálogo`, `Descuento Medio`, `% Productos Destacados`, `Total Productos`. |
| **VertiPaq Engine** | Motor interno de Power BI | Compresión y almacenamiento en columna del modelo semántico (tabla única `products`). |
| **CSV (UTF-8)** | Encoding 65001 | Formato del dataset origen `products_macro.csv`. |
| **Git** | 2.x+ | Control de versiones del proyecto. |
| **winget** | Windows Package Manager | Instalación de Power BI Desktop en Windows. |
| **Markdown** | Estándar | Documentación técnica complementaria (`docs/`). |

## 📚 Contexto formativo o motivación del proyecto
Este proyecto se enmarca en el **Ejercicio 3.8** de un programa formativo de Ciencia de Datos, donde el reto propuesto consistía en construir un cuadro de mando profesional partiendo exclusivamente de un dataset crudo (`products_macro.csv`) en formato anglosajón. La motivación principal es demostrar la aplicación práctica de las tres fases clave de un proyecto de Business Intelligence:

1. **ETL con Power Query**: abordar problemas reales de encoding, separadores decimales y columnas sin valor analítico.
2. **Modelado semántico**: entender cuándo un esquema de tabla única es suficiente (catálogo estático sin hechos transaccionales) y cómo optimizar el peso del modelo.
3. **Capa analítica DAX + Visualización**: traducir preguntas de negocio (% promociones, precio medio, concentración por categorías) en medidas DAX eficientes y visualizaciones accesibles.

Adicionalmente, se hace especial énfasis en **UI/UX y accesibilidad**: uso de paletas de alto contraste, etiquetas con texto (no solo color) y categorización de URLs como Image URL para renderizado dinámico. Estos aspectos suelen ser los grandes olvidados en proyectos BI académicos, pero resultan críticos en entornos profesionales.

---

<p align="center">Creado por @migueljerico y documentado por Ollama Cloud (MiniMax M3) · 2026</p>
