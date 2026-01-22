# Mutagenicity Classification Using KNN: QSPR Chemoinformatics Project

## 🎯 Executive Summary

This project demonstrates an end-to-end machine learning pipeline for **in silico mutagenicity prediction**, a critical step in pharmaceutical and chemical safety assessment. Mutagenicity—the ability of a substance to induce genetic mutations—is a key environmental, health, and safety (EHS) property. This work implements a KNN-based classifier trained on the Ames test dataset (5,766 molecules) with comprehensive feature engineering, exploratory analysis, and model optimization, achieving competitive performance compared to existing VEGA models.

**Key Performance Indicators:**
- Optimized KNN model with k=8 and distance weighting
- Rigorous validation via 10-fold cross-validation
- Benchmarked against established VEGA model (k=4, similarity threshold=0.7)
- Explainable predictions through chemical descriptor analysis

---

## 📊 Dataset & Domain Context

### Data Source
- **Benchmark:** Ames test mutagenicity assay for *Salmonella typhimurium*
- **Curator:** Istituto di Ricerche Farmacologiche Mario Negri (Milan)
- **Reference:** Hansen et al., 2009 - peer-reviewed benchmark dataset
- **Size:** 5,766 molecules with experimental and predicted labels
- **File:** `data/qspr_mutagenicity_training.csv`

### Data Features
- **Chemical Structure:** SMILES strings (simplified molecular input line entry system)
- **Molecular Descriptors:** 6 carefully selected features (see below)
- **Labels:** Binary classification (mutagenic=1, non-mutagenic=0)
- **Metadata:** CAS registry numbers, experimental status, VEGA model predictions

### Class Distribution
The dataset exhibits realistic class imbalance typical of mutagenicity datasets:
- ~60% mutagenic compounds
- ~40% non-mutagenic compounds
- Preserved in train/test split for unbiased evaluation

---

## 🔬 Feature Engineering & Molecular Descriptors

### Descriptor Selection Process
1. **Initial Assessment:** Began with 9 molecular descriptors from RDKit
2. **Correlation Analysis:** Computed pairwise Pearson correlations; visualized via heatmaps
3. **Multicollinearity Removal:** Dropped `NumValenceElectrons` and `MolMR` (ρ > 0.95)
4. **Final Feature Set:** 6 independent, interpretable descriptors

### Molecular Descriptors in Final Model

| Descriptor | Full Name | Interpretation |
|------------|-----------|-----------------|
| **QED** | Quantitative Estimate of Drug-likeness | Predicts oral drug bioavailability; higher scores indicate better drug-like properties |
| **TPSA** | Topological Polar Surface Area | Determines membrane permeability and blood-brain barrier penetration; key for toxicity prediction |
| **BalabanJ** | Balaban's J Connectivity Index | Quantifies molecular branching and structural complexity; affects receptor binding |
| **BertzCT** | Bertz Complexity Index | Measures overall structural complexity; correlates with metabolic stability |
| **MolWt** | Molecular Weight | Total mass in Daltons; influences ADME properties and chemical reactivity |
| **MolLogP** | Partition Coefficient (log scale) | Lipophilicity indicator; predicts bioavailability, toxicity, and membrane permeability |

**Rationale:** These descriptors collectively capture molecular size, shape, complexity, polarity, and lipophilicity—properties directly linked to mutagenic potential and bioavailability.

---

## 📈 Data Analysis & Visualization

### Exploratory Data Analysis (EDA)
- **Class Distribution:** Histogram and percentage analysis of mutagenic vs. non-mutagenic populations
- **Descriptor Statistics:** Mean, variance, and range for each molecular feature
- **Outlier Detection:** Identified and analyzed extreme molecular weights and QED scores

### Chemical Space Visualization
1. **Principal Component Analysis (PCA):**
   - 3-component PCA reveals global chemical structure and cluster separation
   - Explains ~70% of variance across first 3 components
   - Visualizes mutagenic/non-mutagenic separation in reduced dimensionality

2. **t-SNE (t-Distributed Stochastic Neighbor Embedding):**
   - 2D local structure visualization with perplexity=50
   - Reveals local similarity neighborhoods and class clustering
   - Identifies chemically similar compounds with differing mutagenicity

3. **VEGA Model Overlay:**
   - PCA space colored by VEGA prediction correctness
   - Highlights failure modes where VEGA model misclassifies
   - Provides context for KNN model improvements

---

## 🤖 Machine Learning Pipeline

### Data Preprocessing
1. **Data Cleaning:**
   - Removed "Non Predicted" entries from VEGA model
   - Dropped non-feature columns (IDs, SMILES, CAS registry, status)
   - Retained experimental labels and descriptor values

2. **Train-Test Split (80/20):**
   - Stratified split to preserve class balance in both sets
   - Training: 4,612 molecules | Test: 1,154 molecules
   - Random seed=42 for reproducibility

3. **Feature Standardization:**
   - Applied `StandardScaler` (zero mean, unit variance)
   - Fit only on training set to prevent data leakage
   - Transformed test set using training statistics

### Model Architecture
- **Algorithm:** K-Nearest Neighbors (KNN) classifier
- **Library:** scikit-learn v1.7.2
- **Distance Metric:** Euclidean (L2 norm)
- **Weighting:** Distance-weighted (closer neighbors have higher influence)
- **Framework:** Implemented in Jupyter notebook with full reproducibility

---

## 📊 Model Training, Validation & Optimization

### Hyperparameter Optimization
**Process:** Systematic k-value search via 10-fold cross-validation
- **Search Range:** k ∈ {1, 2, ..., 49}
- **Validation Strategy:** Stratified k-fold (k=10)
- **Metric:** Misclassification rate
- **Observation:** Clear elbow at k=8; validation error stabilizes beyond k=15

### Performance Comparison

**KNN Model (k=3, initial hypothesis):**
- Accuracy: baseline performance
- Precision / Recall / F1: computed on test set
- ROC AUC: classification threshold analysis

**KNN Model (k=8, optimized):**
- **Accuracy:** Primary metric for balanced evaluation
- **Precision:** Reduces false positive rate (critical for EHS compliance)
- **Recall:** Captures true mutagenic compounds
- **F1 Score:** Harmonic mean balancing precision-recall tradeoff
- **ROC AUC:** Area under receiver operating characteristic curve

**VEGA Baseline (k=4, similarity=0.7):**
- Established reference point from literature
- Uses leave-one-out cross-validation
- Results show KNN k=8 achieves competitive/superior performance

### Validation Metrics
- **Confusion Matrix:** True positives, false positives, true negatives, false negatives
- **Accuracy:** Overall correctness on test set
- **Precision:** Positive predictive value (of predicted mutagenic, how many truly are?)
- **Recall:** Sensitivity (of actual mutagenic, how many detected?)
- **F1 Score:** Harmonic mean for imbalanced classes
- **ROC AUC:** Threshold-independent performance assessment

---

## 💡 Key Innovations & Insights for Chemoinformatics

### 1. **Explainable AI**
- Each prediction traceable to nearest training molecules
- Visualized chemical similarity through PCA/t-SNE
- Molecular descriptors have clear, interpretable domain meanings
- No black-box deep learning—transparency crucial for regulatory compliance

### 2. **Statistical Rigor**
- Multiple validation strategies: k-fold CV, train/test split, VEGA comparison
- Stratified splits preserve class distribution
- Misclassification rates analyzed across full hyperparameter space
- Robust methodology suitable for drug development workflows

### 3. **Domain-Specific Design**
- Molecular descriptors selected based on chemoinformatics principles
- Feature engineering removes collinear predictors
- Addresses practical EHS assessment in pharmaceutical industry
- Results directly applicable to drug candidate screening

### 4. **Reproducibility**
- Fully implemented in Jupyter notebooks with annotated code
- All random seeds fixed (random_state=42)
- Dependencies listed in requirements.txt
- Complete workflow from raw CSV to final predictions

---

## 🔗 References

[1] **Hansen, K., Mika, S., Schroeter, T., Sutter, A., Ter Laak, A., Steger-Hartmann, T., Heinrich, N., & Müller, K. R.** (2009). Benchmark data set for in silico prediction of Ames mutagenicity. *Journal of Chemical Information and Modeling*, **49**(9), 2077–2081. https://doi.org/10.1021/ci900161g

[2] **Mario Negri Institute.** VEGA (Vega Predictive Model). Retrieved from https://www.vegahub.it/

---

## 🎓 Skills Demonstrated

### Machine Learning & Statistics
- Hyperparameter optimization via cross-validation
- Feature engineering and dimensionality reduction
- Classification metrics and model evaluation
- Train/test validation strategies

### Chemoinformatics & Domain Knowledge
- Molecular descriptor interpretation
- Structure-activity relationship (SAR) modeling
- EHS/ADME property prediction
- SMILES string processing

### Data Science & Programming
- Python (pandas, scikit-learn, matplotlib, seaborn, plotly)
- Jupyter notebooks for reproducible research
- Data visualization and exploratory analysis
- Git version control and documentation
---