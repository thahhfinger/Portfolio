# Portfólio de Análise de Dados 
------
### Apresentação Pessoal
Olá, me chamo Thalytá e seja bem-vindo ao meu Portfólio!
Sou uma jovem profissional em busca de transição para a área de dados, baseado em meus conhecimentos adquiridos com diversos cursos e com meu MBA em AI, Data Science e Big Data, assim como meu Tecnólogo em Marketing. 

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
Iniciei o portfólio pela procura de um dataset que fizesse sentido com a análise que queria demonstrar sobre o consumo brasileiro, encontrei no site **Kaggle** o dataset real de mais de 20 clientes e seus gastos em cartão de crédito em um banco brasileiro. Após a busca, comecei a limpeza de dados no **Excel**, utilizando o **Power Query**. Nessa etapa filtrei informações que não faziam sentido, renomei uma coluna que estava errada, alterei os tipos de colunas para data, moeda, texto, etc. Deixei as cidades com letras maiúsculas iniciais e tirei caracteres especiais:
<br><br>
<img width="800" height="500" alt="Captura de Tela (6)" src="https://github.com/user-attachments/assets/002f85b4-7b27-4d04-9311-fa748e84a07b" />

----
### 👀 Visualização
No **Power BI**, criei colunas de limite usado e faixa etária para análise posterior, utilizando a **Linguagem M** no **Power Query**: 
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
Realizei medidas com **DAX**:
<br><br>
<img width="361" height="166" alt="image" src="https://github.com/user-attachments/assets/c599b7fc-d2fc-474f-a750-d699cbbcf33c" />
<br><br>
<img width="255" height="43" alt="image" src="https://github.com/user-attachments/assets/bf71cc13-e34a-474d-9b01-0f056db05642" />
<br><br>
A partir de insights que tive, contrui um dashboard, utilizando gráficos de barra, cartões, segmentação de dados, treemap, gráfico de linhas e de pizza:
<br><br>
<img width="870" height="429" alt="Captura de tela 2025-08-28 132943" src="https://github.com/user-attachments/assets/daa6ab9b-1e5a-4463-b74f-a451a1dfbc11" />

[Link do Dashboard](Portfolio.pbix)

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
    MIN(idade) AS min_idade,
    MAX(idade) AS max_idade,
    ROUND(AVG(limite_total), 2) AS media_limite_total,
    ROUND(AVG(limite_disp), 2) AS media_limite_disp
FROM portfolio;
```
| media_idade | min_idade | max_idade | media_limite_total | media_limite_disp |
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
Também utilizei **Python**, para elaboração de gráficos e dados estatísticos, usando a ferramenta **Jupyter Notebook**. Importei o csv através da biblioteca do **Pandas** e consultei as 3 primeiras linhas:

```python
import pandas as pd
df = pd.read_csv('C:\\dataset_csv.csv', sep=';', encoding='utf-8')
print(df.head(3))
```
| id           | filial | cidade                  | estado | idade | sexo | limite_total | limite_disp | data       | valor | grupo_estabelecimento | cidade_estabelecimento | pais_estabelecimento |
|--------------|--------|------------------------|--------|-------|------|--------------|-------------|------------|-------|---------------------|-----------------------|--------------------|
| 453000000000 | 201405 | Campo Limpo Paulista    | SP     | 37    | F    | 4700         | 5605        | 04/12/2019 | 31    | Servico             | Sao Paulo             | BR                 |
| 453000000000 | 201405 | Campo Limpo Paulista    | SP     | 37    | F    | 4700         | 5343        | 09/11/2019 | 150   | Farmacias           | Santos                | BR                 |
| 453000000000 | 201405 | Campo Limpo Paulista    | SP     | 37    | F    | 4700         | 2829        | 06/05/2019 | 50    | Servico             | Sao Paulo             | BR                 |

<br><br>

Realizei uma consulta para visualizar os gastos de acordo com as categorias de estabelecimento:

```python
extremos_por_estabelecimento = df.groupby('grupo_estabelecimento')['valor'].agg(['min', 'max']).reset_index()
print(extremos_por_estabelecimento)
```
| grupo_estabelecimento | min  | max   |
|----------------------|------|-------|
| Agencia De Tur       | 4    | 3800  |
| Alug De Carros       | 700  | 700   |
| Artigos Eletro       | 1    | 10602 |
| Auto Pecas           | 30   | 3598  |
| Cia Aereas           | 20   | 14618 |

<details>
<summary>Ver tabela completa</summary>

| grupo_estabelecimento | min  | max   |
|----------------------|------|-------|
| Farmacias            | 5    | 474   |
| Hosp E Clinica       | 35   | 1400  |
| Hoteis               | 20   | 10425 |
| Joalheria            | 150  | 1025  |
| Loja De Depart       | 7    | 2299  |
| Mat Construcao       | 10   | 2596  |
| Moveis E Decor       | 7    | 3405  |
| Online               | 1    | 8445  |
| Outros               | 728  | 9909  |
| Posto De Gas         | 6    | 250   |
| Restaurante          | 1    | 4947  |
| Sem Ramo             | 9    | 188   |
| Servico              | 0    | 4776  |
| Supermercados        | 2    | 1033  |
| Varejo               | 2    | 5161  |
| Vestuario            | 13   | 1818  |

</details>

<br><br>
Criei gráficos usando as bibliotecas **Matplotlib** e **Seaborn**:

**Histograma**
<br>
```python
import matplotlib.pyplot as plt
import seaborn as sns

df_clientes = df.groupby('id', as_index=False).agg({'idade':'first'})

sns.histplot(df_clientes['idade'], kde=True)
plt.ylabel('Quantidade de clientes')
plt.xlabel('Idades')
plt.title('Quantidade de clientes por idade')
plt.show()
```

<img width="554" height="453" alt="histograma1" src="https://github.com/user-attachments/assets/1122d503-6e15-4bc8-947b-59f33816fa3a" />
<br><br><br>

**Gráfico de Barras**

<br>

```python
df_clientes = df.groupby('id', as_index=False).agg({'sexo':'first'})

sns.countplot(x='sexo', data=df_clientes)
plt.ylabel('Quantidade')
plt.xlabel('Sexo')
plt.title('Quantidade de clientes por sexo')
plt.show()
```

<img width="576" height="453" alt="grafico1" src="https://github.com/user-attachments/assets/aa395cbb-bdc0-402e-b9b8-61f2c24b34c1" />
<br><br><br>

```python
df.groupby('sexo')['idade'].mean().plot(kind='bar')
plt.xticks(rotation=0) 
plt.ylabel('Idade')
plt.xlabel('Sexo')
plt.title('Média de idade por sexo')
plt.show()
```

<img width="563" height="454" alt="grafico2" src="https://github.com/user-attachments/assets/1c77f9d0-dee9-45f3-96c0-dcd779cf9676" />
<br><br><br>

**Correlação e Gráfico de Dispersão**

<br>

```python
print("Correlação entre limite total e compras:")
print(df['limite_total'].corr(df['valor']))

plt.scatter(df['limite_total'], df['valor'])
plt.xscale('linear')
plt.ylabel('Compras')
plt.xlabel('Limite Total')
plt.title("Correlação entre o limite total e compras")
plt.show()
```

<img width="492" height="398" alt="image" src="https://github.com/user-attachments/assets/5a28d9e2-da3a-4b5f-b70b-f4e893777367" />
<br><br><br>

```python
gasto_categoria = df.groupby('grupo_estabelecimento')['valor'].sum().sort_values(ascending=False).head(5).reset_index()

sns.barplot(data=gasto_categoria, x='grupo_estabelecimento', y='valor')
plt.title("Top 5 categorias que mais gastaram")
plt.ylabel("Valor Gasto")
plt.xlabel("Categorias")
plt.show()
```

<img width="589" height="453" alt="grafico3" src="https://github.com/user-attachments/assets/76489eb7-0f82-42b5-a921-45b7c3aeacd3" />
<br><br><br>

```python
ticket_medio = df.groupby(['idade', 'sexo'])['valor'].mean().reset_index()

sns.barplot(data=ticket_medio, x='idade', y='valor', hue='sexo')
plt.title('Ticket médio por idade e sexo')
plt.ylabel('Ticket médio')
plt.xlabel('Idade')
plt.legend(title='Sexo')
```

<img width="571" height="454" alt="grafico4" src="https://github.com/user-attachments/assets/077e443c-92fc-4988-9559-e61603bcf31e" />
<br><br><br>

Fiz cálculos estátisticos com as bibliotecas **Pandas** e **Scipy**:

**Calculando média, desvio padrão e coeficiente de variação de valores por gênero**

```pyhton
estatisticas = df.groupby('sexo')['valor'].agg(['mean', 'std']).reset_index()
estatisticas['cv_percent'] = (estatisticas['std'] / estatisticas['mean']) * 100

for index, row in estatisticas.iterrows():
    print(f"Sexo: {row['sexo']}")
    print(f"  Média: {row['mean']:.2f}")
    print(f"  Desvio padrão: {row['std']:.2f}")
    print(f"  Coeficiente de variação: {row['cv_percent']:.2f}%\n")
```
<img width="253" height="138" alt="image" src="https://github.com/user-attachments/assets/69b9f0cb-de68-4119-b4a5-ed87a0eea19c" />
<br><br><br>

```python
from scipy import stats

def faixa_idade(idade):
    if 18 <= idade <= 25:
        return '18-25'
    elif 26 <= idade <= 35:
        return '26-35'
    else:
        return 'outros'

df['faixa_etaria'] = df['idade'].apply(faixa_idade)

grupo1 = df[df['faixa_etaria'] == '18-25']['valor']
grupo2 = df[df['faixa_etaria'] == '26-35']['valor']

t_stat, p_value = stats.ttest_ind(grupo1, grupo2, equal_var=False)

print(f"t-statistic: {t_stat:.4f}")
print(f"p-value: {p_value:.4f}")

if p_value < 0.05:
    print("Diferença significativa entre as médias das faixas etárias.")
else:
    print("Não há diferença significativa entre as médias das faixas etárias.")
```
<img width="386" height="50" alt="image" src="https://github.com/user-attachments/assets/47e717fa-37b4-460d-bdf9-4bbb34922bd0" />

    
<br><br><br>

**Calculando média e desvio padrão por categorias de estabelecimento**
```python
estatisticas2 = df.groupby('grupo_estabelecimento')['valor'].agg(['mean', 'std']).reset_index()
estatisticas2['cv_percent'] = (estatisticas2['std'] / estatisticas2['mean']) * 100

for index, row in estatisticas.iterrows():
    print(f"Estabelecimento: {row['grupo_estabelecimento']}")
    print(f"  Média: {row['mean']:.2f}")
    print(f"  Desvio padrão: {row['std']:.2f}")
    print(f"  Coeficiente de variação: {row['cv_percent']:.2f}%\n")
```
<img width="242" height="401" alt="image" src="https://github.com/user-attachments/assets/264e7a02-0611-4f6e-89df-c2e9eaca9591" />
<br><br><br>

### ✔️ Conclusões
- O limite utilizado por gênero é similar, mas, apesar de haver mais registros de clientes mulheres no dataset, os homens possuem limites muito maiores, cerca de 195%. Com o passar da idade, essa diferença se acentua, enquanto os limites das mulheres tendem a se manter quase estagnados.
- O ticket médio geral é relativamente baixo, um pouco acima de R$100. Porém, o maior ticket, de um cliente homem de 43 anos, é bem superior, chegando a 6 vezes esse valor. Ainda assim, os gastos de ambos os gêneros são bastante irregulares, com os homens apresentando dispersão de até 5 vezes a sua média.
- Serviços, varejo e restaurantes são os estabelecimentos com maior volume de vendas, mas o ticket médio mais alto aparece em companhias aéreas e agências de turismo, o que faz sentido por serem compras geralmente de maior valor.
- A média de idade dos clientes gira em torno dos 30 anos e, nessa base, o limite disponível corresponde a aproximadamente 75% do limite total concedido.
- A cidade com maior volume de compras é, de forma destacada, São Paulo, seguida pelo Rio de Janeiro, possivelmente devido ao tamanho das cidades e ao custo de vida.
- Percebe-se também que a relação entre limite e gasto pode variar bastante dependendo do limite disponível, não havendo um crescimento proporcional direto.
- Por fim, há uma diferença significativa entre as médias de valores gastos nas faixas etárias de 18–25 e 26–35 anos.

### 📞 Informações de Contato:
[Meu LinkedIn](https://www.linkedin.com/in/thalyt%C3%A1-finger-linhares-8124b11a2)

Meu Email: thahh.finger@gmail.com
