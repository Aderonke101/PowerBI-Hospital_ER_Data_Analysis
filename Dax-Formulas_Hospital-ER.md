--- 
### DAX Formulas and Date Table Implementation for Hospital Emergency Room Project

## Overview

This document provides a comprehensive explanation of the DAX formulas used in the Hospital Emergency Room project, detailing how a new date table was created and linked to the main dataset. Additionally, it explains the added columns for age group, age bracket, and time shift.

## Creating and Linking a Date Table

A new date table was created to ensure proper time-based analysis. This table includes additional columns for year, month, week type, weekday, and month number.

## Date Table DAX:

---
1. Date:

Date =
ADDCOLUMNS(
    CALENDARAUTO(),
    "Year", YEAR([Date]),
    "Month", FORMAT([Date], "mmm"),
    "Weektype", IF(WEEKDAY([Date])=1, "Weekend", IF(WEEKDAY([Date])=7, "Weekend", "Weekday")),
    "Weekday", FORMAT([Date], "ddd"),
    "MonthNum", MONTH([Date])
)

This new date table was linked to the dataset's date field for time-based analysis.

## DAX Formulas Used in the Analysis

## Patient Demographics and Visit Percentage

2. % Female Visit:

% Female Visit =
DIVIDE(
    CALCULATE(
        [Total Patients],
        'Patients Clinical Dataset'[patient_gender]="F"
    ),
    [Total Patients]
)

3. % Male Visit:

% Male Visit =
DIVIDE(
    CALCULATE(
        [Total Patients],
        'Patients Clinical Dataset'[patient_gender]="M"
    ),
    [Total Patients]
)

4. % Unclear Gender Visit:

% Unclear =
DIVIDE(
    CALCULATE(
        [Total Patients],
        'Patients Clinical Dataset'[patient_gender]="NC"
    ),
    [Total Patients]
)

## Administrative and Referral Analysis

5. % Healthcare-Admin Scheduled Visits:

% Healthcare-Admin Schedule =
DIVIDE(
    COUNTROWS(
        FILTER(
            'Patients Clinical Dataset',
            'Patients Clinical Dataset'[patient_admin_flag]=TRUE()
        )
    ),
    [Total Patients]
)

6. % None - Healthcare-Admin Schedule:

None - Healthcare-Admin Schedule =
DIVIDE(
    COUNTROWS(
        FILTER(
            'Patients Clinical Dataset',
            'Patients Clinical Dataset'[patient_admin_flag]=FALSE()
        )
    ),
    [Total Patients]
)

7. % Referred Patients:

% Referred Patients =
VAR _FilterPatients =
    CALCULATE(
        [Total Patients],
        'Patients Clinical Dataset'[department_referral]<>"none"
    )
RETURN
DIVIDE(
    _FilterPatients,
    [Total Patients]
)

8. % Self-Referred Patients:

% Self-Referred Patients =
VAR _FilterPatients =
    CALCULATE(
        [Total Patients],
        'Patients Clinical Dataset'[department_referral]="none"
    )
RETURN
DIVIDE(
    _FilterPatients,
    [Total Patients]
)

## Satisfaction and Wait Time Metrics

9. Average Satisfaction Score:

Average Satisfaction Score =
CALCULATE(
    AVERAGE('Patients Clinical Dataset'[Patient_satisfaction]),
    'Patients Clinical Dataset'[Patient_satisfaction]<>BLANK()
)

10. Average Wait Time:

Average Wait Time = AVERAGE('Patients Clinical Dataset'[patient_waittime])

## Classification of Patients by Age

11. Age Group:

Age Group =
VAR _PatientAge = 'Patients Clinical Dataset'[patient_age]
RETURN
IF(
    _PatientAge<=2, "Infant",
    IF( _PatientAge<=6, "Early Childhood",
         IF(_PatientAge<=12, "Middle Childhood",
             IF(_PatientAge<=18,"Teenager",
            "Adult"
            )
         )
    )
)

12. Age Brackets:

Age Brackets =
SWITCH(
    TRUE(),
    'Patients Clinical Dataset'[patient_age]<=10, "0-10",
    'Patients Clinical Dataset'[patient_age]<=20, "11-20",
    'Patients Clinical Dataset'[patient_age]<=30, "21-30",
    'Patients Clinical Dataset'[patient_age]<=40, "31-40",
    'Patients Clinical Dataset'[patient_age]<=50, "41-50",
    'Patients Clinical Dataset'[patient_age]<=60, "51-60",
    'Patients Clinical Dataset'[patient_age]<=70, "61-70",
    "70+"
)

## Conditional Formatting and Parameter Table

13. Color Scale Descriptor:

Colour Scale Descriptor =
VAR _SelectedMeasure =
    SELECTEDVALUE(Parameter[Parameter Order])
RETURN
IF( _SelectedMeasure=0,
   "The darker GREEN on the Scale indicates higher wait time for the Age-Group",
   "Patients are most Satisfied when the Scale is at its lightest shade"
)

## Parameter Table

14. Parameter = {
    ("Avg. Satisfaction", NAMEOF('Calculations'[Average Satisfaction Score]), 0),
    ("Avg. Wait Time", NAMEOF('Calculations'[Average Wait Time]), 1)
}

15. CF Max Point (Month):
VAR _patientTable =
    CALCULATETABLE(
        ADDCOLUMNS(
            SUMMARIZE('Date','Date'[Month]),
            "@Total_Patients",[Total Patients]
        ),
        ALLSELECTED()
    )
VAR _minValue = MINX(_patientTable, [@Total_Patients])
VAR _MaxValue = MAXX(_patientTable, [@Total_Patients])
VAR _TotalPatients = [Total Patients]
RETURN
SWITCH(
    TRUE(),
    _TotalPatients = _minValue, 0,
    _TotalPatients = _MaxValue, 1
)

16. CF Max Point (Year):
VAR _patientTable =
    CALCULATETABLE(
        ADDCOLUMNS(
            SUMMARIZE('Date','Date'[Year]),
            "@Total_Patients",[Total Patients]
        ),
        ALLSELECTED()
    )
VAR _minValue = MINX(_patientTable, [@Total_Patients])
VAR _MaxValue = MAXX(_patientTable, [@Total_Patients])
VAR _TotalPatients = [Total Patients]
RETURN
SWITCH(
    TRUE(),
    _TotalPatients = _minValue, 0,
    _TotalPatients = _MaxValue, 1
)

17. Value Max Point (Month):
VAR _patientTable =
    CALCULATETABLE(
        ADDCOLUMNS(
            SUMMARIZE('Date','Date'[Month]),
            "@Total_Patients",[Total Patients]
        ),
        ALLSELECTED()
    )
VAR _minValue = MINX(_patientTable, [@Total_Patients])
VAR _MaxValue = MAXX(_patientTable, [@Total_Patients])
VAR _TotalPatients = [Total Patients]
RETURN
SWITCH(
    TRUE(),
    _TotalPatients = _minValue, [Total Patients],
    _TotalPatients = _MaxValue, [Total Patients]
)

18. Value Max Point (Year):
VAR _patientTable =
    CALCULATETABLE(
        ADDCOLUMNS(
            SUMMARIZE('Date','Date'[Month]),
            "@Total_Patients",[Total Patients]
        ),
        ALLSELECTED()
    )
VAR _minValue = MINX(_patientTable, [@Total_Patients])
VAR _MaxValue = MAXX(_patientTable, [@Total_Patients])
VAR _TotalPatients = [Total Patients]
RETURN
SWITCH(
    TRUE(),
    _TotalPatients = _minValue, 0,
    _TotalPatients = _MaxValue, 1
)

19. Total patients:
COUNTROWS('Patients Clinical Dataset')

Conclusion

A new date table was created and linked to the main dataset for proper time-based analysis.

Various DAX formulas were used to analyze patient visits, administrative scheduling, referrals, satisfaction, wait times, and age classifications.

The project included the addition of Age Group, Age Bracket, and Time Shift (AM/PM) for enhanced data slicing and visualization.

This README serves as documentation for understanding and maintaining the DAX formulas used in this analysis.

