# Healthcare-Analytics-Length-of-Stay-Project
Developed a reusable analytics layer for a Length of Stay (LOS) prediction model by transforming complex healthcare operational and financial data into clean, ML-ready feature views. These SQL assets served BI dashboards, ad-hoc analysis, and predictive modeling pipelines, balancing data quality, business logic, and performance.
# Core Contributions
1. Layered Data Transformation with CTEs
Built modular, maintainable SQL views using Common Table Expressions to enforce data quality and business rules.
``` SQL
WITH base AS (
    SELECT DISTINCT patient_id, diagnosis_code, metric_name, value, recorded_time
    FROM raw_clinical_data
    WHERE sort_order = 10  -- Primary diagnosis only
),
filtered AS (
    SELECT *
    FROM base
    WHERE recorded_time IS NOT NULL
      AND value IS NOT NULL
)
SELECT * FROM filtered;
```
2. Temporal Feature Engineering with Window Functions
Applied ROW_NUMBER() and PARTITION BY to capture the most recent patient metrics, preventing outdated signals in modeling.
``` SQL
SELECT patient_id, metric_name, value, recorded_time
FROM (
    SELECT *,
           ROW_NUMBER() OVER (
               PARTITION BY patient_id, metric_name 
               ORDER BY recorded_time DESC
           ) AS rnk
    FROM clinical_metrics
    WHERE value <> 0
) WHERE rnk = 1;
```
3. Business Logic & Time-Based Calculations
Engineered revenue-qualified service periods, fiscal-year alignment, and LOS-relevant day counts using window functions and conditional logic.
```SQL
SELECT DISTINCT
    patient_id,
    FIRST_VALUE(start_date) OVER (PARTITION BY patient_id ORDER BY start_date) AS first_service_date,
    DATEDIFF(day, start_date, 
             COALESCE(end_date, CURRENT_DATE())) + 1 AS service_days,
    CASE 
        WHEN (billed_revenue + adjustments) > 1 THEN 'Revenue Qualified'
        ELSE 'Non-Qualified'
    END AS revenue_flag
FROM hospice_revenue_data;
```
