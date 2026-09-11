## CBB6340

# Daily New COVID-19 Cases (Project 9.10)

The following figure compares daily new COVID-19 cases in California,
New York, and Texas.

<img width="1389" height="690" alt="daily_cases" src="https://github.com/user-attachments/assets/cec533f1-68ef-46d7-a197-93018aa8b725" />

### Limitations

Daily case counts were calculated by taking the difference between consecutive
cumulative case totals. This approach may produce negative values when historical
records are corrected. The results may also contain reporting spikes caused by
weekend delays, reporting backlogs, and differences in reporting practices among
states.

### Peak Case Dates Examples

{'state': 'California', 'date': '2022-01-10', 'daily_cases': 227972}
{'state': 'New York', 'date': '2022-01-08', 'daily_cases': 90132}
{'state': 'Texas', 'date': '2022-01-03', 'daily_cases': 164902}

### Compare Dates Examples (CA & NY)
California peak date: 2022-01-10
New York peak date: 2022-01-08
New York reached the peak first.
Days between peaks: 2

## Florida Data Exploration and Anomalies

<img width="1387" height="590" alt="florida daily new cases" src="https://github.com/user-attachments/assets/da18ca8c-e201-476a-b9fb-642167fbe447" />

Florida's daily case data contains several unusual reporting patterns. For
example, the calculated number of new cases on June 4, 2021, is -40,527.
A negative number of actual new infections is not possible, so this observation
likely resulted from a correction to the cumulative total, such as the removal
of duplicate or incorrectly classified cases.

Another anomaly occurs on January 4, 2022, when 193,786 new cases were recorded
after three consecutive days with no reported new cases. This spike may represent
multiple days of cases reported together following the New Year holiday rather
than infections that all occurred on January 4.

The data also contains repeated sequences of zero values followed by large
increases, particularly toward the end of the dataset. A likely explanation is
that Florida changed from daily reporting to less frequent batch reporting.
Therefore, the calculated daily case values represent the dates cases were
reported, not necessarily the dates on which infections occurred.
