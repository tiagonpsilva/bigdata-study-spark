# Spark Streaming

## DStreams (Discretized Streams)

### Conceitos Básicos
- Sequência contínua de RDDs
- Processamento em micro-batches
- Tolerância a falhas
- Integração com fontes de dados em tempo real

### Criação de DStreams

1. **Socket Stream**
```python
from pyspark.streaming import StreamingContext

# Criar contexto
ssc = StreamingContext(sc, batchDuration=1)

# Socket stream
lines = ssc.socketTextStream("localhost", 9999)
```

2. **Kafka Stream**
```python
from pyspark.streaming.kafka import KafkaUtils

# Configurar Kafka
kafkaParams = {
    "bootstrap.servers": "localhost:9092",
    "group.id": "streaming-group"
}

# Criar stream
stream = KafkaUtils.createDirectStream(ssc,
    topics=["meu-topico"],
    kafkaParams=kafkaParams)
```

### Operações

1. **Transformações**
```python
# Word count streaming
words = lines.flatMap(lambda line: line.split(" "))
pairs = words.map(lambda word: (word, 1))
word_counts = pairs.reduceByKey(lambda x, y: x + y)
```

2. **Window Operations**
```python
# Contagem por janela
word_counts = pairs.reduceByKeyAndWindow(
    lambda x, y: x + y,    # reduce function
    lambda x, y: x - y,    # inverse reduce
    windowDuration=30,     # window duration
    slideDuration=10       # sliding duration
)
```

## Structured Streaming

### Configuração Básica
```python
# Criar stream de dados
stream_df = spark \
    .readStream \
    .format("kafka") \
    .option("kafka.bootstrap.servers", "localhost:9092") \
    .option("subscribe", "meu-topico") \
    .load()
```

### Processamento

1. **Operações Básicas**
```python
# Transformações
from pyspark.sql.functions import *

processed = stream_df \
    .select(col("value").cast("string")) \
    .withColumn("timestamp", current_timestamp()) \
    .withWatermark("timestamp", "10 minutes")
```

2. **Agregações**
```python
# Agregações por janela
result = processed \
    .groupBy(
        window("timestamp", "1 hour", "30 minutes"),
        "value"
    ) \
    .count()
```

### Output Modes

1. **Append Mode**
```python
# Somente novos registros
query = result \
    .writeStream \
    .outputMode("append") \
    .format("console") \
    .start()
```

2. **Complete Mode**
```python
# Todo o resultado
query = result \
    .writeStream \
    .outputMode("complete") \
    .format("memory") \
    .queryName("results") \
    .start()
```

3. **Update Mode**
```python
# Apenas registros atualizados
query = result \
    .writeStream \
    .outputMode("update") \
    .format("console") \
    .start()
```

## Integração com Kafka

### Producer
```python
# Enviar dados para Kafka
def send_to_kafka(df, epoch_id):
    df.selectExpr("CAST(key AS STRING)", "CAST(value AS STRING)") \
        .write \
        .format("kafka") \
        .option("kafka.bootstrap.servers", "localhost:9092") \
        .option("topic", "output-topic") \
        .save()

# Stream query
query = processed \
    .writeStream \
    .foreachBatch(send_to_kafka) \
    .start()
```

### Consumer
```python
# Consumir e processar dados
df = spark \
    .readStream \
    .format("kafka") \
    .option("kafka.bootstrap.servers", "localhost:9092") \
    .option("subscribe", "input-topic") \
    .option("startingOffsets", "earliest") \
    .load()

# Processar stream
processed = df \
    .select(from_json(
        col("value").cast("string"),
        schema
    ).alias("data")) \
    .select("data.*")
```

## Exemplos Práticos

### 1. Análise de Logs em Tempo Real
```python
# Schema dos logs
log_schema = StructType([
    StructField("timestamp", TimestampType(), True),
    StructField("level", StringType(), True),
    StructField("message", StringType(), True)
])

# Stream de logs
logs = spark \
    .readStream \
    .format("json") \
    .schema(log_schema) \
    .load("/path/to/logs")

# Análise
alerts = logs \
    .filter(col("level") == "ERROR") \
    .withWatermark("timestamp", "5 minutes") \
    .groupBy(
        window("timestamp", "1 minute"),
        "level"
    ) \
    .count()
```

### 2. Processamento de Eventos IoT
```python
# Processar dados de sensores
def process_sensor_data(df, epoch_id):
    # Calcular estatísticas
    stats = df \
        .groupBy("sensor_id") \
        .agg(
            avg("temperature").alias("avg_temp"),
            max("temperature").alias("max_temp"),
            min("temperature").alias("min_temp")
        )
    
    # Salvar resultados
    stats.write \
        .format("jdbc") \
        .option("url", "jdbc:postgresql:dbserver") \
        .option("dbtable", "sensor_stats") \
        .mode("append") \
        .save()

# Stream query
query = sensor_stream \
    .writeStream \
    .foreachBatch(process_sensor_data) \
    .trigger(processingTime='1 minute') \
    .start()
```

## Exercícios Práticos

1. **Streaming Básico**
   - Configure stream de socket
   - Processe dados em tempo real
   - Implemente word count streaming

2. **Kafka Integration**
   - Configure producer e consumer
   - Processe mensagens JSON
   - Implemente transformações

3. **Window Operations**
   - Use diferentes tipos de janelas
   - Implemente agregações
   - Trabalhe com watermarks

4. **Monitoramento**
   - Configure métricas
   - Monitore performance
   - Implemente recuperação de falhas

## Recursos Adicionais

- [Structured Streaming Guide](https://spark.apache.org/docs/latest/structured-streaming-programming-guide.html)
- [DStreams Guide](https://spark.apache.org/docs/latest/streaming-programming-guide.html)
- [Kafka Integration](https://spark.apache.org/docs/latest/structured-streaming-kafka-integration.html)
- [Performance Tuning](https://spark.apache.org/docs/latest/structured-streaming-programming-guide.html#performance-tuning) 