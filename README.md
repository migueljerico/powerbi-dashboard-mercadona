# 📊 Power BI Dashboard - Catálogo Mercadona

![Power BI](https://img.shields.io/badge/Power_BI-Desktop-F2C811?style=for-the-badge&logo=powerbi&logoColor=black)
![DAX](https://img.shields.io/badge/DAX-Medidas_Anal%C3%ADticas-0072B2?style=for-the-badge)
![Power Query](https://img.shields.io/badge/Power_Query-Lenguaje_M-253494?style=for-the-badge)
![Estado](https://img.shields.io/badge/Estado-Publicado-success?style=for-the-badge)
![Licencia](https://img.shields.io/badge/Licencia-GPL_3.0-blue?style=for-the-badge)

*Cuadro de mando interactivo en Power BI para el análisis estratégico del catálogo de productos de Mercadona, con pipeline ETL regionalizado y métricas DAX de negocio.*

---

## 🔗 Acceso / Demo

Este proyecto está diseñado para ejecutarse localmente en **Power BI Desktop**. No existe un despliegue en la nube público, pero el informe puede publicarse en **Power BI Service** para consumo compartido en la organización.

| Recurso | Descripción |
|---|---|
| Archivo `.pbix` | `Ejercicio_3.8_Proyecto_Ciencia_Datos_Mercadona_Miguel_Jeric_.pbix` — informe completo con datos cargados en memoria |
| Plantilla `.pbit` | `Ejercicio_3.8_Proyecto_Ciencia_Datos_Mercadona_Miguel_Jeric_.pbit` — plantilla ligera sin datos embebidos |
| Documentación técnica | `MANUAL_TECNICO.md` y `docs/Analisis_Catalogo_Mercadona_PowerBI_Miguel_Jerico.md` |
| Análisis en PDF | `Analisis_Catalogo_Mercadona_PowerBI_Miguel_Jerico.pdf` — memoria del proyecto en formato imprimible |

---

## 📋 Descripción

Este proyecto documenta la creación de un cuadro de mando (*Dashboard*) interactivo en Power BI para analizar de manera detallada el catálogo de productos de Mercadona. La solución procesa datos brutos mediante una sólida fase de ETL y modelado para ofrecer a la dirección una herramienta visual e interactiva que facilita la monitorización de la distribución de precios, el peso de las categorías y la efectividad de la estrategia de "Productos Destacados" o promociones.

El conjunto de datos original (`products_macro.csv`) requería una depuración profunda antes de realizar los cálculos del modelo. Se aplicaron transformaciones en Power Query para corregir formatos numéricos anglosajones (punto como separador decimal) y adaptarlos al estándar decimal de España (coma), enriquecer el modelo con variables lógicas de negocio en lenguaje M y optimizar la compresión del motor columnar VertiPaq mediante la eliminación de columnas sin valor analítico. Sobre este modelo limpio, se implementaron medidas DAX específicas para alimentar los KPIs clave del informe.

La interfaz del reporte se divide en dos pantallas estratégicamente diseñadas para diferentes perfiles de toma de decisiones: una página de **Visión General del Catálogo** con KPIs financieros y estructurales, y una página de **Detalle y Análisis Específico** orientada a la exploración granular con imágenes dinámicas de productos. Todo el diseño emplea una paleta corporativa de alto contraste y etiquetas de lectura accesible para garantizar la facilidad de uso.

---

## ✨ Funcionalidades

| Funcionalidad | Descripción |
|---|---|
| **ETL regionalizado** | Reemplazo de separadores decimales anglosajones (`.` → `,`) en `price` y `discount_price` para adaptación al estándar español |
| **Columna condicional `En_Promocion`** | Generada en lenguaje M para segmentar productos con etiqueta promocional activa (`Sí` / `No`) |
| **Optimización VertiPaq** | Eliminación de `secondary_image_url` por carecer de valor analítico, reduciendo dimensionalidad y peso del modelo |
| **URLs de imagen dinámicas** | Categorización de `main_image_url` como *URL de la imagen* para renderizado visual en tablas interactivas |
| **Medida: Precio Medio del Catálogo** | `AVERAGE(products[price])` — promedio general del catálogo completo |
| **Medida: Descuento Medio** | `AVERAGE(products_macro[discount_price])` — promedio de artículos con promoción activa |
| **Medida: % Productos Destacados** | `DIVIDE(CALCULATE(...), COUNTROWS(...))` — peso porcentual de promociones sobre la oferta total |
| **Página 1: Visión General** | KPIs en tarjetas con bordes de acento (`#253494`, `#0072B2`, `#01665E`), Top 10 categorías más caras y gráfico de anillos con etiquetas accesibles |
| **Página 2: Detalle y Análisis** | Tabla dinámica con imágenes reales, gráficos de tendencia y segmentadores por categoría / estado de promoción |
| **Parámetro configurable** | `Ruta_Origen_CSV` permite actualizar la fuente de datos sin modificar la lógica del modelo |

---

## ⚙️ Instalación

1. **Clonar el repositorio:**
```bash
git clone https://github.com/migueljerico/powerbi-dashboard-mercadona.git
cd powerbi-dashboard-mercadona
```

2. **Verificar requisitos previos:**
   - Instalar **Power BI Desktop** (versión gratuita disponible en Microsoft Store o en [powerbi.microsoft.com](https://powerbi.microsoft.com)).
   - Comprobar que el archivo `products_macro.csv` se encuentra en la raíz del repositorio.

3. **Abrir el informe:**
   - Doble clic sobre `Ejercicio_3.8_Proyecto_Ciencia_Datos_Mercadona_Miguel_Jeric_.pbix` para abrir el informe completo con datos cargados.
   - Alternativamente, abrir la plantilla `.pbit` para un inicio ligero; Power BI solicitará la ruta del CSV en el primer arranque.

4. **Configurar la ruta de origen (si es necesario):**
   - En Power BI Desktop → pestaña **Inicio** → **Transformar datos** → **Administrar parámetros** → modificar `Ruta_Origen_CSV` con la ruta local del archivo `products_macro.csv`.

5. **Consultar la documentación técnica:**
   - Revisar `MANUAL_TECNICO.md` para detalles sobre arquitectura de capas, componentes y configuración del motor VertiPaq.
   - Revisar `docs/Analisis_Catalogo_Mercadona_PowerBI_Miguel_Jerico.md` para el análisis funcional completo.

---

## 🚀 Uso

### Apertura y navegación del dashboard

Abre el archivo `.pbix` en Power BI Desktop. El informe contiene dos páginas accesibles desde las pestañas inferiores:

- **Página 1 — Visión General del Catálogo:** Radiografía financiera con KPIs superiores (Total de Productos, Precio Medio, Nº de Categorías), gráfico de barras horizontales (Top 10 categorías más caras) y gráfico de anillos (distribución de volumen del Top 5 de categorías).
- **Página 2 — Detalle y Análisis Específico:** Tabla interactiva con imágenes dinámicas de productos, gráficos de tendencia y segmentadores por categoría o estado de promoción.

### Actualización de datos

Para adaptar el informe a un nuevo conjunto de datos o actualizaciones de inventario, modifica el parámetro de origen en Power Query:

```text
Power BI Desktop → Transformar datos → Administrar parámetros → Ruta_Origen_CSV
```

### Transformaciones ETL aplicadas (Power Query / Lenguaje M)

```powerquery
// Reemplazo de separadores decimales anglosajones por estándar español
Table.ReplaceValue(#"Tipo cambiado", ".", ",", Replacer.ReplaceText, {"price", "discount_price"})

// Generación de columna condicional de negocio
Table.AddColumn(#"Tipo cambiado", "En_Promocion", each if [discount_price] <> null then "Sí" else "No")
```

### Medidas DAX implementadas

```dax
// Precio promedio general del catálogo
Precio Medio del Catálogo = AVERAGE(products[price])

// Precio promedio de artículos con promoción activa
Descuento Medio = AVERAGE(products_macro[discount_price])

// Peso porcentual de productos destacados sobre el total
% Productos Destacados = 
DIVIDE(
    CALCULATE(COUNTROWS(products), products[En_Promocion] = "Sí"),
    COUNTROWS(products)
)
```

### Publicación en Power BI Service

```text
Power BI Desktop → Pestaña "Inicio" → "Publicar" → Seleccionar área de trabajo
```

---

## 📁 Estructura del proyecto

```text
powerbi-dashboard-mercadona/
├── README.md
├── MANUAL_TECNICO.md
├── LICENSE
├── Analisis_Catalogo_Mercadona_PowerBI_Miguel_Jerico.pdf
├── Ejercicio_3.8_Proyecto_Ciencia_Datos_Mercadona_Miguel_Jeric_.pbix
├── Ejercicio_3.8_Proyecto_Ciencia_Datos_Mercadona_Miguel_Jeric_.pbit
├── products_macro.csv
└── docs/
    └── Analisis_Catalogo_Mercadona_PowerBI_Miguel_Jerico.md
```

---

## 🛠️ Tecnologías

| Herramienta | Versión / Detalle | Uso en el proyecto |
|---|---|---|
| **Power BI Desktop** | Aplicación de escritorio (Microsoft) | Motor de visualización, modelado semántico y renderizado del dashboard interactivo |
| **Power Query (Lenguaje M)** | Editor integrado de Power BI | Pipeline ETL: limpieza, transformación regional, enriquecimiento condicional y reducción de dimensionalidad |
| **DAX (Data Analysis Expressions)** | Medidas calculadas | Lógica analítica: `Precio Medio del Catálogo`, `Descuento Medio`, `% Productos Destacados` |
| **Motor VertiPaq** | In-Memory columnar (xVelocity) | Compresión y almacenamiento en memoria del modelo semántico optimizado |
| **CSV (UTF-8)** | `products_macro.csv` — delimitador coma | Fuente de datos plana con catálogo de productos de Mercadona (8 campos: `id`, `title`, `subtitle`, `category`, `price`, `discount_price`, `main_image_url`, `secondary_image_url`) |
| **Power BI Service** | Nube Microsoft (opcional) | Publicación y consumo compartido del informe en la organización |
| **Git

<p align="center">Creado por <a href="https://github.com/migueljerico">@migueljerico</a> y documentado por QwenCloud (glm-5.2) desde la App Asistente de IA · 2026</p>