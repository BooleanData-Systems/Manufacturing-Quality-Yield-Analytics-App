# Manufacturing Quality & Yield Analytics

AI-powered manufacturing quality intelligence for yield optimization, defect reduction, and waste elimination.

## Overview

This Native App provides a comprehensive Manufacturing Quality & Yield dashboard covering:

- **Plant Performance** — Real-time yield trends, production KPIs, plant comparisons, target attainment
- **Defect Analysis** — Defect types, severity, root causes, trend analysis, machine correlation
- **Waste (Scrap & Rework)** — Cost breakdown, waste trends, product/reason analysis
- **Machines & Shifts** — Machine defect ranking, shift yield comparison, category heatmaps
- **Quality Heatmap** — Plant vs product yield and cost cross-analysis
- **AI Assistant** — Cortex-powered recommendations, 30-day improvement plans, natural language Q&A

## Configuration Steps

1. **Install the application** from the Snowflake Marketplace.
2. **Bind table references** — On first launch, the app will prompt you to bind each of the 8 required table references via the Snowsight permissions dialog.
3. **Verify data** — Once bound, the dashboard loads automatically. Check that charts populate with your data.
4. **Explore** — Use the sidebar to filter by Production House, Product, Line, Shift, Year/Month, and Date Range.

## Stored Procedures & UDFs

| Object | Type | Description |
|--------|------|-------------|
| `config.register_single_reference` | Procedure | Callback invoked by Snowsight when a consumer binds/unbinds a table reference |
| `config.setup_views` | Procedure | Creates internal views pointing to bound reference tables |

## Required Privileges

The app requests **SELECT** on each bound table. No other privileges are required. All analytics run in-place on your Snowflake warehouse — no data leaves your account.

## Example SQL Commands

```sql
GRANT USAGE ON DATABASE <YOUR_DB> TO APPLICATION QUALITY_YIELD_ANALYTICS_APP;
GRANT USAGE ON SCHEMA <YOUR_DB>.<YOUR_SCHEMA> TO APPLICATION QUALITY_YIELD_ANALYTICS_APP;
GRANT SELECT ON ALL TABLES IN SCHEMA <YOUR_DB>.<YOUR_SCHEMA> TO APPLICATION QUALITY_YIELD_ANALYTICS_APP;

CALL QUALITY_YIELD_ANALYTICS_APP.config.register_single_reference(
  'FACT_PRODUCTION_TABLE', 'ADD',
  SYSTEM$REFERENCE('TABLE', '<YOUR_DB>.<SCHEMA>.FACT_PRODUCTION', 'PERSISTENT', 'SELECT')
);
```

Repeat for each of the 8 references.

## Required Tables

| Reference | Key Columns |
|-----------|-------------|
| FACT_PRODUCTION_TABLE | ORDER_DATE (DATE), ORDER_TIMESTAMP (TIMESTAMP), PLANT_ID, PRODUCT_ID, SHIFT_NAME, LINE_ID, BATCH_ID, UNITS_PRODUCED, UNITS_PASSED, UNITS_FAILED, PLANNED_QTY, PRODUCED_QTY, GOOD_QTY, REJECTED_QTY, YIELD_PERCENTAGE, FIRST_PASS_YIELD_PCT, DEFECT_COST, UNIT_COST |
| FACT_DEFECT_TABLE | DEFECT_ID, DEFECT_DATE (DATE), DEFECT_YEAR, DEFECT_MONTH, PLANT_ID, PRODUCT_ID, DEFECT_TYPE, DEFECT_CATEGORY, SEVERITY, DEFECT_QTY, ROOT_CAUSE, MACHINE_ID, LINE_ID, SHIFT_NAME, DETECTED_BY, DEFECT_COST |
| FACT_INSPECTION_TABLE | INSPECTION_ID, INSPECTION_DATE (DATE), INSPECTION_YEAR, INSPECTION_MONTH, PLANT_ID, PRODUCT_ID, INSPECTION_TYPE, RESULT, SCORE, NOTES |
| FACT_SCRAP_REWORK_TABLE | RECORD_ID, RECORD_DATE (DATE), RECORD_YEAR, RECORD_MONTH, PLANT_ID, PRODUCT_ID, RECORD_TYPE, QUANTITY, COST, REASON, MACHINE_ID, LINE_ID, SHIFT_NAME |
| DIM_PLANT_TABLE | PLANT_ID, PLANT_NAME, PLANT_TYPE, CITY, STATE_CODE, REGION, SUB_REGION, DAILY_CAPACITY, START_DATE (DATE), STATUS |
| DIM_PRODUCT_TABLE | PRODUCT_ID, PRODUCT_NAME, PRODUCT_CATEGORY |
| DIM_MACHINE_TABLE | MACHINE_ID, MACHINE_NAME, MACHINE_TYPE |
| DIM_DATE_TABLE | DATE_KEY (DATE), YEAR, QUARTER, MONTH, MONTH_NAME |

## Expected Categorical Column Values

### Shift & Line Fields

| Column | Table(s) | Expected Values |
|--------|----------|-----------------|
| `SHIFT_NAME` | FACT_PRODUCTION, FACT_DEFECT, FACT_SCRAP_REWORK | `A` (Morning), `B` (Afternoon), `C` (Night) |
| `LINE_ID` | FACT_PRODUCTION, FACT_DEFECT, FACT_SCRAP_REWORK | `L01`, `L02`, `L03`, `L04`, `L05` |

### Defect Fields

| Column | Table(s) | Expected Values |
|--------|----------|-----------------|
| `SEVERITY` | FACT_DEFECT | `Critical`, `Major`, `Minor`, `Cosmetic` |
| `DEFECT_TYPE` | FACT_DEFECT | Any string (e.g., `Dimensional`, `Surface`, `Electrical`, `Mechanical`, `Assembly`, `Material`) |
| `DEFECT_CATEGORY` | FACT_DEFECT | Any string (e.g., `Process`, `Material`, `Equipment`, `Human Error`, `Design`) |
| `ROOT_CAUSE` | FACT_DEFECT | Any string (e.g., `Machine Calibration`, `Operator Error`, `Raw Material Defect`, `Tool Wear`, `Temperature Variance`) |
| `DETECTED_BY` | FACT_DEFECT | `Automated Inspection`, `Manual Inspection`, `Customer Feedback`, `In-Process Check` |

### Scrap & Rework Fields

| Column | Table(s) | Expected Values |
|--------|----------|-----------------|
| `RECORD_TYPE` | FACT_SCRAP_REWORK | `SCRAP`, `REWORK` |
| `REASON` | FACT_SCRAP_REWORK | Any string (e.g., `Out of Tolerance`, `Surface Defect`, `Wrong Component`, `Contamination`, `Misalignment`) |

### Inspection Fields

| Column | Table(s) | Expected Values |
|--------|----------|-----------------|
| `INSPECTION_TYPE` | FACT_INSPECTION | `Incoming`, `In-Process`, `Final`, `Customer Audit` |
| `RESULT` | FACT_INSPECTION | `PASS`, `FAIL`, `CONDITIONAL` |

### Dimension Fields

| Column | Table(s) | Expected Values |
|--------|----------|-----------------|
| `PLANT_TYPE` | DIM_PLANT | Any string (e.g., `Electronics`, `Automotive`, `Pharmaceutical`, `General Manufacturing`) |
| `REGION` | DIM_PLANT | Any string (e.g., `Northeast`, `Southeast`, `Midwest`, `West`, `Southwest`) |
| `STATUS` | DIM_PLANT | `Active`, `Inactive`, `Maintenance` |
| `PRODUCT_CATEGORY` | DIM_PRODUCT | Any string (e.g., `PCB Assembly`, `Sensors`, `Motors`, `Connectors`, `Power Units`) |
| `MACHINE_TYPE` | DIM_MACHINE | Any string (e.g., `CNC`, `SMT`, `AOI`, `Press`, `Laser`, `Assembly Robot`) |

## Permissions

The app requests SELECT access on bound tables. All analytics run in-place on your Snowflake warehouse. No data leaves your account.

## Version History

| Version | Date | Notes |
|---------|------|-------|
| 1.0.0 | 2026-05-21 | Initial marketplace release |
