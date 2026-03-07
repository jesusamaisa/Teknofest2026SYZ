# Progress Tracker

## Current Status: FULL PIPELINE CREATED — Awaiting First Run

---

## Phase Completion

| Phase | Status | Notes |
|-------|--------|-------|
| Phase 1: Environment Setup & Data Acquisition | DONE | myvariant.info API, JSON cache |
| Phase 2: Data Filtering & Cleaning | DONE | missense SNV, review≥3, binary label |
| Phase 3: Feature Engineering | DONE | biochem deltas, conservation, MAF, in silico scores |
| Phase 4: EDA | DONE | class dist, feature dists, correlation heatmap |
| Phase 5: Model Training & Evaluation | DONE | LR, RF, XGB, LGBM + tuning + ROC |
| Phase 6: Finalization | DONE | save model, scaler, feature list |

---

## Detailed Task Tracking

### Phase 1
- [ ] Install all dependencies
- [ ] Download dbNSFP GRCh38
- [ ] Load and inspect dbNSFP schema
- [ ] Document available columns

### Phase 2
- [ ] Filter SNV + missense only
- [ ] Filter target genes (PAH, CFTR, hereditary)
- [ ] Filter review status >= 3
- [ ] Define target variable (merge benign/likely-benign, pathogenic/likely-pathogenic)
- [ ] Remove VUS
- [ ] Handle missing values

### Phase 3
- [ ] Build amino acid property lookup tables
- [ ] Calculate biochemical delta features
- [ ] Calculate Grantham distance
- [ ] Extract sequence and change info
- [ ] Retrieve local amino acid context (+-5)
- [ ] Retrieve local nucleotide context (+-5)
- [ ] Extract conservation scores
- [ ] Extract population/MAF data
- [ ] Extract in silico risk scores
- [ ] Remove genomic coordinate columns

### Phase 4
- [ ] Class distribution analysis
- [ ] Feature distribution plots
- [ ] Correlation heatmap
- [ ] Missing value analysis

### Phase 5
- [ ] Train/test split
- [ ] Train baseline models (LR, RF, XGB, LGBM)
- [ ] Hyperparameter tuning
- [ ] Evaluate all models (Accuracy, F1, AUC, MCC)
- [ ] Confusion matrices
- [ ] ROC curves
- [ ] Feature importance analysis

### Phase 6
- [ ] Save best model
- [ ] Clean notebook
- [ ] Generate final visualizations
- [ ] Verify reproducibility

---

## Open Questions
1. What exact genes count as "hereditary" besides PAH and CFTR?
2. Should we exclude meta-predictor scores (ClinPred, REVEL, MetaSVM) to avoid feature leakage?
3. What is the minimum acceptable sample size after all filtering?

---

## Session Log

| Date | What was done |
|------|---------------|
| 2026-03-05 | Project reset. Old ClinVar+VEP approach deleted. Memory bank created. Implementation plan written. |
