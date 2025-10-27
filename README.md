# Retail-Store-Location-Strategy
# Nordstrom Site Selection Prediction Project 🏬

## Executive Summary
**Business Question**: Given historical store performance and location characteristics, can we predict which new locations will be most profitable for Nordstrom stores?

**Think of this as**: Building a data-powered real estate recommendation engine - like Netflix recommendations, but for retail locations!

---

## Project Architecture

```
nordstrom_site_selection/
├── data/
│   ├── raw/                    # Messy, as-collected data
│   │   ├── demographics.csv
│   │   ├── competitor_locations.csv
│   │   ├── foot_traffic_pois.csv
│   │   └── nordstrom_stores.csv
│   └── processed/              # Cleaned, feature-engineered data
│       └── (you'll create these)
├── notebooks/
│   └── (your analysis notebooks)
├── src/
│   ├── generate_synthetic_data.py
│   └── (your cleaning/modeling scripts)
└── visualizations/
    └── (your Tableau/Python viz files)
```

---

## Data Dictionary

### 1. Demographics (`demographics.csv`)
**Purpose**: Understanding WHO lives near a location  
**Source**: Simulated Census Bureau data

| Column | Description | Data Type | Issues to Fix |
|--------|-------------|-----------|---------------|
| `zip_code` | Geographic identifier | String | Missing values, inconsistent formatting |
| `metro_area` | Major metropolitan region | String | Clean |
| `median_household_income` | Annual income ($) | Float | Missing values, negative values |
| `population` | Total residents | Integer | Generally clean |
| `median_age` | Age in years | Float | Missing values |
| `pct_bachelors_degree` | % with 4-year degree | Float | Missing values, >1.0 outliers |
| `pct_employed` | Employment rate | Float | Check for impossible values |
| `avg_household_size` | People per household | Float | Generally clean |

**Key Insight**: Nordstrom targets affluent areas. Income and education are crucial!

---

### 2. Competitor Locations (`competitor_locations.csv`)
**Purpose**: Market saturation analysis  
**Source**: Simulated retail location data

| Column | Description | Data Type | Issues to Fix |
|--------|-------------|-----------|---------------|
| `store_name` | Competitor brand | String | Inconsistent naming (Macys vs Macy's) |
| `latitude` | GPS coordinate | Float | Invalid values (>90 or <-90) |
| `longitude` | GPS coordinate | Float | Invalid values (>180 or <-180) |
| `metro_area` | Major metropolitan region | String | Clean |
| `store_sqft` | Store size (sq ft) | Float | Missing values |
| `year_opened` | Opening date | Mixed | Inconsistent formats (YYYY vs MM/DD/YYYY) |

**Key Insight**: Need to calculate "competitors within X miles" for each potential site!

---

### 3. Foot Traffic POIs (`foot_traffic_pois.csv`)
**Purpose**: High traffic = more potential customers  
**Source**: Simulated OpenStreetMap/Google Places data

| Column | Description | Data Type | Issues to Fix |
|--------|-------------|-----------|---------------|
| `poi_type` | Category of location | String | Typos, inconsistent casing |
| `latitude` | GPS coordinate | Float | Check validity |
| `longitude` | GPS coordinate | Float | Check validity |
| `metro_area` | Major metropolitan region | String | Clean |
| `daily_visitors` | Foot traffic estimate | Float | Negative values |
| `parking_spaces` | Available parking | Mixed | Various null representations |

**Key Insight**: Transit stations and malls drive spontaneous visits!

---

### 4. Nordstrom Store Performance (`nordstrom_stores.csv`)
**Purpose**: Ground truth - actual results to learn from  
**Source**: Simulated internal performance data

| Column | Description | Data Type | Issues to Fix |
|--------|-------------|-----------|---------------|
| `store_id` | Unique identifier | String | Duplicate values |
| `store_name` | Store identifier | String | Generally clean |
| `latitude` | GPS coordinate | Float | Check validity |
| `longitude` | GPS coordinate | Float | Check validity |
| `metro_area` | Major metropolitan region | String | Clean |
| `store_sqft` | Store size | Float | Mixed units (some in sq meters!) |
| `year_opened` | Opening date | Mixed | Inconsistent formats |
| `years_operating` | Time since opening | Integer | Generally clean |
| `annual_revenue` | Sales ($) | Float | Missing values |
| `monthly_foot_traffic` | Visitors per month | Float | Generally clean |
| `avg_transaction_value` | Average purchase ($) | Float | Generally clean |
| `customer_satisfaction` | Rating (1-5) | Float | Impossible values (<1 or >5) |
| `nearby_median_income` | Area income ($) | Float | Generally clean |

**This is your TARGET dataset** - you'll build features to predict `annual_revenue` or success metrics!

---

## The Feature Engineering Challenge

### Question for You:
**If you had these 4 separate datasets, how would you combine them into ONE dataset where each row represents a potential store location?**

<details>
<summary>Hint: Think about distance calculations and aggregations...</summary>

You'll need to:
1. **Join demographics** by location (lat/long → zip code mapping)
2. **Calculate competitor density**: "Count competitors within 5 miles"
3. **Aggregate POI features**: "Sum of daily visitors to POIs within 2 miles"
4. **Create target variable**: Use existing store performance as "success" labels

This requires learning:
- **Geospatial calculations** (Haversine distance formula)
- **Spatial joins** (matching points by proximity, not exact keys)
- **Aggregations** (count, sum, mean within radius)
</details>

---

## Technical Concepts Explained

### 1. Why Separate Files? (Database Normalization)

**Technical Definition**: Storing data in its most logical, non-redundant form where each table represents a single entity type.

**Simple Terms**: Imagine you run a library. Do you:
- **Option A**: Write the author's bio inside every single book they wrote?
- **Option B**: Have a separate "Authors" table you reference?

Option B is normalization! It prevents:
- **Redundancy**: Author bio stored 100 times if they wrote 100 books
- **Update anomalies**: If author's bio changes, you'd need to update 100 records
- **Data drift**: Different books might have different versions of the same bio

**In our project**: Demographics change annually (Census), competitors change monthly (new openings), but our stores are static. Keeping them separate mirrors how data actually arrives!

---

### 2. Haversine Distance (Why You Need It)

**Technical Definition**: Formula to calculate great-circle distance between two points on a sphere (Earth) given latitude/longitude.

**Simple Terms**: You can't use Pythagorean theorem (`sqrt(x² + y²)`) on a globe because Earth is curved!

**Analogy**: Imagine trying to measure the distance between New York and London:
- **Wrong way**: Draw a straight line through the Earth's core
- **Right way**: Measure along the Earth's surface (what planes actually fly)

Haversine does the "right way." Formula looks scary but Python has libraries (`geopy`) that do it for you!

**When you'll use it**: Calculating "How many competitors are within 5 miles of this location?"

---

### 3. Feature Engineering vs. Data Cleaning

**Question**: What's the difference?

**Data Cleaning**: Fixing problems with existing data
- Example: Converting `-63940` income to `63940` (fixing negative)
- Example: Standardizing "Macy's" vs "Macys" vs "MACYS"

**Feature Engineering**: Creating NEW columns that help your model learn patterns
- Example: Creating `competitor_density = count_within_5mi / area`
- Example: Creating `affluence_score = median_income * pct_bachelors_degree`

**Analogy**: 
- **Cleaning**: Washing vegetables before cooking
- **Engineering**: Chopping, seasoning, and combining ingredients to make the dish

Both are necessary! Clean first, then engineer.

---

## Your Roadmap (Recommended Order)

### Phase 1: Exploratory Data Analysis (EDA)
```python
# In a Jupyter notebook:
1. Load each CSV
2. Run df.info(), df.describe()
3. Identify missing values (df.isnull().sum())
4. Visualize distributions (histograms)
5. Document every data quality issue
```

**Question**: Why do EDA before cleaning?  
**Answer**: You need to SEE the problems to know how to fix them!

### Phase 2: Data Cleaning Pipeline
```python
# Create src/data_cleaning.py:
1. Handle missing values (imputation strategies)
2. Fix data types (dates, numbers)
3. Standardize formats (zip codes, store names)
4. Remove duplicates
5. Validate ranges (lat/long, percentages)
6. Save cleaned data to data/processed/
```

**Key Principle**: Make it REPRODUCIBLE. Anyone should be able to run your script and get the same clean data.

### Phase 3: Feature Engineering
```python
# Create src/feature_engineering.py:
1. Calculate distances (Haversine)
2. Create spatial aggregations
   - competitors_within_5mi
   - poi_foot_traffic_within_2mi
3. Create derived features
   - store_age
   - revenue_per_sqft
4. Join all datasets into final modeling dataset
```

### Phase 4: Modeling
```python
# In notebooks/modeling.ipynb:
1. Split data (train/test)
2. Try multiple models (Linear, Random Forest, XGBoost)
3. Evaluate performance (RMSE, R²)
4. Interpret feature importance
```

### Phase 5: Visualization (Tableau)
1. Export final predictions to CSV
2. Create interactive map dashboard
3. Build "what-if" analysis tools
4. Executive summary dashboard

---

## Data Quality Issues Summary (Your Checklist)

- [ ] Missing values in multiple columns
- [ ] Negative values where impossible (income, visitors)
- [ ] Inconsistent date formats (YYYY vs MM/DD/YYYY)
- [ ] Inconsistent string formats (store names, POI types)
- [ ] Duplicate entries (store IDs, zip codes)
- [ ] Invalid coordinates (lat >90, long >180)
- [ ] Percentage values >1.0 (should be 0-1 or 0-100?)
- [ ] Mixed units (square feet vs square meters)
- [ ] Multiple null representations ("N/A", "None", "", "null")

---

## Success Metrics for Your Portfolio

**Demonstrate these skills**:
1. ✅ **Data Engineering**: Clean, reproducible pipeline
2. ✅ **Geospatial Analysis**: Distance calculations, spatial joins
3. ✅ **Feature Engineering**: Thoughtful variable creation
4. ✅ **Modeling**: Model selection, evaluation, interpretation
5. ✅ **Communication**: Clear visualizations, actionable insights

**What makes a great portfolio project?**
- Document your thinking (comments + markdown cells)
- Show before/after (messy data → clean data)
- Explain tradeoffs ("I chose to drop these 5 rows because...")
- Connect to business value ("This feature improved accuracy by 8%")

---

## Questions to Think About (For Your Learning)

1. **Should you drop rows with missing values or impute them?** 
   - Depends on % missing and if data is MCAR (Missing Completely At Random)

2. **How do you choose the radius for "competitors within X miles"?**
   - Try multiple (1mi, 3mi, 5mi) and see which predicts best!

3. **Should you weight all competitors equally?**
   - Maybe closer competitors matter more? Try `1/distance` weighting

4. **How do you handle the fact that some metros have WAY more stores?**
   - Stratified sampling or metro-level fixed effects in your model

5. **What if a "bad" location is just poorly managed?**
   - This is confounding! You might add management metrics as controls

These are EXACTLY the kinds of questions interviewers love!

---

## Resources

- **Geopy docs**: https://geopy.readthedocs.io/ (for Haversine)
- **Pandas spatial join**: https://geopandas.org/
- **Tableau public**: Free version for portfolio projects
- **Model interpretation**: SHAP values for explaining predictions

---

**Ready to start?** Open a Jupyter notebook and begin with EDA!

**Next immediate step**: Run `jupyter notebook` in the `notebooks/` directory and create `01_exploratory_data_analysis.ipynb`
