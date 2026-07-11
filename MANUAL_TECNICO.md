# 📘 Manual Técnico - Power BI Dashboard Mercadona

## 1. Arquitectura General
El flujo de datos sigue un proceso lineal de ETL hasta la capa de visualización:

`Origen de Datos (Excel/CSV)` → `Power Query (Limpieza/Transformación)` → `Modelo de Datos (Relaciones/DAX)` → `Capa de Presentación (Visuals)`

### Diagrama de Flujo
```text
  [Data Source] 
         ↓
  [Power Query] ──→ (Eliminación de nulos, Pivotado, Tipado de datos)
         ↓
  [Data Model]  ──→ (Tabla Hechos ↔ Tablas Dimensiones)
         ↓
  [DAX Engine]  ──→ (Cálculo de Medidas: Sums, Average, Time Intelligence)
         ↓
  [Dashboard]   ──→ (Gráficos, Mapas, Slicers)
```

## 2. Descripción de Módulos

### 🛠️ Módulo de ETL (Power Query)
Responsable de la ingesta de datos. Las transformaciones clave incluyen:
- **Normalización**: Conversión de formatos de fecha y moneda.
- **Limpieza**: Eliminación de duplicados en la tabla de productos.
- **Merge**: Combinación de tablas de ventas con tablas de categorías.
- **Estandarización regional**: Columnas `price` y `discount_price`: sustitución de `.` por `,` y conversión a **Número decimal**.
- **Variable de negocio `En_Promocion`**: Columna condicional en lenguaje M: `if [discount_price] <> null then "Sí" else "No"`. Nota: `discount_price` actúa como etiqueta de visibilidad web, no como descuento económico.
- **Reducción de dimensionalidad**: Eliminadas `subtitle` y `secondary_image_url` para optimizar el modelo.
- **Configuración multimedia**: `main_image_url` categorizada como **URL de imagen** para visualización dinámica del catálogo.

### 📊 Modelo de Datos
Se implementó un **Esquema en Estrella (Star Schema)**:
- **Tabla de Hechos**: `Ventas` (Contiene transacciones, cantidades y precios).
- **Dimensiones**: `Productos`, `Calendario`, `Tiendas`, `Categorías`.
- **Relaciones**: Relaciones de uno a muchos (1:*) desde las dimensiones hacia la tabla de hechos.

### 🧮 Capa de Cálculo (DAX)
Medidas clave implementadas:
```dax
-- Precio Medio del Catálogo
Precio Medio del Catálogo = AVERAGE(products[price])

-- % Productos Destacados
% Productos Destacados =
DIVIDE(
    CALCULATE(COUNTROWS(products), products[En_Promocion] = "Sí"),
    COUNTROWS(products)
)

-- Total Ventas
Total Ventas = SUM(Ventas[Monto])

-- Margen %
Margen % = DIVIDE([Beneficio Total], [Total Ventas], 0)

-- Crecimiento MoM
Crecimiento MoM = CALCULATE([Total Ventas], DATEADD('Calendario'[Fecha], -1, MONTH))
```

## 3. APIs y Endpoints
No aplica: el proyecto es un archivo `.pbix` autónomo sin API expuesta.

## 4. Variables de Entorno y Configuración
Dado que es un archivo `.pbix`, no utiliza variables de entorno tradicionales, pero requiere la siguiente configuración de acceso:

| Parámetro | Valor Sugerido | Obligatorio | Descripción |
| :--- | :--- | :--- | :--- |
| **Ruta Datos** | `C:\Datos\Mercadona\` | Sí | Directorio local donde residen los archivos fuente |
| **Permisos** | Lectura | Sí | Acceso al sistema de archivos para actualización de datos |
| **Fuente de datos** | `products_macro.csv` | Sí | Archivo CSV con catálogo de productos (formato anglosajón) |

## 5. Guía de Despliegue

1. **Publicación en la Nube**:
   - Abrir el archivo en Power BI Desktop.
   - Hacer clic en el botón `Publicar` → Seleccionar `Mi área de trabajo` o un Workspace compartido.

2. **Configuración de Gateway**:
   - Para mantener los datos actualizados automáticamente, instalar el *On-premises data gateway* en el servidor donde residen los archivos fuente.
   - Configurar la fuente de datos en el servicio Power BI para usar el gateway.

3. **Acceso Web**:
   - Generar un enlace de compartir o embeber el reporte en un portal de SharePoint/Teams.
   - Configurar permisos de visualización según roles (RLS futuro).

## 6. Limitaciones y Mejoras

### Limitaciones
- **Datos Estáticos**: El reporte depende de archivos locales; no está conectado a una base de datos SQL en tiempo real.
- **Volumen**: El rendimiento puede degradarse si el archivo Excel supera el millón de filas.
- **Falta de temporalidad**: La fuente actual no incluye dimensión de fecha completa para análisis de series temporales avanzadas.

### Mejoras Futuras
- Migración de la fuente de datos a **Azure SQL Database**.
- Implementación de **RLS (Row Level Security)** para que cada gerente de tienda solo vea sus propios datos.
- Integración de predicciones de ventas mediante **AI Visuals** (Forecasting).
- Añadir segmentadores de fecha/categoría si la fuente incorpora temporalidad.
- Implementar *drill-through* desde la categoría al detalle de producto con imagen (`main_image_url`).
- Revisar rendimiento del modelo si el catálogo crece significativamente (actualmente optimizado por reducción de columnas).

## 7. Diseño del Dashboard y UI/UX

### Página 1 — Visión General del Catálogo
**Objetivo**: Radiografía financiera y estructural de la oferta.

| Elemento | Configuración |
|----------|---------------|
| **KPIs superiores** | • **Total de Productos** — `COUNTROWS(products)` / `DISTINCTCOUNT(products[id])` — Borde `#253494`
• **Precio Medio del Catálogo** — Medida DAX — Borde `#0072B2`
• **Nº de Categorías** — `DISTINCTCOUNT(products[Category])` — Borde `#01665E` |
| **Gráfico de barras horizontales** | Top 10 categorías más caras — Eje Y: `Category`, Eje X: `AVERAGE(price)` — Color `#0072B2` |
| **Gráfico de anillos** | Distribución del volumen (Top 5 categorías) — Etiquetas: *categoría + porcentaje* — Paleta de 5 colores alto contraste sobre fondo blanco — Accesibilidad: no dependencia exclusiva del color. |

### Página 2 — Análisis de Promociones
**Objetivo**: Monitorizar la ejecución de la estrategia de "Productos Destacados".
- KPIs: % Productos Destacados, Precio medio en promoción vs no promoción.
- Visuales: Tabla de productos destacados con imagen dinámica (`main_image_url`), gráfico de dispersión precio vs descuento.
- Segmentadores: Categoría, Rango de precio.

### Paleta de colores y accesibilidad
- Colores corporativos de alto contraste: `#253494`, `#0072B2`, `#01665E`, más 5 tonos para el gráfico de anillos.
- Bordes de acento en tarjetas KPI para jerarquía visual.
- Etiquetas de datos activadas en gráficos para interpretación sin depender solo de la distinción cromática.