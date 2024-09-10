# Graduation project: Analyze Panasonic's 72-hour warranty speed using Power BI and DAX tools---> Target: 90%
## *I.Purpose of project.*
- Calculate the warranty speed of Panasonic Viet Nam in 3 months in Southeast of Viet Nam.
- `KPI target: Completed the job within 72H after creation, target 90%. Analyze the time of resolution, analyze which reason make prolong processing time → Find the reason and propose solution.`
- The tool for graduation project is Power BI.
## *II.Project details*

- Use the internal data source from Sales Force.
- Calculate statistical parameters (subtract the completion date and time from the creation date and time)
- Count number of job meet target.
- Count number of job fail target.
- Show number of job meet target 72H / total jobs. (Southeast of Viet Nam)
- Count number of job that use spare part.
- Calculate ratio of job that use spare part. (Part available, part not available [must order])
- Use power BI to visualize the graphic (Column, Bar chart, Pie, donut…)
- Find the reason lead to Fail (if yes)
- Conclusion.
- Propose solution.
  
## *III.Project implementation*

- Download data excel from Salesforce.

![SF report](https://github.com/user-attachments/assets/81971f58-4d62-47d4-ba13-504ac92df2b1)

![image](https://github.com/user-attachments/assets/8e12ee6d-ee94-46a0-b790-9b026a16930e)

- Import the Excel file.(pic,provinceofasc,rawdatafromtoolbriefv2)

![Import data](https://github.com/user-attachments/assets/c42fb116-8e02-4d5a-bb34-aad74d4a5a21)

- Clean data (Check for blank values, columns with incorrect formatting, incorrect dates, etc., fill in the missing information and correct the formatting...)
- Check column headers and adjust them if needed.
- Check the formatting of the columns and reformat if necessary. 
- Check the relationships between tables.
- 
![Check data type(3)](https://github.com/user-attachments/assets/7de0126a-eac1-473e-866c-f6d94a683b91)

![Check data type](https://github.com/user-attachments/assets/70d7ac0b-ab50-409e-b3f9-f8f62a26ad85)

![Show relationship](https://github.com/user-attachments/assets/18446141-45fa-4fbe-a5f7-adb9e6899ec5)

- Create additional relationships. 

![Add relationship](https://github.com/user-attachments/assets/b0d6c05a-1027-424a-b1ed-1c7532d0fc9d)

### Use DAX: At Table rawdata, create column & measure to calculate.
- Create a month column:
```dax
Month = MONTH(rawdata[Creation Date])
```
- Create a year column:
```dax
Year = YEAR(rawdata[Creation Date])
```
- Create a column:
```dax
TAT_hour = ROUND((rawdata[Work Completed Time (1st)] - rawdata[Creation Time]) * 24, 2)
```
- Create a classification column: TAT_hour > 72 then 72H-FAIL, TAT_hour <= 72 then 72H-OK.
```dax
TAT_Result = IF(rawdata[TAT_hour] > 72, "72H-FAIL", "72H-OK")
```
- Create a new column named Part Use:
```dax
Part Use = IF(ISBLANK(rawdata[Part Change]), BLANK(), IF(SEARCH("PSV-SO", rawdata[NSC Order Number], 1, 0) > 0, "order", "avlb"))
```

### Create the following Measures:
- Measure to calculate the total 72h-ok cases (72h-ok):
```dax
72h-ok = COUNTAX(FILTER(rawdata, rawdata[TAT_Result] = "72H-OK"), rawdata[TAT_Result])
```
- Measure to calculate the total 72h-fail cases (72h-fail):
```dax
72h-fail = COUNTAX(FILTER(rawdata, rawdata[TAT_Result] = "72H-FAIL"), rawdata[TAT_Result])
```
- Measure to calculate the total 72h cases (72h total = 72h-ok + 72h-fail):
```dax
72h total = [72h-ok] + [72h-fail]
```
- Measure to calculate the 72h-ok ratio:
```dax
72h-ok Ratio = DIVIDE([72h-ok], [72h-ok] + [72h-fail], 0)
```
- Measure to calculate the cases with parts ordered (Part Order):
```dax
Part Order = COUNTAX(FILTER(rawdata, rawdata[Part Use] = "order"), rawdata[Part Use])
```
- Measure to calculate the cases with parts available (Part Avlb):
```dax
Part Avlb = COUNTAX(FILTER(rawdata, rawdata[Part Use] = "avlb"), rawdata[Part Use])
```
- Measure to calculate the total cases using parts (Part Use Total = Part Order + Part Avlb):
```dax
Part Use Total = [Part Order] + [Part Avlb]
```
- Measure to calculate the parts availability ratio (Part Avlb Ratio):
```dax
Part Avlb Ratio = DIVIDE([Part Avlb], [Part Use Total], 0)
```
- Measure to calculate the ratio of cases with parts used (Part Use Ratio):
```dax
Part Use Ratio = DIVIDE([Part Use Total], [72H Total], 0)
```
- Measure to calculate the cases with parts ordered (Part Order):
```dax
Part Order = COUNTAX(FILTER(rawdata, rawdata[Part Use] = "order"), rawdata[Part Use])
```
- Measure to calculate the cases with parts available (Part Avlb):
```dax
Part Avlb = COUNTAX(FILTER(rawdata, rawdata[Part Use] = "avlb"), rawdata[Part Use])
```
- Measure to calculate the total cases using parts (Part Use Total = Part Order + Part Avlb):
```dax
Part Use Total = [Part Order] + [Part Avlb]
```
- Measure to calculate the parts availability ratio:
```dax
Part Avlb Ratio = DIVIDE([Part Avlb], [Part Use Total], 0)
```
- Measure to calculate the ratio of cases with parts used:
```dax
Part Use Ratio = DIVIDE([Part Use Total], [72H Total], 0)
```

#### Create a column to categorize the Fail Reason:
Fail Reason = 
SWITCH(
    TRUE(),
    rawdata[TAT_Result] = "72H-OK", BLANK(),
    rawdata[TAT_Result] = "72H-FAIL" && ISBLANK(rawdata[Part Use]), "Others",
    rawdata[TAT_Result] = "72H-FAIL" && rawdata[Part Use] = "order", "Part",
    rawdata[TAT_Result] = "72H-FAIL" && rawdata[Part Use] = "avlb", "Part",
    BLANK()  -- Default case if none of the conditions match
)
#### Create a measure to calculate the fail cases due to other reasons (Reason Others):
Reason Others = COUNTAX(FILTER(rawdata, rawdata[Fail Reason] = "Others"), rawdata[Fail Reason])

## Output 

![image](https://github.com/user-attachments/assets/b281fe13-ce95-479a-bc1c-0babf045b0e1)

## Conclusion
- Result of 72-hour warranty speed: 88.5% Vs Target: 90%
- Reason of FAIL: Mainly beacause of spare part (60%) and other reasons (40%)
- HCM related to 47% of spare parts.
- Spare part use ratio over the total number of jobs: 23.8%
- Available spare part ratio over the total number of spare part used: 29.7%
- Take care especialy in: HCM, Dong Nai, Binh Duong, Long An, Tay Ninh, BR-VT...

## Solutions:
- The warehouse department should plan to stock enough parts and part numbers.
- Prioritize key areas: HCM, Dong Nai, Binh Duong, Long An...
- Accelerate the distribution speed of parts to 24 hours for provinces around HCM, and 36 hours for other provinces.
- Encourage service stations to stock spare parts.
