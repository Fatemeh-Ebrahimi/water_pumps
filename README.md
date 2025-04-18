# water_pumps
## Water Pump Functionality Prediction

# Team: 
### Fatemeh Ebrahimi
### Kateryna Ponomarova
### James Donahue

## Project Overview
This project aims to develop machine learning models to predict the functionality status of water pumps in Tanzania based on factors like location, water quality, management structure, and technical specifications. It addresses the challenge of identifying functional, repair-needed, and non-functional pumps to enhance maintenance and ensure access to clean water for communities.

## Competition Link
[DrivenData - Pump it Up: Data Mining the Water Table](https://www.drivendata.org/competitions/7/pump-it-up-data-mining-the-water-table/)


## Project Goals
1. Develop classification models to predict three target classes:
   - Functional water pumps
   - Water pumps that need repair
   - Non-functional water pumps
2. Analyze which factors most strongly influence water pump functionality
3. Create geospatial visualizations to communicate findings effectively
4. Deploy a simple dashboard to showcase models and insights

## Data Description

### Dataset Source
The data comes from Taarifa and the Tanzanian Ministry of Water, which aggregates information about water points across Tanzania.
# Features in this dataset

- `amount_tsh` - Total static head (amount water available to waterpoint)
- `date_recorded` - The date the row was entered
- `funder` - Who funded the well
- `gps_height` - Altitude of the well
- `installer` - Organization that installed the well
- `longitude` - GPS coordinate
- `latitude` - GPS coordinate
- `wpt_name` - Name of the waterpoint if there is one
- `num_private` -
- `basin` - Geographic water basin
- `subvillage` - Geographic location
- `region` - Geographic location
- `region_code` - Geographic location (coded)
- `district_code` - Geographic location (coded)
- `lga` - Geographic location
- `ward` - Geographic location
- `population` - Population around the well
- `public_meeting` - True/False
- `recorded_by` - Group entering this row of data
- `scheme_management` - Who operates the waterpoint
- `scheme_name` - Who operates the waterpoint
- `permit` - If the waterpoint is permitted
- `construction_year` - Year the waterpoint was constructed
- `extraction_type` - The kind of extraction the waterpoint uses
- `extraction_type_group` - The kind of extraction the waterpoint uses
- `extraction_type_class` - The kind of extraction the waterpoint uses
- `management` - How the waterpoint is managed
- `management_group` - How the waterpoint is managed
- `payment` - What the water costs
- `payment_type` - What the water costs
- `water_quality` - The quality of the water
- `quality_group` - The quality of the water
- `quantity` - The quantity of water
- `quantity_group` - The quantity of water
- `source` - The source of the water
- `source_type` - The source of the water
- `source_class` - The source of the water
- `waterpoint_type` - The kind of waterpoint
- `waterpoint_type_group` - The kind of waterpoint

# 📊 Initial Data Exploration & First Impressions  

## ✅ Dataset Overview
- The training dataset contains **59,400 rows** and **40 features**.
- A separate labels file includes the `status_group` target column (pump functionality).

## 🧾 Data Types Summary

| Type     | Count | Description                             |
|----------|-------|-----------------------------------------|
| `object` | 30    | Categorical or string-type columns      |
| `int64`  | 7     | Integer columns (e.g., year, region)    |
| `float64`| 3     | Float columns (e.g., coordinates)       |


## 🧠 Key Observations from `.describe()` and `.head()`

- **Skewed distributions** observed in  `amount_tsh`, `population`, and `construction_year` features.
- **Zero values** in `gps_height`, `longitude`, and `latitude` likely indicate **missing or invalid data**.
- `num_private` appears to have **mostly zero values**, and may be **dropped** if uninformative.
-  Most features are **categorical (`object`)** – 30 out of 40 columns
-   Repetitive features like `extraction_type`, `extraction_type_group`, and `extraction_type_class` might offer **redundant information**
  
### 📌 Summary
- The dataset has **less unique information than it first appears**
- Careful handling of missing values, feature selection, and encoding will be needed before modeling
- Geospatial features (lat/lon) offer opportunities for external enrichment like weather or population data
- 
## 🧠 Observations from visualizing variables
- Some regions seem to have much better luck than others
- `quantity` could be a useful binary (enough, else), or even three?
- `management_group` does not seem to have an effect on the distribution. Nor does `permit`
- `quality_group` does not seem to bring much, as there are probably not enough "bad" observations. Similar with `waterpoint_type_group` (maybe?), `extraction_type_class`
- There's a strong case for a binary `payment` variable. Also `source`
- The watershed variabe `basin` might be good but may simply reflect seasonality captured in `quantity`
- 
## 🗺️ Geospatial Visualization

We created a basic geospatial map of water pumps across Tanzania using `geopandas`.

### Key Steps:
- Used the shapefile from https://www.nbs.go.tz/statistics/topic/gis
- Cleaned the pump data by removing rows with missing latitude or longitude
- Converted the cleaned data into a GeoDataFrame
- Plotted pump locations over a population-colored map of Tanzania
# Data cleaning and preparation

### 🧼 Missing Value Handling: Construction Year

- Replaced invalid `0` values in `construction_year` with `NaN`
- Imputed missing values in three steps:
  1. Used **median construction year** per **region + installer** combination
  2. For remaining missing values, used **region-level median**
  3. For any still missing, used the fallback: `recorded_year - 5`, based on data distribution
- This strategy filled all 20,709 missing entries using a context-aware approach
## 🧼 Missing Value Handling: `longitude`

- Replaced invalid longitude values (`0` or negative) with `NaN`
- Imputed 1,812 missing longitude values in two steps:
  1. Median per (`region`, `district_code`)
  2. Median per `region` (as fallback)
- This accounted for ~3% of the total data (59,400 pumps)
- While the map appearance remained largely unchanged, imputing these values ensures:
  - All pumps are included in geospatial analysis
  - No data is lost due to missing coordinates
  - Visualizations and models are complete and unbiased
- Output saved as: `data/cleaned_data_filled_V2.csv` 
### 🧼 Missing Value Handling: `gps_height`
- Replaced invalid `gps_height` values (≤ 0) with `NaN`
- Total filled entries: **21,934** (~37% of dataset)
- Imputed values using a two-step geographic approach:
  1. Median per `basin` (hydrological unit)
  2. Median per `region` (fallback)
-  No values left missing
- Output saved as: 'data/cleaned_data_filled_V3.csv'
- This method ensures geographic consistency by reflecting local altitude patterns

### 🧼 Missing Value Handling: `population`
- Replaced invalid values (`0`) with `NaN`
- Total filled entries: **21,381** (~36% of the dataset)
- Used a two-step geographic imputation:
  1. Median per `district_code`
  2. Median per `region` (fallback)
- After imputation, all missing values were filled
- As an additional step, we capped population values at **2,500**, based on domain knowledge and national-level population density in Tanzania
  - This avoids skew from unusually large or erroneous values
  #### 📉 Population Clipping
- Capped population values at **1,999** based on:
  - Distribution analysis
  - Realistic rural/urban estimates for Tanzania
- This prevents model distortion caused by a small number of extreme outliers
- Saved before/after distribution plots in `outputs/` folder:
  - `population_distribution_before_clipping.png`
  - `population_distribution_after_clipping.png`