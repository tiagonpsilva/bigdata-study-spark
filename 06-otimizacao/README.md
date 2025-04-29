# Otimização do Apache Spark

## Tunning de Performance

### 1. Configuração de Recursos

#### Memória
```python
# Configuração via SparkConf
conf = SparkConf() \
    .set("spark.executor.memory", "4g") \
    .set("spark.driver.memory", "2g") \
    .set("spark.memory.offHeap.enabled", "true") \
    .set("spark.memory.offHeap.size", "1g")

# Configuração via Runtime
spark.conf.set("spark.sql.shuffle.partitions", 200)
spark.conf.set("spark.sql.autoBroadcastJoinThreshold", 10485760)
```

#### CPU
```python
# Configuração de Executores
spark.conf.set("spark.executor.cores", 4)
spark.conf.set("spark.executor.instances", 2)
spark.conf.set("spark.dynamicAllocation.enabled", "true")
```

### 2. Otimização de Consultas

#### Catalyst Optimizer
```python
# Visualizar plano de execução
df.explain(True)

# Análise de plano físico
df.explain("cost")

# Regras de otimização
spark.conf.set("spark.sql.optimizer.maxIterations", 100)
```

#### Join Optimization
```python
# Broadcast Join
from pyspark.sql.functions import broadcast

# Forçar broadcast
result = df1.join(broadcast(df2), "key")

# Configurar threshold
spark.conf.set("spark.sql.autoBroadcastJoinThreshold", 10485760)
```

## Particionamento

### 1. Particionamento de Dados

#### RDD Partitioning
```python
# Reparticionamento
rdd = sc.parallelize(range(100), 10)
repartitioned = rdd.repartition(20)

# Coalesce
reduced = rdd.coalesce(5)
```

#### DataFrame Partitioning
```python
# Reparticionamento
df = df.repartition(10, "column")

# Particionamento por bucket
df.write \
    .bucketBy(4, "id") \
    .sortBy("timestamp") \
    .saveAsTable("tabela_bucketizada")
```

### 2. Particionamento de Storage

#### Particionamento em Disco
```python
# Escrita particionada
df.write \
    .partitionBy("ano", "mes") \
    .parquet("dados/")

# Leitura particionada
df = spark.read \
    .option("basePath", "dados/") \
    .parquet("dados/ano=2023")
```

## Caching

### 1. Níveis de Storage

```python
from pyspark import StorageLevel

# Cache em memória
df.cache()

# Persist com opções
df.persist(StorageLevel.MEMORY_AND_DISK)

# Unpersist
df.unpersist()
```

### 2. Estratégias de Cache

```python
# Cache seletivo
filtered_df = df.filter(col("value") > 100)
filtered_df.cache()

# Cache de tabelas
df.createOrReplaceTempView("mytable")
spark.sql("CACHE TABLE mytable")
```

## Monitoramento

### 1. Spark UI

#### Métricas Importantes
- Jobs e Stages
- Executors
- Storage
- SQL
- Environment

### 2. Instrumentação

#### Listeners
```python
from pyspark.scheduler import SparkListener

class CustomSparkListener(SparkListener):
    def onApplicationStart(self, applicationStart):
        print("Application started: %s" % applicationStart.appName)
    
    def onTaskEnd(self, taskEnd):
        print("Task finished: %s" % taskEnd.taskInfo.taskId)

# Registrar listener
sc.addSparkListener(CustomSparkListener())
```

#### Métricas Customizadas
```python
from pyspark.accumulators import AccumulatorParam

class CustomAccumulator(AccumulatorParam):
    def zero(self, value):
        return value
    
    def addInPlace(self, v1, v2):
        return v1 + v2

# Criar acumulador
custom_acc = sc.accumulator(0, CustomAccumulator())
```

## Exemplos Práticos

### 1. Otimização de Join
```python
from pyspark.sql.functions import broadcast

# Dados
large_df = spark.createDataFrame([(i, f"value_{i}") for i in range(1000000)], ["id", "value"])
small_df = spark.createDataFrame([(i, f"type_{i}") for i in range(1000)], ["id", "type"])

# Configuração
spark.conf.set("spark.sql.autoBroadcastJoinThreshold", 10485760)

# Join otimizado
result = large_df.join(broadcast(small_df), "id")
```

### 2. Particionamento Eficiente
```python
# Dados temporais
df = spark.createDataFrame([
    (1, "2023-01-01", 100),
    (2, "2023-01-02", 200)
], ["id", "date", "value"])

# Particionamento
df.write \
    .partitionBy("date") \
    .bucketBy(4, "id") \
    .saveAsTable("dados_otimizados")

# Leitura otimizada
spark.read.table("dados_otimizados") \
    .filter(col("date") == "2023-01-01") \
    .explain()
```

### 3. Monitoramento de Performance
```python
from pyspark.scheduler import SparkListener
import json
import time

class PerformanceMonitor(SparkListener):
    def __init__(self):
        self.stage_metrics = {}
    
    def onStageCompleted(self, stageCompleted):
        stage_id = stageCompleted.stageInfo.stageId
        duration = stageCompleted.stageInfo.completionTime - stageCompleted.stageInfo.submissionTime
        
        self.stage_metrics[stage_id] = {
            "duration": duration,
            "tasks": stageCompleted.stageInfo.numTasks,
            "failed_tasks": stageCompleted.stageInfo.numFailedTasks
        }
        
        print(f"Stage {stage_id} completed: {json.dumps(self.stage_metrics[stage_id], indent=2)}")

# Registrar monitor
spark.sparkContext.addSparkListener(PerformanceMonitor())
```

## Exercícios Práticos

1. **Tunning de Memória**
   - Configure diferentes níveis de memória
   - Monitore uso de memória
   - Otimize configurações

2. **Otimização de Joins**
   - Compare tipos de joins
   - Implemente broadcast joins
   - Analise planos de execução

3. **Estratégias de Cache**
   - Teste diferentes níveis de storage
   - Compare performance
   - Implemente cache seletivo

4. **Monitoramento**
   - Configure listeners
   - Colete métricas
   - Crie dashboards

## Recursos Adicionais

- [Spark Configuration](https://spark.apache.org/docs/latest/configuration.html)
- [Spark Monitoring Guide](https://spark.apache.org/docs/latest/monitoring.html)
- [Performance Tuning](https://spark.apache.org/docs/latest/tuning.html)
- [SQL Performance Tuning](https://spark.apache.org/docs/latest/sql-performance-tuning.html)
- [Memory Management](https://spark.apache.org/docs/latest/tuning.html#memory-management) 