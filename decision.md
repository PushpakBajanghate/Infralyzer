# Architecture & Modeling Decision Log

This log documents all architectural, data modeling, ETL, DAX, and design decisions made throughout the lifecycle of the **Infralyzer** Power BI project.

---

## 📝 Decision Log Table

| Date | Decision Made | Reason | Alternative Considered |
| :--- | :--- | :--- | :--- |
| 2026-09-26 | Project tracking initialized with `flow.md` and `decision.md` | Standardize progress tracking, governance, and architectural decision audit trail | Ad-hoc notes or untracked development |
| 2026-09-26 | **Star Schema Architecture Setup**:<br>• Single Fact: `Fact_Project_Execution`<br>• Dimension: `Dim_Projects` (PK: `project_id`)<br>• Dimension: `Dim_Sites` (PK: `location_type`)<br>• Relationships: `1:*` Single Cross-Filter (`Dim -> Fact`) | **1.** Adheres to Kimball dimensional best practices.<br>**2.** Maximizes VertiPaq columnar compression & memory efficiency.<br>**3.** Simplifies DAX filter context and prevents circular dependencies or ambiguity.<br>**4.** Enables clean slicing by project profile & deployment site. | **1. Single Flat Table**: High memory footprint, redundant text storage, complex DAX.<br>**2. Snowflake Schema**: Unnecessary normalization overhead for small dimension footprints.<br>**3. Bidirectional Filtering**: Can lead to unpredictable cross-filtering and severe visual performance penalties. |

### 🔗 Schema Relationship Architecture
```
      [Dim_Projects]                 [Dim_Sites]
      (PK: project_id)             (PK: location_type)
            │                               │
            │ (1)                           │ (1)
            │                               │
            ▼ (*)                           ▼ (*)
   ┌─────────────────────────────────────────────────┐
   │             Fact_Project_Execution              │
   │─────────────────────────────────────────────────│
   │ FK: project_id                                  │
   │ FK: location_type                               │
   │ Metrics: labor_cost, material_cost,             │
   │          equipment_cost, actual_spending,       │
   │          planned_budget, latency, efficiency... │
   └─────────────────────────────────────────────────┘
   Filter Direction: Single (Dimension filters Fact)
   Cardinality: 1 to Many (1:*)
```

---

## 📋 Decision Guidelines
When recording a new decision:
1. **Date**: Date the decision was agreed upon or implemented (`YYYY-MM-DD`).
2. **Decision Made**: Clear, concise statement of the modeling or architectural choice (e.g., star schema design, relationship direction, calculation logic).
3. **Reason**: The business, performance, or technical justification for selecting this approach.
4. **Alternative Considered**: Other options evaluated and the specific reasons why they were rejected (e.g., snowflake schema, bidirectional relationships, calculated columns instead of measures).
