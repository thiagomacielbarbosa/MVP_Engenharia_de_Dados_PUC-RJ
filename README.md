# MVP de Engenharia de Dados: BirdBase Analytics 🐦

Este projeto apresenta um pipeline completo de Engenharia de Dados (**End-to-End**) desenvolvido como MVP para a disciplina de Engenharia de Dados (PUC-Rio). O sistema ingere dados científicos complexos sobre a biodiversidade aviária global, normaliza as estruturas e disponibiliza um **Data Warehouse** analítico para estudos de conservação.

## 📋 Índice

- [Objetivo e Problema de Negócio](#-objetivo-e-problema-de-negócio)
- [Arquitetura da Solução](#-arquitetura-da-solução)
- [Modelagem de Dados](#-modelagem-de-dados)
- [Pipeline ETL](#-pipeline-etl)
- [Resultados e Análises](#-resultados-e-análises)
- [Estrutura do Repositório](#-estrutura-do-repositório)
- [Como Executar](#-como-executar)

---

## 🎯 Objetivo e Problema de Negócio

- **Contexto:** A base de dados **BirdBase** (Sekercioglu et al.) contém informações vitais sobre ecologia, história de vida e risco de extinção de mais de 11.000 espécies de aves. No entanto, os dados originais são disponibilizados em planilhas Excel desnormalizadas, com cabeçalhos complexos ("Double Headers") e legendas em texto livre.
- **O Problema:** Como estruturar esses dados para permitir análises rápidas sobre a correlação entre características biológicas (massa, dieta, migração) e o risco de extinção?
- **A Solução:** Construção de um pipeline ETL no **Databricks** que baixa os dados da fonte oficial, trata inconsistências de formato e modela as informações em um **Esquema Estrela (Star Schema)**.

---

## 🏗 Arquitetura da Solução

O projeto segue a arquitetura **Lakehouse** (Bronze → Silver → Gold) simplificada para o MVP.

```mermaid
graph LR
    A[Fonte: Springer Nature / Figshare] -->|Ingestão Python + Pandas| B(Bronze: Dados Brutos em Memória)
    B -->|PySpark: Limpeza e Tipagem| C(Silver: Staging Area)
    C -->|Modelagem Dimensional| D(Gold: Tabelas Delta)
    D -->|SQL Analytics| E[Dashboards e Insights]
```

- **Fonte:** Arquivo `BirdBase_Final.xlsx` (baixado programaticamente via script).
- **Processamento:** Cluster Databricks Community (Spark 3.x).
- **Bibliotecas:** `pyspark`, `pandas`, `openpyxl`, `requests`.
- **Armazenamento:** Delta Lake.
- **inteligência Artificial:** Uso do `Google Gemini` para auxilio em organização e elaboração de códigos

---

## 🧩 Modelagem de Dados

Foi adotado o modelo dimensional **Star Schema** para facilitar consultas analíticas performáticas.

### Tabela Fato

- **`FATO_METRICAS_AVES`**: Contém as métricas numéricas (Massa, Altitude, Tamanho de Ninhada).

### Tabelas Dimensão

- **`DIM_ESPECIE`**: Taxonomia básica (Nome Inglês, Latim, ID Original).
- **`DIM_CONSERVACAO`**: Status IUCN extraído dinamicamente da legenda (ex: CR, EN, VU).
- **`DIM_HABITAT`**: Habitat primário padronizado.
- **`DIM_DIETA`**: Dieta primária padronizada.
- **`DIM_TAXONOMIA`**: Hierarquia de Ordem e Família.

> _Para detalhes técnicos de cada coluna e tipagem, consulte o arquivo `CATALOGO.md`._

---

## ⚙️ Pipeline ETL

O script `1_Pipeline_ETL.py` executa as seguintes etapas críticas para garantir a robustez do dado:

1.  **Ingestão Resiliente:** O script verifica se o arquivo existe localmente; caso contrário, realiza o download automático da URL oficial da Springer Nature/Figshare.
2.  **Tratamento de "Double Header":** O Excel original possui categorias na Linha 1 e os nomes reais das colunas na Linha 2. O script utiliza pandas com `header=1` para resolver isso.
3.  **Parsing de Legenda Não Estruturada:** Um algoritmo varre a aba de legendas (que contém texto livre) para encontrar a linha exata onde começa a tabela de códigos da IUCN.
4.  **Sanitização de Nomes:** Tratamento de colunas com caracteres especiais (ex: `English Name (BirdLife > IOC > Clements>AviList)`) utilizando backticks no Spark.
5.  **Limpeza de Tipos (`try_cast`):** Conversão segura de colunas numéricas (como _Average Mass_ e _Clutch Size_) para evitar falhas de execução devido a caracteres sujos no Excel.
6.  **Qualidade:** Tratamento de valores nulos (`fillna`) e padronização de strings (`UPPER`/`TRIM`).

---

## 📊 Resultados e Análises

As análises foram realizadas via SQL (ver `2_Analise_SQL.sql`). Abaixo, exemplos dos insights gerados:

1.  **Conservação por Família:** Identificação das famílias taxonômicas com maior proporção de espécies ameaçadas.
2.  **Impacto da Dieta:** Análise da relação entre dieta primária e massa corporal média.
3.  **Check de Qualidade:** O pipeline garantiu 0 falhas de integridade referencial (órfãos) e consistência numérica.

---

## 📂 Estrutura do Repositório

```text
/
├── README.md                 # Documentação principal (Este arquivo)
├── Catálogo.md               # Dicionário de dados detalhado
├── AutoAvaliação.md          # Reflexão e autoavaliação do aluno
│
├── notebooks/
│   ├── 1_Pipeline_ETL_Engenharia.ipynb     # Código Fonte do ETL (PySpark)
│   └── 2_Analise_SQL.ipynb    # Consultas de Análise e Qualidade
│
└── evidencias/               # Screenshots e gráficos gerados
    ├── 1- ANÁLISE DE CONSERVAÇÃO POR FAMÍLIA.png
    ├── 2- RELAÇÃO DIETA vs. MASSA CORPORAL.png
    ├── 3- HABITAT DAS ESPÉCIES AMEAÇADAS.png
    └── ...
```

## 🚀 Como Executar

Para reproduzir este projeto no seu ambiente Databricks:

1.  Clone este repositório.
2.  No **Databricks Workspace**, importe os arquivos da pasta `/notebooks`.
3.  Execute o notebook `1_Pipeline_ETL`.
    > **Nota:** Ele instalará automaticamente a dependência `openpyxl`, baixará os dados e criará as tabelas Delta.
4.  Execute o notebook `2_Analise_SQL` para visualizar os insights e gráficos.
