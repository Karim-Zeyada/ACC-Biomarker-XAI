# ACC Biomarker Discovery via Explainable AI

> **Bachelor's Thesis** — Biomedical Informatics

Applying explainable machine learning (XAI) to classify Adrenocortical Carcinoma (ACC) molecular subtypes and discover candidate biomarker genes using SHAP analysis on high-dimensional RNA-seq data.

---

## Objective

Classify TCGA-ACC samples into two molecular subtypes and extract the **top 30 SHAP-ranked candidate biomarker genes**, bridging computational prediction with biological insight through model interpretability.

## Dataset

| Property | Value |
|----------|-------|
| **Source** | [TCGA-ACC via UCSC Xena (GDC Hub)](https://xenabrowser.net/datapages/?cohort=GDC%20TCGA%20Adrenocortical%20Cancer%20(ACC)) |
| **Expression** | RNA-seq STAR — FPKM (60,660 genes × 79 samples) |
| **Phenotype** | GDC clinical matrix (92 records) |
| **Subtypes** | Cluster 1 vs Cluster 2 (derived via k-means on PCA) |

## Pipeline

```
Raw FPKM ─→ Log2(FPKM+1) ─→ CV Filter (≥0.1) ─→ Top 2,000 Genes
         ─→ k-Means (k=2, PCA) ─→ Subtype Labels
         ─→ Stratified 5-Fold CV (RF / XGBoost / SVM)
         ─→ SHAP Explainability ─→ Top 30 Biomarker Genes
         ─→ Kaplan-Meier Survival ─→ Pathway Enrichment (Enrichr)
```

### Cross-Validation Results (Stratified 5-Fold)

| Model | Accuracy | F1 (macro) | AUC | MCC |
|-------|----------|------------|-----|-----|
| k-Means (Baseline) | 0.975 ± 0.050 | 0.973 ± 0.053 | 0.971 ± 0.057 | 0.953 ± 0.094 |
| Random Forest | 0.962 ± 0.050 | 0.959 ± 0.054 | **1.000 ± 0.000** | 0.926 ± 0.096 |
| XGBoost | 0.898 ± 0.051 | 0.896 ± 0.053 | 0.996 ± 0.007 | 0.808 ± 0.098 |
| **SVM (RBF)** | **0.975 ± 0.031** | **0.975 ± 0.031** | **1.000 ± 0.000** | **0.952 ± 0.059** |

## Project Structure

```
ACC/
├── data/
│   ├── raw/                    # TCGA-ACC raw data (not tracked — see Setup)
│   │   ├── TCGA-ACC.star_fpkm.tsv.gz
│   │   └── TCGA-ACC.clinical.tsv.gz
│   └── processed/              # Preprocessed feature matrices & labels
│       ├── X_normalized.csv
│       ├── X_log2_filtered.csv
│       ├── y_labels.csv
│       ├── clinical_aligned.csv
│       ├── gene_list_top2000.csv
│       └── preprocessing_metadata.json
├── notebooks/
│   ├── 01_preprocessing_eda.ipynb    # Data cleaning, EDA, feature selection
│   └── 02_modeling.ipynb             # Model training & cross-validation
├── results/
│   ├── models/                 # Trained model files (not tracked)
│   ├── cv_performance_comparison.csv
│   ├── cv_results_full.json
│   └── *.png                   # All EDA & performance plots
├── project_context.md          # Full project specification
├── requirements.txt            # Python dependencies
└── .gitignore
```

## Setup

### 1. Clone & create environment

```bash
git clone https://github.com/<your-username>/ACC-Biomarker-XAI.git
cd ACC-Biomarker-XAI
python -m venv .venv
# Windows
.venv\Scripts\activate
# macOS/Linux
source .venv/bin/activate
pip install -r requirements.txt
```

### 2. Download raw data

Download from [UCSC Xena GDC Hub — TCGA-ACC](https://xenabrowser.net/datapages/?cohort=GDC%20TCGA%20Adrenocortical%20Cancer%20(ACC)):

- **Gene Expression RNAseq** (STAR — FPKM) → `data/raw/TCGA-ACC.star_fpkm.tsv.gz`
- **Phenotype** (Clinical) → `data/raw/TCGA-ACC.clinical.tsv.gz`

### 3. Register Jupyter kernel

```bash
python -m ipykernel install --user --name acc_xai --display-name "ACC-XAI (Python 3)"
```

### 4. Run notebooks in order

```
notebooks/01_preprocessing_eda.ipynb  →  notebooks/02_modeling.ipynb
```

## Tech Stack

- **Python 3.10+**
- pandas, numpy, scikit-learn, XGBoost, SHAP, lifelines
- matplotlib, seaborn
- imbalanced-learn (SMOTE)

## Known Biological Targets

Candidate biomarkers are cross-referenced against known ACC driver genes:
- *IGF2*, *CTNNB1*, *ZNRF3*, *SF1*

## License

This project is part of a bachelor's thesis and is intended for academic use.
