# Final Project Proposal: Breast Cancer Biomarker Prediction from RNA-Seq Gene Expression

**Course:** BIOS 26122 - Introduction to Machine Learning for Biology (Winter 2026)

---

## 1. Dataset

**SCAN-B RNA-seq Breast Cancer Dataset (GSE96058)**

- **Source:** NCBI Gene Expression Omnibus
- **URL:** https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE96058
- **Samples:** 3,273 breast cancer tumor samples
- **Features:** RNA-seq gene expression (thousands of genes)
- **Technology:** Illumina HiSeq 2000 / NextSeq 500
- **Publication:** Brueffer et al., "Clinical Value of RNA Sequencing-Based Classifiers for Prediction of the Five Conventional Breast Cancer Biomarkers," JCO Precision Oncology, 2018 (DOI: 10.1200/PO.17.00135)

### Why This Dataset

- Large sample size (3,273) provides robust train/test splits
- High-dimensional gene expression features showcase regularization and feature selection
- Multiple clinically relevant classification targets
- Modern RNA-seq data (more relevant than older microarray platforms)
- Well-documented with peer-reviewed publication for benchmarking

---

## 2. Biological Questions

1. **Can we predict estrogen receptor (ER) status from gene expression?** (Binary classification)
2. **Can we predict histologic grade (NHG) from gene expression?** (Multi-class classification)
3. **Which genes are most predictive of cancer biomarker status?** (Feature selection / interpretation)
4. **How do different ML methods compare for gene expression classification?** (Model comparison)

---

## 3. Methods

We will apply the following methods covered in the course:

| Method | Application | Purpose |
|--------|-------------|---------|
| **KNN** | Classification of ER status | Baseline non-parametric classifier |
| **Logistic Regression** | Binary classification (ER, HER2) | Interpretable linear classifier |
| **LDA / QDA** | Multi-class grade prediction | Discriminant-based classification |
| **Ridge Regression** | Regularized classification | Handle multicollinearity in gene data |
| **LASSO** | Feature selection + classification | Identify key predictive genes |
| **K-Fold Cross-Validation** | All models | Robust performance estimation |
| **Bootstrap** | Confidence intervals on accuracy | Statistical inference on results |
| **Decision Trees** (if covered) | Classification | Interpretable non-linear model |

### Analysis Pipeline

1. **Data acquisition:** Download via GEOparse Python package
2. **Preprocessing:** Log-transform expression values, handle missing data, standardize features
3. **Exploratory analysis:** PCA visualization, class distribution plots, correlation analysis
4. **Feature filtering:** Remove low-variance genes, reduce to top ~1,000-5,000 informative features
5. **Model training:** Train all models with 10-fold CV for hyperparameter tuning
6. **Evaluation:** Accuracy, ROC-AUC, confusion matrices, comparison across methods
7. **Interpretation:** Extract LASSO-selected genes, discuss biological significance

---

## 4. Packages and Tools

- **Language:** Python
- **Data handling:** pandas, numpy, GEOparse
- **ML models:** scikit-learn (KNeighborsClassifier, LogisticRegression, LinearDiscriminantAnalysis, Ridge, Lasso, cross_val_score, GridSearchCV)
- **Visualization:** matplotlib, seaborn
- **Statistical testing:** scipy.stats (bootstrap, permutation tests)
- **Document format:** Quarto / Jupyter notebook

---

## 5. Visualizations and Deliverables

- PCA plot of samples colored by cancer subtype/ER status
- Heatmap of top differentially expressed genes
- ROC curves comparing all classification methods
- Bar chart of cross-validated accuracy across models
- LASSO coefficient path plot (genes selected vs. lambda)
- Confusion matrices for best-performing models
- Bootstrap confidence intervals on test accuracy

---

## 6. Performance Assessment

- **Primary metric:** ROC-AUC (handles class imbalance)
- **Secondary metrics:** Accuracy, precision, recall, F1-score
- **Validation strategy:** 10-fold stratified cross-validation on training set, held-out test set for final evaluation
- **Benchmark:** Compare against results reported in Brueffer et al. (2018)

---

## 7. Timeline

| Week | Task |
|------|------|
| Week 7 | Proposal submission, data download and exploration |
| Week 8 | Preprocessing, EDA, baseline models (KNN, logistic regression) |
| Week 9 | Regularization models (Ridge, LASSO), feature selection, advanced classifiers |
| Week 10 | Final evaluation, visualizations, report writing, presentation prep |
