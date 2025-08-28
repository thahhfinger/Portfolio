# Portfólio de Análise de Dados 
------
## Projeto: Análise de gastos em cartão de credito no brasil 
Este portfólio foi desenvolvido para demonstrar as principais habilidades necessárias para o trabalho de um Analista de Dados. Visando especialmente realizar uma análise baseada nos gastos que o brasileiro tem em cartão de crédito, buscando entender a relação de consumo e idade/gênero/região. 

### 🔧 Ferramentas Utilizadas
- [Dataset real de clientes e transações](https://www.kaggle.com/datasets/sufyant/brazilian-real-bank-dataset/data)
- Excel/Power Query
- Power BI
- PostgreSQL
- Jupyter Notebook (Python)
- Bibliotecas do Python: Pandas, Matplotlib, Seaborn e Scipy
-----
### 🧹 Extração e Limpeza
Iniciei o portfólio pela procura de um dataset que fizesse sentido com a análise que queria demonstrar sobre o consumo brasileiro, encontrei no site **Kaggle** o dataset real de mais de 20 clientes e seus gastos em cartão de crédito em um banco brasileiro. Após a busca, comecei a limpeza de dados no **Excel**, utilizando o **Power Query**:
<br><br>
<img width="800" height="500" alt="Captura de Tela (6)" src="https://github.com/user-attachments/assets/3342a842-345f-48c6-ba79-479eba9c13c1" />
<br><br>
Nessa etapa filtrei informações que não faziam sentido, renomei uma coluna que estava errada, alterei os tipos de colunas para data, moeda, texto, etc. Deixei as cidades com letras maiúsculas iniciais e tirei caracteres especiais. 
<br><br>
<img width="800" height="500" alt="Captura de Tela (6)" src="https://github.com/user-attachments/assets/002f85b4-7b27-4d04-9311-fa748e84a07b" />

----
### 👀 Visualização
No **Power BI**, criei colunas de limite usado e faixa etária para análise posterior, também utilizando **Power Query**: 
<br><br>
<img width="800" height="500" alt="Captura de Tela (7)" src="https://github.com/user-attachments/assets/c12476da-cac4-46a0-a48c-a8d000f12e03" />
<br><br>
<img width="800" height="500" alt="Captura de Tela (10)" src="https://github.com/user-attachments/assets/40ed3463-7ede-472c-a561-cb43c71360f0" />
<br><br>
Depois, criei uma coluna calendário e relacionei com o dataset:
<br><br>
<img width="800" height="500" alt="Captura de Tela (11)" src="https://github.com/user-attachments/assets/b32ab484-7348-4a1f-b034-18db9360abc3" />
<br><br>
<img width="698" height="430" alt="Captura de Tela (12)" src="https://github.com/user-attachments/assets/67ce7792-83cd-4c33-a544-6e430b5a71b2" />
<br><br>
A partir de insishts que tive, contrui um dashboard, utilizando gráficos de barra, cartões, segmentação de dados, treemap, gráfico de linhas e de pizza:
<br><br>
<img width="1021" height="490" alt="Captura de Tela (49)" src="https://github.com/user-attachments/assets/d15c3ddf-1ac7-4020-854c-0eaef456a938" />

----
### 🔍 Consulta
Utilizei o PostgreSQL para realizar consultas, a partir de um csv do dataset já limpo. Criei uma tabela com o cabeçalho e importei o csv:

```sql
CREATE TABLE portfolio (
    id BIGINT,
    filial TEXT,
    cidade TEXT,
    estado TEXT,
    idade INT,
    sexo TEXT,
    limite_total NUMERIC(12,2),
    limite_disp NUMERIC(12,2),
    data date,
    valor NUMERIC (12,2),
    grupo_estabelecimento TEXT,
    cidade_estabelecimento TEXT,
    pais_estabelecimento TEXT
);
```
<br><br>
Queries utilizadas:

```sql
SELECT id, estado, COUNT(id) AS total_compras
FROM portfolio
GROUP BY id, estado
ORDER BY estado, total_compras DESC
LIMIT 5;
```
| id           | estado | total_compras |
|--------------|--------|---------------|
| 221000000000 | RJ     | 268           |
| 502000000000 | SP     | 694           |
| 651000000000 | SP     | 510           |
| 331000000000 | SP     | 457           |
| 94873707154  | SP     | 257           |

<br><br>

```sql
SELECT 
    ROUND(AVG(idade)) AS media_idade,
    MIN(idade) AS idade_min,
    MAX(idade) AS idade_max,
    ROUND(AVG(limite_total), 2) AS media_limite_total,
    ROUND(AVG(limite_disp), 2) AS media_limite_disp
FROM portfolio;
```
| media_idade | idade_min | idade_max | media_limite_total | media_limite_disp |
|-------------|-----------|-----------|------------------|-----------------|
| 34          | 20        | 53        | 8727.60          | 6557.14         |

<br><br>

```sql
SELECT sexo, COUNT(DISTINCT id) AS total_clientes
FROM portfolio
GROUP BY sexo;
```
| sexo | total_clientes |
|------|----------------|
| F    | 18             |
| M    | 10             |

<br><br>

```sql
SELECT sexo, ROUND(AVG(limite_total),2) AS limite_medio
FROM portfolio
GROUP BY sexo;
```
| sexo | limite_medio |
|------|--------------|
| M    | 13453.33     |
| F    | 4554.29      |
<br>

**Observação**: Nota-se que apesar de ter mais registros de mulheres no dataset, o limite médio é bem maior para os homens.
<br><br>

```sql
SELECT cidade_estabelecimento, SUM(valor) AS total_gasto
FROM portfolio
WHERE cidade_estabelecimento != 'Desconhecido'
GROUP BY cidade_estabelecimento
ORDER BY total_gasto DESC
LIMIT 5;
```
| cidade_estabelecimento | total_gasto |
|------------------------|------------|
| Sao Paulo              | 280408.00  |
| Rio De Janeir          | 45273.00   |
| Osasco                 | 31876.00   |
| Sao Jose Do R          | 14281.00   |
| Santo Andre            | 12209.00   |

<br><br>
Também realizei uma CTE, para fazer faixas etárias:

```sql
WITH dados_clientes AS (
    SELECT *,
        CASE 
            WHEN idade BETWEEN 20 AND 29 THEN '20-29'
            WHEN idade BETWEEN 30 AND 39 THEN '30-39'
            WHEN idade BETWEEN 40 AND 49 THEN '40-49'
            ELSE '50+' 
        END AS faixa_etaria
    FROM portfolio
)
SELECT faixa_etaria, sexo, ROUND(AVG(limite_total), 2) AS limite_medio
FROM dados_clientes
GROUP BY faixa_etaria, sexo
ORDER BY faixa_etaria, sexo;
```
| faixa_etaria | sexo | limite_medio |
|--------------|------|--------------|
| 20-29        | F    | 4175.61      |
| 20-29        | M    | 3937.18      |
| 30-39        | F    | 4351.72      |
| 30-39        | M    | 8891.61      |
| 40-49        | F    | 4906.50      |
| 40-49        | M    | 37511.80     |
| 50+          | F    | 9550.00      |
| 50+          | M    | 1420.00      |

---
### 📈 Gráficos e Estatística
Também utilizei **Python**, para elaboração de gráficos e dados estatísticos, usando a ferramenta **Jupyter Notebook**. Importei o csv através da biblioteca do **Pandas**:

```python
import pandas as pd
df = pd.read_csv('C:\\Users\\thalyta\\Downloads\\archive\\dataset_csv.csv', sep=';', encoding='utf-8')
print(df.head())
```
             id  filial                cidade   estado  idade sexo  \
0  453000000000  201405  Campo Limpo Paulista      SP      37    F   
1  453000000000  201405  Campo Limpo Paulista      SP      37    F   
2  453000000000  201405  Campo Limpo Paulista      SP      37    F   
3  453000000000  201405  Campo Limpo Paulista      SP      37    F   
4  453000000000  201405  Campo Limpo Paulista      SP      37    F   

   limite_total  limite_disp        data  valor grupo_estabelecimento  \
0          4700         5605  04/12/2019     31               Servico   
1          4700         5343  09/11/2019    150             Farmacias   
2          4700         2829  06/05/2019     50               Servico   
3          4700         2547  01/06/2019     54                Online   
4          4700         2515  01/06/2019     33                Online   

  cidade_estabelecimento pais_estabelecimento  
0              Sao Paulo                   BR  
1                 Santos                   BR  
2              Sao Paulo                   BR  
3                 Osasco                   BR  
4                 Osasco                   BR  
