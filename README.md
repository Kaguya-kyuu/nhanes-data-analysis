# 🩺 NHANES Health Data Analysis  
**Multivariate Modeling • Robust Statistics • Clustering • ICA • PCA • GMM**

This repository contains a full statistical analysis of latent health structures in the U.S. adult population using NHANES 2013–2014 data (n = 4,931). The project integrates classical and modern multivariate methods—including PCA, Factor Analysis, ICA, and Gaussian Mixture Models—to uncover physiological dimensions and population subgroups.

> “This analysis aimed to identify the underlying structure of health indicators and their relationship with demographic factors in the NHANES dataset.”

---

## 📌 Project overview

The dataset combines **demographic**, **examination**, and **laboratory** files from NHANES, including:

- **10 health indicators:** height, weight, waist circumference, BMI, systolic/diastolic blood pressure, total cholesterol, HDL cholesterol, creatinine, glycohemoglobin  
- **5 demographic variables:** age, gender, race/ethnicity, education, income  
- **Respondents:** 5,367 adults (18–80), with **4,931 complete cases** used for analysis  

Central research questions:

1. **What latent structures or patterns exist among health indicators?**  
2. **How do these health indicators relate to demographic features (age, SES, etc.)?**

---

## 🧹 Data inspection and preprocessing

- **Missingness:**  
  - Lab variables: ~3.5–5% missing  
  - Demographics (education, income): ~5.5–7.5% nonresponse  
  - Analysis uses **complete cases (n = 4,931)** to maintain a coherent covariance structure, with the caveat of reduced generalizability.

- **Outliers:**  
  - Multivariate outliers detected via **Mahalanobis distance** with \(\chi^2_{0.975, p=16}\), yielding **345** outliers.  
  - Variable-wise outliers via **z-score > 4**, with **glycohemoglobin** having the most (79).  
  - Outliers are **retained** to preserve population heterogeneity and motivate robust methods (e.g., MCD).

> “The presence of numerous outliers also motivates the potential use of robust techniques, such as Minimum Covariance Determinant (MCD), in later analyses…”

- **Standardization:**  
  - All numerical health indicators are standardized prior to multivariate analysis to avoid scale dominance.

---

## 📊 Exploratory analysis

### Boxplots (Figure 1)

- Weight, waist, BMI, systolic BP, total cholesterol, and HDL cholesterol show **right-skewed distributions** with many high-value outliers, reflecting obesity and elevated cholesterol in the population.  
- Creatinine and glycohemoglobin show **smaller variance**, indicating more homogeneous distributions.  
- Skewness and heterogeneity suggest **latent structure**, motivating PCA, FA, and ICA.

> “The presence of outliers and heteroskedasticity also indicates deviations from multivariate normality…”

### Correlation structure (Figure 2)

- **Strong body-measurement block:**  
  - Weight, waist, BMI: correlations **> 0.85**, indicating a shared **body composition** dimension.  
- **Blood pressure:**  
  - Systolic vs diastolic BP: moderate correlation (~0.35), suggesting partially distinct processes.  
- **HDL cholesterol:**  
  - Negatively correlated with body measurements, consistent with obesity–HDL patterns.  
- **Creatinine & glycohemoglobin:**  
  - Weak or near-zero correlations with most variables → **independent sources of variation**.

This mixed structure—strongly correlated blocks plus nearly independent variables—supports using **both PCA/FA and ICA**.

---

## 📈 Normality assessment and assumption testing

### QQ plots & Mardia’s test (Figure 3)

- Univariate QQ plots show **right-skewed and heavy-tailed** distributions, especially for weight, waist, BMI.  
- Multivariate QQ plot shows a **heavy tail**, suggesting latent subpopulations with jointly elevated indicators.  
- **Mardia’s test:**
  - Skewness \(b_{1p} = 340.45\), p-value = 0.0  
  - Kurtosis \(b_{2p} = 802.53\), p-value = 0.0  
  - → Strong rejection of multivariate normality.

> “Thus, we conclude that the dataset does not satisfy the assumption of multivariate normality.”

Implications:

- Covariance-based methods (PCA, FA, GMM) may be **sensitive to outliers and tail behavior**.  
- Non-Gaussianity **justifies ICA** and motivates robust covariance estimation (MCD).  
- Heavy tails support the use of **clustering (GMM)** to capture latent subpopulations.

---

## 🧮 Principal Component Analysis (PCA)

### Component selection (Figure 4)

- **Scree plot:** first 4 components have eigenvalues > 1.  
- **Cumulative variance:** first 4 PCs explain **~70%** of total variance.  
- **Eigenvalue ratios:** not strongly informative (no clear “> 2” stability).  
- **Parallel analysis:** supports retaining **4 components**.

Decision:  
- Reduce from **11 to 4 dimensions**, preserving ~70% variance.  
- Use this 4D PCA space as a **stable representation** for downstream clustering (GMM).

Role of PCA:

- Removes redundancy from strongly correlated variables (e.g., weight–waist–BMI).  
- Provides a **noise-reduced, orthogonal basis** for clustering and visualization.

---

## 🧱 Factor Analysis (FA)

### Suitability

- **KMO = 0.53** → in the “miserable” range, indicating weak global factor structure.  
- **Bartlett’s test:** highly significant (p ≈ 0), so the correlation matrix is not identity and some shared structure exists.

Interpretation:

- FA can capture **some** latent structure but is **not ideal** for a fully global factor model.

### Factor selection (Figure 5)

- Likelihood ratio tests reject models with fewer than 4 factors.  
- Parallel analysis suggests **4–5 factors**.  
- A **5-factor solution** is chosen for interpretability.

### Factor structure (Figure 6, left)

- **Factor 1 – Body Mass / Obesity:**  
  - Strong loadings on weight, waist, BMI.  
- **Factor 2 – Aging / Metabolic Risk:**  
  - Dominated by age, with contributions from systolic BP and glycohemoglobin.  
- **Factor 3 – Body Size:**  
  - Height-related.  
- **Factor 4 – Blood Pressure:**  
  - Systolic and diastolic BP.  
- **Factor 5 – HDL Cholesterol:**  
  - Dominated by HDL.

Conclusion:  
- FA yields interpretable dimensions but, given low KMO and MVN violations, provides only a **partial and fragile** representation of the underlying structure.

---

## 🧬 Independent Component Analysis (ICA)

Given strong non-Gaussianity, ICA is used to identify **statistically independent sources** of variation.

### Non-Gaussianity diagnostics

- Kurtosis across different numbers of ICs:  
  - Non-Gaussianity becomes clear after IC2.  
- Negentropy:  
  - First 3 ICs close to Gaussian; after IC4, components are clearly non-Gaussian.  
- To align with FA, **5 independent components** are retained.

### ICA components (Figure 6, right)

- **IC1 – Obesity axis:**  
  - Weight, waist, BMI.  
- **IC2 – Metabolic axis:**  
  - Glycohemoglobin and age, with negative cholesterol loading.  
- **IC3 – Cholesterol axis:**  
  - Total and HDL cholesterol.  
- **IC4 – Blood pressure axis:**  
  - Systolic and diastolic BP.  
- **IC5 – Physiological function:**  
  - Creatinine, age, height.

These components suggest that the data is driven by **multiple independent physiological processes**, not just correlated factors.

### FA vs ICA

- Correlation between loadings:
  - FA1 vs IC1: **0.98** (body measurements).  
  - FA4 vs IC4: **0.86** (blood pressure).  
- Other components show weak correspondence → FA and ICA capture **different aspects** of structure (covariance vs independence).

ICA stability:

- Stability scores: ~0.30–0.33 across components → relatively low stability, indicating a **complex underlying structure** not fully captured by a single ICA model.

---

## 🧩 Gaussian Mixture Model (GMM) clustering

### Setup

- Clustering performed in **4D PCA space** to reduce noise and redundancy.  
- GMM assumes **approximate Gaussianity within clusters**, which is reasonable for health subgroups.

### Model selection (Figure 7)

- **BIC:** suggests **5–6 components**; improvement from 5 to 6 is marginal.  
- **Cross-validated log-likelihood:** clear improvement from 4 to **5 clusters**.  
- Final choice: **5-cluster solution** for interpretability.

### Cluster profiles

- **Cluster 0 – Young healthy adults:**  
  - Age: 20s–30s  
  - Lower body measurements, lower cholesterol, healthy BP → generally healthiest group.

- **Cluster 1 – Middle-aged high-risk:**  
  - Around 40s  
  - Higher weight, BMI, BP, and cholesterol.

- **Cluster 2 – Older, high-risk:**  
  - 60s  
  - Highest creatinine and glycohemoglobin, extreme BP (high systolic, low diastolic).

- **Cluster 3 – Older, healthier:**  
  - 60s  
  - Better health indicators than Cluster 2.

- **Cluster 4 – Obese middle-aged:**  
  - 50s  
  - Highest weight, waist circumference, and BMI.

Visualization:

- Clusters plotted on **PC1 vs PC2** show clear separation in PCA space.

### Demographic associations

- **Income & education:**  
  - Among older clusters (2 vs 3), Cluster 3 has **higher income and education** and is healthier → suggests a link between **socioeconomic status and health**.  
- **Gender:**  
  - No strong separation across clusters.

---

## 🛡️ Robust estimation and sensitivity analysis

### MCD-based robust covariance (Figure 8)

- MCD identifies **1,354 outliers**, far more than classical covariance.  
- DD-plot shows:
  - Some outliers are **swamped** (flagged only by classical).  
  - Many are **masked** under classical covariance but revealed by MCD.

Implications:

1. The dataset **strongly deviates** from multivariate normality.  
2. Outliers **distort classical covariance** and Mahalanobis distances.

### Impact on PCA

- PCA using MCD-based covariance yields **similar loadings** to classical PCA.  
- First PC still loads strongly on BMI, waist, weight, and cardiovascular indicators → a stable **“metabolic risk” axis**.

> “The stability of this component suggests that the primary direction of variation is not an artifact of extreme values but rather reflects a genuine population-level structure.”

### Impact on FA

- FA on MCD inliers produces loadings broadly similar to the full data.  
- However, FA remains limited by:
  - Weak global correlations  
  - Violations of MVN and common factor assumptions  

Conclusion: FA’s limitations are due to **model assumptions**, not just outliers.

### Impact on GMM

- GMM clustering in PCA space is **robust** to:
  - Using robust covariance  
  - Restricting to MCD inliers  
- Cluster profiles (young healthy, middle-aged high-risk, older subgroups, obese middle-aged) remain **consistent**, indicating genuine subpopulation structure.

---

## 🧠 Overall conclusions

- The NHANES health indicators are **not** well described by a small number of global latent factors.  
- The data reflects:
  - A **dominant metabolic-risk axis** (body size + cardiovascular indicators)  
  - Multiple **independent physiological processes** (ICA components)  
  - **Heterogeneous subpopulations** (GMM clusters)

- PCA and GMM are **stable and robust** under outlier-aware methods (MCD).  
- FA provides only a **partial** view due to weak factorability and assumption violations.  
- ICA reveals **independent sources** such as obesity, metabolic risk, cholesterol, blood pressure, and physiological function.  
- Socioeconomic status is associated with **healthier aging**, while gender plays a limited role.

> “Taken together, these results suggest that population health is best understood not through a single latent structure, but through the interaction of independent biological processes and heterogeneous subpopulations.”

---

## 📁 Repository structure

```text
├── data/                 # Raw and cleaned NHANES data (not included in repo)
├── notebooks/            # Jupyter notebooks for EDA, PCA, FA, ICA, GMM, robustness
├── src/                  # Modular Python scripts for analysis pipeline
├── figures/              # Exported figures (boxplots, heatmaps, QQ plots, PCA, FA, ICA, GMM, MCD)
└── README.md             # Project documentation
