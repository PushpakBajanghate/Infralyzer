# Architecture & Modeling Decision Log

This log documents all architectural, data modeling, ETL, DAX, and design decisions made throughout the lifecycle of the **Infralyzer** Power BI project.

---

## 📝 Decision Log Table

| Date | Decision Made | Reason | Alternative Considered |
| :--- | :--- | :--- | :--- |
| 2026-09-26 | Project tracking initialized with `flow.md` and `decision.md` | Standardize progress tracking, governance, and architectural decision audit trail | Ad-hoc notes or untracked development |
| 2026-09-26 | **Star Schema Architecture Setup**:<br>• Single Fact: `Fact_Project_Execution`<br>• Dimension: `Dim_Projects` (PK: `project_id`)<br>• Dimension: `Dim_Sites` (PK: `location_type`)<br>• Relationships: `1:*` Single Cross-Filter (`Dim -> Fact`) | **1.** Adheres to Kimball dimensional best practices.<br>**2.** Maximizes VertiPaq columnar compression & memory efficiency.<br>**3.** Simplifies DAX filter context and prevents circular dependencies or ambiguity.<br>**4.** Enables clean slicing by project profile & deployment site. | **1. Single Flat Table**: High memory footprint, redundant text storage, complex DAX.<br>**2. Snowflake Schema**: Unnecessary normalization overhead for small dimension footprints.<br>**3. Bidirectional Filtering**: Can lead to unpredictable cross-filtering and severe visual performance penalties. |
| 2026-09-26 | **Core DAX Business Measures Implementation**:<br>• Dedicated `_Measures` table<br>• Explicit base aggregations: `Total Planned Cost`, `Total Actual Cost`<br>• Measure branching: `Cost Variance`, `Cost Variance %`<br>• Safe division via `DIVIDE(..., 0)` | **1.** Explicit measures dynamically adapt to any visual filter context.<br>**2.** Measure branching enforces DRY (Don't Repeat Yourself) principle and single source of truth.<br>**3.** `DIVIDE()` guards against division-by-zero errors without runtime exceptions.<br>**4.** Eliminates row-level calculated columns to conserve RAM. | **1. Implicit Measures**: Lack centralized formatting and cannot be referenced in subsequent DAX expressions.<br>**2. Calculated Columns**: Stored in RAM, increases file size and refresh times.<br>**3. Slash Operator (`/`)**: Returns `Infinity` or blank/error on zero planned cost. |
| 2026-09-26 | **3-Tier Executive Dashboard Visual Layout & Theme**:<br>• Native visuals only (Cards, Clustered Bar, Matrix, Line)<br>• Industrial color system (Slate Grey, Steel Blue, Safety Orange accents) | **1.** High rendering speed and zero licensing overhead.<br>**2.** Logical top-down visual hierarchy (KPIs -> Breakdown -> Trend).<br>**3.** Colors reflect construction/engineering domain with high WCAG contrast. | **1. Custom Marketplace Visuals**: Heavy load overhead and enterprise security restrictions.<br>**2. High-Saturation Palette**: Causes cognitive overload and visual fatigue. |
| 2026-09-26 | **Power Query M-Code Staging & ETL Architecture**:<br>• Staged extraction (`_Staging_Raw_Data` with load disabled)<br>• Upstream null-coalescing and strict decimal/integer type casting<br>• Dimension deduplication | **1.** Centralizes upstream data ingestion and avoids duplicate file reads.<br>**2.** Prevents DAX calculation errors from null propagation.<br>**3.** Strict types allow optimal VertiPaq bit-packing compression. | **1. Direct Multiple File Loads**: Causes redundant disk I/O and slower refreshes.<br>**2. Ad-hoc DAX null-checks**: Adds runtime DAX CPU overhead instead of fixing at ETL layer. |
| 2026-09-26 | **Dynamic Hex-Based Conditional Formatting Measure** (`Schedule Status Color`):<br>• `#FF4D4D` (Over-budget / Critical risk)<br>• `#FFC000` (Moderate risk)<br>• `#28A745` (On-track / Feasible) | **1.** Single measure drives uniform visual alerting across Matrix, Cards, and Tables via 'Field Value' formatting.<br>**2.** Prevents hardcoded GUI rule duplication across multiple visuals. | **1. Manual GUI Color Rules**: Tedious to maintain and inconsistent when business threshold definitions evolve. |

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

### 📐 Core DAX Measure Formulas (`_Measures`)

#### 1. Total Planned Cost
```dax
Total Planned Cost = 
SUM(Fact_Project_Execution[planned_budget])
```
*Note: If renamed in Power Query, references `Fact_Project_Execution[Planned_Cost]`.*

#### 2. Total Actual Cost
```dax
Total Actual Cost = 
SUM(Fact_Project_Execution[actual_spending])
```
*Note: If renamed in Power Query, references `Fact_Project_Execution[Actual_Cost]`.*

#### 3. Cost Variance
```dax
Cost Variance = 
[Total Actual Cost] - [Total Planned Cost]
```

#### 4. Cost Variance %
```dax
Cost Variance % = 
DIVIDE(
    [Cost Variance], 
    [Total Planned Cost], 
    0
)
```

#### 5. Active Projects
```dax
Active Projects = 
DISTINCTCOUNT(Fact_Project_Execution[project_id])
```

#### 6. Schedule Status Color (Hex Code Dynamic Formatting)
```dax
Schedule Status Color = 
VAR VariancePct = [Cost Variance %]
RETURN
    SWITCH(
        TRUE(),
        VariancePct > 0.10, "#FF4D4D",  -- Over-budget / Critical risk (>10% overrun)
        VariancePct > 0.00, "#FFC000",  -- Moderate risk (0% to 10% overrun)
        "#28A745"                       -- On-track feasibility (<= 0% variance / under budget)
    )
```

#### 7. High-Risk Project Count
```dax
High-Risk Project Count = 
CALCULATE(
    DISTINCTCOUNT(Fact_Project_Execution[project_id]),
    Fact_Project_Execution[cost_status] IN {"High", "Critical"}
)
```

---

### ⚡ Power Query ETL M-Code Specification

#### `_Staging_Raw_Data` (Load Disabled)
```powerquery
let
    Source = Csv.Document(File.Contents("C:\Users\pushpak\OneDrive\Pushpak Placements\Projects\Infralyzer\edge_ai_cost_monitoring_dataset.csv"),[Delimiter=",", Columns=23, Encoding=65001, QuoteStyle=QuoteStyle.None]),
    #"Promoted Headers" = Table.PromoteHeaders(Source, [PromoteAllScalars=true]),
    #"Changed Type" = Table.TransformColumnTypes(#"Promoted Headers",{
        {"project_id", Int64.Type}, 
        {"day_of_project", Int64.Type}, 
        {"labor_hours", type number}, 
        {"labor_cost", type number}, 
        {"material_usage", type number}, 
        {"material_cost", type number}, 
        {"equipment_hours", type number}, 
        {"equipment_cost", type number}, 
        {"sensor_efficiency", type number}, 
        {"machine_downtime", type number}, 
        {"planned_budget", type number}, 
        {"actual_spending", type number}, 
        {"delay_days", Int64.Type}, 
        {"resource_utilization", type number}, 
        {"edge_latency_ms", type number}, 
        {"data_transfer_rate", type number}, 
        {"cost_variance", type number}, 
        {"efficiency_score", type number}, 
        {"cost_per_hour", type number}, 
        {"cost_status", type text}, 
        {"project_type", type text}, 
        {"enterprise_type", type text}, 
        {"location_type", type text}
    }),
    #"Replaced Nulls in Budget" = Table.ReplaceValue(#"Changed Type", null, 0, Replacer.ReplaceValue, {"planned_budget", "actual_spending", "labor_cost", "material_cost", "equipment_cost"})
in
    #"Replaced Nulls in Budget"
```

#### `Fact_Project_Execution`
```powerquery
let
    Source = _Staging_Raw_Data,
    #"Removed Descriptive Dimension Columns" = Table.RemoveColumns(Source, {"project_type", "enterprise_type"})
in
    #"Removed Descriptive Dimension Columns"
```

#### `Dim_Projects`
```powerquery
let
    Source = _Staging_Raw_Data,
    #"Selected Columns" = Table.SelectColumns(Source, {"project_id", "project_type", "enterprise_type"}),
    #"Removed Duplicates" = Table.Distinct(#"Selected Columns", {"project_id"})
in
    #"Removed Duplicates"
```

#### `Dim_Sites`
```powerquery
let
    Source = _Staging_Raw_Data,
    #"Selected Columns" = Table.SelectColumns(Source, {"location_type"}),
    #"Removed Duplicates" = Table.Distinct(#"Selected Columns")
in
    #"Removed Duplicates"
```

---

### 🎨 Visual Layout & Theme Architecture
- **Layout Structure**: 3-Tier Grid Hierarchy (Executive Cards -> Comparative Analysis -> Longitudinal Trend).
- **Color System**:
  - `Background Canvas`: Clean Neutral (`#F8F9FA`)
  - `Card / Container Fill`: Surface White (`#FFFFFF`) with subtle border (`#E2E8F0`)
  - `Primary Theme / Planned Cost`: Steel Blue (`#1E3A8A` / `#2B6CB0`)
  - `Secondary Accent / Actual Spending`: Safety Orange (`#DD6B20` / `#EA580C`)
  - `Neutral / Typography`: Slate Grey (`#2D3748` / `#4A5568`)
  - `Status Alerts`: Green (`#2F855A` for under budget), Safety Red/Orange (`#C53030` / `#DD6B20` for cost overruns).

---

## 📋 Decision Guidelines
When recording a new decision:
1. **Date**: Date the decision was agreed upon or implemented (`YYYY-MM-DD`).
2. **Decision Made**: Clear, concise statement of the modeling or architectural choice (e.g., star schema design, relationship direction, calculation logic).
3. **Reason**: The business, performance, or technical justification for selecting this approach.
4. **Alternative Considered**: Other options evaluated and the specific reasons why they were rejected (e.g., snowflake schema, bidirectional relationships, calculated columns instead of measures).
