# 📊 Power BI Dashboard - Mercadona

![Power BI](https://img.shields.io/badge/Power%20BI-F2C811?style=for-the-badge&logo=powerbi&logoColor=black) ![DAX](https://img.shields.io/badge/DAX-0078D4?style=for-the-badge) ![Estado](https://img.shields.io/badge/Estado-Publicado-green?style=for-the-badge) ![Licencia](https://img.shields.io/badge/Licencia-MIT-blue?style=for-the-badge)

*Análisis inteligente de datos comerciales y operativos enfocado en la cadena de suministros y ventas de Mercadona.*

## 🔗 Acceso / Demo
Actualmente el proyecto se distribuye como archivo de reporte `.pbix`. Para visualizarlo, es necesario contar con Power BI Desktop instalado en el equipo.

## 📋 Descripción
Este proyecto consiste en el desarrollo de un tablero de control (dashboard) profesional utilizando **Microsoft Power BI**, diseñado para analizar el rendimiento comercial de Mercadona. El objetivo principal es transformar datos brutos en información accionable que permita optimizar la toma de decisiones estratégicas.

El dashboard resuelve la necesidad de monitorizar KPIs críticos en tiempo real, permitiendo a los gestores identificar tendencias de ventas, analizar la rotación de productos y evaluar el desempeño por categorías o regiones geográficas, eliminando la dependencia de reportes estáticos en Excel.

## ✨ Funcionalidades

| Funcionalidad | Descripción |
| :--- | :--- |
| **Análisis de Ventas** | Visualización de ingresos totales, margen de beneficio y volumen de ventas. |
| **Segmentación de Productos** | Filtros dinámicos por categoría, marca y tipo de producto. |
| **KPIs de Rendimiento** | Indicadores clave como el Ticket Promedio y Tasa de Crecimiento Mensual. |
| **Análisis Temporal** | Comparativas interanuales y mensuales mediante slicers de fecha. |
| **Mapas de Calor** | Distribución geográfica de ventas y densidad de clientes. |

## ⚙️ Instalación

1. **Requisitos Previos**:
   - Instalar [Power BI Desktop](https://powerbi.microsoft.com/desktop/).

2. **Clonado del Repositorio**:
   ```bash
   git clone https://github.com/migueljerico/powerbi-dashboard-mercadona.git
   cd powerbi-dashboard-mercadona
   ```

3. **Ejecución**:
   - Localiza el archivo `.pbix` en la raíz del proyecto.
   - Haz doble clic sobre el archivo para abrirlo en Power BI Desktop.
   - Si el origen de datos es externo, actualiza las credenciales en `Transformar datos` $\rightarrow$ `Configuración de origen de datos`.

## 🚀 Uso

Para interactuar con el dashboard, utiliza los siguientes elementos:
- **Slicers (Filtros)**: Ubicados en el panel lateral para filtrar por año, trimestre o región.
- **Cross-filtering**: Haz clic en cualquier segmento de un gráfico de tarta para filtrar automáticamente el resto de las visualizaciones del reporte.
- **Tooltips**: Pasa el cursor sobre los puntos de datos para ver el desglose detallado de la cifra.

## 📁 Estructura del proyecto
```text
. 
└── powerbi-dashboard-mercadona/
    ├── README.md                  # Documentación del proyecto
    └── Mercadona_Analysis.pbix    # Archivo maestro de Power BI
```

## 🛠️ Tecnologías

| Herramienta | Versión/Detalle | Uso en el proyecto |
| :--- | :--- | :--- |
| **Power BI Desktop** | Latest | Diseño de visualizaciones y reportes |
| **Power Query** | M Language | ETL (Extracción, Transformación y Carga) |
| **DAX** | Advanced | Creación de medidas calculadas y columnas virtuales |
| **Microsoft Excel** | .xlsx / .csv | Fuente de datos primaria |

## 📚 Contexto formativo o motivación
Este proyecto ha sido desarrollado con el propósito de aplicar técnicas avanzadas de Business Intelligence (BI), enfocándose en la limpieza de datos complejos y la implementación de un modelo de datos en estrella (Star Schema) para optimizar la performance de las consultas DAX.

<p align="center">Desarrollado por @migueljerico · 2026</p>