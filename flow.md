# Infralyzer - Project Flow & Milestone Tracking

Project execution checklist and progress tracking for the **Infralyzer** Power BI solution.

---

## 📊 Overall Progress Summary
- [ ] **Phase 1: Data Preparation** (Status: Not Started)
- [ ] **Phase 2: Star Schema Modeling** (Status: Not Started)
- [ ] **Phase 3: Basic & Advanced DAX** (Status: Not Started)
- [ ] **Phase 4: Report Layout & Data Visualization** (Status: Not Started)
- [ ] **Phase 5: Interview Preparation & Portfolio Packaging** (Status: Not Started)

---

## Phase 1: Data Preparation
- [ ] Connect to raw source data / extract files
- [ ] Data profiling & quality assessment (missing values, duplicates, outliers)
- [ ] Column data type verification & formatting
- [ ] Column renaming and standardization (business-friendly terminology)
- [ ] Power Query ETL transformations & cleansing steps documented
- [ ] Applied steps optimization & query load staging / disabling staging loads

---

## Phase 2: Star Schema Modeling
- [ ] Identify business processes, grain, and key metrics
- [ ] Separate dimension tables and fact tables
- [ ] Define primary keys (PK) and foreign keys (FK) / surrogate keys
- [ ] Establish 1-to-many (`1:*`) relationships with single directional filters
- [ ] Avoid bidirectional relationships and resolve ambiguity (inactive relationships or role-playing dimensions)
- [ ] Hide foreign key columns and technical keys from the Report view

---

## Phase 3: Basic & Advanced DAX
- [ ] Create dedicated `_Measures` table for organized measure storage
- [ ] Define core base measures (SUM, COUNT, DISTINCTCOUNT)
- [ ] Implement business KPI calculations & ratios (e.g., margins, utilization, efficiency)
- [ ] Build Time Intelligence measures (YTD, MTD, YoY growth, rolling averages)
- [ ] Validate measure performance using DAX Studio / Performance Analyzer
- [ ] Add formatting strings and descriptions/comments to measures

---

## Phase 4: Report Layout & Data Visualization
- [ ] Define visual theme, typography, color palette, and canvas grid
- [ ] Design executive summary / KPI dashboard overview page
- [ ] Design detailed analytical deep-dive pages
- [ ] Implement user navigation, bookmarks, and slicer/filter panels
- [ ] Configure tooltips, drill-through pages, and conditional formatting
- [ ] Mobile layout / responsive view optimization (if applicable)

---

## Phase 5: Interview Prep & Portfolio Packaging
- [ ] Document project architecture, data lineage, and business impact
- [ ] Capture dashboard screenshots and create demonstration GIF/walkthrough
- [ ] Prepare technical Q&A (data modeling decisions, complex DAX explanations, query tuning)
- [ ] Finalize README and documentation for GitHub portfolio presentation
