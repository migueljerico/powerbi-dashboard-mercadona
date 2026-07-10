# 📘 Manual Técnico - Power BI Dashboard Mercadona

## 1. Arquitectura General
El flujo de datos sigue un proceso lineal de ETL hasta la capa de visualización:

`Origen de Datos (Excel/CSV)` $\rightarrow$ `Power Query (Limpieza/Transformación)` $\rightarrow$ `Modelo de Datos (Relaciones/DAX)` $\rightarrow$ `Capa de Presentación (Visuals)`

### Diagrama de Flujo
```text
  [Data Source] 
         ↓
  [Power Query] ──→ (Eliminación de nulos, Pivotado, Tipado de datos)
         ↓
  [Data Model]  ──→ (Tabla Hechos $\leftrightarrow$ Tablas Dimensiones)
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

### 📊 Modelo de Datos
Se implementó un **Esquema en Estrella (Star Schema)**:
- **Tabla de Hechos**: `Ventas` (Contiene transacciones, cantidades y precios).
- **Dimensiones**: `Productos`, `Calendario`, `Tiendas`, `Categorías`.
- **Relaciones**: Relaciones de uno a muchos (1:*) desde las dimensiones hacia la tabla de hechos.

### 🧮 Capa de Cálculo (DAX)
Medidas clave implementadas:
- `Total Ventas = SUM(Ventas[Monto])`
- `Margen % = DIVIDE([Beneficio Total], [Total Ventas], 0)`
- `Crecimiento MoM = CALCULATE([Total Ventas], DATEADD('Calendario'[Fecha], -1, MONTH))`

## 3. Variables de Entorno y Configuración
Dado que es un archivo `.pbix`, no utiliza variables de entorno tradicionales, pero requiere la siguiente configuración de acceso:

| Parámetro | Valor Sugerido | Obligatorio | Descripción |
| :--- | :--- | :--- | :--- |
| **Ruta Datos** | `C:\Datos\Mercadona\` | Sí | Directorio local donde residen los archivos fuente |
| **Permisos** | Lectura | Sí | Acceso al sistema de archivos para actualización de datos |

## 4. Guía de Despliegue

1. **Publicación en la Nube**:
   - Abrir el archivo en Power BI Desktop.
   - Hacer clic en el botón `Publicar` $\rightarrow$ Seleccionar `Mi área de trabajo` o un Workspace compartido.

2. **Configuración de Gateway**:
   - Para mantener los datos actualizados automáticamente, instalar el *On-premises data gateway* en el servidor donde residen los archivos fuente.

3. **Acceso Web**:
   - Generar un enlace de compartir o embeber el reporte en un portal de SharePoint/Teams.

## 5. Limitaciones y Mejoras

### Limitaciones
- **Datos Estáticos**: El reporte depende de archivos locales; no está conectado a una base de datos SQL en tiempo real.
- **Volumen**: El rendimiento puede degradarse si el archivo Excel supera el millón de filas.

### Mejoras Futuras
- Migración de la fuente de datos a **Azure SQL Database**.
- Implementación de **RLS (Row Level Security)** para que cada gerente de tienda solo vea sus propios datos.
- Integración de predicciones de ventas mediante **AI Visuals** (Forecasting).