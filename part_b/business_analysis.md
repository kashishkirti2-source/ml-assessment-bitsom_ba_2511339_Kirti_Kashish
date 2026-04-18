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
