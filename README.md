# TechPay — Projeto Big Data

Projeto desenvolvido para a avaliação da disciplina de Big Data, contemplando a execução dos Labs 1 a 12 e as etapas da Atividade Final.

A implementação utiliza uma abordagem local, sem necessidade de configuração de cluster Hadoop/HDFS/Hive, utilizando principalmente **Python, PySpark, DuckDB, Parquet e Plotly**.

---

## 1. Pré-requisitos

Para executar o projeto, é necessário ter instalado:

* Python 3.11+;
* VS Code ou outra IDE compatível com notebooks Jupyter;
* extensão **Jupyter** do VS Code;
* extensão **Python** do VS Code.

Não é necessário instalar ou configurar:

* Hadoop;
* HDFS;
* Hive;
* Spark Cluster;
* YARN;
* Docker.

A execução foi preparada para ocorrer localmente.

---

## 2. Estrutura do projeto

Antes de iniciar a execução, a estrutura básica deve estar semelhante a:

```text
bigdata-curso-vitoria-e-julio/
│
├── avaliacao_final/ ...
├── content/
│   └──bigdata/.gitkeep ...
├── dia1_fundamentos/ ...
├── dia2_transformacao/ ...
├── dia3_insights_bi/ ...
├── customers_synthetic.csv
├── fraud_labels.csv
├── transactions_synthetic.csv
├── .gitignore
├── README.md
└── ...
```

### Arquivos de dados obrigatórios

Os três arquivos abaixo **devem ser colocados manualmente na raiz do projeto antes da primeira execução**:

```text
customers_synthetic.csv
fraud_labels.csv
transactions_synthetic.csv
```

Esses arquivos não estão versionados no Git.

As bases, arquivos Parquet e demais artefatos gerados durante a execução estão no `.gitignore`, portanto **não é necessário criar manualmente as pastas de dados antes de executar o projeto**.

Os notebooks/códigos são responsáveis pela criação dos diretórios e arquivos necessários durante o processamento.

---

# 3. Instalação das dependências

Caso ainda não tenha, é necessário instalar o Java e configurar a variável JAVA_HOME
Algumas libs são instaladas conforme execução dos notebooks
---

# 4. Execução dos Labs

A execução dos Labs deve respeitar a ordem numérica.

Execute:

```text
Lab 01
  ↓
Lab 02
  ↓
Lab 03
  ↓
Lab 04
  ↓
Lab 05
  ↓
Lab 06
  ↓
Lab 07
  ↓
Lab 08
  ↓
Lab 09
  ↓
Lab 10
  ↓
Lab 11
  ↓
Lab 12
```

### Importante

Os Labs possuem dependências entre si e alguns deles utilizam arquivos ou estruturas produzidas nas etapas anteriores.

Por isso, **não é recomendado executar os notebooks fora da ordem**.

Caso seja necessário reiniciar a avaliação do zero, recomenda-se reiniciar o ambiente e executar novamente desde o Lab 01.

---

# 5. Execução da Avaliação Final

Após concluir os Labs 1 a 12, execute a **Avaliação Final**.

A avaliação final está organizada nas etapas previstas na atividade:

### Etapa 1 — Arquitetura

Apresentação da arquitetura proposta para o pipeline de dados, incluindo as camadas de processamento e armazenamento.

A solução considera o fluxo:

```text
Fontes de dados
      ↓
    Ingestão
      ↓
     RAW
      ↓
    BRONZE
      ↓
    SILVER
      ↓
     GOLD
      ↓
Análise / Dashboard
```

### Etapa 2 — Big Data

Execução do pipeline de processamento dos dados, contemplando:

* ingestão do dataset de avaliação;
* criação da camada RAW;
* definição de tipos e estrutura dos dados;
* estratégia de particionamento;
* tratamento e deduplicação na camada Bronze;
* transformação e enriquecimento na camada Silver;
* criação das tabelas agregadas na camada Gold.

### Etapa 3 — Análise e Dashboard

A etapa final utiliza os dados produzidos pelo pipeline para realizar a análise exploratória e apresentar os resultados por meio de um dashboard interativo.

---

# 6. Dataset da Avaliação Final

O arquivo utilizado na avaliação final é:

```text
avaliacao_transactions.csv
```

Esse arquivo é utilizado pelo pipeline da avaliação para realizar as etapas de ingestão, tratamento, transformação e agregação.

Os dados processados durante a execução são armazenados nas estruturas criadas pelo projeto.

Como essas estruturas estão no `.gitignore`, elas serão recriadas durante a execução do projeto.

---

# 7. Armazenamento dos dados

A solução utiliza arquivos **Parquet** para armazenamento das camadas do pipeline.

A organização segue o conceito de Data Lake em camadas:

```text
RAW
BRONZE
SILVER
GOLD
```

As camadas RAW, Bronze e Silver utilizam particionamento temporal por:

```text
year
month
```

A estrutura gerada será semelhante a:

```text
bigdata/
│
├── raw/
│   └── parquet/
│       ├── year=2025/
│       │   ├── month=1/
│       │   ├── month=2/
│       │   └── ...
│       │
│       └── year=2026/
│
├── bronze/
│   ├── year=2025/
│   └── year=2026/
│
├── ...
│
└── gold/
    ├── risk_by_channel.parquet
    ├── risk_by_category.parquet
    ├── daily_risk.parquet
    └── risk_level.parquet
```

Esses diretórios e arquivos são gerados durante a execução e não fazem parte do repositório Git.

---

# 8. DuckDB e PySpark

O projeto utiliza **PySpark** para as operações de processamento distribuído/local previstas na atividade e **DuckDB** para operações relacionadas ao armazenamento e consulta dos arquivos Parquet.

A execução foi adaptada para ambiente local, portanto **não é necessário configurar HDFS ou Hadoop**.

Em especial, a persistência dos dados em Parquet é realizada utilizando DuckDB nas etapas em que o Spark dependeria da infraestrutura Hadoop para escrita no Windows.

---

# 9. Execução do Dashboard

Após executar todas as etapas da Avaliação Final, será gerado o arquivo HTML do dashboard.

O arquivo HTML deve ser aberto no navegador para visualizar a análise interativa.

No caminho:

```text
avaliacao_final\etapa3_analise\dashboard_techpay.html
```

No VS Code, localize o arquivo HTML gerado e:

**Clique com o botão direito → Open with Default Browser**

ou abra o arquivo diretamente pelo Windows.

O dashboard utiliza gráficos interativos, permitindo explorar os resultados da análise.

> **Importante:** o dashboard deve ser aberto somente após a execução da etapa responsável por sua geração, pois ele depende dos dados produzidos pelas etapas anteriores.

---

# 10. Ordem completa para avaliação

Para avaliar o projeto do zero, siga exatamente esta sequência:

### 1. Clonar/abrir o projeto

Abra a pasta do projeto no VS Code.

### 2. Inserir os datasets

Coloque na **raiz do projeto**:

```text
customers_synthetic.csv
fraud_labels.csv
transactions_synthetic.csv
```

### 3. Criar o ambiente Python

```text
Instalar e configurar Java ou alguma outra depencencia da máquina local
```

### 4. Executar os Labs

Execute na ordem:

```text
Lab 01 → Lab 02 → Lab 03 → ... → Lab 12
```

### 5. Executar a Avaliação Final

Execute as células da avaliação final na ordem apresentada no notebook.

### 6. Aguardar a geração dos artefatos

Durante a execução serão criadas as estruturas de dados e arquivos Parquet necessários para as etapas seguintes.

### 7. Visualizar o Dashboard

Após concluir a avaliação final, abra o arquivo:

```text
avaliacao_final\etapa3_analise\dashboard_techpay.html
```

no navegador.

---

# 8. Em caso de nova execução

Caso seja necessário executar o projeto novamente desde o início, os dados e artefatos gerados podem ser recriados pelos notebooks.

Recomenda-se:

1. reiniciar o kernel do Jupyter;
2. executar novamente os Labs desde o Lab 01;
3. seguir a ordem até o Lab 12;
4. executar a Avaliação Final;
5. abrir o novo HTML do dashboard.

Os arquivos CSV originais devem permanecer na raiz do projeto.

---

# 9. Checklist para avaliação

Antes de iniciar:

* [ ] `customers_synthetic.csv` está na raiz;
* [ ] `fraud_labels.csv` está na raiz;
* [ ] `transactions_synthetic.csv` está na raiz;
* [ ] Python está instalado;
* [ ] Java está instalado;
* [ ] JAVA_HOME está configurado;
* [ ] interpretador correto está selecionado no VS Code.

Execução:

* [ ] Lab 01 executado;
* [ ] Lab 02 executado;
* [ ] Lab 03 executado;
* [ ] Lab 04 executado;
* [ ] Lab 05 executado;
* [ ] Lab 06 executado;
* [ ] Lab 07 executado;
* [ ] Lab 08 executado;
* [ ] Lab 09 executado;
* [ ] Lab 10 executado;
* [ ] Lab 11 executado;
* [ ] Lab 12 executado;
* [ ] Avaliação Etapa 1 executada;
* [ ] Avaliação Etapa 2 executada;
* [ ] Avaliação Etapa 3 executada;
* [ ] Dashboard HTML gerado;
* [ ] Dashboard aberto no navegador.

---

## Observação final

Os arquivos de dados e artefatos gerados não estão presentes no repositório porque estão configurados no `.gitignore`.

Portanto, **a presença dos três arquivos CSV na raiz do projeto é obrigatória para uma execução completa desde o início**:

```text
customers_synthetic.csv
fraud_labels.csv
transactions_synthetic.csv
```

Após esses arquivos serem disponibilizados, basta seguir a ordem dos Labs e, posteriormente, executar a Avaliação Final para reproduzir todo o pipeline e visualizar o dashboard.

