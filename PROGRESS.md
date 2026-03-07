# Progress Tracker

## Current Status: FULL PIPELINE CREATED — Awaiting First Run

---

## Phase Completion

| Phase | Status | Notes |
|-------|--------|-------|
| Phase 1: Environment Setup & Data Acquisition | DONE | myvariant.info API, JSON cache |
| Phase 2: Data Filtering & Cleaning | DONE | missense SNV, review≥1, binary label |
| Phase 3: Feature Engineering | DONE | biochem deltas, conservation, MAF, in silico scores |
| Phase 4: EDA | DONE | class dist, feature dists, correlation heatmap |
| Phase 5: Model Training & Evaluation | DONE | LR, RF, XGB, LGBM + tuning + ROC |
| Phase 6: Finalization | DONE | save model, scaler, feature list |

---

## Detailed Task Tracking

### Phase 1
- [x] Install all dependencies (`%pip install` in Cell 2)
- [x] Import all packages (Cell 3)
- [x] Define TARGET_GENES (25 genes) and FIELDS list (Cell 5)
- [x] Initialize myvariant.info API client (Cell 5)
- [x] Query API per gene & cache to `dbnsfp_raw_variants.json` (Cell 7)
- [x] Flatten JSON → DataFrame with `pd.json_normalize` (Cell 8)

### Phase 2
- [x] Filter missense SNVs only — valid single AA ref ≠ alt, 20 standard AAs (Cell 10)
- [x] Parse ClinVar review_status → star rating 0–4 (Cell 11)
- [x] Filter review_stars >= 1 (Cell 11) *(note: markdown says ≥3 but code uses ≥1)*
- [x] Map clinical_significance to binary label: Benign/Likely Benign → 0, Pathogenic/Likely Pathogenic → 1 (Cell 12)
- [x] Drop VUS and ambiguous variants (label == -1) (Cell 12)

### Phase 3
- [x] Build amino acid property lookup tables — 6 dicts: hydrophobicity, volume, charge, mol_weight, polarity, Grantham components (Cell 14)
- [x] Calculate biochemical delta features — delta_hydrophobicity, delta_volume, delta_charge, delta_mol_weight, delta_polarity (Cell 14)
- [x] Calculate Grantham distance (Cell 14)
- [x] Encode ref/alt AA as 5D physicochemical vectors (10 features) + aa_position, codon_position (Cell 16)
- [ ] ~~Retrieve local amino acid context (+-5)~~ — NOT IMPLEMENTED
- [ ] ~~Retrieve local nucleotide context (+-5)~~ — NOT IMPLEMENTED
- [x] Extract conservation scores — GERP via column mapping (Cell 18)
- [x] Extract population/MAF data — 1000G, ExAC, ESP, gnomAD, max_population_af, is_rare (Cell 20)
- [x] Extract in silico risk scores — SIFT, FATHMM, PROVEAN, MutationTaster, MutationAssessor, DANN, REVEL, MetaLR, MetaSVM (Cell 22)
- [x] Remove genomic coordinate/ID/meta columns via DROP_PATTERNS (Cell 24)
- [x] Handle missing values — drop >80% missing features, median-impute rest via SimpleImputer (Cell 25)

### Phase 4
- [x] Class distribution analysis — bar + pie chart, imbalance ratio (Cell 27)
- [x] Feature distribution histograms by class for key features (Cell 28)
- [x] Correlation heatmap + print pairs with |r| > 0.9 (Cell 29)
- [ ] ~~Missing value analysis~~ — listed in notebook markdown header (4.4) but no code cell; missingness summary already printed in Phase 3 Cell 25

### Phase 5
- [x] 80/20 stratified train/test split + StandardScaler + SMOTE on train only, SEED=42 (Cell 31)
- [x] Train 4 baseline models: LR (max_iter=2000), RF (300), XGB (300), LGBM (300) — evaluate Accuracy, F1, AUC-ROC, MCC (Cell 32)
- [x] GridSearchCV (5-fold stratified, scoring=f1) on top 2 models by F1 (Cell 33)
- [x] Confusion matrices for all models (Cell 34)
- [x] ROC curves for all models (Cell 35)
- [x] Feature importance bar chart (top 30) from best model (Cell 36)
- [x] Classification report for best model (Cell 37)

### Phase 6
- [x] Save best_model.pkl, scaler.pkl, feature_columns.json + print final summary (Cell 39)
- [ ] ~~Clean notebook~~ — NOT IMPLEMENTED (no cell for clearing/restructuring)
- [ ] ~~Generate final visualizations~~ — NOT APPLICABLE (all visualizations already in Phase 4 & 5)
- [ ] ~~Verify reproducibility~~ — NOT IMPLEMENTED (no re-run/validation cell)

---

## Open Questions
1. Should we exclude meta-predictor scores (ClinPred, REVEL, MetaSVM) to avoid feature leakage?
2. What is the minimum acceptable sample size after all filtering?

---

## Session Log

| Date | What was done |
|------|---------------|
| 2026-03-05 | Project reset. Old ClinVar+VEP approach deleted. Memory bank created. Implementation plan written. |
| 2026-03-05 | Full pipeline coded in main.ipynb (Phases 1–6): API data acquisition, filtering, feature engineering, EDA, model training, finalization. |
| 2026-03-07 | Deleted memory_bank/ directory. Copied progress.md → PROGRESS.md. Created AGENTS.md with full project reference. |
| 2026-03-07 | Audited all 6 phases in PROGRESS.md against actual notebook code. Fixed outdated/inaccurate task descriptions. Marked completed tasks [x], flagged unimplemented items as strikethrough. |
