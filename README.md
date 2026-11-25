# Premium Billing and Collections: Operations Dashboard
<img width="1767" height="861" alt="Screenshot 2026-11-24 225925" src="https://github.com/user-attachments/assets/ac8a0aee-fe62-4d97-82b2-823ea09f6a3d" />

## Why I built this

I wanted to show how I approach an operations problem, not just how I build a chart. Billing and collections is a good example because late payments create real work for an insurer: reminders, re-billing, follow-up calls, and delayed cash.

The question I set out to answer: **how many premium bills are paid late, how late, and what is causing it?** I also wanted the answer to end in a recommendation an Operations leader could act on.

## The problem

Operations leaders can see that some bills are late, but they need to know three things to decide what to fix:

1. How big is the problem?
2. How late are the bills (a few days, or months)?
3. Which causes drive most of it?

## How I solved it

### Step 1: Find the data
Insurers do not publish billing data, so no public dataset had bills, due dates, and payment dates. I found a public UK home-insurance dataset with real policy details (premium, start date, status, payment method, mid-term changes). I used those as the foundation.

### Step 2: Build the missing billing layer
On top of 5,000 real policies, I generated 4 quarterly bills each (20,000 total), with due dates, payment dates, and late reasons. I linked lateness to real fields, so the simulation follows a logic you can challenge:
- Policies without direct debit are more likely to be late.
- Policies with a mid-term adjustment are more likely to need re-billing.
- Lapsed or cancelled policies are more likely to be late.

The script uses a fixed random seed, so the data is reproducible (`data-prep/make_billing.py`).

### Step 3: Define the KPIs before charting
I wrote down what each measure means in plain words before building anything (`docs/kpi_dictionary.md`): past due %, aging buckets, cycle time, SLA breach %, backlog, and rework %. This stops people arguing about numbers later.

### Step 4: Build the dashboard in Power BI
I calculated each KPI with DAX (`docs/dax_measures.md`) and built one page: KPI cards, aging buckets, cycle time over time, past-due % by payment method, and a Pareto chart that ranks the causes of late payment.

### Step 5: Check the numbers
I rebuilt the key counts in Excel pivot tables and compared them with the dashboard, so I could trust what it showed.

### Problems I hit and how I fixed them

| Problem | What I did |
|---|---|
| The source file mixes date formats (day/month/year and month/day/year) | Parsed each column with its own format |
| Cycle time dropped sharply in the last month | Unpaid bills have no payment time yet, so only quick payers counted. I excluded the last month and documented why |
| The "Current" bucket (about three quarters of bills) hid the late buckets | Filtered it out of the aging chart so the late buckets are visible |
| The Pareto showed one bar for all late bills | A DAX measure ignored the chart's grouping. I rewrote it to count rows per reason |

## What I found

| Measure | Result |
|---|---|
| Bills analyzed | 20,000 |
| Paid late (past due) | 24.85% |
| Took longer than 30 days (SLA breach) | 13.18% |
| Average days from bill to payment | 22.75 |
| Bills needing correction (rework) | 4.41% |
| Bills still unpaid | 106 |

- **Direct debit is the biggest lever.** Bills without direct debit were late about 35% of the time, versus about 12% with direct debit.
- **Most late bills are only slightly late**, so this is mainly a reminder and process problem.
- **Top causes:** no direct debit set up, customer inattention, incorrect billing information.

## What I recommend

1. Encourage direct debit at policy set-up and renewal.
2. Send automated reminders before the due date, with a person reviewing unusual cases.
3. Check billing information at intake to reduce corrections.

## About the data (please read)

- **Real:** policy details from a public UK home-insurance dataset.
- **Simulated:** bills, due dates, payment dates, and late reasons.
- **So:** this project shows my method, not the performance of a real insurer. The late reasons are assumptions.

Raw dataset: https://github.com/decodedco/datastore/raw/master/data/home-insurance.zip

## Assumptions

- The payment target (SLA) is 30 days from bill date to payment, and the due date is 21 days after the bill date. A real billing team would confirm these.
- The data ends on 31 December 2011. Bills unpaid on that date count as open.

## What I would do with real data

- Replace the simulated reasons with real codes from dispute logs, returned payments, and collector notes.
- Confirm the SLA and due-date rules with the billing team.
- Pilot reminders on one group of policies and track the change in past-due %.

## Author

Abhigna Valambatla. Power BI, DAX, Excel, Python.
