# Weekly vs Monthly Price Fluctuation Prediction Comparison

## Overview

This document compares two different prediction horizons for container freight price fluctuations:
- **Weekly Fluctuation Prediction** (Notebook 11): Predicts price change 1 week ahead
- **Monthly Fluctuation Prediction** (Notebook 12): Predicts price change 4 weeks ahead

Both notebooks use the same three machine learning models (Linear Regression, Decision Tree, K-Nearest Neighbors) to predict **price fluctuations** (change in price) rather than absolute prices.

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
   - RMSE (Root Mean Squared Error)
   - MAE (Mean Absolute Error)
   - R² (variance explained)
   - Direction Accuracy (% of correct up/down predictions)

### Key Differences

| Aspect | Weekly (Notebook 11) | Monthly (Notebook 12) |
|--------|---------------------|----------------------|
| **Prediction Horizon** | 1 week ahead | 4 weeks ahead |
| **Target Variable** | `price_fluctuation_1w` | `price_fluctuation_1m` |
| **Feature Focus** | Seasonal features (holidays, peak seasons) | Lagged price features + rolling stats |
| **Seasonality Analysis** | ✅ Extensive holiday impact analysis | ❌ Not included |
| **Expected Difficulty** | Easier (shorter horizon) | Harder (longer horizon, more uncertainty) |

---

## Expected Results

### Performance Degradation

When comparing weekly to monthly predictions, we expect to see:

1. **Increased RMSE**: Longer prediction horizons introduce more uncertainty
2. **Lower R²**: More unexplained variance over 4 weeks vs 1 week
3. **Lower Direction Accuracy**: Harder to predict direction of change
4. **Performance Degradation Formula**:
   ```
   Degradation % = ((Monthly_RMSE - Weekly_RMSE) / Weekly_RMSE) × 100
   ```

### Interpretation Guidelines

| Degradation % | Interpretation |
|---------------|----------------|
| **> 100%** | SIGNIFICANT - Monthly predictions much harder, indicates price volatility |
| **50-100%** | MODERATE - Expected degradation for longer horizon |
| **20-50%** | SMALL - Suggests high price autocorrelation/stickiness |
| **< 20%** | MINIMAL - Strong evidence of price stickiness over 4 weeks |

---

## Why Compare Weekly vs Monthly?

### 1. Understanding Price Stability

The degradation level tells us about freight contract dynamics:

- **High degradation (>100%)**: Prices change significantly week-to-week, contracts are volatile
- **Low degradation (<50%)**: Prices are "sticky" due to contract persistence, stay stable for weeks

### 2. Optimal Planning Horizon

Different stakeholders need different prediction horizons:

- **Weekly predictions**:
  - Better for short-term operational planning
  - Useful for spot market decisions
  - Higher accuracy for immediate actions

- **Monthly predictions**:
  - Better for contract negotiation timing
  - Useful for budget planning
  - Captures contract renewal cycles

### 3. Contract Lifecycle Hypothesis

Container freight operates on contracts (typically 2-4 weeks):

```
Week 1-3: Contract active → Prices sticky → Weekly predictions excel
Week 4:   Contract renews → Price jumps → Both models struggle
Week 5-7: New contract    → Prices sticky → Weekly predictions excel again
```

**Weekly models** predict within contract periods (easier)
**Monthly models** predict across contract boundaries (harder)

---

## Key Insights

### Weekly Fluctuation Prediction (Notebook 11)

**Strengths**:
- ✅ Higher accuracy due to shorter horizon
- ✅ Captures seasonal patterns (holidays, peak seasons)
- ✅ Better direction accuracy
- ✅ Useful for immediate operational decisions

**Limitations**:
- ⚠️ Limited value for long-term planning
- ⚠️ May miss contract renewal impacts
- ⚠️ High accuracy might be due to price stickiness (not model skill)

**Best Use Cases**:
- Spot market trading decisions
- Short-term capacity planning
- Identifying immediate seasonal impacts
- Crisis response (e.g., Red Sea disruptions)

### Monthly Fluctuation Prediction (Notebook 12)

**Strengths**:
- ✅ Predicts across contract boundaries
- ✅ Better for strategic planning
- ✅ True test of model skill (less reliance on autocorrelation)
- ✅ Identifies longer-term trends

**Limitations**:
- ⚠️ Lower accuracy (higher uncertainty)
- ⚠️ Harder to achieve high direction accuracy
- ⚠️ More sensitive to unexpected events

**Best Use Cases**:
- Contract negotiation timing
- Budget forecasting
- Strategic capacity planning
- Investment decisions

---

## Decision Framework: Which Model to Use?

### Use Weekly Predictions When:
1. You need to make **immediate operational decisions**
2. You're trading in the **spot market**
3. You need to respond to **current crises or events**
4. You want to exploit **seasonal patterns** (holidays, peak seasons)
5. High accuracy is critical for your decision

### Use Monthly Predictions When:
1. You're planning **contract negotiations** (2-4 weeks out)
2. You need **budget forecasts** for next month
3. You're making **strategic investments**
4. You want to understand **trend changes** beyond short-term noise
5. You need to predict when **contract renewals** will change prices

### Use Both When:
1. You want a **comprehensive view** of short and long-term dynamics
2. You're building a **risk management framework**
3. You need to validate if **price stickiness** exists
4. You're presenting to stakeholders with **different planning needs**

---

## Technical Comparison

### Feature Importance

**Weekly Models** (Notebook 11) emphasize:
- Seasonal indicators (Chinese New Year, Christmas, Golden Week)
- Month/quarter effects
- Peak/low shipping seasons
- Cyclical patterns (month_sin, month_cos)

**Monthly Models** (Notebook 12) emphasize:
- Lagged price features (price_lag_1w, price_lag_2w, price_lag_4w)
- Rolling statistics (roll_mean, roll_std, roll_min, roll_max)
- Percentage changes over different windows
- Crisis indicators (if volatility expected)

### Model Selection

Both notebooks compare the same three models, but the **best model** may differ:

| Model | Weekly Performance | Monthly Performance |
|-------|-------------------|---------------------|
| **Linear Regression** | Good baseline, interpretable | Often best for monthly (less prone to overfitting) |
| **Decision Tree** | Can capture seasonal patterns well | May overfit on longer horizon |
| **KNN** | Works if similar patterns exist | Struggles with longer horizon (less similar instances) |

---

## Performance Metrics to Track

### 1. RMSE (Root Mean Squared Error)
- **Lower is better**
- Heavily penalizes large errors
- Compare weekly vs monthly to understand degradation

### 2. MAE (Mean Absolute Error)
- **Lower is better**
- Average magnitude of errors
- More robust to outliers than RMSE

### 3. Direction Accuracy
- **Higher is better** (>50% beats random guessing)
- Critical for trading decisions
- Weekly should have higher direction accuracy

### 4. R² Score
- **Higher is better** (max 1.0)
- Percentage of variance explained
- Monthly will have lower R² due to increased uncertainty

---

## Recommendations

### For Analysts
1. **Always run both notebooks** to understand short vs long-term dynamics
2. **Calculate degradation %** to assess price stickiness
3. **Track direction accuracy separately** - it's often more valuable than RMSE
4. **Document which features** drive each model (seasonal vs price-based)

### For Stakeholders
1. **Weekly predictions** for operational teams (logistics, spot trading)
2. **Monthly predictions** for finance teams (budgeting, contracts)
3. **Present both** when discussing strategic planning
4. **Emphasize direction accuracy** over absolute error for decision-making

### For Model Improvements
1. **Ensemble approach**: Combine weekly and monthly predictions
2. **Contract feature**: If available, add explicit contract renewal dates
3. **Binary classifier**: Predict "will price change significantly?" separately
4. **Separate crisis model**: Train specific model for crisis periods (higher volatility)

---

## Common Questions

### Q1: Why is monthly RMSE so much higher?
**A**: Longer prediction horizons have inherent uncertainty. Prices can change multiple times over 4 weeks due to:
- New events occurring
- Contract renewals
- Market sentiment shifts
- Geopolitical developments

This is expected and doesn't indicate model failure.

### Q2: Should I always use weekly predictions since they're more accurate?
**A**: No. Use the prediction horizon that matches your decision timeframe:
- Buying spot contracts next week? → Weekly
- Negotiating monthly contract? → Monthly
- Both are valuable for different purposes

### Q3: What if degradation is very low (<30%)?
**A**: This suggests high **price stickiness** - prices stay stable for weeks. This means:
- Freight contracts likely last 3-4 weeks
- Your "model skill" may actually be autocorrelation
- Focus on predicting **when contracts renew** (price change events)
- Direction accuracy becomes more important than RMSE

### Q4: Which notebook should I run first?
**A**: Run **Notebook 11 (weekly)** first, then **Notebook 12 (monthly)**. The monthly notebook includes comparison code that loads weekly results.

### Q5: Can I use these models in production?
**A**: Consider:
- ✅ Weekly models: Yes, for short-term operational decisions
- ⚠️ Monthly models: Use with caution, add confidence intervals
- ✅ Direction predictions: Often more reliable than absolute values
- ⚠️ Crisis periods: Both models may underperform during unprecedented events

---

## File Outputs

### Notebook 11 (Weekly) Generates:
- `data/processed/weekly_fluctuation_model_comparison.csv` - Model performance metrics
- `data/processed/weekly_fluctuation_predictions.csv` - Test set predictions
- `data/processed/seasonality_impact_analysis.csv` - Holiday impact analysis
- `data/processed/seasonal_feature_importance.csv` - Feature importance rankings
- `data/processed/seasonality_model_comparison.csv` - Seasonal model comparison

### Notebook 12 (Monthly) Generates:
- `data/processed/monthly_fluctuation_model_comparison.csv` - Model performance metrics
- `data/processed/monthly_fluctuation_predictions.csv` - Test set predictions
- `data/processed/weekly_vs_monthly_comparison.png` - Visual comparison plot

---

## Conclusion

Both weekly and monthly fluctuation predictions serve different but complementary purposes:

- **Weekly predictions (Notebook 11)** excel at short-term forecasting with seasonal context
- **Monthly predictions (Notebook 12)** provide strategic insights for contract planning

**The degradation between them reveals crucial insights about freight contract dynamics and price stability.**

For a complete forecasting system, use both models together to inform decisions across different time horizons.

---

## Related Documentation

- `NOTEBOOK_11_RESULTS_EXPLANATION.md` - Detailed explanation of weekly fluctuation results
- `MODEL_COMPARISON_GUIDE.md` - General guide to all models in the project
- `MODEL_INSIGHTS_SUMMARY.md` - High-level insights across all notebooks
- Notebook 10 has been deprecated (redundant with Notebook 11)
