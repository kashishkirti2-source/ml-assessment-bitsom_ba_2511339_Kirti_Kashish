# B1. Problem Formulation

## (a) Machine Learning Formulation

This problem can be formulated as a **supervised regression problem**.

- **Target Variable:**  
  `items_sold` (number of items sold per store per month)

- **Candidate Input Features:**  
  - Store attributes: `store_id`, `store_size`, `location_type` (urban / semi-urban / rural)  
  - Promotion details: `promotion_type` (Flat Discount, BOGO, Free Gift, etc.)  
  - Temporal features: month, seasonality, festivals, weekends  
  - Market conditions: `competition_density`  
  - Customer behavior indicators: footfall, demographics (if available)

- **Type of ML Problem:**  
  Regression

- **Justification:**  
  The goal is to predict a continuous numerical value (number of items sold). Since we have historical data with known inputs and outputs, this is a supervised learning problem. Regression is appropriate because the output variable is continuous and not categorical.

---

## (b) Why "Items Sold" is a Better Target than Revenue

Using **items sold (sales volume)** is more reliable than total revenue because:

- Revenue can be influenced by **price changes, discounts, and product mix**, which may distort the true effectiveness of a promotion.
- A promotion might increase volume but reduce revenue due to heavy discounts.
- Items sold directly reflects **customer response and demand**, making it a cleaner measure of promotion effectiveness.

### Broader Principle

This illustrates an important ML principle:

> The target variable should directly align with the business objective and should not be influenced by external or confounding factors.

Choosing the right target ensures that the model learns meaningful patterns and produces actionable insights.

---

## (c) Alternative to a Single Global Model

Instead of using one global model for all stores, a better approach is:

### Segmented or Hierarchical Modeling

- Train **separate models for different store segments**, such as:
  - Urban vs Semi-urban vs Rural
  - Small vs Medium vs Large stores

**OR**

- Use a **hierarchical (multi-level) model** that captures both global trends and store-specific variations.

### Justification

- Customer behavior and promotion effectiveness vary significantly across locations.
- A single global model may average out these differences and perform poorly.
- Segmented models allow the system to capture **localized patterns**, leading to more accurate and actionable predictions.

---

### Final Insight

Accounting for store-level heterogeneity leads to better decision-making and more effective promotion strategies across diverse markets.


# B2. Data and EDA Strategy

## (a) Data Joining and Dataset Grain

The raw data is spread across four tables:
- Transactions
- Store attributes
- Promotion details
- Calendar data (weekend and festival flags)

### Data Joining Strategy

- Start with the **transactions table** as the base.
- Join **store attributes** using `store_id`
- Join **promotion details** using `promotion_id` or equivalent key
- Join **calendar table** using `transaction_date`

### Final Dataset Grain

The final modelling dataset should have:

> **One row per store per day (or per store per transaction date)**

### Aggregations Before Modelling

If transactions are at a lower level (e.g., item-level), we would aggregate to:

- Total `items_sold` per store per day
- Average or total `basket_size`
- Total number of transactions
- Promotion applied (if multiple, choose dominant or encode appropriately)

This ensures consistency in the dataset and aligns with the prediction objective.

---

## (b) Exploratory Data Analysis (EDA)

Before building the model, the following analyses should be performed:

### 1. Distribution of Target Variable (items_sold)
- Plot: Histogram / Boxplot  
- Look for: skewness, outliers  
- Impact: May require log transformation or outlier handling

---

### 2. Sales by Promotion Type
- Plot: Bar chart (average items sold per promotion)  
- Look for: which promotions perform better  
- Impact: Helps validate if promotion_type is a strong feature

---

### 3. Sales by Location Type
- Plot: Boxplot or grouped bar chart (urban vs rural vs semi-urban)  
- Look for: variation in customer behavior across regions  
- Impact: Supports segmentation or interaction features

---

### 4. Time-Based Trends
- Plot: Line chart over time (daily/weekly sales)  
- Look for: seasonality, trends, spikes during festivals  
- Impact: Justifies temporal features like month, day_of_week, is_festival

---

### 5. Correlation Analysis
- Plot: Correlation heatmap  
- Look for: relationships between numerical features  
- Impact: Helps in feature selection and avoiding multicollinearity

---

## (c) Handling Promotion Imbalance

Since **80% of transactions occur without any promotion**, the dataset is highly imbalanced.

### Impact on Model

- The model may become biased towards "no promotion" scenarios
- It may fail to properly learn the effect of different promotions
- Predictions for promotional scenarios may be inaccurate

### Steps to Address This

- Ensure **promotion_type is properly encoded** and not ignored
- Use **stratified sampling (if applicable)** to maintain balance
- Consider **resampling techniques** (oversampling minority promotions)
- Add **interaction features** (e.g., promotion × location)
- Evaluate model performance separately for promotional vs non-promotional cases

---

### Final Insight

Proper data integration and thoughtful EDA are critical for uncovering patterns and ensuring that the model captures the true drivers of sales.


# B3. Model Evaluation and Deployment

## (a) Train-Test Split and Evaluation Metrics

### Train-Test Split Strategy

Since the data is **time-based (monthly data over 3 years)**, we should use a **temporal split**:

- Use the **first ~2.5 years (80%)** as training data  
- Use the **most recent ~6 months (20%)** as test data  

### Why Not Random Split?

A random split is inappropriate because:

- It mixes past and future data, causing **data leakage**
- The model may learn patterns from the future that wouldn't be available in real-world predictions
- It breaks the natural time dependency of the data

> In real-world deployment, we always predict the future using past data — the split must reflect that.

---

### Evaluation Metrics

1. **RMSE (Root Mean Squared Error)**
   - Penalizes larger errors more heavily  
   - Useful when large prediction errors are costly  
   - In this context: helps ensure we avoid big mistakes in sales forecasting

2. **MAE (Mean Absolute Error)**
   - Measures average prediction error  
   - Easy to interpret (average units off in prediction)  
   - In this context: tells us how many items we are off on average per store

3. *(Optional but strong)* **MAPE (Mean Absolute Percentage Error)**
   - Measures error in percentage terms  
   - Useful for comparing performance across stores of different sizes  

---

## (b) Explaining Model Recommendations Using Feature Importance

When the model recommends different promotions for the same store in different months, it is due to changes in **input features**.

### Investigation Approach

- Analyze **feature importance** from the model (especially Random Forest)
- Check key features for both months:
  - `month` / seasonality
  - `is_festival`
  - `is_weekend`
  - historical sales patterns
  - promotion effectiveness in similar conditions

### Example Explanation

- In **December**, high festival activity may increase responsiveness to **Loyalty Points Bonus**
- In **March**, lower seasonal demand may make **Flat Discount** more effective

### Communication to Marketing Team

- Present findings in simple business terms:
  - "Customers are more responsive to loyalty rewards during festive months"
  - "Discount-based promotions work better during non-peak periods"

- Use visual aids like:
  - Feature importance charts
  - Month-wise promotion performance comparisons

---

## (c) Deployment and Monitoring Strategy

### Model Saving

- Save the trained pipeline (preprocessing + model) using:
  - `joblib` or `pickle`
- This ensures consistency between training and inference

---

### Monthly Prediction Workflow

1. Collect new monthly data for all 50 stores  
2. Apply the **same preprocessing pipeline** (no refitting)  
3. Pass data into the saved model  
4. Generate predicted `items_sold` for each promotion scenario  
5. Recommend the promotion with the highest predicted sales  

---

### Monitoring and Maintenance

To ensure the model remains reliable:

#### 1. Performance Monitoring
- Track RMSE / MAE over time
- Compare predicted vs actual sales each month

#### 2. Data Drift Detection
- Monitor changes in:
  - customer behavior
  - promotion effectiveness
  - feature distributions

#### 3. Retraining Strategy
- Retrain periodically (e.g., quarterly or bi-annually)
- Retrain when:
  - performance drops significantly
  - new patterns emerge (e.g., new promotion types)

---

### Final Insight

A robust deployment system not only generates predictions but also continuously monitors performance to ensure the model stays relevant in a changing business environment.
