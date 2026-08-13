# Beyond Churn

**From churn prediction to better retention decisions.**

Most churn projects ask:

> **Who is likely to churn?**

This project asks a second question:

> **Who is actually likely to respond to a retention treatment?**

Those are not always the same customers.

A high-risk customer may be unlikely to change behavior, while a moderate-risk customer may be much more responsive to an intervention. The goal of this project is to compare those two ideas directly.

## What this project does

The project uses a public randomized churn-uplift dataset from OpenML and moves through three stages:

1. **Understand the experiment and data**
2. **Build a churn-risk model**
3. **Compare churn-risk targeting with causal uplift models**

The main causal models are:

- T-Learner
- DRLearner
- CausalForestDML

The final comparison asks a practical question:

> **If we can only target part of the customer base, which ranking gives the strongest observed retention uplift?**

## Data

Public dataset:

- OpenML ID: `45580`
- 11,896 observations
- treatment indicator: `t`
- outcome: `y`
- `y = 1` means churn

The dataset contains randomized treatment and control groups, which makes it useful for studying both churn prediction and treatment response.

## Notebooks

### 01 — Data understanding

[`notebooks/01_data_understanding.ipynb`](notebooks/01_data_understanding.ipynb)

Covers:

- treatment and outcome distributions
- missing values and feature checks
- treatment/control balance
- average treatment effect

### 02 — Churn prediction

[`notebooks/02_churn_prediction.ipynb`](notebooks/02_churn_prediction.ipynb)

Builds a control-only churn-risk model.

Models tested:

- Logistic Regression
- CatBoost

CatBoost was selected as the final predictive model.

Held-out performance:

| Metric | Result |
|---|---:|
| ROC AUC | 0.881 |
| Average Precision | 0.184 |
| Brier Score | 0.0327 |
| Log Loss | 0.1263 |

The final CatBoost model is saved in:

```text
artifacts/catboost_churn_model.cbm
```

### 03 — Causal uplift modeling

[`notebooks/03_causal_uplift_modeling.ipynb`](notebooks/03_causal_uplift_modeling.ipynb)

Compares:

- churn-risk targeting
- T-Learner
- DRLearner
- CausalForestDML
- random targeting

The main result is that the causal models produce very different treatment-effect estimates, and the most advanced causal model does not automatically produce the best targeting policy.

In this dataset, churn-risk targeting produced the strongest observed point estimates across the tested targeting levels, while DRLearner generally performed better than the basic T-Learner.

That does **not** mean churn risk and treatment response are the same thing. It shows that causal modeling quality and policy quality still need to be evaluated rather than assumed.

## Main takeaway

The core lesson from this analysis and data set is:

> **The customer most likely to churn is not necessarily the customer most likely to be saved.**

Prediction and causal targeting answer different questions.

A strong churn model can tell us who is at risk. A causal model tries to tell us whose outcome might actually change because of treatment.

The interesting and useful part is comparing both approaches on the same held-out experiment.

## Repository structure

```text
beyond-churn/
├── README.md
├── LICENSE
├── environment.yml
├── artifacts/
│   ├── catboost_churn_model.cbm
│   └── catboost_churn_metadata.json
└── notebooks/
    ├── 01_data_understanding.ipynb
    ├── 02_churn_prediction.ipynb
    └── 03_causal_uplift_modeling.ipynb
```

## Environment

Create the environment with:

```bash
conda env create -f environment.yml
conda activate beyond-churn
```

Main libraries include:

- pandas
- scikit-learn
- CatBoost
- XGBoost
- EconML
- OpenML
- matplotlib
- seaborn

## Status

The main notebook workflow is complete:

- [x] data audit
- [x] churn prediction
- [x] saved CatBoost model
- [x] T-Learner
- [x] DRLearner
- [x] CausalForestDML
- [x] targeting-policy comparison

Possible future work:

- policy-value estimation with uncertainty
- economic targeting using customer value and intervention cost
- moving stable code into reusable modules
