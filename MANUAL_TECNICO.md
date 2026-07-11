# 📘 Manual Técnico - Power BI Dashboard Mercadona

## 1. Arquitectura General

```text
┌─────────────────────────────────────────────────────────────────┐
│                    ARQUITECTURA DE CAPAS                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌──────────────┐    ┌──────────────────┐    ┌──────────────┐  │
│  │  ORIGEN      │    │  POWER QUERY     │    │  MODELO      │  │
│  │  DATOS       │───▶│  (CAPA ETL - M)  │───▶│  SEMÁNTICO   │  │
│  │  products_   │    │                  │    │  (Tabla      │  │
│  │  macro.csv   │    │  • Desactivar    │    │   products)  │  │
│  │  (CSV UTF-8) │    │    auto-tipo     │    │              │  │
│  └──────────────┘    │  • Estandarizar  │    │  • Una tabla │  │
│                      │    decimales .→, │    │  • Columnas  │  │
│                      │  • Col. En_      │    │    calculadas│  │
│                      │    Promocion (M) │    │  • Categoriz.│  │
│                      │  • Eliminar      │    │    Image URL │  │
│                      │    secondary_    │    └──────┬───────┘  │
│                      │    image_url     │           │          │
│                      └────────┬─────────┘           ▼          │
│                               │                    ┌────────┐  │
│                               ▼                    │  DAX   │  │
│                      ┌──────────────────┐          │ MEASURES│  │
│                      │  CARGA EN MODELO │─────────▶│         │  │
│                      │  (VertiPaq)      │          │ • Avg  │  │
│                      └──────────────────┘          │ • %Prom│  │
│                                                    └────┬───┘  │
│                                                         │      │
│                                                         ▼      │
│                                              ┌────────────────┐│
│                                              │  VISUALIZACIÓN ││
│                                              │  (2 Páginas)   ││
│                                              │  • KPIs Cards  ││
│                                              │  • Barras/Anil.││
│                                              │  • Tabla Imgs  ││
│                                              └────────────────┘│
└─────────────────────────────────────────────────────────────────┘
```

**Flujo de datos**: `CSV (anglosajón)` → `Power Query (M)` → `Modelo VertiPaq (tabla única optimizada)` → `Capa DAX (medidas)` → `Visuales interactivos (2 páginas)`

## 2. Descripción de Módulos y Componentes

### 2.1 Power Query (Editor de Consultas) - `products_macro`
**Archivo origen**: `products_macro.csv` (encoding UTF-8, separador coma, decimales con punto)

| Paso | Operación (Lenguaje M) | Descripción técnica |
| :--- | :--- | :--- |
| 1 | `Source = Csv.Document(...)` | Lectura cruda sin promoción de encabezados automática |
| 2 | `PromotedHeaders` | Promoción de primera fila a nombres de columna |
| 3 | **Eliminado** `Changed Type` auto | **Crítico**: Se elimina el paso automático para evitar conversión errónea de decimales (punto → coma) |
| 4 | `ReplacedValue` en `price`, `discount_price` | `Text.Replace([price], ".", ",")` → Conversión a formato español |
| 5 | `Changed Type` manual | `price` → `Decimal Number`, `discount_price` → `Decimal Number` (nullable) |
| 6 | `Table.AddColumn` → `En_Promocion` | `each if [discount_price] <> null then "Sí" else "No"` (tipo `text`) |
| 7 | `Table.RemoveColumns` | Eliminación de `secondary_image_url` (no valor analítico, reduce peso) |
| 8 | `Table.TransformColumnTypes` final | Asegura tipos: `id` (Int64), `category` (text), `price` (decimal), etc. |

**Salida**: Tabla `products` limpia, tipada, con columna de negocio `En_Promocion`.

### 2.2 Modelo Semántico (VertiPaq)
- **Estructura**: Tabla única `products` (no Star Schema clásico al ser catálogo estático sin hechos transaccionales).
- **Optimizaciones**:
  - Eliminación de `secondary_image_url` (URLs largas, alta cardinalidad, sin uso analítico).
  - Conservación de `subtitle` (valor descriptivo para tooltips).
  - Categorización de `main_image_url` → **Data Category: Image URL** (renderizado nativo).
  - `En_Promocion` como columna de texto de baja cardinalidad ("Sí"/"No") ideal para slicers.
- **Relaciones**: N/A (modelo de tabla única).

### 2.3 Capa DAX - Medidas Calculadas
**Tabla de medidas**: `_Measures` (buena práctica: tabla calculada vacía para agrupar medidas)

| Medida | Fórmula DAX | Descripción | Formato |
| :--- | :--- | :--- | :--- |
| `Precio Medio del Catálogo` | `AVERAGE(products[price])` | Media aritmética de precio base de todo el catálogo | Currency (€) |
| `Descuento Medio` | `AVERAGE(products[discount_price])` | Media de precio promocional (solo filas con descuento; ignora BLANK) | Currency (€) |
| `% Productos Destacados` | `DIVIDE(CALCULATE(COUNTROWS(products), products[En_Promocion] = "Sí"), COUNTROWS(products))` | Penetración de productos en promoción sobre total catálogo | Percentage (2 dec.) |
| `Total Productos` | `COUNTROWS(products)` | Conteo de IDs únicos (clave primaria) | Whole Number |
| `Nº Categorías` | `DISTINCTCOUNT(products[category])` | Cardinalidad de categorías distintas | Whole Number |

> **Nota**: `AVERAGE` sobre `discount_price` ignora automáticamente `BLANK` (productos sin promoción), por lo que `Descuento Medio` refleja solo el precio medio *de los productos promocionados*.

### 2.4 Visualización - Detalle de Páginas

#### Página 1: `Visión General`
| Visual | Tipo | Configuración clave |
| :--- | :--- | :--- |
| KPI Cards (3) | Card | Bordes de acento: Azul oscuro `#253494`, Azul `#0072B2`, Verde azulado `#01665E` |
| Top 10 Categorías | Clustered Bar Chart | Eje Y: `category`, Valor: `Precio Medio del Catálogo` (medida), Orden descendente, Top N=10 |
| Distribución Volumen | Donut Chart | Leyenda: `category`, Valores: `Total Productos`, Top N=5, Etiquetas: Categoría + % |
| Slicer Panel | Slicer | Campos: `category`, `En_Promocion` (horizontal, multi-select) |

#### Página 2: `Detalle y Análisis`
| Visual | Tipo | Configuración clave |
| :--- | :--- | :--- |
| Tabla Productos | Table | Columnas: `main_image_url` (Image URL), `id`, `title`, `subtitle`, `category`, `price`, `discount_price`, `En_Promocion` |
| Filtros Avanzados | Slicer | `category` (dropdown), `En_Promocion` (checkbox), `price` (range slider) |
| Tendencia Precios | Line Chart | Eje X: `category` (ordenado por precio medio), Valores: `Precio Medio del Catálogo` |

## 3. APIs y Endpoints
**N/A** — Reporte local autónomo (`.pbix` / `.pbit`). No expone API REST ni endpoints. La integración externa se realiza mediante:
- **Power BI Service**: Publicación en workspace → Embed token / Publish to web.
- **Power BI Embedded**: Para integración en aplicaciones propias (requiere capacidad Azure).

## 4. Variables de Entorno / Parámetros

| Variable / Parámetro | Valor de ejemplo | Obligatoria | Descripción |
| :--- | :--- | :--- | :--- |
| `PATH_DATA` (Parámetro Power Query) | `C:\Datos\Mercadona\products_macro.csv` | **Sí** (en `.pbit`) | Ruta absoluta al archivo CSV de origen. En `.pbix` va embebido. |
| `LOCALE` (Configuración regional) | `es-ES` | Sí | Formato decimal (coma), moneda (€), separador de miles (punto). |
| `GATEWAY_PATH` (Power BI Service) | `\\servidor-datos\compartido\Mercadona\` | Solo si ruta red | Ruta UNC para Gateway de datos empresarial (actualización programada). |

**Configuración en Power Query (Parámetro)**:
```powerquery
// En Power Query → Administrar parámetros → Nuevo parámetro
Nombre: PathOrigen
Tipo: Texto
Valor actual: C:\Datos\Mercadona\products_macro.csv
// Luego en Source:
Source = Csv.Document(File.Contents(PathOrigen), [Delimiter=",", Encoding=65001, QuoteStyle=QuoteStyle.Csv])
```

## 5. Guía de Despliegue Paso a Paso

### 5.1 Desarrollo Local (Power BI Desktop)
1. Abrir `Mercadona_Analysis.pbix` → Verificar datos cargados.
2. Si se usa `.pbit`: Al abrir, introducir ruta válida a `products_macro.csv`.
3. Validar medidas DAX (pestaña **Modelado** → **Nueva medida** → pegar fórmulas).
4. Revisar visuales: Página 1 y 2, interacciones (Formato → Editar interacciones).
5. Guardar como `.pbix` (con datos) o `.pbit` (plantilla).

### 5.2 Publicación en Power BI Service (Cloud)
1. En Power BI Desktop: **Archivo → Publicar → Publicar en Power BI** → Seleccionar Workspace.
2. En Power BI Service (app.powerbi.com):
   - Ir al Workspace → Conjunto de datos `Mercadona_Analysis` → **Configuración**.
   - **Credenciales de origen de datos**: Si usa parámetro `PathOrigen` con ruta local → **No se actualizará en la nube**.
   - **Solución**: Mover CSV a **SharePoint / OneDrive Empresarial / Azure Blob** y reconfigurar origen en Power Query a `Web.Contents` o `SharePoint.Files`.
   - **Gateway**: Si el CSV permanece en red local (ruta UNC), instalar **On-premises Data Gateway** → Configurar en Service → Asignar al dataset.
3. Programar actualización: **Configuración → Actualización programada** → Diaria (ej. 06:00 AM).
4. Crear **Informe** basado en el dataset → Compartir / Incrustar / App.

### 5.3 Despliegue como Plantilla Reutilizable (`.pbit`)
1. **Archivo → Exportar → Plantilla de Power BI**.
2. Incluir parámetro `PathOrigen` con descripción: "Ruta completa al archivo products_macro.csv (formato CSV, decimales con punto)".
3. Distribuir `.pbit`: El usuario final abre → introduce su ruta → modelo se construye automáticamente.

## 6. Limitaciones Conocidas y Mejoras Futuras

### Limitaciones Actuales
| # | Limitación | Impacto | Causa Raíz |
| :--- | :--- | :--- | :--- |
| 1 | **Dependencia de ruta local** | No actualizable en Power BI Service sin Gateway o migración a cloud | Origen `File.Contents(PathOrigen)` en Power Query |
| 2 | **Modelo tabla única** | No soporta análisis transaccional (ventas, tickets, tiempo) | Fuente es catálogo estático, no fact table |
| 3 | **Sin dimensión temporal** | No hay análisis YoY, MoM, YTD | Datos no tienen fecha de carga/versión |
| 4 | `discount_price` semántica ambigua | `Descuento Medio` ≠ % descuento medio; es precio promo medio | Columna original es "precio web destacado", no descuento transaccional |
| 5 | Imágenes externas | Rendimiento variable; roturas de enlace si URL cambia | `main_image_url` apunta a CDN externo (Mercadona) |
| 6 | Sin RLS (Row Level Security) | No hay filtrado por rol/usuario | Proyecto individual, sin multi-tenancy |

### Mejoras Futuras (Roadmap)
| Prioridad | Mejora | Descripción Técnica |
| :--- | :--- | :--- |
| **Alta** | **Migrar a Dataflows / Dataflow Gen2** | ETL en la nube (Power Query Online) → Fuente: CSV en OneDrive/SharePoint → Actualización automática sin Gateway. |
| **Alta** | **Parametrizar origen a Web/SharePoint** | Cambiar `File.Contents` → `SharePoint.Files` o `Web.Contents` con URL pública. |
| **Media** | **Enriquecer con datos de ventas** | Unir catálogo (`products`) con fact table `Sales` (ticket_id, fecha, cantidad, tienda) → Star Schema real. |
| **Media** | **Añadir dimensión Calendario** | Tabla `Calendar` (DAX: `CALENDARAUTO()`) → Time Intelligence (SAMEPERIODLASTYEAR, TOTALYTD). |
| **Media** | **Cachear imágenes localmente** | Power Query: `Binary.From(Web.Contents([main_image_url]))` → Columna binaria → Independencia de CDN externo. |
| **Baja** | **Implementar RLS** | Roles: `GestorCategoria` (filtro `category`), `Direccion` (todo). |
| **Baja** | **Versión móvil (Phone Layout)** | Vista específica en Power BI Desktop → Vista → Disposición de teléfono. |
| **Baja** | **Exportar a PDF / Paginated Report** | Para impresión ejecutiva estática (Report Builder / Export API). |

---

## Apéndice: Código Fuente Clave (Referencia Rápida)

### Power Query (M) - Paso `En_Promocion`
```powerquery
let
    Source = Csv.Document(File.Contents(PathOrigen), [Delimiter=",", Encoding=65001, QuoteStyle=QuoteStyle.Csv]),
    PromotedHeaders = Table.PromoteHeaders(Source, [PromoteAllScalars=true]),
    // NO Changed Type automático
    ReplacedDecimals = Table.TransformColumns(PromotedHeaders, {{"price", each Text.Replace(_, ".", ","), type text}, {"discount_price", each Text.Replace(_, ".", ","), type text}}),
    TypedCols = Table.TransformColumnTypes(ReplacedDecimals, {{"price", type number}, {"discount_price", type number}, {"id", Int64.Type}}),
    AddedPromoFlag = Table.AddColumn(TypedCols, "En_Promocion", each if [discount_price] <> null then "Sí" else "No", type text),
    RemovedCols = Table.RemoveColumns(AddedPromoFlag, {"secondary_image_url"})
in
    RemovedCols
```

### DAX - Medidas Principales
```dax
-- Tabla _Measures (Enter data → Blank query → Name: _Measures)

Precio Medio del Catálogo = AVERAGE(products[price])

Descuento Medio = AVERAGE(products[discount_price])

% Productos Destacados = 
DIVIDE(
    CALCULATE(COUNTROWS(products), products[En_Promocion] = "Sí"),
    COUNTROWS(products)
)

Total Productos = COUNTROWS(products)

Nº Categorías = DISTINCTCOUNT(products[category])
```

---
*Manual técnico generado a partir del análisis del repositorio `migueljerico/powerbi-dashboard-mercadona`. Versión 2026.01.*