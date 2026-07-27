# Hospital Operations Analysis

## Overview

Analysis of hospital operations across 4 departments (Emergency, Surgery,
General Medicine, ICU) using patient, staffing, and weekly service data.
Goal: identify capacity bottlenecks, staffing gaps, and drivers of patient
satisfaction.

## Tools

Python (pandas) for data cleaning and analysis, Excel for dashboard
visualization.

## Data

- patients.csv — 1000 patient records (arrival/departure dates, service,
  satisfaction)
- services_weekly.csv — 208 weekly records per department (capacity,
  demand, refusals, events)
- staff.csv / staff_schedule.csv — staffing roster and weekly attendance

## Business Questions & Key Insights

1. **Which service has the longest stay, and does it affect satisfaction?**
   Surgery has the longest average stay (7.87 days), but length of stay
   has no meaningful correlation with satisfaction (r=0.07).

2. **Does patient age affect satisfaction or length of stay?**
   No meaningful correlation for either (r=-0.06 and r=-0.05).

3. **Which service has the highest refusal rate?**
   Emergency has a 77% average refusal rate — more than double the next
   highest (General Medicine, 35%). This is the single biggest capacity
   bottleneck in the hospital.

4. **How do special events affect operations?**
   Flu outbreaks nearly triple patient requests and quadruple refusals
   compared to normal weeks, while satisfaction drops to its lowest point.

5. **Does staff morale affect patient satisfaction?**
   No correlation (r=0.01) — patient satisfaction is independent of
   self-reported staff morale.

6. **Which service is most understaffed relative to demand?**
   General Medicine has the highest patient-to-staff ratio (1.79).
   Notably, Emergency's crisis is a capacity issue, not a staffing issue —
   its patient-to-staff ratio is actually low.

7. **Does staff attendance drop during high-stress weeks?**
   Yes — attendance falls to 15.8 (avg) during flu weeks vs 18.9 in
   normal weeks, compounding the demand spike during outbreaks.

## Data Note

Week 3 showed zero staff present across all services — flagged as a
likely data anomaly and excluded from staffing ratio calculations.

## Files

- hospital_ops_analysis.ipynb — full Python cleaning & analysis
- hospital_ops_dashboard.xlsx — Excel dashboard with 4 pivot charts
- README.md — this file
