# Predicting Breast Cancer Biomarkers from RNA-seq Gene Expression

**Authors:** Danny Liu and Daniel Gong
**Course:** BIOS 26122 - Introduction to Machine Learning for Biology
**Dataset:** [GSE96058](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE96058) - SCAN-B breast cancer cohort (3,273 primary tumors)

## Overview

This project predicts clinically important breast cancer biomarkers (ER status, HER2 status, NHG grade, PAM50 subtype) directly from RNA-seq gene expression data using three ML approaches:

1. **Regularized Logistic Regression (LASSO/Ridge)** - joint linear modeling with feature selection
2. **Differential Gene Expression (DEG)** - classical single-gene t-tests with FDR correction
3. **HistGradientBoosting** - nonlinear tree-based ensemble to test for gene-gene interactions

Key finding: linear models achieve CV AUC > 0.97 for ER status, and nonlinear models do not improve upon them, indicating the biomarker boundaries in expression space are approximately linear.

## Getting Started

### Requirements

```
pandas numpy matplotlib seaborn scipy scikit-learn statsmodels
```

Install with:
```bash
pip install pandas numpy matplotlib seaborn scipy scikit-learn statsmodels
```

### Running the Notebook

1. Clone this repository
2. Open `final_project.ipynb` in Jupyter
3. Run all cells - the notebook will automatically download the gene expression data (~564MB) from GEO on first run

The phenotype data (`data/phenotype_data.csv`) is included in the repository. The gene expression matrix is downloaded automatically from [NCBI GEO](https://ftp.ncbi.nlm.nih.gov/geo/series/GSE96nnn/GSE96058/suppl/).

## Repository Structure

```
final_project.ipynb      # Main analysis notebook (run this)
data/
  phenotype_data.csv     # Clinical annotations (included, 472KB)
docs/
  final_report.html      # Rendered HTML report
  presentation_slides.md # Slide content for presentation
```

## References

1. Brueffer et al. (2018). Clinical Value of RNA Sequencing-Based Classifiers for Prediction of the Five Conventional Breast Cancer Biomarkers. *JNCI*.
2. Parker et al. (2009). Supervised Risk Predictor of Breast Cancer Based on Intrinsic Subtypes. *JCO*.
3. Hastie, Tibshirani, Friedman. *The Elements of Statistical Learning*. Springer.
4. James et al. *An Introduction to Statistical Learning*. Springer.
