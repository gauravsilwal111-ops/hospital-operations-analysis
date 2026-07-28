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

- `patients.csv` — 1000 patient records (arrival/departure dates, service, satisfaction)
- `services_weekly.csv` — 208 weekly records per department (capacity, demand, refusals, events)
- `staff.csv` / `staff_schedule.csv` — staffing roster and weekly attendance

## Data Cleaning (summary)

- Converted `arrival_date` / `departure_date` to datetime, derived `length_of_stay`
- Checked all 4 files for nulls and duplicates — none found
- Merged `staff_schedule` with `services_weekly` on week + service to compute staffing ratios
- Excluded week 3 from staffing calculations — all services showed 0 staff present, flagged as a likely data anomaly

## Business Questions, Code & Insights

**1. Which service has the longest average stay, and does it affect satisfaction?**

```python
service_stats = patients.groupby('service').agg(
    avg_length_of_stay=('length_of_stay', 'mean'),
    avg_satisfaction=('satisfaction', 'mean'),
    patient_count=('patient_id', 'count')
).sort_values('avg_length_of_stay', ascending=False)

correlation = patients['length_of_stay'].corr(patients['satisfaction'])
```

Surgery has the longest average stay (7.87 days), but length of stay has no
meaningful correlation with satisfaction (r = 0.07).

**2. Does patient age affect satisfaction or length of stay?**

```python
age_corr_satisfaction = patients['age'].corr(patients['satisfaction'])
age_corr_stay = patients['age'].corr(patients['length_of_stay'])
```

No meaningful correlation for either (r = -0.06 and r = -0.05).

**3. Which service has the highest patient refusal rate?**

```python
services['refusal_rate'] = services['patients_refused'] / services['patients_request']
refusal_by_service = services.groupby('service')['refusal_rate'].mean().sort_values(ascending=False)
```

Emergency has a 77% average refusal rate — more than double the next
highest (General Medicine, 35%). This is the single biggest capacity
bottleneck in the hospital.

**4. How do special events affect operations?**

```python
event_impact = services.groupby('event').agg(
    avg_requests=('patients_request', 'mean'),
    avg_admitted=('patients_admitted', 'mean'),
    avg_refused=('patients_refused', 'mean'),
    avg_satisfaction=('patient_satisfaction', 'mean')
)
```

Flu outbreaks nearly triple patient requests and quadruple refusals
compared to normal weeks, while satisfaction drops to its lowest point.

**5. Does staff morale affect patient satisfaction?**

```python
morale_corr = services['staff_morale'].corr(services['patient_satisfaction'])
```

No correlation (r = 0.01) — patient satisfaction is independent of
self-reported staff morale.

**6. Which service is most understaffed relative to demand?**

```python
weekly_attendance = schedule.groupby(['week', 'service'])['present'].sum().reset_index()
merged = services.merge(weekly_attendance, on=['week', 'service'])
merged['patients_per_staff'] = merged['patients_admitted'] / merged['staff_present']

merged_clean = merged[merged['staff_present'] > 0]  # excludes week 3 anomaly
staffing_ratio = merged_clean.groupby('service')['patients_per_staff'].mean().sort_values(ascending=False)
```

General Medicine has the highest patient-to-staff ratio (1.79). Notably,
Emergency's crisis is a capacity issue, not a staffing issue — its
patient-to-staff ratio is actually the lowest (0.43).

**7. Does staff attendance drop during high-stress weeks?**

```python
weekly_attendance_with_event = services[['week','event']].drop_duplicates().merge(weekly_attendance, on='week')
attendance_by_event = weekly_attendance_with_event.groupby('event')['staff_present'].mean().sort_values(ascending=False)
```

Yes — attendance falls to 15.8 (avg) during flu weeks vs 18.9 in normal
weeks, compounding the demand spike during outbreaks.

## Dashboard

Excel workbook (`hospital_ops_dashboard.xlsx`) with 4 pivot charts:
refusal rate by service, event impact, patients-per-staff by service, and
length of stay/satisfaction by service.

## Files

- `hospital_ops_analysis.ipynb` — full Python cleaning & analysis
- `hospital_ops_dashboard.xlsx` — Excel dashboard with 4 pivot charts
- `README.md` — this file
