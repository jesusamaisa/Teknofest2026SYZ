# Project Progress

Last Updated: 2026-03-22

## Status Summary

- Current phase: Full notebook generated — all 18 sections implemented
- Next milestone: Run the notebook end-to-end to validate
- Blocking issues: None

## Milestones

1. **Setup & Data Acquisition**
   - [x] Confirm target genes list and panel membership
   - [x] Pin ClinVar filtering rules and label mapping
   - [x] Load ClinVar variant_summary.txt.gz  
2. **Data Filtering & Preparation**
   - [x] Filter ClinVar by ReviewStatus, type, consequence, genes, labels
   - [x] Build minimal VCF (CHROM, POS, REF, ALT)
3. **Feature Extraction**
   - [x] VEP REST API integration
   - [x] Extract gnomAD AF from VEP output
   - [x] Extract nucleotide context (±5 nt) from GRCh38 FASTA
   - [x] Extract amino-acid context (±5 AA) from idmapping_2026_03_16.fasta
   - [x] Compute biochemical features (delta scales, Grantham)
4. **Data Merging & Preprocessing**
   - [x] Assemble feature table and handle missing values
   - [x] Merge all features and drop genomic address columns
   - [x] EDA: class balance and feature distributions
   - [x] Preprocessing: encoding, scaling, missing value handling
   - [x] Split dataset into 3 panels: PAH, CFTR, Hereditary
   - [x] Independent train/test split per panel
5. **Modeling & Evaluation (Per Panel)**
   - [x] Train candidate models (XGBoost, LightGBM, Random Forest) per panel
   - [x] Hyperparameter tuning per panel (Optuna)
   - [x] Evaluate metrics (Accuracy, F1, ROC-AUC, etc.) per panel
   - [x] Feature importance and interpretability (SHAP) per panel
6. **Finalization**
   - [x] Model export per panel, final feature matrix export

## Work Log

- 2026-03-16:
  - Created the initial notebook scaffold (missense_classification.ipynb) with sections aligned to the project plan.

## Decisions

- Target genes confirmed: PAH, CFTR, BRCA1, BRCA2, PALB2, MLH1, MSH2, MSH6, PMS2, EPCAM, TP53, APC, PTEN, CDH1.
- Label mapping confirmed: Benign + Likely benign -> Benign; Pathogenic + Likely pathogenic -> Pathogenic; drop others.
- Use local ClinVar VCF (clinvar_20260218.vcf) for initial parsing.

## Risks / Open Questions

- ClinVar strict ReviewStatus filters may yield a small dataset
  - Mitigation: track class counts after filtering; allow fallback to 2-star if counts are too low (only if approved).
- VEP REST API rate limits and connection stability
  - Mitigation: implement retry logic and strict delays to avoid being blocked by the Ensembl API.
- Protein position mapping errors for long proteins (e.g., BRCA2)
  - Mitigation: validate protein position parsing with spot checks; flag variants with unmapped positions.
- Missing gnomAD AF for some variants; need consistent imputation
  - Mitigation: impute Global MAF = 0 and add a boolean missing flag.
