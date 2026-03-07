# Technical Decisions & Architecture

## Decision Log

### D1: Database Choice — dbNSFP over ClinVar VCF + VEP
- **Previous approach**: ClinVar VCF parsed locally, then annotated via Ensembl VEP web tool
- **New approach**: dbNSFP which already integrates ClinVar annotations + 100+ precomputed scores
- **Rationale**: dbNSFP is a one-stop database; eliminates the need for external VEP API calls and manual annotation merging
- **Trade-off**: Large file size (~30GB), but richer feature set out of the box

### D2: Genome Build — GRCh38
- All coordinates and references use GRCh38 (hg38)
- dbNSFP v4.x natively supports GRCh38

### D3: Review Status Threshold — >= 3 Stars
- ClinVar review statuses:
  - 0 star: no assertion criteria provided
  - 1 star: criteria provided, single submitter
  - 2 stars: criteria provided, multiple submitters, no conflict
  - 3 stars: reviewed by expert panel
  - 4 stars: practice guideline
- Keeping >= 3 ensures high-quality, expert-reviewed labels
- This will significantly reduce sample size but improve label quality

### D4: Target Genes — PAH, CFTR, Hereditary
- **PAH**: Phenylalanine Hydroxylase (Phenylketonuria)
- **CFTR**: Cystic Fibrosis Transmembrane Conductance Regulator
- **Hereditary**: Needs definition — likely includes well-known hereditary disease genes from OMIM
- **Action needed**: Define the exact list of "hereditary" genes

### D5: Feature Leakage Concern
- Some dbNSFP scores (ClinPred, REVEL, MetaSVM) are meta-predictors trained on ClinVar labels
- Using these as features while predicting ClinVar labels creates **circular reasoning**
- **Options**:
  - A) Use them anyway (common in competition settings for max performance)
  - B) Exclude meta-predictors that use ClinVar in training
  - C) Use them but document the limitation
- **Decision**: TBD — discuss with team

### D6: Class Imbalance Strategy
- Benign variants typically outnumber Pathogenic
- **Options**:
  - SMOTE oversampling of minority class
  - `class_weight='balanced'` in model parameters
  - Undersampling majority class
  - Combination approach
- **Decision**: Start with `class_weight='balanced'`, add SMOTE if needed

### D7: Encoding Strategy for Sequence Context
- Amino acid windows and nucleotide windows need numeric encoding
- **Options**:
  - One-hot encoding (sparse but lossless)
  - Physicochemical property vectors (dense, biologically meaningful)
  - BLOSUM62 encoding (for amino acids, captures evolutionary similarity)
  - k-mer frequency (for nucleotides)
- **Decision**: Use physicochemical vectors for amino acids, one-hot for nucleotides

---

## Architecture: Notebook Structure

```
main.ipynb
├── Cell Block 1: Setup & Imports
├── Cell Block 2: Amino Acid Property Tables & Helper Functions
├── Cell Block 3: Load & Parse dbNSFP
├── Cell Block 4: Data Filtering (variant type, genes, review status)
├── Cell Block 5: Target Variable Definition
├── Cell Block 6: Feature Engineering — Biochemical
├── Cell Block 7: Feature Engineering — Sequence Context (AA)
├── Cell Block 8: Feature Engineering — Sequence Context (NT)
├── Cell Block 9: Feature Engineering — Conservation, Population, In Silico
├── Cell Block 10: Remove Coordinate Columns
├── Cell Block 11: EDA
├── Cell Block 12: Data Preparation & Split
├── Cell Block 13: Model Training
├── Cell Block 14: Hyperparameter Tuning
├── Cell Block 15: Evaluation & Visualization
├── Cell Block 16: Feature Importance
├── Cell Block 17: Save Model & Summary
```

---

## Key Libraries & Versions

| Library | Purpose |
|---------|---------|
| pandas | Data manipulation |
| numpy | Numerical operations |
| scikit-learn | ML models, metrics, preprocessing |
| xgboost | Gradient boosting |
| lightgbm | Gradient boosting (fast) |
| imbalanced-learn | SMOTE, class balancing |
| matplotlib | Visualization |
| seaborn | Statistical visualization |
| biopython | Sequence retrieval |
| tqdm | Progress bars |

---

## dbNSFP Key Columns Mapping

### Direct extraction (no engineering needed)
| Feature Category | dbNSFP Column(s) |
|-----------------|-------------------|
| Conservation | phyloP100way_vertebrate, phyloP30way_mammalian, phastCons100way_vertebrate, phastCons30way_mammalian, GERP++_RS, SiPhy_29way_logOdds |
| Population / MAF | gnomAD_exomes_AF, gnomAD_genomes_AF, 1000Gp3_AF, ExAC_AF, ESP6500_AA_AF, ESP6500_EA_AF |
| In Silico Scores | SIFT_score, Polyphen2_HDIV_score, Polyphen2_HVAR_score, CADD_phred, REVEL_score, MetaSVM_score, MutationTaster_score, FATHMM_score, VEST4_score, MVP_score, MPC_score, PrimateAI_score, DEOGEN2_score, BayesDel_addAF_score, ClinPred_score |
| ClinVar Info | clinvar_clnsig, clinvar_review, clinvar_trait |
| Amino Acid Info | aaref, aaalt, aapos, genename, Ensembl_proteinid |

### Custom engineering needed
| Feature | Source | Method |
|---------|--------|--------|
| delta_hydrophobicity | aaref, aaalt | Lookup table difference |
| delta_volume | aaref, aaalt | Lookup table difference |
| delta_charge | aaref, aaalt | Lookup table difference |
| delta_molecular_weight | aaref, aaalt | Lookup table difference |
| delta_polarity | aaref, aaalt | Lookup table difference |
| grantham_distance | aaref, aaalt | Grantham matrix lookup |
| local_aa_context | Ensembl_proteinid, aapos | Biopython/API fetch |
| local_nt_context | chr, pos | Reference genome fetch |
