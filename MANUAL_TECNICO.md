# 📘 Manual Técnico - Power BI Dashboard Mercadona

## 1. Arquitectura General
`Origen de Datos (CSV)` → `Power Query (ETL)` → `Modelo de Datos (Relaciones)` → `Capa DAX` → `Visualización`

```text
[Datos] → [Transformación M] → [Esquema Estrella] → [Cálculos DAX] → [Dashboard]
```

## 2. Descripción de Módulos
- **Power Query:** Limpieza, tipado de datos y creación de columnas condicionales como `En_Promocion`.
- **Modelo de Datos:** Implementación de Star Schema con tabla de hechos `Ventas` y dimensiones `Productos`, `Calendario`, `Tiendas`.
- **Capa DAX:** Medidas clave (`Total Ventas`, `Margen %`, `% Productos Destacados`).

## 3. APIs y endpoints
N/A (Reporte local autónomo).

## 4. Variables de entorno
| Variable | Valor | Obligatoria | Descripción |
| :--- | :--- | :--- | :--- |
| `PATH_DATA` | `C:\Datos\Mercadona\` | Sí | Ruta local de archivos fuente |

## 5. Guía de despliegue
1. Publicar a través de Power BI Desktop → `Publicar`.
2. Configurar Gateway si el archivo reside en una ruta de red privada.

## 6. Limitaciones y mejoras
- **Limitación:** El archivo depende de una ruta de acceso local.
- **Mejora:** Migrar a Dataflows de Power BI para automatizar la ingesta sin dependencias locales.