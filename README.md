## Project Overview  

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
## Project Structure  

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

## Methodology  

### 1.ETL & Data Warehouse Design  

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

###  EDA & Insights  

**Objective:** Detect anomalies and patterns in CSP enrolments and government funding allocation.  

**Findings:**  
- **EFTSL Distribution:** Most students have EFTSL ≤ 1; outliers (EFTSL > 3) likely caused by unit-level duplication.  
- **Discipline Pattern:** Medicine and Health units dominate total funding (>30%), consistent with federal rates.  
- **Aggregate Check:** UWA total Commonwealth contribution ≈ **$161.9M** vs. Government official **$147.2M**,  
  explained by:  
  - Missing FOE mapping (366 records)  
  - No *grandfathered students* data  
  - Absence of student-level identifiers (possible aggregation bias)  

**Visualization (Tableau):**  
- Bar chart: Total funding by Funding Cluster  
- Pie chart: Proportion of government vs. student contributions  
- Highlight table: FOE vs. Δ (%) funding variance  

---

### 3️⃣ Modelling (Logistic Regression)  

**Goal:**  
Demonstrate how machine learning can identify funding discrepancy risks once full data is available.  

**Method:**  
A logistic regression model was built with an `error_flag` variable triggered when the payment difference between *expected* and *actual* funding exceeded **10%**.  
Since actual payments were not provided, a simulated dataset was used.  

**Performance Summary:**  
| Metric | Value |
|---------|--------|
| Accuracy | 0.87 |
| Precision | 0.95 |
| Recall | 0.70 |
| F1-score | 0.81 |

**Top predictors:**  (for example)
- **Unit Type:** Psychology and Medicine most prone to large discrepancies.  
- **Government Contribution Rate:** Slight positive correlation with error likelihood.  
- **EFTSL:** Larger units more likely to deviate.  

This model provides a **proof-of-concept** for future predictive audit and reconciliation systems.  

### 4. Final Solution  

#### (A) Data-Level Solution – Rebuild Comparable Aggregation Logic  
Problem:  
UWA data recorded by unit/course; Government calculates by *Funding Cluster × FOE*.  

Solution:  
- Re-aggregate existing *student enrolment* + *allocation* data into a **Funding Cluster × FOE summary table**  
- Output annual reconciliation reference table:  

| Cluster | FOE | Total_EFTSL | UWA_Internal_$ | Gov_Indexed_$ | Δ ($) | Δ (%) |
|----------|-----|--------------|----------------|----------------|-------|-------|
| 1 | 090701 | 120.25 | 1,234,500 | 1,210,000 | +24,500 | 2.0% |

Deliverables:  
- Reproducible SQL/Excel/Power BI report  
- Annual finance audit template for UWA  

#### (B) Prototype Dashboard (Governance Layer)  
A lightweight **reconciliation prototype** was designed using mock UI to demonstrate:  
- Upload of student data + government rate tables  
- Auto-calculation of variance by FOE / Cluster  
- Export of reconciliation summary report  

This can be implemented in Excel, Power BI, or a web-based internal audit tool.  

---


## 5. Limitations  

1. **Limited Data Coverage:** Only 2024 data available; no international or self-funded students included.  
2. **Missing Special Cases:** No details on *grandfathered students* or *special FOE codes* (366 unmapped).  
3. **Simulated Model Data:** Logistic regression used synthetic data; results are demonstrative only.  
4. **No Student ID Field:** Prevents direct validation of duplication or per-student aggregation.

## 6. Future Work  

- Integrate real student-level payment data for full discrepancy modelling.  
- Automate reconciliation in an annual ETL pipeline.  
- Extend dashboard prototype with predictive anomaly detection.  
- Improve FOE–Cluster mapping accuracy through external metadata.
-  
##  Key Takeaways  

- **Main finding:** Misalignment in data granularity (unit-level vs. cluster-level) explains funding mismatch.  
- **Main contribution:** Delivered a **repeatable, auditable reconciliation process** using only three datasets.  
- **Impact:** Supports UWA’s goals of **financial integrity**, **data transparency**, and **AI governance alignment**.  
 

