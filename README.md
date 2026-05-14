# Water Quality Clustering — Aprendizaje No Supervisado

Este README documenta un proyecto de aprendizaje no supervisado que aplica algoritmos de clustering para descubrir perfiles fisicoquímicos naturales en muestras de agua, sin usar la etiqueta de potabilidad.

## Detalles del Proyecto

**Dominio:** Medio Ambiente / Calidad del agua  
**Tarea:** Clustering / Segmentación no supervisada  
**Dataset:** Water Quality Dataset — 3,276 muestras, 9 variables numéricas (sensores fisicoquímicos)  
**Grupo:** 8

## Modelos Implementados

Dos algoritmos de clustering fueron aplicados y comparados:

1. **K-means (K=2)** — Mejor silhouette score (0.0773); genera particiones geométricas interpretables del espacio fisicoquímico
2. **DBSCAN (eps=2.5, min_samples=18)** — 1 cluster principal + 37 outliers detectados; útil para identificar muestras anómalas

## Reducción de Dimensionalidad

- **PCA (lineal)** — Las dos primeras componentes capturan solo ~23.9% de la varianza (PC1=12.2%, PC2=11.7%), lo que evidencia que la información está distribuida uniformemente entre las 9 variables
- **t-SNE (no lineal)** — Muestra una nube compacta sin islas separadas, confirmando la ausencia de clusters naturales bien definidos

## Variables del Dataset

| Variable | Descripción | Tipo |
|---|---|---|
| `ph` | Nivel de pH del agua (escala 0–14) | Numérica continua |
| `Hardness` | Dureza del agua — concentración de sales de calcio y magnesio (mg/L) | Numérica continua |
| `Solids` | Total de sólidos disueltos — minerales y sales (ppm) | Numérica continua |
| `Chloramines` | Concentración de cloraminas usadas para desinfección (ppm) | Numérica continua |
| `Sulfate` | Concentración de sulfatos disueltos (mg/L) | Numérica continua |
| `Conductivity` | Conductividad eléctrica del agua — indica contenido iónico (μS/cm) | Numérica continua |
| `Organic_carbon` | Carbono orgánico total — mide materia orgánica disuelta (ppm) | Numérica continua |
| `Trihalomethanes` | Subproductos de la desinfección con cloro (μg/L) | Numérica continua |
| `Turbidity` | Turbiedad — medida de partículas en suspensión (NTU) | Numérica continua |
| `Potability` | Aptitud para consumo humano (1 = potable, 0 = no potable) — **excluida del clustering** | Categórica binaria |

## Hallazgo Principal

Los clusters descubiertos representan **perfiles fisicoquímicos** (por ejemplo, agua con alto contenido de sólidos y conductividad vs. agua con alta turbidez y dureza), no aptitud para consumo. La potabilidad se distribuye de forma casi uniforme en ambos clusters (~41% y ~38%), lo que confirma que el espacio fisicoquímico no se alinea con la etiqueta supervisada. El silhouette score bajo (~0.077) indica clusters muy solapados, coherente con un dataset que varía de forma continua sin fronteras densas bien definidas.

## Preprocesamiento

| Paso | Detalle |
|---|---|
| Eliminación de `Potability` | Etiqueta supervisada; excluida para evitar data leakage |
| Imputación con mediana | 3 variables con nulos: `ph` (14.4%), `Sulfate` (23.2%), `Trihalomethanes` (5.3%) |
| StandardScaler | Obligatorio: rangos muy distintos (Solids ~10⁴ vs. pH ~10⁰) |

## Resultados

### Modelos de Clustering

| Algoritmo | Hiperparámetros | N° Clusters | N° Outliers | Silhouette Score | Observación |
|---|---|---|---|---|---|
| K-means | K = 2 | 2 | 0 | 0.0773 | Cluster 0: 1,595 muestras (41.0% potables) — alto en sólidos, conductividad y carbono orgánico |
| K-means | K = 2 | 2 | 0 | 0.0773 | Cluster 1: 1,681 muestras (38.2% potables) — alto en turbidez, dureza y trihalometanos |
| DBSCAN | eps = 2.5, min\_samples = 18 | 1 | 37 (1.1%) | N/A | Un único cluster denso; los 37 outliers son candidatos a inspección por anomalía |

### Reducción de Dimensionalidad

| Técnica | Tipo | Varianza / Información capturada | Observación |
|---|---|---|---|
| PCA 2D | Lineal | PC1 = 12.2%, PC2 = 11.7% → **23.9% total** | Varianza muy repartida entre 9 componentes; se necesitan 7 componentes para alcanzar el 80% |
| t-SNE 2D | No lineal | Preserva relaciones locales (no mide varianza global) | Nube compacta sin islas separadas — confirma ausencia de clusters naturales bien definidos |

> Los valores de silhouette bajos y la baja varianza capturada por PCA 2D son consistentes entre sí: el dataset varía de forma continua en un espacio de alta dimensión, sin fronteras densas bien definidas.

## Recomendaciones

El equipo sugiere explorar **GMM (Gaussian Mixture Models)** para capturar clusters elípticos con asignación probabilística, aplicar **HDBSCAN** para mayor robustez ante densidades variables, y realizar **feature engineering** con ratios físicamente significativos (e.g., `Conductivity/Hardness` como proxy de mineralización) para obtener clusters más interpretables.

## Ejecución del Proyecto

### Opción A — Google Colab (recomendado)

1. Abrir [Google Colab](https://colab.research.google.com) e iniciar sesión con una cuenta de Google.
2. Subir el notebook: **Archivo → Subir notebook** → seleccionar `notebooks/clustering_no_supervisado.ipynb`.
3. Subir el dataset: en el panel lateral izquierdo (ícono de carpeta) → **Subir archivo** → seleccionar `DATA/dataset.csv`.
4. Ejecutar todas las celdas: **Entorno de ejecución → Ejecutar todo** (`Ctrl+F9`).
5. Los gráficos generados se guardarán en el directorio de trabajo de Colab (`/content/`).

> **Nota:** si el repositorio es público, se puede cargar el dataset directamente desde la URL configurada en la celda de carga (Sección 1 del notebook).

### Opción B — Ejecución Local

**Requisitos previos:** Python 3.8+ y `pip` instalados.

```bash
# 1. Clonar o descargar el repositorio
git clone <url-del-repositorio>
cd TAREA_3

# 2. (Opcional) Crear un entorno virtual
python -m venv .venv
source .venv/bin/activate        # Linux / macOS
# .venv\Scripts\activate         # Windows

# 3. Instalar dependencias
pip install pandas numpy matplotlib seaborn scikit-learn jupyter

# 4. Iniciar Jupyter
jupyter notebook notebooks/clustering_no_supervisado.ipynb
```

5. En el navegador, ejecutar todas las celdas: **Kernel → Restart & Run All**.
6. Los archivos de salida (`.png`) se generarán en la carpeta `outputs/`.

**Estructura de directorios esperada:**

```
TAREA_3/
├── DATA/
│   └── dataset.csv
├── notebooks/
│   └── clustering_no_supervisado.ipynb
└── outputs/
    ├── eda_distribuciones.png
    ├── eda_boxplots.png
    ├── eda_correlacion.png
    ├── kmeans_eleccion_k.png
    ├── kmeans_silhouette_muestras.png
    ├── dbscan_kdistance.png
    ├── pca_varianza.png
    ├── pca_clusters.png
    ├── tsne_clusters.png
    ├── comparacion_kmeans_dbscan.png
    └── perfiles_kmeans_heatmap.png
```

## Conclusiones

1. **La estructura de clusters es débil pero real.** El silhouette score de K-means (0.0773) confirma que los grupos se solapan considerablemente. Sin embargo, el heatmap de perfiles muestra diferencias fisicoquímicas consistentes: el Cluster 0 concentra muestras con mayor conductividad, sólidos disueltos y carbono orgánico, mientras que el Cluster 1 agrupa muestras con mayor turbidez, dureza y trihalometanos.

2. **Los clusters no predicen potabilidad.** Ambos clusters presentan tasas de potabilidad casi idénticas a la distribución global (~40%), lo que confirma que los patrones fisicoquímicos descubiertos son independientes de la aptitud para consumo. Esto es consistente con los resultados del aprendizaje supervisado (TAREA_2), donde ningún modelo superó el 60% de accuracy con estas mismas variables.

3. **K-means y DBSCAN son complementarios.** K-means entrega particiones interpretables útiles para caracterizar perfiles de agua. DBSCAN, aunque colapsa en un único cluster denso para este dataset, identifica 37 muestras anómalas (1.1%) que merecen inspección como posibles errores de sensor o contaminación inusual.

4. **El dataset es intrínsecamente alto-dimensional.** PCA requiere 7 de las 9 componentes para capturar el 80% de la varianza, y t-SNE no revela estructuras locales separadas. Esto prueba que el agua varía de forma continua en el espacio fisicoquímico, sin grupos naturales con fronteras bien definidas.

5. **La imputación con mediana es un factor limitante.** Con un 23.2% de nulos en `Sulfate`, la imputación introduce sesgo que puede difuminar las diferencias entre clusters. Estrategias más robustas (imputación múltiple, `IterativeImputer`) podrían revelar estructura adicional.
