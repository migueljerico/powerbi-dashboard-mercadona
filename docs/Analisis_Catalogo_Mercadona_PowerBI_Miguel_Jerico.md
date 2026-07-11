# Documentación Técnica: Cuadro de Mando del Catálogo de Mercadona en Power BI

## 📋 Resumen
Este proyecto documenta la creación de un cuadro de mando (*Dashboard*) interactivo en Power BI diseñado para analizar de manera visual y estructurada el catálogo de productos de Mercadona. Desarrollada por Miguel Jericó, esta herramienta analítica permite a la dirección monitorizar la distribución de precios, el peso de las categorías de productos y el impacto de la estrategia de "Productos Destacados" o promociones.

## 🔑 Puntos clave
* **Fase ETL Robusta:** Limpieza y transformación de datos de origen anglosajón (`products_macro.csv`) adaptándolos a la configuración regional española.
* **Modelo Semántico Eficiente:** Optimización del peso del modelo mediante reducción de dimensionalidad y categorización de recursos multimedia para visualización dinámica.
* **Métricas DAX Personalizadas:** Desarrollo de medidas clave para analizar precios medios, descuentos y penetración de promociones.
* **Diseño Orientado a UI/UX:** Interfaz interactiva dividida en dos páginas con una paleta corporativa de alto contraste y enfoque en la accesibilidad visual.

## 📝 Detalle

### 🛠️ 1. Fase ETL: Limpieza y Transformación (Power Query)
El conjunto de datos original (`products_macro.csv`) requirió un proceso de depuración en Power Query para garantizar la precisión de los cálculos y su correcta interpretación en España:

1. **Desactivación de conversión automática:** Se eliminó el paso automático de "Tipo cambiado" generado por Power BI para evitar que los decimales con formato anglosajón se interpretaran incorrectamente.
2. **Estandarización regional:** Se seleccionaron las columnas `price` y `discount_price`, sustituyendo los puntos (.) por comas (,) para convertirlos correctamente a formato de tipo *Número decimal*.
3. **Generación de variable de negocio (Lenguaje M):** Se identificó que la columna `discount_price` funcionaba como una etiqueta de visibilidad web más que como un descuento transaccional tradicional. Para segmentar esto de manera analítica, se creó la columna condicional `En_Promocion` usando la siguiente lógica en M:
   ```powerquery
   Table.AddColumn(#"Tipo cambiado", "En_Promocion", each if [discount_price] <> null then "Sí" else "No")
   ```
4. **Reducción de dimensionalidad (Optimización):** Se eliminó la columna `secondary_image_url` por no aportar valor analítico, optimizando el peso final del archivo. *(Nota técnica: Aunque la documentación inicial del PDF contemplaba la eliminación de la columna `subtitle`, en la estructura técnica final del modelo se optó por conservar `subtitle` y remover únicamente `secondary_image_url`)*.
5. **Configuración multimedia:** Fuera de Power Query, se configuró la columna `main_image_url` bajo la categoría de datos "URL de la imagen" para permitir que las imágenes de los productos se rendericen de forma dinámica en los reportes.

---

### 🧠 2. Fórmulas y Medidas DAX
Se implementaron medidas DAX específicas para alimentar los KPIs clave del cuadro de mando:

* **Precio Medio del Catálogo:**
  Calcula el precio promedio de todos los productos del catálogo.
  ```dax
  Precio Medio del Catálogo = AVERAGE(products[price])
  ```

* **Descuento Medio:**
  Establece el precio promedio de los productos identificados con precio promocional.
  ```dax
  Descuento Medio = AVERAGE(products_macro[discount_price])
  ```

* **Porcentaje de Productos Destacados / Promoción:**
  Determina la proporción de productos destacados frente a la oferta total.
  ```dax
  % Productos Destacados = 
  DIVIDE(
      CALCULATE(COUNTROWS(products), products[En_Promocion] = "Sí"), 
      COUNTROWS(products)
  )
  ```

---

### 🎨 3. Diseño del Dashboard y UI/UX
El cuadro de mando utiliza una paleta de colores corporativa de alta accesibilidad y contraste, dividiéndose en dos pantallas con objetivos analíticos diferenciados:

#### Página 1: Visión General del Catálogo
* **Objetivo:** Ofrecer una radiografía financiera y estructural de la oferta comercial.
* **KPIs Superiores (Tarjetas de alto impacto con bordes de acento de color):**
  * **Total de Productos:** Recuento de `id` (Acento azul oscuro `#253494`).
  * **Precio Medio del Catálogo:** Medida DAX (Acento azul `#0072B2`).
  * **Nº de Categorías:** Recuento único de categorías (Acento verde azulado `#01665E`).
* **Gráfico de Barras Horizontales:** Top 10 de categorías más caras según su precio medio (barras en color `#0072B2`).
* **Gráfico de Anillos (Distribución de Volumen):** Muestra el peso porcentual de las top 5 categorías sobre el total del catálogo. Para garantizar la accesibilidad y no depender únicamente de la distinción cromática, se configuraron etiquetas de datos detalladas que muestran la combinación de *categoría + porcentaje* con una paleta de 5 colores de alto contraste sobre fondo blanco.

#### Página 2: Detalle y Análisis Específico
* **Objetivo:** Permitir una exploración granular y la búsqueda directa de productos específicos.
* **Elementos interactivos:** Incluye una tabla interactiva que renderiza dinámicamente las imágenes de los productos utilizando las URLs configuradas, junto con gráficos de tendencias y filtros intuitivos que facilitan la navegación del usuario.

## ✅ Conclusiones / siguientes pasos
1. **Despliegue local:** Descargar el archivo `.pbix` o la plantilla optimizada `.pbit` del repositorio y abrirlo mediante Power BI Desktop.
2. **Actualización de fuentes:** Para trabajar con datos actualizados, se debe modificar la ruta del archivo de origen (`products_macro.csv`) dentro de los parámetros de Power Query.
3. **Publicación en la nube:** Publicar el informe en Power BI Service para habilitar el consumo compartido en la organización o su incorporación en un portafolio web personal.