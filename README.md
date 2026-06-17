# analise-historica-jpmorgan-powerbi
## 📌 Objetivo
Dashboard interativo em Power BI para análise histórica da performance dos ativos da JPMorgan (1980–2026).  
Permite avaliar rentabilidade, volatilidade, liquidez e tendências de mercado em diferentes períodos.

### 📷 Visualização do Dashboard
Visualização geral do dashboard em Power BI, destacando indicadores de retorno, volatilidade, liquidez e tendências históricas dos ativos da JPMorgan  
![Dashboard JPMorgan](Dashboard.png)


## 📊 Indicadores
- Retorno Anual (%)  
- Valor Final Acumulado  
- Crescimento YTD (%)  
- Amplitude Média (volatilidade)  
- Volume Total negociado  
- Variação Percentual Mensal  

## 🛠️ Tecnologias Utilizadas
- Power BI Desktop  
- Modelagem de dados  
- DAX avançado  
- Visualizações interativas  

## ⚠️ Observações
- Os dados de 2026 estão disponíveis apenas até abril.  
- O dashboard inclui simulação de investimentos com capital inicial personalizado.  

## 📊 Análise de Dados e Descobertas (Visão de Negócio)

Construir este dashboard me permitiu praticar a leitura de padrões visuais e entender como o mercado se comporta na prática. Analisando os 6 gráficos do painel, consegui extrair as seguintes conclusões:

* **Padrão de Recuperação Histórica (Preço de Fechamento vs. Tendência de Longo Prazo):** Olhando para a série histórica completa de 1980 até 2026, fica evidente que o preço das ações passou por quedas acentuadas em momentos específicos. No entanto, o padrão visual mostra resiliência: após cada crise, a linha sempre voltou a subir e superou os recordes anteriores no longo prazo.
* **Explosão de Movimentação em Crises (Volume Total por Ano):** No gráfico de colunas históricas, chama a atenção o crescimento enorme das barras entre 2008 e 2010. Isso me mostra que, em períodos de grande instabilidade, a quantidade de ações negociadas aumenta drasticamente, indicando forte movimentação dos investidores.
* **Comportamento Não Linear (Variação Percentual Mensal):** Analisando as colunas de janeiro a dezembro, percebi que o desempenho não segue uma linha reta. O ano de 2008, por exemplo, fechou com um saldo anual negativo expressivo (-23,19%), mas os meses internos (como Setembro) apresentaram picos isolados de alta, mostrando que o mercado oscila muito mês a mês.
* **Acompanhamento de Velocidade (Crescimento YTD % por Mês):** Este gráfico de linha me permite ver o acumulado do ano corrente. Ele mostra como a rentabilidade vai se somando mês após mês, ajudando a identificar em qual período do ano o ativo pegou mais impulso ou perdeu força.
* **Mapeamento de Agitação (Amplitude Média Diária):** Este gráfico me ajudou a entender o conceito de risco na prática. Nos meses onde a linha da amplitude sobe e fica acima da linha média tracejada, significa que a distância entre o preço mais alto e o mais baixo do mesmo dia foi maior, indicando meses de maior agitação e incerteza no mercado.
* **Comparação de Força (Volume Total vs. Preço de Fechamento):** Este gráfico combinado me permite cruzar duas informações vitais. Olhando para o comportamento das barras de volume junto com a linha de preço ao longo dos meses, consigo avaliar se as maiores mudanças no preço das ações aconteceram com muita ou pouca movimentação de capital no mercado.

## 📈 Conclusão do Projeto e Aprendizado

Este projeto foi essencial para o meu desenvolvimento em Business Intelligence. Ele me deu a oportunidade de ir além da teoria e aplicar na prática conceitos importantes de Power BI e DAX. 

Durante o desenvolvimento, meus principais focos foram:
* **Entender as conexões e filtros:** Aprender como os segmentadores influenciam os gráficos e como travar os visuais longos para que a linha do tempo não quebrasse.
* **Praticar a leitura dos gráficos:** Treinar o olhar para identificar onde estão as maiores subidas, quedas e os picos de movimentação nas colunas.
* **Organização visual:** Criar um layout limpo, combinando gráficos mensais e históricos na mesma tela sem deixar o visual poluído.

Este dashboard consolida a minha base em Power BI e me dá muito mais segurança para continuar estudando, montando novos modelos e avançando na análise de dados!
