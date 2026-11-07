# 🌿 Predicting Repeat Purchase & Personalizing Recommendations in Online Skincare

> **Internship Data Science Project** | 8-Week Programme  
> **Domain:** Beauty & Wellness | **Focus:** Customer Retention + Recommender Systems

---

## 📌 Project Overview

An online skincare retailer spends heavily to acquire first-time customers — but a large share never return. This project builds two connected data science solutions:

1. **Repeat Purchase Prediction** — Predict whether a first-time customer will make a second purchase within 90 days.
2. **Product Recommendation** — Recommend relevant next products using reviews, ingredients, and behaviour signals.

**Data Sources Used:** Customer reviews, product attributes (including ingredients), purchase behaviour, and external trend signals.

---

## 🗂️ Project Structure

```
skincare-repeat-purchase-prediction/
│
├── Week1_Problem_Definition/          # Week 1 deliverable
│   └── Week1_Problem_Definition.md
│
├── Week2_Data_Strategy/               # Week 2 deliverable
│   └── Week2_Data_Sourcing_Strategy.md
│
├── Week3_Data_Cleaning_Pipeline/      # Week 3 deliverable
│   └── Week3_Data_Cleaning_Pipeline.md
│
├── Week4_EDA_Feature_Engineering/     # Week 4 deliverable
│   └── Week4_EDA_Feature_Engineering.md
│
├── Week5_Predictive_Modelling_Strategy/# Week 5 deliverable
│   ├── Week5_Predictive_Modelling_Strategy.md
│   └── Week_5_Predictive_Modelling_Strategy_Deliverable.docx
│
├── Week6_Model_Evaluation_Business_Impact/# Week 6 deliverable
│   ├── Week6_Model_Evaluation_Business_Impact.md
│   └── Week_6_Model_Evaluation_Business_Impact_Report.docx
│
├── data/
│   ├── raw/                           # Original, immutable data (not pushed to GitHub)
│   ├── interim/                       # Intermediate cleaned data
│   └── processed/                     # Final, analysis-ready data
│
├── notebooks/                         # Jupyter notebooks (EDA, modelling, etc.)
├── src/                               # Python source scripts
├── reports/
│   └── figures/                       # Generated charts and visualizations
│
├── .gitignore
└── README.md
```

---

## 📅 Weekly Progress

| Week | Focus | Status |
|------|-------|--------|
| Week 1 | Industry Research, Problem Definition & Hypotheses | ✅ Complete |
| Week 2 | Data Identification, Sourcing Strategy & Initial Analysis Plan | ✅ Complete |
| Week 3 | Data Cleaning & Preprocessing Pipeline Development | ✅ Complete |
| Week 4 | Exploratory Data Analysis & Feature Engineering | ✅ Complete |
| Week 5 | Predictive Modelling Strategy & Algorithm Selection | ✅ Complete |
| Week 6 | Model Evaluation & Business Impact Reporting | ✅ Complete |
| Week 7 | Recommendation System Development & Deep Personalization | 🔜 Upcoming |
| Week 8 | Insights, Final Report & Executive Presentation | 🔜 Upcoming |

---

## 🔬 Research Questions

- **RQ1** — What proportion of first-time customers repeat within 90 days, and how does this vary by category and price?
- **RQ2** — Which early behavioural features best predict repeat purchase?
- **RQ3** — Does adding review-derived features improve prediction beyond transactions alone?
- **RQ4** — Are particular ingredients associated with higher satisfaction and repeat rates?
- **RQ5** — Do hybrid recommenders outperform popularity and collaborative filtering baselines?
- **RQ6** — Do external trend signals help explain demand spikes that don't lead to loyalty?

---

## 🧪 Hypotheses

| ID | Hypothesis | RQ |
|----|------------|----|
| H1 | Core replenishable products (cleansers, moisturizers) → higher repeat rate than gift sets | RQ1 |
| H2 | Heavy discount use on first order → lower repeat rate | RQ1, RQ2 |
| H3 | Early positive review → more likely to repeat | RQ2, RQ3 |
| H4 | Review sentiment + topics improve AUC over transactions alone | RQ3 |
| H5 | Reviews mentioning irritation/breakouts → strong negative predictor of repeat | RQ3, RQ4 |
| H6 | Hybrid recommender (collab + content + ingredients) beats baselines, especially cold-start | RQ5 |
| H7 | Trend-spiked products → higher first sales but lower repeat rates | RQ6 |

---

## 🛠️ Tools & Technologies

| Purpose | Tools |
|---------|-------|
| Programming | Python, Jupyter Notebooks |
| Data Handling | pandas, NumPy, DuckDB, Parquet |
| Visualization | Matplotlib, Seaborn, Plotly |
| Statistics | SciPy, statsmodels, lifelines |
| Machine Learning | scikit-learn, XGBoost, LightGBM, SHAP |
| NLP | NLTK, spaCy, VADER, HuggingFace Transformers, BERTopic |
| Recommender Systems | Surprise, implicit, LightFM |
| Customer Analytics | lifetimes (BG/NBD & Gamma-Gamma) |

---

## 📦 Data Sources

| Source | Type | Used For |
|--------|------|----------|
| [Sephora Products & Reviews (Kaggle)](https://www.kaggle.com/datasets/nadyinky/sephora-products-and-skincare-reviews) | Reviews + Ingredients | Core beauty data |
| [Amazon Reviews 2023 (UCSD)](https://amazon-reviews-2023.github.io/) | Reviews | Behaviour proxy |
| [UCI Online Retail II](https://archive.ics.uci.edu/dataset/502/online-retail-ii) | Transactions | Method validation |
| [Olist Brazilian E-Commerce](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce) | Orders + Reviews | Retention modelling |
| [Open Beauty Facts](https://world.openbeautyfacts.org/data) | Ingredients | Enrichment |
| [EU CosIng](https://ec.europa.eu/growth/tools-databases/cosing/) | Ingredient functions | INCI standardization |
| [Google Trends](https://trends.google.com/) | Search interest | Trend signals |

> ⚠️ **Note:** Raw data files are NOT stored in this repository. Download instructions are provided in each week's deliverable. Only code, notebooks, and documentation are tracked here.

---

## ⚖️ Ethics & Privacy

- Only publicly released or properly licensed data is used.
- Personal identifiers are removed or hashed.
- Skin tone, age, and health-related fields are treated as sensitive.
- Models are audited for fairness across skin type, skin tone, and age groups.
- Applicable frameworks: GDPR (EU), India's DPDPA 2023.

---

## 👩‍💻 Author

**Lamiya KP**  
Internship Project — Beauty & Wellness Data Science  
📅 2026
