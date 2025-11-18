# Notebook 10: Weekly Price Fluctuation Models - Results Explanation

This document explains what **Notebook 10** (`10_weekly_price_fluctuation_models.ipynb`) does and how to interpret its results.

---

## What Does Notebook 10 Do?

This notebook predicts **weekly price fluctuations** (the change in price from one week to the next) rather than absolute price values. It trains and compares three different machine learning models to determine which is best at forecasting price movements.

### Key Difference from Other Notebooks:
- **Other notebooks** predict: "Next week's price will be $1,200"
- **This notebook** predicts: "Next week's price will change by **+$50** from current price"

---

## Target Variable

**`price_fluctuation_1w`** = Future Price - Current Price

- **Positive values (+)**: Price will increase
- **Negative values (-)**: Price will decrease
- **Zero (0)**: No change

**Example:**
- Current price: $1,000
- Next week's price: $1,080
- **Fluctuation = +$80** (price increased)

---

## Models Tested

### 1. **Linear Regression**
- **What it is**: Simple model that finds a linear relationship between features and fluctuations
- **Pros**: Fast, interpretable, works well for linear relationships
- **Cons**: Can't capture complex non-linear patterns

### 2. **Decision Tree**
- **What it is**: Tree-based model that makes decisions using if-then-else rules
- **Pros**: Can capture non-linear patterns, no feature scaling needed
- **Cons**: Prone to overfitting (memorizing training data instead of learning patterns)

### 3. **K-Nearest Neighbors (KNN)**
- **What it is**: Predicts based on the average of K similar historical examples
- **Pros**: Simple, no training required, can capture local patterns
- **Cons**: Slow for large datasets, sensitive to irrelevant features

---

## Results Interpretation

### Model Performance Metrics

| Metric | What It Means | How to Interpret |
|--------|--------------|------------------|
| **RMSE** | Root Mean Squared Error - average prediction error | **Lower is better**. $196 means predictions are off by ~$196 on average |
| **MAE** | Mean Absolute Error - average absolute error | **Lower is better**. Easier to interpret than RMSE |
| **R²** | Proportion of variance explained (0 to 1) | **Higher is better**. 0.176 means model explains 17.6% of price fluctuations |
| **Direction Accuracy** | % of times model correctly predicts UP vs DOWN | **Higher is better**. 65% means correct direction 65% of time (vs 50% random) |

### Actual Results from Execution:

```
               Model       RMSE        MAE  Relative_MAE_%        R²  Direction_Accuracy_%
  Linear Regression     $196.16    $140.83          90.65%     0.176                64.94%
      Decision Tree     $339.87    $251.45         161.85%    -1.474                58.44%
K-Nearest Neighbors     $210.39    $151.01          97.20%     0.052                63.64%
```

---

## What Do These Results Mean?

### 🏆 Winner: **Linear Regression**

**Why?**
1. **Lowest RMSE** ($196.16): Most accurate predictions on average
2. **Positive R²** (0.176): Explains 17.6% of variance (Decision Tree has negative R²!)
3. **Best Direction Accuracy** (64.94%): Correctly predicts if price goes UP or DOWN most often

### ⚠️ Decision Tree Failed

**Negative R² (-1.474)** means the model is **worse than just predicting the average every time**!

**Why it failed:**
- Overfitting: Memorized training data but can't generalize
- Too sensitive to noise in fluctuation data
- GridSearchCV couldn't find good hyperparameters

### 😐 KNN: Middle Ground

- Decent RMSE but higher than Linear Regression
- Low R² (0.052): Explains only 5% of variance
- Slightly worse direction accuracy than Linear Regression

---

## Features Used (Top 30)

The model selects the **top 30 features most correlated with price fluctuations**:

### Most Important Features:

1. **`price_lag_1w_pct_change_1w`** (correlation: 0.4197)
   - Percent change in price from last week
   - **Most predictive feature** - recent momentum matters!

2. **`price_lag_1w_pct_change_4w`** (correlation: 0.3528)
   - Percent change over 4 weeks
   - Longer-term momentum indicator

3. **`sh_import_general_cargo_lag_1w_lag_1w`** (correlation: 0.2320)
   - Shanghai port import activity (lagged)
   - Economic activity indicator

### What This Tells Us:
- **Recent price momentum** is the best predictor of future fluctuations
- Longer-term trends also matter
- Port activity provides additional signal

---

## Direction Prediction Analysis

### Confusion Matrix (Linear Regression):

```
               Predicted Down | Predicted Up
Actual Down:        31        |      18
Actual Up:           9        |      19
```

**Interpretation:**

- **True Negatives (31)**: Correctly predicted price would go DOWN
- **False Positives (18)**: Predicted UP but actually went DOWN
- **False Negatives (9)**: Predicted DOWN but actually went UP
- **True Positives (19)**: Correctly predicted price would go UP

**Accuracy Breakdown:**
- **Predicting DOWN**: 31/(31+18) = **63.3% accurate**
- **Predicting UP**: 19/(9+19) = **67.9% accurate**

The model is slightly better at predicting upward movements than downward ones.

---

## Practical Usage

### How to Use These Results:

1. **For Risk Management:**
   - Model gives you **direction** (up/down) with 65% accuracy
   - Use for hedging decisions: "65% chance price will increase next week"

2. **For Planning:**
   - RMSE of $196 means predictions have ~$200 margin of error
   - Don't rely on exact fluctuation amounts
   - Focus on direction instead

3. **Trading Strategy Example:**
   - If model predicts fluctuation > +$100: Consider buying (65% chance it's correct)
   - If model predicts fluctuation < -$100: Consider selling/waiting
   - Use stop-losses since 35% of predictions are wrong

### Limitations:

⚠️ **Low R² (0.176)** means model only explains **17.6% of fluctuations**

**This means:**
- 82.4% of price fluctuations are due to factors NOT in this model
- External shocks (geopolitical events, oil prices, etc.) dominate
- Model is useful but limited

---

## Generated Files

The notebook creates these CSV files in `data/processed/`:

### 1. `weekly_fluctuation_model_comparison.csv`
Contains performance metrics for all three models:
- RMSE, MAE, R², Direction Accuracy for each model
- Use for comparing model performance

### 2. `weekly_fluctuation_predictions.csv`
Contains actual vs predicted fluctuations for test period:
- Columns: Date, Actual_Fluctuation, LR_Predicted, DT_Predicted, KNN_Predicted
- Use for analyzing prediction errors and patterns

---

## Visualizations Explained

### 1. **Distribution of Price Fluctuations**
- Shows most fluctuations are small (near zero)
- Some extreme fluctuations (+/- $500+)
- Roughly symmetric distribution (equal ups and downs)

### 2. **RMSE/MAE Comparison Bar Charts**
- Linear Regression has shortest bars (best performance)
- Decision Tree has tallest bars (worst performance)

### 3. **Direction Accuracy Bar Chart**
- All models > 50% (red line) so better than random
- Linear Regression ~65% is modest but useful

### 4. **Actual vs Predicted Line Chart**
- Black line = actual fluctuations (ground truth)
- Blue/Green/Orange dashed lines = model predictions
- Linear Regression (blue) tracks actual best
- Shows models struggle with extreme movements

---

## Key Takeaways

✅ **What Works:**
- Linear Regression predicts price fluctuations reasonably well
- Recent price momentum is most predictive
- 65% direction accuracy is actionable

⚠️ **What Doesn't Work:**
- Decision Tree overfits badly - don't use it
- Low R² means most fluctuations are unpredictable
- Exact fluctuation amounts are unreliable (use direction instead)

🎯 **Best Use Case:**
- Use for directional forecasting (will price go up or down?)
- Combine with other indicators/judgment
- Don't rely on exact fluctuation magnitude

---

## Next Steps

To improve these results, see **Notebook 11** which adds **seasonal features** that improve:
- RMSE from $196 → **$182** (7% improvement)
- R² from 0.176 → **0.293** (66% more variance explained)

See `NOTEBOOK_11_RESULTS_EXPLANATION.md` for details.
