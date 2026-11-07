# LimpIAr: Detección y Segmentación Semántica de Microbasurales con Drones

> **Caso de Estudio Técnico y Arquitectura de Producto (GovTech / Deep Learning)**  
> *Desarrollo de una solución inteligente para la fiscalización ambiental y optimización de residuos urbanos en gobiernos locales.*

---

## 🔒 Aviso de Propiedad Intelectual y Confidencialidad

El código fuente del motor de inferencia, los pesos neuronales entrenados y los datasets propietarios con anotaciones vectoriales forman parte del **secreto comercial y la propiedad intelectual de LimpIAr**.  
Este repositorio documenta la arquitectura técnica, las decisiones metodológicas de Machine Learning, la estrategia de preprocesamiento geoespacial y el marco de evaluación cuantitativa del producto.

---

## 🎯 El Problema Urbano

La proliferación de microbasurales a cielo abierto representa un grave riesgo sanitario, ambiental y logístico para las administraciones públicas locales. La detección tradicional mediante patrullaje terrestre o denuncias ciudadanas es reactiva, lenta y de cobertura limitada.

**LimpIAr** transforma este enfoque mediante inspecciones aéreas automatizadas con drones, procesando imágenes de alta resolución mediante visión por computadora para identificar focos de residuos con precisión centimétrica.

---
## 🏗️ Arquitectura del Sistema de Inferencia

[ Ortomosaico Aéreo GeoTIFF ] (Gigapíxeles, GSD 6-7 cm, CRS Métrico)
│
▼
[ Tiling & Sliding Window ] ──► Parches de 512×512 px (Stride: 256 px / Overlap: 50%)
│
▼
[ Modelo DeepLabV3+ ] ──► Backbone EfficientNetV2 + Módulos ASPP (Inferencia por Lotes)
│
▼
[ Ponderación Gaussiana ] ──► Gaussian Window Weighting (σ=0.3) para eliminar costuras
│
▼
[ Matriz Acumuladora Espacial ] ──► Umbral de Confianza Adaptativo (Threshold τ = 0.55)
│
▼
[ Vectorización Geoespacial ] ──► Rasterio / Transformada Afín a Polígonos Shapely
│
▼
[ Filtrado Morfológico ] ──► Supresión de falsos positivos por Área Mínima (m²)
│
▼
[ Capa Vectorial GeoJSON ] ──► Integración B2G en QGIS, PostGIS y Tableros Municipales
---

## 🔬 Metodología de Visión por Computadora y Deep Learning

### 1. Arquitectura de Red Neuronal: DeepLabV3+
* **Backbone:** Red preentrenada **EfficientNetV2**, optimizada para balancear precisión topológica y eficiencia computacional.
* **Módulos ASPP (*Atrous Spatial Pyramid Pooling*):** Convoluciones atrosas (*dilated convolutions*) con múltiples tasas de muestreo para extraer información contextual a diferentes campos receptivos, permitiendo segmentar residuos de morfologías muy variadas (desde pequeñas acumulaciones hasta basurales extensos).
* **Resolución de Entrada:** Tensores normalizados de `512 x 512 x 3` 

### 2. Función de Pérdida para Desbalance Crítico: Focal Loss
En imágenes aéreas urbanas, la clase "microbasural" representa menos del 1% del área total del terreno. Las funciones estándar como *Binary Cross-Entropy* colapsan prediciendo siempre fondo.  
Para resolverlo, se calibró una **Focal Loss**.

### 3. Pipeline de Inferencia sobre Ortomosaicos Masivos
* **Slicing Dinámico:** Despliegue de ventanas deslizantes con un factor de solapamiento del 50%.
* **Eliminación de Artefactos de Borde (Gaussian Window Smoothing):** Cada predicción de parche es multiplicada por una máscara gaussiana 2D antes de agregarse a la matriz global. Esto prioriza las predicciones centrales del parche (donde el contexto visual es mayor) y atenúa las inconsistencias en los bordes (*seam artifacts*).
* **Rendimiento:** El modelo optimizado alcanza un **IoU de ~0.60** y un balance de **Precision/Recall superior al 80%** sobre conjuntos de prueba de terreno no estructurado.

---

## 🗺️ Pipeline de Procesamiento Geoespacial (GIS)

1. **Preservación Geográfica:** Mapeo de índices de píxel de la matriz acumulada hacia coordenadas proyectadas reales (UTM / POSGAR) utilizando la transformada afín del archivo GeoTIFF original.
2. **Vectorización Directa a Memoria:** Extracción de geometrías vectoriales (`Shapely` / `GeoPandas`) sin necesidad de persistir máscaras intermedias pesadas en disco.
3. **Filtrado por Área Mínima:** Algoritmo de depuración espacial que descarta automáticamente detecciones con superficies menores al umbral de intervención operativa de cuadrillas (evitando movilizar recursos ante objetos aislados).
4. **Entrega de Datos:** Generación de archivos **GeoJSON** interoperables con sistemas de información geográfica municipales y visualizadores satelitales.

---

## 💼 Liderazgo de Producto y Enfoque B2G

* **Definición de Métricas con Impacto:** Traducción del rendimiento del modelo (IoU y F1-Score) a indicadores de ahorro en logística y horas-cuadrilla para secretarías de servicios públicos y ambiente.
* **Gestión de Ciclo de Vida:** Coordinación entre requerimientos de planeamiento urbano, planes de vuelo de pilotos de drones y optimización del pipeline de datos.
* **Metodología Ágil:** Priorización de backlog, sprints de validación en campo y documentación técnica para perfiles ejecutivos y técnicos.

---

## 🛠️ Stack Tecnológico y Librerías

* **Deep Learning:** `Python`, `TensorFlow`, `Keras`, `KerasCV` (DeepLabV3+, EfficientNetV2)
* **Ingeniería Geoespacial:** `Rasterio`, `GeoPandas`, `Shapely`, `QGIS`, `GeoJSON`
* **Computación Numérica & Imágenes:** `NumPy`, `OpenCV`, `SciPy`
* **Entorno de Datos:** `Jupyter Notebooks`, Google Cloud Platform / GPUs T4

---

## 👤 Autor

**Santiago Rodríguez Garay**  
*Cofounder, Product Owner & Data Scientist en LimpIAr*  
* [LinkedIn](https://www.linkedin.com/in/santiago-rodriguez-garay-9b2a3034a)
* [Email](mailto:santiagomanuelrg@gmail.com)
