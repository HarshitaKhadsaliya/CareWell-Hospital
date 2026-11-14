# 🏥 CareWell-Hospital

An end-to-end analytics project for **CareWell**, a 12-hospital healthcare network operating across **Gujarat, Maharashtra, Madhya Pradesh and Rajasthan**. The project covers four layers: **SQL** (engineering and automation), **Python** (cleaning and analysis), **Power BI** (executive dashboard) and **Excel** (KPI workbook and interactive dashboard).

## 📑 Table of Contents
1. [Problem Statement](#-problem-statement)
2. [Project Objectives](#-project-objectives)
3. [Dataset Overview](#-dataset-overview)
4. [KPI Definitions](#-kpi-definitions)
5. [Data Cleaning & Audit (Python)](#-data-cleaning--audit-python)
6. [Key Findings](#-key-findings)
7. [Python Layer](#-python-layer)
8. [SQL Layer](#-sql-layer)
9. [Power BI Layer](#-power-bi-layer)
10. [Excel Layer](#-excel-layer)
11. [Repository Structure](#-repository-structure)
12. [How to Run](#-how-to-run)
13. [Tech Stack](#-tech-stack)
14. [Notes & Limitations](#-notes--limitations)

---

## 📋 Problem Statement

CareWell faces operational bottlenecks and revenue leakage across its 12 facilities:

- **Bed Capacity Imbalances:** Network-wide occupancy averages only 61.1%–65.9%, yet individual wards frequently run at 90%–100% for consecutive days, causing emergency department boarding delays.
- **Extended Length of Stay:** Major departments exceed their target ALOS benchmarks, tying up high-demand inpatient beds.
- **Patient Readmissions:** Short-stay and specialty units show elevated 30-day readmission rates, hurting clinical quality and bed turnover.
- **Insurance Revenue Leakage:** 12.70% of insurance claims are rejected or short-paid because of missing documents, policy lapses and room-rent cap violations, a shortfall of **₹155.66 Crores**.

## 🎯 Project Objectives

| # | Objective | Description |
|---|-----------|-------------|
| 1 | **Occupancy & Bottleneck Tracking** | Track daily and month-end bed occupancy across all 12 hospitals; flag wards above 90% for ≥ 5 consecutive days. |
| 2 | **ALOS & Diagnosis Benchmarking** | Compare department ALOS to benchmarks and isolate the top 10 diagnosis–department pairs driving excess bed-days. |
| 3 | **Clinical Quality & Readmissions** | Measure 30-day readmission rates by department and attending physician. |
| 4 | **Revenue Cycle & Claims Recovery** | Quantify losses from rejected and short-paid claims by insurer, standardized reason and month, plus settlement turnaround. |
| 5 | **Executive Governance & Automation** | COO views with doctor-level drill-through, hospital-level Row-Level Security, ward alerts, integrity triggers and ACID ward transfers. |

## 📦 Dataset Overview

| Item | Value |
|------|-------|
| Admissions | 350,000 rows (184,456 distinct patients) |
| Bed-day snapshots | 87,600 rows (latest date 2026-09-30) |
| Insurance claims | 244,133 rows |
| Hospitals / Departments / Doctors / Wards | 12 / 18 / 900 / 120 |
| Admission date range | 2024-10-01 to 2026-09-30 |
| Total billed amount | ₹25,069,617,820 |
| Claims claimed / approved | ₹17,490,148,010 / ₹15,933,563,470 |

**Source files (CSV):** `admissions`, `bed_day_snapshot`, `bills`, `claims`, `departments`, `doctors`, `hospitals`, `patients`, `wards`.

## 📐 KPI Definitions

- **Occupancy %** = occupied beds / (total beds − blocked beds); zero available beds is handled explicitly.
- **High-occupancy alert** = occupancy above 90% for at least 5 consecutive days for the same hospital and ward.
- **ALOS** = elapsed days from admission to discharge. For open admissions, a documented reporting cut-off date is used.
- **Excess bed-days** = `MAX(0, LOS − department benchmark ALOS)`.
- **30-day readmission** = a subsequent admission for the same patient within 30 days after the previous discharge.
- **Rejected claim value** = claimed amount of rejected claims.
- **Short-paid value** = `MAX(0, claimed amount − approved amount)`.
- **Settlement time** = settlement date − submission date (use `NETWORKDAYS` if working days are required).

## 🧹 Data Cleaning & Audit (Python)

Before analysing 350,000 admissions, 87,600 bed snapshots and 244,133 claims, the Python pipeline enforces data hygiene:

| File | Cleaning performed |
|------|--------------------|
| `admissions.csv` | Kept only the 10 valid columns (removed stray spreadsheet helper columns); parsed timestamps with `dayfirst=True`; isolated **2,564** active inpatient stays (`discharge_ts` is null); quarantined **1,400** corrupt records where `discharge_ts < admit_ts`. |
| `bed_day_snapshot.csv` | Flagged **3,002** ward-days where `beds_occupied + beds_blocked > beds_total` to separate physical occupancy from total capacity pressure. |
| `bills.csv` | Validated 100% arithmetic reconciliation (`room_charges + pharmacy + procedures = total_amount`) across all 350,000 bills. |
| `claims.csv` | Standardized text casing and placeholder values (`0`, `NaN`) in `reason`; mapped **213,119** approved claims to "Approved / No Reason"; computed shortfall (`claimed_amt − approved_amt`) and settlement turnaround days. |

## 📊 Key Findings

### 1. Bed Occupancy & Overcrowding Streaks
- Network occupancy ranges from **61.1%** (CareWell Rajkot Central) to **65.9%** (CareWell Udaipur Central).
- **1,248** separate streaks of ≥ 5 consecutive days above 90% occupancy were identified at ward level.

### 2. ALOS vs Benchmarks
| Department | Actual ALOS | Benchmark | Variance |
|------------|-------------|-----------|----------|
| Cardiology | 5.66 d | 4.70 d | **+0.96 d** |
| ICU | 7.11 d | 6.20 d | **+0.91 d** |
| Oncology | 6.76 d | 5.90 d | **+0.86 d** |

**Top excess bed-day drivers:** Cardiology diagnoses I50 (29,524 excess bed-days), I25 (28,955) and I21 (28,725), followed by General Medicine (A90, J18, E11).

### 3. 30-Day Readmissions
- **Network baseline:** 5.51% of discharged patients return within 30 days.
- **High-risk departments:** Emergency Short Stay (7.81%), Dermatology (7.10%), Ophthalmology (7.08%), ENT (6.52%).
- **Physician outliers (min. 100 discharges):** Dr. Nidhi Sharma (Dermatology, 12.21%), Dr. Kavita Sharma (ENT, 11.70%), Dr. Nidhi Joshi (Ophthalmology, 11.11%), Dr. Vikram Trivedi (Emergency Short Stay, 10.53%).

### 4. Claims Revenue Leakage
- **₹155.66 Crores (₹1,556,584,540)** lost across **31,014** rejected or partially approved claims, out of ₹17.49 Billion claimed.
- **Average settlement time:** 33.0 days.
- **Top shortfall drivers:** Treatment Not Covered (₹26.53 Cr), Documents Missing (₹26.25 Cr), Policy Lapsed (₹26.09 Cr), Room Rent Cap Exceeded (₹25.91 Cr).

## 🐍 Python Layer

The analysis lives in `carwell.ipynb` (pandas, NumPy, Matplotlib, Seaborn).

| Step | Approach |
|------|----------|
| **Clean admissions** | Keep rows with a discharge on or after admission; compute `los_days` from the timestamps. |
| **High-occupancy streaks** | Sort by hospital, ward and date; flag `occupancy_pct > 90`; start a new `streak_id` whenever the flag changes within a ward (gaps-and-islands); keep streaks of 5+ days. |
| **ALOS and excess bed-days** | Join admissions to departments; `excess_days = max(0, los_days − benchmark_alos_days)`; group by department and diagnosis code and keep the top 10 by total excess bed-days. |
| **30-day readmissions** | Sort by patient and admit time; take the next admission with `shift(-1)`; flag a return within 0–30 days of discharge; aggregate by hospital, department and doctor. |
| **Claims leakage** | Sum `shortfall_amt` for claims with status `Rejected` or `Partially Approved`, broken down by cleaned reason. |
| **Row-level security** | `get_secure_coo_view()` gives the COO all 12 hospitals and a hospital manager only their assigned `hospital_id`; any other role raises `PermissionError`. |

```python
coo_global_df  = get_secure_coo_view(user_role='COO')
h01_manager_df = get_secure_coo_view(user_role='HOSPITAL_MANAGER', assigned_hospital_id='H01')
```

Notebook output confirms **244,133** claims analysed and total financial leakage of **₹1,556,584,540**.

## 🗄️ SQL Layer

Built for **PostgreSQL / Snowflake**.

### Advanced Analytical Queries
- **Gaps-and-Islands streak detection:** `ROW_NUMBER()` and running `SUM()` window functions detect wards above 90% occupancy for ≥ 5 consecutive days.
- **30-day readmission tracking:** `LEAD() OVER (PARTITION BY patient_id ORDER BY admit_ts)` computes the discharge-to-readmission interval (≤ 2,592,000 seconds).
- **Self-join physician hierarchy:** maps junior attending doctors to supervising seniors to evaluate mentorship and clinical outcomes.
- **ALOS variance modelling:** CTEs and `GREATEST(0, actual_los − benchmark_los)` quantify excess bed-days.

### Database Automation & Integrity
| Object | Purpose |
|--------|---------|
| `sp_ward_alert(hospital_id, snapshot_date)` | Stored procedure that raises operational alerts for any ward above 90% occupancy. |
| `trg_validate_discharge_ts` | Trigger that blocks `INSERT`/`UPDATE` on admissions if `discharge_ts < admit_ts`. |
| ACID ward-transfer transaction | Atomically updates the patient's `ward_id`, decrements the source ward's `beds_occupied` and increments the destination's. |
| RLS views | Restrict regional managers to their `hospital_id`; the COO role sees all hospitals. |

**BI views:** `vw_coo_hospital_summary`, `vw_coo_doctor_drillthrough`.

**SQL setup notes:** standardize dates (YYYY-MM-DD), cap logical bed-count anomalies (`beds_occupied + beds_blocked <= beds_total`) and impute NULL claim reasons for approved claims before loading the data into PostgreSQL / Snowflake.

## 📈 Power BI Layer

### Recommended one-page layout (16:9)
1. **Header:** CAREWELL HOSPITAL ANALYTICS with reporting-period subtitle.
2. **KPI row:** Total Admissions, Distinct Patients, Total Billed Amount, Current Inpatients, Latest Snapshot Bed Occupancy %, Average Length of Stay.
3. **Middle visuals:** Admissions over time (line), admissions by type (donut), billed amount by hospital (bar), bed occupancy by hospital/ward (bar).
4. **Bottom visual:** Claims by status, or claimed vs approved amount (clustered bar).
5. **Slicers:** Hospital, City, Department, Admission Type, Admission Date.

### Data preparation
1. **Get Data → Text/CSV:** import the nine operational CSVs (and the data dictionary if useful).
2. **Transform Data:** IDs/keys as Text, timestamps as Date/Time, monetary amounts as Decimal Number, bed counts as Whole Number.
3. Inspect the non-core helper columns in `admissions.csv` before using or removing them.
4. Preserve blank `discharge_ts` values and confirm their meaning with the data owner before labelling them "current inpatients".
5. Check duplicate keys, null foreign keys and orphan records before creating relationships.

### Model relationships (one-to-many, single direction)
```
hospitals[hospital_id]   → admissions[hospital_id]
departments[dept_id]     → admissions[dept_id]
doctors[doctor_id]       → admissions[doctor_id]
patients[patient_id]     → admissions[patient_id]
wards[ward_id]           → admissions[ward_id]   (only if ward_id is unique; else use a composite hospital+ward key)
admissions[admission_id] → bills[admission_id]
bills[bill_id]           → claims[bill_id]
hospitals[hospital_id]   → bed_day_snapshot[hospital_id]
```
Avoid many-to-many and ambiguous bidirectional relationships unless deliberate. For claims charts, use the **claims → bills → admissions** path for hospital/department context.

**Security:** configure RLS roles so hospital managers see only their facility while the COO has global drill-through.

**Modelling notes:** confirm `admission_id` is unique and bill multiplicity is expected; one bill may have zero, one or multiple claims, so validate before relating. For ward-level snapshots, use a unique ward key and check whether `ward_id` alone is globally unique. Use single-direction filtering from dimensions to facts wherever possible.

## 📗 Excel Layer

### Workbook sheets
| Sheet | Contents |
|-------|----------|
| Admissions | Patient, hospital, department, ward, diagnosis, admission and discharge |
| Bed day snapshot | Daily total, occupied and blocked beds |
| Bills | Billing information |
| Claims | Insurer, claim status, claimed/approved amounts and settlement |
| Departments | Department names and benchmark ALOS |
| Doctors | Doctor reference data |
| Hospitals | Hospital reference data |
| Patients | Patient reference data and date of birth |
| Wards | Ward reference data |

### Dashboard layout
1. Dark navy title bar with reporting period and filters.
2. KPI cards across the top.
3. Monthly admissions and occupancy combo chart.
4. Department actual ALOS vs benchmark chart.
5. Rejected vs short-paid claims by insurer.
6. 30-day readmission rate by department.

### Interactivity
Build PivotTables from the prepared data, insert **Hospital/Department slicers** and a **date Timeline**, then use **Slicer → Report Connections** to connect compatible PivotTables so one click filters every PivotTable.

## 📁 Repository Structure

```
CareWell-Hospital/
├── data/                       # CSV files (omit large or sensitive files from Git)
├── sql/                        # analytical queries, procedure, trigger, transfer, RLS views
├── carwell.ipynb               # Python cleaning, analysis and RLS function
├── powerbi/                    # Power BI report (.pbix)
├── excel/                      # KPI workbook and interactive dashboard (.xlsx)
└── README.md
```

## 🚀 How to Run

1. **Python:** install dependencies with `pip install pandas numpy matplotlib seaborn jupyter`, place the CSVs next to the notebook, open `carwell.ipynb` and run all cells top to bottom.
2. **SQL:** load the cleaned CSVs into PostgreSQL or Snowflake, then run the analytical queries, stored procedure, trigger, transfer transaction and RLS views.
3. **Power BI:** connect Power BI (or Tableau) to the CSVs or to `vw_coo_hospital_summary` and `vw_coo_doctor_drillthrough`, using session context variables for RLS; build the relationships above and apply the RLS roles.
4. **Excel:** open the workbook, refresh the PivotTables and use the slicers and timeline.

## 🛠️ Tech Stack

`SQL (PostgreSQL / Snowflake)` · `Python (pandas, NumPy, Matplotlib, Seaborn)` · `Jupyter` · `Power BI` · `Microsoft Excel (PivotTables, Slicers, Timeline)` · `Git / GitHub`

## 📝 Notes & Limitations

- Totals and counts are descriptive figures from the supplied CSVs, not validated business KPIs; the claims rejection rate is **12.70%** (an earlier SQL brief quoted about 13%). Billed amounts are billed values, not necessarily cash collected. Handle the extra `admissions.csv` columns (`Handling Missing Discharge Timestamps`, `Emergency Admissions Count:`) with care and verify their lineage; confirm currency, status definitions and inclusion rules with the data owner before external reporting.
- In the notebook, occupancy is calculated as `beds_occupied / beds_total`; blocked beds are flagged through `is_consistent` but not subtracted from capacity.
- Open inpatient stays and records with discharge before admission are excluded from LOS and readmission results.
- The notebook imports Matplotlib and Seaborn but does not yet draw plots.
