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
| de R$ 4.001 a R$ 6.000/mês | 593 |
| de R$ 16.001 a R$ 20.000/mês | 456 |

## Análise exploratória

- **Tecnologias usadas juntas:** os pares de variáveis mais correlacionados são ferramentas que aparecem no mesmo contexto, como Java e JavaScript (0,88), as respostas sobre Databricks (0,84) e Looker e Looker Studio (0,81).
- **Correlações com a faixa salarial** (faixas numeradas da mais baixa para a mais alta): as variáveis mais associadas a salários maiores são a idade (0,40), trabalhar como engenheiro(a) de dados, cientista de dados ou engenheiro(a) de ML/IA (cerca de 0,33) e fazer gestão de pessoas (0,30). As mais associadas a salários menores são o uso de planilhas (-0,24), de IA generativa gratuita para produtividade (-0,21) e de Power BI (-0,20).
- O notebook inclui o mapa de correlação completo e os gráficos das variáveis mais associadas à faixa salarial.

## Modelagem

Todos os modelos usam **Random Forest**, com separação estratificada em 80% treino e 20% teste e `random_state=42`. Entram na modelagem os 4.863 respondentes que informaram a faixa salarial (973 no teste), e as variáveis são as 324 colunas numéricas da pesquisa.

| Abordagem | Acurácia no teste | F1 ponderado |
| --- | --- | --- |
| 13 faixas, 100 árvores | 27,5% | 0,24 |
| 13 faixas, 50 árvores com pesos balanceados | 26,3% | 0,24 |
| 13 faixas com subamostragem da classe majoritária (`RandomUnderSampler`) | 15,2% | 0,15 |
| **3 macrofaixas** (até R$ 4 mil, R$ 4 a 12 mil, acima de R$ 12 mil), todas as variáveis | **70,1%** | **0,68** |
| 3 macrofaixas, 13 variáveis selecionadas por correlação, ajuste de hiperparâmetros | 59,9% | 0,60 |
| **5 faixas**, 10 variáveis selecionadas por correlação, ajuste de hiperparâmetros | **39,9%** | **0,41** |

### Resultado por macrofaixa (3 classes, todas as variáveis)

| Macrofaixa | Precisão | Recall | F1 |
| --- | --- | --- | --- |
| Base (até R$ 4 mil) | 0,68 | 0,26 | 0,38 |
| Intermediário (R$ 4 a 12 mil) | 0,65 | 0,85 | 0,74 |
| Especialista (acima de R$ 12 mil) | 0,80 | 0,68 | 0,73 |

### Ajuste de hiperparâmetros

Busca em grade (`GridSearchCV`) com validação cruzada de 5 partições (36 combinações, 180 treinos), otimizando o F1 ponderado. Melhores parâmetros na versão de 5 faixas: `n_estimators=300`, `max_depth=10`, `min_samples_leaf=1`, `class_weight='balanced'` (F1 médio na validação cruzada: 0,40).

## O que o projeto mostra

- Agrupar as 13 faixas em poucas macrofaixas melhora muito a capacidade de previsão: de cerca de 28% de acurácia com 13 faixas para 70% com três.
- A subamostragem da classe majoritária piorou o resultado nesta base.
- Selecionar poucas variáveis pela correlação não ajudou: com as três macrofaixas, usar todas as variáveis foi melhor.
- Os grupos mais distantes entre si (base e especialista) são mais fáceis de separar do que as faixas intermediárias vizinhas.

### Correções em relação à primeira versão

A primeira versão do notebook mostrava 31% (14 classes), 71% (macrofaixas) e 63% (6 classes). Esses números tinham três problemas, corrigidos nesta versão:

- **Vazamento do alvo no modelo final:** a seleção de variáveis do modelo de 6 classes incluía a macrofaixa codificada, que é calculada a partir da própria faixa salarial. Sem ela, o modelo equivalente (5 faixas) acerta 40%, e não 63%.
- **Quem não informou o salário:** os 354 respondentes sem faixa salarial entravam como uma classe a mais e, nas macrofaixas, eram contados como "Especialista". Agora ficam fora da modelagem.
- **Ordem das faixas:** as faixas eram numeradas em ordem alfabética ("Acima de R$ 40.001" vinha antes de "Menos de R$ 1.000"), e a correlação com esse número não indicava ganhar mais ou menos. Agora a numeração segue o valor do salário, o que muda as variáveis mais correlacionadas.

## Limitações

- Os dados são de **autodeclaração** e de uma amostra de quem respondeu à pesquisa, que pode não representar todos os profissionais de dados do país.
- Os modelos usam uma única divisão treino/teste, e a validação cruzada só foi aplicada na etapa de ajuste.
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
