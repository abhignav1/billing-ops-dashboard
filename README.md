# Premium Billing and Collections: Operations Dashboard

A one-page Power BI dashboard that shows how premium bills are paid, how late they run, and why. It is built to support Operations decisions in billing and collections: where the backlog sits, which bills breach the payment SLA, and which root causes to fix first.

<img width="1767" height="861" alt="Screenshot 2026-11-24 225925" src="https://github.com/user-attachments/assets/ac8a0aee-fe62-4d97-82b2-823ea09f6a3d" />


## Business problem

Late premium payments create rework, collection effort, and cash-flow delays. Operations leaders need to see how many bills are late, how late they are, and which causes drive most of the lateness, so they can prioritize process fixes instead of chasing individual bills.

## Key findings

| Measure | Result |
|---|---|
| Total bills analyzed | 20,000 (5,000 policies x 4 quarterly bills) |
| Past due | 24.85% |
| SLA breach (paid after 30 days) | 13.18% |
| Average cycle time (bill to payment) | 22.75 days |
| Rework rate | 4.41% |
| Open bills (backlog) | 106 |

- **Direct debit is the biggest lever.** Policies without direct debit were late about 35% of the time, versus about 19% for other direct-debit types and about 12% for pure direct debit.
- **"No direct debit set up" is the top root cause** in the Pareto analysis, followed by customer inattention and incorrect billing information.
- **Most lateness is short.** About three quarters of bills are current, and nearly all late bills fall in the 1-30 day bucket, so this is a reminder-and-process problem more than a collections problem.
- **Mid-term adjustments create re-billing.** "Mid-term adjustment re-bill" is the smallest cause by volume, but it is a pure rework driver.

See [`docs/leadership_summary.md`](docs/leadership_summary.md) for the one-page summary with recommendations.

## About the data

**Real data.** Policy attributes come from a public UK home-insurance dataset (256,136 policies, 2007-2012, originally on Kaggle as "Home Insurance" by ycanario). Real fields used: policy number, annual premium, cover start date, policy status, property type code, payment method (direct debit or not), and mid-term adjustment dates.

**Simulated data.** Insurers do not publish billing or collections data because it is private customer financial information. I generated the billing layer on top of the real policies: quarterly bills, due dates, payment dates, open/paid status, and late-payment reasons.

**How the simulation works.** Late-payment probability depends on real policy attributes: non-direct-debit policies are more likely to be late, policies with mid-term adjustments are more likely to be re-billed, and lapsed or cancelled policies are more likely to be late. A fixed random seed (42) makes the data reproducible. See [`data-prep/make_billing.py`](data-prep/make_billing.py).

**What this project demonstrates.** The analysis method: KPI definitions, validation, root-cause (Pareto) analysis, and recommendations. The findings are illustrative and do not describe the performance of any real insurer. Late reasons and rework flags are simulated assumptions, not observed causes.

Raw dataset (public copy): https://github.com/decodedco/datastore/raw/master/data/home-insurance.zip

## Method

1. Generated premium bills from real policies (`data-prep/make_billing.py`).
2. Loaded `data/premium_bills.csv` into Power BI and set data types in Power Query.
3. Added calculated columns in DAX: `cycle_days`, `days_past_due`, `aging_bucket`, `aging_sort`, `sla_breach`.
4. Wrote DAX measures for each KPI ([`docs/dax_measures.md`](docs/dax_measures.md)).
5. Built the dashboard: KPI cards, aging chart, cycle-time trend, past-due % by payment method, Pareto of late reasons, and slicers for policy status and payment method.
6. Validated the numbers with Excel pivot tables (see Validation).

## Validation

| Check | Dashboard | Excel pivot | Match |
|---|---|---|---|
| Total bills | 20,000 | | |
| Open bills (backlog) | 106 | | |
| Bills by late reason (top cause) | | | |
| Bills by aging bucket | | | |

## Definitions and assumptions

- The payment SLA is **30 days from bill date to payment**. Due dates are 21 days after the bill date. These are assumptions for this project, to be confirmed with the real billing team.
- The snapshot date is **31 December 2011**. Bills unpaid at that date are counted as open.
- The cycle-time chart excludes the last month because open bills have no cycle time yet, which would create a false dip.
- Property type is a numeric code in the source dataset. Its meaning is not documented there, so it is not used as a slicer.

Full definitions: [`docs/kpi_dictionary.md`](docs/kpi_dictionary.md).

## Repository structure

```
billing-ops-dashboard/
├── README.md
├── data/
│   └── premium_bills.csv          (20,000 simulated bills on real policies)
├── data-prep/
│   └── make_billing.py            (generates the billing layer, seed 42)
├── dashboard/
│   ├── billing_dashboard.pbix     (Power BI file)
│   └── dashboard_screenshot.png
└── docs/
    ├── kpi_dictionary.md
    ├── dax_measures.md
    └── leadership_summary.md
```

## How to reproduce

1. Download and unzip the raw dataset (link above) to get `home_insurance.csv`.
2. Run `python make_billing.py home_insurance.csv premium_bills.csv`.
3. Open `dashboard/billing_dashboard.pbix` in Power BI Desktop and point it to your `premium_bills.csv` if prompted.

## Limitations

- Billing events, late reasons, and rework flags are simulated, so root-cause shares reflect my assumptions.
- Real billing data would include dispute logs, returned-payment codes, and collector notes. With those, I would replace the simulated reasons and re-check the Pareto.
- Only one process (quarterly premium billing) is covered.

## Related work

- Process documentation pack (SIPOC, swim lanes, RACI): _link when published_
- Copilot use-case specification for automated payment reminders: _link when published_

## Author

Abhigna Valambatla. Tools: Power BI, DAX, Power Query, Excel, Python.
