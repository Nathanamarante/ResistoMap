# 🧬 ResistoMap

**Mapeamento de genes de resistência a antibióticos em *Salmonella enterica* e *Salmonella bongori***

![Python](https://img.shields.io/badge/python-3.10%2B-blue)
![License](https://img.shields.io/badge/license-MIT-green)
![Status](https://img.shields.io/badge/status-em%20desenvolvimento-yellow)

Projeto final da disciplina **Ambientes de Computação** (Ciência da Computação e Bioinformática).

---

## 📌 Sumário

- [Contexto](#-contexto)
- [Objetivos](#-objetivos)
- [Perguntas de pesquisa](#-perguntas-de-pesquisa)
- [Dados](#-dados)
- [Metodologia](#-metodologia)
- [Estrutura do repositório](#-estrutura-do-repositório)
- [Instalação](#-instalação)
- [Como executar](#-como-executar)
- [Resultados esperados](#-resultados-esperados)
- [Limitações](#-limitações)
- [Equipe](#-equipe)
- [Referências](#-referências)
- [Licença](#-licença)

---

## 🦠 Contexto

A resistência antimicrobiana (AMR) é um dos maiores desafios de saúde pública da atualidade. O gênero *Salmonella* é uma das principais causas de doenças transmitidas por alimentos no mundo, e linhagens multirresistentes dificultam o tratamento de infecções.

O gênero possui apenas duas espécies:

- ***Salmonella enterica***: abrange centenas de sorovares (como Typhimurium, Enteritidis e Typhi), amplamente associados a humanos, animais de produção e alimentos.
- ***Salmonella bongori***: encontrada principalmente em animais de sangue frio e no ambiente, com muito menos genomas disponíveis.

Comparar o repertório de genes de resistência (o *resistoma*) das duas espécies ajuda a entender como a resistência se distribui e evolui dentro do gênero.

## 🎯 Objetivos

**Objetivo geral:** desenvolver um pipeline em Python que identifique, quantifique e visualize genes de resistência a antibióticos em genomas de *S. enterica* e *S. bongori*.

**Objetivos específicos:**

1. Obter e organizar genomas públicos e seus metadados.
2. Detectar genes de resistência com ferramentas de referência (AMRFinderPlus) e, opcionalmente, com uma busca própria por similaridade contra o banco CARD.
3. Construir uma matriz de presença/ausência genomas × genes.
4. Comparar a prevalência de genes e classes de antibióticos entre as duas espécies.
5. Analisar coocorrência de genes, perfis de multirresistência (MDR) e distribuição por país, ano e fonte do isolado.
6. Gerar tabelas, gráficos e um relatório reproduzível.

## ❓ Perguntas de pesquisa

- Quais são os genes de resistência mais prevalentes em cada espécie?
- Quais classes de antibióticos concentram mais genes de resistência?
- Há diferenças significativas entre *S. enterica* e *S. bongori*?
- Quais genes tendem a aparecer juntos (coocorrência)?
- Que proporção dos genomas é multirresistente (resistência a 3 ou mais classes)?
- A prevalência varia por sorovar, país, ano ou fonte (humana, animal, alimento, ambiente)?

## 🗄️ Dados

| Fonte | Uso |
|---|---|
| [NCBI Datasets](https://www.ncbi.nlm.nih.gov/datasets/) | Download de genomas (RefSeq) e metadados |
| [NCBI Pathogen Detection](https://www.ncbi.nlm.nih.gov/pathogens/) | Metadados de isolados (país, ano, fonte, sorovar) e genes AMR pré-anotados |
| [CARD](https://card.mcmaster.ca/) | Banco de referência de genes de resistência e ontologia (ARO) |
| [NCBI AMRFinderPlus](https://github.com/ncbi/amr) | Ferramenta e banco para detecção de genes e mutações de AMR |

> **Atenção ao desbalanceamento:** há ordens de grandeza mais genomas de *S. enterica* do que de *S. bongori*. O projeto usa **amostragem estratificada** de *S. enterica* e **todos os genomas de qualidade** disponíveis de *S. bongori* para permitir comparações mais justas.

Os dados brutos **não são versionados** no repositório (ver `.gitignore`). Os scripts de download garantem a reprodutibilidade, e a lista de acessos (`data/metadata/genomes_selected.csv`) documenta exatamente quais genomas foram usados.

## 🔬 Metodologia

```
Coleta de dados ─▶ Controle de qualidade ─▶ Detecção de genes AMR
        │                                          │
        ▼                                          ▼
   Metadados                           Matriz presença/ausência
                                                   │
                  ┌────────────────────────────────┼────────────────────────┐
                  ▼                                ▼                        ▼
         Prevalência e testes            Coocorrência e redes      Perfis MDR e clustering
                  └────────────────────────────────┼────────────────────────┘
                                                   ▼
                                    Figuras, tabelas e relatório
```

### 1. Coleta
Download dos genomas e metadados das duas espécies via NCBI Datasets.

### 2. Controle de qualidade
Filtros por nível de assembly, número de contigs, tamanho do genoma (~4,5 a 5,5 Mb para o gênero) e metadados essenciais. Remoção de duplicatas.

### 3. Detecção de genes de resistência
- **Abordagem principal:** AMRFinderPlus com a opção de organismo `Salmonella`, que também detecta mutações pontuais relevantes (ex.: em *gyrA*).
- **Abordagem complementar (algorítmica):** busca por similaridade (BLAST local) das sequências do CARD contra os genomas, com filtros de identidade e cobertura mínimas configuráveis, e comparação dos resultados com a ferramenta de referência.

### 4. Matriz presença/ausência
Tabela binária genomas × genes, base de todas as análises seguintes.

### 5. Análises
- Prevalência por gene, classe e espécie.
- Testes estatísticos (qui-quadrado / Fisher exato, com correção para múltiplos testes).
- Coocorrência (índice de Jaccard) e rede de genes (NetworkX).
- Agrupamento hierárquico e redução de dimensionalidade (PCA/UMAP) dos perfis de resistência.
- Identificação de genomas MDR.

### 6. Visualização
Gráficos de barras de prevalência, heatmap de presença/ausência, rede de coocorrência, dendrograma e séries temporais/mapas por país.

## 📁 Estrutura do repositório

```
resistomap/
├── README.md
├── LICENSE
├── environment.yml            # ambiente conda (ferramentas + bibliotecas)
├── requirements.txt           # dependências Python
├── config.yaml                # parâmetros (espécies, filtros, limiares)
├── data/
│   ├── raw/                   # genomas baixados (não versionado)
│   ├── metadata/              # metadados e lista de genomas selecionados
│   └── processed/             # matriz presença/ausência, tabelas intermediárias
├── notebooks/
│   ├── 01_exploracao_metadados.ipynb
│   ├── 02_prevalencia_e_estatistica.ipynb
│   ├── 03_coocorrencia_e_redes.ipynb
│   └── 04_clustering_e_mdr.ipynb
├── scripts/
│   ├── download_genomes.py    # coleta via NCBI Datasets
│   ├── run_amrfinder.sh       # execução em lote do AMRFinderPlus
│   └── run_pipeline.py        # executa o pipeline completo
├── src/resistomap/
│   ├── __init__.py
│   ├── quality.py             # filtros de qualidade
│   ├── detection.py           # leitura/parsing das saídas e busca própria (BLAST)
│   ├── matrix.py              # construção da matriz presença/ausência
│   ├── stats.py               # testes estatísticos
│   ├── network.py             # coocorrência e redes
│   └── plots.py               # funções de visualização
├── results/
│   ├── figures/
│   └── tables/
├── tests/
└── docs/
    └── apresentacao.pdf       # slides do seminário
```

## ⚙️ Instalação

### Pré-requisitos

- Python 3.10+
- [Conda/Mamba](https://docs.conda.io/) (recomendado, pois instala as ferramentas de linha de comando)
- Git

### Passo a passo

```bash
# 1. Clonar o repositório
git clone https://github.com/<seu-usuario>/resistomap.git
cd resistomap

# 2. Criar o ambiente (inclui AMRFinderPlus, BLAST+ e NCBI Datasets)
conda env create -f environment.yml
conda activate resistomap

# 3. Atualizar o banco do AMRFinderPlus
amrfinder -u
```

Alternativa apenas com pip (as ferramentas de linha de comando precisam ser instaladas à parte):

```bash
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

### Principais dependências

| Pacote | Uso |
|---|---|
| `biopython` | Leitura de FASTA/GenBank e utilitários de sequência |
| `pandas`, `numpy` | Manipulação de dados |
| `scipy`, `statsmodels` | Testes estatísticos e correção de p-valores |
| `matplotlib`, `seaborn` | Visualização |
| `networkx` | Redes de coocorrência |
| `scikit-learn`, `umap-learn` | Clustering e redução de dimensionalidade |
| `pyyaml` | Leitura do arquivo de configuração |

## ▶️ Como executar

Ajuste os parâmetros em `config.yaml` (número de genomas, filtros de qualidade, limiares de identidade e cobertura) e execute:

```bash
# Pipeline completo
python scripts/run_pipeline.py --config config.yaml

# Ou etapa por etapa
python scripts/download_genomes.py --config config.yaml
bash scripts/run_amrfinder.sh data/raw results/tables/amrfinder
python -m resistomap.matrix --input results/tables/amrfinder --output data/processed/matriz.csv
```

Para explorar as análises de forma interativa:

```bash
jupyter lab notebooks/
```

> 💡 **Dica:** para testar rapidamente, defina em `config.yaml` um valor pequeno para `max_genomes` (por exemplo, 50).

## 📊 Resultados esperados

Os resultados ficam em `results/`:

- `tables/prevalencia_genes.csv`: prevalência de cada gene por espécie.
- `tables/prevalencia_classes.csv`: prevalência por classe de antibiótico.
- `tables/genomas_mdr.csv`: genomas multirresistentes.
- `figures/`: heatmap de presença/ausência, barras de prevalência, rede de coocorrência, dendrograma e distribuições temporais/geográficas.

*(Adicione aqui as principais figuras após executar o pipeline.)*

```markdown
![Prevalência de genes](results/figures/prevalencia_genes.png)
```

## ⚠️ Limitações

- **Viés de amostragem:** isolados clínicos e de surtos estão super-representados nas bases públicas.
- **Desbalanceamento entre espécies:** o número de genomas de *S. bongori* é muito menor.
- **Genótipo ≠ fenótipo:** a presença de um gene não garante expressão nem resistência clínica.
- **Qualidade dos assemblies:** genes podem ser fragmentados ou perdidos em genomas incompletos.
- **Genes em plasmídeos:** assemblies em rascunho podem perder ou fragmentar elementos móveis.
- **Dependência de versão do banco:** os resultados variam conforme a versão do AMRFinderPlus/CARD (a versão usada é registrada em `results/`).

## 👥 Equipe

| Nome | Função | GitHub |
|---|---|---|
| _Nome 1_ | Coleta e curadoria dos dados | [@usuario1](https://github.com/usuario1) |
| _Nome 2_ | Detecção de genes e matriz | [@usuario2](https://github.com/usuario2) |
| _Nome 3_ | Análise estatística | [@usuario3](https://github.com/usuario3) |
| _Nome 4_ | Visualização | [@usuario4](https://github.com/usuario4) |
| _Nome 5_ | Interpretação e apresentação | [@usuario5](https://github.com/usuario5) |

**Disciplina:** Ambientes de Computação
**Instituição:** _preencher_
**Professor(a):** _preencher_

## 📚 Referências

- Feldgarden M. et al. *AMRFinderPlus and the Reference Gene Catalog facilitate examination of the genomic links among antimicrobial resistance, stress response, and virulence.* Scientific Reports, 2021.
- Alcock B. P. et al. *CARD 2023: expanded curation, support for machine learning, and resistome prediction at the Comprehensive Antibiotic Resistance Database.* Nucleic Acids Research, 2023.
- Cock P. J. A. et al. *Biopython: freely available Python tools for computational molecular biology and bioinformatics.* Bioinformatics, 2009.
- World Health Organization. *Global Antimicrobial Resistance and Use Surveillance System (GLASS).*

## 📄 Licença

Distribuído sob a licença MIT. Consulte o arquivo [LICENSE](LICENSE) para mais informações.

---

<p align="center">Feito com 🧬 e 🐍 para a disciplina Ambientes de Computação</p>
