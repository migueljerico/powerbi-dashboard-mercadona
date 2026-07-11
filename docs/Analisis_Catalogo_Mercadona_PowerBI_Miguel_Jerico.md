# Análisis de Catálogo y Estrategia de Precios — Mercadona (Power BI)

## 📋 Resumen
Este proyecto documenta la construcción de un cuadro de mando interactivo en Power BI para analizar el catálogo de productos de Mercadona. La solución cubre el ciclo completo: ingesta y limpieza de datos con Power Query, modelado y medidas DAX, y diseño de dos páginas de dashboard orientadas a la monitorización de precios, categorías y la ejecución de la estrategia de "Productos Destacados". El objetivo es dotar a la dirección de una herramienta visual para la toma de decisiones basada en datos.

## 🔑 Puntos clave
- **Fuente de datos**: `products_macro.csv` con formato anglosajón, requiriendo estandarización regional (punto → coma) y desactivación de la conversión automática de tipos.
- **ETL en Power Query**: limpieza, creación de columna de negocio `En_Promocion` (basada en `discount_price` como etiqueta de visibilidad web, no descuento real), eliminación de columnas sin valor analítico (`subtitle`, `secondary_image_url`) y configuración de `main_image_url` como URL de imagen para visualización dinámica.
- **Medidas DAX**: `Precio Medio del Catálogo` (AVERAGE) y `% Productos Destacados` (DIVIDE + CALCULATE).
- **Dashboard de dos páginas**:  
  1. **Visión General del Catálogo** — KPIs globales, Top 10 categorías por precio medio y distribución de volumen (Top 5 categorías) en gráfico de anillos con etiquetas accesibles.  
  2. **Análisis de Promociones** — Foco en la estrategia de "Productos Destacados" (detalle truncado en el PDF).
- **UI/UX**: paleta corporativa de alto contraste, bordes de acento en KPIs, etiquetas de datos activadas para no depender solo del color, y accesibilidad visual.

## 📝 Detalle

### 1. Fase ETL: Limpieza y transformación (Power Query)
| Paso | Acción | Detalle técnico |
|------|--------|-----------------|
| 1 | Desactivar conversión automática | Eliminado el paso "Tipo cambiado" generado por Power BI para evitar malinterpretación de números con formato anglosajón. |
| 2 | Estandarización regional | Columnas `price` y `discount_price`: sustitución de `.` por `,` y conversión a **Número decimal**. |
| 3 | Variable de negocio `En_Promocion` | Columna condicional en lenguaje M: `if [discount_price] <> null then "Sí" else "No"`. Nota: `discount_price` actúa como etiqueta de visibilidad web, no como descuento económico. |
| 4 | Reducción de dimensionalidad | Eliminadas `subtitle` y `secondary_image_url` para optimizar el modelo. |
| 5 | Configuración multimedia | Fuera de Power Query, `main_image_url` categorizada como **URL de imagen** para visualización dinámica del catálogo. |

### 2. Medidas DAX
```dax
-- Precio Medio del Catálogo
Precio Medio del Catálogo = AVERAGE(products[price])

-- % Productos Destacados
% Productos Destacados =
DIVIDE(
    CALCULATE(COUNTROWS(products), products[En_Promocion] = "Sí"),
    COUNTROWS(products)
)
```

### 3. Diseño del Dashboard y UI/UX

#### Página 1 — Visión General del Catálogo
**Objetivo**: Radiografía financiera y estructural de la oferta.

| Elemento | Configuración |
|----------|---------------|
| **KPIs superiores** | • **Total de Productos** — `COUNTROWS(products)` / `DISTINCTCOUNT(products[id])` — Borde `#253494`<br>• **Precio Medio del Catálogo** — Medida DAX — Borde `#0072B2`<br>• **Nº de Categorías** — `DISTINCTCOUNT(products[Category])` — Borde `#01665E` |
| **Gráfico de barras horizontales** | Top 10 categorías más caras — Eje Y: `Category`, Eje X: `AVERAGE(price)` — Color `#0072B2` |
| **Gráfico de anillos** | Distribución del volumen (Top 5 categorías) — Etiquetas: *categoría + porcentaje* — Paleta de 5 colores alto contraste sobre fondo blanco — Accesibilidad: no dependencia exclusiva del color. |

#### Página 2 — Análisis de Promociones
**Objetivo**: Monitorizar la ejecución de la estrategia de "Productos Destacados".  
*(El contenido del PDF se trunca antes de detallar KPIs y visuales de esta página.)*

### 4. Paleta de colores y accesibilidad
- Colores corporativos de alto contraste: `#253494`, `#0072B2`, `#01665E`, más 5 tonos para el gráfico de anillos.
- Bordes de acento en tarjetas KPI para jerarquía visual.
- Etiquetas de datos activadas en gráficos para interpretación sin depender solo de la distinción cromática.

## ✅ Conclusiones / siguientes pasos
- **Completar la documentación de la Página 2** (KPIs, visuales, segmentadores) una vez finalizado el desarrollo.
- **Validar la medida `% Productos Destacados`** contra la definición de negocio: confirmar que `discount_price <> null` equivale realmente a "Producto Destacado" y no a otra lógica de visibilidad.
- **Publicar en GitHub** el archivo `.pbix` (o versión sanitizada) junto con este README y, opcionalmente, el PDF original como referencia.
- **Considerar mejoras**:  
  - Añadir segmentadores de fecha/categoría si la fuente incorpora temporalidad.  
  - Implementar *drill-through* desde la categoría al detalle de producto con imagen (`main_image_url`).  
  - Revisar rendimiento del modelo si el catálogo crece significativamente (actualmente optimizado por reducción de columnas).