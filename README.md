# Marketing Mix Modeling: From Regression to Hierarchical Bayesian MMM

## Project Overview

This project studies how Network TV, Cable TV, and Digital spend relate to weekly television revenue across multiple shows and seasons.

The work evolved in two phases:

| Phase | Purpose | Main learning |
|---|---|---|
| Regression MMM | Build an interpretable baseline with adstock, saturation, diagnostics, and budget simulation | Good for learning mechanics, but returns one fixed set of coefficients |
| Hierarchical Bayesian MMM | Estimate uncertainty, show-level differences, and plausible ranges of outcomes | Better for decision-making under uncertainty |

The main goal was not only to fit a model, but to understand when a model is answering the right business question.

## Business Problem

Marketing teams need to allocate limited budget across channels whose effects are delayed and nonlinear.

This project asks:

- Which channels are associated with revenue performance?
- How long can advertising effects carry over?
- Where do diminishing returns begin?
- Do shows respond differently to the same channel?
- How should uncertainty affect budget decisions?
- When can a leakage-safe validation split still be poor business validation?

## Dataset

| Component | Detail |
|---|---:|
| Weekly observations | 420 |
| Shows | 8 |
| Show-season combinations | 28 |
| Weeks per show-season | 15 |
| Media channels | Network TV, Cable TV, Digital |
| Outcome | Revenue |
| Controls | Holiday, Lead-in Bonus, Week Number, Episode Type |

## Methodology Summary

| Component | What was done | Why it mattered |
|---|---|---|
| Spend variation check | Checked range and coefficient of variation by channel | MMM needs variation in spend to estimate response |
| Chronological ordering | Sorted by show, season, week, and air date | Adstock depends on correct time order |
| Time-aware validation | Avoided random splitting | Random splits can leak lifecycle and future information |
| Dummy variables | Used Regular episode as the reference category | Avoided the dummy variable trap |
| Leakage-safe scaling | Fit scalers using training data only | Prevented test-set information from leaking into training |
| Lag tensor | Created current week plus previous 8 weeks for each channel | Captured delayed media effects |
| Early-week normalization | Normalized only over valid historical lags | Prevented early season weeks from being unfairly understated |
| Adstock | Estimated channel-specific carryover | Modeled media memory over time |
| Saturation | Applied diminishing-return curves after adstock | Prevented unlimited linear media response |
| Hierarchical pooling | Allowed show and show-season differences | Balanced flexibility with overfitting control |
| Bayesian priors | Used realistic guardrails on parameters | Stabilized estimation in a small hierarchical dataset |
| Posterior prediction | Generated uncertainty intervals, not just point predictions | Supported risk-aware decision-making |

## Phase 1: Linear Regression MMM

The regression phase was used as the learning and benchmarking layer before moving into Bayesian MMM.

| Step | What was done | What it taught |
|---|---|---|
| Weekly aggregation | Built a show-week dataset and avoided double-counting repeated weekly values | The modeling grain must match the business question |
| OLS inference model | Fit a Statsmodels OLS model on raw media spend and controls | Raw spend had positive directional relationships with viewership |
| Dummy encoding | Dropped one episode type as the reference category | Prevented dummy-variable multicollinearity |
| Removed impressions | Excluded impressions from model features | Impressions were downstream of media spend and could dilute spend interpretation |
| VIF check | Checked multicollinearity across features | Media channel VIFs were close to 1; lifecycle variables showed moderate expected overlap |
| Baseline regression | Fit a plain multiple linear regression with raw spend | Created a simple benchmark before nonlinear transformations |
| Adstock regression | Applied geometric adstock by show-season | Improved fit by capturing media carryover |
| Saturation regression | Applied `log1p(adstock)` transformation | Added diminishing returns and business realism |
| Budget simulation | Reallocated spend across channels and predicted scenario outcomes | Turned the model into a decision-support exercise |

### Regression Model Results

| Model / check | Key result | Interpretation |
|---|---:|---|
| OLS inference R² | 0.571 | Raw media and controls explained about 57% of viewership variation |
| Raw-spend baseline train R² | 0.599 | Baseline captured directional relationships |
| Raw-spend baseline test R² | 0.342 | Generalization was limited without carryover/nonlinearity |
| Adstock model test R² | ~0.678 | Carryover materially improved predictive fit |
| Adstock + saturation train R² | 0.641 | Saturation reduced pure fit versus adstock-only but improved realism |
| Adstock + saturation test R² | 0.465 | Final regression model was more business-realistic but still limited |
| Media VIFs | ~1.02–1.07 | No severe multicollinearity among media spend channels |

### Regression Learnings

| Learning | Why it mattered |
|---|---|
| Linear regression is a useful starting point | It gives interpretable coefficients and a simple benchmark |
| Raw spend coefficients are not enough | Marketing effects can carry over and saturate |
| Adstock improved fit | Delayed effects mattered in the data |
| Saturation improved realism | The model stopped assuming constant returns at higher spend |
| Scenario simulation was useful | Stakeholders care about allocation decisions, not only coefficients |
| Regression still gave one fitted answer | It did not fully express parameter uncertainty or show-level heterogeneity |

The regression phase supported a practical recommendation: shifting some budget from Digital toward TV, especially Network TV, produced higher predicted viewership and revenue in the simulated scenarios. This was treated as directional rather than causal.

## Bayesian Model Assumptions

| Assumption | How it was represented | Why it was reasonable |
|---|---|---|
| Media effects are nonnegative | Log media coefficients were exponentiated | Paid media is expected to create lift, not directly reduce revenue |
| Carryover is bounded | Adstock alpha used a Beta prior between 0 and 1 | Decay should stay within a realistic range |
| Saturation point is positive | Half-saturation used a LogNormal prior | The 50% response point cannot be negative |
| Shows can respond differently | Show-specific media coefficients around global channel effects | Audiences differ by show |
| Similar shows still share information | Partial pooling | Helps avoid overfitting when data is limited |
| Controls can increase or decrease revenue | Normal-style priors on controls | Holidays, lead-ins, and lifecycle effects may move revenue either way |
| Error is positive | Positive residual noise prior | Model uncertainty cannot be negative |

These were weakly informative, business-reasonable priors. In a production setting, stronger priors should be triangulated from lift tests, geo experiments, previous MMMs, MTA, and stakeholder knowledge.

## Validation Lesson

The first Bayesian validation trained on Weeks 1–12 and tested on Weeks 13–15.

That split was chronological and leakage-safe, but it created a support mismatch:

| Issue | Why it mattered |
|---|---|
| Training had no Finale episodes | Test set included Finales |
| Training had no Weeks 13–15 | Model had to extrapolate late lifecycle behavior |
| Regular test episodes were only late-season weeks | Lifecycle behavior was not fully supported |
| 90% interval coverage was only 82.1% | The model was somewhat overconfident |

This version was retained as a late-season extrapolation stress test.

The improved validation trained on complete earlier seasons and tested on the latest complete season for every show.

| Validation design | Train | Test | Why better |
|---|---|---|---|
| Model 1 | Weeks 1–12 | Weeks 13–15 | Good stress test, but missing late lifecycle support |
| Model 2 | Earlier complete seasons | Latest season per show | Forward-looking while keeping all lifecycle types in training |

## Convergence Checks

Model 1 and Model 2 used different sampler settings, so their diagnostics are reported separately (numbers pulled directly from each model's `az.summary()` output).

### Model 1 — Weeks 1–12 train / Weeks 13–15 test (late-season stress test)

| Check | Result | Interpretation |
|---|---:|---|
| Chains | 4 | Independent chains were used to assess mixing |
| Posterior draws | 4,000 | 1,000 draws per chain |
| Tuning steps | 2,000 | Warmup for stable NUTS sampling |
| Target acceptance | 0.98 | Conservative setting for a hierarchical nonlinear model |
| Maximum R-hat | 1.006 | Chains converged well |
| Minimum bulk ESS | 733 | Central posterior estimates were stable |
| Minimum tail ESS | 886 | Uncertainty intervals had strong support |
| Divergences | 1 of 4,000 draws | Negligible; no meaningful sampler geometry failure |

### Model 2 — earlier complete seasons train / latest season test (final validation design)

| Check | Result | Interpretation |
|---|---:|---|
| Chains | 4 | Independent chains were used to assess mixing |
| Posterior draws | 4,000 | 1,000 draws per chain |
| Tuning steps | 2,500 | Long warmup for stable NUTS sampling |
| Target acceptance | 0.99 | Conservative setting for a hierarchical nonlinear model |
| Maximum R-hat | 1.004 | Chains converged well |
| Minimum bulk ESS | 739 | Central posterior estimates were stable |
| Minimum tail ESS | 612 | Uncertainty intervals had acceptable support |
| Divergences | 0 | No sampler geometry failures |

## Key Results

| Area | Takeaway |
|---|---|
| Adstock | Digital showed the longest modeled persistence in the final Bayesian model |
| Saturation | Digital had the lowest half-saturation, suggesting it reached diminishing returns sooner |
| Media coefficients | Network and Cable had larger global posterior mean coefficients than Digital |
| Regression benchmark | Raw linear regression was useful but limited; adstock improved fit the most in Phase 1 |
| Prediction | The hierarchical Bayesian MMM performed well, but strong lifecycle baselines remained competitive |
| Error analysis | Average metrics hid show-level risk, especially underprediction for Show3 |
| Uncertainty | Predictive intervals were useful for understanding risk, not just accuracy |

Important note: media coefficients are not directly ROAS. ROAS and marginal ROAS require posterior counterfactual simulation.

## Business Recommendations

| Recommendation | Reason |
|---|---|
| Use the Bayesian MMM as a decision-support tool, not just a prediction model | Its value is uncertainty, response curves, and scenario analysis |
| Keep the regression model as a benchmark | It helps explain whether added Bayesian complexity is earning its place |
| Keep simulations within historical P05–P95 spend ranges | Avoids unsupported extrapolation |
| Report predictive intervals along with point forecasts | Stakeholders need risk ranges, not just averages |
| Compare MMM against simple lifecycle baselines | A complex model should beat practical business benchmarks |
| Investigate show-level residuals | Large show-specific errors may indicate missing business drivers |
| Add ROAS/mROAS through counterfactual simulation | Coefficients alone are not business return metrics |
| Use experiments to strengthen causal claims | Observational MMM cannot prove causality by itself |

## Limitations

| Limitation | Why it matters |
|---|---|
| Observational data | Strong prediction does not prove media caused revenue changes |
| Only 28 show-seasons | Limits precision of complex hierarchical and nonlinear parameters |
| Eight-week max lag | A modeling choice, not a discovered truth |
| Fixed-slope Hill saturation | Simpler and stable, but less flexible |
| Positive media priors | Prevent negative media coefficients by assumption |
| No explicit interaction effects | TV × Digital or media × episode-type synergy was not modeled |
| Normal likelihood | May not capture different error levels across shows or episode types |
| Missing external factors | Content quality, competition, cast changes, and platform shifts were not included |

## Causal Limitation

This is an observational MMM. It estimates relationships conditional on the model assumptions.

Marketing spend may be higher when demand is already expected to be higher. Unobserved factors may affect both spend and revenue.

Stronger causal confidence would require:

- Randomized lift tests
- Geo experiments
- Budget shocks
- Instrumental variables
- Historical MMM calibration
- Experiment-informed Bayesian priors

## Technologies Used

| Category | Tools |
|---|---|
| Data work | Python, Pandas, NumPy |
| Modeling | Scikit-learn, Statsmodels, PyMC |
| Bayesian diagnostics | ArviZ |
| Sampling | JAX, NumPyro, NUTS |
| Visualization | Matplotlib, Seaborn |
| Environment | Google Colab GPU |

## Repository Contents

| File | Description |
|---|---|
| `MMM_Casestudy.ipynb` | Regression-based MMM, diagnostics, and budget simulation |
| `Hierarchical_Bayesian_MMM.ipynb` | Bayesian MMM, validation redesign, posterior diagnostics, and uncertainty analysis |
| `README.md` | Project summary, assumptions, checks, findings, limitations, and learnings |

## Key Learning

The biggest lesson was not just that Bayesian MMM is more advanced than regression.

The bigger lesson was:

> A model can be mathematically valid and leakage-safe while still being evaluated in the wrong business context.

The project followed a practical Bayesian mindset:

1. Start with reasonable assumptions.
2. Compare them against evidence.
3. Find where the setup fails.
4. Update the validation design.
5. Communicate uncertainty honestly.

Marginal ROAS Simulation

After fitting the hierarchical Bayesian MMM, I ran a posterior forward-pass simulation to estimate marginal ROAS. For each channel, I increased spend by 1%, rebuilt the lag/adstock and saturation features, and compared posterior revenue contribution against the baseline.

The simulation showed the strongest marginal return for Network TV, followed by Cable TV, while Digital had a positive but less certain return. Network TV had a median mROAS of 2.44, Cable TV 1.90, and Digital 1.16. Digital's lower credible bound fell below 1, suggesting it should be scaled more cautiously.

Final recommendation: prioritize small incremental budget increases toward Network TV and Cable TV, while using Digital selectively by show and validating larger changes through controlled experiments or additional scenario testing.
