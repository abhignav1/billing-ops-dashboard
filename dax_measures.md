# DAX columns and measures

Table name: `premium_bills`.

## Calculated columns

```
cycle_days = DATEDIFF(premium_bills[bill_date], premium_bills[paid_date], DAY)
```
```
days_past_due =
VAR endDate = IF(ISBLANK(premium_bills[paid_date]), DATE(2011,12,31), premium_bills[paid_date])
RETURN MAX(0, DATEDIFF(premium_bills[due_date], endDate, DAY))
```
```
aging_bucket =
SWITCH(TRUE(),
    premium_bills[days_past_due] = 0, "Current",
    premium_bills[days_past_due] <= 30, "1-30",
    premium_bills[days_past_due] <= 60, "31-60",
    premium_bills[days_past_due] <= 90, "61-90",
    "90+")
```
```
aging_sort =
SWITCH(TRUE(),
    premium_bills[days_past_due] = 0, 1,
    premium_bills[days_past_due] <= 30, 2,
    premium_bills[days_past_due] <= 60, 3,
    premium_bills[days_past_due] <= 90, 4,
    5)
```
```
sla_breach = IF(ISBLANK(premium_bills[cycle_days]), FALSE(), premium_bills[cycle_days] > 30)
```

## Measures

```
Total Bills = COUNTROWS(premium_bills)
```
```
Past Due % = DIVIDE(CALCULATE([Total Bills], premium_bills[days_past_due] > 0), [Total Bills])
```
```
Backlog = CALCULATE([Total Bills], premium_bills[is_open] = TRUE())
```
```
Avg Cycle Days = AVERAGE(premium_bills[cycle_days])
```
```
SLA Breach % =
DIVIDE(
    CALCULATE([Total Bills], premium_bills[sla_breach] = TRUE()),
    CALCULATE([Total Bills], NOT(ISBLANK(premium_bills[cycle_days])))
)
```
```
Rework % = DIVIDE(CALCULATE([Total Bills], premium_bills[rework_flag] = TRUE()), [Total Bills])
```
```
Late Bills = COUNTROWS(FILTER(premium_bills, premium_bills[late_reason] <> "N/A"))
```
```
Cumulative % =
VAR cur = [Late Bills]
VAR allReasons = ALLSELECTED(premium_bills[late_reason])
VAR runTotal = SUMX(FILTER(allReasons, [Late Bills] >= cur), [Late Bills])
RETURN DIVIDE(runTotal, CALCULATE([Late Bills], allReasons))
```
