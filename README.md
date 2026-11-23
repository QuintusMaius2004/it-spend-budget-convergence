# IT Spend Budget Convergence: Power BI Capstone

Final project for the PL-300 (Power BI) course. It analyses Microsoft's **IT Spend Analysis** sample dataset to answer one question:

> Do our budget forecasts get closer to the truth over time?

## Business context

Each year the IT budget is set once (**Plan**), then re-forecast after each quarter as actuals come in: **LE1**, **LE2** and **LE3** (latest estimates after Q1, Q2 and Q3). The **Actual** is the final number.

If forecasting works, each re-forecast should land closer to the actual than the original plan did. This project measures that.

## The key metric: Convergence

```dax
Convergence % = ABS([Var Plan %]) - ABS([Var LE3 %])
Convergence $ = ABS([Var Plan])   - ABS([Var LE3])
```

- **Positive:** the final forecast (LE3) was closer to the actual than the original plan. The process is self-correcting.
- **Near zero or negative:** the re-forecasts did not improve on the plan. The forecasting process is not learning.

## Findings

From the presentation:
- **Infrastructure** converged the most (about +33.9%). Forecasts moved clearly closer to actuals by Q3.
- **Manufacturing** showed essentially none (about −0.7%). Its final forecast was no better than the original plan.
- **Office & Administrative** also stalled.
- Infrastructure spend is dominated by labor and concentrated in the US, so it is sensitive to a small number of decisions.
- Suggested action: keep monitoring Infrastructure, and review the forecasting process for Manufacturing and Office & Administrative.

## Data model

A star schema with one fact table and seven dimensions, using `dim_` / `fact_` prefixes and snake_case names.

| Table | Role |
|---|---|
| `fact_it_spend` | Spend values by date, department, cost element, business area, IT sub-area, scenario and region |
| `dim_date` | Calendar |
| `dim_scenario` | Plan, Latest Estimate 1 to 3, Actual |
| `dim_business_area`, `dim_cost_element`, `dim_it_area`, `dim_department`, `dim_region` | Slicing dimensions |

All seven relationships are one-to-many from the dimension to `fact_it_spend`.

## DAX measures

| Group | Measures |
|---|---|
| Base | `Amount` = `TOTALYTD(SUM(value), dim_date[date])`, so all figures are year-to-date |
| Scenarios | `Actual`, `Plan`, `LE1`, `LE2`, `LE3`, each `CALCULATE([Amount])` filtered by scenario |
| Variance | `Var Plan`, `Var LE1` to `Var LE3` (Actual minus forecast) and their `%` versions using `DIVIDE` |
| Convergence | `Convergence %`, `Convergence $` |
| Labels | `Var Plan % Label`, `Var LE1 % Label` to `Var LE3 % Label`: formatted text with a ▲ or ▼ arrow for cards |

### Data validation

The measures were reconciled to the source sample's published figures. An unexplained ×0.3 multiplier in the source `Amount` formula was removed, after confirming with the instructor that it deliberately obscures the data.

## Report pages

| Page | Content |
|---|---|
| Overview | Headline KPI cards for Actual, Plan and LE1 to LE3 with variance labels, plus the monthly convergence trend |
| Analysis | Var Plan % vs Var LE3 % and convergence ranking by business area, filterable by IT area |
| Details | Convergence by cost element, switchable to region |
| `actual_tt`, `convergence_tt` | Hidden tooltip pages that show monthly detail on hover |

Navigation buttons link the pages, and two bookmarks toggle the Region and Elements Group views.

## Repository contents

This is a Power BI Project (`.pbip`), so the model and report are plain text (TMDL and PBIR) and diff well in Git.

| Path | Contents |
|---|---|
| `IT_Spend_Convergence.pbip` | Project entry point, open this in Power BI Desktop |
| `IT_Spend_Convergence.SemanticModel/` | Tables, relationships and measures as TMDL |
| `IT_Spend_Convergence.Report/` | Pages, visuals and bookmarks as PBIR |
| `IT_Spend_Convergence_Presentation_VN.pptx` | 10-slide findings presentation (in Vietnamese) |

## How to open it

1. Use Power BI Desktop with the **Power BI Project (.pbip)** and **TMDL** preview features enabled.
2. Get Microsoft's *IT Spend Analysis Sample* workbook. It is not included here because it is third-party data.
3. The Power Query sources point to my local copy of the file. In Transform data, update the file path in each table's source step to where you saved the workbook.
4. Open `IT_Spend_Convergence.pbip` and refresh.

## Author

Tran Phuc Truong (Quintus Maius), aspiring Data Analyst.
