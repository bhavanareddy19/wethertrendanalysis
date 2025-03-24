# wethertrendanalysis
# Global Weather Analysis & Forecasting

This project leverages a global weather dataset to perform advanced data analysis, forecasting, and unique insights into climatic patterns. The analyses were conducted using a combination of classical and modern forecasting techniques, and the insights are presented in a clear, step-by-step format.

---

## PM Accelerator Mission

PM Accelerator is committed to making industry-leading tools and educational resources accessible to individuals from diverse backgrounds, thereby leveling the playing field for future product management leaders. Its mission is to empower aspiring and experienced product managers by granting them unparalleled access to industry expertise, fostering a robust product management ecosystem, and equipping them with advanced skills in AI-driven product management.

---

## Data Understanding & Cleaning

- **Dataset Overview:**
  - The dataset is loaded from `GlobalWeatherRepository.csv` using pandas.
  - It contains **60,218 records** and **41 columns** capturing various weather-related metrics, geographic information, and air quality indicators.

- **Data Exploration:**
  - Using `df.info()`, `df.describe()`, and `isnull()`, it was determined that:
    - There are 41 columns with no missing values.
    - Data types include:
      - **Object:** 11 columns (e.g., `country`, `location_name`, `timezone`, `condition_text`, `sunrise`, `sunset`, etc.)
      - **Float64:** 23 columns (e.g., `latitude`, `longitude`, `temperature_celsius`, etc.)
      - **Int64:** 7 columns (e.g., `last_updated_epoch`, `wind_degree`, `humidity`, `cloud`, and various air quality indexes).

- **Converting Datetime Information:**
  - The `last_updated` column (initially an object) is converted into a datetime format, which is crucial for time series analysis and forecasting.

- **Handling Duplicates:**
  - Descriptive statistics show that although there are 60,218 records, there are only 10,073 unique `last_updated` timestamps. This indicates multiple readings per timestamp.
  - **Aggregation Strategy:**  
    Duplicate timestamps are aggregated by taking the mean of `temperature_celsius` for each unique timestamp. This ensures the time series has a unique index.

---

## Exploratory Data Analysis (EDA)

- **Temperature Distribution:**
  - A histogram of `temperature_celsius` was created.
  - **Findings:**  
    - Most readings fall between 10°C and 35°C with a peak around 25–30°C.
    - A few very low values (below -20°C) and very high values (above 40°C) indicate that the dataset covers a diverse range of climates and extreme weather events.

- **Time Series View:**
  - A time series plot (from May 2024 to April 2025) displays daily fluctuations in temperature.
  - **Observations:**  
    - Daily ups and downs are visible due to data from different locations.
    - Some large drops below 0°C and spikes above 40°C suggest either genuine extreme events or measurement anomalies.

- **Initial Data Size:**
  - The dataset comprises 60,218 rows and 44 columns, with no missing values.

---

## Anomaly Detection

- **Method:**
  - An IsolationForest model was applied to `temperature_celsius` to identify outliers.
  - The model labels each reading as either an inlier (1) or an outlier (-1).

- **Results:**
  - A total of **565** temperature readings were flagged as outliers (mostly extreme values).
  - These points are highlighted in red on the time series plot.

- **Data Cleaning:**
  - After removing the flagged outliers, the dataset reduced from 60,218 rows to 59,623 rows.
  - This removal focuses the analysis on typical temperature patterns, which can improve the accuracy of subsequent forecasting and advanced analyses.

---

## Forecasting with Multiple Models

### Time Series Preparation

- **Aggregation:**
  - Temperature data is grouped by date (by averaging daily temperatures) to ensure a single value per day.
- **Daily Frequency:**
  - The time series is set to a daily frequency (`.asfreq('D')`) and missing dates are filled using forward-fill.
- **Prophet Data Preparation:**
  - For Prophet, the time series is reformatted into two columns:
    - **ds:** Date
    - **y:** Daily temperature (temperature_celsius)

### ARIMA Forecasting

- **Model Overview:**
  - ARIMA (AutoRegressive Integrated Moving Average) uses past values and past errors to forecast future values.
- **Implementation:**
  - An ARIMA model with order (5, 1, 0) is trained on the historical daily temperature data.
  - It forecasts temperatures for the next 30 days.
- **Visualization & Insight:**
  - The ARIMA forecast (green line) suggests that temperatures will remain relatively steady at around 4–5°C after March 2025.

### Prophet Forecasting

- **Model Overview:**
  - Prophet automatically handles trends, seasonality, and holiday effects with minimal manual tuning.
- **Implementation:**
  - Prophet is applied to the reformatted data (with columns `ds` and `y`).
  - The model forecasts the next 30 days.
- **Visualization & Insight:**
  - Prophet’s forecast (orange line) shows that temperatures are expected to continue decreasing from around 5°C to roughly 2–3°C over the next few months.

### Ensemble Forecast

- **Rationale:**
  - Combining forecasts from ARIMA and Prophet leverages the strengths of both models for a more balanced prediction.
- **Method:**
  - For each future date, the forecasts from ARIMA and Prophet are averaged to produce the ensemble forecast.
- **Visualization & Insight:**
  - The ensemble forecast (purple line) is generally more moderate and stable than either individual model, offering a balanced prediction of future temperatures.

- **Key Comparison Points:**
  - **Observed Data (Blue Line):** Historical temperatures start around 25–30°C and drop to about 5°C by early 2025.
  - **ARIMA (Green Line):** Projects steady temperatures near 4–5°C after March 2025.
  - **Prophet (Orange Line):** Predicts a continued downward trend, decreasing to 2–3°C.
  - **Ensemble (Purple Line):** Balances both models’ forecasts, suggesting temperatures will remain moderate (around 5–7°C) with a gradual decline.

---

## Unique Analyses

### 1. Annual Average Temperature Trend by Country

- **Goal:**  
  To analyze how each country’s average temperature changes year by year.
- **Method:**  
  - Data is grouped by `country` and `year` (extracted from the `Date` column).
  - The mean of `temperature_celsius` is computed for each country annually.
  - All countries are plotted on a single graph, with each line representing a country.
- **Interpretation:**  
  - Downward trends indicate countries with decreasing average temperatures.
  - Upward trends indicate countries with increasing average temperatures.
  - A crowded graph may benefit from interactive or segmented views.

### 2. Environmental Impact: Correlation Matrix

- **Goal:**  
  To understand the relationship between air quality (e.g., `air_quality_PM2.5`) and weather parameters (temperature, humidity, pressure, wind).
- **Method:**  
  - A correlation matrix is computed using the cleaned data.
  - A heatmap visualizes positive correlations (yellow) and negative correlations (purple/blue).
- **Interpretation:**  
  - Values close to +1 indicate strong positive correlation.
  - Values close to -1 indicate strong negative correlation.
  - Values near 0 suggest little or no linear relationship.

### 3. Feature Importance (Random Forest)

- **Goal:**  
  To identify which weather and air quality factors most influence temperature.
- **Method:**  
  - A RandomForestRegressor is trained to predict `temperature_celsius` using features like `humidity`, `pressure_mb`, `wind_mph`, and `air_quality_PM2.5`.
  - Feature importance scores and Mean Squared Error (MSE) are computed.
- **Interpretation:**  
  - Higher importance scores indicate a greater influence on temperature predictions.
  - A lower MSE suggests better overall model performance.

### 4. Spatial Distribution of Temperature

- **Goal:**  
  To visualize how temperature varies by geographic location.
- **Method:**  
  - A scatter plot is created with longitude and latitude as coordinates.
  - Each point’s color represents the temperature, using a color scale (e.g., `viridis`).
- **Interpretation:**  
  - Regions with similar colors indicate areas with similar temperatures.
  - Geographic patterns (e.g., cooler areas at higher latitudes) become evident.

### 5. Comparing Average Temperature by Country

- **Goal:**  
  To compare which countries have higher or lower average temperatures.
- **Method:**  
  - The data is grouped by country, and the average `temperature_celsius` is computed.
  - A bar chart is plotted in descending order of average temperature.
- **Interpretation:**  
  - Taller bars indicate countries with higher average temperatures.
  - Shorter bars indicate cooler countries.
  - This visualization provides a quick ranking of countries by climate.

---

## Final Insights

- The analyses reveal a wide range of temperature behaviors across different regions and time periods.
- Outlier detection and removal lead to clearer forecasting and more reliable trend analysis.
- ARIMA and Prophet models provide different perspectives on future temperatures; the ensemble approach often offers a balanced forecast.
- Unique analyses such as climate trends, environmental correlations, feature importance, spatial patterns, and country comparisons collectively offer deep insights into global weather patterns.

---

This README provides a detailed yet concise overview of the project, its methodology, and key findings, making it easy to understand the workflow and insights derived from the analysis.
