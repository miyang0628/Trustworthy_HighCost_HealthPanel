# SAFE-Agentic Health Panel

A reproducible, auditable **agentic pipeline for trustworthy prediction of next-year
high medical expenditure** on longitudinal health-panel data. The framework extends the
cross-sectional *S.A.F.E agentic* credit-scoring paradigm to a **person-year panel**
setting: features observed in year *t* are used to predict whether an individual falls in
the **top decile of total medical expenditure in year *t+1***, under a **prospective
time split**. Each stage emits a reusable artifact (datacard, model, evaluation report,
system card), so the full trustworthiness assessment is transparent and reproducible.

> **Anonymized for peer review.** Author(s) and affiliation(s) are withheld.
> Manuscript under review at an anonymous venue. Do not de-anonymize in issues or commits.

## Framework

A sequential five-agent pipeline, mirrored by the notebooks in `notebooks/`:

| Agent | Notebook | Output artifact |
|-------|----------|-----------------|
| **Data** | `00_data_preparation.ipynb` | person-year panel, `datacard.json`, `attrition.csv` |
| **(EDA)** | `01_eda.ipynb` | grayscale figures, `eda_summary.csv` |
| **Modelling** | `02_modelling.ipynb` | six candidates, 5-fold CV selection, fitted pipeline |
| **Evaluation** | `03_evaluation.ipynb` | RGA/AURGA, RGR, AURGE, fairness, calibration |
| **Governance** | `04_governance.ipynb` | `system_card.json`, SAFE Score, weight sensitivity |
| **Chatbot** | `05_chatbot_agent.ipynb` | artifact-grounded reviewer QA (LLM) |

Robustness and sensitivity appendices (non-LLM):

| Appendix | Notebook | What it adds |
|----------|----------|--------------|
| GroupKFold / death | `06_robustness_groupkfold.ipynb` | person-level `GroupKFold` vs the time split; death competing-risk sensitivity |
| Explainability | `07_explainability.ipynb` | grouped RGE over feature blocks; parsimonious top-k vs full AURGE |
| Recalibration | `08_recalibration.ipynb` | Platt & isotonic calibration; illustrative SAFE_5 with a calibration axis |
| Continuous target | `09_continuous_cost.ipynb` | log-expenditure regression; RGA corroborates the binary ranking |

### Trustworthiness metrics

The **SAFE Score** is the equally weighted mean (0.25 each) of four rank-graduation
components, all in [0, 1]: **AURGA** (accuracy), **RGR** (robustness), **AURGE**
(explainability) and a **Fairness Aggregate** (SPD, EOD, AOD, DIR normalized to parity).
Rank-graduation quantities use the authors' `safeaipackage`; for binary targets RGA
equals ROC-AUC. Probability calibration (Brier, calibration intercept/slope,
Cramér–von Mises) is reported but, following the original framework, is **not** folded
into the SAFE Score — recalibration is an explicit extension point.

### Sensitive attributes

Fairness is evaluated across **Medical-Aid status, disability, sex, age ≥ 65, and low
income**. Because the target is cost, the pipeline foregrounds the *cost-as-need-proxy*
risk (expenditure can understate the need of low-income / Medical-Aid populations); the
governance stage flags these subgroups for audit.

## Repository layout

```
.
├── data/
│   ├── raw/          # KHP SAS files (NOT distributed; see Data access)
│   └── processed/    # generated panel + model artifacts (git-ignored)
├── notebooks/        # 00–05, one code cell each
├── results/
│   ├── figures/      # grayscale PNG + PDF, 600 dpi
│   └── tables/       # datacard, metrics, system card (JSON/CSV)
├── src/              # notebook generators
├── .env.example      # copy to .env at repo root (LLM agent only)
├── requirements.txt
└── README.md
```

## Data access

The analysis uses the **Korea Health Panel Study (KHP), 2nd wave (2019–2024)**. The
microdata are **not** redistributed in this repository and must be obtained from the
official provider under its data-use agreement. After obtaining the data, place the
extracted SAS files so that each year resolves as:

```
data/raw/sas2019/a_ind.sas7bdat , a_ms.sas7bdat , a_phi.sas7bdat , ...
data/raw/sas2020/b_ind.sas7bdat , ...
...
data/raw/sas2024/f_ind.sas7bdat , ...
```

## Reproducibility

```bash
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt

# Notebooks 00–04 need no API key:
jupyter nbconvert --to notebook --execute --inplace notebooks/00_data_preparation.ipynb
jupyter nbconvert --to notebook --execute --inplace notebooks/01_eda.ipynb
jupyter nbconvert --to notebook --execute --inplace notebooks/02_modelling.ipynb
jupyter nbconvert --to notebook --execute --inplace notebooks/03_evaluation.ipynb
jupyter nbconvert --to notebook --execute --inplace notebooks/04_governance.ipynb

# Appendices 06–09 also need no API key (run after 02):
jupyter nbconvert --to notebook --execute --inplace notebooks/06_robustness_groupkfold.ipynb
jupyter nbconvert --to notebook --execute --inplace notebooks/07_explainability.ipynb
jupyter nbconvert --to notebook --execute --inplace notebooks/08_recalibration.ipynb
jupyter nbconvert --to notebook --execute --inplace notebooks/09_continuous_cost.ipynb
```

The **LLM chatbot agent (`05`)** is optional and the only component requiring API access.
Copy `.env.example` to `.env` at the repository root (one level above `notebooks/`) and
set `OPENAI_API_KEY` and `LLM_MODEL_*`.

Conventions: notebooks hold a single code cell with English comments; all randomness is
seeded (`seed = 42`); figures are grayscale, caption-free, and saved as PNG + PDF at
600 dpi; no absolute paths are printed.

## Design notes

- **Prospective split**: train on transitions 2019→2020 … 2022→2023; test on 2023→2024.
  A `GroupKFold`-by-person variant is documented to prevent person-level leakage.
- **Informative attrition**: dropout between *t* and *t+1* is reported by subgroup in the
  datacard; death during a transition censors cost and is flagged for sensitivity analysis.
- **Survey, not claims**: expenditures are self-reported panel values, not administrative
  claims — a stated limitation.

## License

Code released under the MIT License (see `LICENSE`). The KHP microdata are governed by
their own terms and are not covered by this license.

## Citation

Anonymous Author(s). *SAFE-Agentic Health Panel: an auditable agentic pipeline for
trustworthy next-year high-cost prediction.* Under review, 2026.
