# Price Fluctuation Prediction Models - Insights Summary

**Date:** 2025-11-18
**Analysis of:** Weekly and Seasonality-Based Price Fluctuation Prediction Models

---

## Executive Summary

This document summarizes the key findings from running two comprehensive price fluctuation prediction notebooks. The analysis reveals that **seasonal features significantly improve model performance** and identifies specific holidays and seasons that have the greatest impact on container freight price fluctuations.

---

## Model Performance Comparison

### Notebook 1: Weekly Price Fluctuation Models (Feature-Based)
Uses top 30 lagged features based on correlation with price fluctuations.

| Model | RMSE | MAE | R² | Direction Accuracy |
|-------|------|-----|----|--------------------|
| **Linear Regression** | **$196.16** | **$140.83** | **0.176** | **64.94%** |
| Decision Tree | $339.87 | $251.45 | -1.474 | 58.44% |
| K-Nearest Neighbors | $210.39 | $151.01 | 0.052 | 63.64% |

**Top Features:**
1. `price_lag_1w_pct_change_1w` (correlation: 0.4197)
2. `price_lag_1w_pct_change_4w` (correlation: 0.3528)
3. `sh_import_general_cargo_lag_1w_lag_1w` (correlation: 0.2320)

### Notebook 2: Seasonality-Based Price Fluctuation Models
Uses 16 seasonal features + 7 key lagged price features (23 total).

| Model | RMSE | MAE | R² | Direction Accuracy |
|-------|------|-----|----|--------------------|
| **Linear Regression** | **$181.76** | **$136.47** | **0.293** | **64.94%** |
| Decision Tree | $224.83 | $164.84 | -0.083 | 61.04% |
| K-Nearest Neighbors | $186.90 | $138.05 | 0.252 | 63.64% |

**Top Seasonal Features (by model importance):**
1. `week_of_year` (avg importance: 0.1293)
2. `month_cos` (avg importance: 0.0697)
3. `month_sin` (avg importance: 0.0394)
4. `q4` (avg importance: 0.0150)
5. `christmas_period` (avg importance: 0.0040)

---

## Key Findings

### 1. Seasonal Features Improve Performance ✓

**Improvement with seasonality:**
- RMSE improved by **$14.40** (7.3% reduction)
- R² improved from 0.176 to **0.293** (66% increase in explained variance)
- MAE improved by **$4.36**

**Conclusion:** Adding seasonal features to the model provides significant predictive power beyond just lagged price features.

### 2. Most Impactful Seasonal Events

Ranked by absolute impact on price fluctuations:

| Rank | Season/Holiday | Average Impact | Direction | Occurrences |
|------|---------------|----------------|-----------|-------------|
| 1 | **Christmas Period** | **+$133.41** | INCREASE | 28 |
| 2 | **Peak Shipping Season (Aug-Oct)** | **-$92.87** | DECREASE | 89 |
| 3 | **Thanksgiving** | **+$92.05** | INCREASE | 16 |
| 4 | **May Day** | **+$88.70** | INCREASE | 15 |
| 5 | **Mid-Autumn Festival** | **-$83.71** | DECREASE | 16 |
| 6 | **Chinese New Year** | **-$77.01** | DECREASE | 33 |
| 7 | **Low Shipping Season (Jan-Feb)** | **-$72.07** | DECREASE | 59 |
| 8 | **Golden Week (China)** | **-$45.82** | DECREASE | 10 |

#### Interpretation:

**Price Fluctuation INCREASES during:**
- **Christmas Period** (largest impact): +$133 average fluctuation - pre-holiday shipping rush creates volatility
- **Thanksgiving**: +$92 - another pre-holiday demand spike
- **May Day**: +$89 - manufacturing/shipping disruptions in China

**Price Fluctuation DECREASES (more stable) during:**
- **Peak Shipping Season (Aug-Oct)**: -$93 - counter-intuitive but high volume creates stability
- **Mid-Autumn Festival**: -$84 - post-summer stabilization
- **Chinese New Year**: -$77 - predictable shutdown reduces uncertainty
- **Low Shipping Season (Jan-Feb)**: -$72 - post-holiday lull creates stability

### 3. Model Performance Insights

**Linear Regression:**
- Most consistent performer across both notebooks
- Benefits significantly from seasonal features (R² improved 66%)
- Best for interpretability and production deployment

**Decision Tree:**
- Consistently overfits (negative R² in both notebooks)
- Poor generalization despite GridSearchCV tuning
- **Not recommended** for this problem

**K-Nearest Neighbors:**
- Moderate performance (middle ground)
- Slightly worse than Linear Regression with seasonal features
- More computationally expensive, less interpretable

### 4. Direction Prediction Accuracy

All models achieve ~65% direction accuracy (up vs down prediction).

**Confusion Matrix (Best Model - Linear Regression with Seasonality):**
```
               Predicted Down | Predicted Up
Actual Down:        31        |      18
Actual Up:           9        |      19
```

**Analysis:**
- Better at predicting downward movements (31/49 = 63% accuracy)
- Moderate at predicting upward movements (19/28 = 68% accuracy)
- Still only 15 percentage points better than random (50%)

### 5. Limitation: Low R² Overall

Even the best model explains only **29.3%** of variance in price fluctuations.

**Possible reasons:**
- Price fluctuations may be largely driven by unpredictable external shocks
- Missing important features (oil prices, geopolitical events beyond our data)
- Inherent randomness/noise in weekly fluctuations
- Need for more sophisticated models (LSTM, ensemble methods)

---

## Recommendations

### For Production Deployment:

1. **Use Linear Regression with Seasonal Features**
   - Best RMSE ($181.76) and R² (0.293)
   - Most interpretable
   - Stable performance

2. **Focus on Direction Prediction Rather Than Exact Values**
   - 65% direction accuracy is modest but actionable
   - Could be improved with classification-specific models (logistic regression, etc.)

3. **Seasonal Context is Critical**
   - Always include seasonal indicators in production
   - Pay special attention to Christmas period (highest volatility)
   - Consider pre-holiday shipping rush windows

### For Model Improvement:

1. **Add More Advanced Models:**
   - Random Forest (ensemble of trees, less prone to overfitting)
   - Ridge/Lasso Regression (regularization)
   - Gradient Boosting (XGBoost, LightGBM)
   - LSTM/GRU for time series patterns

2. **Feature Engineering:**
   - Momentum indicators (rate of change of change)
   - Volatility measures (rolling std of fluctuations)
   - External factors (oil prices, exchange rates, COVID-like events)
   - Interaction terms between seasonal and price features

3. **Ensemble Approach:**
   - Combine Linear Regression + KNN predictions
   - Use stacking or voting ensemble

4. **Classification Instead of Regression:**
   - Convert to binary classification (up/down)
   - Use logistic regression, SVM, or gradient boosting for classification
   - May achieve better direction accuracy

5. **Cross-Validation:**
   - Implement time series cross-validation
   - Better assess model stability over different periods

---

## Next Steps

### Immediate Actions:
1. ✅ Create combined notebook with best features from both approaches
2. ✅ Add Random Forest and Ridge/Lasso models
3. ✅ Implement proper cross-validation
4. ✅ Add detailed error analysis by seasonal period

### Research Questions:
1. Can we predict "high volatility" periods instead of exact fluctuations?
2. Would a classification model (up/down/stable) perform better?
3. What external data sources could improve R²?
4. Are there specific geopolitical events that correlate with high fluctuations?

---

## Conclusion

The analysis demonstrates that:
- **Seasonal patterns matter** - they improve predictive performance significantly
- **Christmas period causes the highest volatility** (+$133 fluctuation)
- **Peak shipping season is surprisingly stable** (-$93 fluctuation)
- **Linear Regression with seasonal features is the current best model**
- **There's room for improvement** - current R² of 0.29 suggests we're missing important factors

The notebooks provide a solid foundation for price fluctuation forecasting, but the modest R² indicates that price movements are influenced by many factors beyond our current feature set. Future work should focus on incorporating external economic indicators, geopolitical event data, and more sophisticated modeling approaches.

---

**Generated by:** Price Fluctuation Analysis Pipeline
**Data Period:** 2018-01-05 to 2025-08-15
**Test Period:** 2024-02-02 to 2025-08-15 (77 samples)
