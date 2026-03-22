# Missense Variant Classification

A machine-learning pipeline for classifying missense single-nucleotide variants (SNVs) as **Pathogenic** or **Benign** for hereditary disease genes (PAH, CFTR, and a hereditary cancer gene panel). This project was developed for the TEKNOFEST 2026 Health in AI competition.

## Project Overview

The core deliverable of this project is a single Jupyter Notebook (`missense_classification.ipynb`) that handles data acquisition, preprocessing, feature extraction, and model training.

The pipeline integrates multiple data sources to extract robust features for variant classification:
- **ClinVar**: Ground truth labels (Pathogenic vs Benign) and variant metadata.
- **Ensembl VEP & dbNSFP**: In silico risk scores (SIFT, PolyPhen, CADD, REVEL, etc.) and evolutionary conservation metrics.
- **Reference Genome (GRCh38)**: Nucleotide sequence context extraction.
- **UniProt/SwissProt (idmapping)**: Amino acid context extraction.

## Project Structure

- `missense_classification.ipynb`: The main execution notebook containing the entire pipeline.
- `PROJECT_PLAN.md`: Detailed plan, feature logic, and dataset processing rules.
- `AGENTS.md` & `PROGRESS.md`: AI Agent workspace instructions and progress tracking.
- `data/`: Directory for intermediate processed data (filtered VCFs, VEP outputs, etc.).

## Setup & Requirements

The notebook requires:
- Python 3.x
- Jupyter Notebook or JupyterLab
- Standard data science libraries (Pandas, Scikit-learn, PySAM, etc.)
- Internet access for Ensembl VEP REST API (for variant annotation and frequencies).

Basic sequence dependencies (FASTA files and ClinVar VCFs) should be placed in the project root or downloaded directly within the notebook as specified in the execution steps.
