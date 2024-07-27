# Identificação e Previsão de Abastecimento em Papeleira

#### Aluno: [Vinícius Souza Nascimento](https://github.com/vin94sn)
#### Orientador: [Manoela Kohler](https://github.com/manoelakohler) e [Felipe Borges](https://github.com/FelipeBorgesC).

---

Trabalho apresentado ao curso [BI MASTER](https://ica.puc-rio.ai/bi-master) como pré-requisito para conclusão de curso e obtenção de crédito na disciplina "Projetos de Sistemas Inteligentes de Apoio à Decisão".

---

### Resumo

O trabalho possui dois objetivos, o primeiro sendo identificar quando há abastecimento de papel em papeleiras de banheiro, por meio de uma base de dados da variação em porcentagem da quantidade de papel ao longo do tempo. O segundo objetivo é conseguir prever quando ocorrerá o próximo abastecimento, por meio do cruzamento de informações do nível do papel com a identificação de abastecimentos através do primeira etapa e uma segunda base de dados com a informação do fluxo de pessoas ao longo do tempo.

### Abstract

The work has two objectives, the first being to identify when there is restock of paper towels, through a database of the variation in percentage of the amount of paper over time. The second objective is to be able to predict when the next restock will occur, by crossing information from the paper level with the identification of restocks through the first step and a second database with information on the flow of people over time.

### 1. Introdução

Antes de se montar o modelo é importante entender como os dados se comportam em ambas as bases de dados utilizadas.
A base de dados sobre o fluxo de pessoas possui um comportamento padrão ao longo do tempo. A contagem se inicia meia-noite e vai aumentando gradualmente ao longo do dia até que atinge o seu pico as 23:59 e a contagem é reiniciada para o próximo dia.

<img src="https://raw.githubusercontent.com/vin94sn/BIMaster/main/Images/fluxo_pessoas.png?" alt="fluxo_pessoas" style="height: 275px; width:700px;"/>

A base de dados sobre o percentual de papel dentro da papeleira também tem um comportamento padrão. A porcentagem de papel vai descendo ao longo do tempo até que há um um salto para 100% ou valor perto. A ocorrencia desse salto no valor é o comportamento especifico do abastecimento, porém há um problema de oscilação nos valores.

<img src="https://raw.githubusercontent.com/vin94sn/BIMaster/main/Images/nivel_bruto.png?" alt="nivel" style="height: 300px; width:700px;"/>

Como é possível ver na imagem acima, devico a sensibilidade do sensor com a constante manuseio da papeleira, a leitura do sensor pode oscilar de forma consideravel, criando comportamentos que podem ser confundidos com abastecimentos.

<img src="https://raw.githubusercontent.com/vin94sn/BIMaster/main/Images/mediana_nivel.png?" alt="mediana_nivel" style="height: 300px; width:700px;"/>

Sendo que o importante para o modelo é a identificação de uma mudança brusca no nível, a base de dados será  tratada substituindo os valores brutos do nível por suas medianas. A diferença pode ser notada na imagem abaixo, onde o comportamento do gráfico continua o mesmo porém com menos oscilações nos valores.

### 2. Modelagem
#### 2.1 Modelo de Classificação

O modelo de classificação utilizará como informação de entrada o histórico do nível para o teste e treino e, para a saída, a informação do que a série de valores de nível representa. Como exemplo de como se pode classificar o comportamento do nível se tem: 

- Descida do Nível
- Valor Constante 
- Oscilação de Subida 
- Oscilação de Descida 
- Abastecimento (Refil)

Já existe um script que, utilizando diversas condições para identificar grande parte dos cenários, fornece a informação se a sequencia de valores representa um abastecimento ou outro comportamento.

Em relação ao histórico do nível, será escolhido um ponto "n" na base de dados e seus 4 valores anteriores e posteriores, como representado na imagem abaixo:

![histórico_nivel](https://raw.githubusercontent.com/vin94sn/BIMaster/main/Images/exemplo%20hist%C3%B3rico.png?)

Onde os valores de n+1 até n+4 são os valores anteriores e n-1 a n-4 os valores seguintes.

Como pode ocorrer problemas de conexão ou desligamento do equipamento, o intervalo de tempo entre duas leituras do nível pode ser muito maior que o normal de 15 minutos, chegando a horas de diferença. Por isso também será adicionado aos dados de entrada a informação de tempo médio entre cada leitura do nível e o tempo de leitura total do "n+4" até o "n-4".

A imagem abaixo representa o dataframe das informações de entrada e saída (A coluna: Status) do modelo

<img src="https://raw.githubusercontent.com/vin94sn/BIMaster/main/Images/exemplo%20input%20e%20output.png?" alt="inputs_e_outputs" style="height: 100px; width:600px;"/>

Como não se tem dados suficientes para um grande número de exemplos de cada comportamento, serão gerados dados a partir dos já existentes. Aleatóriamente, dados já classificados irão ser escolhidos e valores pequenos (exemplo: de 1 a 5) serão adicionados ou subtraídos das colunas "n+4" até "n-4". Essa geração de dados não irá interferir com o treinamento do modelo pois o que o modelo precisa aprender é o comportamento do nível.

![comportamento](https://raw.githubusercontent.com/vin94sn/BIMaster/main/Images/exemplo%20comportamento%20do%20hist%C3%B3rico.png?)

Na imagem acima foi utilizado como base para a geração de dados a linha 58. Nas linhas 59 a 61 foram somados os valores 1, 2 e 3 em todas as colunas com base nos valores da linha 58, enquanto que as linhas 62 a 64 foram subtraídos os valores 1, 2 e 3. É importante notar que apesar dos valores serem um pouco diferentes do original, o seu comportamento ao longo do tempo (da coluna "n+4" até "n-4") se mantém o mesmo. Da coluna "n+4" até a coluna "n+1" os valores foram diminuindo, o que condiz com o comportamento da papeleira, até que houve uma subida repentina no valor na coluna "n" que se manteve alto nas colunas seguintes.

Como o que está sendo analisado é o ponto na coluna "n", esse ponto seria considerado como o momento em que houve o abastecimento.

Após alguns testes do modelo se chegou na conclusão que mesmo com a geração de novos dados, não se obteve uma quantidade suficiente para o modelo aprender a classificar cada comportamento. Porém, como o objetivo é apenas identificar a ocorrencia do abastecimento, foi realizar alteração para que as informações de saída do modelo só classifique a sequencia de valores como dois comportamentos:

- Refil
- Not Refil

![modelos](https://raw.githubusercontent.com/vin94sn/BIMaster/main/Images/cria%C3%A7%C3%A3o%20dos%20modelos.png?)

Com a alteração foi possível obter uma boa proporção entre cada comportamento nos dados de treino e teste

![modelos](https://raw.githubusercontent.com/vin94sn/BIMaster/main/Images/propor%C3%A7%C3%A3o%20de%20dados.png?)

#### 2.2 Modelo de Regressão

O modelo de regressão irá fornecer como informação de entrada para um ponto "n" no tempo, a porcentagem do nível no momento e os 3 valores anteriores, assim como se o dia atual do ponto, anterior ou seguinte é feríado. Também será fornecido a informação de qual dia da semana é, a quantidade de pessoas que passaram desde o último abastecimento e quanto tempo faz desde o último abastecimento.

Como informação de saída será fornecido ao modelo o tempo até o próximo abastecimento.

O modelo:

<img src="https://github.com/vin94sn/BIMaster/blob/main/Images/modelo%20regress%C3%A3o.png?raw=true" alt="modelos" style="height: 30px; width:600px;"/>

Sendo:

- n+3, n+2, n+1, n : Como no modelo de classificação, são os valores de nível, sendo "n+3" o mais antigo
- dia : dia da semana (0 a 6)
- ontem, hj, amanha: informa com o valor "1" se é feriado ou "0" se não for.
- fluxo: fluxo acumulado de pessoas desde o último abastecimento até o tempo da amostra de "n"
- last: quanto tempo desde o último abastecimento, em horas
- next: quanto tempo até o próximo abastecimento, em horas

### 3. Resultados
#### 3.1 Resultados da Classificação

Para o modelo de classificação, foram utilizados os seguintes algoritmos:

- SVC
- Decision Tree
- Random Forest

![svc](https://github.com/vin94sn/BIMaster/blob/main/Images/svc.png?raw=true)
![decision tree](https://github.com/vin94sn/BIMaster/blob/main/Images/decision%20tree.png?raw=true)
![random forest](https://github.com/vin94sn/BIMaster/blob/main/Images/random%20forest.png?raw=true)

Todos os algoritmos mostraram resultados muito bons.

#### 3.2 Resultados da Regressão

Para o modelo de Regressão, foram utilizados os seguintes algoritmos:

- Linear Regression
- XGBRegressor
- Random Forest
- Decision Tree

<img src="https://github.com/vin94sn/BIMaster/blob/main/Images/Linear%20Regression.png?raw=true" alt="linear regression" style="height: 300px; width:300px;"/>
<img src="https://github.com/vin94sn/BIMaster/blob/main/Images/XGBR.png?raw=true" alt="XGBRegressor" style="height: 300px; width:300px;"/>
<img src="https://github.com/vin94sn/BIMaster/blob/main/Images/Decision%20Tree%20Regress%C3%A3o.png?raw=true" alt="decision tree" style="height: 300px; width:300px;"/>
<img src="https://github.com/vin94sn/BIMaster/blob/main/Images/Random%20Forest%20Regress%C3%A3o.png?raw=true" alt="random forest" style="height: 300px; width:300px;"/>

O algoritmo de Linear Regression foi o único a ter um desempenho ruim com o modelo. Em seguida foi utilizado o GridSearch para melhorar os parametros dos modelos restantes.

Foi possível identificar uma melhoria significativa no algoritmo XGBRegressor

<img src="https://github.com/vin94sn/BIMaster/blob/main/Images/XGBR%20GridSearch.png?raw=true" alt="XGBRegressor" style="height: 300px; width:300px;"/>

Enquanto que a melhoria foi mínima ou inexistente nos demais

![decision tree](https://github.com/vin94sn/BIMaster/blob/main/Images/Decision%20Tree%20GridSearch.png?raw=true)
![random forest](https://github.com/vin94sn/BIMaster/blob/main/Images/Random%20Forest%20GridSearch.png?raw=true)

### 4. Conclusões

Pelo resultados encontrados foi possível identificar quando há reabastecimento e prever quando será o próximo, ambos com precisão igual ou maior que 97% em diversos algoritmos. Sendo assim, ambos os objetivos propostos inicialmente foram alcançados.

Em relação aos melhores algoritmos para cada modelo, apesar da maioria dos algoritmos terem resultados satisfatórios, pelos resultados, é possível afirmar que o Random Forest é o melhor para o modelo de classificação, enquanto que o Decision Tree é o melhor para o modelo de regressão.

---

Matrícula: 202100074

Pontifícia Universidade Católica do Rio de Janeiro

Curso de Pós Graduação *Business Intelligence Master*
