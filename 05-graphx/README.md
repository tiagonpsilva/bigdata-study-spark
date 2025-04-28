# GraphX

## Conceitos Básicos

### Estrutura de Grafos
- Vértices (Nodes)
- Arestas (Edges)
- Propriedades
- Direcionamento

### Criação de Grafos

1. **A partir de RDDs**
```python
from graphframes import GraphFrame
from pyspark.sql.types import *

# Vértices
vertices = spark.createDataFrame([
    ("1", "Alice", 34),
    ("2", "Bob", 36),
    ("3", "Charlie", 30)
], ["id", "name", "age"])

# Arestas
edges = spark.createDataFrame([
    ("1", "2", "friend"),
    ("2", "3", "colleague"),
    ("3", "1", "friend")
], ["src", "dst", "relationship"])

# Criar grafo
g = GraphFrame(vertices, edges)
```

2. **A partir de Arquivos**
```python
# Carregar vértices
vertices = spark.read.csv("vertices.csv", header=True)

# Carregar arestas
edges = spark.read.csv("edges.csv", header=True)

# Criar grafo
g = GraphFrame(vertices, edges)
```

## Algoritmos de Grafos

### 1. PageRank
```python
# Calcular PageRank
results = g.pageRank(resetProbability=0.15, maxIter=10)

# Mostrar resultados
results.vertices.select("id", "pagerank").show()
```

### 2. Connected Components
```python
# Encontrar componentes conectados
components = g.connectedComponents()

# Análise
components.select("id", "component").show()
```

### 3. Shortest Paths
```python
# Calcular caminhos mais curtos
paths = g.shortestPaths(landmarks=["1", "2"])

# Visualizar resultados
paths.select("id", "distances").show()
```

### 4. Triangle Count
```python
# Contar triângulos
triangles = g.triangleCount()

# Resultados
triangles.select("id", "count").show()
```

## Operadores

### 1. Transformações de Vértices
```python
# Filtrar vértices
filtered_vertices = g.vertices.filter("age > 30")

# Adicionar propriedades
with_degree = g.degrees

# Combinar resultados
results = filtered_vertices.join(with_degree, "id")
```

### 2. Transformações de Arestas
```python
# Filtrar arestas
friends = g.edges.filter("relationship = 'friend'")

# Reverter direção
reversed = g.reverse

# Subgrafo
subgraph = g.filterEdges("relationship = 'friend'")
```

## Exemplos Práticos

### 1. Análise de Rede Social
```python
from graphframes import GraphFrame
from pyspark.sql.functions import *

# Criar grafo social
social_vertices = spark.createDataFrame([
    ("1", "Alice", "NY"),
    ("2", "Bob", "SF"),
    ("3", "Charlie", "LA")
], ["id", "name", "city"])

social_edges = spark.createDataFrame([
    ("1", "2", "friend", 5),
    ("2", "3", "colleague", 2),
    ("3", "1", "friend", 4)
], ["src", "dst", "type", "weight"])

social_graph = GraphFrame(social_vertices, social_edges)

# Análise de influência
influence = social_graph.pageRank(resetProbability=0.15, maxIter=10)
communities = social_graph.labelPropagation(maxIter=5)

# Métricas
degrees = social_graph.degrees
in_degrees = social_graph.inDegrees
out_degrees = social_graph.outDegrees
```

### 2. Análise de Transações
```python
# Criar grafo de transações
tx_vertices = spark.createDataFrame([
    ("1", "Account1", 1000),
    ("2", "Account2", 2000),
    ("3", "Account3", 3000)
], ["id", "account", "balance"])

tx_edges = spark.createDataFrame([
    ("1", "2", "transfer", 500),
    ("2", "3", "transfer", 300),
    ("3", "1", "transfer", 200)
], ["src", "dst", "type", "amount"])

tx_graph = GraphFrame(tx_vertices, tx_edges)

# Detectar padrões suspeitos
motifs = tx_graph.find("(a)-[e]->(b); (b)-[e2]->(c)")
cycles = tx_graph.find("(a)-[e]->(b); (b)-[e2]->(c); (c)-[e3]->(a)")
```

### 3. Recomendação de Produtos
```python
# Grafo de produtos
product_vertices = spark.createDataFrame([
    ("1", "Product1", "Electronics"),
    ("2", "Product2", "Books"),
    ("3", "Product3", "Electronics")
], ["id", "name", "category"])

product_edges = spark.createDataFrame([
    ("1", "2", "bought_together", 10),
    ("2", "3", "bought_together", 5),
    ("1", "3", "similar", 8)
], ["src", "dst", "relationship", "weight"])

product_graph = GraphFrame(product_vertices, product_edges)

# Recomendações
similar_products = product_graph.find("(a)-[e]->(b); (b)-[e2]->(c)")
recommendations = product_graph.pageRank(resetProbability=0.15, maxIter=10)
```

## Visualização

### NetworkX Integration
```python
import networkx as nx
import matplotlib.pyplot as plt

# Converter para NetworkX
def to_networkx(g):
    # Coletar vértices e arestas
    vertices = g.vertices.toPandas()
    edges = g.edges.toPandas()
    
    # Criar grafo NetworkX
    G = nx.DiGraph()
    
    # Adicionar vértices
    for _, row in vertices.iterrows():
        G.add_node(row['id'], **row.to_dict())
    
    # Adicionar arestas
    for _, row in edges.iterrows():
        G.add_edge(row['src'], row['dst'], **row.to_dict())
    
    return G

# Visualizar
G = to_networkx(g)
plt.figure(figsize=(10,10))
nx.draw(G, with_labels=True)
plt.show()
```

## Exercícios Práticos

1. **Análise de Redes Sociais**
   - Implemente PageRank
   - Detecte comunidades
   - Analise influenciadores

2. **Detecção de Fraude**
   - Crie grafo de transações
   - Identifique padrões suspeitos
   - Implemente alertas

3. **Sistema de Recomendação**
   - Construa grafo de produtos
   - Implemente recomendações
   - Avalie resultados

4. **Visualização**
   - Use diferentes layouts
   - Adicione atributos visuais
   - Crie dashboard interativo

## Recursos Adicionais

- [GraphX Programming Guide](https://spark.apache.org/docs/latest/graphx-programming-guide.html)
- [GraphFrames User Guide](https://graphframes.github.io/graphframes/docs/_site/user-guide.html)
- [NetworkX Documentation](https://networkx.org/documentation/stable/)
- [Graph Algorithms](https://spark.apache.org/docs/latest/graphx-programming-guide.html#graph-algorithms)
- [Performance Tuning](https://spark.apache.org/docs/latest/graphx-programming-guide.html#optimized-representation) 