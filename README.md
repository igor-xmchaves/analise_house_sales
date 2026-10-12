# Análise de preços de imóveis em King County

> Trabalho 1 da disciplina CC0452 — Modelagem Estatística  
> Equipe 5: Ivna Coelho Loiola, Igor Xavier Martins Chaves e Lucas Ferreira da Silva

## Pergunta e objetivo

O projeto estuda a associação entre a área habitável e o preço de venda de imóveis de King County, nos Estados Unidos, entre maio de 2014 e maio de 2015. A análise é observacional: as associações estimadas não demonstram causalidade.

## Dataset e variáveis

A base [House Sales in King County, USA](https://www.kaggle.com/datasets/harlfoxem/housesalesprediction) contém 21.613 vendas e 21.436 imóveis distintos. Cada linha representa uma venda, portanto um mesmo imóvel pode aparecer mais de uma vez. O preço `price` permanece em dólares americanos (USD), moeda original das transações; não há conversão cambial. A área habitável `sqft_living` é convertida para m² pelo fator **1 sqft = 0,09290304 m²**.

| Grupo | Variáveis originais | Descrição |
| --- | --- | --- |
| Identificação e tempo | `id`, `date` | Imóvel e data da venda |
| Venda | `price` | Preço da transação em USD |
| Características | `bedrooms`, `bathrooms`, `floors`, `waterfront`, `view`, `condition`, `grade`, `yr_built`, `yr_renovated` | Cômodos, pavimentos, atributos, qualidade e anos de construção/reforma |
| Áreas | `sqft_living`, `sqft_lot`, `sqft_above`, `sqft_basement`, `sqft_living15`, `sqft_lot15` | Áreas habitáveis, do terreno, acima/abaixo do solo e referências de imóveis próximos |
| Localização | `zipcode`, `lat`, `long` | CEP e coordenadas geográficas |

Nos modelos principais, `preco` (cópia de `price`) é a resposta; `area`, `grade`, `bathrooms`, `yr_built` e `lat` são explicativas do MRLM. Também são analisadas respostas logarítmicas, `grade` categórica, interação área × `grade` e extensões com `long` e `zipcode`. `id` define grupos na inferência e validação; `date` permite examinar controles mensais. `condition` e `grade` representam características distintas.

## Especificações e inferência

O **MRLS** estima `preco ~ area`. O **MRLM** acrescenta qualidade construtiva, banheiros, ano de construção e latitude. As especificações são ajustadas por mínimos quadrados ordinários; a inferência principal usa covariância agrupada por `id`, considerando vendas repetidas do mesmo imóvel. HC3 é apresentada como análise de sensibilidade à heterocedasticidade. A validação compara previsões por grupos de imóveis e por período.

## Resultados obtidos

| Modelo | R² na amostra |
| --- | ---: |
| MRLS: preço ~ área | 0,493 |
| MRLM: área, grade numérica, banheiros, ano e latitude | 0,643 |
| MRLM com grade categórica | 0,679 |

No MRLM, a inclinação associada à área é de aproximadamente **US$ 1.796 por m²**, com intervalo de 95% agrupado por imóvel de **US$ 1.625 a US$ 1.966**. Na validação por imóvel, o modelo ampliado com interação e localização teve RMSE médio de **US$ 166,9 mil**, comparado a **US$ 218,9 mil** no MRLM linear de referência. Essa comparação é exploratória e não escolhe automaticamente um modelo final.

## Arquitetura do projeto

```text
analise_house_sales/
├── house_sales.ipynb   análise, modelos, diagnósticos e validação
├── data/               base de vendas
├── resultados/         17 tabelas CSV com delimitador | e 2 gráficos PNG
├── requirements.txt    dependências Python
└── README.md           relatório breve e instruções
```

O fluxo do notebook é: leitura → auditoria e preparação → análise descritiva → modelagem → diagnósticos e validação → apresentação e exportação. As análises complementares são mostradas no notebook, sem arquivos separados. Os gráficos exportados são `validacao_grupos_temporal.png` e `diagnosticos_comparacao_cep_mes.png`.

## Como executar

Na pasta do projeto, instale as dependências e abra `house_sales.ipynb` no Jupyter, Google Colab ou VS Code:

```powershell
python -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
```

Execute as células em ordem. O arquivo `data/kc_house_data.csv` acompanha o projeto; caso esteja ausente, o notebook tenta obtê-lo no Kaggle e exige acesso à internet.
