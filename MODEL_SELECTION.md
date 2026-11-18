# Model selection

## The question
This model predicts whether a customer is likely to leave the service (churn).
A wrong prediction may cause the company to miss customers who need retention support, resulting in customer churn and revenue loss.

## Candidates

| Candidate | Why I tried it | AUC | F1 | F1 per slice (lowest) | p95 latency (ms) | Size (KB) | MLflow run ID |
|---|---|---:|---:|---:|---:|---:|---|
| Logistic Regression | Simple and lightweight baseline linear classifier | 0.7982 | 0.4948 | 0.2500 | 20.17 | 3.55 | dc6d6910113e461abbfa07affe6367a2 |
| Random Forest (n=200, depth=8) | Test ensemble tree non-linearities and interactions | 0.7589 | 0.4536 | 0.2222 | 105.50 | 3160.13 | 8ccfdbfe57054019aec6bf136f3128fa |
| Gradient Boosting (default) | Test sequential boosting performance | 0.7476 | 0.4167 | 0.2222 | 20.05 | 141.68 | 79d6c47d75fc4029b2b4afee1a8714db |

## The champion
I selected **Logistic Regression** as the champion because:
1. It achieved the highest overall F1 score (0.4948) and the highest ROC-AUC (0.7982) among all candidates.
2. It exhibits the fastest inference speed (p95 latency of 20.17 ms), easily satisfying the service latency budget. In contrast, Random Forest suffered severe latency degradation (105.50 ms), violating production SLA.
3. It has the smallest memory and storage footprint (3.55 KB compared to 3.16 MB for Random Forest and 141.68 KB for Gradient Boosting), ensuring fast container startup and low memory overhead in deployment.

### Trade-offs Accepted
I accepted that its lowest slice F1 (on one-year and two-year contract customers) is 0.2500. While low, it is still higher than the weakest slice F1 of both Random Forest (0.2222) and Gradient Boosting (0.2222). None of the complex tree-based ensembles improved slice performance, while both suffered from lower overall F1 and higher latency/size.

## My promotion rules
- **Latency budget (p95)**: `30.0 ms`. Real-time customer service endpoints require responsive SLAs (< 30 ms) to avoid blocking user-facing portals. Random Forest (105.5 ms) was disqualified by this rule.
- **Minimum gain to replace champion (F1 points)**: `0.005` (0.5% F1 gain). Retraining and replacing a production model incurs operational churn, cache invalidation, and deployment risk; a marginal gain (< 0.005) does not justify a deployment.

## What I would check next
1. Investigate feature engineering and threshold tuning specifically for one-year and two-year contracts to improve slice F1.
2. Monitor input data drift and prediction probability drift using Prometheus metrics (`/metrics`) to detect customer behavior shifts post-deployment.
