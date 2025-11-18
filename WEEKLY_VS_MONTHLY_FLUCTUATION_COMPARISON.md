# Weekly vs Monthly Price Fluctuation Prediction Comparison

## Overview

This document compares two different prediction horizons for container freight price fluctuations:
- **Weekly Fluctuation Prediction** (Notebook 11): Predicts price change 1 week ahead
- **Monthly Fluctuation Prediction** (Notebook 12): Predicts price change 4 weeks ahead

Both notebooks use the same three machine learning models (Linear Regression, Decision Tree, K-Nearest Neighbors) to predict **price fluctuations** (change in price) rather than absolute prices.

---

## How to Use This Guide

### Running the Notebooks

To populate this comparison with actual results:

1. **Run Notebook 11** (Weekly Fluctuations):
   ```bash
   jupyter notebook 11_seasonality_price_fluctuation_models.ipynb
   ```
   - Generates: `data/processed/seasonality_model_comparison.csv`
   - Generates: `data/processed/seasonality_fluctuation_predictions.csv`

2. **Run Notebook 12** (Monthly Fluctuations):
   ```bash
   jupyter notebook 12_monthly_price_fluctuation_models.ipynb
   ```
   - Generates: `data/processed/monthly_fluctuation_model_comparison.csv`
   - Generates: `data/processed/monthly_fluctuation_predictions.csv`
   - Generates: `data/processed/weekly_vs_monthly_comparison.png`

3. **Review Results**: Check the generated CSV files and update this document with actual metrics

---

## Actual Performance Results

### Weekly Fluctuation Prediction (Notebook 11)

**Prediction Horizon:** 1 week ahead
**Features:** 23 total (16 seasonal + 7 price features)
**Test Set Size:** 77 samples

| Model | RMSE (USD) | MAE (USD) | R² | Direction Accuracy |
|-------|------------|-----------|----|--------------------|
| **Linear Regression** | **181.76** | **136.47** | **0.293** | **64.94%** |
| Decision Tree | 224.83 | 164.84 | -0.083 | 61.04% |
| K-Nearest Neighbors | 186.90 | 138.05 | 0.252 | 63.64% |

**Winner:** Linear Regression
- Explains 29.3% of weekly price fluctuation variance
- Predicts correct direction 65% of the time (15% better than random)
- Average error of $136 (MAE) with $182 RMSE
- Seasonal features (Christmas, peak season) significantly improve performance

### Monthly Fluctuation Prediction (Notebook 12)

**Prediction Horizon:** 4 weeks (1 month) ahead
**Features:** 30 total (lagged price features + rolling statistics)
**Test Set Size:** 76 samples (estimated)

| Model | RMSE (USD) | MAE (USD) | R² | Direction Accuracy |
|-------|------------|-----------|----|--------------------|
| **Linear Regression** | **[Run NB12]** | **[Run NB12]** | **[Run NB12]** | **[Run NB12]** |
| Decision Tree | [Run NB12] | [Run NB12] | [Run NB12] | [Run NB12] |
| K-Nearest Neighbors | [Run NB12] | [Run NB12] | [Run NB12] | [Run NB12] |

**Expected Results** (based on notebook 06 month-ahead price predictions):
- Linear Regression RMSE: $600-$800 (3-4x higher than weekly)
- R²: 0.10-0.25 (lower variance explained)
- Direction Accuracy: 55-62% (harder to predict direction)
- Performance degradation: 200-400% increase in RMSE

**Note:** Run Notebook 12 to populate actual results. The expected values are based on similar monthly prediction tasks from notebook 06.

---

## Performance Degradation Analysis

### Formula

```
Degradation % = ((Monthly_RMSE - Weekly_RMSE) / Weekly_RMSE) × 100
```

### Expected Degradation

Based on typical patterns in freight price prediction:

**Scenario 1: High Volatility (Degradation > 200%)**
```
Weekly RMSE:  $181.76
Monthly RMSE: $600 (estimated)
Degradation:  +230%
```
**Interpretation:** Prices are highly volatile across 4-week periods. Contracts renew frequently with significant price changes. Monthly predictions are much harder due to genuine uncertainty.

**Scenario 2: Moderate Volatility (Degradation 100-200%)**
```
Weekly RMSE:  $181.76
Monthly RMSE: $400 (estimated)
Degradation:  +120%
```
**Interpretation:** Moderate price stickiness exists, but contracts do renew with notable price adjustments. Expected for freight markets with 2-4 week contract cycles.

**Scenario 3: High Stickiness (Degradation < 100%)**
```
Weekly RMSE:  $181.76
Monthly RMSE: $300 (estimated)
Degradation:  +65%
```
**Interpretation:** Strong price persistence over 4 weeks. Contracts remain stable. High autocorrelation means monthly predictions are almost as easy as weekly predictions.

### Interpretation Guidelines

| Degradation % | Market Condition | Contract Dynamics | Actionable Insight |
|---------------|------------------|-------------------|-------------------|
| **> 200%** | High volatility | Frequent renewals, large price swings | Focus on short-term predictions, avoid long contracts |
| **100-200%** | Moderate volatility | Standard 2-4 week contracts | Use weekly for ops, monthly for strategy |
| **50-100%** | Low volatility | Stable contracts, predictable renewals | Monthly predictions viable for planning |
| **< 50%** | Very sticky prices | Long-term contracts, rare price changes | Both horizons work well, high autocorrelation |

---

## Methodology Comparison

### Common Approach

Both notebooks follow the same general methodology:

1. **Target Variable Creation**:
   - Weekly: `price_fluctuation_1w = price_1w_ahead - current_price`
   - Monthly: `price_fluctuation_1m = price_4w_ahead - current_price`

2. **Feature Engineering**:
   - Use lagged features to avoid data leakage
   - Include rolling statistics and percentage changes
   - Select top 30 features based on correlation with target

3. **Models Trained**:
   - Linear Regression (baseline interpretable model)
   - Decision Tree with GridSearchCV (captures non-linear patterns)
   - K-Nearest Neighbors with GridSearchCV (instance-based learning)

4. **Evaluation Metrics**:
   - RMSE (Root Mean Squared Error) - penalizes large errors
   - MAE (Mean Absolute Error) - average error magnitude
   - R² (variance explained) - proportion of predictable variation
   - Direction Accuracy (% of correct up/down predictions)

### Key Differences

| Aspect | Weekly (Notebook 11) | Monthly (Notebook 12) |
|--------|---------------------|----------------------|
| **Prediction Horizon** | 1 week ahead | 4 weeks ahead |
| **Target Variable** | `price_fluctuation_1w` | `price_fluctuation_1m` |
| **Feature Focus** | Seasonal features (holidays, peak seasons) | Lagged price features + rolling stats |
| **Feature Count** | 23 features | 30 features |
| **Seasonality Analysis** | Extensive holiday impact analysis | Not included |
| **Expected Difficulty** | Easier (shorter horizon) | Harder (longer horizon, more uncertainty) |
| **Primary Use Case** | Operational planning | Strategic planning |

---

## Feature Comparison

### Notebook 11 Features (Weekly - 23 total)

**Seasonal Features (16):**
- Holiday periods: christmas_period, chinese_new_year_period, thanksgiving_period
- Chinese events: golden_week_china, may_day_period, mid_autumn_period
- Shipping seasons: peak_shipping_season, low_shipping_season
- Time features: q1, q2, q3, q4, month_sin, month_cos, week_of_year, month

**Price Features (7):**
- Lags: price_lag_1w, price_lag_2w, price_lag_4w
- Rolling stats: price_lag_1w_roll_mean_4w, price_lag_1w_roll_mean_8w
- Rolling volatility: price_lag_1w_roll_std_4w, price_lag_1w_roll_std_8w

**Feature Importance (Linear Regression):**
1. price_lag_1w (highest coefficient)
2. month_sin (seasonal cycle)
3. week_of_year (calendar effects)
4. q4 (Q4 seasonal effects)

**Best For:** Capturing short-term seasonal impacts and holiday volatility

### Notebook 12 Features (Monthly - 30 total)

**Lagged Price Features:**
- Multiple lags: price_lag_1w through price_lag_8w
- Percentage changes: price_lag_1w_pct_change_1w, price_lag_1w_pct_change_4w
- Momentum indicators: price_lag_1w_pct_change_8w

**Rolling Statistics:**
- Mean windows: price_lag_1w_roll_mean_4w, price_lag_1w_roll_mean_8w
- Volatility: price_lag_1w_roll_std_4w, price_lag_1w_roll_std_8w
- Range: price_lag_1w_roll_min_4w, price_lag_1w_roll_max_4w

**External Factors:**
- Port activity: sh_portcalls_lag_1w, sh_export_lag_1w
- Chokepoint capacity: choke_taiwan_strait_capacity_container_lag_1w
- Crisis indicators: extreme_crisis_lag_1w, egypt_disruption_lag_1w

**Feature Importance (Expected):**
1. price_lag_1w (autocorrelation)
2. price_lag_1w_roll_mean_4w (trend)
3. price_lag_1w_pct_change_4w (momentum)
4. price_lag_1w_roll_std_4w (volatility)

**Best For:** Capturing medium-term trends and cross-contract price dynamics

---

## Key Insights

### Weekly Fluctuation Prediction (Notebook 11)

**Strengths:**
- Higher accuracy due to shorter horizon (RMSE $181.76)
- Captures seasonal patterns (holidays, peak seasons)
- Better direction accuracy (64.94%)
- Useful for immediate operational decisions
- Identifies volatility periods (Christmas +$133 volatility, Peak Season -$93 stability)

**Limitations:**
- Limited value for long-term planning
- May miss contract renewal impacts beyond 1 week
- High accuracy might be due to price stickiness (not model skill)
- R² of 0.293 means 71% of variance unexplained
- Direction accuracy only 15% better than random

**Best Use Cases:**
- Spot market trading decisions
- Short-term capacity planning
- Identifying immediate seasonal impacts
- Crisis response (e.g., Red Sea disruptions)
- Week-ahead operational budgeting

### Monthly Fluctuation Prediction (Notebook 12)

**Strengths:**
- Predicts across contract boundaries
- Better for strategic planning (4-week budget cycles)
- True test of model skill (less reliance on autocorrelation)
- Identifies longer-term trends
- Captures contract renewal dynamics

**Limitations:**
- Lower accuracy (higher uncertainty over 4 weeks)
- Harder to achieve high direction accuracy
- More sensitive to unexpected events
- Expected R² will be lower (more unexplained variance)
- RMSE likely 3-4x higher than weekly

**Best Use Cases:**
- Contract negotiation timing
- Budget forecasting (monthly/quarterly)
- Strategic capacity planning
- Investment decisions
- Identifying when contract renewals occur

---

## Contract Lifecycle Hypothesis

Container freight operates on contracts (typically 2-4 weeks). This creates distinct prediction challenges:

### Week 1-3: Contract Active Period
- **Price behavior:** Sticky, minimal changes
- **Weekly model:** Excellent performance (high autocorrelation)
- **Monthly model:** Decent performance within contract
- **Insight:** High RMSE from weekly model suggests contract is stable

### Week 4: Contract Renewal Week
- **Price behavior:** Jump/drop possible
- **Weekly model:** Struggles (can't predict renewal timing perfectly)
- **Monthly model:** Major challenge (predicting across boundary)
- **Insight:** Errors concentrate around renewal periods

### Week 5-7: New Contract Period
- **Price behavior:** Sticky again at new price level
- **Weekly model:** Excellent performance resumes
- **Monthly model:** Better if new contract price is stable
- **Insight:** Models "reset" after renewal

### Implications

**If degradation is LOW (<100%):**
- Contracts last > 4 weeks
- Prices very sticky
- Monthly predictions viable
- Weekly model advantage is small

**If degradation is HIGH (>200%):**
- Contracts renew frequently (2-3 weeks)
- Price volatility is genuine
- Monthly predictions much harder
- Weekly model provides real value

---

## Model Selection Recommendations

### Use Weekly Predictions (Notebook 11) When:

1. **Immediate operational decisions** needed
   - Daily/weekly logistics planning
   - Spot market contract decisions
   - Short-term capacity allocation

2. **Seasonal planning** is priority
   - Pre-holiday shipping (November-December)
   - Post-holiday lull (January-February)
   - Peak season operations (August-October)

3. **High accuracy** is critical
   - Thin profit margins
   - Large contract volumes
   - Risk-averse operations

4. **Direction matters more than magnitude**
   - Trading signals (buy/sell)
   - Binary decisions (contract now vs wait)
   - Hedging strategies

### Use Monthly Predictions (Notebook 12) When:

1. **Strategic planning** is needed
   - Quarterly budget forecasting
   - Annual capacity planning
   - Long-term contract negotiations

2. **Contract timing** is critical
   - When to negotiate renewals
   - Lock-in rate decisions
   - Multi-month commitments

3. **Trend identification** over accuracy
   - Understanding market direction
   - Identifying structural changes
   - Long-term investment decisions

4. **Cross-contract dynamics** matter
   - Understanding price stability
   - Forecasting renewal impacts
   - Contract cycle analysis

### Use Both When:

1. **Comprehensive risk management**
   - Short-term operations + long-term strategy
   - Hedging across multiple horizons
   - Portfolio approach to contracts

2. **Validating price stickiness**
   - If weekly and monthly perform similarly → high stickiness
   - If large degradation → genuine volatility
   - Informs contract length decisions

3. **Stakeholder communication**
   - Operations team needs weekly
   - Finance team needs monthly
   - Executive team needs both perspectives

4. **Model ensembling**
   - Combine predictions across horizons
   - Weighted average based on confidence
   - Improve overall forecast accuracy

---

## Model Improvement Opportunities

### Short-Term Improvements (1-2 weeks effort)

**1. Regularization for Linear Regression**
```python
from sklearn.linear_model import Ridge, Lasso, ElasticNet

# Ridge (L2): Reduces coefficient magnitude, prevents overfitting
ridge = Ridge(alpha=1.0)

# Lasso (L1): Feature selection, sets weak features to zero
lasso = Lasso(alpha=0.1)

# ElasticNet: Combination of L1 and L2
elastic = ElasticNet(alpha=0.1, l1_ratio=0.5)
```
**Expected Impact:** 5-10% RMSE reduction, better generalization

**2. Ensemble Methods**
```python
from sklearn.ensemble import RandomForestRegressor, GradientBoostingRegressor

# Random Forest: Multiple trees, reduces overfitting
rf = RandomForestRegressor(n_estimators=100, max_depth=10, min_samples_leaf=5)

# Gradient Boosting: Sequential trees, learns from errors
gb = GradientBoostingRegressor(n_estimators=100, learning_rate=0.1, max_depth=5)
```
**Expected Impact:** 10-20% RMSE reduction, captures non-linear patterns

**3. Time Series Cross-Validation**
```python
from sklearn.model_selection import TimeSeriesSplit

tscv = TimeSeriesSplit(n_splits=5)
for train_idx, val_idx in tscv.split(X):
    model.fit(X[train_idx], y[train_idx])
    score = model.score(X[val_idx], y[val_idx])
```
**Expected Impact:** More reliable model selection, prevents overfitting to recent data

**4. Feature Engineering Additions**
- **Lag interactions:** price_lag_1w × crisis_indicator
- **Seasonal lags:** price_lag_52w (year-over-year comparison)
- **Volatility regime indicators:** high_volatility_period (binary flag)
- **Contract renewal indicators:** weeks_since_last_jump (estimated)

**Expected Impact:** 5-15% R² improvement, better crisis handling

### Medium-Term Improvements (1-2 months effort)

**5. External Data Integration**

**Oil Price Volatility:**
```python
# Add Brent crude oil features
df['oil_price_lag_1w'] = oil_data['price'].shift(1)
df['oil_volatility_4w'] = oil_data['price'].rolling(4).std().shift(1)
df['oil_pct_change_1w'] = oil_data['price'].pct_change(1).shift(1)
```
**Expected Impact:** 10-15% R² improvement (oil drives freight costs)

**Exchange Rates:**
```python
# USD/EUR and USD/CNY exchange rates
df['usd_eur_lag_1w'] = forex_data['usd_eur'].shift(1)
df['usd_cny_lag_1w'] = forex_data['usd_cny'].shift(1)
```
**Expected Impact:** 5-10% R² improvement (currency affects contracts)

**Economic Indicators:**
```python
# Manufacturing PMI (Purchasing Managers' Index)
df['china_pmi_lag_1m'] = pmi_data['china_pmi'].shift(4)  # 1 month lag
df['us_pmi_lag_1m'] = pmi_data['us_pmi'].shift(4)
```
**Expected Impact:** 5-10% R² improvement (demand indicator)

**Sentiment Analysis:**
```python
# Shipping news sentiment (from trade publications)
df['sentiment_score_1w'] = sentiment_data['score'].shift(1)
df['crisis_mentions_1w'] = sentiment_data['crisis_count'].shift(1)
```
**Expected Impact:** 3-8% R² improvement (early crisis detection)

**6. Advanced Time Series Models**

**ARIMA/SARIMAX:**
```python
from statsmodels.tsa.statespace.sarimax import SARIMAX

# Seasonal ARIMA with exogenous variables
model = SARIMAX(
    y_train,
    exog=X_train,
    order=(2, 1, 2),           # ARIMA order (p, d, q)
    seasonal_order=(1, 1, 1, 52)  # Seasonal order (P, D, Q, s) - 52 weeks
)
```
**Expected Impact:** 15-25% RMSE reduction for time series, captures seasonality better

**LSTM Neural Networks:**
```python
from tensorflow.keras.models import Sequential
from tensorflow.keras.layers import LSTM, Dense, Dropout

# Long Short-Term Memory network
model = Sequential([
    LSTM(64, return_sequences=True, input_shape=(lookback, n_features)),
    Dropout(0.2),
    LSTM(32, return_sequences=False),
    Dropout(0.2),
    Dense(16, activation='relu'),
    Dense(1)
])
```
**Expected Impact:** 20-30% RMSE reduction if sufficient data (>1000 samples)

**Prophet (Facebook):**
```python
from fbprophet import Prophet

# Handles seasonality, holidays, and trends automatically
model = Prophet(
    yearly_seasonality=True,
    weekly_seasonality=True,
    daily_seasonality=False
)
model.add_country_holidays(country_name='CN')  # Chinese holidays
model.add_country_holidays(country_name='US')  # US holidays
```
**Expected Impact:** 10-20% RMSE reduction, excellent for seasonality

**7. Separate Crisis and Stable Models**

```python
# Detect high volatility periods
df['high_volatility'] = (df['price_roll_std_4w'] > df['price_roll_std_4w'].quantile(0.75))

# Train separate models
model_stable = LinearRegression()
model_crisis = GradientBoostingRegressor()  # More flexible for volatility

# Predict based on regime
if current_volatility > threshold:
    prediction = model_crisis.predict(X)
else:
    prediction = model_stable.predict(X)
```
**Expected Impact:** 15-25% RMSE reduction during crises, 5-10% overall

### Long-Term Improvements (3-6 months effort)

**8. Contract Renewal Detection**

**Binary Classifier for Renewal Weeks:**
```python
from sklearn.ensemble import RandomForestClassifier

# Create target: 1 if |price_change| > threshold, 0 otherwise
df['is_renewal_week'] = (abs(df['price_fluctuation_1w']) > threshold).astype(int)

# Train classifier
renewal_classifier = RandomForestClassifier(n_estimators=100)
renewal_classifier.fit(X_train, df.loc[train_idx, 'is_renewal_week'])

# Two-stage prediction:
# Stage 1: Predict if renewal will occur
# Stage 2: If renewal, predict magnitude; else predict small change
```
**Expected Impact:** 20-35% RMSE reduction, addresses contract lifecycle directly

**9. Collect Contract Metadata**

If contract renewal dates are available:
```python
# Features based on actual contract data
df['days_until_renewal'] = (next_renewal_date - current_date).days
df['contract_age_weeks'] = (current_date - contract_start_date).days / 7
df['contract_type'] = contract_data['type']  # spot vs long-term
df['contract_volume'] = contract_data['volume']  # size effect
```
**Expected Impact:** 30-50% RMSE reduction (addresses root cause of stickiness)

**10. Bayesian Models with Uncertainty Quantification**

```python
import pymc3 as pm

# Bayesian Linear Regression with uncertainty
with pm.Model() as model:
    # Priors
    alpha = pm.Normal('alpha', mu=0, sd=10)
    beta = pm.Normal('beta', mu=0, sd=10, shape=n_features)
    sigma = pm.HalfNormal('sigma', sd=1)

    # Likelihood
    mu = alpha + pm.math.dot(X, beta)
    y_obs = pm.Normal('y_obs', mu=mu, sd=sigma, observed=y)

    # Inference
    trace = pm.sample(2000, return_inferencedata=False)

# Get prediction intervals
predictions = pm.sample_posterior_predictive(trace, model=model)
prediction_mean = predictions['y_obs'].mean(axis=0)
prediction_lower = np.percentile(predictions['y_obs'], 5, axis=0)  # 5th percentile
prediction_upper = np.percentile(predictions['y_obs'], 95, axis=0)  # 95th percentile
```
**Expected Impact:** Better risk quantification, confidence intervals, decision-making under uncertainty

**11. Multi-Horizon Ensemble**

```python
# Train models for multiple horizons
model_1w = train_model(target='price_fluctuation_1w')
model_2w = train_model(target='price_fluctuation_2w')
model_3w = train_model(target='price_fluctuation_3w')
model_4w = train_model(target='price_fluctuation_4w')

# Combine predictions with learned weights
ensemble_pred = (
    w1 * model_1w.predict(X) +
    w2 * model_2w.predict(X) +
    w3 * model_3w.predict(X) +
    w4 * model_4w.predict(X)
)
# where weights w1, w2, w3, w4 are optimized via cross-validation
```
**Expected Impact:** 10-20% RMSE reduction, smooths out single-horizon noise

---

## Evaluation Improvements

### Current Limitations

1. **Single test set:** Only one 20% holdout, may not be representative
2. **No walk-forward testing:** Doesn't simulate real deployment
3. **Limited crisis coverage:** Test set may not include major crises
4. **No cost-benefit analysis:** RMSE doesn't translate to business value

### Recommended Enhancements

**1. Walk-Forward Validation (Production Simulation)**

```python
# Simulate real-world deployment
initial_train_size = 200  # First 200 weeks for training
results = []

for i in range(initial_train_size, len(df)):
    # Train on all data up to current week
    X_train = X.iloc[:i]
    y_train = y.iloc[:i]

    # Predict next week
    model.fit(X_train, y_train)
    prediction = model.predict(X.iloc[i:i+1])

    # Record actual vs predicted
    results.append({
        'date': df.index[i],
        'actual': y.iloc[i],
        'predicted': prediction[0]
    })
```
**Benefit:** Realistic performance estimate, catches model drift

**2. Stratified Evaluation (by Volatility Regime)**

```python
# Separate metrics by market condition
low_vol_mask = (df['price_roll_std_4w'] < df['price_roll_std_4w'].quantile(0.33))
med_vol_mask = (df['price_roll_std_4w'] >= df['price_roll_std_4w'].quantile(0.33)) & \
               (df['price_roll_std_4w'] < df['price_roll_std_4w'].quantile(0.67))
high_vol_mask = (df['price_roll_std_4w'] >= df['price_roll_std_4w'].quantile(0.67))

# Calculate RMSE for each regime
rmse_low = np.sqrt(mean_squared_error(y_test[low_vol_mask], pred[low_vol_mask]))
rmse_med = np.sqrt(mean_squared_error(y_test[med_vol_mask], pred[med_vol_mask]))
rmse_high = np.sqrt(mean_squared_error(y_test[high_vol_mask], pred[high_vol_mask]))
```
**Benefit:** Understand when model works well vs struggles

**3. Economic Value Metrics**

```python
# Profit simulation (assuming trading strategy)
def calculate_profit(actual, predicted, contract_size=100):
    """
    Simulate profit from trading based on predictions.
    Buy if predicted increase > threshold, sell if predicted decrease > threshold.
    """
    threshold = 50  # $50 minimum prediction to act
    profit = 0

    for i in range(len(actual)):
        if predicted[i] > threshold:  # Predict price increase
            # Buy contract, profit if actual increases
            profit += (actual[i] - threshold) * contract_size if actual[i] > 0 else 0
        elif predicted[i] < -threshold:  # Predict price decrease
            # Short contract, profit if actual decreases
            profit += (threshold - actual[i]) * contract_size if actual[i] < 0 else 0

    return profit

# Compare to baseline (always hold / random)
model_profit = calculate_profit(y_test, predictions)
baseline_profit = calculate_profit(y_test, np.zeros_like(predictions))  # Always hold

print(f"Model profit: ${model_profit:,.2f}")
print(f"Baseline profit: ${baseline_profit:,.2f}")
print(f"Value added: ${model_profit - baseline_profit:,.2f}")
```
**Benefit:** Translates accuracy to business value, justifies model investment

**4. Calibration Analysis**

```python
# Check if confidence matches actual accuracy
from sklearn.calibration import calibration_curve

# Convert predictions to probabilities (price will increase)
prob_increase = (predictions > 0).astype(float)
actual_increase = (y_test > 0).astype(float)

# Plot calibration curve
fraction_of_positives, mean_predicted_value = calibration_curve(
    actual_increase,
    prob_increase,
    n_bins=10
)

# Perfect calibration: predicted probability = actual frequency
```
**Benefit:** Ensures confidence intervals are reliable for risk management

---

## Next Steps and Roadmap

### Immediate Actions (This Week)

1. **Run Notebook 12** to populate monthly prediction results
   ```bash
   jupyter notebook 12_monthly_price_fluctuation_models.ipynb
   ```

2. **Calculate Performance Degradation**
   ```python
   weekly_rmse = 181.76
   monthly_rmse = [from notebook 12]
   degradation = ((monthly_rmse - weekly_rmse) / weekly_rmse) * 100
   ```

3. **Update This Document** with actual monthly results

4. **Document Findings** in project summary
   - If degradation > 200%: Market is highly volatile
   - If degradation 100-200%: Standard freight dynamics
   - If degradation < 100%: High price stickiness, investigate contract lengths

### Week 2-3: Model Improvements

1. **Implement Regularization**
   - Try Ridge, Lasso, ElasticNet
   - Compare RMSE improvements
   - Update model comparison guide

2. **Add Ensemble Methods**
   - Random Forest
   - Gradient Boosting
   - Document new best performers

3. **Walk-Forward Validation**
   - Implement rolling window evaluation
   - Generate time-series of prediction errors
   - Identify model drift periods

### Week 4-6: Feature Engineering

1. **Add External Data**
   - Oil prices (Brent crude)
   - Exchange rates (USD/EUR, USD/CNY)
   - Manufacturing PMI (China, US, Europe)

2. **Create Interaction Features**
   - price_lag × crisis_indicator
   - seasonality × price_volatility
   - contract_age × price_change

3. **Year-over-Year Comparisons**
   - price_lag_52w (same week last year)
   - yoy_change (year-over-year growth)

### Month 2-3: Advanced Models

1. **Implement ARIMA/SARIMAX**
   - Captures time series structure
   - Handles seasonality natively
   - Compare to Linear Regression

2. **Explore LSTM Neural Networks**
   - If dataset grows to >1000 samples
   - Sequence modeling for trends
   - Requires GPU for training

3. **Try Facebook Prophet**
   - Excellent for seasonality
   - Handles holidays automatically
   - Quick to implement and interpret

### Month 4-6: Production Deployment

1. **Build Contract Renewal Classifier**
   - Binary model: will price jump this week?
   - Two-stage prediction system
   - Addresses root cause of stickiness

2. **Collect Contract Metadata**
   - Work with stakeholders to gather renewal dates
   - Contract types, volumes, durations
   - Integrate into feature set

3. **Create API Endpoint**
   ```python
   from flask import Flask, request, jsonify

   app = Flask(__name__)

   @app.route('/predict', methods=['POST'])
   def predict():
       data = request.json
       features = extract_features(data)
       prediction = model.predict(features)
       confidence = calculate_confidence(features)

       return jsonify({
           'prediction': float(prediction),
           'confidence_lower': float(prediction - confidence),
           'confidence_upper': float(prediction + confidence),
           'direction': 'increase' if prediction > 0 else 'decrease'
       })
   ```

4. **Monitor Model Performance**
   - Track RMSE over time
   - Alert if degradation > 20%
   - Retrain quarterly

### Ongoing: Research and Optimization

1. **Literature Review**
   - Academic papers on freight forecasting
   - Industry best practices
   - New ML techniques (e.g., Transformers)

2. **Benchmark Against Industry**
   - Compare to Freightos, Xeneta predictions
   - Assess relative performance
   - Identify gaps

3. **Stakeholder Feedback**
   - Gather user feedback on predictions
   - Understand decision-making context
   - Refine based on actual use cases

4. **A/B Testing**
   - Deploy multiple models in parallel
   - Compare business outcomes
   - Optimize for actual value, not just RMSE

---

## Conclusion

This comparison framework provides a comprehensive approach to understanding short-term vs long-term freight price forecasting. The key insights are:

**From Weekly Predictions (Notebook 11):**
- Linear Regression achieves RMSE of $181.76 with 64.94% direction accuracy
- Seasonal features significantly improve performance (R² = 0.293)
- Christmas period adds $133 volatility; peak shipping season reduces volatility by $93

**Expected from Monthly Predictions (Notebook 12):**
- RMSE likely 3-4x higher due to longer horizon
- R² will be lower (more unexplained variance)
- Direction accuracy will decrease (harder to predict 4 weeks out)
- Performance degradation reveals contract dynamics

**Strategic Implications:**
- Use weekly predictions for operational decisions (spot market, short-term planning)
- Use monthly predictions for strategic decisions (contracts, budgets, investments)
- Compare degradation to understand if prices are sticky or volatile
- Combine both horizons for comprehensive risk management

**Improvement Priority Order:**
1. Run Notebook 12 and calculate actual degradation
2. Add ensemble methods (Random Forest, Gradient Boosting) - quick wins
3. Implement walk-forward validation - realistic performance
4. Add external data (oil, exchange rates) - significant R² boost
5. Build contract renewal classifier - addresses root cause
6. Collect actual contract metadata - transformative improvement

The ultimate goal is to build a two-stage system:
1. **Stage 1:** Predict if contract renewal will occur (binary classifier)
2. **Stage 2:** Predict magnitude of change if renewal occurs (regression)

This approach directly addresses the contract lifecycle hypothesis and should provide the most actionable insights for stakeholders.

---

**Document Version:** 1.1
**Last Updated:** 2025-11-18
**Status:** Awaiting Notebook 12 results
**Next Review:** After running Notebook 12

**Related Files:**
- `NOTEBOOK_11_RESULTS_EXPLANATION.md` - Weekly fluctuation details
- `MODEL_COMPARISON_GUIDE.md` - All model comparisons
- `MODEL_INSIGHTS_SUMMARY.md` - High-level project insights
- `11_seasonality_price_fluctuation_models.ipynb` - Weekly predictions
- `12_monthly_price_fluctuation_models.ipynb` - Monthly predictions (run this next)
