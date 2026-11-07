# Week 5 Deliverable: Predictive Modelling Strategy and Algorithm Selection

> **Project:** Predicting Repeat Purchase and Personalizing Recommendations in Online Skincare  
> **Week:** 5 of 8  
> **Estimated Hours:** 35 hours  
> **Deliverables:** Word Document (`Week_5_Predictive_Modelling_Strategy_Deliverable.docx`) & Technical Architecture Specification (`Week5_Predictive_Modelling_Strategy.md`)

---

## 1. Purpose and Link to Weeks 1 to 4

Week 1 defined the foundational business problem: predict which first-time customers of an online skincare retailer will make a repeat purchase within 90 days, and recommend products that keep them engaged and maximize Customer Lifetime Value (CLV). Week 2 identified and audited multi-tier data sources (real beauty logs, non-beauty marketplace analogues such as Olist, and synthetic skincare simulators). Week 3 engineered the cleaning and validation pipeline that outputs standardized tables into `data/processed/`. Week 4 conducted an extensive exploratory data analysis (EDA) and established an engineered feature store encompassing transaction behaviour, replenishment run-out timing, INCI chemical ingredient profiles, customer skin-type fit, review sentiment polarities, and external trend spikes.

This Week 5 deliverable establishes the formal **predictive modeling strategy and algorithmic selection framework**. It aligns each commercial question with an appropriate predictive task, describes more than two candidate models in depth, presents an exhaustive comparative rationale, defines evaluation metrics and expected performance ranges, specifies a time-based validation design that strictly eliminates data leakage, and provides actionable frameworks translating model outputs into marketing and CRM decision workflows.

> ⚠️ **Methodological & Honesty Note:**  
> The performance ranges and lift values presented in Section 8 are reasoned empirical expectations designed as testable hypotheses for Weeks 6–8, not realized results. All code blocks in Section 11 are illustrative, modular, production-style pipelines using `scikit-learn`, `xgboost`, `lifelines`, `lifetimes`, and `implicit`, designed to be adapted to specific column taxonomies and library releases.

---

## 2. Aligning the Business Problem with Predictive Tasks

Skincare retention is multi-dimensional. Predicting repeat behavior requires answering whether a customer will return, when their physical product runs out, how much lifetime value they represent, and which product best complements their routine. Consequently, the business problem is mapped across six synergized predictive tasks (**T1 to T6**):

| # | Business Question | Predictive Task | Target Variable | Primary Algorithmic Candidates | Primary Evaluation Metrics |
|---|-------------------|-----------------|-----------------|--------------------------------|----------------------------|
| **T1** | Which first-time customers will reorder within 90 days? | Binary Classification | `repeat_90d`<br>• Model A: at first order (Day 0)<br>• Model B: at Day 30 with early usage signals | Regularized Logistic Regression, Random Forest, HistGradientBoosting, XGBoost | PR-AUC (Average Precision), ROC-AUC, Recall@Fixed Precision, Top-Decile Lift |
| **T2** | When will a customer reorder, and how does timing vary by first product? | Survival (Time-to-Event) Analysis | Days to 2nd order, right-censored | Non-parametric Kaplan-Meier, Cox Proportional Hazards, Penalized Cox | Harrell's Concordance Index (C-index), Integrated Brier Score, Calibration Curves |
| **T3** | How much will a customer spend over the next 6 to 12 months? | Continuous Regression (CLV) | Future Net Spend (currency) | Ridge Regression, Tweedie/Hurdle Gradient Boosting, BG/NBD + Gamma-Gamma | RMSE, MAE, Spearman Rank Correlation, Decile Calibration |
| **T4** | Which specific product should we suggest next? | Personalized Ranking & Recommendation | Next purchase or positive 5-star review | Popularity Baseline, Item-Based kNN (Cosine), Implicit ALS, Hybrid LightFM | Precision@K, Recall@K, MAP, NDCG@K (K=5, 10), Catalog Coverage |
| **T5** | What distinct customer personas exist? | Unsupervised Clustering | Latent customer clusters (unlabeled) | K-Means++, Gaussian Mixture Models (GMM), Agglomerative Hierarchical (Ward) | Silhouette Score, Davies-Bouldin Index, Bootstrap Stability (ARI) |
| **T6** | Is an early review signaling formula irritation or poor skin fit? | Text Classification & NLP Topic Mining | Sentiment polarity / Irritation topic flag | Rule-based VADER, TF-IDF + Logistic Regression, Fine-tuned Transformer | Macro F1-Score, Class Recall on Irritation, Confusion Matrix |

### Task Hierarchy & Synergy
- **T1 is the core decision gate:** Every customer receives a calibrated repeat probability driving marketing spend.
- **T2 and T3 add operational granularity:** T2 determines *when* automated triggers fire (near physical product exhaustion), while T3 identifies *which* high-value customers justify direct promotional margins.
- **T4 populates the content payload:** Recommendations tailored to user skin type populate email and web touchpoints.
- **T5 guides communication tone:** Unsupervised personas determine whether messaging focuses on anti-aging science or minimalist routines.
- **T6 functions upstream:** Early post-purchase review text is mined to generate dynamic churn alerts feeding directly into T1 and T4.

---

## 3. Modeling Strategy Overview

### Eight Core Strategic Principles

1. **Start Simple & Anchor to Baselines:** Every complex model is benchmarked against transparent baselines (majority class prediction, RFM-only logistic regression, popularity recommenders). If an advanced architecture does not deliver statistically significant lift over a baseline, the simpler model is retained.
2. **Dual-Model Interpretability-Accuracy Ladder:** An interpretable reference model (Regularized Logistic Regression) is maintained in parallel with non-linear tree ensembles (Gradient Boosted Trees) to balance executive transparency with operational precision.
3. **Strict Temporal Partitioning:** Random K-fold cross-validation is strictly banned. Models are validated on chronological customer cohorts to reflect production deployment and prevent lookahead bias.
4. **Censoring-Aware Architecture:** Recent cohorts with incomplete 90-day observation windows are not discarded or naively labeled as churned. Survival models (Cox PH) and probabilistic retail models (BG/NBD) formally incorporate right-censoring.
5. **Fair and Equalized Benchmarking:** All candidate models are trained on identical training folds with identical feature subsets, ensuring algorithmic comparisons reflect genuine learning capacity rather than data discrepancies.
6. **Rule of Parsimony with Bootstrap Bounds:** A complex model supersedes a simpler alternative only if its improvement exceeds the 95% bootstrap confidence interval and yields positive incremental margin on the empirical profit curve.
7. **Systematic Probability Calibration:** Raw model outputs are calibrated using Isotonic Regression or Platt Scaling, ensuring predicted scores represent true empirical probabilities necessary for expected-value decision making.
8. **Explainability & Fairness Auditing:** Global and local explanations (TreeSHAP, Partial Dependence Plots) are generated prior to deployment. Fairness slicing across sensitive attributes (skin type, tone, region) guarantees equitable performance.

### Data Context Shaping Model Selection
- **Extreme Class Imbalance:** Repeat rates in general marketplace data (Olist) are between 3% and 7%, whereas dedicated skincare retail data yields repeat rates between 15% and 28%. Loss functions and evaluation metrics must accommodate severe negative class dominance.
- **Mixed Feature Space:** Tabular datasets combine continuous physical metrics (volume in ml, price per ml, replenishment due ratios), high-cardinality nominals (SKU IDs, acquisition channels), multi-label chemical active flags, and continuous NLP sentiment scores.
- **Asymmetric Sample Densities:** Customer records number in the tens or hundreds of thousands, but positive repeat instances within niche sub-categories remain scarce.
- **Evidence Tier Transparency:** Algorithms evaluated on synthetic simulator data validate pipeline functionality and structural recovery; however, real-world claims rest strictly on empirical customer histories.

---

## 4. Candidate Models in Detail

### 4.1 Model 1: Regularized Logistic Regression (Interpretable Baseline for T1)

- **Mathematical Framework:** Models the log-odds of a repeat purchase as a linear combination of features. Under an Elastic Net penalty combining L1 (Lasso) and L2 (Ridge) constraints, the optimization objective is:
  $$\min_{w, b} -\frac{1}{N} \sum_{i=1}^N \left[ y_i \log(p_i) + (1 - y_i) \log(1 - p_i) \right] + \lambda \left( \alpha \|w\|_1 + \frac{1 - \alpha}{2} \|w\|_2^2 \right)$$
  $$\text{where } p_i = \sigma(w^T x_i + b) = \frac{1}{1 + e^{-(w^T x_i + b)}}$$
- **Domain Alignment:** Serves as the primary explanatory benchmark. Coefficients convert directly into **Odds Ratios** ($\text{OR} = e^{w_j}$), enabling marketing stakeholders to quantify the impact of specific drivers (e.g., *"Purchasing a coordinating toner increases the odds of repeat purchase by 1.34x, holding spend and skin type constant"*). It allows rigorous hypothesis testing (H1–H5) via Wald statistics and confidence intervals.

| Strengths | Weaknesses |
|---|---|
| Complete transparency; yields audited odds ratios for CRM strategy. | Cannot capture non-linearities or threshold effects without manual feature engineering. |
| Naturally well-calibrated probabilities when trained without artificial class weights. | Blind to higher-order interactions (e.g., High Irritants $\times$ Sensitive Skin). |
| Highly scalable on high-dimensional sparse data (one-hot categorical features, TF-IDF). | Highly sensitive to multicollinearity among correlated RFM metrics (requires VIF pruning). |
| L1 regularization performs automatic, principled feature selection. | Vulnerable to extreme spend outliers; requires log-transforms. |

- **Key Hyperparameters:** Inverse regularization strength $C \in [0.01, 10.0]$, penalty mix `l1_ratio` $\in [0.0, 1.0]$, `class_weight='balanced'`, solver `saga`.

---

### 4.2 Model 2: Gradient Boosted Decision Trees (Primary Predictive Model for T1 and T3)

- **Mathematical Framework:** Builds an additive ensemble of $M$ shallow regression trees in a forward stagewise manner, minimizing regularized empirical loss:
  $$F_m(x) = F_{m-1}(x) + \eta f_m(x)$$
  $$f_m = \arg\min_f \sum_{i=1}^N \left[ g_i f(x_i) + \frac{1}{2} h_i f(x_i)^2 \right] + \gamma T + \frac{1}{2} \lambda \sum_{j=1}^T w_j^2$$
  where $g_i = \partial \mathcal{L} / \partial F_{m-1}(x_i)$ and $h_i = \partial^2 \mathcal{L} / \partial F_{m-1}(x_i)^2$. Implementations evaluated include `HistGradientBoostingClassifier`, `XGBoost`, and `LightGBM`.
- **Domain Alignment:** Skincare churn is driven by complex interactions. For example, a high chemical irritant load has zero negative impact on oily/resilient skin, but sharply reduces repeat purchase for sensitive/dry skin. Boosted trees discover these multi-way splits automatically, handle mixed feature types and missing values inherently, and provide state-of-the-art predictive performance on tabular business datasets.

| Strengths | Weaknesses |
|---|---|
| State-of-the-art predictive power on tabular customer data. | Opaque decision structure; requires post-hoc attribution tools (TreeSHAP). |
| Discovers complex non-linear thresholds and feature interactions automatically. | Susceptible to overfitting on sparse data without early stopping and leaf regularization. |
| Native support for missing values via optimal default split paths. | Boosting margins distort probability calibration, requiring post-hoc isotonic scaling. |
| Supports domain monotonic constraints (e.g., higher replenishment due ratio cannot lower repeat score). | Incapable of out-of-distribution extrapolation beyond observed training ranges. |

- **Key Hyperparameters:** Learning rate $\eta \in [0.02, 0.08]$, tree depth `max_depth` $\in [3, 6]$, `min_samples_leaf` $\in [20, 80]$, `l2_regularization` $\in [0.5, 5.0]$, early stopping rounds = 30.

---

### 4.3 Model 3: Random Forest (Robust Ensemble Comparator)

- **Mathematical Framework:** Bagging ensemble training $B$ decorrelated decision trees on bootstrap resamples with randomized feature subsets (typically $\sqrt{p}$). Predictions represent the unweighted average of individual tree probabilities:
  $$P(y = 1 \mid x) = \frac{1}{B} \sum_{b=1}^B P_b(y = 1 \mid x)$$
- **Domain Alignment:** A forgiving, low-variance benchmark requiring minimal tuning. It provides out-of-bag (OOB) error estimates and cross-validates Gradient Boosting findings. Standard Gini impurity importance is biased toward high-cardinality continuous features, so Permutation Feature Importance is utilized.

---

### 4.4 Model 4: Single Decision Tree (Executive Communication Aid)

A shallow, cost-complexity pruned CART tree (`max_depth=3 to 4`) trained strictly as an explanatory artifact. While inadequate for deployment due to high variance and step-function approximations, its extracted rule paths (e.g., *"IF First Order Discount > 45% AND Routine Completeness = 0 THEN Repeat Rate = 8.2%"*) provide marketing leadership with clear mental models of customer churn pathways.

---

### 4.5 Model 5: Cox Proportional Hazards & Kaplan-Meier (Repurchase Timing, T2)

- **Mathematical Framework:** Formulates the hazard of a second purchase at time $t$ conditioned on customer covariates:
  $$h(t \mid x) = h_0(t) \exp\left( \sum_{j=1}^p \beta_j x_j \right)$$
  Optimized via Cox's Partial Likelihood over uncensored failure times:
  $$L(\beta) = \prod_{i: C_i = 1} \frac{\exp(\beta^T x_i)}{\sum_{j \in R(t_i)} \exp(\beta^T x_j)}$$
- **Domain Alignment:** In non-contractual retail, customers do not announce when they lapse; they simply stop reordering. Standard classifiers discard customers whose 90-day window is incomplete. The Cox model incorporates right-censored records and generates **Hazard Ratios** ($\text{HR} = e^{\beta_j}$). For example, an HR of 1.42 on *Cleansers* indicates customers reorder 42% faster than average, allowing CRM teams to dynamically trigger replenishment emails at day 35 rather than day 60.

---

### 4.6 Model 6: BG/NBD with Gamma-Gamma (Probabilistic CLV Baseline, T3)

- **Mathematical Framework:** Fader, Hardie, and Lee's (2005) Beta-Geometric/Negative Binomial Distribution captures purchase frequency and latent customer dropout:
  - While active, transactions follow a Poisson process with transaction rate $\lambda \sim \text{Gamma}(r, \alpha)$.
  - After each purchase, unobserved dropout occurs with probability $p \sim \text{Beta}(a, b)$.
  - Monetary order values follow an independent Gamma-Gamma distribution.
- **Domain Alignment:** Provides an elegant stochastic benchmark requiring only transaction recency, frequency, and tenure ($T$). It provides an auditable baseline against which machine learning CLV regressors are compared.

---

### 4.7 Model 7: Supervised CLV Regression (T3, Net Spend Forecasting)

To predict monetary spend over months 2 through 12, three formulations are evaluated:
1. **Log-Transformed Ridge Regression:** Fitted on $\log(1 + \text{spend})$ with L2 regularization.
2. **Tweedie Gradient Boosted Regressor:** Explicitly handles compound Poisson-Gamma distributions characterized by zero-inflation and heavy positive tails.
3. **Two-Part Hurdle Model:** A binary classifier predicts $P(\text{Spend} > 0)$, multiplied by a continuous regressor estimating $E[\text{Spend} \mid \text{Spend} > 0]$. Evaluated on RMSE, MAE, and Spearman Rank Correlation.

---

### 4.8 Model 8: Unsupervised Customer Segmentation (T5)

- **K-Means++:** Standardized, log-transformed RFM and basket composition metrics produce spherical, well-separated operational segments.
- **Gaussian Mixture Models (GMM):** Employs expectation-maximization with full covariance matrices to yield probabilistic soft-clustering memberships, accurately modeling customers exhibiting hybrid behaviors.
- **Hierarchical Clustering (Ward):** Generates dendrogram visualizations on customer sub-samples to validate natural cluster cutoffs. Evaluated on Silhouette coefficients, Davies-Bouldin index, and bootstrap stability (Adjusted Rand Index).

---

### 4.9 Model 9: Personalized Recommender Architectures (T4)

| Recommender Model | Algorithmic Mechanism | Domain Strengths | Domain Limitations |
|---|---|---|---|
| **Popularity Baseline** | Category-level and global purchase velocity ranking | Fast, cold-start safe; high baseline conversion | Zero personalization; reinforces catalog bias |
| **Item-Based kNN** | Cosine similarity over binary co-purchase matrices | Highly explainable (*"Bought X also bought Y"*); zero training | Severe sparsity breakdown; unscalable SKU expansion |
| **Matrix Factorization (Implicit ALS)** | Alternating Least Squares on implicit feedback matrices | Discovers latent preference factors; highly scalable | Severe cold-start failure on first-time buyers |
| **Content-Based (Ingredient Similarity)** | Vector cosine similarity over INCI chemical profiles | Solves cold-start completely; ensures skin compatibility | Limited serendipity; risks redundant recommendations |
| **Hybrid LightFM** | Factorization machine blending collaborative embeddings with user skin profiles and product attributes | Superior accuracy on sparse cold-start profiles (Hypothesis H6) | Higher engineering overhead; requires joint hyperparameter tuning |

---

## 5. Comparative Algorithmic Rationale & Selection Decision

### Master Comparison Across Ten Operational Criteria

| Evaluation Criterion | Logistic Regression | Decision Tree | Random Forest | Gradient Boosted Trees | Cox PH / BG-NBD | K-Means / GMM | Hybrid LightFM |
|---|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| **Primary Task** | T1 (Baseline) | T1 (Explaining) | T1, T3 (Comparator) | **T1, T3 (Champion)** | **T2, T3 (Survival/CLV)** | **T5 (Segmentation)** | **T4 (Recs)** |
| **Expected Accuracy** | Medium | Low | Medium-High | **Highest (SOTA)** | Medium | N/A | High |
| **Interpretability** | **Very High (Odds Ratios)** | High (Shallow Rules) | Medium | Medium (TreeSHAP) | High (Hazard Ratios) | High (Centroids) | Medium (Latent Factors) |
| **Non-Linear Interactions** | Manual Only | Inherent | Inherent | **Inherent (Automatic)** | Semi-Parametric | Geometric | Latent Interaction |
| **Missing Value Resilience** | Imputation Required | Split Paths | Imputation Required | **Inherent (Hist-binned)** | Data Prep Required | Imputation Required | Sparse Native |
| **Class Imbalance Support** | Cost-Sensitive Weights | Class Weights | Balanced Subsample | `scale_pos_weight` | Inherent Hazard Rate | N/A | Implicit BPR Loss |
| **Right-Censoring Aware** | Discards / Distorts | Discards | Discards | Discards | **Mathematically Sound** | N/A | N/A |
| **Overfitting Vulnerability** | Very Low (L1/L2) | Severe | Low-Medium | Moderate (Early Stop) | Low | Low | Moderate |
| **Tuning Complexity** | Minimal ($C, \alpha$) | Low (`max_depth`) | Low ($n_{\text{trees}}, \text{leaf}$) | High ($\eta, d, \lambda$) | Minimal | Moderate ($k$) | High ($d, \text{epochs}$) |
| **Strategic Business Output** | Audited Odds Ratios | Executive Rules | Permutation Importance | **Ranked Lists & SHAP** | **Dynamic Run-out Triggers** | **Customer Personas** | **Personalized Recs** |

### Formal Selection Decisions
1. **Core Repeat Classifier (T1):** **Gradient Boosted Decision Trees (HistGradientBoosting / XGBoost)** is selected as the production model due to its proven superiority on imbalanced tabular data with non-linear ingredient $\times$ skin sensitivity interactions. **Regularized Logistic Regression** is retained as an ongoing reference model to audit feature stability and provide executive odds ratios.
2. **Repurchase Timing Engine (T2):** **Cox Proportional Hazards** is selected to generate customer-specific hazard ratios, supported by Kaplan-Meier curves for cohort communication, resolving right-censoring constraints.
3. **Monetary CLV Forecasting (T3):** A **Two-Part Hurdle Gradient Boosting** model is selected as the primary challenger against the **BG/NBD + Gamma-Gamma** baseline to optimize monetary rank correlation for VIP tiering.
4. **Dermatological Customer Segmentation (T5):** **K-Means++** with GMM soft-clustering confirmation is selected to extract stable customer behavioral personas that feed downstream models.
5. **Next-Product Recommendation (T4):** A **Hybrid Content-Collaborative Recommender (LightFM)** is selected to eliminate cold-start blindness (Hypothesis H6), falling back to category-popularity baselines for anonymous first-time visitors.

---

## 6. Comprehensive Evaluation Metrics

### 6.1 Binary Classification Metrics (T1)

| Metric | Mathematical Definition | Strategic Purpose in Skincare CRM |
|---|---|---|
| **PR-AUC (Average Precision)** | $\sum_n (R_n - R_{n-1}) P_n$ | **Primary tuning metric.** Evaluates minority repeat instances without inflation from high true negative volumes. |
| **ROC-AUC** | Area under True Positive vs. False Positive Rate curve | Evaluates global customer ranking quality across all classification percentiles. |
| **Recall @ Fixed Precision (60%)** | $\text{Recall} \mid \text{Precision} \ge 0.60$ | Bounds marketing budget risk: guarantees at least 6 out of 10 targeted customers convert before releasing incentives. |
| **Top-Decile Lift (Lift@10%)** | $\frac{\text{Repeat Rate in Top 10\% Predicted}}{\text{Overall Population Repeat Rate}}$ | Directly quantifies the performance multiplier of algorithmic targeting over random outreach. |
| **Brier Score Loss** | $\frac{1}{N} \sum_{i=1}^N (p_i - y_i)^2$ | Decomposes probability reliability and sharpness, verifying probability fidelity for expected-value decisioning. |
| **Empirical Profit Curve** | $\sum_{i \in \text{Targeted}} [p_i \cdot \text{Margin} - \text{Cost}_{\text{contact}}]$ | Identifies the optimal operational decision threshold rather than defaulting to an arbitrary $p = 0.5$. |

### 6.2 Regression, Survival, Clustering & Recommender Metrics
- **CLV Regression (T3):** RMSE (penalizes large revenue errors); MAE (median error resilient to high-spending whales); Spearman Rank Correlation ($\rho$, validates customer ranking for VIP tiering); Decile Calibration plots.
- **Survival Analysis (T2):** Harrell's Concordance Index (C-index, ranking concordance between predicted risk and actual event time); Integrated Brier Score over 30, 60, and 90-day horizons.
- **Clustering Quality (T5):** Silhouette Coefficient (cohesion vs. separation); Davies-Bouldin Index; Adjusted Rand Index (ARI across bootstrap resamples to confirm structural cluster stability).
- **Recommendation Ranking (T4):** Precision@K and Recall@K ($K=5, 10$); Normalized Discounted Cumulative Gain (NDCG@K, penalizing relevant items placed lower in the UI carousel); Catalog Coverage and Gini Diversity Index.
- **NLP Text Classification (T6):** Macro-averaged F1-Score; Specific Recall on the Irritation/Formula Adverse Reaction class (minimizing false negatives on safety concerns).

---

## 7. Empirical Baselines to Beat

| Task | Mandatory Baseline Model | Methodological Justification |
|---|---|---|
| **T1 (Repeat)** | 1. Prior Base Rate (Zero Information)<br>2. Logistic Regression on RFM metrics only | Quantifies the exact incremental lift gained by incorporating Week 4 chemical, dermatological, and NLP features. |
| **T2 (Timing)** | Non-parametric Kaplan-Meier on pooled cohorts | Validates whether conditioning hazard on product category and due ratio sharpens replenishment windows. |
| **T3 (CLV)** | Category Mean Spend & Probabilistic BG/NBD | Verifies whether supervised ML regression outperforms classic stochastic lifetime models. |
| **T4 (Recs)** | Global and Category-Level Popularity | Establishes the non-personalized conversion benchmark that collaborative algorithms must beat. |
| **T5 (Segments)** | Heuristic Rule-Based RFM Quintiles | Demonstrates whether unsupervised multi-modal clustering reveals non-obvious behavioral typologies. |

---

## 8. Expected Model Performance (Predictions to be Tested)

| Predictive Setting & Pipeline | Expected Performance Range | Domain Justification & Underlying Mechanics |
|---|:---:|---|
| **T1: Low-Repeat Marketplace (Olist Proxy)** | ROC-AUC: 0.58 – 0.68<br>PR-AUC: 0.12 – 0.22 | Extremely sparse pre-purchase history; marketplace buyers exhibit low brand affinity and high deal-seeking. |
| **T1: Dedicated Skincare Retailer (UCI / D2C)** | ROC-AUC: 0.72 – 0.84<br>PR-AUC: 0.38 – 0.52 | Substantial behavioral signal captured in basket composition, repeat brand affinities, and skincare regimen routines. |
| **T1: Synthetic Skincare Simulator** | ROC-AUC: 0.82 – 0.92<br>PR-AUC: 0.55 – 0.70 | Confirms pipeline integrity and latent rule recovery; serves as an engineering sanity check rather than real-world proof. |
| **Algorithmic Delta: GBDT vs. Logistic Regression** | GBDT Delta: +0.02 – +0.06 AUC<br>Lift@10% Delta: +15% – +30% | Gradient boosting extracts non-linear interactions between irritant load, skin sensitivity, and price tier. |
| **Impact of NLP Review Features (H4)** | Delta: +0.01 – +0.03 PR-AUC | Signal is powerful but truncated; reviews exist only for customer subsets who proactively review within 30 days. |
| **Replenishment Timing Feature Impact** | Top 3 Permutation Importance | Replenishment due ratio is the single strongest physical driver of repurchase in consumable categories. |
| **T4: Hybrid Rec vs. Popularity Baseline** | NDCG@10 Delta: +0.08 – +0.14<br>(Cold-start users) | Hybrid content-collaborative modeling eliminates cold-start blindness by leveraging INCI ingredient profiles. |

> 🚨 **Critical Anomaly Rule:** Any test ROC-AUC exceeding 0.90 on real e-commerce customer data triggers an immediate audit for target leakage, lookahead bias, or post-purchase data contamination prior to any deployment.

---

## 9. Validation Design and Robustness Framework

### 9.1 Temporal Embargo Splitting Architecture

Random K-fold cross-validation is structurally invalid for customer churn modeling because it allows future seasonal surges and marketing campaigns to leak into past evaluation folds. We enforce a **Time-Based Partitioning strategy with a Mandatory 90-Day Embargo Gap**:

| Partition | Cohort Date Window | Structural Purpose | Leakage Mitigation Mechanism |
|---|---|---|---|
| **Training Split** | Months 1 through 12<br>(Cohorts $\le T_1 - 90\text{d}$) | Initial model fitting and recursive feature elimination. | Completely isolated; zero exposure to validation or testing time horizons. |
| **Embargo Gap** | 90-Day Inactive Buffer<br>($T_1 - 90\text{d}$ to $T_1$) | Prevents label overlap between late training buyers and early validation buyers. | Guarantees that 90-day repeat labels of training customers have fully resolved before validation commences. |
| **Validation Split** | Months 14 through 18<br>($T_1$ to $T_2 - 90\text{d}$) | Hyperparameter tuning, threshold selection, and probability calibration. | Completely locked out of training folds; tuned via rolling-origin TimeSeriesSplit. |
| **Locked Test Split** | Months 20 through 24<br>(Cohorts $\ge T_2$) | Final unbiased performance evaluation and audit. | Touched exactly once after all models, thresholds, and calibrators are locked. |

```
Time Axis ─────────────────────────────────────────────────────────────────────────────►
[── Train Cohorts (M1-12) ──] [ 90d Embargo ] [── Valid Cohorts (M14-18) ──] [ 90d Embargo ] [── Locked Test (M20-24) ──]
```

### 9.2 Cross-Validation, Class Imbalance & Statistical Comparison
- **Rolling-Origin Cross-Validation:** Hyperparameter searches inside the training window use `TimeSeriesSplit(n_splits=4)`, expanding training horizons chronologically to simulate quarterly production redeployments.
- **Pipeline-Encapsulated Preprocessing:** All transformations (median imputation, scaling, one-hot encoding, and target encoding) are encapsulated within `Pipeline` objects, preventing test fold parameters from leaking into training folds.
- **Cost-Sensitive Imbalance Handling:** Avoids synthetic oversampling (which distorts posterior probability calibration). Instead, cost-sensitive loss weighting (`scale_pos_weight` in XGBoost, `class_weight='balanced'` in Logistic Regression) is paired with post-hoc Isotonic Calibration on held-out slices.
- **Statistical Significance Testing:** Model superiority is established using 1,000-iteration Stratified Bootstrap Confidence Intervals on PR-AUC and Lift@10%. Pairwise model comparisons utilize the DeLong test for correlated ROC curves and McNemar's test for paired classification discordance on 2x2 decision contingency tables.

### 9.3 Ten-Point Robustness Audit Matrix

| Robustness Test | Experimental Perturbation | Acceptance / Failure Criterion |
|---|---|---|
| **1. Label Window Sensitivity** | Vary repeat horizon across 30, 60, 90, and 120 days. | Top feature rankings and odds ratios must remain directionally stable. |
| **2. Prediction Point Shift** | Compare Model A (Day 0, post-purchase) vs Model B (Day 30). | Model B must capture incremental value from early engagement and reviews. |
| **3. Data Pipeline Stress** | Toggle outlier clipping (1st/99th percentile) & imputation rules. | Model rankings and ROC-AUC must not fluctuate by > 0.02. |
| **4. Evidence Tier Separation** | Evaluate performance on Real Retail vs Synthetic logs. | Synthetic gains must not be used to justify production deployment on real data. |
| **5. Temporal Drift Tracking** | Evaluate PR-AUC cohort-by-cohort across trailing 12 months. | No steady deterioration; sharp drops identify seasonal shifts (e.g., Black Friday). |
| **6. Subgroup Fairness Slicing** | Segment validation metrics by Skin Type, Tone, and Channel. | Recall parity must not degrade by > 15% on sensitive customer demographics. |
| **7. Feature Ablation Study** | Systematically drop feature groups (Chemical, NLP, RFM). | Identifies true performance drivers and confirms value of Week 4 features. |
| **8. Random-Label Control** | Permute target labels randomly across training instances. | Model performance must collapse precisely to random chance (AUC = 0.50). |
| **9. Shuffled-Time Control** | Randomly permute timestamp order of customer events. | If performance does not collapse, lookahead temporal leakage exists. |
| **10. Random Seed Stability** | Train models across 10 distinct initialization seeds. | Standard deviation of PR-AUC must remain below 0.005. |

---

## 10. Pitfalls and Mitigation: Overfitting, Underfitting and Beyond

| Identified Pitfall | Observable Symptoms | Root Causes in Skincare | Engineered Mitigation Controls |
|---|---|---|---|
| **Severe Overfitting** | Training PR-AUC > 0.95; Validation PR-AUC < 0.35; wildly unstable feature importance. | Deep tree architectures; high-dimensional sparse ingredient one-hots; excessive boosting rounds. | Enforce L1/L2 regularization; restrict `max_depth` to 3–5; set `min_samples_leaf` $\ge 30$; early stopping on PR-AUC. |
| **Structural Underfitting** | Both training and validation PR-AUC plateau at low values (~0.25). | Linear model failing to capture vital active ingredient $\times$ skin type interactions; over-regularization. | Transition to gradient boosting; engineer explicit domain interactions (`due_ratio`, skin-match scores). |
| **Data & Target Leakage** | Suspiciously flawless test performance (AUC > 0.94); rapid collapse upon deployment. | Features incorporating post-day-0 actions (e.g., total lifetime orders); fitting scalers across all data. | Timestamp audits on all columns; encapsulate pipelines; perform shuffled-time permutation checks. |
| **Review Selection Bias** | NLP features perform strongly on reviewers but fail across population. | Only 10–15% of buyers submit reviews; reviewers represent polarized extremes (ecstatic or irritated). | Create explicit missingness indicator (`has_reviewed`); treat review features as conditional booster. |
| **Probability Distortion** | Predicted probabilities clustered near 0 or 1; severe Brier loss. | Class weighting or SMOTE artificially warping the prior odds ratio. | Apply post-hoc Isotonic Regression or Sigmoid Platt Scaling on held-out calibration cohorts. |
| **Prediction vs. Causality Conflation** | High predicted repeat customers convert regardless of marketing contact (*"Sure Things"*). | Targeting high-risk scores wastes budget on organic repeaters while spamming them. | Implement Uplift Modeling (Class Transformation / T-Learner) to isolate incremental campaign lift. |
| **Cold-Start Collapse** | Recommender fails completely for newly registered first-time buyers. | Matrix factorization requires non-zero interaction histories in the collaborative matrix. | Deploy hybrid LightFM leveraging INCI ingredient metadata; fall back to bestsellers by category. |

---

## 11. Production Modeling Pipelines (Illustrative Python Code)

### 11.1 Time-Based Cohort Split with 90-Day Embargo
```python
import numpy as np, pandas as pd
from pandas import Timedelta

# Ensure chronological sorting by initial customer transaction
cust = cust.sort_values("first_order_date").reset_index(drop=True)
t1 = cust["first_order_date"].quantile(0.60)
t2 = cust["first_order_date"].quantile(0.80)
embargo = Timedelta(days=90)  # Label resolution window

# Partition splits ensuring complete temporal isolation
train = cust[cust["first_order_date"] < (t1 - embargo)]
valid = cust[(cust["first_order_date"] >= t1) & (cust["first_order_date"] < (t2 - embargo))]
test  = cust[cust["first_order_date"] >= t2]

feature_cols = [c for c in cust.columns if c not in ["customer_id", "first_order_date", "repeat_90d"]]
X_train, y_train = train[feature_cols], train["repeat_90d"].to_numpy()
X_valid, y_valid = valid[feature_cols], valid["repeat_90d"].to_numpy()
X_test,  y_test  = test[feature_cols],  test["repeat_90d"].to_numpy()
```

### 11.2 Preprocessing Pipelines and Candidate Estimators
```python
from sklearn.compose import ColumnTransformer
from sklearn.pipeline import Pipeline
from sklearn.impute import SimpleImputer
from sklearn.preprocessing import StandardScaler, OneHotEncoder, OrdinalEncoder, FunctionTransformer
from sklearn.linear_model import LogisticRegression
from sklearn.ensemble import RandomForestClassifier, HistGradientBoostingClassifier

num_cols = ["spend_30d", "aov", "price_per_ml", "discount_share", "irritant_load", "due_ratio"]
cat_cols = ["first_category", "skin_type", "acquisition_channel"]

# Linear pipeline: log transforms skew, imputes with indicators, standardizes
pre_linear = ColumnTransformer([
    ("num", Pipeline([
        ("log1p", FunctionTransformer(np.log1p, validate=False)),
        ("imp", SimpleImputer(strategy="median", add_indicator=True)),
        ("scaler", StandardScaler())
    ]), num_cols),
    ("cat", Pipeline([
        ("imp", SimpleImputer(strategy="constant", fill_value="missing")),
        ("ohe", OneHotEncoder(handle_unknown="ignore", sparse_output=False))
    ]), cat_cols)
])

# Tree pipeline: preserves numeric values, applies ordinal encoding
pre_tree = ColumnTransformer([
    ("num", SimpleImputer(strategy="median", add_indicator=True), num_cols),
    ("cat", Pipeline([
        ("imp", SimpleImputer(strategy="constant", fill_value="missing")),
        ("oe", OrdinalEncoder(handle_unknown="use_encoded_value", unknown_value=-1))
    ]), cat_cols)
])

models = {
    "Logistic_Baseline": Pipeline([
        ("pre", pre_linear),
        ("clf", LogisticRegression(penalty="l2", C=0.5, class_weight="balanced", solver="lbfgs", max_iter=1000))
    ]),
    "Random_Forest": Pipeline([
        ("pre", pre_tree),
        ("clf", RandomForestClassifier(n_estimators=500, min_samples_leaf=25, class_weight="balanced_subsample",
                                       max_features="sqrt", random_state=42, n_jobs=-1))
    ]),
    "HistGradientBoosting": Pipeline([
        ("pre", pre_tree),
        ("clf", HistGradientBoostingClassifier(learning_rate=0.04, max_leaf_nodes=20, min_samples_leaf=40,
                                               l2_regularization=1.5, early_stopping=True, n_iter_no_change=25,
                                               random_state=42))
    ])
}
```

### 11.3 Hyperparameter Tuning via Rolling TimeSeriesSplit
```python
from sklearn.model_selection import RandomizedSearchCV, TimeSeriesSplit

param_dist = {
    "clf__learning_rate": [0.02, 0.04, 0.08],
    "clf__max_leaf_nodes": [15, 25, 35],
    "clf__min_samples_leaf": [25, 50, 100],
    "clf__l2_regularization": [0.5, 1.5, 5.0]
}

tscv = TimeSeriesSplit(n_splits=4)
search = RandomizedSearchCV(
    estimator=models["HistGradientBoosting"],
    param_distributions=param_dist,
    n_iter=20,
    cv=tscv,
    scoring="average_precision",
    random_state=42,
    n_jobs=-1
)
search.fit(X_train, y_train)
best_hgb = search.best_estimator_
```

### 11.4 Comprehensive Evaluation, Bootstrap CIs & Profit Curves
```python
from sklearn.metrics import roc_auc_score, average_precision_score, brier_score_loss

def evaluate_predictions(y_true, y_prob, top_pct=0.20):
    y_true = np.asarray(y_true)
    order = np.argsort(-y_prob)
    n_top = int(len(y_true) * top_pct)
    return {
        "ROC_AUC": roc_auc_score(y_true, y_prob),
        "PR_AUC": average_precision_score(y_true, y_prob),
        "Brier_Loss": brier_score_loss(y_true, y_prob),
        "Lift@20%": y_true[order[:n_top]].mean() / y_true.mean(),
        "Base_Repeat_Rate": y_true.mean()
    }

def compute_bootstrap_ci(y_true, y_prob, metric_fn=average_precision_score, n_boot=1000, seed=42):
    rng = np.random.default_rng(seed)
    y_true, y_prob = np.asarray(y_true), np.asarray(y_prob)
    scores = []
    for _ in range(n_boot):
        idx = rng.integers(0, len(y_true), len(y_true))
        if len(np.unique(y_true[idx])) > 1:
            scores.append(metric_fn(y_true[idx], y_prob[idx]))
    return np.percentile(scores, [2.5, 97.5])

def compute_profit_curve(y_true, y_prob, unit_margin=28.0, unit_contact_cost=1.50):
    order = np.argsort(-y_prob)
    y_sorted = np.asarray(y_true)[order]
    cumulative_conversions = np.cumsum(y_sorted)
    cumulative_contacts = np.arange(1, len(y_sorted) + 1)
    net_profit = (cumulative_conversions * unit_margin) - (cumulative_contacts * unit_contact_cost)
    optimal_idx = np.argmax(net_profit)
    return net_profit, optimal_idx
```

### 11.5 Probability Calibration & TreeSHAP Attribution
```python
from sklearn.calibration import CalibratedClassifierCV, calibration_curve
import shap, xgboost as xgb

# Isotonic probability calibration
calibrator = CalibratedClassifierCV(estimator=best_hgb, method="isotonic", cv="prefit")
calibrator.fit(X_valid, y_valid)
calibrated_test_probs = calibrator.predict_proba(X_test)[:, 1]

# TreeSHAP Global & Local Attribution
X_tr_t = pre_tree.fit_transform(X_train)
X_va_t = pre_tree.transform(X_valid)

xgb_model = xgb.XGBClassifier(
    n_estimators=1000, learning_rate=0.03, max_depth=4, subsample=0.8, colsample_bytree=0.8,
    reg_lambda=3.0, eval_metric="aucpr", early_stopping_rounds=40,
    scale_pos_weight=(y_train == 0).sum() / (y_train == 1).sum(), random_state=42
)
xgb_model.fit(X_tr_t, y_train, eval_set=[(X_va_t, y_valid)], verbose=False)

explainer = shap.TreeExplainer(xgb_model)
shap_values = explainer.shap_values(X_va_t)
shap.summary_plot(shap_values, X_va_t, feature_names=pre_tree.get_feature_names_out())
```

### 11.6 Survival (Cox PH), CLV (Lifetimes) & Item-Based Recommenders
```python
from lifelines import CoxPHFitter
from lifetimes import BetaGeoFitter, GammaGammaFitter
from lifetimes.utils import summary_data_from_transaction_data
from scipy.sparse import csr_matrix
from sklearn.preprocessing import normalize

# Task T2: Cox Proportional Hazards
cph = CoxPHFitter(penalizer=0.1)
cph.fit(survival_df[["tenure_days", "repeat_event"] + surv_features],
        duration_col="tenure_days", event_col="repeat_event")
cph.print_summary()

# Task T3: BG/NBD + Gamma-Gamma
rfm = summary_data_from_transaction_data(orders_df, "customer_id", "order_date",
                                         monetary_value_col="order_net_total", observation_period_end=cutoff_date)
bgf = BetaGeoFitter(penalizer_coef=0.02).fit(rfm["frequency"], rfm["recency"], rfm["T"])
rfm["expected_purchases_90d"] = bgf.conditional_expected_number_of_purchases_up_to_time(90, rfm["frequency"], rfm["recency"], rfm["T"])
repeat_cust = rfm[rfm["frequency"] > 0]
ggf = GammaGammaFitter(penalizer_coef=0.02).fit(repeat_cust["frequency"], repeat_cust["monetary_value"])

# Task T4: Item-Based Cosine Recommender
user_item_mat = csr_matrix((np.ones(len(interactions)), (interactions["user_idx"], interactions["item_idx"])))
norm_items = normalize(user_item_mat.T.tocsr(), axis=1)
item_similarity = norm_items @ norm_items.T

def generate_recommendations(user_idx, top_k=5):
    user_history = user_item_mat[user_idx]
    scores = (user_history @ item_similarity).toarray().ravel()
    scores[user_history.indices] = -np.inf
    return np.argsort(-scores)[:top_k]
```

---

## 12. Interpreting Model Outcomes for Business Insight & CRM Decisioning

### 12.1 Strategic Toolkit for Stakeholder Interpretation

| Model Artifact | Technical Information Captured | Executive Commercial Translation |
|---|---|---|
| **Logistic Odds Ratios ($\exp(w)$)** | Controlled multiplicative effect of unit feature changes on repeat odds. | *"Holding spend constant, adding a coordinating toner multiplies the odds of 90-day repeat by 1.38x."* |
| **TreeSHAP Summary Plots** | Directional distribution and magnitude of feature impacts across cohorts. | Reveals tipping points: e.g., discount shares above 40% aggressively degrade long-term customer loyalty. |
| **Individual SHAP Waterfall** | Local decomposition of baseline log-odds to individual customer score. | Empowers CRM agents to see why a specific VIP flagged as high churn risk (e.g., severe fulfillment delay). |
| **Partial Dependence Plots (PDP)** | Marginal effect of continuous features on repeat probability. | Visualizes saturation thresholds: replenishment alerts yield peak return when sent at 85–95% due ratio. |
| **Cox Hazard Ratios (HR)** | Relative velocity of repurchase across product categories. | *"Moisturizer buyers reorder 1.6x faster than sunscreen buyers; separate trigger journeys by 20 days."* |
| **Decile Lift & Gain Curves** | Cumulative repeat captures across ranked customer deciles. | Enables CFO and VP of Marketing to set campaign cutoffs based on diminishing marginal ROI. |
| **GMM Cluster Archetypes** | Multi-dimensional behavioral and ingredient preference profiles. | Directs creative messaging: *"Routine Simplifiers"* receive gentle duo bundles; *"Actives Junkies"* receive AHA/BHA drops. |

### 12.2 Operational Decision Framework: Dynamic CRM Tiering

```
Calibrated Repeat Score P(Repeat)
  │
  ├──► P > 0.65  ──────► Tier 1: Organic Loyalists
  │                       • Zero discount dilution (preserve margin)
  │                       • VIP loyalty invites & early launch access
  │                       • Regimen-builder masterclasses
  │
  ├──► 0.25 to 0.65 ───► Tier 2: The Persuadable Core (Primary CRM Target)
  │                       • Precision replenishment email/SMS at due_ratio ~ 0.90
  │                       • Routine-completion cross-sell (e.g., complimentary deluxe sample)
  │                       • Personalized education on first-purchase active ingredients
  │
  └──► P < 0.25  ──────► Tier 3: Churn & Formula Fit Risk
                          • Zero expensive marketing spend
                          • Automated digital survey to diagnose dissatisfaction
                          • If formula irritation detected (T6): barrier repair support
```

### 12.3 Formal Business Value Optimization Formula

$$\max_{\theta} E[\text{Profit}(\theta)] = \sum_{i: p_i \ge \theta} \left[ \tau_i \cdot \text{Margin}_i - \text{Cost}_{\text{contact}} - (1 - \tau_i) \cdot \text{Cost}_{\text{unsubscribe}} \right]$$

- $p_i$: Calibrated predicted repeat probability from Model B.
- $\tau_i$: Incremental uplift probability ($\tau_i = P(\text{Repeat} \mid \text{Contact}) - P(\text{Repeat} \mid \text{No Contact})$).
- $\text{Margin}_i$: Expected net contribution margin of the subsequent transaction.
- $\text{Cost}_{\text{contact}}$: Direct SMS/email delivery cost plus promotional sample expense.
- $\text{Cost}_{\text{unsubscribe}}$: Long-term discounted customer lifetime penalty of brand fatigue and opt-outs.

---

## 13. Ethical Governance, Fairness Auditing & Data Privacy

1. **Disparate Impact & Sliced Fairness Auditing:** Model performance (False Negative Rates and Recall parity) is systematically audited across skin tones (Fitzpatrick scale proxies), self-reported skin concerns (acne-prone, rosacea), and age brackets. Predictive models are strictly prohibited from using sensitive demographic markers to deny premium service, warranties, or access to promotions.
2. **Medical Claim Boundary Enforcement:** Natural Language Generation (NLG) and automated recommendation copy are strictly filtered via regularized safety dictionaries to prevent making illegal dermatological or medical claims (e.g., claiming to *"cure eczema"* or *"eliminate cystic acne"*) in violation of FDA/FTC cosmetics regulations.
3. **Data Minimization & Customer Consent (GDPR/CCPA):** Dermatological quiz responses and skin sensitivity markers are classified as Sensitive Personal Data. Explicit, granular opt-in consent is enforced, customer identifiers are hashed using SHA-256 with rotating salts, and one-click consent revocation immediately removes profiles from algorithmic training corpora.
4. **Standardized Model Cards:** Every deployed artifact is paired with a standardized Model Card documenting intended commercial use cases, out-of-scope deployments, training data provenance, evaluation metrics across fairness slices, and mandated quarterly drift retraining schedules.

---

## 14. Implementation Roadmap for Weeks 6 Through 8

- **Week 6 (Model Training, Tuning & Calibration):** Train baselines, Logistic Regression, Random Forest, and HistGradientBoosting on embargoed time splits; execute Optuna/TimeSeriesSplit hyperparameter searches; apply Isotonic calibration; construct empirical profit curves and select operating thresholds.
- **Week 7 (Survival, CLV & Model Explainability):** Fit Cox Proportional Hazards and BG/NBD + Gamma-Gamma models; train Tweedie/Hurdle CLV regressors; compute global/local TreeSHAP values, interaction matrices, and Partial Dependence curves; extract customer cluster personas via K-Means and GMM.
- **Week 8 (Recommender Systems, Audits & Synthesis):** Train implicit ALS and hybrid LightFM recommenders; conduct cold-start evaluation; execute 10-point robustness matrix, demographic fairness audits, and feature ablation studies; finalize technical documentation and executive presentation.

---

## 15. Time Allocation (35 Hours Total Effort)

| Phase / Analytical Activity | Hours | Methodological Focus & Key Deliverable |
|---|:---:|---|
| **1. Problem Review & Multi-Task Formulation** | 3.0 | Mapping Week 1–4 business problems to Tasks T1–T6. |
| **2. Library Research & Algorithmic Benchmarking** | 8.0 | Auditing scikit-learn, XGBoost, lifelines, lifetimes, and implicit. |
| **3. Comparative Rationale & Decision Formulation** | 5.0 | Drafting 10-criterion comparison matrix and parsimony rules. |
| **4. Metric Design & Expected Performance Hypotheses** | 4.0 | Designing PR-AUC, lift curves, and grounding performance ranges. |
| **5. Validation Architecture & Leakage Defense Design** | 4.0 | Formulating temporal embargo split, rolling CV, and 10 robustness tests. |
| **6. Prototype Pipeline Scripting & Synthetic Verification** | 5.0 | Developing and verifying Section 11 Python code blocks. |
| **7. Prescriptive CRM Operationalization & Value Formula** | 2.0 | Translating SHAP and hazards into commercial decision rules. |
| **8. Technical Document Synthesis & Polish** | 4.0 | Comprehensive structuring, formatting, and proofing. |
| **TOTAL ESTIMATED ANALYTICAL EFFORT** | **35.0** | **Full Week 5 Deliverable Scope Achieved** |

---

## 16. Conclusion

The predictive modeling strategy formulated in this deliverable matches each core retail challenge to a mathematically suited machine learning algorithm: **Gradient Boosted Decision Trees** benchmarked against **Regularized Logistic Regression** for repeat purchase propensity (T1); **Cox Proportional Hazards** for physical replenishment timing (T2); **BG/NBD with Gamma-Gamma** and **Two-Part Hurdle GBDT** regressors for customer value (T3); **K-Means++ and GMM** for dermatological customer typologies (T5); and **Hybrid LightFM** for personalized routine cross-selling (T4).

By anchoring evaluations to PR-AUC, Top-Decile Lift, and the empirical profit curve, and enforcing a strict 90-day temporal embargo validation design, the framework defends against lookahead leakage and optimistic overfitting. Integrated TreeSHAP attribution and dynamic CRM routing transform mathematical scores into concrete business decisions, laying the foundation for empirical execution in Weeks 6 through 8.

---

### Academic & Industry Literature References

1. **Breiman, L. (2001).** Random forests. *Machine Learning*, 45(1), 5–32.
2. **Cawley, G. C., & Talbot, N. L. C. (2010).** On over-fitting in model selection and subsequent selection bias in performance evaluation. *Journal of Machine Learning Research*, 11, 2079–2107.
3. **Chen, T., & Guestrin, C. (2016).** XGBoost: A scalable tree boosting system. In *Proceedings of the 22nd ACM SIGKDD International Conference on Knowledge Discovery and Data Mining* (pp. 785–794).
4. **Cox, D. R. (1972).** Regression models and life-tables. *Journal of the Royal Statistical Society: Series B (Methodological)*, 34(2), 187–220.
5. **DeLong, E. R., DeLong, D. M., & Clarke-Pearson, D. L. (1988).** Comparing the areas under two or more correlated receiver operating characteristic curves: a nonparametric approach. *Biometrics*, 44(3), 837–845.
6. **Dietterich, T. G. (1998).** Approximate statistical tests for comparing supervised classification learning algorithms. *Neural Computation*, 10(7), 1895–1923.
7. **Fader, P. S., Hardie, B. G., & Lee, K. L. (2005).** "Counting your customers" the easy way: An alternative to the Pareto/NBD model. *Marketing Science*, 24(2), 275–284.
8. **Fader, P. S., & Hardie, B. G. (2013).** The Gamma-Gamma model of monetary value. Technical Note, Wharton School of Business.
9. **Friedman, J. H. (2001).** Greedy function approximation: a gradient boosting machine. *Annals of Statistics*, 29(5), 1189–1232.
10. **Hu, Y., Koren, Y., & Volinsky, C. (2008).** Collaborative filtering for implicit feedback datasets. In *IEEE International Conference on Data Mining (ICDM)* (pp. 263–272).
11. **Kapoor, S., & Narayanan, A. (2023).** Leakage and the reproducibility crisis in machine-learning-based science. *Patterns*, 4(9), 100804.
12. **Kaufman, S., Rosset, S., Perlich, C., & Stitelman, O. (2012).** Leakage in data mining: Formulation, detection, and avoidance. *ACM Transactions on Knowledge Discovery from Data (TKDD)*, 6(4), 1–21.
13. **Ke, G., Meng, Q., Finley, T., Wang, T., Chen, W., Ma, W., ... & Liu, T. Y. (2017).** LightGBM: A highly efficient gradient boosting decision tree. *Advances in Neural Information Processing Systems (NeurIPS)*, 30, 3146–3154.
14. **Koren, Y., Bell, R., & Volinsky, C. (2009).** Matrix factorization techniques for recommender systems. *Computer*, 42(8), 30–37.
15. **Lundberg, S. M., & Lee, S. I. (2017).** A unified approach to interpreting model predictions. *Advances in Neural Information Processing Systems (NeurIPS)*, 30, 4765–4774.
16. **Niculescu-Mizil, A., & Caruana, R. (2005).** Predicting good probabilities with supervised learning. In *Proceedings of the 22nd International Conference on Machine Learning (ICML)* (pp. 625–632).
17. **Radcliffe, N. J., & Surry, P. D. (2011).** Real-world uplift modelling with significance-based uplift trees. *Stochastic Solutions White Paper*.
18. **Saito, T., & Rehmsmeier, M. (2015).** The precision-recall plot is more informative than the ROC plot on imbalanced datasets. *PLoS ONE*, 10(3), e0118432.
19. **Tibshirani, R. (1996).** Regression shrinkage and selection via the lasso. *Journal of the Royal Statistical Society: Series B (Methodological)*, 58(1), 267–288.
20. **Zou, H., & Hastie, T. (2005).** Regularization and variable selection via the elastic net. *Journal of the Royal Statistical Society: Series B (Statistical Methodology)*, 67(2), 301–320.
