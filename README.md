# EmotionScope AI 🎭

> Análisis de estados emocionales colectivos mediante procesamiento de texto y NLP

Proyecto final del curso **BD-141 — Big Data**
Colegio Universitario de Cartago (CUC) · Cuatrimestre II, 2026
Profesora: Ericka Valverde Navarro
Autor: Roberto González Gómez

---

## 📋 Descripción del proyecto

Las redes sociales generan enormes volúmenes de texto donde las personas expresan
emociones de forma espontánea. **EmotionScope AI** es un pipeline de Big Data e
Inteligencia Artificial que procesa, limpia, almacena y clasifica el sentimiento de
publicaciones de Twitter, con el objetivo de detectar estados emocionales colectivos
a partir de datos reales.

El proyecto se desarrolló en 6 fases, cubriendo desde la definición del problema
hasta un análisis de escalabilidad y despliegue en la nube.

## 🎯 Objetivos

**Objetivo general:**
Diseñar un sistema capaz de analizar texto proveniente de redes sociales para
identificar los estados emocionales predominantes en un grupo de personas.

**Objetivos específicos:**
- Recolectar y organizar un conjunto de datos textuales.
- Aplicar técnicas de NLP para detectar emociones en cada texto.
- Construir un modelo de clasificación de sentimiento.
- Representar gráficamente los resultados obtenidos.

## 🏗️ Arquitectura del sistema

```
Fuente de datos → Módulo de limpieza → Motor de NLP/IA → Base de datos SQL → Visualización
```

Cuando entra un comentario al sistema, pasa primero por limpieza de texto (URLs,
menciones, hashtags), luego se envía al modelo de IA para clasificar la emoción
detectada, se almacena en un Data Warehouse relacional y finalmente se visualiza
mediante gráficos y consultas SQL.

## 🗄️ Modelo de datos

Los datos se organizan en un **modelo estrella** dentro de una base de datos SQLite
(`emotion_scope.db`):

| Tabla | Descripción |
|---|---|
| `tabla_hechos` | Tabla central: relaciona comentario, fecha, fuente y emoción; guarda la etiqueta original y el resultado del modelo |
| `comentario` | Texto original y texto limpio de cada tweet |
| `fecha` | Dimensión temporal (año, mes, día, día de la semana) |
| `fuente` | Origen del dato (Sentiment140, API simulada, etc.) |
| `emocion` | Categorías que produce el modelo (negativo, neutral, positivo) |
| `stream_simulado` | Registros generados durante la simulación de procesamiento en tiempo real (Fase 5) |

## 📊 Dataset

Se utilizó **[Sentiment140](http://help.sentiment140.com/for-students)**, un dataset
público de ~1.6 millones de tweets recolectados entre abril y junio de 2009,
etiquetados como positivo o negativo (sin categoría neutral).

Debido a limitaciones del entorno académico, se trabajó con una muestra aleatoria
de **10.000 registros** (`random_state=42` para reproducibilidad).

## 🤖 Modelo de IA

Se utilizó el modelo preentrenado
[`cardiffnlp/twitter-xlm-roberta-base-sentiment`](https://huggingface.co/cardiffnlp/twitter-xlm-roberta-base-sentiment)
de Hugging Face, entrenado específicamente sobre datos de Twitter y con soporte
multilenguaje.

**Resultados obtenidos** (evaluados contra las etiquetas originales positivo/negativo):

| Métrica | Valor |
|---|---|
| Accuracy | 0.818 |
| Precision | 0.801 |
| Recall | 0.812 |
| F1 Score | 0.806 |

> Nota: el modelo predice 3 clases (negativo/neutral/positivo), pero el dataset
> original solo contiene positivo/negativo. Las métricas se calcularon excluyendo
> las predicciones neutrales, por no existir una etiqueta original comparable.

## ⚡ Automatización y procesamiento en tiempo real

Como parte de la Fase 5, se implementó una simulación de ingesta de datos en
tiempo real utilizando `asyncio`, integrando dos fuentes:

- El dataset Sentiment140 (muestra aleatoria progresiva).
- Una API pública real ([JSONPlaceholder](https://jsonplaceholder.typicode.com/)).

Cada dato recibido se limpia, se clasifica con el modelo de IA y se almacena
automáticamente en la base de datos, sin intervención manual, usando
procesamiento asíncrono (`asyncio.gather`, `run_in_executor`) para manejar
ambas fuentes en paralelo.

## ☁️ Escalabilidad y nube

Se analizó cómo evolucionaría el sistema en un ambiente de producción real:

- **Cómputo:** servicios PaaS con GPU bajo demanda (ej. Amazon SageMaker).
- **Almacenamiento:** migración de SQLite a un DBMS gestionado en la nube (ej. Amazon RDS).
- **Ingesta de datos:** Apache Kafka + Apache Flink para manejar streaming real de múltiples fuentes.
- **Procesamiento distribuido:** Apache Spark para superar las limitaciones de una sola máquina.

Detalle completo del análisis en `docs/Fase_6.pdf`.

## 🛠️ Tecnologías utilizadas

| Categoría | Herramientas |
|---|---|
| Lenguaje | Python 3 |
| Datos | pandas, re |
| Base de datos | SQLite (sqlite3) |
| Modelo de IA | Hugging Face Transformers, PyTorch |
| Métricas | scikit-learn |
| Procesamiento de texto | NLTK (stopwords) |
| Asíncrono / APIs | asyncio, requests |
| Visualización | matplotlib |
| Entorno | Google Colab |

## 📁 Estructura del repositorio

```
repo_en_coolab/
├── Fase_3_proyecto_BD.ipynb
├── emotion_scope.db
├── sentiment140_procesado.csv
├── README.md
└── requirements.txt
```

## 🚀 Cómo ejecutar el proyecto

1. Clonar el repositorio:
   ```bash
   git clone https://github.com/<tu-usuario>/emotionscope-ai.git
   cd emotionscope-ai
   ```

2. Instalar dependencias:
   ```bash
   pip install pandas transformers torch scikit-learn matplotlib nltk requests nest_asyncio
   ```

3. Abrir `notebooks/Fase_3_proyecto_BD.ipynb` en Google Colab

4. Ejecutar las celdas en orden (**Entorno de ejecución → Ejecutar todas**). El
   notebook cubre, en orden: carga y limpieza de datos, creación del Data
   Warehouse, análisis exploratorio, clasificación con el modelo de IA, y la
   simulación de procesamiento en tiempo real.

> El dataset original de Sentiment140 debe descargarse por separado desde
> [su fuente oficial](http://help.sentiment140.com/for-students) y colocarse en
> la ruta indicada en la celda de carga, ya que no se incluye en este repositorio
> por su tamaño.

## ⚠️ Limitaciones conocidas

- El dataset solo contiene tweets de 2009, en inglés, sin categoría neutral original.
- El modelo no está afinado (fine-tuned) específicamente para este dominio.
- No se evaluó el desempeño del modelo frente a sarcasmo o ironía de forma cuantitativa.
- El procesamiento corre localmente en CPU; no está optimizado para grandes volúmenes.

## 🔮 Mejoras futuras

- Fine-tuning del modelo con datos más recientes y en español.
- Integración con fuentes de datos reales (X/Twitter API, Reddit).
- Dashboard interactivo en tiempo real.
- Migración a arquitectura en la nube (ver Fase 6).

## 👤 Autor

**Roberto González Gómez**
Curso BD-141 — Big Data
Colegio Universitario de Cartago

## 📄 Licencia

Proyecto académico desarrollado con fines educativos para el curso BD-141.
