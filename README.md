# db-macro-ecommerce

Sistema analítico de inteligência de negócios para centralizar dados de faturamento corporativo e cruzá‑los com os principais indicadores da economia brasileira (IPCA, PIB, câmbio, taxa de juros, etc.). O objetivo é fornecer modelos, consultas e visões prontas para análises financeiras, dashboards e relatórios gerenciais.

## Sumário
- [Visão Geral](#visão-geral)
- [Principais Funcionalidades](#principais-funcionalidades)
- [Modelo de Dados](#modelo-de-dados)
- [Estrutura do Repositório](#estrutura-do-repositório)
- [Pré-requisitos](#pré-requisitos)
- [Instalação e execução](#instalação-e-execução)
- [Exemplos de consultas](#exemplos-de-consultas)
- [Cargas (ETL) e atualização de indicadores](#cargas-etl-e-atualização-de-indicadores)
- [Testes e validação](#testes-e-validação)
- [Contribuição](#contribuição)
- [Licença](#licença)
- [Contato](#contato)

## Visão Geral
Este projeto fornece:
- Esquemas SQL (DDL) para criação de tabelas dimensionais e de fatos.
- Views e agregações para análises mensal e anual.
- Scripts de carga/seeds para dados de exemplo.
- Consultas analíticas para cruzar faturamento com indicadores macroeconômicos.

Público-alvo: analistas de BI, cientistas de dados, engenheiros de dados e diretores financeiros.

## Principais Funcionalidades
- Modelo dimensional (fato de faturamento + dimensões cliente, produto, empresa, tempo).
- Tabelas de indicadores econômicos (IPCA, PIB, taxa SELIC, câmbio) alinhadas por data/mês.
- Views prontas para:
  - Receita por cliente / produto / região.
  - Receita ajustada por inflação (IPCA).
  - Comparativos mês a mês e year-over-year.
- Exemplos de queries para KPI financeiros (MRR, churn revenue, ticket médio, crescimento real).

## Modelo de Dados
Estrutura recomendada (exemplo simplificado):

- Dimensões
  - dim_empresa (empresa_id, nome, cnpj, segmento, região)
  - dim_cliente (cliente_id, nome, segmento, porte, região)
  - dim_produto (produto_id, sku, nome, categoria, preço_base)
  - dim_tempo (data, dia, mes, ano, trimestre, mes_ano)

- Fato
  - fat_faturamento (fato_id, empresa_id, cliente_id, produto_id, data_venda, quantidade, valor_bruto, descontos, impostos, valor_liquido)

- Indicadores macro
  - indicadores_mensais (mes_ano, ipca, pib, selic, cambio_usd_brl, observacao)

Diagrama ER (resumido):
dim_empresa ---< fat_faturamento >--- dim_cliente
dim_produto ---< fat_faturamento
dim_tempo ---< fat_faturamento
indicadores_mensais (ligado por mes_ano -> dim_tempo.mes_ano)

## Estrutura do Repositório
Sugestão de organização de pastas e arquivos SQL:

- sql/
  - ddl/
    - 01_create_schema.sql
    - 02_create_tables.sql
    - 03_indexes.sql
  - dml/
    - seeds_sample_data.sql
    - load_indicators.sql
  - views/
    - vw_revenue_monthly.sql
    - vw_revenue_by_client.sql
  - queries/
    - example_kpis.sql
- docs/
  - data_dictionary.md
  - er_diagram.png
- docker/
  - docker-compose.yml (opcional, Postgres + pgAdmin)
- README.md

## Pré-requisitos
- PostgreSQL 12+ (recomendado). O projeto pode ser adaptado para MySQL ou outros RDBMS com pequenas alterações.
- psql (cliente) ou uma ferramenta SQL (pgAdmin, DBeaver).
- Opcional: Docker / Docker Compose (para ambiente local isolado).

## Instalação e execução
1. Clonar o repositório:
   git clone https://github.com/thlimajr/db-macro-ecommerce.git
2. Criar banco e rodar scripts DDL:
   - Usando psql:
     psql -h <host> -U <user> -d <db> -f sql/ddl/01_create_schema.sql
     psql -h <host> -U <user> -d <db> -f sql/ddl/02_create_tables.sql
     psql -h <host> -U <user> -d <db> -f sql/ddl/03_indexes.sql
   - Ou via Docker Compose (se houver docker/docker-compose.yml):
     docker-compose up -d
     # aguardar Postgres subir e rodar scripts pelo container ou conexão externa

3. Carregar dados de exemplo e indicadores:
   psql -h <host> -U <user> -d <db> -f sql/dml/seeds_sample_data.sql
   psql -h <host> -U <user> -d <db> -f sql/dml/load_indicators.sql

## Exemplos de consultas
1) Receita mensal agregada:
```sql
SELECT t.mes_ano, SUM(f.valor_liquido) AS receita_total
FROM fat_faturamento f
JOIN dim_tempo t ON f.data_venda = t.data
GROUP BY t.mes_ano
ORDER BY t.mes_ano;
