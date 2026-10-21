# ⚽ World Cup 2026: Análise Preditiva e Modelagem de Dados

Projeto prático desenvolvido para o **Seminário de Ciência de Dados**, focado na análise exploratória, validação de hipóteses e modelagem preditiva para a **Copa do Mundo FIFA 2026**.

O projeto combina técnicas estatísticas e de *Machine Learning* para responder a perguntas centrais sobre o desempenho histórico de seleções, impacto do fator mandante, agrupamento de perfis de equipes e projeções probabilísticas dos favoritos ao título mundial em 2026.

---

## 👥 Colaboração e Divisão de Responsabilidades

Este projeto foi concebido e implementado de forma colaborativa:

* **Newton Ruck ([@NewtonRuck](https://github.com/NewtonRuck))**:
  * **Aprendizado Supervisionado**: Arquitetura do classificador de confrontos diretos (*Random Forest*) e motor de **Simulação de Monte Carlo** (chaveamento oficial da Copa 2026 com 48 seleções em 1.000 iterações).
  * **Regressão Linear**: Modelagem da relação estatística entre o Ranking FIFA histórico e o volume de gols marcados em Copas do Mundo.
* **Davi**:
  * **Aprendizado Não Supervisionado**: Clusterização das seleções da Copa de 2022 via *K-Means* e determinação do número ideal de grupos através do *Elbow Method* (*KElbowVisualizer*).
* **Trabalho Conjunto**:
  * Definição do escopo, Tratamento/Engenharia de Dados (EDA) e formulação dos Testes de Hipóteses (vantagem do mandante / *Home Advantage*).

---

## 🎯 Principais Objetivos e Metodologia

```
├── 1. Coleta & Limpeza (EDA) ─────> Tratamento de nulos (ex: imputação de Cabo Verde) e tipos
├── 2. Teste de Hipóteses ─────────> Validação do "Fator Mandante" (t-Student pareado e independente)
├── 3. Aprendizado Supervisionado ─> Random Forest + Simulação de Monte Carlo (Copa 2026)
├── 4. Aprendizado Não Sup. ───────> Clusterização K-Means (Perfis das Seleções de 2022)
└── 5. Regressão Linear ───────────> Correlação entre Ranking FIFA e Gols Marcados
```

---

## 🔬 Etapas do Projeto em Detalhes

### 1. 📊 Análise Exploratória e Engenharia de Dados (EDA)
- **Dataset**: Dados históricos de partidas internacionais e atributos temporais (`dataset_temporal_final.csv`), incluindo ranking FIFA, pontuação acumulada, atributos técnicos médios (`GER`, `ATQ`, `MED`, `DEF`), métricas de momento/forma recente (`form_pts`, `form_gd`) e histórico de confrontos diretos (`h2h_pts`).
- **Pré-processamento**:
  - Conversão de tipos de datas e variáveis temporais.
  - Tratamento de registros incompletos e imputação de valores faltantes no ranking histórico de federações menores (ex: imputação pela mediana para seleções como Cabo Verde).
  - Remoção de confrontos sem relevância direta para a amostragem das equipes cotadas para 2026.

### 2. 🧪 Teste de Hipóteses: Validação do Fator Mandante (*Home Advantage*)
- **Pergunta**: Jogar em casa oferece uma vantagem estatisticamente significativa na média de gols marcados em comparação a jogar fora ou em campo neutro?
- **Metodologia**:
  - Aplicação do Teste t de Student para amostras independentes (`scipy.stats.ttest_ind`) comparando gols de mandantes versus jogos em campo neutro.
  - Teste t pareado (`scipy.stats.ttest_rel`) entre gols marcados em casa e fora.
- **Resultado**:
  - Rejeição da Hipótese Nula ($p < 0.05$); evidência estatística de que seleções jogando em casa possuem média de gols significativamente superior, validando o uso do indicador de neutralidade/mando como *feature* preditiva relevante.

---

### 3. 🤖 Aprendizado de Máquina Supervisionado & Monte Carlo (Responsável: Newton)
Em vez de estimar um resultado puramente determinístico, a abordagem migrou para uma **modelagem probabilística com Simulação de Monte Carlo**:

1. **Classificador de Confrontos (`RandomForestClassifier`)**:
   - **Objetivo**: Estimar a probabilidade de vitória de uma seleção frente a outra com base na assimetria de forças (diferença de Ranking FIFA, *Overall*, Ataque, Meio-campo, Defesa, pontuação recente e indicador de campo neutro).
   - **Desempenho**:
     - **Acurácia**: **79,47%** no conjunto de teste.
     - **Recall**: 72% para a classe 0 (visitante vence/empate em mata-mata) e 84% para a classe 1 (mandante/equipe A vence).
2. **Motor de Simulação de Torneio (Monte Carlo)**:
   - Configuração de **12 grupos com 4 seleções** (modelo expandido da Copa de 2026 com 48 participantes).
   - Simulação estocástica jogo a jogo: cada partida é resolvida baseando-se na distribuição de probabilidade gerada pelo classificador treinado.
   - Resolução da fase de grupos, classificação de 1º, 2º e os 8 melhores 3º colocados, avançando para o chaveamento de mata-mata (*Round of 32* até a Final).
   - Execução de **1.000 iterações completas** do torneio para mapear as chances reais de título e vice-campeonato.
3. **Favoritos Mapeados no Top 10 (Simulação)**:
   - 🏴󠁧󠁢󠁥󠁮󠁧󠁿 **Espanha**: ~14,3% de probabilidade de título
   - 🇪🇸 **Inglaterra**: ~13,3%
   - 🇫🇷 **Argentina**: ~12,0%
   - 🇦🇷 **França**: ~9,2%
   - 🇳🇱 **Holanda**: ~8,8%
   - 🇵🇹 **Portugal**: ~8,7%
   - 🇧🇷 **Brasil**: ~7,6%
   - 🇩🇪 **Alemanha**: ~6,5%
   - 🇧🇪 **Suíça**: ~4,5%
   - 🇨🇭 **Bélgica**: ~3,0%

---

### 4. 📈 Regressão Linear (Responsável: Newton)
- **Pergunta**: O quão bem o Ranking FIFA médio de uma seleção explica a quantidade de gols marcados ao longo da sua participação em Copas do Mundo?
- **Modelagem**:
  - Reestruturação da base em nível de "Equipe por Jogo" e agregação por total de gols no torneio.
  - Ajuste de modelo de Regressão Linear Simples via `LinearRegression` do *scikit-learn*.
- **Resultados**:
  - **Coeficiente Angular ($\beta$)**: **-0.1245** (confirma a tendência esperada: seleções com números de ranking menores/melhores tendem a marcar mais gols).
  - **Coeficiente de Determinação ($R^2$)**: **0.2370** (o ranking explica cerca de 23,7% da variabilidade dos gols marcados, evidenciando a alta variabilidade, imprevisibilidade e formato de tiro curto inerentes à Copa do Mundo).

---

### 5. 🔍 Aprendizado Não Supervisionado: Agrupamento K-Means (Responsável: Davi)
- **Objetivo**: Identificar agrupamentos e padrões de comportamento tático/técnico entre as seleções participantes da Copa de 2022.
- **Técnicas**:
  - Padronização de dados numéricos com `StandardScaler`.
  - Escolha do hiperparâmetro ótimo de clusters através do método do cotovelo (`KElbowVisualizer`).
  - Aplicação do algoritmo `KMeans` ($k = 3$).
- **Interpretação dos Grupos**:
  - **Cluster 0 (Desempenho Frustrado)**: Equipes com bom retrospecto pré-torneio (`form_points` e `form_gd` altos) e ranking razoável, mas que performaram abaixo do esperado no mundial.
  - **Cluster 1 (Seleções Modestas/Desafiantes)**: Métricas ofensivas e de saldo inferiores à média geral, com menor competitividade e rankings mais distantes do topo.
  - **Cluster 2 (Elite / Alto Desempenho)**: Equipes com métricas técnicas superiores em todos os setores (ataque, defesa e volume de gols), correspondendo às principais forças do torneio.

---

## 📁 Estrutura de Arquivos

```bash
├── .gitignore
├── README.md               # Documentação detalhada do projeto
├── seminarioWC26.ipynb     # Notebook Jupyter com todas as etapas, códigos e gráficos
└── seminario_CD_WC26.pdf   # Slides visuais apresentados no seminário acadêmico
```

---

## 🛠️ Tecnologias e Bibliotecas Utilizadas

- **Linguagem**: Python (3.10+)
- **Manipulação de Dados**: `pandas`, `numpy`
- **Visualização**: `matplotlib`, `seaborn`, `yellowbrick`
- **Modelagem & Machine Learning**: `scikit-learn` (`RandomForestClassifier`, `LinearRegression`, `KMeans`, `StandardScaler`)
- **Estatística**: `scipy` (`scipy.stats`)

---
