# Beyond Churn

**From churn prediction to causal retention decisions.**

> **Work in progress:** This repository is under active development. The methodology, experiments, results, and project structure will evolve as the project progresses.

## Why this project exists

Most churn modeling projects ask:

> **Who is likely to churn?**

That is an important predictive question, but it is not necessarily the decision a retention team needs to make.

A customer with a very high probability of churn may be unlikely to respond to an intervention. Another customer with only moderate churn risk may be much more likely to change behavior because of treatment.

The customer most likely to churn is therefore not necessarily the customer most likely to be saved.

**Beyond Churn** explores the transition from predictive modeling to causal decision-making:

**Predict risk → Estimate treatment response → Evaluate targeting policies → Make retention decisions**

## Core questions

This project is being developed around several related questions:

1. Who is likely to churn?
2. Whose outcome is likely to change because of a retention intervention?
3. How different are churn risk and estimated treatment response?
4. Does targeting high-risk customers produce the same value as targeting high-uplift customers?
5. How should targeting strategies be evaluated when individual counterfactual outcomes cannot be observed?
6. How does intervention capacity affect the optimal targeting policy?

## Project direction

The initial version will use public randomized-treatment churn data to compare predictive churn modeling with heterogeneous treatment-effect and uplift modeling.

The project will progressively explore:

* churn prediction
* probability calibration
* randomized treatment/control analysis
* heterogeneous treatment effects
* uplift modeling
* meta-learners
* doubly robust estimation
* causal forests
* uplift and Qini evaluation
* treatment-effect ranking
* policy evaluation
* budget-constrained intervention strategies
* causal decision-making

The emphasis is not on building the largest collection of algorithms.

The goal is to understand how predictive performance, causal estimation, and decision quality differ.

## Central idea

A useful way to think about the project is:

### Prediction

Estimate:

[
P(\text{churn} \mid X)
]

This answers:

> Who appears likely to churn?

### Causal response

Estimate a conditional treatment effect:

[
\tau(x)
=======

E[Y(1)-Y(0)\mid X=x]
]

This answers:

> Whose outcome is expected to change because of treatment?

### Decision

Use estimated treatment response to construct a targeting policy:

[
\pi(x)
]

This answers:

> Given limited intervention capacity, whom should we actually treat?

These are related problems, but they are not the same problem.

A central theme of this project is therefore:

[
\text{Prediction quality}
\neq
\text{Causal estimation quality}
\neq
\text{Decision quality}
]

## Planned analyses

The project will initially compare several targeting strategies, including:

* random targeting
* targeting customers with the highest predicted churn risk
* targeting customers with the highest estimated treatment effect
* potentially targeting customers based on expected incremental economic value

Policies will be evaluated at multiple intervention capacities such as the top:

* 5%
* 10%
* 20%
* 30%
* 50%

of eligible customers.

## Causal evaluation

Unlike a synthetic simulation, a real experiment does not reveal both potential outcomes for an individual customer.

For customer (i), we observe either:

[
Y_i(1)
]

or:

[
Y_i(0)
]

but not both.

The project will therefore focus on evaluation methods that are usable in real applied causal-ML settings, including:

* treatment/control differences across ranked customer groups
* uplift curves
* Qini curves
* AUUC
* out-of-sample treatment-effect evaluation
* policy value
* doubly robust policy evaluation

This is an intentional part of the project design rather than a limitation to be hidden.

## Planned modeling progression

The project will be developed incrementally.

### Phase 1 — Churn risk

Initial predictive baselines:

* logistic regression
* CatBoost

Evaluation will include discrimination and probability calibration.

### Phase 2 — Treatment effects

Begin with interpretable causal baselines such as a T-Learner before progressing to more advanced estimators.

Potential methods include:

* T-Learner
* DRLearner
* CausalForestDML

### Phase 3 — Uplift evaluation

Evaluate whether customers ranked as highly responsive actually exhibit stronger incremental treatment effects in held-out experimental data.

### Phase 4 — Policy evaluation

Translate predictions into intervention policies and compare their estimated incremental outcomes under constrained treatment budgets.

## Data

The initial project is planned around a public churn-uplift dataset containing treatment and control observations.

The dataset selection, provenance, licensing, treatment definition, outcome definition, and experimental structure will be documented here once the initial data audit is complete.

No proprietary employer data, code, features, experiments, business rules, or internal model results are used in this repository.

## Repository structure

The project will begin notebook-first for research and learning, while stable functionality will progressively move into reusable Python modules.

Planned structure:

```text
beyond-churn/
├── README.md
├── LICENSE
├── pyproject.toml
├── notebooks/
│   ├── 01_data_understanding.ipynb
│   ├── 02_churn_prediction.ipynb
│   ├── 03_uplift_baselines.ipynb
│   ├── 04_causal_ml.ipynb
│   └── 05_policy_evaluation.ipynb
├── src/
│   └── beyond_churn/
├── tests/
└── artifacts/
```

The exact structure will evolve with the project.

## Development philosophy

This project follows a few principles:

* understand methods before adding complexity
* start with strong, interpretable baselines
* separate prediction from causal inference
* evaluate decisions rather than only models
* prefer out-of-sample evaluation
* build reusable Python only after research logic stabilizes
* add infrastructure when it solves a real problem
* prioritize scientific clarity over resume keywords

The project will not begin with reinforcement learning, large neural networks, elaborate MLOps infrastructure, or a dashboard.

Those components will only be introduced if the research problem eventually justifies them.

## Status

🚧 **Work in progress**

Current priorities:

* [ ] Finalize the public dataset
* [ ] Document the treatment and outcome definitions
* [ ] Audit treatment/control balance
* [ ] Establish the train/test evaluation design
* [ ] Build predictive churn baselines
* [ ] Implement the first uplift baseline
* [ ] Compare churn-risk and uplift rankings
* [ ] Build policy-evaluation metrics
* [ ] Add reusable modules and tests
* [ ] Document results and methodological conclusions

## Long-term goal

The goal of Beyond Churn is not to produce another churn-classification leaderboard.

It is to investigate a more consequential question:

> **If a model tells us who is likely to churn, does that actually tell us who we should intervene on?**
