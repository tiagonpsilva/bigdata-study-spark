# MLlib (Machine Learning Library)

## Algoritmos de Machine Learning

### Classificação
```python
from pyspark.ml.classification import LogisticRegression, RandomForestClassifier
from pyspark.ml.evaluation import BinaryClassificationEvaluator

# Regressão Logística
lr = LogisticRegression(maxIter=10) \
    .setLabelCol("label") \
    .setFeaturesCol("features")

# Random Forest
rf = RandomForestClassifier(numTrees=100) \
    .setLabelCol("label") \
    .setFeaturesCol("features")

# Avaliação
evaluator = BinaryClassificationEvaluator() \
    .setLabelCol("label") \
    .setRawPredictionCol("rawPrediction")
```

### Regressão
```python
from pyspark.ml.regression import LinearRegression, RandomForestRegressor
from pyspark.ml.evaluation import RegressionEvaluator

# Regressão Linear
lr = LinearRegression(maxIter=10) \
    .setLabelCol("price") \
    .setFeaturesCol("features")

# Random Forest Regressor
rf = RandomForestRegressor(numTrees=100) \
    .setLabelCol("price") \
    .setFeaturesCol("features")

# Avaliação
evaluator = RegressionEvaluator() \
    .setLabelCol("price") \
    .setPredictionCol("prediction") \
    .setMetricName("rmse")
```

### Clustering
```python
from pyspark.ml.clustering import KMeans, BisectingKMeans
from pyspark.ml.evaluation import ClusteringEvaluator

# K-Means
kmeans = KMeans(k=3, seed=1) \
    .setFeaturesCol("features")

# Bisecting K-Means
bkm = BisectingKMeans(k=3, seed=1) \
    .setFeaturesCol("features")

# Avaliação
evaluator = ClusteringEvaluator() \
    .setFeaturesCol("features") \
    .setPredictionCol("prediction")
```

## Pipeline API

### Feature Engineering

1. **Transformações Numéricas**
```python
from pyspark.ml.feature import StandardScaler, MinMaxScaler, VectorAssembler

# Combinar features
assembler = VectorAssembler() \
    .setInputCols(["idade", "salario", "score"]) \
    .setOutputCol("features")

# Normalização
scaler = StandardScaler() \
    .setInputCol("features") \
    .setOutputCol("scaled_features")
```

2. **Transformações Categóricas**
```python
from pyspark.ml.feature import StringIndexer, OneHotEncoder

# String Indexer
indexer = StringIndexer() \
    .setInputCol("categoria") \
    .setOutputCol("categoria_index")

# One Hot Encoder
encoder = OneHotEncoder() \
    .setInputCol("categoria_index") \
    .setOutputCol("categoria_vec")
```

### Pipeline Completo
```python
from pyspark.ml import Pipeline

# Definir pipeline
pipeline = Pipeline(stages=[
    # Feature Engineering
    indexer,
    encoder,
    assembler,
    scaler,
    
    # Modelo
    classifier
])

# Treinar pipeline
model = pipeline.fit(train_data)

# Fazer previsões
predictions = model.transform(test_data)
```

## Exemplos Práticos

### 1. Classificação de Churn
```python
from pyspark.ml.feature import VectorAssembler, StandardScaler
from pyspark.ml.classification import RandomForestClassifier
from pyspark.ml import Pipeline

# Preparar dados
assembler = VectorAssembler() \
    .setInputCols([
        "tenure", "MonthlyCharges", "TotalCharges",
        "InternetService_index", "Contract_index"
    ]) \
    .setOutputCol("features")

# Normalizar features
scaler = StandardScaler() \
    .setInputCol("features") \
    .setOutputCol("scaled_features")

# Modelo
rf = RandomForestClassifier() \
    .setLabelCol("Churn_index") \
    .setFeaturesCol("scaled_features") \
    .setNumTrees(100)

# Pipeline
pipeline = Pipeline(stages=[assembler, scaler, rf])
model = pipeline.fit(train_data)

# Avaliar
predictions = model.transform(test_data)
evaluator = BinaryClassificationEvaluator() \
    .setLabelCol("Churn_index")
accuracy = evaluator.evaluate(predictions)
```

### 2. Regressão de Preços
```python
from pyspark.ml.feature import VectorAssembler, MinMaxScaler
from pyspark.ml.regression import LinearRegression
from pyspark.ml import Pipeline

# Preparar features
assembler = VectorAssembler() \
    .setInputCols([
        "area", "bedrooms", "bathrooms",
        "location_index", "condition_index"
    ]) \
    .setOutputCol("features")

# Normalizar
scaler = MinMaxScaler() \
    .setInputCol("features") \
    .setOutputCol("scaled_features")

# Modelo
lr = LinearRegression() \
    .setLabelCol("price") \
    .setFeaturesCol("scaled_features")

# Pipeline
pipeline = Pipeline(stages=[assembler, scaler, lr])
model = pipeline.fit(train_data)

# Métricas
predictions = model.transform(test_data)
evaluator = RegressionEvaluator() \
    .setLabelCol("price")
rmse = evaluator.evaluate(predictions)
```

### 3. Clustering de Clientes
```python
from pyspark.ml.feature import VectorAssembler, StandardScaler
from pyspark.ml.clustering import KMeans
from pyspark.ml import Pipeline

# Features
assembler = VectorAssembler() \
    .setInputCols([
        "recency", "frequency", "monetary"
    ]) \
    .setOutputCol("features")

# Normalizar
scaler = StandardScaler() \
    .setInputCol("features") \
    .setOutputCol("scaled_features")

# K-Means
kmeans = KMeans() \
    .setK(3) \
    .setFeaturesCol("scaled_features")

# Pipeline
pipeline = Pipeline(stages=[assembler, scaler, kmeans])
model = pipeline.fit(data)

# Análise
results = model.transform(data)
centers = model.stages[-1].clusterCenters()
```

## Exercícios Práticos

1. **Classificação**
   - Implemente diferentes algoritmos
   - Compare métricas
   - Faça tuning de hiperparâmetros

2. **Regressão**
   - Use regressão linear e não-linear
   - Avalie diferentes métricas
   - Implemente validação cruzada

3. **Clustering**
   - Experimente K-Means e BisectingKMeans
   - Determine número ótimo de clusters
   - Analise resultados

4. **Feature Engineering**
   - Implemente diferentes transformações
   - Compare impacto na performance
   - Crie pipelines completos

## Recursos Adicionais

- [MLlib Guide](https://spark.apache.org/docs/latest/ml-guide.html)
- [Pipeline API](https://spark.apache.org/docs/latest/ml-pipeline.html)
- [Feature Extraction](https://spark.apache.org/docs/latest/ml-features.html)
- [Classification and Regression](https://spark.apache.org/docs/latest/ml-classification-regression.html)
- [Clustering](https://spark.apache.org/docs/latest/ml-clustering.html)
- [Model Selection](https://spark.apache.org/docs/latest/ml-tuning.html) 