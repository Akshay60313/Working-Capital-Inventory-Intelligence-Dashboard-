
# Working Capital & Inventory Intelligence Dashboard
### Power BI | DAX | Python | Star Schema Data Model

<img width="1167" height="686" alt="executive summary" src="https://github.com/user-attachments/assets/26f614dc-bc24-476b-9d29-7be91d57daf2" />


---

## Overview

A working capital and inventory analysis dashboard built in Power BI, 
designed to answer one question most inventory systems cannot:

**Not how much stock do we have — but how much of that stock 
should not be there.**

The dashboard surfaces trapped working capital, automates reorder 
alerts based on rolling 7-day sales velocity, and quantifies the 
opportunity cost of excess inventory in financial terms — translating 
operational data into CFO-level insight.

<img width="1170" height="796" alt="inventory dashboard" src="https://github.com/user-attachments/assets/a13591cb-273e-468f-858a-a36352d95162" />


---

## Business Problem Solved

Most warehouses track stock levels. Very few track what excess stock 
costs the business every day it sits unsold.

<img width="1156" height="718" alt="stock alert" src="https://github.com/user-attachments/assets/c62a1db8-fce0-4873-9344-fd831a307066" />


Stock beyond 14 days of rolling average sales is frozen cash:
- Earns zero return (opportunity cost at BoE base rate: 4.25%)
- Accumulates storage and insurance costs
- Blocks capital that could fund faster-moving lines

This dashboard makes that cost visible, measurable, and actionable.

---

## Dataset

Built entirely in Python using realistic warehouse simulation logic.
No pre-calculated averages stored — Power BI calculates everything 
dynamically from raw transaction data.

<img width="1172" height="692" alt="order tracker" src="https://github.com/user-attachments/assets/cc058c05-521b-40c8-b0e6-44411cb30101" />


| Table | Rows | Description |
|---|---|---|
| Transactions | 22,435 | Individual order lines |
| Purchase Orders | 559 | Supplier replenishments |
| Opening Stock | 130 | Period-start snapshot |
| Products | 130 SKUs | Static product data only |
| Customers | 3,000 | Individual UK consumers |
| Suppliers | 8 | Lead times and payment terms |
| Date Table | 60 days | 1 Jan – 29 Feb 2024, no gaps |

**Categories:** Electronics · Office Supplies · Furniture · 
Cleaning · Packaging · Beverages · Snacks · Safety

**Sales Channels:** Shopify (30%) · Amazon (28%) · 
Direct Website (20%) · Wholesale Portal (14%) · eBay (8%)

---

## Data Model

Star schema — Transactions as central fact table connected 
to all dimensions via single-direction relationships.

Opening_Stock ──→ Products ←── Purchase_Orders
↑              ↑
Transactions ────────┤          Suppliers
↑               │
Customers        Date_Table
↑               ↑
Date_Table ──────────┘

No bidirectional relationships. No ambiguous filter paths.
No pre-calculated stock levels — all computed dynamically in DAX.

---

## Core Logic — Rolling 7-Day Average

Every inventory threshold is driven by actual rolling sales 
velocity — not hardcoded numbers. If demand changes, 
all alerts update automatically. No manual intervention required.

```dax
Avg 7 Day Sales = 
DIVIDE(
    CALCULATE(
        [Units Sold],
        DATESINPERIOD(
            Date_Table[Date],
            LASTDATE(Date_Table[Date]),
            -7, DAY
        )
    ),
    7
)
```

---

## Stock Alert Framework

| Status | Condition | Business Action |
|---|---|---|
| 🔴 CRITICAL | Stock < 3 days avg | Order immediately |
| 🟠 LOW | Stock < 7 days avg | Order this week |
| 🟢 OK | 7–14 days avg | No action needed |
| 🟡 EXCESS | Stock > 14 days avg | Pause next order |

<img width="1156" height="718" alt="stock alert" src="https://github.com/user-attachments/assets/0e518dff-5daa-4fa4-85cd-4d781316a643" />


With most suppliers delivering in 1–5 days, holding 
more than 14 days of stock serves no operational purpose. 
Every unit beyond that threshold is frozen working capital.

---

## Key DAX Measures

**Current Stock (cumulative, period-to-date):**
```dax
Current Stock Units = 
[Opening Stock Units] + [Units Received] - [Units Sold]
```

**Days of Stock remaining:**
```dax
Days of Stock = 
DIVIDE([Current Stock Units], [Avg 7 Day Sales])
```

**Excess Stock Value (only units above 14-day threshold):**
```dax
Excess Stock Value = 
SUMX(
    Products,
    VAR DaysStock    = [Days of Stock]
    VAR AvgDaily     = [Avg 7 Day Sales]
    VAR UnitCost     = Products[Unit_Cost]
    VAR ExcessDays   = MAX(DaysStock - 14, 0)
    VAR ExcessUnits  = ExcessDays * AvgDaily
    RETURN           ExcessUnits * UnitCost
)
```

> SUMX iterates per SKU because each product has a different 
> unit cost and different excess days. A single SUM would 
> apply one cost to all products — producing incorrect results.

**Opportunity Cost of excess inventory:**
```dax
Opportunity Cost Annual = 
[Excess Stock Value] * 0.0425
```

**On-Time Delivery Rate:**
```dax
OTD % = 
DIVIDE(
    CALCULATE(COUNTROWS(Purchase_Orders),
        Purchase_Orders[On_Time] = 1),
    COUNTROWS(Purchase_Orders)
)
```


---

## Key Findings (Dataset)
Total Revenue:           £712,000
Gross Margin:            47.3%
Current Stock Value:     £62,000
Excess Stock Value:      £8,910  (14.4% of total)
Opportunity Cost/year:   £379
Orders Delayed:          1,405   (6.3% — above 4% benchmark)
Slow Mover SKUs:         13      (10% of SKUs, 22% of stock value)


## Skills Demonstrated

| Area | Detail |
|---|---|
| Power BI | Star schema, 4-page dashboard, conditional formatting |
| DAX | SUMX, CALCULATE, DATESINPERIOD, VAR blocks |
| Python | Pandas dataset generation, simulation logic |
| Data Modelling | Star schema, fact vs dimension, relationship direction |
| Financial Analysis | Working capital, opportunity cost, capital efficiency |
| Business Insight | Operational data → CFO-level financial language |

---

## Author

**Akshay Jain**  
Finance & Data Analyst | Kaizen Analytics  
[LinkedIn](https://linkedin.com/in/akshay-jain-40589633a) · 
