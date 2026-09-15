# Etapa 1 — Arquitetura de Big Data

## 1. Visão geral da arquitetura

A arquitetura proposta para a TechPay foi desenvolvida com o objetivo de organizar o fluxo dos dados desde sua origem até a etapa de análise e tomada de decisão. O pipeline segue uma arquitetura em camadas, composta por **Origem dos Dados → Ingestão → RAW → Bronze → Silver → Gold → Análise/Dashboard**.

As fontes de dados representam os diferentes canais utilizados pela TechPay, como **App, Web, POS e ATM**, responsáveis pela geração das transações. Para a avaliação proposta, os dados são disponibilizados em arquivos CSV, que representam a entrada do pipeline.

A arquitetura foi implementada utilizando ferramentas que permitem reproduzir o fluxo de Big Data em ambiente local, sem a necessidade de administrar um cluster distribuído. Para isso, são utilizados **Python/Pandas na ingestão, PySpark nas etapas de processamento, DuckDB para consultas analíticas e Plotly na visualização dos resultados**.

Essa abordagem permite demonstrar os principais conceitos de uma arquitetura de Big Data, como separação em camadas, processamento distribuído, armazenamento colunar, particionamento e preparação de dados para análise, mantendo o ambiente simples e reproduzível.

---

## 2. Ingestão e camada RAW

A primeira etapa do pipeline é a **ingestão dos dados**. Nessa etapa, os arquivos CSV disponibilizados pela TechPay são carregados para o ambiente de processamento utilizando Python e Pandas. O objetivo da ingestão é receber os dados e realizar apenas as validações iniciais necessárias, preservando o conteúdo original.

Após a ingestão, os dados são armazenados na camada **RAW**. Essa camada mantém os dados em seu formato original, utilizando **CSV**, pois esse é o formato de entrada fornecido pela fonte.

A escolha de manter uma camada RAW sem transformações é importante para garantir **rastreabilidade e possibilidade de reprocessamento**. Caso alguma regra de transformação aplicada nas etapas posteriores esteja incorreta, os dados originais continuam disponíveis para que o pipeline possa ser executado novamente.

Para a solução proposta, a ingestão por arquivos CSV foi escolhida porque atende ao cenário de avaliação, que trabalha com processamento em **batch**. Uma alternativa mais robusta para uma plataforma de produção seria utilizar uma ferramenta como **Apache Kafka**, especialmente caso os dados fossem recebidos continuamente como eventos.

Entretanto, Kafka não foi utilizado neste projeto porque adicionaria uma camada de infraestrutura desnecessária para o cenário atual. O objetivo da avaliação é demonstrar o processamento dos dados fornecidos em arquivos, e não implementar uma plataforma de streaming. A utilização de Kafka faria mais sentido em um cenário em que a TechPay precisasse processar transações continuamente e tomar decisões em poucos segundos.

---

## 3. Camada Bronze

A camada **Bronze** é responsável pela primeira etapa efetiva de tratamento dos dados. Nessa camada, os dados são processados utilizando **PySpark**, permitindo demonstrar o uso de uma ferramenta adequada para processamento de grandes volumes.

Entre as principais operações realizadas estão a **deduplicação dos registros, conversão e padronização dos tipos de dados e tratamento de valores impossíveis ou inconsistentes**.

Após o processamento, os dados são armazenados em **Parquet**. Diferentemente do CSV utilizado na camada RAW, o Parquet é um formato colunar mais adequado para processamento analítico. Ele permite armazenar os dados de forma mais eficiente e ler somente as colunas necessárias durante uma consulta.

A separação entre RAW e Bronze também permite preservar a diferença entre o dado recebido da origem e o dado que já passou por regras de qualidade. Dessa forma, a RAW representa o dado original, enquanto a Bronze representa uma versão limpa e estruturada.

---

## 4. Camada Silver

Na camada **Silver**, os dados já tratados na Bronze recebem informações adicionais necessárias para as análises de negócio.

O processamento continua sendo realizado com **PySpark**, e os resultados permanecem armazenados em **Parquet**. Nessa etapa são criadas colunas derivadas a partir dos dados existentes, como **faixa de valor da transação e período do dia**, além de outras transformações e regras de negócio necessárias para a análise de risco.

Essa camada tem como objetivo transformar dados simplesmente estruturados em dados **enriquecidos e preparados para análises mais específicas**.

A utilização do Parquet também é importante nessa etapa porque as consultas analíticas normalmente trabalham com subconjuntos de colunas e grandes quantidades de registros. O formato colunar reduz a quantidade de dados que precisa ser lida, tornando o processamento mais eficiente.

---

## 5. Estratégia de particionamento

A estratégia de particionamento utilizada nas camadas de dados tratadas é baseada nas colunas **ano e mês**, obtidas a partir do campo `timestamp` das transações.

Assim, os dados são organizados logicamente em partições semelhantes a:

```text
ano=2025/
    mes=01/
    mes=02/
    mes=03/
```

Essa estratégia foi escolhida porque o tempo é uma dimensão natural para análise de transações financeiras. Além disso, consultas de negócio frequentemente possuem filtros temporais.

Por exemplo, uma consulta como:

> "Qual foi a quantidade e o percentual de transações suspeitas realizadas durante o mês de agosto?"

não precisa necessariamente processar todo o histórico disponível. Com os dados particionados por ano e mês, o mecanismo de consulta pode acessar somente a partição correspondente ao período solicitado.

A granularidade de **ano/mês** representa um equilíbrio entre organização e quantidade de partições. Utilizar apenas o ano poderia resultar em partições muito grandes, enquanto utilizar uma granularidade muito pequena, como ano/mês/dia, poderia gerar uma quantidade excessiva de arquivos e aumentar o custo de gerenciamento.

---

## 6. Camada Gold e Serving/BI

A camada **Gold** representa os dados preparados para consumo analítico. Nessa etapa são criadas tabelas agregadas que facilitam a interpretação dos dados e reduzem a necessidade de executar novamente todo o processamento das camadas anteriores.

Entre os exemplos de informações disponibilizadas estão as **métricas de risco por canal** e as **agregações relacionadas ao comportamento temporal das transações**.

O processamento pode utilizar **PySpark**, enquanto o **DuckDB** é utilizado para consultas analíticas sobre os dados estruturados. Os resultados da camada Gold ficam preparados para serem consumidos pela etapa de análise.

A última etapa do fluxo é a **Análise/Dashboard**, realizada utilizando recursos de análise exploratória e visualizações interativas com **Plotly**. O objetivo é transformar as informações agregadas em indicadores que permitam identificar padrões de comportamento, diferenças entre canais e características relacionadas ao risco de fraude.

Dessa forma, a camada Gold funciona como uma camada de serving analítico, entregando dados mais simples e eficientes para o consumo pelo dashboard.

---

## 7. Pontos de falha e crescimento de volume

Um dos principais pontos de atenção da arquitetura é o aumento significativo do volume de dados. Caso a quantidade de dados da TechPay aumentasse em **100 vezes**, o primeiro problema provavelmente seria a limitação do processamento local, principalmente em relação à **memória, CPU e armazenamento**.

A própria etapa de ingestão baseada em arquivos CSV também poderia se tornar um gargalo de entrada e saída. Além disso, o crescimento do número de arquivos Parquet exigiria uma estratégia adequada de gerenciamento e particionamento para evitar partições muito pequenas ou excessivamente grandes.

A arquitetura atual foi planejada para o ambiente da avaliação e, portanto, utiliza processamento local. Em um ambiente de produção com crescimento significativo, seria necessário evoluir para uma infraestrutura distribuída. Entre as possíveis medidas estão a utilização de **armazenamento distribuído ou cloud, ingestão incremental, execução do Spark em cluster e maior paralelização do processamento**.

O particionamento por ano/mês também precisaria ser monitorado. Caso determinados períodos concentrassem um volume muito maior de transações, poderia ser necessário revisar a estratégia de particionamento para evitar partições desbalanceadas.

---

## 8. Evolução para detecção de fraude em tempo real

Caso a TechPay solicitasse **detecção de fraude em tempo real**, o principal impacto seria na camada de ingestão e processamento.

O fluxo atual é orientado a **batch**, no qual os dados são disponibilizados em arquivos e processados posteriormente. Para uma solução de tempo real, seria necessário introduzir uma plataforma de streaming, como **Apache Kafka**, para receber continuamente os eventos de transação.

O fluxo poderia evoluir para:

```text
App / Web / POS / ATM
          ↓
        Kafka
          ↓
Spark Structured Streaming
          ↓
Regras / Modelo de Detecção de Fraude
          ↓
Alerta / Decisão em Tempo Real
```

Nesse cenário, o pipeline batch atual não precisaria necessariamente ser eliminado. Os dois fluxos poderiam coexistir: o **streaming** seria responsável pela tomada de decisão imediata, enquanto o pipeline batch continuaria sendo utilizado para armazenamento histórico, análises, treinamento de modelos e dashboards.

Essa evolução permitiria que uma transação fosse analisada poucos instantes após sua ocorrência, possibilitando ações como bloqueio, alerta ou solicitação de validação adicional ao cliente.

---

## Conclusão

A arquitetura proposta organiza o fluxo de dados da TechPay em camadas com responsabilidades bem definidas. A **RAW** preserva os dados originais, a **Bronze** realiza a limpeza e padronização, a **Silver** enriquece os dados com informações derivadas e regras de negócio, e a **Gold** disponibiliza informações agregadas para análise.

A utilização de **CSV na RAW** e **Parquet nas camadas processadas** permite preservar os dados de origem e, ao mesmo tempo, utilizar um formato mais adequado para consultas analíticas. O particionamento por **ano/mês** melhora o acesso a consultas temporais, que são comuns em análises de transações.

Por fim, a arquitetura foi construída pensando no cenário batch da avaliação, mas também permite identificar claramente como a solução poderia evoluir para um ambiente de produção de maior escala e para **detecção de fraude em tempo real**, através da introdução de tecnologias de streaming e processamento distribuído.
