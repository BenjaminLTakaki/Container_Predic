# Advanced Feature Engineering Guide
## Container Price Prediction Project

---

## 🎯 Current Situation

**Problem:** After fixing data leakage, your Naive baseline (RMSE: $446.85) is still beating advanced models.

**Root Cause:** Your current 1-week lagged features don't capture enough predictive signal because:
- Container shipping operates on **multi-week cycles** (2-8 weeks)
- Price changes have **momentum** and **volatility patterns** that simple lags miss
- **Relationships between variables** (oil prices, port activity, disruptions) aren't captured

---

## 🚀 Solution: 5 Advanced Feature Engineering Strategies

### **Strategy A: Extended Lag Periods (2-8 weeks)**
**Why:** Shipping routes from Shanghai to Europe take 4-6 weeks. Events don't impact prices immediately.

**What we created:**
- `Europe_Base_Price_lag_2w` through `Europe_Base_Price_lag_8w`
- `WTI_Price_lag_2w` through `WTI_Price_lag_8w`
- `Brent_Price_lag_2w` through `Brent_Price_lag_8w`
- Extended lags for `sh_portcalls_container` and `composite_disruption_index`

**Expected impact:** Capture medium-term trends and cyclical patterns.

---

### **Strategy B: Rolling Window Statistics (4-week & 8-week)**
**Why:** Captures trends and volatility that single-point lags miss.

**What we created for each key feature:**
- `_roll_mean_4w` / `_roll_mean_8w` - Trend indicators
- `_roll_std_4w` / `_roll_std_8w` - Volatility indicators
- `_roll_min_4w` / `_roll_max_4w` - Range indicators

**Expected impact:** Models can detect when markets are trending up/down or becoming more volatile.

---

### **Strategy C: Momentum / Rate of Change Features**
**Why:** Price momentum is a powerful predictor - if prices are rising, they tend to keep rising.

**What we created:**
- `_pct_change_1w` - Week-over-week percent change
- `_pct_change_4w` - 4-week percent change (medium-term momentum)
- `_acceleration` - Rate of change of rate of change

**Expected impact:** Capture directional trends and turning points.

---

### **Strategy D: Interaction Features**
**Why:** Relationships between variables matter (e.g., when oil prices spike relative to shipping costs).

**What we created:**
- `oil_spread_brent_wti` - Difference between Brent and WTI prices
- `oil_spread_pct` - Oil spread as percentage
- `price_to_brent_ratio` - Shipping cost relative to fuel cost
- `price_per_portcall` - Price normalized by port activity

**Expected impact:** Capture complex market dynamics that single features miss.

---

### **Strategy E: Volatility Indicators**
**Why:** High volatility often precedes price changes (market uncertainty).

**What we created:**
- `Europe_Base_Price_cv_4w` - Coefficient of variation (4-week)
- `Europe_Base_Price_cv_8w` - Coefficient of variation (8-week)

**Expected impact:** Detect regime changes (stable → crisis markets).

---

## ✅ Data Leakage Prevention

**Critical:** ALL new features are automatically lagged by 1 week (using `.shift(1)`).

This ensures that when predicting **next week's price**, we only use **this week's or earlier** data.

**Features lagged:**
- All rolling statistics → `_lag_1w` versions created
- All momentum features → `_lag_1w` and `_lag_2w` versions created
- All interaction features → `_lag_1w` versions created
- All volatility indicators → `_lag_1w` versions created

---

## 📋 Next Steps

### **Step 1: Re-run `02_data_understanding.ipynb`**

1. Open `02_data_understanding.ipynb`
2. Click **"Run All"** or run cells sequentially
3. The updated cell will create ~100+ new predictive features
4. Verify that `data/processed/model_data.csv` is updated

**Expected output:**
```
✓ Created approximately 100+ new predictive features
✓ All features are properly lagged to prevent data leakage
✓ Saved final modeling dataset to 'data/processed/model_data.csv'
```

---

### **Step 2: Re-run ENTIRE `03_forecasting_pipeline.ipynb`**

**CRITICAL:** You must re-run the **entire pipeline**, including hyperparameter tuning.

**Why?** Your current `XGBoost (Tuned)` model was tuned on the **leaky data**. Those hyperparameters are now invalid.

**What to do:**
1. Open `03_forecasting_pipeline.ipynb`
2. Click **"Run All"** (this will take 5-10 minutes)
3. The pipeline will:
   - Load the new `model_data.csv` with 100+ features
   - Train baseline models (Naive, Moving Average)
   - Train linear models (Linear Regression, Ridge, Lasso)
   - Train tree models (Random Forest, Gradient Boosting, XGBoost)
   - **Perform GridSearchCV** to find new optimal hyperparameters
   - Evaluate all models on the test set
   - Save the best model

**Expected outcome:** 
- Advanced models (XGBoost, Random Forest) should now **beat the naive baseline**
- RMSE should drop from **$446.85** to ideally **$350-400** or lower
- You'll see which new features are most important

---

### **Step 3: Interpret Results**

After re-running the pipeline, check:

1. **Model Comparison Table** - Is XGBoost now the best model?
2. **Feature Importance Plot** - Which new features are most predictive?
   - Look for: `price_lag_4w`, `Europe_Base_Price_roll_mean_4w`, `price_pct_change_4w`
3. **Prediction Plot** - Do predictions track actual prices better?

---

## 🎓 Expected Feature Importance Rankings

Based on time-series forecasting best practices, expect to see:

**Top 10 Most Important Features (likely):**
1. `price_lag_1w` - Last week's price (strongest predictor)
2. `price_lag_2w` - 2 weeks ago price
3. `Europe_Base_Price_roll_mean_4w_lag_1w` - 4-week price trend
4. `Europe_Base_Price_pct_change_1w_lag_1w` - Price momentum
5. `price_lag_4w` - 4 weeks ago price (shipping cycle)
6. `Brent_Price_pct_change_4w_lag_1w` - Oil price momentum
7. `Europe_Base_Price_roll_std_4w_lag_1w` - Price volatility
8. `price_to_brent_ratio_lag_1w` - Shipping-to-fuel cost ratio
9. `sh_portcalls_container_lag_1w` - Shanghai port activity
10. `composite_disruption_index_lag_1w` - Geopolitical events

---

## 🔍 Troubleshooting

### **If models still don't beat naive baseline:**

**Possible causes:**
1. **Test set too small** - With only ~50 test samples, small RMSE differences might not be significant
2. **COVID spike dominates** - The 2020-2021 price spike might be too extreme for models to learn from
3. **Need feature selection** - Too many features (curse of dimensionality)

**Solutions to try:**
1. Use **walk-forward validation** instead of single train/test split
2. Train **separate models** for "normal" vs "crisis" periods
3. Use **Recursive Feature Elimination (RFE)** to select top 20-30 features
4. Try **ensemble methods** (stacking XGBoost + Random Forest)

---

### **If you get errors:**

**Error:** `KeyError: 'composite_disruption_index'`
- **Fix:** Re-run `01_data_collection.ipynb` to ensure GDELT data is loaded

**Error:** `Too many NaN values after lagging`
- **Fix:** This is expected - rolling windows need warm-up periods. You'll lose ~8 weeks at the start of the dataset.

**Error:** `GridSearchCV taking too long`
- **Fix:** Reduce `param_grid` size in the hyperparameter tuning cell (fewer combinations)

---

## 📊 Success Criteria

Your feature engineering is successful if:

✅ **XGBoost (Tuned) RMSE < $400** (below naive baseline of $446.85)  
✅ **R² > 0.70** (models explain >70% of variance)  
✅ **Feature importance shows diverse features** (not just price lags)  
✅ **Predictions track major events** (COVID spike, Suez blockage, Red Sea crisis)

---

## 🚨 Confirmation: Your Next Step

**YES**, you are correct:

> "My next step (after adding new features) should be to re-run the **ENTIRE** `03_forecasting_pipeline.ipynb` notebook, including the hyperparameter tuning (GridSearchCV) step, to find the **new** best parameters."

**Why?** 
- Old hyperparameters were tuned on **leaky data** (not valid anymore)
- New features change the **feature space** (need new optimal parameters)
- Tree depth, learning rate, etc. all depend on **what features are available**

---

## 📝 Summary

1. ✅ **Code added** to `02_data_understanding.ipynb` (cell with advanced feature engineering)
2. ⏳ **Run `02_data_understanding.ipynb`** → Creates new `model_data.csv`
3. ⏳ **Run ENTIRE `03_forecasting_pipeline.ipynb`** → Finds best model with new features
4. 📊 **Analyze results** → Check if XGBoost beats naive baseline

---

**Good luck! You're on the right track. These features should give your models the predictive power they need.**
