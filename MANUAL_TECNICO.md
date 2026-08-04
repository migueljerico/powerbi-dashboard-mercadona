# 📘 Manual Técnico - Power BI Dashboard Mercadona

## 1. Arquitectura General

El proyecto implementa una arquitectura de Business Intelligence (BI) orientada a capas, optimizada para el motor analítico en memoria **VertiPaq** de Microsoft Power BI. La arquitectura garantiza la separación de responsabilidades entre la extracción/transformación de datos (ETL), el modelado semántico, la lógica de negocio en DAX y la capa de presentación interactiva.

```text
┌────────────────────────────────────────────────────────────────────────────────────────┐
│                                ARQUITECTURA DE CAPAS                                   │
├────────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                        │
│  ┌───────────────────────┐                                                             │
│  │  CAPA DE DATOS        │                                                             │
│  │  products_macro.csv   │                                                             │
│  │  (CSV UTF-8, "." dec) │                                                             │
│  └───────────┬───────────┘                                                             │
│              │                                                                         │
│              ▼                                                                         │
│  ┌───────────────────────┐                                                             │
│  │  CAPA DE ETL          │                                                             │
│  │  Power Query (M)      │                                                             │
│  │  • Reemplazo dec .->, │                                                             │
│  │  • Col "En_Promocion" │                                                             │
│  │  • Drop secondary_img │                                                             │
│  └───────────┬───────────┘                                                             │
│              │                                                                         │
│              ▼                                                                         │
│  ┌───────────────────────┐                                                             │
│  │  CAPA DEL MODELO      │                                                             │
│  │  Modelo Semántico     │                                                             │
│  │  • VertiPaq In-Memory │                                                             │
│  │  • Tabla 'products'   │                                                             │
│  │  • Tabla '_Measures'  │                                                             │
│  └───────────┬───────────┘                                                             │
│              │                                                                         │
│              ▼                                                                         │
│  ┌───────────────────────┐      ┌───────────────────────────────────────────────────┐  │
│  │  CAPA DE LÓGICA       │ ───▶ │ CAPA DE PRESENTACIÓN / VISUALES                   │  │
│  │  Medidas DAX          │      │ • Página 1: Visión General (KPIs, Top 10, Donut)  │  │
│  │  • Avg Price, % Promo │      │ • Página 2: Detalle & Galería URL dinámicas       │  │
│  └───────────────────────┘      └───────────────────────────────────────────────────┘  │
└────────────────────────────────────────────────────────────────────────────────────────┘
```

Flujo conceptual de dependencias entre capas:
```text
└── Capa de datos (CSV UTF-8)
    └── Capa de ETL (Power Query / Lenguaje M)
        └── Capa de modelo (VertiPaq In-Memory Engine)
            └── Capa de lógica (Medidas DAX & Tabla _Measures)
                └── Capa de presentación (Dashboard 2 Páginas / UX Cross-filtering)
```

---

## 2. Descripción de Módulos y Componentes

### 2.1 📥 Capa de Datos (Origen)
- **Archivo de datos**: `products_macro.csv`
- **Ubicación**: Raíz del proyecto / Origen configurable en Power Query.
- **Formato**: CSV (UTF-8, codificación 65001).
- **Delimitador de campos**: Coma (`,`).
- **Formato numérico de origen**: Anglosajón (punto `.` como separador decimal).
- **Responsabilidad**: Proporcionar el catálogo plano de productos de Mercadona con atributos descriptivos (`id`, `title`, `subtitle`, `category`, `price`, `discount_price`, `main_image_url`, `secondary_image_url`).

---

### 2.2 🔄 Capa de ETL (Power Query - Lenguaje M)
- **Componente**: Query principal `products` en el editor de Power Query.
- **Responsabilidad**: Extracción, limpieza, estandarización regional de separadores decimales, enriquecimiento de atributos de negocio y optimización dimensional.

#### Pipeline de Pasos Aplicados en M:

| Paso M | Operación / Función M | Descripción Técnica |
| :--- | :--- | :--- |
| 1 | `Csv.Document(File.Contents(...))` | Carga el archivo crudo especificado asegurando codificación UTF-8 (65001) y delimitador coma. |
| 2 | `Table.PromoteHeaders` | Promociona la primera fila del CSV a cabeceras de columna. |
| 3 | **Eliminación del paso automático** `Changed Type` | **Crítico**: Previene la conversión automática errónea de decimales anglosajones (`.`) en entornos con configuración regional de coma (`,`). |
| 4 | `Table.ReplaceValue` en `price` y `discount_price` | Sustituye los caracteres `.` por `,` mediante `Text.Replace` sobre las columnas numéricas. |
| 5 | `Table.TransformColumnTypes` (Tipado Numérico) | Convierte `price` a `Decimal.Type` y `discount_price` a `Nullable Decimal.Type`. |
| 6 | `Table.AddColumn` (`En_Promocion`) | Crea la columna lógica de negocio: `if [discount_price] <> null then "Sí" else "No"`. |
| 7 | `Table.RemoveColumns` | Elimina `secondary_image_url` para reducir espacio en el modelo VertiPaq y acelerar tiempo de recarga. *(Nota: Se conserva `subtitle` para enriquecer tooltips)*. |
| 8 | `Table.TransformColumnTypes` (Tipado Final) | Garantiza tipos definitivos: `id` (Int64.Type), `category` (Text.Type), `title` (Text.Type), `main_image_url` (Text.Type). |

---

### 2.3 🗃️ Capa de Modelo Semántico (VertiPaq In-Memory)
- **Estructura del modelo**: Modelo de tabla única (`products`) complementado por una tabla técnica organizativa (`_Measures`).
- **Ajustes de categorización de datos**:
  - Columna `main_image_url`: Configurada explícitamente con **Data Category = Image URL** para permitir el renderizado dinámico de la imagen en visuales de tabla o matriz.
  - Columna `En_Promocion`: Atributo de baja cardinalidad ("Sí" / "No") optimizado para segmentadores (Slicers) y filtrado de alto rendimiento.
- **Tabla Técnica de Medidas**: `_Measures` (tabla en blanco creada para consolidar todas las métricas calculadas centralizadamente).

---

### 2.4 🧠 Capa de Lógica de Negocio (Medidas DAX)
Todas las fórmulas DAX están agrupadas dentro de la tabla `_Measures`.

| Nombre de Medida | Fórmulas DAX | Descripción y Contexto | Formato / Display |
| :--- | :--- | :--- | :--- |
| `Total Productos` | `COUNTROWS(products)` | Cuenta la cantidad total de filas (productos únicos) en el contexto de filtro actual. | Whole Number (`#,0`) |
| `Precio Medio del Catálogo` | `AVERAGE(products[price])` | Calcula la media aritmética del precio estándar de venta. | Currency (`€ #,##0.00`) |
| `Descuento Medio` | `AVERAGE(products[discount_price])` | Calcula el promedio del precio de descuento considerando únicamente filas con valores no nulos en `discount_price`. | Currency (`€ #,##0.00`) |
| `% Productos Destacados` | `DIVIDE(CALCULATE(COUNTROWS(products), products[En_Promocion] = "Sí"), COUNTROWS(products))` | Determina el porcentaje de penetración de productos promocionados sobre el catálogo filtrado. Evita errores de división por cero mediante `DIVIDE`. | Percentage (`0.00%`) |

---

### 2.5 🎨 Capa de Presentación (UI/UX)
El informe consta de **2 páginas de reporte** diseñadas bajo un estándar visual corporativo de alto contraste y accesibilidad.

#### Página 1: Visión General del Catálogo
- **Header KPI Cards**:
  - *Total Productos*: Conteo global de unidades de catálogo (Acento azul `#253494`).
  - *Precio Medio*: Valor medio global del catálogo (Acento azul `#0072B2`).
  - *Nº Categorías*: Recuento de valores únicos en `category` (Acento verde `#01665E`).
- **Gráfico de Barras Horizontales**: Muestra el Top 10 de categorías con mayor precio promedio (`Precio Medio del Catálogo`).
- **Gráfico de Anillos (Donut Chart)**: Distribución del volumen de productos por Top 5 categorías con etiquetas combinadas (*Categoría + % del total*).
- **Filtros Interactivos (Slicers)**: Slicer de categoría y filtro promocional (`En_Promocion`).

#### Página 2: Detalle y Galería de Productos
- **Tabla Interactiva con Renderizado de Imágenes**: Tabla detallada con `main_image_url` visualizado como imagen, `title`, `subtitle`, `category`, `price` y `discount_price`.
- **Filtros cruzados**: Interacción sincrónica entre selecciones de lista y métricas clave.

---

## 3. APIs y Endpoints

Al ser un proyecto nativo de Power BI Desktop (`.pbix` / `.pbit`), la ejecución local no expone endpoints HTTP directamente. Sin embargo, al desplegarse en **Power BI Service (SaaS)** o al integrarse mediante **Power BI Embedded**, se habilitan los siguientes endpoints de la REST API de Power BI (versión `v1.0`):

| Método | Ruta API / Endpoint | Descripción | Parámetros / Headers |
| :--- | :--- | :--- | :--- |
| `POST` | `https://api.powerbi.com/v1.0/myorg/datasets/{datasetId}/refreshes` | Desencadena una actualización programada o a petición del dataset. | **Headers**: `Authorization: Bearer <Token>`<br>**Path**: `datasetId`<br>**Body**: `{ "notifyOption": "MailOnFailure" }` |
| `GET` | `https://api.powerbi.com/v1.0/myorg/reports/{reportId}` | Obtiene los metadatos del informe (URL de inserción, nombre, webUrl). | **Headers**: `Authorization: Bearer <Token>`<br>**Path**: `reportId` |
| `POST` | `https://api.powerbi.com/v1.0/myorg/groups/{groupId}/reports/{reportId}/GenerateToken` | Genera un Embed Token para integrar el dashboard en aplicaciones externas (Power BI Embedded). | **Headers**: `Authorization: Bearer <Token>`<br>**Body**: `{ "accessLevel": "View" }` |
| `GET` | `https://api.powerbi.com/v1.0/myorg/datasets/{datasetId}/refreshes` | Consulta el historial y estado de los refrescos ejecutados en el dataset. | **Headers**: `Authorization: Bearer <Token>`<br>**Path**: `datasetId` |

---

## 4. Variables de Entorno y Configuración

Configuración técnica requerida para la compilación, actualización de datos y despliegue del modelo:

| Variable / Parámetro | Valor de Ejemplo | Obligatoria | Descripción |
| :--- | :--- | :--- | :--- |
| `Ruta_Origen_CSV` | `C:\Datos\Mercadona\products_macro.csv` | Sí (en `.pbit`) | Parámetro de origen de archivo local o de red para Power Query. |
| `File_Encoding` | `65001` (UTF-8) | Sí | Codificación de caracteres durante la lectura de la fuente en M. |
| `Csv_Delimiter` | `,` | Sí | Separador del archivo de texto origen. |
| `Decimal_Separator_Source` | `.` | Sí | Separador decimal utilizado en el archivo crudo de entrada. |
| `Decimal_Separator_Target` | `,` | Sí | Separador decimal para entornos regionales en español. |
| `PowerBI_Gateway_Name` | `Enterprise_Gateway_Mercadona` | Opcional | Nombre del Gateway On-Premises para actualización automática en Power BI Service. |
| `Workspace_ID` | `a1b2c3d4-e5f6-7890-abcd-ef1234567890` | Opcional | GUID del área de trabajo destino en Power BI Service. |

---

## 5. Guía de Despliegue Paso a Paso

### 5.1 Requisitos Previos
1. Sistema Operativo Microsoft Windows 10/11 (64-bit).
2. **Power BI Desktop** (versión actualizada recomendada).
3. Conexión de red o acceso local al archivo fuente `products_macro.csv`.

---

### 5.2 Despliegue Local (Entorno de Desarrollo)

#### Opción A: Mediante archivo `.pbix` (Modelo con datos embebidos)
1. Clonar el repositorio localmente:
   ```bash
   git clone https://github.com/migueljerico/powerbi-dashboard-mercadona.git
   ```
2. Navegar a la carpeta raíz del proyecto.
3. Hacer doble clic sobre el archivo `Ejercicio_3.8_...pbix`.
4. El cuadro de mando se abrirá completamente operativo con los datos precargados.

#### Opción B: Mediante plantilla `.pbit` (Reutilizable)
1. Abrir el archivo `Ejercicio_3.8_...pbit` en Power BI Desktop.
2. Introducir la ruta absoluta del archivo `products_macro.csv` cuando el cuadro de diálogo de parámetros lo solicite.
3. Presionar **Cargar**. Power Query ejecutará las transformaciones M automáticas y generará el modelo in-memory.

---

### 5.3 Despliegue en la Nube (Power BI Service)

1. **Autenticación**: Iniciar sesión en Power BI Desktop con la cuenta corporativa o Pro/Premium.
2. **Publicación**:
   - En la barra superior, seleccionar **Inicio** → **Publicar**.
   - Seleccionar el **Área de Trabajo** (Workspace) de destino.
   - Confirmar la subida del archivo.
3. **Configuración de la Fuente de Datos en Service**:
   - Acceder a [app.powerbi.com](https://app.powerbi.com).
   - Ir al Workspace y seleccionar el Dataset publicado.
   - Entrar a **Configuración** → **Credenciales de la fuente de datos**.
   - Configurar la autenticación (para archivos locales se requiere **On-premises Data Gateway**).
4. **Configuración del Refresco Programado**:
   - En la pestaña **Actualización programada**, activar la opción.
   - Definir la frecuencia (diaria/semanal) y las franjas horarias deseadas.

---

### 5.4 Automatización de Refresco vía API (Power Automate / CLI)

Ejemplo de automatización mediante script Bash y la Azure CLI para desencadenar el refresco del dataset publicado:

```bash
#!/usr/bin/env bash

# 1. Obtener Token de acceso de Azure AD
TOKEN=$(az account get-access-token \
  --resource https://analysis.windows.net/powerbi/api \
  --query accessToken -o tsv)

# 2. Variables del entorno
WORKSPACE_ID="TU_WORKSPACE_GUID"
DATASET_ID="TU_DATASET_GUID"

# 3. Disparar Refresco Programático
curl -X POST \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"notifyOption": "MailOnFailure"}' \
  "https://api.powerbi.com/v1.0/myorg/groups/${WORKSPACE_ID}/datasets/${DATASET_ID}/refreshes"
```

---

## 6. Limitaciones Conocidas y Posibles Mejoras Futuras

### 6.1 Limitaciones Conocidas
1. **Ausencia de Modelo Estrellas (Star Schema)**: Al ser un dataset estático basado en una única tabla flat (`products`), no existe separación en tablas de dimensión (`Dim_Categoria`, `Dim_Producto`) y tablas de hechos (`Fact_Precios`).
2. **Carencia de Dimensión Temporal (`Dim_Date`)**: Imposibilita el análisis de variaciones históricas de precio, tendencias de la oferta o comparativas interanuales (YoY).
3. **Dependencia de Rutas Absolutas en ETL**: La actualización de datos desde `.pbit` exige redefinir manualmente la ruta local si el archivo cambia de ubicación.
4. **Sin Seguridad a Nivel de Fila (RLS)**: No se restringe la visibilidad de categorías por roles o departamentos dentro de la organización.
5. **Carga de Imágenes desde URLs Externas**: El renderizado dinámico de `main_image_url` depende totalmente de la disponibilidad y tiempo de respuesta de los servidores de origen de las imágenes de Mercadona.

---

### 6.2 Posibles Mejoras Futuras

1. **Evolución a Star Schema**:
   - Extraer la dimensión de categorías a una tabla `Dim_Category` normalizada.
   - Generar una tabla de fechas `Dim_Date` para soporte de Time Intelligence en DAX.
2. **Ingesta Automatizada mediante Web Scraping**:
   - Desarrollar un pipeline en Python (Scrapy/Selenium) que capture diariamente el catálogo y persista los datos en un Data Lake o SQL Database (PostgreSQL/Azure SQL).
3. **Implementación de Row-Level Security (RLS)**:
   - Crear roles como `Gestor_Alimentacion`, `Gestor_Perfumeria` para restringir datos según la categoría del usuario.
4. **Integración con Power Automate / Dataflows**:
   - Migrar la ETL M desde Power BI Desktop hacia **Power BI Dataflows** para reutilización centralizada en la organización.
   - Configurar alertas automáticas vía Power Automate cuando el `% Productos Destacados` caiga por debajo de un umbral estratégico.