# Documentação do Projeto: Desafio 1 – Análise de Crédito (CredAIR)
 
Documentação do notebook `notebook.ipynb`, organizada **passo a passo conforme o enunciado do desafio** (README), com as regras de negócio, as decisões tomadas e a justificativa de cada uma.
 
> Os dados são **sintéticos e fictícios**, criados para fins educacionais. As conclusões valem para o exercício, não para uma carteira real.
 
---
 
## Sumário
 
1. [Visão geral](#1-visão-geral)
2. [Como executar](#2-como-executar)
3. [Estrutura sugerida do repositório](#3-estrutura-sugerida-do-repositório)
4. [Regras de negócio e premissas](#4-regras-de-negócio-e-premissas)
5. [Documentação passo a passo (Seções 1 a 8 do README)](#5-documentação-passo-a-passo)
6. [Histórico: o que mudou em relação à primeira versão](#6-histórico-o-que-mudou-em-relação-à-primeira-versão)
7. [Resultados](#7-resultados)
8. [Limitações e cuidados para uso real](#8-limitações-e-cuidados-para-uso-real)
9. [Checklist de entrega e roteiro da apresentação](#9-checklist-de-entrega-e-roteiro-da-apresentação)
10. [Prática de Git: commits convencionais](#10-prática-de-git-commits-convencionais)
11. [Glossário](#11-glossário)
---
 
## 1. Visão geral
 
| Item | Descrição |
|---|---|
| **Empresa (fictícia)** | CredAIR, financeira de empréstimos pessoais |
| **Problema** | Identificar propostas de crédito com maior risco de inadimplência |
| **Dados** | `credito.csv`: uma linha por proposta, 11 colunas (perfil do solicitante, situação financeira, condições do empréstimo e o desfecho) |
| **Variável resposta** | `inadimplente` (1 = inadimplente, 0 = adimplente) |
| **Tipo de problema** | Aprendizado supervisionado, classificação binária |
| **Modelo exigido** | Regressão Logística |
| **Entregáveis** | Notebook executável no GitHub, com commits convencionais, e apresentação via Microsoft Teams |
 
**Fluxo do notebook:**
 
```
Dados brutos → Entendimento → EDA (incl. outliers) → Limpeza → EDA das relações (dados limpos)
→ Engenharia de atributos → Split treino/teste → Seleção de variáveis (só treino)
→ Regressão Logística → Teste de sensibilidade (só treino) → Baselines → Avaliação no teste
→ Interpretação → Conclusão
```
 
---
 
## 2. Como executar
 
1. Instale as dependências: `pip install -r requirements.txt` (requer scikit-learn ≥ 1.6).
2. Informe onde está o CSV, de **uma** das formas:
   - variável `PATH_DATA` em um arquivo `.env`;
   - ou coloque o arquivo em `data/credito.csv` (o notebook procura esse caminho automaticamente se o `PATH_DATA` não existir).
3. Abra o notebook e execute **Restart Kernel & Run All**. A entrega exige o notebook executado do início ao fim.
O notebook segue a numeração do README (Seções 1 a 8). Como a análise das relações com a inadimplência é feita depois da limpeza, ela aparece como **Seção 2b** (continuação da Seção 2). As células com outputs já gerados são as que vieram da versão original e não mudaram; as demais geram seus resultados ao executar.
 
---
 
## 3. Estrutura do repositório
 
```
.
├── README.md                  # enunciado do desafio
├── DOCUMENTACAO.md            # este documento
├── requirements.txt
├── .env.example               # PATH_DATA=data/credito.csv (não versionar o .env real)
├── data/
│   └── credito.csv
└── notebook.ipynb            
```
 
---
 
## 4. Regras de negócio e premissas
 
As regras abaixo guiam a limpeza e o uso do modelo. As marcadas como **premissa** foram **assumidas** (não vêm do enunciado) e devem ser citadas como tal na apresentação.
 
| # | Regra | Origem | Onde é aplicada |
|---|---|---|---|
| R1 | O alvo é `inadimplente`: 1 = a proposta resultou em inadimplência | Enunciado | Seção 1 |
| R2 | Os dados recebidos devem ser analisados **antes** de qualquer decisão de modelagem | Enunciado | EDA e limpeza |
| R3 | `estado` deve ser padronizado (espaços e caixa), pois são os mesmos 27 estados escritos de formas diferentes | Qualidade dos dados | Limpeza |
| R4 | `idade` acima de 100 é inválida e vira ausente | premissa | Limpeza |
| R5 | Ninguém trabalha antes dos 14 anos: `tempo_emprego_anos` ≥ `idade − 14` é inconsistente | Premissa | Limpeza |
| R6 | `renda_mensal` abaixo de R$ 600 é inválida (valores de R$ 100 a R$ 550 destoam do restante e estão muito abaixo do salário mínimo) | *Premissa | Limpeza |
| R7 | Registros duplicados são removidos depois da padronização do texto | Qualidade dos dados | Limpeza |
| R8 | Parcela comprometendo mais de 30% da renda indica endividamento excessivo (regra de bolso do mercado, **arbitrária**) | Premissa | Engenharia de atributos |
| R9 | A avaliação usa só dados que não participaram do treino; nenhuma decisão guiada pelo alvo usa o teste | Enunciado | Seções 5 a 7 |
| R10 | O modelo deve ser comparado com uma referência simples | Enunciado | Seção 7.1 |
| R11 | Lucro de R$ 800 por bom pagador aprovado e custo de R$ 4.000 por inadimplente aprovado | **Hipotético**, não são dados da CredAIR | Seção 7.3 |
| R12 | Variáveis de localização fina (`estado`) não entram no modelo, para reduzir o risco de discriminação | Boa prática / **premissa** | Seção 5 |
| R13 | Outliers que representam clientes reais (cauda alta de renda e valor, extremos de score) são **mantidos**; só valores impossíveis são tratados. Os tetos em 850 (score) e 50.000 (valor) são censura e ficam registrados como limitação | **Decisão** | Seções 2 e 3 |
| R14 | Uma variante do modelo (renda em log, caudas limitadas, prazo em faixas) só substitui o modelo base se ganhar mais que **1 desvio-padrão da PR AUC** entre os folds; o teste não entra nessa decisão | **Decisão** | Seção 6.1 |
 
---
 
## 5. Documentação passo a passo
 
Legenda de status: ✅ atendido · ⚠️ atendido com ressalva
 
### Seção 1: Entendimento do problema
 
| Requisito do README | O que foi feito | Status |
|---|---|---|
| Identificar a variável resposta | `inadimplente` | ✅ |
| Explicar o que ela representa | Binária; 1 = a proposta resultou em inadimplência após o acompanhamento; observada *depois* da concessão, por isso precisa ser prevista para propostas novas | ✅ |
| Caracterizar o problema de ML | Aprendizado supervisionado de classificação binária; o modelo estima P(inadimplente = 1 \| x) | ✅ |
| Justificar a classificação | Alvo categórico de duas classes; decisão de negócio discreta (aprovar, negar ou revisar); a Regressão Logística devolve probabilidades interpretáveis | ✅ |
 
**Consequências registradas:** a classe positiva é minoritária (~34%), então a acurácia sozinha engana e usamos várias métricas; é obrigatório comparar com um baseline.
 
**Onde:** seção *1. Entendimento do problema*, acima do dicionário de dados.
 
### Seção 2: Análise exploratória
 
A análise tem duas partes: **2.1 a 2.4** olham os **dados brutos** (características, qualidade, alvo e outliers) e a **Seção 2b**, depois da limpeza, investiga as **relações com a inadimplência**. As relações ficam para depois da limpeza porque, nos dados brutos, os valores impossíveis distorceriam as conclusões.
 
| Requisito | O que foi feito | Status |
|---|---|---|
| Características dos dados | `info()`, `describe()`, valores distintos, mínimo e máximo, contagem de nulos e de duplicadas, histogramas | ✅ |
| Distribuição da variável resposta | ~34% de inadimplentes, com gráfico de barras | ✅ |
| Relações com a inadimplência | Spearman, boxplots por classe, **medianas por classe**, taxa de inadimplência por faixa **com número de clientes e intervalo de confiança de 95%** | ✅ |
| Padrões relevantes | Inadimplência mais ligada à **capacidade de pagamento**; perfil demográfico com relação fraca | ✅ |
 
**Achados que sinalizaram problemas de qualidade (insumo da Seção 3):** `idade` com máximo em 750, `renda_mensal` com mínimo muito baixo, `estado` com 85 valores distintos para 27 UFs, tempo de emprego maior que a idade.
 
#### 2.4 Análise dos outliers (dados brutos)
 
Os pontos fora da caixa foram separados em **três grupos**, porque nem todo outlier é erro:
 
| Grupo | Variáveis | O que se vê | Decisão |
|---|---|---|---|
| **Erros de cadastro** | `idade`; `tempo_emprego_anos` | Idade de ~210 a ~750 anos; tempo de emprego de ~46 a 74 anos. Aparecem nas duas classes, sem padrão (erro aleatório de digitação) | Tratar na Seção 3 |
| **Assimetria natural** | `renda_mensal`; `valor_solicitado` | Cauda acima de ~13 mil (renda, com dois pontos perto de 43–45 mil) e de ~30 mil (valor). São clientes reais | **Manter** (R13) |
| **Limite da escala (censura)** | `valor_solicitado`; `score_credito` | Valores empilhados em 50.000 e em 850 | **Manter** e registrar como limitação |
| Sem outliers | `prazo_meses` (6 a 60); `dividas_ativas` (0 a 7) | n/a | Nada a fazer |
 
**Limite do boxplot:** ele só enxerga valores isolados. A inconsistência mais pesada (tempo de emprego incompatível com a idade) só aparece **cruzando duas variáveis**, por isso a regra R5 da Seção 3 atinge ~15% da base, e não os poucos pontos que o gráfico mostra.
 
> O primeiro boxplot tinha um painel extra plotando `inadimplente` contra ela mesma. Foi corrigido tirando o alvo da lista de variáveis numéricas.
 
#### 2b. Relações com a inadimplência (dados limpos)
 
**Boxplots após a limpeza.**
 
| Variável | Antes | Depois |
|---|---|---|
| `idade` | Pontos de ~210 a ~750 | 18 a 75 anos, sem outliers |
| `tempo_emprego_anos` | Pontos de ~46 a 74 anos | 1 a 25 anos, sem outliers |
| `renda_mensal` | Mínimo de R$ 100 a 550 | Mínimo em torno de R$ 800 |
| Demais | Sem erros | Inalteradas |
 
Os outliers que sobram (renda e valor com cauda alta, score nos extremos) são legítimos. Dois cuidados: (1) os dois pontos de renda perto de 43–45 mil ficam a ~10 desvios-padrão da média e pesam em um modelo linear com variáveis padronizadas, o que motiva o teste de sensibilidade da Seção 6.1; (2) os tetos em 850 e 50.000 são censura.
 
**Medianas por classe** (valores lidos do gráfico, aproximados; a tabela do notebook traz os valores exatos):
 
| Variável | Adimplente | Inadimplente | Leitura |
|---|---|---|---|
| `score_credito` | ~670 | ~615 | Separação clara |
| `renda_mensal` | ~4.800 | ~3.300 | Inadimplentes têm renda menor |
| `valor_solicitado` | ~12 mil | ~14 mil | Inadimplentes pedem mais |
| `dividas_ativas` | 1 | 2 | Mais dívidas, mais risco |
| `prazo_meses` | ~36 | ~24 | Inadimplentes têm prazos mais curtos |
| `tempo_emprego_anos`, `idade` | quase iguais | quase iguais | Diferença pequena, mas visível: inadimplentes um pouco mais novos e com menos tempo de emprego |
 
**Taxa de inadimplência por faixa** (a taxa geral é ~34,7%; valores lidos dos gráficos, aproximados), da relação mais forte à mais fraca:
 
| Variável | Menor taxa | Maior taxa | Amplitude | Forma |
|---|---|---|---|---|
| `score_credito` | ~16% (724–850) | ~56% (≤ 581) | ~40 p.p. | Queda contínua e regular |
| `dividas_ativas` | ~27% (0 dívidas) | ~60% (7 dívidas) | ~33 p.p. | Sobe em degraus; salto entre 1 e 2 |
| `renda_mensal` | ~21% (renda alta) | ~53% (< R$ 2.430) | ~32 p.p. | Cai forte no início e depois achata |
| `prazo_meses` | ~29% (36 meses) | ~54% (6 meses) | ~25 p.p. | Concentrada nos prazos curtos |
| `tempo_emprego_anos` | ~27% (19–25 anos) | ~40% (até 4 anos) | ~13 p.p. | Queda gradual |
| `idade` | ~30% (66–75) | ~38% (18–36) | ~8 p.p. | Queda leve e irregular (não monotônica) |
| `finalidade` | ~33% (educação) | ~38% (saúde) | ~5 p.p. | Quase plana |
| `escolaridade` | ~31% (fundamental) | ~36% (médio) | ~4 p.p. | Quase plana, sem ordem lógica |
 
**Número de clientes e intervalo de confiança (IC) de 95%.** Para cada faixa, o notebook calcula `n`, a taxa e o IC de Wilson, e marca `sim` em "difere da média?" quando o IC **não cruza** a taxa geral. É isso que separa diferença confiável de ruído, principalmente nas faixas com poucos clientes (dívidas 6 e 7, prazo 48 e 60) e nas diferenças pequenas de escolaridade e finalidade.
 
**Conclusões desta etapa:**
- As quatro variáveis de maior amplitude (score, dívidas, renda e prazo) são as ligadas à capacidade e ao histórico de pagamento, o que sustenta a hipótese central.
- O gráfico de prazo (6 meses ~54%) é a evidência que motivou a criação de `comprometimento_renda`: prazo curto significa parcela alta para o mesmo valor.
- São relações **bivariadas**: idade e tempo de emprego se sobrepõem, então parte do efeito da idade pode ser do tempo de emprego. Os coeficientes da Seção 7.2 controlam isso.
- **Não linearidade:** renda (efeito que achata), prazo (efeito só nos prazos curtos) e dívidas (degrau em 2) sugerem que a regressão logística, que assume efeito linear no log-odds, pode descrever mal essas curvas. Isso é testado na Seção 6.1.
**Onde:** seções *2. Análise exploratória* (2.1 a 2.4) e *2b* do notebook.
 
### Seção 3: Limpeza e preparação dos dados
 
| Problema | Tratamento | Regra | Justificativa |
|---|---|---|---|
| `estado` com espaços e caixa variados (85 valores) | `strip()` + `upper()` → 27 UFs | R3 | Evita categorias duplicadas para o mesmo estado |
| 19 linhas duplicadas | `drop_duplicates()` após padronizar o texto | R7 | As duplicatas só aparecem depois da padronização; evita contar a mesma proposta duas vezes e vazar linhas iguais entre treino e teste |
| `idade` > 100 (ex.: 660 a 750) | vira `NaN` | R4 | Valor impossível; não há como "adivinhar" o valor correto |
| `tempo_emprego_anos` ≥ `idade − 14` | vira `NaN` | R5 | Marcou **760 registros (~15%)**. É a decisão de maior impacto na base |
| `renda_mensal` < R$ 600 | vira `NaN` | R6 | Valores destoantes e implausíveis |
| `NaN` gerados | linhas removidas (`dropna`) | n/a | Não inventa valores; custa **782 linhas (~15,7%)**, resultando em **4.199 linhas** das 5.000 originais |
| Tipos | Categóricas como `category`, inteiros como `int` | n/a | Tipos adequados para análise e modelagem |
 
**Valores ausentes:** a base original não tinha nulos. Os `NaN` tratados foram criados pela própria limpeza ao invalidar valores impossíveis.
 
**Verificação de viés (seção 3):** compara a taxa de inadimplência e o score médio entre linhas mantidas e removidas. Se forem parecidos, a remoção não distorce o alvo. **Registre o resultado desta verificação na apresentação.**
 
**Implementação:** as regras ficam em uma única função, `limpar_dados_credito`, aplicada aos dados brutos. Com `retornar_marcado=True` ela devolve também a base antes do `dropna`, o que permite auditar o que foi removido (é o que a verificação de viés usa).
 
**Alternativa não adotada:** imputar valores em vez de remover linhas. Fica como melhoria, se a verificação de viés indicar problema.
 
**Onde:** seção *3. Limpeza e preparação dos dados* (tabela de decisões e verificação de viés).
 
### Seção 4: Engenharia de atributos
 
| Variável | Construção | O que representa | Relação esperada | Uso |
|---|---|---|---|---|
| `parcela_estimada` | `valor_solicitado / prazo_meses` | Parcela mensal aproximada, **sem juros** | Parcela maior pesa mais no orçamento | Candidata (descartada na Seção 5 por redundância) |
| **`comprometimento_renda`** | `parcela_estimada / renda_mensal` | **Fração da renda consumida pela parcela** | Quanto maior, maior o risco | **Candidata principal**; seu IV é calculado no treino (Seção 5.3) |
| `risco_renda_alta` | 1 se `comprometimento_renda > 0,30` (R8) | Endividamento excessivo | Acima de 30% → mais risco | Avaliada; redundante com a anterior |
| `regiao` | UF → região do Brasil | 27 estados em 5 categorias | Possível diferença regional | Candidata categórica |
| `renda_log`, `valor_solicitado_log` | `log1p` | Reduz assimetria | Mesma direção da original | Avaliadas; mesma informação das originais |
| `faixa_idade`, `estabilidade_emprego`, `score_por_idade` | Quintis de idade; `tempo_emprego / idade`; `score / idade` | Faixa etária, proporção da vida no emprego atual, score relativo | Mais estabilidade → menos risco | Avaliadas; baixo poder preditivo |
 
**Hipótese central:** a inadimplência é mais explicada pela capacidade de pagamento (renda frente à parcela) do que pelo perfil demográfico. O gráfico de taxa por prazo (prazo curto = parcela alta = ~54% de inadimplência em 6 meses) foi o que reforçou a criação de `comprometimento_renda`.
 
**Limitação:** `parcela_estimada` ignora juros e subestima a parcela real.
 
**Onde:** seção *4. Engenharia de atributos* (com a tabela de explicação).
 
### Seção 5: Preparação para modelagem
 
| Requisito | Decisão | Justificativa |
|---|---|---|
| Definir as variáveis | Candidatas numéricas: `score_credito`, `renda_mensal`, `comprometimento_renda`, `dividas_ativas`, `valor_solicitado`, `prazo_meses`, `tempo_emprego_anos`, `idade`. Candidatas categóricas: `escolaridade`, `finalidade`, `regiao` | Foram retiradas versões redundantes (`renda_log`, `valor_solicitado_log`, `parcela_estimada`, `risco_renda_alta`, `faixa_idade`, `score_por_idade`, `estabilidade_emprego`) para não instabilizar os coeficientes |
| Excluir `estado` | `regiao` já resume a informação | R12 |
| Checar colinearidade | VIF calculado no treino (VIF > 5 pede atenção) | Regressão logística interpretável exige variáveis pouco correlacionadas |
| Transformações | `StandardScaler` nas numéricas | Necessário para a penalização L1/L2 |
| Categóricas | `OneHotEncoder(drop="first", handle_unknown="ignore")` | Interpretável na regressão, com categoria de referência |
| Separar treino e teste | 70% / 30%, estratificado pelo alvo, semente fixa (`RANDOM_STATE = 108`) | Mantém a proporção de inadimplentes e torna o resultado reproduzível |
| Selecionar variáveis | **Somente no treino** (IV + Boruta, ver abaixo) | R9: evita vazamento |
 
**Regra de seleção:**
- **IV** (Information Value, numéricas e categóricas): poder de separação entre bons e maus pagadores; abaixo de 0,02 a variável quase não separa as classes.
- **Boruta** (numéricas): compara cada variável com versões embaralhadas dela mesma.
- Numérica entra se for **confirmada** no Boruta com IV ≥ 0,02, ou **tentativa** com IV ≥ 0,05. Categórica entra se IV ≥ 0,02.
Todas as transformações ficam dentro de um `Pipeline`, então os parâmetros são aprendidos apenas no treino.
 
**Onde:** seção *5. Preparação para modelagem* (5.1 a 5.3).
 
### Seção 6: Modelagem
 
| Requisito | O que foi feito |
|---|---|
| Modelo | Regressão Logística (`LogisticRegression`, solver `saga`) |
| Variáveis utilizadas | As selecionadas na Seção 5; a lista exata é impressa pelo notebook após a execução |
| Processo de treino | Busca em grade (`GridSearchCV`) sobre `C` ∈ {0,001; 0,01; 0,1; 1; 10; 50} e `penalty` ∈ {L1, L2}; validação cruzada estratificada de 5 folds **no treino**; métrica de otimização = **PR AUC** (`average_precision`) |
| Desbalanceamento | `class_weight="balanced"` |
| Resultados | Melhores parâmetros e PR AUC de validação cruzada impressos pelo notebook |
 
**Por que PR AUC como critério:** é mais informativa que a acurácia quando a classe de interesse é minoritária.
 
**Efeito colateral de `class_weight="balanced"`:** as probabilidades ficam superestimadas (não calibradas). O notebook inclui um diagrama de confiabilidade para mostrar isso.
 
#### 6.1 Teste de sensibilidade: caudas e não linearidades
 
A EDA levantou duas suspeitas: (1) os pontos de renda perto de 43–45 mil podem pesar demais nos coeficientes; (2) renda, prazo e dívidas têm efeito não linear. Para verificar, o notebook compara o modelo base com três variantes, usando **validação cruzada no treino** e os mesmos hiperparâmetros do modelo final (**o teste não entra**).
 
| Variante | O que muda |
|---|---|
| Renda em log | `renda_mensal` → `log1p(renda_mensal)` |
| Caudas limitadas | Cada numérica é cortada entre os percentis 1 e 99, aprendidos no treino de cada fold (sem vazamento) |
| Prazo em faixas e dívidas 2+ | `prazo_meses` vira categoria (6, 12, 18, 24, 36+) e cria-se o indicador `dividas_2mais` |
 
**Regra de decisão (R14), definida antes de ver o resultado:** só se troca o modelo base por uma variante que ganhe mais que **1 desvio-padrão da PR AUC** entre os folds. Caso contrário, mantém-se o base, por ser mais simples e mais explicável.
 
**Como ler:** se as variantes ficarem dentro de 1 desvio, o modelo é **robusto** às caudas e a não linearidade não compensa a complexidade extra (bom argumento na apresentação). Se alguma ganhar claramente, a suspeita da EDA se confirmou e vira limitação e próximo passo. O notebook imprime a decisão automaticamente, mas o modelo final continua sendo o base: para adotar uma variante, é preciso alterar o pipeline.
 
**Onde:** seção *6. Modelagem*, subseção 6.1.
 
### Seção 7: Avaliação e interpretação
 
Todas as avaliações abaixo usam o **conjunto de teste**, que não participou do treino nem da seleção de variáveis.
 
**7.1. Métricas e comparação com referência simples (R10)**
 
| Referência | Descrição |
|---|---|
| Dummy (sempre aprovar) | Prevê sempre a classe majoritária |
| Dummy (estratificado) | Sorteia previsões respeitando a proporção das classes |
| Regra simples | "Negar se `score_credito` < mediana do treino" |
 
Métricas reportadas: acurácia, acurácia balanceada, G-Mean, ROC AUC, PR AUC, F1, recall, precision e MCC, além das curvas ROC e Precision-Recall. A referência de PR AUC ao acaso é a taxa de inadimplentes no teste.
 
**7.2. Variáveis mais relacionadas ao resultado**
 
Tabela e gráfico de coeficientes e *odds ratios* do modelo final:
- coeficiente positivo aumenta o risco, negativo reduz;
- as numéricas estão padronizadas, então o odds ratio é por **1 desvio-padrão** e o valor absoluto compara a força relativa;
- o resultado descreve **associação, não causa**.
**7.3. Ponto de corte e valor para o negócio (R11)**
 
O modelo devolve probabilidades, e decidir aprovar ou negar exige um corte que depende dos custos. O corte é escolhido com **predições out-of-fold no treino** e aplicado **uma única vez** no teste. A tabela final compara três cenários: aprovar todos, corte padrão 0,50 e corte ótimo.
 
> Os valores de lucro e custo são **hipotéticos**. Troque-os se a monitoria informar valores reais.
 
**7.4. Decis, KS e calibração**
 
Tabela de decis com **Decil 1 = maior risco**, KS no teste e diagrama de confiabilidade.
 
> Atenção: a primeira versão usava uma função (`gerar_tabela_decis_e_ks`) que numerava os decis no sentido contrário (Decil 1 = menor risco) e um texto com números ilustrativos (40–50%, < 1%). Ambos foram removidos: o notebook final usa `tabela_decis_risco`, com **Decil 1 = maior risco**.
 
**Onde:** seção *7. Avaliação e interpretação* (7.1 a 7.4).
 
### Seção 8: Conclusão
 
A seção *8. Conclusão* cobre os cinco pontos exigidos pelo README:
 
1. **Principais padrões nos dados:** ~34% de inadimplentes; problemas de qualidade tratados; outliers legítimos mantidos e tetos de score e valor registrados como censura; inadimplência ligada à capacidade de pagamento.
2. **Principais fatores:** `score_credito` (taxa de ~56% na faixa mais baixa contra ~16% na mais alta; Spearman ≈ −0,30), `dividas_ativas` (~27% com 0 ou 1 dívida contra ~60% com 7), `renda_mensal` (~53% contra ~21%; Spearman ≈ −0,23), prazo curto (~54% em 6 meses) e `valor_solicitado` (Spearman ≈ +0,14). `comprometimento_renda` resume a capacidade de pagamento (IV calculado no treino, Seção 5.3). Idade, escolaridade e finalidade têm relação fraca (4 a 8 p.p. de amplitude).
3. **Resultados do modelo:** resumo numérico impresso automaticamente pela última célula do notebook.
4. **Limitações:** ver [seção 8 deste documento](#8-limitações-e-cuidados-para-uso-real).
5. **Cuidados para uso real:** ver a mesma seção.
---
 
## 6. Histórico: o que mudou em relação à primeira versão
 
O `notebook_final.ipynb` é a versão **organizada e corrigida**: o desenvolvimento exploratório da primeira versão (seleção na base toda, modelo WoE, calibração e corte no teste) foi removido, e o que valia foi reaproveitado. A tabela abaixo registra o que mudou, e é um bom roteiro para explicar o vazamento da primeira versão na apresentação.
 
| Aspecto | Primeira versão | Notebook final |
|---|---|---|
| Seleção de variáveis (IV + Boruta) | Na base inteira, **antes** do split (vazamento) | **Só no treino** |
| Variáveis categóricas | Eliminadas da seleção sem aviso (o IV só cobria numéricas) | Avaliadas pelo IV e usadas se passarem o critério |
| Colinearidade | Variáveis redundantes juntas (ex.: `renda_mensal` e `renda_log`) | Redundâncias removidas e VIF conferido |
| Calibração | Ajustada e avaliada no mesmo teste | Apenas diagnosticada, sem ajuste |
| Ponto de corte | Otimizado no teste | Escolhido por *out-of-fold* no treino, aplicado uma vez no teste |
| Baseline | Não havia | Dummy (2 estratégias) e regra por score |
| Interpretação das variáveis | Dicionário WoE de outro modelo, sem métricas | Coeficientes e odds ratios do modelo avaliado |
| Teste | Usado em várias decisões | Usado uma vez, só para avaliar |
| Boxplot de outliers | Painel extra plotando o alvo contra ele mesmo | Alvo fora da lista de numéricas; 7 painéis |
| Confiabilidade das diferenças por faixa | Só barras de taxa, sem `n` nem intervalo | Tabela com `n` e IC de 95% por faixa |
| Robustez a caudas e não linearidade | Não testada | Teste de sensibilidade (Seção 6.1), só no treino |
| Comentários | Texto genérico | Leituras em primeira pessoa após cada análise |
 
---
 
## 7. Resultados
 
### 7.1 Primeira versão (histórico; não está mais no notebook)
 
Regressão Logística da primeira versão, avaliada no teste (resultado histórico, a execução não está mais no notebook):
 
| Métrica | Valor aproximado |
|---|---|
| Acurácia balanceada | 0,73 |
| ROC AUC | 0,80 |
| PR AUC | 0,68 |
| Recall | 0,73 |
| Precision | 0,59 |
| F1 | 0,65 |
| MCC | 0,45 |
 
> Estes valores vieram de uma seleção de variáveis feita na base inteira e podem estar um pouco otimistas.
 
### 7.2 Modelo final (sem vazamento)
 
Preencha esta tabela com os valores impressos pela última célula do notebook após *Restart & Run All*:
 
| Modelo | ROC AUC | PR AUC | Recall | Precision | F1 | MCC |
|---|---|---|---|---|---|---|
| Dummy (sempre aprovar) | | | | | | |
| Dummy (estratificado) | | | | | | |
| Regra: score < mediana | | | | | | |
| **Regressão Logística final** | | | | | | |
 
| Cenário de corte | Corte | Aprovados | Inadimplência dos aprovados | Lucro (R$, hipotético) |
|---|---|---|---|---|
| Aprovar todos | | | | |
| Corte padrão 0,50 | | | | |
| Corte ótimo (escolhido no treino) | | | | |
 
KS no teste: ____ %
 
---
 
## 8. Limitações e cuidados para uso real
 
### Limitações da análise
 
- **Dados sintéticos:** os resultados valem para o exercício, não para uma carteira real.
- **Divisão aleatória, sem dimensão temporal:** em produção seria necessária validação *out-of-time*.
- **Viés de seleção:** só se conhece o desfecho das propostas aprovadas e acompanhadas, não o das recusadas.
- **Premissas de limpeza:** trabalho a partir dos 14 anos (R5) e renda mínima de R$ 600 (R6) não vêm do enunciado, e a limpeza removeu ~15,7% das linhas.
- **Censura:** score em 850 e valor em 50.000 são tetos; o modelo não distingue "50 mil" de "mais de 50 mil".
- **`parcela_estimada` sem juros** e **custos hipotéticos** (R11).
- **`class_weight="balanced"`** deixa as probabilidades superestimadas.
- **Modelo linear:** a regressão logística assume relações lineares no log-odds e não captura interações automaticamente.
### Cuidados para uso real
 
- Validar em dados mais recentes que os do treino e **monitorar deriva** (PSI) das variáveis e do desempenho; recalibrar quando necessário.
- Definir o ponto de corte com os **custos reais** da operação e prever revisão humana para casos de fronteira.
- Auditar **justiça e discriminação**: localização, idade e escolaridade podem funcionar como proxies de atributos protegidos.
- Garantir **explicabilidade** ao cliente, conforme a LGPD e a regulação de crédito.
- Usar o modelo como **apoio à decisão**, não como decisor único, e manter um modelo desafiante para comparação.
---
 
## 9. Checklist de entrega e roteiro da apresentação
 
### Checklist
 
- [ ] Notebook executado do início ao fim (*Restart & Run All*), sem erros
- [ ] Tabela da seção 7.2 deste documento preenchida com os resultados reais
- [ ] Texto da conclusão conferido contra os resultados da sua execução
- [ ] Resultado da verificação de viés da limpeza anotado
- [ ] Tabelas de `n` e IC por faixa conferidas (quais diferenças são confiáveis; `n` de dívidas 6 e 7)
- [ ] Resultado do teste de sensibilidade (Seção 6.1) anotado e a decisão registrada
- [ ] Blocos "💬 Minha leitura" conferidos contra os números reais e ajustados onde não baterem
- [ ] Valores de lucro e custo (R11) confirmados ou marcados como hipotéticos
- [ ] Repositório pessoal no GitHub criado, com `README.md`, `DOCUMENTACAO.md`, `requirements.txt` e o notebook
- [ ] Histórico de commits seguindo Conventional Commits
- [ ] Link do repositório enviado junto à entrega
- [ ] Reunião individual no Microsoft Teams agendada com um monitor
### Roteiro da apresentação (um item do README por bloco)
 
| Bloco exigido | O que mostrar |
|---|---|
| Problema e objetivo | Seção 1: alvo, tipo de problema, por que classificação |
| Características dos dados | Dicionário, tamanho da base, ~34% de inadimplentes |
| Problemas na preparação | Análise dos outliers (3 grupos), tabela de limpeza (Seção 3), a regra R5 que atingiu ~15% e a verificação de viés |
| Descobertas da EDA | Capacidade de pagamento como eixo; variáveis fortes e fracas (amplitude das taxas); `n` e IC das faixas; prazo curto como motivo do `comprometimento_renda` |
| Decisões tomadas | Variável derivada, remoção de redundâncias, exclusão de `estado`, seleção só no treino |
| Modelo | Regressão Logística, grade de parâmetros, validação cruzada, PR AUC e teste de sensibilidade (Seção 6.1) |
| Resultados | Tabela de baselines, coeficientes, cenários de corte |
| Conclusões e limitações | Seção 8 deste documento |
 
**Ponto-chave a defender:** a apresentação deve mostrar que você entende as decisões. Esteja pronto para explicar por que a seleção original tinha vazamento e como a versão final o corrige.
 
---
 
## 10. Prática de Git: commits convencionais
 
Sugestão de histórico, no padrão do enunciado (mensagens sem acento, como no exemplo do README):
 
```text
feat: inicia analise dos dados
feat: adiciona entendimento do problema
feat: realiza limpeza dos dados
feat: adiciona analise exploratoria
feat: adiciona analise de outliers e intervalos de confianca
feat: cria atributos para o modelo
feat: desenvolve modelo exploratorio
fix: corrige vazamento na selecao de variaveis
feat: adiciona teste de sensibilidade
feat: adiciona baselines e avaliacao do modelo final
feat: adiciona interpretacao dos coeficientes
docs: adiciona conclusao e documentacao
docs: adiciona apresentacao
chore: adiciona requirements.txt
```
 
Tipos usados: `feat` (nova etapa), `fix` (correção), `docs` (documentação) e `chore` (infraestrutura).
 
---
 
## 11. Glossário
 
| Termo | Significado |
|---|---|
| **Inadimplência** | Não pagamento do empréstimo conforme combinado |
| **Baseline** | Referência simples usada para saber se o modelo agrega valor |
| **Vazamento de dados** | Quando informação do teste influencia decisões do treino, deixando as métricas otimistas |
| **IV (Information Value)** | Medida do poder de uma variável para separar bons e maus pagadores |
| **WoE** | Weight of Evidence: logaritmo da razão entre bons e maus em cada faixa da variável |
| **Boruta** | Método de seleção que compara cada variável com cópias embaralhadas dela |
| **VIF** | Fator de inflação da variância; mede colinearidade entre variáveis |
| **ROC AUC** | Capacidade de ranquear maus pagadores acima dos bons (0,5 = acaso; 1 = perfeito) |
| **PR AUC** | Área sob a curva precisão x recall; mais informativa com classe minoritária |
| **MCC** | Coeficiente de correlação de Matthews; resume a matriz de confusão de forma balanceada |
| **KS** | Maior distância entre as distribuições acumuladas de bons e maus pagadores |
| **Odds ratio** | Variação multiplicativa na chance de inadimplência associada à variável |
| **Out-of-fold** | Predição feita por um modelo que não viu aquela linha durante o treino |
| **PSI** | Índice que mede o quanto a distribuição de uma variável mudou ao longo do tempo |
| **Outlier** | Ponto fora do padrão da distribuição; pode ser erro (tratar) ou valor legítimo de cauda (manter) |
| **Censura** | Quando o dado não passa de um teto ou piso (ex.: score 850, valor 50.000), escondendo o valor real além do limite |
| **Intervalo de confiança (Wilson)** | Faixa em que a taxa verdadeira provavelmente está; se não cruza a taxa geral, a diferença é confiável |
| **p.p. (ponto percentual)** | Diferença entre duas taxas (de 16% para 56% são 40 p.p.) |
| **Winsorização** | Cortar valores extremos nos percentis 1 e 99 para reduzir o peso das caudas |
| **Teste de sensibilidade** | Refazer o modelo com pequenas mudanças para ver se o resultado se mantém |
 