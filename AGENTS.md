# Workspace Instructions

## Project Context

- Goal: Build a missense SNV classification pipeline as defined in PROJECT_PLAN.md.
- Primary deliverable: a single notebook (missense_classification.ipynb).

## Existing Files and Data

- Project plan: PROJECT_PLAN.md
- Prompt notes: _prompt.txt
- ClinVar VCF: clinvar_20260218.vcf
- Reference genome FASTA: Homo_sapiens.GRCh38.dna.primary_assembly.fa
- Protein FASTA (panel genes): idmapping_2026_03_16.fasta

## Data Usage Guidance

- Use idmapping_2026_03_16.fasta for amino-acid context extraction.
- Use Homo_sapiens.GRCh38.dna.primary_assembly.fa for nucleotide context extraction.
- Use Ensembl VEP REST API for fetching annotations and gnomAD frequencies since the sample size is small (~3000 variants); consume the JSON output in the notebook.

## Editing Guidance

- Keep changes consistent with PROJECT_PLAN.md.
- Do not introduce new data downloads unless explicitly requested.
- Use ASCII-only edits unless a file already contains Unicode.
