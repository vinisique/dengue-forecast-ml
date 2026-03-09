# 🦟 Previsão de Surtos de Dengue — Modelo Híbrido MLP + AdaBoost

> Trabalho de Conclusão de Curso — Tecnologia em Banco de Dados  
> Faculdade Impacta de Tecnologia (FIT) — 2025

[![Python](https://img.shields.io/badge/Python-3.10+-blue?style=flat&logo=python)](https://www.python.org/)
[![Scikit-learn](https://img.shields.io/badge/Scikit--learn-1.x-orange?style=flat&logo=scikit-learn)](https://scikit-learn.org/)
[![Status](https://img.shields.io/badge/Status-Concluído-brightgreen?style=flat)]()
[![License](https://img.shields.io/badge/License-MIT-lightgrey?style=flat)](LICENSE)

---

## 📌 Sobre o Projeto

Este projeto desenvolve um **sistema preditivo de 3 fases** para antecipar surtos de dengue no município de São Paulo, cruzando dados epidemiológicos semanais do SINAN/DATASUS com variáveis climáticas históricas do INMET.

O modelo foi treinado com dados de 2023 e testado integralmente em 2024 — o ano com o maior surto de dengue já registrado no estado de São Paulo, com **+556% de casos** em relação ao ano anterior e taxa de incidência de **4.778 casos por 100 mil habitantes**.

### Problema
Surtos de dengue em áreas urbanas exigem resposta rápida de saúde pública. Modelos preditivos capazes de antecipar picos epidêmicos com semanas de antecedência podem salvar vidas ao viabilizar ações preventivas antes que o sistema de saúde seja sobrecarregado.

### Solução
Um pipeline híbrido e adaptativo que detecta automaticamente o contexto epidemiológico (normal vs. surto) e aciona o modelo mais adequado para cada fase — com validação científica rigorosa contra data leakage e overfitting.

---

## 🏗️ Arquitetura do Pipeline — 3 Fases

```
Dados Semanais (SINAN + INMET)
         │
         ▼
┌─────────────────────┐
│  FASE 1: OBSERVAÇÃO │  ← Limiares de alerta + % semanas elevadas
└─────────┬───────────┘
          │ Alerta confirmado?
          ▼
┌──────────────────────────┐
│  FASE 2: CONFIRMAÇÃO     │  ← Magnitude vs. baseline + semanas adicionais
└─────────┬────────────────┘
          │
     ┌────┴────┐
     ▼         ▼
┌─────────┐ ┌──────────┐
│  SURTO  │ │  NORMAL  │
│AdaBoost │ │   MLP    │  ← Modelos especializados por contexto
│DT n=100 │ │ (100,50) │
└─────────┘ └──────────┘
          │
          ▼
   Previsão Semanal
   + Intervalo de Confiança 95%
```

### Por que dois modelos?
- **AdaBoostRegressor** (DecisionTree, n=100): melhor captura de padrões não-lineares e extremos característicos de surtos
- **MLPRegressor** (camadas 100→50, ReLU, Adam, early stopping): mais estável para períodos de baixa variância onde o comportamento é mais regular

---

## 📊 Resultados

| Métrica | Valor |
|---|---|
| **R² Global** | **0,93** |
| **MAE** | **~2.185 casos/semana** |
| **RMSE** | Validado |
| **Cobertura IC 95%** | **95%** |
| **Falsos Positivos de Surto** | **0** |
| **Semanas de teste (2024)** | **40** |
| **Folds de cross-validation** | **5 (TimeSeriesSplit)** |
| **Data Leakage** | **PASS ✅** |

### Contexto do ano de teste (2024)
- Maior surto de dengue já registrado em São Paulo
- Pico de **145.382 casos em uma única semana**
- Taxa de incidência: **4.778/100 mil hab.** (vs. 728/100 mil em 2023)
- O modelo detectou o início do surto e ativou automaticamente o AdaBoost para o período crítico

---

## 🔧 Stack Técnica

| Categoria | Tecnologias |
|---|---|
| Linguagem | Python 3.10+ |
| Machine Learning | Scikit-learn (AdaBoostRegressor, MLPRegressor, DecisionTreeRegressor) |
| Validação | TimeSeriesSplit, cross-validation temporal, análise de resíduos, Q-Q Plot |
| Manipulação de Dados | Pandas, NumPy |
| Visualização | Matplotlib, Seaborn |
| Estatística | SciPy |

---

## 🗂️ Estrutura do Repositório

```
dengue-forecast-ml/
│
├── Pipeline_3fases_ModeloHibrido_MLP_AdaBoost.ipynb   # Notebook principal
├── Saude_Publica_Relatorio_1.pdf                       # Relatório CRISP-DM (Etapas 1 e 2)
├── README.md
└── LICENSE
```

> ⚠️ **As bases de dados não estão incluídas** no repositório por conta do tamanho dos arquivos. Veja a seção abaixo para baixá-las.

---

## 📥 Como Reproduzir

### 1. Clone o repositório
```bash
git clone https://github.com/vinisique/dengue-forecast-ml.git
cd dengue-forecast-ml
```

### 2. Instale as dependências
```bash
pip install pandas numpy matplotlib seaborn scikit-learn scipy
```

### 3. Baixe as bases de dados

**Base Epidemiológica — SINAN/DATASUS**
- Acesse: https://opendatasus.saude.gov.br/dataset/arboviroses-dengue
- Baixe os dados de dengue para o estado de São Paulo (anos 2020–2024)
- Salve como `df_SP.csv` no caminho configurado no notebook

**Base Climática — INMET**
- Acesse: https://portal.inmet.gov.br/dadoshistoricos
- Baixe os dados históricos da estação **São Paulo — Mirante**
- Consolide em um único arquivo e salve como `base_consolidada_inmet.csv`

### 4. Configure os caminhos
No início do notebook, ajuste as variáveis:
```python
INPUT_CSV   = 'caminho/para/df_SP.csv'
INPUT_CLIMA = 'caminho/para/base_consolidada_inmet.csv'
```

### 5. Execute
Abra o notebook no Jupyter ou Google Colab e execute todas as células em ordem.

---

## 🧪 Validações Científicas Implementadas

O pipeline inclui 6 camadas de validação para garantir a confiabilidade dos resultados:

1. **Data Leakage Check** — verificação de overlap temporal entre treino (2023) e teste (2024)
2. **Análise de Overfitting** — comparação de performance entre primeira e segunda metade do período de teste
3. **Cross-Validation Temporal** — 5-fold com `TimeSeriesSplit` (respeita a ordem cronológica dos dados)
4. **Análise de Resíduos** — verificação de média próxima de zero e ausência de autocorrelação
5. **Intervalos de Confiança** — cobertura verificada em 95% das previsões
6. **Análise por Contexto** — métricas separadas para períodos de surto vs. períodos normais

---

## 📈 Feature Engineering

| Feature | Descrição |
|---|---|
| `lag_1` a `lag_4` | Casos nas 1 a 4 semanas anteriores |
| `ma_3` | Média móvel das últimas 3 semanas (shift 1) |
| `semana_do_ano` | Semana epidemiológica (sazonalidade) |
| `ano_val` | Ano — captura tendência de longo prazo |
| `precipitacao` | Precipitação semanal acumulada (mm) — INMET |
| `temperatura` | Temperatura média semanal (°C) — INMET |
| `umidade` | Umidade relativa média semanal (%) — INMET |

---

## 🌍 Contexto Epidemiológico

A dengue é a arbovirose de maior impacto no Brasil, com ciclos epidêmicos fortemente associados a condições climáticas (chuvas e temperatura) que favorecem a reprodução do *Aedes aegypti*. O estado de São Paulo concentra a maior carga de casos da região Sudeste.

Os dados utilizados cobrem o período de **2020 a 2024**, totalizando mais de **3,1 milhões de casos confirmados** no estado, com o ano de 2024 respondendo sozinho por 2,1 milhões — cenário que torna este um dos testes mais desafiadores possíveis para qualquer modelo preditivo epidemiológico.

**Fontes:**
- SINAN — Sistema de Informação de Agravos de Notificação (Ministério da Saúde)
- INMET — Instituto Nacional de Meteorologia
- DATASUS — Departamento de Informática do SUS

---

## 👨‍💻 Autor

**Vinicius Alves Siqueira**  
Tecnólogo em Banco de Dados — Faculdade Impacta de Tecnologia  
[![LinkedIn](https://img.shields.io/badge/LinkedIn-vinicius--siqueira1-blue?style=flat&logo=linkedin)](https://www.linkedin.com/in/vinicius-siqueira1/)
[![GitHub](https://img.shields.io/badge/GitHub-vinisique-black?style=flat&logo=github)](https://github.com/vinisique)

---

## 📄 Licença

Este projeto está sob a licença MIT. Veja o arquivo [LICENSE](LICENSE) para mais detalhes.
