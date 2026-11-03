# Week 1 Deliverable: Data Science Orientation and Problem Definition in Beauty & Wellness

> **Project:** Predicting Repeat Purchase and Personalizing Product Recommendations in Online Skincare  
> **Week:** 1 of 8  
> **Estimated Hours:** 34 hours

---

## 1. Executive Summary

Beauty and wellness is one of the largest and most digitally active consumer categories in the world. It is also one of the most crowded: thousands of brands compete for attention on social media, marketplaces and brand websites, and customers can switch products at almost no cost. In this setting, the economics of the business depend less on winning a first purchase and more on earning a second, third and tenth one.

This document defines a specific business problem for a data science project: **how can an online skincare retailer predict which first-time customers will make a repeat purchase, and which products should be recommended to them to make that happen?**

The project combines two connected tasks:
- **(a)** a repeat-purchase (retention) prediction model
- **(b)** a review- and ingredient-aware recommendation approach

using customer reviews, product attributes and purchase behaviour as the main sources of evidence.

---

## 2. Industry Context: Trends in Beauty & Wellness

### 2.1 Market Scale and Growth

Global beauty and personal care is commonly estimated by consultancies such as McKinsey and market research firms such as Euromonitor and Statista in the **hundreds of billions of US dollars annually**, with skincare as one of the largest and fastest-growing segments. Wellness, extending into nutrition, supplements, fitness, sleep and mental wellbeing, is estimated at well over **a trillion dollars globally**. Both industries have grown faster than general consumer goods over the past decade.

### 2.2 Key Trends Shaping the Sector

| Trend | Description |
|-------|-------------|
| **Digital & Social-First Discovery** | Customers discover products through TikTok, Instagram, YouTube and Reddit; viral trends create sudden demand spikes |
| **Ingredient Literacy** | Consumers search for specific actives: niacinamide, retinol, hyaluronic acid, vitamin C, ceramides, SPF |
| **Personalization** | Skin-type quizzes, AI skin analysis, virtual try-on and tailored routines are becoming standard expectations |
| **Clean & Sustainable Beauty** | Ingredient safety, packaging waste, shade range and inclusivity shape purchase decisions |
| **Wellness Convergence** | Skincare, supplements, sleep and stress products increasingly marketed together |
| **DTC & Omnichannel Retail** | Brands own customer relationships through websites, subscriptions and loyalty programmes |
| **Regional Growth** | Asia-Pacific (K-beauty, J-beauty, India's fast-growing e-commerce) drives global innovation |

### 2.3 Why These Trends Create a Data Problem

These trends produce large volumes of behavioural, textual and product data (clicks, baskets, ratings, reviews, social posts, ingredient lists). At the same time they raise the cost of customer acquisition and make loyalty fragile. Companies that can convert data into better retention and better recommendations hold a structural advantage.

---

## 3. Problem Statement

### 3.1 Business Problem

An online skincare retailer spends heavily to acquire customers (paid social, influencers, discounts), but a large share of first-time buyers never return. Many leave because:
- The product did not suit their skin
- They did not know what to buy next
- They were not engaged after the first order

The retailer lacks a reliable way to:
1. Identify early which first-time customers are likely to repeat
2. Understand why some products drive loyalty while others drive disappointment
3. Suggest the right next product at the right time

### 3.2 Formal Problem Statement

> Using transaction histories, product attributes (including ingredients) and customer review text from an online skincare retailer, build and evaluate models that **(a)** predict whether a first-time customer will make a second purchase within a defined window (90 days), and **(b)** recommend relevant next products, in order to improve customer retention and lifetime value.

### 3.3 Scope

**In scope:**
- Skincare products (cleansers, moisturizers, serums, sunscreens, treatments)
- Customer purchase and review data
- Product ingredient data
- External trend signals (search and social)

**Out of scope (future work):**
- Make-up shade matching
- Image-based skin analysis
- Supplements and wellness services
- Causal measurement of marketing campaigns

### 3.4 Objectives

1. Define and measure repeat purchase and customer lifetime value (CLV) for skincare customers.
2. Identify the behavioural, product and sentiment factors most associated with repeat purchase.
3. Build and compare predictive models for repeat purchase.
4. Build and compare recommendation approaches, including ones that use review text and ingredients.
5. Translate findings into practical recommendations for marketing and CRM teams.

### 3.5 Success Criteria

| Type | Criterion |
|------|-----------|
| **Technical** | Repeat-purchase model clearly outperforms baseline on AUC-ROC and PR-AUC |
| **Technical** | Recommender outperforms popularity baseline on Precision@K, Recall@K, NDCG@K |
| **Business** | Insights can be turned into actions (replenishment reminders, product associations) |

---

## 4. Relevance to the Beauty & Wellness Sector

- **Retention is more profitable than acquisition** — small improvements in retention have disproportionate effect on profit, especially with high paid-social acquisition costs.
- **Skincare is naturally repeat-driven** — products are consumed over weeks/months, so purchase timing is predictable and well suited to modelling.
- **Fit and trust are hard to judge online** — reviews and ingredient lists carry high information value; a model using them can reduce choice anxiety.
- **Rich, messy data** — structured data (orders) + unstructured text (reviews) + domain structure (ingredients) = ideal data science practice setting.
- **Marketing relevance** — connects segmentation, CRM, personalization, assortment planning and brand perception to analytical rigor.

---

## 5. Literature and Evidence Review

### 5.1 Customer Retention and Lifetime Value
- **Reichheld (2003)** — "The One Number You Need to Grow", HBR: links loyalty to growth and profitability.
- **Schmittlein, Morrison & Colombo (1987)** — Pareto/NBD model: probabilistic purchase frequency modelling.
- **Fader, Hardie & Lee (2005)** — BG/NBD model: "Counting Your Customers the Easy Way".
- **Gupta et al. (2006)** — "Modeling Customer Lifetime Value", Journal of Service Research.

### 5.2 Churn and Repeat-Purchase Prediction
- **Chen & Guestrin (2016)** — XGBoost: core gradient boosting reference.
- **Breiman (2001)** — Random forests.
- **Chawla et al. (2002)** — SMOTE: handling imbalanced datasets.
- RFM (Recency, Frequency, Monetary) variables remain a strong baseline set.

### 5.3 Recommender Systems
- **Koren, Bell & Volinsky (2009)** — "Matrix Factorization Techniques for Recommender Systems", IEEE Computer.
- **Hu, Koren & Volinsky (2008)** — "Collaborative Filtering for Implicit Feedback Datasets".
- Content-based and hybrid methods for cold-start users and products.

### 5.4 Text Analytics, Reviews and Sentiment
- **Hutto & Gilbert (2014)** — VADER: rule-based sentiment for social media.
- **Devlin et al. (2019)** — BERT: contextual sentiment and topic understanding.
- **Blei, Ng & Jordan (2003)** — LDA: topic modelling for review themes.

### 5.5 Industry Sources
- McKinsey & Company: beauty industry and consumer wellness reports
- Global Wellness Institute: wellness economy research
- Euromonitor, Statista, Mintel: skincare category reports
- Business of Fashion, Cosmetics Business, WWD Beauty
- Public company annual reports: L'Oréal, Estée Lauder, Nykaa

### 5.6 Gap This Project Addresses

Much of the retention literature uses transactional data only, and much of the review-mining literature stops at sentiment. This project asks whether **combining the two — and adding ingredient-level product information — improves both prediction and recommendation** in skincare, where fit and ingredients matter strongly.

---

## 6. Research Questions and Hypotheses

### 6.1 Research Questions

| ID | Research Question |
|----|-------------------|
| RQ1 | What proportion of first-time skincare customers make a repeat purchase within 90 days, and how does this vary by first-purchase product category, price tier and acquisition channel? |
| RQ2 | Which early behavioural features (order value, basket size, discount use, time between visits, category mix) best predict repeat purchase? |
| RQ3 | Does adding review-derived features (sentiment, topics, rating deviation) improve prediction beyond transactional features alone? |
| RQ4 | Are particular ingredients or product attributes associated with higher satisfaction and repeat rates? |
| RQ5 | Do hybrid recommendation models using reviews and ingredients outperform baselines, especially for new or low-activity users? |
| RQ6 | Do external trend signals help explain demand spikes and short-lived purchases that do not lead to loyalty? |

### 6.2 Hypotheses

| ID | Hypothesis | Related RQ | Expected Evidence |
|----|------------|------------|-------------------|
| H1 | First-time buyers of replenishable core products (cleanser, moisturizer, sunscreen) have higher repeat rate than those who bought gift sets or trending items | RQ1 | Higher repeat rate by category; significant chi-square or logistic regression |
| H2 | Heavy discount use on first order → lower repeat rate (deal-driven customers) | RQ1, RQ2 | Negative coefficient or lower feature importance for discount share |
| H3 | Customers who leave a review quickly, especially positive, are more likely to repeat | RQ2, RQ3 | Higher repeat rate; feature importance of review timing and rating |
| H4 | Adding review sentiment and topic features improves AUC-ROC and PR-AUC over transaction-only model | RQ3 | Statistically meaningful improvement in cross-validated metrics |
| H5 | Reviews mentioning irritation, breakouts or mismatch with skin type are strong negative predictors of repeat | RQ3, RQ4 | Negative association of those topics with repeat |
| H6 | Hybrid recommender (collaborative + content + ingredients) achieves higher NDCG@K than popularity and collaborative-only baselines, with largest gain for cold-start users | RQ5 | Offline ranking metrics by user-activity segment |
| H7 | Products with sudden search/social spikes show higher first-time sales but lower repeat rates than steady-demand products | RQ6 | Comparison of repeat rate against trend volatility |

> All hypotheses are testable and falsifiable. Null results will be reported as findings.

---

## 7. Variables and Data

### 7.1 Target (Dependent) Variables

| Variable | Definition |
|----------|------------|
| **Repeat purchase (binary)** | 1 if customer places second order within 90 days, else 0 |
| **Time to second purchase** | Days until second order (survival analysis with censoring) |
| **CLV proxy** | Total spend in 6–12 months after first purchase |
| **Recommendation relevance** | Whether a recommended product was subsequently purchased or positively reviewed |

### 7.2 Candidate Predictor Variables

| Group | Examples | Type |
|-------|----------|------|
| Customer behaviour | Recency, frequency, monetary value, basket size, items per order, discount share, return rate, device and channel | Numeric / Categorical |
| Customer profile | Skin type, skin concerns, age band, location, quiz answers, loyalty tier | Categorical |
| Product attributes | Category, brand, price tier, size, rating, review count, new-launch flag, clean/vegan/fragrance-free labels | Mixed |
| Ingredients | Key actives (retinol, niacinamide, vitamin C, AHAs/BHAs, SPF), potential irritants (fragrance, alcohol), ingredient embeddings | Binary / Vector |
| Review text | Sentiment score, topic proportions, mentions of skin type or side effects, review length, helpfulness votes | Numeric / Text |
| Rating behaviour | Customer rating minus product average, rating variance | Numeric |
| External signals | Google Trends interest, social mention volume, seasonality, campaign periods | Time series |

### 7.3 Possible Data Sources

| Source | Description |
|--------|-------------|
| [Sephora Products & Skincare Reviews (Kaggle)](https://www.kaggle.com/datasets/nadyinky/sephora-products-and-skincare-reviews) | Product metadata, ingredients, and hundreds of thousands of reviews |
| [Amazon Reviews Dataset (UCSD)](https://amazon-reviews-2023.github.io/) | All Beauty category reviews with timestamps and reviewer IDs |
| [Open Beauty Facts](https://world.openbeautyfacts.org/data) | Open database of cosmetic products and ingredient lists |
| Google Trends / Reddit | Trend and sentiment signals |
| EU CosIng | INCI ingredient name standardization |
| Synthetic transactional data | Simulated order tables, clearly labelled as synthetic |

### 7.4 Data Limitations

- Reviews are a biased sample (satisfied/very dissatisfied write more)
- Public datasets may lack true purchase histories
- Ingredient lists are inconsistently formatted
- Skin response is highly individual

### 7.5 Ethics, Privacy and Compliance

- Use only publicly released or properly licensed data; respect platform terms of service
- Remove or avoid personal identifiers; do not re-identify reviewers
- Treat skin tone, age and health-related fields as **sensitive**; audit models for performance differences across groups
- Avoid making health or medical claims from results
- Comply with **GDPR** and **India's Digital Personal Data Protection Act, 2023**

---

## 8. Analysis Plan and Methodology

### 8.1 Overview

The project follows the **CRISP-DM cycle**: Business Understanding → Data Understanding → Data Preparation → Modelling → Evaluation → Deployment.

### 8.2 Step-by-Step Outline

| Step | Action |
|------|--------|
| 1 | Business understanding and metrics definition |
| 2 | Data collection and audit |
| 3 | Data cleaning and integration |
| 4 | Exploratory data analysis (EDA) |
| 5 | Customer segmentation (RFM + clustering) |
| 6 | Feature engineering (behavioural, text, ingredient) |
| 7 | Predictive modelling for repeat purchase |
| 8 | Model interpretability (SHAP, PDP) |
| 9 | Recommendation system development |
| 10 | Fairness and robustness checks |
| 11 | Insights and business recommendations |
| 12 | Reporting and communication |

### 8.3 Tools and Technologies

| Purpose | Tools |
|---------|-------|
| Programming | Python, Jupyter / Google Colab |
| Data handling | pandas, NumPy, SQL (SQLite/PostgreSQL) |
| Visualization | Matplotlib, Seaborn, Plotly; Tableau/Power BI |
| Statistics | SciPy, statsmodels, lifelines |
| Machine learning | scikit-learn, XGBoost, LightGBM, imbalanced-learn, SHAP |
| NLP | NLTK, spaCy, VADER, HuggingFace Transformers, BERTopic |
| Recommendation | Surprise, implicit, LightFM |
| Customer analytics | lifetimes (BG/NBD and Gamma-Gamma) |
| Version control | Git + GitHub |

---

## 9. Risks and Mitigation

| Risk | Impact | Mitigation |
|------|--------|------------|
| No real purchase histories available | Must use proxies or synthetic data | Transparent labelling; validate approach on both; avoid overclaiming |
| Data leakage | Over-optimistic model performance | Features from observation window only; time-based split |
| Class imbalance | Poor detection of repeaters | Class weights, PR-AUC, threshold tuning |
| Biased/fake reviews | Misleading sentiment signals | Filter duplicates; check helpfulness and verified purchase fields |
| Ingredient data messiness | Weak content features | Standardize via INCI and reference lists; manual spot checks |
| Scope creep | Missed deadlines | Minimum viable project defined; rest treated as extensions |

---

## 10. Project Timeline

| Week | Focus |
|------|-------|
| Week 1 | Industry research, problem definition, hypotheses, data plan ✅ |
| Week 2 | Data collection, audit, cleaning, data dictionary |
| Week 3 | Exploratory analysis, cohort and trend analysis |
| Week 4 | Segmentation and feature engineering |
| Week 5 | Repeat-purchase modelling and evaluation |
| Week 6 | Interpretability, survival analysis, fairness checks |
| Week 7 | Recommendation system development and evaluation |
| Week 8 | Insights, recommendations, final report and presentation |

---

## 11. Week 1 Time Allocation (~34 hours)

| Activity | Hours |
|----------|-------|
| Industry and market trend research | 8 |
| Academic literature review | 8 |
| Problem selection, scoping and comparison of alternatives | 4 |
| Identifying and inspecting candidate datasets | 5 |
| Defining variables, hypotheses and analysis plan | 5 |
| Writing, editing and formatting this document | 4 |
| **Total** | **34** |

---

## 12. Alternative Problems Considered

| Alternative | Why Not Chosen |
|-------------|---------------|
| Brand sentiment analysis across social media | Data access restricted; harder to link to concrete business decisions |
| Trend forecasting for ingredients/products | Forecasting alone gives less actionable guidance than retention + recommendation |
| Image-based skin analysis and shade matching | Requires image datasets; more privacy and ethical complexity than appropriate for a first project |

---

## 13. Conclusion

Retaining customers and helping them find the right products is a central challenge for beauty and wellness brands operating in a crowded, digital and trend-driven market. By combining transaction behaviour, review language and ingredient information, this project aims to predict which new customers will return and to recommend products that fit them.

The plan above sets out the business context, research questions, testable hypotheses, data sources, methods and risks needed to carry the work forward with both **marketing insight** and **analytical rigor**.
