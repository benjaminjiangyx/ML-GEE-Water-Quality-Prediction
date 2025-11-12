# 🌊 SF Bay Water Quality Prediction with GEE & Machine Learning

This repository contains a Jupyter Notebook developed for the **Green AI Hackathon**. The project demonstrates an end-to-end machine learning pipeline to predict water quality parameters (specifically Total Nitrogen) in the San Francisco Bay Area by combining on-site measurements with powerful remote sensing data from Google Earth Engine (GEE).

## Project Goal

The primary objective is to build a regression model that can accurately predict water quality metrics (like total nitrogen) based on environmental and land-use features. This approach allows for monitoring and forecasting water quality in areas or at times where physical samples were not taken.

## Data Sources

This project integrates two key types of data:

1.  **Water Quality (Target Variable):**
    * **Source:** [CEDEN (California Environmental Data Exchange Network)](https://ceden.waterboards.ca.gov/)
    * **Data:** The notebook expects a user-provided `.tsv` file containing water quality measurements. The analysis focuses on **Total Nitrogen** (`nitrogen_total_kjeldahl_total_mg_l`) and **Total Phosphorus** (`phosphorus_as_p_total_mg_l`).

2.  **Environmental Features (Predictors):**
    * **Source:** Google Earth Engine (GEE)
    * **Datasets Used:**
        * **Land Cover (NLCD 2021):** Percentages of developed, forest, agriculture, water, and wetland areas.
        * **Impervious Surfaces (NLCD 2021):** Percentage of impervious surfaces.
        * **Vegetation (Sentinel-2):** 30-day mean NDVI (Normalized Difference Vegetation Index).
        * **Climate (CHIRPS):** 30-day total precipitation.
        * **Temperature (MODIS):** 30-day mean Land Surface Temperature.

## Methodology & Pipeline

The notebook is structured as a complete data pipeline:

1.  **Setup:** Installs all required libraries (`geemap`, `earthengine-api`, `sklearn`, `xgboost`, etc.) and authenticates the user with Google Earth Engine.
2.  **Data Cleaning (`clean_ceden_data`):** Ingests the raw, long-format CEDEN data and performs rigorous cleaning. This includes removing QA samples, filtering by result qualifiers, validating coordinates, and standardizing data types.
3.  **Data Transformation (`transform_ceden_to_wide`):** Pivots the cleaned data from a long to a wide format. Each row now represents a unique station-date pair, with water quality parameters as columns.
4.  **GEE Feature Engineering (`GEEFeatureExtractor`):** For each station-date sample, this class:
    * Creates a 5km buffer around the station's coordinates.
    * Queries all GEE datasets for environmental data within that buffer and relevant time window (e.g., 30-day lookback for weather).
    * Appends these features (e.g., `ndvi_mean`, `precipitation_mm`, `developed_pct`) to the corresponding sample.
5.  **Final Feature Preparation (`engineer_features`):** The merged (WQ + GEE) dataset is prepared for modeling by:
    * Imputing missing values.
    * Adding temporal features (e.g., `day_of_week`).
    * One-hot encoding categorical features (e.g., `season`).
    * Scaling all numerical features using `StandardScaler`.
6.  **Modeling & Evaluation (`WaterQualityPredictor`):**
    * A flexible class trains and evaluates multiple regression models.
    * The notebook successfully trains and compares **Random Forest** and **XGBoost** to predict Total Nitrogen.
    * Results are visualized with "Predicted vs. Actual," "Residual," and "Feature Importance" plots.
7.  **Forecasting (Experimental):** The final cell sets up a framework to predict water quality for the *next year* on a monthly basis, using the trained model and mean historical values for environmental features.

## Results

Based on the sample run in the notebook, the **XGBoost** model showed strong predictive power, achieving an **R² score of 0.92**. This indicates that environmental features derived from GEE are highly effective predictors of Total Nitrogen levels in the SF Bay.

**Feature Importance:** The model evaluation plots show which environmental factors (e.g., NDVI, temperature, land cover) are the most significant drivers of water quality.

## How to Run

To run this notebook yourself, follow these steps:

1.  **Clone the Repository:**
    ```bash
    git clone [your-repo-url]
    cd [your-repo-name]
    ```

2.  **Set up Your Environment:**
    * It is highly recommended to use a virtual environment.
    * Install the required Python packages. You can create a `requirements.txt` file with the following content:

    **`requirements.txt`**
    ```
    earthengine-api
    geemap
    pandas
    numpy
    scikit-learn
    xgboost
    statsmodels
    plotly
    dash
    geopandas
    folium
    matplotlib
    ```
    * Then install:
    ```bash
    pip install -r requirements.txt
    ```

3.  **Google Earth Engine (GEE) Setup:**
    * You **must** have a Google account that is registered for Google Earth Engine. You can sign up [here](https://earthengine.google.com/signup/).
    * You also need a GEE-enabled Google Cloud Project.
    * **In the first code cell**, change the project ID to your own:
        ```python
        # OLD:
        # ee.Initialize(project = 'greenaihackathon-476122')
        
        # NEW:
        ee.Initialize(project = 'your-gcloud-project-id')
        ```
    * When you run the first cell, you will be prompted to authenticate. Follow the link to authorize your Google account.

4.  **Get CEDEN Data:**
    * Go to the [CEDEN website query tool](https://ceden.waterboards.ca.gov/AdvancedQueryTool).
    * Select your parameters (e.g., "Nitrogen, Total Kjeldahl, Total" and "Phosphorus as P, Total"), a date range, and a location (e.g., SF Bay).
    * Download the data as a **Tab Delimited (.tsv)** file.

5.  **Run the Notebook:**
    * Execute the cells in order.
    * When you reach the **final pipeline cell (Cell 10)**, it will prompt you to upload your file.
    * Upload the `.tsv` file you downloaded from CEDEN.
    * The notebook will then run the entire cleaning, feature engineering, and modeling pipeline.

## Future Work

* **Implement Forecasting:** Complete the final cell to engineer the future dataset and run `predictor.predict()` to forecast WQ for the next year.
* **Expand Features:** Incorporate other GEE datasets like soil type (gSSURGO), elevation (DEM), or chlorophyll-a from Sentinel-3.
* **Full-Scale Training:** Run the pipeline on the *full* CEDEN dataset (not just a sample of 50) for a more robust model.
* **Deploy:** Use the imported `Plotly` and `Dash` libraries to create an interactive web application for exploring the results and making predictions.
