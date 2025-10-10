## 📘 Project Overview  

This project investigates inconsistencies between **UWA’s internal Commonwealth Supported Place (CSP)** student enrolment records and the **Australian Government’s official funding allocations**.  
Using three official datasets —  
1. *Student Enrolment Data (2024)*  
2. *Indexed Government Funding Rates (PDF)*  
3. *Funding Agreement (2024)*  

The team built a **data warehouse**, conducted **exploratory data analysis (EDA)**, and designed a **reconciliation prototype** to explain and visualise funding mismatches.  

Our findings reveal that the **root cause of discrepancies** lies in **different data granularity and reporting logic** between UWA and the Government:  
- UWA’s internal data are recorded at the **unit / course** level,  
- Government allocations are calculated at the **Funding Cluster × Field of Education (FOE)** level.  

---
## 🏗️ Project Structure  

```
├── data/
│   ├── student_enrolments_2024.csv
│   ├── csp_allocation_2024.xlsx
│   ├── 2024_indexed_rates.pdf
│   └── funding_agreement_2024.pdf
│
├── sql/
│   ├── create_schema.sql
│   ├── dim_tables.sql
│   ├── fact_enrolment_insert.sql
│   ├── quality_checks.sql
│   └── aggregation_queries.sql
│
├── tableau/
│   ├── CSP_Reconciliation_Dashboard.twbx
│
├── model/
│   ├── logistic_regression_simulation.ipynb
│   ├── simulated_data.csv
│
└── README.md
```

---

## 🧩 Methodology  

### 1️⃣ ETL & Data Warehouse Design  

- Imported and standardised three data sources into PostgreSQL.  
- Built one **fact table (`fact_enrolment_2024`)** and multiple **dimension tables (`dim_foe`, `dim_cluster`, `dim_fees`)**.  
- Created relationships based on `FOE code` and `Funding Cluster`.  
- Designed the data model to support aggregation at *Funding Cluster × FOE* level.  

**Key SQL Logic Example:**  
```sql
SELECT
  ca.funding_cluster,
  se.unit_primary_foe_code AS foe_code,
  SUM(se.eftsl_2024) AS total_eftsl,
  SUM(regexp_replace(ca.commonwealth_contribution_2024, '[^0-9\.]', '', 'g')::numeric * se.eftsl_2024) AS total_gov_contr
FROM raw.student_enrolments_2024 se
JOIN raw.csp_allocation_2024 ca
  ON se.unit_primary_foe_code = ca.foe_code
GROUP BY 1, 2;
```
