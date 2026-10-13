# Análise de preços de imóveis em King County

> Trabalho 1 da disciplina CC0452 — Modelagem Estatística  
> Equipe 5: Ivna Coelho Loiola, Igor Xavier Martins Chaves e Lucas Ferreira da Silva

## Pergunta e objetivo

O projeto estuda a associação entre a área habitável e o preço de venda de imóveis de King County, nos Estados Unidos, entre maio de 2014 e maio de 2015. A análise é observacional: os coeficientes descrevem associações na amostra e não demonstram causalidade.

## Dados, unidades e variáveis

A base [House Sales in King County, USA](https://www.kaggle.com/datasets/harlfoxem/housesalesprediction) reúne **21.613 vendas de 21.436 imóveis distintos**. Cada linha representa uma venda; por isso, um mesmo imóvel pode aparecer mais de uma vez. A resposta é `price`, mantida em dólares americanos nominais (USD), moeda original das transações. A área habitável `sqft_living` é convertida para m² pelo fator **1 sqft = 0,09290304 m²**. Não se aplica conversão cambial.

| Grupo | Variáveis | Papel na análise |
| --- | --- | --- |
| Identificação e tempo | `id`, `date` | Identificar imóveis, revendas e período da venda |
| Venda | `price` | Resposta em USD |
| Características | `bedrooms`, `bathrooms`, `floors`, `waterfront`, `view`, `condition`, `grade`, `yr_built`, `yr_renovated` | Descrição dos imóveis; algumas entram nas especificações ampliadas |
| Áreas | `sqft_living`, `sqft_lot`, `sqft_above`, `sqft_basement`, `sqft_living15`, `sqft_lot15` | Áreas originais em pés quadrados; `sqft_living` gera `area` em m² |
| Localização | `zipcode`, `lat`, `long` | Latitude no MRLM de referência; CEP e longitude em extensões |

`id` também define os grupos da inferência e da validação. `date` permite examinar efeitos mensais. `grade` representa qualidade construtiva; `condition` representa o estado de conservação, conceito distinto.

## Preparação, modelos e inferência

A auditoria verifica ausências, datas, duplicatas exatas e a validade de preço e área. Revendas não são tratadas como duplicatas apenas por compartilharem `id`; registros extremos que atendem aos critérios de auditoria permanecem na análise. As especificações principais usam as mesmas 21.613 vendas. Antes do ajuste, o notebook verifica os domínios usados, a positividade do preço nos modelos logarítmicos e as condições da matriz de desenho, evitando a exclusão silenciosa de linhas.

Os modelos são ajustados por **mínimos quadrados ordinários (MQO)**, com intercepto, usando `statsmodels`. A análise inclui:

- MRLS: `price ~ area`;
- MRLM de referência: `price ~ area + grade + bathrooms + yr_built + lat`;
- alternativas com `grade` categórica e respostas `ln(price / 1 US$)`;
- extensões com interação entre área e `grade`, longitude, CEP categórico e efeitos mensais.

A inferência principal usa covariância agrupada por `id`, para acomodar dependência entre revendas do mesmo imóvel, e pressupõe independência entre imóveis. HC3 é apresentada como análise de sensibilidade à heterocedasticidade sob independência entre vendas; ela não resolve a dependência entre revendas. As tabelas apresentam coeficientes, intervalos de confiança, testes, R², R² ajustado e erro padrão residual. Os intervalos de predição individuais ainda dependem da variância residual OLS e sua cobertura nominal não é garantida quando há heterocedasticidade.

## Resultados na amostra

| Modelo em USD | R² | R² ajustado |
| --- | ---: | ---: |
| MRLS: preço ~ área | 0,493 | 0,493 |
| MRLM: área, `grade` numérica, banheiros, ano e latitude | 0,643 | 0,643 |
| MRLM com `grade` categórica | 0,679 | 0,679 |

No MRLS, a reta estimada é aproximadamente:

```text
preço estimado (USD) = -43.580,74 + 3.020,61 × área (m²)
```

A correlação de Pearson entre área e preço é aproximadamente **0,702**. Na reta, uma diferença de 10 m² corresponde a cerca de **US$ 30,2 mil** nos valores ajustados. No MRLM, a inclinação condicional da área é aproximadamente **US$ 1.796/m²**, com intervalo de 95% agrupado por imóvel de **US$ 1.625 a US$ 1.966/m²**; 10 m² correspondem a cerca de US$ 18,0 mil, mantendo as demais variáveis constantes. A diferença entre as inclinações simples e múltipla mostra que a estimativa depende da especificação e das características que variam em conjunto. Ela não decompõe causalmente o preço. A inclinação representa uma associação com área, não o preço médio por m² de um imóvel. O intercepto do MRLS corresponde a área zero, fora da faixa observada, e não tem interpretação prática direta.

O R² descreve a parcela da variabilidade da resposta representada pelo ajuste **na amostra**; não é taxa de acerto nem medida de efeito causal. O R² dos modelos com resposta logarítmica pertence a outra escala e não deve ser comparado diretamente ao R² em USD. O erro padrão residual mede a dispersão dos resíduos na escala da resposta; não equivale ao erro médio absoluto nem a uma margem fixa para cada venda.

## Diagnósticos, interpretação e limites

Os gráficos de resíduos e Q-Q são acompanhados pelos testes de Jarque–Bera, Breusch–Pagan/Koenker, VIF, condicionamento da matriz, alavancagem e distância de Cook. A transformação logarítmica não elimina todos os desvios: no MRLS em log, Jarque–Bera não rejeita normalidade a 5%, mas Breusch–Pagan/Koenker indica variância não constante; no MRLM em log, ambos os testes rejeitam suas hipóteses nulas. Com mais de 21 mil vendas, os valores-p são lidos junto dos gráficos e da magnitude dos desvios. O valor 5 usado para inspecionar VIF é uma referência, não um limite universal.

As sinalizações por resíduo studentizado, alavancagem e Cook indicam casos para investigação, não justificam exclusão automática. As vendas sinalizadas permanecem nos ajustes oficiais. A seção 19 também compara os coeficientes após retirar apenas o Top 3 de Cook e após retirar todas as vendas sinalizadas; ambas são reestimações diagnósticas e não alteram a amostra principal. Em particular, `grade = 1` tem apenas uma venda; seu indicador categórico produz alavancagem próxima de 1 e deixa alguns diagnósticos de influência instáveis ou indefinidos.

Na interpretação dos coeficientes e cenários:

- `grade` numérica supõe uma associação linear e uniforme por ponto da escala ordinal; a versão categórica compara os níveis com `grade = 7` e não impõe essa tendência;
- `bathrooms` admite contagens fracionárias, e o ano de construção compara imóveis distintos — não mede o efeito de envelhecimento de uma mesma propriedade;
- latitude representa apenas parte da localização. Permanecer dentro da faixa individual observada de cada preditora não garante que uma combinação de características esteja representada na amostra;
- nos modelos logarítmicos, `100 × (exp(coeficiente × variação) − 1)` expressa a variação percentual associada à média geométrica condicional. Exponenciar a predição em log não recupera automaticamente a média aritmética do preço; a retransformação usa correção de Duan e é descrita no notebook.

A validação compara todos os candidatos nas mesmas partições: cinco divisões por grupo de imóveis e uma separação temporal que retira do treino as vendas anteriores dos imóveis presentes no teste. O modelo com interação e localização teve RMSE médio de **US$ 166,9 mil** nas cinco partições por imóvel e **US$ 173,9 mil** na partição temporal; o MRLM linear de referência teve, respectivamente, **US$ 218,9 mil** e **US$ 225,3 mil**. A seleção e as comparações são exploratórias: elas não escolhem automaticamente um modelo final, e a validação apresentada não é um teste independente posterior à seleção entre todos os candidatos.

## Estrutura do projeto

```text
analise_house_sales/
├── house_sales.ipynb   análise, modelos, diagnósticos e validação
├── data/               base de vendas
├── resultados/         20 tabelas CSV (delimitador |) e 3 gráficos PNG
├── requirements.txt    dependências Python
└── README.md           resumo e instruções
```

O notebook documenta as fórmulas, as decisões de tratamento e a interpretação das análises. Funções de apoio centralizam os ajustes, a organização das tabelas e as exportações para `resultados/`. Os gráficos exportados são `validacao_grupos_temporal.png`, `diagnosticos_comparacao_cep_mes.png` e `09_sensibilidade_top3_cook.png`. As tabelas de sensibilidade são `sensibilidade_top3_cook.csv`, `observacoes_top3_cook.csv` e `sensibilidade_sinalizados.csv`.

## Como executar

Na pasta do projeto, instale as dependências e abra `house_sales.ipynb` no Jupyter, Google Colab ou VS Code:

```powershell
python -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
```

Execute as células em ordem. O arquivo `data/kc_house_data.csv` acompanha o projeto; se estiver ausente, o notebook tenta obtê-lo no Kaggle e precisa de acesso à internet.
