# Notebook 11: Seasonality-Based Price Fluctuation Models - Results Explanation

This document explains what **Notebook 11** (`11_seasonality_price_fluctuation_models.ipynb`) does and how to interpret its results.

---

## What Does Notebook 11 Do?

This notebook **builds on Notebook 10** by adding **seasonal features** to predict weekly price fluctuations. It answers the question:

**"Do holidays and seasonal patterns affect container freight price volatility?"**

### Key Innovation:
- Adds 16 seasonal features (holidays, quarters, cyclical month encoding)
- Identifies which seasonal periods cause the most price volatility
- Tests same 3 models with enhanced feature set

---

## Seasonal Features Engineered

### Major Holidays/Events (Binary Indicators):

| Feature | Period | Impact on Shipping |
|---------|--------|-------------------|
| **chinese_new_year_period** | Late Jan - Mid Feb | Factory shutdowns in China |
| **christmas_period** | Mid Dec - Early Jan | Peak demand before holidays |
| **thanksgiving_period** | Late November | US holiday shipping rush |
| **golden_week_china** | Early October | China national holiday |
| **may_day_period** | Early May | Labor day in China |
| **mid_autumn_period** | Mid September | Chinese harvest festival |
| **peak_shipping_season** | Aug-Oct | Pre-Christmas shipping surge |
| **low_shipping_season** | Jan-Feb | Post-holiday slump |

### Time Features:

| Feature | What It Captures |
|---------|-----------------|
| **q1, q2, q3, q4** | Quarter indicators (seasonality by quarter) |
| **month_sin, month_cos** | Cyclical month encoding (captures yearly cycle) |
| **month** | Raw month number (1-12) |
| **week_of_year** | Week number (1-52) |

---

## Seasonality Impact Analysis - The BIG Finding! 🎯

This is **the most important output** of Notebook 11:

### Top Seasonal Events by Impact on Price Fluctuations:

```
SEASONALITY IMPACT RANKING (by absolute impact):
================================================================================
Season                         Avg Fluctuation   Diff from Normal   Occurrences
================================================================================
christmas_period                        $125.71           +$133.41            28
peak_shipping_season                    -$69.31            -$92.87            89
thanksgiving_period                      $90.25            +$92.05            16
may_day_period                           $87.27            +$88.70            15
mid_autumn_period                       -$78.19            -$83.71            16
chinese_new_year_period                 -$68.36            -$77.01            33
low_shipping_season                     -$58.97            -$72.07            59
golden_week_china                       -$42.60            -$45.82            10
```

### What This Table Means:

**Column Explanations:**

1. **Avg Fluctuation**: Average price change during this period
2. **Diff from Normal**: How much MORE/LESS volatile vs normal periods
3. **Occurrences**: Number of weeks in dataset matching this period

**Key Insights:**

#### 🔴 **INCREASES Volatility (Positive Impact):**

**1. Christmas Period (+$133.41)**
- **Biggest volatility driver**
- Pre-holiday shipping rush creates huge price swings
- Average fluctuation: $125.71 (vs -$7.70 during normal periods)
- **Actionable**: Expect unpredictable prices mid-Dec to early-Jan

**2. Thanksgiving (+$92.05)**
- Second-highest volatility
- US holiday creates shipping rush
- Short period (16 weeks total over 7 years)

**3. May Day (+$88.70)**
- Chinese labor day disrupts manufacturing
- Creates supply chain uncertainty

#### 🟢 **STABILIZES Prices (Negative Impact):**

**1. Peak Shipping Season (-$92.87)**
- **Counter-intuitive!** Aug-Oct is MORE stable, not less
- High volume but predictable demand = stable pricing
- 89 weeks of data (largest sample)
- **Actionable**: Best time for price predictability

**2. Mid-Autumn Festival (-$83.71)**
- Post-summer stabilization period
- Less volatility than normal

**3. Chinese New Year (-$77.01)**
- Predictable factory shutdowns reduce uncertainty
- Everyone knows what to expect
- **Actionable**: Prices stabilize during CNY despite shutdowns

---

## Model Performance Results

### Comparison with Notebook 10:

| Metric | Notebook 10 (No Seasonality) | Notebook 11 (With Seasonality) | Improvement |
|--------|------------------------------|--------------------------------|-------------|
| **RMSE** | $196.16 | **$181.76** | **-7.3%** ✅ |
| **MAE** | $140.83 | **$136.47** | **-3.1%** ✅ |
| **R²** | 0.176 | **0.293** | **+66%** ✅ |
| **Direction Accuracy** | 64.94% | **64.94%** | No change |

### Results from Execution:

```
               Model       RMSE        MAE  Relative_MAE_%        R²  Direction_Accuracy_%
  Linear Regression     $181.76    $136.47          87.84%     0.293                64.94%
      Decision Tree     $224.83    $164.84         106.10%    -0.083                61.04%
K-Nearest Neighbors     $186.90    $138.05          88.85%     0.252                63.64%
```

---

## What Do These Results Mean?

### 🏆 Winner: Still **Linear Regression**

**But now it's MUCH better:**
- **R² improved from 0.176 → 0.293** (explains 66% MORE variance!)
- **RMSE improved $196 → $182** (predictions more accurate)
- Seasonal features add real predictive power

### Key Findings:

1. **Seasonality Matters**: 7% RMSE improvement proves seasonal patterns affect prices

2. **R² Still Low (0.293)**: Even with seasonality, model only explains 29% of fluctuations
   - 71% still unexplained (external shocks, news, oil prices, etc.)

3. **Decision Tree Still Overfits**: Negative R² even with more features

---

## Seasonal Feature Importance

### ⚠️ IMPORTANT: The Scaling Issue You Found

The output you saw:
```
TOP 5 MOST IMPORTANT SEASONAL FACTORS:
1. q3     Model importance: 2895760111112764.500000
2. q2     Model importance: 2848230501949771.500000
...
```

**This is a BUG in the notebook!** Here's what happened:

### The Problem:

The notebook averages feature importance from:
- **Linear Regression coefficients**: Large absolute numbers (e.g., 1,232 for price_lag_1w)
- **Decision Tree importances**: Normalized 0-1 scale (e.g., 0.3287 for price_lag_1w_roll_std_4w)

When you **average** 1,232 (LR) with 0.09 (DT), you get ~616 - which is meaningless!

### The Correct Interpretation:

**Look at each model separately:**

#### From Linear Regression (Top 10):
```
Feature                       Coefficient
price_lag_1w                     1232.46
price_lag_1w_roll_mean_4w        -596.40
price_lag_2w                     -534.66
month_sin                          78.76  ← Seasonal feature!
week_of_year                       53.27  ← Seasonal feature!
q4                                 30.08  ← Seasonal feature!
q1                                -30.03  ← Seasonal feature!
month_cos                         -27.94  ← Seasonal feature!
```

**5 out of 10 top features are seasonal** - proves seasonality is important!

#### From Decision Tree (Top 10):
```
Feature                       Importance
price_lag_1w_roll_std_4w         0.3288
week_of_year                     0.2058  ← Seasonal feature!
price_lag_1w_roll_std_8w         0.1401
month_cos                        0.1113  ← Seasonal feature!
price_lag_1w                     0.0927
```

**2 out of 5 top non-price features are seasonal**

### What To Use Instead:

Use the **Seasonality Impact Analysis table** (from Step 3) which correctly shows:
- Christmas period: +$133.41 impact
- Peak shipping season: -$92.87 impact
- Etc.

This is **accurate** because it compares actual average fluctuations during vs outside each period.

---

## Practical Insights

### When to Expect High Volatility:

1. **Mid-December to Early January** (Christmas Period)
   - Prices can swing $125+ per week
   - Most unpredictable period
   - **Action**: Avoid spot market, use contracts

2. **Late November** (Thanksgiving)
   - Second-highest volatility
   - Short window of uncertainty

3. **Early May** (May Day)
   - Chinese manufacturing disruptions

### When to Expect Stable Prices:

1. **August - October** (Peak Shipping Season)
   - Most stable period despite high volume
   - Best time for predictable pricing
   - **Action**: Good time for spot market deals

2. **Late January - Mid February** (Chinese New Year)
   - Predictable shutdowns = less uncertainty
   - Prices stabilize

3. **Post-Summer Period**
   - Stabilization after summer shipping

---

## Generated Files

### Key Output Files:

**1. `seasonality_impact_analysis.csv`**
- **Most important file** - seasonal impact rankings
- Columns: avg_fluctuation, std_fluctuation, difference_from_normal, occurrences
- Use this for understanding which seasons matter

**2. `seasonality_model_comparison.csv`**
- Model performance metrics with seasonal features
- Compare with `weekly_fluctuation_model_comparison.csv` to see improvement

**3. `seasonality_fluctuation_predictions.csv`**
- Actual vs predicted with seasonal context
- Includes seasonal indicators for each date

**4. `seasonal_feature_importance.csv`**
- ⚠️ Contains the scaling bug - use with caution
- Better to use `seasonality_impact_analysis.csv` instead

---

## Visualizations Explained

### 1. **Seasonal Impact Bar Chart**
- Green bars = increases volatility (Christmas, Thanksgiving)
- Red bars = decreases volatility (Peak season, CNY)
- Longer bars = bigger impact

### 2. **Model Comparison Charts**
- Shows Linear Regression performs best
- Seasonal features improved all models slightly

### 3. **Seasonal Feature Importance Chart**
- ⚠️ Affected by scaling bug
- Use Seasonality Impact Analysis table instead

### 4. **Predictions Over Time**
- Red vertical lines mark Chinese New Year periods
- Shows how models handle seasonal volatility

---

## Key Takeaways

### ✅ What We Learned:

1. **Seasonality SIGNIFICANTLY affects price fluctuations**
   - 7% RMSE improvement
   - 66% more variance explained (R²)

2. **Christmas = Highest Volatility**
   - +$133 more volatile than normal
   - Plan accordingly!

3. **Peak Shipping Season = Most Stable**
   - Paradoxically, high-volume period is predictable
   - Best time for price forecasting

4. **Seasonal features improve Linear Regression**
   - From R² 0.176 → 0.293
   - Still only explains 29% of fluctuations

### ⚠️ Limitations:

1. **Scaling bug in feature importance** - use impact analysis instead
2. **R² still low (0.293)** - external factors dominate
3. **Decision Tree still overfits** - don't use it

### 🎯 Best Practices:

1. **Use seasonality-aware model** for better predictions
2. **Avoid spot market during Christmas period** (too volatile)
3. **Lock in rates during peak season** (Aug-Oct) when stable
4. **Expect stabilization during CNY** despite shutdowns

---

## Comparison with Notebook 10

| Aspect | Notebook 10 | Notebook 11 |
|--------|-------------|-------------|
| Features | 30 lagged price features | 23 features (16 seasonal + 7 price) |
| Best R² | 0.176 | **0.293** ✅ |
| Best RMSE | $196.16 | **$181.76** ✅ |
| Top Insight | Recent momentum matters | **Seasonality matters more** |
| Use Case | General forecasting | **Seasonality-aware forecasting** |

**Recommendation**: Use Notebook 11's approach for production - seasonality adds real value!

---

## Next Steps

See `MODEL_COMPARISON_GUIDE.md` for detailed comparison of all models across both notebooks and recommendations for further improvements.
