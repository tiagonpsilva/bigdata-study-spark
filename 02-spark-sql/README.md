# Spark SQL

## DataFrames e Datasets

### Criação de DataFrames

1. **A partir de arquivos**
```python
# CSV
df_csv = spark.read.csv("dados.csv", header=True, inferSchema=True)

# JSON
df_json = spark.read.json("dados.json")

# Parquet
df_parquet = spark.read.parquet("dados.parquet")
```

2. **A partir de RDDs**
```python
from pyspark.sql.types import StructType, StructField, StringType, IntegerType

# Define o schema
schema = StructType([
    StructField("nome", StringType(), True),
    StructField("idade", IntegerType(), True)
])

# Cria DataFrame
rdd = sc.parallelize([("João", 25), ("Maria", 30)])
df = spark.createDataFrame(rdd, schema)
```

3. **A partir de listas**
```python
# Lista de dicionários
dados = [
    {"nome": "João", "idade": 25},
    {"nome": "Maria", "idade": 30}
]
df = spark.createDataFrame(dados)
```

## Operações e Transformações

### Seleção e Filtragem
```python
# Select
df.select("nome", "idade").show()
df.select(df.nome, df.idade + 1).show()

# Filter
df.filter(df.idade > 25).show()
df.filter("idade > 25").show()

# Where
df.where(df.idade.between(25, 30)).show()
```

### Agregações
```python
# GroupBy
df.groupBy("departamento").count().show()

# Funções de agregação
from pyspark.sql.functions import avg, sum, max, min

df.groupBy("departamento").agg(
    avg("salario").alias("media_salario"),
    max("salario").alias("max_salario")
).show()
```

### Joins
```python
# Inner Join
df1.join(df2, "id")

# Left Join
df1.join(df2, "id", "left")

# Multiple conditions
df1.join(df2, ["id", "tipo"])
```

## Catálogo e Metastore

### Tabelas Temporárias
```python
# Criar view temporária
df.createOrReplaceTempView("funcionarios")

# Consultar view
resultado = spark.sql("""
    SELECT departamento, 
           AVG(salario) as media_salario
    FROM funcionarios
    GROUP BY departamento
""")
```

### Tabelas Permanentes
```python
# Salvar como tabela
df.write.saveAsTable("db.funcionarios")

# Ler tabela
df = spark.table("db.funcionarios")
```

## Otimização de Consultas

### Catalyst Optimizer
```python
# Visualizar plano de execução
df.explain()

# Plano detalhado
df.explain(True)
```

### Particionamento
```python
# Escrita particionada
df.write \
    .partitionBy("ano", "mes") \
    .parquet("dados_particionados")

# Leitura seletiva
df = spark.read.parquet("dados_particionados/ano=2023")
```

### Caching
```python
# Cache DataFrame
df.cache()

# Persist com opções
from pyspark.StorageLevel
df.persist(StorageLevel.MEMORY_AND_DISK)
```

## Exemplos Práticos

### 1. Análise de Vendas
```python
# Carregar dados
vendas = spark.read.csv("vendas.csv", header=True)
produtos = spark.read.csv("produtos.csv", header=True)

# Análise
resultado = vendas.join(produtos, "produto_id") \
    .groupBy("categoria") \
    .agg(
        sum("valor").alias("total_vendas"),
        count("*").alias("num_vendas")
    ) \
    .orderBy("total_vendas", ascending=False)
```

### 2. ETL Pipeline
```python
def transform_data(df):
    from pyspark.sql.functions import col, upper, to_date
    
    return df \
        .withColumn("NOME", upper(col("nome"))) \
        .withColumn("DATA", to_date(col("data"), "yyyy-MM-dd")) \
        .dropDuplicates() \
        .na.fill(0)

# Pipeline
df_raw = spark.read.csv("raw_data.csv")
df_transformed = transform_data(df_raw)
df_transformed.write.parquet("processed_data")
```

### 3. Window Functions
```python
from pyspark.sql.window import Window
from pyspark.sql.functions import row_number, dense_rank

# Window spec
window_spec = Window \
    .partitionBy("departamento") \
    .orderBy("salario")

# Ranking
df.withColumn("rank", dense_rank().over(window_spec))
```

## Exercícios Práticos

1. **Manipulação de DataFrames**
   - Carregue dados de diferentes fontes
   - Aplique transformações
   - Salve em diferentes formatos

2. **Análise de Dados**
   - Use funções de agregação
   - Aplique window functions
   - Crie visualizações

3. **Otimização**
   - Compare planos de execução
   - Teste diferentes particionamentos
   - Avalie estratégias de caching

4. **Pipeline Completo**
   - Desenvolva ETL end-to-end
   - Implemente controle de qualidade
   - Otimize performance

## Recursos Adicionais

- [Spark SQL Guide](https://spark.apache.org/docs/latest/sql-programming-guide.html)
- [DataFrame API](https://spark.apache.org/docs/latest/api/python/reference/pyspark.sql/api/pyspark.sql.DataFrame.html)
- [Performance Tuning](https://spark.apache.org/docs/latest/sql-performance-tuning.html)
- [Best Practices](https://spark.apache.org/docs/latest/sql-getting-started.html) 