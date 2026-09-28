# SCCL Coal Production Performance Dashboard

A Power BI dashboard analyzing 10 years of operational performance at *The Singareni Collieries Company Limited (SCCL)*, a government coal mining company in Telangana, along with its progress in the current financial year (FY 2026-27).

Built entirely from *public government data* using Power BI, Power Query and DAX.

![Page 1 – 10-Year Performance](images/page1_overview.png)
![Page 2 – FY 2026-27 Progress](images/page2_current_year.png)

---

## Business Problem

SCCL produced a record 70.02 million tonnes (MT) of coal in 2023-24, but output dropped sharply in 2025-26. This project answers:

1. How has SCCL's production, dispatch and productivity changed over the last 10 years?
2. What is driving the recent decline in output?
3. How is the workforce changing, and how does it affect productivity?
4. Is SCCL on track to meet its FY 2026-27 production target?
5. How is SCCL's share of India's coal production changing?

---

## Data Sources

| Dataset | Source | Period |
|---|---|---|
| Annual performance (production, dispatch, OMS, overburden removal, manpower) | [SCCL website – Performance at a Glance](https://scclmines.com/scclnew/performance_production.asp) | 2015-16 to 2025-26 |
| Current-year production and offtake by company | National Coal Portal, Ministry of Coal (coaldashboard.cmpdi.co.in) | FY 2026-27 (up to 22-09-2026) |
| FY 2026-27 production target (62 MT) | Coal India production portal | FY 2026-27 |
| All-India coal production (for market share) | Ministry of Statistics and Programme Implementation (via Dataful) | 2023-24, 2024-25 |

All data was accessed on 24-09-2026. *No internal company data was used.*

---

## Tools Used

- *Power BI Desktop* – data modeling and dashboard design
- *Power Query* – data cleaning and reshaping
- *DAX* – calculated columns and measures
- *Microsoft Excel* – initial data collection

---

## Data Preparation

The SCCL website publishes data as a wide table (years as columns, metrics as rows), which is not suitable for analysis. Steps taken:

1. Imported the web table into Power Query.
2. *Transposed* the table so each year became a row.
3. Promoted the first row to headers and renamed columns to short names (Production_MT, Dispatch_MT, OMS_T, OB_Removal_MCuM, Manpower).
4. Set correct data types (text for year, decimal for metrics, whole number for manpower).
5. Entered the National Coal Portal snapshot and target data as separate tables.
6. Validated the snapshot data by checking that company totals matched the portal's all-India total (425.34 MT).

### Calculated Columns (DAX)

DAX
Year_Start = VALUE(LEFT(sccl_annual[Year], 4))

Prod_per_Employee = sccl_annual[Production_MT] * 1000000 / sccl_annual[Manpower]

OB_per_Tonne = sccl_annual[OB_Removal_MCuM] / sccl_annual[Production_MT]

Stock_Change = sccl_annual[Dispatch_MT] - sccl_annual[Production_MT]

YoY_Growth =
VAR curYr = VALUE(LEFT(sccl_annual[Year], 4))
VAR prevProd = CALCULATE(SUM(sccl_annual[Production_MT]),
    FILTER(ALL(sccl_annual), VALUE(LEFT(sccl_annual[Year], 4)) = curYr - 1))
RETURN IF(ISBLANK(prevProd), BLANK(), DIVIDE(sccl_annual[Production_MT] - prevProd, prevProd))


### Key Measures (DAX)

DAX
Latest Production =
VAR y = MAX(sccl_annual[Year_Start])
RETURN CALCULATE(SUM(sccl_annual[Production_MT]), sccl_annual[Year_Start] = y)

Change from Peak = DIVIDE([Latest Production] - MAX(sccl_annual[Production_MT]), MAX(sccl_annual[Production_MT]))

Days Elapsed = DATEDIFF(MAX(sccl_target[Start_Date]), MAX(sccl_target[Data_Date]), DAY) + 1

Expected by Now = [Annual Target] * [Days Elapsed] / 365

Required Daily Rate = DIVIDE([Annual Target] - [SCCL YTD Production], 365 - [Days Elapsed])


---

## Dashboard Pages

### Page 1: 10-Year Performance (2015-16 to 2025-26)
- *KPI cards:* latest production, peak production, change from peak, latest manpower, manpower change
- *Production vs dispatch trend* (line chart)
- *Year-over-year production growth* (column chart, red/green for decline/growth)
- *Manpower trend* (column chart)
- *Stock change* – dispatch minus production (column chart)
- *Overburden removal trend* (line chart)

### Page 2: FY 2026-27 Progress (April 1 to September 22, 2026)
- *KPI cards:* year-to-date production and offtake, target achieved, current vs required daily pace
- *Target progress gauge* – actual production vs expected production by this date
- *India's coal production mix* (donut chart) – SCCL's share highlighted
- *Production vs offtake by company* (column chart)
- *Current vs required daily production rate* (column chart)

---

## Key Insights

1. *Production peaked, then fell sharply.* Output grew 16% from 2015-16 to a record 70.02 MT in 2023-24, then fell 16% in 2025-26 to 58.00 MT, the largest single-year drop in the period.
2. *COVID impact was temporary.* Production fell 21% in 2020-21 but fully recovered the following year.
3. *Fewer people, more output per person.* Manpower fell 31% (58,491 to 40,335), while production per employee rose from about 1,030 tonnes to about 1,695 tonnes by 2024-25, pointing to greater mechanization.
4. *Stock is being drawn down.* In 2025-26, SCCL dispatched 4.07 MT more coal than it produced, and the same pattern continues in FY 2026-27 (28.81 MT dispatched vs 24.59 MT produced).
5. *Mining is getting harder.* Overburden removed per tonne of coal rose from about 5.1 to about 6.3 cubic metres, which increases mining costs.
6. *SCCL is behind its FY 2026-27 target.* With 39.7% of the 62 MT target achieved by 22 September, SCCL is about 5 MT behind the expected pace and needs to raise its daily output by about 40% (from ~0.14 to ~0.20 MT/day).
7. *Market share is shrinking.* SCCL's share of India's coal production fell from about 7.0% (2023-24) to 6.6% (2024-25) and about 5.8% so far in FY 2026-27.

### Open Question
Productivity (OMS) reached its highest level (7.56 T) in 2025-26, the same year production fell sharply. This may be due to how man-shifts are counted or a shift towards opencast mining, and needs further investigation.

---

## Limitations

- FY 2026-27 figures are *provisional* and taken from a single daily snapshot.
- Data dates differ slightly: CIL figures are up to 23-09-2026, others up to 22-09-2026.
- Coal production is *seasonal* (slower during the monsoon, faster from January to March), so the current shortfall against target may partly recover.
- All-India totals for market share were available only for selected years.

---

## Next Steps

- Add *monthly and area-wise data* (underground vs opencast, mining areas) from SCCL's monthly performance reports.
- Build a *daily tracking dataset* from the National Coal Portal to monitor progress against the target over time.
- Compare SCCL with *Coal India subsidiaries* using Ministry of Coal annual reports.

---

## Repository Structure


SCCL-Coal-Production-Dashboard/
├── README.md
├── data/
│   ├── sccl_annual.csv
│   └── coal_snapshot.csv
├── dashboard/
│   └── SCCL_Performance_Dashboard.pbix
└── images/
    ├── page1_overview.png
    └── page2_current_year.png


To view the dashboard, download the .pbix file and open it in [Power BI Desktop](https://powerbi.microsoft.com/desktop/) (free).

---

## Disclaimer

This is an independent analysis for learning and portfolio purposes, based only on publicly available data. It is not affiliated with or endorsed by SCCL or the Ministry of Coal.

---

## Author

*Guduri Nikhitha*
[LinkedIn](https://www.linkedin.com/in/guduri-nikhitha-923947245) | [GitHub](https://github.com/Nikhitha458500)