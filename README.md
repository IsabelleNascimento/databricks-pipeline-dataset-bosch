# databricks-pipeline-dataset-bosch
# Pipeline de Dados em Arquitetura Lakehouse para Sensores Industriais

Pipeline de dados construído com **PySpark** e **Delta Lake**, seguindo a **Arquitetura Medalhão** (Bronze → Silver → Gold), para curadoria e avaliação de dados de telemetria industrial de alta esparsidade.

Trabalho de Conclusão de Curso do **MBA em Engenharia de Dados (MBED) — Escola Politécnica / ITLab, UFRJ**.

---

## Sobre o projeto

Dados brutos de sensores industriais, ingeridos sem tratamento, sofrem do efeito *Garbage In, Garbage Out*: leituras inconsistentes, alta esparsidade e baixa completude comprometem a confiabilidade da operação. Este projeto desenvolve e avalia experimentalmente um pipeline Lakehouse capaz de:

- Preservar os dados brutos de forma imutável e auditável (**Bronze**)
- Reduzir a dimensionalidade removendo atributos altamente esparsos e sem variabilidade estatística (**Silver**)
- Consolidar indicadores de qualidade informacional para consumo analítico (**Gold**)

sem perder nenhum registro do dataset original.

## Arquitetura

```mermaid
flowchart LR
    A[Bronze<br/>Ingestão imutável] --> B[Silver<br/>Curadoria e saneamento] --> C[Gold<br/>Indicadores consolidados]
```

| Camada | Função | Regra principal |
|---|---|---|
| **Bronze** | Ingestão imutável dos dados brutos, com metadados de auditoria (timestamp, fonte) | Sem transformação de conteúdo |
| **Silver** | Remoção de colunas com baixa densidade informacional | Descarta colunas com >95% de valores nulos **ou** sem variabilidade (desvio padrão = 0). `Id` e `Response` são preservadas incondicionalmente (rastreabilidade e variável-alvo) |
| **Gold** | Consolidação de KPIs analíticos | Completude, cardinalidade (`approx_count_distinct`) e métricas de negócio |

## Dataset

[Bosch Production Line Performance](https://www.kaggle.com/c/bosch-production-line-performance) (`train_numeric.csv`)

- 1.183.747 registros
- 970 atributos originais de sensores
- Mais de 1,1 bilhão de células de dados (matriz completa)

## Ambiente e stack tecnológico

| Item | Versão / especificação |
|---|---|
| Processamento | Apache Spark / PySpark 4.2.0 |
| Armazenamento transacional | Delta Lake |
| Plataforma | Databricks Serverless (Driver Node Mode) |
| Infraestrutura | Linux ARM64, 4 vCPUs, 15,28 GB RAM |
| Runtime | OpenJDK 17.0.18 LTS (Zulu) · Python 3.12.3 |

> Execução em **nó único** — o paralelismo do Spark ocorreu via concorrência multithread dentro de uma única máquina, não em um cluster distribuído multi-node.

## Como executar

1. Disponibilizar o dataset `train_numeric.csv` em um Volume do Unity Catalog (ou ajustar `input_path` para o ambiente utilizado)
2. Executar `01_pipeline_bronze_silver_gold_-_OFICIAL02.ipynb` em um cluster/serverless Databricks com suporte a Delta Lake
3. As células rodam em sequência: Bronze → baseline de completude → Silver → Gold → telemetria consolidada → auditoria de ambiente

**Protocolo experimental:** cada camada é medida com 1 execução de *warm-up* (descartada) seguida de **N = 3** execuções oficiais; os tempos reportados são a média aritmética simples.

## Resultados

| Indicador | Baseline (Bronze) | Após curadoria (Silver/Gold) |
|---|---|---|
| Completude | 19,08% | 37,49% |
| Dimensionalidade (colunas) | 970 | 478 |
| Retenção de registros (linhas) | 100% | 100% |
| Duplicidade | 0% | 0% |
| Latência por camada | 20,26 s | 14,87 s (Silver) / 13,44 s (Gold) |
| Throughput por camada | 58.436,64 reg/s | 79.604,38 reg/s (Silver) / 88.096,79 reg/s (Gold) |
| Cardinalidade média (Gold) | — | 2.836,60 valores distintos/coluna |

**Desempenho consolidado do pipeline completo:** latência end-to-end de **48,56 s** e throughput de **24.374,89 registros/segundo**.

> A completude de 19,08% não significa que 81% dos dados sejam inutilizáveis — reflete a ativação de sensores em etapas específicas da linha de produção, uma característica esperada de dados industriais esparsos. A curadoria da Silver atua sobre a dimensionalidade (colunas), nunca sobre os registros (linhas).

## Limitações

- Execução em nó único (Databricks Serverless) — não valida escalabilidade em cluster distribuído multi-node
- Processamento em batch, sem ingestão contínua (streaming) em tempo real
- Resultados obtidos em ambiente controlado, com amostra do dataset Bosch — não generalizáveis à operação industrial real
- Consistência de domínio (faixas de valores válidos por sensor) não instrumentada nesta versão

## Trabalhos futuros

- Execução em cluster distribuído real
- Streaming (Kafka / Spark Structured Streaming)
- Detecção de anomalias com Machine Learning
- Dashboards operacionais (Power BI / Grafana)
- Avaliação comparativa de custos computacionais entre arquiteturas (DW, Data Lake, Lakehouse)
- Definição e mensuração de regras de consistência de domínio

## Autoria

**Isabelle Maria Lima do Nascimento**
MBA em Engenharia de Dados (MBED) — Escola Politécnica / ITLab, UFRJ
Orientador: Prof. Me. Cláudio Luiz Latta de Souza
Coorientador: Manoel Villas Boas Junior, M. Sc.

## Licença

Uso acadêmico. Consulte a instituição antes de reutilização comercial ou redistribuição do dataset original (Bosch, via Kaggle).
