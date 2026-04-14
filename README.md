# DrugPredict

**UC Berkeley Master of Information and Data Science — Capstone Project**

A machine learning framework for predicting GLP-1 receptor agonist efficacy in Type 2 Diabetes patients. Given a drug's molecular structure and a patient's clinical profile, DrugPredict outputs a predicted HbA1c reduction and the probability that the patient achieves a clinically meaningful response. This project was developed as a proof of concept for a broader vision: using computational tools to identify responder populations earlier in drug development, reducing trial failures and accelerating the path from molecule to market.

---

## Demo

<!-- TODO: Replace with your actual demo video link -->
▶ [Watch the Demo](#https://github.com/farhanquadri7/DrugPredict/blob/main/DrugPredictDemo.mp4)

---

## Overview

Bringing a new drug to market costs over $2.5 billion and takes 10–15 years. More than 90% of drug candidates fail in clinical trials — often because researchers don't know in advance which patient populations will respond. DrugPredict addresses this by combining molecular cheminformatics with real clinical outcomes data to score drug-patient pairs before a trial begins.

**Two model outputs per prediction:**
- **Predicted ΔHbA1c (%)** — expected reduction in HbA1c at 26 weeks (e.g., −1.4%)
- **P(Responder)** — probability the patient achieves >1% HbA1c reduction (clinically meaningful threshold)

---

## Notebook

[`DrugPredict_code.ipynb`](DrugPredict_code.ipynb) contains the full pipeline across 14 steps:

| Step | Description |
|------|-------------|
| 1 | Drug SMILES strings sourced from ChEMBL and PubChem |
| 2 | ECFP4 Morgan fingerprints (2,048 bits) generated via RDKit |
| 2b–2c | Fragment visualization and chemical group translation |
| 3 | ~104 ADMET properties computed via ADMET-AI |
| 4 | Clinical trial patient data queried from ClinicalTrials.gov (AACT) |
| 4b | Feature enrichment — baseline labs, comorbidities, trial design |
| 5 | Feature matrix assembly (~2,167 columns per drug-patient row) |
| 6 | XGBoost regressor and classifier trained with GroupShuffleSplit |
| 7 | SHAP feature importance analysis |
| 8 | Single-patient predictions for 5 approved drugs |
| 9 | Synthetic 100-patient cohort with segment analysis |
| 10 | `predict_from_smiles()` — core inference function |
| 11 | Demo drug catalog (5 approved GLP-1 drugs) |
| 12 | Minor drug variants (+2 CH₂ fatty acid extensions) |
| 13 | Novel drug candidates — 3 structurally distinct hypothetical molecules |
| 14 | Web app handoff exports |

> **Note:** AACT database credentials have been removed from this notebook. To run Steps 4–5, you will need a free AACT account (see [Data Sources](#data-sources) below).

---

## Model Performance

Trained on **231 GLP-1 clinical trial arms** from ClinicalTrials.gov:

| Metric | Value |
|--------|-------|
| Regression RMSE | ±0.488% ΔHbA1c |
| Regression R² | 0.453 |
| Classification AUC-ROC | 0.830 |
| Train/test split | GroupShuffleSplit by trial ID (prevents leakage) |

**Key SHAP findings:**
-  Age and baseline HbA1c are the two most important features overall — patients with higher baseline HbA1c tend to show larger reductions, consistent with clinical literature
-  Obesity flag is the only comorbidity that contributes meaningfully to predictions
-  Weight (kg), BMI, and diabetes duration highlight key influential patient characteristics
-  ADMET properties — particularly skin reaction potential (Skin_Reaction) and intestinal permeability (Caco-2) — contribute moderate but consistent signal beyond patient demographics

---

## Feature Engineering

Each drug-patient pair is represented as a ~2,167-column feature vector:

| Feature Group | Columns | Source |
|---------------|---------|--------|
| ECFP4 Morgan fingerprint | 2,048 bits | RDKit (`radius=2, nBits=2048`) |
| ADMET properties | ~104 features | ADMET-AI |
| Patient / trial features | ~15 features | AACT + derived |

**Patient features include:** baseline HbA1c, BMI, age, diabetes duration, weight, fasting glucose, treatment duration (weeks), and binary flags for obesity, hypertension, CVD, CKD, dyslipidemia, NAFLD, neuropathy, and retinopathy.

---

## Novel Drug Candidates (Step 13)

Three hypothetical candidates were designed by applying structural modifications to approved GLP-1 drugs using RDKit reaction SMARTS:

| Candidate | Base Drug | Modification | Rationale |
|-----------|-----------|--------------|-----------|
| **LiraBenzyl** | Liraglutide | C16 terminal CH₃ → benzyl (−CH₂Ph) | Aromatic terminus for novel lipophilic binding |
| **SemaPiperi** | Semaglutide | C18 diacid COOH → N-piperidyl amide | Altered polarity and albumin binding profile |
| **TirzDiF** | Tirzepatide | C20 diacid alpha-CH₂ → gem-CF₂ | Bioisosteric difluoro for metabolic stabilization |

> These are **in-silico hypothetical candidates**. Predictions are directional model extrapolations — the molecules have not been synthesized or tested.

---

## Data Sources

| Source | What it provides | Access |
|--------|-----------------|--------|
| [ChEMBL](https://www.ebi.ac.uk/chembl/) | SMILES for liraglutide, exenatide | Public API |
| [PubChem](https://pubchem.ncbi.nlm.nih.gov/) | SMILES for semaglutide, tirzepatide, lixisenatide | Public API |
| [AACT (ClinicalTrials.gov)](https://aact.ctti-clinicaltrials.org/) | 231 GLP-1 trial arms with HbA1c outcomes | Free account required |

To connect to AACT, register for a free account at [aact.ctti-clinicaltrials.org](https://aact.ctti-clinicaltrials.org/users/sign_up) and add your credentials to the AACT query cell in the notebook.

---

## Setup

**Requirements**

```
python >= 3.11
rdkit
admet-ai
xgboost
scikit-learn
pandas
numpy
matplotlib
shap
sqlalchemy
psycopg2
```

**Install**

```bash
pip install rdkit admet-ai xgboost scikit-learn pandas numpy matplotlib shap sqlalchemy psycopg2-binary
```

> ADMET-AI downloads model weights on first run (~500 MB). Each `predict()` call takes 2–4 seconds on CPU.

**Run the notebook**

```bash
jupyter notebook DrugPredict_code.ipynb
```

Steps 1–3, 6–14 can be run without AACT credentials using the pre-exported feature matrix. Steps 4–5 require a live AACT connection to re-query patient data.

---

## Drugs in the Model

| Drug | Mechanism | SMILES Source |
|------|-----------|---------------|
| Semaglutide | GLP-1 agonist | PubChem |
| Liraglutide | GLP-1 agonist | ChEMBL |
| Tirzepatide | GLP-1 / GIP dual agonist | PubChem |
| Exenatide | GLP-1 agonist | ChEMBL |
| Lixisenatide | GLP-1 agonist | PubChem |

---

## Team

**Farhan Quadri** — UC Berkeley MIDS · Data Scientist at Abbott (FreeStyle Libre CGM)
https://www.linkedin.com/in/farhan-quadri-engineer/

**Kevin Coppa** — UC Berkeley MIDS · Sr. Data Scientist at Northwell Health
https://www.linkedin.com/in/kevin-coppa/

---

## Links

Link to Github website: https://farhanquadri7.github.io/DrugPredict/
Video demo: https://github.com/farhanquadri7/DrugPredict/blob/main/DrugPredictDemo.mp4

---

*UC Berkeley School of Information · Master of Information and Data Science · 2025*
