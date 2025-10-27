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
**Technical Definition**: Storing data in its most logical, non-redundant form where each table represents a single entity type.

**In my project**: Demographics change annually (Census), competitors change monthly (new openings), but our stores are static. Keeping them separate mirrors how data actually arrives!

---

### 2. Haversine Distance (Why You Need It)

**Technical Definition**: Formula to calculate great-circle distance between two points on a sphere (Earth) given latitude/longitude.

---

### 3. Feature Engineering vs. Data Cleaning

**Data Cleaning**: Fixing problems with existing data
- Example: Converting `-63940` income to `63940` (fixing negative)
- Example: Standardizing "Macy's" vs "Macys" vs "MACYS"

**Feature Engineering**: Creating NEW columns that help your model learn patterns
- Example: Creating `competitor_density = count_within_5mi / area`
- Example: Creating `affluence_score = median_income * pct_bachelors_degree`

## Your Roadmap (Recommended Order)


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

## Resources

- **Geopy docs**: https://geopy.readthedocs.io/ (for Haversine)
- **Pandas spatial join**: https://geopandas.org/
- **Tableau public**: Free version for portfolio projects
- **Model interpretation**: SHAP values for explaining predictions
