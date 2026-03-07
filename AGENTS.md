# Agents

> **Read this file first when context is full.** It contains everything needed to understand and continue work on this project without re-reading the entire notebook.

---

## Project Overview

**Goal:** Binary classification of human missense variants as **Pathogenic (1)** vs **Benign (0)** using features derived from **dbNSFP** annotations.

**Data source:** [myvariant.info](https://myvariant.info/) REST API — queries the dbNSFP + ClinVar sub-documents per variant. No local dbNSFP download required (~few MB vs ~30 GB).

**Language / Framework:** Python, Jupyter Notebook (`main.ipynb`), scikit-learn, XGBoost, LightGBM, imbalanced-learn (SMOTE).

---

## File Structure

```
main.ipynb                      # Full pipeline (data → EDA → model → evaluation) — single notebook, ~1035 lines
dbnsfp_raw_variants.json        # Cached API responses (auto-generated, gitignored via *.json)
feature_columns.json            # Ordered feature list used by the model (auto-generated, 42 features, gitignored)
best_model_xgb.pkl              # Saved best trained model (auto-generated)
scaler.pkl                      # Saved StandardScaler (auto-generated)
output_finetuned.png            # Output image from finetuned model run
results_without_finetuned.png   # Output image from baseline (pre-tuning) run
AGENTS.md                       # THIS FILE — project knowledge base for agents
PROGRESS.md                     # Phase-level progress tracker (audited 2026-03-07)
README.md                       # User-facing project description
_prompt_for_memorybank.txt      # Legacy prompt file (can be deleted)
.gitignore                      # Ignores .venv/ and *.json
```

---

## Pipeline Summary (main.ipynb)

The notebook is organized into **6 phases**, executed sequentially in a single file.

### Phase 1 — Environment Setup & Data Acquisition
- **Cell 1 (Markdown):** Title & objective.
- **Cell 2:** `%pip install` dependencies.
- **Cell 3:** All imports (pandas, numpy, sklearn, xgboost, lightgbm, imblearn, myvariant, matplotlib, seaborn).
- **Cell 4 (Markdown):** Gene list rationale.
- **Cell 5:** Defines `TARGET_GENES` (25 genes) and `FIELDS` (dbNSFP + ClinVar fields). Initializes `myvariant.MyVariantInfo()` client.
- **Cell 6 (Markdown):** Data acquisition strategy.
- **Cell 7:** Queries myvariant.info per gene (`dbnsfp.genename:<GENE>`, size=1000, fetch_all=True). Caches results to `dbnsfp_raw_variants.json`. Loads from cache on re-run.
- **Cell 8:** `pd.json_normalize(all_hits, sep="_")` → `df_raw`.

### Phase 2 — Data Filtering & Cleaning
- **Cell 9 (Markdown):** Filtering strategy.
- **Cell 10:** Keep only missense SNVs (valid single AA ref ≠ alt, 20 standard amino acids).
- **Cell 11:** Parse ClinVar `review_status` → star rating (0–4). Filter `review_stars >= 1`.
- **Cell 12:** Map `clinical_significance` to binary label: Benign/Likely Benign → 0, Pathogenic/Likely Pathogenic → 1. Drop VUS and ambiguous.

### Phase 3 — Feature Engineering
- **Cell 13 (Markdown):** Biochemical delta features description.
- **Cell 14:** Amino acid property lookup tables (Kyte-Doolittle hydrophobicity, Zamyatnin volume, charge at pH 7, molecular weight, Zimmerman polarity, Grantham components). Computes: `delta_hydrophobicity`, `delta_volume`, `delta_charge`, `delta_mol_weight`, `delta_polarity`, `grantham_distance`.
- **Cell 15 (Markdown):** AA encoding description.
- **Cell 16:** Encodes ref/alt AA as 5D physicochemical vectors (10 features). Extracts `aa_position`, `codon_position`.
- **Cell 17 (Markdown):** Conservation scores description.
- **Cell 18:** Extracts conservation scores from dbNSFP columns (GERP, phyloP, phastCons, SiPhy) via column name mapping.
- **Cell 19 (Markdown):** MAF description.
- **Cell 20:** Extracts population allele frequencies (gnomAD, 1000G, ExAC, ESP). Creates `max_population_af` and binary `is_rare` (AF < 0.01).
- **Cell 21 (Markdown):** In silico scores description.
- **Cell 22:** Extracts prediction scores (SIFT, FATHMM, PROVEAN, MutationTaster, MutationAssessor, DANN, REVEL, MetaLR, MetaSVM, PolyPhen2, CADD) via column name mapping.
- **Cell 23 (Markdown):** Feature matrix cleanup description.
- **Cell 24:** Drops all coordinate/ID/meta/ClinVar columns. Keeps only numeric feature columns. Prints final feature list.
- **Cell 25:** Handles missing values — drops features with >80% missing, median-imputes the rest via `SimpleImputer`.

### Phase 4 — EDA
- **Cell 26 (Markdown):** EDA overview.
- **Cell 27:** Class distribution bar + pie chart. Prints imbalance ratio.
- **Cell 28:** Feature distribution histograms by class (key features).
- **Cell 29:** Correlation heatmap. Prints pairs with |r| > 0.9.

### Phase 5 — Model Training & Evaluation
- **Cell 30 (Markdown):** Training strategy.
- **Cell 31:** 80/20 stratified split. `StandardScaler` on train, transform test. **SMOTE** on training set only. `SEED = 42`.
- **Cell 32:** Train 4 baseline models: Logistic Regression, Random Forest (300 trees), XGBoost (300 trees), LightGBM (300 trees). Evaluate on test set: Accuracy, F1, AUC-ROC, MCC.
- **Cell 33:** GridSearchCV (5-fold stratified, scoring="f1") on top 2 models by F1. Param grids defined for all 4 models.
- **Cell 34:** Confusion matrices for all models.
- **Cell 35:** ROC curves for all models.
- **Cell 36:** Feature importance bar chart (top 30) from best model.
- **Cell 37:** Classification report for best model.

### Phase 6 — Finalization
- **Cell 38 (Markdown):** Finalization description.
- **Cell 39:** Saves `best_model.pkl`, `scaler.pkl`, `feature_columns.json`. Prints final summary.
- **Cell 40:** Empty cell.

---

## Target Genes (25)

| Gene | Disease |
|------|---------|
| PAH | Phenylketonuria |
| CFTR | Cystic Fibrosis |
| BRCA1 | Hereditary Breast/Ovarian Cancer |
| BRCA2 | Hereditary Breast/Ovarian Cancer |
| MLH1 | Lynch Syndrome |
| MSH2 | Lynch Syndrome |
| MSH6 | Lynch Syndrome |
| TP53 | Li-Fraumeni Syndrome |
| RB1 | Retinoblastoma |
| SCN1A | Dravet Syndrome (Epilepsy) |
| HEXA | Tay-Sachs Disease |
| GBA | Gaucher Disease |
| GAA | Pompe Disease |
| HBB | Sickle Cell / Beta-Thalassemia |
| LDLR | Familial Hypercholesterolemia |
| PKD1 | Polycystic Kidney Disease |
| PKD2 | Polycystic Kidney Disease |
| FBN1 | Marfan Syndrome |
| TSC1 | Tuberous Sclerosis |
| TSC2 | Tuberous Sclerosis |
| MYH7 | Hypertrophic Cardiomyopathy |
| MYBPC3 | Hypertrophic Cardiomyopathy |
| RET | Multiple Endocrine Neoplasia |
| VHL | Von Hippel-Lindau |
| ATM | Ataxia-Telangiectasia |

---

## Feature Set (42 features from feature_columns.json)

### Custom-Engineered (6)
- `delta_hydrophobicity`, `delta_volume`, `delta_charge`, `delta_mol_weight`, `delta_polarity`, `grantham_distance`

### dbNSFP Score Columns (passed through from API, 22)
- `_score`, `dbnsfp_dann_rankscore`, `dbnsfp_dann_score`, `dbnsfp_fathmm_converted_rankscore`, `dbnsfp_gerp_91_mammals_rankscore`, `dbnsfp_gerp_91_mammals_score`, `dbnsfp_metalr_rankscore`, `dbnsfp_metalr_score`, `dbnsfp_metasvm_rankscore`, `dbnsfp_metasvm_score`, `dbnsfp_mutationassessor_rankscore`, `dbnsfp_mutationtaster_converted_rankscore`, `dbnsfp_provean_converted_rankscore`, `dbnsfp_revel_rankscore`, `dbnsfp_sift_converted_rankscore`

### Cleaned Score Columns (mapped from dbNSFP, 20)
- `gerp_91_mammals_score`, `gerp_91_mammals_rankscore`, `sift_score`, `sift_rankscore`, `fathmm_score`, `fathmm_rankscore`, `provean_score`, `provean_rankscore`, `mutationtaster_score`, `mutationtaster_rankscore`, `mutationassessor_score`, `mutationassessor_rankscore`, `dann_score`, `dann_rankscore`, `revel_score`, `revel_rankscore`, `metalr_score`, `metalr_rankscore`, `metasvm_score`, `metasvm_rankscore`

### Population (1)
- `is_rare` (binary: max population AF < 0.01)

---

## Key Technical Decisions

1. **Data source:** myvariant.info API (not local dbNSFP download) — lightweight, ~few MB.
2. **ClinVar review filter:** `review_stars >= 1` (was originally planned as >=3 but relaxed for sample size).
3. **Label mapping:** Benign + Likely Benign → 0; Pathogenic + Likely Pathogenic → 1; VUS/other → dropped.
4. **Class imbalance:** SMOTE applied to training set only (not test set).
5. **Missing values:** Features >80% missing are dropped; rest median-imputed.
6. **Scaling:** StandardScaler fit on train, transform on test (important for Logistic Regression).
7. **No genomic coordinates in features** — all chr/pos/hg38 columns removed to prevent location leakage.
8. **Random seed:** 42 throughout.

---

## Dependencies

```
myvariant pandas numpy scikit-learn matplotlib seaborn xgboost lightgbm imbalanced-learn tqdm
```

Python virtual environment: `.venv/` (activate with `source .venv/Scripts/activate` on bash or `.venv\Scripts\Activate.ps1` on PowerShell).

---

## Current Status

**Pipeline is fully coded but has NOT been successfully executed end-to-end yet.** The notebook cells have cached outputs from a previous run but none are currently executed in the active kernel.

### Known Gaps (from 2026-03-07 audit)
- **Local AA context (±5)** — planned but never implemented in Phase 3.
- **Local nucleotide context (±5)** — planned but never implemented in Phase 3.
- **Phase 4 Missing Value Analysis** — section header exists in notebook but no code cell; missingness is printed in Phase 3 Cell 25 instead.
- **Phase 6 notebook cleanup / reproducibility verification** — planned but never implemented.
- **Phase 2 markdown vs code mismatch** — notebook markdown says review ≥3 but code filters ≥1.

### Open Questions
1. Should meta-predictor scores (ClinPred, REVEL, MetaSVM) be excluded to avoid feature leakage?
2. What is the minimum acceptable sample size after all filtering?

---

## How to Run

1. Activate venv: `.venv\Scripts\Activate.ps1`
2. Open `main.ipynb` in VS Code / Jupyter
3. Run all cells sequentially
4. First run queries the API and caches to `dbnsfp_raw_variants.json`; subsequent runs load from cache
5. Outputs: `best_model_xgb.pkl`, `scaler.pkl`, `feature_columns.json`
