# Beyond Churn

**From churn prediction to causal retention decisions.**

> **Work in progress:** This repository is under active development. The predictive modeling stage is complete, and the project is now moving into heterogeneous treatment-effect estimation, uplift modeling, and policy evaluation.

## Why this project exists

Most churn modeling projects ask:

> **Who is likely to churn?**

That is an important predictive question, but it is not necessarily the decision a retention team needs to make.

A customer with a very high probability of churn may be unlikely to respond to an intervention. Another customer with only moderate churn risk may be much more likely to change behavior because of treatment.

The customer most likely to churn is therefore not necessarily the customer most likely to be saved.

**Beyond Churn** explores the transition from predictive modeling to causal decision-making:

**Predict risk → Estimate treatment response → Evaluate targeting policies → Make retention decisions**

The central idea is simple:

> **Risk is not the same as persuadability.**

---

## Core questions

This project is organized around several related questions:

1. Who is likely to churn without intervention?
2. Whose outcome is likely to change because of a retention intervention?
3. How different are predicted churn risk and estimated treatment response?
4. Does targeting high-risk customers produce the same incremental value as targeting high-uplift customers?
5. How should causal targeting strategies be evaluated when individual counterfactual outcomes cannot be observed?
6. How does limited intervention capacity change which customers should be targeted?

---

## Prediction, causality, and decisions

A useful way to think about the project is as three related but distinct problems.

### 1. Prediction

Estimate churn risk under no intervention:

$$
P(Y=1 \mid X, T=0)
$$

where:

- $Y=1$ indicates churn
- $X$ represents customer features
- $T=0$ represents the control condition

This answers:

> **Who is most likely to churn without intervention?**

The predictive model in this project is intentionally trained on the control group so that its output represents untreated churn risk rather than churn risk after exposure to the retention intervention.

### 2. Causal response

Estimate the conditional average treatment effect:

```math
\tau(x) = E[Y(1)-Y(0)\mid X=x]
```

This answers:

> **How does treatment change the expected outcome for customers like this one?**

Because $Y=1$ represents churn, a **negative treatment effect is favorable**:

```math
\tau(x) < 0
```

means treatment is estimated to reduce churn.

For decision-making, it is often convenient to express this as expected retention uplift:

```math
u(x) = -\tau(x)
```

so that larger positive values correspond to larger estimated reductions in churn.

### 3. Decision

Use estimated treatment response to construct a targeting policy:

$$
\pi(x) \in \{0,1\}
$$

where $\pi(x)=1$ means the customer is selected for treatment.

This answers:

> **Given limited intervention capacity, whom should we actually treat?**

These are related problems, but they are not the same problem.

A central theme of this project is therefore:

$$
\text{Prediction quality}
\neq
\text{Causal estimation quality}
\neq
\text{Decision quality}
$$

---

## Data

The project uses the public **`churn-uplift-mlg`** dataset available through OpenML.

- **OpenML dataset ID:** `45580`
- **Observations:** 11,896
- **Predictor columns:** 178 before removing one constant feature
- **Treatment indicator:** `t`
- **Outcome:** `y`
- **Outcome interpretation:** `y = 1` indicates churn

The dataset contains both treatment and control observations and is designed for churn-uplift analysis.

The raw dataset is loaded reproducibly through the OpenML Python API rather than committed directly to the repository.

### Dataset audit

The initial data-understanding analysis found:

- 9,010 treated observations
- 2,886 control observations
- 408 churn outcomes
- overall churn rate of approximately 3.43%
- no missing values
- one globally constant predictor, `FACTOR3`, removed before modeling

Observed churn rates were:

- **Control:** 3.64%
- **Treatment:** 3.36%

The unadjusted treatment-control risk difference was approximately:

$$
-0.28 \text{ percentage points}
$$

with a 95% confidence interval spanning zero.

The weak average treatment effect is useful for this project because a small population-level effect does not imply that treatment effects are homogeneous. A central question is whether meaningful treatment-response heterogeneity exists beneath the average.

No proprietary employer data, code, internal feature definitions, experiments, thresholds, business rules, or confidential model results are used in this repository.

---

## Why randomized treatment data matters

A standard churn dataset can help estimate:

$$
P(Y=1 \mid X)
$$

but it cannot by itself determine whether a retention intervention **caused** a customer to stay.

With randomized treatment data, we can instead investigate:

$$
E[Y(1)-Y(0)\mid X=x]
$$

Random treatment assignment helps separate treatment response from ordinary customer differences that might otherwise confound treatment selection.

This makes the dataset suitable for studying:

- predictive churn risk
- average treatment effects
- heterogeneous treatment effects
- uplift ranking
- targeting policies
- policy value

---

## Completed work

### Notebook 01 — Data understanding

[`notebooks/01_data_understanding.ipynb`](notebooks/01_data_understanding.ipynb)

The first notebook establishes the experimental and data foundation for the project.

It includes:

- reproducible OpenML loading
- treatment and outcome auditing
- data-integrity checks
- missingness analysis
- class-balance analysis
- treatment/control balance diagnostics
- standardized mean differences
- categorical balance checks
- exploratory analysis of anonymized predictors
- unadjusted experimental average treatment-effect estimation

The purpose of this notebook is not to build a model. It establishes what the experiment contains and what causal questions the data can support.

---

### Notebook 02 — Churn prediction

[`notebooks/02_churn_prediction.ipynb`](notebooks/02_churn_prediction.ipynb)

The second notebook establishes the predictive-risk component of the framework.

The question is:

> **Who is most likely to churn without intervention?**

To preserve that interpretation, the predictive model is trained using **control-group observations only**.

Two predictive approaches were evaluated:

- logistic regression
- CatBoost

Model development uses:

- stratified train/test splitting
- 5-fold cross-validation within the training data
- out-of-fold predictions for model comparison
- average precision for hyperparameter selection
- probability-quality diagnostics
- threshold selection using training OOF predictions
- a final untouched test set

### Logistic regression

The logistic model was additionally evaluated with probability calibration because its uncalibrated probabilities were highly polarized.

Sigmoid calibration substantially improved:

- Brier score
- log loss
- probability reliability

while preserving most of the model's ranking performance.

### CatBoost

CatBoost was selected as the final churn-risk model.

Final held-out test performance:

| Metric | Result |
|---|---:|
| ROC-AUC | **0.881** |
| Average Precision | **0.184** |
| Brier Score | **0.0327** |
| Log Loss | **0.1263** |

Because churn prevalence in the control population is only about 3.6%, average precision is particularly informative for this problem.

### Operating threshold

A classification threshold was selected using **out-of-fold training predictions**, before evaluating the final test set.

The selected threshold was:

$$
0.05
$$

At that frozen threshold, test performance was:

| Metric | Result |
|---|---:|
| Precision | **15.8%** |
| Recall | **57.7%** |
| F1 | **0.248** |
| F2 | **0.377** |

The threshold intentionally emphasizes recall because the predictive model is treated as a broad risk-screening layer rather than the final treatment decision.

Later causal models will determine which customers are actually expected to benefit from intervention.

### Model interpretation

The predictive notebook also includes:

- native CatBoost feature importance
- SHAP analysis
- confusion matrix
- precision-recall analysis
- predicted-risk distributions by observed outcome

Because the predictor variables are anonymized, these analyses can reveal predictive importance and directionality but cannot support strong domain interpretations of individual features.

---

## Saved predictive model

The final churn-risk model is saved using CatBoost's native model format:

```text
artifacts/catboost_churn_model.cbm
```

Associated model metadata is stored separately:

```text
artifacts/catboost_churn_metadata.json
```

The metadata includes:

- selected classification threshold
- categorical feature names
- model feature schema
- selected hyperparameters
- final test metrics

The saved CatBoost artifact was reloaded and verified to reproduce the same test probabilities as the in-memory model.

This allows later notebooks to reuse the predictive-risk model without rerunning the full hyperparameter search.

---

## Current work — Causal uplift modeling

### Notebook 03 — Causal uplift modeling

[`notebooks/03_causal_uplift_modeling.ipynb`](notebooks/03_causal_uplift_modeling.ipynb)

The project is now moving from prediction to treatment-response estimation.

The central question becomes:

> **Whose churn outcome is most likely to change because of treatment?**

The notebook has been initialized with:

- project overview
- causal-modeling imports
- reproducible OpenML data loading
- feature setup
- loading of the previously trained CatBoost churn-risk model

The next stages will establish a causal train/evaluation design and begin with transparent heterogeneous treatment-effect baselines before introducing more advanced estimators.

---

## Planned causal modeling progression

The causal portion of the project will be developed incrementally so that the logic of each estimator is understood before additional complexity is introduced.

### Phase 1 — Experimental baseline

Start from the randomized experiment itself.

Estimate and interpret:

- treatment-group outcome rate
- control-group outcome rate
- average treatment effect
- treatment-effect uncertainty

This establishes the population-level causal benchmark before modeling heterogeneity.

---

### Phase 2 — T-Learner

The first heterogeneous treatment-effect model will be a T-Learner.

A T-Learner fits separate response models for the control and treatment groups:

```math
\hat{\mu}_0(x) = \widehat{E}[Y \mid X=x, T=0]
```

and

```math
\hat{\mu}_1(x) = \widehat{E}[Y \mid X=x, T=1]
```

The estimated conditional treatment effect is then:

```math
\hat{\tau}(x) = \hat{\mu}_1(x) - \hat{\mu}_0(x)
```

Because the outcome is churn, beneficial treatment response corresponds to:

```math
\hat{\tau}(x) < 0
```

Equivalently, estimated retention uplift can be defined as:

```math
\hat{u}(x) = \hat{\mu}_0(x) - \hat{\mu}_1(x)
```

where larger values indicate a larger predicted reduction in churn.

The T-Learner is intentionally introduced first because its logic is easy to inspect and explain.

---

### Phase 3 — Uplift evaluation

Individual treatment effects cannot be directly observed in real experimental data because each customer experiences only one treatment condition.

For customer $i$, we observe either:

$$
Y_i(1)
$$

or

$$
Y_i(0)
$$

but never both.

Therefore, causal models cannot be evaluated using ordinary individual-level prediction accuracy against a known treatment-effect label.

Instead, the project will evaluate whether estimated treatment effects produce useful **rankings**.

Planned evaluation includes:

- treatment/control differences across predicted-effect groups
- uplift by decile
- cumulative uplift curves
- Qini curves
- AUUC
- top-$k$ targeting performance
- held-out policy evaluation

The central question is:

> **Does the model rank customers in a way that produces more incremental retention when used for treatment targeting?**

---

### Phase 4 — Advanced heterogeneous treatment-effect models

After establishing the T-Learner baseline, the project will compare more advanced causal estimators.

Planned methods include:

- T-Learner
- DRLearner
- CausalForestDML

Potential additional methods may be explored when they add meaningful methodological value.

The purpose is not to accumulate algorithms.

The purpose is to determine whether stronger causal estimators improve:

- treatment-effect ranking
- uplift concentration
- policy value
- robustness

---

## Risk is not persuadability

One of the project's flagship comparisons will be between:

- predicted untreated churn risk
- estimated treatment response

The churn-risk model estimates:

$$
P(Y=1 \mid X, T=0)
$$

while the causal model estimates:

$$
E[Y(1)-Y(0)\mid X=x]
$$

These quantities answer different questions.

A high-risk customer may have little treatment response.

A moderate-risk customer may be highly responsive.

Therefore, even a strong churn classifier can produce a poor retention strategy if predicted churn probability is treated as a proxy for persuadability.

A planned visualization will compare:

**Predicted churn risk**

against:

**Estimated retention uplift**

to examine where the rankings agree and where they diverge.

---

## From ranking to policy

Ranking customers is only useful if it improves decisions.

Suppose an organization can intervene on only a fraction $k$ of eligible customers.

A targeting policy can be written as:

$$
\pi_k(x)
=
\begin{cases}
1, & \text{if } x \text{ is selected within the treatment budget} \\
0, & \text{otherwise}
\end{cases}
$$

The project will compare policies such as:

- no intervention
- random targeting
- churn-risk targeting
- uplift targeting
- eventually, economic-value targeting

under identical treatment budgets.

Planned intervention capacities include:

- 5%
- 10%
- 20%
- 30%
- 50%

One of the central comparisons will therefore be:

> **If we can intervene on only a limited share of customers, is it better to target those most likely to churn or those most likely to respond to treatment?**

---

## Policy evaluation

The project will eventually compare:

$$
\text{Random targeting}
$$

vs.

$$
\text{Churn-risk targeting}
$$

vs.

$$
\text{Uplift targeting}
$$

under identical intervention budgets.

Policy value will be estimated using held-out randomized experimental data.

Once basic policy evaluation is established, the project will explore **doubly robust policy evaluation**, combining information from:

- treatment assignment probabilities
- outcome models

The goal is to evaluate treatment policies using methods that remain realistic for applied causal machine learning, where individual counterfactual outcomes are unavailable.

---

## Economic decision-making

A later extension may incorporate customer value and intervention cost.

For customer $i$, estimated incremental economic value could take the form:

$$
\widehat{\text{Incremental Value}}_i
=
\hat{u}_i V_i - C_i
$$

where:

- $\hat{u}_i$ is estimated retention uplift
- $V_i$ is the estimated value of retaining the customer
- $C_i$ is intervention cost

This creates an additional distinction between:

- highest churn risk
- highest treatment response
- highest expected economic value

Economic optimization will be introduced only after the causal targeting framework is established.

---

## Repository structure

The project currently follows a notebook-first research workflow:

```text
beyond-churn/
├── README.md
├── LICENSE
├── environment.yml
├── .gitignore
├── artifacts/
│   ├── catboost_churn_model.cbm
│   └── catboost_churn_metadata.json
├── data/
│   ├── raw/
│   └── processed/
└── notebooks/
    ├── 01_data_understanding.ipynb
    ├── 02_churn_prediction.ipynb
    └── 03_causal_uplift_modeling.ipynb
```

Raw and processed local data are excluded from version control.

The project will remain notebook-first while the research logic is evolving.

Once the methodology stabilizes, reusable functionality may be moved into a package structure with components such as:

```text
src/
tests/
.github/workflows/
```

This progression is intentional: research logic will be understood and validated before unnecessary software abstractions are introduced.

---

## Environment

The working environment is reproducible through:

```text
environment.yml
```

Create the environment with:

```bash
conda env create -f environment.yml
conda activate beyond-churn
```

The current causal and machine-learning stack includes:

- Python 3.12
- NumPy
- pandas
- SciPy
- scikit-learn
- statsmodels
- CatBoost
- SHAP
- OpenML
- EconML
- DoWhy
- CausalML
- LightGBM
- XGBoost
- matplotlib
- seaborn
- JupyterLab

The environment file records the tested package versions used by the project.

---

## Development philosophy

This project follows several principles:

- understand methods before adding complexity
- start with strong, interpretable baselines
- distinguish prediction from causal inference
- distinguish causal estimation from policy quality
- preserve held-out evaluation data
- use out-of-sample predictions where appropriate
- compare treatment policies under identical budgets
- use randomized experimental information whenever possible
- build reusable Python only after research logic stabilizes
- add infrastructure when it solves a real problem
- prioritize scientific clarity over resume keywords

The project does **not** begin with:

- reinforcement learning
- neural networks
- elaborate cloud infrastructure
- large MLOps systems
- dashboards
- dozens of competing estimators

Those components will only be introduced if the research problem eventually justifies them.

---

## Current status

### Completed

- [x] Select public randomized-treatment churn dataset
- [x] Build reproducible OpenML data loader
- [x] Audit treatment and outcome distributions
- [x] Inspect missingness and feature integrity
- [x] Evaluate treatment/control covariate balance
- [x] Estimate the experimental average treatment effect
- [x] Establish control-only predictive modeling design
- [x] Build logistic-regression churn baseline
- [x] Evaluate probability calibration
- [x] Tune and evaluate CatBoost
- [x] Generate out-of-fold training predictions
- [x] Select predictive operating threshold without using test data
- [x] Evaluate final churn model on untouched test data
- [x] Add CatBoost feature importance and SHAP analysis
- [x] Save and verify reusable CatBoost model artifact
- [x] Save model metadata
- [x] Add reproducible `environment.yml`
- [x] Initialize causal uplift modeling notebook
- [x] Load saved churn-risk model into causal workflow

### In progress / next

- [ ] Define causal train/evaluation split
- [ ] Re-establish causal estimand and sign convention
- [ ] Implement T-Learner baseline
- [ ] Estimate heterogeneous treatment effects
- [ ] Evaluate uplift across ranked customer groups
- [ ] Implement cumulative uplift curves
- [ ] Implement Qini evaluation
- [ ] Calculate AUUC
- [ ] Compare churn-risk and uplift rankings
- [ ] Evaluate treatment policies at multiple intervention budgets
- [ ] Implement DRLearner
- [ ] Implement CausalForestDML
- [ ] Add doubly robust policy evaluation
- [ ] Compare causal estimators by decision value
- [ ] Move stable functionality into reusable modules
- [ ] Add tests for reusable components
- [ ] Document final methodological conclusions

---

## Long-term goal

The goal of **Beyond Churn** is not to produce another churn-classification leaderboard.

It is to investigate a more consequential question:

> **If a model tells us who is likely to churn, does that actually tell us who we should intervene on?**

The project is designed to show why:

$$
\text{Prediction quality}
\neq
\text{Causal estimation quality}
\neq
\text{Decision quality}
$$

and how randomized experiments, heterogeneous treatment-effect estimation, uplift modeling, and policy evaluation can bridge those gaps.

The ultimate objective is to move from:

> **Who will churn?**

to:

> **Who can we actually help by intervening?**
