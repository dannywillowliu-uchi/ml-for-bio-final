#Presentation Reference Sheet

## Breast Cancer Biomarker Prediction from RNA-seq Gene Expression

**Project:** BIOS 26122 Final Project | **Dataset:** GSE96058 SCAN-B cohort

---

## 1. The Story in 60 Seconds

We took gene expression data from 3,273 breast tumors (about 5,000 genes each) and asked: can machine learning predict a tumor's clinical biomarkers just from which genes are turned on or off? We tried three increasingly sophisticated approaches. First, we tested each gene individually (DEG analysis) and found over a thousand genes that differ between ER+ and ER- tumors. Then we used LASSO and Ridge regression, which look at all genes simultaneously and achieved over 97% AUC for ER status prediction -- with LASSO automatically narrowing down to just 162 key genes. Finally, we tried HistGradientBoosting, a nonlinear tree-based model that can capture gene-gene interactions, but it matched rather than beat LASSO. The punchline: the predictive signal lives in individual genes (especially ESR1, GATA3, FOXA1), not in complex gene interactions, and all three methods converge on the same biologically known markers -- strong evidence that our models are learning real biology, not noise.

---

## 2. The Three Methods (Plain Language)

### Method 1: Differential Expression (DEG) -- "Test one gene at a time"

**Analogy:** Imagine you have 5,000 students and you want to know which ones scored differently on a test between two schools. You compare each student's score individually with a t-test. If the difference is big enough and statistically significant, you flag that student.

**What we did:** For each of the 5,000 genes, we ran a t-test comparing expression levels in ER+ vs. ER- tumors. We kept genes that had both a large fold change (at least 2x difference) AND a significant p-value after correcting for multiple testing. This gave us 1,273 differentially expressed genes.

**Limitation:** This is naive because it treats each gene as if it exists in isolation. In reality, genes work together in pathways. A gene might not look important alone but could be critical in combination with others.

**Why we start here:** It is the simplest, most intuitive approach and gives us a biological baseline to compare against.

### Method 2: LASSO and Ridge Regression -- "Fit all genes at once"

**Analogy:** Imagine trying to predict house prices. Instead of checking one feature at a time (just bedrooms, just square footage), you build a single equation that weighs ALL features together. Ridge regression keeps all features but shrinks unimportant ones toward zero. LASSO goes further -- it actually sets unimportant features to exactly zero, effectively selecting only the genes that matter.

**What we did:** We fit logistic regression models with all 5,000 genes as predictors, using 10-fold cross-validation to tune the regularization strength. Ridge (L2 penalty) achieved AUC = 0.9748, and LASSO (L1 penalty) achieved AUC = 0.9710. LASSO kept only 162 out of 5,000 genes, with ESR1 having the largest coefficient.

**Why this is better than DEG:** It accounts for gene relationships. If two genes carry the same information, LASSO picks one and drops the other, giving you a cleaner, more interpretable model.

### Method 3: HistGradientBoosting -- "Capture gene-gene interactions"

**Analogy:** Think of it as building a decision tree flowchart -- "Is gene A high? If yes, check gene B. If B is also high, predict ER+." It builds hundreds of small trees, each one correcting the mistakes of the previous one. This lets it capture nonlinear patterns and interactions that linear models miss.

**What we did:** We trained HistGradientBoosting with hyperparameter tuning via grid search, on both the full 5,000-gene set and the DEG-selected subset.

**Key finding:** It matched LASSO's performance but did not beat it. This tells us something important: the decision boundaries between cancer subtypes are approximately linear in gene expression space. Gene-gene interactions do not add predictive value for these biomarkers. This is actually a useful result -- it means simpler, more interpretable linear models are the right tool for this problem.

---

## 3. Key Numbers to Know

| What | Value | Context |
|------|-------|---------|
| Total samples | 3,273 | Breast tumor RNA-seq profiles |
| Total genes | ~5,000 | After variance-based filtering |
| Train/test split | 80/20 | Stratified |
| **ER status -- Ridge CV AUC** | **0.9748** | Best single model for ER |
| ER status -- LASSO CV AUC | 0.9710 | Slightly lower but gives feature selection |
| ER status -- Test AUC | 0.9561 | Held-out test set |
| ER status -- Test Accuracy | 96.75% | Held-out test set |
| HER2 -- Best CV AUC (LASSO) | 0.9340 | Binary classification |
| HER2 -- Test AUC | 0.9117 | Held-out test set |
| NHG grade -- Best CV AUC (LDA) | 0.8579 | 3-class problem, hardest target |
| NHG -- Test AUC | 0.8399 | Held-out test set |
| PAM50 subtype -- Best CV AUC (LASSO) | 0.9836 | 5-class problem |
| PAM50 -- Test AUC | 0.9847 | Held-out test set |
| LASSO non-zero genes (ER) | 162 / 5,000 | ~3% of genes kept |
| LASSO non-zero genes (HER2) | 269 / 5,000 | ~5% of genes kept |
| DEGs identified (ER) | 1,273 | \|log2FC\| > 1, adj. p < 0.05 |
| Stable DEGs (cross-validated) | 1,123 | Present in all 5 CV folds |
| Permutation test p-value | < 0.001 (p = 0.0000) | 100 permutations, ER status |
| Observed AUC vs. permuted | 0.9604 vs. ~0.5 | Predictions far exceed chance |

---

## 4. Gene Names and What They Do

### ER Status Prediction Genes

| Gene | What It Does | Role in Our Model |
|------|-------------|-------------------|
| **ESR1** | Encodes the estrogen receptor alpha protein. The defining marker of ER+ breast cancer. | Largest positive LASSO coefficient (0.63). The single most predictive gene. |
| **GATA3** | Transcription factor that works with ESR1 to drive luminal cell identity. | Strong positive predictor of ER+ status. Co-regulator of estrogen signaling. |
| **FOXA1** | "Pioneer factor" that opens up DNA for ESR1 to bind. Required for estrogen receptor function. | Positive predictor. Part of the ESR1/GATA3/FOXA1 triad that defines luminal tumors. |
| **FOXC1** | Transcription factor associated with basal-like (ER-negative) breast cancer. | Negative coefficient -- high FOXC1 predicts ER-negative. |
| **KRT5** | Basal keratin protein. Structural protein in basal epithelial cells. | Negative coefficient -- marks basal-like, ER-negative tumors. |
| **KRT17** | Another basal keratin. Co-expressed with KRT5 in basal tumors. | Negative coefficient -- marks basal-like, ER-negative tumors. |

### HER2 Status Prediction Genes

| Gene | What It Does | Role in Our Model |
|------|-------------|-------------------|
| **ERBB2** | The HER2 gene itself. Encodes a growth factor receptor that is amplified in ~15-20% of breast cancers. | Largest positive LASSO coefficient (1.11). Directly measures the target. |
| **GRB7** | Sits next to ERBB2 on chromosome 17q12. Gets co-amplified when HER2 is amplified. | Second-largest positive predictor. Co-amplified with ERBB2. |

### NHG Grade Prediction Genes (Proliferation Markers)

| Gene | What It Does | Role in Our Model |
|------|-------------|-------------------|
| **MKI67** | Encodes Ki-67, a protein present only in dividing cells. Standard clinical proliferation marker. | High expression predicts high grade (grade 3). Consistent with grading criteria. |
| **TOP2A** | Topoisomerase II alpha. Essential for DNA replication during cell division. | Proliferation-associated gene. High in aggressive, fast-growing tumors. |

---

## 5. Attribution Map

| Contribution Area | Danny | Daniel |
|-------------------|-------|--------|
| All code and implementation | X | |
| Data download and preprocessing | X | |
| Exploratory data analysis (PCA, distributions) | X | |
| Model training (Ridge, LASSO, HGB, KNN, LDA) | X | |
| Feature selection pipeline (LASSO, DEG) | X | |
| Cross-validation and hyperparameter tuning | X | |
| Statistical testing (permutation, bootstrap) | X | |
| Visualizations (ROC curves, volcano plots, etc.) | X | |
| Background research on breast cancer biology | | X |
| Biological interpretation of results | | X |
| Conclusions and discussion writeup | | X |

---

## 6. Likely Q&A Questions and Answers

### Q1: "Why not just use DEG alone?"

DEG tests each gene independently, which ignores the relationships between genes. Two genes might both look significant individually, but they could be carrying the same information (correlated). DEG has no way to account for this. LASSO solves the problem by fitting all genes simultaneously -- if two genes are redundant, LASSO keeps one and drops the other. DEG also cannot directly make predictions on new samples; it just gives you a gene list. LASSO gives you a predictive model and a gene list at the same time.

### Q2: "Why does LASSO outperform other methods for feature selection?"

LASSO uses an L1 penalty, which has a mathematical property that drives coefficients to exactly zero. This is different from Ridge (L2), which shrinks coefficients toward zero but never reaches it. So LASSO naturally performs feature selection as part of model fitting -- you do not need a separate feature selection step. For our data, LASSO kept only 162 out of 5,000 genes for ER prediction while maintaining AUC of 0.97. Ridge actually had a slightly higher AUC (0.9748 vs. 0.9710), but LASSO's advantage is interpretability: you get a short list of named genes you can connect back to biology.

### Q3: "What does it mean that HistGradientBoosting didn't improve over LASSO?"

This is actually one of the most informative results in the project. HistGradientBoosting can capture nonlinear relationships and gene-gene interactions that linear models cannot. The fact that it matched but did not beat LASSO tells us the decision boundary between ER+ and ER- tumors is approximately linear in gene expression space. In practical terms: the cancer subtypes are distinguished by the expression levels of individual genes, not by complex interactions between genes. This validates using simpler, more interpretable linear models.

### Q4: "How do you know you're not overfitting?"

Three safeguards. First, we used 10-fold stratified cross-validation during training, so performance estimates are averaged across multiple train/test splits. Second, we held out 20% of the data as a completely separate test set that was never seen during training -- the test AUC of 0.9561 for ER is close to the CV AUC of 0.9748, showing good generalization. Third, we ran a permutation test (100 permutations), which shuffles the labels randomly and retrains the model. The permuted models scored around AUC = 0.5 (chance), while our real model scored 0.96, giving p < 0.001. This confirms the model is learning real signal, not memorizing noise.

### Q5: "What would you do differently or as next steps?"

Three things. First, validate on independent cohorts like TCGA or METABRIC -- our results are from a single dataset (SCAN-B), and external validation would strengthen confidence. Second, integrate clinical features (age, tumor size, nodal status) alongside gene expression, which could improve the harder targets like NHG grade (our weakest result at AUC 0.84). Third, explore pathway-level features using gene set enrichment, which could provide more biologically interpretable models than individual genes.

### Q6: "Why these specific biomarkers (ER, HER2, NHG, PAM50)?"

These are the biomarkers that actually drive treatment decisions in the clinic. ER status determines whether a patient gets hormonal therapy (tamoxifen). HER2 status determines whether they get targeted therapy (trastuzumab). NHG grade indicates how aggressive the tumor is. PAM50 subtype (Luminal A, Luminal B, HER2-enriched, Basal, Normal-like) is a molecular classification that captures tumor biology beyond traditional markers. Predicting these from RNA-seq has real clinical value because it could supplement or replace traditional immunohistochemistry assays.

### Q7: "Could this work on other cancer types?"

Likely yes for cancer types with strong transcriptional programs, like the luminal vs. basal distinction in breast cancer. The general approach (regularized regression on gene expression) is cancer-agnostic. However, the specific genes would be completely different -- ESR1 is relevant to breast cancer, not lung cancer. The method would need to be retrained from scratch on each cancer type. The key question is whether the target biomarker has a strong enough gene expression signature to be predictable.

### Q8: "What's the clinical relevance?"

Currently, ER and HER2 status are measured by immunohistochemistry (IHC) and FISH assays -- relatively cheap, well-established tests. So our model is not replacing those. The value is threefold: (1) RNA-seq is increasingly used in clinical genomics, so having a computational backup for biomarker calls adds a quality check; (2) it identifies which genes drive the predictions, informing biological understanding; (3) the approach could be extended to predict treatment response or patient outcomes, which IHC alone cannot do.

### Q9: "Why did you filter to ~5,000 genes? Could you miss important genes?"

We kept the top 5,000 genes by variance across samples. The rationale is that a gene with near-zero variance (same expression in all tumors) cannot distinguish cancer subtypes. This is a standard preprocessing step that reduces noise and computational cost. Could we miss a low-variance gene that is important? In theory yes, but in practice, the genes that distinguish cancer subtypes (ESR1, ERBB2, MKI67) are among the most variable genes in the dataset. The risk is low.

### Q10: "Why is NHG grade prediction the worst performing target?"

NHG (Nottingham Histologic Grade) is a 3-class problem (grades 1, 2, 3) assessed by pathologists looking at tissue morphology -- specifically tubule formation, nuclear pleomorphism, and mitotic count. It is inherently more subjective than ER or HER2, which have molecular definitions. Grade 2 is particularly fuzzy -- it is a "middle" category with significant inter-pathologist disagreement. Gene expression captures some of the biology (proliferation genes like MKI67 correlate with grade), but the subjective morphological component is harder to predict from expression alone. Our AUC of 0.84 is still a solid result for a 3-class problem with a noisy label.

---

## 7. Glossary

**AUC (Area Under the ROC Curve):** A measure of how well a classifier distinguishes between classes, ranging from 0.5 (random guessing) to 1.0 (perfect). It represents the probability that the model ranks a randomly chosen positive example higher than a randomly chosen negative example.

**Cross-validation (CV):** A technique for estimating model performance by splitting data into K equal parts ("folds"), training on K-1 folds, testing on the remaining fold, and rotating through all K folds. We used 10-fold CV. It prevents overfitting to a single train/test split.

**Regularization:** Adding a penalty to the model to prevent it from fitting noise. Without regularization, a model with 5,000 gene features and fewer samples would memorize the training data. The penalty forces the model to find simpler patterns.

**LASSO (Least Absolute Shrinkage and Selection Operator):** A regularization method that adds an L1 penalty (sum of absolute values of coefficients) to the loss function. This forces some coefficients to exactly zero, performing automatic feature selection.

**Ridge Regression:** A regularization method that adds an L2 penalty (sum of squared coefficients) to the loss function. This shrinks coefficients toward zero but never sets them exactly to zero. All features are kept but downweighted.

**L1 Penalty:** The sum of the absolute values of model coefficients. Used by LASSO. Produces sparse models (many coefficients = 0).

**L2 Penalty:** The sum of the squared values of model coefficients. Used by Ridge. Produces models where all coefficients are small but nonzero.

**Feature Selection:** The process of choosing a subset of relevant features (genes) for model building. LASSO does this automatically. DEG analysis is a separate, statistics-based approach to feature selection.

**Permutation Test:** A statistical test where you shuffle the outcome labels randomly and retrain the model many times. If the real model's performance is far better than the shuffled versions, the result is statistically significant (not due to chance).

**Volcano Plot:** A scatter plot showing fold change (x-axis) vs. statistical significance (y-axis) for each gene. Genes in the upper corners are both highly significant and strongly differentially expressed. Named because the shape looks like a volcanic eruption.

**DEG (Differentially Expressed Gene):** A gene whose expression level is significantly different between two groups (e.g., ER+ vs. ER- tumors). Identified using statistical tests (t-tests) with multiple testing correction.

**Fold Change:** The ratio of average expression between two groups. A log2 fold change of 1 means the gene is expressed 2x higher in one group. A log2 fold change of 5 (like ESR1) means 32x higher.

**FDR Correction (False Discovery Rate):** When you run thousands of statistical tests (one per gene), some will be significant by chance. FDR correction (e.g., Benjamini-Hochberg method) adjusts p-values to control the expected proportion of false positives among significant results.

**HistGradientBoosting:** A fast, tree-based ensemble method in scikit-learn. It builds many small decision trees sequentially, where each new tree focuses on correcting the errors of the previous trees. "Hist" means it bins continuous features into histograms for speed.

**PCA (Principal Component Analysis):** A dimensionality reduction technique that finds the directions of maximum variance in the data. It projects 5,000 gene dimensions down to a few "principal components" for visualization. PC1 and PC2 together captured enough variance to visually separate ER+ from ER- tumors in our data.
