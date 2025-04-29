# 🧠 Estudo do Apache Spark

Este repositório contém explicações detalhadas e exemplos práticos dos conceitos fundamentais do Apache Spark, com foco em processamento distribuído de dados em larga escala.

## 📋 Índice de Conceitos

1. **🔥 [Conceitos Básicos](./01-conceitos-basicos/README.md)** - Fundamentos, arquitetura, RDDs e contextos do Spark
2. **📊 [Spark SQL](./02-spark-sql/README.md)** - Processamento estruturado de dados com DataFrames e SQL
3. **🌊 [Spark Streaming](./03-spark-streaming/README.md)** - Processamento de dados em tempo real e integração com Kafka
4. **🤖 [MLlib](./04-mllib/README.md)** - Machine Learning distribuído com algoritmos e pipelines
5. **🕸️ [GraphX](./05-graphx/README.md)** - Processamento e análise de grafos em larga escala
6. **⚡ [Otimização](./06-otimizacao/README.md)** - Tunning, particionamento e monitoramento
7. **💡 [Exemplos](./07-exemplos/README.md)** - Cases práticos e integrações

## 🌟 Objetivo

Este repositório tem como objetivo proporcionar um entendimento prático do Apache Spark, com explicações claras e exemplos de casos de uso reais. Cada conceito é explorado em detalhes, com código funcional e boas práticas.

## ⚙️ Pré-requisitos

- Java 8 ou superior
- Python 3.7+
- Docker e Docker Compose
- Conhecimento básico de SQL
- Familiaridade com processamento distribuído

## 🚀 Como Usar

1. Clone o repositório
```bash
git clone https://github.com/tiagonpsilva/bigdata-study-spark.git
```

2. Navegue até o diretório de exemplos
```bash
cd bigdata-study-spark/exemplos
```

3. Execute o ambiente de laboratório
```bash
docker-compose up -d
```

4. Acesse:
   - Spark UI: http://localhost:8080
   - Jupyter: http://localhost:8888
   - History Server: http://localhost:18080

## 🐳 Ambiente de Desenvolvimento

### Docker Compose

```yaml
version: '3.8'

services:
  spark-master:
    image: bitnami/spark:3.4
    environment:
      - SPARK_MODE=master
    ports:
      - "8080:8080"
      - "7077:7077"
    
  spark-worker:
    image: bitnami/spark:3.4
    environment:
      - SPARK_MODE=worker
      - SPARK_MASTER_URL=spark://spark-master:7077
    depends_on:
      - spark-master

  jupyter:
    image: jupyter/pyspark-notebook:latest
    ports:
      - "8888:8888"
    volumes:
      - ./notebooks:/home/jovyan/work
    environment:
      - SPARK_OPTS="--master=spark://spark-master:7077"
    depends_on:
      - spark-master

  history-server:
    image: bitnami/spark:3.4
    environment:
      - SPARK_MODE=master
    ports:
      - "18080:18080"
    command: /opt/bitnami/spark/sbin/start-history-server.sh
    volumes:
      - spark-logs:/opt/bitnami/spark/logs

volumes:
  spark-logs:
```