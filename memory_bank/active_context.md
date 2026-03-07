# Active Context — Quick Reference

> **Read this file first** when resuming work on this project.

## What is this project?
A TEKNOFEST 2026 competition entry: ML classifier for missense variants (Pathogenic vs Benign) using dbNSFP database (GRCh38).

## Current State (Last updated: 2026-03-06)
- **Status**: Full pipeline notebook (`main.ipynb`) created with all 6 phases.
- **Data Strategy**: Uses **myvariant.info API** to query dbNSFP fields on demand (no 30GB download).
- **Working files**: `main.ipynb` (complete pipeline), `main_vcf.ipynb` (old VCF approach, kept for reference).
- **Next step**: Run the notebook end-to-end. Fix any runtime issues. May need to adjust column names based on actual API response structure.

## Memory Bank Files
| File | Purpose |
|------|---------|
| `active_context.md` | THIS FILE — quick resume reference |
| `project_brief.md` | Competition goals, data sources, requirements |
| `implementation_plan.md` | Full 6-phase step-by-step plan |
| `technical_decisions.md` | Architecture, decision log, dbNSFP column mapping |
| `progress.md` | Detailed task checklist and session log |

## Key Requirements (Quick Recap)
1. **Database**: dbNSFP GRCh38 (NOT ClinVar VCF)
2. **Filters**: SNV + missense only, PAH/CFTR/hereditary genes, review status >= 3
3. **Features**: Biochemical deltas, Grantham, sequence context (AA + NT), conservation, MAF, in silico scores
4. **Target**: Binary (Benign vs Pathogenic), VUS removed
5. **Critical**: Remove all chr/pos columns AFTER merging
6. **Output**: Jupyter Notebook (`.ipynb`)

## Open Decisions
- Exact list of "hereditary" genes
- Feature leakage handling (meta-predictor scores)
- Class imbalance strategy
