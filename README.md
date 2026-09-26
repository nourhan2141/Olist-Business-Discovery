# Olist Business Discovery

**HVIA Data & AI Solutions — Data Analysis Internship, Task 01**

An end-to-end analysis of Olist, a Brazilian marketplace-enablement company, using its
public [Brazilian E-Commerce dataset](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce)
(2016–2018, ~100K orders). The project goes from company research through data
profiling, empirical findings, a solution proposal, and a draft outreach message —
following HVIA's own framework: **Understand → Analyze → Propose → Pitch**.

## TL;DR

Olist's core marketplace model is healthy in aggregate, but two specific, measurable
problems sit underneath it:

1. **Delivery delay is geographically concentrated** (late rates of 15–21% in
   AL/MA/SE/PI/CE vs. 6.8% platform-wide) and collapses review scores when it happens
   (4.29 → 2.27).
2. **A defined seller segment — 14.6% of sellers, 28.2% of revenue — underperforms on
   quality**, and a validation check confirmed this is ~55% a genuine seller-quality
   issue and ~45% delivery-driven, not one masquerading as the other.

The proposed solution is a single **Order & Seller Risk Score** with two attributed
components (delivery-risk and seller-risk), routing each flagged case to the right
team instead of collapsing both causes into one number. Full detail in
[`Olist_Business_Discovery_Report.md`](./Olist_Business_Discovery_Report.md).

## Repository Structure

```
.
├── data/
│   ├── raw/                        # 9 original Kaggle CSVs (not committed — see Setup)
│   └── processed/
│       └── master_table.csv        # Joined, order-item-grain table built in notebook 10
├── notebooks/
│   ├── 01_customers.ipynb
│   ├── 02_geolocation.ipynb
│   ├── 03_sellers.ipynb
│   ├── 04_orders.ipynb
│   ├── 05_order_items.ipynb
│   ├── 06_order_payments.ipynb
│   ├── 07_order_reviews.ipynb
│   ├── 08_products.ipynb
│   ├── 09_product_category_name_translation.ipynb
│   ├── 10_integration.ipynb        # Referential integrity, cardinality, master table
│   ├── 11_customer_analysis.ipynb  # RFM segmentation
│   ├── 12_sales_and_delivery.ipynb # Revenue trends, delivery performance
│   └── 13_seller_analysis.ipynb    # Revenue concentration, quality, validation check
├── Olist_Business_Discovery_Report.md   # Full written report (all 5 phases)
├── Olist_Business_Discovery_Full.pptx   # Presentation deck (28 slides)
└── README.md
```

## Setup

1. Download the dataset from
   [Kaggle](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce) and place
   the 9 CSVs in `data/raw/`.
2. Install dependencies:
   ```bash
   pip install pandas numpy matplotlib seaborn jupyter
   ```
3. Run the notebooks **in numeric order** — each profiling notebook (01–09) is
   independent, but 10 (integration) depends on all nine, and 11–13 depend on the
   master table produced by 10.
   ```bash
   jupyter notebook notebooks/
   ```

## Notebooks

| #     | Notebook            | What it covers                                                                                                                                                                                                                      |
| ----- | ------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 01–09 | Per-table profiling | Schema, missing values, duplicates, key uniqueness, and table-specific checks for each of the 9 raw CSVs                                                                                                                            |
| 10    | Integration         | Empirical referential-integrity and cardinality checks across all tables; builds and saves `master_table.csv`                                                                                                                       |
| 11    | Customer analysis   | RFM (recency/frequency/monetary) segmentation                                                                                                                                                                                       |
| 12    | Sales & delivery    | Monthly revenue trend, revenue by state/category, delivery performance, review-score-vs-delay                                                                                                                                       |
| 13    | Seller analysis     | Revenue concentration (Gini/HHI), review-quality variance, noise-vs-signal filtering, volume-vs-quality relationship, 2×2 seller segmentation, and a validation check separating logistics-driven from seller-driven quality issues |

## Data Quality

Referential integrity was checked empirically rather than assumed: **0 orphaned
foreign keys** across every join (orders ↔ customers, order_items ↔ products/sellers,
orders ↔ payments/reviews). The master table (112,650 rows) matches `order_items`
row-for-row, confirming no fan-out occurred during the join.

## Key Findings

- **Delivery**: 6.8% of orders arrive late overall; the rate is 2–3x higher in
  specific North/Northeast states. Late orders score 2.27 on average vs. 4.29 for
  on-time orders (correlation −0.27).
- **Seller concentration**: top 1% of sellers hold 26% of revenue, top 10% hold 68%
  (Gini 0.79) — a long tail, not a monopoly (no seller exceeds 1.7% of revenue).
- **Seller quality**: apparent quality problems in the tail are mostly single-order
  noise. The real risk sits with 451 sellers (14.6%, 28.2% of revenue) who combine
  high volume with below-average scores — and a validation check confirmed ~55% of
  their quality gap is genuine, not a delivery-timing artifact.
- **Out of scope**: credit/payment risk (the original 5th hypothesis) is not
  testable with this dataset — it records payment method and installments, but no
  loan repayment or default outcomes.

Full methodology and numbers: see the report and notebooks 12–13.

## Deliverables

- [`Olist_Business_Discovery_Report.md`](./Olist_Business_Discovery_Report.md) — full written report (company research, dataset understanding, business story, solution proposal, outreach draft)
-  [`Olist_Business_Discovery_Full.pptx`](./Olist_Business_Discovery_Full.pptx) — 28-slide presentation deck
-  `notebooks/` — full analytical evidence trail

## Tech Stack

Python · pandas · NumPy · Matplotlib · Seaborn · Jupyter

## Author

Nourhan — Data Analysis Intern, HVIA Data & AI Solutions
