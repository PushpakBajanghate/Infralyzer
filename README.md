# Infralyzer: Construction Project Progress & Financial Cost Feasibility Dashboard

![Power BI Theme](https://img.shields.io/badge/Power%20BI-Engine-F2C811?logo=powerbi&logoColor=black)
![Architecture](https://img.shields.io/badge/Data%20Model-Star%20Schema-1E3A8A)
![Status](https://img.shields.io/badge/Telemetry-Real--Time-28A745)
![License](https://img.shields.io/badge/License-MIT-gray)

**Infralyzer** is an enterprise-grade construction project tracking and cost feasibility monitoring solution. Built upon 4,000 real-world project telemetry records combining Edge AI metrics and industrial cost structures, it provides executive stakeholders with real-time visibility into budget burn rates, financial variance, schedule delay risk, and operational feasibility.

---

## 🌟 Interactive Real-Time Dashboard

The repository includes a live, self-contained interactive dashboard modeled on Power BI's 3-tier reporting canvas:

- **Top Executive Banner**: 4 KPI Cards (`Total Planned Budget`, `Total Actual Cost`, `Cost Variance & %`, `High-Risk Project Count`).
- **Middle Section**: Clustered Bar Chart (Planned vs. Actual by Project Category) alongside an Execution Feasibility Risk Matrix.
- **Bottom Section**: Longitudinal Cost Trajectory Line Chart tracking cumulative expenditure vs. budget over project timelines.
- **Real-Time Streaming Engine**: Live 2-second telemetry simulation engine with dynamic conditional formatting (`#28A745` Green, `#FFC000` Amber, `#FF4D4D` Red).
- **Interactive Slicers**: Multi-dimensional filtering by Project Type, Enterprise Scale, Deployment Site, and Risk Status.

### How to Run Locally:
```bash
# Option 1: Double-click index.html directly in any web browser!

# Option 2: Run via the included Python local server:
python run_dashboard.py
```
*Access in browser at:* `http://localhost:8080/index.html`

---

## 📐 Data Architecture & Star Schema

The raw dataset (`edge_ai_cost_monitoring_dataset.csv`, 4,000 records) is separated into a Kimball Star Schema:

```
        ┌────────────────────────────┐              ┌────────────────────────────┐
        │        Dim_Projects        │              │         Dim_Sites          │
        ├────────────────────────────┤              ├────────────────────────────┤
        │ PK: project_id             │              │ PK: location_type          │
        │     project_type           │              │     (Urban/Rural/Semi)     │
        │     enterprise_type        │              └────────────────────────────┘
        └──────────────┬─────────────┘                            │
                       │ (1)                                      │ (1)
                       │                                          │
                       │ Single                                   │ Single
                       │                                          │
                       ▼ (*)                                      ▼ (*)
        ┌────────────────────────────────────────────────────────────────────────┐
        │                         Fact_Project_Execution                         │
        ├────────────────────────────────────────────────────────────────────────┤
        │ FK: project_id                                                         │
        │ FK: location_type                                                      │
        │     day_of_project, labor_hours, labor_cost, material_usage,           │
        │     material_cost, equipment_hours, equipment_cost, sensor_efficiency, │
        │     machine_downtime, planned_budget, actual_spending, delay_days,     │
        │     resource_utilization, edge_latency_ms, data_transfer_rate,         │
        │     cost_variance, efficiency_score, cost_per_hour, cost_status        │
        └────────────────────────────────────────────────────────────────────────┘
```

---

## ⚡ Production-Ready DAX Measures

Stored in a dedicated `_Measures` table:

1. **Total Planned Budget**:
   ```dax
   Total Planned Budget = SUM(Fact_Project_Execution[planned_budget])
   ```
2. **Total Actual Cost**:
   ```dax
   Total Actual Cost = SUM(Fact_Project_Execution[actual_spending])
   ```
3. **Cost Variance**:
   ```dax
   Cost Variance = [Total Actual Cost] - [Total Planned Budget]
   ```
4. **Cost Variance %**:
   ```dax
   Cost Variance % = DIVIDE([Cost Variance], [Total Planned Budget], 0)
   ```
5. **Schedule Status Color (Hex Code Dynamic Formatting)**:
   ```dax
   Schedule Status Color = 
   VAR VariancePct = [Cost Variance %]
   RETURN
       SWITCH(
           TRUE(),
           VariancePct > 0.10, "#FF4D4D",  -- Over-budget / Critical Risk
           VariancePct > 0.00, "#FFC000",  -- Moderate Risk
           "#28A745"                       -- On-track Feasibility / Under Budget
       )
   ```
6. **High-Risk Project Count**:
   ```dax
   High-Risk Project Count = 
   CALCULATE(
       DISTINCTCOUNT(Fact_Project_Execution[project_id]),
       Fact_Project_Execution[cost_status] IN {"High", "Critical"}
   )
   ```

---

## 🗂️ Project Tracking & Decision Audit Trail

- [**`flow.md`**](flow.md): Complete milestone progress checklist covering Data Prep, Modeling, DAX, Layout, and Portfolio Packaging.
- [**`decision.md`**](decision.md): Full architectural decision log documenting VertiPaq optimizations, M-code staging pipelines, and visual UI design choices.
