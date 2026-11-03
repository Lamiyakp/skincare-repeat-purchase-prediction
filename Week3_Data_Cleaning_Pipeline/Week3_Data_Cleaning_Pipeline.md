# Week 3 Deliverable: Data Cleaning and Preprocessing Pipeline Development

> **Project:** Predicting Repeat Purchase and Personalizing Product Recommendations in Online Skincare  
> **Week:** 3 of 8  
> **Estimated Hours:** 34 hours

---

## 1. Purpose and Link to Weeks 1 and 2

Week 1 defined the problem: predict which first-time skincare customers will make a repeat purchase and recommend products that keep them engaged. Week 2 identified the data: Sephora and Amazon reviews and product data, transaction analogues (UCI Online Retail II, Olist), ingredient and regulatory references (Open Beauty Facts, CosIng, FDA), and trend signals (Google Trends, Reddit, Wikimedia).

This document designs the **cleaning and preprocessing pipeline** that turns those raw, messy files into analysis-ready tables. It describes the issues expected, the method for each, the Python tools and why they fit, pseudo-code and code sketches, how the process will be documented for reproducibility, and how clean data improves later modelling.

> ⚠️ **Note:** Code is illustrative. Column names must be checked against the real files in Week 4 before running.

### Design Principles

| # | Principle |
|---|-----------|
| 1 | **Raw data is never edited.** Every step reads from one layer and writes to the next. |
| 2 | **Every change is logged** (what, why, how many rows affected). |
| 3 | **No leakage.** Anything learned from data (imputation values, scalers, encoders) is fitted on training data only. |
| 4 | **Keep information.** Missingness, outliers and proxy status are flagged, not silently erased. |
| 5 | **Reproducible by script.** One command re-creates every processed table. |

---

## 2. Pipeline Overview

### 2.1 Stages and Rationale

| # | Stage | What Happens | Why It Matters |
|---|-------|-------------|----------------|
| 1 | Ingest and archive | Download, checksum, store raw files read-only | Traceability; allows reruns |
| 2 | Profile | Row counts, types, missingness, duplicates, ranges | Know the problems before fixing them |
| 3 | Standardize schema | Column names, data types, units, dates, currency | Consistent joins and calculations |
| 4 | Deduplicate | Exact and near-duplicate removal | Prevents double counting and inflated sentiment |
| 5 | Validate and fix inconsistencies | Rule checks, category and brand harmonization | Fixes impossible and conflicting values |
| 6 | Handle missing values | Flag, then impute or keep as category | Avoids bias and keeps usable rows |
| 7 | Detect and treat outliers | Statistical and domain rules; flag, cap or exclude | Protects models and statistics from distortion |
| 8 | Clean text and ingredients | Normalize review text, parse and standardize INCI names | Turns text into usable features |
| 9 | Integrate and resolve entities | Match products, link tables, build customer histories | Creates the modelling dataset |
| 10 | Engineer labels and features | Repeat flag, RFM, sentiment, ingredient flags (past data only) | Defines target without leakage |
| 11 | Transform and encode | Log, scale, one-hot, text vectors inside model pipelines | Meets model assumptions |
| 12 | Split and validate | Time-based split; quality checks; write processed data | Honest evaluation and reliable output |

### 2.2 Data Flow Architecture

```
Raw Files (read-only)
        │
        ▼
  ┌─────────────┐
  │   Ingest &  │  → data/raw/<source>/<date>/
  │   Archive   │  → Checksum + data_log.csv
  └──────┬──────┘
         │
         ▼
  ┌─────────────┐
  │   Profile   │  → ydata-profiling HTML report
  │   & Audit   │  → missingno matrix
  └──────┬──────┘
         │
         ▼
  ┌──────────────────────────────────────┐
  │           INTERIM LAYER              │
  │  Schema → Dedup → Validate → Missing │
  │  → Outliers → Text → Ingredients    │
  │  → Entities → Labels → Features     │
  └──────┬───────────────────────────────┘
         │
    Quality Gate ✅
         │
         ▼
  ┌─────────────┐
  │  PROCESSED  │  → data/processed/
  │    LAYER    │  → Split: train / val / test
  └─────────────┘
         │
  Rejected rows → data/rejected/*.parquet (with reason codes)
```

> Raw files stay **untouched**. Stages 3–11 write to the interim layer. Only data that passes the quality gate is written to the processed layer.

---

## 3. Data Quality Issues Expected in Public Beauty and Wellness Data

| Issue | Where It Is Likely | Example | Handling |
|-------|--------------------|---------|----------|
| Missing profile fields | Sephora reviews | Skin type, skin tone, eye colour often blank | §4.4 |
| Missing ingredients | Amazon, Open Beauty Facts | Product has no ingredient list | §4.4, §4.8 |
| Duplicate reviews | Amazon, Sephora | Same text posted twice, or across product variants | §4.3 |
| Fake or incentivised reviews | Amazon, Sephora | Short, repetitive, "received free product" disclosures | §5 |
| Inconsistent categories and brand names | All product tables | "L'Oreal", "L'Oréal Paris", "LOREAL" | §4.6 |
| Mixed units and formats | Product data | "1.7 oz", "50 mL", "50ml / 1.7 fl oz"; prices as text | §4.2 |
| Inconsistent ingredient lists | Product data | Different separators, "Aqua" vs "Water", marketing words | §4.8 |
| Product variants | Sephora, Amazon | Shade and size variants with separate IDs but same formula | §4.9 |
| Extreme values | Prices, quantities, review lengths | Gift sets at high prices; wholesale orders | §4.5 |
| Cancellations and returns | UCI Online Retail II | Invoice numbers starting with "C", negative quantities | §4.10 |
| Mixed languages and code-mixing | Olist, Reddit | Portuguese text; Hinglish phrases; emojis | §4.7 |
| Rating and text conflicts | Reviews | 5-star rating with complaint text | §4.7 |
| Right-censored behaviour | Transaction data | Customers whose follow-up window has not finished | §4.10 |
| Sampling bias | Reviews | Mostly very positive or very negative | §4.13 |
| Gaps and scaling in trend data | Google Trends | Relative 0–100 values; missing weeks | §4.11 |
| Personal data | Reddit, reviews | Usernames, personal details in text | §4.7 |

---

## 4. Step-by-Step Cleaning and Preprocessing Methods

### 4.1 Ingest and Archive

- Download files with scripts, not by hand, and save them to `data/raw/<source>/<date>/`.
- Record URL, date, file size, SHA-256 checksum and licence in `data_log.csv`.
- Mark raw files **read-only**. Convert very large files to **Parquet** in interim for speed.
- Profile on load: shape, data types, memory use, and first rows.

---

### 4.2 Standardize the Schema

- **Column names:** lowercase, underscores, no spaces or symbols.
- **Types:** convert dates with `pd.to_datetime(..., errors="coerce")` and count the values that fail; convert numbers with `pd.to_numeric(..., errors="coerce")`; use `category` dtype for repeated strings.
- **Units:** parse size strings into a numeric `size_ml` (convert fl oz to mL at 29.5735) and compute `price_per_ml` for fair comparison across pack sizes.
- **Currency:** keep the original currency column; convert only if a documented exchange rate and date are used.
- **Time zones:** store timestamps in UTC.
- **Text encoding:** read as UTF-8 and fix mis-encoded characters.

---

### 4.3 Remove Duplicates

- **Exact duplicates:** drop rows identical on key fields (author, product, timestamp, text).
- **Near duplicates:** normalize text (lowercase, strip punctuation), then compare hashes; for borderline cases use fuzzy similarity (RapidFuzz token ratio > 95) within the same product or author.
- **Cross-variant duplicates:** the same review text on many variants of one product is kept once and linked to the product family.
- Log the duplicate count per source and keep removed rows in a `rejected/` file with a reason code.

---

### 4.4 Handle Missing Values

**First understand why data are missing:**

| Mechanism | Meaning | Beauty Example |
|-----------|---------|----------------|
| MCAR (Missing Completely At Random) | No pattern | Random export error |
| MAR (Missing At Random) | Depends on observed variables | Skin tone missing more for some brands or years |
| MNAR (Missing Not At Random) | Depends on the missing value itself | Customers with sensitive skin not disclosing skin type; ingredients hidden for undisclosed fragrance |

**Treatment by variable type:**

| Variable | Treatment | Reason |
|----------|-----------|--------|
| Skin type, skin tone, hair and eye colour | Fill with `unknown` + add `*_missing` flag | Missingness may carry information; avoids inventing attributes |
| Review text | Keep rows for rating-based work; drop for text models | Rating is still valid |
| Numeric product fields (price, size) | Median imputation by category + missing flag | Robust to skew |
| Ingredient list | Do not impute; flag `has_ingredients = 0`; try matching from Open Beauty Facts first | Imputing ingredients would be fabrication |
| Customer ID in transactions | Drop rows (cannot build customer histories) and report share lost | Required for the target |
| Target variables (repeat flag, rating) | **Never impute** | Would bias evaluation |
| Time series gaps | Interpolate short gaps (≤ 2 weeks); mark longer gaps | Keeps seasonality without inventing long stretches |
| Many missing values in a column (> ~60%) | Review usefulness; drop or keep only as a flag | Little signal and high noise |

> ⚠️ Imputation is **fitted inside the model pipeline on training data only**. For advanced cases, `IterativeImputer` or `KNNImputer` will be compared with simple methods, and results checked for stability.

---

### 4.5 Detect and Treat Outliers

1. **Domain rules first:** rating must be 1–5; price and quantity must be positive; review dates cannot precede product launch; `size_ml` must be within a plausible range.
2. **Statistical detection:** on log-transformed skewed variables use the IQR rule (below Q1 − 1.5·IQR or above Q3 + 1.5·IQR) and the robust z-score based on median and MAD (flag if |z| > 3.5). Visual checks with box plots and histograms.
3. **Multivariate checks (optional):** Isolation Forest or Local Outlier Factor on order-level features.
4. **Decide by cause:**
   - **Data error:** correct or remove.
   - **Valid but extreme** (gift sets, bulk orders): keep, flag `is_outlier`, and cap (winsorize at 1st and 99th percentiles) only for models sensitive to extremes.
   - **Different population** (wholesale buyers in UCI data): separate segment, analyse with and without.
5. **Report impact:** compare descriptive statistics and key model results before and after treatment.

---

### 4.6 Fix Inconsistencies

- **Brand and category harmonization:** strip accents and punctuation, lowercase, apply a manual mapping file (`brand_map.csv`), and fuzzy-match the leftovers with human review. Map categories to one taxonomy: `cleanser`, `toner`, `serum`, `moisturizer`, `sunscreen`, `treatment`, `mask`, `eye_care`, `other`.
- **Logical checks:** a review date before the product's launch date; rating of "recommended" with 1 star; negative prices; repeat purchase dates before the first purchase.
- Rule violations go to a `quality_issues` table with a rule name, row ID and action taken.

---

### 4.7 Clean Text (Reviews and Social Posts)

1. Keep a copy of the **raw text** in a separate column.
2. Remove URLs, HTML tags, repeated whitespace; normalize Unicode.
3. Detect language (`langdetect` or `fasttext`); route English to the main pipeline, other languages to separate handling.
4. Handle emojis by converting to text labels (e.g., with the `emoji` package) as they carry sentiment.
5. **Preserve negations** ("not irritating") — do not remove them as stop words; build a custom stop-word list that keeps words like "not", "no" and "never".
6. Lemmatize (spaCy) for topic models; use raw lightly cleaned text for transformer sentiment models.
7. Flag **rating-text conflicts** (e.g., 5 stars with strongly negative sentiment) for review.
8. **Privacy:** hash user IDs with a salt; remove usernames, emails and phone numbers by regex; do not store identifying content.

---

### 4.8 Clean and Standardize Ingredients

1. Convert the raw string to a list (check the file's format; some are stored as Python-list strings and need `ast.literal_eval`).
2. Lowercase, remove parenthetical notes and percentages, strip punctuation, trim whitespace.
3. Split on commas, handling commas inside brackets before removing them.
4. Map to standard **INCI names** using CosIng, a synonym dictionary (e.g., `aqua → water`, `vitamin c → ascorbic acid`) and fuzzy matching with a threshold; record the match rate and keep an `unmatched_ingredients.csv` for manual review.
5. Remove marketing text and "may contain" sections, or place them in a separate list.
6. Create features: key active flags, irritant flags, ingredient count, position-weighted scores (earlier ingredients present in higher amounts), and TF-IDF vectors.

---

### 4.9 Integrate and Resolve Entities

- **Product matching:** barcode first (Open Beauty Facts), then normalized brand + product name with fuzzy matching and a similarity threshold; manual audit of a random sample of ~200 matches to estimate precision.
- **Product families:** group size and shade variants that share the same formula and name into a `product_family_id`.
- **Joins:** use left joins from the fact table (reviews or orders) to dimension tables (products, ingredients); assert one-to-one or one-to-many relationships and check row counts after each join so no silent duplication happens.
- Keep a `data_origin` column (real / proxy / synthetic) on every table.

---

### 4.10 Transaction Cleaning and Label Construction

- Remove cancellations (invoice numbers starting with `"C"`) and rows with non-positive quantity or price; separate non-product lines (postage, fees).
- Drop rows without customer ID and report the share removed.
- Separate wholesale-type customers by a documented rule (e.g., very large average quantity) and analyse both ways.
- **Repeat label:** for each customer, find the first order date; repeat = 1 if another order occurs from day 1 to day 90 after it. Only customers whose 90-day window ends before the dataset end date get a label; the rest are **right-censored** and handled in survival analysis.
- Build predictors using **data up to the prediction point only** (first order and first 30 days).

---

### 4.11 Trend Time Series

- Align to weekly calendar frequency; reindex to find gaps; interpolate short gaps only.
- Normalize Google Trends by including a stable anchor term in each query; pull twice on separate days and average to reduce sampling noise.
- Compute volatility and spike indicators on raw (not smoothed) series.

---

### 4.12 Transform and Encode

| Variable Type | Method | Notes |
|--------------|--------|-------|
| Skewed variables (price, order value, review length) | `log1p` transform | Reduces right skew |
| Numeric features for linear/distance models | `StandardScaler` | Zero mean, unit variance |
| Numeric features with remaining outliers | `RobustScaler` | Uses median and IQR |
| Tree-based models | No scaling needed | Invariant to monotone transforms |
| Low-cardinality categoricals | One-hot encoding | Avoids ordinal assumption |
| High-cardinality categoricals (brand) | Target or frequency encoding inside cross-validation | Prevents leakage |
| Text | TF-IDF or transformer embeddings | Generated after the split |
| Class imbalance | Class weights or resampling on training data only | Applied in modelling, not cleaning |

---

### 4.13 Split and Validate

- **Time-based split:** train on earlier first-purchase cohorts, validate and test on later ones. Grouped splits by customer so the same customer is never in both sets.
- Fit imputers, scalers and encoders on the training set only (done with scikit-learn pipelines).
- Run validation tests (Section 9) before writing to `processed/`.
- **Check representativeness after cleaning:** compare distributions by skin type, tone, category and year before and after to make sure cleaning did not remove one group disproportionately.

---

## 5. Domain-Specific Solutions for Beauty and Wellness Data

| Problem | Proposed Solution | Benefit |
|---------|------------------|---------|
| Skin profile often undisclosed | Treat "undisclosed" as an informative category; add missing flags; test whether it predicts outcomes | Avoids wrong imputations; respects privacy |
| Pack sizes vary widely | Compute `price_per_ml` and price tier within category | Fair price comparison |
| Ingredient order signals concentration | Position-weighted ingredient scores; flag actives in the top 10 positions | Better proxy for potency than simple presence |
| Same formula sold in many variants | `product_family_id` through normalized names and identical ingredient lists | Prevents duplicated reviews and inflated popularity |
| Fake or incentivised reviews | Authenticity score from: duplicate text, very short length, verified-purchase flag, "received for free" phrases, burst posting; use as weight or filter and test sensitivity | More honest sentiment |
| Rating bias across users | `rating_deviation = user rating − product mean` (and user mean) | Separates user harshness from product quality |
| Skin-concern vocabulary differs | Domain lexicon mapping terms to concerns (acne, dryness, irritation, sensitivity) | Improves topic and sentiment features |
| Purging versus allergic reaction | Keep context words ("purging", "first week") so negative text is not automatically coded as a bad product | Fewer misread negatives |
| Code-mixed and regional language (Hinglish, Portuguese) | Language detection, separate pipelines, multilingual embeddings | Includes Indian-market voices |
| Seasonality of beauty demand | Add month and season features; compare against seasonal baselines | Avoids false trend findings |
| Gift sets and promotions distort first purchases | Flag gift-like items and heavy discounts | Cleaner repeat-rate comparisons |
| Proxy and synthetic data mixed with real | `data_origin` flag carried through all steps | Honest reporting |

---

## 6. Tools and Libraries

| Purpose | Tool | Why It Fits |
|---------|------|-------------|
| Tables and cleaning | `pandas` | Standard for tabular data: type conversion, `drop_duplicates`, `groupby`, merges, string methods |
| Numerics | `NumPy` | Fast array maths for log transforms, percentiles, vectorized rules |
| Large files | `DuckDB` / `Polars` / `PyArrow` (Parquet) | Handle millions of Amazon reviews with SQL-style queries and low memory use |
| Text matching | `RapidFuzz` | Fast fuzzy string matching for brands, products and ingredients |
| Regular expressions | `re` | Pattern-based cleaning of URLs, units, sizes and personal data |
| Language detection | `langdetect` / `fasttext`, `emoji` | Language routing; emoji to text |
| NLP preprocessing | `spaCy`, `NLTK` | Tokenization, lemmatization, custom stop-word handling |
| Missing data profile | `missingno` | Visual missingness matrix and correlations |
| Imputation, scaling, encoding, outliers | `scikit-learn` (`SimpleImputer`, `IterativeImputer`, `KNNImputer`, `RobustScaler`, `StandardScaler`, `OneHotEncoder`, `ColumnTransformer`, `Pipeline`, `IsolationForest`) | Fits on training data only and reapplies identically to new data, preventing leakage |
| Imbalance | `imbalanced-learn` | Resampling inside cross-validation |
| Validation | `pandera` or `Great Expectations` | Declare and test data rules (ranges, types, uniqueness) |
| Profiling | `ydata-profiling` | Quick automated first look |
| Time series | `pandas`, `statsmodels` | Resampling, interpolation, decomposition |
| Reproducibility | `Git`, `DVC` (optional), `Makefile` or script runner, `logging`, YAML config | Version code and data, run the pipeline with one command |
| Notebooks | `Jupyter` | Exploration and documentation (final logic moved into scripts) |

---

## 7. Pseudo-code and Code Sketches

> Adapt column names to the real files after inspecting them in Week 4.

### 7.1 Load, Standardize and Validate Types

```python
import pandas as pd, numpy as np, re

raw = pd.read_csv("data/raw/sephora/reviews.csv", low_memory=False)
df = raw.copy()                      # never modify raw

# standard column names
df.columns = (df.columns.str.strip().str.lower()
              .str.replace(r"[^0-9a-z]+", "_", regex=True).str.strip("_"))

# types (invalid values become NaT/NaN, then counted)
df["submission_time"] = pd.to_datetime(df["submission_time"], errors="coerce", utc=True)
df["rating"] = pd.to_numeric(df["rating"], errors="coerce")
bad_dates = df["submission_time"].isna().sum()

# domain rule: rating must be 1-5
rejected = df[~df["rating"].between(1, 5)]
df = df[df["rating"].between(1, 5)]
```

### 7.2 Duplicates

```python
df["text_key"] = (df["review_text"].fillna("").str.lower()
                  .str.replace(r"[^a-z0-9 ]", "", regex=True).str.strip())
before = len(df)
df = df.drop_duplicates(subset=["author_id", "product_id", "submission_time", "text_key"])
print("duplicates removed:", before - len(df))
```

### 7.3 Missing Values with Flags

```python
for col in ["skin_type", "skin_tone", "eye_color", "hair_color"]:
    df[f"{col}_missing"] = df[col].isna().astype(int)
    df[col] = df[col].fillna("unknown")

# numeric product fields: category median (fit on training data in the real pipeline)
df["price_missing"] = df["price_usd"].isna().astype(int)
df["price_usd"] = df["price_usd"].fillna(
    df.groupby("category")["price_usd"].transform("median")
)
```

### 7.4 Outlier Flags

```python
def iqr_flag(s, k=1.5):
    q1, q3 = s.quantile([0.25, 0.75])
    iqr = q3 - q1
    return (s < q1 - k * iqr) | (s > q3 + k * iqr)

def robust_z(s):
    med = s.median()
    mad = (s - med).abs().median()
    return 0.6745 * (s - med) / mad if mad else s * 0

logp = np.log1p(df["price_usd"])
df["price_outlier"] = iqr_flag(logp) | (robust_z(logp).abs() > 3.5)

# winsorize only for models sensitive to extremes
lo, hi = df["price_usd"].quantile([0.01, 0.99])
df["price_wins"] = df["price_usd"].clip(lo, hi)
```

### 7.5 Ingredient Parsing

```python
def parse_ingredients(text):
    if pd.isna(text):
        return []
    t = text.lower()
    t = re.sub(r"\([^)]*\)", "", t)           # remove parenthetical notes
    t = re.sub(r"\d+(\.\d+)?\s*%", "", t)     # remove percentages
    items = [re.sub(r"[^a-z0-9\s\-/]", "", p).strip() for p in t.split(",")]
    return [i for i in items if i]

SYNONYMS = {"aqua": "water", "vitamin c": "ascorbic acid"}   # extended from CosIng

df["ingredients"] = df["ingredients_raw"].apply(parse_ingredients)
df["ingredients"] = df["ingredients"].apply(
    lambda xs: [SYNONYMS.get(x, x) for x in xs]
)
df["has_ingredients"] = df["ingredients"].str.len().gt(0).astype(int)
df["has_retinol"]     = df["ingredients"].apply(lambda xs: int("retinol" in xs))
df["has_niacinamide"] = df["ingredients"].apply(lambda xs: int("niacinamide" in xs))
df["has_spf"]         = df["ingredients"].apply(
    lambda xs: int(any("ethylhexyl" in x or "avobenzone" in x for x in xs))
)
```

### 7.6 Text Cleaning and Privacy

```python
def clean_text(s):
    if pd.isna(s):
        return ""
    s = re.sub(r"https?://\S+", " ", s)
    s = re.sub(r"\S+@\S+", " ", s)              # emails
    s = re.sub(r"\s+", " ", s).strip()
    return s

df["review_clean"] = df["review_text"].apply(clean_text)
df["review_len"]   = df["review_clean"].str.split().str.len()
df["short_review"] = (df["review_len"] < 3).astype(int)   # input to authenticity score
```

### 7.7 Repeat-Purchase Label (Transaction Data)

```python
from pandas import Timedelta

# Remove cancellations and invalid rows
orders = orders[(orders["quantity"] > 0) & (orders["price"] > 0)]
orders = orders[~orders["invoice"].astype(str).str.startswith("C")]
orders = orders.dropna(subset=["customer_id"])

# Build first purchase date per customer
first = orders.groupby("customer_id")["order_date"].min().rename("first_date")
o = orders.merge(first, on="customer_id")
o["days_since_first"] = (o["order_date"] - o["first_date"]).dt.days

# Label: any order between day 1 and day 90 after first purchase
repeat = (
    o[(o["days_since_first"] > 0) & (o["days_since_first"] <= 90)]
    .groupby("customer_id").size().gt(0).rename("repeat_90d")
)
cust = first.to_frame().join(repeat).fillna({"repeat_90d": False})

# Keep only customers whose 90-day window is complete (others are right-censored)
data_end = orders["order_date"].max()
cust = cust[cust["first_date"] + Timedelta(days=90) <= data_end]
```

### 7.8 Leakage-Safe Preprocessing Inside a Model Pipeline

```python
from sklearn.compose import ColumnTransformer
from sklearn.pipeline import Pipeline
from sklearn.impute import SimpleImputer
from sklearn.preprocessing import StandardScaler, OneHotEncoder

num_cols = ["price_usd", "review_len", "rating_deviation"]
cat_cols = ["skin_type", "category", "brand"]

num = Pipeline([
    ("imp", SimpleImputer(strategy="median", add_indicator=True)),
    ("sc",  StandardScaler())
])
cat = Pipeline([
    ("imp", SimpleImputer(strategy="constant", fill_value="unknown")),
    ("oh",  OneHotEncoder(handle_unknown="ignore"))
])

pre = ColumnTransformer([("num", num, num_cols), ("cat", cat, cat_cols)])

# pre.fit(X_train)  → learns medians, means and categories from training data ONLY
# pre.transform(X_valid) → reuses them unchanged (no leakage)
```

### 7.9 Validation Tests

```python
import pandera as pa

schema = pa.DataFrameSchema({
    "rating":          pa.Column(float, pa.Check.isin([1.0, 2.0, 3.0, 4.0, 5.0])),
    "price_usd":       pa.Column(float, pa.Check.gt(0), nullable=True),
    "submission_time": pa.Column("datetime64[ns, UTC]"),
    "review_id":       pa.Column(str, pa.Check.str_length(min_value=1), unique=True),
})

schema.validate(df, lazy=True)   # collect ALL failures, not just the first
```

---

## 8. Documentation and Reproducibility

### 8.1 What Will Be Documented

| Artefact | Content | Format |
|----------|---------|--------|
| Data log | Source, URL, date accessed, version, licence, checksum, rows and columns | `data_log.csv` |
| Data dictionary | Each column: meaning, type, allowed values, unit, source, cleaning applied | Markdown or CSV |
| Cleaning log | For each step: rule, number of rows affected before and after, reason | Auto-written with Python `logging` to a file per run |
| Rejected rows | Rows removed with reason codes | `rejected/*.parquet` |
| Quality report | Missingness, duplicates, outliers, join match rates, before/after comparison | HTML from ydata-profiling + short summary |
| Decision record | Why a method was chosen and its impact on results | `docs/decisions.md` |
| Config | Thresholds and file paths (IQR factor, fuzzy-match cutoff, repeat window) | `config.yaml` |

### 8.2 Reproducibility Practices

- **One command runs everything:** `python run_pipeline.py --config config.yaml` or `make clean-data`, reading from `raw/` and writing to `interim/` and `processed/`.
- **Modular code:** one function per step in `src/cleaning/`, with notebooks used for exploration only.
- **Version control:** Git for code and config; raw data excluded through `.gitignore` but listed in the data log with download instructions; DVC can version large data files.
- **Fixed random seeds** wherever sampling or models are used.
- **Pinned environment:** `requirements.txt` or `environment.yml` with versions.
- **Unit tests** with `pytest` for key functions (ingredient parser, size converter, repeat-label builder) using small hand-made examples.
- **Run metadata:** each run records date, Git commit hash and config values.

### 8.3 Proposed Repository Layout

```
project/
├── config.yaml
├── data/
│   ├── raw/
│   ├── interim/
│   ├── processed/
│   └── rejected/
├── src/
│   ├── cleaning/
│   ├── features/
│   └── validation/
├── notebooks/
├── tests/
├── logs/
├── docs/
│   ├── data_dictionary.md
│   └── decisions.md
├── run_pipeline.py
└── requirements.txt
```

---

## 9. Quality Metrics and Validation

### 9.1 Before and After Metrics

| Metric | How Measured | Target |
|--------|-------------|--------|
| Missing share per column | `df.isna().mean()` | Documented; flagged columns have indicator |
| Duplicate rate | Duplicates removed / rows | Near zero in processed data |
| Invalid values | Rows breaking domain rules | Zero in processed data |
| Outlier share | Flagged rows / total rows | Reported per variable |
| Ingredient match rate | Matched INCI names / all ingredient tokens | > ~90% on a manual sample |
| Product match precision | Correct matches in a manual sample of ~200 | > ~95% |
| Join integrity | Row counts before and after each join | No unexplained change |
| Group retention | Share of rows kept by skin type, tone, category, year | No group loses much more than others |

### 9.2 Automated Tests (pandera or Great Expectations)

- Ratings between 1 and 5; prices and quantities above zero.
- Dates between a sensible minimum and the data extraction date.
- Primary keys unique (review ID, order line ID).
- Foreign keys exist (every review has a product).
- Every repeat label has a first order date earlier than the second.
- Pipeline is **deterministic:** running it twice gives identical file hashes.

### 9.3 Sensitivity Checks

To show cleaning choices do not drive the results, key findings will be repeated with:

| Setting | Alternatives Compared |
|---------|-----------------------|
| Outlier treatment | Kept vs. capped (winsorized) |
| Imputation method | Median vs. Iterative |
| Authenticity filter | On vs. off |
| Repeat purchase window | 30, 60, and 90 days |

---

## 10. How Clean Data Improves Analysis and Modelling

| Cleaning Step | Benefit for Analysis and Models |
|--------------|--------------------------------|
| Deduplication and authenticity scoring | More honest sentiment; models do not learn from spam |
| Missing value flags and sensible imputation | Keeps more rows; lets models use missingness as a signal; avoids false skin profiles |
| Outlier treatment | More stable coefficients, scaling and distance measures; fairer averages |
| Unit and price-per-ml standardization | Valid price comparisons and cleaner price-tier features |
| Ingredient standardization with position weights | Reliable ingredient features for prediction and content-based recommendation |
| Entity resolution and product families | Fewer duplicated products; better collaborative filtering and popularity baselines |
| Correct repeat labels with censoring handled | Valid target variable; correct survival analysis |
| Leakage-safe pipelines and time-based split | Realistic performance estimates that hold up in deployment |
| Privacy cleaning | Lower legal and ethical risk |
| Documentation and tests | Results others can reproduce and trust |

**Summary:** Clean and well-documented data makes the Week 1 hypotheses testable:
- Repeat-rate comparisons (**H1, H2**) need correct labels
- Review-feature models (**H3–H5**) need trustworthy text and ratings
- The recommender (**H6**) needs de-duplicated products and good ingredient vectors
- The trend analysis (**H7**) needs aligned, noise-aware time series

---

## 11. Risks and Limitations

| Risk | Mitigation |
|------|-----------|
| Over-cleaning removes real but unusual customers or products | Flag instead of delete where possible; compare results with and without treatment |
| Imputation introduces bias | Prefer flags and `unknown` categories; run sensitivity analysis |
| Fuzzy matching creates false matches | Conservative thresholds; manual audit; log match scores |
| Authenticity score is itself uncertain | Use as a weight or sensitivity filter, never as absolute truth |
| Cleaning differs by group and harms fairness | Compare group retention and error rates after cleaning |
| Time overrun on large files | Use Parquet, DuckDB and samples for development |
| Raw data or source terms change | Keep checksummed raw copies and the data log |

---

## 12. Time Allocation (~34 hours)

| Activity | Hours |
|----------|-------|
| Reviewing documentation and sample data for quality issues | 6 |
| Designing the pipeline stages and flowchart | 5 |
| Writing and testing cleaning code on sample data | 9 |
| Designing domain-specific solutions (ingredients, authenticity, variants) | 4 |
| Designing validation, logging and reproducibility setup | 4 |
| Quality and sensitivity planning | 2 |
| Writing, editing and formatting this document | 4 |
| **Total** | **34** |

### Next Steps (Week 4)
- Run the pipeline on the real files
- Report the quality metrics
- Use the cleaned data for exploratory analysis and segmentation

---

## 13. Conclusion

This pipeline converts messy public beauty and wellness data into reliable, documented tables ready for exploratory analysis and modelling. It combines standard practice (type fixing, deduplication, imputation, outlier handling, scaling) with **domain-specific solutions** for ingredients, product variants, undisclosed skin profiles, review authenticity and seasonality.

Leakage-safe scikit-learn pipelines, automated validation tests, logs and a single-command run make the work **transparent and repeatable**. In Week 4 the pipeline will be run on the real files, the quality metrics reported, and the cleaned data used for exploratory analysis and segmentation.

---

## 14. References and Tool Documentation

| Tool / Reference | Link |
|-----------------|------|
| pandas | https://pandas.pydata.org/docs/ |
| NumPy | https://numpy.org/doc/ |
| scikit-learn preprocessing | https://scikit-learn.org/stable/modules/preprocessing.html |
| scikit-learn pipelines | https://scikit-learn.org/stable/modules/compose.html |
| imbalanced-learn | https://imbalanced-learn.org/ |
| DuckDB | https://duckdb.org/docs/ |
| RapidFuzz | https://rapidfuzz.github.io/RapidFuzz/ |
| spaCy | https://spacy.io/usage |
| pandera | https://pandera.readthedocs.io/ |
| Great Expectations | https://docs.greatexpectations.io/ |
| ydata-profiling | https://docs.profiling.ydata.ai/ |
| missingno | https://github.com/ResidentMario/missingno |
| DVC | https://dvc.org/doc |
| Rubin (1976) — missing data mechanisms | Biometrika, 63(3), 581–592 |
| Little & Rubin (2019) — Statistical Analysis with Missing Data (3rd ed.) | Wiley |
| Iglewicz & Hoaglin (1993) — How to Detect and Handle Outliers | ASQC Quality Press |
| Kaufman et al. (2012) — Leakage in data mining | ACM TKDD, 6(4) |
| Data sources | See Week 2 Source Link Register |
