# Transição Energética Brasileira — Análise de Dados

> *"Vi uma notícia dizendo que os carros elétricos estavam explodindo no Brasil. Em vez de só compartilhar, resolvi verificar nos dados — 3 fontes oficiais, 4 milhões de registros e 5 notebooks depois, aqui está o que encontrei."*



---

## Pergunta central

**O Brasil está realmente em transição energética — ou é só narrativa?**

Para responder, cruzei três séries históricas de dados públicos oficiais:
- Crescimento da frota de veículos elétricos (SENATRAN)
- Expansão da energia solar instalada (ANEEL)
- Evolução das vendas de combustíveis fósseis (ANP)

---

## Principais achados

| Indicador | Jan/2021 | Dez/2025 | Variação |
|---|---|---|---|
| Frota elétrica | 55.631 veículos | 836.151 veículos | **+1.403%** |
| Energia solar (GD) | 5,3 GW | 44,8 GW | **+743%** |
| Vendas de combustíveis | 11,2 M m³/mês | 14,0 M m³/mês | **+25%** |

**Insight não óbvio:** A capacidade solar instalada já gera energia suficiente para cobrir **4.695x** a demanda estimada de toda a frota elétrica atual. O gargalo da transição não é energético — é o preço dos veículos e a infraestrutura de recarga.

**Previsões para 2030 (Prophet):**
- Frota elétrica: ~2,4 milhões de veículos (+193%)
- Energia solar: ~89 GW (+99%)
- Combustíveis: +11% — desaceleração clara, ponto de inflexão provavelmente pós-2030

---

## Arquitetura do projeto

```
Fontes públicas → Python (coleta + tratamento) → BigQuery → Power BI Mobile
```

```
portfolio-dados/
└── transicao-energetica/
    ├── notebooks/
    │   ├── 01_coleta_senatran.ipynb    ← Web scraping + 60 arquivos Excel
    │   ├── 02_coleta_aneel.ipynb       ← 4,1 milhões de instalações solares
    │   ├── 03_coleta_anp.ipynb         ← Série histórica de combustíveis
    │   ├── 04_integracao.ipynb         ← Cruzamento das três fontes
    │   └── 05_forecasting.ipynb        ← Prophet em três séries temporais
    ├── data/
    │   └── raw/                        ← Dados brutos — nunca modificados
    ├── output/                         ← Datasets tratados e forecasts
    └── dashboard/                      ← Power BI (.pbix)
```

---

## Fontes de dados

| Fonte | Descrição | Acesso | Atualização |
|---|---|---|---|
| **SENATRAN** | Frota por tipo de combustível, UF e município | Portal gov.br (Excel mensal) | Mensal |
| **ANEEL/SIGA** | Usinas solares centralizadas em operação | Portal dados abertos ANEEL (CSV) | Mensal |
| **ANEEL MMGD** | Micro e minigeração distribuída (telhados) | Portal dados abertos ANEEL (CSV) | Mensal |
| **ANP** | Vendas de combustíveis pelas distribuidoras | Portal gov.br (Excel) | Mensal |

---

## Pipeline de coleta

A coleta foi inteiramente automatizada via Python — sem downloads manuais.

**SENATRAN:** Web scraping com `BeautifulSoup` para identificar os 60 arquivos Excel mensais (2021–2025), download automático, filtragem por tipo de combustível elétrico, tratamento de outlier em Nov/2021 por interpolação.

**ANEEL:** Download direto via URL do CSV de 4,1 milhões de registros de instalações solares, conversão de potência de string para float, agregação por mês e estado.

**ANP:** Web scraping da página de dados estatísticos, identificação do arquivo Excel com cabeçalho de múltiplas linhas, tratamento para transformar formato wide → long.

---

## Stack técnica

| Ferramenta | Uso |
|---|---|
| `Python 3.13` | Pipeline de coleta e tratamento |
| `pandas` | Manipulação e análise de dados |
| `requests` + `BeautifulSoup` | Web scraping |
| `Prophet` | Forecasting de séries temporais |
| `matplotlib` + `seaborn` | Visualização exploratória |
| `Google BigQuery` | Armazenamento em nuvem |
| `Power BI` | Dashboard final (mobile first) |

---

## Como reproduzir

### Pré-requisitos

```bash
pip install pandas requests beautifulsoup4 prophet matplotlib seaborn openpyxl basedosdados
```

### Executar os notebooks em ordem

```bash
jupyter lab
```

Execute os notebooks de `01` a `05` em sequência. Os dados brutos serão baixados automaticamente na primeira execução.

> **Nota:** A coleta da ANEEL (notebook 02) pode demorar 5–10 minutos devido ao tamanho do arquivo de geração distribuída (~4 milhões de registros).

### Escalabilidade

A arquitetura suporta atualização automática mensal — basta reexecutar os notebooks de coleta. O pipeline detecta novos arquivos publicados pelo governo automaticamente via web scraping.

---

## Decisões metodológicas

**Por que usar geração distribuída separada do SIGA?**
O SIGA centralizado cobre ~21 GW (usinas grandes). A MMGD cobre ~45 GW (telhados residenciais e comerciais). A soma de ~67 GW é consistente com os 64 GW reportados pela ABSOLAR, considerando defasagem de atualização.

**Por que o `changepoint_prior_scale` diferente para EVs?**
A frota elétrica teve aceleração recente mais intensa — o parâmetro `0.1` (vs `0.05` para as outras séries) permite que o modelo capture melhor mudanças de tendência sem overfitting.

**Por que combustíveis ainda crescem?**
A frota elétrica representa menos de 1% da frota total brasileira em 2025. O impacto na demanda de combustíveis ainda é estatisticamente insignificante. O ponto de inflexão provavelmente ocorre quando a frota elétrica ultrapassar 5–10% da frota total.

---

## Autor

**Otávio Domingues**
Analista de Dados | Power BI | DAX | Python

[LinkedIn](https://linkedin.com/in/otaviodomingues) · [GitHub](https://github.com/otaviodomingues)
