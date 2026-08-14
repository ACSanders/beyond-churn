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
2. **Build and compare churn-risk models**
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

Builds a churn-risk model using untreated customers.

Models compared:

- Logistic Regression
- CatBoost
- XGBoost
- LightGBM

LightGBM performed best and was selected as the final churn-risk model.

Held-out performance:

| Metric | Result |
|---|---:|
| ROC AUC | 0.841 |
| Average Precision | 0.187 |
| Precision | 0.108 |
| Recall | 0.346 |
| F1 | 0.165 |
| F2 | 0.241 |

The final classification threshold was selected using out-of-fold training predictions with a minimum recall requirement of 25%.

The final LightGBM pipeline is saved in:

```text
artifacts/lightgbm_churn_model.joblib
```

with model metadata saved in:

```text
artifacts/lightgbm_churn_metadata.json
```

### 03 — Causal uplift modeling

[`notebooks/03_causal_uplift_modeling.ipynb`](notebooks/03_causal_uplift_modeling.ipynb)

Compares:

- LightGBM churn-risk targeting
- T-Learner
- DRLearner
- CausalForestDML
- random targeting

The churn-risk model continues to show strong ranking performance on the causal evaluation sample:

| Metric | Result |
|---|---:|
| ROC AUC | 0.831 |
| Average Precision | 0.173 |

The causal models produce very different treatment-effect estimates.

In the policy comparison, churn-risk targeting produced the highest observed retention uplift at every tested targeting level. The largest point estimate appeared when targeting only the top 5% of customers.

T-Learner produced smaller but consistently positive uplift. DRLearner showed some strong targeting results but was less stable. CausalForestDML produced negative observed uplift at most of the smaller targeting levels.

Random targeting stayed close to zero and served as a useful benchmark.

These results do **not** mean churn risk and treatment response are the same thing. They show that a causal model does not automatically create a better targeting policy.

The smaller targeting groups also contain relatively few customers, so the largest point estimates should be interpreted cautiously.

## Main takeaway

The core idea of the project is:

> **The customer most likely to churn is not necessarily the customer most likely to be saved.**

Prediction and causal targeting answer different questions.

A churn model estimates who is most at risk. A causal model estimates whose outcome may change because of treatment.

In this dataset, the churn-risk ranking happened to produce the strongest observed targeting results.

One key takeaway is that a good prediction model is not always the best decision model. Causal models also do not automatically produce better targeting. What matters is how well each approach performs for the actual decision we need to make.

## Repository structure

```text
beyond-churn/
├── README.md
├── LICENSE
├── environment.yml
├── artifacts/
│   ├── lightgbm_churn_model.joblib
│   └── lightgbm_churn_metadata.json
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
- LightGBM
- CatBoost
- XGBoost
- EconML
- OpenML
- matplotlib
- seaborn

## Status

The main notebook workflow is complete:

- [x] data audit
- [x] churn model comparison
- [x] saved LightGBM pipeline
- [x] T-Learner
- [x] DRLearner
- [x] CausalForestDML
- [x] targeting-policy comparison

Possible future work:

- policy-value estimation with uncertainty
- economic targeting using customer value and intervention cost
- moving stable code into reusable modules
