# TRIAGE: Trustworthy Rank-graduation Index for Auditable Governance Evaluation

A reproducible, auditable **agentic pipeline for trustworthy prediction of next-year high
medical expenditure** on longitudinal health-panel data. TRIAGE extends the cross-sectional
*S.A.F.E agentic* rank-graduation paradigm to a **person-year panel** setting: features
observed in year *t* predict whether an individual falls in the **top decile of total
medical expenditure in year *t+1***, under a **prospective time split**. Each stage emits a
reusable artifact (datacard, model, evaluation report, system card), so the full
trustworthiness assessment is transparent and reproducible.

> **Name.** The composite governance score in this project is the **TRIAGE Score** (four
> equally weighted rank-graduation dimensions). Some generated artifacts retain a `safe_`
> filename from an earlier revision (e.g. `results/tables/safe_components.json`); the
> contents are the TRIAGE components.

> **Anonymized for peer review.** Author(s) and affiliation(s) are withheld. Manuscript
> under review at an anonymous venue (Elsevier `elsarticle`, project
> `Trustworthy_HighCost_HealthPanel_ESWA`). Do not de-anonymize in issues or commits.

## Framework

A sequential five-stage pipeline (notebooks `00`–`05`) plus four robustness appendices
(`06`–`09`):

| Stage | Notebook | Output artifact |
|-------|----------|-----------------|
| **Data** | `00_data_preparation.ipynb` | person-year panel, `datacard.json`, `attrition.csv` |
| **(EDA)** | `01_eda.ipynb` | grayscale figures, `eda_summary.csv` |
| **Modelling** | `02_modelling.ipynb` | six candidates, 5-fold CV selection, fitted pipeline |
| **Evaluation** | `03_evaluation.ipynb` | RGA/AURGA, RGR, AURGE, fairness, calibration, `subgroup_outcomes.csv` |
| **Governance** | `04_governance.ipynb` | `system_card.json`, TRIAGE Score, weight sensitivity |
| **Chatbot** | `05_chatbot_agent.ipynb` | artifact-grounded reviewer QA (LLM) |

| Appendix | Notebook | What it adds |
|----------|----------|--------------|
| GroupKFold / death | `06_robustness_groupkfold.ipynb` | person-level `GroupKFold` vs the time split; death competing-risk sensitivity |
| Explainability | `07_explainability.ipynb` | grouped RGE over feature blocks; parsimonious top-k vs full AURGE |
| Recalibration | `08_recalibration.ipynb` | Platt & isotonic calibration; illustrative five-axis TRIAGE score |
| Continuous target | `09_continuous_cost.ipynb` | log-expenditure regression; RGA corroborates the binary ranking |

### Trustworthiness metrics

The **TRIAGE Score** is the equally weighted mean (0.25 each) of four rank-graduation
components, all in [0, 1]: **AURGA** (accuracy), **RGR** (robustness), **AURGE**
(explainability) and a **Fairness Aggregate** (SPD, EOD, AOD, DIR normalized to parity).
Rank-graduation quantities use the `safeaipackage` implementation of the RGA family; for
binary targets RGA equals ROC-AUC. Probability calibration (Brier, calibration
intercept/slope, Cramér–von Mises) is reported but **not** folded into the TRIAGE Score —
recalibration is an explicit extension axis (appendix `08`).

### Sensitive attributes

Fairness is evaluated across **Medical-Aid status, disability, sex, age ≥ 65, and low
income**. Because the target is cost, the pipeline foregrounds the *cost-as-need-proxy*
risk; the governance stage flags these subgroups for audit, and `subgroup_outcomes.csv`
separates the *observed* subgroup high-cost rate from the model's *predicted* positive rate.

## Headline results (held-out 2023→2024)

- Selected model **XGBoost** (5-fold CV ROC-AUC 0.799); test ROC-AUC 0.797.
- **TRIAGE Score 0.750** → governance verdict **REVIEW**; weakest dimension **AURGE** (0.666).
- GroupKFold-by-person AUC 0.799 ≈ time split → no person-level leakage.
- Over-prediction (calibration intercept −2.40) largely removed by Platt scaling
  (Brier 0.210 → 0.078).
- Binary and continuous (log-expenditure) framings rank individuals consistently
  (Spearman 0.80).

## Repository layout

```
.
├── data/
│   ├── raw/          # KHP SAS files (NOT distributed; see Data access)
│   └── processed/    # generated panel + model artifacts (git-ignored)
├── notebooks/        # 00–09, one code cell each
├── results/
│   ├── figures/      # grayscale PNG + PDF, 600 dpi (fig01–fig20)
│   └── tables/       # datacard, metrics, system card, subgroup outcomes (JSON/CSV)
├── src/              # notebook generators
├── .env.example      # copy to .env at repo root (LLM agent only)
├── requirements.txt
└── README.md
```

## Data access

The analysis uses the **Korea Health Panel Study (KHP), 2nd wave (2019–2024)**. The
microdata are **not** redistributed here and must be obtained from the official provider
under its data-use agreement. After obtaining the data, place the extracted SAS files so
that each year resolves as:

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

# Notebooks 00–04 and appendices 06–09 need no API key:
for nb in 00_data_preparation 01_eda 02_modelling 03_evaluation 04_governance \
          06_robustness_groupkfold 07_explainability 08_recalibration 09_continuous_cost; do
  jupyter nbconvert --to notebook --execute --inplace notebooks/$nb.ipynb
done
```

The **LLM chatbot agent (`05`)** is optional and the only component requiring API access;
it produces no numbers the other notebooks depend on. Copy `.env.example` to `.env` at the
repository root (one level above `notebooks/`) and set `OPENAI_API_KEY` and `LLM_MODEL_*`.
Run it once (after `03`) only to reproduce the governance-chatbot transcript.

Conventions: notebooks hold a single code cell with English comments; all randomness is
seeded (`seed = 42`); figures are grayscale, caption-free, and saved as PNG + PDF at 600 dpi;
no absolute paths are printed.

## Manuscript

The paper is written separately in LaTeX (Elsevier `elsarticle`, Overleaf project
`Trustworthy_HighCost_HealthPanel_ESWA`, targeting *Expert Systems with Applications*). The
`figures/fig*.pdf` files here are the camera-ready sources; `results/tables/*.json` back the
reported numbers.

## Design notes

- **Prospective split**: train on transitions 2019→2020 … 2022→2023; test on 2023→2024.
  A `GroupKFold`-by-person variant (appendix `06`) prevents person-level leakage.
- **Informative attrition**: dropout between *t* and *t+1* is reported by subgroup in the
  datacard; death during a transition censors cost and is treated as a competing risk.
- **Survey, not claims**: expenditures are self-reported panel values, not administrative
  claims — a stated limitation.

## License

Code released under the MIT License (see `LICENSE`). The KHP microdata are governed by their
own terms and are not covered by this license.

## Citation

Anonymous Author(s). *TRIAGE: an auditable agentic pipeline for trustworthy next-year
high-cost prediction on a longitudinal health panel.* Under review, 2026.
