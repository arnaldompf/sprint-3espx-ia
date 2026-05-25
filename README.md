# Segmentação e Classificação de Clientes: Rede de Manutenção Ford
## Relatório Executivo e Documentação Técnica do Projeto

Este repositório contém a solução analítica desenvolvida para a **Ford** com o objetivo de aumentar a retenção de clientes em sua rede oficial de concessionárias e oficinas autorizadas. O projeto soluciona um desafio crítico de negócio: **prever o comportamento de retenção de um cliente no momento exato da compra do veículo (Dia 0)**, permitindo ações de marketing e pós-venda preventivas e personalizadas.

---

## 1. Introdução e Objetivos de Negócio

Manter o cliente engajado na rede oficial de pós-venda ao longo do ciclo de vida do veículo é um dos pilares de rentabilidade e fidelização da Ford. O principal desafio deste projeto reside na assimetria temporal: as ações de retenção precisam ser tomadas no momento da venda, mas o comportamento real do consumidor só se revela no futuro.

Para mitigar este problema, a solução foi dividida em duas frentes complementares:
1. **Fase de Segmentação (Não Supervisionada):** Análise do histórico transacional completo para identificar e agrupar os clientes em perfis comportamentais reais.
2. **Fase de Classificação (Supervisionada):** Criação de um modelo preditivo treinado exclusivamente com dados coletados no ato da primeira compra para inferir a qual perfil o novo cliente pertencerá no futuro.

---

## 2. Preparação dos Dados e Mitigação Estrita de *Data Leakage*

O dataset original consiste em uma base transacional pura com **541.909 registros**, onde cada linha representa um item faturado. 

### Limpeza e Tratamento Inicial
* **Filtragem de Identificação:** Remoção de registros sem identificação do cliente (`CustomerID` nulo), resultando em **406.829 linhas** válidas estruturadas de forma consistente.
* **Correção de Tipos:** Conversão da coluna de preços (`UnitPrice`) para ponto flutuante e enriquecimento da base com a variável `Total_Spent` (Quantidade $\times$ Preço Unitário).
* **Padronização Temporal:** Tratamento das datas da fatura (`InvoiceDate`) tratando inconsistências de formato de maneira segura.

### Engenharia de Atributos e Divisão de Janelas Temporais
Para cumprir rigorosamente a regra de negócio que proíbe o uso de variáveis futuras na predição (evitando o **vazamento de dados** ou *data leakage*), a base única foi dividida em duas estruturas lógicas totalmente independentes por cliente (`CustomerID`):

1. **Base 1 (Histórico Completo - Visão de Futuro):** Consolida todo o comportamento histórico de retorno do cliente (Frequência total de visitas, Gasto acumulado e Recência em dias desde a última visita). Utilizada **apenas** para gerar os agrupamentos (*clusters*).
2. **Base 2 (Momento da Compra - Visão do Dia 0):** Isola estritamente a **primeira transação** cronológica de cada um dos **4.372 clientes únicos**. Variáveis como o mês da compra, o dia da semana, o valor gasto inicial e a quantidade de itens comprados no primeiro dia foram extraídas para servirem de preditores.

---

## 3. Fase 1: Segmentação Comportamental (Machine Learning Não Supervisionado)

Utilizando a **Base 1**, os dados de comportamento foram normalizados através do `StandardScaler` para balancear as escalas de valores monetários e contagens de dias. O número ideal de segmentos foi definido aplicando o **Método do Cotovelo (Inertia Curve)**, revelando uma inflexão matemática clara no ponto de **4 clusters**, validando perfeitamente a hipótese de negócio inicial.

### Métricas Médias dos Grupos Encontrados (K-Means)

| Métrica | Cluster 0 | Cluster 1 | Cluster 2 | Cluster 3 |
| :--- | :---: | :---: | :---: | :---: |
| **Volume de Clientes** | 3.169 | 1.087 | 6 | 110 |
| **Frequência Média (Visitas)** | 4,80 | 1,81 | 89,00 | 40,67 |
| **Gasto Médio Histórico (R$)** | 1.478,52 | 453,49 | 182.181,98 | 18.441,96 |
| **Dias de Ausência Média (Recência)** | 40,61 | 246,95 | 6,67 | 8,18 |

### Tradução dos Segmentos em Perfis de Negócio

* **Cluster 1 — Cliente de Abandono (Alto Risco):** Apresenta uma frequência baixíssima (média de 1,8 visitas), ticket médio reduzido e está ausente da rede há mais de **246 dias**. Representa o consumidor que realiza apenas as revisões mandatórias iniciais e migra para oficinas independentes baratas.
* **Cluster 0 — Cliente Econômico / Massa:** É a maioria absoluta da base de clientes (3.169 indivíduos). Frequenta a concessionária de forma moderada (4,8 vezes) e está ativo (40 dias de ausência), mas gasta valores baixos por passagem. É o cliente altamente sensível a preços, campanhas de peças e combos de serviços básicos.
* **Cluster 3 — Cliente Fiel:** Grupo composto por 110 clientes de alto valor. Possuem engajamento contínuo (mais de 40 visitas registradas), gasto médio robusto de R$ 18,4k e altíssima assiduidade (ausentes há apenas 8 dias). Consomem serviços e acessórios premium e priorizam a chancela oficial da marca à economia financeira.
* **Cluster 2 — Grandes Contas / Frotistas (Anomalia Estatística):** Uma minoria extrema de 6 clientes que gerou um volume astronômico de interações (89 visitas) e faturamento médio superior a R$ 182 mil. Comportam-se de forma completamente distinta de pessoas físicas, representando contas corporativas (B2B), frotas comerciais ou locadoras.

---

## 4. Estratégias de Retenção Recomendadas

Com base nos perfis comportamentais descobertos, foram desenhadas quatro réguas de relacionamento acionáveis para a rede de concessionárias Ford:

* **Ação para o Cliente de Abandono:** Implementação imediata de uma régua de reativação via canais digitais com pesquisas de satisfação estruturadas (NPS pós-venda) e ofertas agressivas de serviços essenciais (como troca de óleo e pastilhas de freio) a preço de custo para trazer o cliente de volta à oficina oficial.
* **Ação para o Cliente Econômico:** Criação de um menu de serviços com "Preço Fixo Transparente" e parcelamento estendido. Introdução de programas de fidelidade estruturados com acúmulo de pontos que geram descontos progressivos nas revisões subsequentes, neutralizando a atratividade do mercado paralelo.
* **Ação para o Cliente Fiel:** Desenvolvimento de uma esteira de relacionamento VIP (*Ford Concierge*). Inclusão de benefícios de conveniência como serviço de leva-e-traz por guincho plataforma, lavagem detalhada premium como cortesia em toda manutenção e convites para pré-lançamentos de veículos novos.
* **Ação para Grandes Contas / Frotistas:** Alocação de um Gerente de Contas B2B dedicado para negociação de contratos de manutenção preventiva programada com faturamento corporativo unificado de frotas e pacotes de peças com desconto por volume.

---

## 5. Fase 2: Modelagem Preditiva Supervisionada (Previsão no Dia 0)

Para realizar a predição no ato da compra, os clusters definidos na Fase 1 foram convertidos na variável-alvo (`y`). O modelo foi treinado utilizando o algoritmo **Random Forest Classifier** com pesos balanceados (`class_weight='balanced'`) para mitigar o forte desbalanceamento das classes. 

### Resultados de Desempenho do Modelo
* **Acurácia Geral:** **66,39%**

```
=== RELATÓRIO DE CLASSIFICAÇÃO (Precision, Recall, F1) ===
              precision    recall  f1-score   support

0: Econômico       0.76      0.81      0.78       951
1: Abandono        0.37      0.31      0.34       326
2: Frotista        0.00      0.00      0.00         2
3: Fiel            0.04      0.03      0.03        33
```

### Análise Crítica da Matriz de Confusão
O modelo apresenta uma excelente performance para identificar o cliente majoritário (**Econômico**), alcançando um *Recall* de **81%** (769 acertos de 951). No entanto, o principal desafio analítico mapeado reside no **Cliente de Abandono**: o modelo classificou incorretamente 220 clientes de abandono real como sendo econômicos.

* **Impacto Operacional:** Esse erro faz com que a concessionária deixe de intervir preventivamente em 220 clientes propensos à evasão, tratando-os como clientes de rotina. Esse desbalanceamento indica que os dados disponíveis estritamente no Dia 0 possuem limitações para antecipar comportamentos de fidelidade extrema de forma puramente linear, exigindo futuras coletas de dados demográficos adicionais (como renda ou idade do comprador).

### Importância das Variáveis (Feature Importance)
O algoritmo revelou as 5 métricas coletadas no momento da compra que possuem maior poder preditivo sobre o comportamento futuro do cliente:

1. **Gasto Inicial (32,3%):** O volume financeiro investido na primeira compra é o maior indicador do perfil de retenção.
2. **Mês da Compra (26,2%):** Revela fortes padrões sazonais ligados a ciclos financeiros do consumidor (IPVA, bônus corporativos, férias).
3. **Quantidade de Itens Iniciais (18,5%):** Mede a diversidade de produtos/serviços adquiridos logo na primeira interação.
4. **Dia da Semana da Compra (11,2%):** Indica padrões de conveniência do comprador.
5. **Geografia / Country (4,1%):** Pequenas variações regionais de mercado.

---

## 6. Aplicação Prática no Dia a Dia da Concessionária

A engenharia de software desenvolvida permite que esta solução seja consumida em tempo real na operação física das concessionárias Ford através dos seguintes passos:

1. **Pontuação em Tempo Real (Real-Time Scoring):** Integração do modelo preditivo Random Forest (via API REST compilada em arquivo pickle) diretamente com o ERP de faturamento de veículos e ordens de serviço da oficina.
2. **Gatilho de Balcão (Point-of-Sale Triggers):** No instante em que a primeira transação comercial de um novo veículo é fechada, o sistema envia as variáveis de entrada (Valor, Itens, Data) para o modelo. Caso o cliente seja classificado como **Perfil 1 (Abandono)**, um alerta visual é emitido na tela do consultor técnico de pós-venda.
3. **Retenção Preventiva Proativa:** Antes que o cliente saia da concessionária com seu veículo novo, o consultor realiza uma abordagem consultiva entregando um voucher impresso e intransferível que concede descontos expressivos ou a primeira troca de óleo gratuita condicionado ao cumprimento exato do prazo da próxima revisão.
4. **Automação de Marketing Direcionada:** Clientes identificados como **Econômicos** entram de forma automática em uma esteira de nutrição de e-mails e mensagens de WhatsApp focadas em combos de peças e preços promocionais, otimizando os investimentos de marketing da marca.

---

## 7. Estrutura do Jupyter Notebook (`.ipynb`)

O arquivo técnico está organizado de forma estrita para reprodutibilidade no ambiente **WSL2 (Ubuntu)** dentro da seguinte ordem de células:
1. `Instalação de Dependências e Configuração do Ambiente Virtual (.venv)`
2. `Carga e Tratamento Inicial de Tipos da Base Transacional`
3. `Segregação Temporal de Dados para Prevenção do Data Leakage`
4. `Padronização de Dados e Plotagem do Gráfico de Curva do Cotovelo`
5. `Execução do Algoritmo K-Means com 4 Clusters Definidos`
6. `Análise Agrupada de Médias e Rotulação de Negócio`
7. `Treinamento do Classificador Supervisionado Random Forest`
8. `Geração do Classification Report e Matriz de Confusão com Seaborn`

## 8. Integrantes Do Grupo

- **Arnaldo Filho** - RM: 555780
- **Carlos Menezes** - RM 99849
- **Vinicius Gardim** - RM 556013
- **Fabricio Carlos** - RM 555017
- **Leonardo Moura** - RM 550413
