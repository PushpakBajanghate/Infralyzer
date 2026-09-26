# Infralyzer - Project Flow & Milestone Tracking

Project execution checklist and progress tracking for the **Infralyzer** Power BI solution.

---

## 📊 Overall Progress Summary
- [x] **Phase 1: Data Preparation** (Status: Completed)
- [x] **Phase 2: Star Schema Modeling** (Status: Completed)
- [x] **Phase 3: Basic & Advanced DAX** (Status: Completed)
- [x] **Phase 4: Report Layout & Data Visualization** (Status: Completed)
- [ ] **Phase 5: Interview Preparation & Portfolio Packaging** (Status: In Progress)

---

## Phase 1: Data Preparation
- [x] Connect to raw source data / extract files (`edge_ai_cost_monitoring_dataset.csv`)
- [x] Data profiling & quality assessment (missing values, duplicates, outliers)
- [x] Column data type verification & formatting
- [x] Column renaming and standardization (business-friendly terminology)
- [x] Power Query ETL transformations & cleansing steps documented
- [x] Applied steps optimization & query load staging / disabling staging loads

---

## Phase 2: Star Schema Modeling
- [x] Identify business processes, grain, and key metrics
- [x] Separate dimension tables and fact tables (`Fact_Project_Execution`, `Dim_Projects`, `Dim_Sites`)
- [x] Define primary keys (PK) and foreign keys (FK) / surrogate keys
- [x] Establish 1-to-many (`1:*`) relationships with single directional filters
- [x] Avoid bidirectional relationships and resolve ambiguity (inactive relationships or role-playing dimensions)
- [x] Hide foreign key columns and technical keys from the Report view

---

## Phase 3: Basic & Advanced DAX
- [x] Create dedicated `_Measures` table for organized measure storage
- [x] Define core base measures (`Total Planned Cost`, `Total Actual Cost`)
- [x] Implement business KPI calculations & ratios (`Cost Variance`, `Cost Variance %`)
- [ ] Build Time Intelligence measures (YTD, MTD, YoY growth, rolling averages)
- [ ] Validate measure performance using DAX Studio / Performance Analyzer
- [x] Add formatting strings and descriptions/comments to measures

---

## Phase 4: Report Layout & Data Visualization
- [x] Define visual theme, typography, color palette, and canvas grid (Slate Grey, Steel Blue, Safety Orange)
- [x] Design executive summary / KPI dashboard overview page (Top 4 KPI Cards)
- [x] Design analytical middle section (Clustered Bar Chart & Matrix Table)
- [x] Design bottom trend section (Line Chart showing cost over time)
- [x] Configure tooltips, drill-through pages, and conditional formatting
- [ ] Mobile layout / responsive view optimization (if applicable)

---

## Phase 5: Interview Prep & Portfolio Packaging
- [ ] Document project architecture, data lineage, and business impact
- [ ] Capture dashboard screenshots and create demonstration GIF/walkthrough
- [ ] Prepare technical Q&A (data modeling decisions, complex DAX explanations, query tuning)
- [ ] Finalize README and documentation for GitHub portfolio presentation
