# Análise de preços de imóveis com regressão linear

> Trabalho 1 da disciplina CC0452 — Modelagem Estatística
>
> Equipe 5: Ivna Coelho Loiola, Igor Xavier Martins Chaves e Lucas Ferreira da Silva

O projeto analisa a associação entre características de imóveis e seu preço de venda na base **House Sales in King County, USA**, com transações entre 2 de maio de 2014 e 27 de maio de 2015. A pergunta central é como a área habitável e as demais características selecionadas se associam ao preço e como a especificação do modelo altera o ajuste e o comportamento dos resíduos.

O notebook reúne auditoria dos dados, análise descritiva, regressão linear simples (MRLS), regressão linear múltipla (MRLM) e uma alternativa com resposta em log. Os resultados são apresentados em metros quadrados e reais nominais, preservando os dados originais em pés quadrados e dólares.

## Dados e conversões

### Amostra e tratamento

A fonte é a base pública [House Sales in King County, USA, no Kaggle](https://www.kaggle.com/datasets/harlfoxem/housesalesprediction), identificador `harlfoxem/housesalesprediction`. O CSV original contém 21.613 vendas e 21 colunas, com 21.436 IDs distintos. Cada observação representa uma **venda**: 176 propriedades aparecem mais de uma vez, reunindo 353 transações e 177 ocorrências adicionais de IDs já presentes.

A auditoria verifica tipos, valores ausentes, datas, duplicatas completas e domínios. Somente cópias integralmente idênticas e registros sem os campos essenciais válidos são excluídos, com motivo e linha de origem registrados. Nas regras atuais, nenhuma venda foi excluída; 23 registros foram sinalizados para conferência e mantidos. Revendas e valores extremos são preservados, e diferenças cadastrais não são corrigidas por suposição.

### Variáveis e unidades

| Coluna | Significado | Unidade ou escala |
| --- | --- | --- |
| `price` | Preço de venda original | US$ |
| `sqft_living` | Área interna habitável original | pés² |
| `area` | Área habitável convertida | m² |
| `preco` | Preço convertido pela cotação histórica da venda | R$ nominais |
| `cambio` | PTAX de venda de fechamento utilizada | R$/US$ |
| `dia_cot` | Data efetiva da cotação utilizada | data |
| `grade` | Qualidade construtiva e do projeto | escala ordinal de 1 a 13 |
| `bathrooms` | Banheiros, incluindo contagens fracionárias | contagem |
| `yr_built` | Ano de construção | ano |
| `lat` | Latitude | graus decimais |
| `log_preco` | Logaritmo natural do valor numérico de `preco` em reais | escala logarítmica |

### Conversão da área e do preço

As conversões utilizadas são:

```python
area = sqft_living * 0.09290304
preco = price * cambio
```

A cotação vem da [série oficial PTAX do Banco Central](https://dadosabertos.bcb.gov.br/pt_PT/dataset/dolar-americano-usd-todos-os-boletins-diarios). Utiliza-se a PTAX de venda de fechamento da data da transação ou, quando indisponível naquele dia, a última cotação anterior. O arquivo `data/cambio.csv` conserva a série; `data/cambio.json` registra a fonte, a série e o período consultado. A coluna `dia_cot` permite conferir a data efetivamente aplicada.

Os preços são nominais, sem correção pela inflação. A conversão fixa da área muda sua escala; o câmbio histórico varia entre vendas e pode alterar a correlação e o ajuste. Para examinar essa diferença, o notebook inclui uma referência de regressão simples em dólares, usando as mesmas vendas e a área em m².

## Modelos e metodologia

Os três modelos principais são ajustados por mínimos quadrados ordinários, com intercepto, utilizando `statsmodels` e as mesmas 21.613 vendas. Antes do ajuste, são verificados valores ausentes, domínios das preditoras e condições da matriz do MRLM, evitando descarte silencioso de observações.

| Modelo | Objeto no notebook | Fórmula |
| --- | --- | --- |
| MRLS | `modelo_simples` | `preco ~ area` |
| MRLM | `modelo_multiplo` | `preco ~ area + grade + bathrooms + yr_built + lat` |
| MRLM com resposta em log | `modelo_log` | `log_preco ~ area + grade + bathrooms + yr_built + lat` |

O ajuste auxiliar em dólares utiliza `price ~ area`. A análise segue quatro etapas:

1. Leitura e auditoria, conversão das unidades e identificação das revendas por `id`.
2. Estatísticas descritivas, distribuições, gráficos e correlações de Pearson.
3. Ajuste dos modelos, coeficientes, intervalos de 95%, testes t e F, R², R² ajustado e erro padrão residual.
4. Diagnósticos de resíduos e Q-Q, Jarque-Bera, Breusch-Pagan/Koenker, VIF, alavancagem e distância de Cook.

O código permanece no notebook. Funções de apoio centralizam ajuste, organização dos resultados e exportação; as células de cada etapa apresentam as fórmulas e as decisões de tratamento. As tabelas e os gráficos são gravados em `resultados/`.

## Resultados

### Ajuste dos modelos

| Modelo | Resposta | R² | R² ajustado | Erro padrão residual |
| --- | --- | ---: | ---: | ---: |
| MRLS | Preço em R$ nominais | 0,45291 | 0,45289 | R$ 710.368,84 |
| MRLM | Preço em R$ nominais | 0,59183 | 0,59174 | R$ 613.642,46 |
| MRLM com resposta em log | Log do preço em reais | 0,67761 | 0,67753 | 0,30768 na escala logarítmica |

O MRLM em reais descreve mais variabilidade na amostra que o MRLS. O R² do modelo em log se refere a outra resposta e não permite classificá-lo diretamente como superior aos modelos em reais. O erro padrão residual mede a dispersão dos resíduos na escala de cada modelo; não equivale ao erro médio absoluto nem a uma margem fixa para cada venda.

### Regressão linear simples

A correlação entre área e preço em reais é 0,6730. A equação estimada é:

```text
preco_estimado = -86.607,14 + 7.574,84 * area
```

Uma diferença de 10 m² corresponde a aproximadamente R$ 75.748,40 entre os valores ajustados. A inclinação expressa a associação com a área, e não o preço médio por m² de um imóvel. O intercepto corresponde a zero m², fora da faixa observada de 26,94 a 1.257,91 m², e não tem interpretação prática direta como preço de uma propriedade.

Na referência em dólares, o R² é 0,49285. A diferença em relação ao ajuste em reais evidencia a mudança da resposta provocada pela conversão histórica, sem estabelecer superioridade preditiva.

### Regressão múltipla e associações condicionais

No MRLM em reais, a inclinação da área é R$ 4.492,84/m². O MRLS reúne a associação com a área e diferenças entre as vendas, incluindo características dos imóveis e variação cambial; o MRLM condiciona a associação às cinco preditoras incluídas. A mudança da inclinação mostra que a estimativa depende da especificação e da distribuição conjunta das características, sem decompor causalmente o preço.

Os contrastes abaixo variam uma preditora por vez, mantendo as outras quatro constantes na equação:

| Preditora | Variação | Diferença ajustada no preço |
| --- | --- | ---: |
| Área habitável | +10 m² | +R$ 44.928,40 |
| Qualidade construtiva (`grade`) | +1 ponto | +R$ 309.890,44 |
| Banheiros (`bathrooms`) | +1 unidade na contagem | +R$ 90.419,57 |
| Ano de construção (`yr_built`) | Ano de construção 10 anos mais recente | −R$ 82.525,83 |
| Latitude (`lat`) | +0,01 grau, em direção ao norte | +R$ 13.317,03 |

A leitura desses contrastes depende da codificação: `grade` é ordinal e seu uso numérico supõe uma diferença linear uniforme por ponto; banheiros admite frações; ano de construção compara imóveis distintos, sem representar o envelhecimento de uma propriedade; latitude cobre apenas parte da localização, sem substituir informações de bairro ou longitude. Os valores são registrados em `efeitos_modelos.csv` e interpretados na seção 13.1 do notebook.

O ano de construção ilustra a diferença entre associação simples e condicional: sua correlação com o preço é positiva (0,05231), enquanto o coeficiente do MRLM é −R$ 8.252,58/ano. Essa mudança de sinal não é, por si só, um erro do ajuste. A correlação descreve uma relação bivariada sem unidade; o coeficiente expressa uma associação condicional em R$/ano. Suas magnitudes numéricas não são diretamente comparáveis. A tabela `associacoes_mrlm.csv` reúne essas medidas para as cinco preditoras.

### Modelo com resposta em log

O coeficiente da área é 0,00199505 por m². Para uma variação `delta` em uma preditora, a mudança percentual associada é calculada por:

```python
100 * (exp(coeficiente * delta) - 1)
```

Para 10 m², o resultado é aproximadamente 2,02%, mantendo as demais preditoras constantes. A interpretação se refere à média geométrica condicional proposta pelo modelo. Exponenciar o log ajustado não recupera automaticamente a média aritmética do preço em reais; uma previsão nessa escala exige explicitar a retransformação.

### Diagnósticos

O maior VIF das cinco preditoras é aproximadamente 3,47. O cálculo utiliza a matriz do MRLM com intercepto e apresenta somente as explicativas. O valor 5 é adotado no notebook como referência de inspeção, sem representar um limite universal ou garantir estabilidade dos coeficientes.

Os gráficos de resíduos e Q-Q são acompanhados pelos testes de Jarque-Bera (normalidade) e Breusch-Pagan na versão de Koenker (variância constante). No modelo em log, as estatísticas são 547,69 e 470,22, respectivamente, ambas com valor-p < 0,001. A transformação não resolveu todos os desvios avaliados. Com uma amostra grande, os testes devem ser lidos junto dos gráficos e da magnitude dos desvios.

No MRLM em reais, os critérios de investigação são:

| Critério | Limiar | Vendas sinalizadas |
| --- | --- | ---: |
| Resíduo studentizado interno | `abs(residuo_estud) > 3` | 325 |
| Alavancagem | `h_ii > 2 * p / n` | 1.259 |
| Distância de Cook | `D_i > 4 / n` | 1.000 |

Utilizam-se `n = 21613` e `p = 6`, incluindo o intercepto. Os critérios se sobrepõem: a união reúne 1.784 vendas distintas. O maior Cook é 0,36977, na linha 7.254 do CSV original, contando o cabeçalho como linha 1. Todas as vendas aparecem nos gráficos e permanecem no ajuste; `id` e `linha_csv` permitem localizar os casos nas tabelas.

## Limites da interpretação

- **Alcance:** a base é observacional. Os coeficientes descrevem associações na amostra e os contrastes ilustram a equação, sem demonstrar causalidade ou representar necessariamente pares de imóveis observados.
- **Inferência:** testes e intervalos utilizam covariância clássica, com nível de significância de 5%. Heterocedasticidade e possível dependência entre revendas limitam sua interpretação. Os diagnósticos não corrigem automaticamente a covariância nem asseguram independência dos erros; um valor-p pequeno não garante precisão para vendas individuais.
- **Cenários:** respeitar as faixas individuais de `faixas_preditoras.csv` não garante que uma combinação das cinco características esteja representada na amostra. O suporte conjunto dos contrastes não foi verificado, e cenários específicos exigem conferir observações próximas e evitar extrapolações.

Covariâncias robustas ou agrupadas por imóvel, análises de sensibilidade e avaliação preditiva fora da amostra permanecem como possibilidades de continuação. A capacidade de previsão para novos imóveis ou vendas futuras ainda não foi avaliada.

## Como executar

Mantenha o diretório de trabalho na pasta do repositório. Com o CSV de vendas e os dois arquivos de câmbio em `data/`, a análise pode ser executada sem rede. Quando o CSV de vendas ou o cache estão ausentes, o notebook tenta consultar as fontes; ampliar o período coberto pelo cache também exige novas cotações.

### Google Colab — ambiente padrão

1. Carregue `house_sales.ipynb` no Colab.
2. Disponibilize a pasta `data/`, incluindo `kc_house_data.csv`, `cambio.csv` e `cambio.json`. Se o CSV de vendas estiver em outro local, ajuste `CAMINHO` na célula de leitura.
3. Execute as células na ordem; os resultados serão gravados em `resultados/`.

Se faltar alguma biblioteca, descomente a linha `%pip install numpy pandas matplotlib seaborn scipy statsmodels` na célula de imports. Após a instalação, reinicie o ambiente e execute desde o início. Os dados não acompanham automaticamente um notebook compartilhado e precisam estar disponíveis em cada nova sessão. Consulte a [documentação do Google Colab](https://research.google.com/colaboratory/faq.html).

### VS Code — execução local

Utilize Python 3.12 ou superior e as extensões Python e Jupyter. Na pasta do repositório, prepare o ambiente:

```powershell
python -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
.\.venv\Scripts\python.exe -m pip check
```

Abra o notebook, selecione o kernel desse ambiente e execute todas as células. Se já houver um ambiente virtual, utilize seu interpretador. A linha comentada `%pip install -r requirements.txt` também permite instalar as dependências no kernel selecionado, desde que o arquivo esteja no diretório de trabalho. Reinicie o kernel após a instalação. Consulte a [documentação de notebooks no VS Code](https://code.visualstudio.com/docs/datascience/jupyter-notebooks).

O ambiente de referência utiliza Python 3.13 e as versões registradas em `requirements.txt`. As bibliotecas disponibilizadas pelo Colab podem ter versões diferentes.

### Problemas de importação

Se ocorrer `ModuleNotFoundError`, instale a biblioteca no mesmo ambiente utilizado pelo notebook. Mensagens sobre módulos parcialmente inicializados, como a ausência de `_pandas_datetime_CAPI`, podem ocorrer após uma importação interrompida; reinicie o kernel antes de tentar novamente.

Para conferir a versão instalada do pandas:

```powershell
.\.venv\Scripts\python.exe -c "import pandas as pd; print(pd.__version__)"
```

Se o problema persistir e indicar uma instalação inconsistente, reinstale as dependências e reinicie o kernel:

```powershell
.\.venv\Scripts\python.exe -m pip install --force-reinstall --no-cache-dir -r requirements.txt
```

Um bloqueio de DLL pelo Controle de Aplicativo do Windows precisa ser verificado nas configurações de segurança ou com o administrador do computador.

## Organização do repositório

```text
analise_house_sales/
├── data/
│   ├── kc_house_data.csv
│   ├── cambio.csv
│   └── cambio.json
├── resultados/           # 17 tabelas CSV e seis gráficos PNG
├── house_sales.ipynb
├── requirements.txt
├── README.md
└── .gitignore
```

Os arquivos de resultados estão organizados por finalidade:

| Conteúdo | Arquivos em `resultados/` |
| --- | --- |
| Descrição, auditoria e revendas | `analise_descritiva.csv`, `linhas_excluidas.csv`, `analise_ids.csv`, `vendas_repetidas.csv` |
| Coeficientes e medidas do ajuste | `coeficientes_mrls.csv`, `medidas_mrls.csv`, `coeficientes_modelos.csv`, `medidas_modelos.csv` |
| Contrastes, associações e faixas das preditoras | `efeitos_modelos.csv`, `associacoes_mrlm.csv`, `faixas_preditoras.csv` |
| Referência da conversão cambial | `referencia_cambio.csv` |
| Resíduos e multicolinearidade | `diagnostico_residuos.csv`, `vif_preditoras.csv` |
| Influência por venda e resumo dos critérios | `diagnostico_vendas.csv`, `diagnostico_atipicos.csv`, `observacoes_sinalizadas.csv` |
| Gráficos | `01_preco_area.png`, `02_demais_variaveis.png`, `03_correlacoes.png`, `04_ajuste_mrls.png`, `05_diagnostico_atipicos.png`, `06_diagnostico_residuos.png` |


