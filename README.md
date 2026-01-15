# ETL_ProducaoAgricolaMunicipal

# Data Warehouse: Produção Agrícola Brasileira (IBGE)

Este projeto consiste em um pipeline de Engenharia de Dados completo (ETL) para a construção de um Data Warehouse focado na análise de lavouras temporárias e permanentes no Brasil. O projeto consome dados da API do IBGE (SIDRA), processa-os utilizando **Databricks** e **PySpark**, e disponibiliza insights sobre área plantada, colhida e valor de produção.

---

## Objetivo
Criar uma fonte única e confiável de dados (Single Source of Truth) para analisar o desempenho agrícola de culturas estratégicas (**Arroz, Feijão, Café e Laranja**) entre os anos de **2021 a 2024**.

O sistema permite responder perguntas como:
* Qual o valor total de produção por cultura?
* Qual a perda de área produtiva (diferença entre plantada e colhida) por estado?
* Comparativo de eficiência entre lavouras temporárias e permanentes.

---

## Arquitetura e Fluxo de Dados

O projeto segue a arquitetura **Medallion (Bronze, Silver, Gold)** dentro do Databricks:

### 1. Camada Bronze (Ingestão)
* **Fonte:** API do SIDRA/IBGE (Tabela 5457 - Pesquisa Agrícola Municipal).
* **Processo:** Extração parametrizada por blocos de anos e variáveis para contornar limites de requisição da API.
* **Dados Brutos:** Consolidação de métricas de *Área Plantada*, *Área Colhida* e *Valor de Produção*.

### 2. Camada Prata (Tratamento)
* **Limpeza:** Tratamento de símbolos do IBGE:
    * `-` (Zero absoluto) convertido para `0`.
    * Remoção de registros inválidos (`..`, `...`).
* **Tipagem:** Conversão de strings para tipos numéricos (`Integer` e `Double`) para cálculos precisos.
* **Enriquecimento:** Integração com API de Localidades para obter Regiões e Siglas de UF.

### 3. Camada Ouro (Modelagem Dimensional)
Construção de um **Modelo Star Schema** otimizado para OLAP:

* **Tabela Fato:**
    * `fato_producao`: Contém as métricas e chaves estrangeiras.
* **Tabelas Dimensão:**
    * `dim_local`: Município, Estado (UF) e Região.
    * `dim_tempo`: Ano (granularidade anual).
    * `dim_produto`: Produto, Tipo de Lavoura (Temporária/Permanente) e Categoria (Cereal, Fruta, etc.).

---

## Tecnologias Utilizadas
* **Databricks & Delta Lake:** Plataforma de processamento e armazenamento.
* **PySpark:** Transformação massiva de dados e manipulação de DataFrames.
* **Python (Requests):** Consumo de APIs REST.
* **Spark SQL:** Consultas analíticas e DDL.
* **Matplotlib:** Visualização de dados.

---

## Estrutura do Data Warehouse

### Diagrama Simplificado
```mermaid
erDiagram
    FACT_PRODUCAO }|--|| DIM_LOCAL : "loc"
    FACT_PRODUCAO }|--|| DIM_TEMPO : "time"
    FACT_PRODUCAO }|--|| DIM_PRODUTO : "prod"

    FACT_PRODUCAO {
        int chave_local FK
        int chave_produto FK
        int chave_tempo FK
        int area_plantada
        int area_colhida
        double valor
    }
