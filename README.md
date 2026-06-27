# analise-historica-jpmorgan-powerbi
## 📌 Objetivo
Dashboard interativo em Power BI para análise histórica da performance dos ativos da JPMorgan (1980–2026).  
Permite avaliar rentabilidade, volatilidade, liquidez e tendências de mercado em diferentes períodos.

### 📷 Visualização do Dashboard
Visualização geral do dashboard em Power BI, destacando indicadores de retorno, volatilidade, liquidez e tendências históricas dos ativos da JPMorgan  
![Dashboard JPMorgan](Dashboard.png)


## 📌 Objetivo
Dashboard interativo em Power BI para análise histórica da performance dos ativos da JPMorgan (1980–2026). Permite avaliar rentabilidade, volatilidade, liquidez e tendências de mercado em diferentes períodos econômicos.

## 📊 Indicadores e Métricas Mapeadas
* **Retorno no Ano (%)** (Métrica dinâmica que calcula o rendimento do ativo no ano selecionado)[cite: 6].
* **Valor Final Acumulado** (Resultado da simulação com base no capital inicial)[cite: 6].
* **Capital Inicial** (Parâmetro interativo inserido pelo usuário para simulações)[cite: 6].
* **Crescimento YTD % por Mês** (*Year-to-Date* - velocidade do acúmulo de ganho ao longo dos meses)[cite: 6].
* **Amplitude Média Diária** (Termômetro de volatilidade baseado na distância entre as máximas e mínimas - *High-Low*)[cite: 6].
* **Volume Total por Ano** (Métrica de liquidez e fluxo de negociação a longo prazo)[cite: 6].
* **Variação Percentual Mensal** (Performance isolada de cada mês do ano)[cite: 6].
* **Volume Total vs. Preço de Fechamento** (Análise combinada de fluxo financeiro contra tendência de preço)[cite: 6].

## 🛠️ Tecnologias e Conceitos Aplicados
* **Power BI Desktop:** Construção integral do ecossistema de análise[cite: 6].
* **Modelagem de Dados:** Estruturação de tabelas e relacionamentos eficientes[cite: 6].
* **Linguagem DAX Avançada:** Criação de medidas calculadas e inteligência de tempo para análises comparativas[cite: 6].
* **Design de Dashboards (UX/UI):** Criação de uma interface limpa, intuitiva e estruturada sob princípios de Storytelling de Dados e clareza visual[cite: 6].

## ⚠️ Observações Técnicas
* Os dados do ano de 2026 estão disponíveis e atualizados até o mês de abril[cite: 6].
* O dashboard inclui uma funcionalidade exclusiva de simulação de investimentos com base em um capital inicial totalmente personalizado pelo usuário[cite: 6].

## 📊 Estrutura e Mecânica dos Gráficos

<details>
<summary>💡 Clique aqui para entender o que cada gráfico do painel analisa (Sem Spoiler)</summary>

Para mapear a saúde financeira e o comportamento dos ativos da JPMorgan, este dashboard foi estruturado a partir de seis visões complementares:

* **Preço de Fechamento vs. Tendência de Longo Prazo:** Mostra a evolução histórica do valor da ação ao longo das décadas e valida visualmente o princípio de resiliência e recuperação do banco após ciclos de baixa.
* **Volume Total por Ano:** Mede a quantidade total de ações negociadas ano a ano, servindo como um termômetro para identificar picos de movimentação e o comportamento do investidor em cenários de instabilidade.
* **Variação Percentual Mensal:** Revela como as oscilações acontecem na escala mensal, permitindo identificar que o desempenho do mercado não é linear e apresenta picos isolados de alta ou baixa mesmo em anos difíceis.
* **Crescimento YTD % por Mês (Year To Date):** Monitora a velocidade e o acúmulo da rentabilidade desde o início de janeiro até o mês corrente, facilitando a identificação exata do trimestre onde o ativo ganhou tração.
* **Amplitude Média Diária:** Atua como o nosso indicador direto de risco e volatilidade prática, calculando o distanciamento entre as máximas e mínimas diárias (*High-Low*) para apontar períodos de forte agitação de mercado.
* **Volume Total vs. Preço de Fechamento:** Um gráfico combinado que cruza a liquidez com a linha de preço para diagnosticar se uma tendência de alta ou de baixa é sustentável e apoiada por fluxo institucional de capital.

</details>

--- 

### 🔍 Estudos de Caso Práticos (Navegando pelo Dashboard)

Para validar a utilidade prática das métricas desenvolvidas, utilizei os filtros do painel para isolar e analisar  três momentos históricos distintos da instituição:

#### 1️⃣ Crise Financeira Global (Estudo de Caso: 2008)
* **Constatação Visual:** No gráfico de longo prazo, o preço das ações tendeu a "andar de lado", apresentando uma leve queda seguida de estabilização. Em contrapartida, o gráfico de *Volume Total por Ano* registrou um salto massivo, atingindo picos históricos entre 2008 e 2010. Ao nível mensal, o gráfico de *Variação Percentual Mensal* mostra que o ano fechou com saldo negativo de -23,19%, mas com oscilações intensas e picos isolados de alta em meses específicos, como Setembro.
* **Contexto de Negócio:** Esse padrão reflete o comportamento típico de mercado durante a Crise do *Subprime*. O pico drástico de volume indica uma busca extrema por liquidez e realocação de capital por parte dos investidores (pânico e negociação em massa). A estabilidade relativa do preço do JPM, enquanto concorrentes gigantes faliam na mesma época, reforça a sólida resiliência institucional do JPMorgan frente ao colapso do sistema bancário.

#### 😷 2️⃣ Choque Global Repentino e Recuperação em "V" (Estudo de Caso: 2020)
* **Constatação Visual:** O gráfico de *Variação Percentual Mensal* desenhou o clássico efeito de "Montanha-Russa". Registrou-se um recuo severo no primeiro quadrimestre, com quedas consecutivas de -11,9% em março e um agravamento para -14,6% em abril. No mesmo período, o gráfico de *Amplitude Média Diária* disparou significativamente acima da linha tracejada de média. Já o mês de novembro apresentou uma virada espetacular com uma alta isolada de +20,2%.
* **Contexto de Negócio:** Este comportamento traduz visualmente o impacto da pandemia da COVID-19 e o conceito prático de volatilidade. O colapso e a forte agitação de preços (High-Low) em março e abril refletem o pânico macroeconômico e a paralisação da economia com o início dos *lockdowns* mundiais. Por outro lado, a forte valorização e quebra de tendência em novembro quantificam o otimismo imediato dos investidores e a reação direta do mercado financeiro ao anúncio da eficácia das primeiras vacinas globais.

<details>
<summary>📸 Clique aqui para ver o Dashboard filtrado em 2020</summary>

![Dashboard JPMorgan 2020](Dashboard_2020.png)
</details>

#### 🚀 3️⃣ Ciclo de Expansão e Maturidade de Mercado (Estudo de Caso: 2025)
* **Constatação Visual:** O indicador de *Retorno no Ano (%)* consolidou-se fortemente no campo positivo, atingindo uma valorização expressiva de 37,10%. O gráfico de *Crescimento YTD % por Mês* desenhando uma trajetória de ascensão contínua e linear (como uma escada para cima), interrompida apenas por breves correções saudáveis. Diferente dos anos de crise, as colunas de volume mantiveram-se estáveis, previsíveis e próximas à média histórica.
* **Contexto de Negócio:** Este padrão representa um ambiente clássico de *Bull Market* (mercado em alta) saudável e sustentável. A valorização expressiva do preço de fechamento, sem a necessidade de uma explosão desordenada no volume de transações, indica que a alta é movida por confiança institucional genuína e solidez nos fundamentos econômicos do banco, e não por especulação barata ou pânico de liquidez.

<details>
<summary>📸 Clique aqui para ver o Dashboard filtrado em 2025</summary>

![Dashboard JPMorgan 2025](Dashboard_2025.png)
</details>

## 📈 Conclusão do Projeto e Aprendizado

Este projeto foi essencial para o meu desenvolvimento em Business Intelligence. Ele me deu a oportunidade de ir além da teoria e aplicar na prática conceitos importantes de Power BI e DAX. 

Durante o desenvolvimento, meus principais focos foram:
* **Entender as conexões e filtros:** Aprender como os segmentadores influenciam os gráficos e como travar os visuais longos para que a linha do tempo não quebrasse.
* **Praticar a leitura dos gráficos:** Treinar o olhar para identificar onde estão as maiores subidas, quedas e os picos de movimentação nas colunas.
* **Organização visual:** Criar um layout limpo, combinando gráficos mensais e históricos na mesma tela sem deixar o visual poluído.

Este dashboard consolida a minha base em Power BI e me dá muito mais segurança para continuar estudando, montando novos modelos e avançando na análise de dados!
