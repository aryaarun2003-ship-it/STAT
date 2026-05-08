# Social Vulnerability in Virginia Counties: A Statistical Learning Approach
Shouyu Li & Arya Arun
5/6/2026`
## Introduction and Research Questions
Social vulnerability describes how strongly communities may be affected by external stressors such as natural disasters, public health emergencies, and economic shocks (Flanagan et al., 2011). The CDC/ATSDR Social Vulnerability Index (SVI) condenses sixteen U.S. Census variables into a single percentile score so that public-health and emergency-management agencies can prioritize support to the most disadvantaged areas. Although the SVI is widely used, relatively little county-level work has compared modern statistical-learning methods on its component indicators in a single state. This project performs that comparison on the 133 Virginia counties and independent cities contained in the 2022 SVI release.
### Table 1. Research Questions and Statistical Learning Methods

| RQ | Question | Main Method(s) |
| :--- | :--- | :--- |
| **RQ1** | Which socioeconomic, demographic, housing, transportation, and access-related indicators are most strongly associated with overall social vulnerability in Virginia counties? (Descriptive structural analysis.) | Pearson correlation; LASSO with 10-fold CV; bootstrap LASSO selection stability; Elastic Net (alpha=0.5) sensitivity check. |
| **RQ2** | What underlying dimensions of vulnerability can be recovered from the indicators using PCA, and how do counties group into interpretable profiles using K-means and hierarchical clustering? (Descriptive structural analysis.) | Principal Component Analysis; K-means clustering; Ward.D2 hierarchical clustering; cluster-agreement via Adjusted Rand Index. |
| **RQ3** | Using only the 17 SVI indicators, can supervised learners distinguish urban from rural Virginia counties (a response that is independent of the SVI formula), and which classifier offers the best bias–variance trade-off? (External predictive analysis.) | Logistic Regression, Linear Discriminant Analysis, Random Forest, and Gradient Boosting; 5-fold cross-validation repeated 10 times on a stratified 70/30 split; ROC, AUC, calibration. |
The unit of analysis is the county. The continuous outcome of interest is the overall SVI percentile score (RPL_THEMES). However, because this composite score is mathematically constructed from many of the same indicators we use as predictors, any predictive model that uses the SVI components to recover the SVI score is subject to target leakage (Kaufman et al., 2012). A model that appears to achieve perfect classification accuracy under such conditions cannot be interpreted as evidence of genuine out-of-sample performance. To address this, we organize the project around three research questions, the first two of which are explicitly descriptive / structural (no predictive claim), and the third of which uses an external response variable (urbanity, defined by population density) to evaluate genuine predictive performance.


## Data Source and Variable Preparation
The dataset is the 2022 SVI database for Virginia, published by the CDC/ATSDR Geospatial Research, Analysis, and Services Program (CDC/ATSDR, 2024). The file contains percentile rankings, raw counts, and percentage estimates for each county and independent city. After excluding rows with missing overall SVI score and missing demographic denominators, the working sample contains 133 counties. The continuous response variable is the overall SVI percentile (RPL_THEMES); the predictor set consists of the 17 EP-percentage indicators that span the four official SVI themes (Table 2). We retain the original CDC variable names for code-data fidelity but display human-readable names throughout the report.

A central methodological point is that the overall SVI score is by construction a composite of these 17 indicators. Consequently, in RQ1 and RQ2 we treat the score not as a target to be predicted, but as a summary whose internal structure we wish to understand. RQ3 instead uses an external binary response derived from population density (Section RQ3), avoiding any leakage between predictor and response.
### Table 2. Predictor Variables Used in the Analysis

| Variable | Original Name | Theme |
| :--- | :--- | :--- |
| Poverty Rate | EP_POV150 | Socioeconomic status |
| Unemployment Rate | EP_UNEMP | Socioeconomic status |
| Housing Cost Burden | EP_HBURD | Socioeconomic status |
| No High School Diploma | EP_NOHSDP | Socioeconomic status |
| Uninsured Rate | EP_UNINSUR | Socioeconomic status |
| Age 65 or Older | EP_AGE65 | Household & demographic composition |
| Age 17 or Younger | EP_AGE17 | Household & demographic composition |
| Disability Rate | EP_DISABL | Household & demographic composition |
| Single-Parent Households | EP_SNGPNT | Household & demographic composition |
| Limited English | EP_LIMENG | Household & demographic composition |
| Minority Population | EP_MINRTY | Minority status |
| Multi-Unit Housing | EP_MUNIT | Housing, transportation & access |
| Mobile Homes | EP_MOBILE | Housing, transportation & access |
| Crowded Housing | EP_CROWD | Housing, transportation & access |
| No Vehicle Access | EP_NOVEH | Housing, transportation & access |
| Group Quarters | EP_GROUPQ | Housing, transportation & access |
| No Internet Access | EP_NOINT | Housing, transportation & access |
### Table 3. Descriptive Statistics of SVI Indicators and Outcome

| Variable | Mean | Median | SD | Min | Max |
| :--- | :--- | :--- | :--- | :--- | :--- |
| Age 17 or Younger | 19.87 | 19.6 | 3.40 | 7.2 | 27.3 |
| Age 65 or Older | 19.88 | 19.7 | 5.48 | 9.3 | 39.4 |
| Crowded Housing | 1.68 | 1.6 | 1.19 | 0.0 | 8.9 |
| Disability Rate | 15.72 | 14.9 | 5.32 | 6.5 | 36.4 |
| Group Quarters | 4.34 | 2.2 | 6.30 | 0.0 | 43.0 |
| Housing Cost Burden | 22.97 | 21.7 | 6.25 | 11.5 | 41.7 |
| Limited English | 1.28 | 0.6 | 1.88 | 0.0 | 13.3 |
| Minority Population | 29.97 | 27.2 | 18.60 | 2.7 | 85.1 |
| Mobile Homes | 8.89 | 6.4 | 8.24 | 0.0 | 31.8 |
| Multi-Unit Housing | 7.30 | 3.5 | 9.46 | 0.0 | 54.9 |
| No High School Diploma | 11.64 | 10.7 | 4.80 | 2.6 | 25.8 |
| No Internet Access | 17.30 | 17.5 | 7.87 | 3.3 | 37.2 |
| No Vehicle Access | 6.10 | 5.4 | 3.21 | 0.6 | 16.5 |
| **Overall SVI Score** | **0.50** | **0.5** | **0.29** | **0.0** | **1.0** |
| Poverty Rate | 21.36 | 20.3 | 8.69 | 3.9 | 42.3 |
| Single-Parent Households | 5.65 | 5.4 | 2.50 | 0.9 | 13.2 |
| Unemployment Rate | 4.70 | 4.3 | 1.99 | 1.3 | 14.7 |
| Uninsured Rate | 7.46 | 7.0 | 2.73 | 0.9 | 19.7 |

## Exploratory Analysis
The exploratory analysis examines whether predictor distributions justify the modelling choices that follow. Because LASSO, PCA, and K-means are sensitive to variable scale, we standardize predictors before fitting any multivariate model. A short check of skewness and zero-inflation is also useful, since strongly zero-inflated indicators (e.g., crowded housing, limited English) can distort correlation-based methods.


Figure 1. Distribution of the overall SVI percentile score across Virginia counties. The score is approximately uniform-tilted toward higher values, with a small mode in the upper percentile band.

Figure 1 shows that the overall SVI scores are spread across the full [0, 1] percentile range, which is expected because RPL_THEMES is itself a percentile rank rather than a raw measure. A small concentration of counties sits in the upper band (0.85–1.0), corresponding to the most socially vulnerable counties; this upper-tail mass is large enough to support a binary high-vulnerability framing without producing extreme class imbalance, and it foreshadows the existence of a substantively distinct cluster in RQ2.

Figure 2. Marginal distributions of the 17 SVI predictors. Several variables (Limited English, Crowded Housing, Group Quarters) are right-skewed and zero-inflated; standardization alone cannot remove this, so coefficient interpretations should be made on the original (percentage) scale.

Figure 2 reveals substantial heterogeneity in shape across the 17 predictors. Variables such as Housing Cost Burden, Disability Rate, and No Internet Access are roughly bell-shaped, while Limited English, Crowded Housing, and Group Quarters are strongly right-skewed and zero-inflated. We retain the percentages on their original scale rather than log-transforming them, because z-score standardization preserves rank order and the penalized regression and tree-based methods we use are not sensitive to monotone transformations of the predictors.

The descriptive plots show that the indicators have very different ranges and spreads, and several display strong skewness or floor effects. The VIF table (Table 4) confirms substantial multicollinearity: at least three indicators exceed VIF = 5, which would inflate the variance of ordinary least-squares coefficients. This is the principled reason for using LASSO and Elastic Net in RQ1 instead of OLS.

RQ1: Factors Most Strongly Associated with Overall Vulnerability
To identify which indicators are most closely connected to the composite SVI score, we combine four complementary tools:

Pearson correlation — a transparent bivariate baseline.
LASSO regression — penalized regression that performs variable selection in the presence of multicollinearity, with the regularization parameter chosen by 10-fold cross-validation (we report both lambda.min and the one-standard-error rule lambda.1se).
Bootstrap LASSO selection — we resample the data 500 times and refit LASSO at lambda.1se to obtain the empirical selection frequency of each predictor, a robust measure of which variables remain useful under sampling variation.
Elastic Net (alpha = 0.5) — a compromise between LASSO and Ridge, used as a sensitivity check because the LASSO can be unstable when groups of correlated predictors carry similar signal.
All predictors are standardized (z-score) before fitting penalized models, so that coefficient magnitudes are directly comparable.

Figure 3. Pearson correlation matrix among the 17 SVI indicators. Strong positive correlations between Poverty Rate, Single-Parent Households, and No High School Diploma motivate penalized regression rather than OLS.

Figure 3 shows that several indicators move together in coherent blocks. Poverty Rate, No High School Diploma, Single-Parent Households, and Uninsured Rate form a socioeconomic block with pairwise correlations between roughly 0.5 and 0.8, while Mobile Homes, No Internet Access, and Age 65 or Older form a second rural-character block. These overlapping correlation patterns motivate penalized regression in the next subsection: when groups of indicators carry redundant signal, ordinary least-squares coefficients become unstable, whereas LASSO either picks one representative or shrinks the whole group toward zero.

Figure 4. Cross-validation curve for LASSO. The vertical dashed lines mark lambda.min (left) and the more parsimonious lambda.1se (right); we report results at both, with lambda.1se as our preferred sparse solution.

Figure 4 displays the typical U-shape of cross-validation error against log-lambda, with a relatively flat minimum-error region. The left dashed line marks lambda.min (lower λ, more variables retained) and the right dashed line marks lambda.1se (higher λ, sparser model whose CV error is within one standard error of the minimum). We adopt lambda.1se as the default sparse solution because the right-hand line still falls within the flat error band, indicating that the sparser model loses very little predictive accuracy while being substantially more robust to sampling variation.

Figure 5. Bootstrap LASSO selection frequencies (B = 500, lambda.1se). Variables retained in more than 80% of bootstrap samples (dashed line) constitute the stable signal: Housing Cost Burden, Poverty Rate, Single-Parent Households, No High School Diploma, and Uninsured Rate.

Figure 6. LASSO regression diagnostics at lambda.1se. Left: predicted vs actual SVI scores show tight calibration. Right: residual plot shows no systematic non-linearity or heteroskedasticity.

The LASSO at lambda.min retains nearly all 17 indicators because the indicators are by construction informative about the composite SVI; the parsimonious lambda.1se solution drops 8 indicators while keeping the structural picture intact (Table 6). The bootstrap analysis (Figure 5) further confirms that the stable signal is dominated by Housing Cost Burden, Poverty Rate, Single-Parent Households, No High School Diploma, and Uninsured Rate — the classical socioeconomic correlates of vulnerability. The Elastic Net solution is qualitatively similar but slightly less sparse, which is expected because Elastic Net retains correlated predictors as groups. The diagnostic plots (Figure 6) show no systematic non-linearity or heteroskedasticity, supporting the linear penalized specification. We emphasize that these results are associative and structural, not causal: because the response is a known function of the predictors, the analysis characterizes which indicators dominate the SVI formula in this state, not which factors cause social vulnerability.

## RQ2: Main Dimensions of Vulnerability and County Grouping
The second question asks how the 17 indicators collapse into a smaller number of dimensions and whether counties cluster into recognizable profiles. We use PCA on the standardized predictors, then cluster counties using both K-means (a partitional method) and Ward.D2 hierarchical clustering (an agglomerative method). Comparing two clustering algorithms is a fairness safeguard: if the two methods agree, the resulting county profiles are not an artifact of one algorithm’s bias. We measure agreement using the Adjusted Rand Index (Hubert & Arabie, 1985).

##Principal Component Analysis
