#Graduation project: Analyze Panasonic's 72-hour warranty speed using Power BI and DAX tools.

Import the Excel file.
Adjust the column headers.
Format the columns.
Check the relationships.
Create additional relationships.
From the creation time column, create additional columns for Date-Month-Year:

Create a month column: Month = MONTH(rawdata[Creation Date])
Create a year column: Year = YEAR(rawdata[Creation Date])
