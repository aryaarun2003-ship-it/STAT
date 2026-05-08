# Analyzing Social Vulnerability Across Virginia Counties
**Group 2: Arya Arun & Shouyu Li**

## Data Overview
[cite_start]This project utilizes a Virginia county-level social vulnerability dataset based on the CDC/ATSDR Social Vulnerability Index. [cite_start]The dataset includes variables such as poverty, unemployment, and housing burden to predict social vulnerability.

## Research Question 2: Dimensions of Social Vulnerability
[cite_start]**Rationale:** Since many variables in the dataset are related to each other, we use **Principal Component Analysis (PCA)** to identify broader dimensions of vulnerability. [cite_start]This helps summarize complex indicators into patterns like economic vulnerability or health and access barriers.

**Methodology:**
[cite_start]We applied PCA to reduce related variables into a smaller number of patterns. [cite_start]We focus on factors including poverty (`EP_POV150`), unemployment (`EP_UNEMP`), and internet access (`EP_NOINT`). [cite_start]We use a **Scree Plot** to determine the components to retain and analyze **loadings** to interpret each dimension.

### Results
(Paste the specific results from the file Shouyu sent you here, including the percentage of variance explained.)

## References
* Centers for Disease Control and Prevention/ Agency for Toxic Substances and Disease Registry. (2022). [cite_start]CDC/ATSDR Social Vulnerability Index.
