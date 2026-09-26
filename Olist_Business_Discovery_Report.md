# Olist Business Discovery
### HVIA Data & AI Solutions — Task 01, Data Analysis Internship
**Prepared by:** Nourhan · **Date:** September 2026

---

## Executive Summary

Olist's core model — enabling thousands of small Brazilian sellers to reach customers
across a large, geographically uneven country — is fundamentally healthy at the
aggregate level. No single region, seller, or category is large enough to threaten the
business on its own. But underneath that healthy aggregate, two specific, measurable
problems are quietly costing customer satisfaction and revenue quality:

1. **Delivery delay is geographically concentrated** and severely damages customer
   satisfaction where it occurs.
2. **A defined segment of high-volume sellers (14.6% of the base) underperforms on
   quality** and holds 28.2% of platform revenue — a real, largely delivery-independent
   problem, not a logistics illusion.

Both findings point toward the same proposed solution: an **Order & Seller Risk Score**
that predicts and attributes risk at the point of order, so the right team — logistics or
account management — can act on the right cause. This report documents the full
process behind that conclusion: company research, dataset structure, the analysis that
produced these findings, the solution proposal, and a draft outreach message to open a
conversation with Olist.

---

## Phase 1 — Company Research

### 1.1 Company Snapshot
Olist is a Brazilian retail-technology company headquartered in Curitiba, founded in
2015 by **Tiago Dalvi**, who remains founder and CEO today. It began as a way to help
small merchants list products on major marketplaces and has since grown into an
integrated commerce ecosystem.

| Metric | Value |
|---|---|
| Founded (as Olist) | 2015 |
| Founder & CEO | Tiago Dalvi |
| Valuation | $1.5B (Series E, December 2021) |
| Total raised | $320M+ across six funding rounds |
| Annual GMV | R$70B+ (2026) |
| Active sellers | 45,000+ |
| Products listed | 6.5M+ |
| End consumers reached | 2.5M+ |

### 1.2 The Pivot Story
Olist's history is one of repeated re-scoping around whatever pain its sellers hit next —
a pattern worth understanding before drawing conclusions from a single three-year
data window:

- **2007** — Tiago Dalvi, 21, opens **Solidarium**, a physical shop in Curitiba selling
  goods from small artisans, with a fair-trade mission and commission capped at 10%.
  The business nearly fails in its first year.
- **2008** — Pivots to become a **distributor**, pitching large retailers directly;
  lands a deal with Walmart within about six months.
- **2011** — Solidarium moves online and becomes **its own marketplace** — at its
  peak, roughly 15,000 sellers, ~1M unique visitors/month, ~380,000 products.
- **January 2014** — Near-collapse: cash projected to run out within a week, no
  founder salary for months. Bedy Yang of 500 Startups closes an investment within
  that same week to keep the company alive.
- **2015** — Full pivot and rebrand into **Olist**: instead of running its own
  marketplace, it now lists sellers' products directly on Brazil's major marketplaces
  (Americanas, Submarino, Extra, Casas Bahia, Mercado Livre).
- **2019 onward** — After a Series C (R$190M from SoftBank), Olist expands into an
  ecosystem strategy: acquiring Tiny (ERP), PAX (logistics, became Olist Pax), VNDA
  (e-commerce platform), and building out Olist Credit/Pay.
- **2025–2026** — Olist builds its own logistics vertical and enters credit (via the
  Flip acquisition); leadership frames the current strategic push as making Olist
  "AI-native," with net revenue up 65% year-over-year and its shipping unit ("Envios")
  growing 8x.

**Insight:** Olist's core competency was never "selling on marketplaces" — it is
continuously re-scoping the business around the seller's next real pain point. This
matters for how a solution proposal should be framed: not as a foreign idea, but as
the next natural step in a pattern the company has already proven it will make.

### 1.3 Business Model
Olist generates revenue through three layers, built in roughly this order:

| Layer | Products | Revenue mechanism |
|---|---|---|
| **Commerce** | Olist Store (marketplace listing), Olist Shops (branded storefront) | Take-rate + SaaS fees |
| **Logistics** | Olist Pax (fulfillment network), Envios (AI-driven shipping) | Shipping & fulfillment fees |
| **Capital** | Olist Credit (merchant lending), Olist Pay (payments) | Lending margin + processing fees |

The stated strategy since the 2019 Series C is to grow **wallet share per seller** —
not simply add more sellers.

### 1.4 Customers
Olist's paying customers are **the sellers**, not the end shoppers — small and medium
Brazilian merchants, plus a growing base of digitally-native brands, who can't justify
building direct integrations with 15+ individual marketplaces. Olist gives them one
dashboard for listings, orders, payments, and increasingly logistics and credit.

### 1.5 Pain-Point Hypotheses (going in)
Five hypotheses were identified as worth testing against the order data:

1. Logistics complexity — delivery time/cost likely varies by region
2. Fragmented seller quality — a long tail of small sellers may mean inconsistent service
3. Credit & payment risk — Olist now underwrites merchant loans
4. Competitive pressure — named rivals include Mirakl, Marketplacer, and regional players
5. AI-native execution — Olist's own stated 2026 strategic challenge

Hypotheses 4 and 5 are qualitative, company-research findings, not testable against a
2016–2018 order dataset. Hypotheses 1–3 became the subject of the data analysis in
Phases 2–3.

---

## Phase 2 — Dataset Understanding

### 2.1 The Nine Raw Tables

| Table | Rows | Role |
|---|---|---|
| `orders` | 99,441 | Central hub — one row per order |
| `order_items` | 112,650 | One row per item (orders can have multiple items/sellers) |
| `customers` | 99,441 | Buyer + delivery location |
| `sellers` | 3,095 | Merchant + location |
| `order_payments` | 103,886 | Payment method/installments per order |
| `order_reviews` | 99,224 | One review per order (roughly) |
| `products` | 32,951 | Category + physical specs |
| `geolocation` | 1,000,163 | Zip-code-prefix → lat/lng (many rows per prefix) |
| `product_category_name_translation` | 71 | Portuguese → English category lookup |

**How they connect:** `orders` is the hub — most tables join back to it via `order_id`.
`order_items` is the natural analysis grain, connecting further to `products` and
`sellers`. `customers` and `sellers` both join to `geolocation` by zip prefix.
`products` joins to the translation table by category name.

**Key modeling decisions:**
- `customer_id` is really an order-instance ID, not a person — 18.5% of customer_ids
  collapse when grouped by the real `customer_unique_id`, which is the correct key
  for any repeat-purchase analysis.
- `geolocation` is not unique per zip prefix — it must be aggregated before joining
  onto customers/sellers.
- Revenue at the seller or product level should use `price` (item-grain), not
  `payment_value` (order-grain) — using `payment_value` would attribute a whole
  multi-item order's value to every seller involved in it.

### 2.2 Data Quality — Verified, Not Assumed
A dedicated integration notebook checked every join empirically rather than trusting
the documented schema:

- **0 orphaned foreign keys** across every table relationship checked (order_items,
  order_payments, order_reviews → orders; orders → customers; order_items →
  products/sellers).
- **Cardinality confirmed empirically**: average 1.14 items per order (max 21);
  reviews-per-order max of 3.
- **Master table** built at the order-item grain (112,650 rows — matching
  `order_items` exactly, confirming the join introduced no row fan-out), with
  `order_payments` and `order_reviews` pre-aggregated before joining to avoid
  inflating row counts.

Thirteen notebooks were built in total: nine per-table profiling notebooks, one
integration/master-table notebook, one RFM customer-segmentation notebook, one
sales-and-delivery notebook, and one seller-level analysis notebook (which also
contains the validation check described in Phase 3).

---

## Phase 3 — The Business Story

### 3.1 Finding 1: Delivery Performance and Its Cost (Confirmed)

On average, Olist beats its own promise: actual delivery averages **12.1 days**
against an average *estimated* delivery of **23.4 days** — the platform under-promises
by design. But the **6.8%** of orders that are late are not evenly distributed. Late
rates spike sharply in specific Northeast/North states, far from the São Paulo/Rio
core where most sellers and infrastructure are concentrated:

| State | Late-delivery rate |
|---|---|
| Alagoas (AL) | ~21% |
| Maranhão (MA) | ~17% |
| Sergipe (SE) | ~15% |
| Piauí (PI) | ~14% |
| Ceará (CE) | ~14% |

The cost of lateness is severe: average review score falls from **4.29** (on-time) to
**2.27** (late) — a correlation of **-0.27** between delay length and score. Since most
customers in this dataset never purchase from Olist a second time, a single late
delivery may be the only impression a customer ever forms of the platform.

### 3.2 Finding 2: Seller Revenue Concentration and Quality (Confirmed, refined through validation)

**Revenue is concentrated but not dangerously so.** The top 1% of sellers hold 26% of
platform revenue, the top 5% hold 53%, and the top 10% hold 68% (Gini coefficient:
0.79). This is a long tail, not a monopoly — no single seller exceeds 1.7% of total
revenue, so losing any one seller is survivable.

**Quality issues at the tail are mostly statistical noise.** 9.9% of sellers average
below a 3.0 review score, but the median low-scorer has just two orders. Re-cutting
by genuine order volume leaves only 48 low-scorers with 5+ orders (1.6% of revenue)
and 7 with 20+ orders (0.6%) — most of the "bad" scores disappear once single-order
noise is filtered out.

**Quality does not degrade as sellers scale — it improves slightly.** Average review
score rises from 3.79 for single-order sellers to over 4.1 for sellers with 21–100
orders, a mildly positive trend (+0.12 score per decade of order volume on a log
scale).

**The segment that actually matters:** cutting sellers on both volume (median split)
and quality (platform-mean split) reveals a defined risk segment — **451 sellers
(14.6% of rated sellers)** combining above-median volume with below-average review
scores. Together they hold **28.2% of platform revenue** across nearly **30,693
orders**. This is a meaningfully different finding from "the tail is messy" — it is a
sizeable, identifiable population of established sellers whose service quality has
not kept pace with their scale.

### 3.3 Validation Check: Is the Risk Segment a Logistics Problem in Disguise?

Because Finding 1 already showed that late delivery collapses review scores, the
risk segment identified in Finding 2 was tested directly: is its low quality actually
just a symptom of delivery delay?

**The risk segment is late more often** — 10.6% of its orders arrive late, against
5.0% for the high-volume/high-quality segment and 6.8% platform-wide. So delivery
is a real, measurable contributing factor.

**But the sharper test rules out delivery as the primary cause.** The on-time-to-late
score gap *within* the risk segment (1.89 points) is nearly identical to the
platform-wide gap (2.02 points) — meaning their late orders are punished exactly as
hard as everyone else's, not disproportionately. More tellingly, the segment's
**on-time** orders still only average 4.03, well below the platform's 4.29 on-time
baseline. A clean confirmation: **88 of the 451 risk-segment sellers have a literal
0% late-delivery rate** and still average 3.64 — below the platform on-time average.

**Counterfactual test:** if every one of the segment's 2,685 late orders had scored
like a normal on-time order elsewhere, the segment's average would move from 3.86 to
only 4.05 — recovering just **45%** of its 0.43-point deficit against the
high-quality segment. The remaining **55%** of the gap survives delivery entirely.

**Verdict: a genuine mix, leaning seller-driven** (~45% logistics, ~55% real quality
issue). This matters concretely for the solution design: this segment alone produces
**41% of all late orders on the entire platform** (2,685 of 6,535) despite being only
14.6% of sellers — a naive, unattributed risk score would have booked most of that
volume as a pure "seller quality" problem and missed that it is also a meaningful
source of the platform's overall logistics performance.

### 3.4 Scope Limitation: Credit & Payment Risk

The original fifth hypothesis — credit & payment risk — is **not empirically
testable with this dataset**. The data records payment method and installment count
(`payment_type`, `payment_installments`), but contains no loan origination,
repayment, delinquency, default, or chargeback outcomes. Treating installment count
as a proxy for credit risk would produce a confident-looking number with no real
evidence behind it. This is stated here as an explicit limitation rather than
approximated with a weak substitute.

### 3.5 The Throughline

Olist's core model is healthy in aggregate — no single point of failure exists. But
two specific, quantifiable problems sit underneath that health: a geographically
concentrated delivery failure disproportionately punishing already-underserved
regions, and a defined high-volume seller segment (28.2% of revenue) whose quality
gap is real and only partially explained by logistics. Neither requires a strategic
pivot to fix — both are exactly the kind of problem targeted analytics and prediction
exist to solve.

---

## Phase 4 — Solution Proposal

### 4.1 The Order & Seller Risk Score

Rather than two separate tools, the findings point to **one unified prediction with
two attributed components**, generated at the point an order is placed:

- **Delivery-risk component** — predicts whether this specific order is likely to be
  late, based on the confirmed regional pattern (customer state, seller, season).
- **Seller-risk component** — flags whether this order's seller currently sits in the
  confirmed high-volume/low-quality segment.

**Why attribution — not just a single combined score — is necessary:** the validation
check in Section 3.3 demonstrated this empirically. A model that only asked "is this
seller often late" would completely miss the 88 risk-segment sellers with a 0% late
rate who still underperform on quality. Conversely, collapsing everything into one
undifferentiated risk number would have booked the segment's very real 41% share of
platform-wide late orders as a pure seller-quality issue. The two causes are
distinguishable in the data, and the score needs to keep them separate to be useful.

### 4.2 The Dual-Track Action Layer

| Track | Share of the risk segment's gap | Actions |
|---|---|---|
| **Logistics track** | ~45% | Carrier prioritization on at-risk routes; honest, region-adjusted delivery estimates; proactive delay notifications to customers |
| **Quality track** | ~55% | Account-management outreach to flagged sellers; listing-quality review and coaching; response-time support (not a shipping fix) |

### 4.3 Reference Pattern: How Amazon Approaches the Same Problem Shape

Amazon — a much larger version of the same marketplace-plus-logistics structure —
has already validated both halves of this approach at scale:

- **Delivery**: *regionalization* (fulfilling orders from the nearest capable node
  rather than the nearest warehouse with stock) cut average shipping distance by
  roughly 60%; *anticipatory shipping* uses ML to pre-position inventory before an
  order is even placed.
- **Seller quality**: the *Account Health Rating* is a continuously monitored
  composite of order defect rate, late shipment rate, cancellation rate, and valid
  tracking rate — triggering proactive intervention before a seller's public
  reputation visibly collapses, rather than waiting on lagging review scores alone.

Olist does not operate fulfillment infrastructure at Amazon's scale, so the pitch is
framed around the underlying *mechanism* Amazon proved out (predict early, attribute
correctly, act proactively) rather than its literal infrastructure spend, which would
undercut credibility rather than build it.

### 4.4 Mapped to HVIA's Capability Categories

| HVIA Category | Component |
|---|---|
| Forecasting & Predictive Solutions | The delivery-risk component |
| Customer Intelligence | The seller-risk component and segment attribution (sellers are Olist's real customers) |
| AI-Assisted Operations | Automatic routing of each flagged order to the correct track |
| Analytics & Decision Systems | The unified dashboard surfacing both components together |

---

## Phase 5 — Outreach Draft

**Target:** a role such as Head of Data/Analytics, Head of Operations, or Head of
Seller Experience at Olist — a data-driven, operationally-grounded audience rather
than executive leadership, since this is a specific, evidence-based pitch rather than
a general partnership proposal. (The exact current name for this role should be
confirmed via LinkedIn or Olist's official site before sending.)

**Channel:** LinkedIn.

**Step 1 — Connection request** (character-limited):
> Hi [Name] — I've been analyzing Olist's public order dataset and found a pattern in
> seller performance that I think your team would find genuinely useful. Would love
> to connect and share it.

**Step 2 — Full message** (sent after connecting):
> Hi [Name],
>
> I've been digging into Olist's public order dataset as part of a data analysis
> project, and one pattern stood out: about 15% of sellers account for over 40% of
> all late deliveries platform-wide, while holding roughly 28% of total revenue.
> Looking closer, it's not simply a shipping issue — some of it traces back to
> delivery timing, but most of the quality gap holds up even on their on-time orders.
>
> That's the kind of pattern a seller-level risk score could catch early — flagging
> whether a seller's risk is logistics-driven or service-driven before it shows up in
> reviews, so the right team can act on the right cause.
>
> I'd love to share the full analysis and hear whether this lines up with something
> your team is already looking at. Would you be open to a short 15-minute call in the
> next couple of weeks?
>
> Best,
> Nourhan — Data Analyst Intern, HVIA Data & AI Solutions

---

## Appendix: Notebooks Produced

1. `01_customers.ipynb` — Customers profiling
2. `02_geolocation.ipynb` — Geolocation profiling
3. `03_sellers.ipynb` — Sellers profiling
4. `04_orders.ipynb` — Orders profiling
5. `05_order_items.ipynb` — Order items profiling
6. `06_order_payments.ipynb` — Payments profiling
7. `07_order_reviews.ipynb` — Reviews profiling
8. `08_products.ipynb` — Products profiling
9. `09_product_category_name_translation.ipynb` — Category lookup profiling
10. `10_integration.ipynb` — Referential integrity, cardinality, master table
11. `11_customer_analysis.ipynb` — RFM customer segmentation
12. `12_sales_and_delivery.ipynb` — Revenue trends, delivery performance
13. `13_seller_analysis.ipynb` — Revenue concentration, quality variance, segmentation, and the logistics-vs-seller validation check
