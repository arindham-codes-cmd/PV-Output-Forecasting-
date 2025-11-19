**Originally developed by Arindham Krishna**  
GitHub: [arindham-codes-cmd](https://github.com/arindham-codes-cmd)
# PV-Output-Forecasting
## Project Overview
- This project explores solar energy forecasting using time series models like ARIMA and SARIMA. We analyse generation and weather sensor data from two solar plants, the goal is to predict daily energy output and support smarter grid management, load balancing, and consumer awareness.

- The dataset includes DC/AC output power readings, ambient and module temperature, irradiation, and cumulative yield, merged and modeled to simulate real-world forecasting scenarios. This project also benchmarks model performance for academic and portfolio purposes, with plans to visualize insights through a Power BI dashboard for business ready storytelling.
## Business Problem
Solar energy production is highly sensitive to weather conditions like cloud cover, temperature, and irradiation, These can cause unpredictable fluctuations in daily output. For solar companies managing multiple plants, this variability makes it difficult to plan grid operations, balance loads, and inform consumers about expected energy availability.

This project simulates a real-world scenario where a solar company operates two plants and wants to:
- Forecast next 7 day energy generation based on weather data
- Compare operational performance between plants
- Benchmark forecasting models for future deployment
  
By building predictive models and visualizing insights, the project supports better operational planning and showcases data driven decision making for both academic and portfolio purposes.
## Project Blueprint
This project was executed following a structured, five-phase blueprint designed to take raw solar data all the way to a dashboard-ready forecasting solution:
### Phase 1: Data Preparation
- Imported generation and weather data for both plants
- Cleaned nulls, duplicaes, and outliers
- Converted timestamps and extracted HOUR, DAY, MONTH features
- Merged weather + generation data on timestamp and source key
  
 Outcome: Two clean datasets—plant1_merged.csv and plant2_merged.csv
 
### Phase 2: Exploratory Data Analysis (EDA)
Investigated how weather variables influence energy generation

Key visualizations included
- IRRADIATION vs DC_POWER correlation Daily Yield trends over time.
- Hourly generation patterns (peak hours)
- Plant-wise generation comparison
- Inverter efficiency: DC to AC conversion
  
Outcome: Visual insights into solar plant behavior and performance drivers

### Phase 3: Time Series Forecasting
- Resampled data to daily yield
- Applied seasonal decomposition and stationarity checks
- Built ARIMA and SARIMA models for each plant
- Forecasted next 7 days and compared with regression models
  
Outcome: Forecast comparison charts and residual analysis

### Phase 4: Operational Dashboard (Power BI)
- Developed an interactive dashboard for decision-makers
- Used Python output CSVs for actuals and predictions
- Visuals included forecast trends, plant comparisons, and model metrics

Outcome: Interview-ready dashboard showcasing forecasting logic and insights

### Phase 5: Documentation
- Structured findings into a professional README

Outcome: A complete data-to-dashboard story, ready for portfolio and interviews

## Dataset Description
The project uses real-world solar energy data collected over 34 days from two solar power plants in India. Each plant provides two datasets:
- Generation Data: Captured at the inverter level, includes DC/AC power, daily yield, and total yield
- Weather Sensor Data: Captured at the plant level, includes ambient temperature, module temperature, and solar irradiation

Raw Data Folder
Contains the original CSV files:
- Plant_1_Generation_Data.csv
- Plant_1_Weather_Sensor_Data.csv
- Plant_2_Generation_Data.csv
- Plant_2_Weather_Sensor_Data.csv
  
Merged Data Folder
Cleaned and merged datasets for analysis:
- plant1_merged.csv
- Plant2_merged.csv

Dataset Structure
Each file includes timestamped entries with the following key columns:
### Generation Data Columns
- DATE_TIME: Timestamp of reading
- PLANT_ID: Unique plant identifier
- SOURCE_KEY: Inverter ID
- DC_POWER, AC_POWER: Power readings , units: KW
- DAILY_YIELD, TOTAL_YIELD: Energy produced , units: KWh
### Weather Data Columns
- DATE_TIME: Timestamp of reading
- PLANT_ID: Unique plant identifier
- SOURCE_KEY: Sensor ID
- AMBIENT_TEMPERATURE, MODULE_TEMPERATURE: Temperature readings , units: Degree Celsius
- IRRADIATION: Solar irradiance level, units: KW/m2
  
These datasets were merged on DATE_TIME to create a unified view of generation and weather conditions for each plant.

## Phase 1: Data Cleaning and Exploration (EDA)
This phase covers the first notebook: Exploratory Data Analysis EDA.ipynb, where we load, inspect, clean, and prepare the raw datasets for modeling.

### Step 1: Loading and Structuring the Data
We started by loading four CSVs, generation and weather data for both plants. To avoid repetitive code, we created a dictionary of datasets and used loops for inspection, reducing 4–5 lines into 2–3 clean ones. This approach improves readability, scalability, and keeps the notebook DRY (Don’t Repeat Yourself).

### Step 2: Initial Checks
We looped through each dataset to check:
- Shape and sample rows
- Null values and duplicates
- Data types and structure via .info()
  
All datasets are clean and had no missing or duplicate rows. However, the DATE_TIME column was imported as object, which is expected in pandas.

### Step 3: Date Format Fix

Before converting DATE_TIME to datetime, we checked format consistency. We found that Plant 1 Generation used DD-MM-YYYY, while others used YYYY-MM-DD. So we used:

<pre>pd.to_datetime(df['DATE_TIME'], dayfirst=True, errors='coerce')</pre> 

We didn’t manually set a format because mixed formats caused parsing errors. Using dayfirst=True allowed pandas to infer and standardize the format safely. The warning was triggered because pandas tried to parse YYYY-MM-DD with dayfirst=True, which can be ambiguous—but errors='coerce' ensured invalid formats were handled gracefully.

### Step 4: Statistical Insights

<img src="/Visuals/1.Statistical Information EDA.png" width="600"/>

we explored key metrics:
- DC_POWER & AC_POWER: Plant 1 had slightly higher average values than Plant 2
- DAILY_YIELD: Median yield was ~2600 KWh for Plant 1 and ~2900 KWh for Plant 2
- IRRADIATION: Mean values hovered around 0.66–0.69, with some zero readings indicating nighttime or cloudy conditions
  
These insights helped us understand baseline performance and variability.

### Step 5: Outlier Detection:

We applied IQR-based logic to detect outliers

<pre>```python 
#simple IQR-based outlier detection
for name, df in datasets.items():
    print(f"\n{name} — Outlier Summary:")

    # exclude non-numeric columns
    num_df = df.select_dtypes(include='number')

    for col in num_df.columns:
        Q1 = num_df[col].quantile(0.25)
        Q3 = num_df[col].quantile(0.75)
        IQR = Q3 - Q1
        lower_bound = Q1 - 1.5 * IQR
        upper_bound = Q3 + 1.5 * IQR

        outliers = ((num_df[col] < lower_bound) | (num_df[col] > upper_bound)).sum()

        if outliers > 0:
          print(f"  {col}: {outliers} potential outliers out of total {num_df.shape[0]} rows")
```</pre>

Findings:
- Plant 2 Generation had ~3000 outliers in DC/AC power
- Weather datasets had minimal outliers (1–2 values)
  
We chose not to remove outliers at this stage to preserve real-world variability. If needed, we’ll apply capping using .clip() and re-evaluate model performance.

If we want to clip the outliers we will go ahead with - <pre> df[col] = df[col].clip(lower_bound, upper_bound) </pre>

### Step 6: Timestamp Frequency Check

Before merging generation and weather data, we checked timestamp intervals. All datasets had consistent 15-minute gaps.

Why it matters:
Ensures proper alignment during merging Prevents accidental NaNs due to mismatched timestamps

Helps decide between merge() (exact match) vs merge_asof() (nearest match)
Since timestamps were regular, we used pd.merge()

### Step 7: Merging Datasets
We merged generation and weather data for each plant. Plant 2 merged cleanly, but Plant 1 had missing weather readings at 2020-06-03 14:00.

### Step 8: Fixing Missing Weather Data

We filled gaps using time-based interpolation because as seeing the above table looks like there is no record in Plant 1 Weather Dataset at 14:00:00. That is why when we merge Plant 1 Generation and Weather dataset we get null values

<pre>```Python
#we will try to fix this by interpolating values in plant 1 weather dataset.

# Make sure DATE_TIME is datetime and sorted
plant1_weather['DATE_TIME'] = pd.to_datetime(plant1_weather['DATE_TIME'])
plant1_weather = plant1_weather.set_index('DATE_TIME').sort_index()

# Create a full 15-minute timeline from start to end
full_index = pd.date_range(start=plant1_weather.index.min(), end=plant1_weather.index.max(), freq='15T')

# Reindex your DataFrame to include *all* 15-min timestamps
plant1_weather_reindexed = plant1_weather.reindex(full_index)

# Now interpolate missing weather readings
plant1_weather_reindexed[['AMBIENT_TEMPERATURE', 'MODULE_TEMPERATURE', 'IRRADIATION']] = (
    plant1_weather_reindexed[['AMBIENT_TEMPERATURE', 'MODULE_TEMPERATURE', 'IRRADIATION']]
        .interpolate(method='time', limit_direction='both')
)

plant1_weather_reindexed.head(10)

</pre>

What this does:
- Ensures every 15-min slot has a value
- Uses time-aware interpolation to estimate missing readings
- Preserves temporal continuity without introducing bias

### Step 9: Feature Engineering 
Here, we remerge the data, then we go ahead and extract date time components required for modeling. 

## Phase 2: Visual Plots, Insights and Analysis

This phase uses the merged datasets to uncover operational patterns and performance differences between Plant 1 and Plant 2. Each plot is backed by code logic and real-world interpretation.

1. Irradiation vs DC Power

   <img src="/Visuals/2.Irradiation vs DC Power.png" width="600"/>
We plotted daily average irradiation against daily DC power output for both plants. The Pearson correlation coefficient was:
- Plant 1: 0.993 → near-perfect linear relationship
- Plant 2: 0.871 → weaker correlation, more scatter
  
Interpretation:
- In Plant 1, higher sunlight (irradiation) consistently leads to higher DC power output.
- In Plant 2, even when irradiation increases, DC power remains low or inconsistent.
- 
Possible reasons:
- Inverter degradation or partial shading
- Sensor misalignment or calibration issues
- Maintenance gaps or panel-level faults
- Data quality issues (e.g., timestamp mismatches or unit inconsistencies)

2. Daily Yield Comparison:

We grouped data by DATE_TIME.dt.date and aggregated DAILY_YIELD to compare total daily energy output in MWh.

<img src="/Visuals/3.Daily_Yield Comparison.png" width="900"/>

Insights:
- Plant 1 consistently outperforms Plant 2 across most days.
- The gap widens on high-irradiation days, suggesting better conversion and fewer losses.
  
This grouped data was stored in a list and later concatenated into a combined DataFrame for plotting.

3. Hourly AC Power Trend (Peak Hour Analysis)
<img src="/Visuals/4.AC Power Trend Hourly.png" width="600"/>
We plotted average AC power per hour across all days.
- Plant 1 peaks smoothly between 11 AM to 1 PM, averaging 800–850 Watts per inverter
- Plant 2 shows erratic peaks and lower output (~600 Watts max)
  
Interpretation:
- If each plant has 20 inverters:
- Plant 1 → 20 × 800 = 16,000 Watts or 16 kW peak AC output
- Plant 2 → 20 × 600 = 12,000 Watts or 12 kW
  
This suggests Plant 1 has better panel exposure, inverter health, or fewer operational disruptions.

4. Inverter Conversion Efficiency (DC to AC)

We calculated energy using:
<pre>Energy (MWh) = Power (W or kW) × Time (0.25 hours) / 1e6</pre>
Then grouped by SOURCE_KEY_x to compute total DC and AC energy per inverter and their conversion ratio:
<pre>Conversion Ratio (%) = (AC Energy / DC Energy) × 100 </pre>
Initial finding:
- Plant 1 showed ~9% conversion, which is unrealistically low.
Diagnosis:
- DC_POWER in Plant 1 was likely in Watts, while AC_POWER was in kW
- After converting DC_POWER by dividing by 1000, the efficiency corrected to ~97–98%—which is realistic and healthy.

## Phase 3: Time Series Forecast Model - ARIMA & SARIMA  
This phase focuses on building forecasting models to predict next-day solar energy generation using historical daily yield data. We used both ARIMA and SARIMA to benchmark performance and capture trends, seasonality, and residual noise.

### Step 1: Daily Aggregation
We grouped the merged datasets by DATE_TIME.dt.date and summed DAILY_YIELD to get daily totals. These were converted to MWh for clarity:

daily_yield['DAILY_YIELD_MWh'] = round(daily_yield['DAILY_YIELD'] / 1e6, 2)

This gave us 34 days of daily yield data per plant, which we stored in daily_df for modeling.

### Step 2: Seasonal Decomposition
Using seasonal_decompose(), we broke down each time series into:
- Trend: Long-term direction of energy output
- Seasonality: Daily cycles or repeating patterns
- Residual: Random noise or anomalies

<img src="/Visuals/5.Seasonal Decompose.png" width="600"/>
Both plants showed clear seasonal patterns and smooth trends, validating the use of SARIMA.

### Step 3: Stationarity Check (ADF Test)
We applied the Augmented Dickey-Fuller test to check if the data was stationary:
- Plant 1: p-value = 0.00007 → Stationary
- Plant 2: p-value = 0.0389 → Stationary
Since both passed the test (p ≤ 0.05), we proceeded without differencing.

### Step 4: ACF & PACF Plots
<img src="/Visuals/6.ACF Plot.png" width="600"/>
<img src="/Visuals/7.PACF Plot.png" width="600"/>
We plotted Autocorrelation (ACF) and Partial Autocorrelation (PACF) to identify lag dependencies. These plots helped us choose ARIMA parameters:
- ARIMA Order: (2, 0, 2)
- SARIMA Seasonal Order: (1, 1, 1, 7) → Weekly seasonality

### Step 5: Train-Test Split
We split the time series into:
- Train: First 27 days
- Test: Last 7 days
  
This setup allowed us to forecast the final week and compare predictions with actuals.

### Step 6: ARIMA Modeling
We fit an ARIMA(2,0,2) model and forecasted 7 days ahead. 
<img src="/Visuals/8.ARIMA Plot.png" width="900"/>

Visuals: Forecast lines closely followed actuals for Plant 1, with slightly more deviation in Plant 2.

### Step 7: SARIMA Modeling
We fit a SARIMA(2,0,2)(1,1,1,7) model to capture weekly seasonality. 
<img src="/Visuals/9.SARIMA Plot.png" width="900"/>

Interpretation:
SARIMA captured seasonality but slightly overfit Plant 1’s fluctuations.
ARIMA performed better for Plant 1; SARIMA was comparable for Plant 2.
Key Takeaways
- ARIMA is effective for short-term yield forecasting when seasonality is minimal or stable.
- SARIMA adds value when patterns repeat weekly or monthly.
  Forecasting accuracy depends on data quality, inverter consistency, and weather variability.

##  Results & Insights
The forecasting models, ARIMA and SARIMA successfully predicted solar power output with reasonable accuracy across both plants. Despite minor deviations, the predictions aligned well with actual trends, validating the use of time series techniques for short-term energy planning.

The Power BI dashboard, powered by these forecasts, enables energy planners to:
 - Anticipate dips in solar generation due to weather variability or system inefficiencies
 - Optimise energy dispatch by aligning predicted output with grid demand
 - Schedule maintenance proactively, especially for underperforming inverters
 - Compare plant performance and conversion efficiency in real-time
   
Together, the models and dashboard form a robust decision-support system for solar operations and turning raw data into actionable insights.

## Credits

This project was originally created and maintained by **Arindham Krishna**.  
All logic, forecasting methodology, and data handling scripts were authored by [arindham-codes-cmd](https://github.com/arindham-codes-cmd).

## Fork Notice

If you’re viewing this project from a forked repository, please note that the original version was developed by **Arindham Krishna**.  
Visit the source repo here: [PV-Output-Forecasting](https://github.com/arindham-codes-cmd/PV-Output-Forecasting)

