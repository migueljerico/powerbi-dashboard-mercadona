# 📘 Manual Técnico - Power BI Dashboard Mercadona

## 1. Arquitectura General

El proyecto implementa una arquitectura de Business Intelligence (BI) orientada a capas, optimizada para el motor analítico columnar en memoria **VertiPaq** de Microsoft Power BI. La arquitectura garantiza una estricta separación de responsabilidades entre la capa de datos de origen, el procesamiento ETL en Power Query (Lenguaje M), el modelado semántico, la lógica analítica centralizada en DAX y la interfaz de visualización interactiva.

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
└── Capa de datos (CSV UTF-8: products_macro.csv)
    └── Capa de ETL (Power Query / Lenguaje M Pipeline)
        └── Capa de modelo (VertiPaq In-Memory Engine & Categorización Metadata)
            └── Capa de lógica (Medidas DAX & Tabla Técnica _Measures)
                └── Capa de presentación (Dashboard 2 Páginas / UX Cross-filtering / Service REST API)
```

---

## 2. Descripción de Módulos y Componentes

### 2.1 📥 Capa de Datos (Origen)
- **Archivo/Fichero**: `products_macro.csv`
- **Ubicación**: Raíz del repositorio / Configurable mediante parámetro `Ruta_Origen_CSV`.
- **Formato**: CSV en codificación UTF-8 (`65001`).
- **Delimitador de campos**: Coma (`,`).
- **Formato numérico de origen**: Anglosajón (punto `.` como separador decimal).
- **Responsabilidad**: Proporcionar la información primaria plana del catálogo de productos de Mercadona.
- **Campos expuestos**:
  - `id` (Identificador único del producto)
  - `title` (Nombre comercial del producto)
  - `subtitle` (Descripción secundaria o formato)
  - `category` (Categoría o departamento comercial)
  - `price` (Precio de venta estándar en formato texto/anglosajón)
  - `discount_price` (Precio de venta rebajado o en promoción)
  - `main_image_url` (Enlace público HTTP a la imagen principal del producto)
  - `secondary_image_url` (Enlace a imagen secundaria)

---

### 2.2 🔄 Capa de ETL (Power Query - Lenguaje M)
- **Componente**: Consulta principal `products` en el editor de Power Query.
- **Responsabilidad**: Extracción del archivo crudo, limpieza de datos, estandarización de delimitadores numéricos para entornos de habla hispana, enriquecimiento condicional de negocio y eliminación de columnas innecesarias para optimizar la compresión VertiPaq.

#### Pipeline de Pasos Aplicados en M:

| Paso M | Operación / Función M | Descripción Técnica |
| :--- | :--- | :--- |
| 1 | `Csv.Document(File.Contents(...))` | Lee el archivo plano asegurando la codificación UTF-8 (`65001`) y delimitador por comas. |
| 2 | `Table.PromoteHeaders` | Eleva la primera fila del dataset a nombres formales de columna. |
| 3 | **Eliminación del paso automático `Changed Type`** | **Paso Crítico**: Evita que Power BI convierta automáticamente decimales con punto (`.`) a enteros o nulos bajo entornos con configuración regional regional de coma (`,`). |
| 4 | `Table.ReplaceValue` (`price`, `discount_price`) | Reemplaza cadenas con separador `.` por `,` mediante `Text.Replace` sobre las columnas de precio. |
| 5 | `Table.TransformColumnTypes` (Tipado Decimal) | Convierte la columna `price` a `Decimal.Type` y `discount_price` a `Nullable Decimal.Type`. |
| 6 | `Table.AddColumn` (`En_Promocion`) | Agrega la columna lógica de negocio mediante expresión: `if [discount_price] <> null then "Sí" else "No"`. |
| 7 | `Table.RemoveColumns` | Elimina la columna `secondary_image_url` para reducir espacio en el diccionario del motor VertiPaq. *(Se preserva `subtitle` para contextualizar tooltips)*. |
| 8 | `Table.TransformColumnTypes` (Tipado Definitivo) | Fija tipos de datos definitivos: `id` (`Int64.Type`), `category` (`Text.Type`), `title` (`Text.Type`), `subtitle` (`Text.Type`), `main_image_url` (`Text.Type`). |

---

### 2.3 🗃️ Capa de Modelo Semántico (VertiPaq In-Memory)
- **Estructura del modelo**: Modelo de tabla única (`products`) acoplado a una tabla contenedora técnica (`_Measures`).
- **Configuración de Metadatos y Categorías de Datos**:
  - `products[main_image_url]`: Categoría de datos asignada explícitamente a **URL de la imagen** (*Image URL*), permitiendo al motor de renderizado HTML de Power BI mostrar las imágenes dinámicamente en tablas y matrices.
  - `products[En_Promocion]`: Campo categórico de baja cardinalidad ("Sí" / "No") indexado para un filtrado de alta velocidad en segmentadores.
- **Tabla Técnica de Medidas**: `_Measures` (tabla de soporte creada sin columnas de datos para agrupar y organizar las métricas DAX).

---

### 2.4 🧠 Capa de Lógica de Negocio (Medidas DAX)
Las fórmulas de cálculo están alojadas dentro de la tabla `_Measures`.

| Nombre de Medida | Fórmula DAX | Descripción y Propósito | Formato / Display |
| :--- | :--- | :--- | :--- |
| `Total Productos` | `COUNTROWS(products)` | Cuenta la cantidad total de artículos según el contexto de filtro activo. | Número entero (`#,0`) |
| `Precio Medio del Catálogo` | `AVERAGE(products[price])` | Promedio aritmético del precio estándar de catálogo. | Moneda (`€ #,##0.00`) |
| `Descuento Medio` | `AVERAGE(products[discount_price])` | Promedio del precio promocional evaluado exclusivamente sobre productos con descuento no nulo. | Moneda (`€ #,##0.00`) |
| `% Productos Destacados` | `DIVIDE(CALCULATE(COUNTROWS(products), products[En_Promocion] = "Sí"), COUNTROWS(products))` | Proporción de productos en promoción respecto al total del catálogo filtrado. Utiliza `DIVIDE` para prevenir divisiones entre cero. | Porcentaje (`0.00%`) |

---

### 2.5 🎨 Capa de Presentación (UI/UX)
El cuadro de mando contiene **2 páginas de informe** diseñadas bajo un enfoque de accesibilidad visual, alto contraste cromático y respuesta interactiva sincrónica.

#### Documentación de Componentes de Reporte:

* **Página 1: Visión General del Catálogo**
  - **KPI Cards (Encabezado)**:
    - *Total Productos*: Conteo global de referencias (`#253494`).
    - *Precio Medio del Catálogo*: Medida de precio promedio (`#0072B2`).
    - *Nº Categorías*: Distinto recuento de categorías (`#01665E`).
  - **Gráfico de Barras Horizontales**: Ranking Top 10 de categorías con mayor precio medio.
  - **Gráfico de Anillos (Donut Chart)**: Distribución del Top 5 de categorías por volumen de oferta, configurado con etiquetas compuestas (*Categoría + % del Total*).
  - **Slicers Interactivos**: Filtro desplegable por `category` y filtro de botones para `En_Promocion` ("Sí" / "No").

* **Página 2: Detalle y Galería de Productos**
  - **Tabla de Detalle Granular**: Muestra el catálogo fila a fila incluyendo la renderización visual de `main_image_url`, `title`, `subtitle`, `category`, `price` y `discount_price`.
  - **Filtros Cruzados**: Cross-filtering habilitado entre elementos visuales y métricas de encabezado.

---

### 2.6 📁 Estructura de Ficheros del Repositorio

| Archivo / Ruta | Responsabilidad / Descripción |
| :--- | :--- |
| `README.md` | Documentación general del proyecto, características, capturas y guía rápida de uso. |
| `MANUAL_TECNICO.md` | Especificación técnica detallada de la arquitectura, pipeline ETL, modelo semántico y despliegues. |
| `LICENSE` | Términos de licitamiento del software bajo la licencia libre **GNU General Public License v3.0 (GPL-3.0)**. |
| `docs/Analisis_Catalogo_Mercadona_PowerBI_Miguel_Jerico.md` | Informe técnico extendido sobre decisiones de diseño, modelo DAX y validación ETL. |
| `*.pbix` / `*.pbit` | Archivos ejecutables de Power BI (Informe con datos y Plantilla reutilizable). |

---

## 3. APIs y Endpoints

Aunque Power BI Desktop opera como un ejecutable cliente sin expicación directa de HTTP listeners, la integración del informe en **Power BI Service (SaaS)** y **Power BI Embedded** pone a disposición los siguientes endpoints mediante la REST API oficial de Power BI (`v1.0`):

| Método | Ruta API / Endpoint | Descripción | Parámetros / Headers |
| :--- | :--- | :--- | :--- |
| `POST` | `https://api.powerbi.com/v1.0/myorg/datasets/{datasetId}/refreshes` | Ejecuta un refresco bajo demanda del dataset cargado en el workspace. | **Headers**: `Authorization: Bearer <Token>`<br>**Path**: `datasetId`<br>**Body**: `{ "notifyOption": "MailOnFailure" }` |
| `GET` | `https://api.powerbi.com/v1.0/myorg/reports/{reportId}` | Devuelve la información de metadatos del informe y su URL de inserción (Embed URL). | **Headers**: `Authorization: Bearer <Token>`<br>**Path**: `reportId` |
| `POST` | `https://api.powerbi.com/v1.0/myorg/groups/{groupId}/reports/{reportId}/GenerateToken` | Genera un Embed Token con permisos de lectura para aplicaciones web externas. | **Headers**: `Authorization: Bearer <Token>`<br>**Body**: `{ "accessLevel": "View" }` |
| `GET` | `https://api.powerbi.com/v1.0/myorg/datasets/{datasetId}/refreshes` | Retorna el histórico de ejecuciones de actualización y su estado (`Completed`, `Failed`, `In Progress`). | **Headers**: `Authorization: Bearer <Token>`<br>**Path**: `datasetId` |

---

## 4. Variables de Entorno y Configuración

Parámetros técnicos y variables de entorno necesarias para la compilación, parametrización de fuentes y publicación del modelo:

| Variable / Parámetro | Valor de Ejemplo | Obligatoria | Descripción |
| :--- | :--- | :--- | :--- |
| `Ruta_Origen_CSV` | `C:\Datos\Mercadona\products_macro.csv` | Sí (en `.pbit`) | Parámetro de Power Query que especifica la ruta absoluta del dataset plano de origen. |
| `File_Encoding` | `65001` (UTF-8) | Sí | Código de página de caracteres utilizado por la función `Csv.Document`. |
| `Csv_Delimiter` | `,` | Sí | Carácter delimitador de columnas utilizado en la lectura de la fuente. |
| `Decimal_Separator_Source` | `.` | Sí | Separador decimal utilizado originalmente en el archivo CSV crudo. |
| `Decimal_Separator_Target` | `,` | Sí | Separador decimal de destino para compatibilidad con la configuración regional española. |
| `PowerBI_Gateway_Name` | `Enterprise_Gateway_Mercadona` | Opcional | Nombre del conector On-Premises Data Gateway para refresco automático en Power BI Service. |
| `Workspace_ID` | `a1b2c3d4-e5f6-7890-abcd-ef1234567890` | Opcional | Identificador GUID del área de trabajo destino en Power BI Service. |

---

## 5. Guía de Despliegue Paso a Paso

### 5.1 Requisitos Previos
1. Sistema Operativo Microsoft Windows 10/11 (64-bit).
2. **Power BI Desktop** (última versión disponible).
3. Clonación o descarga del repositorio en el equipo de desarrollo.

---

### 5.2 Despliegue Local (Entorno de Desarrollo)

#### Opción A: Mediante archivo de informe completo (`.pbix`)
1. Clonar el repositorio localmente:
   ```bash
   git clone https://github.com/migueljerico/powerbi-dashboard-mercadona.git
   ```
2. Acceder al directorio clonado:
   ```bash
   cd powerbi-dashboard-mercadona
   ```
3. Abrir el archivo `.pbix` con Power BI Desktop.
4. El informe cargará directamente con el modelo de datos en memoria precargado.

#### Opción B: Mediante plantilla reutilizable (`.pbit`)
1. Abrir el archivo `.pbit` desde Power BI Desktop.
2. Al desplegarse la ventana emergente de parámetros, introducir la ruta absoluta hacia el archivo `products_macro.csv` (ejemplo: `C:\Proyectos\powerbi-dashboard-mercadona\products_macro.csv`).
3. Hacer clic en **Cargar**. Power Query ejecutará las transformaciones M y generará la estructura del modelo en el motor VertiPaq local.

---

### 5.3 Despliegue en la Nube (Power BI Service)

1. **Autenticación**: Iniciar sesión en Power BI Desktop con una cuenta corporativa o educativa que disponga de licencia Power BI Pro o Premium Per User (PPU).
2. **Publicación del Informe**:
   - Ir a la pestaña **Inicio** → **Publicar**.
   - Seleccionar el **Área de trabajo** (Workspace) de destino.
   - Guardar los cambios y confirmar la subida.
3. **Configuración de la Fuente de Datos**:
   - Abrir el navegador e ingresar a [app.powerbi.com](https://app.powerbi.com).
   - Navegar hasta el Workspace y ubicar el **Dataset / Modelo semántico**.
   - Seleccionar **Configuración** → **Credenciales de la fuente de datos**.
   - En caso de usar rutas locales, vincular la fuente a un **On-premises Data Gateway** previamente instalado.
4. **Programación de Actualización Automática**:
   - Dentro de **Configuración**, desplegar la sección **Actualización programada**.
   - Activar la frecuencia deseada (Diaria o Semanal) y especificar las horas de ejecución.

---

### 5.4 Automatización del Refresco mediante Scripting (Azure CLI + Bash)

Script de automatización para desencadenar el refresco del modelo en Power BI Service mediante la REST API:

```bash
#!/usr/bin/env bash
set -euo pipefail

# 1. Obtener Token de acceso de Azure AD para Power BI Service API
TOKEN=$(az account get-access-token \
  --resource https://analysis.windows.net/powerbi/api \
  --query accessToken -o tsv)

# 2. Configurar variables de entorno del Workspace y Dataset
WORKSPACE_ID="TU_WORKSPACE_GUID"
DATASET_ID="TU_DATASET_GUID"

# 3. Disparar petición POST para iniciar el refresco del dataset
echo "Iniciando refresco programático para el Dataset ID: ${DATASET_ID}..."
curl -s -X POST \
  -H "Authorization: Bearer ${TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{"notifyOption": "MailOnFailure"}' \
  "https://api.powerbi.com/v1.0/myorg/groups/${WORKSPACE_ID}/datasets/${DATASET_ID}/refreshes"

echo "Petición de refresco enviada exitosamente."
```

---

## 6. Limitaciones Conocidas y Posibles Mejoras Futuras

### 6.1 Limitaciones Conocidas
1. **Ausencia de Modelo en Estrella (Star Schema)**: El modelo analítico está estructurado en una única tabla de datos plana (`products`), lo que restringe la separación formal de entidades en tablas de dimensión (`Dim_Category`) y tablas de hechos (`Fact_Products`).
2. **Carencia de Dimensión Temporal (`Dim_Date`)**: Al ser un dataset estático tipo foto fija (snapshot), el modelo no dispone de una tabla de fechas ni permite aplicar cálculos de inteligencia temporal DAX (`YTD`, `SAMEPERIODLASTYEAR`, variaciones porcentuales de precio en el tiempo).
3. **Ruta de Acceso Estática en Plantillas**: La reutilización del archivo `.pbit` requiere la redefinición manual de la ruta absoluta del fichero fuente en el equipo cliente.
4. **Sin Seguridad a Nivel de Fila (RLS)**: No existen roles de seguridad configurados para restringir el acceso a categorías específicas de productos según el usuario que consulte el informe.
5. **Dependencia de Red para Recursos Multimedia**: El renderizado dinámico de las imágenes en la tabla de detalles (`main_image_url`) requiere conectividad activa a los servidores externos donde se alojan las imágenes de Mercadona.

---

### 6.2 Posibles Mejoras Futuras

1. **Evolución Hacia un Esquema en Estrella (Star Schema)**:
   - Desacoplar la tabla plana `products` separando las categorías en una dimensión `Dim_Category` con claves primarias enteras (`CategoryKey`).
   - Crear una tabla de dimensiones de producto `Dim_Product` y una tabla de hechos de precios `Fact_Product_Prices`.
2. **Incorporación de Data Analytics Histórico**:
   - Implementar un proceso de almacenamiento diario que registre el histórico de precios para construir una dimensión `Dim_Date` y habilitar análisis de tendencia temporal en DAX.
3. **Ingesta Automatizada mediante Pipeline ETL (Python / Web Scraping)**:
   - Desarrollar una función serverless en Azure Functions o script Python (Scrapy/Playwright) que capture diariamente los precios desde la API pública web de Mercadona y los deposite en un Data Lake o base de datos SQL (Azure SQL / PostgreSQL).
4. **Implementación de Seguridad por Rol (Row-Level Security - RLS)**:
   - Configurar roles de usuario DAX (ej. `Rol_Frescos`, `Rol_Secos`, `Rol_Perfumeria`) para restringir la visibilidad de datos según las atribuciones del departamento.
5. **Migración a Power BI Dataflows**:
   - Mover la lógica de transformación en M desde Power BI Desktop hacia **Power BI Dataflows** en la nube, promoviendo la reutilización centralizada del modelo de catálogo en múltiples informes corporativos.