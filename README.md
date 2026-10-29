# Clinic dashboard

The clinic names are now **Clinic A**, **Clinic B**, and **Clinic C**. All 26 supplied appointment records are fabricated examples.

## Files and use

- `mock_dashboard_clinic.html`: complete dashboard with embedded sample data and a CSV import control.
- `clinic_data.csv`: editable input file containing the same sample records.
- `README.md`: input schema, usage, and metric definitions.

1. Extract the ZIP into a folder.
2. Double-click `mock_dashboard_clinic.html` to open it in a modern browser.
3. Click **Load data CSV** and select `clinic_data.csv` or your replacement file.
4. Select a clinic, start/end months, and schedule day, then click **Apply**.

Import replaces the full dataset; it does not append. On import, filters reset to the first clinic and the full date range in the file. The selected schedule day starts at that clinic's earliest appointment. Clinic names in the dropdown come directly from your CSV. Edits to the CSV require importing it again. Reloading the HTML restores the embedded sample; **Restore sample data** also restores it. Nothing is automatically saved in the browser.

The HTML can open directly from disk; no server or Python is needed. Charts use Chart.js from a CDN and require internet access. If Chart.js cannot load, a message appears and the table and numeric KPIs continue to work. An optional Google font also uses the internet. The CSV is read in the browser; this dashboard has no data upload endpoint.

## One CSV is sufficient

Use **one row per appointment**, combining all clinics, dates, patients, and visit types. The dashboard calculates the schedule, KPIs, reason breakdown, and monthly trends from these same rows. Do not supply separate monthly summaries or schedule files: that would risk counting the same appointment twice.

Separate patient and clinic lookup tables could help a larger system maintain demographics, clinic metadata, or changing names. A separate appointments/encounters design would also help if you need to distinguish scheduled, completed, cancelled, and missed appointments. Those additional tables and statuses are not supported by this version and are not needed for the current display.

## Expected CSV schema

Use UTF-8 CSV with a header row. Headers are case sensitive and must match the nine names below exactly. Their order may vary. All columns must be present; only `reason` may be empty. Extra columns are rejected. IDs and names must not have surrounding spaces.

| Column | Type / format | Meaning and rules |
| --- | --- | --- |
| `visitId` | Nonempty text | Unique appointment ID across the entire file. Never repeat it, even in another clinic. |
| `patientId` | Nonempty text | Stable patient ID. Reuse for the same person across dates and clinics. Different people need different IDs even when names match. |
| `date` | `YYYY-MM-DD` | Real calendar date, including leading zeros, e.g. `2026-01-07`. |
| `time` | `HH:MM` | 24-hour local appointment time, e.g. `09:30` or `15:00`. |
| `patient` | Nonempty text | Display name or pseudonym shown in the schedule. Use the same display value for every row with the same patient ID. |
| `clinic` | Nonempty text | Clinic display name. Use exactly the same spelling/case for all rows belonging to one clinic. |
| `visitType` | `New` or `Repeat` | Classification supplied by your source system. Case sensitive. |
| `reason` | `A`, `B`, `C`, or empty | Consult category. Required as A/B/C if `consult=true`; must be empty if `consult=false`. |
| `consult` | `true` or `false` | Whether this appointment is a consult. Values are case insensitive; 1/0 and yes/no are rejected. |

A/B/C are generic category codes. Your export must map actual consult reasons to these three categories consistently. This version does not support additional reason categories or configurable reason labels.

Example:

```csv
visitId,patientId,date,time,patient,clinic,visitType,reason,consult
V0001,P001,2026-01-07,09:00,Patient One,Clinic A,New,A,true
V0002,P001,2026-02-04,09:30,Patient One,Clinic A,Repeat,,false
V0003,P002,2026-02-04,10:00,"Example, Patient",Clinic B,New,B,true
```

The importer supports quoted commas, escaped double quotes, quoted multiline cells, Windows or Unix line endings, and a UTF-8 BOM. Write a literal quote inside a quoted cell as two quotes. Completely empty lines are ignored. At least one appointment is required.

An invalid import displays the offending record/rule and retains the previous dataset. Exact duplicate appointment IDs are rejected; duplicate appointments with different IDs cannot be detected reliably, so deduplicate them before export. Dates/times must be exported as text in the specified format; Excel may otherwise change dates or strip leading zeros from IDs.

## How the dashboard counts

| Display | Definition |
| --- | --- |
| Daily schedule | All rows for the selected clinic and exact schedule date, sorted by time. This date is independent of the monthly filter; a day without records displays an empty schedule. |
| Consults KPI | Number of rows for the selected clinic and inclusive month range with `consult=true` and reason A/B/C. |
| New patients KPI | Number of distinct `patientId` values with at least one `visitType=New` row at the selected clinic in the selected month range. The dashboard trusts your classification; it does not infer first-ever visits from incomplete history. |
| Repeat visits KPI | Number of appointment rows in the selected clinic/range for which the same patient ID has a prior appointment in that same clinic on an earlier calendar date, at most **183 days** earlier. Uses all imported history, including dates outside the selected month range. Same-day appointments do not qualify as prior visits. This is computed independently of `visitType`. |
| Consult reason chart | Counts of eligible consult rows by A, B, and C in the selected clinic/range. |
| Visits by month | All appointment rows in each month for the selected clinic/range. |
| Consults by month | Eligible consult rows in each month for the selected clinic/range. |

The repeat threshold is a fixed 183 days, preserving the original dashboard's definition; it is not six calendar months. Include at least the preceding 183 days of appointment history to classify returns at the start of your reporting period. Missing history can undercount repeat visits.

There is no appointment status column. Every imported row counts in monthly totals, including future appointments if they fall within the selected range. Export only appointments you intend to count. These totals do not distinguish completed encounters from scheduled appointments, cancellations, or no-shows. For completed-visit reporting, filter the export upstream. In that case, the daily schedule also shows only those exported records.

Missing months display zero recorded rows, which does not prove zero actual activity if the export is incomplete. Use consistent clinic-local dates/times; this dashboard does not convert time zones.

For demos, use fabricated or de-identified records. This standalone dashboard has no authentication, access controls, or persistence layer for clinical deployment.
