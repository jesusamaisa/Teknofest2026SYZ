# Project Brief: Missense Variant Classification

## Competition
**TEKNOFEST 2026 - Healthcare AI Competition (University Level)**

## Objective
Build a machine learning classifier that predicts whether missense variants are **Pathogenic** or **Benign** using features derived from the dbNSFP database and custom biochemical engineering.

## Data Source
- **Database**: dbNSFP (GRCh38 build)
- **Variant Type**: Single Nucleotide Variants (SNVs) only, missense subtype
- **Target Genes**: PAH (Phenylalanine Hydroxylase), CFTR (Cystic Fibrosis), and hereditary disease genes
- **Quality Filter**: Review status >= 3 stars (3 and 4 included, below 3 excluded)

## Target Variable (Binary Classification)
| Class | Includes |
|-------|----------|
| **Benign** | benign + likely-benign |
| **Pathogenic** | pathogenic + likely-pathogenic |
| **Removed** | VUS (Variants of Uncertain Significance) |

## Required Feature Categories
1. **Biochemical & Structural Effects** - delta Hydrophobicity, delta Volume, delta Charge, delta Molecular Weight, Polarity, Grantham Distance
2. **Sequence and Change Info** - amino acid substitution details, codon changes
3. **Local Sequence Context (Amino Acids)** - surrounding protein sequence window
4. **Local Sequence Context (Nucleotides)** - surrounding DNA sequence window
5. **Evolutionary Conservation** - conservation scores from dbNSFP
6. **Population Data / MAF** - allele frequency from population databases
7. **In Silico Risk Scores** - SIFT, PolyPhen, CADD, REVEL, etc.

## Critical Rules
- Keep `chr` and `pos` columns for merging, then **remove all genomic coordinate columns** after merge
- Analysis must be conducted in a `.ipynb` (Jupyter Notebook) file
- Only SNV + missense variants; all other types removed
- Only PAH, CFTR, and hereditary disease genes; all others removed

## Previous Work
- An earlier version existed using ClinVar VCF + Ensembl VEP annotation pipeline
- That version has been deleted (commit `fd3053f`) to start fresh with dbNSFP approach
- Git history preserves the old implementation for reference if needed

## Team
- Developer: Umut
- Branch: `main`
