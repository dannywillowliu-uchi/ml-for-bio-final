# Breast Cancer Gene Expression ML Project - Presentation Slides

**Duration:** 10 minutes
**Authors:** Danny Liu, Daniel Gong

---

## Slide 1: Title Slide

**Content:**
- **Title:** Predicting Breast Cancer Biomarkers from Gene Expression Using Machine Learning
- **Authors:** Danny Liu & Daniel Gong
- **Course:** [Course Name / Number]
- **Date:** [Presentation Date]

**Speaker Notes:**
Welcome everyone. Our project is titled "Predicting Breast Cancer Biomarkers from Gene Expression Using Machine Learning." The central question we set out to answer is: can we use computational methods to predict clinically important breast cancer markers directly from gene expression data, and more importantly, can we identify which specific genes are driving those predictions? My name is Danny Liu and I worked on the code implementation, data processing, and model training. My partner Daniel Gong handled the background research, biological interpretation, and conclusions. Over the next ten minutes, we will walk you through our data, three progressively more sophisticated analytical methods, and what we learned from comparing them. Let us start with some background.

---

## Slide 2: Background & Research Question

**Content:**
- Breast cancer is classified by molecular biomarkers: ER (estrogen receptor), HER2, histological grade (NHG), and PAM50 intrinsic subtype
- These biomarkers guide treatment decisions (e.g., hormone therapy for ER+ tumors)
- Gene expression profiling measures the activity of thousands of genes simultaneously
- **Research Question:** Can we predict these biomarkers from gene expression data, and which genes matter most?
- **Why it matters:** Understanding the gene-level drivers behind biomarker status could improve diagnostics and reveal biological mechanisms

**Speaker Notes:**
To give some context, breast cancer is not a single disease. It is classified into subtypes based on molecular biomarkers. ER status tells us whether a tumor has estrogen receptors, which determines whether hormone therapy will be effective. HER2 status indicates overexpression of the HER2 protein, which has its own targeted therapies. NHG is a histological grading system from 1 to 3 that measures how abnormal tumor cells look under a microscope. PAM50 is a gene-expression-based classification that assigns tumors to one of several intrinsic subtypes like Luminal A, Luminal B, Basal, and HER2-enriched. Our research question is straightforward: given that we can measure the expression levels of thousands of genes in a tumor, can we build models that predict these clinical biomarkers? And critically, which genes does each model rely on to make those predictions? If multiple independent methods all point to the same genes, that gives us strong evidence that those genes are genuinely important, not just statistical artifacts. Next, let us look at the dataset we used.

---

## Slide 3: The Data (SCAN-B Cohort)

**Content:**
- **Dataset:** GSE96058 from the SCAN-B (Sweden Cancerome Analysis Network - Breast) initiative
- **Size:** 3,273 primary breast tumor samples
- **Features:** ~5,000 genes retained after variance-based filtering (originally ~20,000+)
- **Targets:** ER status (binary), HER2 status (binary), NHG grade (ordinal 1-3), PAM50 subtype (multiclass)
- **Class balance:** ER+ tumors are the majority (~80%), ER- is the minority (~20%)
- **Preprocessing:** Log2 transformation, gene filtering by expression variance, standardization

**Speaker Notes:**
Our dataset comes from the SCAN-B initiative, which is a large-scale Swedish study that profiled thousands of breast tumors using RNA sequencing. The specific dataset is GSE96058, publicly available on the Gene Expression Omnibus. It contains 3,273 tumor samples, each with expression measurements for tens of thousands of genes. We filtered this down to approximately 5,000 genes by removing those with very low variance across samples, since low-variance genes carry little discriminative information. Our primary prediction target is ER status because it has a clean binary split and is the most clinically actionable. However, we also tested our models on HER2 status, NHG grade, and PAM50 subtype to see how well the approach generalizes. One thing to note is the class imbalance: roughly 80% of tumors are ER-positive and 20% are ER-negative, which means a naive classifier that always predicts ER+ would get 80% accuracy. That is why we use AUC rather than accuracy as our primary metric. Before we get into modeling, let us look at some exploratory data analysis.

---

## Slide 4: EDA - PCA Shows ER+/ER- Separation

**Content:**
- PCA (Principal Component Analysis) plot of all 3,273 tumors, colored by ER status
- PC1 and PC2 capture the dominant variation in gene expression
- Clear visual separation between ER+ and ER- clusters along PC1
- This suggests the ER signal is strong and should be learnable by ML models
- [Show PCA scatter plot figure]

**Speaker Notes:**
Before building any predictive models, we performed exploratory data analysis. This PCA plot projects all 3,273 tumors from the ~5,000-dimensional gene expression space down to two dimensions. Each dot is a tumor, colored by ER status: one color for ER-positive, another for ER-negative. What you can see is a clear separation between the two groups, primarily along the first principal component. This is a very encouraging sign because it tells us that the gene expression differences between ER-positive and ER-negative tumors are large enough to dominate the overall variation in the dataset. If there were no separation here, we would expect our models to struggle. The fact that we see this clean split in just the first two components suggests that even relatively simple models should be able to classify ER status accurately. It also hints that a relatively small number of genes are driving most of this separation. With this motivation, let us now walk through three progressively more sophisticated methods for identifying which genes matter.

---

## Slide 5: Method 1 - Differential Gene Expression (DEG) Analysis

**Content:**
- **Approach:** Test each gene independently using a t-test (ER+ vs. ER-)
- **Visualization:** Volcano plot with log2 fold-change (x-axis) vs. -log10 p-value (y-axis)
- Genes in upper-left and upper-right corners are significantly differentially expressed
- **Thresholds:** |log2 FC| > 1 and adjusted p-value < 0.05
- **Top genes identified:** ESR1, GATA3, FOXA1, ERBB2, among others
- **Limitation:** Tests genes one at a time, ignoring correlations and interactions between genes
- [Show volcano plot figure]

**Speaker Notes:**
Our first method is differential gene expression analysis, or DEG. This is the classical bioinformatics approach. For each of the ~5,000 genes, we run an independent t-test comparing its expression in ER-positive tumors versus ER-negative tumors. The result is a volcano plot, which you see here. The x-axis shows the log2 fold-change, meaning how much more or less a gene is expressed in ER+ versus ER-. The y-axis shows the negative log10 of the p-value, so genes higher up are more statistically significant. Genes that appear in the upper-right are significantly upregulated in ER+ tumors, and genes in the upper-left are significantly downregulated. We apply standard thresholds: an absolute fold-change greater than 1 and an adjusted p-value below 0.05. This identifies genes like ESR1, which is the estrogen receptor gene itself, GATA3 and FOXA1, which are known transcription factors in estrogen signaling. The critical limitation of DEG is that it tests each gene in isolation. If two genes carry redundant information, DEG will flag both of them as significant. It cannot distinguish between a gene that provides unique predictive value and one that is merely correlated with a truly important gene. To address this, we need methods that consider all genes simultaneously, which brings us to LASSO regression.

---

## Slide 6: Method 2 - LASSO Logistic Regression

**Content:**
- **Approach:** Fit a logistic regression with ALL ~5,000 genes as features simultaneously
- **LASSO (L1 regularization):** Penalizes the sum of absolute coefficients, driving irrelevant gene weights to exactly zero
- **Ridge (L2 regularization):** Penalizes the sum of squared coefficients, shrinks all weights but none to zero
- **Performance:** Cross-validated AUC > 0.97 for ER status prediction
- **Key advantage:** Automatic feature selection -- only ~50-100 genes retain nonzero coefficients
- **Top genes by LASSO coefficient magnitude:** ESR1, GATA3, FOXA1, NAT1, AGR2
- [Show LASSO coefficient path plot and/or bar chart of top gene coefficients]

**Speaker Notes:**
Our second method is LASSO logistic regression, which represents a major step up from DEG. Instead of testing genes one at a time, LASSO fits a single model that uses all ~5,000 genes as input features simultaneously. The key idea behind LASSO is L1 regularization: it adds a penalty equal to the sum of the absolute values of all model coefficients. As you increase this penalty, genes that contribute less to prediction have their coefficients shrunk to exactly zero. This means LASSO performs automatic feature selection, keeping only the genes that provide unique, non-redundant predictive value. By contrast, Ridge regression uses L2 regularization, which penalizes the sum of squared coefficients. Ridge shrinks everything toward zero but never sets anything to exactly zero, so it does not perform feature selection. We tested both. In cross-validation, LASSO achieves an AUC above 0.97 for predicting ER status, which is excellent. The top genes by LASSO coefficient magnitude are ESR1, GATA3, FOXA1, NAT1, and AGR2. Notice that several of these overlap with what DEG found, but LASSO gives us a cleaner, more parsimonious set because it accounts for correlations between genes. If two genes carry the same information, LASSO will keep one and drop the other. The natural next question is: are there nonlinear interactions between genes that a linear model like LASSO cannot capture? That leads us to our third method.

---

## Slide 7: Method 3 - HistGradientBoosting

**Content:**
- **Approach:** Histogram-based gradient boosting (tree ensemble), a nonlinear method
- Builds hundreds of shallow decision trees sequentially, each correcting the errors of the previous
- Can capture gene-gene interactions (e.g., gene A matters only when gene B is high)
- **Performance:** Cross-validated AUC ~0.97 for ER status -- matches LASSO but does NOT outperform it
- **Interpretation:** This suggests the ER signal is predominantly linear (individual gene effects), not driven by complex gene-gene interactions
- **Feature importance:** Permutation-based importance identifies ESR1, GATA3, FOXA1 as top features
- [Show feature importance bar chart]

**Speaker Notes:**
Our third and most complex method is HistGradientBoosting, which is scikit-learn's implementation of histogram-based gradient boosted decision trees. Unlike LASSO, this is a nonlinear model. It builds an ensemble of hundreds of shallow decision trees, where each tree is trained to correct the mistakes of the trees that came before it. The key advantage of tree-based models is that they can naturally capture interaction effects between genes. For example, maybe gene A only matters for ER prediction when gene B is expressed above a certain level. LASSO would miss this entirely because it can only model additive linear effects. So what happened? HistGradientBoosting achieved a cross-validated AUC of approximately 0.97, which matches LASSO but does not exceed it. This is actually a very informative result. If gene interactions were important for predicting ER status, the nonlinear model should have outperformed the linear one. The fact that it did not tells us that the ER signal is predominantly captured by individual gene effects, not by complex combinatorial patterns. When we look at permutation-based feature importance, the same genes rise to the top: ESR1, GATA3, FOXA1. This convergence across all three methods is the most important finding in our project, which we will discuss next.

---

## Slide 8: Results - Cross-Method Performance Comparison

**Content:**
- Bar chart comparing AUC across all three methods for ER status:
  - DEG-selected genes + simple classifier: baseline reference
  - LASSO Logistic Regression: AUC > 0.97
  - Ridge Logistic Regression: AUC > 0.97
  - HistGradientBoosting: AUC ~0.97
- Generalization to other targets:
  - HER2 status: strong performance (high AUC)
  - PAM50 subtype: good multiclass performance
  - NHG grade: lower performance (ordinal target, more ambiguous boundaries)
- **Permutation test:** p < 0.001 confirms predictions are not due to chance
- [Show grouped bar chart of AUC by method and target]

**Speaker Notes:**
This slide summarizes performance across all methods and prediction targets. For ER status, which was our primary focus, all methods achieve AUC values above 0.97. This is near-perfect discrimination. The permutation test, where we shuffled the ER labels 1,000 times and re-evaluated, confirms a p-value below 0.001, meaning this performance is not an artifact of overfitting or chance. When we generalize to other targets, performance varies. HER2 status also achieves high AUC, which makes sense because HER2 overexpression has a clear molecular signature driven largely by the ERBB2 gene. PAM50 subtype classification performs well in the multiclass setting, which is expected since PAM50 subtypes are themselves defined by gene expression patterns. NHG grade shows lower performance. This is biologically reasonable because histological grading is a subjective assessment by pathologists and captures tissue-level morphological features that may not map as directly to individual gene expression levels. The key takeaway from this slide is that regularized linear models match or rival the nonlinear approach, suggesting that for gene expression based biomarker prediction, complexity beyond linear models is often unnecessary. Now let us look at the most important finding.

---

## Slide 9: Key Finding - Gene Overlap Across All Three Methods

**Content:**
- Venn diagram or overlap table showing top genes identified by each method:
  - **DEG:** ESR1, GATA3, FOXA1, ERBB2, NAT1, ... (many genes, broad list)
  - **LASSO:** ESR1, GATA3, FOXA1, NAT1, AGR2 (sparse, selective list)
  - **HistGradientBoosting:** ESR1, GATA3, FOXA1, ERBB2 (importance-ranked list)
- **Core convergent genes:** ESR1, GATA3, FOXA1 appear across all three methods
- These are biologically validated: ESR1 encodes the estrogen receptor itself; GATA3 and FOXA1 are known co-regulators in ER signaling
- **Implication:** When independent methods agree, the signal is real, not a statistical artifact
- [Show Venn diagram or heatmap of gene rankings across methods]

**Speaker Notes:**
This is the central finding of our project and arguably the most important slide. When we compare the top genes identified by each of our three methods, a clear pattern emerges: ESR1, GATA3, and FOXA1 appear at or near the top of every single method. ESR1 is the gene that encodes the estrogen receptor protein itself, so finding it as the top predictor of ER status is a direct validation that our models are learning real biology and not noise. GATA3 and FOXA1 are transcription factors that are well-documented co-regulators of estrogen receptor signaling in the breast cancer literature. The fact that three fundamentally different approaches -- a univariate statistical test, a regularized linear model, and a nonlinear tree ensemble -- all converge on the same small set of genes is powerful evidence that these genes are genuinely driving the ER phenotype. DEG casts the widest net and flags many genes, some of which may be redundant. LASSO narrows the list by accounting for inter-gene correlations. HistGradientBoosting confirms the same genes matter even when nonlinear effects are allowed. This multi-method convergence is a useful validation strategy that could be applied to other genomic prediction problems. Let us wrap up with our conclusions.

---

## Slide 10: Conclusions & Future Directions

**Content:**
- **Conclusions:**
  - Gene expression reliably predicts breast cancer biomarkers (AUC > 0.97 for ER)
  - LASSO provides the best balance of performance and interpretability
  - Nonlinear models (HistGradientBoosting) do not outperform linear models for this task
  - Cross-method gene convergence (ESR1, GATA3, FOXA1) confirms genuine biological signal
- **Future Directions:**
  - Apply to external validation cohorts (e.g., TCGA, METABRIC)
  - Explore deep learning approaches (autoencoders for latent gene representations)
  - Extend to survival prediction (time-to-event outcomes)
  - Investigate the small set of genes where methods disagree
- **Acknowledgments:** [Course name, instructor, data source (GEO: GSE96058)]

**Speaker Notes:**
To summarize, we demonstrated that machine learning models can predict breast cancer biomarkers from gene expression with very high accuracy. The most practically useful model is LASSO logistic regression because it achieves top-tier performance while simultaneously performing feature selection, giving us a short, interpretable list of important genes. The fact that HistGradientBoosting does not improve over LASSO tells us that gene-gene interactions are not a major factor for these prediction tasks, at least in this dataset. Our strongest finding is the cross-method convergence on core genes like ESR1, GATA3, and FOXA1, which provides confidence that the models are capturing real biological mechanisms. For future work, validating on independent cohorts like TCGA or METABRIC would strengthen these findings. Deep learning methods like autoencoders could uncover latent gene expression patterns. And extending the approach to survival prediction would move closer to clinical utility. Thank you for your attention. We are happy to take questions. Danny Liu handled all code implementation, data processing, and model training. Daniel Gong contributed the background research, biological interpretation, and conclusions.

---
