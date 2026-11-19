# Model Selection

## The question

The model predicts whether a telecom customer will churn. Missing a customer who is likely to leave can result in losing a retention opportunity. Incorrectly identifying a customer as likely to churn can waste outreach time and discounts.

Therefore, I compare the models using overall F1, AUC, F1 across contract types, prediction latency, and model size.

## Candidates

All three candidates were trained and evaluated using the same dataset, `data/churn_v1.csv`, with the same data hash (`d1f47e3335d6`). All runs belong to the `churn` MLflow experiment and use the `local` batch.

| Candidate | Why I tried it | AUC | F1 | F1 per contract type | p95 latency (ms) | Size (KB) | MLflow run ID |
|---|---|---:|---:|---|---:|---:|---|
| `logreg_balanced` | Compact and interpretable baseline. Balanced class weights help address class imbalance. | 0.7968 | 0.5854 | Month-to-month: 0.6260; One-year: 0.5714; Two-year: 0.3158 | 15.5285 | 3.5479 | `bbd8f748fd4b4026bc19550dff8e0f94` |
| `random_forest_limited` | Tests non-linear feature interactions while limiting tree complexity. | 0.7631 | 0.5469 | Month-to-month: 0.5686; One-year: 0.4615; Two-year: 0.4615 | 17.9723 | 2532.9238 | `a5a33ce639e64d19a90f1a49d770ea5f` |
| `gradient_boosting_shallow` | Tests whether sequential shallow trees improve prediction quality. | 0.7783 | 0.4000 | Month-to-month: 0.4595; One-year: 0.0000; Two-year: 0.2222 | 10.0882 | 100.3477 | `40fdcab9b9b84fceb4d2ed1dfbff084e` |

## The champion

I selected `logreg_balanced` as the champion because it achieved the highest overall F1 score (0.5854) and the highest AUC (0.7968).

It also has the smallest model size (3.5479 KB), making it easier to store and deploy.

However, it is not the fastest candidate. `gradient_boosting_shallow` has the lowest p95 latency (10.0882 ms), while `random_forest_limited` has the highest latency (17.9723 ms).

An important limitation is that the champion performs less well on customers with two-year contracts, where its F1 is 0.3158. The random forest performs better on this group, with an F1 of 0.4615.

I would therefore select logistic regression as the overall champion, but investigate its performance on two-year-contract customers before a broad production rollout.

## My promotion rules

- **Latency budget (p95): 25 ms.** All three candidates are below this limit. The slowest candidate measured 17.9723 ms, leaving some room for additional deployment overhead.
- **Minimum F1 improvement to replace the champion: 0.0100.** A new candidate must improve F1 by at least one percentage point before replacing the current champion.

A replacement must also be checked for performance across contract types, model size, and latency.

## What I would check next

The two-year-contract group has the weakest F1 for the selected logistic regression model.

I would inspect false negatives and false positives for this group, check how many examples it contains, and investigate whether additional customer data could improve its predictions.

I would also monitor prediction latency and model performance after deployment to make sure the model continues to meet the promotion requirements.

## Reproducibility

All three candidate runs used:

- Dataset: `data/churn_v1.csv`
- Data SHA256: `d1f47e3335d6`
- MLflow experiment: `churn`
- Batch: `local`

The data hash helps identify the exact dataset version used in the experiments.