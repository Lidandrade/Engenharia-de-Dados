# Engenharia-de-Dados
Projeto de Engenharia de Dados desenvolvido em Databricks, utilizando SQL e o dataset de gorjetas.
# MVP: Construção de um Pipeline de Dados na Nuvem

**Aluno:** Lidiane Andrade de Sousa de Aragão  
**Curso:** Pós-Graduação em Ciência de Dados e Analytics — PUC-Rio  
**Plataforma de Nuvem:** Databricks Free Edition  

---

## 1. Contexto de Negócio e Perguntas (Etapa 2 e 4.1)

### Contexto de Negócio
O presente projeto consiste no desenvolvimento de um **Produto Mínimo Viável (MVP)** de Engenharia de Dados focado no setor de gastronomia/restaurantes. O objetivo principal é analisar o comportamento de consumo e o padrão de gorjetas oferecidas pelos clientes, permitindo entender fatores como variação por dia da semana e impacto do perfil do cliente (fumantes vs. não fumantes) no faturamento final do estabelecimento.

### Perguntas de Negócio
Para guiar o pipeline e fornecer respostas estratégicas, foram formuladas as seguintes perguntas:
1. **Qual é o valor médio da conta e das gorjetas por dia da semana?** (Buscando identificar os dias de maior movimento e rentabilidade).
2. **Existe diferença no valor médio da conta e na proporção da gorjeta oferecida entre clientes fumantes e não fumantes?** (Avaliando o impacto do perfil do cliente na receita extra).

### Estrutura dos Dados Brutos e Licença
* **Fonte:** Dataset público `tips` integrado à plataforma Databricks.
* **Licença:** Domínio público / Uso educacional aberto (BSD License).
* **Estrutura Bruta:** Tabela contendo 7 colunas originais (`total_bill`, `tip`, `sex`, `smoker`, `day`, `time`, `size`).

---

## 2. Carga dos Dados (Etapa 4.2)

A ingestão e disponibilização dos dados foram realizadas na plataforma de nuvem **Databricks Free Edition**. Por se tratar de um dataset de demonstração curado e nativo da plataforma, a carga inicial foi executada via consulta direta à tabela `workspace.default.tips`, persistindo a entrada na **Camada Bronze** do nosso Lakehouse.

* **Script de Origem/Execução:** Os comandos de carga e transformação estão totalmente documentados no notebook do projeto disponível neste repositório.

---

## 3. Modelagem e Catálogo de Dados (Etapa 4.3)

A modelagem adotou o conceito de **Flat Model / Tabela Analítica** estruturado em um ambiente **Data Lakehouse** com formato nativo Delta Lake, garantindo consistência transacional e performance.

### Catálogo de Dados (Unity Catalog / Schemas)

#### Tabela 1: `gorjetas_bronze` (Camada Bronze - Dados Brutos)
* **Descrição:** Cópia fidedigna do dataset original ingerido na nuvem.
* **Campos:**
  * `total_bill` (STRING/DOUBLE): Valor total da conta.
  * `tip` (STRING/DOUBLE): Valor da gorjeta.
  * `sex` (STRING): Gênero do pagador (`Male`, `Female`).
  * `smoker` (STRING): Indicador de fumante (`Yes`, `No`).
  * `day` (STRING): Dia da semana em inglês (`Thur`, `Fri`, `Sat`, `Sun`).
  * `time` (STRING): Período da refeição (`Lunch`, `Dinner`).
  * `size` (INT): Quantidade de pessoas na mesa.

#### Tabela 2: `gorjetas_silver` (Camada Silver - Dados Limpos e Padronizados)
* **Descrição:** Tabela higienizada com tratamento de tipos, padronização de valores e tradução de domínios para o português.
* **Campos:**
  * `valor_conta` (DOUBLE): Valor total da conta formatado.
  * `valor_gorjeta` (DOUBLE): Valor da gorjeta formatado.
  * `sexo` (STRING): Traduzido para `Feminino` / `Masculino`.
  * `cliente_fumante` (STRING): Traduzido para `Sim` / `Não`.
  * `dia_semana` (STRING): Traduzido para `Quinta`, `Sexta`, `Sábado`, `Domingo`.
  * `refeicao` (STRING): Traduzido para `Almoço` / `Jantar`.
  * `qtde_pessoas` (INT): Quantidade de pessoas na mesa.

#### Tabela 3: `gold_resumo_por_dia` (Camada Gold - Agregação Comercial)
* **Descrição:** Métrica consolidada agrupada por dia da semana.
* **Campos:** `dia_semana`, `total_mesas`, `conta_media`, `gorjeta_media`.

#### Tabela 4: `gold_resumo_fumantes` (Camada Gold - Perfil de Consumo)
* **Descrição:** Métrica consolidada agregando o comportamento por perfil fumante.
* **Campos:** `cliente_fumante`, `total_mesas`, `gorjeta_media`, `percentual_gorjeta_medio`.

---

## 4. Pipeline de Dados (Etapa 4.4)

O pipeline ETL foi estruturado no Databricks utilizando **SQL**, seguindo a **Arquitetura Medalhão (Medallion Architecture)**:
