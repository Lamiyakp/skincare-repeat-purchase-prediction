# Week 2 Deliverable: Data Identification, Sourcing Strategy & Initial Analysis Planning

> **Project:** Predicting Repeat Purchase and Personalizing Product Recommendations in Online Skincare  
> **Week:** 2 of 8  
> **Estimated Hours:** 34 hours

---

## 1. Purpose and Link to Week 1

In Week 1 the project defined the following business problem: predict which first-time customers of an online skincare retailer will make a repeat purchase, and recommend the products most likely to keep them engaged, using transaction behaviour, review text and ingredient information. Seven hypotheses (H1 to H7) and six research questions (RQ1 to RQ6) were set out.

This document turns that plan into a **concrete data strategy**. It:
1. States what data the problem needs
2. Catalogues and evaluates public sources with links
3. Lays out how the data will be collected
4. Assesses quality and relevance
5. Describes the cleaning and preprocessing needed
6. Shows how each hypothesis can be tested with the data
7. Plans the initial exploratory analysis

> ⚠️ **Important:** URLs and dataset terms were compiled from public information and can change. Each link should be opened and each licence re-read before use, and the access date recorded in the data log.

---

## 2. Data Requirements Derived from the Problem

No single public source provides all the data needed — hence a **multi-source strategy** is required.

| Data Need | Why It Is Needed | Week 1 Link |
|-----------|-----------------|-------------|
| Customer purchase histories (order dates, items, prices, discounts) | Define first purchase, repeat purchase, RFM, CLV | RQ1, RQ2, H1, H2 |
| Customer reviews and ratings with timestamps and reviewer identifiers | Sentiment, topics, rating behaviour, repeat-reviewer proxy | RQ3, H3, H4, H5 |
| Product metadata (category, brand, price, size, labels) | Product-level features, category repeat rates, content-based recommendation | RQ1, RQ4, H1, H6 |
| Ingredient lists and ingredient functions | Ingredient features, irritant flags, similarity between products | RQ4, H5, H6 |
| Customer profile attributes (skin type, concerns, tone) | Personalization, fairness checks, cold-start handling | RQ5, H6 |
| External demand and sentiment signals (search interest, social mentions) | Trend spikes versus steady demand | RQ6, H7 |

---

## 3. Sourcing Strategy

### 3.1 Guiding Principles

- **Alignment:** A source is included only if it helps answer at least one research question.
- **Layered design:** Combine a rich beauty-specific source with a transaction-rich source and external signals.
- **Public and lawful:** Use open licences, official downloads and official APIs; avoid scraping where TOS forbid it.
- **Reproducibility:** Every pull is logged, versioned and repeatable by script.
- **Transparency about proxies:** Where a source only approximates a concept, this is stated and tested.

### 3.2 Four-Tier Source Architecture

| Tier | Role | Sources |
|------|------|---------|
| **Tier 1: Core beauty data** | Products, ingredients, reviews, skin attributes | Sephora Products & Reviews; Amazon Reviews 2023 (All Beauty) |
| **Tier 2: Transaction analogues** | Real purchase-order data for retention and CLV modelling | UCI Online Retail II; Olist Brazilian E-Commerce; Synthetic orders |
| **Tier 3: Enrichment** | Ingredient functions, safety, product labels, regulatory data | Open Beauty Facts; EU CosIng; U.S. FDA cosmetics data |
| **Tier 4: External signals** | Trends, social conversation, attention | Google Trends; Reddit API; YouTube Data API; Wikimedia Pageviews |

### 3.3 Why Synthetic Data Is Part of the Plan

No public skincare retailer releases customer-level order histories. To still test retention methods on beauty products, the project uses two approaches:
- **(a)** Real transaction analogues (UCI Online Retail II and Olist) to build and validate the retention pipeline on genuine purchase behaviour
- **(b)** A clearly labelled synthetic order table linking simulated customers to real Sephora/Amazon products

> Every result based on synthetic data will be labelled as such. Where reviewer timestamps allow, repeat reviewers in the Amazon data provide a further real-behaviour proxy.

---

## 4. Catalogue of Potential Data Sources

### 4.1 Tier 1: Core Beauty Datasets

#### 1. Sephora Products and Skincare Reviews (Kaggle)
- **Link:** https://www.kaggle.com/datasets/nadyinky/sephora-products-and-skincare-reviews
- **Content:** Product information (brand, category, price, ingredients, ratings, labels such as vegan or clean) and several hundred thousand customer reviews with ratings, recommendation flag, helpfulness votes, review text and reviewer attributes (skin type, skin tone, eye and hair colour).
- **Access:** Free download with a Kaggle account (CSV files). Static snapshot.
- **Pros:** Directly on topic; ingredients and skin-type fields support H4–H6 and fairness checks; text-rich; easy to load.
- **Cons:** No purchase histories or true customer identifiers beyond an author ID; review sample biased; one retailer only.
- **Reliability:** Medium-high for structure; medium for representativeness.

#### 2. Amazon Reviews 2023 (UCSD McAuley Lab)
- **Link:** https://amazon-reviews-2023.github.io/
- **Content:** Reviews and item metadata by category (including All Beauty and Health and Personal Care), with ratings, text, timestamps, helpful votes and verified-purchase flags.
- **Access:** Free for research use; JSON lines format; read terms of use carefully.
- **Pros:** Very large; timestamps and reviewer IDs allow repeat-reviewer analysis; verified-purchase flag helps filter fake reviews; widely used in research.
- **Cons:** No ingredient lists in most items; beauty category mixes cosmetics, tools and accessories; very large files require sampling.
- **Reliability:** High for provenance; medium for behaviour proxies.

#### 3. Kaggle Cosmetics Collections (Supplementary)
- **Example Link:** https://www.kaggle.com/datasets/kingabzpro/cosmetics-datasets
- **Content:** Smaller cosmetics product and ingredient tables, useful for cross-checking ingredient parsing.
- **Pros:** Small and quick to prototype with.
- **Cons:** Unclear provenance and licences on some uploads; verify before using.
- **Reliability:** Low-medium; use for validation only.

---

### 4.2 Tier 2: Transaction Analogues

#### 4. UCI Online Retail II
- **Link:** https://archive.ics.uci.edu/dataset/502/online-retail-ii
- **Content:** Two years of invoice-level transactions from a UK-based online retailer (invoice date, product code, description, quantity, unit price, customer ID, country).
- **Access:** Free download (CC BY 4.0 licence; confirm on the page). Static (2009–2011).
- **Pros:** Real customer-level order histories; well suited to RFM, cohort retention, BG/NBD and CLV; widely cited.
- **Cons:** Giftware retailer, not beauty; many wholesale customers; returns and cancellations need careful handling.
- **Reliability:** High for structure; relevance to beauty is indirect.

#### 5. Brazilian E-Commerce Public Dataset by Olist (Kaggle)
- **Link:** https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce
- **Content:** ~100,000 orders (2016–2018) across a marketplace with customers, orders, items, payments, reviews (scores and comments), products and sellers; includes health and beauty product categories.
- **Access:** Free download with a Kaggle account. Static.
- **Pros:** Orders, reviews and products linked in one relational dataset; includes a beauty/health category; Portuguese review text enables multilingual NLP practice.
- **Cons:** Most customers purchase only once (heavily imbalanced); Portuguese comments; anonymized.
- **Reliability:** High for linkage; medium for beauty relevance.

#### 6. Synthetic Skincare Order Table (Self-Generated)
- **Source:** Built in Python using real products and review-derived satisfaction from Tier 1.
- **Pros:** Full control; includes all desired variables (discount use, channel, replenishment timing).
- **Cons:** Results reflect assumptions; cannot be used as evidence of real customer behaviour.
- **Note:** Always labelled and documented with a `data_origin` column.

---

### 4.3 Tier 3: Enrichment and Regulatory Data

#### 7. Open Beauty Facts
- **Link:** https://world.openbeautyfacts.org/data
- **Content:** Crowdsourced database of cosmetic products with barcodes, brands, ingredient lists, labels and categories.
- **Access:** Free; CSV, MongoDB and JSON exports and an API; Open Database Licence.
- **Pros:** Large and open; barcodes allow product matching; ingredient lists in standard format.
- **Cons:** Crowdsourced so coverage, spelling and completeness vary.
- **Reliability:** Medium.

#### 8. EU CosIng Database (European Commission)
- **Link:** https://ec.europa.eu/growth/tools-databases/cosing/
- **Content:** Reference database of cosmetic ingredients with INCI names, functions (humectant, emollient, UV filter), CAS numbers and regulatory annex information.
- **Access:** Free web search; downloadable ingredient inventory files; updated by the Commission.
- **Pros:** Authoritative source to standardize ingredient names; maps each to a function and restrictions.
- **Cons:** No product-level data; matching messy ingredient strings to INCI names takes effort.
- **Reliability:** High (official).

#### 9. U.S. FDA Cosmetics Data
- **Links:** https://www.fda.gov/cosmetics | https://open.fda.gov/
- **Content:** Voluntary and mandatory adverse event reports on cosmetics, product listing and facility registration information.
- **Access:** Free; downloadable files and openFDA API.
- **Pros:** Government source; adverse-event categories (rash, hair loss) give external reference for irritation topics in reviews (H5).
- **Cons:** Under-reporting; reports rarely identify exact product or ingredient cleanly.
- **Reliability:** High for provenance; low-medium for completeness.

#### 10. Government and Market-Statistics Portals
- **Examples:** https://data.gov | https://data.gov.in | https://ec.europa.eu/eurostat | https://data.worldbank.org
- **Role:** Background context only (macro indicators, e-commerce penetration, population by age group).
- **Reliability:** High.

---

### 4.4 Tier 4: External Signals (Trends and Social Media)

#### 11. Google Trends
- **Link:** https://trends.google.com/
- **Unofficial Python wrapper:** https://github.com/GeneralMills/pytrends
- **Content:** Relative search interest (0–100) for terms such as "retinol", "niacinamide", "sunscreen" by country and region; weekly or daily series.
- **Pros:** Clean signal of consumer interest and seasonality; supports H7; regional views for India.
- **Cons:** Values are relative, not absolute; unofficial API may change.
- **Reliability:** Medium-high.

#### 12. Reddit API
- **Links:** https://www.reddit.com/dev/api/ | https://www.redditinc.com/policies/data-api-terms
- **Communities:** r/SkincareAddiction, r/IndianSkincareAddicts
- **Content:** Posts, comments, scores and timestamps; rich discussion of ingredients, routines and reactions.
- **Pros:** Authentic, detailed language about skin concerns and ingredients.
- **Cons:** Access terms have tightened; self-selected users; sarcasm and slang; usernames are personal data.
- **Reliability:** Medium.

#### 13. YouTube Data API
- **Link:** https://developers.google.com/youtube/v3
- **Content:** Video titles, descriptions, view/like counts and comments for beauty creators and product reviews.
- **Access:** Free with API key and daily quota.
- **Reliability:** Medium.

#### 14. Wikimedia Pageviews API
- **Link:** https://wikimedia.org/api/rest_v1/
- **Content:** Daily page views for articles such as "Retinol" or "Niacinamide".
- **Access:** Free, no key required.
- **Pros:** Simple, stable, absolute counts; cross-check for Google Trends.
- **Reliability:** High.

### 4.5 Sources Considered but Excluded

| Source | Reason Excluded |
|--------|----------------|
| Instagram and TikTok scraping | Restricted by terms of service and technical limits |
| Commercial syndicated data (Euromonitor, Mintel, NielsenIQ) | Paid; used only through publicly quoted summaries |
| Retailer websites (Nykaa, Ulta) directly scraped | Terms generally prohibit automated scraping |

---

## 5. Evaluation of Data Sources

### 5.1 Scoring Criteria (1 = Poor, 5 = Excellent)
Relevance | Accessibility | Reliability | Volume | Timeliness | Linkability

### 5.2 Scoring Matrix

| Source | Relevance | Accessibility | Reliability | Volume | Timeliness | Linkability | **Total /30** | Role |
|--------|-----------|---------------|-------------|--------|------------|-------------|---------------|------|
| Sephora Products & Reviews | 5 | 5 | 4 | 4 | 2 | 4 | **24** | Core |
| Amazon Reviews 2023 | 4 | 4 | 4 | 5 | 3 | 3 | **23** | Core |
| UCI Online Retail II | 2 | 5 | 4 | 4 | 1 | 2 | **18** | Method validation |
| Olist Brazilian E-Commerce | 3 | 5 | 4 | 4 | 1 | 5 | **22** | Method validation |
| Open Beauty Facts | 4 | 4 | 3 | 4 | 4 | 3 | **22** | Enrichment |
| EU CosIng | 4 | 3 | 5 | 3 | 3 | 3 | **21** | Ingredient reference |
| FDA cosmetics / CAERS | 3 | 4 | 4 | 3 | 3 | 2 | **19** | External validation |
| Google Trends | 4 | 4 | 4 | 3 | 5 | 3 | **23** | Trend signal |
| Reddit API | 3 | 3 | 3 | 4 | 5 | 2 | **20** | Language / concerns |
| YouTube Data API | 3 | 3 | 3 | 3 | 5 | 2 | **19** | Optional |
| Wikimedia Pageviews | 2 | 5 | 5 | 3 | 5 | 3 | **23** | Optional cross-check |
| Government portals | 2 | 5 | 5 | 2 | 3 | 1 | **18** | Context only |

### 5.3 Decision

| Priority | Sources |
|----------|---------|
| **Must have** (Weeks 3–5) | Sephora, Amazon Reviews (beauty subset), Olist, UCI Online Retail II, Open Beauty Facts, CosIng, Google Trends |
| **Should have** | FDA cosmetics data, Reddit, Synthetic skincare orders |
| **Could have** | YouTube, Wikimedia Pageviews, government context data |

---

## 6. Data Collection Plan

### 6.1 Step-by-Step Plan

1. **Set up environment:** Python environment, project folders (`data/raw`, `data/interim`, `data/processed`), Git repository and a data log file.
2. **Download static datasets:** Sephora, Amazon beauty subset, Olist, UCI — record URL, version, date, file size, checksum and licence; store raw files untouched and read-only.
3. **Download reference data:** Open Beauty Facts export; CosIng ingredient reference; relevant FDA files.
4. **Collect trend signals:** Choose a keyword list (~30–50 terms: key actives, concerns, category terms) and pull Google Trends series for 5 years by country (global, US, India); export weekly Wikipedia pageviews for the same actives.
5. **Collect social text (optional):** A limited, rate-compliant Reddit pull for a set of skincare communities over a defined period, storing only text and non-identifying metadata.
6. **Build the synthetic order generator** after the first EDA, using parameters estimated from real data.
7. **Validate each file on arrival:** Row counts, column types, date ranges, duplicate keys.
8. **Document everything** in a data dictionary and a source register.

### 6.2 Sampling Strategy for Large Files

- **Amazon Reviews:** Restrict to relevant categories and a defined time window; filter to skincare-related products using product title and category keywords; use chunked reads and Parquet storage.
- Keep a stratified sample (by rating and year) for fast prototyping; the full filtered set for final runs.

### 6.3 Target Schema for Integration

| Table | Key Fields |
|-------|-----------|
| `products` | product_id, source, name, brand, category, price, size, labels, ingredient_list_raw |
| `product_ingredients` | product_id, ingredient_inci, function, flag_active, flag_irritant |
| `customers` | customer_id, source, skin_type, skin_tone, concerns (where available) |
| `orders` | order_id, customer_id, order_date, channel, discount_flag *(real or synthetic, flagged)* |
| `order_items` | order_id, product_id, quantity, unit_price |
| `reviews` | review_id, customer_id, product_id, rating, review_date, text, helpful_votes, verified_flag |
| `trends` | term, date, region, value, source |

> A `data_origin` column (real / proxy / synthetic) will be kept on every table.

### 6.4 Tools

Python (pandas, polars or DuckDB for large files, requests, pytrends, PRAW for Reddit), Kaggle CLI, Parquet for storage, Git for code, and a simple CSV or Markdown data log.

### 6.5 Data Log Template

| Field | Example |
|-------|---------|
| Source name and URL | Sephora Products and Skincare Reviews (Kaggle URL) |
| Date accessed | *(fill in)* |
| Version / file names | *(fill in)* |
| Licence and terms | *(fill in after reading)* |
| Rows / columns | *(fill in)* |
| Known issues | *(fill in)* |

### 6.6 Legal and Ethical Safeguards

- Read and record the licence and terms for each source; do not redistribute raw data where not permitted.
- Do not collect or keep personal identifiers; hash or drop user IDs and usernames where possible.
- Treat skin tone, age and any health-related fields as sensitive; use them for fairness audits only.
- Respect API rate limits and `robots.txt`; do not circumvent access controls.
- Applicable privacy frameworks: **GDPR (EU)** and **India's Digital Personal Data Protection Act, 2023**.

---

## 7. Data Quality and Relevance Assessment

### 7.1 Quality Dimensions
Completeness | Accuracy | Consistency | Validity | Uniqueness | Timeliness | Representativeness | Linkability

### 7.2 Expected Issues by Source

| Source | Expected Issues | Planned Check |
|--------|----------------|---------------|
| Sephora | Missing skin type/tone fields; inconsistent ingredient formatting; duplicates; review sample skewed toward high ratings | Missing-value profile; duplicate detection; rating distribution plot |
| Amazon Reviews | Non-skincare items in beauty category; huge size; missing ingredients | Keyword and category filters; verified-purchase split; near-duplicate text check |
| UCI Online Retail II | Cancellations (invoices starting with "C"), negative quantities, missing customer IDs, wholesale buyers | Filter rules documented; compare stats with/without wholesale outliers |
| Olist | Mostly single-purchase customers; Portuguese text; customer_unique_id vs customer_id confusion | Verify repeat rates using unique ID; language detection |
| Open Beauty Facts | Sparse or misspelled fields, inconsistent languages, missing ingredient lists | Completeness rate by country; sample manual audit |
| CosIng | Name mismatches versus product ingredient strings | Fuzzy-match success rate on a sample |
| FDA data | Under-reporting, vague product descriptors | Use as qualitative cross-check only |
| Google Trends | Relative scaling; sampling variation | Pull twice on different days and compare; normalize to stable anchor term |
| Reddit | Sarcasm, slang, bot posts, deleted content | Remove bots and deleted posts; manual sample review |

### 7.3 Relevance and Suitability Summary

| Requirement | Best Source | Gap | Mitigation |
|-------------|-------------|-----|------------|
| Beauty purchase histories | None public | No real skincare order data | Transaction analogues + labelled synthetic orders; repeat-reviewer proxy |
| Reviews with text | Sephora, Amazon | Self-selection bias | Report bias; use verified-purchase subsets; compare across two sources |
| Ingredients | Sephora, Open Beauty Facts, CosIng | Messy text, incomplete coverage | Standardize to INCI, report match rates |
| Skin type and tone | Sephora | Only reviewers who disclose | Treat as optional features; analyze missingness |
| Trend signals | Google Trends, Wikipedia, Reddit | Not linked to sales | Use as context variables and timing signals only |

### 7.4 Representativeness and Bias

Sephora and Amazon represent online shoppers in mainly Western markets who chose to write reviews. Findings should not be presented as true for all consumers, particularly in India or other markets where product types, skin concerns and climate differ. Where possible, regional Google Trends views will be added to note differences.

---

## 8. Data Cleaning and Preprocessing Plan

### 8.1 General Steps

1. **Standardize formats:** Consistent date-time parsing and time zones; numeric conversion of price and size fields.
2. **Handle duplicates:** Remove exact duplicates; flag near-duplicate review texts; keep one record per review ID.
3. **Missing values:** Document missingness patterns; keep missing as its own category for skin type and tone; impute numeric fields only after splitting data to avoid leakage; never impute target variables.
4. **Outliers:** Examine order values, quantities and review lengths; winsorize or log-transform heavily skewed variables for modelling, but keep originals.
5. **Entity matching:** Link products across sources using barcodes (Open Beauty Facts), then brand + normalized product name with fuzzy matching (e.g., RapidFuzz), with manual check on random sample.
6. **Category harmonization:** Map each source's categories to a common skincare taxonomy (cleanser, toner, serum, moisturizer, sunscreen, treatment, mask, eye care, other).

### 8.2 Text Preprocessing (Reviews and Social Posts)

- Language detection; keep English for main analysis; treat Portuguese Olist text separately or translate a sample.
- Lowercase, remove URLs and excess whitespace, normalize emojis, expand common abbreviations and handle negations carefully ("not irritating").
- Keep raw text alongside processed text.
- Remove or hash usernames and personal details.

### 8.3 Ingredient Preprocessing

- Split raw ingredient string into items, lowercase, strip punctuation and parenthetical notes, remove marketing words.
- Standardize to INCI names using CosIng and fuzzy matching; record the match rate and unmatched list.
- Create features: binary flags for key actives and irritants, ingredient count, position-weighted flags, TF-IDF vectors.

### 8.4 Transaction Preprocessing

- Remove or separate cancellations and returns; drop non-product line items.
- Define first purchase date per customer, observation window and 90-day repeat label (with 30 and 60 days as sensitivity checks).
- Exclude customers whose follow-up window is incomplete (right-censoring) from the binary label, or handle them with survival methods.
- Create cohort month, RFM values and basket-level features using **only data available before the prediction point** to prevent leakage.

### 8.5 Time-Series Preprocessing (Trends)

- Align weekly series to calendar weeks; fill gaps; smooth with rolling average for visuals but keep raw series for tests.
- Compute volatility measures (rolling SD, spike indicators) per keyword for H7.

### 8.6 Quality Gates

A dataset moves from `interim` to `processed` only after:
- ✅ Schema checks pass
- ✅ Duplicate rate below agreed threshold
- ✅ Date ranges verified
- ✅ Key columns have documented missingness
- ✅ Short data-quality note added to the log

---

## 9. How the Data Will Test the Week 1 Hypotheses

| Hypothesis | Data Used | Variables | Method | Caveat |
|-----------|-----------|-----------|--------|--------|
| **H1:** Replenishable core products → higher repeat rate | Synthetic orders; Olist beauty; UCI; Amazon repeat reviewers | First-purchase category, repeat flag | Repeat rate by category, chi-square, logistic regression | Real beauty purchase data missing; report evidence tier |
| **H2:** Heavy discount use → lower repeat rate | UCI, Olist payments/vouchers, synthetic orders | Discount share, repeat flag | Logistic regression; Mann-Whitney tests | Discount fields incomplete in public data |
| **H3:** Early, positive reviews → repeat | Olist (reviews linked to orders); Amazon (reviewer timing) | Review delay, rating, repeat flag | Group comparison, logistic regression, survival curves | Reviewers are self-selected |
| **H4:** Review features improve prediction | Olist and synthetic orders linked to review features | Sentiment, topic shares, rating deviation | Compare models with/without review features; DeLong or bootstrap tests | Gains on Olist may differ from skincare |
| **H5:** Irritation and mismatch topics → lower repeat or ratings | Sephora and Amazon reviews; FDA categories | Topic share of irritation words, rating, recommend flag, skin type | Topic model + regression; compare topics across skin types | Link to repeat is indirect |
| **H6:** Hybrid recommender beats baselines | Sephora and Amazon user-item interactions; ingredient and attribute vectors | User-item matrix, product content vectors | Offline evaluation (Precision@K, Recall@K, NDCG@K) by user activity level | Reviews stand in for purchases |
| **H7:** Trend-driven products → more first sales, less repeat | Google Trends and Wikipedia series with review volume as sales proxy | Spike index, review volume growth, repeat proxy | Correlation, comparison of spike vs. steady groups, time-series plots | Review volume ≠ sales |

> **Evidence grading:** Each finding will be marked as based on real beauty data, real non-beauty analogue data or synthetic data.

---

## 10. Initial Exploratory Data Analysis Plan

### 10.1 Aims

Understand each dataset, spot quality problems early, identify promising predictors, and refine hypotheses before modelling.

### 10.2 Key Variables to Profile First

| Category | Variables |
|----------|-----------|
| Outcome-related | Repeat flag, days to second purchase, review rating, "would recommend" flag |
| Behavioural | Order count, recency, frequency, monetary value, basket size, discount share, return rate |
| Product | Category, brand, price tier, size, labels, review count, average rating, number of ingredients |
| Ingredient | Presence of key actives and irritants |
| Text | Review length, sentiment score, topic proportions |
| Customer profile | Skin type, skin tone, concerns |
| External | Search interest, volatility and seasonality indicators |

### 10.3 Descriptive Statistics

| Variable Type | Statistics |
|---------------|-----------|
| Numeric | Count, mean, median, SD, min, max, quartiles, skewness, kurtosis, missing share |
| Categorical | Frequency tables, proportions, number of unique values, top categories |
| Text | Review length distribution, vocabulary size, top terms and bigrams, language shares |
| Time series | Range, trend, seasonality, number of gaps |
| Customer-level | Orders per customer distribution, share of one-time buyers, median time between orders |

### 10.4 Visualization Plan

| Question | Visualization | Tool |
|----------|---------------|------|
| Rating distribution — is there a positivity bias? | Bar chart of ratings; rating by year | Seaborn, Matplotlib |
| Which categories and brands dominate? | Horizontal bar charts, treemap | Plotly |
| How skewed are prices and order values? | Histograms with log scale, box plots by category | Seaborn |
| What share of customers return? | Cohort retention heatmap (cohort month × months since first purchase) | Seaborn |
| How long until the second purchase? | Kaplan-Meier survival curves by first-purchase category | lifelines |
| How do recency and frequency interact? | RFM scatter and segment profile charts | Plotly |
| What do customers talk about? | Word clouds and bar charts of top terms for 1–2 star vs. 4–5 star reviews; topic prevalence | WordCloud, pyLDAvis |
| Which ingredients are popular and linked? | Ingredient frequency bars, co-occurrence heatmap or network graph | Seaborn, NetworkX |
| Does rating differ by skin type or tone? | Grouped box plots or violin plots | Seaborn |
| How do trends and seasonality behave? | Line charts of Google Trends for key actives, decomposition plots | Matplotlib, statsmodels |
| How are features related? | Correlation heatmap (Spearman), pair plots for key variables | Seaborn |
| How much data is missing? | Missingness matrix and bar chart | missingno |
| Where are customers located? | Choropleth map (if available) | Plotly |

### 10.5 Planned Statistical Analyses at EDA Stage

- **Group comparisons:** Chi-square (repeat by category), t-tests or Mann-Whitney U (numeric differences), ANOVA or Kruskal-Wallis (>2 groups), with effect sizes.
- **Correlation:** Spearman and point-biserial; VIF to flag multicollinearity.
- **Distribution checks:** Shapiro-Wilk and Q-Q plots to choose parametric or non-parametric tests.
- **Time series:** Seasonal decomposition, autocorrelation plots, rolling statistics.
- **Text:** TF-IDF top terms by rating group; VADER sentiment vs. ratings agreement check; initial LDA with 5–15 topics.
- **Clustering preview:** k-means on RFM features with silhouette score.
- **Multiple testing:** Benjamini-Hochberg correction when many tests are run.
- **Reporting:** Confidence intervals and effect sizes, not only p-values.

### 10.6 Preliminary Questions the EDA Should Answer

1. What is the baseline repeat rate in each transaction dataset, and how imbalanced is it?
2. Do ratings and sentiment differ by product category, price tier and skin type?
3. Which ingredients are most common, and which co-occur most often?
4. How well does VADER sentiment agree with star ratings?
5. Which topics appear most in low-rated reviews?
6. Is there visible seasonality or spike behaviour in search interest for key actives?
7. How complete are ingredient and skin-type fields, and does missingness differ by brand or year?

### 10.7 Deliverables from the EDA Stage (Week 3)

- A documented notebook
- A short data-quality report
- A figure set
- A table of descriptive statistics
- An updated hypothesis list showing which hypotheses look promising and which data gaps remain

---

## 11. Risks and Mitigation

| Risk | Impact | Mitigation |
|------|--------|------------|
| Licence or terms prohibit a planned use | Source cannot be used | Read terms first; keep a fallback source for each need |
| Links or datasets change or disappear | Plan breaks | Download early; store raw copies locally; keep checksums and a log |
| API rate limits or access changes (Reddit, Google Trends) | Missing external signals | Treat social data as optional; use manual exports; Wikipedia as backup |
| Large files exceed memory | Slow work | DuckDB or Parquet, chunking, stratified samples |
| Proxy and synthetic data mistaken for real evidence | Misleading conclusions | data_origin flag, evidence grading, clear wording in reports |
| Poor product matching across sources | Weak joins | Barcode-first matching, fuzzy matching with manual audit, report match rates |
| Fake or incentivised reviews | Distorted sentiment | Verified-purchase filter, duplicate detection, sensitivity analysis |
| Privacy concerns | Ethical or legal issue | No identifiers kept; hashed IDs; aggregate reporting |

---

## 12. Timeline and Time Allocation for Week 2 (~34 hours)

| Activity | Hours |
|----------|-------|
| Searching and reviewing candidate sources | 8 |
| Reading documentation, licences and terms; testing downloads and APIs | 6 |
| Evaluating sources (scoring matrix, pros and cons) | 4 |
| Inspecting sample data and drafting first quality assessment | 5 |
| Designing collection plan, schema, cleaning and preprocessing steps | 4 |
| Mapping hypotheses to data and planning EDA | 3 |
| Writing, editing and formatting this document | 4 |
| **Total** | **34** |

### Next Steps (Week 3)
- Download and profile the must-have datasets
- Run the first quality checks
- Complete the EDA described in Section 10
- Begin the synthetic order generator design

---

## 13. Source Link Register

| # | Source | Link |
|---|--------|------|
| 1 | Sephora Products and Skincare Reviews | https://www.kaggle.com/datasets/nadyinky/sephora-products-and-skincare-reviews |
| 2 | Amazon Reviews 2023 | https://amazon-reviews-2023.github.io/ |
| 3 | Kaggle cosmetics datasets (supplementary) | https://www.kaggle.com/datasets/kingabzpro/cosmetics-datasets |
| 4 | UCI Online Retail II | https://archive.ics.uci.edu/dataset/502/online-retail-ii |
| 5 | Olist Brazilian E-Commerce | https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce |
| 6 | Open Beauty Facts | https://world.openbeautyfacts.org/data |
| 7 | EU CosIng | https://ec.europa.eu/growth/tools-databases/cosing/ |
| 8 | U.S. FDA cosmetics | https://www.fda.gov/cosmetics |
| 9 | openFDA | https://open.fda.gov/ |
| 10 | data.gov (U.S.) | https://data.gov |
| 11 | data.gov.in (India) | https://data.gov.in |
| 12 | Eurostat | https://ec.europa.eu/eurostat |
| 13 | World Bank Open Data | https://data.worldbank.org |
| 14 | Google Trends | https://trends.google.com/ |
| 15 | pytrends (unofficial library) | https://github.com/GeneralMills/pytrends |
| 16 | Reddit API | https://www.reddit.com/dev/api/ |
| 17 | YouTube Data API | https://developers.google.com/youtube/v3 |
| 18 | Wikimedia REST API (pageviews) | https://wikimedia.org/api/rest_v1/ |

> ⚠️ Verify that each link still works and re-read each licence before downloading. Add the date accessed for every source to your final reference list.

---

## 14. Conclusion

No single public dataset covers the full Week 1 problem, so this plan combines **four tiers of data**: beauty-specific reviews and products, real transaction analogues, authoritative ingredient and regulatory references, and external trend signals.

The strategy explains where each source helps, where it falls short, and how proxies and synthetic data will be kept clearly separate from real evidence. With the collection plan, quality checks, preprocessing steps and exploratory analysis plan in place, the project can move into data profiling and analysis in **Week 3** with a clear link back to each hypothesis.
