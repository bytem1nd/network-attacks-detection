<p align="left"><b>🇪🇸 Español</b> | <a href="README.en.md">🇬🇧 English</a></p>

# Detección de ataques en una infraestructura de red

Proyecto de aprendizaje automático supervisado que clasifica conexiones de red como **tráfico normal** o **anómalo (ataque)** a partir de sus características, usando el conjunto de datos **NSL-KDD**. Se comparan dos algoritmos: **regresión logística** y **árbol de decisión**.

Todo el trabajo está en un único notebook: [`Detección de ataques en red.ipynb`](Detecci%C3%B3n%20de%20ataques%20en%20red.ipynb).

## Contenido del repositorio

```
.
├── Detección de ataques en red.ipynb   # Notebook con todo el análisis
└── NSL-KDD/                                          # Dataset (ARFF y TXT)
    ├── KDDTrain+.arff / .txt                         # Entrenamiento completo (125 973 registros)
    ├── KDDTrain+_20Percent.arff / .txt               # Subconjunto del 20 % del entrenamiento
    ├── KDDTest+.arff / .txt                          # Prueba completo (22 544 registros)
    ├── KDDTest-21.arff / .txt                        # Prueba sin los registros de dificultad 21 (11 850)
    └── index.html                                    # Descripción oficial del dataset
```

## Dataset: NSL-KDD

NSL-KDD es una versión depurada de KDD'99 que elimina registros redundantes y duplicados, lo que lo hace un benchmark razonable para comparar métodos de detección de intrusiones (IDS). Cada registro describe una conexión con **41 características** (duración, protocolo, servicio, bytes enviados/recibidos, tasas de error, etc.) más la etiqueta `class` (`normal` / `anomaly`).

- Más información: ver `NSL-KDD/index.html` o la [página oficial del UNB](https://www.unb.ca/cic/datasets/nsl.html).
- Formato ARFF: formato de texto de Weka; se lee con la librería `liac-arff`.

## Flujo de trabajo del notebook

0. **Imports**: instalación de `liac-arff` y carga de librerías.
1. **Lectura de datos**: funciones auxiliares `load_kdd_data`, `get_df_from_dict`, `remove_label` y `evaluate_classifier`.
2. **Análisis exploratorio**: dimensiones (125 973 × 42), tipos, ausencia de nulos, estadísticos y variables categóricas (`protocol_type`, `service`, `flag`).
3. **Preprocesamiento**
   - Comprobación de valores nulos.
   - `OneHotEncoder` para las variables categóricas (evita sesgos ordinales de `LabelEncoder`).
   - Selección de características por correlación con la clase (|r| > 0.5) → **12 variables**:
     `count`, `serror_rate`, `srv_serror_rate`, `dst_host_serror_rate`, `dst_host_srv_serror_rate`, `flag_S0`, `logged_in`, `same_srv_rate`, `dst_host_srv_count`, `dst_host_same_srv_rate`, `service_http`, `flag_SF`.
   - Escalado con `RobustScaler` (robusto frente a outliers).
   - División train/test 80/20 (`random_state=42`).
4. **Modelos**
   - `LogisticRegression(max_iter=1000)` sobre las 12 variables escaladas.
   - `DecisionTreeClassifier(max_depth=50, random_state=42)` sobre las variables sin escalar ni reducir.
5. **Evaluación**: `F1-score` y matriz de confusión.

## Resultados

| Modelo               | F1 (train) | F1 (test) |
|----------------------|-----------:|----------:|
| Árbol de decisión    |     0.9999 |    0.9983 |
| Regresión logística  |     0.9144 |    0.9143 |

El árbol de decisión supera claramente a la regresión logística: es más flexible, captura interacciones entre variables y es más resistente a outliers, mientras que la regresión logística solo modela fronteras lineales.

## Requisitos

- Python 3.12 (probado con 3.12.8)
- Jupyter Notebook / JupyterLab
- `pandas`, `scikit-learn`, `matplotlib`, `seaborn`, `liac-arff`

## Ejecución

```bash
# 1. Crear y activar un entorno virtual
python3 -m venv .venv
source .venv/bin/activate

# 2. Instalar dependencias
pip install jupyterlab pandas scikit-learn matplotlib seaborn liac-arff

# 3. Abrir el notebook desde la raíz del proyecto
jupyter lab "Detección de ataques en red.ipynb"
```

Las rutas del dataset son relativas (`NSL-KDD/KDDTrain+.arff`), por lo que el notebook debe ejecutarse desde la raíz del repositorio.

## Referencias

- M. Tavallaee, E. Bagheri, W. Lu, A. Ghorbani, *A Detailed Analysis of the KDD CUP 99 Data Set*, IEEE CISDA, 2009.
- Formato ARFF: <https://www.cs.waikato.ac.nz/ml/weka/arff.html>
