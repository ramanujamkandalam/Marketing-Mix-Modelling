# Marketing Mix Modeling & Budget Optimization

## Project Overview

This project explores how marketing investments across multiple media channels influence television viewership and how budget allocation decisions can be optimized using Marketing Mix Modeling (MMM).

The objective was to quantify the relationship between media spend and audience performance, simulate alternative allocation strategies, and identify opportunities to improve return on marketing investment while maintaining realistic advertising response behavior.

---

## Business Problem

Marketing teams must decide how to allocate limited budgets across channels such as Network TV, Cable TV, and Digital Media. However, advertising effects are rarely immediate or linear.

This project investigates:

* Which channels are most strongly associated with viewership performance
* How advertising carryover impacts future audience reach
* Where diminishing returns begin to occur
* How alternative budget allocation strategies affect projected performance

---

## Methodology

### Data Preparation

* Data cleaning and feature engineering
* Time-based validation strategy
* Episode lifecycle and contextual variable creation

### Marketing Mix Modeling

A regression-based MMM framework was developed using:

* Linear Regression
* Adstock Transformations (carryover effects)
* Saturation Transformations (diminishing returns)
* Multicollinearity Diagnostics (VIF)
* Out-of-Sample Evaluation (R², RMSE, MAE)

### Budget Optimization Framework

The optimization workflow included:

1. Establish baseline historical spend allocation
2. Recompute adstock effects
3. Apply saturation transformations
4. Generate new viewership forecasts
5. Estimate revenue impact
6. Compare scenarios against baseline performance

---

## Key Findings

* Advertising carryover effects significantly improved model realism compared to raw spend models.
* Saturation transformations captured diminishing returns at higher investment levels.
* Traditional television channels demonstrated the strongest modeled association with viewership.
* Digital media contributed positively but showed faster saturation behavior.
* Budget reallocation scenarios suggested that relatively small shifts in media allocation could generate measurable performance improvements.

---

## Results

* Best Model Train R²: 0.64
* Best Model Test R²: 0.46
* Evaluated multiple adstock and saturation specifications
* Conducted scenario-based budget optimization simulations
* Estimated revenue and audience lift under alternative spending strategies

---

## Important Modeling Considerations

This project uses observational marketing data and should be interpreted as a decision-support framework rather than a strictly causal model.

Key limitations include:

* Potential endogeneity in media allocation decisions
* Omitted variables not represented in the dataset
* Regression-based attribution assumptions
* Simplified simulation environment

Results are therefore interpreted as directional insights for budget planning rather than definitive causal estimates.

---

## Technologies Used

* Python
* Pandas
* NumPy
* Statsmodels
* Scikit-Learn
* Matplotlib
* Seaborn

---

## Repository Contents

* MMM_Casestudy.ipynb — Complete modeling workflow includes
* Data Preparation & Feature Engineering
* Adstock and Saturation Analysis
* Model Comparison & Diagnostics
* Budget Reallocation Simulations
* Scenario-Based Revenue Optimization

---

## Key Learning

One of the most valuable lessons from this project was understanding the difference between predictive performance and causal interpretation.

While adstock and saturation transformations improved model realism and predictive accuracy, they do not fully resolve the underlying challenge that marketing spend is often allocated based on expected demand. This reinforced the importance of interpreting Marketing Mix Models as strategic decision-support tools rather than direct measures of causal impact.
