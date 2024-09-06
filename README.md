# Graduation project: Analyze Panasonic's 72-hour warranty speed using Power BI and DAX tools.

Import the Excel file.(pic,provinceofasc,rawdatafromtoolbriefv2)
Adjust the column headers.
Format the columns.
Check the relationships.
Create additional relationships.
From the creation time column (Table rawdatafromtoolbriefv2), create additional columns for Date-Month-Year:

## Use DAX:
# Create a month column: 
Month = MONTH(rawdata[Creation Date])
# Create a year column: 
Year = YEAR(rawdata[Creation Date])
# Create a column: 
TAT_hour = WC - creation time.
# Create a classification column: TAT_hour > 72 then 72H-FAIL, TAT_hour <= 72 then 72H-OK.
TAT_Result = IF(rawdata[TAT_hour] > 72, "72H-FAIL", "72H-OK")
# Create a new column named Part Use: 
Part Use = IF(ISBLANK(rawdata[Part Change]), BLANK(), IF(SEARCH("PSV-SO", rawdata[NSC Order Number], 1, 0) > 0, "order", "avlb"))

## Create the following Measures:
# Measure to calculate the total 72h-ok cases (72h-ok):
72h-ok = COUNTAX(FILTER(rawdata, rawdata[TAT_Result] = "72H-OK"), rawdata[TAT_Result])
# Measure to calculate the total 72h-fail cases (72h-fail):
72h-fail = COUNTAX(FILTER(rawdata, rawdata[TAT_Result] = "72H-FAIL"), rawdata[TAT_Result])
# Measure to calculate the total 72h cases (72h total = 72h-ok + 72h-fail):
72h total = [72h-ok] + [72h-fail]
# Measure to calculate the 72h-ok ratio:
72h-ok Ratio = DIVIDE([72h-ok], [72h-ok] + [72h-fail], 0)
# Measure to calculate the cases with parts ordered (Part Order):
Part Order = COUNTAX(FILTER(rawdata, rawdata[Part Use] = "order"), rawdata[Part Use])
# Measure to calculate the cases with parts available (Part Avlb):
Part Avlb = COUNTAX(FILTER(rawdata, rawdata[Part Use] = "avlb"), rawdata[Part Use])
# Measure to calculate the total cases using parts (Part Use Total = Part Order + Part Avlb):
Part Use Total = [Part Order] + [Part Avlb]
# Measure to calculate the parts availability ratio (Part Avlb Ratio):
Part Avlb Ratio = DIVIDE([Part Avlb], [Part Use Total], 0)
# Measure to calculate the ratio of cases with parts used (Part Use Ratio):
Part Use Ratio = DIVIDE([Part Use Total], [72H Total], 0)
# Measure to calculate the cases with parts ordered (Part Order):
Part Order = COUNTAX(FILTER(rawdata, rawdata[Part Use] = "order"), rawdata[Part Use])
# Measure to calculate the cases with parts available (Part Avlb):
Part Avlb = COUNTAX(FILTER(rawdata, rawdata[Part Use] = "avlb"), rawdata[Part Use])
# Measure to calculate the total cases using parts (Part Use Total = Part Order + Part Avlb):
Part Use Total = [Part Order] + [Part Avlb]
# Measure to calculate the parts availability ratio:
Part Avlb Ratio = DIVIDE([Part Avlb], [Part Use Total], 0)
# Measure to calculate the ratio of cases with parts used:
Part Use Ratio = DIVIDE([Part Use Total], [72H Total], 0)
# Create a column to categorize the Fail Reason:
Fail Reason = 
SWITCH(
    TRUE(),
    rawdata[TAT_Result] = "72H-OK", BLANK(),
    rawdata[TAT_Result] = "72H-FAIL" && ISBLANK(rawdata[Part Use]), "Others",
    rawdata[TAT_Result] = "72H-FAIL" && rawdata[Part Use] = "order", "Part",
    rawdata[TAT_Result] = "72H-FAIL" && rawdata[Part Use] = "avlb", "Part",
    BLANK()  -- Default case if none of the conditions match
)
# Create a measure to calculate the fail cases due to other reasons (Reason Others):
Reason Others = COUNTAX(FILTER(rawdata, rawdata[Fail Reason] = "Others"), rawdata[Fail Reaso
