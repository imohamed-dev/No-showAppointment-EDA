# Medical Appointment No-Shows: Exploratory Data Analysis

An exploratory analysis of why patients miss medical appointments in Brazil, using the public **Medical Appointment No Shows** dataset (Kaggle, May 2016). The notebook identifies which factors are associated with no-shows, measures how strong each association is, and prepares the ground for a predictive model.

**Notebook:** [`ExploratoryDataAnalysis.ipynb`](ExploratoryDataAnalysis.ipynb)

## Problem

About **20%** of scheduled appointments are missed. Missed appointments waste clinic capacity and block access for other patients. The goal of this analysis is to find which features are associated with no-shows and to turn those findings into testable ideas for reducing them.

**Primary KPI:** no-show rate.

## Dataset

- **Source:** [Medical Appointment No Shows (Kaggle)](https://www.kaggle.com/datasets/joniarroba/noshowappointments)
- **Size:** 110,527 appointments, 14 columns, no missing values
- **Target:** `No-show` (Yes = the patient did not attend), about 20% positive
- **Features:** patient demographics (age, gender, neighbourhood), health conditions (hypertension, diabetes, alcoholism, handicap), scholarship (a Brazilian welfare program), SMS reminder received, scheduling and appointment dates

The dataset is included in this repository. You can also download `KaggleV2-May-2016.csv` from Kaggle and place it in the same folder as the notebook.

## Approach

1. **Inspection:** structure, data types, missing values, summary statistics.
2. **Cleaning:** lower-cased column names, removed rows with impossible values (negative age, appointments dated before they were booked), recoded `handicap` as a binary flag, and encoded the target as 0/1.
3. **Feature engineering:**
   - `waitingdays`: days between booking and appointment.
   - Weekday of booking and of appointment.
   - **Patient history features** (`prev_appt`, `prev_noshow`, `noshow_rate`, `days_since_last_appt`, `is_first_appnt`, `is_returning`), built **point-in-time**: an earlier appointment only counts if it was completed before the current appointment was booked. This avoids leaking future outcomes into the features.
4. **Univariate and bivariate analysis:** distributions and no-show rates for every feature.
5. **Statistical tests**, with effect sizes alongside p-values, because with 110k rows almost everything is statistically significant:
   - Continuous features: Welch's t-test, point-biserial correlation, Spearman correlation.
   - Categorical features: chi-square test with bias-corrected Cramér's V (Weak below 0.10, Moderate up to 0.25, Strong above 0.25).
6. **Feature interaction:** waiting time vs SMS reminders.

## Key Findings

| Tier | Feature | Evidence |
|---|---|---|
| **1** | **Waiting time** | Point-biserial r = 0.19, Spearman ρ = 0.28. The no-show rate rises from about 5% for same-day appointments to about 38% for waits of three weeks or more (among patients without an SMS). |
| **1** | **SMS reminder** | Cramér's V = 0.13 (Moderate). Confounded by lead time (see below). |
| 2 | Age group, neighbourhood, first-time vs returning, hypertension, scholarship | Cramér's V between 0.03 and 0.08 (Weak). |
| 2 | History features (`noshow_rate`, `prev_noshow`, `days_since_last_appt`) | \|r\| ≤ 0.06, practically weak. |
| 3 | Gender, scheduling weekday, alcoholism | No statistically significant association. |

### The SMS paradox

Looking at all patients together, those who received an SMS miss *more* appointments. This is misleading: SMS reminders are mostly sent for appointments booked far in advance (same-day appointments never receive one), and those appointments are riskier to begin with.

Comparing patients **within** the same waiting-time group reverses the picture. For waits longer than a week, SMS recipients miss fewer appointments (for example 28.8% vs 34.7% for 8-21 days). This is an association in observational data, not proof that SMS reminders cause better attendance.

### Neighbourhood

Neighbourhood no-show rates mostly sit between about 15% and 23%, close to the overall rate. The most extreme rates come from neighbourhoods with only one or two appointments and should not be interpreted.

## Recommendations

These are ideas to test, not conclusions:

- Reduce waiting times where possible, and place overflow patients on a dynamic waiting list.
- Test whether sending SMS reminders to more patients lowers no-shows (for example with a pilot or A/B test).
- Let patients confirm or cancel by reply, so cancelled slots can be filled in advance.

## Limitations

- The data covers a few months of 2016 from one region, so patterns may not transfer to other clinics or periods.
- All findings are associations. The SMS analysis in particular cannot separate the effect of the reminder from the clinic's choice of who receives one.
- Patient history is short because of the limited time window, so most patients have little or no prior record.

## Next Steps

A predictive modeling notebook (classification of no-shows), with these points carried over from the EDA:

- Rebuild history features point-in-time, and exclude the target from the features.
- Fit encoders and scalers on the training data only.
- Use metrics suited to a 20% positive class (PR-AUC and recall, not just accuracy).

## How to Run

1. Install Python 3.11 or later.
2. Install the dependencies, with either pip or Poetry:
   ```bash
   # Option A: pip
   pip install -r requirements.txt

   # Option B: Poetry (uses pyproject.toml)
   poetry install
   ```
3. Download `KaggleV2-May-2016.csv` from [Kaggle](https://www.kaggle.com/datasets/joniarroba/noshowappointments) and put it next to the notebook.
4. Open the notebook and run all cells:
   ```bash
   jupyter notebook ExploratoryDataAnalysis.ipynb
   ```

## Repository Structure

```
.
├── ExploratoryDataAnalysis.ipynb   # the full analysis, with outputs
├── requirements.txt                # dependencies (pip)
├── pyproject.toml                  # dependencies (Poetry)
├── .gitignore
├── KaggleV2-May-2016.csv           # included, download from Kaggle
└── README.md
```

## References

- Hoppen, J. *Medical Appointment No Shows* (KaggleV2-May-2016). Kaggle.
- Bergsma, W. (2013). A bias-correction for Cramér's V and Tschuprow's T. *Journal of the Korean Statistical Society*, 42(3), 323-328.
- Welch, B. L. (1947). The generalization of "Student's" problem when several different population variances are involved. *Biometrika*, 34(1-2), 28-35.
