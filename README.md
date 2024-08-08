# 🏞️ DataLake Big Data Lab

Bem-vindo ao **DataLake Big Data Lab**! 🎉 Este projeto é um ambiente de laboratório pronto para uso, projetado para fornecer uma plataforma de análise de big data usando Hadoop, Spark e Trino. Aqui, você encontrará instruções detalhadas sobre como configurar e usar esse ambiente incrível.

## Ferramentas Incluídas

- **[Apache Hadoop](https://hadoop.apache.org/)**: O backbone do nosso sistema de arquivos distribuídos e processamento em larga escala.
- **[Apache Spark](https://spark.apache.org/)**: O motor de processamento de dados rápidos para análise em tempo real.
- **[Trino (Presto)](https://trino.io/)**: O motor de consulta SQL distribuído para explorar seus dados rapidamente.

## 🚀 Como Configurar

### Pré-requisitos

- [Docker](https://www.docker.com/get-started)
- [Docker Compose](https://docs.docker.com/compose/install/)

### Passo a Passo

1. **Clone o repositório:**

```bash
git clone https://github.com/seu-usuario/lakehouse-big-data-lab.git
cd lakehouse-big-data-lab
```

2. **Configure o ambiente:**

   Certifique-se de que todas as variáveis de ambiente estão corretamente configuradas no arquivo `./hadoop/config/config`.

3. **Inicie o ambiente:**

```bash
docker-compose up -d
```

4. **Verifique se tudo está rodando:**

   - Acesse a interface do NameNode do Hadoop: [http://localhost:9870](http://localhost:9870)
   - Acesse a interface do ResourceManager do Hadoop: [http://localhost:8088](http://localhost:8088)
   - Acesse a interface do Spark Master: [http://localhost:8080](http://localhost:8080)
   - Acesse a interface do Spark Worker: [http://localhost:8081](http://localhost:8081)
   - Acesse a interface do Trino: [http://localhost:8082](http://localhost:8082)

## 🛠️ Estrutura do Projeto

```plaintext
datalake-big-data-lab/
│
├── hadoop/
│   ├── config/
│   │   └── config
│   ├── namenode/
│   └── datanode/
│
├── spark/
│   ├── master/
│   └── worker/
│
├── trino/
│   ├── config/
│   ├── catalog/
│   └── data/
│
├── docker-compose.yml
└── README.md
```

## 🧰 Configuração dos Serviços

### Hadoop NameNode

O NameNode gerencia o armazenamento distribuído no HDFS.

```yaml
hadoop-namenode:
  image: apache/hadoop:3.3.5
  hostname: hadoop-namenode
  command: ["hdfs", "namenode"]
  ports:
    - 9870:9870 # Node Manager UI
  env_file:
    - ./hadoop/config/config
  environment:
    ENSURE_NAMENODE_DIR: "/tmp/hadoop-root/dfs/name"
  volumes:
    - type: bind
      source: ./hadoop/namenode
      target: /etc/hadoop
  networks:
    - lakehouse-net
```

### Hadoop DataNode

O DataNode armazena os dados reais.

```yaml
hadoop-datanode:
  image: apache/hadoop:3.3.5
  command: ["hdfs", "datanode"]
  env_file:
    - ./hadoop/config/config
  networks:
    - lakehouse-net
```

### Hadoop ResourceManager

Gerencia recursos e alocação de tarefas no cluster.

```yaml
hadoop-resourcemanager:
  image: apache/hadoop:3.3.5
  hostname: resourcemanager
  command: ["yarn", "resourcemanager"]
  ports:
    - 8088:8088 # Hadoop Cluster Manager UI
  env_file:
    - ./hadoop/config/config
  volumes:
    - ./test.sh:/opt/test.sh
  networks:
    - lakehouse-net
```

### Hadoop NodeManager

Executa tarefas no cluster.

```yaml
hadoop-nodemanager:
  image: apache/hadoop:3.3.5
  command: ["yarn", "nodemanager"]
  env_file:
    - ./hadoop/config/config
  networks:
    - lakehouse-net
```

### Spark Master

Gerencia o cluster Spark.

```yaml
spark-master:
  image: bitnami/spark:latest
  container_name: spark-master
  hostname: spark-master
  ports:
    - "8080:8080"  # Spark Web UI
    - "7077:7077"  # Spark Master
  environment:
    - SPARK_MODE=master
    - SPARK_MASTER_URL=spark://spark-master:7077
  volumes:
    - ./spark/master/data:/bitnami/spark
    - ./hadoop/config:/etc/hadoop/conf
  networks:
    - lakehouse-net
  depends_on:
    - hadoop-namenode
```

### Spark Worker

Executa tarefas de processamento distribuído.

```yaml
spark-worker:
  image: bitnami/spark:latest
  container_name: spark-worker
  hostname: spark-worker
  ports:
    - "8081:8081"  # Spark Worker UI
  environment:
    - SPARK_MODE=worker
    - SPARK_MASTER_URL=spark://spark-master:7077
  volumes:
    - ./spark/worker/data:/bitnami/spark
    - ./hadoop/config:/etc/hadoop/conf
  networks:
    - lakehouse-net
  depends_on:
    - spark-master
```

### Trino

Motor de consulta SQL distribuído para grandes volumes de dados.

```yaml
trino:
  image: trinodb/trino:latest
  container_name: trino
  hostname: trino
  ports:
    - "8082:8080"  # Trino Web UI
  environment:
    - JAVA_TOOL_OPTIONS=-Dnode.environment=production -Dnode.id=trino -Dnode.data-dir=/data
  volumes:
    - ./trino/config:/etc/trino
    - ./trino/catalog:/etc/trino/catalog
    - ./trino/data:/data
  networks:
    - lakehouse-net
  depends_on:
    - hadoop-namenode
```

## 🔗 Links Úteis

- [Docker](https://www.docker.com/)
- [Docker Compose](https://docs.docker.com/compose/)
- [Apache Hadoop](https://hadoop.apache.org/)
- [Apache Spark](https://spark.apache.org/)
- [Trino (Presto)](https://trino.io/)

## 📚 Fontes

- [Documentação do Hadoop](https://hadoop.apache.org/docs/)
- [Documentação do Spark](https://spark.apache.org/docs/latest/)
- [Documentação do Trino](https://trino.io/docs/current/)

Divirta-se explorando o mundo dos dados! 🌐📊🚀
