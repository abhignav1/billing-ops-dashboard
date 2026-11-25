# KPI dictionary

## KPIs

| KPI | Plain meaning | Formula | Who uses it | What a bad number means |
|---|---|---|---|---|
| Total bills | Number of premium bills in view | Count of bills | Operations | Context for every other KPI |
| Past due % | Share of bills paid after the due date or still unpaid past due | Bills with days past due > 0 / total bills | Billing and collections leads | Too many customers pay late, so more reminders and rework |
| Aging buckets | How late bills are, grouped | Current, 1-30, 31-60, 61-90, 90+ days past due | Collections | Growth in the 61+ buckets signals collection risk |
| Avg cycle days | Days from bill date to payment | Average of (paid date - bill date) for paid bills | Operations leadership | Longer cycles delay cash |
| SLA breach % | Bills that took longer than 30 days to pay | Paid bills with cycle days > 30 / paid bills | Operations leadership | The payment process is missing its service target |
| Backlog | Bills still unpaid at the snapshot date | Count where open = true | Collections | Cash is outstanding and needs follow-up |
| Rework % | Bills needing correction or re-billing | Bills with rework flag / total bills | Billing operations | Errors at intake or adjustment steps create extra work |
| Late bills | Bills with a late reason | Count where late reason is not "N/A" | Process improvement | Used for the Pareto |
| Cumulative % | Running share of late bills across sorted reasons | Running total of late bills / all late bills | Process improvement | Shows the few causes behind most lateness |

## Fields

| Field | Source | Notes |
|---|---|---|
| bill_id | Simulated | Unique bill number |
| policy_id | Real | Policy number from the source dataset (anonymized) |
| installment_no | Simulated | 1 to 4, quarterly |
| policy_status | Real | Live, Lapsed, Cancelled |
| property_type | Real | Numeric code in the source dataset; meaning not documented |
| payment_method | Real | PureDD, DD-Other, NonDD |
| has_mta | Real | True if the policy has a mid-term adjustment date |
| bill_date | Derived from real cover start | Cover start plus 3 months x (installment - 1) |
| due_date | Simulated rule | Bill date plus 21 days |
| paid_date | Simulated | Blank if open at the snapshot date |
| amount | Derived from real premium | Annual premium / 4 |
| is_open | Simulated | Unpaid at 31 Dec 2011 |
| late_reason | Simulated | "N/A" if on time |
| rework_flag | Simulated | True for "Mid-term adjustment re-bill" and "Incorrect billing info" |
| cycle_days | DAX column | Days from bill date to paid date |
| days_past_due | DAX column | Zero if on time; open bills are measured to the snapshot date |
| aging_bucket | DAX column | Grouping of days past due |
| aging_sort | DAX column | Sort order for the buckets |
| sla_breach | DAX column | True if cycle days > 30 |
