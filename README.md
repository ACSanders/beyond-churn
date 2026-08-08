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
4. Does targeting high-risk customers produce the same incremental value as targeting high-uplift customers?
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

The emphasis is not on building the largest possible collection of algorithms.

The goal is to understand how predictive performance, causal estimation, and decision quality differ.

## Central idea

A useful way to think about the project is as three related but distinct problems.

### Prediction

Estimate:

$$
P(\text{churn} \mid X)
$$

This answers:

> Who appears likely to churn?

### Causal response

Estimate a conditional treatment effect:

$$
\tau(x) = E[Y(1) - Y(0) \mid X = x]
$$

This answers:

> Whose outcome is expected to change because of treatment?

### Decision

Use estimated treatment response to construct a targeting policy:

$$
\pi(x)
$$

This answers:

> Given limited intervention capacity, whom should we actually treat?

These are related problems, but they are not the same problem.

A central theme of this project is therefore:

$$
\text{Prediction quality}
\neq
\text{Causal estimation quality}
\neq
\text{Decision quality}
$$

## Planned analyses

The project will initially compare several targeting strategies, including:

* random targeting
* targeting customers with the highest predicted churn risk
* targeting customers with the highest estimated treatment effect
* potentially targeting customers based on expected incremental economic value

Policies will be evaluated at multiple intervention capacities, such as targeting the top:

* 5%
* 10%
* 20%
* 30%
* 50%

of eligible customers.

One of the central comparisons will be:

> **If we can intervene on only a limited share of customers, is it better to target those most likely to churn or those most likely to respond to treatment?**

## Causal evaluation

Unlike a synthetic simulation, a real randomized experiment does not reveal both potential outcomes for an individual customer.

For customer (i), we observe either:

$$
Y_i(1)
$$

or:

$$
Y_i(0)
$$

but not both.

This means the true individual treatment effect cannot be directly observed.

The project will therefore focus on evaluation methods that are usable in real applied causal-ML settings, including:

* treatment/control differences across ranked customer groups
* uplift curves
* Qini curves
* AUUC
* out-of-sample treatment-effect evaluation
* policy value
* budget-constrained policy evaluation
* doubly robust policy evaluation

This is an intentional part of the project design rather than a limitation to be hidden.

## Planned modeling progression

The project will be developed incrementally so that each method is understood before more advanced techniques are introduced.

### Phase 1: Churn risk

Build predictive baselines for the question:

> Who is likely to churn?

Initial models:

* logistic regression
* CatBoost

Evaluation will include:

* ROC-AUC
* PR-AUC
* log loss
* Brier score
* calibration curves
* probability calibration diagnostics

The goal is not simply to maximize classifier performance, but to establish a strong predictive baseline against which causal targeting can later be compared.

### Phase 2: Causal fundamentals

Introduce the potential-outcomes framework and treatment-effect estimation.

The initial focus will be on concepts such as:

* average treatment effect
* conditional average treatment effect
* treatment and control response surfaces
* heterogeneous treatment effects

The first practical uplift model will likely be a T-Learner because it makes the underlying logic transparent.

A T-Learner estimates separate response models:

$$
\hat{\mu}_0(x)
==============

E[Y \mid X=x, T=0]
$$

and:

$$
\hat{\mu}_1(x)
==============

E[Y \mid X=x, T=1]
$$

Then estimated uplift is:

$$
\hat{\tau}(x)
=============

## \hat{\mu}_1(x)

\hat{\mu}_0(x)
$$

### Phase 3: Uplift evaluation

Once treatment effects can be estimated, the project will evaluate whether the ranking produced by those estimates is useful.

Potential analyses include:

* uplift by predicted-effect decile
* treatment/control differences within ranked groups
* uplift curves
* Qini curves
* AUUC
* top-k treatment strategies

The important question is not merely:

> Did the model predict treatment effects?

It is:

> Did the model rank customers in a way that produces more incremental retention when used for targeting?

### Phase 4: More advanced causal ML

After the uplift baseline is understood, the project may introduce more robust heterogeneous treatment-effect estimators.

Potential methods include:

* T-Learner
* DRLearner
* CausalForestDML

The purpose of adding these methods is to determine whether more sophisticated causal estimators materially improve treatment-effect ranking or policy value.

They will not be added solely for algorithmic complexity.

### Phase 5: Policy evaluation

Treatment-effect estimates will then be translated into actual intervention policies.

For example:

$$
\pi_k(x)
========

\begin{cases}
1, & \text{if customer } x \text{ is in the top } k% \text{ of the ranking} \
0, & \text{otherwise}
\end{cases}
$$

Policies may include:

* no intervention
* random targeting
* churn-risk targeting
* uplift targeting
* economic-value targeting

The central comparison will be between:

$$
\text{Random targeting}
$$

$$
\text{Churn-risk targeting}
$$

$$
\text{Uplift targeting}
$$

under identical intervention budgets.

### Phase 6: Doubly robust policy evaluation

Once basic policy evaluation is established, the project may add doubly robust approaches for estimating policy value.

The goal will be to evaluate targeting strategies using information from both:

* treatment assignment probabilities
* outcome models

This provides a more rigorous framework for estimating the incremental value of a learned policy on held-out experimental data.

## Data

The initial project is planned around a public churn-uplift dataset containing treatment and control observations.

The leading candidate is a public telecom churn-uplift dataset associated with Orange Belgium.

Before modeling begins, the project will document:

* dataset provenance
* licensing
* experimental design
* treatment definition
* outcome definition
* feature definitions
* sample size
* treatment/control balance
* class balance
* missingness
* evaluation constraints

No proprietary employer data, code, internal feature definitions, experiments, thresholds, business rules, or confidential model results are used in this repository.

## Why randomized treatment data matters

A standard churn dataset can help estimate:

$$
P(\text{churn} \mid X)
$$

but that does not identify whether an intervention caused a customer to stay.

With randomized treatment data, we can instead investigate the causal question:

$$
E[Y(1)-Y(0)\mid X=x]
$$

Randomization helps separate treatment response from ordinary differences between customers who happened to receive different actions.

This makes the dataset suitable for studying not only prediction, but also treatment targeting and policy evaluation.

## Risk is not persuadability

One of the project's most important analyses will compare:

* predicted churn risk
* estimated treatment effect

A high-risk customer may have little or no treatment response.

A moderate-risk customer may be highly responsive.

This means a strong churn classifier can still produce a poor intervention strategy if churn probability is treated as a proxy for persuadability.

A flagship visualization will eventually compare:

**Predicted churn risk**

against:

**Estimated treatment effect**

to examine where these rankings agree and where they diverge.

## From ranking to policy

Ranking customers is only useful if it improves decisions.

Suppose an organization can intervene on only 20% of customers.

The project will compare the incremental outcomes produced by selecting that 20% using different policies.

For example:

$$
\text{Top 20% by churn risk}
$$

versus:

$$
\text{Top 20% by estimated uplift}
$$

If the uplift-based policy generates a larger treatment/control improvement, that provides direct evidence that predictive risk and intervention value are different quantities.

## Economic decision-making

A later extension may incorporate customer value and intervention cost.

For customer (i), estimated incremental value could take the form:

$$
\widehat{\text{Incremental Value}}_i
====================================

\hat{\tau}_i V_i - C_i
$$

where:

* (\hat{\tau}_i) is estimated treatment effect
* (V_i) is the value of retaining the customer
* (C_i) is intervention cost

This creates a further distinction between:

* highest churn risk
* highest uplift
* highest expected economic value

Economic modeling will be introduced only after the causal targeting framework is established.

## Repository structure

The project will begin notebook-first for research and learning, while stable functionality will progressively move into reusable Python modules.

Planned structure:

```text
beyond-churn/
├── README.md
├── LICENSE
├── pyproject.toml
├── data/
├── notebooks/
│   ├── 01_data_understanding.ipynb
│   ├── 02_churn_prediction.ipynb
│   ├── 03_uplift_baselines.ipynb
│   ├── 04_causal_ml.ipynb
│   └── 05_policy_evaluation.ipynb
├── src/
│   └── beyond_churn/
├── tests/
├── artifacts/
└── .github/
    └── workflows/
```

The exact structure will evolve as the research stabilizes.

## Development philosophy

This project follows a few principles:

* understand methods before adding complexity
* start with strong, interpretable baselines
* distinguish prediction from causal inference
* distinguish causal estimation from policy quality
* use out-of-sample evaluation
* compare policies under identical treatment budgets
* build reusable Python only after research logic stabilizes
* add infrastructure when it solves a real problem
* prioritize scientific clarity over resume keywords

The project will not begin with:

* reinforcement learning
* neural networks
* elaborate cloud infrastructure
* large MLOps systems
* dashboards
* dozens of competing estimators

Those components will only be introduced if the research problem eventually justifies them.

## Planned technical stack

Likely tools include:

* Python
* pandas
* NumPy
* scikit-learn
* CatBoost
* EconML
* SciPy
* matplotlib
* pytest

Additional dependencies will be added only when they serve a clear methodological or engineering purpose.

## Status

🚧 **Work in progress**

Current priorities:

* [ ] Finalize the public dataset
* [ ] Verify dataset provenance and licensing
* [ ] Document treatment and outcome definitions
* [ ] Audit treatment/control balance
* [ ] Inspect class balance and missingness
* [ ] Establish train/test evaluation design
* [ ] Build predictive churn baselines
* [ ] Evaluate probability calibration
* [ ] Implement a T-Learner uplift baseline
* [ ] Evaluate uplift across ranked customer groups
* [ ] Implement uplift and Qini curves
* [ ] Compare churn-risk and uplift targeting
* [ ] Evaluate policies at multiple intervention budgets
* [ ] Explore DRLearner and CausalForestDML
* [ ] Add reusable modules and tests
* [ ] Document methodological conclusions and results

## Long-term goal

The goal of **Beyond Churn** is not to produce another churn-classification leaderboard.

It is to investigate a more consequential question:

> **If a model tells us who is likely to churn, does that actually tell us who we should intervene on?**

More broadly, the project aims to demonstrate why:

$$
\text{Prediction quality}
\neq
\text{Causal estimation quality}
\neq
\text{Decision quality}
$$

and how experimental data and causal machine learning can help bridge those gaps.
