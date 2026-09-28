# Personal-Loan Offer Amount Model · TVS Credit EPIC 8

**Analytics track, Case Study 2: Offer Amount Generation for Personal Loans**

*Affordability, not just eligibility.* A redesign of how a lender sizes pre-approved
personal-loan offers: anchor the amount on what each customer has already proved
they can repay, then modulate it by risk.

![Python](https://img.shields.io/badge/Python-LightGBM%20%7C%20scikit--learn-3776AB)
![Metric](https://img.shields.io/badge/brief%20metric-89.6%20vs%2060.0-2ea44f)
![Holdout AUC](https://img.shields.io/badge/holdout%20AUC-0.624-555)

---

## The problem

Customers were told their pre-approved amount was "not enough" and walked away,
while the largest offers leaked to other lenders. The brief: a better offer logic,
higher for lower-risk customers, lower or unchanged for higher-risk ones.

**Data:** 157,557 customer records × 128 fields (bureau history, on-us products,
enquiries, EMI history, offer, disbursal and 6-month delinquency). The 10 fields
the data dictionary bars from loan logic were removed before any modelling.

## Diagnosis

| Finding | Evidence |
|---|---|
| The grid prices eligibility, not affordability | Offer moves 2.6× across bureau risk grades; correlation with the income proxy is 0.04 |
| Most offers exceed proven capacity | **80.9%** of offers exceed demonstrated EMI headroom × annuity factor |
| Customers who ask for more *can* carry more | 808 "require higher amount" customers have the same EMI capacity as the base, got 7% lower offers, and converted at 0% |
| Large tickets leak | Off-us take-up rises from 1.1% to 7.1% as the offer grows |

## The engine

```
Proposed offer = clip[ (0.75 × incumbent + 0.25 × affordability cap) × m , ₹40k , ₹6L ]
m              = 0.30 + 1.25 × (1 − risk percentile)
Affordability  = (max EMI ever serviced − live EMI) × 25.49   # 24% p.a., 36 months
```

- **Risk model:** LightGBM on 30+ DPD in the first 6 months on book, 5-fold CV × 3 seeds,
  **selected on holdout AUC, not CV AUC** (the ensemble won CV and lost out of sample)
- **Every model beats the bureau score alone** by ~6 AUC points: the value of behavioural and on-us data
- **Policy parameters** chosen by constrained grid search over 1,219 feasible points
- **Evaluation metric** made explicit and recomputable (30/40/30 weighting of the three brief criteria), scored out-of-fold

## Results

| | Incumbent | Proposed |
|---|---:|---:|
| Brief metric (0–100) | 60.0 | **89.6** (bootstrap 95% CI 78.1–99.3) |
| Total book | ₹2,138 Cr | ₹2,169 Cr (+1.5%) |
| Offers above proven capacity | 52.5% | 41.8% |
| Average offer to customers who went 30+ DPD | – | 18.5% lower |

The book is redistributed, not inflated: under-offered, low-risk customers go up, over-extended
customers come down. On the untouched holdout (5,203 customers) risk falls monotonically as
the offer rises.

## What I will not claim

Holdout AUC 0.624 on 216 training events is honest, not impressive, and every claim is sized
to it. Adjacent risk quintiles are not separable on 72 holdout events. 40.6% of customers have
no EMI history and keep the incumbent offer.

## Repository contents

```
Submission_Deck.pptx     Competition submission (cover + 2 pages)
Method_Report.docx       Data, governance, models, metric, iterations and results
Analysis_Workbook.xlsx   Diagnostic, model comparison, risk drivers, metric ladder,
                         sensitivity, holdout quintiles and assumptions
```

The competition dataset and the customer-level scored file are not published.
