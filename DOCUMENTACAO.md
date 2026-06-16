# 🧠 Documentação das Fórmulas e Medidas DAX

Este arquivo centraliza a documentação de todas as regras de negócio, tabelas de suporte e fórmulas personalizadas criadas em DAX para o funcionamento do painel do JPMorgan.


## 📅 1. Estrutura de Filtros e Segmentação

### Elemento: Segmentador – Ano Selecionado
Para garantir uma navegação fluida e intuitiva, o projeto utiliza uma tabela de dimensão do tipo **Calendário** (`dCalendario`) conectada à base histórica de dados. O ano selecionado pelo usuário atua como o motor principal de contexto do painel.

* **Origem do Filtro:** `'Calendario'[Ano]`
* **Comportamento Direto no Painel:** O ano escolhido altera simultaneamente todos os cartões principais de performance do ano corrente e filtra dinamicamente os **4 gráficos interativos** da página (Crescimento YTD, Retorno Anual %, Variação Mensal e Amplitude Média). Além disso, serve como o ponto de corte final para as medidas de inteligência de tempo.

### 📊 Controle de Interações e Visuais Congelados
Para enriquecer a experiência do usuário (UX) e manter uma perspectiva histórica macro essencial para o mercado financeiro, o painel foi configurado para congelar o contexto de determinados elementos, ignorando o segmentador de ano através do menu *Formato > Editar Interações*.

**Visuais com Interações Desativadas:**
1. **Preço de Fechamento vs. Tendência de Longo Prazo (Gráfico Combinado)**
2. **Volume Total por Ano (Gráfico de Colunas)**

* **Por que isso foi feito?** Se esses dois gráficos sofressem o filtro direto do ano selecionado, a linha do tempo histórica seria destruída, exibindo apenas um único ponto ou uma única coluna isolada na tela. Mantendo as interações desativadas, garantimos que o usuário analise a linha de tendência inteira e o histórico de volume desde 1980 até 2026 de forma contínua.
* **Inteligência de Contexto via Tooltip:** Embora esses dois gráficos ignorem o filtro estático do topo da página, eles permanecem dinâmicos no nível de detalhe (contexto de linha). Através das Tooltips Avançadas em DAX, o modelo lê o ponto exato onde o mouse do usuário está posicionado, calculando e exibindo os valores reais daquele ano ou mês específico em tempo real.

## 🧠 Modelagem de Dados e Medidas DAX

* ### 📈 Medida: Retorno Anual %

Esta medida é responsável por calcular o rendimento percentual da ação do JPMorgan exclusivamente dentro do ano que o usuário selecionou no filtro da tela. Ela identifica o preço de fechamento no primeiro e no último dia útil daquele ano específico para calcular a variação.

```dax
Retorno Anual % = 
VAR AnoFiltrado = SELECTEDVALUE('Calendario'[Ano])

-- 1. Cria uma tabela na memória contendo apenas os dados do ano selecionado
VAR TabelaDoAno = 
    FILTER(
        ALLSELECTED(JPM_Data), 
        YEAR(JPM_Data[Date]) = AnoFiltrado
    )

-- 2. Encontra o preço da ação no primeiro dia útil daquele ano específico
VAR PrecoInicial = 
    CALCULATE(
        SELECTEDVALUE(JPM_Data[Close]),
        TOPN(1, TabelaDoAno, JPM_Data[Date], ASC)
    )

-- 3. Encontra o preço da ação no último dia útil daquele ano específico
VAR PrecoFinal = 
    CALCULATE(
        SELECTEDVALUE(JPM_Data[Close]),
        TOPN(1, TabelaDoAno, JPM_Data[Date], DESC)
    )

RETURN
    -- 4. Realiza o cálculo de variação percentual com divisão segura
    DIVIDE(PrecoFinal - PrecoInicial, PrecoInicial)
```

🔍 Como essa fórmula funciona:
TabelaDoAno: Isola a massa de dados para que o Power BI não perca tempo calculando o histórico inteiro, focando apenas no ano da tela.

TOPN com ASC e DESC: Garante que o modelo pegue os preços exatos de abertura e fechamento do ano, ignorando finais de semana e feriados em que a bolsa estava fechada.

DIVIDE: Faz a conta matemática padrão de rentabilidade sem risco de quebrar o visual com erros de divisão por zero.

---

## 💰 Parâmetros Dinâmicos (Inputs do Usuário)

Para permitir que o usuário interaja com o simulador e teste diferentes cenários de investimento, o modelo utiliza uma estrutura combinada: uma tabela gerada por série numérica para o visual da tela e uma medida dinâmica para capturar a escolha do usuário.

### 📊 Tabela Calculada: Segmentador – Capital Inicial

Esta tabela de suporte foi criada para gerar as opções numéricas que alimentam o segmentador flutuante (filtro) onde o usuário define o seu aporte.

```dax
Capital Inicial = 
    -- Gera uma sequência numérica que começa em US$ 100, vai até US$ 10.000, pulando de 100 em 100
    GENERATESERIES(100, 10000, 100)
```

🔍 Como essa estrutura funciona:

GENERATESERIES: Cria automaticamente uma tabela de coluna única (chamada [Value]) com 100 linhas (100, 200, 300... até 10.000).
Otimização: Essa abordagem via DAX poupa memória do relatório e evita a necessidade de carregar uma tabela externa apenas para fins de filtragem de valores.

📐 Medida: Valor Capital Inicial
Esta medida atua como a ponte de comunicação, capturando o valor selecionado na tabela acima e distribuindo-o para os cálculos matemáticos do simulador.
```
Valor Capital Inicial = SELECTEDVALUE('Capital Inicial'[Capital Inicial], 2000)
```
🔍 Como essa fórmula funciona no modelo:
SELECTEDVALUE: Captura o valor exato que o usuário selecionou no filtro interativo na tela.

Valor de Backup (2000): Caso o usuário limpe o filtro ou não selecione nenhum valor, o modelo adota automaticamente o padrão de USD 2.000,00 como contingência, impedindo que os gráficos ou cartões fiquem em branco.

Regra de Negócio: O valor capturado por esta medida serve de insumo direto para as variáveis das fórmulas principais de cálculo de ações acumuladas e auditoria de dados (Valor_Num).

### ⏳ Medida: Valor_Final_Acumulado

Esta é a fórmula central do simulador de longo prazo. Ela garante a consistência financeira do modelo ao fixar o investimento inicial na data de origem da base histórica (1980) e atualizar o patrimônio com base no preço de fechamento do ano selecionado. Com isso, o painel reflete com precisão os impactos de crises reais do mercado financeiro.

```dax
Valor_Final_Acumulado = 
VAR CapitalInicialDigitado = SELECTEDVALUE('Capital Inicial'[Capital Inicial], 2000)
VAR AnoFiltrado = SELECTEDVALUE('Calendario'[Ano])

-- 1. Encontra o preço da ação no primeiríssimo dia da história (em 1980)
VAR PrecoOrigem1980 = 
    CALCULATE (
        FIRSTNONBLANKVALUE ( JPM_Data[Date], SUM ( JPM_Data[Close] ) ),
        ALL ( JPM_Data )
    )

-- 2. Calcula quantas ações foram compradas lá in 1980 com o capital digitado
VAR QuantidadeAcoesCompradas = DIVIDE(CapitalInicialDigitado, PrecoOrigem1980)

-- 3. Encontra o preço de fechamento no último dia útil do ano que está selecionado na tela
VAR PrecoFechamentoAnoSelecionado = 
    CALCULATE (
        LASTNONBLANKVALUE ( JPM_Data[Date], SUM ( JPM_Data[Close] ) ),
        FILTER ( ALL ( 'Calendario' ), 'Calendario'[Ano] = AnoFiltrado )
    )

-- 4. Multiplica as ações guardadas desde 1980 pelo preço do final do ano selecionado
VAR ValorPatrimonioNoAno = QuantidadeAcoesCompradas * PrecoFechamentoAnoSelecionado

RETURN
IF(
    NOT(ISBLANK(PrecoFechamentoAnoSelecionado)),
    "US$ " & FORMAT(ValorPatrimonioNoAno, "#,##0.00", "pt-BR"),
    "US$ " & FORMAT(CapitalInicialDigitado, "#,##0.00", "pt-BR")
)

🔍 Engenharia e Lógica de Contexto Aplicada:
FIRSTNONBLANKVALUE & ALL: Bloqueia a busca do preço inicial estritamente em 1980, limpando qualquer filtro temporal que o usuário clique na tela. Isso evita o erro clássico de simulações que reiniciam do zero a cada ano.

LASTNONBLANKVALUE & FILTER: Permite que o Power BI navegue até o final do ano selecionado na tela para capturar o preço de fechamento atualizado, garantindo que o patrimônio caia ou suba de forma realista (como a queda visível nos testes de 2007 e 2008).

FORMAT: Padroniza a exibição final em texto com a máscara internacional monetária (US$).

### 📉 Medida: Amplitude Média (Volatilidade)

Esta medida calcula a variação média diária entre o preço máximo e o preço mínimo da ação do JPMorgan. Ela serve como um excelente indicador de volatilidade, mostrando o quanto o preço costuma oscilar dentro de um mesmo dia útil.

```dax
Amplitude Média = 
    -- Avalia linha por linha da tabela, subtraindo o preço mínimo do preço máximo do dia, e depois calcula a média global
    AVERAGEX(
        JPM_Data, 
        JPM_Data[High] - JPM_Data[Low]
    )

🔍 Como essa fórmula funciona:
AVERAGEX: É uma função iteradora. Ao contrário da AVERAGE comum, ela cria um contexto de linha, permitindo fazer uma conta matemática coluna menos coluna (High - Low) dentro de cada registro antes de extrair a média final.

Contexto de Negócio: Útil para identificar períodos de grande incerteza no mercado (onde a amplitude diária cresce muito, como em crises) versus períodos de estabilidade.

### 📈 Medida: Crescimento YTD % (Year-To-Date)

Esta medida calcula o desempenho acumulado da ação do JPMorgan ao longo do ano selecionado, comparando o preço de fechamento mais recente (ou do mês selecionado no gráfico) contra o preço do primeiríssimo dia útil daquele mesmo ano.

```dax
Crescimento YTD % = 
-- 1. Identifica a primeira data do histórico que possui registro de preço dentro do ano filtrado
VAR PrimeiraDataComPrecoNoAno =
    CALCULATE (
        MIN ( JPM_Data[Date] ),
        YEAR ( JPM_Data[Date] ) = MAX ( Calendario[Ano] )
    )

-- 2. Captura o preço de fechamento simples exatamente nessa primeira data útil do ano
VAR PrecoInicialAno =
    CALCULATE (
        [Preço Fechamento Simples],
        JPM_Data[Date] = PrimeiraDataComPrecoNoAno
    )

RETURN
    -- 3. Calcula a variação percentual entre o preço atual e o preço do início do ano
    DIVIDE (
        [Preço Fechamento Simples] - PrecoInicialAno,
        PrecoInicialAno
    )

🔍 Como essa fórmula funciona:
MIN + MAX: A variável PrimeiraDataComPrecoNoAno força o Power BI a olhar para o ano do contexto atual e extrair a menor data disponível (geralmente o primeiro dia útil de janeiro), garantindo o ponto de partida do YTD.

[Preço Fechamento Simples]: É uma medida base reutilizada dentro do cálculo. Chamar medidas prontas dentro de outras variáveis (como feito aqui) é uma excelente prática de reaproveitamento de código no DAX.

Contexto de Negócio: Permite avaliar em gráficos mensais como a ação se comportou mês a mês à medida que o ano avançava (por exemplo, mostrando se ela estava se recuperando ou afundando em relação a janeiro daquele ano).

### 📈 Bloco de Medidas: Análise de Preço e Desempenho Histórico

Este grupo de medidas centraliza a captura e o tratamento dos preços de fechamento da ação do JPMorgan, aplicando inteligência de tempo para comparar o valor atual com períodos anteriores, tratar lacunas de dias não úteis e calcular oscilações de mercado.

```dax
-- 1. Medida Base: Captura o valor bruto de fechamento no contexto atual
Preço Fechamento Simples = 
    MAX(JPM_Data[Close])


-- 2. Medida com Tratamento de Lacunas: Garante o preço do último dia útil em finais de semana e feriados
Preço Fechamento = 
CALCULATE (
    MAX ( JPM_Data[Close] ),
    LASTNONBLANK ( Calendario[Date], CALCULATE ( COUNT ( JPM_Data[Close] ) ) )
)


-- 3. Inteligência de Tempo (Diária): Desloca o contexto para capturar o preço do dia útil anterior
Fechamento Anterior = 
CALCULATE(
    [Preço Fechamento],
    PREVIOUSDAY('Calendario'[Date])
)


-- 4. Análise de Tendência Mensal: Calcula a variação percentual (MoM) de fechamento entre o mês atual e o mês anterior
Variação % Mensal = 
VAR FechamentoAtual =
    CALCULATE (
        MAX ( JPM_Data[Close] ),
        DATESMTD ( Calendario[Date] )
    )
VAR FechamentoAnterior =
    CALCULATE (
        MAX ( JPM_Data[Close] ),
        DATEADD ( Calendario[Date], -1, MONTH )
    )
RETURN
IF (
    ISBLANK ( FechamentoAnterior ) || FechamentoAnterior = 0,
    BLANK(),
    DIVIDE ( FechamentoAtual - FechamentoAnterior, FechamentoAnterior )
)

🔍 Como essas estruturas funcionam e se conectam:
DATESMTD: Delimita o cálculo do preço ao período do início do mês até a data atual mapeada (Month-to-Date).

DATEADD com -1, MONTH: Desloca a janela temporal exatamente um mês para trás na linha do tempo, permitindo capturar o valor de fechamento do período anterior.

DIVIDE com Validação: O teste lógico com ISBLANK e || (operador OU) age como uma trava de segurança. Ele garante que o Power BI não retorne erros visuais nas extremidades do gráfico (como no primeiríssimo mês de 1980, onde não existia histórico anterior para comparar).

Aplicação Visual: Esta fórmula alimenta diretamente o gráfico de colunas "Variação Percentual Mensal", dando clareza visual sobre os meses de maior ganho ou retração do ativo.


### 📊 Bloco de Medidas: Análise, Escala e Média de Volume Negociado

Este conjunto de medidas é responsável por mensurar a quantidade de ações transacionadas no mercado, adaptando os valores brutos para diferentes contextos visuais (cartões, gráficos e linhas de referência) para garantir a melhor leitura estética e aplicação dos princípios de Gestalt.

```dax
-- 1. Medida Base: Soma bruta de todas as ações negociadas no período
Volume Total = 
SUM ( JPM_Data[Volume] )


-- 2. Medida de Exibição: Formata o volume em bilhões com texto para uso em Cartões ou Tooltips
Volume Total Formatado = 
"USD " & FORMAT([Volume Total] / 1000000000, "0.00") & " Bi"


-- 3. Medida de Gráfico (Eixo Duplo): Reduz a escala para milhões para equilibrar o gráfico composto
Volume = [Volume Total] / 1000000


-- 4. Medida de Gráfico (Volume por Ano): Divide por 1 bilhão para simplificar os rótulos de dados
Volume para Eixo Bi = 
    DIVIDE([Volume Total], 1000000000)


-- 5. Linha de Referência: Calcula a média histórica diária global, ignorando filtros temporais da tela
Media Volume = 
CALCULATE(
    AVERAGEX(
        ALL('Calendario'),
        [Volume Total]
    )
)

🔍 Como essas estruturas funcionam no modelo:
SUM: Consolida o volume diário da tabela de fatos de acordo com os filtros de tempo ativos.

FORMAT com sufixo "Bi": Converte o número em uma string de texto amigável, ideal para KPIs.

Proporção e Escala (Volume e Volume para Eixo Bi): Aplicam a redução de escala para evitar a poluição visual de zeros excessivos nos gráficos, harmonizando os eixos visuais.

AVERAGEX + ALL: Cria uma linha de tendência ou linha de corte estática no gráfico. Ao usar o ALL, a fórmula limpa o contexto de filtro de tempo, permitindo comparar o desempenho de um mês específico contra a média histórica de toda a série temporal desde 1980.

---

## 🧪 Medidas de Auditoria e Validação (Testes de Qualidade)

Para garantir a acurácia dos cálculos de rentabilidade histórica e validar se as variações anuais estavam refletindo as quedas reais do mercado (especialmente em anos de crise), foram desenvolvidas duas medidas de auditoria interna.

### 📐 Medida Base de Teste: Valor_Num
Esta medida replica a matemática do patrimônio acumulado, mas retorna um valor puramente numérico (sem formatação de texto ou cifrão), servindo de base matemática para a medida de validação lógica.

```dax
Valor_Num = 
VAR CapitalInicialDigitado = SELECTEDVALUE('Capital Inicial'[Capital Inicial], 2000)
VAR AnoFiltrado = SELECTEDVALUE('Calendario'[Ano])

-- 1. Encontra o preço exato do primeiro dia da série temporal (1980) usando TOPN
VAR PrecoOrigem1980 =
    CALCULATE (
        MAX ( JPM_Data[Close] ),
        TOPN (
            1,
            ALL ( JPM_Data ),
            JPM_Data[Date], ASC
        )
    )

-- 2. Descobre a quantidade de cotas/ações adquiridas no início
VAR QuantidadeAcoesCompradas = DIVIDE(CapitalInicialDigitado, PrecoOrigem1980)

-- 3. Captura o preço do último dia útil do ano filtrado
VAR PrecoFechamentoAnoSelecionado =
    CALCULATE (
        MAX ( JPM_Data[Close] ),
        TOPN (
            1,
            FILTER (
                ALL ( JPM_Data ),
                YEAR ( JPM_Data[Date] ) = AnoFiltrado
            ),
            JPM_Data[Date], DESC
        )
    )

RETURN
    -- 4. Retorna a multiplicação bruta para ser consumida pela medida de teste
    QuantidadeAcoesCompradas * PrecoFechamentoAnoSelecionado

🔬 Medida de Validação: Teste_Queda
Esta medida atua como um teste lógico automatizado. Ela compara o patrimônio do ano atual contra o patrimônio do ano anterior (AnoAtual - 1) e valida visualmente se a curva de performance respondeu corretamente aos fechamentos negativos.

Teste_Queda = 
VAR AnoAtual = MAX('Calendario'[Ano])
VAR TemUmAno = HASONEVALUE('Calendario'[Ano])

VAR ValorAtual = [Valor_Num]
-- 1. Desloca o contexto de filtro para calcular o valor do patrimônio no ano anterior
VAR ValorAnterior =
    CALCULATE (
        [Valor_Num],
        FILTER (
            ALL ( 'Calendario' ),
            'Calendario'[Ano] = AnoAtual - 1
        )
    )

RETURN
-- 2. Validação lógica com travas de segurança
IF(
    NOT TemUmAno,
    BLANK(), -- Evita exibir o teste nas linhas de total geral do relatório
    IF(
        ValorAtual < ValorAnterior,
        "✅ Caiu",             -- Confirma que o modelo capturou a desvalorização anual
        "⚠ Subiu ou manteve"  -- Confirma estabilidade ou ganho
    )
)

🔍 Engenharia e Lógica Aplicada:
HASONEVALUE: Garante que o teste só seja executado se houver apenas um ano selecionado no contexto de linha (como em uma tabela ou matriz), limpando o total geral onde a comparação ano a ano perderia o sentido.

AnoAtual - 1: Executa uma subtração direta no contexto de filtro para criar um comparativo dinâmico sem depender de funções rígidas de inteligência de tempo.

Propósito do Teste: Ao cruzar o resultado desta medida com eventos históricos (como o estouro da bolha pontocom em 2000 ou a crise do Subprime em 2008), valida-se instantaneamente a integridade matemática de todo o simulador.

---

## 📅 Modelagem de Dados: Tabela Dimensão Calendário (`dCalendario`)

Para habilitar as análises de Inteligência de Tempo (Time Intelligence) e garantir a integridade dos relacionamentos no modelo Star Schema, foi desenvolvida uma tabela de dimensão de tempo customizada utilizando código DAX avançado.

```dax
Calendario = 
ADDCOLUMNS(
    -- 1. Cria automaticamente uma coluna única de datas cobrindo todo o período do modelo
    CALENDARAUTO(),
    
    "Dia Num", DAY([Date]),
    "Dia Nome", FORMAT([Date], "dddd"),
    "Dia Semana Num", WEEKDAY([Date]),
    "Semana Num", WEEKNUM([Date]),
    "Mês Num", MONTH([Date]),
    
    -- 2. Tratamento Estético: Garante a primeira letra do mês em maiúscula (Ex: "Janeiro")
    "Mês Nome", UPPER(LEFT(FORMAT([Date], "MMMM"), 1)) & RIGHT(FORMAT([Date], "MMMM"), LEN(FORMAT([Date], "MMMM")) - 1),
    
    -- 3. Tratamento Estético: Formata a abreviação do mês com a primeira letra maiúscula (Ex: "Jan")
    "Mês Curto", UPPER(LEFT(FORMAT([Date], "mmm"), 1)) & RIGHT(FORMAT([Date], "mmm"), 2),
    
    "Trimestre", "T" & QUARTER([Date]),
    "Ano", YEAR([Date])
)

🔍 Arquitetura e Engenharia de Atributos:
CALENDARAUTO(): Varre a tabela de fatos (JPM_Data) automaticamente para identificar o menor e o maior ano com registros históricos, eliminando a manutenção manual de datas.

Manipulação de Strings (UPPER + LEFT + RIGHT): Corrige a formatação padrão do Power BI em português (que gera os meses totalmente em letras minúsculas). Essa técnica isola a primeira letra, converte-a para maiúscula e concatena com o restante do texto da palavra, otimizando o visual de filtros, eixos e gráficos.

Colunas de Ordenação: As colunas textuais (como Mês Curto e Dia Nome) utilizam as colunas numéricas de suporte (Mês Num e Dia Semana Num) para ordenação correta nos visuais, impedindo que os meses sejam exibidos em ordem alfabética (Abril, Agosto, etc.).


---

## 🎨  Experiência do Usuário (UX) e Tooltips Dinâmicas

Para evitar a poluição visual no painel principal e fornecer insights profundos sob demanda, o projeto utiliza o conceito de **Custom Tooltips (Dicas de Ferramenta Personalizadas)**. Foi desenvolvida uma inteligência baseada em texto dinâmico que avalia a performance do ativo em tempo real.

### 📜 Medida: Texto_Tooltip Retorno
Esta medida unifica múltiplos cálculos matemáticos em blocos de texto formatados, injetando dinamicamente quebras de linha e testes condicionais de performance.

```dax
Texto_Tooltip Retorno = 
VAR ValorRetorno = [Retorno Anual %]

-- 1. Máscara de Formatação customizada para evidenciar valores positivos e negativos
VAR TextoFormatado = 
    FORMAT(ValorRetorno, "+0.00%;-0.00%;0.00%")

-- 2. Calcula a média histórica geral de retorno do ativo para servir de linha de corte
VAR MediaHistorica = 
    AVERAGEX(
        ALL(JPM_Data),
        [Retorno Anual %]
    )

VAR MediaFormatada = 
    FORMAT(MediaHistorica, "0.00%")

RETURN
-- 3. Trava de segurança caso o cursor não esteja posicionado sobre um ponto de dado útil
IF(
    ISBLANK(ValorRetorno),
    "Passe o mouse em um mês",

    -- 4. Concatenação de strings com formatação e quebra de linha automatizada (UNICHAR)
    "Retorno Anual: " & TextoFormatado &
    UNICHAR(10) & UNICHAR(10) &
    "Média histórica: " & MediaFormatada &
    UNICHAR(10) &
    "Situação: " &
    IF(
        ValorRetorno > MediaHistorica,
        "Acima da média",
        "Abaixo da média"
    )
)

🔍 Engenharia de Design e Princípios Aplicados:
FORMAT(..., "+0.00%;-0.00%"): Configura o comportamento estético do número. Se o retorno for positivo, força a exibição do sinal de mais (+), facilitando a leitura imediata pelo usuário (Princípio de Percepção Visual).

UNICHAR(10): Representa o caractere de quebra de linha (Line Feed) no padrão Unicode. Permite que uma única medida de texto distribua as informações verticalmente em formato de "parágrafos" dentro de um cartão comum, simulando uma tabela inteira.

Análise Comparativa Contextual (IF Aninhado): Avalia o desempenho do mês selecionado contra a MediaHistorica. Em vez de apenas exibir números brutos, a Tooltip entrega uma resposta interpretada para o tomador de decisão ("Acima da média" ou "Abaixo da média").

### 📜 Medida: Texto_Tooltip_Amplitude
Esta medida foi desenvolvida especificamente para enriquecer a análise de volatilidade no gráfico de Amplitude Média Diária (High–Low). Ela exibe de forma contextualizada o mês avaliado e a variação média em dólares.

```dax
Texto_Tooltip_Amplitude = 
    -- 1. Captura o nome do mês abreviado tratado esteticamente na dimensão calendário
    "Mês: " & SELECTEDVALUE( 'Calendario'[Mês Curto] ) & 
    UNICHAR(10) & 
    "Amplitude Média" & 
    UNICHAR(10) & 
    -- 2. Concatena o símbolo da moeda com o valor formatado em duas casas decimais
    "US$ " & FORMAT( [Amplitude Média], "0.00" )

🔍 Engenharia de Design e Princípios Aplicados:
Integração com a dCalendario (SELECTEDVALUE): O modelo lê dinamicamente o ponto do gráfico onde o usuário posicionou o mouse, isolando o atributo de tempo correto para a exibição do texto (ex: "Mês: Set").

Uso Estratégico de Espaçamento (UNICHAR(10)): A quebra de linha por caractere Unicode organiza as informações em três blocos verticais bem definidos, otimizando a leitura rápida (Scannability) sem demandar espaço excessivo na tela principal.

Consistência de Notação Financeira (FORMAT): Força a exibição padronizada das casas decimais mesmo para valores inteiros, mantendo a integridade visual dos dados financeiros apresentados.

### 📜 Medida: Texto_Tooltip_YTD
Esta medida foi estruturada para fornecer uma análise comparativa robusta de Year-to-Date (YTD) versus o ano anterior (YoY) no gráfico de linhas de Crescimento Acumulado, entregando contexto temporal imediato ao usuário.

```dax
Texto_Tooltip_YTD = 
VAR RetornoAtualYTD = [Crescimento YTD %]

-- 1. Desloca o período atual em exatamente um ano para trás para buscar o histórico equivalente
VAR RetornoAnoAnterior =
    CALCULATE([Crescimento YTD %], SAMEPERIODLASTYEAR(Calendario[Date]))

VAR MesSelecionado = SELECTEDVALUE(Calendario[Mês Curto])
VAR AnoSelecionado = SELECTEDVALUE(Calendario[Ano])

-- 2. Subtração direta para montar dinamicamente o rótulo de texto do ano passado
VAR AnoAnterior = AnoSelecionado - 1

RETURN
-- 3. Validação de segurança de contexto
IF(
    ISBLANK(RetornoAtualYTD),
    "Passe o mouse em um mês",
    
    -- 4. Construção da string concatenada com máscaras de porcentagem idênticas
    MesSelecionado & "/" & AnoSelecionado &
    UNICHAR(10) &
    "Crescimento YTD: " & FORMAT(RetornoAtualYTD, "0.0%") &
    UNICHAR(10) &
    "Ano anterior (" & MesSelecionado & "/" & AnoAnterior & "): " & 
    FORMAT(RetornoAnoAnterior, "0.0%")
)

🔍 Engenharia de Design e Princípios Aplicados:
SAMEPERIODLASTYEAR: Avalia o contexto de filtro temporal ativo no ponto do gráfico e desloca a janela de datas em exatamente -1 ano, mantendo o cálculo acumulado (YTD) perfeitamente alinhado com o ciclo anterior.

Construção Dinâmica de Strings: Ao criar a variável AnoAnterior e concatená-la diretamente no texto, a Tooltip evita mensagens estáticas e genéricas. O resultado entrega um rótulo altamente profissional focado na experiência de leitura (ex: "Ano anterior (Abr/2007):").

Alinhamento Estético de Unidades: Mantém o padrão de exibição percentual com uma casa decimal ("0.0%"), garantindo consistência visual com os demais indicadores macro do relatório.

### 📜 Medida: Texto_Tooltip_Preco_Tendencia
Esta medida atua no gráfico composto de Preço vs. Tendência de Longo Prazo. Ela disseca o preço real de fechamento contra a linha matemática de tendência, calculando o spread (diferença) absoluto e injetando formatação financeira cirúrgica para valores acima ou abaixo da média histórica.

```dax
Texto_Tooltip_Preco_Tendencia = 
VAR PrecoFechamento = [Preço Fechamento]
VAR TendenciaLongoPrazo = [Tendencia_Longo_Prazo]
VAR Diferenca = PrecoFechamento - TendenciaLongoPrazo

VAR DataSelecionada = MAX(Calendario[Date])
VAR MesRaw = FORMAT(DataSelecionada, "MMM")

-- 1. Tratamento Estético: Garante consistência visual com a dCalendario (Ex: "Dez")
VAR MesSelecionado = UPPER(LEFT(MesRaw,1)) & RIGHT(MesRaw,LEN(MesRaw)-1)
VAR AnoSelecionado = YEAR(DataSelecionada)

RETURN
-- 2. Validação de presença de dados
IF(
    ISBLANK(PrecoFechamento),
    "Passe o mouse em um mês",
    MesSelecionado & "/" & AnoSelecionado &
    UNICHAR(10) &
    "Preço de fechamento: US$ " & FORMAT(PrecoFechamento, "#,##0.00") &
    UNICHAR(10) &
    "Tendência de longo prazo: US$ " & FORMAT(TendenciaLongoPrazo, "#,##0.00") &
    UNICHAR(10) &
    "Diferença: " &
        -- 3. Engenharia de String Financeira: Evita quebras de máscara com números negativos
        IF(
            Diferenca > 0,
            "+US$ " & FORMAT(Diferenca, "#,##0.00"),
            "-US$ " & FORMAT(ABS(Diferenca), "#,##0.00") -- Usa ABS para isolar o sinal e fixá-lo antes do cifrão
        )
)

🔍 Engenharia de Design e Princípios Aplicados:
Uso da Função ABS() (Valor Absoluto): Retira o sinal negativo nativo do número quando a diferença está abaixo da tendência. Isso permite que o desenvolvedor manipule textualmente a posição exata do caractere "-", evitando bugs visuais na máscara da moeda (garantindo "-US$ 4,11" em vez de "US$ -4,11").

Análise de Spread de Mercado: Permite que o usuário identifique em frações de segundo se o ativo do JPMorgan está sobrevalorizado (negociado com prêmio acima da tendência) ou descontado (negociado com desconto abaixo da tendência) naquele mês específico.

Consistência Visual Centrada: Padroniza os separadores de milhar e decimal ("#,##0.00"), alinhando a experiência estética com relatórios de auditoria internacionais.

### 📜 Medida: Texto_Tooltip_Volume
Esta medida foi projetada para o gráfico de barras de Volume Histórico. Ela converte os valores brutos de transações em bilhões, calcula a variação percentual em relação ao ano anterior (YoY) e gera rótulos temporais dinâmicos e contextualizados.

```dax
Texto_Tooltip_Volume = 
VAR VolumeAno = [Volume Total]

-- 1. Desloca o contexto de tempo para buscar o volume financeiro do ano anterior
VAR VolumeAnoAnterior =
    CALCULATE(
        [Volume Total],
        SAMEPERIODLASTYEAR(Calendario[Date])
    )

-- 2. Calcula a variação percentual ano a ano de forma segura contra divisão por zero
VAR Variacao = DIVIDE(VolumeAno - VolumeAnoAnterior, VolumeAnoAnterior)

VAR AnoSelecionado = SELECTEDVALUE(Calendario[Ano])

RETURN
-- 3. Validação de segurança de contexto
IF(
    ISBLANK(VolumeAno),
    "Passe o mouse em um ano",
    
    -- 4. Formatação e conversão direta para a escala de Bilhões (Bi)
    "Ano " & AnoSelecionado &
    UNICHAR(10) &
    "Volume Total: US$ " & FORMAT(VolumeAno / 1000000000, "0.00") & " Bi" &
    UNICHAR(10) &
    "Ano anterior (" & AnoSelecionado - 1 & "): US$ " & FORMAT(VolumeAnoAnterior / 1000000000, "0.00") & " Bi" &
    UNICHAR(10) &
    "Variação: " & FORMAT(Variacao, "0.0%")
)

🔍 Engenharia de Design e Princípios Aplicados (Tooltip & Eixo):
Otimização de Escala Financeira: Ao dividir os valores por 1.000.000.000 tanto na Tooltip quanto no Eixo, o modelo elimina sequências massivas de zeros na tela (ex: exibindo US$ 10,37 Bi ao invés de US$ 10.370.000.000), o que reduz drasticamente a carga cognitiva do usuário.

Cálculo YoY Seguro (DIVIDE): A função DIVIDE intercepta internamente qualquer falha de herança de dados ou anos iniciais vazios na tabela de fatos, retornando um resultado limpo no cálculo da Variacao.

Rótulos Matemáticos Inteligentes: A expressão AnoSelecionado - 1 monta de forma automatizada o ano de comparação no texto, permitindo que qualquer alteração de filtro ou segmentação atualize a legenda da dica de ferramenta em tempo real.

### 📜 Medida: Texto_Tooltip_Variação Mensal
Esta medida foi desenvolvida para monitorar as oscilações pontuais e mensais do ativo. Ela compara o rendimento percentual do mês atual diretamente com o desempenho do mesmo mês no ano anterior, fornecendo uma visão clara de sazonalidade e tendências de curto prazo.

```dax
Texto_Tooltip_Variação Mensal = 
VAR RetornoMensal = [Variação % Mensal]

-- 1. Captura o comportamento do indicador recuando exatamente um ano no tempo
VAR RetornoMensalAnoAnterior =
    CALCULATE(
        [Variação % Mensal],
        SAMEPERIODLASTYEAR(Calendario[Date])
    )

VAR MesSelecionado = SELECTEDVALUE(Calendario[Mês Curto])
VAR AnoSelecionado = SELECTEDVALUE(Calendario[Ano])
VAR AnoAnterior = AnoSelecionado - 1

RETURN
-- 2. Validação de segurança de contexto
IF(
    ISBLANK(RetornoMensal),
    "Passe o mouse em um mês",
    
    -- 3. Interpolação de texto dinâmico com máscaras percentuais de uma casa decimal
    MesSelecionado & "/" & AnoSelecionado &
    UNICHAR(10) &
    "Variação mensal: " & FORMAT(RetornoMensal, "0.0%") &
    UNICHAR(10) &
    "Ano anterior (" & MesSelecionado & "/" & AnoAnterior & "): " & 
    FORMAT(RetornoMensalAnoAnterior, "0.0%")
)

🔍 Engenharia de Design e Princípios Aplicados:
Análise de Sazonalidade YoY Mensal: Permite ao investidor avaliar se a queda ou alta em um mês específico (Ex: Junho) é uma anomalia de mercado ou um comportamento padrão repetitivo do ativo quando comparado ao ano anterior.

Preservação de Contexto Sincronizado: Utiliza as variáveis de tempo para reconstruir o rótulo da legenda dinamicamente, mantendo o padrão estético e a fácil leitura (Readability) de todo o ecossistema de dicas de ferramenta do relatório.

### 📜 Medida: Texto_Tooltip_Volume_Preco
Esta medida foi projetada de forma exclusiva para o gráfico combinado de barras e linhas (Volume Total vs. Preço de Fechamento). Ela unifica dados de duas naturezas diferentes (volume transacionado em milhões e preço do ativo em dólares) em um único bloco de insights rápidos, calculando o desvio em relação à tendência.

```dax
Texto_Tooltip_Volume_Preco = 
VAR VolumeAno = [Volume Total]
VAR PrecoFechamento = [Preço Fechamento]
VAR TendenciaLongoPrazo = [Tendencia_Longo_Prazo]
VAR DiferencaPreco = PrecoFechamento - TendenciaLongoPrazo

VAR DataSelecionada = MAX(Calendario[Date])
VAR MesRaw = FORMAT(DataSelecionada, "MMM")

-- 1. Tratamento Estético: Mantém a primeira letra do mês em maiúscula (Ex: "Dez")
VAR MesSelecionado = UPPER(LEFT(MesRaw,1)) & RIGHT(MesRaw,LEN(MesRaw)-1)
VAR AnoSelecionado = YEAR(DataSelecionada)

RETURN
-- 2. Validação Lógica Restritiva: Só exibe o texto se ambos os indicadores coexistirem no ponto
IF(
    ISBLANK(VolumeAno) || ISBLANK(PrecoFechamento),
    "Passe o mouse em um ponto",
    MesSelecionado & "/" & AnoSelecionado &
    UNICHAR(10) &
    -- 3. Escala em Milhões: Ajuste milimétrico para o contexto gráfico mensal
    "Volume Total: US$ " & FORMAT(VolumeAno / 1000000, "0.00") & " Mi" &
    UNICHAR(10) &
    "Preço de fechamento: US$ " & FORMAT(PrecoFechamento, "#,##0.00") &
    UNICHAR(10) &
    "Diferença vs tendência: " &
        -- 4. Formatação de Sinais Isolados com ABS para evitar erros na string monetária
        IF(
            DiferencaPreco > 0,
            "+US$ " & FORMAT(DiferencaPreco, "#,##0.00"),
            "-US$ " & FORMAT(ABS(DiferencaPreco), "#,##0.00")
        )
)

🔍 Engenharia de Design e Princípios Aplicados:
Escalabilidade Adaptativa (/ 1000000): Demonstra maturidade na modelagem visual ao perceber que o volume agregado mensal deve ser expresso na casa dos milhões (Mi), enquanto o volume agregado anual utiliza bilhões (Bi). Isso respeita a carga cognitiva ideal para o usuário técnico.

Operador de Curto-Circuito (||): Aplica uma validação rigorosa que protege o componente visual caso ocorra um mês atípico com negociações suspensas (preço ou volume nulos), impedindo exibições parciais ou incoerentes.

