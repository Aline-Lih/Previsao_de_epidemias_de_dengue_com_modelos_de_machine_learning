# Previsão de Epidemias de Dengue Utilizando Modelos de Machine Learning

Projeto de conclusão de curso (Big Data), desenvolvido em equipe, sob orientação do tutor Alessandro Brassanini, com acesso a dados fornecido pelo Dr. Flávio Codeço Coelho (Infodengue).

## Objetivo

Desenvolver modelos de aprendizado de máquina para **prever o número de notificações de dengue** por semana epidemiológica no estado do Ceará, com base em dados históricos de notificação e variáveis climáticas (temperatura, umidade, precipitação, pressão atmosférica).

## Dataset

- **Fonte:** Sistema [Infodengue](https://info.dengue.mat.br/), que integra dados do DataSUS (via [PySUS](https://github.com/AlertaDengue/pysus)) e dados meteorológicos da API Copernicus (via [Satellite-Weather-Downloader](https://github.com/osl-incubator/satellite-weather-downloader)).
- **Formato:** CSV, ~1.4 milhão de observações, 10 colunas (geocódigo do município, data/semana/ano de notificação, temperatura média, precipitação média, pressão média, umidade média).
- **Recorte:** estado do Ceará, 2010–2023.

## Metodologia

1. **Extração e engenharia de dados:** consulta SQL que combina notificações (SINAN/DataSUS) com dados climáticos (Copernicus) por município e data.
2. **Agregação:** contagem de casos por semana epidemiológica/ano/município, com médias das variáveis climáticas.
3. **Análise exploratória:** municípios com mais casos, dispersão de casos vs. temperatura/umidade, mapas de incidência por 100 mil habitantes (Censo IBGE 2022).
4. **Modelagem (regressão):** comparação entre Random Forest, Decision Tree, Gradient Boosting e AdaBoost, avaliados por RMSE e R².
5. **Interpretação:** análise de `feature_importances_` do Random Forest, indicando `ano_notif` e `se_notif` como variáveis mais relevantes — reforçando a sazonalidade da doença.

## Tecnologias

`Python` · `pandas` · `numpy` · `scikit-learn` (Random Forest, Decision Tree, Gradient Boosting, AdaBoost) · `Plotly` · `Matplotlib` · `Seaborn` · `mapclassify`

## Retrospectiva técnica (revisão posterior)

Ao revisitar este projeto depois de formados, identificamos duas falhas metodológicas que valem registrar aqui — deixamos o notebook original intacto de propósito, como registro do nosso processo de aprendizado, mas é importante ser transparente sobre elas:

1. **Vazamento de dados por split aleatório em série temporal.** O `train_test_split` foi feito com `test_size=0.2` e embaralhamento aleatório (comportamento padrão da função), sem levar em conta que o problema é sequencial no tempo. Isso significa que o modelo pôde treinar com semanas "futuras" e ser avaliado em semanas "passadas" em relação a elas, o que infla artificialmente as métricas de RMSE e R² — elas não refletem o desempenho real que o modelo teria ao prever genuinamente o futuro a partir do passado. O jeito correto teria sido um corte temporal (por exemplo, treinar com dados até 2021 e testar em 2022–2023), sem embaralhamento.

2. **Variáveis climáticas zeradas na visualização de previsão de longo prazo.** Na célula que gera as previsões para 2018–2023, `temp_med` e `umid_med` foram fixadas em `0` para montar o dataframe de entrada. Isso faz com que essa previsão específica dependa exclusivamente de `ano_notif` e `se_notif`, ignorando por completo o sinal climático — o que é contraditório com a tese central do projeto (a de que clima influencia a incidência de dengue). O ideal teria sido usar médias históricas das variáveis climáticas para cada semana, preservando esse sinal na projeção.

Nenhuma dessas falhas invalida o valor educacional do projeto — a construção do pipeline de dados, a comparação de algoritmos e a análise exploratória continuam sólidas — mas achamos importante documentar que hoje saberíamos evitá-las, principalmente a primeira, que é um erro clássico (e comum) ao aplicar ML a problemas de séries temporais.

## Limitações discutidas no projeto original

- Análise feita em nível estadual, sem considerar particularidades de cada município.
- Ausência de expertise específica em entomologia/climatologia na interpretação dos resultados.
- Não incorpora variáveis de mobilidade populacional entre municípios.

## Como executar

O notebook foi desenvolvido para rodar no Google Colab / Jupyter, com os dados carregados diretamente via URL pública do Infodengue.

```bash
pip install pandas numpy scikit-learn matplotlib seaborn plotly mapclassify
```

## Autoria

Projeto em equipe (5 pessoas) — trabalho de conclusão de curso.

## Referências

Ver seção "Referências Bibliográficas" no notebook original.
