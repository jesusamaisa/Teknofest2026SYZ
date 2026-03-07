# Implementation Plan

## Overview
End-to-end pipeline for missense variant classification using dbNSFP database.
All work is done inside a single Jupyter Notebook (`main.ipynb`).

---

## Phase 1: Environment Setup & Data Acquisition

### Step 1.1 — Install Dependencies
- pandas, numpy, scikit-learn, matplotlib, seaborn
- xgboost, lightgbm
- imbalanced-learn (for class balancing if needed)
- biopython (for sequence context retrieval)
- tqdm

### Step 1.2 — Download & Load dbNSFP
- Download dbNSFP v4.x (GRCh38) from https://sites.google.com/site/jpaborern/dbNSFP
- The database is large (~30GB uncompressed); load only relevant chromosomes/genes
- Parse the tab-separated dbNSFP files
- Keep `chr` and `pos` columns for later merging

### Step 1.3 — Understand dbNSFP Schema
- Document all available columns
- Map columns to required feature categories
- Identify which features come directly from dbNSFP vs. need custom engineering

---

## Phase 2: Data Filtering & Cleaning

### Step 2.1 — Filter by Variant Type
- Keep only SNVs (Single Nucleotide Variants)
- Keep only missense variants (check `aaref`, `aaalt`, or `Ensembl_proteinid` fields)
- Remove synonymous, nonsense, frameshift, etc.

### Step 2.2 — Filter by Gene
- Keep variants in: **PAH**, **CFTR**, and hereditary disease genes
- Use `genename` column from dbNSFP
- Define the list of hereditary disease genes (consult OMIM or ClinVar phenotype mapping)

### Step 2.3 — Filter by Clinical Review Status
- Use ClinVar review status field in dbNSFP (`clinvar_review`)
- Keep only variants with review status >= 3 stars
- Remove all variants with review status < 3

### Step 2.4 — Define Target Variable
- Map ClinVar clinical significance:
  - "Benign" + "Likely_benign" → **0 (Benign)**
  - "Pathogenic" + "Likely_pathogenic" → **1 (Pathogenic)**
  - Remove "Uncertain_significance" (VUS) and any other class

### Step 2.5 — Handle Missing Data
- Assess missingness per column
- Decide imputation strategy (median for numeric, mode for categorical, or drop)
- Document decisions

---

## Phase 3: Feature Engineering

### Step 3.1 — Biochemical & Structural Effects
**Custom-engineered features using amino acid property lookup tables:**

| Feature | Description |
|---------|-------------|
| delta_hydrophobicity | Hydrophobicity(alt) - Hydrophobicity(ref) |
| delta_volume | Volume(alt) - Volume(ref) |
| delta_charge | Charge(alt) - Charge(ref) |
| delta_molecular_weight | MW(alt) - MW(ref) |
| delta_polarity | Polarity(alt) - Polarity(ref) |
| grantham_distance | Grantham substitution matrix score |

- Build amino acid property dictionaries from published biochemistry literature
- Implement Grantham distance calculation from the original 1974 paper formula

### Step 3.2 — Sequence and Change Info
**From dbNSFP columns:**
- `aaref` / `aaalt` — reference and alternate amino acids
- `codonpos` — position within codon
- `refcodon` / `codon_degeneracy`
- One-hot encode amino acid changes or use ordinal encoding

### Step 3.3 — Local Sequence Context (Amino Acids)
- Extract +-5 amino acids around the variant position from protein sequence
- Use UniProt protein sequences (via Biopython or dbNSFP protein ID)
- Encode the local window (e.g., one-hot, BLOSUM encoding, or physicochemical vectors)

### Step 3.4 — Local Sequence Context (Nucleotides)
- Extract +-5 nucleotides around the variant position from reference genome
- Use GRCh38 reference FASTA or Ensembl REST API
- Encode the local window (one-hot or k-mer features)

### Step 3.5 — Evolutionary Conservation
**From dbNSFP columns (direct extraction):**
- `phyloP100way_vertebrate`
- `phyloP30way_mammalian`
- `phastCons100way_vertebrate`
- `phastCons30way_mammalian`
- `GERP++_RS`
- `GERP++_NR`
- `SiPhy_29way_logOdds`

### Step 3.6 — Population Data / MAF
**From dbNSFP columns:**
- `gnomAD_exomes_AF` — gnomAD exome allele frequency
- `gnomAD_genomes_AF` — gnomAD genome allele frequency
- `1000Gp3_AF` — 1000 Genomes allele frequency
- `ExAC_AF` — ExAC allele frequency
- `ESP6500_AA_AF` / `ESP6500_EA_AF` — ESP frequencies

### Step 3.7 — In Silico Risk Scores
**From dbNSFP columns:**
- `SIFT_score` / `SIFT_pred`
- `Polyphen2_HDIV_score` / `Polyphen2_HVAR_score`
- `CADD_phred`
- `REVEL_score`
- `MetaSVM_score` / `MetaLR_score`
- `MutationTaster_score`
- `FATHMM_score`
- `VEST4_score`
- `MutPred_score`
- `MVP_score`
- `MPC_score`
- `PrimateAI_score`
- `DEOGEN2_score`
- `BayesDel_addAF_score`
- `ClinPred_score`
- `LIST-S2_score`
- `Aloft_pred`

### Step 3.8 — Remove Genomic Coordinate Columns
- After all merging is complete, DROP: `chr`, `pos`, `#chr`, `pos(1-based)`, `hg38_chr`, `hg38_pos`, and any other positional columns
- This is a critical requirement from the prompt

---

## Phase 4: Exploratory Data Analysis (EDA)

### Step 4.1 — Class Distribution
- Bar chart of Benign vs Pathogenic counts
- Assess class imbalance ratio

### Step 4.2 — Feature Distributions
- Histograms / box plots for key numeric features
- Grouped by class (Benign vs Pathogenic)

### Step 4.3 — Correlation Analysis
- Correlation heatmap of all numeric features
- Identify highly correlated features (candidates for removal)

### Step 4.4 — Missing Value Analysis
- Heatmap of missing values per feature
- Document which features have high missingness

---

## Phase 5: Model Training & Evaluation

### Step 5.1 — Data Preparation
- Train/test split (80/20 stratified)
- Feature scaling if needed (StandardScaler for SVM/LR, not needed for tree models)
- Handle class imbalance: SMOTE, class_weight='balanced', or oversampling

### Step 5.2 — Baseline Models
Train and evaluate multiple models:
1. **Logistic Regression** — interpretable baseline
2. **Random Forest** — robust ensemble
3. **XGBoost** — gradient boosting
4. **LightGBM** — fast gradient boosting
5. **SVM** — support vector machine (optional, slower)

### Step 5.3 — Hyperparameter Tuning
- Use GridSearchCV or RandomizedSearchCV with stratified k-fold (5-fold)
- Tune top 2-3 performing models
- Optimize for F1-score (important with potential class imbalance)

### Step 5.4 — Model Evaluation
- **Metrics**: Accuracy, Precision, Recall, F1-Score, AUC-ROC, MCC
- Confusion matrix visualization
- ROC curve comparison across models
- Precision-Recall curve

### Step 5.5 — Feature Importance
- Plot top 30 features by importance (from best model)
- SHAP values analysis (if time permits)
- Discuss which feature categories contribute most

---

## Phase 6: Finalization

### Step 6.1 — Save Best Model
- Pickle the best model (`best_model.pkl`)
- Save the preprocessing pipeline

### Step 6.2 — Documentation
- Clean notebook with markdown headers and explanations
- Add inline comments
- Generate summary visualizations

### Step 6.3 — Competition Deliverables
- Final `.ipynb` notebook
- Trained model file
- Feature importance and evaluation plots
- Ensure reproducibility (random seeds, version info)

---

## Dependency Graph

```
Phase 1 (Setup)
    → Phase 2 (Filtering)
        → Phase 3 (Feature Engineering)
            → Phase 4 (EDA)
                → Phase 5 (Modeling)
                    → Phase 6 (Finalization)
```

## Key Risk Areas
1. **dbNSFP file size** — may need chunked reading or gene-specific extraction
2. **Sequence context retrieval** — external API calls can be slow; consider caching
3. **Class imbalance** — benign variants often outnumber pathogenic significantly
4. **Feature leakage** — ClinPred and some meta-scores are trained on ClinVar labels; may need to exclude
5. **Gene list for hereditary** — needs clear definition of which genes count as "hereditary"
