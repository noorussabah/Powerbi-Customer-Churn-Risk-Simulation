# DAX Measures Library

This document contains all DAX measures and calculated columns used in the Customer Churn Risk & Revenue Impact Dashboard.

Measures are organized by dashboard page to match the structure of the Power BI report.

## Page 1 – Overview

### Total Customers
```DAX
Total Customers :=
VAR _value =
    DISTINCTCOUNT(Telco_Churn[CustomerID])
RETURN
IF(
    ISBLANK(_value),
    BLANK(),
    _value
)
```
Format: Whole Number

### Churn Rate %
```DAX
Churn Rate % :=
VAR _value =
    DIVIDE(
        CALCULATE(
            COUNTROWS(Telco_Churn),
            Telco_Churn[Churn Label] = "Yes"
        ),
        DISTINCTCOUNT(Telco_Churn[CustomerID])
    )
RETURN
IF(
    ISBLANK(_value),
    BLANK(),
    _value
)
```
Format: Percentage (2 decimal places)

### Revenue at Risk
```DAX
Revenue at Risk :=
VAR _value =
    CALCULATE(
        SUM(Telco_Churn[Monthly Charges]),
        Telco_Churn[Churn Label] = "Yes"
    )
RETURN
IF(
    ISBLANK(_value),
    BLANK(),
    _value
)
```
Format: Currency

### CLTV Lost
```DAX
CLTV Lost :=
VAR _value =
    CALCULATE(
        SUM(Telco_Churn[CLTV]),
        Telco_Churn[Churn Label] = "Yes"
    )
RETURN
IF(
    ISBLANK(_value),
    BLANK(),
    _value
)
```
Format: Currency

### High Risk %
High Risk defined as Churn Score ≥ 80.

```DAX
High Risk % :=
VAR _value =
    DIVIDE(
        CALCULATE(
            COUNTROWS(Telco_Churn),
            Telco_Churn[Churn Score] >= 80
        ),
        DISTINCTCOUNT(Telco_Churn[CustomerID])
    )
RETURN
IF(
    ISBLANK(_value),
    BLANK(),
    _value
)
```
Format: Percentage

### Calculated Column – Risk Segment
```DAX
Risk Segment =
SWITCH(
    TRUE(),
    Telco_Churn[Churn Score] >= 80, "High Risk",
    Telco_Churn[Churn Score] >= 50, "Medium Risk",
    "Low Risk"
)
```

## Page 2 – Risk & Segmentation

### High Risk Customers
```DAX
High Risk Customers :=
VAR _value =
    CALCULATE(
        DISTINCTCOUNT(Telco_Churn[CustomerID]),
        Telco_Churn[Churn Score] >= 80
    )
RETURN
IF(ISBLANK(_value), BLANK(), _value)
```
Format: Whole Number

### High Risk Revenue
```DAX
High Risk Revenue :=
VAR _value =
    CALCULATE(
        SUM(Telco_Churn[Monthly Charges]),
        Telco_Churn[Churn Score] >= 80
    )
RETURN
IF(ISBLANK(_value), BLANK(), _value)
```
Format: Currency

### Avg CLTV (High Risk)
```DAX
Avg CLTV (High Risk) :=
VAR _value =
    CALCULATE(
        AVERAGE(Telco_Churn[CLTV]),
        Telco_Churn[Churn Score] >= 80
    )
RETURN
IF(ISBLANK(_value), BLANK(), _value)
```
Format: Currency

### Avg CLTV (Low Risk)
```DAX
Avg CLTV (Low Risk) :=
VAR _value =
    CALCULATE(
        AVERAGE(Telco_Churn[CLTV]),
        Telco_Churn[Churn Score] < 50
    )
RETURN
IF(ISBLANK(_value), BLANK(), _value)
```
Format: Currency

### High Risk Revenue %
```DAX
High Risk Revenue % :=
VAR _value =
    DIVIDE(
        CALCULATE(
            SUM(Telco_Churn[Monthly Charges]),
            Telco_Churn[Churn Score] >= 80
        ),
        SUM(Telco_Churn[Monthly Charges])
    )
RETURN
IF(ISBLANK(_value), BLANK(), _value)
```
Format: Percentage

## Page 3 – Retention Scenario Modeling

### Retention Improvement Parameter
What-If parameter settings:

- Minimum: 0
- Maximum: 0.30
- Increment: 0.01
- Default: 0.05

This parameter generates a table, measure, and slider used for scenario simulation.

### Customers Saved
```DAX
Customers Saved :=
VAR _churned =
    CALCULATE(
        DISTINCTCOUNT(Telco_Churn[CustomerID]),
        Telco_Churn[Churn Label] = "Yes"
    )
RETURN
_churned * 'Retention Improvement'[Retention Improvement Value]
```

### Revenue Recovered
```DAX
Revenue Recovered :=
VAR _lost =
    CALCULATE(
        SUM(Telco_Churn[Monthly Charges]),
        Telco_Churn[Churn Label] = "Yes"
    )
RETURN
_lost * 'Retention Improvement'[Retention Improvement Value]
```

### CLTV Recovered
```DAX
CLTV Recovered :=
VAR _lost =
    CALCULATE(
        SUM(Telco_Churn[CLTV]),
        Telco_Churn[Churn Label] = "Yes"
    )
RETURN
_lost * 'Retention Improvement'[Retention Improvement Value]
```

### Adjusted Churn Rate
```DAX
Adjusted Churn Rate :=
VAR _current =
    [Churn Rate %]
VAR _improvement =
    'Retention Improvement'[Retention Improvement Value]
RETURN
_current * (1 - _improvement)
```

### Retention Impact %
```DAX
Retention Impact % :=
'Retention Improvement'[Retention Improvement Value]
```

### Scenario Modeling Concept
The retention simulation uses a What-If parameter to dynamically estimate the impact of improving customer retention.

This enables decision makers to evaluate:

- Potential customers saved
- Revenue recovered through improved retention
- Lifetime value preserved
- Reduction in overall churn rate

This transforms churn analysis into a forward-looking strategic decision tool rather than a purely descriptive report.

## Notes

- Measures assume the dataset structure from the Kaggle Telco Customer Churn dataset.
- Defensive DAX patterns are used to prevent invalid results or blank errors.
- KPI calculations are optimized for interactive filtering within Power BI.
