# Projeto_Saude_Mental

Este projeto foi criado com o objetivo de aplicar habilidades em Python usando Pandas, criação de métricas e criação de um Dashboard intuitivo mostrando os principais insights da base de dados.
Esta base foi extraida da plataforma Kaggle. Este conjunto de dados contém informações sobre a saúde mental, empatia e esgotamento de estudantes de medicina na Suíça. Ele inclui dados sobre fatores demográficos, como idade, sexo e idioma falado, bem como medidas internas, como satisfação no trabalho, sofrimento psicológico, notas educacionais, estado de saúde autorrelatado e escalas de classificação de empatia.

Link para o conjunto de dados: https://www.kaggle.com/datasets/thedevastator/medical-student-mental-health

## ETAPA 1

### Entendendo os dados

O conjunto de dados possui 2 arquivos: 

- **"Data Carrard et al. 2022 MedTeach"**: este arquivo contém as colunas e valores principais. No entanto, utiliza valores numéricos como identificadores. Como no exemplo abaixo:

| id | age | year | sex | glang | part | job | stud_h | health | psyt | jspe | qcae_cog | qcae_aff | amsp | erec_mean | cesd | stai_t | mbi_ex | mbi_cy | mbi_ea |
|----|-----|------|-----|-------|------|-----|--------|--------|------|------|-----------|-----------|------|-----------|------|--------|--------|--------|--------|
| 0  | 2   | 18   | 1   | 1     | 120  | 1   | 0      | 56     | 3    | 0    | 88        | 62        | 27   | 17        | 0.738 | 34     | 61     | 17     | 13     | 20     |
| 1  | 4   | 26   | 4   | 1     | 1    | 1   | 0      | 20     | 4    | 0    | 109       | 55        | 37   | 22        | 0.690 | 7      | 33     | 14     | 11     | 26     |
| 2  | 9   | 21   | 3   | 2     | 1    | 0   | 0      | 36     | 3    | 0    | 106       | 64        | 39   | 17        | 0.690 | 25     | 73     | 24     | 7      | 23     |

- **"Codebook Carrard et al. 2022 MedTeach"**: este arquivo possui a legenda dos valores, explicando o que cada valor numérico representa:

| Variable Name | Variable Label | Variable Scale |
|---------------|----------------|-----------------|
| id            | Participants ID number | string |
| age           | age at questionnaire 20-21 | numeric |
| year          | CURICULUM YEAR : In which curriculum year are ... | 1=Bmed1; 2=Bmed2; 3=Bmed3; 4=Mmed1; 5=Mmed2; 6=Mmed3 |
| sex           | GENDER : To which gender do you identify the most? | 1=Man; 2=Woman; 3=Non-binary |
| glang         | MOTHER TONGUE: What is your mother tongue? | 1=French; 15=German; 20=English; 37=Arab; 51=Basque... |
| part          | PARTNERSHIP STATUS : Do you have a partner? | 0=No; 1=Yes |
| job           | HAVING A JOB : Do you have a paid job? | 0=No; 1=Yes |


## ETAPA 2
### Preparação dos dados
#### Importação e Criação do DataFrame

```python
import pandas as pd

codebook = pd.read_csv('Codebook Carrard et al. 2022 MedTeach.csv', sep=';', engine='python')
dados = pd.read_csv('Data Carrard et al. 2022 MedTeach.csv')
df = pd.DataFrame(dados)
````

#### Substituição de Valores Usando o Codebook Como Parâmetro

Nossa base de dados contém várias variáveis representadas por valores numéricos, como gênero, ano do currículo e língua materna. Por exemplo:
- Na variável de gênero, temos `1` para "Homem", `2` para "Mulher" e `3` para "Não-binário".

Para melhorar a compreensão e a análise desses dados, decidi criar um dicionário de mapeamento. Este dicionário serve para:

1. **Converter Números em Letras**: Ele traduz os identificadores numéricos em suas representações textuais usando o Codebook como parâmetro, facilitando a leitura e interpretação dos dados.

2. **Uniformizar a Representação**: Ao usar um mapeamento consistente, garantimos que os mesmos valores sejam interpretados de forma igual em todas as análises, evitando confusões e erros.

3. **Aumentar a Eficiência nas Análises**: O uso desse dicionário nos permite realizar substituições de forma rápida e organizada, tornando o nosso código mais claro e fácil de manter.


**Primeiro, excluímos colunas nulas do codebook:**
``` python
del codebook['Unnamed: 3']
del codebook['Unnamed: 4']
del codebook['Unnamed: 5']
````
**A seguir, criamos um mapeamento de valores para substituir os valores no conjunto de dados principal:**
```python
# Criar um dicionário de mapeamento
codebook['Variable Scale'] = codebook['Variable Scale'].fillna('')

mapeamento = {}

for _, row in codebook.iterrows():
  variable_name = row['Variable Name']
  variable_labels = row['Variable Scale']

  if '=' in variable_labels:
    mapeamento[variable_name] = {
        int(k.strip()): v.strip() for k, v in
        (item.split('=') for item in variable_labels.split(';'))
    }
```
Assim, inserimos no dicionário as seguintes informações:
{'year': {1: 'Bmed1', 2: 'Bmed2', 3: 'Bmed3', 4: 'Mmed1', 5: 'Mmed2', 6: 'Mmed3'}, 'sex': {1: 'Man', 2: 'Woman', 3: 'Non-binary'}, 'glang': {1: 'French', 15: 'German', 20: 'English', 37: 'Arab', 51: 'Basque'...}

**Agora, substituimos os valores no DataFrame usando o mapeamento:**
```` python
# Substituindo valores no DataFrame
for col in mapeamento.keys():
  if col in df.columns:
    df[col] = df[col].replace(mapeamento[col])

df.head()  # Mostra as primeiras linhas do DataFrame após a substituição
````

| id | age | year  | sex   | glang     | part | job | stud_h | health                             | psyt | jspe | qcae_cog | qcae_aff | amsp | erec_mean | cesd | stai_t | mbi_ex | mbi_cy | mbi_ea |
|----|-----|-------|-------|-----------|------|-----|--------|------------------------------------|------|------|-----------|-----------|------|-----------|------|--------|--------|--------|--------|
| 0  | 2   | 18    | Bmed1 | Man       | Vietnamese | Yes  | No     | 56                                  | No   | 88   | 62        | 27        | 17   | 0.738095  | 34   | 61     | 17     | 13     | 20     |
| 1  | 4   | 26    | Mmed1 | Man       | French     | Yes  | No     | 20                                  | Satisfied | 109  | 55        | 37        | 22   | 0.690476  | 7    | 33     | 14     | 11     | 26     |
| 2  | 9   | 21    | Bmed3 | Woman     | French     | No   | No     | 36                                  | No   | 106  | 64        | 39        | 17   | 0.690476  | 25   | 73     | 24     | 7      | 23     |


**Por fim, exportamos o DataFrame tratado para um novo arquivo CSV:**
```` python
df.to_csv('Dados_Saude_Emocional.csv', index=False)
````


### Notas:
#### Justificativa para a Ausência de Tratamento de Nulos e Inconsistências
Após uma análise detalhada da base de dados, verificou-se que não havia valores nulos ou inconsistentes. Dado que a integridade da base foi mantida e todos os registros estavam completos, decidi que não era necessário incluir etapas de tratamento de dados nulos no presente projeto. Essa abordagem permitiu focar em outras análises críticas sem a sobrecarga de processamento adicional que não traria um valor significativo para nossos resultados.

## ETAPA 3

### Contextualizando os dados e criando métricas

A seguir, explico as métricas que foram criadas a partir dos dados originais do conjunto. Estas métricas foram desenvolvidas no **Looker Studio** utilizando **campos calculados**, permitindo agrupar, categorizar e tirar insights relevantes para facilitar a análise dos dados.

#### Faixas de Idade (age)

Para agrupar a idade dos participantes, criamos uma categorização no campo calculado do Looker Studio que facilita a análise dos diferentes grupos etários. A categorização é feita da seguinte forma:

```sql
CASE
    WHEN age > 40 THEN "Mais de 40"
    WHEN age BETWEEN 30 AND 40 THEN "31 a 40 anos"
    WHEN age < 30 THEN "Até 30 anos"
END
````
Essa divisão permite analisar as variáveis de saúde mental com base em diferentes faixas etárias.

#### MBI Cinismo (mbi_cy)
O **MBI Cinismo** é uma medida de cinismo profissional, avaliada pelo Maslach Burnout Inventory. A categorização da escala foi realizada no campo calculado do Looker Studio com base nos seguintes intervalos:
````sql
CASE
    WHEN mbi_cy < 6 THEN "Baixo"
    WHEN mbi_cy BETWEEN 6 AND 12 THEN "Médio"
    WHEN mbi_cy > 12 THEN "Alto"
END
````
Essas faixas são usadas para entender o nível de distanciamento e desinteresse que os participantes têm em relação ao seu trabalho.

#### MBI Exaustão Emocional (mbi_ex)
O **MBI Exaustão Emocional** mede o grau de cansaço emocional e físico causado pelo estresse no trabalho. A categorização foi feita no campo calculado do Looker Studio da seguinte forma:
```sql
CASE
    WHEN mbi_ex < 19 THEN "Baixo"
    WHEN mbi_ex BETWEEN 19 AND 27 THEN "Médio"
    WHEN mbi_ex > 27 THEN "Alto"
END
````
Essa métrica ajuda a identificar o nível de desgaste emocional que os participantes experimentam.

#### MBI Realização Pessoal (mbi_ea)
O **MBI Realização Pessoal** mede o nível de satisfação e realização no trabalho. A categorização da escala é:
````sql
CASE
    WHEN mbi_ea < 24 THEN "Baixo"
    WHEN mbi_ea BETWEEN 24 AND 31 THEN "Médio"
    WHEN mbi_ea > 31 THEN "Alto"
END
````
Essa métrica permite entender o nível de satisfação profissional dos participantes.

#### Métrica Burnout
A **Métrica Burnout** foi criada combinando as três dimensões do Maslach Burnout Inventory: Exaustão Emocional, Cinismo e Realização Pessoal. As faixas de Burnout foram definidas no campo calculado do Looker Studio da seguinte maneira:
````sql
CASE
    WHEN mbi_ex = "Baixo" AND mbi_cy = "Baixo" AND mbi_ea = "Alto" THEN "Baixo Burnout"
    WHEN mbi_ex = "Médio" AND mbi_cy = "Médio" AND mbi_ea = "Médio" THEN "Médio Burnout"
    WHEN mbi_ex = "Alto" AND mbi_cy = "Alto" AND mbi_ea = "Baixo" THEN "Alto Burnout"
    ELSE "Médio Burnout"
END
````
Essa métrica é usada para entender o grau geral de burnout dos participantes, combinando os três aspectos avaliados.

#### Níveis de Depressão (cesd)
O **CESD (Center for Epidemiologic Studies Depression Scale)** é usado para medir sintomas de depressão. A categorização da escala foi realizada no campo calculado do Looker Studio da seguinte forma:
````sql
CASE 
    WHEN cesd BETWEEN 0 AND 9 THEN "1-Não Deprimido"
    WHEN cesd BETWEEN 10 AND 15 THEN "2-Leve"
    WHEN cesd BETWEEN 16 AND 24 THEN "3-Moderado"
    WHEN cesd > 24 THEN "4-Severo"
END
````
Essa métrica classifica os participantes com base na gravidade de sintomas depressivos.

#### Nível de Ansiedade (stai_t)
O **STAI (State-Trait Anxiety Inventory)** mede o nível de ansiedade dos participantes. A categorização foi feita no campo calculado do Looker Studio da seguinte forma:
````sql
CASE
    WHEN stai_t BETWEEN 20 AND 40 THEN "1-Leve"
    WHEN stai_t BETWEEN 40 AND 60 THEN "2-Moderada"
    WHEN stai_t > 60 THEN "3-Alta"
END
````
Essa métrica classifica os participantes de acordo com o nível de ansiedade que eles experienciam.

## ETAPA 4
### Estrutura do Dashboard

O dashboard está dividido em duas principais seções, cada uma abordando diferentes aspectos da saúde mental:

### 1. Distribuição e Relação com Satisfação de Saúde
Essa seção apresenta informações sobre como os indivíduos avaliam sua própria saúde e como essa percepção varia de acordo com o gênero.

- **Gráficos Utilizados:**
  - Gráficos de pizza para representar a distribuição da satisfação com a saúde para mulheres, homens e pessoas não binárias.
  - Gráfico de barras para exibir a contagem de cada nível de satisfação com a saúde.
  - Gráfico de barras comparando a satisfação com a saúde por gênero e faixa etária.

![Gráficos de Satisfação com a Saúde](https://github.com/RitzyTr/Projeto_Saude_Mental/blob/RitzyTr-patch-1/parte%201.png)

### 2. Relação de Níveis de Depressão
Essa seção analisa os níveis de depressão e sua correlação com outras variáveis, como ansiedade e faixa etária.

- **Gráficos Utilizados:**
  - Gráficos de barras empilhadas para mostrar a distribuição dos níveis de depressão por gênero.
  - Tabela para exibir a média de depressão conforme os níveis de ansiedade e faixa etária.
  - Gráfico de dispersão para ilustrar a correlação entre ansiedade e depressão (R² = 0.51).

![Gráficos de Depressão](https://github.com/RitzyTr/Projeto_Saude_Mental/blob/RitzyTr-patch-1/parte%202.png)
