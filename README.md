# Beyond Churn

**From churn prediction to better retention decisions.**

Most churn projects ask:

> **Who is likely to churn?**

This project asks a second question:

> **Who is actually likely to respond to a retention treatment?**

Those are not always the same customers.

A high-risk customer may be unlikely to change behavior, while a moderate-risk customer may be more responsive to an intervention. The goal of this project is to compare those two ideas directly and then evaluate what the targeting results mean in practice.

## What this project does

The project uses a public randomized churn-uplift dataset from OpenML and moves through four stages:

1. **Understand the experiment and data**
2. **Build and compare churn-risk models**
3. **Compare churn-risk targeting with causal uplift models**
4. **Evaluate uncertainty and simple business value**

The main causal models are:

- T-Learner
- DRLearner
- CausalForestDML

The main decision question is:

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

The churn-risk model continued to rank customers well on the causal evaluation sample:

| Metric | Result |
|---|---:|
| ROC AUC | 0.831 |
| Average Precision | 0.173 |

In the policy comparison, churn-risk targeting produced the highest observed retention uplift at every tested targeting level.

The causal models produced different treatment-effect estimates and different targeting results. T-Learner was consistently positive, DRLearner was less stable, and CausalForestDML was more conservative.

This does **not** mean churn risk and treatment response are the same thing. It shows that a causal model does not automatically produce the best targeting policy.

The evaluation scores used for the final policy analysis are saved in:

```text
artifacts/causal_eval_results.csv
```

### 04 — Policy value and uncertainty

[`notebooks/04_policy_value_and_uncertainty.ipynb`](notebooks/04_policy_value_and_uncertainty.ipynb)

This notebook takes the targeting results from Notebook 03 and asks two practical questions:

- how uncertain are the policy estimates?
- what could those results mean in business terms?

Bootstrap confidence intervals show that the point estimates are noisy, especially for smaller targeting groups. All tested 95% confidence intervals include zero.

The notebook also translates uplift into expected retained customers per 1,000 targeted and uses a simple hypothetical value example to show how treatment cost and customer value can change the decision.

The business values are illustrative only. They are included to show how model results can be connected to a real decision.

## Main takeaway

The core idea of the project is:

> **The customer most likely to churn is not necessarily the customer most likely to be saved.**

Prediction and causal targeting answer different questions.

A churn model estimates who is most at risk. A causal model estimates whose outcome may change because of treatment.

In this dataset, the churn-risk ranking produced the strongest observed targeting results. But the final uncertainty analysis also shows why point estimates alone are not enough.

A useful retention policy needs to consider model performance, uncertainty, targeting scale, intervention cost, and the value of the outcome being changed.

## Repository structure

```text
beyond-churn/
├── README.md
├── LICENSE
├── environment.yml
├── artifacts/
│   ├── causal_eval_results.csv
│   ├── lightgbm_churn_model.joblib
│   └── lightgbm_churn_metadata.json
└── notebooks/
    ├── 01_data_understanding.ipynb
    ├── 02_churn_prediction.ipynb
    ├── 03_causal_uplift_modeling.ipynb
    └── 04_policy_value_and_uncertainty.ipynb
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
