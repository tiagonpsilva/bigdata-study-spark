# Exemplos Práticos do Apache Spark

## 1. Análise de Dados de E-commerce

### Estrutura de Dados
```python
# Schema dos dados
from pyspark.sql.types import *

customer_schema = StructType([
    StructField("customer_id", StringType(), False),
    StructField("name", StringType(), True),
    StructField("email", StringType(), True),
    StructField("registration_date", DateType(), True)
])

order_schema = StructType([
    StructField("order_id", StringType(), False),
    StructField("customer_id", StringType(), False),
    StructField("order_date", TimestampType(), False),
    StructField("total_amount", DecimalType(10,2), False),
    StructField("status", StringType(), False)
])

product_schema = StructType([
    StructField("product_id", StringType(), False),
    StructField("name", StringType(), False),
    StructField("category", StringType(), True),
    StructField("price", DecimalType(10,2), False)
])
```

### Pipeline de Processamento
```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import *

# Criar sessão
spark = SparkSession.builder \
    .appName("EcommerceAnalysis") \
    .config("spark.sql.warehouse.dir", "spark-warehouse") \
    .getOrCreate()

# Carregar dados
customers = spark.read.csv("data/customers.csv", schema=customer_schema)
orders = spark.read.csv("data/orders.csv", schema=order_schema)
products = spark.read.csv("data/products.csv", schema=product_schema)

# Análises
# 1. Vendas por categoria
sales_by_category = orders \
    .join(order_items, "order_id") \
    .join(products, "product_id") \
    .groupBy("category") \
    .agg(
        sum("total_amount").alias("total_sales"),
        count("order_id").alias("num_orders")
    ) \
    .orderBy(desc("total_sales"))

# 2. Clientes mais valiosos
valuable_customers = orders \
    .join(customers, "customer_id") \
    .groupBy("customer_id", "name") \
    .agg(
        sum("total_amount").alias("total_spent"),
        count("order_id").alias("num_orders"),
        avg("total_amount").alias("avg_order_value")
    ) \
    .orderBy(desc("total_spent"))

# 3. Análise temporal
daily_sales = orders \
    .groupBy(date_format("order_date", "yyyy-MM-dd").alias("date")) \
    .agg(
        sum("total_amount").alias("daily_revenue"),
        count("order_id").alias("num_orders")
    ) \
    .orderBy("date")

# Salvar resultados
sales_by_category.write.parquet("output/sales_by_category")
valuable_customers.write.parquet("output/valuable_customers")
daily_sales.write.parquet("output/daily_sales")
```

## 2. Processamento de Logs em Tempo Real

### Configuração do Stream
```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import *
from pyspark.sql.types import *

# Schema dos logs
log_schema = StructType([
    StructField("timestamp", TimestampType(), True),
    StructField("level", StringType(), True),
    StructField("service", StringType(), True),
    StructField("message", StringType(), True)
])

# Criar sessão Spark
spark = SparkSession.builder \
    .appName("LogProcessing") \
    .config("spark.streaming.stopGracefullyOnShutdown", "true") \
    .getOrCreate()

# Stream de logs
logs = spark \
    .readStream \
    .format("kafka") \
    .option("kafka.bootstrap.servers", "localhost:9092") \
    .option("subscribe", "logs") \
    .load()

# Parse JSON
parsed_logs = logs \
    .select(from_json(
        col("value").cast("string"),
        log_schema
    ).alias("data")) \
    .select("data.*")
```

### Processamento e Alertas
```python
# Detectar erros
errors = parsed_logs \
    .filter(col("level") == "ERROR") \
    .withWatermark("timestamp", "5 minutes") \
    .groupBy(
        window("timestamp", "1 minute"),
        "service"
    ) \
    .count()

# Alertas
def process_alerts(df, epoch_id):
    # Verificar thresholds
    alerts = df.filter(col("count") > 10)
    
    if alerts.count() > 0:
        # Enviar alertas
        alerts.write \
            .format("kafka") \
            .option("kafka.bootstrap.servers", "localhost:9092") \
            .option("topic", "alerts") \
            .save()

# Stream query
query = errors \
    .writeStream \
    .foreachBatch(process_alerts) \
    .outputMode("update") \
    .start()

# Console output para debug
debug_query = parsed_logs \
    .writeStream \
    .outputMode("append") \
    .format("console") \
    .start()
```

## 3. Pipeline de Machine Learning

### Preparação de Dados
```python
from pyspark.sql import SparkSession
from pyspark.ml.feature import VectorAssembler, StandardScaler
from pyspark.ml.classification import RandomForestClassifier
from pyspark.ml.evaluation import BinaryClassificationEvaluator
from pyspark.ml import Pipeline

# Carregar dados
data = spark.read.parquet("data/customer_data.parquet")

# Feature engineering
assembler = VectorAssembler() \
    .setInputCols([
        "age", "income", "spending_score",
        "num_purchases", "avg_purchase_value"
    ]) \
    .setOutputCol("features")

# Normalização
scaler = StandardScaler() \
    .setInputCol("features") \
    .setOutputCol("scaled_features")

# Split dados
train_data, test_data = data.randomSplit([0.8, 0.2], seed=42)
```

### Treinamento e Avaliação
```python
# Modelo
rf = RandomForestClassifier() \
    .setLabelCol("churn") \
    .setFeaturesCol("scaled_features") \
    .setNumTrees(100)

# Pipeline
pipeline = Pipeline(stages=[
    assembler,
    scaler,
    rf
])

# Treinar
model = pipeline.fit(train_data)

# Avaliar
predictions = model.transform(test_data)
evaluator = BinaryClassificationEvaluator() \
    .setLabelCol("churn")

auc = evaluator.evaluate(predictions)
print(f"AUC: {auc}")

# Salvar modelo
model.write().overwrite().save("models/churn_model")
```

### Serviço de Previsão
```python
def predict_churn(batch_df, batch_id):
    # Carregar modelo
    model = PipelineModel.load("models/churn_model")
    
    # Fazer previsões
    predictions = model.transform(batch_df)
    
    # Salvar resultados
    predictions \
        .select("customer_id", "prediction", "probability") \
        .write \
        .mode("append") \
        .parquet("predictions/churn")

# Stream de dados
stream_data = spark \
    .readStream \
    .format("kafka") \
    .option("kafka.bootstrap.servers", "localhost:9092") \
    .option("subscribe", "customer-events") \
    .load()

# Processar stream
query = stream_data \
    .writeStream \
    .foreachBatch(predict_churn) \
    .start()
```

## 4. Análise de Grafos

### Construção do Grafo
```python
from graphframes import GraphFrame
from pyspark.sql.functions import *

# Vértices (usuários)
users = spark.createDataFrame([
    ("1", "Alice", 25),
    ("2", "Bob", 30),
    ("3", "Charlie", 35)
], ["id", "name", "age"])

# Arestas (conexões)
connections = spark.createDataFrame([
    ("1", "2", "friend"),
    ("2", "3", "colleague"),
    ("3", "1", "friend")
], ["src", "dst", "relationship"])

# Criar grafo
g = GraphFrame(users, connections)
```

### Análise de Rede
```python
# PageRank
results = g.pageRank(resetProbability=0.15, maxIter=10)

# Componentes conectados
components = g.connectedComponents()

# Detecção de comunidades
communities = g.labelPropagation(maxIter=5)

# Análise de influenciadores
influencers = results.vertices \
    .orderBy(desc("pagerank")) \
    .limit(10)

# Padrões de conexão
patterns = g.find("(a)-[e]->(b); (b)-[e2]->(c)") \
    .filter("a.id != c.id")
```

## Exercícios Práticos

1. **E-commerce**
   - Implemente análise de cohort
   - Calcule métricas de retenção
   - Crie dashboard de vendas

2. **Processamento de Logs**
   - Adicione detecção de anomalias
   - Implemente sistema de alertas
   - Crie visualizações em tempo real

3. **Machine Learning**
   - Experimente diferentes algoritmos
   - Implemente validação cruzada
   - Otimize hiperparâmetros

4. **Análise de Grafos**
   - Detecte comunidades
   - Identifique influenciadores
   - Visualize conexões

## Recursos Adicionais

- [Spark Examples](https://spark.apache.org/examples.html)
- [Spark Streaming Examples](https://spark.apache.org/docs/latest/streaming-programming-guide.html#examples)
- [MLlib Examples](https://spark.apache.org/docs/latest/ml-guide.html)
- [GraphX Examples](https://spark.apache.org/docs/latest/graphx-programming-guide.html#examples)
- [Best Practices](https://spark.apache.org/docs/latest/sql-performance-tuning.html) 