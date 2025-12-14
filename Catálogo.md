# Catálogo de Dados - BirdBase Analytics

**Documentação técnica do Data Warehouse gerado a partir da base BirdBase v2025.1 (Sekercioglu et al.).**

Este documento descreve o esquema das tabelas na camada **Gold** (Schema: `birdbase`), detalhando a origem dos dados, tipos e descrições de negócio.

## Visão Geral do Modelo

O modelo segue o padrão **Star Schema** (Esquema Estrela), composto por:

- **1 Tabela Fato:** Métricas quantitativas.
- **5 Tabelas Dimensão:** Contexto descritivo.

---

## 1. Tabela Fato: FATO_METRICAS_AVES

Centraliza as métricas biológicas, ecológicas e geográficas das espécies.

| Coluna              | Tipo (Spark) | Origem (Excel/CSV)          | Descrição                                                                    |
| :------------------ | :----------- | :-------------------------- | :--------------------------------------------------------------------------- |
| `sk_especie`        | `LONG`       | -                           | Chave artificial (FK) para Dimensão Espécie.                                 |
| `sk_status_iucn`    | `LONG`       | 2024 IUCN Red List category | Chave artificial (FK) para Dimensão Conservação.                             |
| `sk_habitat`        | `LONG`       | Primary Habitat             | Chave artificial (FK) para Dimensão Habitat.                                 |
| `sk_dieta`          | `LONG`       | Primary Diet                | Chave artificial (FK) para Dimensão Dieta.                                   |
| `sk_familia`        | `LONG`       | Family IOC 15.1             | Chave artificial (FK) para Dimensão Taxonomia.                               |
| `massa_media`       | `FLOAT`      | Average Mass                | Massa corporal média da ave em gramas (g). Valores nulos convertidos para 0. |
| `ninhada_max`       | `INT`        | Clutch_Max                  | Tamanho máximo da ninhada (número de ovos) registrada para a espécie.        |
| `altitude_min`      | `FLOAT`      | NormMin                     | Altitude mínima normalizada de ocorrência (metros).                          |
| `altitude_max`      | `FLOAT`      | NormMax                     | Altitude máxima normalizada de ocorrência (metros).                          |
| `eh_migratorio`     | `BOOLEAN`    | Mig                         | Flag indicando se a espécie é migratória (1=True, 0=False).                  |
| `eh_sedentario`     | `BOOLEAN`    | Sed                         | Flag indicando se a espécie é sedentária (1=True, 0=False).                  |
| `peso_dieta_inseto` | `FLOAT`      | IN-Wt                       | Porcentagem da dieta composta por Invertebrados/Insetos (0-100).             |
| `peso_dieta_fruta`  | `FLOAT`      | FR-Wt                       | Porcentagem da dieta composta por Frutas (0-100).                            |

---

## 2. Tabelas Dimensão

### 2.1 DIM_ESPECIE

Contém os dados de identificação e nomenclatura da espécie.

| Coluna        | Tipo     | Origem (Excel/CSV) | Descrição                                  |
| :------------ | :------- | :----------------- | :----------------------------------------- |
| `sk_especie`  | `LONG`   | -                  | Chave Primária (Surrogada).                |
| `id_original` | `STRING` | IOC 15.1           | ID único original da fonte (Catálogo IOC). |
| `nome_ingles` | `STRING` | English Name (...) | Nome comum da ave em inglês.               |
| `nome_latin`  | `STRING` | Latin (...)        | Nome científico (gênero e espécie).        |

### 2.2 DIM_CONSERVACAO

Contém o status de ameaça da IUCN, enriquecido com a descrição completa extraída da legenda.

| Coluna               | Tipo     | Origem (Excel/CSV) | Descrição                                                     |
| :------------------- | :------- | :----------------- | :------------------------------------------------------------ |
| `sk_status_iucn`     | `LONG`   | -                  | Chave Primária (Surrogada).                                   |
| `cod_categoria_iucn` | `STRING` | Aba Legend (Col A) | Código curto do status. Ex: CR, EN, VU, LC.                   |
| `desc_status`        | `STRING` | Aba Legend (Col B) | Descrição completa. Ex: Critically Endangered, Least Concern. |

### 2.3 DIM_TAXONOMIA

Hierarquia biológica da ave.

| Coluna       | Tipo     | Origem (Excel/CSV) | Descrição                               |
| :----------- | :------- | :----------------- | :-------------------------------------- |
| `sk_familia` | `LONG`   | -                  | Chave Primária (Surrogada).             |
| `familia`    | `STRING` | Family IOC 15.1    | Família taxonômica. Ex: Struthionidae.  |
| `ordem`      | `STRING` | Order              | Ordem taxonômica. Ex: Struthioniformes. |

### 2.4 DIM_HABITAT

Classificação do habitat principal da espécie.

| Coluna             | Tipo     | Origem (Excel/CSV) | Descrição                                                                                  |
| :----------------- | :------- | :----------------- | :----------------------------------------------------------------------------------------- |
| `sk_habitat`       | `LONG`   | -                  | Chave Primária (Surrogada).                                                                |
| `habitat_primario` | `STRING` | Primary Habitat    | Descrição do habitat principal. Ex: FOREST, SAVANNA, WETLAND. Normalizado para maiúsculas. |

### 2.5 DIM_DIETA

Classificação da guilda trófica (alimentação) principal.

| Coluna           | Tipo     | Origem (Excel/CSV) | Descrição                                                                                    |
| :--------------- | :------- | :----------------- | :------------------------------------------------------------------------------------------- |
| `sk_dieta`       | `LONG`   | -                  | Chave Primária (Surrogada).                                                                  |
| `dieta_primaria` | `STRING` | Primary Diet       | Tipo de dieta principal. Ex: FRUGIVORE, INSECTIVORE, CARNIVORE. Normalizado para maiúsculas. |

---

## 3. Linhagem de Dados (Data Lineage)

- **Fonte (Bronze):** Arquivo `BirdBase_Final.xlsx` (baixado da Springer Nature/Figshare).
  - _Aba Data:_ Dados principais (Header na linha 2).
  - _Aba Legend:_ Metadados de conservação (Texto não estruturado).
- **Transformação (Silver):** Script `1_Pipeline_ETL.py` (PySpark/Pandas).
  - Ingestão automatizada.
  - Limpeza de cabeçalhos duplos.
  - Parsing da legenda para extrair tabela de status.
  - Conversão de tipos (`try_cast`) e tratamento de nulos.
- **Armazenamento (Gold):** Tabelas Delta no banco de dados `birdbase`.
