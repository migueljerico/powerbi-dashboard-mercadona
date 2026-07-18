# 📘 Manual Técnico - Power BI Dashboard Mercadona

## 1. Arquitectura General

El proyecto implementa una arquitectura BI clásica de **4 capas** con énfasis en optimización del modelo y separación clara de responsabilidades:

```text
┌──────────────────────────────────────────────────────────────────────┐
│                     ARQUITECTURA DE CAPAS                            │
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌────────────────┐    ┌──────────────────┐    ┌─────────────────┐  │
│  │  CAPA 1        │    │  CAPA 2          │    │  CAPA 3         │  │
│  │  ORIGEN DATOS  │───▶│  ETL (POWER      │───▶│  MODELO         │  │
│  │                │    │  QUERY - M)      │    │  SEMÁNTICO      │  │
│  │  products_     │    │                  │    │  (VertiPaq)     │  │
│  │  macro.csv     │    │  • Desactivar    │    │                 │  │
│  │  (CSV UTF-8)   │    │    auto-tipo     │    │  • Tabla única  │  │
│  │                │    │  • Estandarizar  │    │  • Columnas     │  │
│  │  • Enc: 65001  │    │    decimales     │    │    calculadas   │  │
│  │  • Sep: ","    │    │    . → ,         │    │  • Image URL    │  │
│  │  • Dec: "."    │    │  • Col.          │    │    category     │  │
│  │                │    │    En_Promocion  │    │                 │  │
│  │                │    │  • Eliminar      │    │                 │  │
│  │                │    │    secondary_    │    │                 │  │
│  │                │    │    image_url     │    │                 │  │
│  └────────────────┘    └────────┬─────────┘    └────────┬────────┘  │
│                                 │                      │           │
│                                 ▼                      ▼           │
│                       ┌──────────────────┐    ┌─────────────────┐  │
│                       │  CARGA MODELO    │    │  CAPA 4         │  │
│                       │  (VertiPaq       │    │  DAX MEASURES   │  │
│                       │   in-memory)     │───▶│  + VISUALES     │  │
│                       └──────────────────┘    │                 │  │
│                                                │  • Avg Price    │  │
│                                                │  • % Promoc.    │  │
│                                                │  • 2 Páginas    │  │
│                                                │  • Slicers      │  │
│                                                │  • Cross-filter │  │
│                                                └─────────────────┘  │
└──────────────────────────────────────────────────────────────────────┘
```

**Flujo de datos resumido**:
`CSV (anglosajón)` → `Power Query (M)` → `Modelo VertiPaq (tabla única optimizada)` → `Capa DAX (medidas)` → `Visuales interactivos (2 páginas)`

---

## 2. Descripción de Módulos y Componentes

### 2.1 📥 Capa 1 - Origen de Datos
- **Archivo**: `products_macro.csv`
- **Encoding**: UTF-8 (65001)
- **Separador**: Coma (,)
- **Decimales**: Punto (.) — formato anglosajón
- **Responsabilidad**: Proveer el dataset crudo del catálogo de productos Mercadona con todas las columnas originales incluyendo `secondary_image_url` (que será descartada en ETL).

### 2.2 🔄 Capa 2 - ETL Power Query (`products`)
**Archivo lógico**: Query `products` en el Editor de Power Query.

| Paso | Operación (Lenguaje M) | Descripción técnica |
| :--- | :--- | :--- |
| 1 | `Source = Csv.Document(...)` | Lectura cruda sin promoción de encabezados automática. |
| 2 | `PromotedHeaders` | Promoción de primera fila a nombres de columna. |
| 3 | **Eliminado** `Changed Type` auto | **Crítico**: Se elimina el paso automático para evitar conversión errónea de decimales (punto → coma). |
| 4 | `ReplacedValue` en `price`, `discount_price` | `Text.Replace([price], ".", ",")` → Conversión a formato español. |
| 5 | `Changed Type` manual | `price` → `Decimal Number`, `discount_price` → `Decimal Number` (nullable). |
| 6 | `Table.AddColumn` → `En_Promocion` | `each if [discount_price] <> null then "Sí" else "No"` (tipo `text`). |
| 7 | `Table.RemoveColumns` | Eliminación de `secondary_image_url` (no valor analítico, reduce peso). |
| 8 | `Table.TransformColumnTypes` final | Asegura tipos: `id` (Int64), `category` (text), `price` (decimal), etc. |

**Salida**: Tabla `products` limpia, tipada, con columna de negocio `En_Promocion`.

**Funciones / pasos exportados clave**:
- `En_Promocion`: columna calculada de baja cardinalidad ("Sí"/"No") para slicers.
- Tipado final estable que evita errores downstream en DAX.

### 2.3 🗃️ Capa 3 - Modelo Semántico (VertiPaq)
- **Estructura**: Tabla única `products` (no Star Schema clásico al ser catálogo estático sin hechos transaccionales).
- **Optimizaciones aplicadas**:
  - Eliminación de `secondary_image_url` (URLs largas, alta cardinalidad, sin uso analítico).
  - Conservación de `subtitle` (valor descriptivo para tooltips).
  - Categorización de `main_image_url` → **Data Category: Image URL** (renderizado nativo).
  - `En_Promocion` como columna de texto de baja cardinalidad ideal para slicers.
- **Relaciones**: N/A (modelo de tabla única).
- **Tabla auxiliar**: `_Measures` (tabla calculada vacía para alojar medidas DAX — buena práctica de organización).

### 2.4 🧠 Capa 4 - Medidas DAX
Las medidas se alojan en la tabla `_Measures` para facilitar mantenimiento.

| Medida | Fórmula DAX | Descripción | Formato |
| :--- | :--- | :--- | :--- |
| `Precio Medio del Catálogo` | `AVERAGE(products[price])` | Media aritmética de precio base de todo el catálogo. | Currency (€) |
| `Descuento Medio` | `AVERAGE(products[discount_price])` | Media de precio promocional (solo filas con descuento; ignora BLANK). | Currency (€) |
| `% Productos Destacados` | `DIVIDE(CALCULATE(COUNTROWS(products), products[En_Promocion] = "Sí"), COUNTROWS(products))` | Penetración de productos en promoción sobre total catálogo. | Percentage (2 dec.) |
| `Total Productos` | `COUNTROWS(products)` | Conteo de IDs únicos (clave primaria). | Whole Number |

---

## 3. APIs y Endpoints

> ⚠️ Este proyecto **no expone APIs REST** al ser un informe Power BI Desktop autocontenido. Sin embargo, la capa de datos podría integrarse en el futuro con:

| Origen Potencial | Tipo | Descripción |
| :--- | :--- | :--- |
| Power BI Service | Nube (SaaS) | Publicación del `.pbix` para consumo compartido. |
| Power BI Embedded | Embed en web | Inserción de visuales en aplicaciones externas vía REST API de Microsoft. |
| Actualización programada | Power BI Gateway | Refresh automático desde CSV local o OneDrive. |
| Power BI REST API | REST | Para refresh programático, exportación o integración con terceros. |

**Endpoint clave si se publica en Power BI Service**:
```
POST https://api.powerbi.com/v1.0/myorg/datasets/{datasetId}/refreshes
```
Parámetros: `datasetId`, cabeceras con token OAuth2 de Azure AD.

---

## 4. Variables de Entorno y Configuración

Power BI Desktop no utiliza variables de entorno al estilo de aplicaciones backend, pero el proyecto tiene parámetros configurables en la ETL y publicación:

| Parámetro / Variable | Valor de ejemplo | Obligatoria | Descripción |
| :--- | :--- | :--- | :--- |
| `Ruta CSV origen` | `C:\Datos\Mercadona\products_macro.csv` | Sí (en `.pbit`) | Ruta absoluta del dataset. En `.pbix` los datos van embebidos. |
| `Encoding` | `65001` (UTF-8) | Sí | Encoding de lectura del CSV en `Csv.Document`. |
| `Delimitador` | `","` | Sí | Separador de campos. |
| `Decimal` | `.` (origen) / `,` (transformado) | Sí | Power BI almacena en formato neutro; la transformación M estandariza a coma para display. |
| `Power BI Gateway` (si se publica) | Modo personal o empresarial | No (solo cloud) | Para refresh programado desde origen local. |
| `Workspace Power BI Service` | `MiWorkspace` | No (solo cloud) | Destino de publicación en la nube. |
| `Token Azure AD` (API) | OAuth2 Bearer | Solo si automatiza refresh | Acceso programático vía REST API. |

---

## 5. Guía de Despliegue

### 5.1 Despliegue Local (Escritorio)
1. Instalar Power BI Desktop (ver sección Instalación del README).
2. Clonar el repositorio y abrir el archivo `.pbix` (datos embebidos) o `.pbit` (requiere ruta CSV).
3. Validar visualización de KPIs y slicers en Página 1 y Página 2.

### 5.2 Despliegue en Power BI Service (Nube)
1. Iniciar sesión en [app.powerbi.com](https://app.powerbi.com).
2. Dentro de Power BI Desktop, ir a **Inicio → Publicar → Mi área de trabajo** (o un workspace específico).
3. Esperar la carga del dataset y el informe.
4. (Opcional) Configurar **Actualización programada**:
   - **Mi área de trabajo → Datasets → Configuración → Actualización programada**.
   - Definir frecuencia y hora.
   - Si el origen es local, instalar **On-premises data gateway** y vincularlo al workspace.
5. (Opcional) **Insertar en SharePoint o Web**:
   - Informe → Insertar → Copiar código HTML / enlace de inserción.
   - Pegar en intranet o sitio web corporativo.

### 5.3 Despliegue como Plantilla (.pbit)
Para distribuir a múltiples analistas con sus propios datos:
1. Compartir el archivo `.pbit`.
2. El destinatario abre en Power BI Desktop, introduce la ruta de su CSV local.
3. La ETL se aplica automáticamente y el dashboard queda operativo.

### 5.4 Automatización vía REST API (Avanzado)
```bash
# Autenticación con Azure AD (ejemplo con curl + MSAL)
TOKEN=$(az account get-access-token --resource https://analysis.windows.net/powerbi/api --query accessToken -o tsv)

# Refrescar dataset
curl -X POST \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  https://api.powerbi.com/v1.0/myorg/datasets/{datasetId}/refreshes
```

---

## 6. Limitaciones Conocidas y Mejoras Futuras

### 6.1 Limitaciones Actuales
- **Modelo de tabla única**: No permite análisis temporal avanzado (tendencias, comparativas YoY) por carecer de tabla de fechas y hechos transaccionales.
- **Datos estáticos**: El CSV es una instantánea; no hay histórico de precios ni cambios de categoría.
- **Sin segmentación geográfica**: No se analiza distribución por tienda, provincia o comunidad autónoma.
- **Encoding regional manual**: La ETL requiere intervención explícita para evitar mala interpretación de decimales — no es 100% plug & play.
- **Dependencia de Power BI Desktop**: No se puede ejecutar en sistemas sin licencia (o requiere cuenta Pro/PP para Service).
- **Sin seguridad RLS**: Todos los usuarios ven el mismo dataset sin filtros a nivel de fila.

### 6.2 Posibles Mejoras Futuras
- **Incorporar tabla Calendario (`DimDate`)** y modelar Star Schema con hechos transaccionales de ventas.
- **Scraping automatizado** del catálogo Mercadona para actualización periódica (Python + BeautifulSoup o Selenium).
- **Implementar RLS** para segmentar visibilidad por rol (compras, marketing, dirección).
- **Añadir forecasting** con Python/R visual integration (predicción de precios, demanda).
- **Integración con Power Apps** para captura de feedback sobre productos destacados.
- **Alerting automático** vía Power BI Dataflows + Power Automate cuando `% Productos Destacados` supere umbrales.
- **Versionado semántico de la plantilla `.pbit`** y parametrización avanzada de rutas CSV.
- **Migración a dataset compartido en Power BI Service** con certificación y linaje de datos.
