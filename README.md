# Google Advanced Data Analytics — Portfólio

Projetos desenvolvidos ao longo do Certificado Profissional Google Advanced
Data Analytics (Coursera), com foco em análise exploratória, estatística
inferencial e modelagem preditiva em Python.

**Autor:** Lucas Barboza — Analista de Business Intelligence
**GitHub:** [@lfbarboza](https://github.com/lfbarboza)

## Projetos

| Curso | Projeto | Tema | Status |
|-------|---------|------|--------|
| 1 — Foundations of Data Science | Automatidata (NYC TLC) | Inspeção e estruturação de dados | Em andamento |

## Stack

Python 3.14 · pandas · NumPy · Matplotlib · Seaborn · statsmodels ·
scikit-learn · XGBoost · Jupyter

## Como reproduzir

```bash
git clone https://github.com/lfbarboza/google-advanced-data-analytics.git
cd google-advanced-data-analytics
python -m venv .venv
.venv\Scripts\Activate.ps1     # Windows
pip install -r requirements.txt
```

Os datasets não são versionados (ver `.gitignore`). Cada projeto documenta
no seu próprio README onde obter os dados e em qual pasta colocá-los.

## Organização

```
curso-01-foundations/
└── automatidata/
    ├── notebooks/    análise em Jupyter
    ├── docs/         documentos de projeto e resumos executivos
    └── data/raw/     dados brutos (não versionados)
```

Os projetos seguem o framework PACE (Plan, Analyze, Construct, Execute)
adotado no certificado; os documentos de estratégia de cada projeto estão
em `docs/`.