# 📊 Northwind Sales Analytics

Projeto de análise de dados utilizando **SQL e PostgreSQL** a partir do banco de dados **Northwind**.

O objetivo do projeto é responder perguntas de negócio relacionadas a **receita, vendas, clientes e produtos**, utilizando consultas SQL e técnicas de análise como **agregações, JOINs, CTEs e Window Functions**.

---

## 🎯 Objetivo

Este projeto tem como objetivo transformar dados transacionais do banco Northwind em informações úteis para apoiar a tomada de decisões.

As análises desenvolvidas buscam responder perguntas como:

* 💰 Qual foi a receita total da empresa?
* 📈 Como a receita evoluiu ao longo do tempo?
* 📊 Qual foi o crescimento mensal das vendas?
* 📅 Qual é a receita acumulada no ano (YTD)?
* 👥 Quais clientes geram mais receita?
* 🎯 Como os clientes podem ser segmentados de acordo com seu valor?
* 🏆 Quais são os produtos que geram mais receita?
* 📉 Quais clientes possuem menor volume de compras e podem ser analisados em campanhas de marketing?

---

# 🗂️ Estrutura do Projeto

```text
northwind-sales-analytics/
│
├── README.md
│
├── sql/
│   ├── 01_revenue_analysis.sql
│   ├── 02_customer_analysis.sql
│   ├── 03_customer_segmentation.sql
│   ├── 04_product_analysis.sql
│   └── 05_sales_growth.sql
│
├── database/
│   └── northwind.sql
│
├── docker/
│   └── docker-compose.yml
│
└── images/
    └── northwind-er-diagram.png
```

---

# 🛠️ Tecnologias Utilizadas

* **SQL**
* **PostgreSQL**
* **Docker**
* **Docker Compose**
* **pgAdmin**

---

# 🧠 Conceitos de SQL Aplicados

Durante o desenvolvimento das análises são utilizados diversos conceitos importantes para análise e manipulação de dados.

### Consultas e Agregações

```sql
SELECT
FROM
WHERE
GROUP BY
HAVING
ORDER BY
LIMIT
```

### JOINs

```sql
INNER JOIN
LEFT JOIN
```

### Funções de Agregação

```sql
SUM()
COUNT()
AVG()
MIN()
MAX()
```

### CTEs

```sql
WITH nome_cte AS (
    SELECT ...
)
SELECT ...
FROM nome_cte;
```

### Window Functions

```sql
SUM() OVER()
LAG()
LEAD()
ROW_NUMBER()
RANK()
DENSE_RANK()
NTILE()
```

### Análise Temporal

```sql
EXTRACT(YEAR FROM data)
EXTRACT(MONTH FROM data)
```

---

# 📊 Análises Desenvolvidas

## 1️⃣ Análise de Receita

A primeira análise tem como objetivo calcular a receita total da empresa.

A fórmula utilizada considera:

```text
Receita = Preço Unitário × Quantidade × (1 - Desconto)
```

Exemplo:

```sql
SELECT
    SUM(
        order_details.unit_price *
        order_details.quantity *
        (1.0 - order_details.discount)
    ) AS total_revenue
FROM order_details;
```

---

## 2️⃣ Análise de Receita Mensal

Nesta análise, as vendas são agrupadas por ano e mês.

```sql
SELECT
    EXTRACT(YEAR FROM orders.order_date) AS year,
    EXTRACT(MONTH FROM orders.order_date) AS month,
    SUM(
        order_details.unit_price *
        order_details.quantity *
        (1.0 - order_details.discount)
    ) AS monthly_revenue
FROM orders
INNER JOIN order_details
    ON orders.order_id = order_details.order_id
GROUP BY
    EXTRACT(YEAR FROM orders.order_date),
    EXTRACT(MONTH FROM orders.order_date)
ORDER BY
    year,
    month;
```

Essa análise permite acompanhar a evolução da receita ao longo do tempo.

---

## 3️⃣ Crescimento Mensal e Receita Acumulada (YTD)

Para analisar a evolução das vendas, são utilizadas **Window Functions**.

A função `LAG()` permite comparar a receita atual com a receita do período anterior.

```sql
LAG(monthly_revenue) OVER (
    PARTITION BY year
    ORDER BY month
)
```

A receita acumulada no ano é calculada utilizando:

```sql
SUM(monthly_revenue) OVER (
    PARTITION BY year
    ORDER BY month
)
```

Essa métrica é conhecida como:

> **YTD — Year To Date**

Ela representa o valor acumulado desde o início do ano até determinado período.

---

## 4️⃣ Análise de Clientes

Nesta etapa é calculado o valor total gasto por cada cliente.

```sql
SELECT
    customers.company_name,
    SUM(
        order_details.unit_price *
        order_details.quantity *
        (1.0 - order_details.discount)
    ) AS total_revenue
FROM customers
INNER JOIN orders
    ON customers.customer_id = orders.customer_id
INNER JOIN order_details
    ON orders.order_id = order_details.order_id
GROUP BY
    customers.company_name
ORDER BY
    total_revenue DESC;
```

Essa análise permite identificar os clientes que geram maior receita para a empresa.

---

## 5️⃣ Segmentação de Clientes

Os clientes são divididos em grupos de acordo com o valor total gasto.

Para isso, é utilizada a Window Function:

```sql
NTILE(5)
```

Exemplo:

```sql
NTILE(5) OVER (
    ORDER BY total_revenue DESC
)
```

Os clientes são divididos em **5 grupos**.

| Grupo | Característica                  |
| ----- | ------------------------------- |
| 1     | Clientes com maior valor        |
| 2     | Clientes de alto valor          |
| 3     | Clientes de valor intermediário |
| 4     | Clientes de menor valor         |
| 5     | Clientes com menor valor        |

Essa segmentação pode ser utilizada para análises de:

* Marketing;
* Retenção de clientes;
* Fidelização;
* Estratégias comerciais.

---

## 6️⃣ Ranking dos Produtos

Nesta análise são identificados os produtos que geram maior receita.

```sql
SELECT
    products.product_name,
    SUM(
        order_details.unit_price *
        order_details.quantity *
        (1.0 - order_details.discount)
    ) AS sales
FROM products
INNER JOIN order_details
    ON products.product_id = order_details.product_id
GROUP BY
    products.product_name
ORDER BY
    sales DESC
LIMIT 10;
```

O resultado apresenta os **10 produtos com maior receita**.

---

# 🗃️ Modelo de Dados

O banco de dados Northwind representa uma empresa fictícia chamada **Northwind Traders**.

O banco possui informações relacionadas a:

* 👥 Clientes
* 📦 Produtos
* 🛒 Pedidos
* 📋 Detalhes dos pedidos
* 🏢 Fornecedores
* 👨‍💼 Funcionários
* 🚚 Transportadoras

As principais tabelas utilizadas neste projeto são:

```text
customers
    │
    │ customer_id
    ▼
orders
    │
    │ order_id
    ▼
order_details
    │
    │ product_id
    ▼
products
```

---

# 🐳 Executando o Projeto com Docker

## Pré-requisitos

Antes de iniciar, é necessário possuir instalado:

* Docker
* Docker Compose

Após clonar o repositório, execute:

```bash
docker compose up -d
```

Isso iniciará os serviços necessários para executar o banco PostgreSQL e o pgAdmin.

Para parar os containers:

```bash
docker compose down
```

Caso seja necessário remover também os volumes e os dados:

```bash
docker compose down -v
```

---

# 🚀 Próximos Passos

Este projeto será evoluído com novas análises e tecnologias.

Algumas melhorias planejadas incluem:

* [ ] Adicionar análise de crescimento **MoM**
* [ ] Adicionar análise de crescimento **YoY**
* [ ] Criar ranking de clientes utilizando `RANK()`
* [ ] Implementar análise RFM
* [ ] Adicionar análise de retenção de clientes
* [ ] Criar novas análises utilizando CTEs
* [ ] Utilizar Python para conexão e análise dos dados
* [ ] Criar visualizações utilizando Pandas e Matplotlib
* [ ] Criar pipeline ETL para processamento dos dados
* [ ] Evoluir o projeto para uma arquitetura de Data Engineering

---

# 📚 Principais Aprendizados

Durante o desenvolvimento deste projeto foram praticados conceitos relacionados a:

* Manipulação e consulta de dados utilizando SQL;
* Relacionamento entre tabelas utilizando JOINs;
* Agregação de dados;
* Filtros utilizando `WHERE` e `HAVING`;
* Criação de CTEs;
* Utilização de Window Functions;
* Análise temporal;
* Cálculo de métricas de negócio;
* Segmentação e ranking de clientes;
* Análise de receita e vendas;
* Utilização de PostgreSQL em ambiente Docker.

---

# 👨‍💻 Autor

**Pedro Henrique Prado**

Graduado em Engenharia de Controle e Automação e em transição para a área de Dados, com foco em **Data Analytics, Data Engineering, Machine Learning e Inteligência Artificial**.

🔗 LinkedIn: https://www.linkedin.com/in/pedro-prado-34369a1b5

🔗 GitHub: https://github.com/Dev-PPrado

---
