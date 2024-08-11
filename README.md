# HR-Analytics-in-Power-BI
I am using fictitious datasets from a Tech Company called Atlas Labs

**Primary Goal**: Atlas Labs HR team want to be able to monitor key metrics on employees
<br>
**Secondary Goal**: Understand what factors impact employee attrition

I will working with 1 Fact Table, and 5 Dimension Tables (```Employee```,```EducationLevel```,```RatingLevel```,```SatisfiedLevel``` & ```Date```)
The final data model will follow a snowflake schema.

## First Step: Loading and Preparing Dataset

### 1.Import dataset
Import all the dataset, rename the dataset by add either ```Fact``` or ```Dim``` at the beginning of each table name, depending on the type of table it is.
<br>
Review all the dataset, and ensure that the columns are correctly formatted as text, numbers, and dates where expected.


### 2.Create a dedicated date table
Create a new calculated table that uses the DAX code from ```Dimdate.text```.
```dax
DimDate = 
VAR _minYear = YEAR(MIN(DimEmployee[HireDate]))
VAR _maxYear = YEAR(MAX(DimEmployee[HireDate]))
VAR _fiscalStart = 4 

RETURN
ADDCOLUMNS(

    CALENDAR(

                DATE(_minYear,1,1),

                DATE(_maxYear,12,31)

),

"Year",YEAR([Date]),
"Year Start",DATE( YEAR([Date]),1,1),
"YearEnd",DATE( YEAR([Date]),12,31),
"MonthNumber",MONTH([Date]),
"MonthStart",DATE( YEAR([Date]), MONTH([Date]), 1),
"MonthEnd",EOMONTH([Date],0),
"DaysInMonth",DATEDIFF(DATE( YEAR([Date]), MONTH([Date]), 1),EOMONTH([Date],0),DAY)+1,
"YearMonthNumber",INT(FORMAT([Date],"YYYYMM")),
"YearMonthName",FORMAT([Date],"YYYY-MMM"),
"DayNumber",DAY([Date]),
"DayName",FORMAT([Date],"DDDD"),
"DayNameShort",FORMAT([Date],"DDD"),
"DayOfWeek",WEEKDAY([Date]),
"MonthName",FORMAT([Date],"MMMM"),
"MonthNameShort",FORMAT([Date],"MMM"),
"Quarter",QUARTER([Date]),
"QuarterName","Q"&FORMAT([Date],"Q"),
"YearQuarterNumber",INT(FORMAT([Date],"YYYYQ")),
"YearQuarterName",FORMAT([Date],"YYYY")&" Q"&FORMAT([Date],"Q"),
"QuarterStart",DATE( YEAR([Date]), (QUARTER([Date])*3)-2, 1),
"QuarterEnd",EOMONTH(DATE( YEAR([Date]), QUARTER([Date])*3, 1),0),
"WeekNumber",WEEKNUM([Date]),
"WeekStart", [Date]-WEEKDAY([Date])+1,
"WeekEnd",[Date]+7-WEEKDAY([Date]),
"FiscalYear",if(_fiscalStart=1,YEAR([Date]),YEAR([Date])+ QUOTIENT(MONTH([Date])+ (13-_fiscalStart),13)),
"FiscalQuarter",QUARTER( DATE( YEAR([Date]),MOD( MONTH([Date])+ (13-_fiscalStart) -1 ,12) +1,1) ),
"FiscalMonth",MOD( MONTH([Date])+ (13-_fiscalStart) -1 ,12) +1
)
```

Once a new table is created, connect 'DimDate' Tble with ```FactPerformanceRating``` table and ```DimEmployee``` Table. And it will shows like the figure belowed.
Now we can see that there are ten relationships between all six tables in our data model.
<div align="center">
  
![image](https://github.com/user-attachments/assets/e4c56992-b1aa-48a4-9060-0c016f98ef92)
  
</div>

### 3.Calculate the key measure
Now I am looking to have visibility on high-level metrics about the state of the employees. Trying to understand the attrition at the company. So I am going to explore the avaliable data and calculate this key measure that will be useful throughout the project.

<br>

Create a new table called ```_Measures```, and then create a ```TotalEmployees``` Measure inside the table, which takes the count of all employees.

The DAX code:
```dax
TotalEmployees = COUNT(DimEmployee[EmployeeID])
```
Create a new measure called ```Active Employees```
```dax
Active Employee = CALCULATE ([TotalEmployees], DimEmployee[Attrition] = "No")
```
<br>

Create a new measure called ```Inactive Employees```

```dax
Inactive Employee = CALCULATE ([TotalEmployees], DimEmployee[Attrition] = "Yes"
```
<br>
These two measures takes the count of all employees that are currently active or inactive, respectively. I used ```DimEmployee[Attrition]``` to determine whether an employeeis active or inactive.
<br>
Then I calculate the '%Attrition Rate' based on these measures. 
<div align="center">
  
![image](https://github.com/user-attachments/assets/59c3b291-7f2f-41a0-810c-f6ea2d62fb3a)

</div>


### 4. Activate relationships between tables.

Now I would like to start with analyzing company's hiring trends over time to see where Hiring Manager see the biggest growth in new employees. This will enable Hiring Team to be able to benchmark their HR metrics against organizations across their industry as well as understand how their employees are performing.
<br>
In this step, I am looking to activate relationships between tables. ```DimDate``` already has an active relationship with ```FactPerformanceRating```. Therefore I will need to utilize ```USERELATIONSHIP()``` to count the number of employees by date.
<br>
Create a new measure called ```TotalEmployeesDate``` that uses the ```CALCULATE()``` function on the previous measure and the ```USERELATIONSHIP()``` function in the filter.
```dax
TotalEmployeesDate =
CALCULATE([TotalEmployees],
USERELATIONSHIP (DimEmployee [HireDate], DimDate[Date]))
```

The figure shows like :

<div align="center">
  
![image](https://github.com/user-attachments/assets/7df49f8e-0c17-4789-8f92-62605f159067)
</div>

### 4. Analyzing Departments and Job Roles
It is essential to ask the HR team working with department managers to understand their teams and what type of typical roles they are hiring into the organization. This will enable every department to plan for new hiring requests in the future. 
<br>

Create a Culstered bar chart to show ```ActiveEmployees``` by ```Department```.
<div align="center">
  
![image](https://github.com/user-attachments/assets/59b8c437-5028-42c0-9dd2-6bd4e05ecdef)

</div>

Create a treemap to display ```ActiveEmployees``` by ```Department``` and ```JobRole```, and we can see that the Software Engineer is the most common job title for the Technology Team.

<div align="center">
  
![image](https://github.com/user-attachments/assets/305b3ced-45f3-4017-8cbd-9be40b30cd96)

</div>
