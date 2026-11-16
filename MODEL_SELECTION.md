# Model selection

## The question
This model predicts whether a customer is likely to leave the service (churn).
A wrong prediction may cause the company to miss customers who need retention support.

## Candidates

| Candidate | Why I tried it | AUC | F1 | F1 per slice (lowest) | p95 latency (ms) | Size (KB) | MLflow run ID |
|---|---|---:|---:|---:|---:|---:|---|
| Logistic Regression | Simple and lightweight baseline | 0.7982 | 0.4948 | 0.2500 | 2.6264 | 3.55 | dd6046f668fe427d80854c21614ab742 |
| Random Forest | To test a tree-based ensemble | 0.7589 | 0.4536 | 0.2222 | 10.0772 | 3160.10 | 141f1a07eebd469caf5e088bba3910ef |
| Gradient Boosting | To test boosting performance | 0.7476 | 0.4167 | 0.2222 | 2.6897 | 141.64 | b5dd14fab78944f78137ddb5099ac48f |

## The champion
I selected Logistic Regression as the champion because it achieved the highest overall F1 (0.4948) and AUC (0.7982). It also had the smallest model size (3.55 KB) and the lowest measured p95 latency (2.63 ms).

I accepted that its lowest slice F1 was only 0.25, which indicates weaker performance for some contract groups. Random Forest and Gradient Boosting did not achieve a better overall F1.

## My promotion rules
- Latency budget (p95): 25 ms, because the prediction API should respond quickly.
- Minimum gain to replace the champion (F1 points): 0.005, to avoid replacing the current model for a very small improvement.

## What I would check next
I would investigate the low F1 scores for one-year and two-year contract customers. I would also monitor changes in customer data and model performance after deployment.
