# Conceitos Básicos do Apache Spark

## Fundamentos

O Apache Spark é um framework de processamento distribuído de código aberto, projetado para computação em larga escala. Suas principais características são:

- Processamento em memória
- Computação distribuída
- Tolerância a falhas
- Lazy evaluation
- APIs em múltiplas linguagens (Scala, Java, Python, R)
- Ecossistema rico (SQL, Streaming, ML, Graph)

## Arquitetura

```plaintext
┌─────────────────┐
│  Driver Program │
│  SparkContext  │
└───────┬─────────┘
        │
        ▼
┌─────────────────┐
│ Cluster Manager │
│ (Standalone/   │
│  YARN/Mesos)   │
└───────┬─────────┘
        │
    ┌───┴───┐
    ▼       ▼
┌─────┐   ┌─────┐
│Worker│   │Worker│
│ Node │   │ Node │
└─────┘   └─────┘
```

### Componentes

1. **Driver Program**
   - Contém a aplicação principal
   - Cria SparkContext
   - Declara transformações e ações
   - Coordena a execução

2. **Cluster Manager**
   - Gerencia recursos do cluster
   - Aloca executores
   - Monitora recursos

3. **Worker Nodes**
   - Executam tarefas
   - Armazenam dados em memória/disco
   - Processam partições de RDDs

## RDDs (Resilient Distributed Datasets)

### Características
- Imutável
- Distribuído
- Resiliente
- Lazy evaluation
- Particionado

### Operações

1. **Transformações**
```python
# Exemplo de transformações
rdd = sc.parallelize([1, 2, 3, 4, 5])
mapped = rdd.map(lambda x: x * 2)
filtered = rdd.filter(lambda x: x % 2 == 0)
```

2. **Ações**
```python
# Exemplo de ações
count = rdd.count()
collected = rdd.collect()
first = rdd.first()
```

### Persistência

```python
# Níveis de armazenamento
from pyspark import StorageLevel

rdd.persist(StorageLevel.MEMORY_ONLY)
rdd.persist(StorageLevel.MEMORY_AND_DISK)
rdd.persist(StorageLevel.DISK_ONLY)
```

## SparkContext e SparkSession

### SparkContext
```python
from pyspark import SparkContext, SparkConf

conf = SparkConf().setAppName("MinhaApp").setMaster("local[*]")
sc = SparkContext(conf=conf)

# Operações com RDD
rdd = sc.textFile("dados.txt")
```

### SparkSession
```python
from pyspark.sql import SparkSession

spark = SparkSession.builder \
    .appName("MinhaApp") \
    .config("spark.some.config.option", "some-value") \
    .getOrCreate()

# Operações com DataFrame
df = spark.read.csv("dados.csv")
```

## Exemplos Práticos

### 1. Contagem de Palavras
```python
# Word Count com RDD
text_file = sc.textFile("texto.txt")
counts = text_file \
    .flatMap(lambda line: line.split(" ")) \
    .map(lambda word: (word, 1)) \
    .reduceByKey(lambda a, b: a + b)

counts.saveAsTextFile("output")
```

### 2. Análise de Logs
```python
# Análise de logs com RDD
logs = sc.textFile("logs.txt")
errors = logs \
    .filter(lambda line: "ERROR" in line) \
    .map(lambda line: line.split("\t")) \
    .map(lambda fields: (fields[0], 1)) \
    .reduceByKey(lambda a, b: a + b)

errors.collect()
```

### 3. Processamento de CSV
```python
# Processamento de CSV com DataFrame
df = spark.read.csv("dados.csv", header=True)
resultado = df \
    .filter(df.idade > 18) \
    .groupBy("cidade") \
    .count()

resultado.show()
```

## Exercícios Práticos

1. **RDD Básico**
   - Crie um RDD a partir de uma lista
   - Aplique transformações map e filter
   - Execute ações como count e collect

2. **Persistência**
   - Experimente diferentes níveis de storage
   - Compare performance
   - Monitore memória e disco

3. **SparkContext vs SparkSession**
   - Crie aplicações usando ambos
   - Trabalhe com RDDs e DataFrames
   - Compare funcionalidades

4. **Word Count**
   - Implemente contagem de palavras
   - Use diferentes transformações
   - Otimize o processamento

## Recursos Adicionais

- [Documentação Oficial do Spark](https://spark.apache.org/docs/latest/)
- [RDD Programming Guide](https://spark.apache.org/docs/latest/rdd-programming-guide.html)
- [Spark Configuration](https://spark.apache.org/docs/latest/configuration.html)
- [Best Practices](https://spark.apache.org/docs/latest/tuning.html) 