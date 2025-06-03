# Entendendo RDBMS vs. NoSQL

Este documento resume as diferenças fundamentais entre Sistemas Gerenciadores de Banco de Dados Relacionais (RDBMS) e bancos de dados NoSQL, com base em exemplos comuns de tecnologias e seus usos.

---

## RDBMS (Relational Database Management System)

Sistemas RDBMS organizam os dados em tabelas com linhas e colunas, utilizando esquemas pré-definidos e a linguagem SQL para manipulação e consulta. São ideais para dados estruturados onde a integridade e a consistência transacional são cruciais.

**Exemplos e Características:**

* **MySQL:** Frequentemente usado para armazenar dados de maneira relacional tradicional, com forte foco em transações.
* **PostgreSQL:** Conhecido por sua robustez e conformidade com padrões SQL, armazenando dados relacionais orientados por linhas.
* **MS SQL (Microsoft SQL Server):** Permite a persistência de dados em tabelas, organizados por linha, amplamente utilizado em ambientes corporativos.

---

## NoSQL (Not Only SQL)

Bancos de dados NoSQL são projetados para lidar com grandes volumes de dados não estruturados, semiestruturados ou polimórficos, oferecendo flexibilidade de esquema, alta escalabilidade e desempenho em cenários específicos. Existem diversos tipos de bancos NoSQL, cada um otimizado para um tipo diferente de modelagem de dados.

**Exemplos e Características:**

* **PostgreSQL JSON:** Embora seja um RDBMS, o PostgreSQL com suporte a JSON (especialmente `JSONB`) pode ser usado para armazenar e consultar dados de documentos, oferecendo flexibilidade de esquema similar a bancos de dados de documentos NoSQL.
* **Snowflake (com armazenamento colunar):** Snowflake é um data warehouse baseado em nuvem que utiliza armazenamento orientado por coluna. Esse modelo é otimizado para consultas analíticas complexas sobre grandes volumes de dados, uma característica que se alinha com a escalabilidade e a otimização de big data frequentemente associadas ao universo NoSQL.
* **Redis (pares de valores-chave):** Um banco de dados em memória que armazena dados como pares de valores-chave, ideal para caching, gerenciamento de sessões e outras aplicações de alta velocidade.
* **Bancos de dados gráficos (nós e bordas):** Projetados para armazenar e consultar dados em uma estrutura de rede de nós (entidades) e bordas (relacionamentos), excelentes para modelar e analisar relações complexas (ex: redes sociais, grafos de conhecimento).

---

**Escolha entre RDBMS e NoSQL:**

A escolha entre RDBMS e NoSQL depende das necessidades específicas do projeto, do tipo de dados a serem armazenados, dos requisitos de escalabilidade, flexibilidade e consistência. Muitos projetos modernos utilizam uma combinação de ambos (abordagem poliglota de persistência) para aproveitar os pontos fortes de cada tipo de banco de dados.

## Bancos de Dados Orientados por Colunas

Bancos de dados orientados por colunas, como o Snowflake, armazenam dados em colunas em vez de linhas. Isso oferece vantagens significativas em certos cenários:

* **Leitura e Recuperação Seletiva de Colunas:** Permitem que o sistema leia apenas as colunas necessárias para uma consulta, reduzindo a quantidade de dados lidos do disco e melhorando o desempenho, especialmente em consultas analíticas.
* **Facilidade em Alterações de Esquema:** Lidar com alterações de esquema, como adicionar novas colunas, é mais eficiente. Menos dados precisam ser acessados e modificados em comparação com bancos de dados orientados por linhas. A adição de uma nova coluna torna-se uma operação de metadados, sem a necessidade de modificar fisicamente todas as linhas existentes.
* **Armazenamento Eficiente e Desempenho de Consultas:** O armazenamento colunar permite uma melhor compressão dos dados e, consequentemente, um melhor desempenho das consultas.

* ## Conectando e Executando Queries no Snowflake com Python

Para interagir com o Snowflake a partir de Python, você pode usar o conector oficial do Snowflake. Abaixo estão exemplos de como estabelecer uma conexão e executar consultas SQL.

### Conectando a um Banco de Dados Snowflake

Primeiro, você precisará importar o conector e usar a função `connect()` para estabelecer a conexão. Substitua os placeholders (`<user>`, `<password>`, etc.) com suas credenciais e informações do Snowflake.

```python
import snowflake.connector

conn = snowflake.connector.connect(
    user="<user>",
    password="<password>",
    account="<account_identifier>",
    database="<database_name>",
    schema="<schema_name>",
    warehouse="<warehouse_name>"
)

# Construa uma query em uma string (ou string multi-linha)
query = """
SELECT
    title,
    price
FROM books
WHERE price < 50.00;
"""

# Execute a query, imprima os resultados
results = conn.cursor().execute(query).fetch_pandas_all()
print(results)

# Retorna time, nome e ano para todos os anos maiores que 2000
query = """
SELECT
    team,
    name,
    year
FROM olympic_medals
WHERE year > 2000;
"""

# Execute a query, imprima os resultados
results = conn.cursor().execute(query).fetch_pandas_all()
print(results)

# Query para extrair 'statement' e 'location' do campo 'review' aninhado
query = """
    SELECT
        review -> 'statement' AS statement,
        review -> 'location' AS location
    FROM nested_reviews;
"""

# Execute a query e renderize os resultados (assumindo 'db_engine' para conexão)
# Nota: Se estiver usando o 'conn' do snowflake.connector,
# a linha de execução seria 'data = conn.cursor().execute(query).fetch_pandas_all()'
# O exemplo original usa 'pd.read_sql' com 'db_engine', que é uma alternativa comum.
data = pd.read_sql(query, db_engine) # ou data = conn.cursor().execute(query).fetch_pandas_all()
print(data)

```

## Definição de Esquema (DDL)

Além de consultar dados, você pode usar o Snowflake para definir e gerenciar seu esquema de banco de dados. Abaixo está um exemplo de como criar uma tabela. É uma boa prática **organizar as colunas em ordem alfabética** para melhorar a legibilidade e a manutenção do esquema.

```sql
CREATE TABLE olympic_athletes (
    age INT,
    country VARCHAR(64),
    first_name VARCHAR(64),
    is_first_games BOOLEAN,
    last_name VARCHAR(64)
);
```
## Preenchimento de Tabelas no Snowflake

O Snowflake oferece diferentes métodos para preencher tabelas, seja criando uma nova tabela a partir de uma consulta (CTAS) ou carregando dados de arquivos externos (`COPY INTO`). É crucial usar a sintaxe e o comando corretos para cada cenário.

### Exemplos de Preenchimento de Tabelas

Aqui demonstramos exemplos de uso correto e incorreto das técnicas de preenchimento:

#### **CREATE TABLE ... AS (CTAS)**

* **Uso Correto:** Criar uma nova tabela a partir dos resultados de uma consulta SQL.

    ```sql
    CREATE TABLE gold_medal_winners AS
    SELECT
        team,
        year,
        sport,
        event
    FROM olympic_medals
    WHERE medal = 'Gold';
    ```
    *Este comando cria uma nova tabela `gold_medal_winners` contendo apenas os dados de medalhas de ouro da tabela `olympic_medals`.*

* **Uso Incorreto:** Tentar usar `COPY INTO ... AS SELECT` para criar uma tabela a partir de uma consulta. `COPY INTO` é para carregar dados de arquivos.

    ```sql
    -- Este comando está incorreto para o propósito de criar uma tabela a partir de uma consulta
    COPY INTO silver_medal_winners AS
    SELECT
        team,
        year,
        sport,
        event
    FROM olympic_medals
    WHERE medal = 'Silver';
    ```

#### **COPY INTO**

* **Uso Correto:** Carregar dados de um arquivo externo (CSV, por exemplo) para uma tabela Snowflake.

    ```sql
    COPY INTO olympic_medals_analysis
    FROM "@~/raw_olympic_medals.csv" -- Exemplo de um estágio de usuário
    FILE_FORMAT = (
        TYPE = 'CSV',
        FIELD_DELIMITER = ','
    );
    ```
    *Este comando carrega dados de um arquivo CSV (assumindo que ele está em um estágio acessível, como o estágio de usuário `~/`) para a tabela `olympic_medals_analysis`.*

* **Uso Incorreto:** Tentar usar `CREATE TABLE ... USING` para carregar dados de um arquivo. `CREATE TABLE` define a estrutura da tabela, não a preenche a partir de um arquivo com essa sintaxe.

    ```sql
    -- Este comando está incorreto para o propósito de carregar dados de um arquivo
    CREATE TABLE olympic_medals_analysis USING
    FROM olympic_medals
    FILE_FORMAT = (
        TYPE = 'CSV',
        FIELD_DELIMITER = ','
    );
    ```

## Microparticionamento e Clustering de Dados no Snowflake

O Snowflake utiliza conceitos avançados de armazenamento e otimização para garantir alta performance, mesmo com grandes volumes de dados. Dois conceitos chave são o **Microparticionamento** e o **Agrupamento de Dados (Clustering)**.

### Microparticionamento

* **Definição:** As tabelas no Snowflake são automaticamente divididas em **micropartições**. Cada micropartição é um grupo de linhas de uma tabela maior, armazenada de forma otimizada em formato colunar.
* **Metadados:** Cada micropartição contém metadados ricos (como intervalos de valores para cada coluna, contagens de valores distintos, etc.) que o Snowflake usa para otimizar as consultas.
* **Otimização:** A existência das micropartições permite que o Snowflake realize a "poda de micropartições" (micro-partition pruning), lendo apenas os blocos de dados relevantes para uma consulta, o que reduz significativamente a quantidade de dados escaneados.

### Agrupamento de Dados (Clustering)

* **Objetivo:** O agrupamento de dados no Snowflake refere-se à organização e classificação das linhas dentro das micropartições. O objetivo principal é agrupar linhas com valores similares em colunas específicas (as "chaves de clustering") para que elas sejam armazenadas próximas umas das outras.
* **Benefício:** Essa organização otimizada melhora drasticamente o desempenho das consultas. Quando uma consulta filtra dados com base nas chaves de clustering, o Snowflake pode usar os metadados das micropartições para "podar" (ignorar) rapidamente as micropartições que não contêm os dados relevantes, diminuindo a leitura de dados e acelerando a resposta da consulta.

Esses mecanismos internos do Snowflake são fundamentais para sua capacidade de processar grandes cargas de trabalho analíticas com eficiência e escalabilidade.

## Redução de Consultas com Microparticionamento no Snowflake

O Snowflake utiliza suas **micropartições** para otimizar drasticamente o desempenho das consultas, especialmente aquelas com cláusulas `WHERE` que filtram grandes volumes de dados.

Considere a seguinte consulta:

```sql
SELECT
    team,
    year,
    sport,
    event,
    medal
FROM olympic_medals
WHERE year >= 2000;
```
# Common Table Expressions (CTEs) e Views no Snowflake

Este documento explica o uso de Common Table Expressions (CTEs) e Views (incluindo Views Materializadas) no Snowflake, apresentando suas características e exemplos de código.

---

## Common Table Expressions (CTEs)

CTEs são subconsultas nomeadas temporárias que você pode definir dentro de uma única instrução SQL (como `SELECT`, `INSERT`, `UPDATE` ou `DELETE`). Elas são definidas usando a palavra-chave `WITH`.

### Características das CTEs

* **Subconsultas Nomeadas/Tabelas Temporárias:** Permitem dar um nome a uma subconsulta complexa, tornando o código mais legível e autoexplicativo.
* **Criação de Objeto Temporário:** Cria um objeto temporário que existe apenas para a duração da consulta principal e pode ser referenciado posteriormente.
* **Redução de Dados:** Podem ajudar a reduzir a quantidade de dados que são consultados e/ou unidos, ao pré-filtrar ou agregar dados na CTE antes de operações mais complexas.
* **Modularidade e Facilidade de Depuração:** Quebram consultas complexas em blocos lógicos menores, tornando-as mais fáceis de entender, manter e solucionar problemas.

### Exemplo de Uso de CTE

Este exemplo mostra como criar uma CTE chamada `premium_books` para filtrar livros com preço superior a 25.00 e, em seguida, usar essa CTE para calcular as avaliações mínimas e máximas por autor.

```sql
WITH premium_books AS (
    SELECT
        title,
        author,
        avg_reviews
    FROM books
    WHERE price > 25.00
)
SELECT
    author,
    MIN(avg_reviews) AS min_avg_reviews,
    MAX(avg_reviews) AS max_avg_reviews
FROM premium_books
GROUP BY author;
```
### Criando Múltiplos Objetos Temporários com CTEs
Você pode definir múltiplas CTEs em uma única cláusula WITH, separadas por vírgulas:

``` SQL
WITH
    <first-name> AS (
        SELECT
            -- ...
        FROM <table-name>
        [JOIN | WHERE | ...]
    ),
    <second-name> AS (
        -- ...
    ),
    -- ... outras CTEs
SELECT
    -- ...
FROM <cte-name>; -- A CTE final pode se referir às anteriores
```

### Views no Snowflake
Views são consultas SQL salvas que atuam como tabelas virtuais. Quando uma view é consultada, a consulta subjacente é executada, e os resultados são apresentados como se viessem de uma tabela real.

Views Regulares (Não Materializadas)

Execução da Consulta: A consulta subjacente à view é executada cada vez que a view é chamada. Os resultados não são armazenados fisicamente no disco.

Definição Nomeada: É essencialmente uma "definição nomeada" de uma consulta, que simplifica consultas complexas e promove a reutilização.

Exemplo de Criação e Uso de View Regular:
``` SQL
CREATE VIEW premium_books AS
SELECT
    title,
    author,
    avg_reviews
FROM books
WHERE price >= 25.00;

-- Consultando a view
SELECT * FROM premium_books;
```
### Views Materializadas (Materialized Views)
- Resultados Armazenados: Ao contrário das views regulares, as views materializadas armazenam fisicamente os resultados da consulta no momento da sua criação e são mantidas atualizadas.
- Melhor Desempenho de Consulta: Como os dados já estão pré-calculados e armazenados, consultar uma view materializada geralmente oferece um desempenho significativamente melhor do que consultar uma view regular ou a consulta subjacente diretamente.
- Requer Atualização: Os resultados devem ser atualizados (refreshing) quando os dados da tabela base mudam para garantir que a view materializada esteja sincronizada. O Snowflake gerencia essa atualização automaticamente na maioria dos casos, mas há custos de computação associados.
Exemplo de Criação e Uso de View Materializada:
``` SQL
CREATE MATERIALIZED VIEW premium_books AS
SELECT
    title,
    author,
    avg_reviews
FROM books
WHERE price >= 25.00;

-- Consultando a view materializada
SELECT * FROM premium_books;
```
