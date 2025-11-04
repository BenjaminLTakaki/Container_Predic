# Container Price Prediction Project

Predicting Europe Base Port container prices 1-week ahead using Shanghai freight data, oil prices, geopolitical events, port activity, and chokepoint metrics.

## Setup Instructions

### 1. Create and Activate Virtual Environment

```powershell
# Create venv
python -m venv venv

# Activate (Windows PowerShell)
.\venv\Scripts\Activate.ps1

# Activate (Windows CMD)
venv\Scripts\activate.bat

# Activate (Linux/Mac)
source venv/bin/activate
```

### 2. Install Dependencies

```powershell
pip install -r requirements.txt
```

### 3. Run Notebooks in Order

1. **01_data_collection.ipynb** - Collect and process all raw data sources
   - Shanghai Containerized Freight Index
   - WTI & Brent crude oil prices
   - GDELT geopolitical disruption data
   - Shanghai port activity metrics
   - Chokepoint traffic data (Suez, Malacca, etc.)
   - China monthly trade statistics

2. **02_data_understanding.ipynb** - Feature engineering and exploratory analysis
   - Create time-lagged features
   - Analyze correlations
   - Visualize disruption events
   - Generate feature importance rankings

3. **03_forecasting_pipeline.ipynb** - Build and evaluate prediction models

## Project Structure

```
Container_Predic/
├── data/
│   ├── raw/                          # Raw CSV files (manual downloads)
│   │   ├── Shanghai_Containerized_Freight_Index.csv
│   │   ├── DCOILWTICO.csv
│   │   ├── DCOILBRENTEU.csv
│   │   ├── china_port_activity.csv
│   │   ├── chokepoints.csv
│   │   └── china_monthly_trade.csv
│   └── processed/                    # Cleaned weekly datasets
│       ├── freight_data.csv
│       ├── oil_wti.csv
│       ├── oil_brent.csv
│       ├── geopolitical_data.csv
│       ├── shanghai_port_weekly.csv
│       ├── chokepoints_weekly.csv
│       ├── china_monthly_trade_weekly.csv
│       └── model_data.csv           # Final feature matrix
├── 01_data_collection.ipynb
├── 02_data_understanding.ipynb
├── 03_forecasting_pipeline.ipynb
├── requirements.txt
└── README.md
```

## Data Sources

1. **Shanghai Containerized Freight Index (SCFI)**
   - Weekly container freight rates from Shanghai to global routes
   - Target: Europe Base Port price

2. **Oil Prices (EIA)**
   - Daily WTI and Brent crude prices
   - Affects shipping fuel costs

3. **Geopolitical Disruptions (GDELT via BigQuery)**
   - Weekly event counts for shipping-critical regions
   - Identifies black swan events (Red Sea crisis, Suez blockage, etc.)

4. **Shanghai Port Activity (IMF PortWatch)**
   - Daily container port calls and cargo volumes
   - Origin port for Europe routes

5. **Chokepoint Traffic (IMF PortWatch)**
   - Daily vessel traffic through critical maritime chokepoints
   - Suez Canal, Malacca Strait, Bab el-Mandeb, etc.

6. **China Monthly Trade (IMF PortWatch)**
   - Monthly trade volumes and values
   - Macro context for shipping demand

## Key Features Engineered

- **Temporal lags**: 1-6 week lags for disruption events based on shipping times
- **Port metrics**: Shanghai container throughput, congestion indicators
- **Chokepoint metrics**: Vessel counts and capacity utilization
- **Oil derivatives**: Price changes and volatility
- **Trade indicators**: Month-over-month growth rates

## Dependencies

See `requirements.txt` for full list. Key packages:
- pandas, numpy - data manipulation
- matplotlib, seaborn, plotly - visualization
- statsmodels - time series analysis
- scikit-learn, xgboost - machine learning
- yfinance - financial data (if needed)

## Notes

- All processed data is saved to `data/processed/` for reproducibility
- Notebooks are designed to run sequentially
- Feature importance analysis uses both correlation and XGBoost
