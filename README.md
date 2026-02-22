# 📊 HR Attendance Analytics Dashboard — Power BI

> An interactive Power BI dashboard built to help HR teams monitor employee attendance, track work-from-home trends, and analyze sick leave patterns — turning messy multi-sheet Excel data into a clean, decision-ready report.

---

## 📌 Table of Contents

- [Project Overview](#project-overview)
- [Problem Statement](#problem-statement)
- [Objectives](#objectives)
- [Dataset](#dataset)
- [Tools & Technologies](#tools--technologies)
- [Project Workflow](#project-workflow)
  - [Step 1: Data Understanding](#step-1-data-understanding)
  - [Step 2: Data Transformation in Power Query](#step-2-data-transformation-in-power-query)
  - [Step 3: DAX Measures](#step-3-dax-measures)
  - [Step 4: Dashboard Design](#step-4-dashboard-design)
- [Dashboard Overview](#dashboard-overview)
- [Key Insights](#key-insights)
- [Business Recommendations](#business-recommendations)
- [Limitations](#limitations)
- [What I Learned](#what-i-learned)
- [Future Scope](#future-scope)
- [Project Structure](#project-structure)
- [Connect](#connect)

---

## Project Overview

HR teams often deal with attendance data scattered across multiple Excel sheets — one per month, dozens of columns, and inconsistent leave codes. Reading through this manually is time-consuming and error-prone, making it nearly impossible to spot trends or flag concerns in real time.

This project takes raw attendance data from AtliQ (a fictional company used for the course), transforms it into a structured data model, and delivers an interactive Power BI dashboard that gives HR managers instant visibility into workforce attendance behavior across a 3-month period.

---

## Problem Statement

The HR team at AtliQ maintained employee attendance records in wide-format Excel sheets — one sheet per month, with each date occupying a separate column and attendance codes (P, WFH, SL, etc.) entered manually for every employee.

The challenge: there was no easy way to answer questions like —
- What is the overall presence rate this quarter?
- Which employees are working from home the most?
- Are sick leaves spiking on specific days of the week?
- How is attendance trending over time?

The goal of this project was to consolidate, model, and visualize this data so that HR could answer these questions in seconds rather than hours.

---

## Objectives

- Consolidate 3 months of attendance data from separate Excel sheets into a single unified data model
- Calculate accurate HR metrics — Presence %, WFH %, and Sick Leave % — while correctly excluding weekends and public holidays from working day counts
- Build an interactive, filterable dashboard that works at both team and individual employee level
- Surface day-of-week and date-level trends to help HR identify behavioral patterns in attendance

---

## Dataset

| Detail | Info |
|---|---|
| **Company** | AtliQ (fictional dataset used for learning) |
| **Period Covered** | April 2022 — June 2022 |
| **Format** | Microsoft Excel (.xlsx) |
| **Sheets** | Apr 2022, May 2022, June 2022, Attendance Key |
| **Structure** | Wide format — one row per employee, one column per date |
| **Total Employees** | ~50+ |

**Attendance Codes used in the dataset:**

| Code | Meaning |
|---|---|
| P | Present (in office) |
| WFH | Work From Home |
| SL | Sick Leave |
| PL | Paid Leave |
| WO | Weekly Off |
| HO | Holiday Off |
| BL | Birthday Leave |
| ML | Menstrual Leave |
| LWP | Leave Without Pay |
| FFL | Floating Festival Leave |
| BRL | Bereavement Leave |

---

## Tools & Technologies

| Tool | Purpose |
|---|---|
| **Power BI Desktop** | Data modeling, DAX, and dashboard development |
| **Power Query (M Language)** | Data transformation — unpivoting, appending, cleaning |
| **DAX (Data Analysis Expressions)** | Custom calculated measures for HR metrics |
| **Microsoft Excel** | Raw data source |

---

## Project Workflow

### Step 1: Data Understanding

The raw data arrived as a wide-format Excel file with three monthly sheets. Each sheet had:
- Employee code and name in the first two columns
- One column per working date across the month
- Summary columns at the end counting totals per leave type
- An Attendance Key sheet defining all leave codes

Before building anything, the structure of the data was mapped out to understand how it needed to be reshaped for Power BI to process it correctly.

---

### Step 2: Data Transformation in Power Query

This was the most critical step. Wide-format data — where each date is its own column — cannot be analyzed effectively in Power BI without being reshaped first.

**Key transformation steps:**

**Unpivoting the date columns:** All date columns were unpivoted into two columns — `Date` and `Value` (the attendance code). This converted each employee's row into multiple rows, one per date, creating a long-format table that Power BI can work with properly.

**Appending three monthly sheets:** The three monthly sheets (Apr, May, Jun) were processed with the same transformation steps and then appended into a single unified table named `Final_Data`. This made it possible to analyze the full quarter in one view.

**Removing summary columns:** The Excel sheets contained pre-calculated summary columns (total present days, leave counts, etc.) at the end of each row. These were removed during transformation since the calculations would be rebuilt more accurately using DAX.

**Data type corrections:** Date columns were formatted correctly, and employee codes and names were cleaned for consistency.

---

### Step 3: DAX Measures

All key metrics were built as DAX measures rather than calculated columns, keeping the model efficient and dynamic — meaning they respond correctly to any slicer or filter applied on the dashboard.

---

**Measure 1 — WFH Count**

Counts the total number of WFH entries across the dataset.

```dax
WFH Count = SUM('Final_Data'[WHF Count])
```

---

**Measure 2 — WFH %**

Calculates WFH as a percentage of total present days. Uses DIVIDE to safely handle division by zero.

```dax
WFH % = DIVIDE([WFH Count],[Present Days],0)
```

---

**Measure 3 — Total Working Days**

Calculates working days by subtracting Weekly Off (WO) and Holiday Off (HO) entries from the total count. This ensures that weekends and public holidays are never counted as available working days.

```dax
Total Working Days =
Var totaldays = COUNT('Final_Data'[Value])
Var nonworkdays = CALCULATE(COUNT('Final_Data'[Value]),'Final_Data'[Value] in {"WO","HO"})
RETURN
totaldays - nonworkdays
```

---

**Measure 4 — Present Days**

Counts all days where the employee was physically present (P) and adds WFH days, since remote employees are counted as present from an attendance standpoint.

```dax
Present Days =
Var Presentdays = CALCULATE(COUNT('Final_Data'[Value]),'Final_Data'[Value]="P")
RETURN
Presentdays + [WFH Count]
```

---

**Measure 5 — Presence %**

Calculates the overall presence rate as a proportion of total working days. References the `Measure Table` to ensure the denominator is always the correct working day count.

```dax
Presence % = DIVIDE([Present Days], 'Measure Table'[Total Working Days],0)
```

---

**Measure 6 — SL Count**

Counts total sick leave entries across the dataset.

```dax
SL Count = SUM('Final_Data'[SL Count])
```

---

**Measure 7 — SL %**

Calculates sick leave as a percentage of total working days.

```dax
SL% = DIVIDE([SL Count],[Total Working Days],0)
```

---

### Step 4: Dashboard Design

The dashboard was designed with HR managers as the primary audience — people who need quick answers without having to dig through numbers. The layout follows a top-down reading pattern: summary first, then trends, then detail.

Design decisions made:
- Dark background with high-contrast card colors (green for presence, yellow for WFH, red for SL) to make KPIs instantly scannable
- Month-level slicers at the top for quick period filtering
- Trend charts placed centrally since time-based patterns are the most actionable insight for HR
- Day-of-week tables placed on the right as supporting context
- Employee drill-down table kept at the bottom for individual-level review

---

## Dashboard Overview

The dashboard is organized into four sections:

**KPI Summary Cards (Top Left)**
Three headline metrics covering the full selected period at a glance — Presence % (91.83%), WFH % (10.00%), and SL % (1.10%). These update dynamically based on the month slicer.

**Employee-Level Table (Middle Left)**
A sortable table showing Presence %, WFH %, and SL% for every employee individually. HR can quickly identify outliers — employees with unusually low presence or high sick leave.

**Raw Attendance Grid (Bottom Left)**
A date-level matrix showing the actual attendance code entered for each employee on each day, giving HR a detailed audit view when needed.

**Trend Charts (Center)**
Three area/line charts showing how Presence %, WFH %, and SL% moved day by day across April to June 2022. Key data point labels are shown at inflection points to make trends easy to read without hovering.

**Day of Week Breakdown (Right)**
Three separate tables showing average Presence %, WFH %, and SL% broken down by day of the week. This helps HR spot structural patterns — for example, whether Fridays consistently show lower attendance.

---

## Key Insights

- **Overall presence was strong at 91.83%** across the 3-month period, suggesting a generally healthy attendance culture at AtliQ.
- **WFH peaked in May 2022**, reaching as high as 23.44% on certain dates — indicating either a specific policy change or external circumstance during that period.
- **Monday recorded the highest presence rate (93.21%)** while Friday had the lowest (90.19%), suggesting a mild but consistent end-of-week dip in office attendance.
- **WFH is highest on Wednesdays (8.43%) and Fridays (13.01%)**, indicating employees tend to plan remote days around the middle and end of the week.
- **Sick leave is highest on Mondays (1.62%) and lowest on Fridays (0.70%)** — a pattern commonly seen in organizations where some employees extend weekends through Monday sick days.
- **SL% spiked in June 2022** reaching 5.42% on certain dates, which could warrant a follow-up investigation by HR.
- Some employees maintained 100% presence throughout the quarter, while others like Ana Little had noticeably lower presence (76.36%), which could prompt a check-in conversation.

---

## Business Recommendations

Based on the dashboard findings, here are suggested actions for the HR team:

**Attendance Policy:** The Friday presence dip and Monday sick leave spike are worth addressing in team conversations or policy nudges — not punitively, but as a way to understand if workload or morale is contributing to the pattern.

**WFH Policy Planning:** Since WFH usage peaks mid-week and on Fridays, HR and team leads could use this data to plan in-office collaboration days more deliberately — ensuring important meetings fall on days when most people are physically present.

**Sick Leave Spike Investigation:** The SL% surge in June 2022 should be cross-referenced with any organizational events, workload peaks, or seasonal health trends from that period to determine if it was situational or structural.

**Individual Follow-Ups:** Employees with presence below a certain threshold (e.g., under 80%) could be flagged for a supportive check-in to understand if there are underlying issues affecting their attendance.

---

## Limitations

- The dataset covers only 3 months, which limits the ability to identify long-term or seasonal trends.
- Half-day leave types were accounted for in the original Excel formulas (counted as 0.5), but DAX measures treated attendance as binary — a more precise model would handle fractional day logic directly in DAX.
- No department, team, or manager-level data was available in the dataset, so segmentation beyond individual employees was not possible.
- The dataset is fictional and adapted for educational purposes, so findings should not be interpreted as reflecting real organizational behavior.

---

## What I Learned

- How to unpivot wide-format Excel data into a long format suitable for Power BI data modeling — this is one of the most common real-world data preparation challenges.
- How to use Power Query to append multiple sheets into one unified table while applying consistent transformation logic across all of them.
- Writing DAX measures using variables (`VAR` / `RETURN`) for readability and performance, and using `CALCULATE` to apply filters within a measure context.
- The difference between a DAX measure and a calculated column, and why measures are preferred for dynamic, filter-responsive metrics.
- How to design a dashboard layout that serves both summary and drill-down needs without becoming cluttered.
- The importance of separating data transformation logic (Power Query) from calculation logic (DAX) for a cleaner, more maintainable model.

---

> 💡 To explore the dashboard interactively, download `Attendance_Dashboard.pbix` and open it in [Power BI Desktop](https://powerbi.microsoft.com/desktop/) (free to download).
---

*This project was completed as part of a structured data analytics course. The dataset is fictional and used for educational purposes only.*
