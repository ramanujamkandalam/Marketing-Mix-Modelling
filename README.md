# Marketing Mix Modeling: From Regression to Hierarchical Bayesian MMM

## Project Overview

This project examines how spending across Network TV, Cable TV, and Digital Media relates to television audience and revenue performance.

The analysis was developed in two phases:

1. A regression-based MMM for learning adstock, saturation, model diagnostics, and budget simulation.
2. A hierarchical Bayesian MMM for estimating uncertainty, differences between shows, and distributions of possible outcomes.

The project demonstrates a natural progression from producing one fitted set of coefficients to reasoning through posterior distributions and decision uncertainty.

## Business Problem

Marketing teams must allocate limited budgets across channels whose effects are neither immediate nor linear.

This project investigates:

- Which channels are associated with audience and revenue performance?
- How long do advertising effects persist?
- Where do diminishing returns begin?
- How does media effectiveness differ between shows?
- How should uncertainty influence budget decisions?
- When does a technically valid model fail to represent the actual business context?

## Dataset

The data contains:

- 420 show-week observations
- 8 television shows
- 28 show-season combinations
- 15 lifecycle weeks per show-season
- Network TV, Cable TV, and Digital spending
- Viewership and revenue outcomes
- Holiday, Lead-in Bonus, lifecycle week, and episode-type controls

## Phase 1: Regression-Based MMM

The first phase established an interpretable benchmark using:

- Linear regression
- Geometric adstock transformations
- Saturation transformations
- Time-based validation
- Multicollinearity diagnostics using VIF
- Out-of-sample R², RMSE, and MAE
- Scenario-based budget reallocation

### Regression Results

- Best train R²: `0.64`
- Best test R²: `0.46`
- Evaluated multiple adstock and saturation specifications
- Simulated audience and revenue outcomes under alternative allocations

This phase helped establish why carryover and diminishing returns are necessary. Its main limitation was that it selected one transformation and one fitted coefficient for each variable.

## Phase 2: Hierarchical Bayesian MMM

The second phase extended the regression framework using PyMC.

The Bayesian model included:

- Channel-specific geometric adstock
- Hill saturation with a fixed slope of one
- Positive media-effect priors
- Show-specific media coefficients
- Partially pooled show and show-season baselines
- Holiday, Lead-in Bonus, lifecycle-week, and episode-type controls
- Prior predictive checks
- NUTS-MCMC posterior sampling
- Posterior predictive intervals

Instead of returning one parameter combination, the model generated 4,000 joint posterior draws representing plausible combinations of adstock, saturation, coefficients, baselines, and unexplained error.

## Validation Lesson: What Went Wrong

The initial Bayesian validation trained on Weeks 1–12 and tested on Weeks 13–15.

Although this prevented future-outcome leakage, it created a lifecycle distribution shift:

- Training contained no Finales.
- Finales represented 33.3% of the test set.
- Training contained no Weeks 13–15.
- Regular test observations came only from late lifecycle Weeks 13–14.
- The model was forced to extrapolate lifecycle behavior beyond its training support.

The model still achieved:

- Test R²: `0.892`
- Test RMSE: `$12,736`
- Test MAE: `$11,203`
- 90% predictive interval coverage: `82.1%`

The predictive intervals were somewhat overconfident, and some late-season observations showed systematic overprediction.

This result was retained as a **late-season extrapolation stress test** rather than treated as the final validation design.

## Redesigned Validation

The second validation design used:

- Complete earlier seasons for training
- The latest complete season of every show for testing

This ensured that training contained:

- All eight shows
- Weeks 1–15
- PreLaunch, Premiere, Regular, and Finale episodes
- Complete historical lifecycle behavior

The test set remained forward-looking because it contained only the latest season of each show.

### Bayesian Convergence

- Retained posterior draws: `4,000`
- Maximum R-hat: `1.004`
- Minimum bulk ESS: `739`
- Minimum tail ESS: `612`
- Divergences: `0`

### Latest-Season Error Distribution

- Median absolute error: `$6,561`
- Median percentage error: `8.73%`
- P90 absolute error: `$20,143`
- P90 percentage error: `26.67%`
- Maximum absolute error: `$30,932`

The model performed well for many show-weeks, but prediction risk was not uniform. Show3’s latest season was underpredicted by approximately `$21,873` on average, indicating an unseen season-level increase that could not be inferred from earlier seasons alone.

## Key Findings

- Adstock improved realism by carrying part of an advertising effect into later weeks.
- Saturation prevented the model from assuming constant returns at higher spending levels.
- Hierarchical pooling allowed shows to differ while still sharing information.
- Predictive uncertainty varied meaningfully across shows and lifecycle stages.
- Average metrics such as R² and RMSE hid important show-level and tail errors.
- Validation design materially changed the business question answered by the model.
- A leakage-safe split can still be inappropriate when training lacks relevant lifecycle support.

## Decision-Support Application

The posterior can support a stakeholder-facing scenario tool with:

- Show and lifecycle selection
- Channel-spend sliders
- Posterior median revenue
- Predictive intervals
- Probability of exceeding a revenue target
- Channel contribution distributions
- ROAS and marginal ROAS scenarios
- Warnings for spending outside historical support

Scenario recommendations should generally remain within each channel’s historical P05–P95 range. Predictions beyond that range should be clearly labeled as extrapolation.

## Important Limitations

- The dataset contains only 28 show-seasons, limiting the precision of complex nonlinear and show-specific parameters.
- Geometric adstock, an eight-week maximum lag, and fixed-slope Hill saturation are modeling choices.
- Future seasons may shift because of content, cast, platform availability, competition, or external audience trends not included in the data.
- A single Normal likelihood may not fully represent different uncertainty levels across shows and episode types.
- Baseline revenue at zero media is weakly identified because historical observations contain nonzero media spending.
- Positive media priors prevent negative channel coefficients, so posterior positivity partly reflects the model assumption.
- MCMC convergence confirms that the specified model was sampled reliably; it does not prove that the model structure is correct.

## Causal Limitation

This is an observational MMM. Strong out-of-sample prediction does not prove that media spending caused the predicted revenue.

Marketing budgets may increase when higher demand is already expected. Unobserved factors may also influence both media spending and revenue, including:

- Content quality
- Competitive activity
- Pricing and distribution changes
- Audience interest
- Broader promotional activity

Channel contributions, ROAS, and marginal ROAS should therefore be interpreted as model-based estimates conditional on the assumptions—not proven causal effects.

Stronger causal confidence would require evidence from randomized lift tests, geo experiments, budget shocks, instrumental variables, or externally calibrated priors.

## Technologies Used

- Python
- Pandas
- NumPy
- Scikit-learn
- Statsmodels
- PyMC
- ArviZ
- PyTensor
- JAX and NumPyro
- Matplotlib
- Seaborn
- Google Colab GPU

## Repository Contents

- `MMM_Casestudy.ipynb` — Regression-based MMM, diagnostics, and budget simulations
- `Hierarchical_Bayesian_MMM.ipynb` — Bayesian modeling, validation redesign, posterior diagnostics, and uncertainty analysis
- `README.md` — Business context, methods, results, limitations, and key learnings

## Key Learning

The most important lesson was not simply how to fit a more complex model.

The regression model provided an interpretable starting point. The Bayesian extension added uncertainty and partial pooling. However, the largest improvement in reasoning came from discovering that the original temporal split answered only a partial business question.

The project reinforced a practical modeling principle:

> A model can be mathematically correct and leakage-safe while still being evaluated in the wrong business context.

The process reflected Bayesian thinking itself: begin with reasonable assumptions, compare them with evidence, identify where they fail, and update the approach.
