# 📊 Power BI Dashboard - Catálogo Mercadona

![Power BI](https://img.shields.io/badge/Power%20BI-F2C811?style=for-the-badge&logo=powerbi&logoColor=black)
![DAX](https://img.shields.io/badge/DAX-0078D4?style=for-the-badge)
![Power Query](https://img.shields.io/badge/Power%20Query-M_Language-8A2BE2?style=for-the-badge)
![Estado](https://img.shields.io/badge/Estado-Publicado-green?style=for-the-badge)
![Licencia](https://img.shields.io/badge/Licencia-GPL--3.0-blue?style=for-the-badge)

Análisis inteligente e interactivo del catálogo de productos de Mercadona mediante Business Intelligence para la optimización de estrategias comerciales y de precios.

---

## 📸 Vista Previa del Dashboard

![Vista Previa del Dashboard de Mercadona](Captura%20Dashboard%20Mercadona.png)

*Pantalla principal del cuadro de mando interactivo desarrollado en Power BI Desktop.*

---

## 📋 Descripción del Proyecto

Este proyecto ofrece una solución integral de **Business Intelligence** diseñada para analizar el catálogo completo de productos de Mercadona a partir de datos del archivo `products_macro.csv`.

Transforma datos brutos sin estructurar en **información clave de negocio (KPIs)** para la toma de decisiones:
* **Evolución y distribución de precios medios** por categoría.
* **Impacto y penetración de promociones** ("Productos Destacados").
* **Estructura y concentración de la oferta** comercial.

Está pensado para analistas de datos, equipos de marketing, responsables de pricing y profesionales del sector *retail*.

---

## ✨ Funcionalidades Destacadas

| Funcionalidad | Descripción |
| :--- | :--- |
| **Análisis de Precios** | Indicadores de precio medio global y precio promocional mediante medidas DAX optimizadas. |
| **Top Categorías** | Gráficos horizontales con el Top 10 de categorías con mayor precio medio y gráfico de anillos de volumen. |
| **Métricas de Promoción** | KPI de **% Productos Destacados** calculated dinámicamente con DAX. |
| **Filtros Interactivos** | Panel lateral (Slicers) para segmentar por categorías y estado promocional. |
| **Galería de Imágenes** | Detalle de productos con imágenes dinámicas renderizadas desde URL. |
| **UX y Accesibilidad** | Paleta de colores de alto contraste adaptable a estándares corporativos y etiquetas detalladas. |

---

## 📂 Contenido del Repositorio

* `Ejercicio_3.8_...pbix`: Archivo principal con el modelo de datos y panel interactivo.
* `Ejercicio_3.8_...pbit`: Plantilla reutilizable sin datos embebidos.
* `MANUAL_TECNICO.md`: Documentación técnica detallada de la arquitectura de capas, ETL y fórmulas DAX.
* `docs/`: Documentación complementaria del proyecto.

---

## ⚙️ Cómo Ejecutar el Proyecto

1. **Requisito:** Tener instalado **Power BI Desktop** (versión gratuita para Windows).
2. **Descargar / Clonar:** Clonar este repositorio en tu equipo:
   ```bash
   git clone https://github.com/migueljerico/powerbi-dashboard-mercadona.git
   ```
3. **Abrir el cuadro de mando:** Abre el archivo `.pbix` en Power BI Desktop para interactuar libremente con los datos.

---

## ✒️ Autor y Licencia

Desarrollado por **Miguel Jericó**.
Licenciado bajo la **GNU General Public License v3.0** (GPL-3.0). Consulta el archivo `LICENSE` para más detalles.