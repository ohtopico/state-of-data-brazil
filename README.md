# State of Data Brazil 2024: análise exploratória e previsão de faixa salarial

Análise da pesquisa **State of Data Brazil 2024** (disponível no Kaggle), que mapeia o perfil de profissionais de dados no Brasil. O projeto combina uma **análise exploratória** dos respondentes com a **modelagem de classificação** da faixa salarial, comparando diferentes formas de agrupar as faixas e de selecionar variáveis.

## Pergunta do projeto

> Que características dos profissionais de dados (cargo, ferramentas, tarefas, expectativas de carreira) se relacionam com a faixa salarial, e o quanto é possível prever essa faixa a partir delas?

## Dados

- **Fonte:** State of Data Brazil 2024 (arquivo `Final Dataset - State of Data 2024 - Kaggle - df_survey_2024.csv`)
- **Tamanho:** 5.217 respondentes e 403 colunas
- **Variável-alvo:** `2.h_faixa_salarial`, com 13 faixas, de "Menos de R$ 1.000/mês" a "Acima de R$ 40.001/mês"

### Perfil dos respondentes

- Idades de 18 a 68 anos
- Cerca de 75% do sexo masculino e cerca de 67% brancos
- Entre os que não vivem no Brasil, a maioria mora em Portugal
- 30% em cargos de nível sênior e 21% em cargos de gestão
- 180 pessoas com deficiência (cerca de 3,45%)

### Faixas salariais mais frequentes

| Faixa | Respondentes |
| --- | --- |
| de R$ 8.001 a R$ 12.000/mês | 1.080 |
| de R$ 12.001 a R$ 16.000/mês | 716 |
| de R$ 6.001 a R$ 8.000/mês | 656 |
| de R$ 2.001 a R$ 6.000/mês | 593 |
| de R$ 16.001 a R$ 20.000/mês | 456 |

## Análise exploratória

- **Tecnologias usadas juntas:** os pares de variáveis mais correlacionados são ferramentas que aparecem no mesmo contexto, como Java e JavaScript (0,88), as respostas sobre Databricks (0,84) e Looker e Looker Studio (0,81).
- **Correlações com a faixa salarial:** a variável mais associada a faixas mais altas foi criar consultas em SQL para compartilhar informações com as áreas de negócio (cerca de 0,13 a 0,19, conforme o recorte), seguida pelo uso de planilhas e pela senioridade das vagas recebidas.
- O notebook inclui o mapa de correlação completo e os gráficos das variáveis mais associadas à faixa salarial.

## Modelagem

Todos os modelos usam **Random Forest**, com separação estratificada em 80% treino e 20% teste (1.044 respondentes no teste) e `random_state=42`.

| Abordagem | Resultado no teste |
| --- | --- |
| 14 classes (13 faixas + sem faixa informada), 100 árvores | acurácia de 31,23% |
| 14 classes, 50 árvores com pesos balanceados | acurácia de 30,46% |
| 14 classes com subamostragem da classe majoritária (`RandomUnderSampler`) | acurácia de 20,59% |
| **3 macrofaixas** (até R$ 4 mil, R$ 4 a 12 mil, acima de R$ 12 mil) | **acurácia de 71%**, F1 ponderado de 0,69 |
| **6 classes agrupadas, variáveis selecionadas por correlação, ajuste de hiperparâmetros** | **acurácia de 63%**, F1 ponderado de 0,56 (F1 médio na validação cruzada: 0,54) |

### Resultado por macrofaixa (3 classes)

| Macrofaixa | Precisão | Recall | F1 |
| --- | --- | --- | --- |
| Base (até R$ 4 mil) | 0,70 | 0,24 | 0,35 |
| Intermediário (R$ 4 a 12 mil) | 0,64 | 0,85 | 0,73 |
| Especialista (acima de R$ 12 mil) | 0,83 | 0,70 | 0,76 |

### Ajuste de hiperparâmetros

Busca em grade (`GridSearchCV`) com validação cruzada de 5 partições (36 combinações, 180 treinos), otimizando o F1 ponderado. Melhores parâmetros na versão de 6 classes: `n_estimators=100`, `max_depth=10`, `min_samples_leaf=1`, `class_weight='balanced'`.

## O que o projeto mostra

- Agrupar as 13 faixas em poucas macrofaixas melhora muito a capacidade de previsão: de cerca de 31% de acurácia com muitas classes para 71% com três.
- A subamostragem da classe majoritária piorou o resultado nesta base.
- Os grupos mais distantes entre si (base e especialista) são mais fáceis de separar do que as faixas intermediárias vizinhas.

## Limitações

- Os dados são de **autodeclaração** e de uma amostra de quem respondeu à pesquisa, que pode não representar todos os profissionais de dados do país.
- Os modelos usam uma única divisão treino/teste, e a validação cruzada só foi aplicada na etapa de ajuste.
- Uma etapa de seleção das 10 principais variáveis chegou a 99% de acurácia porque a própria faixa salarial codificada entrou como variável preditora (vazamento do alvo). Esse resultado foi **descartado** e não está na tabela acima.
- Faixas salariais vizinhas se confundem, então as macrofaixas são mais confiáveis do que a previsão detalhada.

## Tecnologias

Python, pandas, NumPy, seaborn, matplotlib, Plotly, scikit-learn (Random Forest, `GridSearchCV`) e imbalanced-learn.

## Como executar

1. Baixe o arquivo da pesquisa no Kaggle e coloque-o na mesma pasta do notebook, com o nome `Final Dataset - State of Data 2024 - Kaggle - df_survey_2024.csv`.
2. Instale as dependências:
   ```bash
   pip install pandas numpy seaborn matplotlib plotly scikit-learn imbalanced-learn jupyter
   ```
3. Abra e execute `State_of_Data_Brazil.ipynb`.

## Autora

**Júlia Roberta Quoos Alves**: engenheira de dados e mestranda em Ciência da Computação (UFSM).
[LinkedIn](https://www.linkedin.com/in/ohtopico) · [Site](http://ohtopico.com.br)
