# Week 4 Deliverable: Exploratory Data Analysis and Feature Engineering

> **Project:** Predicting Repeat Purchase and Personalizing Product Recommendations in Online Skincare  
> **Week:** 4 of 8  
> **Estimated Hours:** 35 hours

---

## 1. Purpose and Link to Weeks 1 to 3

Week 1 defined the problem (predict repeat purchase and recommend products in online skincare), Week 2 identified the data sources, and Week 3 designed the cleaning pipeline that produces tables in `data/processed/`. This document plans what happens next: a structured **exploratory data analysis (EDA)** of those tables and a **feature engineering strategy** that turns them into model-ready variables.

The EDA has three jobs:
1. Describe the data and confirm it is fit for purpose.
2. Give a first, honest test of the Week 1 hypotheses (H1 to H7).
3. Show which variables deserve to become features.

The feature engineering plan then converts what the EDA reveals into variables that capture customer behaviour, product fit and market trends.

> ⚠️ **Working rules:**
> - Explore on the **training period only** (earlier cohorts); keep later cohorts untouched for final testing so EDA does not leak into evaluation.
> - Separate **description** (what is in the data) from **inference** (what may be true in general); report effect sizes and confidence intervals, not only p-values.
> - **Control the false discovery rate** when many tests are run (Benjamini-Hochberg).
> - Label every result with its **evidence tier:** real beauty data, real non-beauty analogue, or synthetic.
> - All figures quoted in Section 12 (hypothetical scenario) are illustrative only and not actual findings.

---

## 2. EDA Workflow

| Step | Question | Main Output | Hypotheses Served |
|------|----------|-------------|-------------------|
| 1. Data audit | Is the processed data complete, consistent and the size expected? | Row counts, missingness map, quality note | All |
| 2. Univariate profiling | What does each variable look like alone? | Distribution plots and summary table | All |
| 3. Target analysis | How common is repeat purchase, and how does it vary over time? | Baseline repeat rate, cohort retention heatmap, survival curves | H1, H2 |
| 4. Bivariate analysis | Which variables differ between repeaters and non-repeaters? | Group comparisons, tests, effect sizes | H1–H5 |
| 5. Multivariate analysis | How do variables combine and overlap? | Correlation maps, PCA, clustering, interaction plots | H2, H4, H6 |
| 6. Text and ingredient exploration | What do reviews say and which ingredients matter? | Topics, n-grams, sentiment checks, ingredient co-occurrence | H3, H5, H6 |
| 7. Trend and seasonality analysis | Do search interest and seasons move demand? | Decomposition plots, spike comparisons | H7 |
| 8. Fairness and bias slices | Do patterns differ by skin type, skin tone, region or data source? | Sliced charts and tables | Cross-cutting |
| 9. Synthesis | What did we learn, and what should become features? | Findings table, feature shortlist, updated hypotheses | All |

---

## 3. Basic EDA Techniques and How They Apply

### 3.1 Univariate Analysis

| Variable Type | Technique | Beauty & Wellness Example | What to Look For |
|--------------|-----------|--------------------------|------------------|
| Numeric | Histogram, KDE, box plot, summary statistics (mean, median, SD, skewness, kurtosis) | Price, price per ml, order value, review length | Skew (prices are usually right-skewed), heavy tails, impossible values |
| Categorical | Frequency bar chart, proportion table | Category, brand, skin type, channel | Rare categories to group, dominance by a few brands |
| Ordinal | Bar chart of ratings | 1–5 star ratings | J-shaped distribution (many 5 and 1 star), positivity bias |
| Binary | Proportion with confidence interval | Repeat flag, "would recommend", has ingredients | Class imbalance |
| Date and time | Line chart of counts per week or month | Orders per month, reviews per month | Seasonality, launch spikes, gaps |
| Text | Length distribution, top words and bigrams | Review text | Language mix, boilerplate, spam |

### 3.2 Bivariate Analysis

| Pair | Technique | Example |
|------|-----------|---------|
| Numeric vs numeric | Scatter plot with trend line, Spearman correlation | Price per ml vs rating |
| Numeric vs binary target | Box or violin plot by group, Mann-Whitney U test | Discount share by repeat flag |
| Categorical vs binary target | Stacked or grouped bar, chi-square test, Cramer's V | First-purchase category vs repeat |
| Categorical vs numeric | Box plot, Kruskal-Wallis test | Rating by skin type |
| Time vs numeric | Line chart with smoothing | Weekly search interest for retinol |

### 3.3 Multivariate Analysis

- **Correlation matrix** (Spearman for skewed or ordinal data) with hierarchical ordering to reveal blocks of related variables.
- **Stratified analysis** to avoid Simpson's paradox: for example, a discount may look harmful overall but neutral within each category, because discounted items are mostly gift sets.
- **Faceted plots** (small multiples) of the repeat rate by category, price tier and season.
- **Cross-tabulation heatmaps** of repeat rate by two variables at once (category by skin type).

---

## 4. Advanced EDA Techniques for Multi-Dimensional Data

| Technique | Purpose | Beauty & Wellness Use | Tool |
|-----------|---------|----------------------|------|
| Principal Component Analysis (PCA) | Compress correlated numeric variables and see structure | Reduce RFM, basket and price features to a few axes; check which features move together | `scikit-learn` |
| UMAP or t-SNE | Non-linear 2D map of high-dimensional data | Map products by ingredient vectors; map customers by behaviour | `umap-learn`, `scikit-learn` |
| Clustering (k-means, hierarchical, GMM, DBSCAN) | Discover natural groups | Customer segments (routine builders, deal seekers, trend chasers, one-time gifters); product groups | `scikit-learn` |
| Mutual information and Cramer's V | Detect non-linear and categorical associations | Which categorical variables relate to repeat | `scikit-learn`, `SciPy` |
| Association rule mining (Apriori, FP-growth) | Find items or ingredients that occur together | Products bought together; ingredient pairs in top-rated products | `mlxtend` |
| Cohort and survival analysis | Retention over time with censoring | Kaplan-Meier curves of time to second purchase by first category | `lifelines` |
| Time-series decomposition and autocorrelation | Separate trend, season and noise | Sunscreen seasonality; viral ingredient spikes | `statsmodels` |
| Topic modelling (LDA, BERTopic) | Themes in text | Irritation, hydration, fragrance, packaging, texture | `gensim`, `BERTopic` |
| Aspect and sentiment analysis | Opinion about specific aspects | Sentiment about texture vs price vs results | `VADER`, `transformers` |
| Network analysis | Structure of relationships | Ingredient co-occurrence network; product similarity graph | `NetworkX` |
| Anomaly detection | Unusual accounts or products | Review bursts, bot-like reviewers | `Isolation Forest` |
| Missingness analysis | Whether gaps carry information | Is missing skin type linked to lower repeat? | `missingno`, `pandas` |
| Partial dependence pre-checks | Rough shape of relationships | Repeat rate vs discount share in bins | `pandas`, `matplotlib` |
| Fairness slicing | Check patterns by subgroup | Rating and repeat by skin tone and skin type | `pandas` |

---

## 5. Statistical Methods Plan

| Question | Method | Assumptions and Notes | Library |
|----------|--------|----------------------|---------|
| Does repeat rate differ across categories (H1)? | Chi-square test of independence; Cramer's V; follow-up pairwise tests with correction | Expected counts > 5 per cell; otherwise Fisher exact | `SciPy` |
| Does discount share differ between repeaters and non-repeaters (H2)? | Mann-Whitney U with rank-biserial correlation; logistic regression with controls | Non-normal data; independence of customers | `SciPy`, `statsmodels` |
| Does review timing or rating relate to repeat (H3)? | Logistic regression; Kaplan-Meier with log-rank test | Right-censoring handled | `statsmodels`, `lifelines` |
| Do irritation topics predict lower ratings (H5)? | Spearman correlation; ordinal or linear regression of rating on topic shares | Topics are compositional, so drop one | `statsmodels` |
| Do ratings differ by skin type? | Kruskal-Wallis with Dunn post-hoc tests | Many groups; correct for multiple tests | `SciPy`, `scikit-posthocs` |
| Do VADER sentiment and ratings agree? | Spearman correlation; confusion table of sentiment class vs rating class | Ratings are ordinal | `SciPy` |
| Is demand seasonal or spiking (H7)? | Seasonal decomposition; autocorrelation; compare spike and steady groups with Mann-Whitney | Weekly regularity | `statsmodels` |
| Are features collinear? | Variance inflation factor; Spearman clusters | VIF > ~5 to 10 flags a problem | `statsmodels` |
| Are distributions normal? | Shapiro-Wilk (small samples), Q-Q plots | With large n, tests flag tiny deviations — rely on plots | `SciPy` |
| How uncertain is a rate or difference? | Bootstrap confidence intervals | Resample customers, not rows | `NumPy`, `SciPy` |
| Many tests at once | Benjamini-Hochberg false discovery rate | Report adjusted p-values | `statsmodels` |

---

## 6. Visualization Plan

### 6.1 Library Roles

| Library | Role | Why |
|---------|------|-----|
| `matplotlib` | Base plotting, full control of layout and annotations | Publication-quality figures; foundation for other libraries |
| `seaborn` | Statistical charts: distributions, box/violin plots, heatmaps, pair plots, faceting | Concise syntax and good defaults for grouped comparisons |
| `SciPy (stats)` | Tests and effect sizes shown on charts | Standard statistical functions |
| `statsmodels` | Decomposition, regression diagnostics, multiple-testing correction | Time-series and inference tools |
| `plotly` | Interactive charts (hover, zoom) for exploration and dashboards | Useful for UMAP maps and treemaps |
| `lifelines` | Kaplan-Meier curves, log-rank tests | Survival analysis with censoring |
| `missingno` | Missing-value matrix and heatmap | Quick view of patterns in gaps |
| `wordcloud` / `pyLDAvis` / BERTopic visuals | Text exploration | Quick look at vocabulary and topics |
| `NetworkX` | Ingredient and product networks | Graph layout and metrics |
| `pandas` (`.plot`, `.style`) | Fast first-look charts and formatted tables | Convenience during exploration |

### 6.2 Figure Catalogue

| ID | Figure | Chart Type | Variables | Insight Sought | Hypothesis |
|----|--------|-----------|-----------|---------------|------------|
| F1 | Rating distribution | Bar chart | `rating` | Positivity bias | General |
| F2 | Price and price-per-ml distributions | Histogram (log scale), box plot by category | `price`, `size_ml`, `category` | Skew, price tiers | H2 |
| F3 | Missingness map | Matrix, bar chart | All columns | Which fields are reliable | General |
| F4 | Cohort retention heatmap | Heatmap | Cohort month, months since first order | Retention curve shape and cohort quality | H1 |
| F5 | Time to second purchase | Kaplan-Meier curves by first category | `days`, `event`, `category` | Replenishment timing | H1 |
| F6 | Repeat rate by first-purchase category | Bar chart with confidence intervals | `category`, `repeat` | Core vs gift vs trend items | H1 |
| F7 | Repeat rate by discount share bin | Line or bar by bin, faceted by category | `discount_share`, `repeat` | Deal-driven customers | H2 |
| F8 | RFM segments | Scatter, segment profile bars | `recency`, `frequency`, `monetary` | Customer groups | RQ2 |
| F9 | Review delay and rating vs repeat | Box plots, survival curves | `review_delay`, `rating` | Early engagement | H3 |
| F10 | Sentiment vs rating | Box plot of VADER score by star rating | `sentiment`, `rating` | Text signal validity | H4 |
| F11 | Topic prevalence by rating group | Stacked bars | Topic shares | Reasons for dissatisfaction | H5 |
| F12 | Rating by skin type and tone | Violin or box plot | `rating`, `skin_type`, `tone` | Fit differences and fairness | H5, Fairness |
| F13 | Ingredient frequency and co-occurrence | Bar chart, heatmap or network | Ingredient flags | Common actives and pairs | H6 |
| F14 | Product map | UMAP scatter coloured by category | Ingredient vectors | Similar formulas | H6 |
| F15 | Search interest and decomposition | Line charts, decomposition | Google Trends, week | Seasonality and spikes | H7 |
| F16 | Spike vs steady products | Box plot of repeat proxy by spike group | `spike_index`, `repeat_proxy` | Trend items and loyalty | H7 |
| F17 | Correlation heatmap | Clustered heatmap | Numeric features | Collinearity, feature groups | All |
| F18 | PCA scree and loadings | Line plot, bar chart | Numeric features | Dimension structure | All |

### 6.3 Visual Design Rules

- One message per chart, with a **title that states the finding**.
- **Colour-blind-safe palettes** (e.g., `seaborn colorblind`), consistent colours for the same category across charts.
- Label axes with units, use **log scales** for skewed money values, show sample sizes (n) and confidence intervals.
- Avoid pie charts and 3D charts; **order bars by value**; limit categories to ~10 and group the rest as "Other".
- Save figures programmatically (`fig.savefig(..., dpi=200)`) with fixed names so the report can be rebuilt.
- **Annotate statistical results** on the chart (effect size and adjusted p-value) where helpful.

---

## 7. Draft EDA Code

> Adapt column names to the real files after inspecting them in Week 4.

### 7.1 Cohort Retention Heatmap (F4)

```python
import pandas as pd, numpy as np, seaborn as sns, matplotlib.pyplot as plt

orders["first_date"] = orders.groupby("customer_id")["order_date"].transform("min")
orders["cohort"]     = orders["first_date"].dt.to_period("M")
orders["period"]     = orders["order_date"].dt.to_period("M")
orders["age"]        = (orders["period"] - orders["cohort"]).apply(lambda x: x.n)

cohort    = orders.groupby(["cohort", "age"])["customer_id"].nunique().unstack(fill_value=0)
retention = cohort.div(cohort[0], axis=0)   # share of the cohort still ordering

fig, ax = plt.subplots(figsize=(10, 6))
sns.heatmap(retention, cmap="YlGnBu", vmin=0, vmax=0.5, ax=ax)
ax.set(title="Share of each monthly cohort that orders again",
       xlabel="Months since first order")
fig.savefig("reports/figures/f4_cohort_retention.png", dpi=200, bbox_inches="tight")
```

### 7.2 Repeat Rate by Category with Chi-Square Test (F6, H1)

```python
from scipy import stats

ct = pd.crosstab(cust["first_category"], cust["repeat_90d"])
chi2, p, dof, expected = stats.chi2_contingency(ct)
n = ct.values.sum()
cramers_v = np.sqrt(chi2 / (n * (min(ct.shape) - 1)))

rate = cust.groupby("first_category")["repeat_90d"].agg(["mean", "count"])
rate["se"] = np.sqrt(rate["mean"] * (1 - rate["mean"]) / rate["count"])
rate = rate.sort_values("mean")

plt.barh(rate.index, rate["mean"], xerr=1.96 * rate["se"])
plt.xlabel(f"90-day repeat rate  (chi2 p={p:.3g}, Cramer's V={cramers_v:.2f})")
plt.title("Repeat rate differs significantly across first-purchase categories")
plt.tight_layout()
plt.savefig("reports/figures/f6_repeat_by_category.png", dpi=200, bbox_inches="tight")
```

### 7.3 Group Difference with Effect Size (H2)

```python
from statsmodels.stats.multitest import multipletests

def mwu(df, col, target="repeat_90d"):
    a = df.loc[df[target] == 1, col].dropna()
    b = df.loc[df[target] == 0, col].dropna()
    u, p = stats.mannwhitneyu(a, b, alternative="two-sided")
    rbc = 2 * u / (len(a) * len(b)) - 1    # rank-biserial correlation
    return pd.Series({"feature": col, "u": u, "p": p, "effect_rbc": rbc})

cols = ["discount_share", "spend_30d", "n_categories_30d", "review_len"]
res  = pd.DataFrame([mwu(cust, c) for c in cols])
res["p_adj"] = multipletests(res["p"], method="fdr_bh")[1]
print(res.sort_values("p_adj"))
```

### 7.4 Survival Curves (F5)

```python
from lifelines import KaplanMeierFitter
from lifelines.statistics import multivariate_logrank_test

kmf = KaplanMeierFitter()
ax  = plt.subplot(111)

for cat, g in cust.groupby("first_category"):
    kmf.fit(g["days_to_second"], event_observed=g["returned"], label=cat)
    kmf.plot_survival_function(ax=ax, ci_show=False)

lr = multivariate_logrank_test(
    cust["days_to_second"], cust["first_category"], cust["returned"]
)
ax.set(title=f"Time to second purchase  (log-rank p={lr.p_value:.3g})",
       xlabel="Days since first order")
plt.tight_layout()
plt.savefig("reports/figures/f5_survival_curves.png", dpi=200, bbox_inches="tight")
```

### 7.5 Correlation, PCA and Clustering Preview

```python
from sklearn.preprocessing import StandardScaler
from sklearn.decomposition import PCA
from sklearn.cluster import KMeans
from sklearn.metrics import silhouette_score

num = ["recency", "frequency", "monetary", "discount_share", "basket_size"]

# Spearman correlation heatmap
corr = cust[num].corr(method="spearman")
sns.heatmap(corr, annot=True, fmt=".2f", cmap="vlag", center=0)
plt.title("Spearman correlation — customer behaviour features")
plt.tight_layout()
plt.savefig("reports/figures/f17_corr_heatmap.png", dpi=200, bbox_inches="tight")

# PCA
Xs  = StandardScaler().fit_transform(np.log1p(cust[num].clip(lower=0)))
pca = PCA(n_components=0.90).fit(Xs)
print("Cumulative variance:", pca.explained_variance_ratio_.cumsum())

# k-means silhouette sweep
for k in range(2, 9):
    labels = KMeans(n_clusters=k, n_init=10, random_state=42).fit_predict(Xs)
    print(k, round(silhouette_score(Xs, labels, sample_size=10_000, random_state=42), 3))
```

### 7.6 Text Check: Does VADER Agree with Ratings? (F10)

```python
from nltk.sentiment import SentimentIntensityAnalyzer   # nltk.download("vader_lexicon")

sia = SentimentIntensityAnalyzer()
reviews["vader"] = reviews["review_clean"].apply(
    lambda t: sia.polarity_scores(t)["compound"]
)

rho, p = stats.spearmanr(reviews["rating"], reviews["vader"])

ax = sns.boxplot(data=reviews, x="rating", y="vader", palette="coolwarm")
ax.set(title=f"VADER compound score by star rating  (ρ={rho:.2f}, p={p:.3g})",
       xlabel="Star rating", ylabel="VADER compound score")
plt.tight_layout()
plt.savefig("reports/figures/f10_sentiment_vs_rating.png", dpi=200, bbox_inches="tight")
```

---

## 8. Feature Engineering Plan

### 8.1 Prediction Setup (Leakage Prevention)

Two prediction points will be used, and every feature is built **only from information available at that point**:

| Model | Prediction Point | Features Use | Label |
|-------|-----------------|-------------|-------|
| **Model A** | Day 0 (first order) | First basket, customer profile, product information | Any order on days 1–90 |
| **Model B** | Day 30 | Days 0–30 behaviour (reviews, site engagement, second visits) | Any order on days 31–90 |

> ⚠️ All statistics learned from data (category means, target encodings, scalers) are computed on **training cohorts only**.

### 8.2 Feature Families

#### Behaviour (RFM and Early Life)

| Feature | Definition | Why It May Help | Hypothesis |
|---------|-----------|----------------|------------|
| `recency_days`, `frequency_30d`, `spend_30d`, `aov` | Days since last order, orders in first 30 days, spend, average order value | Standard strong baseline for retention | RQ2 |
| `n_categories_30d`, `routine_completeness` | Number of distinct skincare steps bought ÷ 3-step core routine | Customers building a routine are more invested | H1 |
| `basket_size`, `items_per_order` | Items in the first order | Size of first commitment | RQ2 |
| `days_between_orders_mean` | Average gap between orders (Model B) | Rhythm of buying | RQ2 |

#### Replenishment Timing

| Feature | Definition | Why It May Help | Hypothesis |
|---------|-----------|----------------|------------|
| `due_ratio` | Days since last order ÷ expected product usage days (`size_ml` ÷ assumed daily use) | Captures "time to restock", the natural trigger for skincare repeat | H1 |
| `ran_out_flag` | `due_ratio > 1` | Customers past their expected run-out date | H1 |

#### Price and Discount

| Feature | Definition | Why It May Help | Hypothesis |
|---------|-----------|----------------|------------|
| `price_per_ml`, `price_tier` | Price standardized by size, tier within category | Fair price comparison | H2 |
| `discount_share`, `first_order_discount` | Share of spend that was discounted | Deal-driven behaviour | H2 |
| `promo_period_flag`, `gift_set_flag` | Order placed in sale period; gift-like product | Separates promotion buyers and one-off gifts | H1, H2 |

#### Product

| Feature | Definition | Why It May Help | Hypothesis |
|---------|-----------|----------------|------------|
| `category_first`, `brand_first`, `product_family_id` | First product attributes | Category and brand loyalty differ | H1 |
| `product_avg_rating_prior`, `product_n_reviews_prior` | Rating and review count **before** the order date (time-aware) | Quality signal without future leakage | H4 |
| `is_new_launch` | Product < 90 days old at order | New launches attract trial buyers | H7 |

#### Ingredient

| Feature | Definition | Why It May Help | Hypothesis |
|---------|-----------|----------------|------------|
| `has_<active>` flags | Retinol, niacinamide, vitamin C, AHA/BHA, ceramides, SPF filters | Active-led interest | RQ4 |
| `active_weight_<active>` | 1 ÷ position in ingredient list (earlier = higher concentration) | Better potency proxy than presence | RQ4, H6 |
| `irritant_load` | Count (or weighted count) of potential irritants (fragrance, drying alcohols) | Link to irritation and drop-off | H5 |
| `n_actives`, `ingredient_count`, `active_diversity` | Number of key actives, list length, share of distinct active classes | Complexity of formula | RQ4 |
| `ingredient_embedding_1..k` | TF-IDF + SVD (or learned embedding) of ingredient lists | Content similarity for recommendation | H6 |

#### Review and Text

| Feature | Definition | Why It May Help | Hypothesis |
|---------|-----------|----------------|------------|
| `vader_compound`, `transformer_sentiment` | Sentiment of the customer's review (Model B only if posted by day 30) | Satisfaction signal | H3, H4 |
| `topic_irritation`, `topic_hydration`, `topic_texture`, `topic_price` | Topic shares from LDA or BERTopic | Reasons behind satisfaction | H5 |
| `rating_deviation` | Customer rating − product's prior mean | Separates harsh raters from bad products | H3 |
| `review_delay_days`, `review_len`, `has_photo` | Time to review, length, photo presence | Engagement | H3 |
| `authenticity_score` | Combined score from duplicates, length, verified flag, "free product" phrases | Down-weights unreliable reviews | H4 |

#### Customer Profile and Fit

| Feature | Definition | Why It May Help | Hypothesis |
|---------|-----------|----------------|------------|
| `skin_type`, `skin_tone`, `skin_missing_flags` | Profile and missingness indicators | Personalization and fairness | RQ5 |
| `skin_match_score` | Smoothed average rating by reviewers with the same skin type (shrunk toward product mean) | "Will this suit people like me?" | H5, H6 |
| `sensitive_x_fragrance`, `sensitive_x_irritant_load` | Interaction of sensitive skin with fragrance or irritants | Fit risk for sensitive customers | H5 |

#### Market and Trend

| Feature | Definition | Why It May Help | Hypothesis |
|---------|-----------|----------------|------------|
| `trend_at_purchase` | Google Trends value for the ingredient or category in the order week | Hype level at purchase | H7 |
| `trend_spike_z` | Standardized deviation from 12-week rolling mean | Identifies spike-driven purchases | H7 |
| `season_sin`, `season_cos`, `is_summer` | Cyclical month encoding | Seasonal demand (sunscreen, moisturizer) | H7 |
| `category_trend_slope` | Recent slope of category search interest | Rising or falling category | H7 |

#### Interactions

| Feature | Definition | Hypothesis |
|---------|-----------|------------|
| `discount_x_category` | Discount share × category indicator | H2 |
| `price_tier_x_tenure` | Price tier × customer age | H2 |
| `trend_x_new_launch` | Trend spike × is-new-launch flag | H7 |

#### Recommender-Specific

| Feature | Definition | Hypothesis |
|---------|-----------|------------|
| `user_item_affinity` | Collaborative filtering score | H6 |
| `item_popularity_prior` | Prior popularity (time-aware) | H6 |
| `ingredient_similarity_to_history` | Cosine similarity of product to past purchases | H6 |

### 8.3 Creative Features Targeting "Hidden" Behaviour

| Feature Idea | Rationale |
|-------------|-----------|
| **Routine completeness and step gaps** | Missing a core step (e.g., no sunscreen) may signal a cross-sell opportunity and an incomplete routine |
| **Replenishment due ratio** | Turns purchase history into a predicted run-out date — more actionable than raw recency |
| **Skin-match score** | Adds personalization even when the customer has not reviewed anything |
| **Hype vs habit index** | High trend spike + new launch + single-item basket flags a likely one-time trend chaser |
| **Review honesty weighting** | Sentiment features multiplied by the authenticity score |
| **Ingredient exposure history** | Which actives the customer has already tried, used to recommend a logical next step |
| **Purging-aware text feature** | Flag reviews mentioning "purging" or "first week" so temporary skin reactions are not counted as product rejection |

---

## 9. Transformations and Draft Feature Engineering Code

### 9.1 Transformation Plan

| Need | Method | Notes |
|------|--------|-------|
| Skewed money and count variables | `log1p` | Apply before scaling |
| Different scales | `StandardScaler` (linear, SVM, k-means); `RobustScaler` if outliers remain; none for trees | Fit on training data only |
| Low-cardinality categories | One-hot encoding | `handle_unknown="ignore"` |
| High-cardinality categories (brand, product family) | Frequency or smoothed target encoding inside cross-validation | Prevents leakage |
| Ordinal variables (price tier, rating) | Ordinal encoding | Keep natural order |
| Continuous to bands | Quantile binning (`KBinsDiscretizer`) | Useful for explanation and non-linear effects in linear models |
| Cyclical time | Sine and cosine of month | Keeps December next to January |
| Interactions | Manual products and `PolynomialFeatures(interaction_only=True)` on a small set | Avoid blind explosion of features |
| Text | TF-IDF + SVD, or sentence embeddings | Fit after the split |
| Ingredients | Binary flags, position weights, TF-IDF over ingredient tokens | Standardize names first |
| Class imbalance | Class weights; resampling inside training folds | Do not resample before splitting |

### 9.2 Early-Life Behavioural Features at Day 30 (Model B)

```python
T0 = 30                                     # prediction point: day 30

o = o.merge(first[["first_date"]], on="customer_id")
o["days_since_first"] = (o["order_date"] - o["first_date"]).dt.days

early = o[o["days_since_first"] <= T0]      # features: days 0–30 only
feat = early.groupby("customer_id").agg(
    n_orders_30d       = ("order_id",        "nunique"),
    items_30d          = ("quantity",         "sum"),
    spend_30d          = ("line_total",       "sum"),
    n_categories_30d   = ("category",         "nunique"),
    discount_share     = ("discount_flag",    "mean"),
)
feat["routine_completeness"] = (feat["n_categories_30d"] / 3).clip(upper=1)

# label: order on days 31–90, only for customers who had not reordered by day 30
later = o[(o["days_since_first"] > T0) & (o["days_since_first"] <= 90)]
feat["repeat_31_90"] = feat.index.isin(later["customer_id"]).astype(int)
```

### 9.3 Replenishment Due Ratio

```python
# Assumed daily use (ml/day) by category — documented assumption, tested in sensitivity analysis
DAILY_USE_ML = {"cleanser": 2.0, "moisturizer": 1.5, "serum": 0.6, "sunscreen": 2.0}

o["expected_days"] = o["size_ml"] / o["category"].map(DAILY_USE_ML)

last = o.sort_values("order_date").groupby("customer_id").tail(1)
last = last.assign(
    due_ratio = (pd.Timestamp(cutoff) - last["order_date"]).dt.days / last["expected_days"]
)
last["ran_out_flag"] = (last["due_ratio"] > 1).astype(int)
```

### 9.4 Ingredient Position Weights and Irritant Load

```python
ACTIVES   = ["retinol", "niacinamide", "ascorbic acid",
             "hyaluronic acid", "salicylic acid"]
IRRITANTS = ["fragrance", "parfum", "alcohol denat", "limonene", "linalool"]

def position_weight(ings, target):
    for i, x in enumerate(ings):
        if x == target:
            return 1.0 / (i + 1)        # earlier ingredient = higher concentration
    return 0.0

for a in ACTIVES:
    col = f"w_{a.replace(' ', '_')}"
    products[col] = products["ingredients"].apply(position_weight, target=a)

products["irritant_load"] = products["ingredients"].apply(
    lambda xs: sum(x in IRRITANTS for x in xs)
)
products["n_actives"] = products["ingredients"].apply(
    lambda xs: sum(x in ACTIVES for x in xs)
)
```

### 9.5 Skin-Match Score (Time-Aware, Smoothed)

```python
K = 10                                      # shrinkage strength

train    = reviews[reviews["review_date"] < split_date]
gm       = train["rating"].mean()
prod     = train.groupby("product_id")["rating"].agg(prod_mean="mean")

ps = (
    train.groupby(["product_id", "skin_type"])["rating"]
    .agg(n="count", mean="mean")
    .reset_index()
    .merge(prod, on="product_id")
)
ps["prod_mean"]   = ps["prod_mean"].fillna(gm)
ps["skin_match"]  = (ps["n"] * ps["mean"] + K * ps["prod_mean"]) / (ps["n"] + K)

# join to customers on (product_id, skin_type); unknown skin type falls back to prod_mean
```

### 9.6 Time-Aware Rating Deviation, Trend Spike and Seasonality

```python
# Rating deviation — product mean using only earlier reviews (no future information)
reviews = reviews.sort_values("review_date")
reviews["prior_mean"] = (
    reviews.groupby("product_id")["rating"]
    .transform(lambda s: s.expanding().mean().shift(1))
)
reviews["rating_deviation"] = reviews["rating"] - reviews["prior_mean"]

# Search-interest spike (weekly series 'trend' indexed by week)
roll    = trend.rolling(12)
spike_z = (trend - roll.mean().shift(1)) / roll.std().shift(1)

# Cyclical month encoding
orders["m"]          = orders["order_date"].dt.month
orders["season_sin"] = np.sin(2 * np.pi * orders["m"] / 12)
orders["season_cos"] = np.cos(2 * np.pi * orders["m"] / 12)
```

### 9.7 Interactions, Encoding and Scaling in One Leakage-Safe Pipeline

```python
from sklearn.compose import ColumnTransformer
from sklearn.pipeline import Pipeline
from sklearn.impute import SimpleImputer
from sklearn.preprocessing import StandardScaler, OneHotEncoder, FunctionTransformer

# Manual interaction terms
X["sensitive_x_fragrance"] = (X["skin_type"].eq("sensitive") & X["has_fragrance"].eq(1)).astype(int)
X["discount_x_gift"]       = X["discount_share"] * X["gift_set_flag"]

num_cols  = ["spend_30d", "aov", "price_per_ml", "discount_share", "irritant_load", "skin_match"]
cat_cols  = ["category_first", "skin_type", "channel"]
pass_cols = ["sensitive_x_fragrance", "discount_x_gift", "season_sin", "season_cos"]

num = Pipeline([
    ("log", FunctionTransformer(np.log1p, validate=False)),
    ("imp", SimpleImputer(strategy="median", add_indicator=True)),
    ("sc",  StandardScaler()),
])
cat = Pipeline([
    ("imp", SimpleImputer(strategy="constant", fill_value="unknown")),
    ("oh",  OneHotEncoder(handle_unknown="ignore")),
])

pre = ColumnTransformer([
    ("num",  num,            num_cols),
    ("cat",  cat,            cat_cols),
    ("pass", "passthrough",  pass_cols),
])

# pre.fit(X_train)   → learns medians, means and categories from training data ONLY
# pre.transform(X_valid) → reuses them unchanged (no leakage)
```

> ⚠️ `log1p` is applied only to non-negative columns. Verify that `skin_match` and similar scores are positive before including them in the log step.

---

## 10. Feature Selection and Validation

| Stage | Method | Purpose |
|-------|--------|---------|
| Screening | Remove near-constant features (variance threshold), very high missingness, and features built after the prediction point | Clean feature set |
| Redundancy | Spearman clusters, VIF; drop one of each highly correlated pair (> ~0.9) | Reduce collinearity |
| Relevance | Mutual information, univariate tests with FDR control, L1 (lasso) logistic regression | Short list of useful features |
| Model-based | Permutation importance and SHAP on validation data | Importance that reflects actual model use |
| Stability | Repeat importance over several time-based folds and bootstrap resamples | Keep consistently useful features |
| Leakage audit | Check each feature's timestamp against the prediction point; test with a "shuffled time" control and watch for suspiciously perfect AUC | Honest performance |
| Fairness audit | Compare model error and recommendation quality across skin type, tone and region | Avoid systematic disadvantage |

---

## 11. How Features Improve Insight and Model Accuracy

### 11.1 Ablation Design

Feature groups will be added step by step to the same model and the same time-based split, recording the change in AUC-ROC, PR-AUC and lift in the top decile each time.

| Experiment | Feature Set | Question Answered |
|-----------|------------|------------------|
| M0 | Majority-class or RFM-only logistic regression | Baseline |
| M1 | M0 + early-life behaviour and replenishment features | Do timing features add signal? |
| M2 | M1 + price and discount features | H2 |
| M3 | M2 + product and ingredient features | RQ4 |
| M4 | M3 + review and text features (with authenticity weights) | H4 |
| M5 | M4 + skin-match and interaction features | H5, personalization |
| M6 | M5 + trend and seasonality features | H7 |
| M7 | Best subset, tuned gradient boosting | Final model |

Improvements are tested with **bootstrap confidence intervals** on the metric difference, and gains are judged against the cost of complexity.

### 11.2 Expected Contributions

| Feature Group | Expected Effect on Model | Expected Effect on Insight |
|--------------|-------------------------|--------------------------|
| Replenishment timing | Large gain for replenishable products (moisturizer, cleanser) | Gives a precise "when to contact" signal |
| Discount and price | Moderate gain, interaction with category | Separates deal seekers from routine builders |
| Ingredient and skin-match | Moderate gain for recommendations and churn linked to fit | Explains why some products hold customers and others do not |
| Review and text | Gain mainly in Model B when reviews are available | Reveals reasons behind low repeat (irritation, texture, price) |
| Trend and season | Small to moderate gain; helps separate hype from habit | Better demand and campaign timing |
| Interactions | Helps linear models; trees find many on their own | Shows context-specific effects |

> These are **expectations to be tested**, not results.

---

## 12. Hypothetical Scenario: Using the Features to Improve a Decision

> ⚠️ **All numbers below are illustrative assumptions made up for explanation, not actual findings.**

A fictional online skincare retailer, **GlowLab**, gets 10,000 first-time customers a month. About 28% place a second order within 90 days. The CRM team can afford a replenishment campaign (reminder email + a personalized product suggestion) for 3,000 customers a month.

**Without the engineered features:** the team contacts customers at random, or those with the biggest first basket. Suppose the campaign lifts repeat probability by 3 percentage points on average → ~90 extra repeat customers per month.

**With the engineered features:** the Model B score at day 30 and the `due_ratio` identify three groups:

| Group | How Identified | Action |
|-------|---------------|--------|
| **Habit builders** (high predicted repeat, high routine completeness) | High model score | Do not discount; send a light reminder, saving margin |
| **On the fence, due to restock** | Mid score, `due_ratio` ≈ 1, good `skin_match` | Replenishment reminder + a recommended next-step product |
| **Trend chasers and poor-fit buyers** | High trend spike at purchase, new launch, single item, or high `irritant_load` with sensitive skin | Low-cost education content or a better-fitting alternative; avoid heavy discounts |

If the team concentrates the 3,000 contacts on the second group, and the campaign lifts repeat probability by 8 points there *(an assumption to be checked by an A/B test)*, that is roughly **240 extra repeat customers** instead of 90. At an assumed second-order value of 40 currency units, the difference is about **6,000 extra revenue units per month**, with less discount spend on customers who would have returned anyway.

**Decision impact:** targeting, timing, offer type and product recommendation each improve, and the skin-match score also lets the team withdraw or reposition products that sensitive-skin customers rate poorly.

> **Caution:** a risk model predicts *who will return*, not *who will respond to a campaign*, so the plan includes an A/B test and, later, **uplift modelling**.

---

## 13. Linking Expected EDA Findings to Business Insight

| Possible EDA Finding | Business Interpretation | Possible Action |
|---------------------|------------------------|----------------|
| Replenishable core products show higher 90-day repeat than gift sets (H1) | Acquisition through gifting brings low-loyalty buyers | Rebalance acquisition spend; bundle core products for first orders |
| Heavy first-order discounts coincide with lower repeat (H2) | Promotions attract bargain hunters | Cap first-order discounts; use value-add offers instead |
| Reviews mentioning irritation concentrate in certain skin types and products (H5) | Product-fit problem, not a service problem | Add skin-type guidance, adjust recommendations, review formulation |
| Customers who review early return more (H3) | Engagement predicts loyalty | Prompt early reviews; early-journey email series |
| Search spikes precede sales jumps but not repeat (H7) | Trend products are one-off purchases | Plan inventory for spikes; pair trend items with core products |
| Missing skin type is common and linked to lower repeat | Less engaged customers skip the profile | Add short skin quiz at checkout |
| Cohort retention dips after month 2 | Natural restock point is missed | Time reminders to the typical run-out date |
| Ratings differ by skin tone or type | Uneven product fit or review coverage | Improve inclusive content and test recommenders for fairness |

---

## 14. Risks and Ethical Considerations

| Risk | Mitigation |
|------|-----------|
| Data leakage through features built from future information | Prediction-point timestamps, time-based split, leakage audit |
| Spurious findings from many tests | FDR control; confirm key findings on held-out cohorts |
| Confusing correlation with cause | Use cautious wording; plan experiments for causal claims |
| Synthetic data driving conclusions | Evidence tier labels; separate reporting |
| Review bias (self-selection) | Compare with verified-purchase subsets and across two sources |
| Fairness issues from skin tone or skin type features | Subgroup performance checks; do not use sensitive attributes to exclude customers from offers |
| Over-engineering features | Ablation and stability tests; prefer simple features when gains are small |
| Privacy | Aggregate reporting; no personal identifiers in figures or features |

---

## 15. Time Allocation (~35 hours)

| Activity | Hours |
|----------|-------|
| Reviewing basic and advanced EDA techniques | 5 |
| Designing the EDA workflow, statistical and visualization plan | 7 |
| Writing and testing EDA code on sample data | 6 |
| Designing the feature engineering catalogue and transformations | 7 |
| Drafting feature code and leakage checks | 4 |
| Building the scenario and linking findings to business insight | 2 |
| Writing, editing and formatting this document | 4 |
| **Total** | **35** |

### Next Steps (Week 5)

- Execute the pipeline on the real files and report quality metrics.
- Run the full EDA on processed data and document findings.
- Build and evaluate the first predictive models using the ablation design above.

---

## 16. Conclusion

This plan sets out how the cleaned skincare data will be explored and how the findings will become features. The EDA moves from simple distributions to cohort, survival, text, ingredient, trend and fairness analysis, with each step linked to a Week 1 hypothesis.

The feature catalogue captures:
- **Behaviour** — replenishment timing, routine completeness
- **Product fit** — ingredient position weights, skin-match score
- **Sentiment** — authenticity-weighted reviews
- **Market context** — trend spikes, seasonality

All features are built to **avoid leakage**, validated through ablation, and linked to concrete business decisions. In Week 5 the pipeline will be executed on the real files, features will be tested through ablation, and the first predictive models will be trained and evaluated.

---

## 17. References and Tool Documentation

| Reference / Tool | Link or Citation |
|-----------------|-----------------|
| Tukey, J. W. (1977) | *Exploratory Data Analysis.* Addison-Wesley |
| Benjamini & Hochberg (1995) | FDR control. *JRSS-B*, 57(1), 289–300 |
| Kaplan & Meier (1958) | Survival estimation. *JASA*, 53, 457–481 |
| Agrawal et al. (1993) | Association rules. *ACM SIGMOD* |
| McInnes et al. (2018) | UMAP. *arXiv:1802.03426* |
| Zheng & Casari (2018) | *Feature Engineering for Machine Learning.* O'Reilly |
| Kuhn & Johnson (2019) | *Feature Engineering and Selection.* CRC Press |
| Lundberg & Lee (2017) | SHAP. *NeurIPS* |
| Fader, Hardie & Lee (2005) | CLV. *Marketing Science*, 24(2) |
| Hunter (2007) | Matplotlib. *Computing in Science & Engineering*, 9(3) |
| Waskom (2021) | seaborn. *JOSS*, 6(60) |
| Virtanen et al. (2020) | SciPy 1.0. *Nature Methods*, 17, 261–272 |
| matplotlib | https://matplotlib.org/stable/ |
| seaborn | https://seaborn.pydata.org/ |
| SciPy | https://docs.scipy.org/doc/scipy/ |
| statsmodels | https://www.statsmodels.org/ |
| lifelines | https://lifelines.readthedocs.io/ |
| scikit-learn | https://scikit-learn.org/stable/ |
| umap-learn | https://umap-learn.readthedocs.io/ |
| mlxtend | https://rasbt.github.io/mlxtend/ |
| Data sources | See Week 2 Source Register |
