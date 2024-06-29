# Exploração e Análise de Dados de Crédito com SQL

## Dados

Os dados representam informações de clientes de um banco e incluem as seguintes colunas:

- **idade**: Idade do cliente
- **sexo**: Sexo do cliente (F ou M)
- **dependentes**: Número de dependentes do cliente
- **escolaridade**: Nível de escolaridade do cliente
- **salario_anual**: Faixa salarial do cliente
- **tipo_cartao**: Tipo de cartão do cliente
- **qtd_produtos**: Quantidade de produtos comprados nos últimos 12 meses
- **iteracoes_12m**: Quantidade de interações/transações nos últimos 12 meses
- **meses_inativo_12m**: Quantidade de meses que o cliente ficou inativo
- **limite_credito**: Limite de crédito do cliente
- **valor_transacoes_12m**: Valor das transações dos últimos 12 meses
- **qtd_transacoes_12m**: Quantidade de transações nos últimos 12 meses

Os dados foram armazenados no AWS Athena com acesso aos dados no S3 Bucket. Uma versão dos dados está disponível em: [Dataset da EBAC](https://github.com/andre-marcos-perez/ebac-course-utils/tree/main/dataset)


---

# Exploração e Análise de Dados de Crédito

Este notebook detalha a análise de crédito com base nos dados coletados de clientes. Os dados incluem informações como idade, sexo, dependentes, escolaridade, faixa salarial, tipo de cartão, e métricas de transações.

## Exploração de Dados

### Quantidade de Registros

Iniciamos verificando a quantidade de registros na base de dados:

```sql
-- Query: Quantidade de registros na tabela
SELECT COUNT(*) AS quantidade_registros FROM credito;
```
|  #  | quantidade_registros |
|----:|----------------------|
|   1 |                 2564 |



**Resultado:** Observamos um total de 2564 registros.

### Visão Geral dos Dados

Para entender a estrutura dos dados, visualizamos as primeiras linhas do dataset:

```sql
-- Query: Visualização das primeiras linhas do dataset
SELECT * FROM credito LIMIT 10;
```
|  #  | idade | sexo | dependentes | escolaridade          | estado_civil | salario_anual    | tipo_cartao | qtd_produtos | iteracoes_12m | meses_inativo_12m | limite_credito | valor_transacoes_12m | qtd_transacoes_12m |
|----:|-------|------|-------------|-----------------------|--------------|------------------|-------------|--------------|---------------|-------------------|----------------|----------------------|-------------------|
|   1 |   45  |  M   |      3      | ensino medio          | casado       | 60K - 80K        | blue        |      5       |       3       |         1         |   12691.51     |       1144.9         |        42         |
|   2 |   49  |  F   |      5      | mestrado              | solteiro     | menos que 40K     | blue        |      6       |       2       |         1         |    8256.96     |       1291.45        |        33         |
|   3 |   51  |  M   |      3      | mestrado              | casado       | 80K - 120K       | blue        |      4       |       0       |         1         |    3418.56     |       1887.72        |        20         |
|   4 |   40  |  F   |      4      | ensino medio          | na           | menos que 40K     | blue        |      3       |       1       |         4         |    3313.03     |       1171.56        |        20         |
|   5 |   40  |  M   |      3      | sem educacao formal   | casado       | 60K - 80K        | blue        |      5       |       0       |         1         |    4716.22     |        816.08        |        28         |
|   6 |   44  |  M   |      2      | mestrado              | casado       | 40K - 60K        | blue        |      3       |       2       |         1         |    4010.69     |       1088.07        |        24         |
|   7 |   51  |  M   |      4      | na                    | casado       | 120K +           | gold        |      6       |       3       |         1         |   34516.72     |       1330.87        |        31         |
|   8 |   32  |  M   |      0      | ensino medio          | na           | 60K - 80K        | silver      |      2       |       2       |         2         |   29081.49     |       1538.32        |        36         |
|   9 |   37  |  M   |      3      | sem educacao formal   | solteiro     | 60K - 80K        | blue        |      5       |       0       |         2         |   22352.5      |       1350.14        |        24         |
|  10 |   48  |  M   |      2      | mestrado              | solteiro     | 80K - 120K       | blue        |      6       |       3       |         3         |   11656.41     |       1441.73        |        32         |



### Tipos de Dados:

```sql
DESCRIBE credito;
```

| Coluna              | Tipo      |
|---------------------|-----------|
| idade               | int       |
| sexo                | string    |
| dependentes         | int       |
| escolaridade        | string    |
| salario_anual       | string    |
| tipo_cartao         | string    |
| qtd_produtos        | int       |
| iteracoes_12m       | int       |
| meses_inativo_12m   | int       |
| limite_credito      | float     |
| valor_transacoes_12m| float     |
| qtd_transacoes_12m  | int       |

**Tipos de Escolaridade:**

```sql
SELECT DISTINCT escolaridade FROM credito;
```

| escolaridade    |
|-----------------|
| ensino médio    |
| mestrado        |
| sem educação    |
| ...             |
| na              |

**Tipos de Salário Anual:**

```sql
SELECT DISTINCT salario_anual FROM credito;
```

| salario_anual   |
|-----------------|
| 60 - 80     |
| menos que 40  |
| 80 - 120    |
| ...             |
| na              |

#### Distribuição Demográfica dos Clientes:

##### Distribuição por Idade
A distribuição por idade mostra a quantidade de clientes em cada faixa etária. Vamos observar as faixas etárias mais representadas e entender melhor a distribuição:

```sql
-- Distribuição por idade ordenada de forma decrescente pela quantidade de clientes
SELECT
    idade,
    quantidade_clientes
FROM
    (
    -- Subquery para numerar as linhas e ordenar pela quantidade de clientes
    SELECT
        idade,
        quantidade_clientes,
        ROW_NUMBER() OVER (ORDER BY quantidade_clientes DESC) AS rank
    FROM
        (
        -- Subquery para contar a quantidade de clientes por idade
        SELECT
            idade,
            COUNT(*) AS quantidade_clientes
        FROM
            default.credito
        GROUP BY
            idade
        ) AS subquery
    ) AS ranked
WHERE
    rank <= 10  -- Mostrar as 10 faixas etárias mais populares
ORDER BY
    idade;

```
Essa consulta mostra as 10 faixas etárias mais populares entre os clientes, ordenadas por idade.

|   #   | idade | quantidade_clientes |
|:-----:|:-----:|:-------------------:|
|   1   |   35  |          85         |
|   2   |   36  |          99         |
|   3   |   37  |         111         |
|   4   |   38  |          94         |
|   5   |   39  |         110         |
|   6   |   46  |          85         |
|   7   |   48  |          84         |
|   8   |   53  |         134         |
|   9   |   54  |         101         |
|  10   |   55  |          95         |



#### Distribuição por Sexo
Vamos verificar como está distribuído o sexo dos clientes:

```sql

-- Distribuição por sexo
SELECT
    sexo,
    COUNT(*) AS quantidade_clientes
FROM
    default.credito
GROUP BY
    sexo;


```
Essa consulta agrupa os clientes por sexo e conta quantos clientes há em cada categoria (Masculino e Feminino).

|  #  | sexo | quantidade_clientes |
|----:|------|---------------------|
|   1 |   F  |                1001 |
|   2 |   M  |                1563 |


###Distribuição por Número de Dependentes:
Vamos explorar como o número de dependentes está distribuído entre os clientes:

```sql
-- Distribuição por número de dependentes
SELECT
    dependentes,
    COUNT(*) AS quantidade_clientes
FROM
    default.credito
GROUP BY
    dependentes
ORDER BY
    dependentes;

 ```   
Esta consulta mostra a distribuição do número de dependentes entre os clientes, agrupando e contando quantos clientes têm 0, 1, 2, etc., dependentes.

|  #  | dependentes | quantidade_clientes |
|----:|-------------|---------------------|
|   1 |          0  |                 337 |
|   2 |          1  |                 545 |
|   3 |          2  |                 680 |
|   4 |          3  |                 612 |
|   5 |          4  |                 324 |
|   6 |          5  |                  66 |


### Segmentação por Faixa Etária

Vamos analisar como os clientes estão distribuídos por faixa etária. Podemos definir categorias de idade e contar quantos clientes estão em cada categoria:

```sql
-- Segmentação por faixa etária
SELECT
    CASE
        WHEN idade BETWEEN 18 AND 30 THEN '18-30'
        WHEN idade BETWEEN 31 AND 40 THEN '31-40'
        WHEN idade BETWEEN 41 AND 50 THEN '41-50'
        WHEN idade BETWEEN 51 AND 60 THEN '51-60'
        ELSE 'Mais de 60'
    END AS faixa_etaria,
    COUNT(*) AS quantidade_clientes
FROM
    default.credito
GROUP BY
    CASE
        WHEN idade BETWEEN 18 AND 30 THEN '18-30'
        WHEN idade BETWEEN 31 AND 40 THEN '31-40'
        WHEN idade BETWEEN 41 AND 50 THEN '41-50'
        WHEN idade BETWEEN 51 AND 60 THEN '51-60'
        ELSE 'Mais de 60'
    END
ORDER BY
    faixa_etaria;
```

Esta consulta agrupa os clientes em faixas etárias pré-definidas e conta quantos clientes estão em cada faixa etária.

# Faixa Etária e Quantidade de Clientes

| #   | faixa_etaria | quantidade_clientes |
|-----|--------------|---------------------|
| 1   | 18-30        | 138                 |
| 2   | 31-40        | 742                 |
| 3   | 41-50        | 752                 |
| 4   | 51-60        | 752                 |
| 5   | Mais de 60   | 180                 |


### Segmentação por Sexo e Número de Dependentes

Vamos explorar como o sexo dos clientes se relaciona com o número de dependentes que têm:

```sql
-- Segmentação por sexo e número de dependentes
SELECT
    sexo,
    dependentes,
    COUNT(*) AS quantidade_clientes
FROM
    default.credito
GROUP BY
    sexo,
    dependentes
ORDER BY
    sexo,
    dependentes;
```

Esta consulta mostra quantos clientes existem para cada combinação de sexo e número de dependentes, proporcionando insights sobre como essas características podem estar relacionadas.

# Sexo, Dependentes e Quantidade de Clientes

| #   | sexo | dependentes | quantidade_clientes |
|-----|------|-------------|---------------------|
| 1   | F    | 0           | 147                 |
| 2   | F    | 1           | 212                 |
| 3   | F    | 2           | 259                 |
| 4   | F    | 3           | 221                 |
| 5   | F    | 4           | 134                 |
| 6   | F    | 5           | 28                  |
| 7   | M    | 0           | 190                 |
| 8   | M    | 1           | 333                 |
| 9   | M    | 2           | 421                 |
| 10  | M    | 3           | 391                 |
| 11  | M    | 4           | 190                 |
| 12  | M    | 5           | 38                  |

### Segmentação por Faixa Salarial e Uso do Limite de Crédito

Vamos analisar como a faixa salarial dos clientes se relaciona com o uso médio do limite de crédito:

```sql
-- Segmentação por faixa salarial e uso médio do limite de crédito
SELECT
    salario_anual,
    AVG(limite_credito) AS media_limite_credito
FROM
    default.credito
GROUP BY
    salario_anual
ORDER BY
    salario_anual;
```

Essa consulta calcula a média do limite de crédito para cada faixa salarial, permitindo-nos entender como o uso do crédito pode variar com diferentes níveis de renda.

|  #  | Salário Anual    | Média Limite de Crédito |
|----:|------------------|-------------------------|
|  1  | 120K +          |         17801.488       |
|  2  | 40K - 60K      |          5348.356       |
|  3  | 60K - 80K      |          9096.028       |
|  4  | 80K - 120K     |         14886.556       |
|  5  | Menos que 40K   |          4099.476       |
|  6  | Não informado    |         10945.369       |



#### Relação entre Faixa Salarial e Comportamento de Crédito

Agora, vamos explorar a relação entre a faixa salarial dos clientes e métricas de crédito, como o uso do limite de crédito e o valor das transações.

```sql
-- Uso médio do limite de crédito por faixa salarial
SELECT
    salario_anual,
    AVG(limite_credito) AS media_limite_credito
FROM
    default.credito
GROUP BY
    salario_anual;
```

Esta consulta calcula a média do limite de crédito para cada faixa salarial, ajudando a entender como o limite de crédito varia com a faixa salarial dos clientes.

| #   | Salário Anual    | Média Limite de Crédito |
|-----|------------------|-------------------------|
| 1   | Menos que 40K   |          4099.476       |
| 2   | Não informado    |         10945.369       |
| 3   | 60K - 80K      |          9096.028       |
| 4   | 40K - 60K      |          5348.356       |
| 5   | 120K +          |         17801.488       |
| 6   | 80K - 120K     |         14886.556       |



### Valor Médio das Transações por Faixa Salarial

```sql
-- Valor médio das transações por faixa salarial
SELECT
    salario_anual,
    AVG(valor_transacoes_12m) AS media_valor_transacoes
FROM
    default.credito
GROUP BY
    salario_anual;
```
Essa consulta calcula a média do valor das transações realizadas nos últimos 12 meses para cada faixa salarial, oferecendo insights sobre como o comportamento de transações varia com a renda dos clientes.

| #   | Salário Anual    | Média Valor de Transações |
|-----|------------------|---------------------------|
| 1   | 60K - 80K      |          1818.6364        |
| 2   | 40K - 60K      |          1838.2643        |
| 3   | 120K +          |          1701.4652        |
| 4   | 80K - 120K     |          1755.2499        |
| 5   | Menos que 40K   |          1862.7195        |
| 6   | Não informado    |          1908.8855        |




### Segmentação por Faixa Etária

Vamos analisar como os clientes estão distribuídos por faixa etária. Podemos definir categorias de idade e contar quantos clientes estão em cada categoria:

```sql
-- Segmentação por faixa etária
SELECT
    CASE
        WHEN idade BETWEEN 18 AND 30 THEN '18-30'
        WHEN idade BETWEEN 31 AND 40 THEN '31-40'
        WHEN idade BETWEEN 41 AND 50 THEN '41-50'
        WHEN idade BETWEEN 51 AND 60 THEN '51-60'
        ELSE 'Mais de 60'
    END AS faixa_etaria,
    COUNT(*) AS quantidade_clientes
FROM
    default.credito
GROUP BY
    CASE
        WHEN idade BETWEEN 18 AND 30 THEN '18-30'
        WHEN idade BETWEEN 31 AND 40 THEN '31-40'
        WHEN idade BETWEEN 41 AND 50 THEN '41-50'
        WHEN idade BETWEEN 51 AND 60 THEN '51-60'
        ELSE 'Mais de 60'
    END
ORDER BY
    faixa_etaria;
```

Esta consulta agrupa os clientes em faixas etárias pré-definidas e conta quantos clientes estão em cada faixa etária.

| # | faixa_etaria | quantidade_clientes |
|---|--------------|---------------------|
| 1 | 18-30        | 138                 |
| 2 | 31-40        | 742                 |
| 3 | 41-50        | 752                 |
| 4 | 51-60        | 752                 |
| 5 | Mais de 60   | 180                 |


### Segmentação por Sexo e Número de Dependentes

Vamos explorar como o sexo dos clientes se relaciona com o número de dependentes que têm:

```sql
-- Segmentação por sexo e número de dependentes
SELECT
    sexo,
    dependentes,
    COUNT(*) AS quantidade_clientes
FROM
    default.credito
GROUP BY
    sexo,
    dependentes
ORDER BY
    sexo,
    dependentes;
```

Esta consulta mostra quantos clientes existem para cada combinação de sexo e número de dependentes, proporcionando insights sobre como essas características podem estar relacionadas.

| #  | sexo | dependentes | quantidade_clientes |
|----|------|-------------|---------------------|
| 1  | F    | 0           | 147                 |
| 2  | F    | 1           | 212                 |
| 3  | F    | 2           | 259                 |
| 4  | F    | 3           | 221                 |
| 5  | F    | 4           | 134                 |
| 6  | F    | 5           | 28                  |
| 7  | M    | 0           | 190                 |
| 8  | M    | 1           | 333                 |
| 9  | M    | 2           | 421                 |
| 10 | M    | 3           | 391                 |
| 11 | M    | 4           | 190                 |
| 12 | M    | 5           | 38                  |


### Segmentação por Faixa Salarial e Uso do Limite de Crédito

Vamos analisar como a faixa salarial dos clientes se relaciona com o uso médio do limite de crédito:

```sql
-- Segmentação por faixa salarial e uso médio do limite de crédito
SELECT
    salario_anual,
    AVG(limite_credito) AS media_limite_credito
FROM
    default.credito
GROUP BY
    salario_anual
ORDER BY
    salario_anual;
```
Essa consulta calcula a média do limite de crédito para cada faixa salarial, permitindo-nos entender como o uso do crédito pode variar com diferentes níveis de renda.

| #  | salario_anual | media_limite_credito |
|----|---------------|----------------------|
| 1  | 120K +        | 17801.488            |
| 2  | 40K - 60K     | 5348.356             |
| 3  | 60K - 80K     | 9096.028             |
| 4  | 80K - 120K    | 14886.556            |
| 5  | menos que 40K | 4099.476             |
| 6  | na            | 10945.369            


### Análise de Risco por Faixa Etária

Vamos analisar como o risco de crédito pode variar por faixa etária dos clientes:

```sql
-- Análise de risco por faixa etária
SELECT
    faixa_etaria,
    AVG(limite_credito) AS media_limite_credito,
    AVG(valor_transacoes_12m) AS media_valor_transacoes
FROM
    (
    SELECT
        *,
        CASE
            WHEN idade BETWEEN 18 AND 30 THEN '18-30'
            WHEN idade BETWEEN 31 AND 40 THEN '31-40'
            WHEN idade BETWEEN 41 AND 50 THEN '41-50'
            WHEN idade BETWEEN 51 AND 60 THEN '51-60'
            ELSE 'Mais de 60'
        END AS faixa_etaria
    FROM
        default.credito
    ) AS faixa_etaria_dados
GROUP BY
    faixa_etaria
ORDER BY
    faixa_etaria;
```

Esta consulta calcula a média do limite de crédito e do valor das transações nos últimos 12 meses para cada faixa etária, ajudando a identificar padrões de comportamento que possam indicar diferentes níveis de risco.

| # | faixa_etaria | media_limite_credito | media_valor_transacoes |
|---|--------------|----------------------|------------------------|
| 1 | 18-30        | 4960.852             | 2239.3499              |
| 2 | 31-40        | 8296.331             | 2076.37                |
| 3 | 41-50        | 11452.195            | 1710.2089              |
| 4 | 51-60        | 9095.714             | 1628.5142              |
| 5 | Mais de 60   | 5388.6206            | 1704.9979              |


### Análise de Risco por Sexo e Número de Dependentes

Vamos explorar como o risco de crédito pode variar com base no sexo e no número de dependentes dos clientes:

```sql
-- Análise de risco por sexo e número de dependentes
SELECT
    sexo,
    dependentes,
    AVG(limite_credito) AS media_limite_credito,
    AVG(valor_transacoes_12m) AS media_valor_transacoes
FROM
    default.credito
GROUP BY
    sexo,
    dependentes
ORDER BY
    sexo,
    dependentes;
```

Esta consulta calcula a média do limite de crédito e do valor das transações para cada combinação de sexo e número de dependentes, permitindo uma análise detalhada do comportamento de crédito associado a essas características.

| #  | sexo | dependentes | media_limite_credito | media_valor_transacoes |
|----|------|-------------|----------------------|------------------------|
| 1  | F    | 0           | 4587.272             | 1961.1945              |
| 2  | F    | 1           | 5612.6577            | 1898.8469              |
| 3  | F    | 2           | 5723.157             | 1806.2113              |
| 4  | F    | 3           | 6101.881             | 1837.6248              |
| 5  | F    | 4           | 6652.548             | 1734.9519              |
| 6  | F    | 5           | 6009.498             | 1578.701               |
| 7  | M    | 0           | 6841.6245            | 1917.5116              |
| 8  | M    | 1           | 10213.064            | 1820.8241              |
| 9  | M    | 2           | 11246.514            | 1804.1416              |
| 10 | M    | 3           | 11991.875            | 1774.2754              |
| 11 | M    | 4           | 14869.42             | 1792.2384              |
| 12 | M    | 5           | 14685.987            | 1615.31                |


### Análise de Risco por Faixa Salarial

Vamos investigar como o risco de crédito pode variar com base na faixa salarial dos clientes:

```sql
-- Análise de risco por faixa salarial
SELECT
    salario_anual,
    AVG(limite_credito) AS media_limite_credito,
    AVG(valor_transacoes_12m) AS media_valor_transacoes
FROM
    default.credito
GROUP BY
    salario_anual
ORDER BY
    salario_anual;
```

Esta consulta calcula a média do limite de crédito e do valor das transações para cada faixa salarial, ajudando a entender como o comportamento de crédito pode ser influenciado pelo nível de renda dos clientes.

| #  | salario_anual   | media_limite_credito | media_valor_transacoes |
|----|-----------------|----------------------|------------------------|
| 1  | 120K +         | 17801.488            | 1701.4652              |
| 2  | 40K - 60K     | 5348.356             | 1838.2643              |
| 3  | 60K - 80K     | 9096.028             | 1818.6364              |
| 4  | 80K - 120K    | 14886.556            | 1755.2499              |
| 5  | menos que 40K  | 4099.476             | 1862.7195              |
| 6  | na              | 10945.369            | 1908.8855              |



## Análise de Dados

### Distribuição por Sexo

Visualizamos a distribuição de clientes por sexo:

![distribuicao_por_sexo](https://github.com/Bruxteclas/Analise-Credito-SQL-AWS/assets/144251717/c2186518-33f3-4b51-bc07-684053de1526)


### Relação entre Idade e Limite de Crédito

Analisamos a relação entre a idade dos clientes e seus limites de crédito:

![media_limite_credito_faixa_etaria](https://github.com/Bruxteclas/Analise-Credito-SQL-AWS/assets/144251717/b95f9254-0dd5-47eb-b682-404560d9939b)


### Relação entre Limite de Crédito e Valor de Transações

Investigamos como o limite de crédito concedido está relacionado com o valor total das transações realizadas:

![relacao_limite_credito_valor_transacoes](https://github.com/Bruxteclas/Analise-Credito-SQL-AWS/assets/144251717/6e84235f-99fc-4f05-8e06-4d616aaeb7fb)

### Conclusão

**Insights Extraídos:**

Com base nas análises realizadas, aqui estão alguns insights que podem ser identificados:

1. **Faixa Etária**:
   - Clientes mais jovens (18-30 anos) tendem a ter limites de crédito mais baixos em comparação com clientes de faixas etárias mais altas (41-50 anos e 51-60 anos).
   - Não há uma clara correlação entre faixa etária e valor médio das transações, mas clientes mais jovens podem ter valores de transações mais elevados, possivelmente devido a um uso mais frequente do crédito.

2. **Sexo e Número de Dependentes**:
   - Os clientes do sexo masculino geralmente têm limites de crédito mais altos do que os clientes do sexo feminino, especialmente em faixas com maior número de dependentes.
   - No entanto, o valor médio das transações não mostra uma diferença significativa entre os sexos.

3. **Faixa Salarial**:
   - Clientes com faixas salariais mais altas ($120K +) têm os maiores limites de crédito médios.
   - A faixa salarial também influencia o valor médio das transações, com clientes de faixas salariais mais altas geralmente realizando transações de valores mais altos.

4. **Comportamento de Crédito**:
   - Clientes com limites de crédito mais altos tendem a realizar transações de valores mais altos, o que pode indicar uma maior capacidade de gastos e maior confiança no crédito.

5. **Risco de Crédito Potencial**:
   - Os clientes mais jovens (18-30 anos) podem representar um risco maior de crédito devido a limites de crédito mais baixos e possivelmente transações de valor mais alto, indicando um potencial maior de endividamento.
   - Clientes com salários mais baixos (<$40K) têm limites de crédito mais baixos, o que pode indicar uma maior restrição financeira e possivelmente maior risco de inadimplência.

Esses insights ajudam a identificar padrões e tendências nos dados dos clientes que podem ser fundamentais para avaliações de risco de crédito.
