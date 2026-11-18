# Model selection

The candidate runs below use the `retry1` batch in the `churn` MLflow experiment. They use the same train/test split and report F1 separately for each contract type.

## The question
The model predicts whether a telecom customer will churn. Missing a customer who is about to leave can lose a retention opportunity; incorrectly flagging a customer can waste outreach time and discounts.

## Candidates
| Candidate | Why I tried it | AUC | F1 | F1 per slice (lowest) | p95 latency (ms) | Size (KB) | MLflow run ID |
|---|---|---:|---:|---|---:|---:|---|
| `logreg_balanced` | Compact, interpretable baseline; balanced class weights can help detect churners. | 0.7968 | 0.5854 | month-to-month 0.6260; one-year 0.5714; two-year 0.3158 (**lowest: 0.3158**) | 2.5406 | 3.5479 | `0322c905f758424cb7343c3363df46d4` |
| `random_forest_limited` | Tests non-linear feature interactions; depth and leaf limits control complexity. | 0.7631 | 0.5469 | month-to-month 0.5686; one-year 0.4615; two-year 0.4615 (**lowest: 0.4615**) | 10.2105 | 2532.9238 | `b8525071f2404269843fcfa3bc8a05c3` |
| `gradient_boosting_shallow` | Tests whether sequential shallow trees improve quality with a small model. | 0.7783 | 0.4000 | month-to-month 0.4595; one-year 0.0000; two-year 0.2222 (**lowest: 0.0000**) | 2.7105 | 100.3477 | `239a9cee44484fe499f67f3561a6a19e` |

## The champion
I chose `logreg_balanced` because it has the highest overall F1 (0.5854) and AUC (0.7968), while also being the fastest (2.5406 ms p95) and smallest (3.5479 KB). I accept that its two-year-contract F1 (0.3158) is lower than the random forest's 0.4615. This is a meaningful customer-group risk, so I would review two-year-contract errors and collect more evidence before a broad production rollout.

## My promotion rules
- Latency budget (p95): 25 ms, because the slowest measured candidate was 10.2105 ms on this laptop and the extra margin allows for a slower CI runner.
- Minimum gain to replace the champion (F1 points): 0.0100 (one percentage point), because a tiny change should not replace the current model without a meaningful measured improvement.

## What I would check next
The two-year-contract slice has the weakest F1 for the selected model. I would inspect its false negatives and false positives, check the number of test examples in that slice, and re-evaluate after collecting newer customer data.
