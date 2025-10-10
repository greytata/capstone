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

